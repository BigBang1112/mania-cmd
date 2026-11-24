# TrackMania Sunrise command line arguments

Command line arguments of `TmSunrise.exe`, found by reading its code in Ghidra. None of them were tested by running the game.

- [Quick reference](#quick-reference)
- [Syntax](#syntax)
- [General](#general)
- [Benchmark](#benchmark)
- [Examples](#examples)
- [Where this comes from](#where-this-comes-from)

How to read the descriptions:

- No mark: read directly from the code.
- *(inferred)*: based on names or surrounding code.
- *(no reader found)*: the value is stored, and a search of the likely code found nothing that reads it. This doesn't prove that it's unused.
- *(not traced)*: the argument is read, but what happens afterwards wasn't followed.

## Quick reference

| Argument | Description |
|---|---|
| [`/config=<file>`](#configfile) | System config file |
| [`/title=<text>`](#titletext) | Text added to the window title |
| [`/data=<path>`](#datapath) | Replaces the `GameData` directory |
| [`/game=<value>`](#gamevalue) | Accepted, but no use of it was found |
| [`/netip=<value>`](#netipvalue) | Accepted, but no use of it was found |
| [`/bench`](#bench) | Measure the graphics performance, then exit |
| [`/bench=<file>`](#benchfile) | Benchmark a replay, then exit |
| [`/list_benchs=<file>`](#list_benchsfile) | Benchmark a list of replays, **write the results into the list**, then exit |
| [`/out=<file>`](#outfile) | File name of the benchmark results |

There are no arguments for login, joining a server, opening a file or running without a window.

## Syntax

```bat
TmSunrise.exe /name=value /name=value
```

The parser is much simpler than in the later games:

- **Arguments are split at spaces, and quotes have no meaning.** A value can't contain a space.
  - `/title=My Game` sets the title to `My`.
  - `/title="My Game"` sets the title to `"My`, including the quote.
  - `"/title=Game"` isn't recognized at all, as it doesn't start with `/`.
- Tabs don't separate arguments.
- Names are case-insensitive.
- Values are attached with `=`. Apart from `/bench`, an argument without its `=` isn't recognized, so `/config` alone is unknown.
- `/bench` only has to match as a prefix, so `/benchXYZ` also enables benchmark mode.
- Arguments are processed in order, so if one is repeated, **the last one wins**.
- An unknown argument starting with `/` is logged as `Unknown command line argument ignored` and skipped.
- An argument without a `/` is stored as if it was given with `/game=`.
- The command line is read as ANSI.

## General

### `/config=<file>`

The system config file to load. The default is `Default.SystemConfig.Gbx`. If the file can't be loaded, a new config is created and **saved under that name**.

### `/title=<text>`

Text added to the window title, which becomes `TmSunrise - <text>`. It doesn't replace the title.

### `/data=<path>`

Uses `<current directory>\<path>` as the game data directory instead of `<current directory>\GameData`. The path is always appended to the current directory, so it has to be **relative**.

### `/game=<value>`

Stored *(no reader found)*. The game always loads `TrackManiaSunrise\TrackMania.gbx`.

### `/netip=<value>`

Stored *(no reader found)*.

## Benchmark

The game exits when the benchmark is done.

### `/bench`

Enables benchmark mode. Without a file or a list, it forces the game into a window, measures the graphics performance and saves it.

### `/bench=<file>`

Benchmarks this replay. An empty value is the same as `/bench`. Which folder a relative name starts from is *(not traced)*.

The results are written to the `Bench` folder of the game's user location *(inferred)*, as `Bench000.txt` and so on, or with the name given to `/out`.

### `/list_benchs=<file>`

Benchmarks the replays in a text file with one name per line. Lines starting with `//` are skipped. A file given to `/bench` takes priority over the list.

> [!CAUTION]
> **The list file is overwritten.** It's written back with the results added as `//` comments, even if it was read-only. Use a copy.

### `/out=<file>`

The file name for the results of `/bench=<file>`, inside the `Bench` folder. It isn't used with a list, and doesn't enable benchmark mode by itself.

## Examples

These follow from the code and weren't run.

Use another system config and label the window:

```bat
TmSunrise.exe /config=Other.SystemConfig.Gbx /title=Test
```

Run the benchmark:

```bat
TmSunrise.exe /bench
```

## Where this comes from

The executable has no symbols, so the roles of these functions were worked out from their code.

| Address | Role |
|---|---|
| `0043da30` | WinMain, splits the command line |
| `00419170` | Tokenizer |
| `004014a0` | Game argument handler: `/list_benchs=`, `/bench`, `/out=` |
| `004103f0` | Application argument handler: everything else |
| `004178d0` | Prefix comparison |
| `00401620` | Game initialization |
| `00401970` | Game start, passes the benchmark values on |
| `00401c00` | Decides whether to force a window |
| `00410a50` | Application initialization, uses `/config=` and `/data=` |

The command line only reaches the two handlers. The executable was also scanned byte by byte for references to the handlers and to argument-like strings, which includes code that Ghidra didn't disassemble. No other arguments were found.

*(no reader found)* here covers the code of the two application classes and the uses of the global application object.
