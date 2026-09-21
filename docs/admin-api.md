# The admin plane: RCON and the HTTP API

Every Lodestone server carries an admin plane inside the game process: Source RCON on game port + 3 and a signed HTTP API on game port + 4. Both are off until an admin password is set. The Lodestone app's Console tab, a hosting panel, a scheduler or your own script all use the same two doors.

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

An empty password is a hard off switch for both listeners. An unauthenticated RCON port on a public host is a remote shell, so the server fails closed rather than open.

Files the admin plane writes live in `<game>\lodestone\`: `roster.json` (the live roster, see [status-and-roster.md](status-and-roster.md)), `bans.json` (the ban list, kept across restarts) and `restart-requested` (a marker written just before a restart-flavoured exit, for whatever supervises the process).

## Source RCON

Standard Valve RCON on game port + 3. Commands before a successful login are refused and the connection is closed.

```
help                        this list
version                     admin plugin version
status                      server, world and online state
players | list              connected players
roster                      the roster document for this instant
say <message>               broadcast into the in-game chat
kick <id|name> [reason]     drop a connected player
ban <id|name> [reason]      ban and drop
unban <id>                  lift a ban
bans                        list bans
save                        save the world now
shutdown [secs] [reason]    save and exit
restart [secs] [reason]     save, exit, and leave the restart marker
cancel                      cancel a pending shutdown or restart
```

## The HTTP API

Base URL `http://<server ip>:<game port + 4>`. Every response carries an `X-Lodestone-Instance` header naming the server.

### Public, no signature

| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/health` | `200` when the world is running and the snapshot is fresh, `503` otherwise; the body says why in `verdict` and `verdict_detail` |
| GET | `/api/v1/players` | Names, session length, ping. No positions. Can be made signed with `PublicPlayerList = false` |
| GET | `/api/v1/chat/recent?since=<ms>&limit=<n>` | The broadcast log |
| GET | `/api/v1/manifest` | What the server runs and what it asks of a player; the app's Mods tab reads it |

```json
// GET /api/v1/health -> 200
{"ok":true,"instance":"Deep Hollow","server_name":"Deep Hollow",
 "lodestone_version":"0.2.4","gameplay_port":27016,"query_port":27017,
 "rcon_port":27019,"http_port":27020,"max_players":8,"player_count":3,
 "world_running":true,"verdict":"HOSTING","verdict_detail":"",
 "snapshot_age_ms":350,"uptime_seconds":4210}
```

`verdict` is the first line of `<game>\lodestone\boot-report.txt`, one of `HOSTING`, `STARTING`, `WILL NOT HOST` and `WORLD DID NOT START`; the meanings are in [status-and-roster.md](status-and-roster.md).

### Signed

| Method | Path | Body | Notes |
|---|---|---|---|
| GET | `/api/v1/status` | | The full roster document |
| GET | `/api/v1/roster` | | Same document, see [status-and-roster.md](status-and-roster.md) |
| GET | `/api/v1/bans` | | |
| POST | `/api/v1/bans` | `{"id":"...","name":"...","reason":"..."}` | Bans, and kicks if connected |
| POST | `/api/v1/bans/remove` | `{"id":"..."}` | |
| POST | `/api/v1/players/kick` | `{"id":"...","reason":"..."}` | `id` may be an id or a character name; `404` when nobody matches |
| POST | `/api/v1/chat/say` | `{"sender":"...","msg":"..."}` | Broadcast into the in-game chat |
| POST | `/api/v1/save` | | Saves the world and returns when it is done |
| POST | `/api/v1/shutdown` | `{"delay_seconds":0,"reason":"..."}` | Saves, then exits. `202` |
| POST | `/api/v1/restart` | `{"delay_seconds":0,"reason":"..."}` | Same, plus the restart marker. `202` |
| POST | `/api/v1/console/exec` | `{"cmd":"..."}` | Runs any RCON command and returns its text |
| GET | `/api/v1/console/recent?since=<seq>&limit=<n>` | | Buffered console lines |
| GET | `/api/v1/console/stream?since=<seq>` | | The live console as server-sent events |

### Signing a request

```
key       = SHA256(password)                                  (32 raw bytes)
canonical = "<METHOD>\n<path>\n<unix seconds>\n<sha256hex(body)>"
signature = lowercase hex of HMACSHA256(key, canonical)

X-Lodestone-Timestamp: <unix seconds>
X-Lodestone-Signature: <signature>
```

- `<path>` is the path without the query string.
- `<sha256hex(body)>` is the hash of the empty string for a GET.
- A timestamp more than 300 seconds from the server's clock is rejected.
- A signature is single-use inside that window.
- Every failure is `401 {"error":"unauthorized"}`, with no hint about which check failed.

### The console stream

`GET /api/v1/console/stream` is `text/event-stream`, one event per line:

```
id: 162663
event: console
data: {"seq":162663,"ts":1785796970741,"source":"game","level":"Info","msg":"..."}
```

Omit `since` to start from now, or pass a sequence number to replay the backlog (up to 500 lines) so a reconnecting reader misses nothing. A `: keepalive` comment goes out every 15 seconds. `source` is `game` for the game's own log, `rcon`, `online`, `chat`, `admin`, or the name of the plugin that wrote the line.

## Server query

The server answers Source A2S on game port + 1 with the server name, map, player count and the player names. `AdvertisePlayersToSteam` (on by default) is what makes the count real: the game's own networking never reports occupants to the query responder, so without it every server reads `0/8` no matter who is in the world.
