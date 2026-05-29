---
name: competitive-research
description: Produce a structured competitive research report (profile, feature comparison, pricing, marketing/messaging) for one or more competitors. Triggers on "competitive research", "competitor profile", "battle card", "research <competitor>", "compare X vs Y", or the slash command `/competitive-research`.
---

# Competitive Research

You are running the `competitive-research` workflow. Your job is to orchestrate a multi-dimensional competitive research run and deliver a markdown report to disk.

## Inputs

The user may invoke this as:
- `/competitive-research <competitor>`
- `/competitive-research <competitorA>,<competitorB>`
- `/competitive-research` (no args)
- Natural language ("research Snowflake against us")

## Steps

1. **Parse competitor names** from the user's message / args. Strip whitespace, split on commas.
2. **If no competitors were given**, use `AskUserQuestion` to ask which competitor(s) to research. Accept free-text comma-separated input.
3. **Ask one clarifying round** with `AskUserQuestion` (combine into a single call with up to 2 questions):
   - Comparison target (default: **DataRobot**) — what to compare each competitor *against* in the feature matrix.
   - Source allowlist — which of `web`, `amplitude-ai-visibility`, `g2-reviews`, `internal-docs` to use. Default: all on.
   Skip this step if the user already specified these in their message.
4. **Compute the output path**: `reports/<kebab-slug-of-competitors>-YYYY-MM-DD.md`. Use today's date. Slug example: `snowflake-databricks-2026-05-28.md`.
5. **Spawn the `competitive-researcher` orchestrator subagent** via the `Agent` tool (subagent_type: `competitive-researcher`). The prompt must be self-contained and include:
   - The competitor list
   - The comparison target
   - The sources allowlist
   - The exact output path to write to
   - Today's date (so the subagent can pass it down)
6. **When the subagent returns**, print a 3–5 line summary and a markdown link to the report file (path relative to the working directory).

## Rules

- Do **not** do the research yourself in the main context. Always delegate to the orchestrator subagent.
- Do **not** ask more than one round of clarifying questions.
- Always write to the `reports/` directory at the repo root. Create it if missing.
- If the orchestrator fails or returns no file, surface the error to the user with what was attempted.
