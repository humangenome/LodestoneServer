# Changelog

All notable changes to Lodestone are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and Lodestone uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each version is split into **Server** (LodestoneServer and the host mod) and
**Client** (the Lodestone app and the client mod). A release publishes the
section for its own version verbatim as the release body, so a version with no
section here cannot be released.

## [Unreleased]

First working version. Nothing has been released yet, so everything below is new.

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
- In-game chat, including server-wide messages — useful for warning a group
  about a scheduled restart without needing them in Discord.
- A live world map served to any browser, drawn with the game's own art, with
  the surface and the cavern systems below it, refreshed as the world changes.
- Admin control: player roster, live console, kick and ban, save on demand,
  restart and shutdown — all reachable without logging into the machine.

### Client

#### Added

- Lodestone app for Windows: save servers by address, pick a character, and
  connect. Servers and characters are remembered between sessions.
- Live status for each saved server, so you can see what is up before you
  launch anything.
- Signed automatic updates, so the app keeps itself current.

#### Notes

- Delverium has no server browser of its own, so every player needs the
  Lodestone app to reach a Lodestone server. It is a one-time install per
  person, and normal Delverium co-op is unaffected.

[Unreleased]: https://github.com/HumanGenome/LodestoneServer/commits/main
