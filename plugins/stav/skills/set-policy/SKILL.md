---
name: set-policy
description: Read and set the Stav workspace policy — where requests may run (EU, EEA, a country), the minimum sovereignty level, zero data retention, no training, and which provider countries or Inference Providers are blocked — explained in plain words, changed only after a yes. Through Stav's MCP server. Use when someone asks "where do my requests run", "make sure nothing leaves the EU", "only zero-retention providers", "block US providers", "block provider X", or "what is our sovereignty setting".
---

# Set the workspace policy

You are the person's own agent, connected to Stav's MCP server. The
workspace policy is a wall: Stav refuses every Provider Endpoint outside it,
for `model="auto"`, for a Router and for a named model alike. A Router can
make the rule stricter for its own traffic; it can never loosen it.

## Rules

1. **Read first, then show the change, then ask.** Never write a setting
   the person has not seen in a diff.
2. **Say when a change loosens the rule.** Turning ZDR off, widening the
   region, lowering the level or unblocking a provider lets more endpoints
   serve. Say that in one line before the yes.
3. **Say what a stricter rule costs.** Fewer endpoints can mean fewer
   models, higher prices or a model the app uses today that no longer
   serves. Check before you write (step 3).
4. **Owner or Admin only.** A Member can read the policy, not change it. If
   a write is refused, say who must do it and where: Your workspace →
   Govern → Key policies (https://stav.ai/app/govern/policies).

## The settings

| Setting | Tool and field | Values | Plain words |
|---|---|---|---|
| Compute region | `smartrouter_update_config` · `required_compute_region` | `"eu"`, `"eea"`, `null` | Where the model runs. Unknown fails closed. |
| Sovereignty level | `smartrouter_update_config` · `min_sovereignty_level` | `L0`–`L4`, `null` | L1 = EU/EEA location · L2 = + certification · L3 = EU/EEA ownership and control · L4 = full |
| Zero data retention | `smartrouter_update_config` · `require_zdr` | `true` / `false` | Only providers that keep nothing. Unknown never qualifies. |
| No training | `smartrouter_update_config` · `require_no_training` | `true` / `false` | On by default. Only providers that do not train on input. |
| Blocked countries | `smartrouter_update_config` · `blocked_provider_countries` | ISO codes, e.g. `["US", "CN"]` | Never a provider from these jurisdictions. |
| Blocked providers | `teamkeys_policy_update` · `blocked_provider_ids` | Inference Provider ids | Never these providers. Replaces the whole list. |

`smartrouter_get_config` reads the first five. `teamkeys_policy_get` reads
the blocked providers and lists every provider (`providers`: id, slug,
name) to pick from.

## The flow

### 1. Read

Call `smartrouter_get_config` and `teamkeys_policy_get`. Tell the person
the current rule in at most five lines, in plain words.

### 2. Ask what they want

One message. Offer the common shapes if they are unsure:

- **EU, no training** — `required_compute_region: "eu"`,
  `require_no_training: true`.
- **EU, zero retention** — the above plus `require_zdr: true`.
- **EU-owned** — the above plus `min_sovereignty_level: "L3"`.

### 3. Check the effect (free)

Call `models_shortlist` with the new rule (`runs_in`, `min_rung`, `zdr`,
`no_training`) and no other filter, and once with the old rule. Say how
many models serve under each. If the person names models their apps use,
add them to `explain` and say which stop serving.

### 4. Write

Show the diff as `setting: old → new`. On a yes, call
`smartrouter_update_config` with only the fields that change, and
`teamkeys_policy_update` only if blocked providers change (send the whole
new list). Read back with the two get tools and confirm.

The change is in the workspace audit log (Your workspace → Govern → Audit
log) with what moved.

## Finish

Two lines: the rule now, and how many models serve under it.
