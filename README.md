# Pocket FM Internship Assignment

This repository contains the submission artifacts for the Pocket FM AI-assisted data extraction assignment.

## What This Repository Includes

- `kindle_paranormal_romance_bestsellers.csv`: Final structured dataset (50 books).
- `workflow.md`: Process note, AI prompt log, scaling ideas, and observations.
- `Intern_assignment.md`: Original assignment brief and requirements.
- `LICENSE`: Project license.

## Assignment Summary

The assignment required extracting data from:

1. The Amazon Kindle Paranormal Romance bestseller listing page.
2. Each listed book's product page.

Expected data points included ranking, title, author, ratings, reviews, pricing, URL, and selected metadata from product pages (description, publisher, publication date).

## Dataset Overview

File: `kindle_paranormal_romance_bestsellers.csv`

- Total records: 50 books
- Header columns:
	- `rank`
	- `title`
	- `author`
	- `rating`
	- `number_of_reviews`
	- `price`
	- `book_url`
	- `description`
	- `publisher`
	- `publication_date`

## Data Notes

- Ratings are stored as numeric decimals where available.
- Review counts are stored as integers where available.
- Publication dates are standardized to `YYYY-MM-DD` when present.
- Some values may be missing and are intentionally left blank or marked as unavailable instead of guessed.
- Prices may contain mixed currencies depending on source session/region.

## How To Use This Repository

### 1) Review the assignment context

Read `Intern_assignment.md` to understand goals and deliverables.

### 2) Understand the extraction workflow

Read `workflow.md` for:

- Tooling choices
- Extraction strategy
- Prompt history
- Edge cases and trade-offs
- Scaling recommendations

### 3) Analyze the dataset

Open `kindle_paranormal_romance_bestsellers.csv` in Excel, Google Sheets, or any data analysis tool.

Suggested checks:

- Validate rank continuity (`1` through `50`).
- Inspect missing values in `publisher` and `publication_date`.
- Compare `rank` against `number_of_reviews` for trend analysis.

## Reproducibility

This repository stores final deliverables and documentation. It does not include an executable scraping script. The extraction process, prompts, and decisions used to generate the dataset are fully documented in `workflow.md`.