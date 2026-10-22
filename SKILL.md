---
name: harness-pick
description: Decide which AI coding harness and model should take a task (whatever is installed on this machine: Claude Code, Codex CLI, pi, Cursor, and their configured models) with one typed decision on the OpenAI Decisions API (gpt-6-luna) over a task-to-harness map that the skill builds and refreshes from current evidence. Ships no presets. TRIGGER before opening a thread, delegating a task to an agent, assigning work to a subagent of another harness, or when the user asks "which harness", "which model", "who should do this", "Claude or codex"; TRIGGER with the word "update" (/harness-pick update) to build or refresh the map, and on the first use when no map exists yet. SKIP for choosing a model inside one API call.
---

# harness-pick

One decision per task, made by a model with no stake in the answer, over a map that names its evidence and only offers what is installed here. The skill ships empty: the first use builds the map and the catalog from current sources and this machine's inventory, and later uses decide.

## First use, and `/harness-pick update`

Run `scripts/harness-pick --status`. When the catalog is missing, or the user asked for an update, or the map's sources are more than a month old, do the update procedure before anything else:

1. **Inventory.** `scripts/harness-inventory --refresh --text`: the harnesses installed here and the models each is configured for. Only these can be offered.
2. **Research with web search, one batch, the current month in each query:** the SWE-bench Pro leaderboard; the latest Terminal-Bench leaderboard; a comparison of the installed harnesses with cost per run; a code-review benchmark on real bugs; a knowledge-work ranking (GDPval or similar); the current model tiers and prices of each installed harness's vendor; the Decisions API status and price; any harness or model released since the last update. Prefer leaderboards and vendor pages over blogs; put a date on every figure; when sources disagree, keep both with dates.
3. **Ask the user once**, with a multiple-choice question when the host has one: what kind of work this machine mostly does (a few words), and whether a one-paragraph context should be prepended to every decision. Write the answer to `references/context.md` when they want one; otherwise leave it out.
4. **Write `references/harness-selection.md`:** a table with one row per task type (agentic code change in a repository; terminal and CI work; documentation; code review of another model's work; design judgement and adversarial counter-argument; spec, schema and codegen; triage and classification; mechanical bulk edits; research and synthesis; plus any row the user's work needs), each with the first choice among the installed harnesses, the reason with its figure and date, and the installed runner-up; then "Sources read" with dates; then "Caveats"; then an "Evidence log" table (date, task, harness and model, outcome) starting with one row "Map built". A harness that is not installed is never a first choice; say "not installed here" where the evidence would have picked it.
5. **Write `references/catalog.json`:** `{"updated": "<date>", "options": [{"value": "<harness>-<tier>", "needs": {"harness": "<binary name>", "model": "<regex on the configured model names>"}, "description": "<harness and model>: <the task types it is first choice for, from the table>"}, ...]}`. One option per installed harness and tier that the table names, plus `{"value": "luna-decision", "needs": {}, "description": "OpenAI Decisions API on gpt-6-luna: a classification, triage or routing answer with a confidence, no code and no prose"}`. The descriptions are what Luna reads, so they must say what the table says. Keep every value the evidence log still names.
6. **Calibrate.** Run the three calibration tasks below through `scripts/harness-pick` and record the picks in the map under "Calibration" with the date. On an update, a flipped pick is reported, not silently accepted.
7. **Report** in under 150 words: rows written or changed with the reason, rows kept against new evidence and why, calibration picks, sources that could not be reached.

On an update, change a row's choice only when the new evidence and the evidence log agree, or when the log alone shows a repeated failure (two or more tasks). Append to the evidence log; never rewrite old rows. Never let one vendor's own benchmark decide alone.

## Decide

```bash
scripts/harness-pick "<one paragraph: what the task changes, where, and how it is proven>"
```

Output: `pick`, its confidence, the runner-up, and the input tokens. `--json` prints the raw answer; `--options` shows exactly what the decider is offered on this machine; `--refresh` re-probes the inventory (after installing a harness or changing a configured model; otherwise cached a day).

- Describe the work, not the wished-for answer: name the repositories, the kind of change (code, CI, docs, bulk edit, review, triage) and the proof it needs.
- Confidence under 0.6, a refusal, or a network error: decide by the map and say so.
- Within the picked harness, choose the tier by stakes: the top tier for shared contracts and production paths, the mid tier for bounded work in one repository, the small tier or Luna for triage.
- Code review goes to a different model family from the one that wrote the code; no decision is needed for that.
- Never restart a running task to match a pick.

## Record

Append one row to the evidence log in `references/harness-selection.md` (newest first) when the pick is made and again when the outcome is known (review findings, merges, restarts). The log is the only evidence specific to this machine; the public benchmarks are snapshots.

## Calibration tasks

```
scripts/harness-pick "Migrate a service's API verifier to a new auth library with identity mapping at the boundary, proven on a staging deployment"
scripts/harness-pick "Write the operations runbook for a service: deploys, key rotation, incident steps, from the repository's workflows and infrastructure code"
scripts/harness-pick "Classify a one-line chat message as a bug report, a question or a release note"
```

## Requirements

An OpenAI key with access to the Decisions API (`POST /v1/decisions`, model `gpt-6-luna`): `OPENAI_API_KEY`, else the key Codex CLI is signed in with (`~/.codex/auth.json`). Python 3. Web search in the harness running the update. Nothing else; the inventory only reports what it finds. Keys are never printed.

## Files

- `scripts/harness-pick`: the decider. Exit 3 means no catalog yet.
- `scripts/harness-inventory`: the probe (claude, codex, pi, cursor-agent, kimi, and presence of others on PATH; `--text` for a readable list). Extend it with a probe when it misses an installed harness.
- `references/`: generated on this machine, gitignored, never shipped. Delete to start over.
