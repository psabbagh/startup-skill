---
name: startup-skill
description: >-
  Tests a startup or business idea before any money is spent, start to finish: a
  board of advisors scores it, 100 simulated buyers say whether they would pay, the
  numbers show whether it makes money, and a business plan ends with a computed
  verdict, Profitable or Not yet. Use when someone says "test my startup idea", "test
  my business idea", "is this idea any good", "would this business work", "validate
  my idea", "run the startup skill", or describes a business they are thinking of
  starting and asks what you think of it.
argument-hint: "<your business idea, or leave blank to use the conversation> [--quick]"
---

# startup-skill

Four steps, each its own skill, run in this order. Each one reads what the earlier
ones wrote, in a `startup/` folder in the user's project.

```
startup-board  ->  startup-buyers  ->  startup-numbers  ->  startup-plan
   the board        100 buyers          the numbers         the verdict
```

What the user typed after the command: `$ARGUMENTS`. If it is blank, take the idea
from the conversation. `--quick` runs 20 buyers instead of 100.

## Before you start, tell the user what will happen

In a few plain lines:

1. **The board**: several advisors, each applying one published playbook, score the
   idea and vote.
2. **The buyers**: 100 simulated buyers (20 with `--quick`), each a separate
   sub-agent, say whether they would pay and why. This is the longest step and uses
   the most of their plan. Each buyer writes one small file, so let it write without
   asking: in Claude Code, accept-edits mode (Shift+Tab); in Codex, the auto approval
   mode (or `codex --full-auto`).
3. **The numbers**: you will ask them for real numbers: the price, the costs, what they
   will pay themselves, how they will find customers.
4. **The plan**: one document with a verdict that is computed, not written: Profitable,
   or Not yet with the numbers that fail.

## Run the steps

For each step, run that skill. In Claude Code, invoke it with the Skill tool
(`startup-board`, then `startup-buyers`, `startup-numbers`, `startup-plan`). In
Codex, and on any host without that tool, read the step's `SKILL.md` in the sibling
folder next to this skill's folder (`../startup-board/SKILL.md` and so on, usually
under `~/.codex/skills/` in Codex) and follow it. Both hosts run the board and the
buyers as separate agents: Claude Code with its Agent tool, Codex with its
multi-agent feature.

- **Step 1, `startup-board`.** It writes `startup/idea.md` first and asks for anything
  missing. After the vote, show the user the result in two lines and carry on, unless
  the board's strongest version of the business is different enough that the user may
  want to change the idea first. Ask.
- **Step 2, `startup-buyers`** (pass `--quick` if the user asked for it). The user
  confirms the target customer before any buyer is dealt. Do not skip that. If few
  buyers would buy for reasons a better offer could answer (proof, risk, price, effort,
  missing information), offer one round of the skill's "Improve the pitch and re-test"
  before moving on. If the reasons are "no need" or "happy with what I use today",
  say so plainly instead: that is the market, not the pitch.
- **Step 3, `startup-numbers`.** Ask for everything only the user knows in one message.
- **Step 4, `startup-plan`.** It computes the verdict and writes the plan.

If a step cannot run (for example, the host has no sub-agents for the buyers), say so
plainly and stop. Never skip a step silently, and never fill a step's output from
guesses.

## At the end

Give the user, in this order:

1. The verdict in one line, and the three numbers behind it.
2. What the buyers and the board agreed was the biggest problem.
3. The one thing to do this week: usually the real-world test in `startup/real-test.md`.
4. Where everything is: `startup/plan.md`.

Then say once: the buyers are simulated and lean agreeable, so their yes is the best
case; real people and real quotes confirm it. Not financial, legal or tax advice.
