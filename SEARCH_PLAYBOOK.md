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
2. **Individual job-posting links do not work as a deliverable — do not send them, at all, even with a caveat.** `WebFetch` is blocked by this sandbox's network policy on every job-board domain tested (`careers.google.com`, `boards.greenhouse.io`, `www.google.com`, `jobs.apple.com`, `jobs.careers.microsoft.com`, `amazon.jobs`, all return `EGRESS_BLOCKED`), so there is no way to confirm a specific req is still open before handing it over. This was tried twice (2026-08-18) — labeling links "unverified" was not enough; the user reported the links dead/missing both times. **Lesson learned: a URL containing a specific job/req ID (e.g. `/jobs/3077884/`, `/details/200633578/`, `/Program-Manager-2/989365`) must never be sent as a deliverable**, because it expires the moment that req is filled or reposted, and WebSearch's index visibly lags that. Instead:
   - Only ever send **search-query URLs** — company career-page searches, LinkedIn/Indeed searches — never a fixed posting URL. A search URL can't go dead because it isn't pointing at one specific req; it just shows whatever is currently open.
   - `daily_search_links.md` is the entire deliverable for "here's where to click." Extend it with more company search URLs (not individual postings) rather than trying to hand-pick specific reqs.
   - It's fine to *describe* what WebSearch turned up in prose ("Microsoft's PM2 band looks like the best early-career fit right now") to help prioritize which search link to check first — just never paste the dead-endable job-ID URL itself.
