---
name: graphify-wrapper-sync
description: Use when a worktree's domain graphs need building or refreshing - "sync the index", "rebuild the graph", "refresh graphify" - AST-only by default, --semantic for the full extract.
argument-hint: "[<name> - defaults to all registered] [--semantic]"
allowed-tools:
  - Bash
---

# /graphify-wrapper-sync

This is the only thing that builds graphs.

```bash
. .better-dev/bin/bd-gfx 2>/dev/null || . "${CLAUDE_PLUGIN_ROOT}/scripts/bd-gfx"
reg=$(gfx_registry)
[ -f "$reg" ] || { echo "run /graphify-wrapper-setup first"; exit 1; }
this=$(gfx_this_worktree); main=$(gfx_main_worktree)
sem=false; case "$*" in *--semantic*) sem=true;; esac
target="${1:-}"; case "$target" in --*) target="";; esac   # ignore flags as name
names=$(if [ -n "$target" ]; then echo "$target"; else gfx_index_names; fi)
[ -n "$names" ] || { echo "no indexes registered - run /graphify-wrapper-index"; exit 1; }
```

## Per-index loop

For each name, resolve its subtree, then build. The rule:

- **Refresh** if a graph already exists at `<path>/graphify-out/`.
- **Seed then refresh** if not, and this worktree is _not_ the main one and main
  has a graph for that domain: copy main's graph in first so the worktree
  inherits main's (expensive, possibly semantic) layer, then AST-reconcile the
  branch diff cheaply.
- **Build from scratch** otherwise.

```bash
backend=$(gfx_backend)
# Image/asset files graphify would otherwise read as text "docs" - token noise
# with no architectural signal, and the source of oversized chunks that time out.
exc_args=(); while IFS= read -r p; do [ -n "$p" ] && exc_args+=(--exclude "$p"); done < <(gfx_extract_excludes)
for name in $names; do
  idx_path=$(gfx_index_field "$name" path)
  [ -n "$idx_path" ] || { echo "skip '$name': not registered"; continue; }
  want_sem=$(gfx_index_field "$name" semantic)
  if [ "$sem" = true ] || [ "$want_sem" = true ]; then do_sem=true; else do_sem=false; fi
  dst="$this/$idx_path"; out="$dst/graphify-out"
  [ -d "$dst" ] || { echo "skip '$name': $idx_path absent in this worktree"; continue; }

  # Repair a torn local graph.json (a partial/killed prior copy) so the seed
  # decision below treats the domain as missing rather than complete - graphify's
  # shrink guard would otherwise refuse `update` on an unparsable graph.
  if [ -f "$out/graph.json" ] && ! jq -e . "$out/graph.json" >/dev/null 2>&1; then
    echo "[$name] removing unparsable graph.json (will re-seed/rebuild)"
    rm -f "$out/graph.json"
  fi

  # Seed from main if this worktree has no graph yet. Copy the siblings first,
  # then install main's graph.json last via a same-dir temp + mv, so an
  # interrupted copy never leaves an unparsable graph a later path trusts.
  src="$main/$idx_path/graphify-out"
  if [ ! -f "$out/graph.json" ] && [ -n "$main" ] && [ "$main" != "$this" ] \
     && jq -e . "$src/graph.json" >/dev/null 2>&1; then
    echo "[$name] seeding from main worktree"
    mkdir -p "$out"
    for f in "$src"/* "$src"/.*; do
      b=${f##*/}; case "$b" in "*"|.|..|graph.json) continue;; esac
      [ -e "$f" ] || continue; cp -R "$f" "$out/" 2>/dev/null || true
    done
    tmp="$out/.graph.$$.tmp"
    if cp "$src/graph.json" "$tmp" 2>/dev/null; then
      mv "$tmp" "$out/graph.json" 2>/dev/null || rm -f "$tmp"
    else
      rm -f "$tmp"
    fi
  fi

  if [ "$do_sem" = true ]; then
    # claude-cli defaults to Opus; pin the registry model (sonnet) for extraction.
    # Its subprocess timeout is a fixed 600s, so cap chunk size for this backend.
    budget_args=()
    if [ "$backend" = claude-cli ]; then
      export GRAPHIFY_CLAUDE_CLI_MODEL="$(gfx_cli_model)"
      budget_args=(--token-budget "$(gfx_cli_token_budget)")
    fi
    echo "[$name] semantic extract ($backend${GRAPHIFY_CLAUDE_CLI_MODEL:+/$GRAPHIFY_CLAUDE_CLI_MODEL}) on $idx_path"
    graphify extract "$dst" --backend "$backend" "${exc_args[@]}" "${budget_args[@]}"; built=$?
  else
    echo "[$name] AST update on $idx_path"
    graphify update "$dst"; built=$?
  fi

  # Check the carve against the graph just built, not against the proposal that
  # recommended it: how many edges cross the domain's own top-level subtrees. Only
  # when the build succeeded - a failed one leaves the seeded or stale graph in
  # place, and it parses, so a count off it reads as a fresh measurement.
  [ "$built" = 0 ] || { echo "[$name] build failed (rc=$built) - no carve check"; continue; }
  cross=$(gfx_cross_edges "$out/graph.json") && echo "[$name] cross-subtree edges: $cross"
done
```

## Notes

- `update` is AST-only and free; `extract` runs the LLM backend. `claude-cli` is
  serial - a large `--semantic` domain is slow and consumes plan quota.
- `extract` sends docs **and images** to the LLM as text; SVG markup and decoded
  binary bytes are token noise and the cause of chunks that time out, so semantic
  builds drop image/asset globs (`gfx_extract_excludes`; override per-repo with an
  `.extract_excludes` array in the registry, which replaces the default set).
- `claude-cli`'s per-chunk subprocess timeout is a hardcoded 600s (`--api-timeout`
  only affects HTTP backends), so semantic builds on it cap `--token-budget`
  (`gfx_cli_token_budget`, default 20000; override with `.cli_token_budget` in the
  registry).
- A semantic build seeded onto a worktree is reconciled by AST `update` on later
  plain syncs; the named/semantic layer goes stale until the next `--semantic`
  run. Re-run with `--semantic` when you need fresh community naming.
- Graphs are gitignored (`/graphify-wrapper-setup` step 2) so nothing here is
  committable.

## Report

One line per index - action taken (seed/refresh/scratch, AST/semantic), node+edge
counts from the build output, the `graph.json` path, and the
cross-subtree edge count the loop printed - in this shape:

    [api] refresh, AST - 812 nodes / 3104 edges - apps/api/graphify-out/graph.json - cross-subtree edges: 1

For a domain spanning several subtrees that count is the carve check
`/graphify-wrapper-map` deferred here: zero means the carve bought no
cross-subtree edges and a split would cost nothing - say so plainly. A domain
whose build failed has no count: report the failure, never a stale number.
