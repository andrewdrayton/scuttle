---
name: scuttle
description: Job search agent that learns from your feedback and gets better every run.
version: 1.0.0
author: andrewdrayton
license: MIT
---

# Scuttle

A job search agent that learns. Tell it what you're looking for once. Give it short reactions to what it finds. It builds a model of your taste over time — filtering out what you'd never take, surfacing more of what you'd actually apply to.

## Skills Included

- **setup** — guided profile creation, conversational, ~5 minutes
- **discover** — finds and verifies ATS endpoints and job boards for your target companies/industries
- **scuttle** — targeted search against your verified sources, scored report, learning loop
- **dig** — broad weekly sweep for roles standard ATS monitoring misses

## Install (Hermes)

```bash
hermes skills tap add andrewdrayton/scuttle
hermes skills install andrewdrayton/scuttle/scuttle --yes
hermes skills install andrewdrayton/scuttle/job-search --yes
hermes skills install andrewdrayton/scuttle/dig --yes
hermes skills install andrewdrayton/scuttle/setup --yes
hermes skills install andrewdrayton/scuttle/discover --yes
```

Requires a free [Tavily API key](https://tavily.com).

## Requires

- [Hermes](https://hermes-agent.nousresearch.com) agent runtime
- Tavily API key (free tier works)
