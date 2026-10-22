# Generated files

Nothing in this folder ships with the skill. The first run of the update procedure (`/harness-pick update`) writes:

- `catalog.json`: the options the decider is offered, each with the harness binary and model-name pattern it requires and a one-sentence description with dated evidence.
- `harness-selection.md`: the task-to-harness map, the sources read with dates, the caveats, and the evidence log of picks and their outcomes on this machine.
- `context.md` (optional): one paragraph about the team and the kind of work, prepended to every decision input.

All three are gitignored. Delete them to start over.
