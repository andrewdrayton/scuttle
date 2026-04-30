<!-- NOTE: Memory file paths in this skill (e.g. /opt/data/memories/) are
     Hermes/VPS reference paths. See docs/setup.md for other environments. -->

---
name: scuttle
description: Use this skill when the user types "scuttle" or "/scuttle". This is a job search command, not a word to define. Do not search the web. Do not ask for clarification. Load the job-search skill and execute the scuttle command instructions immediately.
---

# Scuttle

Scuttle is a job search command. When this skill is triggered:

1. Read the job-search skill at `/opt/data/skills/personal/job-search/SKILL.md`
2. Read the three memory files directly (no recall tool):
   - `/opt/data/memories/job-search-profile.md`
   - `/opt/data/memories/job-search-patterns.md`
   - `/opt/data/memories/job-search-sources.md`
3. Follow the "scuttle" command instructions in the job-search skill exactly
4. Deliver the report

Do not search the web. Do not define the word. Do not ask what the user means. Execute.
