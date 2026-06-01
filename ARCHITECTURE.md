# Signal & Send — Architecture Overview

## System layers

### Layer 1 — User interface
The CRM Marketing Coordinator interacts with Signal & Send through Claude Code's natural language chat interface. No dashboards, no query builders — just a plain English question.

**Example input:**
> "Using the loyalty_update and purchase events in neon-walrus, analyze customer engagement patterns. Are there any customers showing declining purchase frequency that could indicate churn risk?"

---

### Layer 2 — Agent runtime
**Tool:** Claude Code (Anthropic API — claude-sonnet-4-20250514)

The agent runtime is responsible for:
- Understanding the coordinator's natural language question
- Deciding which MCP tools to call and in what order
- Reasoning over the data returned by those tool calls
- Generating a structured stakeholder narrative

The agent does not just retrieve and display data. It connects signals across multiple tool calls to reason about causality — identifying patterns, anomalies, and likely root causes before generating output.

---

### Layer 3 — Loomi Connect MCP layer
**Endpoint:** `https://loomi-mcp-alpha.bloomreach.com/mcp`  
**Authentication:** SSO (OIDC) via Bloomreach login  
**Access type:** Read-only

The MCP layer is the bridge between the agent runtime and Bloomreach data. It exposes Bloomreach intelligence as structured tools the agent can call. For this hackathon, two MCP surfaces were used:

#### Marketing MCP
| Tool call | Purpose |
|---|---|
| List projects | Discover available workspaces |
| Get customer identifier schema | Understand how customers are identified |
| Get event schema | Retrieve available behavioral event types |
| List segments | Check for defined audience segments |
| List campaigns | Check for active or historical campaigns |

#### Analytics MCP
| Tool call | Purpose |
|---|---|
| Query loyalty events | Retrieve points earned vs. redeemed counts |
| Query purchase events | Retrieve transaction history and frequency |
| Aggregate by time period | Build monthly trend tables |
| Aggregate by loyalty tier | Break down engagement by Standard/Silver/Gold |
| Detect anomalies | Identify unusual drops or spikes in activity |

---

### Layer 4 — Bloomreach Engagement sandbox
**Workspace:** neon-walrus (ID redacted)
**Organization:** Hackathon Org (ID redacted)

The sandbox contains:
- 123,160 synthetic customer profiles
- Customer identifiers: registered (hard), cookie (soft), google_analytics (soft)
- Customer properties: city, country, email, first/last name, gender, language, phone, external_score
- Event schema including: loyalty_update, purchase, purchase_item, store_visit, view_item, cart_update, checkout, campaign, consent, survey, experiment

**Note:** No production customer data was used at any point. All analysis was performed on Bloomreach-provided synthetic sandbox data.

---

### Layer 5 — Output
The agent produces a structured stakeholder narrative containing:

1. **Executive summary** — plain language overview of what was found
2. **Engagement metrics** — key numbers with benchmark comparisons
3. **Findings** — root cause analysis with confidence levels (High / Medium / Confirmed)
4. **Recommended actions** — prioritized, specific, and immediately actionable
5. **Human review note** — explicit flag for what requires verification before acting

---

## Data flow

```
User question
    ↓
Claude Code parses intent
    ↓
Agent calls Marketing MCP → gets project structure and event schema
    ↓
Agent calls Analytics MCP → queries loyalty_update and purchase events
    ↓
Agent aggregates by month, by tier, by action type
    ↓
Agent identifies anomalies and patterns
    ↓
Agent reasons: what does this mean? what caused it? what should happen next?
    ↓
Agent generates stakeholder narrative
    ↓
Human reviews output
    ↓
Human decides whether to act
```

---

## Responsible design decisions

| Decision | Rationale |
|---|---|
| Read-only MCP | Prevents any accidental modification of Bloomreach data |
| Human review note in every output | Keeps human judgment in the loop for all business decisions |
| Confidence levels on findings | Helps the coordinator prioritize verification effort |
| No PII in output | Agent works with aggregates, not individual customer records |
| Sandbox data only | No production customer data used at any stage |
| No automated sends | Agent recommends only — execution requires human approval |

---

## Known limitations

- **No campaign history in sandbox**: neon-walrus had no pre-built campaigns or segments, so campaign-level diagnosis used event data only
- **MCP is read-only**: cannot create segments or campaigns directly through the MCP layer (write operations require Bloomreach REST API)
- **Authentication session**: SSO session required re-authentication if the local listener timed out during OAuth callback

---

## Production path

To move Signal & Send toward production:

1. **Replace sandbox with production Bloomreach workspace** — swap project ID in MCP config
2. **Add PII guardrails** — filter output to prevent individual customer data from appearing in reports
3. **Add Slack delivery** — post stakeholder narrative to a designated channel automatically after campaigns close
4. **Add proactive monitoring** — schedule weekly analysis runs rather than requiring a manual prompt
5. **Add write capability via REST API** — allow agent to prepare (not send) campaign drafts for human review

---

*Signal & Send · Loomi Connect AI Hackathon · June 2026 · Claudio Chalom*
