<p align="center">
  <img src="docs/img/lodestone-lockup.png" alt="Lodestone" width="420">
</p>

<p align="center">
  <a href="#requirements"><img src="https://img.shields.io/badge/Platform-Windows_10%2F11%2FServer-blue.svg" alt="Platform"></a>
  <a href="https://store.steampowered.com/app/2710040/"><img src="https://img.shields.io/badge/Game-Delverium-darkgreen.svg" alt="Game"></a>
  <a href="https://github.com/HumanGenome/Lodestone"><img src="https://img.shields.io/badge/Player_App-Lodestone-brightgreen.svg" alt="Player app"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-lightgrey.svg" alt="License"></a>
</p>

# LodestoneServer

The server half of [Lodestone](https://github.com/HumanGenome/Lodestone) — the piece that runs on a host machine and keeps a [Delverium](https://store.steampowered.com/app/2710040/) world online around the clock.

Players use the Lodestone app; hosts use this. Start here if you are running the server, and at the [Lodestone hub](https://github.com/HumanGenome/Lodestone) if you are joining one.

> **Pre-release.** Lodestone is being built for Delverium's Early Access launch on 22 September 2026. The server source and the first downloadable package land here before that date. Nothing on this page is downloadable yet.

## What it does

- Runs Delverium as a headless host — no graphics card, no desktop session, no logged-in Steam account
- Owns the world. The world is created, generated, and saved on the host, and it stays there when players leave
- Accepts direct connections on a normal UDP port, so players reach it by address instead of a friend invite
- Answers the standard Steam server query, so server lists, monitoring, and bots can read its status and player count
- Holds up to eight players, matching Delverium's own limit

## What it does not do

- **It does not ship Delverium.** You install the game files yourself; the server launches them from a folder you choose.
- **It does not replace the player's game.** Everyone joining still runs retail Delverium, with the Lodestone app on top.
- **It is not an anti-cheat.** Lodestone does not police what connected clients do.

## Requirements

| | |
|---|---|
| OS | Windows 10, Windows 11, or Windows Server |
| Game files | A Delverium installation on the host |
| Ports | Two UDP ports — the gameplay port, and the port immediately above it for server query |
| Hardware | No GPU required |

## Ports

Everything derives from one number, so you only ever choose the gameplay port.

| Port | Protocol | Used for |
|---|---|---|
| base (default `27016`) | UDP | Gameplay — the port players connect to |
| base + 1 (default `27017`) | UDP | Steam server query |

Both must be open on the host firewall and forwarded if the server sits behind NAT.

## Setup

Setup instructions land here with the first server package. They will cover installing the game files, choosing ports, picking a world name, and running the server as a service.

## Managed hosting

If you would rather not run the machine, [SurvivalServers.com](https://www.survivalservers.com/services/game_servers/delverium/?utm_source=github&utm_medium=readme_install&utm_campaign=lodestone) runs Delverium servers with Lodestone already installed and the ports already open.

## Changelog

Every released version is listed in [CHANGELOG.md](CHANGELOG.md), split by what changed on the **Server** and what changed on the **Client**. Release pages quote the matching section verbatim.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Security issues go through [private reporting](.github/SECURITY.md), never a public issue.

## Community note

Lodestone is an independent community project. It is not affiliated with, endorsed by, or supported by Sagestone Games.

## License

MIT — see [LICENSE](LICENSE).
