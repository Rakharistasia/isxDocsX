# ISXLUA Scripting Guide

A guide for people who **write Lua scripts** for InnerSpace using **ISXLUA**. It
covers the Lua-facing API only -- how to run scripts, how the `IS` bridge works,
how to reach LavishScript top-level objects and datatypes from Lua, how timing
and events behave, how to build UIs, which libraries are bundled, and the gotchas
you will hit if you are coming from LavishScript. You do not need any source access or C/C++
knowledge to use this guide.

## What ISXLUA is

ISXLUA is an InnerSpace extension that embeds **Lua 5.4.9** directly. It lets you
automate InnerSpace and any game extension your InnerSpace session has loaded from
Lua instead of LavishScript. The whole Lua core and standard library are built in --
there is no separate Lua install, no DLL, nothing extra to ship. Load it from the
InnerSpace console with:

```
ext ISXLUA
```

When it loads you will see a banner like `ISXLUA v20260906.143107 (Lua 5.4.9)`,
and `${ISXLUA.Version}` will report the same version.

## The files in this guide

Read them in order the first time; after that use them as a reference.

| File | What it covers |
|---|---|
| **[`README.md`](README.md)** (this file) | Orientation and the file map. |
| [`01_Getting_Started.md`](01_Getting_Started.md) | Running scripts (`lua` / `endlua` / `luas`, and native `run` / `endscript`), autoexec (`autoload.lua`, plus the per-game `autoload_<game>.lua`), pausing / resuming / reloading a running script, script arguments, the `${ISXLUA}` object, what the Lua environment gives you. |
| [`02_The_IS_Bridge.md`](02_The_IS_Bridge.md) | The `IS` global -- `IS.Execute` / `IS.Parse` (escape hatches into LavishScript), `print` / `echo`, persistent storage (`IS.SaveTable` / `IS.LoadTable` and the hierarchical `IS.Settings` config store), and the reverse bridge (call Lua from LavishScript). |
| [`03_Object_Model.md`](03_Object_Model.md) | The heart of ISXLUA: bare-global TLOs (`ISXLUA`, plus whatever your loaded game extension provides), `.Member` / `.Member(args)`, `:Method(args)`, native scalar values, the typed getters, `Exists()`, and `obj[i]` indexing. |
| [`04_Timing_And_Events.md`](04_Timing_And_Events.md) | `wait(seconds)` / `waitframe()` / `waituntil` / `waitforevent`, timers (`setTimeout` / `setInterval` / `clearTimer`), the events layer (`IS.AttachEvent` / `IS.AttachEventTyped` / `IS.DetachEvent` / `IS.FireEvent` / `IS.EventSource`) with atomic handlers, sharing data between scripts (the `IS.Share` / `IS.Shared` value store and the `IS.Publish` / `IS.Subscribe` message bus), and asynchronous HTTP (`IS.HttpGet` / `IS.HttpPost` -- "with libisxgames" build only). |
| [`05_Building_GUIs.md`](05_Building_GUIs.md) | Building and driving **LavishGUI 2** UIs from Lua with the bundled `lgui2` module -- load a package (a `.json` file or an inline Lua table), find elements, read / set their state, show / hide them, and wire a button straight to a Lua callback. |
| [`06_Bundled_Libraries.md`](06_Bundled_Libraries.md) | The `require`-able modules that ship inside ISXLUA (`cjson`, `lfs`, `lpeg`, `serpent`, `inspect`, `json`, `re`, `socket` networking, `zlib` compression, `lsqlite3` database, `lgui2`, `middleclass` classes, the `Penlight` `pl.*` standard-library extensions) and how to load your own loose modules. |
| [`07_Migration_Gotchas.md`](07_Migration_Gotchas.md) | The differences that will bite a LavishScript scripter moving to Lua. **Read this if you know LavishScript.** |
| [`08_Examples.md`](08_Examples.md) | Complete, runnable scripts -- simple (hello world, reading game data, a wait loop, an event handler) and larger realistic ones that combine them. (These are concrete and game-specific, unlike the generic topics above.) |

## The five-minute version

- Save a `.lua` file in your InnerSpace **Scripts** directory and run it with
  `lua myscript` (the `.lua` is optional). `endlua myscript` stops it; `luas`
  lists what is running.
- **Top-level objects are bare Lua globals.** `ISXLUA.Version`, and every TLO your
  loaded game extension provides (e.g. `SomeTLO.SomeMember`, `SomeTLO(1)`), just
  work.
- **Member arguments use parentheses, methods use a colon:**
  `SomeTLO.Lookup("name").SomeMember`, `SomeTLO(1):SomeMethod()`.
- **Scalar results come back as real Lua values** (numbers, strings, booleans),
  so `SomeTLO.SomeNumber == 95` and `ISXLUA.Version:upper()` work directly.
- **`wait()` takes SECONDS**, not tenths of a second like LavishScript.
- Coming from LavishScript? Read [`07_Migration_Gotchas.md`](07_Migration_Gotchas.md) before anything else.

## A note on scope

This guide documents the **Lua-facing behavior** of ISXLUA. The list of members
and methods available on any given top-level object is not defined by ISXLUA --
it comes from whichever game extension your InnerSpace session has loaded. For
those, consult that extension's own scripting guide or quick reference; every
member/method it documents is reachable from Lua exactly as described here.

The examples are kept in sync as ISXLUA features are added.
