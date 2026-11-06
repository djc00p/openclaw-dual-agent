# 🤖 OpenClaw Dual-Agent

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/openclaw-dual-agent) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![OpenClaw](https://img.shields.io/badge/Openclaw-Dual_Agent-red)](#) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

The **OpenClaw Dual-Agent Setup** is a specialized deployment pattern for users who want to maximize AI utility while minimizing operational costs. By running a "Paid Agent" (e.g., Anthropic Claude Sonnet) alongside a "Free/Cheap Agent" (e.g., OpenRouter-powered models), you can delegate high-reason_reasoning tasks to premium models and routine,-low-cost tasks to lightweight models—all managed through separate, dedicated Telegram interfaces.

---

## 🏗️ Architecture: Parallel Agent Isolation

This architecture relies on strict **Agent Isolation**. Unlike standard single-agent setups, each agent in a dual-agent configuration operates within its own siloed environment.

| Component | **Premium Agent (Anth_Agent)** | **Economy Agent (Free_Agent)** |
| :--- | :--- | :--- |
| **Provider** | Anthropic (Claude 3.5/4) | OpenRouter (Llama, Mistral, etc.) |
| **Directory** | `~/.openclaw/agents/premium-agent/` | `~/.openclaw/agents/free-agent/` |
| **Auth Method** | `openclaw onboard` (Global) | `auth-profiles.json` (Local to Agent) |
| **Interface**
| **Telegram Account ID** | `tg_premium_01` | `tg_free_01` |
| **Primary Task** | Complex Reasoning, Coding, Strategy | Tasks, Lookups, Monitoring, Summarization |

---

## 🚀 Deployment Workflow

### 1. Telegram Bot Provisioning

You must create two distinct bots via [@BotFather](https_t.me/BotFather) to ensure that messages are routed to the correct agent.

**Extract your Chat ID:**
Use the following command to retrieve the unique numeric ID for your new bot:

```bash
curl https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates | jq '.result[0].message.chat.id'
```

### 2. Authentication & Credential Management

**Security Warning:** Never pass API keys via the command line as they persist in your shell history. Use the interactive onboarding or secure environment files.

* **Premium Agent:** Run the interactive onboarder:

    ```bash
    openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
    ```

* **Economy Agent:** Navigate to the specific agent directory and create a local `auth-profiles.json`:

    ```bash
    cd ~/.openclaw/agents/free-agent/agent
    echo '{"openrouter": "your_openrouter_key"}' > auth-profiles.json
    chmod 600 auth-profiles.json
    ```

### 3. Global Configuration (`openclaw.json`)

The `openclaw.json` file acts as the "Router." You must define both agents and their respective Telegram bindings.

```json
{
    "agents": {
        "list": [
            {
                "agentDir": "~/.openclaw/agents/main/agent",
                "heartbeat": {
                    "every": "55m",
                    "model": "anthropic/claude-haiku-4-5-20251001"
                },
                "id": "main",
                "model": {
                    "primary": "anthropic/claude-sonnet-4-6"
                },
                "name": "main",
                "subagents": {
                    "model": "anthropic/claude-haiku-4-5-20251001"
                },
                "tools": {
                    "alsoAllow": [
                        "ollama_web_fetch",
                        "ollama_web_search"
                    ]
                },
                "workspace": "~/.openclaw/workspace"
            },
            {
                "agentDir": "~/.openclaw/agents/free-agent/agent",
                "heartbeat": {
                    "every": "1h",
                    "model": "openrouter/google/gemini-flash-1.5"
                },
                "id": "free-agent",
                "model": {
                    "primary": "openrouter/google/gemini-flash-1.5",
                    "fallbacks": [
                        "openrouter/free"
                    ]
                },
                "name": "Free",
                "workspace": "~/.openclaw/workspace-free-agent"
            }
        ]
    },
    "auth": {
        "profiles": {
        "anthropic": {
            "mode": "api_key",
            "provider": "anthropic"
        },
        "anthropic:default": {
            "mode": "api_key",
            "provider": "anthropic"
        },
        "openrouter": {
            "mode": "api_key",
            "provider": "openrouter"
        }
        }
    },
    "bindings": [
        {
            "agentId": "main",
            "match": {
                "accountId": "default",
                "channel": "telegram"
            }
        },
        {
            "agentId": "free-agent",
            "match": {
                "accountId": "telegram2",
                "channel": "telegram"
            }
        }
    ],
    "channels": {
        "telegram": {
            "accounts": {
                "default": {
                    "allowFrom": [
                        "TOKEN"
                    ],
                    "botToken": "TOKEN",
                    "dmPolicy": "pairing",
                    "groupPolicy": "allowlist"
                    },
                    "telegram2": {
                    "allowFrom": [
                        "TOKEN"
                    ],
                    "botToken": "TOKEN",
                    "dmPolicy": "pairing",
                    "enabled": true,
                    "groupPolicy": "allowlist",
                    "groups": {
                        "*": {
                        "requireMention": true
                        }
                    }
                }
            },
            "dmPolicy": "pairing",
            "enabled": true,
            "groupPolicy": "allowlist",
            "groups": {
                "*": {
                "requireMention": true
                }
            },
            "streaming": {
                "mode": "partial"
            }
        }
    }
}
```

---

## 🛠️ Operational Commands

### Verification & Health Checks

Before assuming your dual-agent setup is functional, run the **Doctor** and **Cleanup** protocols to ensure the session stores are synchronized.

```bash
# Check for configuration errors and path mismatches
openclaw doctor

# Sync session stores and repair missing paths
openclaw sessions cleanup \
  --store ~/.openclaw/agents/premium-agent/store \
  --enforce --fix-missing

# Apply all changes
openclaw restart
```

### Security: Sanitizing Logs

When sharing logs or debugging, **never** expose your `botToken` or `apiKey`. Use `jq` to redact sensitive strings from your configuration file before sharing.

```bash
# Redact all Telegram Bot Tokens in the config
cat ~/.openclaw/open_claw.json | jq '.channels.telegram.accounts |= map_values(.botToken = "[REDACTED]")'
```

---

## 💡 Best Practices

* **The "Cost-Switch" Strategy:** Use the Free Agent for all `cron` jobs, `web_scrapes`, and `summarizations`. Reserve the Premium Agent for `code_reviews`, `architecture_design`, and `complex_debugging`.
* **Filesystem Permissions:** Always `chmod 600` on any file containing API keys or `auth-profiles.json`.
* **Context Separation:** Remember that each agent has its own `agentDir`. If you need the Free Agent to see a file, it must be accessible within its specific directory or shared via a workspace mount.

---
**Standardized by the OpenClaw Engineering Team.**
