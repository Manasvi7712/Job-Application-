# Daily Search Playbook

This is what the daily agent (see the Routine described in `README.md`) actually does each morning. It's written as an explicit playbook — not a script — because this environment's network policy blocks direct calls to job-board APIs and ATS endpoints (Greenhouse, Lever, LinkedIn, Indeed all return `EGRESS_BLOCKED` on raw HTTP). The only reliable path to live web data here is Claude's `WebSearch` tool, so "last 24 hours" filtering is delegated to the job platforms' own date filters rather than done by scraping.

## Step 1 — Refresh the evergreen "past 24 hours" links
`daily_search_links.md` contains search URLs with the platform's own date-posted filter baked in (LinkedIn `f_TPR=r86400` = past 24h, Indeed `fromage=1` = past 1 day). These links are **always live** — clicking them on any day shows that day's fresh postings, no scraping needed. Regenerate this file only if a role cluster or location is added/removed in `candidate_profile.md`.

## Step 2 — Run WebSearch queries for each role cluster
For each of the 5 role clusters in `candidate_profile.md`, run 1–2 targeted WebSearch queries such as:
- `"Technical Program Manager" jobs posted today apply United States`
- `"Operations Analyst" new grad job posting today California OR Washington OR Texas apply`
- `"Capacity Planning" OR "Technical Program Manager" data center jobs Google Amazon Meta apply`
- `site:careers.google.com OR site:jobs.apple.com OR site:careers.microsoft.com OR site:amazon.jobs "Program Manager" OR "Operations Analyst"`
- `"Program Manager University Grad" OR "TPM rotational" 2027 apply`

Pull out individual postings with a direct application link (not just search-result-page links), and note the company, title, location, and which resume variant fits best.

## Step 3 — Filter and rank
- Keep only US-based roles matching a target location tier (see `candidate_profile.md`).
- Drop senior/staff/principal-only postings.
- Rank by: (1) target company match, (2) location tier, (3) role-cluster fit.

## Step 4 — Log and report
- Append a new dated file to `logs/YYYY-MM-DD.md` with the day's findings (title, company, location, link, resume to use, notes on posting recency).
- Message the user with the top matches and their application links, plus a reminder that the evergreen links in `daily_search_links.md` are the most reliable way to see the exact last-24-hours feed live.

## Known limitations — read before reporting results
1. **WebSearch results are not guaranteed to be posted within exactly 24 hours** — search engines index on their own schedule. Treat WebSearch hits as "recent, worth checking," and treat the platform-native filtered links (Step 1) as the ground truth for true last-24-hours freshness.
2. **Individual job-posting links cannot be verified as still live.** `WebFetch` is blocked by this sandbox's network policy on every job-board domain tested (`careers.google.com`, `boards.greenhouse.io`, `www.google.com`, etc. all return `EGRESS_BLOCKED`), and there is no other tool here that can open a URL to confirm it. WebSearch's index has been observed serving job postings that were already taken down (confirmed 2026-08-18: a Google TPM I link returned "Job not found"). **Never present an individual WebSearch-sourced job link as confirmed-live.** Always:
   - Lead the report with the evergreen search links from `daily_search_links.md` — these are queries, not postings, so they can't go stale.
   - List individual postings found via WebSearch as "found via search — unverified, may already be filled; if the link is dead, use the matching search link above to find the current equivalent."
   - Never claim to have "confirmed" or "verified" a specific posting's status.
