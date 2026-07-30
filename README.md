<h1 align="center">agent-tools</h1>

<p align="center">
  <strong>Agent skills and plugins by Yoel Gal</strong> - tools that run <em>inside</em> the coding agent you already use.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT" />
</p>

---

This is a monorepo. Each top-level directory is one independently installable tool; the repo doubles
as a Claude Code plugin marketplace (`.claude-plugin/marketplace.json`).

| Tool | What it is |
|---|---|
| [`better-dev/`](better-dev/) | Portable dev practices as skills - onboard a repo, plan or diagnose in worktrees, drive an autonomous loop to proven-done, review, source tools, and self-extend. Agent-agnostic: Claude Code, Codex, pi, hermes. |

## Install

**As a Claude Code plugin** - add this repo as a plugin marketplace, then install the tool you want:

```
/plugin marketplace add yoelgal/agent-tools
/plugin install better-dev@agent-tools
```

**Any host (clone path)** - each tool ships its own installer; see its README. For better-dev:

```sh
git clone https://github.com/yoelgal/agent-tools ~/agent-tools
~/agent-tools/better-dev/install.sh
```

## Versioning

The repo promotes `staging` to `main` as a whole; each tool carries its own manifest version and
release ledger (better-dev's is `better-dev/docs/RELEASES.md`). The repo itself has no global version.

## License

MIT for everything authored here. Vendored cuts carry their upstream license alongside their code
(`LICENSE-upstream` and `UPSTREAM` provenance files inside the tool that vendors them).
