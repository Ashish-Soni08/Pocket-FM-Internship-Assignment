# Context

At Pocket FM, we are interested in people who can use AI tools intelligently to design workflows, solve problems, and execute high-volume tasks with good judgment.
This assignment is not just about producing the final output. We care equally about how you think, how you use AI, and how you structure your approach.
You are encouraged to use tools such as ChatGPT, Claude, Codex, Claude Code, Cursor, or similar AI tools as part of your workflow.
Time limit: Please submit the assignment within 24 hours of receiving it; however, please ensure that you do not spend more than 3-4 hours on it
This is the Google Form to submit your response

## Assignment Objective

You are given the following Kindle bestseller page:
[Amazon Kindle Best Sellers – Paranormal Romance](https://www.amazon.com/Best-Sellers-Kindle-Store-Paranormal-Romance/zgbs/digital-text/6190484011)

Your task is to build an AI-assisted workflow to extract structured information from this page and the linked individual book pages.

## Assignment Steps

### Part 1: Extract data from the bestseller page

For each book listed on the page, extract the following fields:

- Rank in the list (e.g., #1, #56, etc.)
- Book Title
- Author
- Rating
- Number of reviews
- Price
- Book URL

### Part 2: Visit each individual book page

For each book, click through to its individual product page and extract:

- Description
- Publisher
- Publication date

### Part 3: Clean and structure the data

Please ensure your final output is clean and usable.
Where possible:

- Keep ratings in numeric format
- Keep number of reviews in numeric format
- Standardize publication dates into a consistent format
- Ensure URLs are clean and functional
- Handle missing values gracefully

If any field is unavailable, leave it blank or mark it clearly rather than guessing.

## Deliverables

Please submit (Bundle them together in a single Google Drive link):

- Document 1: Dataset file
- Document 2:
  - Process / approach note
  - AI prompt log
  - 3 observations

### Document 1: Final dataset

A CSV, Excel file, or Google Sheet containing the extracted data.

### Document 2: Word Doc divided in 3 sections

1. Process / approach note

- A short write-up (roughly 1 page) covering:
  - What tools you used
  - How you approached the task, especially incorporating AI
  - Any issues or edge cases you encountered, and how you solved / worked around them
  - [Important] Tell us how you would adapt your workflow so that it could work for:
    - Another Kindle category page, or
    - Many such pages at scale
[Note for the final question: You do not need to fully build this, but we would like to see how you think about scaling workflows]

1. AI prompt log

- Please include the actual prompts you used with AI tools.
- This can be in a simple document or appendix format. We are not looking for “perfect prompts”; we want to understand how you think and work with AI.

1. Brief observations

- Share 3 short observations from the final dataset.
- These can be simple but relevant observations. Examples include:
  - pricing patterns
  - publisher concentration
  - relationship between rank and reviews
anything else you find interesting
