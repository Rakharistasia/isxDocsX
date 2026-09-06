# Getting Started

This chapter covers loading ISXLUA, writing your first script, running and
stopping scripts, passing arguments, and what the Lua environment gives you.

## Loading ISXLUA

From the InnerSpace console:

```
ext ISXLUA
```

You will see a load banner such as `ISXLUA v20260906.143107 (Lua 5.4.9)`. The
version carries a build-time suffix so you can tell same-day rebuilds apart;
`${ISXLUA.Version}` reports the same string.

ISXLUA attaches to no particular game -- it is a Lua runtime for InnerSpace. To
automate a game you also load that game's own InnerSpace extension (its `ext`
command).

## Your first script

Create a file called `hello.lua` in your InnerSpace **Scripts** directory:

```lua
echo("Hello from Lua!")
```

Run it from the console:

```
lua hello
```

The `.lua` extension is optional -- `lua hello` and `lua hello.lua` are the same.

Note that you use `echo(...)` (or `print(...)`), **not** Lua's raw output. Lua's
built-in `print` writes to standard output, which is invisible in-game; ISXLUA
replaces `print` and adds `echo` so both go to the InnerSpace console. See
[`02_The_IS_Bridge.md`](02_The_IS_Bridge.md).

## Running and stopping scripts

| Command | Effect |
|---|---|
| `lua <name> [args...]` | Run a `.lua` script. |
| `lua -c "<chunk>"` | Run an inline one-liner (see below). |
| `endlua <name>` | Stop one running script. |
| `endlua all` (or `endlua *`) | Stop every running Lua script. |
| `lua -pause <name>` | Freeze a running script (see [Pausing, resuming, and reloading](#pausing-resuming-and-reloading-scripts)). |
| `lua -resume <name>` | Un-freeze a paused script. |
| `lua -reload <name>` | Restart a script from its file with the same arguments. |
| `luas` | List the running Lua scripts and how long each has been running (a frozen one is tagged `[PAUSED]`). |

The `-pause` / `-resume` / `-reload` forms also accept `all` (or `*`) to act on every running script at once.

**Name resolution** works like the native `run` command: the name is looked for
(1) as given (an absolute path, or relative to the current game directory), then
(2) in the InnerSpace **Scripts** directory, then (3) in the InnerSpace base
directory. The first file found wins.

**Names are matched loosely** -- without the directory, without the `.lua`
extension, case-insensitively. So `endlua Hello` stops the script started by
`lua hello.lua`.

**One instance per name.** Starting a script whose name is already running is
refused (just like `run`). Different scripts run **simultaneously**, each fully
isolated in its own Lua state -- one script's globals never leak into another's.

### The native `run` / `endscript` commands also work

ISXLUA registers a Lua script engine, so InnerSpace's built-in commands drive
`.lua` files too:

```
run hello.lua
endscript hello
```

A script started with `run` and one started with `lua` share the **same**
running-script set, so `endlua`, `endscript`, and `luas` all see both.

> **One limitation to know:** the native `scripts` / `scripts -running` listing
> does **not** show `.lua` scripts (its table is LavishScript-scoped). This is an
> InnerSpace-side constraint, not something ISXLUA can change. **Use `luas` to
> list running Lua scripts.**

## One-liners: `lua -c "<chunk>"`

To run a snippet of Lua straight from the console without creating a file, use the
`-c` flag (`-e` works too):

```
lua -c "print(1 + 2)"
lua -c "print(ISXLUA.Version)"
```

The chunk runs immediately, to completion, with the full ISXLUA environment
available (the `IS` bridge, the object model, the bundled libraries). Quote the
chunk so the console passes it through as a single argument.

This is ideal for quickly testing an expression. A one-liner is **not** a scheduled
script, so it **cannot** use `wait()` / `waitframe()` / `waituntil()` (an attempt is
reported as an error). For anything that needs to wait, put it in a `.lua` file and
run it with `lua <name>`.

## Autoexec: `autoload.lua`

When ISXLUA finishes loading, it looks for a file named **`autoload.lua`** in your
InnerSpace **Scripts** directory and, if it exists, runs it automatically -- exactly
as if you had typed `lua autoload`. If the file is not there, nothing happens (no
error, no message).

This is the place to start whatever you always want running: load a toolbar, attach
your event handlers, kick off a background helper. A simple example:

```lua
-- autoload.lua
echo("autoload: starting up")
IS.Execute("lua mybot")     -- start another script
```

`autoload.lua` runs once each time the extension loads, so reloading ISXLUA
(`ext -unload ISXLUA` then `ext ISXLUA`) runs it again. Running `endlua all` (or
`endlua autoload`) does **not** re-run it -- it fires only on a fresh extension load.

### Per-game autoexec: `autoload_<game>.lua`

Right after `autoload.lua`, ISXLUA also looks for a **per-game** autoexec file and
runs it if the matching game extension is loaded and ready. The file is named
`autoload_<game>.lua`, where `<game>` is the short tag for the extension:

| If this game extension is loaded | ISXLUA also runs |
|---|---|
| ISXEQ2 | `autoload_eq2.lua` |
| ISXEVE | `autoload_eve.lua` |
| ISXPantheon | `autoload_pantheon.lua` |

This is the place to put startup that only makes sense for one game -- e.g. an
EverQuest II toolbar in `autoload_eq2.lua`, an EVE Online mining helper in
`autoload_eve.lua`. `autoload.lua` (which always runs first) stays the place for
game-independent setup.

A few details worth knowing:

- **It is silent when there is nothing to do.** If the file is absent, or no
  matching game extension is loaded, nothing happens (no error, no message) --
  exactly like `autoload.lua`.
- **A late-loading game extension is still caught.** A game extension sometimes
  finishes loading a moment *after* ISXLUA. ISXLUA keeps watching for a short,
  bounded window (about 30 seconds) after it loads, so `autoload_<game>.lua` still
  runs once the game extension becomes ready. After that window it stops looking.
- **Each file runs once per ISXLUA load**, and it does not re-run on `endlua all`.
- **More than one at a time is fine.** If several game extensions are loaded in the
  same session, each one's `autoload_<game>.lua` runs independently.

## Pausing, resuming, and reloading scripts

You can freeze a running script and later thaw it, or restart it from disk, without
losing the other scripts that are running.

| Command | Lua equivalent | Effect |
|---|---|---|
| `lua -pause <name>` | `IS.PauseScript("<name>")` | Freeze the script. |
| `lua -resume <name>` | `IS.ResumeScript("<name>")` | Un-freeze it. |
| `lua -reload <name>` | `IS.ReloadScript("<name>")` | Stop it and re-run it from its file. |

**What pausing freezes.** A paused script stops advancing: its `wait()`,
`waitframe()`, `waituntil()`, and `waitforevent()` are all **held**, and its
`setTimeout` / `setInterval` timers stop firing. The time remaining is *preserved*,
not consumed -- if a script was 6 seconds into a `wait(10)` when you paused it, it
still has 4 seconds left when you resume it (even if it was paused for an hour). An
event awaited with `waitforevent()` that fires while the script is paused is
remembered and delivered when you resume.

**What pausing does *not* freeze.** Anything that is not the script's own step-by-step
flow keeps working: persistent event handlers you attached with `IS.AttachEvent`
still fire, functions you published with `IS.Register` are still callable from
LavishScript, and messages published on the bus are still delivered. Pause suspends a
script's *main flow and timers*, not its callbacks.

**Reloading** stops the script and starts it again from its file, with the same
arguments it was originally given -- the quickest way to pick up an edit you just
saved. A reload happens on the next frame, which means a script is even allowed to
reload *itself*:

```lua
-- reload myself when the user asks
IS.AttachEvent("MyReloadRequested", function() IS.ReloadScript() end)
```

Called with no name, `IS.PauseScript()`, `IS.ResumeScript()`, and `IS.ReloadScript()`
act on the **calling** script; pass a name to control another script. To pause
yourself and wait to be resumed by someone else, pause and then yield:

```lua
IS.PauseScript()        -- freeze myself...
waitframe()             -- ...and hand control back; I resume when something calls IS.ResumeScript("me")
```

## When a script errors

If your script (or an event handler) hits an uncaught error, ISXLUA prints a full
Lua **traceback** to the console -- the message plus the call stack with file names
and line numbers -- so you can see exactly where it happened. The error is confined
to that script or handler; it never crashes the game or other scripts.

## Script arguments

Anything after the script name is passed to your script two ways: as the chunk's
varargs (`...`) and as a global 1-indexed table named `args`.

```lua
-- args.lua   ->   lua args hello 42
echo("first arg:  " .. tostring(args[1]))   -- hello
echo("second arg: " .. tostring(args[2]))   -- 42

-- ... works too:
local a, b = ...
echo(a .. " / " .. b)
```

Because the `lua` command parses LavishScript first, a `${...}` in your arguments
is evaluated before the script sees it. For example `lua showver ${ISXLUA.Version}`
passes ISXLUA's version string as `args[1]`.

## The `${ISXLUA}` object

ISXLUA exposes its own status through a top-level object, reachable from Lua as
the bare global `ISXLUA`:

```lua
echo("ISXLUA version: " .. ISXLUA.Version)
if ISXLUA.IsReady then echo("ISXLUA is ready") end
```

| Member | Type | Meaning |
|---|---|---|
| `ISXLUA.Version` | string | Version string (`<YYYYMMDD>.<HHMMSS>`), same as the banner. |
| `ISXLUA.IsReady` | bool | True once the extension has finished loading. |
| `ISXLUA.IsLoading` | bool | True while the extension is still loading. |
| `ISXLUA.InQuietMode` | bool | True when quiet mode is on. |

| Method | Effect |
|---|---|
| `ISXLUA:QuietMode()` | Toggles quiet mode on/off. |

## What the Lua environment gives you

- **Lua 5.4.9**, the full standard library (`string`, `table`, `math`, `os`,
  `io`, `coroutine`, `utf8`, `package`, ...). ISXLUA does not currently sandbox
  the standard library.
- **`echo` / `print`** -- console output (see [`02_The_IS_Bridge.md`](02_The_IS_Bridge.md)).
- **`wait(seconds)` / `waitframe()` / `waituntil(condfn, timeout)`** -- cooperative
  timing and condition waits (see [`04_Timing_And_Events.md`](04_Timing_And_Events.md)).
- **The `IS` bridge** -- `IS.Execute`, `IS.Parse`, the event functions, and
  `IS.SaveTable` / `IS.LoadTable` for persistent storage ([`02_The_IS_Bridge.md`](02_The_IS_Bridge.md),
  [`04_Timing_And_Events.md`](04_Timing_And_Events.md)).
- **The object model** -- bare-global TLOs and generic member/method access
  ([`03_Object_Model.md`](03_Object_Model.md)).
- **Lifecycle control** -- pause / resume / reload a running script from the console
  or from Lua (`IS.PauseScript` / `IS.ResumeScript` / `IS.ReloadScript`), plus
  autoexec via `autoload.lua` and the per-game `autoload_<game>.lua` (above).
- **Cross-script data sharing** -- a shared value store and a publish/subscribe
  message bus (`IS.Share` / `IS.Shared`, `IS.Publish` / `IS.Subscribe`) that copy
  data between otherwise-isolated scripts ([`04_Timing_And_Events.md`](04_Timing_And_Events.md)).
- **Bundled libraries** -- `require("cjson")`, `require("serpent")`, and more
  ([`06_Bundled_Libraries.md`](06_Bundled_Libraries.md)).

Next: [`02_The_IS_Bridge.md`](02_The_IS_Bridge.md).
