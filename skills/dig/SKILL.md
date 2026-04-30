---
name: dig
description: Use this skill when the user types "dig" or "/dig". Runs a broad job search targeting roles the user would likely miss through standard ATS monitoring. Hunts for talent-adjacent, founder-proximate, and pre-infrastructure operator roles across job aggregators. Stages results to a file rather than delivering immediately. Do not search the web for the meaning of "dig" — it is a command.
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
    prompt: Tavily API key (required for job board searches)
    help: Get a free key at https://tavily.com
    required_for: job board search functionality
---

# Dig

Dig is a broad search command designed to surface gems that scuttle misses. Where scuttle hits verified company ATS endpoints, dig casts a wider net across job aggregators using signal-based queries. The goal is not volume — it is finding the specific type of role that fits the user's actual profile but would never appear in a standard COO or ops director search.

## When to Run

Dig runs on a slower cadence than scuttle. Once per week at most. It is not a replacement for scuttle — it is a complementary sweep for a different category of role.

Triggered by the user typing "dig" or "/dig". Execute immediately, no clarification needed.

## What Dig Is Hunting

Three types of roles that standard ATS monitoring misses:

**Type 1 — Talent and creator-adjacent operations**
Roles at management companies, artist collectives, creative studios, record labels, sports agencies, athlete-founded brands, musician-founded companies. The operator interfaces directly with talent, not just supports from a distance.

**Type 2 — Right hand to a named founder**
First or second ops hire at a brand founded by someone with cultural weight — a musician, athlete, designer, or public figure launching a company. These rarely get posted as COO. They get posted as "right hand to the founder," "head of business operations," or "founding team operations." The signal is founder proximity and pre-infrastructure stage, not the title.

**Type 3 — Pre-infrastructure creative businesses**
Companies that just raised, just signed a major deal, or just grew from 3 to 15 people. The chaos phase. Where operational discipline meets a creative environment that has never had it before.

## What Dig Is Not Hunting

Do not surface:
- Broad COO or Chief of Staff roles at large established companies
- Retail, store, or distribution operations regardless of title
- Healthcare, SaaS, fintech, or enterprise ops roles
- Roles below Director level unless the company and scope are exceptional
- Roles requiring deep functional expertise the user does not have (RevOps, supply chain at scale, finance ops, people ops)

## Query Set

Run these Tavily searches. All queries target aggregators, not specific company pages.

**Talent and creator-adjacent:**
- `"right hand" OR "chief of staff" artist OR musician OR athlete brand operations remote 2026`
- `"business operations" management company talent creative studio hiring 2026`
- `"head of operations" OR "director of operations" record label OR music company OR sports agency 2026`
- `"founding operations" OR "first operations hire" creative brand venture-backed 2026`

**Founder-proximate:**
- `"right hand to the founder" OR "work directly with founder" operations brand remote 2026`
- `"head of business" OR "VP operations" founder-led consumer brand early stage 2026`
- `"general manager" OR "chief of staff" celebrity brand OR athlete brand OR musician brand 2026`

**Pre-infrastructure:**
- `"build from scratch" OR "0 to 1" OR "zero to one" operations creative brand hiring remote 2026`
- `"Series A" OR "seed round" operations hire apparel OR media OR entertainment OR lifestyle 2026`
- `"newly funded" OR "just raised" operations director brand startup 2026`

Cap at 10 Tavily searches per dig session. If early queries return strong results, stop early. Do not run all 10 for the sake of thoroughness.

## Filtering Rules

For each result returned apply the profile rubric from the job-search skill. Additionally:

- If the role is at a large established company (over 500 employees), skip unless the mandate is explicitly 0-to-1
- If the title contains "coordinator," "specialist," or "analyst," skip unless the JD describes director-level scope
- If the location is Boston, skip
- If the location is NYC, SF, London, or Milan, apply the relocation scoring rules from the profile file
- If the JD mentions "retail floor," "store operations," "visual merchandising," or "distribution center," skip
- Read the JD for founder proximity signals: "work closely with founders," "report directly to CEO," "join a small team" — these are positive signals even if the title is generic

## Output

Deliver results inline in the standard scuttle report format: numbered list, Role Fit and Cred scores, one-sentence justification, URL.

Cap report at 15 roles. If more passed filtering, surface the highest-scoring ones only.

After delivering the report, run the feedback loop: prompt the user to react to each role (applied / pass / ignore [reason] / tell me more). Write reactions to the patterns file exactly as the scuttle feedback loop does.

If nothing worth surfacing was found, say so plainly: "Nothing strong this cycle. [N] queries run, all noise."

## Token Budget

A clean dig session should cost 25-40K tokens. Cap is 50K. If approaching the cap, stop querying and deliver what you have.
