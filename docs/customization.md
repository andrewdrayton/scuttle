# Customization

Scuttle is designed to be adapted. The skill logic is fixed; your three memory files are where all customization lives. You never need to edit the skill files themselves.

---

## Profile Customization

`job-search-profile.md` is the primary lever. Changes here take effect on the next run.

### Tuning the Track System

The Track 1 / Track 2 / Track 2B definitions are yours to define. The defaults in `profile.example.md` are a starting point. Rewrite them to match your actual target:

```markdown
## Track System

Track 1: [Your definition of a perfect fit]
Track 2: [Solid but imperfect fit]
Track 2B: [Loose match — surface but flag the gap]
```

Be specific. The more precisely you define Track 1, the less the skill has to guess.

### Sharpening Hard Exclusions

Hard exclusions are applied before scoring — roles matching them are dropped immediately with no token cost. Use them aggressively:

```markdown
## Hard Exclusions (skip without scoring)

- Roles requiring [FUNCTIONAL EXPERTISE YOU LACK]
- Roles at companies in [INDUSTRY YOU'RE NOT TARGETING]
- Roles based in [EXCLUDED CITY]
- Titles containing "coordinator" or "specialist"
```

The more specific your exclusions, the cleaner your reports.

### Adding Company-Specific Filters

If a high-volume company keeps surfacing noisy results, add a named filter section:

```markdown
### [Company] Filter

Surface [Company] roles only if the JD explicitly mentions:
- [SIGNAL 1]
- [SIGNAL 2]

Auto-skip [Company] roles that are primarily:
- [SKIP SIGNAL 1]
- [SKIP SIGNAL 2]
```

See `examples/andrew_profile.md` for a worked example of this pattern.

---

## Source Customization

### Seeding Initial Sources

When you first copy `sources.template.md`, seed the **Curated Boards** section with boards relevant to your target industry. These are Tier 1 — the skill hits them on every `scuttle` run:

```markdown
## Curated Boards (Tier 1)

- [Board Name] | [URL] | site:[domain] "[YOUR TARGET ROLE]"
```

Then seed **Verified ATS Endpoints** with companies you want monitored from day one. The comment block in the template shows the format for each ATS type (Greenhouse, Lever, Ashby, Workday, custom RSS).

### Letting the Skill Discover Sources

After initial seeding, the skill handles source discovery automatically when you mention a company in feedback:

```
4 tell me more about [Company]
```

The skill will find the careers page, detect the ATS, verify the endpoint, and add it to your sources file. You never need to find ATS slugs manually.

### Manually Adding a Source

If you know a company's ATS endpoint:

```markdown
## Verified ATS Endpoints (Tier 2)

- [Company] | Greenhouse | https://boards-api.greenhouse.io/v1/boards/[slug]/jobs | [date]
```

Common ATS endpoint patterns:
- **Greenhouse**: `https://boards-api.greenhouse.io/v1/boards/{slug}/jobs`
- **Lever**: `https://api.lever.co/v0/postings/{slug}`
- **Ashby**: `https://api.ashbyhq.com/posting-api/job-board/{slug}`
- **Workday**: `https://[company].wd[N].myworkdayjobs.com/[board]` *(JS-rendered — no JSON API, browser session required)*

---

## Filter Tuning via Feedback

The fastest way to improve results is consistent feedback. The skill extracts reusable patterns from your reactions:

| Reaction | What the skill learns |
|---|---|
| `3 ignore, too analytical` | Reduces credential match for analytical-heavy JDs |
| `4 hate this company` | Permanently suppresses the company from all future results |
| `2 pass` (no reason) | Mild negative signal — low-confidence skip |
| `1 applied` | Strong positive — boosts similar roles in future runs |

The more specific the reason, the more useful the extracted pattern. `ignore, requires production finance depth` is more actionable than `ignore, wrong fit`.

---

## Adjusting the Scoring Framework

The scoring rubric lives in your profile. Change the dimensions or scale at any time:

```markdown
## Scoring Framework

1. Role Fit (1-10): ...
2. Credential Match (1-10): ...
3. Application Effort (Low / Medium / High): ...
4. Network Path: Warm / Cold
5. Verdict: Track 1 / Track 2 / Track 2B / Pass
```

You can add dimensions (e.g. a "Comp Fit" score if compensation clarity matters to you) or remove ones that aren't useful. The skill applies whatever framework is defined here.

---

## Changing Your Memory Path

If you want to move your memory files to a different location:

```bash
hermes config set scuttle.memory_path /your/new/path
```

Then move the three memory files to the new location. The skill will pick up the new path on the next run.

---

## Non-Hermes Environments

Scuttle was built for Hermes and is not officially supported on other runtimes in v1. The Hermes config injection (`scuttle.memory_path`, `TAVILY_API_KEY`) won't be available in other environments.

If you want to run Scuttle in Claude Code, Cursor, or another agent runtime, the minimum required manual steps are:

1. Replace all `{scuttle.memory_path}` references in the skill files with absolute paths for your environment
2. Set `TAVILY_API_KEY` as an environment variable your runtime can access
3. Ensure your runtime supports `execute_code` for Python (required for ATS endpoint batching)

Non-Hermes environment adapters are planned for v2.
