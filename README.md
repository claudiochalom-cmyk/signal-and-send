# Signal & Send

> A CRM-coordinator workflow that uses Claude Code, connected to Bloomreach via the Loomi Connect MCP, to diagnose loyalty program performance and produce a stakeholder-ready intelligence report from natural-language questions.

## Bloomreach Loomi Connect AI Hackathon · June 2026

**Track:** T4 — Engagement Intelligence & Lifecycle Agents
**Team:** Signal & Send · Solo entry · Claudio Chalom

---

## What this is (and isn't)

This is **not** a custom-coded application. It is a working configuration and prompting approach: Claude Code acts as the agent runtime, the Loomi Connect MCP server exposes Bloomreach's Marketing and Analytics tools, and a CRM coordinator drives the whole thing in plain English. The contribution here is the workflow design, the diagnostic prompting, and the responsible-use guardrails — not new software. Everything in this repo can be reproduced with the config and prompts below; there is no application code to install.

I built it as the person who actually does this job, so the framing is from a practitioner's chair rather than an engineer's.

---

## The problem

CRM teams running grocery loyalty programs generate continuous engagement data — email open rates, click-through rates, segment performance, points redemption activity — but have no efficient way to connect those signals into a coherent explanation of what happened and why.

When a re-engagement campaign underperforms, the typical diagnostic process means manually cross-referencing campaign reports, segment lists, event histories, and analytics dashboards. It takes hours and still often produces incomplete findings.

## The approach

A CRM coordinator asks, in plain language: *"Why did our re-engagement campaign underperform?"*

Claude Code, connected to Bloomreach through the Loomi Connect MCP, calls the relevant Marketing and Analytics tools, reasons over the returned signals, and produces a structured narrative report — root-cause findings, confidence levels, and recommended actions — that a VP could read in about a minute.

The "build" is the configuration, the diagnostic prompts, and the guardrails that make the output trustworthy.

---

## Architecture

```
CRM Coordinator (natural language question)
        |
Claude Code (Anthropic API) — agent runtime, not custom code
        |
Loomi Connect MCP layer (SSO authenticated, read-only)
        |                          |
Marketing MCP              Analytics MCP
Segments · campaigns       Performance metrics
Journeys · scenarios       Trends · anomalies
        |                          |
Bloomreach Engagement sandbox (neon-walrus)
123K synthetic customers · loyalty events · purchase history
        |
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

## Workflow

```
1. Coordinator asks a natural language question
2. Claude Code authenticates to Loomi Connect via SSO (OIDC)
3. Calls Marketing MCP tools -> retrieves engagement structure
4. Calls Analytics MCP tools -> retrieves performance signals
5. Reasons over the combined data
6. Produces a stakeholder narrative with:
   - Executive summary
   - Findings with confidence levels
   - Prioritized recommended actions
   - Human review note
```

---

## Tech stack

| Component      | Tool                                                 |
| -------------- | ---------------------------------------------------- |
| Agent runtime  | Claude Code (Anthropic API)                          |
| MCP connection | Loomi Connect — `loomi-mcp-alpha.bloomreach.com/mcp` |
| Authentication | SSO (OIDC) via Bloomreach login                      |
| Sandbox        | Bloomreach Engagement · neon-walrus workspace        |
| Data           | 123,000 synthetic loyalty customers                  |

---

## Setup

### Prerequisites

- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)
- Bloomreach sandbox credentials (provisioned by hackathon team)
- Claude account with Pro, Max, Team, or Enterprise subscription

### Connect the MCP server

```
claude mcp add loomi-connect --transport http https://loomi-mcp-alpha.bloomreach.com/mcp
```

### Run

```
claude
```

When prompted, authenticate with your Bloomreach sandbox credentials via the browser window, then use the prompts below.

### Example prompts

```
List the Bloomreach organizations I have access to.

Explore the neon-walrus project — show me the customer schema, available
segments, and any campaign data.

Using the loyalty_update and purchase events in neon-walrus, analyze customer
engagement patterns. How many customers have loyalty events? What does points
activity look like? Are there any customers showing declining purchase
frequency that could indicate churn risk?
```

---

## Responsible design

- **Read-only MCP**: the workflow only reads data — it cannot create, edit, or send campaigns
- **Human review note**: every report includes an explicit note identifying what to verify before acting
- **Confidence levels**: findings are labelled High, Medium, or Confirmed
- **No PII exposure**: works with aggregate segment and campaign data only
- **Synthetic data**: this build uses the Bloomreach neon-walrus sandbox only — no production data
- **Approval gate**: recommended actions require manager approval before any deployment

---

## Findings from the demo run

The figures below are outputs Claude Code produced from the neon-walrus **synthetic** sandbox during the recorded demo. They illustrate the kind of diagnosis the workflow generates; they are not real customer results.

- **22.1% redemption rate** — points accumulating faster than being redeemed (a disengagement signal)
- **78% of loyalty activity** from Silver and Gold tier customers
- **7% structural decline** in monthly active purchasers over 12 months
- **February 2026 cliff** — 149 customers dropped off, roughly half never returned
- **Post-March softening** — ongoing passive churn of 30–40 customers/month projected

---

## Demo

Watch the demo on YouTube: https://youtu.be/xHBul263So0

---

## If I took this further

1. **Live MCP integration** — real-time Bloomreach data once production authentication is stable
2. **Multi-campaign analysis** — compare performance across an entire campaign calendar
3. **Proactive monitoring** — flag anomalies before the coordinator asks
4. **Slack delivery** — push reports automatically after campaigns close

---

*Signal & Send · Loomi Connect AI Hackathon · June 2026*
