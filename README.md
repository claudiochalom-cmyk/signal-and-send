# Signal & Send

> AI agent that connects to Bloomreach via Loomi Connect MCP to automatically diagnose loyalty program performance and generate stakeholder-ready lifecycle intelligence reports.

## Bloomreach Loomi Connect AI Hackathon · June 2026
**Track:** T4 — Engagement Intelligence & Lifecycle Agents  
**Team:** Signal & Send · Solo entry · Claudio Chalom

---

## The problem

CRM teams running grocery loyalty programs generate continuous engagement data — email open rates, click-through rates, segment performance, points redemption activity — but have no efficient way to connect those signals into a coherent explanation of what happened and why.

When a re-engagement campaign underperforms, the typical diagnostic process involves manually cross-referencing campaign reports, segment lists, event histories, and analytics dashboards. This takes hours and still often produces incomplete findings.

## The solution

Signal & Send is an agent that connects Bloomreach's Marketing and Analytics MCP surfaces to automatically diagnose loyalty campaign underperformance and generate a stakeholder-ready narrative report — complete with root cause findings, confidence levels, and specific recommended actions.

A CRM coordinator asks: *"Why did our re-engagement campaign underperform?"*  
The agent calls the relevant Loomi Connect MCP tools, reasons over the signals, and returns a structured report a VP could read in 60 seconds.

## Demo

▶️ [Watch the demo on YouTube](https://youtu.be/xHBul263So0?si=vQ4q_Y_OwS7Emvvy)

---

## Architecture

```
CRM Coordinator (natural language question)
        ↓
Claude Code (Anthropic API) — agent runtime
        ↓
Loomi Connect MCP layer (SSO authenticated, read-only)
        ↓                          ↓
Marketing MCP              Analytics MCP
Segments · campaigns       Performance metrics
Journeys · scenarios       Trends · anomalies
        ↓                          ↓
Bloomreach Engagement sandbox (neon-walrus)
123K synthetic customers · loyalty events · purchase history
        ↓
Stakeholder narrative report
What happened · Why · Recommended actions · Human review note
```

---

## MCP surfaces used

### Marketing MCP
- Project hierarchy and workspace exploration
- Customer identifier schema
- Event schema and behavioral event definitions

### Analytics MCP
- Loyalty event analysis (points earned vs. redeemed)
- Purchase frequency trends (12-month window)
- Loyalty tier breakdown (Standard / Silver / Gold)
- Churn risk signal detection
- Monthly active purchaser tracking

---

## Agent workflow

```
1. User asks a natural language question
2. Agent authenticates to Loomi Connect via SSO (OIDC)
3. Agent calls Marketing MCP → retrieves engagement structure
4. Agent calls Analytics MCP → retrieves performance signals
5. Agent reasons over combined data
6. Agent generates stakeholder narrative with:
   - Executive summary
   - Findings with confidence levels
   - Prioritized recommended actions
   - Human review note
```

---

## Tech stack

| Component | Tool |
|---|---|
| Agent runtime | Claude Code (Anthropic API) |
| MCP connection | Loomi Connect — `loomi-mcp-alpha.bloomreach.com/mcp` |
| Authentication | SSO (OIDC) via Bloomreach login |
| Sandbox | Bloomreach Engagement · neon-walrus workspace |
| Data | 123,000 synthetic loyalty customers |

---

## Setup

### Prerequisites
- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)
- Bloomreach sandbox credentials (provisioned by hackathon team)
- Claude account with Pro, Max, Team, or Enterprise subscription

### Connect the MCP server

```bash
claude mcp add loomi-connect --transport http https://loomi-mcp-alpha.bloomreach.com/mcp
```

### Run the agent

```bash
claude
```

When prompted, authenticate with your Bloomreach sandbox credentials via the browser window.

### Example prompts

```
List the Bloomreach organizations I have access to.

Explore the neon-walrus project — show me the customer schema, available segments, and any campaign data.

Using the loyalty_update and purchase events in neon-walrus, analyze customer engagement patterns. How many customers have loyalty events? What does points activity look like? Are there any customers showing declining purchase frequency that could indicate churn risk?
```

---

## Responsible design

- **Read-only MCP**: the agent only reads data — it cannot create, edit, or send campaigns
- **Human review note**: every report includes an explicit note identifying what to verify before acting
- **Confidence levels**: findings are labelled High, Medium, or Confirmed
- **No PII exposure**: agent works with aggregate segment and campaign data only
- **Synthetic data**: hackathon build uses Bloomreach neon-walrus sandbox only — no production data
- **Approval gate**: recommended actions require manager approval before deployment

---

## Key findings from live demo

From the neon-walrus sandbox analysis:

- **22.1% redemption rate** — points accumulating faster than being redeemed (disengagement signal)
- **78% of loyalty activity** from Silver and Gold tier customers
- **7% structural decline** in monthly active purchasers over 12 months
- **February 2026 cliff** — 149 customers dropped off, half never returned
- **Post-March softening** — ongoing passive churn of 30–40 customers/month projected

---

## Future roadmap

1. **Live MCP integration** — real-time Bloomreach data once authentication is stable in production
2. **Multi-campaign analysis** — compare performance across an entire campaign calendar
3. **Proactive monitoring** — flag anomalies before the coordinator asks
4. **Slack integration** — deliver reports automatically after campaigns close

---

*Signal & Send · Loomi Connect AI Hackathon · June 2026*
