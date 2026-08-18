# Daily Job Search Agent — Manasvi Patil

Watches the US job market every day for Technical Program Manager, Operations Analyst, Capacity Planning, and Data Ops roles matching Manasvi's resumes, and reports back fresh postings with direct application links.

## How it works

1. **`candidate_profile.md`** — the target profile: role clusters, target companies, priority locations, which resume to use for which role.
2. **`daily_search_links.md`** — evergreen search links with the job platforms' own "posted in the last 24 hours" filters baked in. These are always live/fresh, click any day.
3. **`SEARCH_PLAYBOOK.md`** — the exact steps the daily agent runs: refresh links, run WebSearch queries per role cluster, filter to priority locations, log results.
4. **`logs/YYYY-MM-DD.md`** — one file per day with that day's matched postings (title, company, location, application link, resume to use).

## Why it's built this way

This runs inside a sandboxed Claude Code environment where direct HTTP calls to job-board APIs (Greenhouse, Lever, LinkedIn, Indeed, etc.) are blocked by network policy — confirmed by testing (`EGRESS_BLOCKED`). The only way to reach live web data here is Claude's own `WebSearch` tool. So instead of a Python scraper that can't actually reach the internet in this sandbox, the "agent" is a scheduled Claude session that runs `WebSearch` queries directly and writes structured results — this is both more reliable and doesn't fight the platforms' anti-scraping measures (which would violate their Terms of Service anyway).

For true last-24-hours precision, the evergreen links in `daily_search_links.md` are the source of truth — LinkedIn/Indeed apply their own date filter the moment you click, live. WebSearch results supplement those with specific standout postings, but aren't guaranteed to be exactly <24h old.

## The daily schedule

A Routine fires into this project every morning (~8:30am Pacific) and:
- Re-runs the searches in `SEARCH_PLAYBOOK.md`
- Appends a new `logs/YYYY-MM-DD.md`
- Messages the top matches + application links back to Manasvi

## Running it manually

Ask Claude in this repo: "run today's job search" — it will follow `SEARCH_PLAYBOOK.md` and produce a new log entry.

## Target profile snapshot
- MS Engineering Management, USC (Dec 2026) — available to start **Jan 2027**
- Roles: Technical Program Manager, Operations Analyst, Capacity Planning TPM, Data Ops Analyst, early-career rotational PM/TPM
- Priority locations: LA / SF Bay Area (CA), Seattle (WA), NYC (NY), Austin (TX), Remote-US
- Target companies: Apple, Google, Microsoft, Amazon, Meta, and adjacent high-growth tech

See `candidate_profile.md` for the full detail.
