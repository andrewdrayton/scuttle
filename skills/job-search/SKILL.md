---
name: job-search
description: Use this skill for any scheduled or ad-hoc job search task. Reads the user's profile, learned patterns, and verified sources from memory files. Surfaces, scores, and reports relevant open roles. Includes a learning loop that improves recommendations over time and a source discovery workflow that maintains its own registry of working career page URLs and ATS endpoints.
version: 1.0.0
author: andrewdrayton
license: MIT
metadata:
  hermes:
    tags: [job-search, career, productivity]
    category: personal
    requires_toolsets: [terminal, web]
    config:
      - key: scuttle.memory_path
        description: "Directory where Scuttle memory files (profile, patterns, sources) are stored"
        default: "~/.hermes/memories"
        prompt: "Where should Scuttle store its memory files? (default: ~/.hermes/memories)"
required_environment_variables:
  - name: TAVILY_API_KEY
    prompt: Tavily API key (required for job board searches)
    help: Get a free key at https://tavily.com
    required_for: job board search functionality
---

# Job Search Skill

A self-improving job search agent that surfaces, scores, and tracks roles based on a user-defined profile, learns from feedback over time, and maintains its own registry of working sources.

## Required Setup

This skill reads from three memory files:

- `{scuttle.memory_path}/job-search-profile.md` — user-defined profile (target roles, exclusions, scoring criteria)
- `{scuttle.memory_path}/job-search-patterns.md` — learned patterns from user feedback
- `{scuttle.memory_path}/job-search-sources.md` — verified working sources, discovered career pages, and failed sources

IMPORTANT: Always use the path configured in `scuttle.memory_path`. Do not hardcode paths. Reading from the wrong path returns a blank scaffold that looks like an unfilled template — do not interrupt the user to ask them to fill it in without first confirming the file is also empty at the configured path.

If the profile file does not exist, do not proceed with the search. Tell the user:
"No profile found at `{scuttle.memory_path}/job-search-profile.md`. Run `setup` to create your profile conversationally, or copy `memory/profile.example.md` from the skill directory and fill in the placeholders manually."

If the sources file does not exist, create it with empty sections and seed it with a few known-good ATS endpoints based on the user's watch list.

## Goal of a Search Session

Surface 5 to 15 high-quality, currently-open roles that match the user's profile. Score and filter them. Deliver one structured report.

A search session is bounded:

- Maximum 30 iterations
- Maximum 25 web searches or fetches
- Stop when there's enough material for a useful report, even if not every source has been exhausted

## The "scuttle" Command

If the user sends a message that is exactly or approximately "scuttle" (case-insensitive, with or without punctuation), treat it as a full search run command. Do not ask for clarification. Execute the following sequence automatically:

1. Read `{scuttle.memory_path}/job-search-profile.md` — direct file read, no recall tool
2. Read `{scuttle.memory_path}/job-search-patterns.md` — direct file read, no recall tool
3. Read `{scuttle.memory_path}/job-search-sources.md` — direct file read, no recall tool
4. Run one Python script that hits all verified ATS endpoints sequentially, filters by relevant titles, deduplicates against already-seen roles in the patterns file, and scores results. All in a single execution.
5. Run Tavily searches on Tier 1 boards using site-scoped queries (max 3 searches)
6. Apply learned patterns to adjust scores
7. Deliver the report using the Report Format below

### Hard Rules for "scuttle"

- **No recall tool.** Read the three memory files directly. Never use semantic recall on "scuttle" — it is slow, expensive, and less reliable than a direct file read.
- **No discovery. This applies even on the first run with an empty sources file.** Never probe new companies, test new ATS endpoints, fetch career pages, or run adjacency searches during a "scuttle" run. If the sources file is empty or has no verified endpoints, run Tier 1 Tavily board searches only (max 5) and deliver the report with what you find. Do not compensate for an empty sources file by doing discovery inline — tell the user to run a dedicated discovery session after the report.
- **One Python script for ATS fetching.** Batch all verified ATS endpoint calls into a single execution. Filter and deduplicate inside the same script before returning results.
- **No redundant fetches.** If a Tavily search returns a result, do not also fetch the page unless the user asks for more detail. Tavily snippets are sufficient for scoring.
- **Max 15 sources per run.** If the verified sources list exceeds 15, prioritize those most recently updated or most likely to have relevant roles. Log skipped sources in session stats.
- **Lululemon timeout rule.** If Lululemon RSS times out on the first attempt, skip it. Do not retry during a scuttle run. Log the skip in session stats.

### Token Budget

A clean scuttle run against 15 verified sources should cost approximately 35-45K tokens. If a run exceeds 55K tokens, something went wrong. Likely causes: discovery ran when it should not have, recall tool was used instead of direct file reads, multiple Python scripts ran instead of one, redundant page fetches happened after Tavily searches, or a source returned a large payload that was not truncated at 100KB.

## Critical: Response Size Discipline

Many job sources return large payloads. A single naive fetch of an ATS endpoint can return 5MB+ of JSON, which will blow context budgets, hit rate limits, and fail the session. Always route large responses through code execution to filter before bringing data into context.

Rules:

- Never load a raw API response larger than ~100KB directly into context.
- For ATS endpoints (Greenhouse, Lever, Ashby, etc.), use code execution to fetch, parse, and filter to ops-relevant titles before returning data to context.
- For web pages, use Tavily's extract function with content limits, not full-page fetches.
- When in doubt, write a Python snippet that fetches, filters by title keywords from the profile's "Target Role Titles," and returns only matching jobs as a small JSON list.

The pattern for filtered ATS fetches is: import requests, fetch the JSON, define a keyword list from the user's target titles, filter the jobs array by case-insensitive title matching, and print only title/location/url for matches. This returns 5-20 relevant entries instead of 400+ raw entries. The same pattern applies across Lever, Ashby, Workable, and Recruitee.

BATCH PROBING: Probe multiple companies in a single execute_code block. One HTTP call per company, all in one script. This is dramatically more efficient than individual tool calls. A single block can probe 8-10 companies in under 5 seconds.

RETAIL NOISE WARNING: Companies like Alo Yoga (652 jobs), Lululemon, and any large DTC apparel brand will match "operations" thousands of times — almost all retail floor, store, or distribution roles. Before scoring any "Director of Operations" match from a large retail brand, check the location and job description. If the location is a store address or mall name, it is a retail floor role — hard exclude. Add a secondary filter: exclude titles containing "store," "retail," "visual," "3PL," "distribution center," or "seasonal" before surfacing matches.

## Source Priority (In Order)

### Tier 1 — Curated industry job boards

Use Tavily search with specific role queries, not homepage fetches. The user's profile lists which boards are relevant for their target industries.

Effective query patterns:

- `site:businessoffashion.com/careers "head of operations"`
- `site:hypebeast.com/jobs "chief of staff"`
- `site:wellfound.com "VP operations" series A`

Avoid bare homepage fetches of these boards. They return luxury retail and sales noise.

Known board behaviors (update as discovered):

- Business of Fashion: site: search works. For senior ops roles, also fetch the category page directly:
  `https://www.businessoffashion.com/careers/jobs/operations/director-president-c-suite/`
  WARNING: As of 2026-04-27 this page only had 3 listings total, none relevant for a senior ops generalist. It is not a reliable source for the user's target level. Use site: searches instead and set expectations low for BoF at the director/C-suite level.

- Hypebeast: `site:hypebeast.com/jobs` search returns noise and editorial content, not job listings.
  Use the Lever API directly: `https://api.lever.co/v0/postings/hypebeast` (verified active, 22 postings as of 2026-04-27).
  Do NOT rely on Hypebeast's job board UI or site: search.

- Wellfound: web_extract is blocked — returns empty content. site: search returns only listing-page URLs, not actual job postings.
  Wellfound is not reliably queryable via these tools without a browser session. Deprioritize until a working access pattern is found.

- Lululemon: Custom portal at careers.lululemon.com (iCIMS-style, /portal/4/ path). Not Greenhouse/Lever/Workday.
  No JSON API. The RSS feed is the only programmatic endpoint:
  `https://careers.lululemon.com/en_US/careers/SearchCareer/remote/feed/?jobRecordsPerPage=50`
  Parse the XML for <title> and <link> elements.
  WARNING: As of 2026-04-27 this endpoint timed out on two consecutive attempts (30s timeout). It may be intermittently unavailable or rate-limited. Do not block a session on it — log the failure and move on.

- Patagonia: Workday tenant at https://patagonia.wd503.myworkdayjobs.com/PWCareers (verified 2026-04-27).
  JS-rendered — no public JSON API. Requires browser session to pull job listings.

### Tier 2 — ATS API endpoints (preferred for company career pages)

Most company career pages run on standard Applicant Tracking Systems with public JSON APIs. Detect the ATS, hit the API directly via code execution with title filtering. Cleaner data, faster, no browser needed.

Common ATS endpoints (these patterns are stable infrastructure):

- Greenhouse: `https://boards-api.greenhouse.io/v1/boards/[company]/jobs`
- Lever: `https://api.lever.co/v0/postings/[company]`
- Ashby: `https://api.ashbyhq.com/posting-api/job-board/[company]`
  NOTE: Several culture-adjacent AI and creative companies use Ashby rather than Greenhouse or Lever.
  When Greenhouse and Lever both return 404 for a company, try Ashby before giving up.
  Known Ashby users: Runway ML (slug: runway-ml), ElevenLabs (slug: elevenlabs).
  The Ashby response shape differs slightly — jobs are in `jobPostings[]`, location is in `locationName`, URL is in `jobUrl`.
- Workable: `https://apply.workable.com/api/v3/accounts/[slug]/jobs`
- Recruitee: `https://[company].recruitee.com/api/offers`
- Greenhouse-embed: check page source for `boards.greenhouse.io/embed/job_app?for=[company]`
- Workday: NO public JSON API. Workday pages are JS-rendered. The careers URL follows the pattern
  `https://[company].wd[NNN].myworkdayjobs.com/[board]` — record this URL in sources but note it
  requires a browser session to pull listings. Do not attempt programmatic JSON fetches against
  Workday URLs; they will return an empty shell page, not job data.

Always filter ATS responses to relevant titles via code execution before pulling into context.

### Tier 3 — Broad job aggregators (last resort)

Google Jobs via Tavily, LinkedIn (expect partial results, anti-bot).

Always start with Tier 1 and Tier 2. Only escalate to Tier 3 if Tier 1 and 2 produce fewer than 5 candidate roles.

## Source Discovery and Maintenance

The sources file is the agent's evolving registry of working career pages and verified ATS endpoints. The skill describes the patterns; the registry holds the specific working URLs and the agent maintains it conservatively.

### When to Discover

Discover sources only when there is clear reason. Do not spend session budget on speculative discovery.

Triggers for discovery:

1. **New company of interest** — surfaced through adjacency search, user feedback, or pattern matching, and not already in the sources file.
2. **User explicitly mentions a company** — through feedback like "tell me about [Company]" or "check [Company]'s jobs."
3. **Source failure** — a previously verified URL returns 404, redirect to homepage, or layout change. Mark as stale, attempt rediscovery once.
4. **Stale verification** — opportunistically, when a verified ATS endpoint hasn't been hit in 30+ days, verify it still works at the start of a relevant search.

### Discovery Workflow

For a target company:

1. Search for the careers page: `[Company] careers` via Tavily. Take the top result that points to the company's own domain.
2. Fetch the careers page (HTML, small response).
3. Detect ATS by inspecting the HTML:
   - Look for `boards.greenhouse.io/embed` in iframe sources or links → Greenhouse, slug after `for=`
   - Look for `jobs.lever.co/[slug]` in links → Lever
   - Look for `[slug].ashbyhq.com` or `jobs.ashbyhq.com/[slug]` → Ashby
   - Look for `apply.workable.com/[slug]` → Workable
   - Look for `[slug].recruitee.com` → Recruitee
   - Look for `[slug].wd[NNN].myworkdayjobs.com` in links or page source → Workday, capture full tenant URL
   - "Powered by [ATS]" text is also a reliable signal
4. If ATS is detected, construct the API URL using the patterns above. Verify it returns valid JSON with a jobs array. You MUST then use the file edit tool to write the entry to the sources file under "Verified ATS Endpoints." After writing, re-read the sources file to confirm the entry is present. Do not consider discovery complete until the write is verified on disk.
5. If no ATS is detected, record the careers page URL under "Discovered Career Pages (ATS unknown)" so a future session can revisit.
6. If discovery fails entirely (no careers page found, or all detection fails), record under "Failed Sources" with the date and reason.

### Updating the Sources File

After discovery:

- Move successful finds to "Verified ATS Endpoints" with the date.
- Mark stale entries with a note: `last verified: [date]` or `failed: [date], reason`.
- Do not delete failed entries. They serve as memory of what was tried.

Use the file tool to append or rewrite sections. Keep the format consistent so it remains parsable.

## Adjacency Search (Discovering New Companies)

After exhausting known sources, spend a portion of the session budget on adjacency. Use the patterns memory file to find companies similar to ones the user has reacted positively to.

Patterns to try:

- Search "companies similar to [liked company]"
- Search "[VC firm] portfolio companies" if the user liked one of their portcos
- Search "[industry] startups raised seed [year]"
- Look for founders who left [liked company] and started something new

Surface these in the report under "New to Watch." Even without an open role, flagging the company creates future opportunities. If a discovered company seems promising, run discovery on its careers page and add to the sources file.

## Learning Loop

This is the part that makes the skill compounding rather than static. After each report the user replies with reactions using the numbered feedback format described below.

When the user reacts, write to the patterns memory file using sections labeled Liked, Disliked, Active Filters, and Wants More Info. Each entry should include date, company, role, and the relevant signals (industry, stage, founder profile, etc.).

Before each new search, read the patterns file and the sources file. Use accumulated patterns to:

- Boost fit score for roles matching liked patterns (+1 to +2 points)
- Auto-skip roles matching disliked patterns or filter phrases
- Surface adjacent companies to liked ones during adjacency search
- Pull deeper info on companies in the "Wants more info" section
- Run discovery on companies that appear in patterns but not in sources

The system should noticeably improve after 3-5 feedback cycles.

## Feedback Loop

After delivering a report, wait for user feedback. The user responds with a number from the report and a short reaction. Multiple items can be given in one message.

Examples:
- `1 applied`
- `2 pass`
- `3 ignore, too analytical`
- `4 hate this company`
- `2 tell me more`

### Parsing Feedback

**"applied"**
- Strong positive signal on both Role Fit and Credential Match
- Log to patterns file: company, role type, industry, stage, what scored well

**"pass"** (no reason)
- Mild negative signal. Log as low-confidence skip. Do not draw strong conclusions.

**"ignore" or "skip" + reason**
- Highest-value feedback. Parse the reason to extract a reusable pattern.
- Examples of what to extract:
  - "too analytical" → reduce Credential Match for roles requiring quantitative or data-heavy profiles
  - "wrong stage" → deprioritize roles at companies past Series A or over target headcount
  - "hate this company" → suppress this company from all future results permanently; log to sources file as excluded
  - "too corporate" → reduce Fit score for roles at large established companies
  - "coordinator level" → reduce Fit for roles where scope is support rather than ownership
- Write the extracted pattern to `{scuttle.memory_path}/job-search-patterns.md` in plain language, dated

**"tell me more"**
- Pull full job description via Tavily fetch or ATS endpoint detail
- Summarize: key responsibilities, reporting structure, compensation if listed, application method
- Do not re-score. Let the user decide.

### Writing Feedback to Patterns File

After processing all feedback in a session, append a dated entry:

```
## [Date] — Feedback Session

Applied: [list roles/companies]
Skipped with reason:
- [Company / Role]: [extracted pattern]
Suppressed companies: [list if any]
```

Always re-read the file after writing to confirm the write succeeded.

## Deduplication

Before scoring a role, check if it appeared in a prior session by reading the patterns file. Track dedup key: company name plus role title. Skip duplicates unless posting date is newer or description changed materially.

The dedup section in the patterns file is named "Companies Already Surfaced (Dedup)" — use that exact heading when appending entries. Format: `| Company | Role Title | Date Seen |` table.

## Filtering Workflow

For each role found:

1. Apply hard exclusions from profile. Drop with no scoring. Tally count.
2. Read description carefully. Watch for bait signals (rockstar/ninja, grind language, junior disguised as senior, retail-floor-as-ops).
3. Apply learned filters from patterns file.
4. Score on the user's scoring framework from profile.
5. Apply pattern-based score boost or penalty.
6. Assign track label per user's framework.

## Report Format

Numbered list grouped by verdict. Track 1 candidates first, then maybes. Low-scoring roles are silently dropped — do not list them individually.

Each entry:
```
[N]. [Company] — [Role Title]
     [Location]
     Fit: X/10  |  Cred: X/10  |  Effort: [Low/Med/High]  |  Network: [Warm/Cold]
     [One sentence on why it scored this way]
     [URL]
```

Full report structure:
```
🦀 SCUTTLE REPORT — [Date]

TRACK 1
1. ...
2. ...

MAYBES
3. ...

NEW TO WATCH
- [Company]: [one line on why it's on radar, no open role]

FILTERED OUT
[Count and brief tally: e.g. "32 roles scanned. 8 retail floor, 5 wrong level, 4 finance ops, 3 international only, 6 duplicates"]

SOURCE UPDATES
[One or two lines on any new endpoints verified or sources flagged this session]

SESSION STATS
Sources checked: [N] | New roles surfaced: [N] | Duplicates skipped: [N] | Token estimate: [N]K
---
[N] roles scanned. [N] surfaced. Type a number + reaction to give feedback.
```

If nothing scored above 5 on Fit, say so plainly: "Nothing strong this cycle. [N] roles scanned, nothing worth surfacing."

## Failure Modes to Avoid

- Do not pad reports. If only 2 priority roles surfaced, report 2.
- Do not invent roles. Verify each URL resolves.
- Do not editorialize. Score with the rubric, flag bait signals, let the user decide.
- Do not write application materials. Surfacing and scoring only.
- Do not exceed iteration limits. Send what you have and note the budget cap.
- Do not narrate memory updates in the report. Just write to the file.
- Do not load large API responses into context without filtering first.
- Do not run speculative source discovery. Only discover when triggered.
- Do not delete entries from the sources file. Mark them stale or failed instead.
- Do not claim a memory file update succeeded without verifying the write on disk. After any write to a memory file, re-read it to confirm the new content is present before reporting the action as complete.
- Do not retry the same operation more than twice. If a tool call fails after two attempts with the same approach, change strategy or skip and report it as failed in the sources file.
- Do not run more than 5 discovery operations in a single session. Discovery is expensive. Spread it across multiple sessions.
- Do not use the recall tool during a "scuttle" run. Read memory files directly.
- Do not run discovery during a "scuttle" run. Discovery is a separate explicit command.
- Do not split ATS fetching across multiple Python executions during a "scuttle" run. One script, all endpoints, filter and deduplicate inside it.

## Sources File Template

If the sources file does not exist, create it with this structure:

```
# Job Search Sources

## Verified ATS Endpoints (Tier 2)

(empty - populated as companies are discovered)

## Discovered Career Pages (ATS Unknown)

(empty)

## Curated Boards (Tier 1)

(seeded from the user's profile "Curated Job Boards" section)

## Failed Sources

(empty)
```
