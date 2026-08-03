---
name: graphify-wrapper-setup
description: Use when a repo or machine needs graphify-wrapper wired up for the first time - installing the graphify CLI, adding the never-commit guard, seeding the per-repo registry, and picking a semantic backend - the one-time "set up graphify" step before any mapping or indexing.
allowed-tools:
  - Bash
  - Read
---

# /graphify-wrapper-setup

Make a repo ready for graphify-wrapper. Idempotent - safe to re-run.

```bash
. .better-dev/bin/bd-gfx 2>/dev/null || . "${CLAUDE_PLUGIN_ROOT}/scripts/bd-gfx"
```

## 1. Ensure the CLI

The PyPI package is `graphifyy` (double-y); the binary is `graphify`. Pin the
version: below the 0.9.18 floor graph writes are non-atomic (the refresh hook's
own floor is far older - `built_at_commit`, 0.7.0). `--default-index` is
best-effort and bounds nothing: it sets uv's **lowest-priority** index, so it
turns back a `UV_DEFAULT_INDEX` / `UV_INDEX_URL` redirect of the base index and
nothing more. `UV_INDEX` and `UV_EXTRA_INDEX_URL` add indexes uv searches first
and they still decide whose build backend runs here (measured, uv 0.11.7).

```bash
floor=0.9.18; idx=(--default-index https://pypi.org/simple)
gfxver() { graphify --version 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -1; }
gfxlow() { [ -z "$1" ] || [ "$(printf '%s\n%s\n' "$floor" "$1" | sort -V | head -1)" != "$floor" ]; }
if command -v graphify >/dev/null; then echo "graphifyy: already present"
else uv tool install "graphifyy>=$floor" "${idx[@]}" && echo "graphifyy: INSTALLED"; fi
have=$(gfxver); echo "graphify ${have:-unknown}"
if ! command -v graphify >/dev/null; then echo "graphifyy: NOT ON PATH"
elif gfxlow "$have"; then
  uv tool upgrade graphifyy "${idx[@]}" || true; now=$(gfxver)
  [ "$now" = "$have" ] || echo "graphifyy: UPGRADED from $have to $now"
  if gfxlow "$now"; then echo "graphifyy: BELOW FLOOR $floor (still ${now:-unknown})"; fi
fi
```

`uv tool upgrade` exits 0 on a no-op, so the announce is the observed version
change, never the exit code. Two stops, and both skip steps 2-4. `BELOW FLOOR`
means the upgrade did not clear it (an exact `==` pin does that): hand the
operator `uv tool install 'graphifyy@latest'`. `NOT ON PATH` means the version was
never read, because the binary uv installed is not resolvable here - a re-install
cannot fix that, so hand `uv tool update-shell` instead (uv warns about this on a
fresh machine). No `uv` at all: stop and say `brew install uv` - never a global
`pip install`.

## 2. Never-commit guard (global gitignore)

Graphs live in-tree at `<path>/graphify-out/`. Keep them out of every repo via
the operator's **global** gitignore so a host repo's tracked files are never
touched and nothing can be pushed.

```bash
gi=$(git config --global --get core.excludesfile || echo "$HOME/.config/git/ignore")
mkdir -p "$(dirname "$gi")"
git config --global core.excludesfile "$gi"
grep -qxF 'graphify-out/' "$gi" 2>/dev/null || printf 'graphify-out/\n' >> "$gi"
echo "global gitignore: $gi"
```

## 3. Init the per-repo registry

The central home holds **only** the registry (graphs stay in-tree). Keyed by git
identity, so every worktree of this repo shares it.

```bash
home=$(gfx_home); reg=$(gfx_registry)
mkdir -p "$home"
if [ ! -f "$reg" ]; then
  jq -n \
    --arg key "$(gfx_repo_key)" \
    --arg main "$(gfx_main_worktree)" \
    --arg branch "$(gfx_default_branch)" \
    '{repo_key:$key, main_worktree:$main, default_branch:$branch, backend:"claude-cli", cli_model:"sonnet", indexes:{}}' \
    > "$reg" && echo "registry: CREATED $home"
fi
cat "$reg"
```

## 4. Pick a semantic backend

`--semantic` builds need a backend. Default is `claude-cli` (the local `claude` CLI
on the operator's plan - **no API key**, billed to the plan), pinned in the registry
to `cli_model: "sonnet"` because that path otherwise defaults to Opus, overkill for
structured-JSON extraction; `/graphify-wrapper-sync --semantic` exports that pin as
`GRAPHIFY_CLAUDE_CLI_MODEL`. Prefer an API key already in the env:

```bash
if   [ -n "${ANTHROPIC_API_KEY:-}" ]; then b=claude
elif [ -n "${GEMINI_API_KEY:-}${GOOGLE_API_KEY:-}" ]; then b=gemini
elif [ -n "${OPENAI_API_KEY:-}" ]; then b=openai
elif [ -n "${DEEPSEEK_API_KEY:-}" ]; then b=deepseek
else b=claude-cli; fi
reg=$(gfx_registry); tmp=$(mktemp "$(dirname "$reg")/.reg.XXXXXX")
jq --arg b "$b" '.backend=$b' "$reg" > "$tmp" && mv "$tmp" "$reg"
echo "semantic backend: $b"
```

## 5. Report

Name what this run changed outside the repo, so a machine-global write is visible
rather than silent (D26). Steps 1 and 3 each printed which branch they took: report
a bullet for every line below that fired, with its undo, and none for the rest.
Step 2's bullet fires on every run that reaches step 2, because that write is
unconditional once there - but both step-1 stops skip it, and a stopped run
reports no bullet for a write it never made.

- `global gitignore: <file>` - pointed git's **global** `core.excludesfile` at that
  file and added `graphify-out/` to it, so that path is ignored in every repo on
  this machine (undo: remove the added line, plus
  `git config --global --unset core.excludesfile` where setup set the pointer -
  removing the line alone leaves the pointer moved). A re-run on an
  already-guarded machine is a no-op, but this run cannot tell which it was, so
  report the write.
- `graphifyy: INSTALLED` - installed `graphifyy` (undo: `uv tool uninstall graphifyy`)
- `graphifyy: UPGRADED from <v> to <w>` - replaced the machine's `graphifyy` (undo:
  `uv tool install 'graphifyy==<v>'`, an exact pin; `graphifyy@latest` unpins)
- `registry: CREATED <dir>` - created this repo's registry home under
  `~/.claude/graphify/` (undo: `rm -rf` that directory)

Then hand the operator the next verbs: `/graphify-wrapper-index <name> <path>` to
register a domain (no args = I analyze the repo and suggest them),
`/graphify-wrapper-sync` to build this worktree's indexes, and
`/graphify-wrapper-query <name> "<question>"` to ask something now. Do **not**
build any index here - that is `-index` + `-sync`.
