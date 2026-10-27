# LovinBot AI ExpertOS Plugins

A [Claude Code](https://code.claude.com) plugin marketplace for ExpertOS (marketplace name: `lovinbot`).

It currently ships one plugin, **`lovinbot-expertos`**, which installs the skill that
teaches Claude Code how to drive the **Lovinbot ExpertOS Funnel Config MCP** server — so admins
can configure sales funnels by chatting with the agent instead of clicking through the
dashboard (landing / checkout / success pages, emails, email campaigns, leads,
discounts, the media library and analytics).

> This repo contains **only** the plugin and skill files — no backend code. The MCP
> server itself runs on your Lovinbot ExpertOS deployment; this plugin just gives the agent the
> instructions to use it.

## Install

### 1. Add the marketplace and install the plugin

```bash
/plugin marketplace add LovinBot-AI/LovinBot-AI-ExpertOS-Plugins
/plugin install lovinbot-expertos@lovinbot
```

`/plugin marketplace add` also accepts the full git URL:
`https://github.com/LovinBot-AI/LovinBot-AI-ExpertOS-Plugins.git`

To update later: `/plugin update lovinbot-expertos@lovinbot`.

### 2. Register the MCP server

The plugin installs the **skill** (how to use the tools). You still connect Claude Code
to **your workspace's** MCP server. The **AI & MCP → Admin MCP** page in your admin
dashboard shows the exact URL and command.

**Sign in (recommended, no key):**

```bash
claude mcp add --transport http lovinbot-expertos "https://<your-site>/api/v1/mcp/"
```

Then type `/mcp`, pick `lovinbot-expertos` and choose **Authenticate**. Your browser opens
the sign-in window; sign in with your admin account and click **Allow**.

**Or use a personal API key:** on the Admin MCP page open the connect guide, pick
**Claude Code**, click **Use an API key instead** and create a key. Copy the key (`xos_…`); it is shown only once. Then run:

```bash
claude mcp add --transport http lovinbot-expertos \
  "https://<your-site>/api/v1/mcp/" \
  --header "X-Admin-MCP-Secret: <YOUR_PERSONAL_KEY>"
```

> Access is personal and per-workspace: the agent acts as you, with your role, on that
> workspace only. Disconnect an app or revoke a key any time from the Admin MCP page.
> Claude.ai / ChatGPT users don't need the plugin: add the URL as a custom connector
> named "Lovinbot ExpertOS" and sign in.

> **Upgrading from `coachio-system`:** run `/plugin uninstall coachio-system@coachio` and
> `claude mcp remove coachio-system`, then follow the steps above.

### 3. Verify

```bash
/mcp                # confirms lovinbot-expertos is connected and lists the tools
claude mcp list     # lovinbot-expertos: Connected
```

Then try a prompt such as *"list funnels"*.

## What's in the plugin

```
plugins/lovinbot-expertos/
├── .claude-plugin/plugin.json
└── skills/lovinbot-expertos/
    ├── SKILL.md                 # how the agent uses the tools (scope + safety policy)
    └── references/tool-catalog.md
```

## Safety

Destructive tools (delete, send, status changes) require explicit confirmation. The
skill instructs the agent never to leak your personal key and to respect each tool's
applicability scope. See `SKILL.md` for the full policy.

## License

Internal use for Lovinbot ExpertOS. © LovinBot AI.
