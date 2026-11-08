# Running the buyers on each host

Each buyer must be a separate run that sees only its own brief. Nothing from this
conversation, and nothing from another buyer, may reach it. That separation is what
makes the panel 100 opinions instead of one opinion repeated.

| host | how to run one wave |
| --- | --- |
| Claude Code | One message with one Agent tool call per buyer (Task in older versions), `subagent_type: general-purpose`. About 10 run at once, so waves of 10. |
| Codex, multi-agent on | Spawn one agent per buyer in the wave with the same one-line prompt, then wait for all of them. |
| Codex, multi-agent off | Run one `codex exec` per buyer from the project folder, a wave at a time in the background, for example `codex exec -s workspace-write "Read <brief path> and do exactly what it says. It is your whole brief."`. Wait for the wave before starting the next. |
| A chat with no sub-agents | Do not run the panel. Say plainly that one model playing 100 buyers in a single conversation is one opinion, not 100, and suggest Claude Code. |

The prompt is the same everywhere: `Read <brief path> and do exactly what it says. It
is your whole brief.` After each host's run, `buyers.py check` tells you which
answers are missing.
