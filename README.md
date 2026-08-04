# RankControl

[RankControl](https://rctrl.com) wins Google rankings and AI citations for your
site: seven AI agents plan, write, and publish citation-optimized articles as
native posts on your own CMS (WordPress, Webflow, Shopify, Ghost, Notion, Wix,
Framer, and headless). They watch citations and rankings, refresh pages that
slip, draft link outreach for your approval, and capture intent-scored leads
from Google and AI answers.

This repo is the RankControl **agent skill**. It teaches Claude Code, Cursor,
Codex, and any [skills](https://skills.sh)-compatible agent how to run a
RankControl workspace from the command line: AI visibility, content planning
and publishing, leads, backlinks, repurposing, and reports.

```bash
npx skills add rankcontrol/rankcontrol
```

Prefer MCP? The same package is an MCP server:

```bash
claude mcp add rankcontrol -- npx rankcontrol mcp
```

Safety: publishing, outreach, and other publicly visible actions are dry-run
by default and require `--confirm`. Approval gates run server-side.

## Links

- Website: [rctrl.com](https://rctrl.com)
- Docs: [rctrl.com/docs](https://rctrl.com/docs)
- npm: [rankcontrol](https://www.npmjs.com/package/rankcontrol) (CLI + MCP server in one package)
- X: [@rctrlcom](https://x.com/rctrlcom)
- Support: [rctrl.com/contact](https://rctrl.com/contact)
