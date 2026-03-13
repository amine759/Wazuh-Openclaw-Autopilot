---
name: project_env_vars
description: Required environment variables for both submodules and where to obtain them
type: project
---

## Wazuh-MCP-Server (.env)
- WAZUH_HOST, WAZUH_USER, WAZUH_PASS, WAZUH_PORT (55000) - Wazuh Manager credentials
- MCP_HOST (0.0.0.0 in Docker), MCP_PORT (3000)
- AUTH_MODE (bearer/oauth/none), AUTH_SECRET_KEY (openssl rand -hex 32)
- WAZUH_VERIFY_SSL (false for dev with self-signed certs)
- Optional: WAZUH_INDEXER_* for vuln tools on Wazuh 4.8+, REDIS_URL for multi-instance

## Wazuh-Openclaw-Autopilot (.env)
- MCP_URL (https://localhost:3000), AUTOPILOT_MCP_AUTH (token from MCP server)
- OPENCLAW_TOKEN, OPENCLAW_WEBHOOK_TOKEN (gateway auth)
- OPENCLAW_GATEWAY_URL (http://127.0.0.1:18789)
- AUTOPILOT_RUNTIME_URL (http://127.0.0.1:9090)
- At least one LLM API key: ANTHROPIC_API_KEY, OPENAI_API_KEY, GROQ_API_KEY, or OPENROUTER_API_KEY
- Optional: SLACK_APP_TOKEN, SLACK_BOT_TOKEN for Slack integration
- Optional: ABUSEIPDB_API_KEY for IP enrichment

**Why:** No .env files exist yet (only .env.example). User must create them before any service can start.

**How to apply:** Copy .env.example to .env in each submodule and fill in values. The MCP server's AUTH_SECRET_KEY should be generated, and the same token must be shared as AUTOPILOT_MCP_AUTH in the autopilot config.
