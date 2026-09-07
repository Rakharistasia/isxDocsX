# Bundled Libraries

ISXLUA ships several popular Lua libraries **built in**. There is nothing to
install and no files to place on disk -- just `require` them. Every bundled module
is require-able only; none is installed as a global, so they never shadow your own
variables or the top-level-object bridge.

Most are MIT-licensed; the exceptions are `zlib` (the zlib license) and `lsqlite3`'s
underlying SQLite engine (public domain). All are bundled in **both** ISXLUA builds --
none requires the "with libisxgames" build.

## The modules

| `require` | What it is |
|---|---|
| `require("cjson")` | Fast JSON encode/decode. |
| `require("cjson.safe")` | The never-throwing variant of cjson (returns `nil` + error instead of raising). |
| `require("lfs")` | LuaFileSystem -- directory listing, file attributes, `mkdir`, `chdir`. |
| `require("lpeg")` | LPeg -- Parsing Expression Grammars, for robust parsing/lexing. |
| `require("serpent")` | Table serializer and pretty-printer. |
| `require("inspect")` | Human-readable dump of nested tables (great for debugging). |
| `require("json")` | A pure-Lua JSON implementation (a lightweight alternative to cjson). |
| `require("re")` | LPeg's regex-like front-end (uses `lpeg` under the hood). |
| `require("socket")` | LuaSocket -- TCP/UDP networking. Also brings `socket.http`, `socket.url`, `socket.ftp`, `socket.smtp`, `socket.tp`, `socket.headers`, and the `ltn12` / `mime` helpers. |
| `require("zlib")` | lua-zlib -- deflate/inflate (zlib and gzip) compression, plus `adler32` / `crc32` checksums. |
| `require("lsqlite3")` | lsqlite3 -- an embedded SQLite 3 database, backed by on-disk files or a fast `:memory:` database. |
| `require("lgui2")` | An ergonomic layer for building and driving **LavishGUI 2** (JSON, newer) UIs from Lua. It has its own chapter -- see [`05_Building_GUIs.md`](05_Building_GUIs.md). |
| `require("lgui1")` | The sibling layer for **LavishGUI 1** (XML, older) UIs. Same chapter -- see [`05_Building_GUIs.md`](05_Building_GUIs.md). |
| `require("isxlua")` | An optional, ISXLUA-specific convenience layer over the `IS` bridge and the runtime -- typed data reads, command/print/log sugar, and event/timer sugar. See below. |
| `require("middleclass")` | A small, widely-used object-orientation / class system: `class(name[, super])`, `:new(...)`, single inheritance, `:isInstanceOf`, mixins, operator metamethods. |
| `require("pl.tablex")`, `require("pl.stringx")`, ... | [Penlight](https://lunarmodules.github.io/Penlight/) -- a broad standard-library extension. The whole `pl` tree is bundled: table utilities (`pl.tablex`), string utilities (`pl.stringx`), pretty-printing (`pl.pretty`), an OO system (`pl.class`), container classes (`pl.List` / `pl.Map` / `pl.Set` / `pl.OrderedMap`), plus `pl.data`, `pl.Date`, `pl.path`, `pl.dir`, `pl.seq`, `pl.func`, `pl.lexer`, `pl.template`, and more. |

### JSON with `cjson`

```lua
local cjson = require("cjson")

local s = cjson.encode({ name = "example", count = 3, enabled = true })
echo(s)                              -- {"name":"example","count":3,"enabled":true}

local t = cjson.decode(s)
echo(t.name .. " has count " .. t.count)
```

Use `cjson.safe` when you are decoding data you do not control and do not want an
error to stop your script:

```lua
local cjson = require("cjson.safe")
local t, err = cjson.decode(maybeBadInput)
if not t then
    echo("bad JSON: " .. tostring(err))
end
```

`require("json")` is a pure-Lua fallback with a similar `encode` / `decode` API if
you prefer it.

### Listing files with `lfs`

Lua's own `io` / `os` cannot enumerate a directory; `lfs` fills that gap:

```lua
local lfs = require("lfs")
for entry in lfs.dir(".") do
    echo(entry)
end

lfs.mkdir("logs")                    -- create a directory
local attr = lfs.attributes("hello.lua")
echo("size: " .. tostring(attr and attr.size))
```

### Serializing tables with `serpent`

`serpent` turns a table into Lua source text you can save and load back -- ideal
for persisting state between runs:

```lua
local serpent = require("serpent")

local state = { runs = 3, best = 128, names = { "a", "b" } }
local text = serpent.dump(state)     -- compact, loadable
-- ...or serpent.block(state) for a pretty, human-readable form

-- read it back:
local ok, restored = serpent.load(text)
if ok then echo("runs so far: " .. restored.runs) end
```

### Debugging with `inspect`

```lua
local inspect = require("inspect")
echo(inspect(ISXLUA.Version))         -- see exactly what you have
echo(inspect({ 1, 2, nested = { x = true } }))
```

### Parsing with `lpeg` / `re`

For parsing structured text -- chat logs, combat logs, small DSLs -- `lpeg` (and
its friendlier `re` front-end) is far more robust than string patterns:

```lua
local re = require("re")
local number = re.compile("%d+")
echo(tostring(number:match("abc123") == nil))   -- pattern basics
```

See the upstream LPeg / re documentation for the full grammar syntax.

### Networking with `socket`

`socket` is LuaSocket -- raw TCP/UDP plus higher-level helpers. On Windows it needs
nothing extra (it uses the built-in Winsock stack). A minimal TCP client:

```lua
local socket = require("socket")

echo(socket._VERSION)                 -- e.g. "LuaSocket 3.0.0"

local c = assert(socket.tcp())
c:settimeout(5)                       -- never block the session forever
local ok, err = c:connect("example.com", 80)
if ok then
    c:send("GET / HTTP/1.0\r\nHost: example.com\r\n\r\n")
    echo(c:receive("*l"))             -- first response line
end
c:close()
```

The higher-level submodules load on demand:

```lua
local http = require("socket.http")   -- also pulls ltn12 + mime
local body, code = http.request("http://example.com/")
echo("HTTP status: " .. tostring(code))
```

`ltn12` (streaming filters/sinks/sources) and `mime` (Base64 / quoted-printable)
are available as `require("ltn12")` and `require("mime")`.

> ISXLUA also has its own non-blocking `IS.HttpGet` / `IS.HttpPost` (see
> [`04_Timing_And_Events.md`](04_Timing_And_Events.md)). Prefer those for simple
> fetches that must not stall the frame; reach for `socket` when you need raw
> sockets or a protocol LuaSocket already speaks. LuaSocket calls are blocking, so
> always set a timeout.

### Compression with `zlib`

`zlib` is lua-zlib: streaming deflate/inflate plus checksums. A round-trip:

```lua
local zlib = require("zlib")

local original = string.rep("ISXLUA! ", 512)

-- compress: feed data, then finish
local deflate = zlib.deflate()
local compressed = deflate(original, "finish")

-- decompress
local inflate = zlib.inflate()
local restored = inflate(compressed)

echo("match: " .. tostring(restored == original))
echo("crc32: " .. tostring(zlib.crc32()(original)))
```

`zlib.deflate([level])` returns a stream you call with chunks; pass `"finish"` on
the last call. `zlib.inflate()` auto-detects a zlib or gzip header.

### A database with `lsqlite3`

`lsqlite3` binds the SQLite 3 engine, so a script can keep real structured state.
Use a filename for a persistent database or `:memory:` for a scratch one:

```lua
local sqlite3 = require("lsqlite3")

local db = sqlite3.open_memory()      -- or sqlite3.open("Scripts/mydata.db")

db:exec[[
    CREATE TABLE kills (id INTEGER PRIMARY KEY, mob TEXT, ts INTEGER);
    INSERT INTO kills (mob, ts) VALUES ('orc', 100);
    INSERT INTO kills (mob, ts) VALUES ('goblin', 200);
]]

for row in db:nrows("SELECT mob, ts FROM kills ORDER BY ts") do
    echo(row.mob .. " @ " .. row.ts)
end

-- parameter binding avoids SQL-injection and quoting headaches:
local stmt = db:prepare("INSERT INTO kills (mob, ts) VALUES (?, ?)")
stmt:bind_values("dragon", 300)
stmt:step()
stmt:finalize()

db:close()
```

`db:nrows` yields a table keyed by column name; `db:rows` yields positional values;
`db:exec` runs one or more statements with no result rows. Always `finalize` prepared
statements and `close` the database when done.

### Classes with `middleclass`

`middleclass` gives you clean object-orientation -- classes, single inheritance,
and instance checks -- without writing metatable boilerplate yourself. It is pure
Lua, bundled in both builds:

```lua
local class = require("middleclass")

local Animal = class("Animal")
function Animal:initialize(name) self.name = name end
function Animal:speak() return "..." end

local Dog = Animal:subclass("Dog")          -- or class("Dog", Animal)
function Dog:speak() return "Woof! I am " .. self.name end

local d = Dog:new("Rex")
echo(d:speak())                              -- Woof! I am Rex
echo(tostring(d:isInstanceOf(Dog)))          -- true
echo(tostring(d:isInstanceOf(Animal)))       -- true (inherited)
echo(d.class.name)                           -- Dog
```

### Standard-library extensions with Penlight

[Penlight](https://lunarmodules.github.io/Penlight/) is a large collection of
utilities that fill gaps in Lua's standard library. Each submodule is required by
name (`pl.tablex`, `pl.stringx`, `pl.pretty`, ...); it is pure Lua and bundled in
both builds. A few representative pieces:

```lua
local tablex  = require("pl.tablex")
local stringx = require("pl.stringx")
local pretty  = require("pl.pretty")

-- deep copy / deep compare of nested tables
local original = { hp = 100, pos = { x = 1, y = 2 } }
local copy = tablex.deepcopy(original)
echo(tostring(tablex.deepcompare(original, copy)))   -- true

-- string helpers Lua's own library lacks
echo(stringx.strip("   padded   "))                  -- "padded"
local parts = stringx.split("a,b,c", ",")            -- { "a", "b", "c" }
echo(parts[1] .. "-" .. parts[3])                    -- a-c

-- pretty-print a table (and read one back)
echo(pretty.write({ mob = "orc", count = 3 }))
```

Container classes are handy too:

```lua
local List = require("pl.List")
local nums = List{ 3, 1, 2 }
nums:append(4)
echo(nums:sort():join(", "))                         -- 1, 2, 3, 4
```

> `require("pl")` on its own loads the whole library lazily *into globals* -- handy
> for exploration, but prefer requiring the specific `pl.*` submodules you use so
> your script's globals stay clean.

## The `isxlua` helper library

`require("isxlua")` is a small, **optional** convenience layer written specifically
for ISXLUA. Unlike the libraries above (which are general-purpose Lua projects), it
just wraps the ISXLUA primitives you use most -- `IS.Parse` / `IS.Execute`, `echo`,
the events layer, and the pulse timers -- to cut everyday boilerplate. Everything it
does you can also do by hand; it is sugar, not a new capability. Keep it in a local
(it installs no globals):

```lua
local isxlua = require("isxlua")
```

It is deliberately **curated**. It does **not** re-implement string/table/class
utilities (use `pl.*` and `middleclass`), it does not rename `waituntil`, and it does
not wrap JSON or persistence (those primitives are already one-liners).

### Typed data reads

These wrap `IS.Parse` and convert the result to a Lua type for you. **You pass the
inner LavishScript expression** -- no `${ }` -- and the helper wraps it. (A string
that already starts with `${` is used as-is, so an expression you built elsewhere
still works.) Absent, empty, or `"NULL"` results resolve to the default you give,
never a surprise error.

```lua
local isxlua = require("isxlua")

local hp    = isxlua.num("Me.Health", 0)        -- a number, or 0 if not readable
local name  = isxlua.str("Me.Name", "unknown")  -- a string, or "unknown" if absent
local ready = isxlua.bool("ISXLUA.IsReady")     -- a real boolean
local raw   = isxlua.parse("ISXLUA.Version")    -- the raw string, or nil

if isxlua.exists("Me.Pet") then                 -- ${Me.Pet(exists)} == TRUE ?
    echo("you have a pet")
end
```

| Call | Returns |
|---|---|
| `isxlua.parse(expr)` | The raw string result, or `nil` (parse failure / result over the 8 KB cap). |
| `isxlua.num(expr [, default])` | A number, or `default` (default `nil`) when the result is not numeric. |
| `isxlua.bool(expr)` | `true` for `"TRUE"` / `"true"` / `"1"`; otherwise `false`. |
| `isxlua.str(expr [, default])` | The string, or `default` (default `nil`) when it is `nil`, `""`, or `"NULL"`. |
| `isxlua.exists(expr)` | `true` when `${<expr>(exists)}` is `TRUE`. Pass the inner expression. |

> The [object model](03_Object_Model.md) already returns native numbers/strings for a
> direct read like `Me.Health`, so reach for these mainly when you are building an
> expression as a string, using bracketed index expressions, or want the uniform
> "give me a value or this default" behavior.

### Command and output sugar

```lua
local isxlua = require("isxlua")

isxlua.exec("echo hello from %s", "isxlua")     -- string.format + IS.Execute
isxlua.printf("loaded %d items in %.1fs", 12, 0.3)   -- string.format + echo

isxlua.setLogPrefix("[MyBot]")                  -- optional leading tag
isxlua.log("started")                           -- [MyBot] [LOG] started
isxlua.warn("low health:", hp)                  -- [MyBot] [WARN] low health: 42
isxlua.error("target lost")                     -- [MyBot] [ERROR] target lost
```

`exec` returns `IS.Execute`'s integer result. With no extra arguments it runs the
command verbatim, so a literal `%` in a plain command is never treated as a format
spec. The `log` / `warn` / `error` trio all just **echo** a tagged line -- `error`
is a log *level*, not Lua's `error()`, so it never raises.

### Event and timer sugar

```lua
local isxlua = require("isxlua")

-- run a handler for exactly the FIRST firing of an event, then auto-detach:
isxlua.once("SomeEvent", function(a, b)
    echo("first fire only: " .. tostring(a))
end)

-- friendly names for the pulse timers (see 04_Timing_And_Events.md):
local h = isxlua.every(1.0, function() echo("tick") end)   -- setInterval
isxlua.after(5.0, function() isxlua.cancel(h) end)         -- setTimeout + clearTimer
```

`isxlua.after` / `every` / `cancel` are thin aliases for `setTimeout` /
`setInterval` / `clearTimer`, sharing one vocabulary. As with any handler or timer
callback, an `once` handler and the timer callbacks run atomically and must not
`wait()`.

## Loading your own modules

`package.path` and `package.cpath` are set to your InnerSpace **Scripts**
directory (and a `lua_modules` subdirectory inside it), so you can split a script
into your own reusable modules and `require` them:

```lua
-- Scripts\mymodule.lua  or  Scripts\lua_modules\mymodule.lua
local mymodule = require("mymodule")
```

The search covers `mymodule.lua`, `mymodule/init.lua`, and the same under
`lua_modules`. The bundled modules above resolve first (they are already loaded
internally), so they never depend on the path.

Next: [`07_Migration_Gotchas.md`](07_Migration_Gotchas.md).
