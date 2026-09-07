# Building GUIs (LavishGUI 2)

InnerSpace has a modern, JSON-driven UI system called **LavishGUI 2** (LGUI2):
windows, buttons, text blocks, checkboxes, lists, and so on, described as JSON
"packages". ISXLUA bundles a small module, **`lgui2`**, that lets you build and
drive those UIs from Lua -- load a package, find elements, read and change them,
and wire a button straight to a Lua function.

```lua
local gui = require("lgui2")
```

The `lgui2` module is a thin convenience layer over the `IS` bridge you already
know (`IS.Execute` / `IS.Parse`) and the reverse bridge (calling Lua from
LavishScript). Anything it does, you could do by hand with those; it just makes
the common cases pleasant.

> LGUI2 itself is an InnerSpace feature and is the same in every game. This
> chapter describes it generically; for a complete, game-specific window see
> [`08_Examples.md`](08_Examples.md).

> **Two UI systems.** LGUI2 (JSON) is the **newer** system and the recommended
> choice for new UIs. InnerSpace also has an older, XML-driven system, **LavishGUI 1
> (LGUI1)**, with its own sibling module, **`lgui1`**. If you are maintaining an
> existing `.xml` UI, jump to [The older system: LavishGUI 1](#the-older-system-lavishgui-1-lgui1)
> at the end of this chapter.

---

## Loading a package file

If you already have a `.json` UI package on disk (in your **Scripts** directory),
load and unload it directly:

```lua
local gui = require("lgui2")

gui.loadFile("myui.json")            -- LGUI2:LoadPackageFile[myui.json]
-- ... use it ...
gui.unloadFile("myui.json")          -- LGUI2:UnloadPackageFile[myui.json]
```

Paths are resolved by LGUI2 the same way the `LGUI2:LoadPackageFile` command
resolves them (relative to your Scripts directory).

---

## Elements: find, read, change

Every UI element has a **name**. `gui.element(name)` returns a small handle you
can read from and act on. The element does not have to exist yet when you make the
handle -- it is looked up fresh on each call by name.

```lua
local win = gui.element("myWindow")

echo(win:exists())                   -- true / false
echo(win:getText())                  -- read the .Text member
win:setText("Hello")                 -- change the text
win:setTitle("My Window")            -- set a property by name (title)
win:hide()                           -- SetVisibility[Hidden]
win:show()                           -- SetVisibility[Visible]
```

### Handle methods

| Method | What it does |
|---|---|
| `el:name()` | The element's name. |
| `el:exists()` | `true` if an element with this name currently exists. |
| `el:get(member)` | Read any member as a string (`${LGUI2.Element[name].member}`). |
| `el:getNumber(member)` | Same, as a number. |
| `el:getBool(member)` | Same, as a boolean. |
| `el:getText()` | Shorthand for `el:get("Text")`. |
| `el:setText(s)` | `:SetText[s]`. |
| `el:setContent(s)` | `:SetContent[s]`. |
| `el:setTitle(s)` | Set the `title` property. |
| `el:set(key, value)` | Set any property by name (`:Set[key,value]`). |
| `el:show()` / `el:hide()` / `el:collapse()` | Set visibility. |
| `el:setVisibility(v)` | Set visibility to `"Visible"` / `"Hidden"` / `"Collapsed"`. |
| `el:isChecked()` / `el:setChecked(b)` | Checkbox state. |
| `el:fireEventHandler(name)` | Fire a named event handler on the element. |
| `el:call(method, ...)` | Escape hatch: call any element method (`:method[arg1,arg2,...]`). |

The setter methods return the handle, so they chain:

```lua
gui.element("status"):setText("Ready"):show()
```

> **A note on special characters.** Because this layer builds LavishScript command
> strings under the hood, the characters `]`, `[`, and `$` inside an element
> *name* or a *value* can confuse the parser. Keep element names to plain
> identifiers. If you need to set text that contains those characters, prefer
> [building the UI from a Lua table](#building-ui-from-a-lua-table) (below), where the text is serialized verbatim.

---

## Building UI from a Lua table

You do not need a separate `.json` file. `gui.load(def)` takes a Lua table,
serializes it to an LGUI2 package (using the bundled `cjson`), and loads it. Pass
either a full package (`{ elements = { ... } }`) or a single element (it is
wrapped for you).

```lua
local gui = require("lgui2")

local ui = gui.load{
    elements = {
        {
            type = "window",
            name = "demoWindow",
            title = "Demo",
            content = "Hello from Lua!",
        },
    },
}

-- ... later, when you are done:
ui:unload()
```

`gui.load` returns a **package handle**:

- `ui:unload()` -- unloads the package (and releases any callbacks it registered).
- `ui:element(name)` -- an element handle for something in the package.

---

## Wiring a button to a Lua function

This is the part that is genuinely new. In an inline table, give an element an
event handler whose value is a **Lua function**. `lgui2` registers it and wires
the element's event to it for you. When the control fires, your function is
called with the element's name and the event name.

```lua
local gui = require("lgui2")

local ui = gui.load{
    elements = {
        {
            type = "window",
            name = "counterWindow",
            title = "Counter",
            content = {
                {
                    type = "button",
                    name = "goButton",
                    content = "Click me",
                    eventHandlers = {
                        onPress = function(element, event)
                            echo("pressed " .. element .. " (" .. event .. ")")
                        end,
                    },
                },
            },
        },
    },
}
```

You can also put the handler directly on the element under the event name
(LGUI2 accepts both styles):

```lua
{ type = "button", name = "goButton", content = "Go",
  onPress = function() echo("clicked!") end }
```

### The atomic-handler rule

An LGUI2 event handler runs as an **atom** -- instantly and to completion. Your
callback therefore **must not** call `wait()`, `waitframe()`, or any blocking
operation (the same rule applies to reverse-bridge functions in general). If a
press needs to start a long-running action, set a flag or publish a message and
let your main loop or another script do the slow work:

```lua
local pending = false

local ui = gui.load{
    elements = {
        { type = "window", name = "w", title = "Worker",
          content = {
            { type = "button", name = "start", content = "Start",
              onPress = function() pending = true end },
          } },
    },
}

while true do
    if pending then
        pending = false
        echo("doing the slow work now...")
        wait(2)                      -- fine here -- NOT inside the handler
    end
    waitframe()
end
```

---

## Callbacks for a file-based UI

If your UI lives in a `.json` file, author the button's handler as a `code`
handler that runs `luacall`, and register the Lua function with the same name:

```json
{ "type": "button", "content": "Go",
  "eventHandlers": { "onPress": { "type": "code", "code": "luacall onGo" } } }
```

```lua
local gui = require("lgui2")

gui.register("onGo", function()
    echo("the file-based button was pressed")
end)

gui.loadFile("myui.json")
```

`gui.register(name, fn)` / `gui.unregister(name)` are thin wrappers over the
reverse bridge (`IS.Register` / `IS.Unregister`), so `${ISXLUA.Call[onGo]}` and
`luacall onGo` both reach your function.

---

## Cleaning up

Registered callbacks are released automatically when your script ends, so a
stale `luacall` can never fire into a dead script. The **window itself is not**
-- if your script ends without unloading its UI, the window stays on screen (just
like any LavishScript UI script). Unload it explicitly when you are done:

```lua
local ui = gui.load{ ... }
-- ...
ui:unload()                          -- for a gui.load package
-- or, for a file you loaded yourself:
gui.unloadFile("myui.json")
```

A common pattern is to unload at the end of `main`, or when a "Close" button is
pressed:

```lua
local ui
ui = gui.load{
    elements = {
        { type = "window", name = "w", title = "Closable",
          content = {
            { type = "button", name = "close", content = "Close",
              onPress = function() ui:unload() end },
          } },
    },
}

while gui.exists("w") do
    waitframe()
end
```

---

## Simple example

A one-window status display your script updates over time:

```lua
local gui = require("lgui2")

local ui = gui.load{
    elements = {
        { type = "window", name = "statusWindow", title = "Status",
          content = { type = "textblock", name = "statusText", content = "starting..." } },
    },
}

local text = ui:element("statusText")

for i = 1, 5 do
    text:setText("tick " .. i)
    wait(1)
end

ui:unload()
```

---

## Complex example

A little control panel: a text field showing a running count, a **+1** button, a
**Reset** button, and a **Close** button -- all wired to Lua functions, with the
script looping until the window is closed.

```lua
local gui = require("lgui2")

local count = 0
local ui                             -- forward declaration so handlers can see it

local function refresh()
    ui:element("countText"):setText("Count: " .. count)
end

ui = gui.load{
    elements = {
        {
            type = "window",
            name = "panel",
            title = "Control Panel",
            content = {
                type = "stackpanel",
                orientation = "vertical",
                children = {
                    { type = "textblock", name = "countText", content = "Count: 0" },
                    { type = "button", name = "incBtn", content = "+1",
                      onPress = function() count = count + 1; refresh() end },
                    { type = "button", name = "resetBtn", content = "Reset",
                      onPress = function() count = 0; refresh() end },
                    { type = "button", name = "closeBtn", content = "Close",
                      onPress = function() ui:unload() end },
                },
            },
        },
    },
}

refresh()

-- keep the script alive while the window is up
while gui.exists("panel") do
    waitframe()
end

echo("panel closed")
```

Because the button handlers only touch Lua state (`count`) and call quick element
setters, they obey the atomic-handler rule. The slow part -- waiting -- happens in
the main loop, never in a handler.

---

## What this layer does and does not do

- It **does** cover loading packages (file or inline table), finding elements,
  reading and setting element state, showing/hiding, and wiring element events to
  Lua functions.
- It **does not** yet add a way to attach a Lua callback to an element that was
  loaded by *some other* script or file the module did not build. For those, use a
  `code` handler with `luacall` ([above](#callbacks-for-a-file-based-ui)), or `IS.AttachEvent` for a custom event.
- Reading element state uses `IS.Parse`, so values come back as strings unless you
  use `el:getNumber` / `el:getBool`. That is fine for UIs; it also means there is
  no object-lifetime hazard to worry about (each call re-resolves the element by
  name).

For the full LGUI2 element/property vocabulary (which element types exist, what
properties they take), consult InnerSpace's own LavishGUI 2 documentation -- every
element and property it describes is reachable through the handles here.

---

## The older system: LavishGUI 1 (`lgui1`)

LavishGUI 1 (LGUI1) is InnerSpace's **older, XML-driven** UI system. LGUI2 (above)
is newer and is the recommended choice for new UIs, but LGUI1 is still fully
supported and there are many existing `.xml` UIs. The bundled **`lgui1`** module is
the sibling of `lgui2`: the same idea (load a UI, find elements, read/change them,
wire a button to a Lua function), adapted to LGUI1's XML-and-commands model.

```lua
local gui = require("lgui1")
```

LGUI1 differs from LGUI2 in a few ways the module surfaces:

- UIs are **XML files** loaded with `ui -load` (LGUI2 uses JSON packages).
- Visibility is `:show()` / `:hide()` / `:toggleVisible()` (there is no "collapsed"
  state).
- A checkbox is `:setChecked(true/false)` / `:toggleChecked()`.
- A button's click handler is its **command**; wire it with an `onCommand` function.

### Loading an XML file

```lua
local gui = require("lgui1")

gui.loadFile("MyWindow.xml")                       -- ui -load "MyWindow.xml"
gui.loadFile("MyWindow.xml", { skin = "EQ2-Green" })  -- with a skin
-- ... use it ...
gui.unloadFile("MyWindow.xml")                     -- ui -unload "MyWindow.xml"
```

`loadFile`'s options table accepts `skin` (a skin name) and `parent` (an
`"element@path"` to load the UI into an existing element). `gui.reloadFile(path)`
reloads a file in place.

### Elements: find, read, change

`gui.element(name)` returns a handle that is re-resolved by name on each call.
LGUI1 addresses a nested element with an **`@` path** (`child@parent@window`);
`el:child(name)` builds that path for you.

```lua
local win = gui.element("MyWindow")

echo(win:exists())                   -- true / false
echo(win:isVisible())
win:hide()
win:show()

local label = win:child("StatusText")     -- UIElement[StatusText@MyWindow]
label:setText("Ready")

local box = gui.element("MyCheckbox")
box:setChecked(true)
echo(box:isChecked())
```

Handle methods: `:name()`, `:exists()`, `:get(member)` / `:getNumber` / `:getBool`,
`:getText()` / `:setText(s)`, `:setX/:setY/:setWidth/:setHeight`,
`:show/:hide/:toggleVisible/:isVisible/:setVisible`,
`:isChecked/:setChecked/:toggleChecked`, `:leftClick()`, `:child(name[,type])`, and
`:call(method, ...)` as a generic escape hatch. The setters chain.

### Building UI from a Lua table

`gui.load(def)` takes a Lua table, generates LGUI1 XML, and loads it. Pass either a
full package (`{ elements = { ... } }`) or a single element. Give an event a Lua
**function** and it is wired to your function automatically (called as
`fn(elementName, eventName)`); a common one for a `commandbutton` is `onCommand`.

```lua
local gui = require("lgui1")

local ui = gui.load{
    elements = {
        {
            type = "Window", name = "demoWin", title = "Demo",
            x = 200, y = 200, width = 220, height = 120,
            children = {
                { type = "Text", name = "demoLbl", text = "Hello from Lua!",
                  x = 10, y = 10, width = 200, height = 20 },
                { type = "commandbutton", name = "demoBtn", text = "Close",
                  x = 10, y = 40, width = 100, height = 24,
                  onCommand = function() ui:unload() end },
            },
        },
    },
}
```

`gui.load` returns a package handle with `:unload()` (unloads the UI and releases
any callbacks it wired) and `:element(name)`. Common element property keys the
builder understands: `x`, `y`, `width`, `height`, `text`, `visible`, `alpha`,
`tooltip`, `alignment`; any other simple property can go in a `props = { Tag =
value }` table. `children` is an array of child elements. (Complex property
elements such as `<Font>`, which take nested children, are best authored in a
file-based `.xml` package.)

> The same **[atomic-handler rule](#the-atomic-handler-rule)** applies as for LGUI2: a button handler runs to
> completion instantly and must not `wait()`. Set a flag and let your main loop do
> slow work.

### Callbacks for a file-based XML UI

If your UI is an `.xml` file, put `luacall <name>` in the element's command/handler
text, and register the Lua function with the same name:

```xml
<commandbutton name='GoButton'>
    <Text>Go</Text>
    <Command>luacall onGo</Command>
</commandbutton>
```

```lua
local gui = require("lgui1")

gui.register("onGo", function()
    echo("the XML button was pressed")
end)

gui.loadFile("MyWindow.xml")
```

`gui.register(name, fn)` / `gui.unregister(name)` are the same reverse-bridge
wrappers as in `lgui2`. Registered callbacks are released automatically when your
script ends; the window is not -- call `:unload()` / `gui.unloadFile` when you are
done.

### What this layer does not do

Like `lgui2`, it does **not** yet add a way to attach a Lua callback to an element
that some *other* script or file loaded. For those, use a `luacall` handler in the
XML ([above](#callbacks-for-a-file-based-xml-ui)), or `IS.AttachEvent` for a custom event.

Next: [`06_Bundled_Libraries.md`](06_Bundled_Libraries.md).
