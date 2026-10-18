---
name: hosted-model-cost-evaluation
description: "Use when comparing hosted LLM costs, limits, or quotas."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [llm, pricing, quotas, subscriptions, model-evaluation, cost]
    related_skills: [mlops-model-operations, research-knowledge-work, grounded-citations, hermes-agent]
---

# Hosted Model Cost Evaluation

## Procedure

1. **Identify the actual route before comparing limits.** Record the model ID, provider, authentication method, subscription tier, and product surface. Treat Chat, agent/CLI products, API keys, aggregators, and enterprise workspaces as separate billing and quota systems even when they expose the same model.

2. **Fetch current primary documentation.** Prefer the provider's model page, subscription-limit page, agent/CLI pricing page, and rate card. Record whether limits are fixed messages, rolling windows, weekly caps, credits, tokens, or dynamic estimates. Cite dated community reports only as experience, never as the quota source.

3. **Normalize the rate card.** Compare uncached input, cached input, cache writes, output/reasoning tokens, speed tiers, long-context multipliers, and tool charges. Use a calculation tool for every ratio and worked example. State whether hidden reasoning is billed as output and whether prompt caching applies to the user's harness.

4. **Estimate cost per completed task, not only cost per token.** Combine the normalized rate multiplier with measured token efficiency, retries, tool-loop count, completion rate, and latency. Keep coding-agent evidence separate from general-chat or knowledge-work evidence; efficiency on one workload does not transfer automatically to another.

5. **Scope usage evidence correctly.** Inventory all providers and clients the user actually uses before inferring workload from local telemetry. Label each usage log by provider and surface. If local logs omit activity performed in another app, provider, or coding client, treat them as a partial baseline and say so explicitly rather than extrapolating total daily demand.

6. **Account for agent amplification.** One user request can create many model turns through tool results, retries, subagents, and verification. Evaluate agentic quotas from token/credit traces or the provider usage dashboard; do not equate one user prompt with one billable message.

7. **Give a workload-specific recommendation.** Separate routine work, difficult reasoning, coding, research, browser/computer use, and long-running autonomy. Recommend a default model, escalation model, reasoning level, fallback, and the conditions that justify the premium.

8. **Prefer a measured rollout when evidence is new or mixed.** Run representative tasks for several days, keep speed/effort settings fixed, and compare accepted-result rate, tokens/credits, latency, retries, and remaining allowance in the provider dashboard. Disable automatic credit reload during the trial unless the user explicitly wants paid overage.

## Standing Output Rules

- Lead with a direct answer to whether the proposed model is viable as a daily driver and whether the relevant limit is likely to bind.
- Distinguish documented facts, third-party benchmarks, firsthand anecdotes, and your estimate.
- Show the decisive rate and quota comparison in a compact table.
- Give a practical default/fallback strategy rather than declaring one universal winner.
- When the user's usage spans multiple providers, acknowledge that split before discussing their personal burn rate.

## Pitfalls

- **Do not apply a Chat message cap to an agent/CLI session** — products often maintain separate allowances even under one subscription.
- **Do not infer total workload from one provider's telemetry** — activity in other apps and providers is invisible to that log.
- **Do not multiply sticker price by historical tokens and call it a forecast** — a stronger model may change token count, retries, and success rate.
- **Do not treat benchmark token efficiency as universal** — coding harnesses and general knowledge work can show opposite cost behavior.
- **Do not quote dynamic message ranges as guarantees** — task complexity, context, tool use, caching, and weekly caps alter effective capacity.
- **Do not recommend the highest reasoning or speed tier by default** — these settings can consume allowance much faster without improving routine tasks.

## Verification Checklist

- [ ] Model, provider, auth route, plan, and product surface are explicit.
- [ ] Current primary documentation supports the limits and rates.
- [ ] Rate ratios and examples were calculated with a tool.
- [ ] User telemetry is labeled complete or partial across providers.
- [ ] Agent-loop amplification and caching are considered.
- [ ] Recommendation includes default, escalation, fallback, and a measurement plan.
