# RankControl

[RankControl](https://rctrl.com) wins Google rankings and AI citations for your
site: seven AI agents plan, write, and publish citation-optimized articles as
native posts on your own CMS (WordPress, Webflow, Shopify, Ghost, Notion, Wix,
Framer, and headless). They watch citations and rankings, refresh pages that
slip, and draft link outreach for your approval.

This repo is the RankControl **Claude Code plugin** and **agent skill**. It
teaches Claude Code, Cursor, Codex, and any [skills](https://skills.sh)-compatible
agent how to run a RankControl workspace from the command line: AI visibility,
content planning and publishing, backlinks, repurposing, and reports.

Claude Code, as a plugin:

```bash
/plugin marketplace add anthropics/claude-plugins-community
/plugin install rankcontrol@claude-community
```

Any skills-compatible agent:

```bash
npx skills add rankcontrol/rankcontrol
```

Prefer MCP tools? Local stdio via the same npm package, or the remote
connector with OAuth:

```bash
claude mcp add rankcontrol -- npx rankcontrol mcp
claude mcp add --transport http rankcontrol https://api.rctrl.com/mcp
```

Safety: publishing, outreach, and other publicly visible actions are dry-run
by default and require `--confirm`. Approval gates run server-side.

## Links

- Website: [rctrl.com](https://rctrl.com)
- Docs: [rctrl.com/docs](https://rctrl.com/docs)
- npm: [rankcontrol](https://www.npmjs.com/package/rankcontrol) (CLI + MCP server in one package)
- X: [@rctrlcom](https://x.com/rctrlcom)
- Support: [rctrl.com/contact](https://rctrl.com/contact)
