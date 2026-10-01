# Briefcase for Claude and ChatGPT

Connect your AI assistant to [Briefcase](https://briefcase.so). The assistant acts as you, on the clients you choose, with your Briefcase permissions. Every change it makes is recorded in Briefcase under Settings → AI assistants.

Your Briefcase workspace needs assistant connections enabled. Ask your Briefcase contact if the connect page says it is not enabled.

## Claude (claude.ai and the desktop app)

1. Open this link: [Add Briefcase to Claude](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Briefcase&connectorUrl=https%3A%2F%2Fapi.briefcase.so%2Fmcp)
2. Press **Add**, then **Connect**. Sign in to Briefcase, choose your clients and approve.
3. Start a chat and ask: *Check my Briefcase connection.*

Once Briefcase is listed in the Claude connectors directory you can also find it under **Settings → Connectors → Browse**.

## Claude Code and Cowork

```
/plugin marketplace add Briefcase-AI/briefcase-plugin
/plugin install briefcase@briefcase
/mcp
```

Choose Briefcase under `/mcp` to sign in. The plugin adds seven skills: transaction review, document submission, financial close review, ledger reports, sales invoicing, manual journals and expense claims.

## ChatGPT

Open [Briefcase in the ChatGPT plugin directory](https://chatgpt.com/plugins/plugin_asdk_app_6abd0fee216c819191349465958a49d5) and press **+**, or search for Briefcase under **Plugins**. Sign in to Briefcase, choose your clients and approve.

## Codex

```
codex plugin marketplace add Briefcase-AI/briefcase-plugin
```

Then run `/plugins` inside Codex, install Briefcase and start a new session.

## What this repository contains

- `plugins/briefcase/.claude-plugin/plugin.json` and `.mcp.json` — Claude plugin manifest and server declaration
- `plugins/briefcase/plugin.json` and `mcp.json` — the same plugin in the portable format used by ChatGPT and Codex
- `plugins/briefcase/skills/` — workflow skills shared by both
- `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json` — marketplace catalogues for Claude Code and Codex

The server at `https://api.briefcase.so/mcp` enforces every permission; the skills only guide the assistant. Support: support@briefcase.so.
