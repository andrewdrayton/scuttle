# Setup

## Prerequisites

- **Hermes** installed and running ([hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com))
- **Tavily API key** — free tier works ([tavily.com](https://tavily.com))

---

## Install

```bash
hermes skills tap add andrewdrayton/scuttle
hermes skills install andrewdrayton/scuttle/scuttle --yes
hermes skills install andrewdrayton/scuttle/job-search --yes
hermes skills install andrewdrayton/scuttle/dig --yes
hermes skills install andrewdrayton/scuttle/setup --yes
hermes skills install andrewdrayton/scuttle/discover --yes
```

This installs five skills: `scuttle`, `job-search`, `dig`, `setup`, and `discover`.

---

## First Run Configuration

The first time you run `scuttle`, Hermes will prompt for two things:

**1. Memory path** — where your profile, patterns, and sources files will live.
The default is `~/.hermes/memories/`. Press Enter to accept unless you want a custom location.

**2. Tavily API key** — paste your key when prompted. Hermes stores it securely and passes it through to the skill automatically on every run.

You can update these at any time:
```bash
hermes config show        # view current settings
hermes config migrate     # re-run setup prompts
```

---

## Create Your Profile

Your profile is the most important file — it tells Scuttle who you are, what you're looking for, and how to score roles. The skill won't run without it.

Copy the example profile to your memory directory and fill it in:

```bash
cp ~/.hermes/skills/personal/scuttle/memory/profile.example.md \
   ~/.hermes/memories/job-search-profile.md
```

Then open `job-search-profile.md` and replace every `[PLACEHOLDER]` with your actual information. Key sections:

- **Who I Am** — one paragraph background. Be specific: companies, context, years.
- **Target Role Titles** — the exact titles you want surfaced.
- **Target Company Profile** — stage, size, type.
- **Hard Exclusions** — role types to skip without scoring. More is better here.
- **Track System** — define your own Track 1 / Track 2 / Track 2B. This drives the report structure.
- **Scoring Framework** — the rubric the agent uses to score every role.
- **Curated Job Boards** — industry-specific boards for your sector. These are Tier 1 sources.

The `memory/` directory also contains two template files the skill manages automatically:
- `patterns.template.md` → copy to `job-search-patterns.md` (start empty, skill fills it)
- `sources.template.md` → copy to `job-search-sources.md` (seed with companies you want monitored)

```bash
cp ~/.hermes/skills/personal/scuttle/memory/patterns.template.md \
   ~/.hermes/memories/job-search-patterns.md

cp ~/.hermes/skills/personal/scuttle/memory/sources.template.md \
   ~/.hermes/memories/job-search-sources.md
```

---

## Seed Your Sources

Before your first search, run discover to build your sources file:

```
discover
```

Discover finds and verifies ATS endpoints for companies you're watching and job boards for your target industries. It will ask if there are specific companies you want to probe, then sweep for relevant boards based on your profile.

A populated sources file means faster, better results on your first scuttle run. If you skip this step, scuttle will fall back to Tier 1 board searches only.

---

## Run Your First Search

In any Hermes session:

```
scuttle
```

The skill reads your three memory files, hits your verified sources, and delivers a scored report.

---

## The Two Commands

| Command | What it does |
|---|---|
| `setup` | Guided profile creation — walks you through all sections conversationally and writes the file |
| `discover` | Source building — finds and verifies ATS endpoints and curated boards, writes them to your sources file |
| `scuttle` | Full search run — hits all verified ATS endpoints and curated boards, applies learned patterns, delivers report |
| `dig` | Broad sweep — wide-net Tavily searches for roles that scuttle misses. Slower, run weekly at most. |

---

## Giving Feedback

After each report, reply with a number and a reaction:

```
1 applied
2 pass
3 ignore, too analytical
4 tell me more
```

The skill writes your feedback to `job-search-patterns.md` automatically. After 3-5 sessions, it starts filtering and boosting based on what you've told it. That's the learning loop.

---

## Non-Hermes Environments

Scuttle was built for Hermes. Running it in other agent runtimes (Claude Code, Cursor, etc.) is possible but requires manual configuration — the native Hermes config injection won't be available. See [customization.md](customization.md) for details.
