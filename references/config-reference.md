# Configuration Reference

Complete examples for multi-agent OpenClaw setup.

## openclaw.json — Agents List

Add both agents to the `agents` array:

```json
{
  "id": "main",
  "name": "main",
  "workspace": "/Users/YOUR_USERNAME/.openclaw/workspace",
  "agentDir": "/Users/YOUR_USERNAME/.openclaw/agents/main/agent",
  "model": {
    "primary": "anthropic/claude-sonnet-4-6"
  },
  "heartbeat": {
    "every": "55m"
  },
  "subagents": {
    "model": "anthropic/claude-haiku-4-5-20251001"
  }
},
{
  "id": "free-agent",
  "name": "Free Agent",
  "workspace": "/Users/YOUR_USERNAME/.openclaw/workspace-free",
  "agentDir": "/Users/YOUR_USERNAME/.openclaw/agents/free-agent/agent",
  "model": {
    "primary": "openrouter/nvidia/nemotron-3-super-120b-a12b:free",
    "fallbacks": [
      "openrouter/free",
      "openrouter/nvidia/nemotron-3-nano-30b-a3b:free"
    ]
  }
}
```

## openclaw.json — Bindings

Configure routing of Telegram messages to agents via unique `accountId`:

```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "telegram",
        "accountId": "default"
      }
    },
    {
      "agentId": "free-agent",
      "match": {
        "channel": "telegram",
        "accountId": "tg2"
      }
    }
  ]
}
```

## openclaw.json — Telegram Accounts

Add both bot tokens under `channels.telegram.accounts`:

```json
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "botToken": "YOUR_MAIN_BOT_TOKEN",
          "allowFrom": [YOUR_CHAT_ID]
        },
        "tg2": {
          "botToken": "YOUR_FREE_AGENT_BOT_TOKEN",
          "allowFrom": [YOUR_CHAT_ID]
        }
      }
    }
  }
}
```

## auth-profiles.json — OpenRouter Authentication

Place in the free agent's agentDir: `/Users/YOUR_USERNAME/.openclaw/agents/free-agent/agent/auth-profiles.json`

```json
{
  "version": 1,
  "profiles": {
    "openrouter:default": {
      "type": "api_key",
      "provider": "openrouter",
      "key": "sk-or-v1-YOUR_OPENROUTER_KEY"
    }
  },
  "lastGood": {
    "openrouter": "openrouter:default"
  }
}
```

## models.json — OpenRouter Provider (Free Agent)

Place at `/Users/YOUR_USERNAME/.openclaw/agents/free-agent/agent/models.json`:

```json
{
  "providers": {
    "openrouter": {
      "baseUrl": "https://openrouter.ai/api/v1",
      "api": "openai-completions",
      "apiKey": "openrouter",
      "models": [
        {
          "id": "nvidia/nemotron-3-super-120b-a12b:free",
          "name": "Nemotron Super 120B",
          "reasoning": false,
          "input": ["text"],
          "cost": {
            "input": 0,
            "output": 0,
            "cacheRead": 0,
            "cacheWrite": 0
          },
          "contextWindow": 131072,
          "maxTokens": 8192
        },
        {
          "id": "nvidia/nemotron-3-nano-30b-a3b:free",
          "name": "Nemotron Nano 30B",
          "reasoning": false,
          "input": ["text"],
          "cost": {
            "input": 0,
            "output": 0,
            "cacheRead": 0,
            "cacheWrite": 0
          },
          "contextWindow": 131072,
          "maxTokens": 8192
        }
      ]
    }
  }
}
```

> Do NOT add an `anthropic` provider block here — OpenClaw registers Anthropic internally. Adding it manually causes `No API provider registered for api: anthropic`.

## Getting Chat IDs

Extract your Telegram chat ID for allowFrom:

```bash
curl https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates | \
  jq '.result[0].message.chat.id'
```

Send a message to your bot, then run the command above and extract the `id` field.

## Masking Secrets for Logs

Before sharing configs, redact sensitive tokens:

```bash
cat ~/.openclaw/openclaw.json | \
  jq '.channels.telegram.accounts |= map_values(.botToken = "[REDACTED]")'
```

This masks all bot tokens in the accounts section while preserving structure for debugging.
