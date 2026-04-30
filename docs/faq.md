# FAQ

## Getting Started

**The skill says my profile file doesn't exist.**
You need to create it before the first run. Copy the example and fill it in:
```bash
cp ~/.hermes/skills/personal/scuttle/memory/profile.example.md \
   ~/.hermes/memories/job-search-profile.md
```
Then replace every `[PLACEHOLDER]` with your actual information. See [setup.md](setup.md).

**The skill loaded but shows a blank template instead of my profile.**
You have two memory files at different paths. Check that your profile is at the path configured in `scuttle.memory_path`:
```bash
hermes config show | grep scuttle
ls ~/.hermes/memories/
```
If the file is somewhere else, either move it or update `scuttle.memory_path` to match.

**Hermes isn't prompting me for my Tavily API key.**
Run `hermes setup` in your terminal to trigger the environment variable setup flow. Messaging surfaces (Telegram, Slack, etc.) never prompt for secrets — setup must be done in the local CLI.

---

## Search Results

**The report came back empty / "nothing strong this cycle."**
A few likely causes:
- Your hard exclusions are too broad — review `job-search-profile.md` and check if roles you'd actually want are being filtered out
- Your verified sources list is thin — seed more companies in `job-search-sources.md` or ask the skill to discover sources for specific companies
- The ATS endpoints were checked but had no matching open roles this cycle — this is normal, try again in a few days

**I'm getting lots of retail/store operations roles.**
Add title-level exclusions to your hard exclusions list:
```markdown
- Roles with "store," "retail," "visual merchandising," or "distribution center" in the title
```
Large DTC apparel brands (Alo Yoga, Lululemon, etc.) can return hundreds of store-ops roles matching "operations." The skill has a retail noise filter but your profile exclusions are an additional layer.

**The same roles keep appearing every run.**
Check `job-search-patterns.md` — the dedup table should be tracking previously surfaced roles. If it's empty or the role isn't in it, the skill may not have written to the file correctly. After each run, verify the dedup table was updated before closing the session.

**I'm seeing roles I explicitly passed on.**
Give feedback with a reason: `3 ignore, [reason]`. A bare `pass` is a weak signal — the skill logs it but doesn't extract a durable filter. `ignore` with a reason creates an active filter that auto-skips similar roles in the future.

---

## Sources and Discovery

**A company's ATS endpoint keeps returning 404.**
The slug may have changed or the company switched ATS providers. Tell the skill:
```
check [Company]'s jobs
```
It will re-run discovery, find the current endpoint, and update the sources file. The old failed entry stays in the Failed Sources section as a record.

**Wellfound / AngelList isn't returning results.**
Wellfound blocks programmatic access without a browser session. The skill uses site: searches as a workaround but these return listing URLs, not full job data. Wellfound is deprioritized until a reliable access pattern is available. Use it as a manual check outside of Scuttle.

**Can I add LinkedIn as a source?**
LinkedIn aggressively blocks bots. Tavily searches can surface LinkedIn listings as snippets but full job data is unreliable. Don't add LinkedIn as a verified ATS endpoint — treat any LinkedIn results as low-confidence.

---

## Token Costs

**A run cost way more than expected.**
Common causes:
- Discovery ran during a `scuttle` run (it shouldn't — discovery is a separate command)
- A large ATS endpoint returned raw JSON without filtering
- Multiple Python executions ran instead of one batched script
- The recall tool was used instead of direct file reads

If runs are consistently over 55K tokens, check whether the patterns and sources files are being read correctly at session start.

**How do I keep costs down on `dig`?**
`dig` is designed for a weekly cadence at most. Cap it at 10 Tavily searches per session — if early queries return strong results, stop early. Don't run `dig` and `scuttle` in the same session.

---

## The Learning Loop

**My feedback doesn't seem to be affecting results.**
The skill reads `job-search-patterns.md` at the start of each session. If a run happened before you gave feedback, the new patterns won't apply until the next session. Also check that feedback was actually written to the file — after giving feedback, ask the skill to confirm the write.

**How long until results noticeably improve?**
Typically 3–5 feedback sessions. The first couple of sessions calibrate the basics (what industries, what levels). By session 5 you should see meaningfully fewer irrelevant roles and better scoring on the roles that do surface.

**Can I edit the patterns file directly?**
Yes, but carefully. The file header says "do not edit manually unless correcting an error" — that's the intent. If you want to add a permanent filter without waiting for the skill to learn it, you can add it to the Active Filters section directly. Keep the format consistent with existing entries.

---

## Hermes-Specific

**The skill installed but doesn't appear in `/skills list`.**
Check that the install completed without errors:
```bash
hermes skills list | grep scuttle
```
If it's missing, try reinstalling:
```bash
hermes skills install github:andrewdrayton/scuttle --force
```

**I want to run Scuttle on a schedule.**
Scheduling is out of scope for Scuttle v1. Hermes supports scheduled tasks natively — if you want automated runs, refer to the Hermes docs to set it up yourself. Scuttle will work fine as the scheduled command, we just don't document the setup.

**I edited a skill file and now updates are blocked.**
Hermes marks manually-edited bundled skills as user-modified and stops pulling upstream changes. To reset:
```bash
hermes skills reset scuttle
hermes skills reset job-search
hermes skills reset dig
```
This re-baselines the skill without deleting your edits. Use `--restore` if you want to revert to the upstream version.
