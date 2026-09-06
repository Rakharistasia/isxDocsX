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
| `require("lgui2")` | An ergonomic layer for building and driving **LavishGUI 2** UIs from Lua. It has its own chapter -- see [`05_Building_GUIs.md`](05_Building_GUIs.md). |

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
