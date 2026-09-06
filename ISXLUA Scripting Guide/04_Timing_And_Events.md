# Timing and Events

Two things make a script do work over time: **cooperative waiting** (yielding the
script for a while and resuming later) and **events** (running a function when
something happens in the game).

## `wait(seconds)` and `waitframe()`

`wait(seconds)` suspends your script for that many **seconds** (a float), then
resumes it. `waitframe()` suspends until the next frame.

```lua
wait(1)       -- one second
wait(0.5)     -- half a second
wait(2.5)     -- two and a half seconds
waitframe()   -- resume next frame
```

> **Coming from LavishScript?** LavishScript's `wait` counts **tenths of a
> second**; ISXLUA's `wait` counts **seconds**. An LS `wait 10` (one second)
> becomes `wait(1)` here. This is the single most common porting mistake -- see
> [`07_Migration_Gotchas.md`](07_Migration_Gotchas.md).

While a script is waiting, other scripts keep running and the game keeps ticking;
your script simply resumes when its time is up. A typical polling loop looks like:

```lua
local running = true
while running do
    -- do a little work (read a value your game extension provides)
    echo("value: " .. SomeTLO.SomeNumber)
    wait(1)
end
```

Because ISXLUA resumes waiting scripts each frame, a loop like this costs almost
nothing between iterations.

**Remember the object-lifetime rule:** do not keep an object wrapper across a
`wait()` / `waitframe()`. Re-fetch it from its TLO afterward, or copy out scalar
values (which are native and safe to keep) before you wait. See
[`03_Object_Model.md`](03_Object_Model.md).

## `waituntil(condfn, timeoutSeconds)` -- wait for a condition

`waituntil` is the clean replacement for a hand-rolled polling loop. You give it a
**condition function** and (optionally) a **timeout in seconds**. It yields your
script and re-checks the condition every frame; it resumes the moment the condition
returns a truthy value, or when the timeout elapses -- whichever comes first.

It **returns `true` if the condition was met**, or **`false` if it timed out**, so
you can branch on the outcome.

```lua
-- Wait up to 30 seconds for some condition to become true.
if waituntil(function() return someReadyCheck() end, 30) then
    echo("ready")
else
    echo("timed out")
end
```

Compare that with the equivalent by hand -- `waituntil` replaces all of this:

```lua
local waited = 0
while not someReadyCheck() do
    if waited >= 30 then break end
    wait(0.5)
    waited = waited + 0.5
end
```

The condition function takes no arguments and returns a value that is checked for
truthiness (in Lua, everything except `false` and `nil` is truthy).

**Omit the timeout to wait indefinitely:**

```lua
waituntil(function() return done end)   -- no timeout: waits until done is truthy
```

The condition is re-evaluated once per frame, so keep it cheap -- read a value and
compare it, do not do heavy work inside it. If the condition function raises an
error, it is reported to the console (with a traceback) and `waituntil` returns
`false`.

Because `waituntil` yields, the same rule as `wait()` applies: **you cannot call it
inside an event handler** (handlers run atomically -- see below), and do not hold an
object wrapper across it.

### `wait(seconds, condfn)` -- a timed wait that can finish early

`wait` also accepts an optional condition function as a second argument. This waits
**up to** `seconds`, but resumes early the moment the condition becomes truthy. It
is the same behavior as `waituntil` with the arguments in the order a LavishScript
scripter tends to expect (time first). Like `waituntil`, it returns `true` if the
condition was met and `false` if the full time elapsed:

```lua
-- Give it at most 5 seconds, but continue as soon as ready() is true.
local met = wait(5, function() return ready() end)
```

Plain `wait(seconds)` with no condition returns nothing, exactly as before.

## `waitforevent(name, timeoutSeconds)` -- wait for the next event

Sometimes you do not want a handler that runs on *every* occurrence of an event --
you just want your script to **pause until the next time it fires**, then continue
inline. `waitforevent` does that: it yields your script until the named event fires,
then resumes and **returns `true` plus the event's arguments** (each a string). Give
it an optional **timeout in seconds**; if the event does not fire in time it resumes
and **returns `false`** instead (and nothing else).

The event names come from whatever game extension you have loaded -- the
`GameExtension_onSomething` names below are placeholders; use the real ones your
loaded extension documents. (Events are covered in full [below](#events).)

```lua
-- Wait up to 10 seconds for the next "something happened" event.
local fired, a, b = waitforevent("GameExtension_onSomething", 10)
if fired then
    echo("it fired; first arg = " .. tostring(a))
else
    echo("timed out -- no event within 10 seconds")
end
```

**Omit the timeout to wait indefinitely:**

```lua
local fired, text = waitforevent("GameExtension_onSomething")
-- with no timeout, fired is always true when it returns
```

It is a **one-shot** wait -- it returns on the *next* firing only. To react to every
firing, either call it again in a loop, or attach a persistent handler with
[`IS.AttachEvent`](#isattacheventname-fn). Like `wait()` and `waituntil()`,
`waitforevent` **yields**, so you cannot call it inside an event handler or a
[timer callback](#timers----settimeout-setinterval-cleartimer) (both run
atomically), and do not hold an object wrapper across it.

## Events

An event lets you run a Lua function when something happens -- a chat line
arrives, a zone changes, and so on. The set of available events comes from the
game extension you have loaded (each names its own events, for example a chat or
zone event); the mechanism below is the same for all of them. The names below
(`GameExtension_onSomething`) are placeholders -- use the real event names your
loaded extension documents.

**A Lua function *is* the event target.** There is no separate "atom" concept to
declare as in LavishScript -- you just hand ISXLUA a function.

### `IS.AttachEvent(name, fn)`

Registers `fn` to be called whenever the named event fires. Your function
receives the event's arguments as **varargs**, each a string:

```lua
IS.AttachEvent("GameExtension_onSomething", function(text, ...)
    echo("event fired: " .. tostring(text))
end)
```

- Multiple scripts (and multiple functions) may attach to the same event.
- `IS.AttachEvent` returns the function you passed, so you can keep a reference to
  it for later detaching.
- Handlers **auto-detach when their script ends** -- you do not have to clean up
  on exit.

### `IS.DetachEvent(name, fn)`

Removes a handler you previously attached (matched by identity). Returns `true` if
one was removed.

```lua
local function onSomething(text) echo("event: " .. tostring(text)) end
IS.AttachEvent("GameExtension_onSomething", onSomething)
-- ...later...
IS.DetachEvent("GameExtension_onSomething", onSomething)
```

Keep the function in a variable if you intend to detach it -- you need the same
value you attached.

### `IS.FireEvent(name, ...)`

Fires an event yourself, passing extra arguments (converted to strings) to every
handler:

```lua
IS.FireEvent("MyScript_onSomething", "payload", 42)
```

This is handy for your own custom events, or for testing a handler.

### `IS.EventSource()`

Inside a handler, `IS.EventSource()` returns the **object the event came from** --
LavishScript's event "this" -- as an ISXLUA object, so you can read members off it
just like any other object result. It returns a NULL object if the event carried no
source (or if you call it outside a handler), so guard it with `Exists()`.

```lua
IS.AttachEvent("GameExtension_onSomething", function(text)
    local src = IS.EventSource()
    if Exists(src) then
        echo("from: " .. src.Name)   -- read a member off the originating object
    end
end)
```

This is an **opt-in** accessor: your handler's `(arg1, arg2, ...)` string arguments
are exactly as before -- the source object is *not* one of them, so nothing about
existing handlers changes. Call `IS.EventSource()` only in the handlers that need
it. (For the object model -- `.Member`, `:Method`, `Exists`, the typed getters --
see [`03_Object_Model.md`](03_Object_Model.md).)

## Handlers run atomically -- no `wait()` inside them

An event handler runs **atomically**, exactly like a LavishScript atom: it must
run to completion in a single step. **You cannot call `wait()` or `waitframe()`
inside a handler** -- doing so is reported to the console as an error (it does not
crash anything). If a handler needs to do timed work, have it set a flag or push
some data into a table, and let your main script loop (which *can* wait) act on it.

```lua
local pending = {}

IS.AttachEvent("GameExtension_onSomething", function(text)
    -- fast, no waiting -- just record it
    pending[#pending + 1] = text
end)

-- main loop handles the queued work, and here waiting is fine
while true do
    while #pending > 0 do
        local item = table.remove(pending, 1)
        echo("saw: " .. tostring(item))
    end
    wait(0.5)
end
```

Any error thrown inside a handler is printed to the console -- it will not take
down the game or your script.

## Timers -- `setTimeout`, `setInterval`, `clearTimer`

Timers run a function *later* without stopping your script. Unlike `wait()`, they do
not suspend anything -- you schedule a function and your code keeps running.

- **`setTimeout(seconds, fn)`** runs `fn` **once**, `seconds` from now. Returns a
  **handle**.
- **`setInterval(seconds, fn)`** runs `fn` **repeatedly**, every `seconds`. Returns a
  **handle**.
- **`clearTimer(handle)`** cancels a timer (either kind) by its handle. Returns
  `true` if it cancelled one.

```lua
-- one-shot: fire once, 5 seconds from now
setTimeout(5, function() echo("5 seconds later") end)

-- repeating: every 2 seconds, until we cancel it
local ticks = 0
local h = setInterval(2, function()
    ticks = ticks + 1
    echo("tick " .. ticks)
    if ticks >= 3 then
        clearTimer(h)   -- stop after three
    end
end)
```

Timers belong to the script that created them and are **cancelled automatically when
the script ends** -- you do not have to clear them on exit.

**Timer callbacks run atomically, exactly like event handlers:** the function takes
no arguments, must run to completion, and **cannot call `wait()` / `waitframe()` /
`waituntil()` / `waitforevent()`** (doing so is reported as an error -- it does not
crash anything). Keep them short; if a callback needs to wait, have it set a flag or
queue some data for your main loop (which *can* wait) to act on.

The resolution is one frame, so a very small interval (or `0`) simply runs the
callback about once per frame.

## Asynchronous HTTP -- `IS.HttpGet`, `IS.HttpPost`

> **Requires the "with libisxgames" build of ISXLUA.** These two functions exist
> **only** in that build. In the plain build they are simply **absent** from the `IS`
> table, so you can feature-detect them before use:
>
> ```lua
> if IS.HttpGet then
>     -- HTTP is available in this build
> end
> ```

`IS.HttpGet` and `IS.HttpPost` make an HTTP request **without blocking your script**.
They return immediately; when the response arrives, a **callback** you supply is
run. The callback receives `(ok, status, body)`:

- **`ok`** -- `true` when the server answered with a 2xx status; `false` otherwise
  (including a timeout or a connection failure).
- **`status`** -- the HTTP status code as a number (`200`, `404`, ...), or `0` if no
  response arrived (timeout / connection failure).
- **`body`** -- the response body as a string (empty on failure).

Each call returns an opaque **handle** (a number) identifying the request.

### `IS.HttpGet(url, callback)`

```lua
IS.HttpGet("https://example.com/api/status", function(ok, status, body)
    if ok then
        echo("got " .. #body .. " bytes, status " .. status)
    else
        echo("request failed (status " .. status .. ")")
    end
end)
```

### `IS.HttpPost(url, body [, contentType], callback)`

`body` can be a **string**, a number (converted to text), or a **table** -- and a
table is **auto-encoded** for you. The optional `contentType` (for example
`"application/json"`) is sent as the request's content type when given.

**Table body -> JSON (the default).** Pass a table and it is encoded as JSON with the
bundled `cjson`, and the content type defaults to `application/json` automatically --
you do not have to encode or set the header yourself:

```lua
-- POST JSON -- just pass the table; ISXLUA encodes it and sets Content-Type:
IS.HttpPost("https://example.com/api/report", { name = "test", value = 42 },
    function(ok, status, body)
        if ok then
            echo("reported ok")
        else
            echo("report failed: " .. status)
        end
    end)
```

**Table body -> form fields.** If you pass a content type that names form encoding
(`"application/x-www-form-urlencoded"`), the same table is instead url-encoded into
`key=value&...` pairs:

```lua
IS.HttpPost("https://example.com/submit", { user = "bob", score = 10 },
    "application/x-www-form-urlencoded",
    function(ok, status, body) echo("done: " .. tostring(ok)) end)
```

**What can be encoded.** For JSON, the same rules as the bundled `cjson` apply
(see [`06_Bundled_Libraries.md`](06_Bundled_Libraries.md)); a value that cannot be
serialized (a function or userdata) raises a clear error. For form encoding the table
must be **flat**: keys are strings/numbers, values are strings/numbers/booleans, and a
nested table (or a function/userdata) raises a clear error.

**String body (unchanged).** A string (or number) body is sent as-is, with the content
type you give -- or none:

```lua
-- Explicit string body with a content type:
IS.HttpPost("https://example.com/api/raw", "<xml/>", "text/xml",
    function(ok, status, body) echo("done: " .. tostring(ok)) end)

-- Without a content type, omit it (the callback is then the third argument):
IS.HttpPost("https://example.com/hook", "raw body text", function(ok, status, body)
    echo("done: " .. tostring(ok))
end)

-- You can still pre-encode yourself if you prefer (see 06_Bundled_Libraries.md):
local cjson = require("cjson")
IS.HttpPost("https://example.com/api/report", cjson.encode({ a = 1 }),
    "application/json", function(ok, status, body) end)
```

### The callback runs atomically -- no `wait()` inside it

Like an event handler or a timer callback, the completion callback runs
**atomically**: it must run to completion and **cannot call `wait()` /
`waitframe()` / `waituntil()` / `waitforevent()`**. If the response needs to kick off
timed work, record it (set a flag or push it into a table) and let your main loop --
which *can* wait -- act on it. An error thrown inside the callback is printed to the
console (with a traceback) and never crashes the game.

### Notes

- **These require a running script.** Call them from a script (`lua`/`run`), not from
  a `lua -c "..."` one-liner -- the callback would outlive the one-liner's state. A
  request is also dropped cleanly if its script ends before the response arrives.
- **Timeouts / connection failures.** A request that gets no response within a fixed
  timeout (about 30 seconds), or that fails to connect at all, completes with
  `ok = false, status = 0` so your callback always runs exactly once.
- **Concurrent requests are matched correctly.** Each request is tracked
  individually, so you can fire several at once -- even to the *exact same URL* -- and
  every callback receives its own response. (If a request is redirected to a different
  address, correlation for that one request falls back to matching by URL, which is
  only ambiguous in the rare case of several redirected same-URL requests in flight at
  once.)

## Sharing data between scripts

Every script runs in its own isolated Lua state, so a global in one script is
invisible to another and you cannot pass a table straight from one to the next.
ISXLUA gives you two ways to share across that boundary -- a **value store** and a
**message bus** -- and both work by **deep-copying** the data (each script always
gets its own private copy; nothing is ever shared by reference).

### What can be copied

The same rules apply to both facilities:

- **Copyable:** `nil`, booleans, numbers, strings, and **tables** of those (nested
  as deep as you like), with string or integer keys.
- **Not copyable (raises an error):** functions, userdata/threads, and tables that
  contain a **reference cycle**. Object wrappers from the object model are not
  copyable either -- copy out the scalar values you need (`.Name`, `.Level`, ...)
  into a plain table first.

### The shared value store -- `IS.Share`, `IS.Shared`

```lua
-- in one script:
IS.Share("boss", { name = "Fippy", hp = 100, adds = { "a", "b" } })

-- in another script (or the same one):
local boss = IS.Shared("boss")
if boss then echo(boss.name .. " has " .. boss.hp .. " hp") end
```

- **`IS.Share(key, value)`** stores a deep copy of `value` under a string `key`.
  Passing `nil` as the value removes the key.
- **`IS.Shared(key)`** returns a fresh deep copy of the stored value, or `nil` if
  the key was never set (so `local t = IS.Shared("k") or {}` is a safe idiom).

Because each side gets a copy, changing the table you got back from `IS.Shared` does
**not** change the stored value -- call `IS.Share` again to publish an update. Shared
values persist until you overwrite or remove them (or ISXLUA unloads); they are not
tied to the lifetime of the script that set them.

### The message bus -- `IS.Publish`, `IS.Subscribe`, `IS.Unsubscribe`

Where the store is a shared *value*, the bus is a broadcast: one script publishes to
a named **channel**, and every script subscribed to that channel has its callback run
with a copy of the published arguments.

```lua
-- subscriber (in any script):
local handle = IS.Subscribe("alerts", function(kind, detail)
    echo("ALERT [" .. kind .. "]: " .. tostring(detail))
end)

-- publisher (in any script, including the same one):
local n = IS.Publish("alerts", "low-health", 15)   -- delivers to every subscriber
echo("delivered to " .. n .. " subscriber(s)")
```

- **`IS.Subscribe(channel, fn)`** registers `fn` as a subscriber of `channel` and
  returns a **handle**.
- **`IS.Unsubscribe(channel)`** removes all of *your* subscriptions to a channel;
  pass the handle -- `IS.Unsubscribe(channel, handle)` -- to remove just that one.
- **`IS.Publish(channel, ...)`** delivers a deep copy of the arguments to every
  subscriber on the channel (across all scripts, including the publisher) and returns
  how many callbacks ran.

Subscriptions belong to the subscribing script and are **removed automatically when
that script ends** -- no cleanup needed on exit. `IS.Subscribe` needs a running
script, so call it from a `.lua` script, not a `lua -c "..."` one-liner.

**Subscriber callbacks run atomically, like event handlers:** they must run to
completion and **cannot call `wait()` / `waitframe()` / `waituntil()` /
`waitforevent()`**. If a message needs to start timed work, record it (set a flag or
queue it in a table) and let your main loop act on it. An error in a subscriber is
printed to the console (with a traceback) and never crashes the game or the publisher.

## Limits

You can have up to **64 distinct events** attached at once (across all scripts).
Attaching handlers to the *same* event does not count against this -- only the
number of different event names does.

Next: [`05_Building_GUIs.md`](05_Building_GUIs.md).
