# The Object Model

This is the heart of ISXLUA. A Lua script reaches **every** LavishScript
top-level object (TLO) and datatype directly -- no setup, no per-game code. If a
game extension exposes `${SomeTLO.SomeMember}` to LavishScript, you read it in Lua
as `SomeTLO.SomeMember`. Whatever your loaded game extension registers is
automatically available.

> **Placeholders in this chapter.** Names like `SomeTLO`, `SomeObject`,
> `SomeMember`, `SomeNumber`, and `SomeMethod` are **placeholders** -- substitute
> the real top-level objects, members, and methods your loaded game extension
> provides (see that extension's own scripting guide). The one concrete top-level
> object ISXLUA itself always provides is `ISXLUA` (its status object), used below
> where a real example helps. [`07_Examples.md`](07_Examples.md) shows complete, game-specific
> scripts.

## Top-level objects are bare Lua globals

The TLOs you know from LavishScript are just Lua globals: `ISXLUA`, plus every
top-level object your loaded game extension registers.

```lua
echo(ISXLUA.Version)
echo(SomeTLO.SomeMember)
echo(SomeTLO(1).SomeMember)
```

A real Lua global (or one you define) shadows a TLO of the same name normally --
the TLO lookup only happens when there is no Lua value for that name. If you read
a name that is neither a Lua value nor a TLO, you get `nil` plus a one-time
warning (silence it with `IS.WarnUnknownGlobals(false)`).

## Members: `obj.Member` and `obj.Member(args)`

Read a member with dot syntax. If the member takes arguments, pass them in
**parentheses** -- not the square brackets LavishScript uses.

```lua
-- LavishScript:  ${SomeTLO.Lookup[name].SomeMember}
-- Lua:
SomeTLO.Lookup("name").SomeMember

-- LavishScript:  ${SomeTLO[1]}
-- Lua:
SomeTLO(1)
```

Chaining works the way you would expect -- each member, index, or method result
you can keep chaining from:

```lua
echo(SomeTLO.Lookup("name").SomeMember)
echo(SomeTLO(1).SomeMember)
```

## Methods: `obj:Method(args)`

Methods use Lua's colon syntax:

```lua
ISXLUA:QuietMode()          -- a real, no-argument method
SomeTLO(1):SomeMethod()     -- your game extension's methods
```

(A method is an *action*; a member is a *value*. Same split as LavishScript's
`Object:Method` vs `${Object.Member}`.)

## Scalar results are native Lua values

This is the single most important thing to understand. When a member, TLO, or
index resolves to a **scalar** -- an int, float, boolean, or string -- you get a
**real Lua value**, not a wrapper. So Lua's own operators and methods work
directly:

```lua
if SomeTLO.SomeNumber == 95 then ... end
if SomeTLO.SomeText == "ready" then ... end
if SomeTLO(1).SomeNumber < 10 then ... end
echo(ISXLUA.Version:upper())              -- a Lua string method on a scalar string
local total = SomeTLO.SomeNumber + SomeTLO.OtherNumber   -- arithmetic
```

You do **not** need `:Int()`, `:Str()`, or `tostring` to compare or use a scalar
-- it already is one. (This differs from some early advice; scalars are native.)

Only **object** results (things that themselves have members/methods) come back
as wrappers.

## Object wrappers

An object result -- something like `SomeTLO.SomeObject`, `SomeTLO.Lookup("name")`,
or `SomeTLO(1)` before you read a scalar off it -- is a wrapper. Wrappers:

- **Coerce to text** via `tostring()` and `..` (string concatenation), using the
  object's display text:
  ```lua
  echo("Object: " .. SomeTLO(1))     -- the object coerces to its text
  ```
- **Support numeric operators** (`<`, `<=`, `==`, `+`, `-`, `*`, `/`, `%`, `^`,
  `//`, unary `-`), coercing to a number as needed.
- **Chain** -- read members, call methods, index them.

### Typed getters (for coercing an OBJECT to a scalar)

When you want to pull a plain value out of an *object* wrapper, use a typed
getter. (You do not need these for scalar members -- those are already native.)

| Getter | Returns |
|---|---|
| `obj:Int()` | The object's value as an integer. |
| `obj:Number()` | The object's value as a number (float). |
| `obj:Bool()` | The object's value as a boolean. |
| `obj:Str()` | The object's value as a string. |
| `obj:LSType()` | The LavishScript type name of the object. |
| `obj:IsNull()` | `true` if the object did not resolve to anything real. |
| `obj:Exists()` | `true` if the object resolved to a real, non-null object. |

```lua
local n = SomeTLO.SomeObject:Int()
echo("type name: " .. SomeTLO(1):LSType())     -- the object's LavishScript type name
```

### A real member/method always wins over a getter

If a datatype has a real member or method whose name matches a getter, **you get
the real one**. That is why the type-name helper is `:LSType()` and not `:Type()`
-- many datatypes have a genuine `.Type` member, so `SomeObject.Type` returns that
real member, not the getter. The getters only answer when the type has no member
or method of that name.

## Testing whether something EXISTS

This is a hard Lua rule that trips everyone up: **a bare
`if SomeTLO.SomeOptionalObject then` is ALWAYS true.** An absent object (nothing
there) comes back as a *null object wrapper*, and a Lua userdata can never be
falsy -- so the `if` always passes.

Test presence explicitly with the global **`Exists(x)`** or the **`:Exists()`**
method:

```lua
if Exists(SomeTLO.SomeOptionalObject) then     -- true only if it is really there
    echo("got it: " .. SomeTLO.SomeOptionalObject.SomeMember)
end

if SomeTLO.SomeOptionalObject:Exists() then ... end     -- same idea, method form
if SomeTLO.SomeOptionalObject:IsNull() then ... end     -- the negative
```

`Exists(x)` rules:
- `nil` -> `false`
- a null / unresolved object wrapper -> `false`
- a resolved object, or **any** native value (including `0`, `""`, `false`) ->
  `true`

`Exists` is a plain global, so a game member can never shadow it.

## NULL vs. typo: what raises an error

- A member or method that **fails to resolve** (a member that is absent, an object
  that is not there) yields a **null object wrapper** -- `:IsNull()` is `true`,
  `tostring` is `"NULL"`, the getters return `0` / `false` / `""`. It is **not**
  an error, and chaining off it stays safe.
- A genuinely **misspelled** member or method name (one the datatype does not
  have at all) raises a **Lua error** with a traceback -- so typos surface loudly
  instead of silently returning nothing.
- The one exception is a **bare global**: an unknown global returns `nil` (with a
  one-time warning), because Lua legitimately probes undefined globals.

## Numeric indexing: `obj[i]`

A **numeric** key uses the LavishScript index operator:

```lua
SomeCollection[1]
SomeTLO(1)[1]
```

String keys are unchanged -- `obj.Member` is member access and `obj:Method()` is a
method call. Remember that member *arguments* still use parentheses
(`obj.Member(args)`); the `[i]` form is specifically the numeric index operator.

> Numeric indexing is currently wired on directly-resolved object wrappers. If you
> reach an object *through* an eager member (for example
> `SomeTLO.SomeCollection[1]`), index the resolved object explicitly or use the
> argument form instead.

## Do not hold an object across a `wait()`

Object wrappers point at frame-scoped game data that can go stale after your
script yields. After a `wait()` or `waitframe()`, **re-fetch** from the TLO rather
than reusing a wrapper you captured earlier:

```lua
local o = SomeTLO(1)
wait(1)
-- Don't trust `o` now. Re-fetch:
o = SomeTLO(1)
if Exists(o) then echo(o.SomeMember) end
```

If you only need a value, copy it out as a native scalar *before* the wait (scalar
members are native Lua values, so they are safe to keep).

This rule is important enough that it also appears in [`06_Migration_Gotchas.md`](06_Migration_Gotchas.md).

## The escape hatch

Anything the object model cannot express, you can still do with `IS.Execute` and
`IS.Parse` ([`02_The_IS_Bridge.md`](02_The_IS_Bridge.md)). The two coexist -- mix them freely.

Next: [`04_Timing_And_Events.md`](04_Timing_And_Events.md).
