# Multiplayer Agent for Claude Code

Make any browser game multiplayer from Claude Code, and host it at a share link a friend can open. This plugin connects Claude Code to the hosted Multiplayer Agent backend, which provides rooms, shared real-time state, player identity, saved progress, leaderboards and game hosting, so Claude never has to write or run a game server. It also adds a `make-it-multiplayer` skill that teaches Claude how to design, wire, test and publish a multiplayer game on it.

No account or key is needed to start. Claude asks for your email and opens a page where you sign up or log in (your password is typed on the website, never in the chat), so your games are saved to your own account.

## Install

Type these in Claude Code:

```
/plugin marketplace add multiplayeragent/multiplayer-agent
/plugin install multiplayer-agent@multiplayer-agent
```

If the tools don't show up straight away, type `/reload-plugins`. Then ask Claude something like "build me a multiplayer snake game" or "make this game multiplayer".

## What it includes

- **MCP server** `multiplayer-agent` at `https://api.multiplayeragent.com/mcp` (remote, streamable HTTP, no key required).
- **Skill** `make-it-multiplayer`, used automatically when you ask for multiplayer, co-op, a lobby, an invite link, a leaderboard or hosting.

## Data and privacy

The plugin sends requests to `api.multiplayeragent.com`: the game definitions, room settings and game files Claude creates or publishes for you, plus the email you choose to connect. Games you publish are served from `*.play.multiplayeragent.com`, and players' game state goes to the same backend while they play. See the [privacy policy](https://multiplayeragent.com/privacy) and [terms](https://multiplayeragent.com/terms).

## Links

- How the connection works: https://multiplayeragent.com/mcp
- SDK reference: https://multiplayeragent.com/docs
- Pricing (free tier, no card): https://multiplayeragent.com/pricing
