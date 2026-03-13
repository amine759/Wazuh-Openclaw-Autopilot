---
name: project_architecture
description: Service architecture, ports, and dependencies for the wazuh-openclaw-autopilot orchestration project
type: project
---

The project has 4 services that must run together:

1. **Wazuh Manager** (external dependency, port 55000) - the SIEM platform, must be pre-existing
2. **Wazuh MCP Server** (submodule: Wazuh-MCP-Server/, port 3000) - Python 3.11+, bridges AI to Wazuh API via MCP/SSE
3. **OpenClaw Gateway** (external dependency, port 18789) - AI agent framework, must be installed separately (`openclaw` CLI)
4. **Autopilot Runtime Service** (submodule: Wazuh-Openclaw-Autopilot/runtime/autopilot-service/, port 9090) - Node.js 20+, case management/approval/metrics

Data flow: Agents --> web_fetch --> Runtime :9090 --> callMcpTool --> MCP Server :3000 --> Wazuh API :55000
           Runtime --> webhook --> OpenClaw Gateway :18789 --> Next Agent

**Why:** Understanding this topology is essential for local setup -- services must start in dependency order and network between each other.

**How to apply:** When setting up locally, ensure Wazuh Manager is reachable first, then MCP Server, then OpenClaw Gateway, then Autopilot Runtime. Docker compose files exist in both submodules but there is no root-level compose file to orchestrate everything together.
