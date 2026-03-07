# Medium Daily Hot Topics Scraper — CLAUDE.md

## Project Overview

Daily scraper that fetches top articles from Medium RSS feeds across 24 topics, saves results to JSON, emails an HTML digest, updates README.md, and pushes to GitHub — all in one command.

## Run

```bash
cd /home/ubuntu/medium_repo
python3 scraper.py
```

## Project Structure

```
/home/ubuntu/medium_repo/
├── CLAUDE.md           # This file
├── scraper.py          # Main script
├── requirements.txt    # feedparser, rich, beautifulsoup4
├── README.md           # Auto-updated daily digest (do not edit manually)
└── output/
    └── YYYY-MM-DD.json # Daily output, one file per run
```

## Install Dependencies

```bash
python3 -m pip install feedparser rich beautifulsoup4 --break-system-packages
```

## Topics (24 active)

| Category | Topics |
|---|---|
| Original | AI, LLM, Claude, ChatGPT, Gemini, Architecture, Crypto, BTC |
| Tech & Dev | Programming, Python, JavaScript, DevOps, Kubernetes, Cybersecurity, Data Science, Open Source |
| AI / Future | Machine Learning, Prompt Engineering, Robotics |
| Business & Finance | Startup, Venture Capital, Entrepreneurship, Personal Finance, Ethereum |

RSS URL pattern: `https://medium.com/feed/tag/<tag-name>`

Tags with no active Medium RSS feed (do not use): `AGI`, `Web3`, `Productivity`, `Remote Work`, `Career`, `Leadership`, `Climate Change`, `Space`, `Quantum Computing`

## Configuration (in scraper.py)

- `TOP_N = 5` — articles per topic per run
- `EMAIL_FROM / EMAIL_TO` — `fredxuzhou@gmail.com`
- `EMAIL_APP_PASSWORD` — Gmail App Password (see memory file)
- Output deduplicates by URL across topics

## What Each Run Does

1. Fetches RSS feeds for all 24 topics
2. Strips HTML from summaries (BeautifulSoup)
3. Deduplicates articles by URL across topics
4. Keeps top 5 per topic sorted by publish date descending
5. Saves to `output/YYYY-MM-DD.json`
6. Overwrites `README.md` with a Markdown digest table
7. Sends HTML email digest to `fredxuzhou@gmail.com`
8. `git add output/YYYY-MM-DD.json README.md` → commits → pushes to GitHub

## Cron Job

Runs daily at **6:00 AM China Standard Time (CST, UTC+8)** = 14:00 PST (server timezone).

```
0 14 * * * /usr/bin/python3 /home/ubuntu/medium_repo/scraper.py >> /home/ubuntu/medium_repo/cron.log 2>&1
```

Check logs: `tail -f /home/ubuntu/medium_repo/cron.log`

## Git / GitHub

- **Repo:** https://github.com/Fredxuzhou/ingest_medium.git
- **Branch:** main
- **User:** Fredxuzhou
- Remote is configured with PAT token in the URL — push works without extra auth
- Git user identity set locally: `Fredxuzhou / fredxuzhou@gmail.com`

## Email

- **Provider:** Gmail SMTP SSL (smtp.gmail.com:465)
- **Format:** HTML email with a table grouped by topic — clickable titles, summary snippets, publish dates
- App Password stored in memory file and hardcoded in `scraper.py`

## Key Design Decisions

- Uses `feedparser` (not Selenium/Playwright) — no login needed for RSS
- Full article scraping was discussed but not implemented:
  - Medium is React-rendered, `requests` alone gets empty shells
  - Paywalled articles need authenticated browser session cookies
  - 120 full page loads per run would trigger rate limiting
  - RSS summaries (up to 300 chars) are sufficient for a digest
- Deduplication is URL-based: if an article appears under multiple tags, only the first tag keeps it
