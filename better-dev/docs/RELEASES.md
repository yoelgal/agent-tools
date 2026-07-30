# Releases

Machine-read by the session-start hook and `/update`. One line per release, newest first:
`<version> <flags> - <summary>`, where `<flags>` is a comma-joined subset of `install,reonboard`.
Three tiers: **pull-only** (the default - `git pull` in the clone is the whole update), **install**
(a skill dir was added, removed, or renamed - re-run `install.sh` once per machine), **reonboard**
(a repo surface changed - re-run `/onboard` once per wired repo). A version with no line here is
pull-only; flags are never empty.

0.8.0 install,reonboard - new /gauntlet skill (grill an idea into a Gauntlet Loop handoff prompt for a fresh session); routing row added to the onboard CLAUDE.md block template, /groundwork offers the gauntlet fork at entry
0.7.0 install,reonboard - repo became the agent-tools monorepo: better-dev moved into better-dev/, GitHub repo renamed yoelgal/agent-tools; re-run the installer at its NEW path <clone>/better-dev/install.sh so host links follow, then /onboard per wired repo to refresh the .better-dev/bin bridge
0.6.0 install,reonboard - /update verb and versioned update path (wired-version stamp, reonboard nudge); ADHD comms-style block at onboard
