# Process, AI Prompt Log & Observations

## Section 1: Process / Approach Note

### Tools Used

- **Firecrawl (MCP Server)**: Cloud-based web scraping API used to extract structured data from Amazon pages. Firecrawl handles JavaScript rendering, anti-bot measures, and provides JSON-structured extraction via LLM-powered parsing.
- **Antigravity (AI Coding Assistant by Google DeepMind)**, powered by **Claude Opus 4.6 Thinking**: Used as the orchestration layer — designing the scraping strategy, managing Firecrawl API calls, consolidating results, cleaning data, and generating deliverables.
- **No local Python environment was used** — the AI assistant (Antigravity) orchestrated the entire pipeline: it issued Firecrawl API calls, parsed the returned JSON outputs, deduplicated overlapping results, cleaned the data, and assembled the final CSV programmatically.

### Approach

The task was broken into three phases:

**Phase 1 — Listing Page Extraction (Ranks 1–50)**

- Used `firecrawl_scrape` with JSON format + a defined schema to extract all books from the bestseller page.
- Amazon's page lazy-loads content beyond rank ~30, so two scrapes were needed: one default and one with scroll actions to trigger the remaining items.
- The two scrapes overlapped at ranks 27–30, which were deduplicated by matching book URLs.
- A third targeted scrape was needed to capture ranks 46–49, which fell in a gap between scrape batches.

**Phase 2 — Individual Book Page Extraction**

- Initially tried `firecrawl_extract` (batch mode) on 10 URLs simultaneously. This returned incomplete data for most books.
- Switched to individual `firecrawl_scrape` calls per book URL with a JSON schema for `description`, `publisher`, and `publication_date`. This was 100% reliable.
- Scraped all 50 book pages in batches of 5 to manage API throughput.

**Phase 3 — Data Cleaning & CSV Export**

- Ratings: kept as numeric decimals (e.g., 4.5).
- Reviews: kept as integers (e.g., 112404).
- Publication dates: standardized to YYYY-MM-DD format where available.
- Prices: retained as-is with currency prefix (GBP or USD depending on scrape session).
- URLs: cleaned of tracking parameters.
- Missing values: marked as `N/A`.

### Issues & Edge Cases

1. **Lazy-loaded content**: Amazon's bestseller page doesn't render all 50 books without scrolling. The first scrape only captured ranks 1–30. A second scrape with explicit scroll actions was needed to load ranks 31–50.
2. **Live ranking shifts causing duplicates and gaps**: Rankings shifted between scrapes — books at ranks 27–30 in scrape 1 appeared at ranks 31–34 in scrape 2. This caused 4 duplicate entries and, after deduplication, a gap where the *actual* books at source ranks 31–34 were missing. Multiple additional targeted scrapes were required to fill these gaps.
3. **Missing ranks discovered during QA**: After the initial CSV was assembled, manual review revealed that 4 books (source ranks 46–49) were silently omitted during the merge. This required another round of listing + individual page scrapes to fill in. The lesson: always verify the final row count and rank sequence.
4. **Batch extraction unreliability**: `firecrawl_extract` (designed for batch URL processing) returned empty or incomplete fields for ~60% of books. Switched to individual `firecrawl_scrape` calls which were 100% reliable but significantly more API calls (50 individual scrapes vs 5 batch calls).
5. **Python environment unavailable**: An initial attempt to write a Python script for automated merging/deduplication failed because the execution environment didn't have Python installed. The AI assistant had to handle all data consolidation within its own context.
6. **Currency inconsistency**: Firecrawl proxies accessed Amazon from different regions across scrape sessions, resulting in GBP prices for ranks 1–30 and USD prices for ranks 31–50. Ideally this would be normalized, but we kept prices as-returned to avoid introducing conversion errors.
7. **Missing publisher/publication_date on some book pages**: Several individual book pages didn't expose publisher or publication date in a way Firecrawl could extract (e.g., pre-order titles, Kindle Unlimited exclusives, or pages with non-standard layouts). These were marked as `N/A` rather than guessed.
8. **Truncated or generic descriptions for some books**: A few individual book scrapes returned shortened or generic descriptions rather than the full blurb, likely due to Amazon's dynamic page rendering or A/B testing on product pages.

### Scaling This Workflow

To adapt this workflow for another Kindle category or many categories at scale:

1. **Parameterize the category URL**: Replace the hard-coded URL with a variable, allowing the same scraping pipeline to target any Amazon bestseller subcategory.
2. **Automate pagination**: Use a loop with scroll actions to dynamically capture all books per page, rather than relying on a fixed number of scrapes.
3. **Batch with rate limiting**: For hundreds of categories, queue scrape jobs with rate limiting (e.g., 5 concurrent requests) to avoid being blocked.
4. **Use a script-based pipeline**: Move from manual orchestration to a Python/Node.js script that:
   - Takes a list of category URLs as input
   - Scrapes each listing page
   - Extracts individual book URLs
   - Scrapes each book page for details
   - Merges, deduplicates, and exports to CSV
5. **Add caching & retry logic**: Cache successful scrapes to avoid redundant API calls, and retry failed ones with exponential backoff.
6. **Schedule regular runs**: Use a cron job or cloud function to run the pipeline daily/weekly for tracking rank changes over time.

---

## Section 2: AI Prompt Log

The prompts below reflect an iterative approach — each prompt was shaped by the results (or failures) of the previous one. Key AI usage decisions are annotated with **[Why]** to explain the reasoning.

### Prompt 1 — Listing Page Scrape

**[Why]** Used JSON format with a strict schema (explicit types for each field) rather than markdown extraction. This ensures ratings come back as numbers (4.5, not "4.5 out of 5 stars"), reviews as integers, and URLs as clean strings — eliminating most post-processing.

```Tool: firecrawl_scrape
URL: https://www.amazon.com/Best-Sellers-Kindle-Store-Paranormal-Romance/zgbs/digital-text/6190484011
Format: JSON
waitFor: 10000
Prompt: "Extract ALL books from this Amazon bestseller page. For each book extract: rank (integer), title (string), author (string), rating (decimal number), number of reviews (integer), price (string), and full book URL (string)"
Schema: { books: [{ rank, title, author, rating, number_of_reviews, price, book_url }] }
```

**Result**: Successfully extracted ranks 1–30 on first call. Only 30 of 50 books appeared because Amazon lazy-loads the rest on scroll.

### Prompt 2 — Listing Page Scrape with Scroll

**[Why]** After Prompt 1 returned only 30 books, added explicit scroll actions to simulate user behavior and trigger Amazon's lazy loading. Deliberately started extraction from rank ~25 to create overlap with Prompt 1 for deduplication safety.

```Same as Prompt 1, but with actions:
- wait 5000ms
- scroll down (x5 with 2000ms waits between)
Prompt: "Extract ALL books from this bestseller page, especially books ranked #25 and higher..."
```

**Result**: Captured ranks 27–50 (with 4 overlapping entries at ranks 27–30 from Prompt 1). Overlap was intentional to avoid gaps.

### Prompt 3 — Individual Book Page Scrape (repeated 50 times)

**[Why]** This was the second strategy attempted. Initially tried batch extraction (see Prompt 5 below), which failed. Pivoted to individual scrapes with a minimal 3-field schema. Individual scrapes were slower (50 API calls vs 5 batch calls) but 100% reliable — reliability was prioritized over speed.

```Tool: firecrawl_scrape
URL: <individual book URL>
Format: JSON
waitFor: 10000
Prompt: "Extract the book description, publisher name, and publication date from this Amazon product page"
Schema: { description: string, publisher: string, publication_date: string }
```

**Result**: Successfully returned data for all 50 books. Some returned N/A for publisher or date — these were kept as N/A rather than guessed.

### Prompt 4 — Targeted Rank Extraction (for missing ranks)

**[Why]** After assembling the CSV, manual QA revealed gaps at specific ranks (31–34 and 46–49). Rather than re-scraping all 50 books, used narrowly targeted prompts asking for only the specific missing ranks. This minimized API calls while filling exact gaps.

```Tool: firecrawl_scrape with scroll actions
Prompt: "Extract ONLY the books ranked #47, #48, #49, and #50..."
Same schema as Prompt 1.
```

**Result**: Used multiple times to fill gaps caused by overlapping/shifting ranks between scrapes. Each targeted call needed its own scroll actions to reach the relevant section of the page.

### Prompt 5 — Failed Batch Extraction Attempt (tried first, abandoned)

**[Why]** Initially tried `firecrawl_extract` for efficiency — it accepts multiple URLs at once. This would have been 5x faster. However, the results were unreliable: most books returned empty fields. The likely cause is that Amazon's individual book pages are JavaScript-heavy SPAs that need full rendering, which batch extraction doesn't handle as well as individual scrapes.

```Tool: firecrawl_extract
URLs: [first 10 book URLs]
Prompt: "Extract the book description, publisher name, and publication date"
Schema: { description, publisher, publication_date }
```

**Result**: Only 4 of 10 books returned complete data. **Decision**: Abandoned batch extraction entirely and switched to individual `firecrawl_scrape` calls (Prompt 3), trading speed for reliability.

---

## Section 3: Three Observations from the Dataset

### Observation 1: Zodiac Academy Dominates the Rankings

Caroline Peckham's **Zodiac Academy** series occupies 8 of the top 50 spots (ranks 6, 7, 9, 10, 12, 17, 27), making it the single most represented series. This suggests that Amazon's bestseller ranking heavily favors established series with large backlists, as readers who discover one book tend to purchase multiple entries, creating a compounding visibility effect.

### Observation 2: Self-Published Authors Dominate; Traditional Publishers Are Nearly Absent

The vast majority of the top 50 are self-published or published through small imprints (e.g., *Special Fiction Books*, *King's Hollow*, *Lyra Mishon*, *Jaymin Eve*). The only recognizable traditional publisher presence is **Blue Box Press** (Jennifer L. Armentrout's books). This indicates that paranormal romance is overwhelmingly an indie/self-published market, where authors can price aggressively (many under $5) and publish rapidly.

### Observation 3: No Correlation Between Rank and Review Count

Books with massive review counts don't necessarily rank higher. For example:

- **Rank 6**: Zodiac Academy 2 has **112,404 reviews**
- **Rank 1**: My Guide Dog is a Wolf Shifter has only **125 reviews**
- **Rank 35**: From Blood and Ash has **80,386 reviews**

This confirms that Amazon's bestseller ranking is based on **recent sales velocity**, not cumulative popularity. Newer releases with fewer reviews can outrank established titles with hundreds of thousands of reviews if they have higher short-term sales momentum.
