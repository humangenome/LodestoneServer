# Changelog

All notable changes to Lodestone are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and Lodestone uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each version is split into **Server** (LodestoneServer and the host mod) and
**Client** (the Lodestone app and the client mod). A release publishes the
section for its own version verbatim as the release body, so a version with no
section here cannot be released.

## [Unreleased]

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
