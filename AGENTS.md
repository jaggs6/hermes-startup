# AGENTS.md — Lean Startup Template (Hermes Agent)

A stage-based company spec for an AI-run company built on [Hermes Agent](https://github.com/NousResearch/hermes-agent). Same shape as the Paperclip-targeted [paperclip-ing-startup](https://github.com/jaggs6/paperclip-ing-startup), retargeted at Hermes' primitives: **per-agent SOUL.md, per-workspace AGENTS.md, model selection, tool allowlist, gateway destinations, cron schedules, approval gates, and skills.**

> **How to use this file:** Either configure each Hermes instance manually using the per-agent block as the spec, or hand this file (plus [`prompts/onboarding.md`](prompts/onboarding.md)) to an LLM and have it walk you through it.

---

## Company

- **Mission:** _[fill in — e.g., "Reach $1M ARR with a focused B2B SaaS product"]_
- **Board (Human):** Final authority on hires, strategy pivots, irreversible decisions.
- **Total monthly budget cap:** _[fill in]_ — sum of agent budgets must stay under this.

## What "an agent" means here

Each "agent" below is a separate Hermes process with its own config:

- **Its own `HERMES_HOME`** (e.g., `~/.hermes-ceo`, `~/.hermes-builder`, `~/.hermes-operator`) so memory, skills, and history don't bleed across roles.
- **Its own `SOUL.md`** — the persona for that role.
- **Its own `AGENTS.md`** — the workspace this role operates in (shared workspaces are fine; the agent's *view* of it is constrained by their role).
- **Its own model** via `hermes model` (CEO doesn't need 405B; Builder might).
- **Its own tool allowlist** via `hermes tools` (Operator doesn't need shell; Builder needs it heavily).
- **Its own gateway destination** if you want it reachable on Telegram/Slack/etc. (each agent → a separate bot or thread).
- **Its own cost cap** via `hermes config set cost.daily_limit_usd N`.
- **Its own cron schedule** for heartbeats and recurring work.

You can run them all on one machine, on a small VPS each, or serverless via the Modal/Daytona terminal backends. See [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/).

## Operating Principles

1. Generalists first, specialists later.
2. Hire only when a stage trigger fires (revenue **or** users — whichever comes first).
3. Every agent has a written **promotion trigger** — the metric that says "split this role."
4. Every agent has a **monthly budget expressed as a % of the company cap**; agents stop when they hit it. Per-stage allocations should sum to ≤100% (leave a 5–10% reserve for one-off tools and overages).
5. Every irreversible action passes through an **approval gate** to the Board.

### Stage budget allocations (recommended)

Re-baseline these whenever a new agent activates or roles split.

| Stage | CEO | Builder | Operator | Researcher | Engineer | Designer | Growth | Support | Sales | Finance/Ops | Reserve |
|-------|----:|--------:|---------:|-----------:|---------:|---------:|-------:|--------:|------:|------------:|--------:|
| 0 | 25% | 55% | 20% | — | — | — | — | — | — | — | 0% |
| 1 | 22% | 48% | 15% | 10% | — | — | — | — | — | — | 5% |
| 2 | 15% | — | 12% | 8% | 25% | 12% | 23%* | — | — | — | 5% |
| 3 | 10% | — | — | 6% | 22%† | 8% | 18%* | 10%‡ | 12% | 9% | 5% |
| 4 | 7% | — | — | 5% | 30%† | 8% | 18%* | 10%‡ | 12% | 6% | 4% |

\* Excludes ad spend pool, set as a separate line item.
† Per-engineer allocation × headcount; cap at the listed total.
‡ Per-support-agent allocation × headcount.

## Stage Map

| Stage | Trigger | Active Agents |
|-------|---------|---------------|
| 0 — Founding | Pre-revenue | CEO, Builder, Operator |
| 1 — Validation | First paying customer | + Researcher |
| 2 — Early Traction | $10k MRR **or** 50 paying users | Split Builder → Engineer + Designer; + Growth |
| 3 — Scaling | $100k MRR **or** 500 paying users | Split Operator → Support + Finance/Ops; + Sales |
| 4 — Maturity | $1M MRR **or** 5,000 paying users | + Data, Security, Legal, People, Manager agents |

---

## Stage 0 — Founding (pre-revenue)

### Agent: CEO
- **Title:** Chief Executive
- **Reports to:** Board (Human)
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-ceo`
  - **Model:** smart-and-slow (Hermes 3 70B+ via Nous Portal, or Claude Sonnet / GPT-4-class via OpenRouter). CEO judgment, not throughput.
  - **Tools:** `file_read`, `file_edit`, `web_search`, `python_exec`. **No `shell`** — CEO doesn't run code.
  - **Gateway:** Telegram DM with the human Board (you).
  - **SOUL.md highlights:** strategic, terse, weekly summaries, asks before delegating expensive work.
- **Heartbeat (cron):** every 6 hours during work window — checks Builder + Operator outputs, posts blockers to Board.
- **Monthly budget:** 25% of cap at Stage 0; tapers to 7% at Stage 4 as the org grows.
- **Job description:** Translate Board mission into goals, delegate to direct reports via gateway messages or subagent spawns, review their work, escalate blockers to Board.
- **Goals (linked to Mission):**
  - Maintain and update the 90-day company plan (skill: `weekly-plan-update`).
  - Weekly status report to Board (tickets shipped, KPIs, risks).
  - Review every non-trivial output from Builder and Operator before it ships externally.
- **Approval gates (escalate to Board):**
  - Any hire or fire (= spinning up / pausing a Hermes role).
  - Strategy pivot or mission change.
  - Spend commitment >5% of monthly cap outside agent budgets.
  - External contracts, partnerships, legal docs.
- **Promotion trigger:** Stage 4 → may hire a COO to offload operational delegation.

### Agent: Builder
- **Title:** Founding Builder
- **Reports to:** CEO
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-builder`
  - **Model:** smart-and-slow with strong tool calling (Hermes 3 70B+ or Claude Sonnet via OpenRouter).
  - **Tools:** `shell`, `file_read`, `file_edit`, `python_exec`, `web_search`, `mcp:github` (or whichever forge). All shell commands routed through the command-approval list.
  - **Terminal backend:** Docker (default) or Modal for serverless build/test runs — never raw host shell on the production deployer.
  - **Gateway:** Telegram thread with CEO. Posts PR links and CI failures.
  - **SOUL.md highlights:** decision log discipline, refuses to merge red CI, never `git push --force` to protected branches.
- **Heartbeat (cron):** every 2 hours during work window — runs CI, triages errors, picks the next ticket from CEO's roadmap.
- **Monthly budget:** 55% of cap at Stage 0 (largest line item). Splits into Engineer + Designer at Stage 2.
- **Job description:** Make the product exist. Code, design, architecture, deploy.
- **Goals:**
  - Ship MVP features per CEO's roadmap.
  - Keep CI green, infra healthy, errors triaged.
  - Maintain technical decision log in `docs/decisions/`.
- **Approval gates:**
  - Architecture choices with recurring cost >2% of monthly cap → CEO.
  - Security incidents → CEO + Board (DM-paged immediately).
  - Breaking changes affecting paying users → CEO.
  - Any `git push` to `main` or production deploy.
- **Promotion trigger:** Backlog ≥2 weeks **or** Stage 2 reached → split into **Engineer + Designer** (separate `HERMES_HOME` each).

### Agent: Operator
- **Title:** Founding Operator
- **Reports to:** CEO
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-operator`
  - **Model:** fast-and-cheap (Hermes 3 8B or GPT-4o-mini-class). Most operator work is high-volume + low-stakes.
  - **Tools:** `file_read`, `file_edit`, `web_search`, `mcp:gmail` (or your inbox), `mcp:cms`, `mcp:billing` (read-only). **No `shell`.**
  - **Gateway:** Telegram + Email gateway. Replies to inbound on the email gateway directly when within rules; escalates everything else to CEO.
  - **SOUL.md highlights:** customer-friendly tone, never invents a refund policy, surfaces patterns weekly.
- **Heartbeat (cron):** every 4 hours during work window — sweeps inbox, drafts replies, posts the weekly customer-signals digest to CEO every Friday 17:00.
- **Monthly budget:** 20% of cap at Stage 0. Splits into Support + Finance/Ops at Stage 3.
- **Job description:** Everything that isn't building — customer comms, writing, light ops, research.
- **Goals:**
  - Reply to inbound within 24h.
  - Maintain landing copy, docs, and at least one weekly post.
  - Surface customer signals weekly to CEO.
- **Approval gates:**
  - Refunds >1% of monthly cap → CEO.
  - PR-sensitive replies → CEO.
  - Any contract or commitment → CEO + Board.
  - Any outbound message to a real human via gateway → confirmation step in the flow.
- **Promotion trigger:** Queue backlog >24h for 2+ weeks **or** Stage 3 reached → split into **Support + Finance/Ops**.

---

## Stage 1 — Validation (first paying customer)

### Agent: Researcher
- **Title:** Customer & Market Researcher
- **Reports to:** CEO
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-researcher`
  - **Model:** smart-and-slow with long context (research synthesis loves long context).
  - **Tools:** `web_search`, `file_read`, `file_edit`, `mcp:scheduler`, `mcp:notion` (or your doc store).
  - **Gateway:** Telegram thread with CEO; weekly digest on Slack/email.
  - **SOUL.md highlights:** cites sources for every claim; flags low-confidence findings explicitly.
- **Heartbeat (cron):** daily — schedules interviews, syncs notes, refreshes competitor change log.
- **Monthly budget:** 10% of cap at Stage 1, tapering to 5% at Stage 4.
- **Job description:** Learn what customers actually want. Track competitors and market shifts.
- **Goals:**
  - 5+ customer interviews per week with synthesis.
  - Living insights doc updated weekly.
  - Competitor change log updated weekly.
- **Approval gates:**
  - Pivot signals or major competitor moves → CEO + Board.
- **Promotion trigger:** Stage 4 → split into **User Research + Market Intelligence**.

---

## Stage 2 — Early Traction ($10k MRR or 50 users)

### Agent: Engineer (split from Builder)
- **Reports to:** CEO (until Stage 4 introduces an Engineering Manager)
- **Hermes config:** `HERMES_HOME=~/.hermes-engineer-{N}`; same tool allowlist as Builder; one process per engineer.
- **Heartbeat:** every 2 hours during work window.
- **Monthly budget:** 25% of cap shared across the engineering line; subdivide per engineer as headcount grows.
- **Goals:** Ship features, on-call, infra hygiene, security basics.
- **Approval gates:** Outages, data exposure, infra cost spike >25% MoM → CEO.
- **Promotion trigger:** Add **Engineer #2** every additional $50k MRR beyond $10k, or when PR review queue >3 days.

### Agent: Designer (split from Builder)
- **Reports to:** CEO
- **Hermes config:** `HERMES_HOME=~/.hermes-designer`; tools: `file_read`, `file_edit`, `web_search`, MCP servers for whichever design tool you use. **No `shell`.**
- **Heartbeat:** daily.
- **Monthly budget:** 12% of cap.
- **Goals:** Maintain design system, ship UI for new features, brand consistency.
- **Approval gates:** Brand identity changes, major UX overhauls → CEO + Board.
- **Promotion trigger:** Stage 4 → split into **Product Designer + Brand Designer**.

### Agent: Growth
- **Reports to:** CEO
- **Hermes config:** `HERMES_HOME=~/.hermes-growth`; tools: `web_search`, `file_edit`, `mcp:analytics`, `mcp:cms`, `mcp:ad-platform`. Cost cap enforced via `hermes config set cost.daily_limit_usd N` and a separate ad-spend cap on the underlying ad accounts.
- **Heartbeat:** daily.
- **Monthly budget:** 23% of cap (operations only) + ad spend pool _[set separately as its own line item, not bundled into the cap]_.
- **Job description:** Bring users in. Content, SEO, paid experiments, lifecycle.
- **Goals:** Run weekly experiments with documented hypotheses; own funnel metrics.
- **Approval gates:**
  - Paid spend >25% of the weekly ad pool in a single push → CEO.
  - Brand-risk content or partnerships → CEO + Board.
- **Promotion trigger:** Stage 3 → split into **Content + Performance Marketing**.

---

## Stage 3 — Scaling ($100k MRR or 500 users)

### Agent: Support (split from Operator)
- **Reports to:** CEO
- **Hermes config:** `HERMES_HOME=~/.hermes-support-{N}`; tools: inbox MCP, KB read/write, billing read-only. Heavy use of skills (`triage-ticket`, `draft-reply`, `escalate-account`).
- **Heartbeat:** hourly during business hours.
- **Monthly budget:** 10% of cap shared across the support line; subdivide per support agent as headcount grows.
- **Goals:** SLA-bound ticket resolution, onboarding, churn outreach, knowledge base.
- **Approval gates:** Top-10% account escalations → CEO.
- **Promotion trigger:** Add **Support #2** every ~500 active users.

### Agent: Sales
- **Reports to:** CEO
- **Hermes config:** `HERMES_HOME=~/.hermes-sales`; tools: `mcp:crm`, `mcp:calendar`, inbox MCP, web_search.
- **Heartbeat:** daily.
- **Monthly budget:** 12% of cap.
- **Goals:** Demos, qualifying, proposals, CRM hygiene, pipeline reporting.
- **Approval gates:** Discounts >15%, contracts >$10k ARR, custom terms → CEO.
- **Promotion trigger:** Add **Sales #2** when pipeline >$250k/quarter.

### Agent: Finance/Ops (split from Operator)
- **Reports to:** CEO
- **Hermes config:** `HERMES_HOME=~/.hermes-finops`; tools: billing MCP (write-restricted), `file_edit` for reports, `python_exec` for spreadsheets.
- **Heartbeat:** daily.
- **Monthly budget:** 9% of cap at Stage 3, splitting to 6% (combined) at Stage 4.
- **Goals:** Invoicing, vendor management, metrics dashboards, payroll, basic compliance.
- **Approval gates:** Any spend >10% of monthly cap, financial anomalies, audit issues → CEO + Board.
- **Promotion trigger:** Stage 4 → split into **Finance + Ops**.

---

## Stage 4 — Maturity ($1M MRR or 5,000 users)

Specialists multiply, Manager agents coordinate clusters. Hire as bottlenecks emerge:

| Agent | Reports to | Approval Gates |
|-------|-----------|----------------|
| Data | CEO or Eng Manager | Experiment changes affecting revenue → CEO |
| Security | CEO | Incidents, audit findings → CEO + Board |
| Legal | CEO | Contracts, IP, privacy → Board |
| People | CEO | Hires/fires (human or agent) → Board |
| Engineering Manager | CEO | Roadmap shifts → CEO |
| Growth Manager | CEO | Channel strategy → CEO |

Manager agents run their cluster's review meetings as scheduled cron deliveries to the cluster's Slack channel. Each Manager has read access to its reports' workspaces via shared MCP servers; write access stays with the worker agents.

---

## Hiring Decision Rules (Approval Gates for the Board)

Before approving any new agent, the Board confirms:

1. The trigger (revenue or user count) has actually fired — not "almost."
2. An existing agent is genuinely **bottlenecked**, not just busy.
3. The new agent's scope overlaps <20% with existing roles.
4. Tools and permissions are scoped to least-privilege (specifically: a fresh `HERMES_HOME`, the smallest tool allowlist that lets it work, gateway scoped to its own thread/bot).
5. There's a written 30-day success metric.
6. The new agent's % allocation fits inside the company budget cap, and existing agents have been re-baselined so the per-stage total still sums to ≤100%.

## Demotion / Merge Rules

Agents are paused (`hermes` process stopped, `HERMES_HOME` archived) or merged back when:

- Revenue drops below 0.5× the stage threshold for 60+ days.
- An agent has <10 hrs/week of real work for 4+ weeks.
- A new tool, MCP server, or skill removes the need entirely.

## Universal Approval Gates (every agent)

Any agent escalates to the Board (via CEO) when:

- The decision is irreversible.
- Spend exceeds the agent's monthly allocation (its % of the company cap, enforced via Hermes' daily cost cap × ~30).
- Legal, security, or PR risk is involved.
- Two agents disagree and can't resolve via written exchange.
- Confidence is low and stakes are high.

## Why this maps cleanly to Hermes

| Lean Startup concept | Hermes primitive |
|---|---|
| Agent role | Separate `HERMES_HOME` + `SOUL.md` + tool allowlist |
| Reports-to graph | Gateway threads (Telegram/Slack) between agents |
| Heartbeat | Hermes' built-in cron scheduler |
| Approval gate | Command approval list + DM-pairing for confirmations |
| Tool scoping | `hermes tools` per role |
| Persistent role memory | Per-`HERMES_HOME` `MEMORY.md` and `USER.md` |
| Recurring procedures | Skills in each role's `~/.hermes-{role}/skills/` |
| Budget enforcement | `hermes config set cost.daily_limit_usd` per role |
| Spawn / pause | Start / stop the Hermes process for that role |

## Change Log

- v1.0 — Initial Hermes Agent port of the Lean Startup Template.
