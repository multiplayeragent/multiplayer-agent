# Installing the Multiplayer Agent MCP server (for agents: Cline, Cursor, others)

This is a **remote** MCP server. There is nothing to download, build or run locally, and no API key is needed to start.

- Server URL: `https://api.multiplayeragent.com/mcp`
- Transport: Streamable HTTP
- Auth: none required. Optional `Authorization: Bearer sk_...` header for a studio secret key from https://multiplayeragent.com/dashboard. Never ask the user to paste a password.

## Cline

Add this entry to `cline_mcp_settings.json` (MCP Servers → Configure → "Configure MCP Servers"):

```json
{
  "mcpServers": {
    "multiplayer-agent": {
      "type": "streamableHttp",
      "url": "https://api.multiplayeragent.com/mcp?src=cline",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Or use MCP Servers → Remote Servers: name `multiplayer-agent`, URL `https://api.multiplayeragent.com/mcp?src=cline`, transport Streamable HTTP.

## Any other MCP client

Add a remote (HTTP) server named `multiplayer-agent` with URL `https://api.multiplayeragent.com/mcp`. No command, no environment variables.

## Check it worked

The server lists tools such as `list_games`, `create_game`, `get_sdk_reference` and `publish_game`. Call `list_games`: with no key it opens an unclaimed studio (lasts 72 hours) and returns an empty list. To keep games, call `connect_account` with the user's email and open the returned link for them.

Then ask for a game, for example "build me a multiplayer snake game and give me a share link".
