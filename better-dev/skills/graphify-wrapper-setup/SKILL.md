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

The PyPI package is `graphifyy` (double-y); the binary is `graphify`. Below the
0.9.18 floor the refresh hook is dead (no built_at_commit provenance, added in
0.7.0) and graph writes are non-atomic, so setup upgrades a stale install first.

```bash
if command -v graphify >/dev/null; then echo "graphifyy: already present"
else uv tool install graphifyy && echo "graphifyy: INSTALLED"; fi
graphify --version

floor=0.9.18
have=$(graphify --version 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -1)
if [ -n "$have" ] && [ "$(printf '%s\n%s\n' "$floor" "$have" | sort -V | head -1)" != "$floor" ]; then
  uv tool upgrade graphifyy && echo "graphifyy: UPGRADED from $have"
  graphify --version
fi
```

If `uv` is missing, stop and tell the operator to install it (`brew install uv`);
do not fall back to a global `pip install`.

## 2. Never-commit guard (global gitignore)

Graphs live in-tree at `<path>/graphify-out/`. Keep them out of every repo via
the operator's **global** gitignore so a host repo's tracked files are never
touched and nothing can be pushed.

`core.excludesFile` **replaces** git's default resolution, so re-pointing one the
operator already set deactivates every rule it carried (`.env`, `*.pem`) in every
repo on the machine. Append to their file; set the key only where git had none.

```bash
gi=$(git config --global --get core.excludesfile || true)
if [ -n "$gi" ]; then set_key=no
else gi="${XDG_CONFIG_HOME:-$HOME/.config}/git/ignore"; set_key=yes; fi   # git's own default
gi=${gi/#\~\//$HOME/}   # git expands a leading ~ in this value; the shell here does not
mkdir -p "$(dirname "$gi")"
if grep -qxF 'graphify-out/' "$gi" 2>/dev/null; then appended=no
else printf 'graphify-out/\n' >> "$gi"; appended=yes; fi
[ "$set_key" = yes ] && git config --global core.excludesfile "$gi"
echo "global gitignore: $gi (appended graphify-out/: $appended; set core.excludesfile: $set_key)"
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
    > "$reg"
fi
cat "$reg"
```

## 4. Pick a semantic backend

`--semantic` builds need a backend. Default is `claude-cli` (routes through the
local `claude` CLI on the operator's Pro/Max plan - **no API key**, billed to the
plan), pinned to `cli_model: "sonnet"` in the registry because that path otherwise
defaults to Opus, overkill for structured-JSON extraction. If an API key is
already in the env, prefer it:

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
rather than silent (D26). Steps 1-2 each printed which branch they took: report a
bullet for every line below that fired, with its undo, and none for the rest - a
run that printed none of them changed nothing outside the repo and says that.

- `graphifyy: INSTALLED` - installed `graphifyy` (undo: `uv tool uninstall graphifyy`)
- `graphifyy: UPGRADED from <v>` - replaced the machine's `graphifyy` (undo:
  `uv tool install 'graphifyy==<v>'`)
- `appended graphify-out/: yes` - added `graphify-out/` to the global gitignore
  step 2 printed (undo: drop that line)
- `set core.excludesfile: yes` - pointed git's global excludes at that file, a key
  that had none (undo: `git config --global --unset core.excludesfile`)

Then hand the operator the next verbs:

- `/graphify-wrapper-index <name> <path>` to register a domain (or
  `/graphify-wrapper-index` with no args to have me analyze the repo and suggest
  domains).
- `/graphify-wrapper-sync` to build/refresh the current worktree's indexes.
- `/graphify-wrapper-query <name> "<question>"` to ask something now.

Do **not** build any index here - that is `/graphify-wrapper-index` +
`/graphify-wrapper-sync`.
