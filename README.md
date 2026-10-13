# Coachio Claude Plugins

A [Claude Code](https://code.claude.com) plugin marketplace for Coachio.

It currently ships one plugin, **`coachio-system`**, which installs the skill that
teaches Claude Code how to drive the **Coachio Funnel Config MCP** server — so admins
can configure sales funnels by chatting with the agent instead of clicking through the
dashboard (landing / checkout / success pages, emails, email campaigns, leads,
discounts, the media library and analytics).

> This repo contains **only** the plugin and skill files — no backend code. The MCP
> server itself runs on your Coachio deployment; this plugin just gives the agent the
> instructions to use it.

## Install

### 1. Add the marketplace and install the plugin

```bash
/plugin marketplace add sonlovinbot/Coachio-Claude-Plugins
/plugin install coachio-system@coachio
```

`/plugin marketplace add` also accepts the full git URL:
`https://github.com/sonlovinbot/Coachio-Claude-Plugins.git`

### 2. Register the MCP server

The plugin installs the **skill** (how to use the tools). You still connect Claude Code
to your MCP server, which needs your deployment URL and admin secret. Get the exact
command — with the secret pre-filled — from the **System Admin → MCP** page in your
Coachio dashboard, or run:

```bash
claude mcp add --transport http coachio-system \
  "https://<your-api-host>/api/v1/mcp/" \
  --header "X-Admin-MCP-Secret: <YOUR_ADMIN_MCP_SECRET>"
```

> The secret is per-deployment and must not be committed here, which is why server
> registration is a separate step from the public skill plugin.

### 3. Verify

```bash
/mcp                # confirms coachio-system is connected and lists the tools
claude mcp list     # coachio-system: Connected
```

Then try a prompt such as *"list funnels"*.

## What's in the plugin

```
plugins/coachio-system/
├── .claude-plugin/plugin.json
└── skills/coachio-system/
    ├── SKILL.md                 # how the agent uses the tools (scope + safety policy)
    └── references/tool-catalog.md
```

## Safety

Destructive tools (delete, send, status changes) require explicit confirmation. The
skill instructs the agent never to leak the admin secret and to respect each tool's
applicability scope. See `SKILL.md` for the full policy.

## License

Internal use for Coachio. © Coachio.
