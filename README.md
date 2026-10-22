# harness-pick

A skill for Claude Code (and any harness that reads `~/.agents/skills`) that decides which installed AI coding harness and model should take a task. One typed decision per task on the OpenAI Decisions API (`gpt-6-luna`), judged over a task-to-harness map the skill builds itself from current leaderboards, vendor pages and the outcomes recorded on your machine.

It ships no presets. The first use runs the update procedure: it inventories the harnesses installed here and the models each is configured for, researches current evidence, writes the map and the decider's catalog under `references/` (gitignored), and calibrates. After that, `/harness-pick <task>` decides and `/harness-pick update` refreshes.

## Install

Claude Code, user scope:

```bash
git clone https://github.com/MasonRhodesDev/harness-pick ~/.claude/skills/harness-pick
```

Shared across harnesses (Codex CLI and pi read this directory natively; Claude Code and Cursor need a copy, which a skill registry can make):

```bash
git clone https://github.com/MasonRhodesDev/harness-pick ~/.agents/skills/harness-pick
```

Then, in a Claude Code session: `/harness-pick update`. That first run needs web search and an OpenAI key with Decisions API access (`OPENAI_API_KEY`, or the key Codex CLI is signed in with).

## Use

```
/harness-pick Migrate the billing service's token verifier to the new auth library, proven on the staging stack
```

The reply names the pick, its confidence and the runner-up, and the session records the outcome in the map's evidence log when it is known.

Command line:

```bash
scripts/harness-pick "<task>"      # decide
scripts/harness-pick --options     # what the decider is offered on this machine
scripts/harness-pick --status      # catalog and map present? dates?
scripts/harness-inventory --text   # installed harnesses and configured models
```

## How it chooses

- Only installed harnesses are offered, each described with its version, matching configured models and default; the whole inventory goes into the decision input.
- The decision weighs capability first and prefers the cheaper option when the quality gap is small, so the strongest tier wins only the tasks that need it.
- The map's rows carry a figure and a date for every claim, and the evidence log on your machine is the tie-break.
- Confidence under 0.6, a refusal or an outage means "decide by the map", and the skill says so.
- Code review goes to a different model family from the author; a running task is never restarted to match a pick.

## Requirements

Python 3; an OpenAI key with Decisions API access; a harness with web search for the update. The inventory probes `claude`, `codex`, `pi`, `cursor-agent` and `kimi` and notes the presence of others on PATH.

## License

Apache 2.0.
