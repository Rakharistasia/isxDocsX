# Examples

Complete, runnable example scripts. Every ISXLUA feature and every bundled
`require("...")` module is demonstrated in at least one example below.

These examples ship with ISXLUA in `Install\Scripts\Examples\`. Copy any of them
into your InnerSpace **Scripts** directory (or run them from where they are) and
run with `lua <name>` -- e.g. `lua 01_bridge_and_events`. The `.lua` extension is
optional; `endlua <name>` stops a running one and `luas` lists what is running.
(`run <name>.lua` / `endscript <name>` work too; the native `scripts` listing does
NOT show `.lua` scripts, so use `luas`.) For a one-off line without a file, use
`lua -c "print(Me.Name)"`.

> These examples are kept in sync as ISXLUA features are added. If something here
> disagrees with the reference chapters, the reference chapters are current.

The example files are a graded set: the early ones are game-agnostic and run with
no game loaded; [`11_game_character_eq2.lua`](#11_game_character_eq2lua) shows the object bridge in a real game
(EverQuest II); the [`12`](#12_autoexec_autoloadlua--sample-autoloadlua)/[`13`](#13_autoexec_autoload_eq2lua--sample-autoload_eq2lua) files are sample autoexec content you activate by
renaming. Unlike the topic chapters, examples may be game-specific.

## Feature -> example coverage matrix

| Feature / module | Example file(s) |
|---|---|
| [`IS.Execute`](02_The_IS_Bridge.md#isexecutecommand----run-a-command) | [`01_bridge_and_events`](#01_bridge_and_eventslua) (used throughout) |
| [`IS.Parse`](02_The_IS_Bridge.md#isparsedatasequence----evaluate-a--expression) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua), [`08_isxlua_helpers`](#08_isxlua_helperslua) |
| [`print` / `echo`](02_The_IS_Bridge.md#print-and-echo----console-output) | [`01_bridge_and_events`](#01_bridge_and_eventslua) (used throughout) |
| [`ISXLUA.Version` / `.IsReady` / `.IsLoading` / `.InQuietMode`](01_Getting_Started.md#the-isxlua-object) | [`01_bridge_and_events`](#01_bridge_and_eventslua) |
| [`IS.WarnUnknownGlobals`](02_The_IS_Bridge.md#iswarnunknownglobalsenabled----control-the-unknown-global-warning) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [Script args](01_Getting_Started.md#script-arguments) (`args` table + `...`) | [`01_bridge_and_events`](#01_bridge_and_eventslua) |
| [Bare-global TLOs](03_Object_Model.md#top-level-objects-are-bare-lua-globals) | [`01_bridge_and_events`](#01_bridge_and_eventslua) (ISXLUA), [`11_game_character_eq2`](#11_game_character_eq2lua) (Me/Actor/EQ2/Zone/Target) |
| [`.Member`](03_Object_Model.md#members-objmember-and-objmemberargs) (native scalar leaves) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [`.Member(args)`](03_Object_Model.md#members-objmember-and-objmemberargs) | [`11_game_character_eq2`](#11_game_character_eq2lua) (`Me.Group(1)`), [`01_bridge_and_events`](#01_bridge_and_eventslua) (`ISXLUA.Call(name, n)`) |
| [`:Method(args)`](03_Object_Model.md#methods-objmethodargs) | [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [`obj[i]`](03_Object_Model.md#numeric-indexing-obji) numeric index | [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [Typed getters](03_Object_Model.md#typed-getters-for-coercing-an-object-to-a-scalar) (`:Int`/`:Number`/`:Bool`/`:Str`/`:LSType`) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [`:IsNull` / `:Exists` / global `Exists(x)`](03_Object_Model.md#testing-whether-something-exists) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [Object coercion](03_Object_Model.md#object-wrappers) (`tostring`/concat) + scalar operators | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [Typed-numeric arg coercion](03_Object_Model.md#passing-arguments-numbers-and-booleans-convert-automatically) (int/float/bool) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [`wait(seconds)` / `wait(seconds, condfn)`](04_Timing_And_Events.md#waitseconds-and-waitframe) | [`02_timing_and_async`](#02_timing_and_asynclua) |
| [`waitframe()`](04_Timing_And_Events.md#waitseconds-and-waitframe) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`02_timing_and_async`](#02_timing_and_asynclua) |
| [`waituntil(condfn [, timeout])`](04_Timing_And_Events.md#waituntilcondfn-timeoutseconds----wait-for-a-condition) | [`02_timing_and_async`](#02_timing_and_asynclua), [`05_http_and_networking`](#05_http_and_networkinglua), [`11_game_character_eq2`](#11_game_character_eq2lua) |
| [`waitforevent(name [, timeout])`](04_Timing_And_Events.md#waitforeventname-timeoutseconds----wait-for-the-next-event) | [`02_timing_and_async`](#02_timing_and_asynclua) |
| [`setTimeout` / `setInterval` / `clearTimer`](04_Timing_And_Events.md#timers----settimeout-setinterval-cleartimer) | [`02_timing_and_async`](#02_timing_and_asynclua) |
| [`await(starter [, timeout])`](04_Timing_And_Events.md#awaitstarter-timeoutseconds----turn-a-callback-into-a-linear-call) (callback -> linear) | [`02_timing_and_async`](#02_timing_and_asynclua), [`05_http_and_networking`](#05_http_and_networkinglua) |
| [`IS.AttachEvent` / `IS.AttachEventTyped`](04_Timing_And_Events.md#isattacheventname-fn) | [`01_bridge_and_events`](#01_bridge_and_eventslua) |
| [`IS.DetachEvent` / `IS.FireEvent`](04_Timing_And_Events.md#isdetacheventname-fn) | [`01_bridge_and_events`](#01_bridge_and_eventslua), [`02_timing_and_async`](#02_timing_and_asynclua) |
| [`IS.EventSource()`](04_Timing_And_Events.md#iseventsource) | [`01_bridge_and_events`](#01_bridge_and_eventslua) |
| [`IS.SaveTable` / `IS.LoadTable`](02_The_IS_Bridge.md#issavetablename-tbl--isloadtablename----persistent-storage) | [`03_persistence_and_settings`](#03_persistence_and_settingslua), [`12_autoexec_autoload`](#12_autoexec_autoloadlua--sample-autoloadlua) |
| [`IS.Settings`](02_The_IS_Bridge.md#issettingsname----a-hierarchical-persistent-config-store) (Set/Get/GetString/Exists/Delete/Section/Settings/Sets/Name/Save/Load/Clear/Sort) | [`03_persistence_and_settings`](#03_persistence_and_settingslua) |
| [`IS.Register` / `IS.Unregister`](02_The_IS_Bridge.md#isregistername-fn--isunregistername----the-lua-side) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua) |
| [`${ISXLUA.Call[...]}` / `luacall`](02_The_IS_Bridge.md#isxluacallname-args----call-and-get-a-value-the-clean-form) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua) |
| [`IS.GetVar`](02_The_IS_Bridge.md#isgetvar----read-a-lavishscript-variable-typed) / [`IS.SetVar`](02_The_IS_Bridge.md#issetvar----write-a-lavishscript-variable) (LavishScript variables, typed) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua) |
| [`IS.CallAtom`](02_The_IS_Bridge.md#iscallatom----call-a-lavishscript-atom-get-its-return) (call a LavishScript atom) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua) |
| [`IS.Share` / `IS.Shared`](04_Timing_And_Events.md#the-shared-value-store----isshare-isshared) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua), [`14_pause_resume_reload`](#14_pause_resume_reloadlua) |
| [`IS.Subscribe` / `IS.Publish` / `IS.Unsubscribe`](04_Timing_And_Events.md#the-message-bus----ispublish-issubscribe-isunsubscribe) | [`04_reverse_bridge_and_ipc`](#04_reverse_bridge_and_ipclua) |
| [Autoexec `autoload.lua`](01_Getting_Started.md#autoexec-autoloadlua) | [`12_autoexec_autoload`](#12_autoexec_autoloadlua--sample-autoloadlua) |
| [Per-game autoexec `autoload_<game>.lua`](01_Getting_Started.md#per-game-autoexec-autoload_gamelua) | [`13_autoexec_autoload_eq2`](#13_autoexec_autoload_eq2lua--sample-autoload_eq2lua) |
| [Pause / resume / reload](01_Getting_Started.md#pausing-resuming-and-reloading-scripts) (`IS.PauseScript`/`ResumeScript`/`ReloadScript` + `lua -pause`/`-resume`/`-reload`) | [`14_pause_resume_reload`](#14_pause_resume_reloadlua) |
| [Commands `lua` / `endlua` / `luas`](01_Getting_Started.md#running-and-stopping-scripts) | [`14_pause_resume_reload`](#14_pause_resume_reloadlua) (+ every file's how-to-run) |
| [`IS.HttpGet` / `IS.HttpPost`](04_Timing_And_Events.md#asynchronous-http----ishttpget-ishttppost) (table body: JSON + form) + `IS.HttpGetSync` / `IS.HttpPostSync` | [`05_http_and_networking`](#05_http_and_networkinglua) |
| [`require("socket")`](06_Bundled_Libraries.md#networking-with-socket) + `socket.http` / `socket.url` / `ltn12` / `mime` | [`05_http_and_networking`](#05_http_and_networkinglua) |
| [`require("cjson")`](06_Bundled_Libraries.md#json-with-cjson) (+ `cjson.safe`) | [`06_data_libraries`](#06_data_librarieslua), [`05_http_and_networking`](#05_http_and_networkinglua) |
| [`require("json")`](06_Bundled_Libraries.md#json-with-cjson) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("serpent")`](06_Bundled_Libraries.md#serializing-tables-with-serpent) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("inspect")`](06_Bundled_Libraries.md#debugging-with-inspect) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("lpeg")` / `require("re")`](06_Bundled_Libraries.md#parsing-with-lpeg--re) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("lfs")`](06_Bundled_Libraries.md#listing-files-with-lfs) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("zlib")`](06_Bundled_Libraries.md#compression-with-zlib) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("lsqlite3")`](06_Bundled_Libraries.md#a-database-with-lsqlite3) | [`06_data_libraries`](#06_data_librarieslua) |
| [`require("middleclass")`](06_Bundled_Libraries.md#classes-with-middleclass) | [`07_oop_and_utilities`](#07_oop_and_utilitieslua) |
| [`require("pl.*")`](06_Bundled_Libraries.md#standard-library-extensions-with-penlight) (Penlight: class/List/Map/Set/tablex/stringx/pretty/seq + aggregate) | [`07_oop_and_utilities`](#07_oop_and_utilitieslua) |
| [`require("isxlua")`](06_Bundled_Libraries.md#the-isxlua-helper-library) (helper library) | [`08_isxlua_helpers`](#08_isxlua_helperslua) |
| [`require("lgui2")`](05_Building_GUIs.md#building-guis-lavishgui-2) | [`09_gui_lgui2`](#09_gui_lgui2lua) |
| [`require("lgui1")`](05_Building_GUIs.md#the-older-system-lavishgui-1-lgui1) | [`10_gui_lgui1`](#10_gui_lgui1lua) |

---

## 01_bridge_and_events.lua

The core bridge and the events layer -- `ISXLUA` object members, `print`/`echo`,
`IS.Execute`/`IS.Parse`, `IS.WarnUnknownGlobals`, script args, typed-numeric arg
coercion, the object typed-getters (via a NULL object), and
`IS.AttachEvent`/`AttachEventTyped`/`FireEvent`/`DetachEvent`/`EventSource`. Fully
game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 01_bridge_and_events.lua
--------------------------------------------------------------------------------
-- Demonstrates the CORE bridge and the EVENTS layer -- all game-agnostic, so this
-- runs with no game extension loaded.
--
--   * the ISXLUA object   (.Version / .IsReady / .IsLoading / .InQuietMode)
--   * print / echo         (console output; Lua's raw print is invisible in-game)
--   * IS.Execute           (run a LavishScript command, get the int result)
--   * IS.Parse             (evaluate a ${...} data sequence, get the string)
--   * IS.WarnUnknownGlobals(false)   (silence the unknown-global warning)
--   * script args          (the `args` table and the vararg `...`)
--   * typed-numeric arg coercion   (a Lua number -> the string LavishScript receives)
--   * the object typed-getters      (:IsNull / :Exists / :Int / :Str / :LSType) via a NULL object
--   * IS.AttachEvent / IS.AttachEventTyped / IS.FireEvent / IS.DetachEvent
--   * IS.EventSource()     (the event's originating object, inside a handler)
--
-- HOW TO RUN (from the InnerSpace console):
--     lua 01_bridge_and_events
--   or with a couple of arguments to see the `args` table:
--     lua 01_bridge_and_events hello 42
--------------------------------------------------------------------------------

-- IS.WarnUnknownGlobals(false) silences the one-time "unknown global" warning that
-- fires the first time you touch a name that is neither a Lua value nor a TLO. The
-- default is ON (the warning helps catch typos); turn it off while iterating.
IS.WarnUnknownGlobals(false)

--------------------------------------------------------------------------------
-- 1. The ISXLUA object -- always present, no game required.
--------------------------------------------------------------------------------
-- Top-level objects are bare Lua globals. `ISXLUA` is the extension's own TLO.
-- Scalar members come back as REAL Lua values (string / boolean / number), so you
-- can use them directly with Lua's own operators and string methods.
echo("ISXLUA version : " .. ISXLUA.Version)          -- a native Lua string
echo("  uppercased   : " .. ISXLUA.Version:upper())  -- ...so :upper() just works
echo("IsReady        : " .. tostring(ISXLUA.IsReady))     -- a native boolean
echo("IsLoading      : " .. tostring(ISXLUA.IsLoading))   -- a native boolean
echo("InQuietMode    : " .. tostring(ISXLUA.InQuietMode)) -- a native boolean

-- print(...) works too; it is an alias for echo here (both go to the console).
print("print() and echo() both write to the InnerSpace console.")

--------------------------------------------------------------------------------
-- 2. Script arguments -- `args[1..]` and the vararg `...`.
--------------------------------------------------------------------------------
-- Anything after the script name on the command line arrives two ways: as a
-- 1-indexed global `args` table, and as the chunk's varargs.
local varargs = { ... }
if #varargs > 0 then
    echo("Received " .. #varargs .. " argument(s):")
    for i, v in ipairs(args) do
        echo(string.format("   args[%d] = %s", i, tostring(v)))
    end
else
    echo("No arguments passed. Try:  lua 01_bridge_and_events hello 42")
end

--------------------------------------------------------------------------------
-- 3. IS.Execute / IS.Parse -- the escape hatches into LavishScript.
--------------------------------------------------------------------------------
-- IS.Execute runs a console/LavishScript command and returns its int result.
local rc = IS.Execute("echo (this line came from IS.Execute)")
echo("IS.Execute returned: " .. tostring(rc))

-- IS.Parse evaluates a full ${...} data sequence and returns the string result
-- (or nil). ${Math.Calc[...]} is always available, so it makes a good agnostic
-- demonstration vehicle.
local sum = IS.Parse("${Math.Calc[2+3]}")
echo("IS.Parse('${Math.Calc[2+3]}') = " .. tostring(sum))

--------------------------------------------------------------------------------
-- 4. Typed-numeric argument coercion.
--------------------------------------------------------------------------------
-- When you pass a Lua number/boolean as an ARGUMENT to a member/method, ISXLUA
-- stringifies it for LavishScript: an integer gets NO decimal point ("42"), a
-- float keeps full precision with a locale-independent '.' ("42.5"), and a bool
-- becomes TRUE/FALSE. To SEE the exact string LavishScript received -- with no game
-- loaded -- register a tiny Lua "echo" function and call it back through the
-- bare-global ISXLUA object's .Call member (the reverse bridge). The argument travels
-- the SAME coercion path a real game member/method arg uses.
--   (Aside: ${Math...} is reachable via IS.Parse("${Math.Calc[...]}") but Math is a
--   core LavishScript pseudo-TLO, NOT a bare global -- so `Math.Calc(42)` would index
--   nil. Bare globals are the game/extension TLOs, e.g. Me / Actor / EQ2.)
IS.Register("example_echoarg", function(s) return "<" .. tostring(s) .. ">" end)
echo("ISXLUA.Call(echo, 42)   = " .. tostring(ISXLUA.Call("example_echoarg", 42)) ..
     "   (integer arg -> \"42\", no decimal point)")
echo("ISXLUA.Call(echo, 42.5) = " .. tostring(ISXLUA.Call("example_echoarg", 42.5)) ..
     "   (float arg keeps its decimal, locale-independent)")
echo("ISXLUA.Call(echo, true) = " .. tostring(ISXLUA.Call("example_echoarg", true)) ..
     "   (boolean arg -> TRUE/FALSE)")
IS.Unregister("example_echoarg")

--------------------------------------------------------------------------------
-- 5. Object wrappers and the typed getters -- shown on a NULL object.
--------------------------------------------------------------------------------
-- Not every result is a scalar. An unresolved object comes back as a "NULL object
-- wrapper" -- never nil, never an error. IS.EventSource() called OUTSIDE a handler
-- returns exactly such a NULL object, which lets us exercise the object helpers
-- without needing a game loaded:
--   :IsNull()  -> true when the object did not resolve
--   :Exists()  -> the inverse (and the global Exists(x) does the same job)
--   :Int() / :Number() / :Bool() / :Str()  -> coerce an OBJECT to a scalar
--   :LSType()  -> the LavishScript type name
local nullobj = IS.EventSource()
echo("A NULL object wrapper:")
echo("   :IsNull() = " .. tostring(nullobj:IsNull()))   -- true
echo("   :Exists() = " .. tostring(nullobj:Exists()))   -- false
echo("   Exists(nullobj) = " .. tostring(Exists(nullobj)))  -- false (global form)
echo("   :Int()  = " .. tostring(nullobj:Int()))        -- 0
echo("   :Str()  = '" .. tostring(nullobj:Str()) .. "'")-- ""
echo("   tostring() = " .. tostring(nullobj))           -- "NULL"

-- The global Exists() is the correct presence test: nil -> false, a NULL wrapper
-- -> false, but ANY real native value (including 0, "", false) -> true. This is
-- why you must never write a bare `if Me.Pet then` (a wrapper is always truthy).
echo("Exists(nil)   = " .. tostring(Exists(nil)))       -- false
echo("Exists(0)     = " .. tostring(Exists(0)))         -- true (0 is a real value)
echo("Exists('')    = " .. tostring(Exists("")))        -- true

--------------------------------------------------------------------------------
-- 6. Events -- attach a Lua function to a LavishScript event.
--------------------------------------------------------------------------------
-- A Lua function IS the event target. We define our own private event name and
-- self-fire it so the demo is fully self-contained. Handlers run atomically: they
-- must NOT call wait()/waitframe().

-- 6a. A plain handler receives every arg as a STRING.
local plainSeen
local function onDemo(a, b)
    plainSeen = { a, b }
    -- IS.EventSource() inside a handler gives the event's originating object. A
    -- self-fired event carries no source, so it is a NULL object here.
    local src = IS.EventSource()
    echo(string.format("[onDemo] a=%q b=%q  source present? %s",
        tostring(a), tostring(b), tostring(Exists(src))))
end
IS.AttachEvent("example_demo_event", onDemo)
IS.FireEvent("example_demo_event", "hello", "world")
waitframe()   -- dispatch is synchronous, but yield a frame to be tidy

-- 6b. A TYPED handler receives args coerced to their natural Lua type: a numeric
-- string becomes a number, "TRUE"/"FALSE" become booleans, everything else stays
-- a string. Same event system, opt-in via IS.AttachEventTyped.
local function onTyped(i, f, flag)
    echo(string.format("[onTyped] i=%s(%s)  f=%s(%s)  flag=%s(%s)",
        tostring(i), math.type(i) or type(i),
        tostring(f), math.type(f) or type(f),
        tostring(flag), type(flag)))
end
IS.AttachEventTyped("example_typed_event", onTyped)
IS.FireEvent("example_typed_event", "7", "1.5", "TRUE")   -- fired as strings
waitframe()

-- 6c. Detaching stops delivery. Handlers also auto-detach when the script ends,
-- so this is only needed while the script keeps running.
IS.DetachEvent("example_demo_event", onDemo)
IS.DetachEvent("example_typed_event", onTyped)
plainSeen = nil
IS.FireEvent("example_demo_event", "should", "not fire")
waitframe()
echo("After DetachEvent, the handler fired again? " .. tostring(plainSeen ~= nil))

echo("Done. (See 04_reverse_bridge_and_ipc.lua for calling Lua FROM LavishScript.)")
```

---

## 02_timing_and_async.lua

Every timing and async primitive -- `wait`, `wait(sec, condfn)`, `waitframe`,
`waituntil`, `waitforevent`, the `setTimeout`/`setInterval`/`clearTimer` timers,
and `await` (turning a callback into a linear call). Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 02_timing_and_async.lua
--------------------------------------------------------------------------------
-- Demonstrates every timing and async primitive -- all game-agnostic.
--
--   * wait(seconds)                yield for N seconds (SECONDS, not tenths)
--   * wait(seconds, condfn)        yield up to N seconds, resume early on a condition
--   * waitframe()                  yield until the next frame
--   * waituntil(condfn [, timeout])poll a condition every frame until true/timeout
--   * waitforevent(name [, timeout])one-shot: block until an event fires
--   * setTimeout(seconds, fn)      run fn once, N seconds from now
--   * setInterval(seconds, fn)     run fn repeatedly every N seconds
--   * clearTimer(handle)           cancel a timer of either kind
--   * await(starter [, timeout])   turn a callback into a linear call
--
-- Timers and waitforevent are driven by the per-frame scheduler -- no threads.
-- Timer callbacks run atomically (like event handlers): they must NOT wait().
--
-- HOW TO RUN:
--     lua 02_timing_and_async
--------------------------------------------------------------------------------

echo("== Timing and async primitives ==")

--------------------------------------------------------------------------------
-- 1. wait(seconds) -- the basic cooperative yield. LavishScript's `wait 20` (two
--    seconds, in tenths) becomes simply wait(2) here.
--------------------------------------------------------------------------------
echo("wait(1)...")
wait(1)
echo("  ...1 second elapsed.")

-- waitframe() yields exactly one frame -- handy for letting a just-fired event or
-- a just-issued command settle before you read the result.
waitframe()

--------------------------------------------------------------------------------
-- 2. waituntil(condfn [, timeout]) -- the clean replacement for a hand-rolled
--    poll loop. It re-checks condfn() every frame until it returns truthy, or the
--    timeout elapses. Returns true if met, false if it timed out.
--------------------------------------------------------------------------------
do
    local checks = 0
    local met = waituntil(function()
        checks = checks + 1
        return checks >= 3        -- becomes true on the third poll
    end, 5)
    echo(string.format("waituntil met=%s after %d checks", tostring(met), checks))

    -- The timeout path: a condition that never becomes true returns false.
    local timedOut = waituntil(function() return false end, 0.3)
    echo("waituntil that never met -> " .. tostring(timedOut))   -- false
end

--------------------------------------------------------------------------------
-- 3. wait(seconds, condfn) -- a bounded wait that resumes EARLY the moment the
--    condition is true. Same semantics as waituntil, time-first.
--------------------------------------------------------------------------------
do
    -- Arm a one-shot timer to flip a flag after 0.3s, then wait up to 3s for it.
    local ready = false
    setTimeout(0.3, function() ready = true end)
    local early = wait(3, function() return ready end)
    echo("wait(3, cond) finished early? " .. tostring(early))    -- true (~0.3s in)
end

--------------------------------------------------------------------------------
-- 4. setTimeout / setInterval / clearTimer.
--------------------------------------------------------------------------------
-- setTimeout: fire exactly once.
do
    local fired = 0
    setTimeout(0.25, function() fired = fired + 1 end)
    wait(0.6)
    echo("setTimeout fired " .. fired .. " time(s) (expected 1).")
end

-- setInterval: fire repeatedly until cleared. clearTimer stops it and returns
-- true if it cancelled a live timer.
do
    local ticks = 0
    local handle = setInterval(0.2, function() ticks = ticks + 1 end)
    wait(0.9)                       -- ~4 ticks
    local atStop = ticks
    local cleared = clearTimer(handle)
    wait(0.5)                       -- would be ~2 more ticks if still running
    echo(string.format("setInterval ticked %d times, clearTimer=%s, ticks after clear=%d",
        atStop, tostring(cleared), ticks))
end

--------------------------------------------------------------------------------
-- 5. waitforevent(name [, timeout]) -- block until a named event fires, then
--    resume with (true, args...); or (false) if the timeout elapses first. It is
--    a ONE-SHOT wait; use IS.AttachEvent for a persistent handler.
--------------------------------------------------------------------------------
do
    -- Have a timer fire our event shortly after we start waiting for it.
    setTimeout(0.4, function() IS.FireEvent("example_ready", "payload-A", "payload-B") end)

    local fired, a, b = waitforevent("example_ready", 5)
    echo(string.format("waitforevent -> fired=%s  a=%s  b=%s",
        tostring(fired), tostring(a), tostring(b)))

    -- The timeout path: an event that never fires, with a short deadline.
    local none = waitforevent("example_never_fires", 1)
    echo("waitforevent that never fired -> " .. tostring(none))   -- false
end

--------------------------------------------------------------------------------
-- 6. await(starter [, timeout]) -- turn any callback-style API into a linear
--    call. await runs starter(resolve) once, then SUSPENDS the script until
--    something calls resolve(...); the values passed to resolve become await's
--    return values. Any later callback (a timer, an event, an HTTP completion)
--    can resolve it. With a timeout, an unresolved wait returns the sentinel
--    (nil, "timeout"). Like wait()/waitforevent() it yields, so it cannot be
--    used inside an event handler or a timer callback.
--------------------------------------------------------------------------------
do
    -- Turn a setTimeout callback into a straight-line value: instead of nesting
    -- the rest of the work inside the callback, we await its resolve.
    local value = await(function(resolve)
        setTimeout(0.3, function() resolve(42) end)
    end)
    echo("await(resolve 42) -> " .. tostring(value))             -- 42

    -- resolve can deliver several values; await returns them all, in order.
    local a, b, c = await(function(resolve)
        setTimeout(0.2, function() resolve("x", 2, true) end)
    end)
    echo(string.format("await multi -> a=%s b=%s c=%s",
        tostring(a), tostring(b), tostring(c)))

    -- The timeout path: nobody resolves, so after 0.5s await returns nil,"timeout".
    local v, err = await(function(resolve) --[[ never resolves ]] end, 0.5)
    echo(string.format("await timeout -> v=%s err=%s", tostring(v), tostring(err)))
end

echo("Done. (Timers/waitforevent/await yield the main coroutine, so they cannot be")
echo("used inside an event handler or a timer callback -- those run atomically.)")
```

---

## 03_persistence_and_settings.lua

The two persistence facilities -- the flat `IS.SaveTable`/`IS.LoadTable` and the
hierarchical `IS.Settings` config store (every method: Set/Get/GetString/Exists/
Delete/Section/Settings/Sets/Name/Save/Load/Clear/Sort). Game-agnostic; run it
twice to watch the persisted run counter grow.

```lua
--------------------------------------------------------------------------------
-- 03_persistence_and_settings.lua
--------------------------------------------------------------------------------
-- Demonstrates the two persistence facilities -- both game-agnostic.
--
--   * IS.SaveTable(name, tbl) / IS.LoadTable(name)
--         Flat "save a whole table, load it back" storage. Great for script state.
--         Files live in <Scripts>\ISXLUAData\<name>.lua (serialized via serpent).
--
--   * IS.Settings(name) -> a handle to a hierarchical, persistent config store
--         (backed by InnerSpace's LavishSettings). Richer than SaveTable: named
--         sections, typed round-trip, XML import/export. Methods:
--            :Set / :Get / :GetString / :Exists / :Delete
--            :Section / :Settings / :Sets / :Name
--            :Save / :Load / :Clear / :Sort
--
-- HOW TO RUN (run it twice -- the run counter persists between runs):
--     lua 03_persistence_and_settings
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- 1. IS.SaveTable / IS.LoadTable -- a persistent run counter.
--------------------------------------------------------------------------------
-- LoadTable returns the stored table, or nil if there is no file yet -- so the
-- idiomatic "load with default" is `IS.LoadTable(name) or { ... }`.
echo("== IS.SaveTable / IS.LoadTable ==")

local state = IS.LoadTable("example_state") or { runs = 0, lastMessage = "(first run)" }
echo("Previous run count : " .. state.runs)
echo("Previous message   : " .. state.lastMessage)

state.runs = state.runs + 1
state.lastMessage = "run #" .. state.runs .. " at " .. os.date("%H:%M:%S")

local okSave, err = IS.SaveTable("example_state", state)
if okSave then
    echo("Saved. This is run #" .. state.runs .. ". Run the script again to see it grow.")
else
    echo("Save failed: " .. tostring(err))
end

--------------------------------------------------------------------------------
-- 2. IS.Settings -- a hierarchical config store with typed values.
--------------------------------------------------------------------------------
echo("")
echo("== IS.Settings ==")

-- The first IS.Settings("name") of a session auto-loads ISXLUAData\name.xml if it
-- exists, so a config persists with zero ceremony. We Clear() here only to make
-- the demo deterministic on repeated runs.
local cfg = IS.Settings("example_config")
cfg:Clear()

-- :Set stores strings, numbers, and booleans. Intermediate sections in a
-- "section/sub/leaf" path are created automatically.
cfg:Set("enabled", true)                 -- boolean
cfg:Set("maxTargets", 5)                 -- number
cfg:Set("label", "MyBot")                -- string
cfg:Set("ui/window/x", 100)              -- nested path -> creates ui, then window
cfg:Set("ui/window/y", 240)

-- :Get infers the type back ("TRUE"/"FALSE" -> boolean, a whole-number string ->
-- number, else string). Pass a default for the absent case.
echo("enabled        = " .. tostring(cfg:Get("enabled")))         -- boolean true
echo("maxTargets     = " .. tostring(cfg:Get("maxTargets")))      -- number 5
echo("label          = " .. tostring(cfg:Get("label")))           -- string "MyBot"
echo("ui/window/x    = " .. tostring(cfg:Get("ui/window/x")))     -- number 100
echo("missing (dflt) = " .. tostring(cfg:Get("missing", 42)))     -- default 42

-- :GetString bypasses inference and returns the exact stored text -- use it for a
-- numeric-looking string you want to keep as a string (a zip code, an ID).
echo("maxTargets as string = '" .. cfg:GetString("maxTargets") .. "'")   -- "5"

-- :Exists / :Delete.
echo("Exists('enabled')  = " .. tostring(cfg:Exists("enabled")))
cfg:Delete("enabled")
echo("after Delete       = " .. tostring(cfg:Exists("enabled")))

-- :Section returns a handle to a child section (find-or-create by default; pass
-- false to only find). This is the same navigation as a "/"-path, as an object.
local win = cfg:Section("ui"):Section("window")
echo("via Section, ui/window/x = " .. tostring(win:Get("x")))
echo("Section('nope', false) is nil? " .. tostring(cfg:Section("nope", false) == nil))

-- :Settings() lists the leaf setting names in a set; :Sets() lists child sections.
local leaves = cfg:Settings()
echo("leaf settings at root  : " .. table.concat(leaves, ", "))
local sections = cfg:Sets()
echo("child sections at root : " .. table.concat(sections, ", "))
echo("this set's :Name()     = " .. cfg:Name())

-- :Sort() orders entries; :Save() writes the XML to disk; :Load() reads it back.
cfg:Sort()
if cfg:Save() then
    echo("Saved config to ISXLUAData\\example_config.xml")

    -- Prove the disk round-trip: change a value in memory, then reload from disk.
    cfg:Set("maxTargets", 999)
    cfg:Load()
    echo("after Save/change/Load, maxTargets = " .. tostring(cfg:Get("maxTargets")))
end

echo("Done. SaveTable is best for whole-table state; Settings for named,")
echo("user-editable config with sections.")
```

---

## 04_reverse_bridge_and_ipc.lua

Interop beyond the object bridge: the reverse bridge (`IS.Register`/`Unregister`
+ `${ISXLUA.Call}` / `luacall`), the typed forward interop (`IS.SetVar`/`IS.GetVar`
read/write LavishScript variables, `IS.CallAtom` calls a LavishScript atom), the
cross-script shared store (`IS.Share`/`IS.Shared`), and the message bus
(`IS.Subscribe`/`IS.Publish`/`IS.Unsubscribe`). Game-agnostic; stays alive ~30s so
the console or an `.iss` bot can call in.

```lua
--------------------------------------------------------------------------------
-- 04_reverse_bridge_and_ipc.lua
--------------------------------------------------------------------------------
-- Demonstrates the two interop directions BEYOND the object bridge -- all
-- game-agnostic:
--
--   REVERSE BRIDGE (call Lua FROM LavishScript):
--     * IS.Register(name, fn) / IS.Unregister(name)
--     * ${ISXLUA.Call[name, args...]}   (LavishScript data member -- returns the value)
--     * luacall <name> [args...]         (console command -- prints the return)
--
--   FORWARD INTEROP (reach INTO LavishScript from Lua -- typed):
--     * IS.SetVar(name, value) / IS.GetVar(name [, default])   (LavishScript variables)
--     * IS.CallAtom(name [, args...])                          (call a LavishScript atom)
--
--   CROSS-SCRIPT SHARED STORE (deep-copied between the isolated Lua states):
--     * IS.Share(key, value) / IS.Shared(key)
--
--   CROSS-SCRIPT MESSAGE BUS (publish/subscribe):
--     * IS.Subscribe(channel, fn) / IS.Publish(channel, ...) / IS.Unsubscribe(channel [, handle])
--
-- HOW TO RUN:
--     lua 04_reverse_bridge_and_ipc
--   While it is running (it stays alive ~30s), try from the console:
--     luacall example_greet World
--     ${ISXLUA.Call[example_add, 2, 3]}
--   Stop early with:  endlua 04_reverse_bridge_and_ipc
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- 1. Reverse bridge -- publish Lua functions callable from LavishScript by name.
--------------------------------------------------------------------------------
-- Registered functions run SYNCHRONOUSLY and ATOMICALLY when LavishScript calls
-- them (like an event handler) -- they must NOT wait(). Args arrive as strings;
-- the return value is typed back to LavishScript (string/int/float/bool/NULL).
echo("== Reverse bridge ==")

IS.Register("example_add", function(a, b)
    return tonumber(a) + tonumber(b)     -- returns a number -> LS sees a float/int
end)

IS.Register("example_greet", function(who)
    return "Hello, " .. tostring(who) .. "!"   -- returns a string
end)

-- We can call our own registered functions from Lua via the same LS-side member.
-- IS.Parse returns the value as the string LavishScript would receive.
echo("Call example_add(2,3) via ${ISXLUA.Call} = " ..
    tostring(IS.Parse("${ISXLUA.Call[example_add, 2, 3]}")))       -- "5"
echo("Call example_greet('Norrath')          = " ..
    tostring(IS.Parse("${ISXLUA.Call[example_greet, Norrath]}")))  -- "Hello, Norrath!"

echo("From the console you can also run:  luacall example_greet World")
echo("or use it in an .iss bot:           ${ISXLUA.Call[example_add, 10, 20]}")

--------------------------------------------------------------------------------
-- 2. Forward interop -- read/write LavishScript VARIABLES and call a LavishScript
--    ATOM as typed Lua values (the counterpart to the reverse bridge above).
--------------------------------------------------------------------------------
echo("")
echo("== LavishScript interop (variables & atoms) ==")

-- IS.SetVar creates the variable in GLOBAL scope if needed; the Lua type picks the
-- LavishScript type (integer -> int64, float -> float64, bool -> bool, string ->
-- string). A LavishScript (.iss) script could read these back with ${...}.
IS.SetVar("example_ammo", 250)          -- a global int64
IS.SetVar("example_name", "Norrath")    -- a global string
IS.SetVar("example_ready", true)        -- a global bool

-- IS.GetVar reads them back as NATIVE Lua values (typed), not strings.
local ammo  = IS.GetVar("example_ammo")     -- 250       (a Lua number)
local name  = IS.GetVar("example_name")     -- "Norrath" (a Lua string)
local ready = IS.GetVar("example_ready")    -- true      (a Lua boolean)
echo(("Read back: ammo=%d (%s)  name=%q  ready=%s")
    :format(ammo, math.type(ammo), name, tostring(ready)))

-- A default is returned when the variable does not resolve.
echo("Missing var, with default: " .. tostring(IS.GetVar("example_missing", "n/a")))

-- The SAME variable is visible to LavishScript as ${example_ammo} -- that is the interop.
echo("LavishScript's view via ${example_ammo}: " .. tostring(IS.Parse("${example_ammo}")))

-- IS.CallAtom invokes a GLOBAL LavishScript atom and returns its value, typed. We
-- define one on the fly here with the AddAtom console command; normally a running
-- .iss script would have declared it with `atom(global) example_bonus(...)`.
IS.Execute("DeleteAtom example_bonus")   -- clear any prior definition (harmless if none)
IS.Execute([[AddAtom -global "atom example_bonus(int base, int mult)\n{\nreturn ${Math.Calc[${base}*${mult}]}\n}"]])

-- CallAtom raises if the atom is not a resolvable GLOBAL atom, so guard with pcall.
local okCall, bonus = pcall(IS.CallAtom, "example_bonus", ammo, 2)
if okCall and type(bonus) == "number" then
    echo("IS.CallAtom('example_bonus', " .. ammo .. ", 2) = " .. bonus .. " (a Lua number)")
else
    echo("[note] the example atom was not callable this run (" .. tostring(bonus) .. ");")
    echo("       declare a global atom in an .iss script and call it with IS.CallAtom.")
end
IS.Execute("DeleteAtom example_bonus")   -- cleanup

--------------------------------------------------------------------------------
-- 3. Shared data store -- values are DEEP-COPIED between scripts (never shared by
--    reference). Copyable: nil/boolean/number/string and tables of those.
--------------------------------------------------------------------------------
echo("")
echo("== IS.Share / IS.Shared ==")

IS.Share("example_status", { state = "running", ticks = 0, tags = { "demo", "ipc" } })
local snapshot = IS.Shared("example_status")
echo("Shared status: state=" .. snapshot.state ..
     " tags=" .. table.concat(snapshot.tags, ","))

-- The copy is independent: mutating what you read back does NOT change the store.
snapshot.state = "mutated locally"
echo("After local mutation, the STORE still says: " .. IS.Shared("example_status").state)

-- Trying to share an uncopyable value (a function) raises a clear error.
local ok = pcall(function() IS.Share("example_bad", function() end) end)
echo("Sharing a function raised an error (as expected)? " .. tostring(not ok))

--------------------------------------------------------------------------------
-- 4. Message bus -- publish/subscribe across scripts (args deep-copied too).
--------------------------------------------------------------------------------
echo("")
echo("== IS.Subscribe / IS.Publish / IS.Unsubscribe ==")

local received
local handle = IS.Subscribe("example_channel", function(kind, detail)
    received = { kind = kind, detail = detail }
end)

-- Publish delivers synchronously to EVERY running script subscribed to the
-- channel (including ourselves) and returns how many callbacks ran.
local delivered = IS.Publish("example_channel", "alert", "low power")
echo("Publish delivered to " .. delivered .. " subscriber(s).")
echo("Subscriber received: kind=" .. tostring(received.kind) ..
     " detail=" .. tostring(received.detail))

-- Unsubscribe stops delivery to this callback.
IS.Unsubscribe("example_channel", handle)
received = nil
IS.Publish("example_channel", "alert", "should not arrive")
echo("After Unsubscribe, a further publish reached us? " .. tostring(received ~= nil))

--------------------------------------------------------------------------------
-- 5. Stay alive briefly so the console / an .iss bot can drive the reverse-bridge
--    calls above. Registrations auto-clear when the script ends.
--------------------------------------------------------------------------------
echo("")
echo("Registrations are live for ~30s. Try 'luacall example_greet World' now.")
echo("(Or stop immediately with: endlua 04_reverse_bridge_and_ipc)")

-- Publish a heartbeat to our own channel each second, keeping shared state fresh.
local subCount = 0
local hb = IS.Subscribe("example_channel", function() subCount = subCount + 1 end)
local elapsed = 0
while elapsed < 30 do
    wait(1)
    elapsed = elapsed + 1
    IS.Share("example_status", { state = "running", ticks = elapsed })
    IS.Publish("example_channel", "heartbeat", elapsed)
end

IS.Unsubscribe("example_channel", hb)
IS.Unregister("example_add")
IS.Unregister("example_greet")
echo("Done. (Registrations and subscriptions also auto-clear on script end.)")
```

---

## 05_http_and_networking.lua

Networking two ways: LuaSocket (`socket`, `socket.http`, `socket.url`, `ltn12`,
`mime`) which is bundled in both builds and works everywhere, and the async
`IS.HttpGet`/`IS.HttpPost` (feature-detected -- "with libisxgames" build only,
including a table body auto-encoded to JSON or url-encoded form). Also shows
`await` turning the async callback into a linear call and the inline
`IS.HttpGetSync`/`IS.HttpPostSync` wrappers. Bounded and best-effort, so it never
hangs offline.

```lua
--------------------------------------------------------------------------------
-- 05_http_and_networking.lua
--------------------------------------------------------------------------------
-- Demonstrates the networking options. Two independent paths:
--
--   LuaSocket -- bundled in BOTH builds, works everywhere:
--     * require("socket")        core (socket.gettime, TCP objects)
--     * require("socket.http")   blocking HTTP client
--     * require("socket.url")    URL parse/build helpers
--     * require("ltn12")         the streaming sink used to capture a response body
--     * require("mime")          base64 / quoted-printable helpers
--
--   IS.HttpGet / IS.HttpPost -- ASYNCHRONOUS HTTP, "with libisxgames" build ONLY:
--     * feature-detected (absent in the plain build), non-blocking, callback-based
--     * IS.HttpPost accepts a Lua TABLE body (auto-encoded to JSON, or url-encoded
--       when the content type names form encoding)
--     * await(...) turns the callback into a single LINEAR call
--     * IS.HttpGetSync / IS.HttpPostSync -- INLINE wrappers over await (same
--       "with libisxgames" build only); feature-detected; return (ok, status, body)
--
-- Network calls are bounded and best-effort here: if the machine is offline the
-- demo just reports that and moves on -- it never hangs.
--
-- HOW TO RUN:
--     lua 05_http_and_networking
--------------------------------------------------------------------------------

local cjson = require("cjson")

--------------------------------------------------------------------------------
-- 1. LuaSocket core + helpers that need no network.
--------------------------------------------------------------------------------
echo("== LuaSocket (bundled, both builds) ==")

local socket = require("socket")
echo("LuaSocket version: " .. tostring(socket._VERSION))

-- socket.gettime() is a high-resolution wall clock in seconds (a float).
local t0 = socket.gettime()

-- Create and immediately close a TCP master to prove the C core is wired up
-- without touching the network.
local tcp = socket.tcp()
if tcp then tcp:close() end
echo("Created + closed a TCP object OK.")

-- socket.url parses and rebuilds URLs.
local url = require("socket.url")
local parts = url.parse("https://example.com:8443/path?q=1")
echo(string.format("Parsed URL -> scheme=%s host=%s port=%s path=%s",
    parts.scheme, parts.host, tostring(parts.port), parts.path))

-- mime provides base64 (handy for Basic auth headers, small blobs, etc.).
local mime = require("mime")
local encoded = (mime.b64("ISXLUA"))
echo("mime.b64('ISXLUA') = " .. tostring(encoded))

echo(string.format("(socket.gettime delta so far: %.4fs)", socket.gettime() - t0))

--------------------------------------------------------------------------------
-- 2. Blocking HTTP via socket.http -- always available. Bounded by a short
--    TIMEOUT so it never hangs when offline. ltn12 captures the response body.
--------------------------------------------------------------------------------
echo("")
echo("== socket.http (blocking, bounded) ==")

local http  = require("socket.http")
local ltn12 = require("ltn12")
http.TIMEOUT = 5   -- seconds; keeps us from hanging if the host is unreachable

do
    local chunks = {}
    -- The table form returns (1, statuscode, headers, statusline) on success, so
    -- capture the SECOND value for the HTTP status (the first is a success sentinel).
    local ok, sentinel, code = pcall(function()
        local ok1, status = http.request{
            url = "http://example.com/",
            sink = ltn12.sink.table(chunks),   -- stream the body into `chunks`
        }
        return ok1, status
    end)
    if ok and sentinel == 1 and code == 200 then
        local body = table.concat(chunks)
        echo(string.format("socket.http GET example.com -> %d, %d bytes", code, #body))
    else
        echo("socket.http request could not complete (offline?): status=" .. tostring(code))
    end
end

--------------------------------------------------------------------------------
-- 3. Asynchronous HTTP -- IS.HttpGet / IS.HttpPost. Feature-detected: present
--    only in the "with libisxgames" build. The callback is cb(ok, status, body).
--------------------------------------------------------------------------------
echo("")
echo("== IS.HttpGet / IS.HttpPost (async, libisxgames build only) ==")

if not IS.HttpGet then
    echo("This is the plain build -- IS.HttpGet / IS.HttpPost are not available.")
    echo("(Use the socket.http path above, which works in every build.)")
else
    -- 3a. A simple async GET. We fire it, then wait (bounded) for the callback.
    do
        local done, ok, status = false, nil, nil
        IS.HttpGet("https://www.google.com/generate_204", function(o, s, body)
            done, ok, status = true, o, s
        end)
        waituntil(function() return done end, 15)
        echo(string.format("IS.HttpGet -> done=%s ok=%s status=%s",
            tostring(done), tostring(ok), tostring(status)))
    end

    -- 3b. IS.HttpPost with a TABLE body. By default a table is auto-encoded to
    -- JSON with Content-Type: application/json. httpbin echoes it back under
    -- "json", so we can confirm the round-trip when it is reachable.
    do
        local done, ok, body = false, nil, nil
        IS.HttpPost("https://httpbin.org/post", { name = "isxlua", value = 42 },
            function(o, s, b) done, ok, body = true, o, b end)
        if waituntil(function() return done end, 15) and ok and type(body) == "string" then
            -- The echoed body is itself JSON; decode it with cjson and read back.
            local okDecode, parsed = pcall(cjson.decode, body)
            if okDecode and parsed and parsed.json then
                echo("IS.HttpPost JSON body echoed back: name=" ..
                    tostring(parsed.json.name) .. " value=" .. tostring(parsed.json.value))
            else
                echo("IS.HttpPost completed (ok=" .. tostring(ok) .. ").")
            end
        else
            echo("IS.HttpPost could not reach httpbin (offline?) -- skipped.")
        end
    end

    -- 3c. A table body PLUS an explicit form content type -> url-encoded instead.
    do
        local done, ok = false, nil
        IS.HttpPost("https://httpbin.org/post", { user = "bob", score = 10 },
            "application/x-www-form-urlencoded",
            function(o, s, b) done, ok = true, o end)
        waituntil(function() return done end, 15)
        echo("IS.HttpPost (form-encoded) completed? " .. tostring(done) ..
             " ok=" .. tostring(ok))
    end

    -- 3d. await turns the async callback into a LINEAR call: no nested callback,
    -- no done/ok/status flags + waituntil -- just a straight assignment. await
    -- suspends the script until resolve(...) runs inside the completion callback.
    do
        local ok, status, body = await(function(resolve)
            IS.HttpGet("https://www.google.com/generate_204", function(o, s, b)
                resolve(o, s, b)
            end)
        end)
        echo(string.format("await IS.HttpGet -> ok=%s status=%s bytes=%s",
            tostring(ok), tostring(status), tostring(body and #body or 0)))
    end

    -- 3e. IS.HttpGetSync / IS.HttpPostSync -- that same await pattern pre-packaged
    -- as inline calls. They ship with the libisxgames build too, but feature-detect
    -- them separately to be safe. Each returns (ok, status, body), like the callback.
    if IS.HttpGetSync then
        local ok, status, body = IS.HttpGetSync("https://www.google.com/generate_204")
        echo(string.format("IS.HttpGetSync -> ok=%s status=%s bytes=%s",
            tostring(ok), tostring(status), tostring(body and #body or 0)))

        -- HttpPostSync keeps the async POST's table-body handling (JSON by default).
        local pok, pstatus = IS.HttpPostSync("https://httpbin.org/post",
            { name = "isxlua", value = 42 })
        echo(string.format("IS.HttpPostSync -> ok=%s status=%s",
            tostring(pok), tostring(pstatus)))
    else
        echo("IS.HttpGetSync / IS.HttpPostSync not present in this build.")
    end
end

echo("Done.")
```

---

## 06_data_libraries.lua

A tour of the bundled data / serialization / parsing / storage libraries:
`cjson` (+ `cjson.safe`), the pure-Lua `json`, `serpent`, `inspect`, `lpeg` + `re`,
`lfs`, `zlib`, and `lsqlite3`. Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 06_data_libraries.lua
--------------------------------------------------------------------------------
-- A tour of the bundled DATA / serialization / parsing / storage libraries. Every
-- one is compiled or embedded directly into ISXLUA.dll -- nothing to install, just
-- require() it. All game-agnostic.
--
--   * cjson  / cjson.safe   fast JSON encode/decode (safe = never throws)
--   * json                  pure-Lua JSON (a portable fallback/alternative)
--   * serpent               table serializer + pretty-printer (loadable Lua)
--   * inspect               human-readable nested-table dump for debugging
--   * lpeg                  PEG parsing/lexing engine
--   * re                    LPeg's regex-like frontend
--   * lfs                   LuaFileSystem: list dirs, stat files, mkdir
--   * zlib                  deflate/inflate compression + crc32/adler32
--   * lsqlite3              embedded SQLite 3 database (:memory: or on disk)
--
-- HOW TO RUN:
--     lua 06_data_libraries
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- 1. JSON -- cjson (fast, C) and the pure-Lua json fallback.
--------------------------------------------------------------------------------
echo("== JSON ==")

local cjson = require("cjson")
local payload = { name = "Fippy", level = 95, alive = true, skills = { "taunt", "kick" } }
local encoded = cjson.encode(payload)
echo("cjson.encode -> " .. encoded)
local decoded = cjson.decode(encoded)
echo("cjson.decode -> name=" .. decoded.name .. " skills[2]=" .. decoded.skills[2])

-- cjson.safe returns nil + error instead of throwing on bad input.
local bad, err = require("cjson.safe").decode("{ not valid json ")
echo("cjson.safe on bad input -> " .. tostring(bad) .. " / " .. tostring(err))

-- The pure-Lua json module is a drop-in alternative (useful for portability).
local json = require("json")
echo("json (pure-Lua) round-trip -> " .. json.decode(json.encode({ x = 1 })).x)

--------------------------------------------------------------------------------
-- 2. serpent -- serialize a table to LOADABLE Lua source (and read it back).
--------------------------------------------------------------------------------
echo("")
echo("== serpent ==")

local serpent = require("serpent")
local dumped = serpent.block({ hp = 100, pos = { x = 1.5, y = 2.5 } }, { comment = false })
echo("serpent.block ->")
echo(dumped)
local okLoad, restored = serpent.load(serpent.dump({ a = 1, b = { 2, 3 } }))
echo("serpent.load round-trip -> a=" .. tostring(restored.a) .. " b[2]=" .. tostring(restored.b[2]))

--------------------------------------------------------------------------------
-- 3. inspect -- a readable dump for debugging arbitrary tables.
--------------------------------------------------------------------------------
echo("")
echo("== inspect ==")

local inspect = require("inspect")
echo(inspect({ 1, 2, three = { nested = true }, list = { "a", "b" } }))

--------------------------------------------------------------------------------
-- 4. lpeg + re -- pattern matching / lightweight parsing.
--------------------------------------------------------------------------------
echo("")
echo("== lpeg / re ==")

local lpeg = require("lpeg")
-- A tiny grammar: match one or more digits and capture them as a number.
local number = lpeg.C(lpeg.R("09") ^ 1) / tonumber
echo("lpeg match of '  4321' digits -> " ..
    tostring(number:match("4321")))

-- re is a regex-like frontend over lpeg. Split "key=value" pairs.
local re = require("re")
local pattern = re.compile([[ pair <- {%w+} '=' {%w+} ]])
local k, v = pattern:match("level=95")
echo("re parsed 'level=95' -> key=" .. tostring(k) .. " value=" .. tostring(v))

--------------------------------------------------------------------------------
-- 5. lfs -- LuaFileSystem. List the current directory and stat a file.
--------------------------------------------------------------------------------
echo("")
echo("== lfs ==")

local lfs = require("lfs")
echo("current dir: " .. lfs.currentdir())
local count = 0
for entry in lfs.dir(lfs.currentdir()) do
    if entry ~= "." and entry ~= ".." then
        count = count + 1
    end
end
echo("entries in the current dir: " .. count)

--------------------------------------------------------------------------------
-- 6. zlib -- compress and decompress; plus a checksum.
--------------------------------------------------------------------------------
echo("")
echo("== zlib ==")

local zlib = require("zlib")
local original = string.rep("ISXLUA compresses well! ", 100)
local compressed = zlib.deflate()(original, "finish")   -- one-shot compress
local restoredZ  = zlib.inflate()(compressed)           -- inflate back
echo(string.format("deflate: %d bytes -> %d bytes (round-trip ok: %s)",
    #original, #compressed, tostring(restoredZ == original)))

--------------------------------------------------------------------------------
-- 7. lsqlite3 -- an embedded SQLite database, here in-memory.
--------------------------------------------------------------------------------
echo("")
echo("== lsqlite3 ==")

local sqlite3 = require("lsqlite3")
local db = assert(sqlite3.open_memory())
db:exec([[ CREATE TABLE mobs(id INTEGER PRIMARY KEY, name TEXT, level INTEGER); ]])

-- Parameterized insert with a prepared statement.
local stmt = db:prepare("INSERT INTO mobs(name, level) VALUES(?, ?)")
for _, m in ipairs({ { "a rat", 1 }, { "a decrepit servant", 42 }, { "a dragon", 90 } }) do
    stmt:bind_values(m[1], m[2])
    stmt:step()
    stmt:reset()
end
stmt:finalize()

echo("mobs at level >= 40:")
for row in db:nrows("SELECT name, level FROM mobs WHERE level >= 40 ORDER BY level") do
    echo(string.format("   %-20s (level %d)", row.name, row.level))
end
db:close()
echo("(SQLite version " .. sqlite3.version() .. ")")

echo("")
echo("Done. See 07_oop_and_utilities.lua for middleclass + Penlight.")
```

---

## 07_oop_and_utilities.lua

The bundled object-orientation and utility libraries: `middleclass`, and Penlight
(`pl.class`, `pl.List`/`Map`/`Set`, `pl.tablex`, `pl.stringx`, `pl.pretty`,
`pl.seq`, and the lazy aggregate `require("pl")`). Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 07_oop_and_utilities.lua
--------------------------------------------------------------------------------
-- The bundled OBJECT-ORIENTATION and general-purpose UTILITY libraries. All
-- game-agnostic, all embedded in ISXLUA.dll (just require() them).
--
--   * middleclass   a small, popular OO/class system (single inheritance, mixins)
--   * pl.class      Penlight's own class system
--   * pl.List       list container (map/filter/append/...)
--   * pl.Map        key/value container
--   * pl.Set        set container (union/intersection/difference)
--   * pl.tablex     table utilities (deepcopy, deepcompare, map, keys, ...)
--   * pl.stringx    string utilities (split, strip, startswith, ...)
--   * pl.pretty     pretty-print / read Lua data
--   * pl.seq        lazy sequence operations (range/map/filter/sum)
--   * require("pl") the lazy aggregate that pulls in submodules on demand
--
-- HOW TO RUN:
--     lua 07_oop_and_utilities
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- 1. middleclass -- classic OO with inheritance.
--------------------------------------------------------------------------------
echo("== middleclass ==")

local class = require("middleclass")

local Animal = class("Animal")
function Animal:initialize(name) self.name = name end       -- constructor
function Animal:speak() return "..." end
function Animal:describe() return self.name .. " says " .. self:speak() end

local Dog = class("Dog", Animal)                            -- subclass
function Dog:speak() return "Woof" end

local rex = Dog:new("Rex")
echo(rex:describe())                                        -- Rex says Woof
echo("rex isInstanceOf Dog?    " .. tostring(rex:isInstanceOf(Dog)))
echo("rex isInstanceOf Animal? " .. tostring(rex:isInstanceOf(Animal)))  -- also true

--------------------------------------------------------------------------------
-- 2. Penlight's class -- pl.class, with a superclass constructor call.
--------------------------------------------------------------------------------
echo("")
echo("== pl.class ==")

local plclass = require("pl.class")

local Shape = plclass()
function Shape:_init(name) self.name = name end             -- _init is the ctor
function Shape:area() return 0 end

local Circle = plclass(Shape)
function Circle:_init(r)
    self:super("circle")                                    -- call Shape:_init
    self.r = r
end
function Circle:area() return math.pi * self.r * self.r end

local c = Circle(2)
echo(string.format("%s area = %.3f", c.name, c:area()))
echo("c is_a Shape? " .. tostring(c:is_a(Shape)))

--------------------------------------------------------------------------------
-- 3. Penlight containers -- List, Map, Set.
--------------------------------------------------------------------------------
echo("")
echo("== pl.List / pl.Map / pl.Set ==")

local List = require("pl.List")
local nums = List{ 1, 2, 3, 4 }
nums:append(5)
local doubled = nums:map(function(x) return x * 2 end)
echo("List doubled = " .. tostring(doubled))               -- {2,4,6,8,10}
echo("List sum     = " .. tostring(nums:reduce(function(a, b) return a + b end)))  -- 15

local Map = require("pl.Map")
local m = Map{ a = 1, b = 2 }
m:set("c", 3)
echo("Map keys (sorted) = " .. tostring(m:keys():sort()))

local Set = require("pl.Set")
local s1 = Set{ 1, 2, 3 }
local s2 = Set{ 2, 3, 4 }
echo("Set union        = " .. tostring(s1 + s2))            -- {1,2,3,4}
echo("Set intersection = " .. tostring(s1 * s2))            -- {2,3}
echo("Set difference   = " .. tostring(s1 - s2))            -- {1}

--------------------------------------------------------------------------------
-- 4. pl.tablex -- table utilities.
--------------------------------------------------------------------------------
echo("")
echo("== pl.tablex ==")

local tablex = require("pl.tablex")
local orig = { hp = 100, pos = { x = 1, y = 2 } }
local copy = tablex.deepcopy(orig)
echo("deepcopy is deep-equal? " .. tostring(tablex.deepcompare(orig, copy)))
echo("deepcopy is a distinct table? " .. tostring(copy ~= orig))
local squares = tablex.map(function(v) return v * v end, { 1, 2, 3, 4 })
echo("tablex.map squares = " .. table.concat(squares, ","))

--------------------------------------------------------------------------------
-- 5. pl.stringx -- string utilities.
--------------------------------------------------------------------------------
echo("")
echo("== pl.stringx ==")

local stringx = require("pl.stringx")
local parts = stringx.split("taunt,kick,parry", ",")
echo("split       -> " .. table.concat(parts, " | "))
echo("strip('  x  ') -> '" .. stringx.strip("  x  ") .. "'")
echo("startswith('ISXLUA','ISX')? " .. tostring(stringx.startswith("ISXLUA", "ISX")))

--------------------------------------------------------------------------------
-- 6. pl.pretty -- pretty-print / read back Lua data.
--------------------------------------------------------------------------------
echo("")
echo("== pl.pretty ==")

local pretty = require("pl.pretty")
local text = pretty.write({ role = "healer", spells = { "heal", "cure" } })
echo("pretty.write ->")
echo(text)
local back = pretty.read(text)
echo("pretty.read round-trip -> role=" .. back.role .. " spells[1]=" .. back.spells[1])

--------------------------------------------------------------------------------
-- 7. pl.seq -- lazy sequences, and the lazy aggregate require("pl").
--------------------------------------------------------------------------------
echo("")
echo("== pl.seq / require('pl') ==")

local seq = require("pl.seq")
echo("seq.sum(range 1..10) = " .. tostring(seq.sum(seq.range(1, 10))))   -- 55

-- require("pl") is the aggregate: submodules load lazily as you reference them.
local pl = require("pl")
echo("via aggregate, pl.tablex.size{a,b,c} = " .. tostring(pl.tablex.size({ "a", "b", "c" })))

echo("")
echo("Done.")
```

---

## 08_isxlua_helpers.lua

The optional `isxlua` helper library -- typed `IS.Parse` reads
(`parse`/`num`/`bool`/`str`/`exists`), command/output sugar (`exec`/`printf` and
the `log`/`warn`/`error` prefixed logger), and event/timer sugar
(`once`/`after`/`every`/`cancel`). Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 08_isxlua_helpers.lua
--------------------------------------------------------------------------------
-- The optional `isxlua` helper library -- a curated convenience layer over the
-- bridge and the runtime. It installs NO globals; you require it into a local and
-- use it to cut the most common boilerplate. All game-agnostic.
--
--   Typed reads over IS.Parse (you pass the INNER expression; it wraps ${...}):
--     * isxlua.parse(expr)              raw string result (nil on failure/>8KB)
--     * isxlua.num(expr [, default])    a number, or the default
--     * isxlua.bool(expr)               a boolean
--     * isxlua.str(expr [, default])    a string, or the default when absent
--     * isxlua.exists(expr)             true when ${<expr>(exists)} is TRUE
--
--   Command / output sugar:
--     * isxlua.exec(fmt, ...)           string.format + IS.Execute -> int
--     * isxlua.printf(fmt, ...)         string.format + echo
--     * isxlua.log / .warn / .error     a prefixed logger (all echo)
--     * isxlua.setLogPrefix(tag)        set the leading tag
--
--   Event / timer sugar:
--     * isxlua.once(event, fn)          attach a handler that auto-detaches after 1 fire
--     * isxlua.after(sec, fn)  = setTimeout
--     * isxlua.every(sec, fn)  = setInterval
--     * isxlua.cancel(handle)  = clearTimer
--
-- HOW TO RUN:
--     lua 08_isxlua_helpers
--------------------------------------------------------------------------------

local isxlua = require("isxlua")

--------------------------------------------------------------------------------
-- 1. Typed reads. You pass the LavishScript expression WITHOUT the ${...}; the
--    helper wraps it and coerces the result, returning your default when absent.
--------------------------------------------------------------------------------
echo("== Typed reads (over IS.Parse) ==")

-- ${Math.Calc[...]} is always available, so it makes a good agnostic demo.
echo("isxlua.num('Math.Calc[2+3]')      = " .. tostring(isxlua.num("Math.Calc[2+3]")))
echo("isxlua.num('nope', -1)            = " .. tostring(isxlua.num("nope", -1)))   -- default
echo("isxlua.str('ISXLUA.Version')      = " .. tostring(isxlua.str("ISXLUA.Version")))
echo("isxlua.str('nope', '(absent)')    = " .. tostring(isxlua.str("nope", "(absent)")))
echo("isxlua.bool('ISXLUA.IsReady')     = " .. tostring(isxlua.bool("ISXLUA.IsReady")))
echo("isxlua.exists('ISXLUA')           = " .. tostring(isxlua.exists("ISXLUA")))
echo("isxlua.parse('${ISXLUA.Version}') = " .. tostring(isxlua.parse("${ISXLUA.Version}")))

--------------------------------------------------------------------------------
-- 2. Command + output sugar.
--------------------------------------------------------------------------------
echo("")
echo("== Command / output sugar ==")

local rc = isxlua.exec("echo isxlua.exec ran with arg %s", "here")
echo("isxlua.exec returned: " .. tostring(rc))
isxlua.printf("isxlua.printf formats like string.format: %d + %d = %d", 2, 3, 5)

-- A small prefixed logger. All three levels echo (error is a LOG LEVEL here -- it
-- never raises a Lua error).
isxlua.setLogPrefix("[demo]")
isxlua.log("this is an informational line")
isxlua.warn("this is a warning line")
isxlua.error("this is an error-LEVEL line (it does NOT raise)")

--------------------------------------------------------------------------------
-- 3. Event + timer sugar.
--------------------------------------------------------------------------------
echo("")
echo("== Event / timer sugar ==")

-- isxlua.once: fire the event twice, but the handler runs only the first time.
local fires = 0
isxlua.once("example_once_event", function() fires = fires + 1 end)
IS.FireEvent("example_once_event")
waitframe()
IS.FireEvent("example_once_event")
waitframe()
echo("isxlua.once handler fired " .. fires .. " time(s) (expected 1).")

-- after / every / cancel are friendly aliases for setTimeout/setInterval/clearTimer.
local afterFired = false
isxlua.after(0.3, function() afterFired = true end)
wait(0.6)
echo("isxlua.after fired? " .. tostring(afterFired))

local everyN = 0
local handle = isxlua.every(0.2, function() everyN = everyN + 1 end)
wait(0.7)                       -- ~3 ticks
isxlua.cancel(handle)
echo("isxlua.every ticked " .. everyN .. " time(s), then cancelled.")

echo("")
echo("Done. isxlua is optional -- everything it does is also doable by hand with")
echo("IS.Parse / IS.Execute / IS.AttachEvent / setTimeout.")
```

---

## 09_gui_lgui2.lua

Build and drive a LavishGUI 2 (JSON) window from Lua with `require("lgui2")` -- a
counter window whose buttons are wired straight to Lua callbacks, described as a
Lua table (no `.json` file). Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 09_gui_lgui2.lua
--------------------------------------------------------------------------------
-- Build and drive a LavishGUI 2 (JSON) window from Lua with the bundled `lgui2`
-- module. This demo is game-agnostic: a small counter window with buttons wired
-- straight to Lua callbacks. No .json file is needed -- the window is described
-- as a Lua table and serialized for you.
--
--   * require("lgui2")
--   * gui.load{ elements = {...} }   build a package from a Lua table
--   * inline function event handlers (onPress) -> auto-wired Lua callbacks
--   * gui.element(name)              a handle: :setText / :getText / :exists / :call
--   * gui.exists(name)              is the element currently loaded?
--   * package:unload()              tear the window down + release callbacks
--
-- Button/element callbacks run as atoms -- they must NOT wait(). Do slow work in
-- the main loop instead (as this script does).
--
-- HOW TO RUN:
--     lua 09_gui_lgui2
--   Close the window with its Close button, or:  endlua 09_gui_lgui2
--------------------------------------------------------------------------------

local gui = require("lgui2")

local count = 0
local ui                        -- forward-declared so the button handlers can see it

local WIN = "example_lgui2_win"
local LBL = "example_lgui2_count"

-- Quick helper to push the current count into the label. Called from callbacks
-- (fast, atomic) and from the main loop.
local function render()
    if gui.exists(LBL) then
        gui.element(LBL):setText("Count: " .. count)
    end
end

--------------------------------------------------------------------------------
-- Build the window. Event handlers given as Lua FUNCTIONS are auto-registered and
-- wired through the reverse bridge -- when the button fires, your function runs.
--------------------------------------------------------------------------------
ui = gui.load{
    elements = {
        {
            type = "window",
            name = WIN,
            title = "ISXLUA lgui2 demo",
            content = {
                type = "stackpanel",
                orientation = "vertical",
                children = {
                    { type = "textblock", name = LBL, content = "Count: 0" },
                    { type = "button", name = "incBtn", content = "Increment",
                      onPress = function()
                          count = count + 1
                          render()
                      end },
                    { type = "button", name = "resetBtn", content = "Reset",
                      onPress = function()
                          count = 0
                          render()
                      end },
                    { type = "button", name = "closeBtn", content = "Close",
                      onPress = function() ui:unload() end },
                },
            },
        },
    },
}

if not ui then
    echo("gui.load failed -- is LGUI2 available in this session?")
    return
end

echo("lgui2 window loaded. Click Increment/Reset; Close (or endlua) to quit.")
render()

--------------------------------------------------------------------------------
-- Main loop: keep the script (and its window + callbacks) alive. All the waiting
-- happens here, never in a callback. When the window is gone, we are done.
--------------------------------------------------------------------------------
while gui.exists(WIN) do
    wait(0.5)
end

echo("Window closed -- 09_gui_lgui2 exiting.")
```

---

## 10_gui_lgui1.lua

The LavishGUI 1 (XML) sibling: build and drive an XML-driven window from Lua with
`require("lgui1")` -- a counter window with a commandbutton wired to a Lua
callback, plus nested-child access via the `@` path. Game-agnostic. (Prefer
`lgui2` for new UIs.)

```lua
--------------------------------------------------------------------------------
-- 10_gui_lgui1.lua
--------------------------------------------------------------------------------
-- Build and drive a LavishGUI 1 (XML) window from Lua with the bundled `lgui1`
-- module -- the sibling of `lgui2` for the OLDER, XML-driven UI system. Prefer
-- lgui2 for new UIs; lgui1 is here for parity and for existing XML-based skins.
-- Game-agnostic: a counter window with a commandbutton wired to a Lua callback.
--
--   * require("lgui1")
--   * gui.load{ elements = {...} }   build an XML package from a Lua table
--   * inline function handlers (onCommand) -> auto-wired Lua callbacks
--   * gui.element(name):child(name)  reach a nested element via the "@" path
--   * :setText / :getText / :leftClick / :exists
--   * package:unload()              ui -unload + delete the temp file + release callbacks
--
-- LGUI1 differences from lgui2 are noted inline. Callbacks run as atoms -- no wait().
--
-- HOW TO RUN:
--     lua 10_gui_lgui1
--   Click Increment, then close with:  endlua 10_gui_lgui1
--------------------------------------------------------------------------------

local gui = require("lgui1")

local count = 0
local WIN = "example_lgui1_win"
local LBL = "example_lgui1_count"
local BTN = "example_lgui1_btn"

--------------------------------------------------------------------------------
-- Build the window. LGUI1 uses explicit x/y/width/height on each element, generates
-- XML (not JSON), and its click handler tag for a commandbutton is <Command>, which
-- the module exposes as the `onCommand` event key.
--------------------------------------------------------------------------------
local ui = gui.load{
    elements = {
        {
            type = "Window",
            name = WIN,
            title = "ISXLUA lgui1 demo",
            x = 300, y = 300, width = 240, height = 120,
            children = {
                { type = "Text", name = LBL, text = "Count: 0",
                  x = 10, y = 10, width = 220, height = 20 },
                { type = "commandbutton", name = BTN, text = "Increment",
                  x = 10, y = 40, width = 110, height = 26,
                  onCommand = function()
                      count = count + 1
                      -- Update the label. In lgui1 we reach the nested Text element
                      -- through the parent window's "@" child path.
                      gui.element(WIN):child(LBL):setText("Count: " .. count)
                  end },
            },
        },
    },
}

if not ui then
    echo("gui.load failed -- is LGUI1 available in this session?")
    return
end

echo("lgui1 window loaded. Click Increment; stop with:  endlua 10_gui_lgui1")

-- Property round-trip on the nested Text element via the child handle.
local label = gui.element(WIN):child(LBL)
echo("Label text is currently: '" .. tostring(label:getText()) .. "'")

--------------------------------------------------------------------------------
-- Keep the script (and its window) alive. :unload() (called here on exit) does the
-- ui -unload, deletes the generated temp XML, and releases the callback.
--------------------------------------------------------------------------------
while gui.exists(WIN) do
    wait(0.5)
end

ui:unload()
echo("Window closed -- 10_gui_lgui1 exiting.")
```

---

## 11_game_character_eq2.lua

The object bridge in a real game (EverQuest II via ISXEQ2): the load-and-ready
gate, bare-global TLOs, native scalar members, `Exists()`, `.Member(args)`,
`:Method(args)`, `obj[i]` indexing, chaining, and the typed getters -- all in a
live game context. This is the example to copy the `ensureISXEQ2()` gate from.

```lua
--------------------------------------------------------------------------------
-- 11_game_character_eq2.lua
--------------------------------------------------------------------------------
-- The OBJECT BRIDGE in a real game (EverQuest II via ISXEQ2). This is where the
-- bridge earns its keep: top-level objects are bare Lua globals, scalar members
-- are native Lua values, and members/methods/indexing all "just work".
--
-- Demonstrates (in a real game context):
--   * the load-and-ready gate  (reuse this ensureISXEQ2() in your own scripts)
--   * bare-global TLOs          (Me, Actor, EQ2, Zone, ...)
--   * native scalar leaves      (Me.Name, Me.Level -- real strings/numbers)
--   * scalars in expressions     (arithmetic, comparison, Lua string methods)
--   * Exists()                   (the correct presence test -- Me.Pet, Target)
--   * .Member(args)              (a member that takes arguments)
--   * :Method(args)              (a colon method -- performs an action)
--   * obj[i]                     (the LavishScript numeric index operator)
--   * chaining                   (wrapper.Member.Member ...)
--   * typed getters              (:Str / :Int on an object result)
--
-- The member/method NAMES below come from ISXEQ2 (not ISXLUA); consult the ISXEQ2
-- scripting guide for the full list. Every read here is best-effort and guarded --
-- the script never hard-fails just because you are not fully in-world.
--
-- HOW TO RUN:
--     lua 11_game_character_eq2
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- The idiomatic ISXEQ2 load-and-ready gate. Load the extension if absent, then
-- waituntil it exists AND reports ready -- each step bounded to ~30s. Copy this
-- into any script that touches Me / Actor / game data.
--------------------------------------------------------------------------------
local function ensureISXEQ2()
    IS.WarnUnknownGlobals(false)   -- we probe possibly-absent globals below

    if not Exists(Extension("ISXEQ2")) then
        echo("ISXEQ2 not loaded -- loading it now...")
        IS.Execute("ext isxeq2")
    end
    if not waituntil(function() return Exists(Extension("ISXEQ2")) end, 30) then
        echo("ERROR: ISXEQ2 failed to load within 30 seconds.")
        return false
    end
    -- ISXEQ2 becomes a resolvable global only once it registers its TLO, so guard
    -- against indexing it before then.
    if not waituntil(function() local e = ISXEQ2; return e ~= nil and e.IsReady end, 30) then
        echo("ERROR: ISXEQ2 did not become ready within 30 seconds.")
        return false
    end
    return true
end

if not ensureISXEQ2() then
    return
end
echo("ISXEQ2 is loaded and ready.")

if not Exists(Me) then
    echo("Me is not resolvable yet -- log in / zone in fully, then re-run.")
    return
end

--------------------------------------------------------------------------------
-- 1. Native scalar leaves. Me.Name is a real Lua string; Me.Level a real number.
--    No :Str()/:Int() needed -- use them directly with Lua's own operators.
--------------------------------------------------------------------------------
echo("== Character ==")
echo("Name  : " .. Me.Name)                       -- string concat, direct
echo("Level : " .. Me.Level)                       -- number, coerced in concat
echo("Class : " .. tostring(Me.Class))
echo("Health: " .. tostring(Me.Health) .. "%")

-- Scalars work in arithmetic and comparisons with no ceremony.
if Me.Level >= 90 then
    echo("You are level " .. Me.Level .. " -- max-level-ish content awaits.")
else
    echo("Only " .. (100 - Me.Level) .. " levels to 100 (if that is the cap).")
end
echo("Name uppercased (Lua string method): " .. Me.Name:upper())

--------------------------------------------------------------------------------
-- 2. Exists() -- the correct presence test. A missing object is a truthy NULL
--    wrapper, so `if Me.Pet then` is ALWAYS true; use Exists() instead.
--------------------------------------------------------------------------------
echo("")
echo("== Presence tests ==")
if Exists(Me.Pet) then
    echo("Pet    : " .. Me.Pet.Name)               -- chaining: Me -> Pet -> Name
else
    echo("Pet    : (none)")
end
if Exists(Target) then
    -- Chaining + a typed getter: Target.Name is native; :Str() shown for contrast.
    echo("Target : " .. Target.Name .. "  (distance " ..
        tostring(Target.Distance) .. ")")
else
    echo("Target : (nothing targeted)")
end
if Exists(Zone) then
    echo("Zone   : " .. tostring(Zone.Name))
end

--------------------------------------------------------------------------------
-- 3. .Member(args) and obj[i] -- a member that takes arguments, and the numeric
--    index operator. Group membership is a good example; both forms are shown,
--    guarded because you may be solo.
--------------------------------------------------------------------------------
echo("")
echo("== Member(args) and obj[i] ==")
do
    -- .Member(args): pass the arg in parentheses. (Numbers are coerced to the
    -- exact LS string, e.g. 1 -> "1", so the index is never mis-parsed.)
    local okA, member = pcall(function() return Me.Group(1) end)
    if okA and Exists(member) then
        echo("Me.Group(1) -> " .. member.Name .. "  (.Member(args) form)")
    else
        echo("Me.Group(1): no group member #1 (solo?) -- .Member(args) form shown above.")
    end

    -- obj[i]: index a resolved object with the LavishScript index operator.
    local okB, indexed = pcall(function() return Me.Group[1] end)
    if okB and Exists(indexed) then
        echo("Me.Group[1] -> " .. indexed.Name .. "  (obj[i] index form)")
    else
        echo("Me.Group[1]: not indexable right now (solo?) -- obj[i] syntax shown above.")
    end
end

--------------------------------------------------------------------------------
-- 4. :Method(args) -- a colon method. Methods PERFORM ACTIONS in-game, so we call
--    one only guardedly and pick a benign, reversible one. The point is the call
--    SHAPE: obj:Method(args). Any ISXEQ2 method works this way.
--------------------------------------------------------------------------------
echo("")
echo("== :Method(args) ==")
do
    -- Example benign method: face your character forward. Guarded with pcall so an
    -- unknown method name (which the bridge would flag) can never abort the script.
    local ok = pcall(function() Me:DoFace() end)
    if ok then
        echo("Called Me:DoFace() -- a colon method invoked via the bridge.")
    else
        echo("Me:DoFace() not available in this build -- the obj:Method(args) shape")
        echo("is the same for every method (e.g. Target:DoubleClick(), Actor(id):WaypointTo()).")
    end
end

--------------------------------------------------------------------------------
-- 5. A short live readout loop -- re-reading Me fresh each pass (never hold a
--    wrapper across a wait). Ctrl-stop it with:  endlua 11_game_character_eq2
--------------------------------------------------------------------------------
echo("")
echo("Live readout for ~10s (re-reading values each second)...")
for i = 1, 10 do
    echo(string.format("  [%2ds] HP %s%%  Power %s%%  Level %s",
        i, tostring(Me.Health), tostring(Me.Power), tostring(Me.Level)))
    wait(1)
end

echo("Done.")
```

---

## 12_autoexec_autoload.lua  (sample `autoload.lua`)

Sample content for the base autoexec file. When ISXLUA loads, a file named exactly
`autoload.lua` in your Scripts directory runs automatically. To activate this
demo, copy it into your Scripts directory RENAMED to `autoload.lua` (it is named
`12_autoexec_autoload.lua` here so it does not auto-run just by existing).

```lua
--------------------------------------------------------------------------------
-- 12_autoexec_autoload.lua        (SAMPLE CONTENT for autoload.lua)
--------------------------------------------------------------------------------
-- AUTOEXEC: when ISXLUA loads, if a file named exactly `autoload.lua` exists in
-- your InnerSpace Scripts directory, it is run automatically (exactly like
-- `lua autoload`, with no args) once the extension is up. It is silent if absent.
-- Use it to auto-start your toolbars / handlers / bots whenever ISXLUA loads.
--
-- TO ACTIVATE: copy this file into your Scripts directory RENAMED to `autoload.lua`
--     (this example is named 12_autoexec_autoload.lua on purpose so it does NOT
--      auto-run just by sitting in your Examples/Scripts folder).
--
-- Keep an autoload short: announce, then hand off to your real scripts. If you
-- want a PERSISTENT handler, start a dedicated script (which stays alive) rather
-- than attaching here and returning -- handlers auto-detach when THIS script ends.
--------------------------------------------------------------------------------

echo("[autoload] ISXLUA " .. ISXLUA.Version .. " loaded -- autoload.lua running.")

-- Example: kick off your always-on scripts. (Commented so this sample is inert.)
-- IS.Execute("lua mytoolbar")
-- IS.Execute("lua mychatlogger")

-- Example: a one-off setup task -- record when ISXLUA last loaded.
local state = IS.LoadTable("autoload_state") or { loads = 0 }
state.loads = state.loads + 1
state.lastLoad = os.date("%Y-%m-%d %H:%M:%S")
IS.SaveTable("autoload_state", state)
echo(string.format("[autoload] this is load #%d (last at %s).", state.loads, state.lastLoad))

-- Per-game startup goes in autoload_<game>.lua instead (see the eq2 sample), so it
-- only runs when that game extension is actually loaded and ready.
echo("[autoload] done.")
```

---

## 13_autoexec_autoload_eq2.lua  (sample `autoload_eq2.lua`)

Sample content for the per-game autoexec file. Right after `autoload.lua`, ISXLUA
runs `autoload_eq2.lua` when ISXEQ2 is loaded and ready (and `autoload_eve.lua` /
`autoload_pantheon.lua` for those), watched across a bounded ~30s window. To
activate, copy this into your Scripts directory RENAMED to `autoload_eq2.lua`.

```lua
--------------------------------------------------------------------------------
-- 13_autoexec_autoload_eq2.lua        (SAMPLE CONTENT for autoload_eq2.lua)
--------------------------------------------------------------------------------
-- PER-GAME AUTOEXEC: right after autoload.lua, ISXLUA also runs a per-game file
-- for each KNOWN game extension that is loaded AND ready:
--     autoload_eq2.lua       (when ISXEQ2 is loaded + ready)
--     autoload_eve.lua       (when ISXEVE is loaded + ready)
--     autoload_pantheon.lua  (when ISXPantheon is loaded + ready)
-- Because a game extension can finish loading a moment after ISXLUA, ISXLUA keeps
-- watching for a bounded ~30s window, so a late-loading game still triggers its
-- file exactly once. Silent if the file is absent or that game is not loaded.
--
-- TO ACTIVATE: copy this file into your Scripts directory RENAMED to
--     `autoload_eq2.lua` (named 13_autoexec_autoload_eq2.lua here so it does not
--      auto-run merely by existing).
--
-- Because this file only runs when ISXEQ2 is already loaded + ready, you do NOT
-- need the full load-and-ready gate here -- ISXEQ2 is guaranteed available.
--------------------------------------------------------------------------------

IS.WarnUnknownGlobals(false)

echo("[autoload_eq2] ISXEQ2 is up -- per-game autoexec running.")

-- ISXEQ2 is ready by contract here, but the character may not be fully in-world
-- yet, so still guard game reads with Exists().
if Exists(Me) then
    echo(string.format("[autoload_eq2] Welcome, %s (level %s).",
        Me.Name, tostring(Me.Level)))
else
    echo("[autoload_eq2] Not in-world yet -- your EQ2 scripts can waituntil Exists(Me).")
end

-- Example: auto-start your EQ2-specific bot/toolbar. (Commented so this is inert.)
-- IS.Execute("lua eq2_buffbot")
-- IS.Execute("lua eq2_hud")

echo("[autoload_eq2] done.")
```

---

## 14_pause_resume_reload.lua

The script lifecycle controls: launch a helper script, then pause / resume /
reload / end it (`IS.PauseScript`/`ResumeScript`/`ReloadScript`, the console
`lua -pause`/`-resume`/`-reload`, `luas`, and `endlua`), watching the effect
through `IS.Share`. Game-agnostic.

```lua
--------------------------------------------------------------------------------
-- 14_pause_resume_reload.lua
--------------------------------------------------------------------------------
-- Demonstrates the script LIFECYCLE controls by launching a small helper script,
-- then pausing / resuming / reloading / ending it and watching what happens
-- through the cross-script shared store.
--
--   * launch a script from Lua          IS.Execute("lua <name>")
--   * IS.PauseScript(name)  / lua -pause  <name>   (freeze the scheduler + timers)
--   * IS.ResumeScript(name) / lua -resume <name>   (unfreeze; held time is preserved)
--   * IS.ReloadScript(name) / lua -reload <name>   (restart from file with same args)
--   * list running scripts               luas   (native `scripts` does NOT show .lua)
--   * stop a script                      endlua <name>
--
-- The helper ticks a counter into IS.Share every 0.25s. While PAUSED its ticks
-- freeze; after RESUME they advance again; RELOAD restarts it; endlua stops it.
--
-- HOW TO RUN:
--     lua 14_pause_resume_reload
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
-- Find the InnerSpace Scripts directory from package.path (entries look like
-- "<dir>?.lua"). We write a tiny helper there so we have something to control.
--------------------------------------------------------------------------------
local function detectScriptsDir()
    local fallback
    for entry in string.gmatch(package.path, "[^;]+") do
        if entry:find("%?%.lua$") and (entry:find("^%a:[\\/]") or entry:find("^\\\\")) then
            local dir = entry:gsub("%?%.lua$", "")
            if not dir:find("lua_modules") then return dir end
            fallback = fallback or dir
        end
    end
    return fallback
end

local scriptsDir = detectScriptsDir()
if not scriptsDir then
    echo("Could not locate the Scripts directory -- cannot write the helper.")
    echo("You can still try the controls by hand on any running script:")
    echo("  lua -pause <name> / lua -resume <name> / lua -reload <name>")
    return
end

local helperName = "example_lifecycle_helper"
local helperPath = scriptsDir .. helperName .. ".lua"

-- The helper's source: it ticks a shared counter on a setInterval and then parks.
local HELPER_SRC = [==[
-- example_lifecycle_helper.lua  (auto-generated by 14_pause_resume_reload.lua)
-- Ticks a counter into the shared store every 0.25s. Its timer is frozen while
-- the script is paused and cancelled when the script ends.
local n = 0
IS.Share("example_lifecycle_tick", 0)
setInterval(0.25, function()
    n = n + 1
    IS.Share("example_lifecycle_tick", n)
end)
while true do waitframe() end
]==]

do
    local f, err = io.open(helperPath, "w")
    if not f then
        echo("Could not write the helper file: " .. tostring(err))
        return
    end
    f:write(HELPER_SRC)
    f:close()
end

--------------------------------------------------------------------------------
-- Launch the helper and wait for its first tick to appear.
--------------------------------------------------------------------------------
IS.Share("example_lifecycle_tick", nil)      -- clear any stale value
IS.Execute("lua " .. helperName)
echo("Launched helper '" .. helperName .. "'.")

if not waituntil(function() return IS.Shared("example_lifecycle_tick") ~= nil end, 5) then
    echo("Helper did not start ticking -- aborting.")
    IS.Execute("endlua " .. helperName)
    return
end

wait(1.0)   -- let it tick a few times
echo("Ticks so far: " .. tostring(IS.Shared("example_lifecycle_tick")))

--------------------------------------------------------------------------------
-- PAUSE -- the ticks must freeze while paused.
--------------------------------------------------------------------------------
echo("")
echo("Pausing the helper (IS.PauseScript)...")
IS.PauseScript(helperName)
wait(0.4)
local before = IS.Shared("example_lifecycle_tick")
wait(0.8)
local after = IS.Shared("example_lifecycle_tick")
echo(string.format("While paused: tick was %s, still %s (%s).",
    tostring(before), tostring(after),
    (before == after) and "frozen -- correct" or "NOT frozen"))

-- 'luas' now tags a paused script with [PAUSED].
echo("luas listing (helper should show [PAUSED]):")
IS.Execute("luas")

--------------------------------------------------------------------------------
-- RESUME -- ticks advance again; the paused time is not replayed as a backlog.
--------------------------------------------------------------------------------
echo("")
echo("Resuming (IS.ResumeScript)...")
IS.ResumeScript(helperName)
wait(1.0)
local resumed = IS.Shared("example_lifecycle_tick")
echo(string.format("After resume: tick advanced to %s (was %s).",
    tostring(resumed), tostring(after)))

--------------------------------------------------------------------------------
-- RELOAD -- the script is stopped and re-run from its file with the same args.
--------------------------------------------------------------------------------
echo("")
echo("Reloading (IS.ReloadScript)...")
IS.ReloadScript(helperName)
wait(0.8)
local r1 = IS.Shared("example_lifecycle_tick")
wait(0.8)
local r2 = IS.Shared("example_lifecycle_tick")
echo(string.format("After reload: ticking again (%s -> %s).", tostring(r1), tostring(r2)))

--------------------------------------------------------------------------------
-- END -- endlua stops the helper; its timers are cancelled, so ticks stop.
--------------------------------------------------------------------------------
echo("")
echo("Ending the helper (endlua)...")
IS.Execute("endlua " .. helperName)
wait(0.5)
local e1 = IS.Shared("example_lifecycle_tick")
wait(0.6)
local e2 = IS.Shared("example_lifecycle_tick")
echo(string.format("After endlua: ticks %s (%s).",
    tostring(e1), (e1 == e2) and "stopped -- correct" or "still moving?"))

-- Cleanup: remove the shared value and the generated helper file.
IS.Share("example_lifecycle_tick", nil)
pcall(os.remove, helperPath)
echo("")
echo("Done. The same controls work from the console: lua -pause/-resume/-reload <name>.")
```
