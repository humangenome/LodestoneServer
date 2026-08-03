# Changelog

All notable changes to Lodestone are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and Lodestone uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each version is split into **Server** (LodestoneServer and the host mod) and
**Client** (the Lodestone app and the client mod). A release publishes the
section for its own version verbatim as the release body, so a version with no
section here cannot be released.

## [Unreleased]

### Server

#### Added

- Headless Delverium host: the server generates a world, saves it, and reloads
  the same world on the next start.
- Direct connections on a configurable UDP port, so players join by address
  rather than by friend invite.
- Steam server query on the port above the gameplay port, reporting server
  name, map, game, player count, and version.
- Runs without a graphics card, a desktop session, or a logged-in Steam
  account.

### Client

#### Added

- Lodestone desktop app for Windows: save servers by address, pick a
  character, and connect.
- Servers appear in Delverium's own Online list, so joining works the same way
  as joining any other game.
- Live status for each saved server, read from the server query port.

[Unreleased]: https://github.com/HumanGenome/LodestoneServer/commits/main
