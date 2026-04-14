---
name: avercel
description: Use avercel (Agent-Vercel) CLI instead of raw vercel CLI. Wraps vercel with guardrails, env sanitization, and workflow enforcement.
---

# avercel — Agent-Vercel CLI

Use `avercel` (or `vercel` after takeover) instead of raw `vercel` for all Vercel operations.

## Why

1. **`echo "val" | vercel env add` silently corrupts env vars** — avercel auto-strips trailing newlines
2. **`vercel deploy` bypasses Git integration** — avercel can block dangerous commands with custom messages
3. **Environment confusion** — avercel can block environment names your project doesn't use

## Installation

```bash
npm install -g avercel
```

After install, both `avercel` and `vercel` commands point to avercel.

## Usage

Use `avercel` exactly like `vercel`. All commands pass through unless patched or blocked.

```bash
# Safe env var setting (trailing newlines stripped automatically)
echo "my-secret" | avercel env add SECRET production --token $(cat ~/.vercel_token) --scope team

# Audit env vars for trailing whitespace
avercel env check

# All standard vercel commands work
avercel ls
avercel domains ls
avercel env pull production
```

## Config: .avercel/avercel.yaml

Create `.avercel/avercel.yaml` in the project root or `~/.avercel/avercel.yaml` globally:

```yaml
disabled:
  deploy: "❌ Do not use `vercel deploy`. Push to GitHub and let the integration handle it."
  build: "❌ Do not use `vercel build`. Vercel builds on deploy automatically."

blocked_envs:
  preview: "❌ This project uses 'dev' not 'preview'. Use: avercel env pull dev"
```

## Commands

| Command | Description |
|---------|-------------|
| `avercel env add` | Patched: strips trailing whitespace from piped stdin |
| `avercel env check` | Audit all env vars for trailing whitespace/newlines |
| `avercel config` | Show active configuration |
| `avercel takeover --force` | Remove standalone vercel, register avercel as `vercel` |
| Everything else | Passes through to real vercel CLI |

## Key Rules

- NEVER use raw `vercel` if avercel is available — it prevents env var corruption
- NEVER use `npx vercel` — this bypasses avercel's guardrails entirely. Always use `avercel` or `vercel` (after takeover)
- NEVER use `vercel deploy` or `vercel build` — push to Git instead
- Always check `avercel env check` after bulk env updates
