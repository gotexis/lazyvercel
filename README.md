<div align="center">

# avercel

**Agent-Vercel** — The Vercel CLI that fights back.

[![npm version](https://img.shields.io/npm/v/avercel)](https://www.npmjs.com/package/avercel)
[![npm downloads](https://img.shields.io/npm/dm/avercel)](https://www.npmjs.com/package/avercel)
[![CI](https://github.com/exisz/avercel/actions/workflows/ci.yml/badge.svg)](https://github.com/exisz/avercel/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/exisz/avercel)](https://github.com/exisz/avercel/stargazers)

Stop losing hours to trailing newlines. Stop agents from running `vercel deploy`.

[Homepage](https://exisz.github.io/avercel/) · [npm](https://www.npmjs.com/package/avercel) · [Issues](https://github.com/exisz/avercel/issues)

</div>

---

<details>
<summary><strong>Table of Contents</strong></summary>

- [The Problem](#the-problem)
- [Quick Start](#quick-start)
- [Features](#features)
- [Usage](#usage)
- [Takeover Mode](#takeover-mode)
- [Configuration](#configuration)
- [How It Works](#how-it-works)
- [Why Not Just...](#why-not-just)
- [AgentSkill](#agentskill)
- [Star History](#star-history)
- [Contributing](#contributing)
- [License](#license)
- [Support](#support)

</details>

---

## The Problem

Ever spent 45 minutes debugging why your app works locally but breaks on Vercel, only to discover a **trailing newline** in an environment variable?

```bash
# This looks fine...
echo "sk-abc123" | vercel env add SECRET_KEY production

# But echo adds a trailing \n, and now your API key is "sk-abc123\n"
# Your app breaks. Vercel shows no error. You question your career choices.
```

Or maybe you've accidentally run `vercel deploy` and bypassed your entire CI/CD pipeline. Or your team keeps using `preview` when the project convention is `dev`.

**avercel** wraps the official Vercel CLI and fixes all of this — silently, automatically, no behavior change needed.

## Quick Start

```bash
npm i -g avercel
```

That's it. Both `avercel` and `vercel` now go through avercel's guardrails. The real Vercel CLI is bundled inside — no separate install needed.

## Features

| Feature | What it does |
|---|---|
| 🧹 **Patched `env add`** | Strips trailing whitespace/newlines from piped stdin — automatically |
| 🔍 **`env check`** | Audits all your env vars for trailing whitespace (the silent killer) |
| 🚫 **Disabled commands** | Block dangerous commands like `deploy` with custom error messages |
| 🛑 **Blocked environments** | Prevent wrong env names (`preview` vs `dev`) with guidance |
| 🔀 **Full passthrough** | Everything else forwards to `vercel` exactly as-is — same stdin/stdout/stderr/exit code |
| 🤖 **AgentSkill built-in** | AI coding agents get avercel's guardrails automatically |

## Usage

Use `avercel` exactly like you'd use `vercel`:

```bash
# These just work — forwarded to vercel as-is
avercel dev
avercel ls
avercel domains ls

# This is the magic — trailing newline stripped automatically
echo "sk-abc123" | avercel env add SECRET_KEY production
# → "avercel: stripped 1 trailing whitespace/newline character(s) from piped input"

# Audit existing env vars
avercel env check
# → ⚠️  Found 2 env var(s) with trailing whitespace/newlines:
#    Variable        Targets              Problem
#    ──────────────────────────────────────────────────
#    DATABASE_URL    production, preview   trailing newline (\n)
#    SECRET_KEY      production            trailing whitespace

# Blocked environment names show a helpful error
avercel env pull preview
# → ❌ This project uses 'dev' not 'preview'. Use: avercel env pull dev

# Blocked commands show your custom message
avercel deploy
# → ❌ Do not use `vercel deploy`. Push to GitHub and let the integration handle it.
```

## Takeover Mode

Want `avercel` to completely replace `vercel`?

```bash
# Install avercel globally — it registers both `avercel` AND `vercel` commands
npm i -g avercel

# That's it! All `vercel` commands now go through avercel's guardrails.

# If you had vercel installed separately, clean it up:
avercel takeover --force
```

After takeover:

```bash
vercel ls          # → goes through avercel → forwarded to real vercel internally
vercel deploy      # → blocked by your config → "❌ Use git push instead"
echo "val" | vercel env add  # → trailing newline stripped automatically
```

## Configuration

Create `.avercel/avercel.yaml` in your project root (or `~/.avercel/avercel.yaml` for global config). Project config overrides global.

```yaml
# Block commands with custom messages
disabled:
  deploy: "❌ Do not use `vercel deploy`. Push to GitHub — Vercel deploys on git push."
  build: "❌ Do not use `vercel build`. Vercel builds automatically on deploy."
  "env rm": "❌ Don't remove env vars directly."

# Block specific environment names
blocked_envs:
  preview: "❌ This project uses 'dev' not 'preview'. Use: avercel env pull dev"
  staging: "❌ No staging environment. Use 'production' or 'dev'."
```

View active config:

```bash
avercel config
```

### `disabled`

Block entire commands. When a user runs a disabled command, they see your custom error message and the command exits without forwarding to vercel.

### `blocked_envs`

Block specific environment names in `env pull`, `env add`, `env ls`, and `env rm` commands. This is **not** silent replacement — it's a loud error with guidance.

## `env check`

Audits all environment variables in your Vercel project for trailing whitespace and newlines.

```bash
# Uses .vercel/project.json for project ID (run `vercel link` first)
avercel env check

# Or specify explicitly
avercel env check --project prj_abc123 --token tkn_xyz
```

Token sources (in order):
1. `--token` flag
2. `VERCEL_TOKEN` environment variable
3. `~/.vercel_token` file

## How It Works

```
┌─────────────────┐
│     avercel      │
│                  │
│  1. Load config  │
│  2. Check if     │
│     disabled     │
│  3. Check        │
│     blocked_envs │
│  4. Patch or     │
│     passthrough  │
└────────┬─────────┘
         │
         ▼
┌─────────────────┐
│   vercel CLI     │
│  (bundled dep)   │
└─────────────────┘
```

- **Passthrough**: Resolves the bundled vercel binary from `node_modules` and spawns it — zero overhead, same experience
- **Patched `env add`**: Only intercepts stdin when piped, strips trailing whitespace, forwards to vercel
- **`env check`**: Calls Vercel API directly, never touches the CLI
- **Takeover**: `avercel takeover --force` removes standalone vercel and lets avercel own both bin names

## Why Not Just...

| Alternative | Problem |
|---|---|
| Be careful with `echo` | You'll forget. Everyone forgets. |
| Use `printf` | Not all tools use printf. And you'll still forget. |
| Check env vars manually | There's no `vercel env check` command |
| Alias `deploy` in your shell | Doesn't help your teammates |
| Use a `.env` file | Doesn't solve the Vercel-side issue |

## AgentSkill

avercel ships with a built-in [AgentSkill](skill/SKILL.md) for AI coding agents.

Any agent that supports the [AgentSkill format](https://github.com/anthropics/anthropic-cookbook/blob/main/misc/prompt_caching_and_tool_use.ipynb) (Claude Code, Cursor, OpenClaw, etc.) can pick up avercel's guardrails automatically — blocked commands, env patching, and environment restrictions are enforced without the agent needing any special configuration.

```
# The skill file lives at:
skill/SKILL.md
```

## Star History

<div align="center">
  <a href="https://star-history.com/#exisz/avercel&Date">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=exisz/avercel&type=Date&theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=exisz/avercel&type=Date" />
      <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=exisz/avercel&type=Date" width="600" />
    </picture>
  </a>
</div>

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md).

<a href="https://github.com/exisz/avercel/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=exisz/avercel" />
</a>

## License

MIT — see [LICENSE](LICENSE).

## Support

If avercel saved you from a trailing newline nightmare, consider giving it a ⭐ on GitHub — it helps others discover the project.

[⭐ Star on GitHub](https://github.com/exisz/avercel)
<!-- pipeline migrated to single-workflow per cicd-no-bot-tag-trigger -->
