---
name: graphify-wrapper-index
description: Use when a named domain index needs registering or removing by hand - "add an index for services/backend", "drop that domain" - discovery via /graphify-wrapper-map, build via /graphify-wrapper-sync.
argument-hint: "<name> <path> [--semantic]  (or: <name> to remove)"
allowed-tools:
  - Bash
  - Read
---

# /graphify-wrapper-index

Manage the set of named domain indexes for this repo. A domain = a `name` + a
repo-relative subtree `path` (e.g. `backend services/backend`). Scoping the
monorepo into domains keeps each graph small and each semantic build cheap.

```bash
. .better-dev/bin/bd-gfx 2>/dev/null || . "${CLAUDE_PLUGIN_ROOT}/scripts/bd-gfx"
reg=$(gfx_registry)
[ -f "$reg" ] || { echo "run /graphify-wrapper-setup first"; exit 1; }
```

## Register a domain

Validate and upsert into the registry. `--semantic` marks the domain for full
extract on sync (default AST-only).

```bash
name="$1"; idx_path="$2"; sem=false
case "$*" in *--semantic*) sem=true;; esac
root=$(gfx_this_worktree)
# The name becomes a directory under the graph home, so it is checked like one.
case "$name" in ''|*[!A-Za-z0-9._-]*|*..*)
  echo "invalid domain name: $name (letters, digits, dot, underscore, dash)"; exit 1;; esac
[ -d "$root/$idx_path" ] || { echo "path not found in repo: $idx_path"; exit 1; }
tmp=$(mktemp "$(dirname "$reg")/.reg.XXXXXX")
jq --arg n "$name" --arg p "$idx_path" --argjson s "$sem" \
   '.indexes[$n]={path:$p, semantic:$s}' "$reg" > "$tmp" && mv "$tmp" "$reg"
echo "registered '$name' -> $idx_path (semantic=$sem)"
jq '.indexes' "$reg"
```

## Don't know the domains yet?

Use **`/graphify-wrapper-map`** - it analyzes the repo, proposes a focused
domain set, refines it with you interactively, and registers the chosen ones.

## Removing a domain

Drop the registry entry, then this worktree's graph for it: nothing can reach that
output once the name is gone, so leaving it behind leaks the whole dir forever.
Other worktrees' copies go when their own tree does (they are reaped at session
start), or on their next `/graphify-wrapper-index` removal.

```bash
tmp=$(mktemp "$(dirname "$reg")/.reg.XXXXXX"); jq --arg n "$1" 'del(.indexes[$n])' "$reg" > "$tmp" && mv "$tmp" "$reg"
out=$(gfx_out_dir "$1") && rm -rf "$out"
```
