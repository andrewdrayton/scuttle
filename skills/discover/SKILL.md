---
name: discover
description: Use this skill when the user types "discover" or "/discover". Finds and verifies ATS endpoints and curated job boards for target companies and industries. Writes verified sources to the sources file. Does not search for jobs — only finds where jobs are posted.
version: 1.0.0
metadata:
  hermes:
    tags: [job-search, career, productivity]
    category: personal
    requires_toolsets: [terminal, web]
    config:
      - key: scuttle.memory_path
        description: "Directory where Scuttle memory files are stored"
        default: "~/.hermes/memories"
        prompt: "Where should Scuttle store its memory files? (default: ~/.hermes/memories)"
required_environment_variables:
  - name: TAVILY_API_KEY
    prompt: Tavily API key (required for ATS endpoint discovery)
    help: Get a free key at https://tavily.com
    required_for: ATS endpoint discovery
---

# Discover

Discover is a source-building command. It finds and verifies where your target companies post jobs, and surfaces job boards relevant to your target industries. It does not search for open roles — that is scuttle's job.

Run discover after setup to seed your sources file before your first scuttle run, and any time you want to add companies to your watch list.

## When to Run

- After setup, before your first scuttle
- When adding new companies to your watch list
- When a company's ATS endpoint breaks (returns 404 or empty)
- Whenever your sources file feels thin

Triggered by typing "discover" or "/discover". Execute immediately. No clarification needed unless no profile exists.

## Step 1 — Read Context

Read `{scuttle.memory_path}/job-search-profile.md` and `{scuttle.memory_path}/job-search-sources.md`.

If the profile file does not exist, stop and tell the user:
"Run `setup` first — I need your target companies and industries before I can discover sources."

From the profile, extract:
- Any companies listed in a Watch List or De-Prioritize section
- Industries from Target Industries
- Any boards already listed under Curated Job Boards (skip re-discovering these)

From the sources file, extract all URLs already listed under Verified ATS Endpoints and Curated Boards (skip re-discovering these).

## Step 2 — Identify Targets

Ask once:
> "Any specific companies you want me to find ATS endpoints for? I'll also search for boards in your target industries. Type company names, or 'skip' to go straight to industry boards."

Wait for the answer before proceeding.

If they name companies, add them to the discovery queue. If they say "skip", go directly to Step 4.

## Step 3 — ATS Discovery (Company-Specific)

For each company named:

1. Run a Tavily search to locate their ATS endpoint:
   `[Company] careers jobs site:lever.co OR site:greenhouse.io OR site:ashby.io OR site:workable.com OR site:jobs.ashbyhq.com`

2. If a result URL looks like a real ATS endpoint, fetch it to confirm it loads and contains open roles.

3. Log the outcome:
   - Verified (loads, has open roles) → add to Tier 2 list
   - Loads but empty (no open roles right now) → add to Tier 2 with note "(no open roles at time of discovery)"
   - 404 or unreachable → add to Failed Sources with date

Cap at 3 Tavily searches + 3 fetches per company. If no ATS endpoint is found after 3 searches, log as failed.

## Step 4 — Industry Board Discovery

For each industry in the user's target profile, run up to 2 Tavily searches:
- `best job boards [industry] [year]`
- `[industry] jobs site:wellfound.com OR site:workatastartup.com OR site:jobs.lever.co`

For each board candidate, assess:
- Is it a real, actively-updated job board?
- Does it list roles relevant to the user's titles and stage?
- Is it already in the sources file? If so, skip.

Cap at 5 Tavily searches total across all industries for board discovery.

## Step 5 — Write Results

Write all discoveries to `{scuttle.memory_path}/job-search-sources.md`:

- Verified ATS endpoints → append to `## Verified ATS Endpoints (Tier 2)` section
- New curated boards → append to `## Curated Boards (Tier 1)` section
- Failed or unverifiable → append to `## Failed Sources` section with date and reason

Do not overwrite or remove existing entries. Append only. If a source already exists in the file, skip it.

Re-read the file immediately after writing to confirm the write succeeded.

## Step 6 — Confirm

```
🦀 DISCOVER COMPLETE

[N] ATS endpoints verified
[N] boards added to Tier 1
[N] sources failed or unreachable

Your sources file now has [total] active sources. Type "scuttle" to run a search.
```

If nothing was added (all sources already existed or all searches returned noise), say:
"Nothing new found. Your sources file already covers these companies and industries, or the searches returned no verifiable endpoints. Check the Failed Sources section if you expected a specific company to be found."

## Token Budget

A clean discover session should cost 15-30K tokens. Cap at 40K. If approaching the cap, stop adding new searches and write what you have.
