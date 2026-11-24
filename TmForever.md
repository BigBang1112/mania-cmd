# TrackMania Forever command line arguments

Command line arguments of `TmForever.exe`, found by reading its code in Ghidra. None of them were tested by running the game.

- [Quick reference](#quick-reference)
- [Syntax](#syntax)
- [Opening files and links](#opening-files-and-links)
- [Account](#account)
- [Joining a server](#joining-a-server)
- [Automated modes](#automated-modes)
- [Window and process](#window-and-process)
- [Paths and files](#paths-and-files)
- [Logging](#logging)
- [Internal](#internal)
- [Examples](#examples)
- [Open questions](#open-questions)
- [Where this comes from](#where-this-comes-from)

How to read the descriptions:

- No mark: read directly from the code.
- *(inferred)*: based on names or surrounding code.
- *(no reader found)*: the value is stored, and a search of the likely code found nothing that reads it. This doesn't prove that it's unused.
- *(not traced)*: the argument is read, but what happens afterwards wasn't followed.

## Quick reference

| Argument | Type | Description |
|---|---|---|
| `/file` | File | Map or replay to open |
| `/url` | Text | Link to open |
| `/profile` | Text | Profile to select |
| `/login` | Text | Account login |
| `/password` | Text | Account password |
| `/validation` | Text | Account validation value |
| `/nickname` | Text | Nickname |
| `/join` | Text | Server to join as a player |
| `/spectate` | Text | Server to join as a spectator |
| `/joinasreferee` | Text | Server to join as a referee |
| `/joinpassword` | Text | Server password |
| `/serverpassword` | Text | Server password, if there's no `/joinpassword` |
| `/lan` | Switch | LAN mode |
| `/silent` | Switch | Skip the intro |
| `/team` | Text | Team to request |
| `/bench` | Presence, file | Benchmark, then exit |
| `/list_benchs` | File | Benchmark a list of replays, **write the results into the list**, then exit |
| `/out` | File | File name of the benchmark results |
| `/shootvideo` | File | Shoot the video of a replay, then exit |
| `/validatepath` | File | Validate replays, then exit |
| `/checksolorecords` | File | No use of it was found |
| `/computeallshadows` | Switch | Compute the shadows of Stadium maps, then exit |
| `/windowless` | Switch | No window, with a console |
| `/singleinst` | Switch | Reuse the running game |
| `/title` | Text | **No effect**, overwritten during startup |
| `/nodaemon` | Switch | No use of it was found |
| `/useexedir` | Switch | Run in the exe directory |
| `/config` | File | System config file |
| `/ini` | File | Replacement for `Nadeo.ini` |
| `/userdir` | File | User directory |
| `/noscancache` | Switch | Skip the file scan cache |
| `/logfilename` | File | No use of it was found |
| `/nologs` | Switch | Clears a network flag |
| `/hMapFileAC` | Text | Internal shared memory handle |

## Syntax

```bat
TmForever.exe /switch /name=value "/name=value with spaces"
```

- **Names are case-sensitive.** `/Login` is not `/login`, and `/hMapFileAC` needs its exact casing.
- The name is everything before the first `=`, and it has to match completely.
- `/name value` doesn't work, the `=` is required.
- If an argument is repeated, **the first one wins**.
- Unknown arguments are silently ignored.
- **Arguments without a `/` are ignored.** Open a map with `/file=`, as just passing its path does nothing.
- Values with spaces are quoted, as either `"/name=a b"` or `/name="a b"`.

### Types

| Type | Meaning |
|---|---|
| Switch | On when given, except with the values listed below |
| Presence | On whenever given, whatever the value is |
| Text | The value as a string |
| File | A file system path, converted to the game's format |

A switch is off if its value is `0`, `false`, `no`, `n`, `f` or starts with `disab` (case-insensitive). Anything else turns it on, including an empty value and `off`.

> [!WARNING]
> The game keeps its whole command line in its log buffer. A normal run doesn't create a log file, but **a crash log gets that buffer**, with the value of `/password` in plain text. Check crash logs before sharing them.

## Opening files and links

### `/file=<path>`

Opens an existing file:

- A map is played.
- A replay is viewed.
- Any other type is only logged.

A file outside of the user directory is first copied into it.

### `/url=<link>`

Opens a game link, which starts with `tmtp://`. It also **forces the game into a window**.

These commands can follow the scheme:

| Command | Action |
|---|---|
| `#join=<server>` | Join as a player |
| `#spectate=<server>` | Join as a spectator |
| `#joinasreferee=<server>` | Join as a referee |
| `#addbuddy=<login>` | Add a buddy |
| `#invitebuddy=<login>` | Invite a buddy |
| `#addfavourite=<server>` | Add a favorite server |
| `#score=<value>` | Start a direct score challenge |

`<server>` in these links has the format `address[:password[:lan]]`, where the brackets mark optional parts and `lan` is a boolean.

What follows the scheme can also be an address. It's opened as `http://<address>`, or unchanged if it starts with `:` (the colon is removed).

A `#quit` command is defined in the executable, but never handled.

## Account

| Argument | Description |
|---|---|
| `/profile=<name>` | Selects this profile |
| `/login=<login>` | Account login. It goes through the login character filter. |
| `/password=<password>` | Account password |
| `/validation=<value>` | Stored as the account validation value *(not traced)* |
| `/nickname=<name>` | Nickname, ignored if empty |

## Joining a server

Only one of these is used, in this order of priority:

1. `/join=<server>`: as a player
2. `/spectate=<server>`: as a spectator
3. `/joinasreferee=<server>`: as a referee

These go with them:

| Argument | Description |
|---|---|
| `/joinpassword=<password>` | Server password |
| `/serverpassword=<password>` | Used if `/joinpassword` is empty or missing |
| `/lan` | LAN mode |
| `/silent` | Skips the intro. Other effects are *(not traced)*. |
| `/team=<team>` | `0` or `blue`, `1` or `red`. The color names are case-insensitive. |

`<server>` is a login or an address such as `192.168.1.10:2350`. Unlike in links, it **isn't split** into `address:password:lan`, so use `/joinpassword` and `/lan` for those.

## Automated modes

Each of these exits the game when it's done. If several are given, only the first in this list runs:

1. `/bench` or `/list_benchs`
2. `/shootvideo`
3. `/validatepath`
4. `/computeallshadows`

### `/bench`, `/bench=<file>`

Benchmark mode. It only has to be present, so `/bench=0` enables it too, with `0` as the file name.

- With a file, that replay is benchmarked. A relative name starts from `Tracks\Replays\`.
- Without a file or list, the graphics performance is measured and saved. This skips the startup menus and forces a window.
- The exit code is `1` unless the benchmark succeeds.

### `/list_benchs=<file>`

Benchmarks the replays in a text file with one name per line. Lines starting with `//` are skipped. A file given to `/bench` takes priority over the list.

> [!CAUTION]
> **The list file is overwritten.** It's written back with the results added as `//` comments, even if it was read-only. Use a copy.

### `/out=<file>`

The file name for the results of `/bench=<file>`, inside `Bench\` in the user directory. The default is `Bench000.txt` and so on. It isn't used with a list.

### `/shootvideo=<replay>`

Loads the replay, opens the video shooting flow and exits afterwards. The name is relative to `Tracks\Replays\`. Forces a window. If the replay can't be loaded, the game shows an error and exits once it's closed.

### `/validatepath=<path>`

Validates a replay, or all replays in a folder.

- The path is relative to `Tracks\Replays\` in the user directory.
- The results are written to `ValidateLog.txt` in the user directory.
- Progress is shown in a dialog that can be canceled.
- An empty value doesn't start the validation.
- **Don't use `.`**. It skips the search for replays and then uses a count that was never initialized, so it doesn't validate the whole folder and what it does is undefined. Give a folder name instead.

### `/checksolorecords=<file>`

Passed to the game as a file name *(no reader found)*. `/validatepath` doesn't use it. In ManiaPlanet, the argument with this name is a switch instead.

### `/computeallshadows`

Computes the shadows of the **Stadium** maps that don't have an up-to-date shadow cache, instead of showing the main menu, then exits.

- Lightmaps have to be enabled and supported.
- After a map, the game can ask `Are shadows ok ?` for 5 seconds. One of the answers deletes the new cache file, likely "No" *(inferred)*.

## Window and process

### `/windowless`

Runs without a window and opens a console. If the game then exits with a non-zero code, the console **waits for a key press**, which matters in scripts.

### `/singleinst`

If a game window already exists, focuses it, sends it `/url` (or `/file` if there's no URL) and exits. Otherwise the game starts normally.

### `/title=<text>`

**Has no effect on the window title.** The value is stored, then replaced during initialization, before the window is created. The title comes from `WindowTitle` in `Nadeo.ini`, or is `TmForever`.

### `/nodaemon`

Clears a flag that is on by default *(no reader found)*.

## Paths and files

### `/useexedir`

The game uses the **current directory** as its root by default. This makes it use the directory of the executable.

### `/config=<file>`

The system config file to load.

### `/ini=<file>`

Replaces `Nadeo.ini`. The value is appended to the game's root directory, so it has to be a **relative path**.

### `/userdir=<path>`

Overrides the user directory. It only works if the ini file defines `UserSubDir`.

- `exe` (case-insensitive) uses `<root>\<UserSubDir>`.
- Any other value is used as the directory.

### `/noscancache`

Doesn't load or update the file scan caches, `Game.FidCache.Gbx` and `User.FidCache.Gbx`.

## Logging

### `/nologs`

- For the application, it sets a flag that is already on by default, so the per-process console log is off either way.
- For the network, it clears a flag *(not traced)*.
- It doesn't prevent `ValidateLog.txt`.

### `/logfilename=<file>`

Stored *(no reader found)*. The log is set up with an empty file name, so a normal run has no log file. Only the crash handlers create one.

## Internal

### `/hMapFileAC=<handle>`

A hexadecimal handle of shared memory that the game connects to as the receiving side at startup. The `AC` likely stands for anticheat *(inferred)*. It's meant to be passed by the process that launches the game.

## Examples

These follow from the code and weren't run.

Open a map:

```bat
TmForever.exe /useexedir "/file=C:\Maps\My Map.Challenge.Gbx"
```

Join a password protected server as a spectator:

```bat
TmForever.exe /useexedir /login=YOUR_LOGIN /password=YOUR_PASSWORD /spectate=SERVER_LOGIN /joinpassword=SERVER_PASSWORD
```

Validate the replays in `Tracks\Replays\ToCheck`:

```bat
TmForever.exe /useexedir /validatepath=ToCheck
```

## Open questions

These need a run of the game or more tracing:

- What the master server connection does with `/validation`.
- Whether `/silent` does more than skip the intro.
- What the network flag of `/nologs` controls.
- What is sent through the shared memory of `/hMapFileAC`.

## Where this comes from

| Address | Function | What it reads or does |
|---|---|---|
| `00529c00` | `WinMainInternal` | Splits the command line, `/singleinst`, console |
| `00421340` | `CSystemCommandLine::AddCmdLineArg` | Splits names from values |
| `00420e30` | `CSystemCommandLine::FindArg` | Case-sensitive lookup |
| `0052aa20` | `CGbxApp::ApplyCommandLineArgs` | Application arguments |
| `00401b80` | `CGbxGame::ApplyCommandLineArgs` | Game arguments |
| `004021b0` | `CGbxGame::Init` | Overwrites the window title |
| `004023e0` | `CGbxGame::StartApp` | Passes values to the game, `/hMapFileAC` |
| `00401610` | `CGbxGame::IsForceWindowed` | Forced window |
| `0041e340` | `InternalComputePaths` | `/ini`, `/userdir` |
| `00497830` | `CTrackMania::UpdateGame` | Account, joining, automated modes |
| `0048c9b0` | `CTrackMania::ReplayValidatePath` | Replay validation |
| `00488f10` | `CTrackMania::ComputeAllChallengeShadows` | Shadow computation |
| `004ff040` | `CTrackMania::CmdBenchReplayList` | Benchmark list, written back |
| `00698b50` | `CGameCtnBench::FpsSnapSaveResult` | Benchmark results, `/out` |
| `0068eb50` | `CGameCtnNetwork::MainLoop_Menus` | Joining |
| `005e7170` | `CGameCtnApp::SkipIntroBecauseAutomaticActionRequested` | `/silent` |
| `009085d0` | `CClassicLog::SetOutputFile` | Copies the log buffer into the file |
| `005ba0d0` | `CGameNetwork::Init` | `/nologs` |
| `005eb690` | `CGameCtnApp::LoadAndRefreshFidCache` | `/noscancache` |
| `0059ebc0` | `CGameApp::CmdLineUrlTmtp_Check` | Link commands |

The list of arguments comes from a byte by byte scan of the executable for calls to the `CSystemCommandLine` getters and for references to argument-like strings, which includes code that Ghidra didn't disassemble. An argument name built at runtime wouldn't be found.
