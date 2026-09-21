# Changelog

All notable changes to Lodestone are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and Lodestone uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each version is split into **Server** (LodestoneServer and the host mod) and
**Client** (the Lodestone app and the client mod). A release publishes the
section for its own version verbatim as the release body, so a version with no
section here cannot be released.

## [Unreleased]

## [0.2.6] - 2026-09-21

### Server

#### Added

- World backups, kept by the server itself: back up the running world, list
  the backups, download one, put one back, or bring a player's own world in. A
  world cannot be swapped under a running game, so a restore or an import
  backs up the world that is running, stages the incoming one, and restarts
  the server; the new world is in place on that start. The world it replaced
  is moved to `lodestone/world-import/replaced/` in the server's files, never
  deleted. The answer says whether the server restarts by itself or has to be
  started again.
- A world file is found by the server's World Name. The name stored inside a
  save does not have to match it, so a world brought in from a player's own
  game keeps its inner name and runs under the server's World Name.
- The owner's console says in one line when a world was brought in, and where
  the old one went.

#### Fixed

- The owner's console opened with the last error lines of the previous start.
  It now starts at this start's first line.
- A world that began on the demo makes the game log a few error lines about
  areas the demo never had. Beside the demo-world notice they now read as one
  plain line saying they are expected, instead of red lines with nothing
  beside them.
- The boot report's world file line names the file the game really holds, not
  a file built from the world's inner name.

### Client

#### Added

- A new character is made on Delverium's own Create Character screen. When the
  character the app picked does not exist yet, the game opens Create Character
  with the name filled in; the player chooses hair, colours, body, voice and
  death penalty and presses Confirm, and the join carries on with that
  character. Closing the screen stops the join with a plain sentence, and no
  character is made behind the player's back.
- The app says so: "You will customize this character in Delverium when you
  first connect." While the screen is open the app reads "Make your character
  in Delverium", and the character the game made is the one the app remembers
  for that server, under the name the player confirmed.
- The World window: "Back up now", and "Choose a world" takes a world's `.wsv`
  straight from the player's own saves (the picker opens in their World
  folder). What the server said is shown in the server's own words, and a
  server too old to keep backups is named as such.
- Headless checks: `--characters <folder> --report <file>` and `--world-import
  <host> <port> <password> <world> <report>`.

## [0.2.5] - 2026-09-21

### Server

#### Added

- Delverium Early Access. The server runs the full game (Steam app 2710040) as
  well as the free demo, and one build of the server files serves both.
- A clean stop. `shutdown` saves the world, closes its listeners, writes
  `lodestone/last-shutdown.json` saying whether the world was saved, and
  leaves within seconds. The process can no longer be left half closed and
  holding its ports.
- A world check before the game's loader runs. A world saved by a newer
  version of the game than the server runs, or one too old for it, is held
  with a plain sentence for the owner, and the file is left untouched.
- A world made on the demo loads on the full game, and the owner is told once
  per start: "This world was made on the Delverium demo. It keeps the demo's 4
  areas and the full game does not add the rest. Pick a new world name in the
  server's settings for a full-game world; this world stays in the server's
  files." `[World] DemoWall = true` holds such a world instead of loading it.
- The game build and its label are in `health`, `info`, the roster, the
  `build` command and the Steam query tags, so the app can tell a mismatch
  before it connects.
- The game turns away a player whose game is on another build without telling
  anyone why. The server now says so in the owner's console, and says when the
  cause may be a wrong join password instead.
- An allow list (`allow`, `deny`, `allowlist`). When it is on, only listed
  players get in, and anyone already in who is not listed is shown out.
- Live ports. When the console or admin port is held by another process, the
  server moves to a spare port in its own block and publishes the real ports
  in `health`, `info`, the roster and the query tags.
- Console commands: `time`, `time set`, `tp`, `spawn`, `mem`, `ports`, `build`
  and `ping`.
- The boot report says where the world file is, and whether that is inside the
  server's own folder.
- The live map is drawn by the game's own rules: edge pieces, every part of a
  composite tile, the floor plane, and the full game's art on all nine levels.
  The server keeps a fog of war, and the viewer has a zoom ladder, player
  chips, a scale bar and an embed mode.

#### Fixed

- Players can join a full-game world. The game sends a world in 256 KB pieces,
  and the server's connections were opened with Steam's small default buffer,
  so a full-game world never arrived and the player sat at the loading screen.
- A server's saves stay in the server's own folder on the full game, which
  otherwise writes them to the Windows profile.
- The chat commands `/tp`, `/spawn` and `/back` no longer crash the server
  about a minute later when a player is connected.
- Two signed reads of the admin API inside the same second no longer refuse
  the second one, and every refusal says why.
- The owner's console reads in plain words. Debug lines, the mod loader's own
  narration and the game's boot noise stay out of it.

### Client

#### Added

- The app checks the game build before it connects and says in words when the
  server runs another version, or the full game against the demo.
- The app finds the full game's install. With the demo and the full game both
  installed it picks the full game, and a demo folder is no longer read as the
  full game.
- The app looks for a player's characters where the full game keeps them.
- The Console tab opens connected and follows a server whose console and admin
  ports moved.
- Headless checks for server owners: `--probe <host:port>` and `--rcon-probe`.

#### Fixed

- The connect files open their connection with the game's own buffer size, the
  same fix as the server.
- Long console answers are read whole instead of being cut at one packet.
- The saved admin password is masked, and the New character button no longer
  shows two plus signs.

## [0.2.4] - 2026-09-20

### Server

#### Fixed

- The server starts its world again on the September 15 Delverium demo update.
  That update changed how a new character's defaults are built, and the server
  never got past creating its own character, so every world sat unstarted and
  the gameplay port stayed closed.
- The live map draws again on that update. Item and ground-tile descriptions
  moved, and the map scanner was reading fields that no longer exist.

### Client

#### Fixed

- Creating a character for a new server on the September 15 demo update works
  again; the connect files the app installs carry the same fix as the server.

## [0.2.3] - 2026-08-10

The first public release. Everything below is new in the sense that matters:
this is the first version of Lodestone anyone outside its makers can download
from this page.

### Server

#### Added

- A Delverium server that runs on its own: it generates a world, keeps it
  saved, and reloads the same world on the next start. No player has to be
  online for the world to exist.
- Runs without a graphics card, a monitor, a desktop session, or a signed-in
  Steam account, so it is happy on a spare machine or a rented one.
- Players connect by address on a configurable port, up to Delverium's usual
  eight.
- Server status on the port above the gameplay port, so the app can show
  whether a saved server is up and how many people are on it.
- In-game chat, including server-wide messages, useful for warning a group
  about a scheduled restart without needing them in Discord.
- A live world map served to any browser, drawn with the game's own art, with
  the surface and the cavern systems below it, refreshed as the world changes.
- Admin control: player roster, live console, kick and ban, save on demand,
  restart and shutdown, all reachable without logging into the machine.
- A public mod list: the server reports what it is running, and a server owner
  can declare required, recommended, and blocked mods for the app to show.

### Client

#### Added

- Lodestone app for Windows: save servers by address, pick a character, and
  connect. Servers and characters are remembered between sessions.
- Live status for each saved server, so you can see what is up before you
  launch anything.
- Signed automatic updates, so the app keeps itself current.
- A Mods tab showing what the selected server runs, plus any mods the server
  owner recommends, requires, or blocks.
- Delverium has no server browser of its own, so every player needs the
  Lodestone app to reach a Lodestone server. It is a one-time install per
  person, and normal Delverium co-op is unaffected.

[Unreleased]: https://github.com/HumanGenome/LodestoneServer/commits/main
[0.2.4]: https://github.com/HumanGenome/LodestoneServer/releases/tag/v0.2.4
[0.2.3]: https://github.com/HumanGenome/LodestoneServer/releases/tag/v0.2.3
