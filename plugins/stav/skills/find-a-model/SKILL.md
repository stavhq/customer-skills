---
name: find-a-model
description: Turn a task a person describes ("answer support emails in Norwegian, EU only, cheap") into a short, explained list of Stav models and, on request, a working Router — through Stav's MCP server, without ever sending the task text to Stav. Use when someone asks "which model should I use for X", "find me a model that…", "get me off GPT-4o", "why is model Y not on the list", or "build a Router for this".
---

# Find a model

You are the person's own agent, connected to Stav's MCP server. Stav never
reads the task. **You** read it, turn it into structured needs, and ask the
catalogue. Every answer you give is a **catalogue ranking** — recorded
facts and benchmark-derived scores — not what Stav's Smart Router would
pick for a live request.

## Rules

1. **Never send the task text to Stav.** No field on any Stav tool takes
   prose. Turn what the person said into the fields below. If a need has no
   field, say so instead of stuffing it into one.
2. **Say "catalogue ranking".** When you present a list, say it comes from
   catalogue facts. For the real pick, offer `smartrouter_preview_route` or
   `routers_try`, and say first that those send a sample request to Stav.
3. **Not measured is not zero.** A model with no score on a criterion is
   left out of a minimum, scores nothing on it, and sorts last. Say "not
   measured" when the reason says so; never guess a value.
4. **A Router is created only after the person says yes** to the config you
   show them. Never make it the workspace default unless they ask.
5. **Keep it short:** 3–5 models, one line each on why, the price for their
   task, where it runs.

## The flow (≤ 6 tool calls)

### 1. Ask for the needs you cannot infer

From the task, fill what you can; ask for the rest in ONE message:

| Need | Field on `models_shortlist` | Examples |
|---|---|---|
| What kind of work | `use_case` | `customer_support_chatbot`, `code_review`, `structured_extraction`, `summarisation`, `translation`, `agentic_coding` |
| What it must do | `capabilities` | `tool_calling`, `structured_output`, `json_mode`, `document_understanding` (PDF), `vision_input`, `audio_input`, `extended_thinking` |
| How good, per axis | `strength` | `{"coding": 70}`, `{"multilingual": 60}` — axes: reasoning, coding, instruction_following, multilingual, long_context, safety |
| Where it must run | `runs_in`, `min_rung`, `zdr`, `no_training` | `"eu"`, `"eea"`, `"NO"`; `L3`; `true` |
| Price | `max_price` + `price_unit` | `1.0` per 1M input tokens (team currency) |
| How much | `volume` | requests per month, input and output tokens per request |
| What matters | `priorities` | `{quality, cost, speed}` summing to 100 |
| A model to explain | `explain` | `["meta/llama-4-maverick"]` |

Chat models carry no language list; for Norwegian, Swedish or German ask
for `strength.multilingual` rather than `languages` (which is a hard rule
only on embedding and transcription models).

### 2. `models_shortlist`

One call. Read `models[].reasons` (structured: strength + percentile,
use-case fit and where it came from, price per 1M in the team currency,
tokens per second, compute countries / rung / ZDR / no-training), and
`estimated_cost` when a volume was given. Present 3–5 lines:

> **mistral/mistral-small-4** — Norwegian strength 71 (top 20%), fits
> customer support (curator), runs in SE and FR, zero retention,
> ≈ €0.0007 per email → €0.70 per 1,000. Not measured: speed.

Then the exclusions, from `excluded`: "41 models had no EU endpoint, 12
had no tool calling."

### 3. Why not that one?

If the person asks about a model, call `models_shortlist` again with the
same needs and `explain: ["<slug>"]`. Read `why_not[].rule` and `detail`:
the FIRST rule that removed it (`endpoint:runs_in`, `capability:tool_calling`,
`strength:coding`, `use_case`, `not_servable`, `not_in_catalogue`).

### 4. Depth, on request

`public_model_scorecard` for one model's full benchmark card,
`public_models_compare` for two or more side by side, `models_get` for
every endpoint fact. These are public catalogue reads — no sign-in, no
MCP switch.

### 5. A Router, on request

Call `routers_suggest` with the SAME needs. It returns an **unsaved**
`config`: the shortlist as `allowed_models`, plus only the fields the
gateway enforces (`required_capabilities`, `require_zdr`,
`require_no_training`, `required_compute_region`, `min_sovereignty_level`,
`objective_preset`). `not_expressible` lists what the Router cannot name
by itself — the allow-list carries it; say so.

Show the config. On the person's yes: `routers_create` with it. The
answer carries `portal_url` — hand it over. A new Router is live at once
under its own new slug; nothing routes to it until something names it as
`model="@workspace/<slug>"`.

To change an **existing** Router: `routers_draft_set` → `routers_try`
(target `draft`) → `routers_publish`. Never overwrite it blind. The
published version keeps serving until publish.

`routers_set_default` only when the person asks for it, in those words.

### 6. One test call, on request

`inference_chat` with `model: "@workspace/<slug>"` and a short message,
so the person sees it answer. Say that this one call is billed to the
workspace wallet.

## Worked examples

**"Answer customer emails in Norwegian, with tools, EU compute, ZDR,
cheapest that is good enough."**
`models_shortlist` with `use_case: customer_support_chatbot`,
`capabilities: ["tool_calling"]`, `strength: {"multilingual": 55}`,
`runs_in: "eu"`, `zdr: true`, `priorities: {quality: 40, cost: 50, speed: 10}`,
`volume: {requests_per_month: 1000, input_tokens_per_request: 800,
output_tokens_per_request: 300}`. Then `routers_suggest` with the same
needs, confirm, `routers_create`.

**"I use gpt-4o. Find EU-sovereign models of about the same quality for
less."** `public_models_comparable` for `openai/gpt-4o`, then
`models_shortlist` with `runs_in: "eu"`, `strength: {"reasoning": <gpt-4o's
reasoning strength − 10>}`, `priorities: {quality: 50, cost: 50, speed: 0}`,
`explain: ["openai/gpt-4o"]`. Present the quality gap and the cost gap.

**"A model for Claude Code / Codex: long context, tools, strong at
engineering."** `models_shortlist` with `capabilities: ["tool_calling"]`,
`strength: {"coding": 70, "long_context": 60}`, `sort` is the ranking;
then `routers_suggest` for a cheap-first Router, and say a second Router
with `highest_quality` is the way to split easy and hard steps.

**"Read PDF invoices and return JSON."** `models_shortlist` with
`use_case: structured_extraction`, `capabilities:
["document_understanding", "structured_output"]`, `priorities: {quality: 30,
cost: 60, speed: 10}`.

**"Why is Llama X not on the list?"** The same needs, `explain: ["<slug>"]`.
