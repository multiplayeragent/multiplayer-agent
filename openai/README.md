# OpenAI plugin (ChatGPT + Codex)

`multiplayer-agent/` is the package submitted to the OpenAI plugin directory (shared by ChatGPT and Codex). It holds the same `make-it-multiplayer` skill as the Claude Code plugin, adapted for ChatGPT and Codex, plus the remote MCP server with no authentication (`mcp.json`, tagged `?src=openai`).

Build the ZIP (`plugin.json` at the root of the archive):

```sh
cd openai/multiplayer-agent && zip -r ../multiplayer-agent-openai.zip . -x '.*'
```

Bump `version` in `plugin.json` for every new ZIP. MCP tool changes on the server go live without a new ZIP; skill and listing changes need one.

Before the first submission, add `extensions.com.openai.review.demo_recording_url` (a video of a positive test case) to `plugin.json`.
