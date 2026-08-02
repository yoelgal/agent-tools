---
name: gauntlet
description: Use when the deliverable is a Gauntlet Loop prompt - "gauntlet this", "one-shot the whole thing", "write me a prompt that builds X in a fresh session against a real bar", or a front-end offered the gauntlet route on a greenfield ask and the user took it. The output is one handoff prompt for a fresh agentic session; the build itself never runs here. For a feature in an existing codebase reach for /plan-grill; for an epic that needs a shared foundation and parallelizable work-items, /groundwork.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Gauntlet - grill the idea, hand off one loop prompt

A Gauntlet Loop is a way to run one long autonomous build: hand the lead agent a destination
plus a reference artifact worth beating, then let builder-critic loops chase that reference
until the human calls the run (step 3 owns the operative mechanics). The method comes from Matt
Shumer's Claude-of-Duty run (credited in `NOTICE`); it generalizes past games to any software
the loop can inspect: a web app, a CLI, a design system, an API.

This skill's whole job is the prompt. It grills the idea until the prompt can be written, writes
it, and hands it to the user to paste into a **fresh** session - fresh because the run wants its
entire context budget, and because this conversation's history would leak your framing into
decompositions the lead agent should own. When the conversation drifts toward building the thing
here, hand the prompt over instead - a gauntlet run inside a half-spent session is the method minus
its advantage.

Read `.better-dev/overrides.md` first when a repo is wired; a project override wins over any
default below.

## 1. Grill until the slots are settled

Six slots decide the prompt. Batch the questions the conversation has not already answered; propose
a default wherever one is sensible and let the user correct it - the grill is done when every slot
is either filled or deliberately delegated to the run, not when the user stops answering.

| Slot | What settles it |
|---|---|
| Goal | The destination in one or two sentences - what exists when the run is over, for whom. |
| Bar | A concrete reference the critic can hold the artifact against (step 2). |
| House rules | The handful of always-true fences (stack constraints, "no hard-coded special cases", licensing) - an empty list is a valid, deliberate answer. Asset provenance belongs here whenever the artifact has assets: nobody is around to fetch a mesh or hand over a logo, so default to generating them - textures, meshes, and sounds for a game, logo and icons for an app, real copy over lorem - and name the few the run may fetch instead. |
| Harness | Which agentic harness runs it, and whether subagents and looping are available there - the prompt leans on both. A plain chat window fails this slot - the run needs an agent that can open files, run code, render output, and spawn workers - so when the user has only a chat surface, name a harness they could run it in and hold the prompt until one is settled. |
| Stop condition | Who stops the run and on what: a spend ceiling, a wall-clock bound, or "user stops it when satisfied". The loop never decides it is finished. |
| Progress surface | Where the human watches without interrupting: a live HTML page or a running markdown doc the agent keeps updating with screenshots, drafts, and test results. |

Predict the user's answer before asking a slot's question; ask only where the prediction is
genuinely uncertain. Six questions fired as a form is a worse grill than two aimed ones.

## 2. Pin a real bar

The bar is the load-bearing slot. An adjective is not a bar - "amazing", "production-ready", and
"AAA quality" each let the run coast to the model's private sense of sufficient, which sits
below the user's. A bar passes one test: could a fresh critic, given only the bar and the
artifact, decide which is better without asking a question? Screenshots of the market leader,
a set of reference sites, paragraphs at the clarity the writing should reach, a test suite plus
a latency target - all pass. The bar may be unreachable on purpose: it exists to give the loop a
direction and to keep it
from settling the moment the result is merely impressive for a machine, not to be met.

When the user has no bar, offer the two honest moves: pick a comp together now, or open the
prompt with a bar-finding task - the run locates a real-world reference its critic can inspect,
states in one sentence why that reference deserves to be the bar, and holds every round against
it. Writing the prompt around an adjective because the user shrugged is the one failure this
skill exists to prevent.

## 3. Write the prompt - goal and bar, never the route

Keep the ratio of the original: everything about the quality process is present, everything
specific to the artifact's construction is absent. The prompt names the goal, the bar, the fences,
and the loop mechanics - and leaves architecture, decomposition, and ordering to the lead agent.
Every step you dictate replaces the run's judgment with this conversation's, and this conversation
has not seen the artifact; a gauntlet prompt that has grown past roughly a dozen sentences is
prescribing, so move the detail into a house rule or delete it.

The mechanics the prompt must carry, in whatever words fit: the lead agent carves the goal into
the smallest units the loop can better and grade on their own; a builder and a separate
fresh-context critic per unit; the critic inspects the real artifact - rendered pixels, a
running binary, actual test output - never the builder's summary, compares it against the bar
blind where possible, and
names the biggest remaining gap; loop with no fixed round count; keep the progress surface updated;
optionally, one fresh smoothing agent per major wave to make separately improved pieces feel like
one thing. Spend the harness's heavy mode (ultracode or equivalent) only when the run is a
foundation others will build on for months; an ordinary loop held to a demanding bar usually
gets there without it.

One filled example - a non-game, mixed visual and mechanical bar:

> Build a local-first personal-finance dashboard: CSV import from any bank export, monthly
> spending breakdowns, and budget alerts, as a web app I run on my own machine. Two bars, judged
> separately: the UI must win a blind side-by-side against the attached Copilot Money screenshots,
> and the importer must pass a test suite you write first that covers the five bank-export formats
> in the samples/ folder, including the malformed ones. House rules: TypeScript, no cloud calls,
> data stays in one local SQLite file, and every icon, chart style, and bit of copy is generated by
> you - no stock imagery, no lorem. Carve the work into the smallest units you can improve and
> grade independently - the split is yours. Assign each unit one builder and one harsh critic
> that starts from a clean context; the critic looks at the real rendered UI or the real
> test run, compares against the bar side by side while blind to which one is ours, names the
> biggest gap, and sends it back. Keep looping - there is no final round; I stop the run. Maintain
> a live progress page with screenshots and test results as you go.

## 4. Hand it off

Deliver the prompt as one paste-ready block, and put it on the clipboard where the host offers a
clipboard command. With it, three run notes: paste into a fresh session of the named harness, not
into this one and not into a plain chat; monitoring happens on the progress surface, so resist
interrupting the run for status; stopping is the user's move, per the stop-condition slot. The
skill's terminal state is the handoff - a run started, watched, or debugged afterwards is its own
new work, and a wired repo's loop items still route through `/plan-grill` and `/autonomous-loop`.

Close out in one line: record the keyed lesson if the grill taught one
(`.better-dev/bin/bd-mem learn "<lesson>" <confidence> "<key>"` where a repo is wired), else say
`no durable lesson` and why.
