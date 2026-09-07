# Bundled Libraries

ISXLUA ships several popular Lua libraries **built in**. There is nothing to
install and no files to place on disk -- just `require` them. Every bundled module
is require-able only; none is installed as a global, so they never shadow your own
variables or the top-level-object bridge.

Most are MIT-licensed; the rest use equally permissive terms -- `zlib` the zlib license,
`lsqlite3`'s underlying SQLite engine and `base64` the public domain, and `luaunit` the
BSD license. All are bundled in **both** ISXLUA builds -- none requires the
"with libisxgames" build.

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
| `require("isxlua")` | An optional, ISXLUA-specific convenience layer over the `IS` bridge and the runtime -- typed data reads, command/print/log sugar, and event/timer sugar. See [below](#the-isxlua-helper-library). |
| `require("input")` | Typed keyboard / mouse / bind automation -- press keys, hold and release, move and click the mouse, and fire named binds, instead of hand-building command strings. See [below](#the-input-automation-module). |
| `require("middleclass")` | A small, widely-used object-orientation / class system: `class(name[, super])`, `:new(...)`, single inheritance, `:isInstanceOf`, mixins, operator metamethods. |
| `require("pl.tablex")`, `require("pl.stringx")`, ... | [Penlight](https://lunarmodules.github.io/Penlight/) -- a broad standard-library extension. The whole `pl` tree is bundled: table utilities (`pl.tablex`), string utilities (`pl.stringx`), pretty-printing (`pl.pretty`), an OO system (`pl.class`), container classes (`pl.List` / `pl.Map` / `pl.Set` / `pl.OrderedMap`), plus `pl.data`, `pl.Date`, `pl.path`, `pl.dir`, `pl.seq`, `pl.func`, `pl.lexer`, `pl.template`, and more. |
| `require("sha2")` | Cryptographic hashing and HMAC -- SHA-1/224/256/384/512, SHA-3, MD5, HMAC, and the BLAKE family. Each hash returns a lowercase hex string. See [Hashing and encoding](#hashing-and-encoding-with-crypto) below. |
| `require("base64")` | Base64 encode/decode, over the standard alphabet or one you supply. See [Hashing and encoding](#hashing-and-encoding-with-crypto) below. |
| `require("crypto")` | A small convenience facade that pulls `sha2` and `base64` together and adds hex: `crypto.sha256` / `crypto.hmac` / `crypto.base64` / `crypto.hex`. See [Hashing and encoding](#hashing-and-encoding-with-crypto) below. |
| `require("MessagePack")` | Compact binary serialization (the [MessagePack](https://msgpack.org/) format) -- `pack` a Lua value to a byte string and `unpack` it back. See [Binary serialization](#binary-serialization-with-messagepack) below. |
| `require("luaunit")` | A unit-test framework (xUnit-style assertions and test runners) for testing your own Lua modules. See [Unit tests](#unit-tests-with-luaunit) below. |

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
> [`04_Timing_And_Events.md`](04_Timing_And_Events.md#asynchronous-http----ishttpget-ishttppost)). Prefer those for simple
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

### Hashing and encoding with `crypto`

`crypto` is a small facade that bundles the two building blocks -- `sha2` (hashing and
HMAC) and `base64` (Base64) -- and adds hex, so the common needs are one `require`:

```lua
local crypto = require("crypto")

-- Hashes return a lowercase hex digest string:
echo(crypto.sha256("abc"))              -- ba7816bf...f20015ad
echo(crypto.md5("abc"))                 -- 900150983cd24fb0d6963f7d28e17f72

-- HMAC (keyed hash) -- pass one of the crypto.* hash functions as the first argument:
local mac = crypto.hmac(crypto.sha256, "Jefe", "what do ya want for nothing?")
echo(mac)                               -- 5bdcc146bf60754e...964ec3843

-- Base64 encode/decode:
local b64 = crypto.base64.encode("Man is distinguished")
echo(b64)                               -- TWFuIGlzIGRpc3Rpbmd1aXNoZWQ=
echo(crypto.base64.decode(b64))         -- Man is distinguished

-- Hex encode/decode:
local hex = crypto.hex.encode("Man")
echo(hex)                               -- 4d616e
echo(crypto.hex.decode(hex))            -- Man
```

| Call | Returns |
|---|---|
| `crypto.sha256(s)` / `sha1` / `sha512` / `md5` | The digest of `s` as a lowercase hex string. |
| `crypto.hmac(hashfn, key, msg)` | The HMAC of `msg` under `key`, as hex. `hashfn` is a `crypto.*` hash (e.g. `crypto.sha256`). |
| `crypto.base64.encode(s)` / `.decode(s)` | Base64 text / the original bytes. |
| `crypto.hex.encode(s)` / `.decode(s)` | Lowercase hex text / the original bytes. |

The facade covers the everyday cases. For more, `require` the underlying modules
directly: `require("sha2")` also provides SHA-3, SHAKE, and the BLAKE2/BLAKE3 families
(and its own `bin_to_hex` / `bin_to_base64` converters), and `require("base64")` can
build encoders/decoders for a custom alphabet with `makeencoder` / `makedecoder`.

> These are pure-Lua implementations -- convenient and dependency-free, but not
> constant-time. Use them for checksums, content hashes, HMAC signatures, and encoding,
> not as a substitute for a vetted native crypto library in a security-critical setting.
> MD5 and SHA-1 are provided for compatibility with existing data; prefer SHA-256 for
> anything new.

### Binary serialization with `MessagePack`

[MessagePack](https://msgpack.org/) is a compact binary alternative to JSON. `pack`
turns a Lua value into a short byte string; `unpack` turns it back:

```lua
local mp = require("MessagePack")

local data = { name = "example", hp = 100, tags = { "a", "b" }, ok = true }

local packed = mp.pack(data)            -- a compact binary string
echo(#packed .. " bytes")

local restored = mp.unpack(packed)      -- back to a Lua table
echo(restored.name .. " / " .. restored.hp)   -- example / 100
echo(restored.tags[2])                        -- b
```

It round-trips Lua `nil`, booleans, numbers (integer and float), strings, and tables.
It is smaller and faster to parse than JSON, which makes it a good fit for persisting
state (write the bytes to a file) or sending structured data over
[`socket`](#networking-with-socket). When you need a *human-readable* form instead,
use [`cjson`](#json-with-cjson) or [`serpent`](#serializing-tables-with-serpent).

### Unit tests with `luaunit`

`luaunit` is an xUnit-style test framework. Group tests as functions (named with a
`test` prefix) in a table, then run them with a `LuaUnit` runner:

```lua
local lu = require("luaunit")

TestMath = {}
function TestMath:testAdd()
    lu.assertEquals(1 + 1, 2)
end
function TestMath:testChecks()
    lu.assertTrue(2 > 1)
    lu.assertNil(nil)
    lu.assertError(function() error("boom") end)   -- asserts the call raises
end

-- Run just these instances quietly and read the failure count (0 == all passed):
local runner = lu.LuaUnit.new()
runner:setOutputType("NIL")             -- suppress console output; omit for a TAP/text report
runner:runSuiteByInstances({ { "TestMath", TestMath } })
echo("failures: " .. tostring(runner.result.notSuccessCount))
```

Common assertions include `assertEquals`, `assertNotEquals`, `assertTrue`, `assertFalse`,
`assertNil`, `assertNotNil`, `assertStrContains`, and `assertError` (and its
`assertErrorMsgContains` variant). `assertEquals` compares tables deeply, so you can
assert on whole result tables at once.

> Requiring `luaunit` replaces the global `os.exit` with a guarded version (so a stray
> test cannot silently end the run); it behaves normally when no suite is running. The
> runner methods return a failure count rather than exiting, so running tests never
> stops your script. Keep tests in their own `.lua` files you run on demand, not inside
> a production script.

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

## The `input` automation module

`require("input")` is an optional convenience layer for **sending keyboard and mouse
input, and firing binds**. It saves you from hand-building command strings: you call
`input.press("ctrl+1")` instead of `IS.Execute("Press ctrl+1")`, and keys, mouse
buttons, and binds all share one small, typed vocabulary. Like `isxlua` it installs
no globals -- `require` it into a local.

```lua
local input = require("input")

input.press("ctrl+1")            -- tap a key or combo (press + release)
input.press("2", { nomodifiers = true })   -- send the exact key, no modifier remapping

input.keydown("w")               -- press and HOLD a key ...
wait(1.0)
input.keyup("w")                 -- ... then release it

input.move(640, 400)             -- move the cursor to client-pixel (640, 400)
input.click("left", 640, 400)    -- move there, then left-click
input.click("right")             -- right-click at the current cursor

input.execBind("MoveForward")    -- fire a named InnerSpace bind (press + release)

if input.keyPressed("shift") then    -- read current key state
    echo("shift is held")
end
local x, y = input.mousePos()    -- current cursor position, as numbers
```

### Keyboard

| Function | What it does |
|---|---|
| `input.press(combo [, opts])` | Tap a key or combo (press + release). `opts.nomodifiers = true` sends the exact key without modifier remapping (tap only). |
| `input.keydown(combo)` (alias `input.hold`) | Press and **hold** a key/combo until you release it. |
| `input.keyup(combo)` (alias `input.release`) | Release a key/combo held with `keydown`. |

A `combo` is an InnerSpace key name or combination: `"a"`, `"1"`, `"ctrl+tab"`,
`"shift+f1"`, `"\\"`, the mouse-button names `"mouse1"` ... `"mouse5"`, or
`"MouseWheelUp"` / `"MouseWheelDown"`.

### Mouse

| Function | What it does |
|---|---|
| `input.move(x, y)` | Move the cursor to absolute client-pixel `(x, y)`. |
| `input.click([button] [, x, y])` | Optionally move to `(x, y)` first, then click. `button` is `"left"` (default), `"right"`, or `"middle"` (their first letter or `1`/`2`/`3` also work). |
| `input.mousedown([button])` | Press and **hold** a mouse button. |
| `input.mouseup([button])` | Release a mouse button held with `mousedown`. |

### Binds

| Function | What it does |
|---|---|
| `input.execBind(name)` | Fire a named InnerSpace bind -- runs its press action then its release action (a one-shot). |
| `input.bindDown(name)` | Run only the bind's **press** action (hold it). |
| `input.bindUp(name)` | Run only the bind's **release** action. |

A bind is an InnerSpace bind you created with the `Bind` command; the name goes
inside `${Keyboard.Bind[...]}`, so it may not contain `]`.

### Reading input state

| Function | What it returns |
|---|---|
| `input.keyPressed(name)` | `true` while the named key/button is currently held. `name` is a key/button name (`"alt"`, `"shift"`, `"ctrl"`, `"space"`, a letter/number, or `"mouse1"` ... `"mouse5"`). |
| `input.mouseX()` / `input.mouseY()` | The cursor's current client-pixel X / Y, as a number. |
| `input.mousePos()` | Both at once: `local x, y = input.mousePos()`. |

### Notes and limits

- **It targets your session.** Input goes to the game window this InnerSpace session
  is attached to -- not other sessions or other Windows programs. Coordinates are the
  game window's client pixels, with the origin at the top-left.
- **Middle click is emulated** as the `mouse3` button (there is no dedicated
  middle-click), so it behaves as a middle-button tap.
- **Binds take no arguments.** InnerSpace binds are argument-less; use `bindDown` /
  `bindUp` when you need press-and-hold rather than a one-shot `execBind`.
- Everything here is a thin wrapper over commands you can also issue directly with
  [`IS.Execute`](02_The_IS_Bridge.md#isexecutecommand----run-a-command) (`Press`,
  `MouseClick`, `Mouse:...`) and reads you can do with
  [`IS.Parse`](02_The_IS_Bridge.md#isparsedatasequence----evaluate-a--expression) --
  the module just makes them typed and discoverable.

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
