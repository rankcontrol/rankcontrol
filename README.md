# rankcontrol

The [RankControl](https://rctrl.com) skill for AI agents. Teaches Claude Code,
Cursor, Codex, and any [skills](https://skills.sh)-compatible agent how to run
a RankControl workspace from the command line: AI visibility, content
planning and publishing, leads, backlinks, repurposing, and reports.

```bash
npx skills add r-sri-ram/rankcontrol
```

Prefer MCP? The same package is an MCP server:

```bash
claude mcp add rankcontrol -- npx rankcontrol mcp
```

- Docs: https://rctrl.com/docs
- npm: https://www.npmjs.com/package/rankcontrol
- Safety: publishing, outreach, and other publicly visible actions are dry-run
  by default and require `--confirm`. Approval gates run server-side.
