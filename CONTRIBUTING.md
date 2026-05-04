# Contributing

Thanks for wanting to make this template better. Three rules:

1. **Keep it lean.** This template's value is what it leaves out. If you're adding a section, justify why a Stage 0 founder can't ship without it.
2. **Cite the trigger.** Any new role, gate, or rule needs a measurable trigger — revenue, users, latency, queue depth. No "it depends" advice.
3. **Cite Hermes upstream.** Anything Hermes-specific (CLI flags, model strings, tool names, file paths, cron syntax) must link to [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) or [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/). Don't paraphrase from memory.

## Good PRs

- A new filled-in example under `examples/` for a business shape we don't cover yet (marketplace, agency, content business, dev tool).
- Tightening a vague approval gate into a numeric threshold.
- Replacing a placeholder Hermes config (model, tool list, cron) with one you've actually run.
- Fixing a stage trigger that didn't match reality (with a one-line explanation of what you saw).
- Tighter mappings from a Lean Startup primitive → a Hermes primitive.

## Out of scope

- Generic startup advice not tied to agent configuration.
- Forking Hermes' own behavior. This repo doesn't patch Hermes; it helps users configure it.
- Adding agents at Stage 0 that aren't CEO / Builder / Operator.

## Process

1. Fork, branch, edit.
2. Open a PR with a one-paragraph "what changed and why."
3. If your change adds an agent, gate, or Hermes config, mention which Hermes version you tested with.
