---
name: move-to-stav
description: Move an app, or one feature of it, from OpenAI, Anthropic, Azure or another provider onto Stav — set where its requests may run, pick models inside that rule, make a scoped key with a spend cap, prove one call, and change the code. Through Stav's MCP server. Use when someone says "move this app to Stav", "switch our OpenAI calls to Stav", "get this feature off US providers", "make this run in the EU", or "how do I use Stav in this codebase".
---

# Move to Stav

You are the person's own agent, connected to Stav's MCP server, usually
inside the codebase you are moving. Stav is a gateway: the app keeps its
OpenAI or Anthropic SDK and changes a base URL, a key and a model name.

## Rules

1. **Policy before models.** Set where requests may run before you pick a
   model. Never suggest `model="auto"` before the policy is set.
2. **Every change needs a yes.** Show what you will create or change, then
   wait. That covers the policy, the key, the cap and every code edit.
3. **Say what costs money.** Reads are free. The test call in step 5 is
   billed to the workspace wallet; say so before you make it.
4. **Never put a key in the repository.** The person stores it in their
   secret store or environment. Show it once.
5. **Stav is the processor.** The customer's data processing agreement is
   with Stav (https://stav.ai/legal/dpa). Never tell them to sign with an
   Inference Provider.
6. **Send no app content to Stav** except the one short test message.

## The flow

### 1. Read the app (no tool calls)

Find every place the app calls a model: the SDK, the base URL, the model
names, streaming, tools, JSON output, images or PDFs, embeddings. List them
in a short table: file, model, what it needs. Ask which ones to move if
the answer is not "all".

### 2. Ask for the rule (one message)

Ask only what you cannot infer:

- Where may requests run? `eu`, `eea`, or a country.
- Minimum sovereignty level, L0–L4 (L1 = EU/EEA location; L3 = EU/EEA
  ownership and control; L4 = full).
- Zero data retention required?
- No training on the data? (On by default.)

### 3. Set the policy

`smartrouter_get_config` reads the workspace rule. If it already says
what the person wants, say so and go on. Otherwise show the change and,
on a yes, call `smartrouter_update_config` with only the fields that move:
`required_compute_region`, `min_sovereignty_level`, `require_zdr`,
`require_no_training`, `blocked_provider_countries`. Only an Owner or
Admin can change it; if the call is refused, say who must do it and stop.

### 4. Pick models inside the policy

For each model the app uses today, one `models_shortlist` call with the
same rule (`runs_in`, `min_rung`, `zdr`, `no_training`) and the needs from
step 1 (`capabilities`), plus `explain: ["<today's model>"]` so the person
sees whether today's model fits. Offer 2–3 models per use, one line each:
why, price per 1M tokens, where it runs. Let the person choose.

Today's model may fit: an OpenAI or Anthropic model can be served through
Stav under the same policy. Say so when it does.

### 5. A key, a cap, one call

On a yes:

1. `apikeys_create` — `name` after the app, `environment: "test"` for the
   first run, `scopes` only for what it uses (`inference:chat` for chat),
   `expires_at` within 30 days for a test key.
2. `budget_set` — `scope_type: "key"`, `scope_id` = the new key's id, a
   small `limit` and `period: "day"`.
3. Show the key once. Ask the person to store it, not you.
4. `inference_chat` with the chosen model, one short message, and
   `api_key` = the new key. Then `request_routing_get` with the request id:
   where it ran, which Inference Provider, why, and what it cost.

### 6. The code change

Change only what you listed in step 1:

- OpenAI SDK: `base_url="https://api.stav.ai/v1"`, key from the
  environment, model slug from step 4.
- Anthropic SDK: `base_url="https://api.stav.ai"` (the SDK adds `/v1`),
  same key, same model slug.
- Keep timeouts, retries and streaming as they were.

Show the diff. Run the app's tests if it has them. Tell the person the
two things left to do: a `live` key for production with its own cap, and
where to see requests (Your workspace → Monitor → Requests,
https://stav.ai/app/monitor/requests).

## Finish

Three lines: what moved, where it runs now, what it cost to test.
