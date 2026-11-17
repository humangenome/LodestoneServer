# In-game chat

Delverium has no chat of its own. A Lodestone server adds one in the game's own font, and every player with the Lodestone app has it as soon as they join.

## For players

- Press **T** to open the chat line, type, press Enter.
- Player messages, join and leave notices, server announcements and restart countdowns all show in one feed. Player lines, server lines and notices each look different.
- Lines fade after a while. Open the chat to see the recent ones again.
- A line starting with `/` is a command. `/help` lists them.

| Command | Does |
|---|---|
| `/help [command]` | list the commands, or one command's usage |
| `/players` (`/who`, `/list`) | who is online |
| `/whoami` | your name and whether you're an admin |
| `/motd` | show the server's message of the day |
| `/version` | the chat version |

## For admins

An admin is a player whose Steam ID is listed under `Admins` in the chat config. A player can't make themselves an admin: the server checks the Steam ID itself.

| Command | Does |
|---|---|
| `/say <text>` (`/announce`) | broadcast on the server channel |
| `/motd set <text>` | set the message of the day |
| `/tp <player>` | teleport yourself to a player |
| `/tp <player> <target>` | teleport one player to another |
| `/tp <x> <y>` | teleport yourself to coordinates |
| `/spawn [player]` | teleport to the world spawn |
| `/back` | undo your last teleport |
| `/warn <minutes> [reason]` | a countdown broadcast with no restart at the end |
| `/restart <minutes> [reason]` | a countdown, then save and restart |
| `/cancel` | cancel a pending countdown |
| `/kick`, `/ban`, `/unban`, `/bans`, `/save`, `/status` | the same as the RCON commands of the same name |

Countdowns announce at 30, 15, 10, 5, 2 and 1 minutes and at 30 and 10 seconds. Teleports go through the game's own movement message, so every player sees the move. Every admin command, and every refusal, is written to the server console so the log shows who did what.

## Configuration

`BepInEx\config\com.humangenome.lodestone.chat.cfg`, written with defaults on the first start:

```ini
[Chat]
Enable = true
OpenKey = T                      # the key that opens the chat line
MaxMessageLength = 200           # bytes per message; longer messages are cut
JoinLeaveNotices = true          # announce joins and leaves
Motd =                           # sent to each player once, on join, and shown by /motd

[Chat.Admin]
Admins =                         # comma-separated player ids allowed to run admin commands

[Chat.Limits]
Burst = 3                        # messages a player may send at once
PerMinute = 20                   # and per minute
HistoryLines = 200               # lines kept for players who join later

[Chat.Ui]
Lines = 8                        # messages kept on screen
HoldSeconds = 9                  # full brightness after the last message
FadeSeconds = 2                  # then fade to nothing
Backdrop = true                  # translucent bar behind each line
FontScale = 1                    # 1 = the game's own message text size
```

## Safety

Messages are plain text. Anything that is not printable text is stripped, every message is capped at `MaxMessageLength` bytes, and a player who floods is rate-limited on the server (`Burst` and `PerMinute`). A player whose Lodestone app has a different chat version doesn't see the chat, and the chat never reaches their game, so they can still play.
