---
name: make-it-multiplayer
description: Make a browser game multiplayer, or build a new multiplayer web game, on the hosted Multiplayer Agent backend, then host it at a share link. Use when the user wants players to play together online, with friends, in the same room or lobby, or to sync players, add co-op or versus, an invite or share link, leaderboards, saved progress or player accounts, instead of writing and running their own game server.
when_to_use: Trigger on "make it multiplayer", "add multiplayer", "online multiplayer", "play with friends", "co-op", "versus mode", "lobby", "invite link", "share the game", "host my game", "sync players", "real-time game", "leaderboard", "multiplayer game from scratch", or "use Multiplayer Agent".
---

# Make it multiplayer

You have a hosted multiplayer backend through the `multiplayer-agent` MCP server. It provides rooms, shared state, player identity, saved progress, leaderboards and hosting, so there is no game server for you to write, run or deploy. Do not write a socket server, a relay, or a backend for the user: use these tools.

## 0. Check the tools are there

The tools are named `mcp__plugin_multiplayer-agent_multiplayer-agent__<tool>` (for example `..._create_game`). If you can't see them, the plugin was installed during this chat, and its tools only load when a new chat starts (`/reload-plugins` and `/mcp` don't bring them in). Stop there and reply with just this, adapted to where they are:

- In an editor (VS Code, Cursor, JetBrains): "Start a new Claude chat and say: make my game multiplayer."
- In a terminal: "Type `/exit`, then run `claude --continue` to pick this chat back up with the tools loaded."

Don't plan the game, read the website or research pricing and limits while the tools are missing: the tools' replies and `get_sdk_reference` carry everything you need. Nothing else is needed either: no key, no account.

## 1. Connect the user's account early, in one question

The connection starts without an account, in an unclaimed studio that only this session can reach. Save the user's work to their own account up front:

1. Ask once: "What email should I save your games under?"
2. Call `connect_account` with that email. It returns a `url` and a `code`.
3. Open the url in their browser yourself: Windows `start "" "<url>"`, macOS `open "<url>"`, Linux `xdg-open "<url>"`.
4. Tell them: "Sign up or log in on that page, check it shows the code `<code>`, then press Connect."
5. Keep building while they do it. When they say it's done, call `list_games`: it shows `connectedAs` once connected.

Never ask for a password in the chat, and refuse one if offered: passwords are only typed on that page. If the link expires (10 minutes), call `connect_account` again. If the user would rather not connect, carry on: the replies to `create_game` and `publish_game` carry a claim link (`studio.claimUrl`) that keeps the games later. Unclaimed studios expire after 72 hours.

## 2. Read the reference before writing client code

1. `create_game` with a clear, human name (or `list_games` to reuse one). Pass its publishable key (`pk_…`) as `game` on every later call once there is more than one game.
2. `get_sdk_reference` and read it fully. It has working snippets wired to THIS game's key and endpoints. Copy the endpoint, `apiUrl` and key from it or from `get_connection_info`. Never type them from memory.
3. For a quick start, `get_client_snippet` returns a complete single-file multiplayer page wired to a room.

## 3. Design the state before the code

Decide, per piece of game state, who owns it:

- **Per-player** (position, name, color, score, input): each client writes only its OWN entity with `me.set(...)`. A client cannot write another player's entity.
- **Room-wide** (round timer, level, whose turn, pickups): `setShared(...)`, written by one agreed client (the host), not by everyone.
- **One-off happenings** (shot fired, hit, chat line): events with `mp.send` / `mp.on`, so they are never lost between state updates.
- **Must not be cheatable** (scores that matter, currency, hidden cards): authoritative server logic (`deploy_room_logic`). The server computes; clients only send input. It needs the account connected in step 1, which is one more reason to ask for the email up front. Don't turn it into a separate question: if they haven't connected when you get there, build the rest, then say the cheat-proof part goes live once they press Connect on that page.

Keep room state lean: it is sent to everyone many times a second. Writes past the documented limits are dropped silently, so pack big grids into fewer fields (one string per row, not per cell).

For an existing game: read its code first, find the game state and the game loop, and change as little as possible. Local input still drives the local player; remote players are drawn from room state.

## 4. Write the client

- Load the SDK from its absolute URL, and pass both the realtime `endpoint` and `apiUrl` to `MP.join`. Without `apiUrl` identity, saves, leaderboards and analytics silently do nothing.
- Leave cleanly, or every reload leaves a ghost player for ~30 seconds: `window.addEventListener("pagehide", () => mp.leave())`.
- Show "reconnecting…" from `mp.onDisconnect` / `mp.onReconnect` instead of freezing.
- For moving things, interpolate remote players between updates and predict the local player; the reference has the pattern.
- Wrap risky code (asset loading, storage) in try/catch and report handled failures with `mp.track("error", { message, where })`.
- Never put a secret key (`sk_…`) in client code. The publishable key is safe to ship.

## 5. Test like two real players

Open the game in TWO DIFFERENT browsers, or one normal and one private window. Two tabs of the same browser share one saved player, which hides identity bugs. Check: both players appear, movement or turns sync both ways, a reload doesn't leave a ghost, and the second player joining mid-game sees the current state.

## 6. Host it and hand over the link

Call `publish_game` with every file the game needs: `index.html` at the root plus any JS, CSS and assets (text as plain content, images and audio as base64 with `encoding: "base64"`). The files must use this game's publishable key. It returns the share link, `https://<slug>.play.multiplayeragent.com/`, the game's own address. Re-run it after each change; the link stays the same.

Finish by giving the user the share link ("send this to a friend and play together"), plus the claim link if they didn't connect an account.

## Common mistakes

- Writing a custom server or socket code. The backend already does it.
- Missing `apiUrl` in `MP.join`, or a localhost URL in a hosted page.
- Every client writing shared state, so it flickers between versions. One host writes it.
- Pasting a key from another game. Every tool response has a `target` block: check it names the game you mean.
- Testing in two tabs of one browser and calling it two players.
- Previewing in an artifact or sandboxed iframe, where WebSockets are blocked. Open a real page or use the share link.

## What it is not for

Competitive twitch shooters or racing that need rollback netcode, native Unity/Unreal builds, and huge shared worlds. It suits turn-based, casual, co-op, party and .io-style browser games.
