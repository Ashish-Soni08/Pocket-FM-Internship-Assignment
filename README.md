# Kindle Bestseller Data Extraction

An AI-assisted web scraping project that extracts structured data from Amazon's [Kindle Best Sellers – Paranormal Romance](https://www.amazon.com/Best-Sellers-Kindle-Store-Paranormal-Romance/zgbs/digital-text/6190484011) page. Built as an intern assignment for **Pocket FM**.

## Project Structure

```
.
├── kindle_paranormal_romance_bestsellers.csv   # Final dataset (50 books)
├── workflow.md                                  # Process note, AI prompt log & observations
├── Intern_assignment.md                         # Original assignment brief
└── LICENSE
```

## What's Inside

### Dataset (`kindle_paranormal_romance_bestsellers.csv`)

50 books scraped from Amazon's Paranormal Romance bestseller list, with the following fields:

| Field | Description |
|---|---|
| `rank` | Bestseller rank (1–50) |
| `title` | Book title |
| `author` | Author name |
| `rating` | Average rating (numeric) |
| `number_of_reviews` | Total review count (integer) |
| `price` | Listed price |
| `book_url` | Direct Amazon product URL |
| `description` | Book blurb (from individual product page) |
| `publisher` | Publisher name |
| `publication_date` | Standardized to YYYY-MM-DD |

### Workflow Document (`workflow.md`)

A detailed write-up covering:
- **Process / Approach** — tools used, scraping strategy, edge cases encountered
- **AI Prompt Log** — every prompt used with Firecrawl, annotated with reasoning
- **3 Observations** — insights from the final dataset

## Tools Used

- **[Firecrawl](https://firecrawl.dev)** (MCP Server) — JavaScript-aware web scraping with LLM-powered JSON extraction
- **Claude Opus 4.6 Thinking** via Antigravity — orchestration, data cleaning, and consolidation

## Key Challenges

- Amazon lazy-loads books beyond rank ~30, requiring scroll actions
- Live ranking shifts between scrapes caused duplicates and gaps
- Batch extraction (`firecrawl_extract`) was unreliable for Amazon SPAs; switched to 50 individual scrapes
- No local Python environment — all merging/deduplication handled in-context by the AI

## Scaling Considerations

To run this pipeline across many Kindle categories at scale:
1. Parameterize the category URL
2. Script scroll-based pagination in a loop
3. Queue scrapes with rate limiting (e.g. 5 concurrent)
4. Add caching + exponential-backoff retry logic
5. Schedule with a cron job for time-series rank tracking
