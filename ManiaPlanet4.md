# ManiaPlanet command line arguments

Command line arguments of ManiaPlanet, found by reading the code of `ManiaPlanetLogs.exe` (x64) in Ghidra. None of them were tested by running the game, and the other executables of the game weren't checked.

- [Quick reference](#quick-reference)
- [Syntax](#syntax)
- [Opening files and links](#opening-files-and-links)
- [Account](#account)
- [Joining and hosting](#joining-and-hosting)
- [Titles](#titles)
- [Tools and automated modes](#tools-and-automated-modes)
- [Shadow computation](#shadow-computation)
- [Window and process](#window-and-process)
- [Paths and files](#paths-and-files)
- [Graphics, input and audio](#graphics-input-and-audio)
- [Network](#network)
- [Development](#development)
- [Logging](#logging)
- [NextCmdLine.txt](#nextcmdlinetxt)
- [Examples](#examples)
- [Open questions](#open-questions)
- [Where this comes from](#where-this-comes-from)

How to read the descriptions:

- No mark: read directly from the code.
- *(inferred)*: based on names or surrounding code.
- *(no reader found)*: the value is stored, and a search of the likely code found nothing that reads it. This doesn't prove that it's unused.
- *(not traced)*: the argument is read, but what happens afterwards wasn't followed.

## Quick reference

| Task | Arguments |
|---|---|
| [Open a map, replay or link](#opening-files-and-links) | `/file`, `/url` |
| [Log in](#account) | `/profile`, `/login`, `/password`, `/validation`, `/nickname`, `/uplayemail`, `/uplaypassword` |
| [Join a server](#joining-and-hosting) | `/join`, `/spectate`, `/joinasreferee`, `/joinpassword`, `/serverpassword`, `/lan`, `/silent`, `/team` |
| [Host a server](#joining-and-hosting) | `/server`, `/serverplugin`, `/allowoutdatedlightmaps`, `/consolededicatedserver` |
| [Start in a title](#titles) | `/title`, `/testtitle`, `/startuptitle` |
| [Create a title pack](#titles) | `/createpack` |
| [Read a Gbx as JSON](#parsegbxfile) | `/parsegbx` |
| [Validate replays](#validatepathpath) | `/validatepath`, `/checksolorecords` |
| [Generate the script API documentation](#generatescriptdocfile) | `/generatescriptdoc`, `/editplugins` |
| [Shoot a video](#shootvideoreplay) | `/shootvideo` |
| [Benchmark](#bench-benchfile) | `/bench`, `/list_benchs`, `/out` |
| [Compute shadows](#shadow-computation) | `/computeallshadows`, `/LmQuality`, `/titles`, `/collections`, `/folder`, `/user`, `/useronly`, `/fullcheck`, `/upgrademaps`, `/thumbnail`, `/only_embedded` |
| [Control the window and process](#window-and-process) | `/windowless`, `/inputless`, `/kiosk`, `/singleinst`, `/windowtitle`, `/nodaemon`, `/uplay` |
| [Change paths](#paths-and-files) | `/usecurrentdir`, `/config`, `/ini`, `/userdir`, `/noscancache`, `/checkdataintegrity` |
| [Change graphics, input and audio](#graphics-input-and-audio) | `/viewport`, `/d3d_max_level`, `/d3d_debug_layer`, `/enablehmd`, `/noxinput`, `/noopenvrinput` |
| [Change network settings](#network) | `/offline`, `/proxy`, `/nadeoservices_hostname`, `/nadeoservices_port`, `/nadeoservices_unsecure_id`, `/specdelay` |
| [Develop plugins and stations](#development) | `/editplugins`, `/additionalpluginurl`, `/forcestationurl`, `/skinchooser`, `/autoregendata` |
| [Control logs](#logging) | `/nologs`, `/logfilename` |

## Syntax

```bat
ManiaPlanet.exe /switch /name=value "/name=value with spaces"
```

- Names are case-insensitive.
- The name is everything before the first `=`, and it has to match completely.
- `/name value` doesn't work, the `=` is required.
- If an argument is repeated, **the first one wins**.
- Unknown arguments are silently ignored.
- Values with spaces are quoted, as either `"/name=a b"` or `/name="a b"`.

An argument without a `/` is converted:

| Argument | Becomes |
|---|---|
| Contains `://` | `/url=<argument>` |
| Has the `.gbx` extension | `/file=<argument>` |
| Anything else | Ignored |

### Types

| Type | Meaning |
|---|---|
| Switch | On when given, except with the values listed below |
| Presence | On whenever given, whatever the value is |
| Number | A natural number. The default is used if it can't be parsed. |
| Text | The value as a string |
| File | A file system path, converted to the game's format |

A switch is off if its value is `0`, `false`, `no`, `n`, `f` or starts with `disab` (case-insensitive). Anything else turns it on, including an empty value and `off`.

In this document, an argument shown without a value is a switch, unless it's described otherwise.

> [!WARNING]
> The game writes its whole command line to its log, so values given to `/password` and `/uplaypassword` end up there in plain text.

## Opening files and links

### `/file=<path>`

Opens an existing file:

- A map is played.
- A replay is viewed.
- Any other type is only logged.

A file that isn't in one of the game's own locations is first copied to the cache. If both `/file` and `/url` are given, the file is handled first.

### `/url=<link>`

Opens a `maniaplanet://` link, or a command starting with `#`. It also **forces the game into a window**.

| Command | Action |
|---|---|
| `#join=<server>` | Join as a player |
| `#spectate=<server>` | Join as a spectator |
| `#qjoin=<server>` | Join as a player, with the silent flag |
| `#qspectate=<server>` | Join as a spectator, with the silent flag |
| `#joinasreferee=<server>` | Join as a referee |
| `#addbuddy=<login>` | Add a buddy |
| `#invitebuddy=<login>` | Invite a buddy |
| `#addfavorite=<server>` | Add a favorite server. `#addfavourite=` works too. |
| `#mailto=<value>` | Write a message |
| `#score=<value>` | Start a direct score challenge |
| `#opponent=<value>` | Start the campaign against an opponent |
| `#campaign=<value>` | Play a campaign |
| `#openstore=<value>` | Open the store. A `:0`, `:1` or `:2` suffix selects the mode. |
| `#menustations=<value>` | Open the stations menu |
| `#menutitle=<value>` | Open a title's menu |
| `#channel=<value>` | Open a channel |
| `#channel_server=<value>` | Join a channel server |
| `#channel_server_spectate=<value>` | Spectate a channel server |

A `maniaplanet://` link without a command is opened as `http://<rest>`, or as `<rest>` unchanged if it starts with `:` (the colon is removed).

## Account

| Argument | Description |
|---|---|
| `/profile=<name>` | Profile to use |
| `/login=<login>` | Account login. Defaults to `/nadeoservices_unsecure_id`. It goes through the login character filter. |
| `/password=<password>` | Account password |
| `/validation=<value>` | Stored as the account validation value *(not traced)* |
| `/nickname=<name>` | Nickname, ignored if empty |
| `/uplayemail=<email>` | Uplay email |
| `/uplaypassword=<password>` | Uplay password |

## Joining and hosting

### Joining

Only one of these is used, in this order of priority:

1. `/join=<server>`: as a player
2. `/spectate=<server>`: as a spectator
3. `/joinasreferee=<server>`: as a referee

`<server>` has this format, where the brackets mark optional parts:

```
address[:password[:lanOrPort]][@titleId]
```

- `lanOrPort` as `1` or another true boolean turns on LAN.
- `lanOrPort` as a number above 1 turns on LAN and is appended to the address as the port.
- **`address:port` doesn't work**, as the port would be taken as the password. Leave the password empty instead: `192.168.1.10::2350`.
- Everything after the first `@` is the title ID, so a password in this format can contain neither `@` nor `:`. Use `/joinpassword` for those.

These go with it:

| Argument | Description |
|---|---|
| `/joinpassword=<password>` | Server password. Takes priority over the one in `<server>`. |
| `/serverpassword=<password>` | Used if `/joinpassword` is empty or missing |
| `/lan` | LAN mode. Also affects the network settings, see [Network](#network). |
| `/silent` | The same flag as in `#qjoin`, which also skips the intro. Only read when windowless or when there's a server to join. Always on with `/inputless`. |
| `/team=<team>` | `0` or `blue`, `1` or `red`. The color names are case-insensitive. |

### Hosting

| Argument | Description |
|---|---|
| `/server` | Starts the flow of hosting a server. The three join arguments and both password arguments are then not read. |
| `/serverplugin=<value>` | Stored as the server plugin |
| `/allowoutdatedlightmaps` | Lets the server accept maps it would reject with `Map lightmap is not up to date` |
| `/consolededicatedserver` | See below |

`/consolededicatedserver` has these effects:

- Without `/title` and `/testtitle`, the title becomes `TMTurbo@nadeolabs`.
- The P2P server port becomes `0`.
- Banning players is disabled.
- The multiplayer Internet play time limit is enabled (`MultiInternetPlayTimeLimit`).

## Titles

### `/title=<titleId>`

Starts in single title mode. This is a title ID, not a window title.

- If the title is installed but not playable, the game quits.
- If it isn't installed, the argument is dropped and startup continues.

### `/testtitle=<name>`

Starts in single title mode with the title from `WorkTitles\<name>` in the user directory. Takes priority over `/title`. If it can't be loaded, an error is shown and the game quits.

### `/startuptitle=<titleId>`

The title to select in the menu after startup. If it isn't installed, the store is opened with automatic installation. The value survives a relaunch through [NextCmdLine.txt](#nextcmdlinetxt).

### `/createpack=<title>`

Opens the title pack creation menu for this title, then quits.

## Tools and automated modes

### `/parsegbx=<file>`

Prints information about a `.Gbx` or `.Pack.Gbx` file as JSON to the standard output, then exits before the game initializes.

The exit code is `0` on success and `1` on failure. Don't combine it with `/windowless`, which opens a new console and waits for a key press on failure.

Every result has `ClassId` and `ClassName`. The rest depends on the class:

| Class | Fields |
|---|---|
| Map | `Name`, `AuthorLogin`, `TitleId`, `Comment`, `MapUid`, `PlayerModel`, `Environment`, `Decoration`, `Mod`, `MapType`, `MapStyle`, `PasswordProtected`, `IsValidated`, `BronzeTime`, `SilverTime`, `GoldTime`, `AuthorTime`, `LightmapVersion`, `DisplayCost`, `NbLaps`, `CheckpointsPerLaps` |
| Replay | `Ghosts`, an array of ghost information |
| Ghost | Ghost information |
| Pack | `ContentsChecksum`, `IsImpostor`, `AuthorLogin`, `UsageSubDir`, `TitleId`, `Comment`, `InfoManialinkUrl`, `DownloadUrl`, `IncludedPacks`, `TitleDescXml`, `PublishedFiles` |

### `/validatepath=<path>`

Validates a replay, a ghost or a folder of them instead of going to the menus, then exits. It only has to be present.

- An empty value or `.` validates the `Replays` folder of the user directory.
- A file can be given by its full name or one relative to `Replays`.
- A folder is looked up in the **root of the user directory** first, then in `Replays`. `ToCheck` finds `Replays\ToCheck`, unless there's also a non-empty `ToCheck` in the root.
- Results and a summary are written to `ValidateLog.txt` in the user directory, unless `/nologs` is given.
- With `/windowless`, progress is printed to the console. Otherwise it's shown in a dialog that can be canceled.

### `/checksolorecords`

With `/validatepath`, skips applying the race settings that are stored for validation (`UpdateFromSettingsForValidation`) before simulating. Unlike in TrackMania Forever, it's a switch.

### `/generatescriptdoc=<file>`

Writes the ManiaScript API documentation to the file, then quits. Implies `/windowless` and `/inputless`. Add `/editplugins` to include the `nadeointernal` classes.

### `/shootvideo=<replay>`

Shoots the video of a replay without going through the menus, then exits. Forces a window.

- The name is either a full one or relative to `Replays`.
- Several replays can be given, separated by `;`. Quote the argument in shells where `;` has a meaning.

A login can be put in front of the name, with literal brackets:

```
/shootvideo=[Login=somelogin]MyReplay.Replay.Gbx
```

The value is converted to ASCII.

### `/bench`, `/bench=<file>`

Benchmark mode, which exits when it's done. It only has to be present, so `/bench=0` enables it too, with `0` as the file name.

- With a file, that replay is benchmarked. The name is either a full one or relative to `Replays`.
- Without a file or list, it forces a window, measures the graphics performance and saves it.

### `/list_benchs=<file>`

Benchmarks the replays in a text file with one name per line. Lines starting with `//` are skipped. A file given to `/bench` takes priority over the list.

> [!CAUTION]
> **The list file is overwritten.** It's written back with the results added as `//` comments, even if it was read-only. Use a copy.

### `/out=<file>`

The file name for the results of `/bench=<file>`, inside the `Bench` folder. The default is `Bench000.txt` and so on. It isn't used with a list.

## Shadow computation

`/computeallshadows` computes the lightmaps of maps by title, then quits. It also sets the graphics adapter index to 1 and forces a window. Lightmaps have to be enabled in the settings and supported by the hardware.

By default, it only processes maps in the game's data location and skips those with up-to-date lightmaps. These options change that:

| Option | Description |
|---|---|
| `/LmQuality=<quality>` | `Very fast`, `Fast`, `Default`, `High` or `Ultra` (case-insensitive). Anything else means `Default`. |
| `/titles=<id,id>` | Titles to process, separated by commas. Ignored in single title mode (`/title`, `/testtitle`). |
| `/collections=<name,name>` | Only these collections, separated by commas (case-insensitive) |
| `/folder=<path>` | Only maps whose folder path starts with this (case-insensitive) |
| `/user` | Removes the restriction to the data location, so user maps are included |
| `/useronly` | Only maps that are loose files in the user location. Implies `/user`. |
| `/fullcheck` | Doesn't skip maps with up-to-date lightmaps |
| `/upgrademaps` | Turns on map fixing for all maps, not just those with a lightmap version below 7. Implies `/fullcheck`. |
| `/thumbnail` | Always takes a new thumbnail and saves the map |
| `/only_embedded` | Lowers the cache level from 2 to 1: only the small lightmap in the map is checked and updated, not the separate lightmap cache file |

> [!CAUTION]
> This mode saves over map files, and requests write access for read-only ones. Back up the maps first.

Separately from the parser, startup searches the raw command line for the text `/computeallshadows` (case-sensitive) and turns off the Windows error dialogs if it's there. That check also matches `/computeallshadows=0` and the text inside other values.

## Window and process

### `/windowless`

- Runs without a window, using a null viewport, and opens a console.
- Implies `/inputless` and turns kiosk mode off.
- If the game exits with a non-zero code, the console **waits for a key press**, which matters in scripts.

### `/inputless`

Runs without input. Also makes joining a server silent.

### `/kiosk`, `/kiosk=<number>`

Kiosk mode `0` to `2`. `/kiosk` alone is mode `1`, and values above `2` fail an assert and become `1`. Any mode but `0` keeps the game locked in the foreground and starts it offline, like [`/offline`](#offline).

### `/singleinst`

If a game window already exists, focuses it, sends it `/url` (or `/file` if there's no URL) and exits. It also exits if another instance is running without a window that can be found.

### `/windowtitle=<text>`

**Has no effect on the window title.** The value is stored, then replaced during initialization, before the window is created. It can only show up as the caption of some early error messages. The title comes from `WindowTitle` in `Nadeo.ini`, or is `ManiaPlanet`.

### `/nodaemon`

**Has no effect.** It clears a flag that is already off by default.

### `/uplay`, `/uplay=<number>`

Activates Uplay PC and sets its application ID, which is `0` when no number is given. `/uplay=0` counts as off.

## Paths and files

The game's root is the directory of the executable, without a trailing `x64` directory.

| Argument | Description |
|---|---|
| `/usecurrentdir` | Uses the current directory instead of the executable's |
| `/config=<file>` | System config file |
| `/ini=<file>` | Replaces `Nadeo.ini`. Appended to the root, so it has to be a **relative path**. If it can't be used, startup fails. |
| `/userdir=<path>` | User directory, see below |
| `/noscancache` | Doesn't use the file scan cache |
| `/checkdataintegrity` | Makes the updater verify the checksums of files |

Values of `/userdir`:

| Value | User directory |
|---|---|
| Empty or `userdocs` | The one from the ini file (the default) |
| `exe` | `<root>\UserData`, and the common directory becomes `<root>\CommonData` |
| Anything else | That path |

## Graphics, input and audio

| Argument | Description |
|---|---|
| `/viewport=null` | Uses the null viewport. Other values do nothing. |
| `/d3d_max_level=<level>` | `10_0` or `10_1`, limits the Direct3D feature level |
| `/d3d_debug_layer` | Presence. Asks whether to use the Direct3D 11 debug layer. |
| `/d3d_debug_layer=error` | The same, with the message logger at level 1 instead of 2 |
| `/enablehmd` | Turns on HMD stereoscopy *(inferred)* and allows HRTF audio |
| `/noxinput` | Disables XInput |
| `/noopenvrinput` | Disables OpenVR input |

## Network

### `/offline`

Starts the game offline, as if "Stay offline" had been chosen:

- The Internet check at startup is skipped.
- The game doesn't connect to the Internet or ask whether to connect.

The `ForceOffline` setting has the same effect.

### Other settings

| Argument | Description |
|---|---|
| `/proxy=<url>` | Forced HTTP proxy |
| `/nadeoservices_hostname=<host>` | Overrides the Nadeo services host |
| `/nadeoservices_port=<number>` | Overrides the Nadeo services port, unless `0` |
| `/nadeoservices_unsecure_id=<id>` | Unsecure user ID, also used as the login if there's no `/login` |
| `/specdelay=<number>` | Prediction delay when spectating. Read once per game session. |
| `/lan` | Besides LAN joining, uses LAN network settings, where the default prediction delay is 50 |

## Development

### `/editplugins`

- Enables the `ButtonEditPlugins` toolbar button.
- Gives script "super powers" to pages from `localhost` and `127.0.0.1`.
- Turns on a script mode in the playground that is otherwise used on `channel_` servers *(not traced)*.
- Includes `nadeointernal` classes in `/generatescriptdoc`.

### `/additionalpluginurl=<url>`

Loads an additional system plugin.

- There are 11 names: `/additionalpluginurl`, then `/additionalpluginurl0` to `/additionalpluginurl9`.
- A plugin replaces the built-in one with the same file name.
- There can be 15 plugins in total.
- Plugin editing is limited to the first additional plugin *(inferred)*.
- They aren't loaded when windowless or in the automated modes.

### `/forcestationurl=<url>`

Replaces the station URL of titles, except for one type of title that keeps its own *(which type wasn't identified)*. Read only once.

### `/skinchooser`

Opens the skin editor menu (`MenuSkinEditor`) once after startup.

### `/autoregendata`

Enables data regeneration (`NSysFidReGen`).

## Logging

### `/nologs`

- Doesn't create `UGCErrorsLog.txt`.
- Doesn't create `ValidateLog.txt`.
- Clears a network flag before the anticheat initializes *(no reader found)*.
- For `ConsoleLog.<pid>.txt` it changes nothing, as that log is already off by default.

### `/logfilename=<file>`

A fixed name for the log file, instead of `ManiaPlanetLog-<date>.txt`. This also skips the cleanup that keeps the number of old logs at 10.

## NextCmdLine.txt

After the early initialization, the game appends the arguments from `Config\NextCmdLine.txt` in the user directory, then deletes that file.

- The real command line takes priority, as the first occurrence of an argument wins.
- Only arguments starting with `/` are taken, without the conversion of files and links.
- Arguments that are read earlier don't work from this file, such as `/parsegbx`, `/windowless` or the paths.

The game itself only writes `/startuptitle` to it, when it exits to run the launcher.

## Examples

These follow from the code and weren't run.

Open a map:

```bat
ManiaPlanet.exe "C:\Maps\My Map.Map.Gbx"
```

Save information about a map as JSON, from a batch file:

```bat
ManiaPlanet.exe "/parsegbx=C:\Maps\My Map.Map.Gbx" > map.json
```

Generate the script documentation, including internal classes:

```bat
ManiaPlanet.exe /generatescriptdoc=C:\Temp\doc.h /editplugins
```

Validate a folder of replays:

```bat
ManiaPlanet.exe /windowless /validatepath=ToCheck
```

Join a password protected server as a spectator:

```bat
ManiaPlanet.exe /login=YOUR_LOGIN /password=YOUR_PASSWORD "/spectate=SERVER_LOGIN:SERVER_PASSWORD@TMStadium@nadeo"
```

Join a LAN server on a specific port:

```bat
ManiaPlanet.exe /join=192.168.1.10::2350
```

Recompute the shadows of all user maps of a title:

```bat
ManiaPlanet.exe /computeallshadows /useronly /fullcheck /LmQuality=High /titles=TMStadium@nadeo
```

## Open questions

These need a run of the game or more tracing:

- What the master server connection does with `/validation`.
- Whether the silent flag does more than skip the intro.
- What the script mode of `/editplugins` in the playground allows.
- How plugin editing behaves with `/additionalpluginurl`.
- Whether `/enablehmd` is enough to get HMD output.
- Which type of title ignores `/forcestationurl`.

## Where this comes from

| Address | Function | What it reads or does |
|---|---|---|
| `14087ce70` | `WinMainInternal` | Conversion of arguments, `/singleinst`, console, kiosk |
| `1406ac850` | `CSystemCommandLine::FindArg` | Case-insensitive lookup |
| `1408776e0` | `CGbxApp::ApplyCommandLineArgs` | Application arguments |
| `140161b70` | `CGbxGame::ApplyCommandLineArgs` | Game arguments |
| `140879ad0` | `CGbxApp::Init1` | `/parsegbx` |
| `140878020` | `Dedicated_ParseGbx` | JSON output |
| `140162dd0` | `CGbxGame::PreInitApp` | Overwrites the window title |
| `140162130` | `CGbxGame::InitApp` | `/generatescriptdoc` |
| `140162c70` | `CGbxGame::IsForceWindowed` | Forced window |
| `1406b7170` | `CSystemEngine::ComputePaths` | `/ini`, `/userdir` |
| `14087ba10` | `CGbxApp::Log_SetOutputFile` | `/logfilename` |
| `14087c900` | `ReadNextCmdLine` | `NextCmdLine.txt` |
| `14096f740` | `CGameCtnApp::UpdateGame_Init` | Account, joining |
| `140962f5e` | In `CGameCtnApp::UpdateGame` (not disassembled by Ghidra) | `/offline` |
| `140168600` | `CFastAssert::StaticInit` | Raw search for `/computeallshadows` |
| `140bbfec0` | `CGameCtnApp::CmdBench` | Benchmarks |
| `140bc11f0` | `CGameCtnApp::CmdBenchReplayList` | Benchmark list, written back |
| `1412d2a80` | `CTrackManiaRace::Validate` | `/checksolorecords` |
| `1408e4120` | `CGameApp::CmdLineParseJoinServerString` | Join string |
| `1408e4420` | `CGameApp::CmdLineUrlMptp_Check` | Link commands |
| `140a99e50` | `CGameManiaPlanet::Start` | Titles |
| `140a878c0` | `CGameManiaPlanet::InitAfterProfiles` | Pack creation, shadows, video, skin chooser |
| `140a7f610` | `CGameManiaPlanet::ComputeAllChallengeShadowsByTitle` | Shadow options |
| `1412b98a0` | `CTrackMania::UpdateGame_StartUp` | `/validatepath` |
| `1412b6540` | `CTrackMania::ReplayValidatePath` | Validation and its log |
| `140ac3930` | `CGameNetwork::InitNet` | Proxy, Nadeo services |
| `140ab03f0` | `CGameSystemOverlay::Plugins_Update` | Additional plugins |

The list of arguments comes from a byte by byte scan of the executable for calls to the `CSystemCommandLine` getters and for references to narrow and wide argument-like strings, which includes code that Ghidra didn't disassemble. An argument name built at runtime wouldn't be found.

The code also has a check for `/parsegbx` or `/validatepath` at the end of `CGbxApp::ApplyCommandLineArgs`. It requires a console state that is only set after that function returns, so it isn't taken during a normal startup, and neither argument depends on `/windowless`.
