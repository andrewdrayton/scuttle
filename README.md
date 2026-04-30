# 🦀 Scuttle

**A job search agent that learns.**

Tell it what you're looking for once. Give it short reactions to what it finds. It builds a model of your taste over time — filtering out what you'd never take, surfacing more of what you'd actually apply to, and discovering new sources along the way.

Most AI job search tools are static wrappers around a search. Scuttle isn't — it gets better every session.

---

## Quick Start

```bash
hermes skills tap add andrewdrayton/scuttle
hermes skills install andrewdrayton/scuttle/scuttle --yes
hermes skills install andrewdrayton/scuttle/job-search --yes
hermes skills install andrewdrayton/scuttle/dig --yes
hermes skills install andrewdrayton/scuttle/setup --yes
```

**Before running setup, get a Tavily API key** — it's required for job board searches and free to start: [tavily.com](https://tavily.com). Hermes will prompt you for it when setup loads.

Then run setup and your first search:

```bash
# Guided profile setup (have your Tavily key ready)
setup

# Run a search
scuttle
```

Setup walks you through building your profile conversationally. Paste your resume to skip the background questions, or answer them manually. Takes about 5 minutes.

> **First search takes longer.** With no verified sources yet, the skill discovers ATS endpoints and curated boards from scratch — expect 8-15 minutes. Subsequent runs are faster as your sources file builds up. By run 3-4 you should be down to 2-3 minutes.

---

## How It Works

**Two commands:**

`scuttle` — targeted search. Hits your verified ATS endpoints and curated job boards, applies everything it's learned from your feedback, delivers a scored report. Run it whenever you want an update.

`dig` — broad sweep. Casts a wider net via signal-based queries across aggregators, hunting for roles that standard ATS monitoring misses — founder-proximate ops, talent-adjacent roles, newly-funded startups in chaos mode. Run it weekly at most. Results stage to a file; review them with `digest`.

**The learning loop:**

After each report, react to what you see:
```
1 applied
2 pass
3 ignore, too analytical
4 tell me more
```

The skill writes your reactions to a patterns file. Next run, it applies them — auto-skipping roles that match your dislikes, boosting roles that match your likes, surfacing adjacent companies to ones you reacted to positively. After 3–5 sessions the signal accumulates enough to meaningfully change what gets surfaced.

**Three memory files, all yours:**

| File | What it is |
|---|---|
| `job-search-profile.md` | Who you are, what you want, how to score. You write this once. |
| `job-search-patterns.md` | What the skill has learned from your feedback. Skill-managed. |
| `job-search-sources.md` | Verified ATS endpoints and curated boards. Grows over time. |

---

## Requirements

- [Hermes](https://hermes-agent.nousresearch.com) — the agent runtime Scuttle runs on
- [Tavily API key](https://tavily.com) — free tier works, used for job board searches

---

## Docs

- [Setup](docs/setup.md) — install, configure, first run
- [Architecture](docs/architecture.md) — how the three files and two commands fit together
- [Customization](docs/customization.md) — tuning your profile, adding sources, adjusting filters
- [FAQ](docs/faq.md) — common issues and edge cases

The `examples/` directory contains a fully worked profile for reference.

---

## Roadmap

**v1 (current)**
- Hermes-native install
- Guided conversational setup with resume paste
- Scuttle + dig + digest commands
- Learning loop via feedback reactions
- Source discovery and maintenance
- Hermes config system for environment paths

**v2 (planned)**
- Environment adapters for Claude Code, Cursor, and other runtimes
- digest skill as a standalone command with richer review UI
- Credential calibration improvements

---

## Author

Built by [Andrew Drayton](https://github.com/andrewdrayton). I hate job hunting, so I built a crab.

MIT License.
