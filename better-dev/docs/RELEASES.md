# Releases

Machine-read by the session-start hook and `/update`. One line per release, newest first:
`<version> <flags> - <summary>`, where `<flags>` is a comma-joined subset of `install,reonboard`.
Three tiers: **pull-only** (the default - `git pull` in the clone is the whole update), **install**
(a skill dir was added, removed, or renamed - re-run `install.sh` once per machine), **reonboard**
(a repo surface changed - re-run `/onboard` once per wired repo). A version with no line here is
pull-only; flags are never empty.

0.7.0 install,reonboard - repo became the agent-tools monorepo: better-dev moved into better-dev/, GitHub repo renamed yoelgal/agent-tools; re-run install.sh so host links point at the new path, re-run /onboard so each repo's .better-dev/bin bridge follows
0.6.0 install,reonboard - /update verb and versioned update path (wired-version stamp, reonboard nudge); ADHD comms-style block at onboard
