# The IS Bridge

Every script gets a global table named **`IS`** -- your direct line into
InnerSpace and LavishScript. Most of the time you will use the object model
([`03_Object_Model.md`](03_Object_Model.md)) instead, because it is cleaner and returns real Lua
values. But `IS.Execute` and `IS.Parse` are the escape hatches: anything the
object model cannot express, you can still do through the string API.

`IS` also holds the console output helpers and the event functions (events are
covered separately in [`04_Timing_And_Events.md`](04_Timing_And_Events.md)).

## `IS.Execute(command)` -- run a command

Runs an InnerSpace / LavishScript command, exactly as if you had typed it at the
console. Returns the command's integer result.

```lua
IS.Execute("echo hello from a command")
IS.Execute("ext yourgame")          -- load a game extension
IS.Execute("run otherscript")       -- run another script
```

Use this to trigger commands that have no object-model equivalent (loading
extensions, running other scripts, invoking console commands).

## `IS.Parse(dataSequence)` -- evaluate a `${...}` expression

Evaluates a LavishScript data sequence and returns the result as a string, or
`nil` if it could not be evaluated. **Pass the full `${...}` form.**

```lua
local ver = IS.Parse("${ISXLUA.Version}")
echo("ISXLUA version is " .. tostring(ver))

-- A numeric member: IS.Parse still returns text, so convert it yourself.
local n = tonumber(IS.Parse("${SomeTLO.SomeNumber}"))
```

`IS.Parse` always returns a **string** (or nil), so convert it yourself with
`tonumber` when you need a number. Compare this with the object model, where
`SomeTLO.SomeNumber` comes back as a real Lua number already. (`SomeTLO` /
`SomeNumber` here are placeholders -- use the real top-level objects and members
your loaded game extension provides.)

> **8 KB limit.** `IS.Parse` returns at most 8 KB of text; a longer result comes
> back as `nil` rather than truncated. The object model does not have this limit
> (it reads values directly), so prefer it for large data.

**When to use `IS.Parse` instead of the object model:** the rare case of a member
that returns a scalar with no arguments but *also* takes arguments for a
different result. The object model gives you the native scalar and you cannot
then pass arguments to it; `IS.Parse("${...}")` lets you build the full
expression by hand. This is uncommon -- see [`03_Object_Model.md`](03_Object_Model.md) and
[`07_Migration_Gotchas.md`](07_Migration_Gotchas.md).

## `print(...)` and `echo(...)` -- console output

Both write to the InnerSpace console. Arguments are converted with `tostring`
(honoring a value's `__tostring`, so object wrappers print sensibly) and
separated by tabs.

```lua
print("x", 1, true)          -- x    1    true
echo("Version is " .. ISXLUA.Version)
echo(SomeObject)             -- object wrappers coerce to text
```

They are interchangeable; `echo` exists because it reads naturally to
LavishScript scripters, and `print` is replaced so the standard Lua idiom still
produces visible output (raw Lua `print` goes to stdout, which you cannot see
in-game).

Your script text is never treated as a format string, so `echo(userText)` is safe
even if `userText` contains `%` characters.

## `IS.WarnUnknownGlobals(enabled)` -- control the unknown-global warning

When you read a global that is neither a Lua value nor a LavishScript top-level
object, ISXLUA returns `nil` and prints a one-time warning for that name (to help
you catch typos). Turn the warning off if you intentionally probe possibly-absent
globals:

```lua
IS.WarnUnknownGlobals(false)   -- silence the warning
```

See [`03_Object_Model.md`](03_Object_Model.md) for how bare-global lookup works.

## `IS.SaveTable(name, tbl)` / `IS.LoadTable(name)` -- persistent storage

These let a script remember state between runs and sessions. `IS.SaveTable`
serializes a Lua table to disk under a name of your choosing; `IS.LoadTable` reads
it back.

```lua
-- Save some state:
IS.SaveTable("mystate", { runs = 3, best = 128, names = { "a", "b" } })

-- ...later, in another run:
local state = IS.LoadTable("mystate") or {}   -- nil if never saved -> default to {}
state.runs = (state.runs or 0) + 1
IS.SaveTable("mystate", state)
echo("run #" .. state.runs)
```

- **`name`** is a logical key, not a path. Files are stored together in an
  `ISXLUAData` folder inside your InnerSpace **Scripts** directory, as
  `<name>.lua`. The name is sanitized (only letters, digits, `_`, `-`, and `.` are
  kept; anything else becomes `_`), so it can never point outside that folder.
- **`IS.SaveTable`** returns `true` on success, or `false` plus an error message if
  the file could not be written.
- **`IS.LoadTable`** returns the restored table, or **`nil`** if the file does not
  exist (or could not be parsed). The `... or {}` idiom above is the clean way to
  load-with-a-default.
- Only a **table** can be saved. Its contents must be serializable -- plain values
  and nested tables are fine; functions, userdata, and object wrappers are not.
  (Copy scalar values out of object wrappers before saving them.)

Under the hood this uses the bundled `serpent` library, so a saved file is
human-readable, loadable Lua. See [`06_Bundled_Libraries.md`](06_Bundled_Libraries.md) if you want to drive
the serialization yourself (for example to persist as JSON instead).

## `IS.Settings(name)` -- a hierarchical, persistent config store

Where `IS.SaveTable` stores one flat table under a name, **`IS.Settings`** gives you
a richer, structured configuration store: named **settings** (leaf values) organized
into named **sections** that can nest, saved to and loaded from XML. Use it for
user-facing config -- options, profiles, per-character preferences -- where you want
sections and typed values rather than one big table.

`IS.Settings(name)` returns a **handle** to a config set. The first time you open a
given name in a session, ISXLUA automatically loads its saved file (if one exists),
so your config is there with no extra step; call `handle:Save()` to write changes
back.

```lua
local cfg = IS.Settings("MyBot")     -- open (and auto-load) the "MyBot" config

cfg:Set("enabled", true)             -- store a boolean
cfg:Set("range", 35)                 -- store a number
cfg:Set("ui/window/x", 100)          -- a path: section "ui" -> section "window" -> "x"
cfg:Set("ui/window/y", 240)

local range = cfg:Get("range")       -- 35 (a real number)
local on    = cfg:Get("enabled")     -- true (a real boolean)
local x     = cfg:Get("ui/window/x") -- 100

cfg:Save()                           -- persist to disk (as MyBot.xml)
```

### Values round-trip as the natural Lua type

`Set` accepts a **string, number, or boolean**; passing **`nil` deletes** the
setting. `Get` reads it back and infers the type: `"TRUE"`/`"FALSE"` come back as a
**boolean**, a value that is wholly a number comes back as a **number**, and anything
else comes back as a **string**. So `cfg:Set("n", 5)` then `cfg:Get("n")` gives you
the number `5`.

```lua
local hits = cfg:Get("hits", 0)      -- second arg is a default when the setting is absent
cfg:Set("hits", hits + 1)
```

If you need the **exact stored text** (for example a numeric-looking string like a
zip code that you do not want turned into a number), use `GetString`:

```lua
local zip = cfg:GetString("zip")     -- always a string, no inference
```

### Paths and sections

A setting name can be a **path** with `/` separators: every segment except the last
is a section (created as needed on `Set`). You can also navigate explicitly with
`Section`, which returns a handle to a child section:

```lua
local win = cfg:Section("ui"):Section("window")
win:Set("x", 100)
win:Set("y", 240)
echo(win:Get("x"))                   -- 100

-- Section(name, false) only *finds* an existing section (returns nil if absent):
local maybe = cfg:Section("profiles", false)
```

### Inspecting, saving, and loading

| Method | What it does |
|---|---|
| `handle:Set(path, value)` | store a value (`nil` deletes); creates sections in the path |
| `handle:Get(path [, default])` | read, type-inferred; `default` (or `nil`) if absent |
| `handle:GetString(path [, default])` | read the exact stored string |
| `handle:Exists(path)` | `true` if the setting exists |
| `handle:Delete(path)` | remove a setting |
| `handle:Settings()` | array of the setting names directly in this set |
| `handle:Sets()` | array of the child-section names directly in this set |
| `handle:Section(name [, create])` | a handle to a child section (find-or-create by default) |
| `handle:Name()` | this set's name |
| `handle:Save([name])` | write to XML (default file: this set's name) |
| `handle:Load([name])` | read from XML (default file: this set's name) |
| `handle:Clear()` | remove all settings and child sections |
| `handle:Sort()` | sort the set by name |

```lua
for _, key in ipairs(cfg:Settings()) do
    echo(key .. " = " .. tostring(cfg:Get(key)))
end
```

- **`name`** (for `IS.Settings`, `Save`, and `Load`) is a logical key, not a path.
  Files live together in the same `ISXLUAData` folder inside your InnerSpace
  **Scripts** directory, as `<name>.xml`, and the name is sanitized so it can never
  point outside that folder.
- `Save` / `Load` return `true` on success. `Load` returns `false` (gracefully) when
  the file does not exist.
- A handle is cleaned up automatically when your script ends. Your saved config on
  disk persists, of course -- that is the point.

**`IS.SaveTable` vs `IS.Settings`:** reach for `IS.SaveTable` when you just want to
stash a Lua table and get it back; reach for `IS.Settings` when you want structured,
sectioned, typed configuration (especially config a user edits).

## The reverse bridge -- calling Lua from LavishScript

Everything so far goes **Lua -> LavishScript**: your script reads
`SomeTLO.SomeMember`, runs commands, evaluates `${...}`. The **reverse bridge**
goes the other way --
it lets a LavishScript program (an `.iss` script, another extension, or a line you
type at the console) **call a Lua function you have written and get its return
value back**. This is the direct call/return path; events ([`04_Timing_And_Events.md`](04_Timing_And_Events.md))
are the other LS -> Lua direction, but they are one-way notifications with no
return value.

There are two halves: your script **registers** a function under a name, and
LavishScript **calls** it by that name.

### `IS.Register(name, fn)` / `IS.Unregister(name)` -- the Lua side

`IS.Register` publishes a Lua function under a name so LavishScript can call it.
`IS.Unregister` removes it. Registration returns the function you passed.

```lua
-- Publish a couple of functions LavishScript can call by name:
IS.Register("Add", function(a, b)
    return tonumber(a) + tonumber(b)
end)

IS.Register("Greet", function(who)
    return "Hello, " .. tostring(who) .. "!"
end)
```

- Registrations are **owned by your script** and are **removed automatically when
  it ends** -- LavishScript can never call into a script that is no longer running.
- Names are **global**. If two scripts register the same name, the **later one
  wins** (the earlier function is quietly replaced). Pick distinctive names (a
  script-name prefix is a good habit, e.g. `"MyBot_GetTarget"`).
- Arguments arrive as **strings** (LavishScript passes everything as text), so
  convert them yourself -- note the `tonumber` in `Add` above.

### `${ISXLUA.Call[name, args...]}` -- call and get a value (the clean form)

From LavishScript, call a registered function through the `ISXLUA` top-level
object. The first argument is the function name; any further arguments are passed
through to your function as strings. The function's return value becomes the value
of the data sequence:

```
echo ${ISXLUA.Call[Add, 2, 3]}
echo ${ISXLUA.Call[Greet, World]}

if ${ISXLUA.Call[IsSafeToPull]}
    echo "clear to pull"
```

This is the **cleanest** form because it returns a real value you can use anywhere
LavishScript expects one. The return value is **typed** by what your Lua function
returns:

| Your Lua function returns | LavishScript sees |
|---|---|
| a **string** | a string |
| an **integer** | an int64 |
| a **number** (non-integer) | a float64 |
| a **boolean** | a bool (usable in `if`) |
| **nil** / nothing | NULL (the member does not exist) |
| a table / other value | its text form (via `tostring`) |

Because a nil / no-return comes back as **NULL**, a call used only for its side
effect simply yields nothing -- `${ISXLUA.Call[DoThing]}` is fine even when
`DoThing` returns nothing.

### `luacall name [args...]` -- call from the console (fire-and-forget)

`luacall` is the console command form. It calls the same registered function and
**prints** the return value. Use it for interactive testing and for triggering
Lua work from a spot where you do not need to capture a value:

```
luacall Add 2 3
    ISXLUA: Add -> 5

luacall Greet World
    ISXLUA: Greet -> Hello, World!
```

Arguments after the name are split on spaces (ordinary console tokenizing). For a
value you want to *use* in a LavishScript expression, prefer `${ISXLUA.Call[...]}`
above; a console command can only print.

### Registered functions run atomically -- no `wait()` inside

When LavishScript calls your function it runs **synchronously and atomically**,
right then, exactly like an event handler: it must run to completion and return a
value in one step. **You cannot call `wait()` / `waitframe()` / `waituntil()`
inside a registered function** (a yield attempt is reported as an error and does
not crash anything). If a call needs to kick off timed work, have it record a
request in a table and let your main script loop -- which *can* wait -- act on it.

An error thrown inside a registered function is printed to the console with a
traceback; `${ISXLUA.Call[...]}` then yields NULL and `luacall` reports the error.

> **A note on commas.** LavishScript splits `${ISXLUA.Call[a, b, c]}` on commas,
> so an argument cannot itself contain a comma. If you need to pass free-form text,
> register a function that takes a single string and parse it yourself.

For a complete, game-specific worked example of the reverse bridge, see
[`08_Examples.md`](08_Examples.md).

## The event and async functions

`IS.AttachEvent`, `IS.DetachEvent`, and `IS.FireEvent` also live on the `IS`
table, as do the asynchronous HTTP functions `IS.HttpGet` / `IS.HttpPost` (in the
"with libisxgames" build). All of these are documented in
[`04_Timing_And_Events.md`](04_Timing_And_Events.md).

Next: [`03_Object_Model.md`](03_Object_Model.md).
