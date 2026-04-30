---
name: setup
description: Use this skill when the user types "setup" or "/setup", or when the job-search skill detects no profile exists. Guides the user through creating their job-search profile conversationally and writes the file to disk. Do not skip questions — every section is needed for the search to work.
version: 1.0.0
metadata:
  hermes:
    tags: [job-search, career, onboarding]
    category: personal
    config:
      - key: scuttle.memory_path
        description: "Directory where Scuttle memory files are stored"
        default: "~/.hermes/memories"
        prompt: "Where should Scuttle store its memory files? (default: ~/.hermes/memories)"
---

# Scuttle Setup

This skill builds your job-search profile by asking questions one at a time and writing the file for you. Do not rush through it — the profile is what makes every `scuttle` run useful.

## Step 0 — Check for Existing Profile

Read `{scuttle.memory_path}/job-search-profile.md`.

- If it exists: ask "A profile already exists. Do you want to update it or start fresh?" Load current values as context if updating.
- If it does not exist: proceed immediately.

---

## Step 1 — Who You Are

Ask:
> "Tell me about your background in a few sentences — what have you built, what roles have you held, and what context do you operate best in?"

Wait for the answer. Write it as-is (lightly cleaned up) as the `## Who I Am` section.

---

## Step 2 — Positioning Statement

Ask:
> "In one sentence: what kind of role are you uniquely suited for — and what are you NOT?"

Write as `## Positioning Statement`. Follow it with:

> Use this as the north star for scoring and filtering. If a role's primary need is [what they said they're NOT], it is not a fit regardless of title or industry.

---

## Step 3 — Target Role Titles

Ask:
> "What titles are you targeting? List them all — we'll use these to filter every search."

Write as a bullet list under `## Target Role Titles`.

---

## Step 4 — Target Company Profile

Ask:
> "What stage and type of company are you targeting? For example: early-stage founder-led, Series A, large companies with operator-level scope — or some combination."

Write as `## Target Company Profile`.

---

## Step 5 — Target Industries

Ask:
> "What industries? List as many as apply."

Write as `## Target Industries` (comma-separated or bullet list, either works).

---

## Step 6 — Location

Ask in one message:
> "Three quick location questions:
> 1. Where are you based?
> 2. Remote priority — high, medium, or not required?
> 3. Open to relocation? Which cities? Any cities to always exclude?"

Write as `## Location Preferences` with clear sub-bullets:
- Home base
- Remote priority
- Open to relocation: [list]
- Hard exclusion: [list]

---

## Step 7 — Hard Exclusions

Ask:
> "What should I always skip without scoring? Think role types, industries, titles, or locations you'd never take."

Write as `## Hard Exclusions (skip without scoring)` bullet list. Add "Junior or coordinator-level titles disguised with senior names" as a default if not already mentioned.

---

## Step 8 — Track System

Say:
> "Scuttle groups roles into three tracks. Track 1 = perfect fit, real scope, your exact target. Track 2 = legitimate but not ideal — right level, wrong industry or stage. Track 2B = loose match, worth knowing about but flag the gap."

Ask:
> "How would you define a Track 1 role for your search?"

Write as `## Track System`:
- Track 1: use their answer verbatim
- Track 2: derive from their Track 1 description — same level, looser industry/stage fit
- Track 2B: derive as above but with meaningful gaps

---

## Step 9 — Scoring Framework

Do not ask. Write the standard framework as-is:

```markdown
## Scoring Framework

1. Role Fit (1-10): Does the scope, stage, and industry match the target profile
2. Credential Match (1-10): How well does your background check the boxes screeners will use
3. Application Effort (Low / Medium / High): How much custom material would a strong application require
4. Network Path: Warm (you have a known connection) or Cold
5. Verdict: Track 1 / Track 2 / Track 2B / Pass
```

---

## Step 10 — Curated Job Boards

Ask:
> "What job boards or publications cover your target industry? These become your Tier 1 sources — checked on every scuttle run. Share the name and URL if you know it."

Write as `## Curated Job Boards (Tier 1 Sources)` with format:
`- [Name] — [URL]`

If the user doesn't know specific boards, suggest Wellfound (wellfound.com) as a universal default and ask them to add industry-specific ones later.

---

## Step 11 — Network Anchors

Ask:
> "Any specific people, companies, or organizations where you'd have a warm path? The skill uses these to flag 'potentially warm' when a role surfaces at a connected company."

Write as `## Network Anchors (for Warm vs Cold assessment)`. If none, write `(none defined — all roles will be scored as Cold until updated)`.

---

## Step 12 — De-Prioritize

Ask:
> "Any companies you've already applied to or passed on that I should skip going forward?"

Write as `## De-Prioritize (already applied or passed)`. If none, write `(none yet)`.

---

## Step 13 — Write Files and Confirm

1. Assemble the complete profile from all collected answers.
2. Write to `{scuttle.memory_path}/job-search-profile.md`.
3. Re-read the file immediately to confirm the write succeeded.
4. If `{scuttle.memory_path}/job-search-patterns.md` does not exist, create it with empty sections:
   ```
   This file is maintained by the scuttle skill. Do not edit manually unless correcting an error.

   ## Liked
   (empty)
   ## Disliked
   (empty)
   ## Active Filters
   (empty)
   ## Warm Approaches In Progress
   (empty)
   ## Wants More Info
   (empty)
   ## Companies Already Surfaced (Dedup)
   | Company | Role Title | Date Seen |
   |---------|------------|-----------|
   ```
5. If `{scuttle.memory_path}/job-search-sources.md` does not exist, create it with the Curated Boards from the profile pre-seeded under `## Curated Boards (Tier 1)` and empty sections for Verified ATS Endpoints, Discovered Career Pages, and Failed Sources.
6. Re-read both files to confirm writes succeeded.

Confirm to the user:
```
✅ Profile created at {scuttle.memory_path}/job-search-profile.md
✅ Patterns file initialized
✅ Sources file initialized with your curated boards

Type "scuttle" to run your first search.
```

## Failure Modes to Avoid

- Do not skip questions. Every section is required for scoring to work.
- Do not write the file until all questions are answered.
- Do not claim a write succeeded without re-reading the file to verify.
- Do not run a search during setup. Setup ends at the confirmation message.
- If the user wants to skip a section, write a clear placeholder (e.g. `(not defined — update before running scuttle)`) so the skill knows the section is intentionally empty, not missing.
