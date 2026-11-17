# RCON and the HTTP API

Every Lodestone server has Source RCON on game port + 3 and a signed HTTP API on game port + 4, both inside the game. They stay off until you set an admin password. The Lodestone app's Console tab, a hosting panel, a scheduler or your own script all use them.

## Configuration

`BepInEx\config\com.humangenome.lodestone.admin.cfg`, written with defaults on the first start:

```ini
[Admin]
Enable = true
BindAddress = 0.0.0.0            # 127.0.0.1 keeps both listeners on the box
Password =                       # the RCON password and the HTTP signing secret; EMPTY = both listeners stay OFF
InstanceId =                     # defaults to the host's ServerName
GamePort = 0                     # 0 = read from the host config
RconPort = 0                     # 0 = game port + 3
HttpPort = 0                     # 0 = game port + 4
PublicPlayerList = true          # serve /api/v1/players without a signature
AdvertisePlayersToSteam = true   # feed the real player count to server query
IncludeHostPlayer = false

[Roster]
IntervalMs = 5000
Path =                           # empty = <game>\lodestone\roster.json

[Console]
BufferLines = 2000
```

With no password, both stay off. An RCON port with no password on a public server would let anyone run admin commands, so the server keeps it shut.

RCON and the API keep their files in `<game>\lodestone\`: `roster.json` (the live roster, see [status-and-roster.md](status-and-roster.md)), `bans.json` (the ban list, kept across restarts), `allowlist.json` (the whitelist, kept across restarts), `snapshots\` (world backups, see [World backups](#world-backups)) and `restart-requested` (written just before a restart, for whatever starts your server again).

## Source RCON

Standard Valve RCON on game port + 3. Commands before a successful login are refused and the connection is closed.

```
help                        this list
version                     admin plugin version
status                      server, world and online state
players | list              connected players
roster                      the roster document as it is right now
say <message>               broadcast into the in-game chat
announce <message>          the same as say: send a message to every player
kick <id|name> [reason]     kick a player
ban <id|name> [reason]      ban and kick a player
unban <id>                  lift a ban
bans                        list bans
allow <id|name>             add a player to the whitelist
deny <id>                   take a player off the whitelist
allowlist                   show the whitelist (empty = everyone can join)
save                        save the world now
shutdown [secs] [reason]    save and exit
restart [secs] [reason]     save and close the server. It comes back only if something starts it again
cancel                      cancel a pending shutdown or restart
time                        the world clock
time set <hh:mm> [day]      set the world clock
tp <player> <x> <y> [level] move a player; also tp <player> <other player> and tp <player> spawn
spawn <player>              send a player to the spawn point
mem                         the server's memory use
ports                       the ports this server is using
build                       the game build this server runs
ping                        pong
```

While the whitelist has at least one player, only those players can join, and anyone already on the server who isn't on it is kicked. The ban list applies either way.

## The HTTP API

Base URL `http://<server ip>:<game port + 4>`. Every response carries an `X-Lodestone-Instance` header naming the server.

### Public, no signature

| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/health` | `200` when the world is running and the snapshot is fresh, `503` otherwise; the body says why in `verdict` and `verdict_detail`, and names the game build the server runs |
| GET | `/api/v1/players` | Names, session length, ping. No positions. Can be made signed with `PublicPlayerList = false` |
| GET | `/api/v1/manifest` | What the server runs and what it asks of a player |

```json
// GET /api/v1/health -> 200
{"ok":true,"instance":"Deep Hollow","server_name":"Deep Hollow",
 "lodestone_version":"0.2.7","gameplay_port":27016,"query_port":27017,
 "rcon_port":27019,"http_port":27020,
 "ports":{"game":27016,"query":27017,"rcon":27019,"http":27020},
 "game_build":"26197027","game_version":"0.1","game_label":"3.35",
 "steam_build":"26197027","steam_app":"2710040",
 "max_players":8,"player_count":3,"world_running":true,
 "verdict":"HOSTING","verdict_detail":"","online_state":"GAME_CREATED",
 "snapshot_age_ms":350,"uptime_seconds":4210}
```

`ports` holds the ports the server really listens on. When the RCON or admin port is taken by another process, the server moves to a spare port in its own block and says so here, in `info`, in the roster and in the query tags.

`verdict` is the first line of `<game>\lodestone\boot-report.txt`, one of `HOSTING`, `STARTING`, `WILL NOT HOST` and `WORLD DID NOT START`; the meanings are in [status-and-roster.md](status-and-roster.md).

### Signed

| Method | Path | Body | Notes |
|---|---|---|---|
| GET | `/api/v1/info` | | Server name, game build, the live ports, player count, whether the whitelist is on, and the world clock |
| GET | `/api/v1/status` | | The full roster document |
| GET | `/api/v1/roster` | | Same document, see [status-and-roster.md](status-and-roster.md) |
| GET | `/api/v1/bans` | | |
| GET | `/api/v1/chat/recent?since=<ms>&limit=<n>` | | The chat log: what players typed and the server's broadcasts (signed since 0.2.7) |
| POST | `/api/v1/bans` | `{"id":"...","name":"...","reason":"..."}` | Bans, and kicks if connected |
| POST | `/api/v1/bans/remove` | `{"id":"..."}` | |
| GET | `/api/v1/allowlist` | | |
| POST | `/api/v1/allowlist` | `{"id":"..."}` or `{"name":"..."}` | Adds a player by Steam id, or by the name of someone connected |
| POST | `/api/v1/allowlist/remove` | `{"id":"..."}` | |
| POST | `/api/v1/players/kick` | `{"id":"...","reason":"..."}` | `id` may be an id or a character name; `404` when nobody matches |
| POST | `/api/v1/chat/say` | `{"sender":"...","msg":"..."}` | Broadcast into the in-game chat |
| POST | `/api/v1/save` | | Saves the world and returns when it is done |
| POST | `/api/v1/shutdown` | `{"delay_seconds":0,"reason":"..."}` | Saves, then exits. `202` |
| POST | `/api/v1/restart` | `{"delay_seconds":0,"reason":"..."}` | Same, plus the restart marker. `202` |
| POST | `/api/v1/console/exec` | `{"cmd":"..."}` | Runs any RCON command and returns its text |
| GET | `/api/v1/console/recent?since=<seq>&limit=<n>` | | Buffered console lines |
| GET | `/api/v1/console/stream?since=<seq>` | | The live console as server-sent events |
| GET | `/api/v1/snapshots` | | World backups, newest first, each with its id, size and SHA-256 |
| POST | `/api/v1/snapshots` | none, or a world zip | No body: back up the running world now. A zip body: keep that zip as a backup |
| GET | `/api/v1/snapshots/<id>/download` | | The backup as a zip; `X-Lodestone-Sha256` carries its hash |
| POST | `/api/v1/snapshots/<id>/restore` | | Restores that backup, then restarts the server |
| POST | `/api/v1/snapshots/import-restore` | a world zip | Keeps the zip as a backup and restores it, then restarts the server |

### Signing a request

```
key       = SHA256(password)                                  (32 raw bytes)
canonical = "<METHOD>\n<path>\n<unix seconds>\n<sha256hex(body)>"
signature = lowercase hex of HMACSHA256(key, canonical)

X-Lodestone-Timestamp: <unix seconds>
X-Lodestone-Signature: <signature>
X-Lodestone-Content-SHA256: <sha256hex(body)>        (required for a body over 64 KB)
```

- `<path>` is the path without the query string.
- `<sha256hex(body)>` is the hash of the empty string for a GET.
- The server reads no body until the signature headers are there and the timestamp is current. A body up to 64 KB is then read and checked. A bigger one (a world zip) must name its hash in `X-Lodestone-Content-SHA256`: the signature is checked against that hash before the first byte is read, and the body must match it. Without the header such a request is refused unread. Chunked bodies are not accepted.
- A timestamp more than 300 seconds from the server's clock is rejected.
- A signature is single-use inside that window.
- Five wrong signatures from one address in ten minutes turn that address away for a minute, then longer on repeats (up to 15 minutes). The count is shared with RCON's wrong passwords. A missing signature or a clock problem does not count.
- Every failure is `401 {"error":"unauthorized: <reason>"}`. The reason is a short sentence (no signature, clock skew, a password that differs from the caller's, a replayed signature) so an admin tool can tell its user what to fix; it never carries the password or the expected signature.

### The console stream

`GET /api/v1/console/stream` is `text/event-stream`, one event per line:

```
id: 162663
event: console
data: {"seq":162663,"ts":1785796970741,"source":"game","level":"Info","msg":"..."}
```

Omit `since` to start from now, or pass a sequence number to replay the backlog (up to 500 lines) so a reconnecting reader misses nothing. A `: keepalive` comment goes out every 15 seconds. `source` is `game` for the game's own log, `rcon`, `online`, `chat`, `admin`, or the name of the plugin that wrote the line.

## World backups

A backup is a zip of the world file and the folder beside it, kept in `<game>\lodestone\snapshots\`. The Lodestone app's World backups window uses these routes to back up, download, restore, and upload a save from a player's own game.

A running game holds its world in memory and writes it back when it saves, so a world can't be swapped while the game runs. A restore or an upload backs up the running world first, then restarts the server, which loads the new world under the server's World Name. The world it replaced is moved to `<game>\lodestone\world-import\replaced\`, never deleted. The answer carries `restarts_by_itself`: `false` means nothing will start the server again by itself, so start it by hand.

Backups the owner asks for or uploads are kept until deleted by hand. The automatic backups taken before a restore or upload are kept to the newest 10. An upload may be up to 64 MB.

## Server query

The server answers Source A2S on game port + 1 with the server name, the world name in the map field, the player count and the player names, and marks itself password-protected when a join password is set (both since 0.2.7), so a launcher can ask for the password before it starts the game. `AdvertisePlayersToSteam` (on by default) is what makes the count real: the game's own networking never reports occupants to the query responder, so without it every server reads `0/8` no matter who is in the world.
