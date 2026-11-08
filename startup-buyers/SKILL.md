---
name: startup-buyers
description: >-
  Puts a business idea in front of 100 simulated buyers (20 with --quick), each a
  separate sub-agent with their own income, budget, shopping habits, what they use
  today and how much the problem really bothers them, and counts who would pay now,
  who passes and why, what would change their mind and the price range they would
  accept. Use when someone asks "would people buy this", "test it on customers",
  "simulate buyers", "run the buyer panel", "is there demand", "who is my customer"
  or "what should I charge", or as step 2 of the startup-skill pack.
argument-hint: "[--quick | --n 100] [--seed S]"
---

# startup-buyers

Instead of guessing whether people will buy, ask a hundred of them. Each buyer is a
separate sub-agent playing one specific person drawn from the target customer. You
run the panel. You never answer for a buyer.

What the user typed after the command: `$ARGUMENTS`. `--quick` means 20 buyers.

## The tool

All the bookkeeping goes through `buyers.py`, in the same folder as this file:

```bash
python3 "${CLAUDE_SKILL_DIR}/buyers.py" <command>
```

Below, `TOOL` means exactly that. If `${CLAUDE_SKILL_DIR}` is not expanded, use this
skill's base directory (Claude Code prints it when the skill loads; in Codex it is
the folder this SKILL.md was read from). Everything is written to `startup/buyers/`
in the user's project.

## Step 1: the target customer

Read `startup/idea.md` if it exists (`/startup-board` writes it). Then write
`startup/buyers/customer.json`. The shape is in `customer.example.json` next to this
file:

| field | what goes in it |
| --- | --- |
| `business`, `target` | the business, and who the customer is in one specific line |
| `category` | what the buyer is spending on, in their words ("software that runs your bookings") |
| `budget_period`, `offer_price` | `month`, `year` or `purchase`, and the price in the pitch as a number in that period |
| `free_trial` | `true` if the pitch starts with a free trial. A buy then means "I would start the trial today and expect to pay after it", since nobody pays on day one of a free trial. Leave it out otherwise |
| `segments` | 2 to 4 groups, each with a `share`, an `income` range and a `budget` range: what they already spend on this kind of problem, or would spend without thinking twice. Not the most they would ever pay: buyers can go above it when the value is clear |
| `behaviours` | 4 to 6 ways these people shop, each with a `share` and a `how` line |
| `today` | what they do about the problem today, each with a `share`. Include "nothing, they live with it" when that is true for some. Write each so it reads after "What you do about this today:" |
| `situations` | everyday details that make a buyer concrete. For business buyers, include what the problem costs them in money ("a missed call last month was a $600 job"), so they can weigh the value against the price |
| `pain` | optional: how much the problem bothers them. Leave it out to use the default spread, where a quarter barely feel it and one in ten is looking for a fix right now. Change it only with evidence the user gives you |
| `known_objections` | the doubts the founder already expects. Never shown to buyers: you compare them with what buyers say, to find the objections nobody saw coming |

Ground it in what the user knows: their customers, interviews, reviews, survey
answers. Where you are guessing, say so. Show the user the profile in a few lines
(who, the segments with their budgets, what they use today) and let them correct it
before you deal. A wrong customer makes the whole panel wrong.

## Step 2: the pitch

Write `startup/buyers/pitch.md`: what a buyer would actually see. What it is, the
price, how to get it, and the first step the business will really offer (a free
trial, a money-back guarantee, a founding-member price). Buyers judge the first step
they are offered, so leaving out a trial the business would give makes the panel
harsher than reality; never add one the business would not give. Three to six plain
lines, no superlatives, nothing the business cannot deliver. Every buyer sees exactly
this text. Check that `offer_price` in the profile matches the price in the pitch. If
the price changes later, run the buyers again at the new price (see "Testing a changed
pitch") so the verdict and the panel describe the same offer.

## Step 3: deal and run

```bash
TOOL init --customer startup/buyers/customer.json --pitch startup/buyers/pitch.md --n 100
TOOL briefs
```

`init` deals the buyer cards and records the seed (the same profile and seed deal the
same buyers again). `briefs` writes one brief per buyer and lists them in waves of 10.

If `startup/buyers/` already holds answers from an earlier panel, `init` refuses, so
old answers never mix into a new panel. Ask the user which they want: keep the old
panel and deal the new one into its own folder (`TOOL --dir startup/buyers-v2 init
...`, then point the other steps at it), or add `--replace` to delete the old answers
and start again.

Before a 100-buyer run, tell the user it means 100 sub-agent runs. Each one is a full
sub-agent, so the run uses a large share of their Claude plan's usage; `--quick` (20
buyers) is a much cheaper first look. If they want to save usage on a full run, offer
to launch the buyers on a smaller model where the host allows it (in Claude Code, the
Agent tool's `model` set to `haiku`), and say in the results that you did. Each buyer
writes one file, so in Claude Code suggest accept-edits mode (Shift+Tab) for the run,
or the user approves every file.

For each wave, send ONE message with one sub-agent call per buyer in that wave. In
Claude Code that is the Agent tool (called Task in older versions):

- `subagent_type`: `general-purpose`
- `description`: `buyer <id>`
- `prompt`: `Read <brief path> and do exactly what it says. It is your whole brief.`

Wait for the whole wave to finish before starting the next. In Codex, see
`references/hosts.md` in this folder. After the last wave:

```bash
TOOL check     # lists missing or broken answers
TOOL briefs    # rewrites briefs for those buyers only; run them once more
TOOL tally
```

If a buyer printed its JSON in its reply instead of writing the file, file that reply
with `TOOL save <id>` and the reply on stdin. `save` refuses anything that is not a
valid answer. A buyer that fails twice is left out and counted as left out. Never
write or fix an answer for a buyer, and never edit one buyer's brief.

## Step 4: tell the user what happened

`TOOL tally` writes `startup/buyers/results.md` and `results.json`. Give the user:

1. The headline: how many would buy (pay now, or start the free trial and mean to pay
   after it), how many said probably, how many passed. With a free trial, say once that
   no money changes hands in a trial, so fewer real trials turn into customers.
2. Who buys and who doesn't: the rows that differ most (segment, how much the problem
   bothers them, what they use today).
3. Why they pass: the top three reasons, with one buyer's words for each.
4. **The objections nobody expected**: what buyers raised that is not in
   `known_objections`. This is often the most valuable line in the whole panel.
5. What would change their mind: the answers that repeat.
6. The price range they would accept, where the price sits in it, and the budget
   check (buyers who said yes but whose budget is below the price).
7. One sentence on what to change before spending money.

## Honest limits, said once

These are simulated buyers. They are good at surfacing objections, segments and
wording, and bad at predicting how many people will really pay: language models lean
agreeable, so read the buy rate as the best case. Confirm the big calls with real
people before signing anything (`/startup-plan` ends with that test). Never present a
buyer's words as a customer testimonial.

## Testing a changed pitch

To see whether a new pitch or price moves the result, keep the old results, write the
new pitch, and deal the same buyers again into a new folder:

```bash
TOOL --dir startup/buyers-v2 init --customer startup/buyers/customer.json --pitch startup/buyers-v2/pitch.md --n 100 --seed <the seed from results.md>
```

Then compare the two `results.md`. If nothing moved, say so: the problem may be the
market, not the wording.

## Output

`startup/buyers/` (customer.json, pitch.md, cards, briefs, answers, results.md,
results.json). Next: `/startup-numbers`, which uses the buy rate and the price range.
