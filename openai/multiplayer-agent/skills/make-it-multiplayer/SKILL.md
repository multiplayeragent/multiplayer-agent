---
name: make-it-multiplayer
description: Make a browser game multiplayer, or build a new multiplayer web game, with the Multiplayer Agent tools, then host it at a share link. Use when the user wants players to play together online, with friends, in the same room or lobby, or asks to sync players, add co-op or versus, an invite or share link, a leaderboard, saved progress or player accounts, instead of writing and running their own game server. Trigger on "make it multiplayer", "add multiplayer", "online multiplayer", "play with friends", "co-op", "versus mode", "lobby", "invite link", "share the game", "host my game", "leaderboard", "multiplayer game from scratch".
---

# Make it multiplayer

The Multiplayer Agent tools give you a hosted multiplayer backend: rooms, shared state, player identity, saved progress, leaderboards and hosting. There is no game server for you to write, run or deploy. Do not write a socket server, a relay or a backend: use the tools.

Explicit instructions from the user take priority over this guide.

## 1. Offer to save the games to the user's account, once

The tools start without an account, in a temporary studio that lasts 72 hours. To keep the games:

1. Ask once: "What email should I save your games under?" If the user would rather not say, carry on without it.
2. Call `connect_account` with that email. It returns a `url` and a `code`.
3. In ChatGPT, show the url as a link. In Codex, you can open it for them (Windows `start "" "<url>"`, macOS `open "<url>"`, Linux `xdg-open "<url>"`).
4. Tell them: "Sign up or log in on that page, check it shows the code `<code>`, then press Connect."
5. Keep building while they do it. When they say it's done, call `list_games`: it shows `connectedAs` once connected.

Never ask for a password, and never use one if it is offered: passwords are only typed on the website. If the link expires (10 minutes), call `connect_account` again. Without an account, the replies to `create_game` and `publish_game` include a claim link (`studio.claimUrl`) that keeps the games later.

## 2. Read the reference before writing client code

1. `create_game` with a clear, human name (or `list_games` to reuse one). Pass its publishable key (`pk_…`) as `game` on later calls once there is more than one game.
2. `get_sdk_reference`, and read it fully. Its snippets are wired to THIS game's key and endpoints. Copy the endpoint, `apiUrl` and key from it or from `get_connection_info`; never type them from memory.
3. For a quick start, `get_client_snippet` returns a complete single-file multiplayer page wired to a room.

## 3. Decide who owns each piece of state

- **Per-player** (position, name, color, score, input): each client writes only its own entity with `me.set(...)`.
- **Room-wide** (round timer, level, whose turn): `setShared(...)`, written by one agreed client (the host).
- **One-off happenings** (shot fired, hit, chat line): events with `mp.send` / `mp.on`.
- **Must not be cheatable** (scores that matter, hidden cards): server rules with `deploy_room_logic`. The server computes and clients only send input. This needs a connected account; until then the tool answers "Connect an account first", so build the rest and add it once they connect.

Keep room state small: it is sent to everyone many times a second.

For an existing game, read its code first and change as little as possible. Local input still drives the local player; remote players are drawn from room state.

## 4. Write the client

- Load the SDK from its absolute URL and pass both `endpoint` and `apiUrl` to `MP.join`. Without `apiUrl`, identity, saves and leaderboards do nothing.
- Leave cleanly: `window.addEventListener("pagehide", () => mp.leave())`.
- Show "reconnecting…" from `mp.onDisconnect` / `mp.onReconnect` instead of freezing.
- For moving things, interpolate remote players and predict the local one; the reference has the pattern.
- Never put a secret key (`sk_…`) in client code. The publishable key is safe to ship.

## 5. Host it and hand over the link

The game can't run inside a chat preview or sandbox (WebSockets are blocked there), so always publish it and give the user the link.

Call `publish_game` with every file the game needs, `index.html` at the root. It returns the share link, `https://<slug>.play.multiplayeragent.com/`. Publishing again updates the same link.

- HTML, JS and CSS go inline as `{ path, content }`. In ChatGPT, keep the game to inline text files (draw shapes in code rather than loading image files).
- In Codex, images, audio and any file over about 50 KB go through `create_upload` first: PUT each file to the URL it returns (`curl -sS -T "<file>" "<url>"`) within 15 minutes, then pass `{ path, uploadId }` to `publish_game`.
- To change a few files later, publish with `mode: "patch"`.

Finish with the share link ("open it in two browsers, or send it to a friend, and play together"), plus the claim link if they didn't connect an account.

## 6. Test like two real players

Ask the user to open the link in two different browsers, or one normal and one private window (two tabs of one browser share one saved player). Check that both players appear, movement or turns sync both ways, and someone joining mid-game sees the current state. If something is wrong, fix it and publish again.

## Common mistakes

- Writing a custom server or socket code. The backend already does it.
- Missing `apiUrl` in `MP.join`.
- Every client writing shared state, so it flickers. One host writes it.
- Using a key from another game. Every tool response has a `target` block: check it names the right game.

## What it suits

Browser games of every pace: shooters, racing, .io, co-op, party, casual and turn-based. Fast games use the prediction and interpolation patterns in `get_sdk_reference`, with hits decided in server rules. The one poor fit is one huge shared world: split players across rooms.
