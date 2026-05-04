# hermes-startup — a lean AGENTS.md template for [Hermes Agent](https://github.com/NousResearch/hermes-agent)

Spin up an AI-run company on [Hermes Agent](https://hermes-agent.nousresearch.com) without staring at a blank screen.

This repo gives you a single source of truth — [`AGENTS.md`](AGENTS.md) — that describes a stage-based org chart (CEO, Builder, Operator → Engineer, Designer, Growth, Support, Sales, …) where every "agent" is a separate Hermes process with its own `HERMES_HOME`, `SOUL.md`, model, tool allowlist, gateway destination, cron schedule, and cost cap. Edit the placeholders, run a few `hermes` commands, and you have a running company.

> **Sister repo:** the same template targeted at [Paperclip](https://paperclip.ing) lives at [paperclip-ing-startup](https://github.com/jaggs6/paperclip-ing-startup). Same shape, different runtime.

## Quick start (with an LLM — recommended)

Install Hermes first if you haven't:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Then open ChatGPT, Claude, or any LLM that can fetch URLs and paste this prompt:

```
You are helping me bootstrap an AI-run company on Hermes Agent
(https://github.com/NousResearch/hermes-agent).

Read these two files first:
- https://raw.githubusercontent.com/jaggs6/hermes-startup/main/AGENTS.md
- https://raw.githubusercontent.com/jaggs6/hermes-startup/main/prompts/onboarding.md

Then follow the interview script in onboarding.md exactly:
- Ask me ONE question at a time, in order.
- Default aggressively when I don't know.
- When you have enough answers, output a complete filled-in AGENTS.md
  for my company (strip stages above my current one, compute $ alongside %,
  bias each agent's first-week goals toward my biggest constraint).
- Also output ./hermes-setup.md with the exact `hermes` CLI commands
  to spin up each agent: HERMES_HOME, model, tools, gateway, cost cap,
  and cron schedule.
- Save both files to my current working directory (warn before
  overwriting), then confirm the paths.

Start with question 1.
```

The LLM walks you through it and hands back a ready-to-run setup.

## Quick start (manual)

1. **Fork or download** this repo.
2. Open [`AGENTS.md`](AGENTS.md) and fill in the placeholders at the top:
   - Mission
   - Total monthly budget cap
   - Anything else in `_[brackets]_`
3. **For each agent in your current stage**, follow the per-agent block:
   ```bash
   HERMES_HOME=~/.hermes-ceo hermes setup
   # pick the model, tools, gateway listed in the CEO block
   HERMES_HOME=~/.hermes-ceo hermes config set cost.daily_limit_usd <cap × % ÷ 30>
   # write the SOUL.md for that role at ~/.hermes-ceo/SOUL.md
   ```
   Repeat for Builder, Operator, etc.
4. Wire **gateways** so agents can talk to each other and to you:
   - One Telegram bot per agent, in a single group with the human Board.
   - Or one Slack workspace, one channel per role.
5. Wire **heartbeats** via Hermes' cron scheduler — see each agent block for the recommended cadence.
6. As you hit a stage trigger ($10k MRR, 50 users, etc.), open `AGENTS.md`, follow the **Promotion trigger** lines, and split roles (= spin up a new `HERMES_HOME`).

## Why this template

Most "AI-native company" guides skip the boring parts: who reports to whom, what each agent is *not* allowed to do, when to hire the next role. This template encodes:

- **Stage gates** — you don't hire a Sales agent at $0 MRR.
- **Approval gates** — irreversible actions always escalate to the human Board via Hermes' command-approval list.
- **Budgets** — every agent gets a **%-of-cap allocation** (not a hardcoded dollar amount), so the same template works whether your monthly cap is $400 or $40,000. Per-stage allocations sum to ≤100%. Enforced via `hermes config set cost.daily_limit_usd` per role.
- **Promotion triggers** — explicit metrics that say "split this role now."
- **Demotion rules** — when to pause or merge agents back.

## What's in here

| File | Purpose |
|------|---------|
| [`AGENTS.md`](AGENTS.md) | The template. Edit this. |
| [`llms.txt`](llms.txt) | Sitemap for LLMs reading this repo (per [llmstxt.org](https://llmstxt.org)). |
| [`prompts/onboarding.md`](prompts/onboarding.md) | Interview script — paste into ChatGPT/Claude and it'll walk you through filling out `AGENTS.md` and `hermes-setup.md`. |
| [`examples/b2b-saas.md`](examples/b2b-saas.md) | Filled-in example for a B2B SaaS at Stage 1 with concrete Hermes config per agent. |
| [`examples/indie-app.md`](examples/indie-app.md) | Filled-in example for a solo indie consumer app at Stage 0. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to propose changes. |
| [`LICENSE`](LICENSE) | MIT. |

## Upstream — verify against these

- [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) — Hermes Agent source.
- [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/) — official docs.
- [agentskills.io](https://agentskills.io) — open skills standard Hermes is compatible with.

## Contributing

PRs welcome — especially:

- New filled-in examples (marketplaces, dev tools, agencies, content businesses).
- Better promotion/demotion heuristics from real Hermes operators.
- Tighter mappings from Lean Startup primitives → Hermes primitives.

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE). Use it, fork it, sell what you build with it.
