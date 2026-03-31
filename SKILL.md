---
name: openclaw-dual-agent
description: "Run two OpenClaw agents simultaneously — a paid Anthropic agent and a free OpenRouter agent, each with its own Telegram bot. Trigger phrases: multi-agent setup, add a second agent, free agent openclaw, run two agents, openrouter openclaw, parallel agents, cost optimization agent."
metadata: {"clawdbot": {"emoji": "🤖", "requires": {"bins": ["jq"]}, "env": ["ANTHROPIC_API_KEY", "OPENROUTER_API_KEY", "TELEGRAM_BOT_TOKEN"], "os": ["darwin", "linux", "win32"]}, "homepage": "https://clawhub.com/djc00p/openclaw-dual-agent"}
---

# Multi-Agent OpenClaw Setup

Run a paid Anthropic agent and free OpenRouter agent side by side with separate Telegram bots.

## Quick Start

1. **Create two Telegram bots** via @BotFather and extract chat IDs:
   ```bash
   curl https://api.telegram.org/bot{TOKEN}/getUpdates | jq '.result[0].message.chat.id'
   ```

2. **Authenticate agents:**
   ```bash
   openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
   # For free agent: create auth-profiles.json in its agentDir with OpenRouter key
   ```
   > ⚠️ Passing keys on the CLI can expose them in shell history. Run `history -d $(history 1)` after or use `openclaw onboard` interactively. Credential files (`auth-profiles.json`, `openclaw.json`) should be `chmod 600`.

3. **Configure** `openclaw.json` with two agents, separate bindings, and Telegram accounts.

4. **Verify setup:**
   ```bash
   openclaw doctor
   openclaw sessions cleanup \
     --store /Users/YOUR_USERNAME/.openclaw/agents/main/store \
     --enforce --fix-missing
   openclaw restart
   ```

## Key Concepts

- **Agent isolation:** Each agent has its own `agentDir`, workspace, and model config.
- **Binding routing:** `accountId` in bindings directs Telegram messages to the correct agent.
- **Model refs:** Use `provider/modelid` format (e.g., `anthropic/claude-sonnet-4-6`).
- **Per-agent auth:** OpenRouter requires `auth-profiles.json` in each agent's directory.

## Common Usage

**Adding a free agent:**
- Create agentDir at `/Users/YOUR_USERNAME/.openclaw/agents/free-agent/agent`
- Add agent entry to `openclaw.json` with `model.primary: "openrouter/..."`
- Create `auth-profiles.json` with OpenRouter API key in agent's directory
- Add binding with unique `accountId` (e.g., `"tg2"`)
- Restart: `openclaw restart`

**Switching models:**
Edit `openclaw.json` agent's `model.primary` and `fallbacks` with valid provider/id strings.

**Masking secrets for logs:**
```bash
cat ~/.openclaw/openclaw.json | \
  jq '.channels.telegram.accounts |= map_values(.botToken = "[REDACTED]")'
```

## References

- `references/config-reference.md` — Full openclaw.json, bindings, and auth-profiles.json examples
- `references/troubleshooting.md` — Common errors, fixes, and Node.js compatibility notes


