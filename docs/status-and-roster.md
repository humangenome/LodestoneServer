# What the server writes: the boot report and roster.json

A Lodestone server keeps a few plain files in `<game>\lodestone\` so a panel, a script or a support person can read its state without a login or a port. Nothing here needs the admin password.

## boot-report.txt

Written on every start, rewritten as the start progresses. The first line is the verdict; the rest is a page of plain text: the game build and engine version, which patches applied and what each one is for, the world state, the ports, and whether a join password is in force.

| Verdict | Means |
|---|---|
| `STARTING` | Still coming up |
| `HOSTING` | World running, gameplay port open, accepting players |
| `WILL NOT HOST` | A required patch did not apply, Steam would not start, or the gameplay port could not be opened. The gameplay port is deliberately never opened in this state, so nobody joins a half-working server |
| `WORLD DID NOT START` | Hosting began but the world never came up inside its deadline. The port is shut again |
| `CLIENT` | This process is a joiner, not a server |

The same verdict is served by `GET /api/v1/health` on the admin API, which answers `200` only for `HOSTING`.

## roster.json

Rebuilt once a second and written every 5 seconds (`Roster.IntervalMs` in the admin config), atomically, so a reader never sees a half-written document. One more write happens on shutdown. `Roster.Path` moves it; the default is `<game>\lodestone\roster.json`. The signed `GET /api/v1/roster` and `GET /api/v1/status` serve the identical document.

```json
{
  "version": 1,
  "instance": "Deep Hollow",
  "generated_by": "LodestoneAdmin 0.2.4",
  "unix_ms": 1785797188997,
  "stale": false,
  "server": {
    "name": "Deep Hollow",
    "game_port": 27016,
    "query_port": 27017,
    "rcon_port": 27019,
    "http_port": 27020,
    "max_players": 8
  },
  "world": {
    "name": "Deep Hollow",
    "difficulty": "NORMAL",
    "size": "MEDIUM",
    "horde_mode": false,
    "running": true
  },
  "player_count": 2,
  "players": [
    {
      "id": "90000000000000241",
      "name": "Rowan",
      "state": "RUNNING",
      "spawned": true,
      "level_id": 0,
      "x": 64.5,
      "y": 99.5,
      "ping_ms": 31,
      "connected_unix_ms": 1785797139760,
      "session_seconds": 8049
    }
  ]
}
```

| Field | Meaning |
|---|---|
| `stale` | `true` when the snapshot is older than it should be; the rest of the document is the last good one |
| `server.*` | The advertised name and where each listener is |
| `world.*` | The world being hosted, its settings, and whether it is running |
| `players[]` | Everyone connected: character name, whether they have spawned, which level they are on and where, ping, and how long they have been on |

`x` and `y` are world units, which are tile units, so a map can place a player without conversion. `level_id` is the game's own level index; `0` is the surface.

## bans.json

The ban list, kept across restarts, in the same folder. Edit it through RCON (`ban`, `unban`, `bans`) or the admin API rather than by hand while the server is running.

## Stopping and restarting

`shutdown` and `restart` over RCON or the API save the world first and then exit; `restart` also drops a `restart-requested` marker in the folder so whatever supervises the process knows to bring it back. Both take a delay in seconds and a reason, and the in-game chat counts the delay down. `cancel` stops a pending one.
