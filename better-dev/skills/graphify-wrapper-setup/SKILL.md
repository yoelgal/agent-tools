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

The PyPI package is `graphifyy` (double-y); the binary is `graphify`. Install it
pinned in both axes: below the 0.9.18 floor graph writes are non-atomic (the
refresh hook's own floor is far older - `built_at_commit`, 0.7.0), and an unpinned
index lets `UV_INDEX` / `UV_DEFAULT_INDEX` / `UV_INDEX_URL` decide whose build
backend runs under the operator's account here.

```bash
floor=0.9.18; idx=(--default-index https://pypi.org/simple)
gfxver() { graphify --version 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -1; }
gfxlow() { [ -z "$1" ] || [ "$(printf '%s\n%s\n' "$floor" "$1" | sort -V | head -1)" != "$floor" ]; }
if command -v graphify >/dev/null; then echo "graphifyy: already present"
else uv tool install "graphifyy>=$floor" "${idx[@]}" && echo "graphifyy: INSTALLED"; fi
have=$(gfxver); echo "graphify ${have:-unknown}"
if gfxlow "$have"; then
  uv tool upgrade graphifyy "${idx[@]}" || true; now=$(gfxver)
  [ "$now" = "$have" ] || echo "graphifyy: UPGRADED from $have to $now"
  if gfxlow "$now"; then echo "graphifyy: BELOW FLOOR $floor (still ${now:-unknown})"; fi
fi
```

`uv tool upgrade` exits 0 on a no-op, so the announce is the observed version
change, never the exit code. `BELOW FLOOR` means the upgrade did not clear it (an
exact `==` pin does that): stop, hand the operator `uv tool install
'graphifyy@latest'`, and do not run steps 2-4. No `uv` at all: stop and say `brew
install uv` - never a global `pip install`.

## 2. Never-commit guard (global gitignore)

Graphs live in-tree at `<path>/graphify-out/`. Keep them out of every repo via the
gitignore git already reads - never by re-pointing `core.excludesFile`, which
**replaces** git's default resolution, so re-pointing a value any scope already set
deactivates every rule it carried (`.env`, `*.pem`) in every repo on the machine.
Writing the key at all does the same, freezing a path git re-resolves per
invocation to whatever this run's `HOME`/`XDG_CONFIG_HOME` said - and an agent's
environment is not the operator's shell.

```bash
. .better-dev/bin/bd-gfx 2>/dev/null || . "${CLAUDE_PLUGIN_ROOT}/scripts/bd-gfx"
gfx_ignore_guard   # every scope + probe + refusal live here, under selftest
```

A non-absolute value is refused (it would land inside this repo) and a failed
append prints `FAILED`. Either way the never-commit guard is **not** in place:
name the file the operator has to add `graphify-out/` to, rather than reporting a
write that did not land.

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
on the operator's plan - **no API key**), pinned to `cli_model: "sonnet"` because
that path otherwise defaults to Opus, overkill for structured-JSON extraction.
Prefer an API key already in the env:

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
rather than silent (D26). Steps 1-3 each printed which branch they took: report a
bullet for every line below that fired, with its undo, and none for the rest - a
run that printed none of them changed nothing outside the repo and says that.

- `graphifyy: INSTALLED` - installed `graphifyy` (undo: `uv tool uninstall graphifyy`)
- `graphifyy: UPGRADED from <v> to <w>` - replaced the machine's `graphifyy` (undo:
  `uv tool install 'graphifyy==<v>'`, an exact pin; `graphifyy@latest` unpins)
- `appended graphify-out/: yes` - added `graphify-out/` to the gitignore step 2
  printed (undo: drop that line, and delete the file and its parent dir if this run
  made them). `FAILED`, or a refusal line, means no guard: say so and name the file.
- `set core.excludesfile: yes` - pointed git's global excludes at that file, which
  git did not otherwise reach (undo: `git config --global --unset core.excludesfile`)
- `registry: CREATED <dir>` - created this repo's registry home under
  `~/.claude/graphify/` (undo: `rm -rf` that directory)

Then hand the operator the next verbs: `/graphify-wrapper-index <name> <path>` to
register a domain (no args = I analyze the repo and suggest them),
`/graphify-wrapper-sync` to build this worktree's indexes, and
`/graphify-wrapper-query <name> "<question>"` to ask something now. Do **not**
build any index here - that is `-index` + `-sync`.
