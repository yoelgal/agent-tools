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

0.9.0 reonboard,offer - communication style can be installed machine-wide at bootstrap (BOOTSTRAP step 2b writes the block into the host's bd_host_global_entry file); block body single-sourced to docs/comms-block.md; /onboard now skips the per-repo block on a solo adoption when the machine already carries it globally, and still writes it for team
0.8.0 install,reonboard - new /gauntlet skill (grill an idea into a Gauntlet Loop handoff prompt for a fresh session); routing row added to the onboard CLAUDE.md block template, /groundwork offers the gauntlet fork at entry
0.7.0 install,reonboard - repo became the agent-tools monorepo: better-dev moved into better-dev/, GitHub repo renamed yoelgal/agent-tools; re-run the installer at its NEW path <clone>/better-dev/install.sh so host links follow, then /onboard per wired repo to refresh the .better-dev/bin bridge
0.6.0 install,reonboard - /update verb and versioned update path (wired-version stamp, reonboard nudge); ADHD comms-style block at onboard
