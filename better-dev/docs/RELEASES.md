# Releases

Machine-read by the session-start hook and `/update`. One line per release, newest first:
`<version> <flags> - <summary>`, where `<flags>` is a comma-joined subset of
`install,reonboard,offer`. Four tiers: **pull-only** (the default - `git pull` in the clone is the
whole update), **install** (a skill dir was added, removed, or renamed - re-run `install.sh` once
per machine), **reonboard** (a repo surface changed - re-run `/onboard` once per wired repo), and
**offer** (the release added something opt-in - ask the operator once whether they want it). A
version with no line here is pull-only; flags are never empty.

The first three tiers say *do this to stay current*; `offer` is the only one that asks a question,
and it exists because a new capability otherwise reaches new installs alone. A first install meets
an opt-in capability at its own front door (`BOOTSTRAP.md`); an already-wired machine never runs
that again, so without this flag the capability ships invisible to every existing user. `/update`
collects pending `offer` flags exactly like the other two and puts each summary to the operator as
a one-time question; the `wired-version` stamp closes it, so a declined offer is not re-asked.

**An `offer` line always carries `reonboard` too.** Only the reonboard nudge fires from
`bd-session-start`, so an offer without it is an offer nobody is ever prompted to collect. The
package gate enforces the pairing.

0.9.7 reonboard,offer - graphify graphs leave the tree they index: every build now writes under `~/.claude/graphify/<repo key>/worktrees/<worktree key>/<domain>/`, so a build leaves the indexed repo byte-unchanged and the never-commit guard setup used to install (a global-gitignore write) is retired. Also: a graph left behind by a previous worktree at the same path is rebuilt instead of answered from, and a graphify home that sits inside the tree it indexes is refused rather than written into (an absolute path outside the repo is now required). Known limitation: a torn-down worktree's graph dir stays under the home; `rm -rf` of the home is safe and costs one AST rebuild. OFFER: undo the retired guard on this machine, since retiring it going forward does not reverse the write already made - drop the `graphify-out/` line from your global gitignore file, and run `git config --global --unset core.excludesfile` **only** where better-dev set that pointer (dropping the line alone leaves git's global ignore resolution re-pointed at a file it did not choose)
0.9.6 reonboard,offer - a greenfield onboard now ends owing the operator nothing: /guardrails-install gains the no-stack guard it never had (installs the stack-agnostic secret scan, then defers the none-placeholders, the merge-policy and release-cadence questions, and the bd-guard enforcement paste block to a re-run after /groundwork lands a stack), and the standing bd-mem/bd-guard permission allowance moves to install time so it is granted once per machine instead of re-asked in every repo forever. OFFER: grant the two allow rules in ~/.claude/settings.json now, so no future repo asks again
0.9.4 reonboard - graphify actually runs: fixed the zsh path-variable trap that wiped PATH inside every graphify wrapper block (the reason no repo ever built a graph), the query path now builds registry+domain+AST graph on first use instead of exiting 1, added the --hubs (god-nodes) verb and the save-result/reflect retrieval-memory loop, and /review, /autonomous-loop, /groundwork, /codebase-audit, /codebase-map now name the specific verb for their specific question; package gate refuses a bare `path` variable in skill blocks
0.9.3 reonboard - /onboard friction pass: it now ends standing on the integration branch, defers graphify until the repo has code to index, proposes the bd-* permission allowance machine-wide (one paste per machine instead of one per repo), fixes the .better-dev/.gitignore 'bin/' pattern that left the bridge tracked, and skips the comms-block confirm where no teammates exist; the greenfield close-out and the CLAUDE.md block row now name the steered-vs-one-shot fork, and /groundwork asks it rather than judging it; /gauntlet prompts must carry subagents and ultracode
0.9.0 reonboard,offer - communication style can be installed machine-wide at bootstrap (BOOTSTRAP step 2b writes the block into the host's bd_host_global_entry file); block body single-sourced to docs/comms-block.md; /onboard now skips the per-repo block on a solo adoption when the machine already carries it globally, and still writes it for team
0.8.0 install,reonboard - new /gauntlet skill (grill an idea into a Gauntlet Loop handoff prompt for a fresh session); routing row added to the onboard CLAUDE.md block template, /groundwork offers the gauntlet fork at entry
0.7.0 install,reonboard - repo became the agent-tools monorepo: better-dev moved into better-dev/, GitHub repo renamed yoelgal/agent-tools; re-run the installer at its NEW path <clone>/better-dev/install.sh so host links follow, then /onboard per wired repo to refresh the .better-dev/bin bridge
0.6.0 install,reonboard - /update verb and versioned update path (wired-version stamp, reonboard nudge); ADHD comms-style block at onboard
