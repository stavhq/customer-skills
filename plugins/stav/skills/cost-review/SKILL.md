---
name: cost-review
description: Show where a Stav workspace's credits went — by model, by API key and by Router — and for the biggest costs, suggest a cheaper model that keeps the same policy and needs, with the saving worked out. Read-only unless the person asks for a change. Through Stav's MCP server. Use when someone asks "where did my credits go", "why is Stav expensive this month", "what costs the most", "how do I spend less", "how long will my credits last", or "is there a cheaper model for this".
---

# Cost review

You are the person's own agent, connected to Stav's MCP server. Every
tool in this review is a free read. You change nothing unless the person
asks, and then only after a yes.

## Rules

1. **Money in the workspace currency.** Use `cost` and `currency` from the
   tools — what the workspace was charged. `cost_eur` is a catalogue
   estimate; do not show it beside `cost` unless asked.
2. **Same policy, same needs.** A cheaper model counts only if it serves
   under the workspace policy and has every capability the current one is
   used for (tools, JSON output, images, PDFs, context length). Check it;
   never assume.
3. **Not measured is not zero.** A model with no score on what matters is
   not "as good". Say "not measured".
4. **Short.** At most three costs, one suggestion each.

## The flow

### 1. The picture (four reads)

- `wallet_get` — balance and currency.
- `usage_get_summary` — requests, tokens and the model breakdown.
- `analytics_top_models` with `rank_by: "cost"`, `days: 30`.
- `apikeys_spend` with `days: 30` — which keys spend.

If the person uses Routers (`routers_list`), add `routers_cost_by_model`
with `router_id` and `days: 30` for each Router that spent, to see which
models the Router picked.

Tell the person, in at most six lines: total spent in the period, the
balance and how long it lasts at this rate, and the top three costs, each
with model, key or Router, requests, and cost.

### 2. Cheaper models (one call per top cost)

`smartrouter_get_config` once, for the policy. Then for each of the top
three models, `models_shortlist` with:

- the policy: `runs_in`, `min_rung`, `zdr`, `no_training`;
- the needs: `capabilities` the app uses (ask if you cannot tell);
- `strength` a little below the current model's on what matters;
- `volume` from step 1 (requests per month, average input and output
  tokens per request), so `estimated_cost` is worked out;
- `priorities: {quality: 40, cost: 60, speed: 0}`;
- `explain: ["<current model>"]`.

Offer one alternative per cost: the model, the quality gap in plain words,
and the saving per month at today's volume.

`routers_baselines` shows what a Router saves against one fixed model, if
the person wants that comparison.

### 3. A change, on request

Only on a yes:

- **Use the cheaper model in a Router** — `routers_draft_set` →
  `routers_try` → `routers_publish` for an existing Router; a new Router
  with `routers_suggest` → `routers_create`.
- **A spend cap** — `budget_set` with `scope_type` `team`, `key` or
  `router`, a `limit`, `period` `day` or `month`, and `mode` `soft`
  (alert) or `hard` (block). Owner or Admin only.

A model named in app code changes in the code, not in Stav. Show the line
to change.

## Finish

Three lines: what cost the most, what could be cheaper, the saving.
