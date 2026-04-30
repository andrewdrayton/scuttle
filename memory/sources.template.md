# Job Search Sources

This file is the agent's evolving registry of working career page URLs and verified ATS endpoints. The agent maintains it conservatively: discovers on demand, marks stale rather than deleting, and re-verifies opportunistically.

## Verified ATS Endpoints (Tier 2)

Format: Company | ATS | API URL | Last verified

<!-- Add verified ATS endpoints here as you discover them.
     Format examples:
       Greenhouse:  https://boards-api.greenhouse.io/v1/boards/{company-slug}/jobs
       Lever:       https://api.lever.co/v0/postings/{company-slug}?mode=json
       Ashby:       https://api.ashbyhq.com/posting-api/job-board/{company-slug}
       Workday:     https://[company].wd5.myworkdayjobs.com/[company-career-site]/jobs
       Custom RSS:  [company career feed URL]

     The agent will populate this section automatically during discovery runs.
     You can also seed it manually with companies you want monitored from day one.
-->

(empty - populated as companies are discovered)

## Discovered Career Pages (ATS Unknown)

Format: Company | URL | Notes

(empty - populated when a careers page is found but ATS detection fails or is incomplete)

## Curated Boards (Tier 1)

Format: Name | URL | Query pattern

- [INDUSTRY TRADE PUBLICATION] Careers | [URL] | site:[domain]/jobs "[YOUR TARGET ROLE]"
- [NICHE JOB BOARD] | [URL] | "[YOUR TARGET ROLE]" "[YOUR TARGET CITY]"
- Wellfound | https://wellfound.com | site:wellfound.com + role keywords + "series A" filter
- [ADDITIONAL BOARD] | [URL] | [QUERY PATTERN]

## Failed Sources

Format: Company/URL | Date failed | Reason

(empty - populated when discovery attempts fail; entries are never deleted, only annotated)
