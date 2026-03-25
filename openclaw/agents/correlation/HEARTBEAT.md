# Correlation Agent -- Heartbeat (Cron-Triggered)

**IMPORTANT:** Use the `exec` tool with `curl` for all HTTP requests. Do NOT use `web_fetch` (it blocks internal addresses). `curl` can reach `http://127.0.0.1:9090`.

## 5-Minute Active Case Recorrelation

This procedure runs on a 5-minute cron cycle. Follow each step in order.

### 1. Fetch new triage cases
Use `exec` with `curl` to query the runtime API for triaged cases:

    exec(command="curl -s 'http://127.0.0.1:9090/api/cases?token=<AUTOPILOT_MCP_AUTH>'")

Filter the response for cases with status `triaged` created since the last heartbeat (last 5 minutes).

### 2. Fetch active clusters
Load all open correlated clusters from the last 24 hours that have not been closed or resolved.

### 3. Attempt to merge new cases into existing clusters
For each new triage case:
- Compute entity overlap, temporal proximity, rule similarity, and attack chain scores against each active cluster.
- If the composite score meets the minimum link threshold (0.5), merge the case into the cluster.
- If the composite score meets the strong correlation threshold (0.8), flag the cluster for immediate handoff to the Investigation Agent.

### 4. Form new clusters from unmerged cases
Group remaining unmerged cases using the four clustering strategies. Any group of 2+ cases exceeding the minimum link threshold becomes a new cluster.

### 5. Recalculate cluster metadata
For every modified or new cluster:
- Rebuild the timeline with kill chain phase tags
- Recalculate the blast radius
- Update the correlation score
- Re-evaluate the attack pattern classification
- Apply severity boosts

### 6. Hand off high-priority clusters
Any cluster with correlation score >= 0.8 or boosted severity of `critical` — invoke `exec` with `curl` to update the case status to `correlated` (see AGENTS.md MANDATORY section) to hand off to the Investigation Agent.

### 7. Log heartbeat summary
Record: new cases processed, clusters updated, new clusters formed, clusters handed to investigation, processing duration.
