---
name: graphify-wrapper-status
description: Use when someone wants to see the registered domain indexes and their freshness for this worktree - "graphify status", "which graphs are built", "are my indexes stale".
allowed-tools:
  - Bash
---

# /graphify-wrapper-status

```bash
. .better-dev/bin/bd-gfx 2>/dev/null || . "${CLAUDE_PLUGIN_ROOT}/scripts/bd-gfx"
reg=$(gfx_registry)
[ -f "$reg" ] || { echo "not set up - run /graphify-wrapper-setup"; exit 0; }

this=$(gfx_this_worktree)
echo "repo_key   : $(gfx_repo_key)"
echo "backend    : $(gfx_backend)"
echo "this wt    : $this"
echo "graphs     : $(gfx_out_dir)"
echo "registry   : $reg"
echo

head=$(git -C "$this" rev-parse HEAD 2>/dev/null)
printf '%-14s %-28s %-9s %-8s %-8s %s\n' INDEX PATH SEMANTIC GRAPH FRESH BUILT
for name in $(gfx_index_names); do
  idx_path=$(gfx_index_field "$name" path)
  sem=$(gfx_index_field "$name" semantic)
  g="$(gfx_out_dir "$name")/graph.json"
  if [ -f "$g" ]; then
    read -r nodes built <<<"$(jq -r '"\(.nodes|length) \(.built_at_commit // "?")"' "$g" 2>/dev/null)"
    ts=$(date -r "$g" '+%Y-%m-%d %H:%M' 2>/dev/null)
    if [ -z "$head" ] || [ "$built" = "?" ]; then fresh="?"
    elif [ "$built" = "$head" ]; then fresh="current"
    else fresh="behind"; fi
    printf '%-14s %-28s %-9s %-8s %-8s %s\n' "$name" "$idx_path" "$sem" "${nodes}n" "$fresh" "$ts"
  else
    printf '%-14s %-28s %-9s %-8s %-8s %s\n' "$name" "$idx_path" "$sem" "-" "-" "(not built here)"
  fi
done
```

- `(not built here)` → run `/graphify-wrapper-sync <name>`, or just ask a
  question: `/graphify-wrapper-query` builds a missing graph on first use. Graphs
  live under `graphs:` above, outside this or any repo, keyed per worktree - a
  fresh worktree starts with none.
- `FRESH = behind` → the graph's `built_at_commit` is behind HEAD. A
  SessionStart hook auto-runs an AST `graphify update` on drifted, affected
  domains; or run `/graphify-wrapper-sync <name>` now. The semantic layer isn't
  auto-refreshed - use `--semantic` when you need fresh community naming.
