# AppSkyline Agent Skills

[Agent Skills](https://agentskills.io) for
[AppSkyline](https://www.appskyline.com/) — App Store Optimization across the
iOS App Store, macOS App Store, Google Play, and the Microsoft Store: keyword
rank tracking, live search results, search-volume and difficulty research, and
store-listing metadata audits.

## Install

```bash
npx skills add appskyline/skills
```

Works with any agent that supports the Skills standard — Claude Code, Cursor,
Codex, Gemini CLI, GitHub Copilot, OpenCode, and
[others](https://agentskills.io/clients). Update later with
`npx skills update`.

To install a single skill:

```bash
npx skills add appskyline/skills@appskyline
npx skills add appskyline/skills@aso-research
```

## Skills

| Skill                                        | Backed by                                                       | Use it for                                                                                          |
| -------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| [`appskyline`](skills/appskyline/SKILL.md)   | the [`appskyline` CLI](https://github.com/appskyline/cli) (npm) | Rankings, keyword research, metadata audits, and reviews from the shell — `npx appskyline`          |
| [`aso-research`](skills/aso-research/SKILL.md) | the AppSkyline MCP connector (`https://mcp.appskyline.com/mcp`) | End-to-end ASO research through MCP tools in hosts like Claude, ChatGPT, Cursor, and Codex |

Both need an AppSkyline account with at least one tracked app —
[sign up](https://app.appskyline.com), then `appskyline login` (CLI) or
connect the MCP server (OAuth).

## Other entrypoints

- **CLI** — `npm install -g appskyline` or `npx appskyline`, source at
  [appskyline/cli](https://github.com/appskyline/cli).
- **MCP connector** — `https://mcp.appskyline.com/mcp` (Streamable HTTP,
  OAuth). Listed in the Claude Connectors Directory and the
  [Official MCP Registry](https://registry.modelcontextprotocol.io); setup
  for every MCP client at
  [appskyline.com/developers](https://www.appskyline.com/developers/#mcp-clients).
- **Claude Code plugin** — the connector bundled with the `aso-research`
  skill: [appskyline/claude-plugin](https://github.com/appskyline/claude-plugin).
- **REST API** — API keys and reference at
  [appskyline.com/developers](https://www.appskyline.com/developers/).

## License

[Apache-2.0](LICENSE)
