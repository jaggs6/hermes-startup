# Example: B2B SaaS at Stage 1 (on Hermes Agent)

A worked example of [`AGENTS.md`](../AGENTS.md) for a small B2B SaaS — a Linear-style issue tracker for legal teams — that just landed its first paying customer.

---

## Company

- **Mission:** Reach $50k MRR in 12 months by building a focused issue tracker for in-house legal teams.
- **Board:** Two human co-founders.
- **Total monthly budget cap:** $1,500.

## Active Stage

**Stage 1 — Validation.** First paying customer signed last week ($400/mo). Researcher agent activated.

## Active Agents

### CEO
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-ceo`
  - **Model:** `openrouter:anthropic/claude-sonnet-4.5`
  - **Tools:** `file_read`, `file_edit`, `web_search`, `python_exec`. No `shell`.
  - **Gateway:** Telegram bot `@legal_ceo_bot`, scoped to a 3-person group with both founders.
  - **Cost cap:** `hermes config set cost.daily_limit_usd 11` (≈ 22% × $1500 ÷ 30).
  - **SOUL.md:** terse, decision-log discipline, weekly Friday digest at 17:00 IST.
- **Heartbeat:** every 6h, work window 09:00–19:00 IST.
- **Monthly budget:** 22% of cap → $330.
- **This week's focus:**
  - Cut the 90-day plan from 14 goals to 5.
  - Review every outbound email Operator sends to the new paying customer.
  - Decide whether the next hire is a second customer interview push or a Growth experiment.

### Builder
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-builder`
  - **Model:** `openrouter:anthropic/claude-sonnet-4.5`
  - **Tools:** `shell` (Docker terminal backend, command-approval list pre-loaded), `file_read`, `file_edit`, `python_exec`, `web_search`, `mcp:github`, `mcp:sentry`.
  - **Gateway:** Telegram thread with CEO; CI failures and PR links posted automatically.
  - **Cost cap:** `hermes config set cost.daily_limit_usd 24` (≈ 48% × $1500 ÷ 30).
  - **SOUL.md:** never `git push --force` to `main`, always run tests before claiming done.
- **Heartbeat:** every 2h, 09:00–19:00 IST. Cron: triage Sentry errors at 08:00 daily.
- **Monthly budget:** 48% of cap → $720.
- **This week's focus:**
  - Ship SAML SSO (blocking enterprise pilot #2).
  - Set up error monitoring (Sentry) — currently flying blind. **Approval gate fires** at $210/mo recurring (over the 2%-of-cap = $30 threshold).
  - Decision log entry: chose Postgres over DynamoDB; revisit at Stage 2.

### Operator
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-operator`
  - **Model:** `openrouter:openai/gpt-4o-mini` (high-volume, low-stakes inbound).
  - **Tools:** `file_read`, `file_edit`, `web_search`, `mcp:gmail`, `mcp:webflow` (CMS), `mcp:stripe` (read-only).
  - **Gateway:** Email gateway as primary inbound; Telegram thread with CEO for surfacing signals.
  - **Cost cap:** `hermes config set cost.daily_limit_usd 7.5` (≈ 15% × $1500 ÷ 30).
  - **SOUL.md:** never invents refund policy, escalates anything PR-sensitive, drafts but never sends to a real human without confirmation.
- **Heartbeat:** every 4h. Cron: weekly customer-signals digest to CEO Friday 17:00 IST.
- **Monthly budget:** 15% of cap → $225.
- **This week's focus:**
  - Reply to 3 inbound demo requests within 24h.
  - Publish one post: "Why we built this for legal, not engineering."
  - Hand-off summary of customer signals to CEO every Friday.

### Researcher *(newly activated)*
- **Hermes config:**
  - `HERMES_HOME=~/.hermes-researcher`
  - **Model:** `openrouter:anthropic/claude-sonnet-4.5` (long context for synthesis).
  - **Tools:** `web_search`, `file_read`, `file_edit`, `mcp:cal` (scheduling), `mcp:notion`.
  - **Gateway:** Telegram thread with CEO; weekly insights digest published to a private Notion page.
  - **Cost cap:** `hermes config set cost.daily_limit_usd 5` (≈ 10% × $1500 ÷ 30).
  - **SOUL.md:** cites sources, flags low-confidence findings, never paraphrases without quoting.
- **Heartbeat:** daily at 09:30 IST.
- **Monthly budget:** 10% of cap → $150. Stage 1 reserve (5% → $75) is held back for one-off tools.
- **30-day success metric:** 20 interviews completed, insights doc updated weekly, 3 pivot signals flagged or explicitly ruled out.
- **First focus:** Interview the 12 trial accounts that didn't convert. Find the single biggest objection.

## Live Approval Gates

- Builder needs CEO sign-off before turning on a paid Sentry plan ($210/mo recurring — over the 2%-of-cap = $30 threshold for recurring infra).
- Operator needs CEO sign-off on the refund request from trial account #7 ($120 — over the 1%-of-cap = $15 refund threshold).
- CEO needs Board sign-off on the verbal partnership offer from a legal-ops consultancy.

## Hermes-specific notes

- All four agents run on a single $20/mo VPS, each in its own `HERMES_HOME`. Memory and skills are isolated.
- The Telegram group has all four bots + both founders. Each agent posts to a dedicated thread. CEO subscribes to all threads; founders only read CEO + escalations.
- Weekly cron job: Sunday 21:00 IST, runs `hermes update` on all four configs.

## Next Promotion Trigger

Splitting Builder → Engineer + Designer triggers at **$10k MRR or 50 paying users**. Currently at $400 MRR / 1 paying user. Not close.
