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
> [`06_Migration_Gotchas.md`](06_Migration_Gotchas.md).

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

## Limits

You can have up to **64 distinct events** attached at once (across all scripts).
Attaching handlers to the *same* event does not count against this -- only the
number of different event names does.

Next: [`05_Bundled_Libraries.md`](05_Bundled_Libraries.md).
