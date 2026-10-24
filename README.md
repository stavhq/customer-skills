# Stav AI skills

Skills for building on Stav AI, the EU-sovereign inference platform.

> **Generated — do not edit here.** The source is `skills/customer/` in
> `stavhq/stav-code`. Change a skill with a pull request there; CI publishes
> it here on merge. Version 1.0.5 (stav-code 0681c21c0).

## Install

**claude.ai, Desktop, Cowork:** Customize → Plugins → add marketplace
`stavhq/customer-skills`, then install `stav`. Press *Check for updates* to pull a
new version.

**Claude Code:**

```bash
claude plugin marketplace add stavhq/customer-skills
claude plugin install stav@customer-skills
```

Turn on auto-update for this marketplace in `/plugin` → Marketplaces, or
Claude Code keeps the version you installed.

The plugin also connects the Stav MCP server (`https://stav.ai/mcp`). Sign in
in your browser when your agent asks.

**Codex CLI:**

```bash
codex plugin marketplace add stavhq/customer-skills
codex plugin add stav@customer-skills
codex mcp login stav
```

**Cursor:** add the marketplace `stavhq/customer-skills` and install `stav`.

**Other agents:** add `https://stav.ai/mcp` as a remote MCP server, then install
the skills with the skills CLI:

```bash
npx skills add https://github.com/stavhq/customer-skills/tree/main/plugins/stav/skills
```

Easiest of all: paste the setup prompt from
https://stav.ai/docs/getting-started/agent-setup into your agent.

## Skills

- `cost-review`
- `find-a-model`
- `move-to-stav`
- `set-policy`
