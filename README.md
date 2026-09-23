# NC

A single-file, dependency-free UI library for Roblox executor environments.
Dark, purple-accented, heavily animated, and fully themeable at runtime.

No dependencies. No asset ids. No external services. One file you load and build
against — the entire library is ~10,000 lines of Lua, currently **Gen 1.1.0**.

---

## New here? Start in three steps

You need an **executor** (the program that runs Lua inside Roblox). Any popular
one works — NC uses nothing exotic.

### Step 1 — Copy this

```lua
local NC = loadstring(game:HttpGet("https://raw.githubusercontent.com/Naru571/NC/refs/heads/main/NC"))()

NC:Init({
    name = "My Script",
    build = function(window)
        local tab = window:CreateTab({ name = "Main", icon = "grid" })
        local section = tab:CreateSection({ name = "Movement" })

        section:CreateToggle({
            name = "Speed enabled",
            flag = "SpeedEnabled",
            callback = function(state)
                print("toggled:", state)
            end,
        })
    end,
})
```

### Step 2 — Paste it into your executor and run

That's it. No dependencies to install, no files to download separately.

### Step 3 — Press **N**

A loading screen appears, then the window. Press **N** any time to hide or show
it. There is also a floating **Show NC** pill when you minimise the window.

### What you should see

- A **loading screen** with a progress bar, then a cross-fade into the window.
- Top-left: three **macOS-style dots** — red closes, yellow minimises, green
  zooms (you can turn them off in Settings).
- On the left, a **navigation rail** with your tabs.
- In the middle, your **section** with the toggle inside it.
- Top-right: **search** and **settings** (plus minimise/close/expand buttons if
  you switch the dots off).

If nothing appears, jump to [Troubleshooting](#troubleshooting).

### Key terms in 30 seconds

New to scripting? These five words cover 90% of this document:

| Term | Meaning |
| ---- | ------- |
| **Executor** | The app that injects and runs your Lua in Roblox. |
| **loadstring** | A Roblox function that runs a string of Lua code. We download the library as text and run it. |
| **Callback** | A function *you* write and hand to NC; NC calls it when something happens (toggle flipped, slider moved, button clicked). |
| **Flag** | A name you give an element so its value can be saved and restored by configs. |
| **Handle** | The table you get back when creating an element. It lets you change that element later (`:Set`, `:Get`, `:Lock`, ...). |

The whole library follows one pattern: **window → tab → section → element**.
Everything else is detail on top of those four things.

---

## What's new in Gen 1.1.0

- **macOS traffic lights** — red/yellow/green dots in the header: red closes
  (with a confirm dialog), yellow minimises to the pill, green zooms the window
  to fit your screen. They spring larger on hover and pop in with a staggered
  animation. Switching the setting on tells you what each colour does
  (*"Red closes · Yellow minimises · Green zooms"*).
- **Expand button on the classic controls** — with the dots off, the top-right
  bar is search, settings, expand, minimise, close, so you can zoom without the
  dots.
- **The NC logo follows your accent** — change the colour in Settings and the
  drawn NC mark recolours with it (the C and beads take the accent, the N stays
  white).
- **Settings panel refresh** — new header with a gradient icon chip, icons on
  every group, a ring marking your active accent swatch/theme, and the theme
  chips now wrap onto two rows instead of clipping.
- **Player info fixes** — the info card is fully themed, "Copy all" copies the
  **full** server ID (not the shortened one), and the eye toggle now hides the
  info button and card along with your name.
- **No more FPS drop on colour changes** — theme changes repaint in small
  per-frame slices through `RunService` instead of one big stall.

---

## Your first real element

Once the above works, adding elements is a matter of adding lines inside your
`build` function.

**A slider** — give it a range, a starting value, and a callback:

```lua
section:CreateSlider({
    name = "Walk speed",
    description = "Studs per second",
    flag = "WalkSpeed",
    min = 16, max = 250, step = 1, value = 16,
    callback = function(value)
        local player = game.Players.LocalPlayer
        local humanoid = player.Character
            and player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid.WalkSpeed = value end
    end,
})
```

**A button**:

```lua
section:CreateButton({
    name = "Say hello",
    icon = "star",
    callback = function()
        NC:Notify({ Title = "Hello", Content = "It works.", Type = "success" })
    end,
})
```

**A second tab** — just call `CreateTab` again:

```lua
local visuals = window:CreateTab({ name = "Visuals", icon = "eye" })
local esp = visuals:CreateSection({ name = "Player ESP" })
```

> **`flag` is how settings get remembered.** Give an element a `flag` and its
> value is saved and restored automatically. Leave it off and the element resets
> every time. See [Configs](#configs).

---

## The full example

`NC_example.lua` is a complete, working script that uses **every** component and
every library feature — tabs, sections, groups, toggles, sliders, dropdowns,
multi-selects, inputs, labels, paragraphs, dividers, buttons, notifications,
configs, themes, plugins and the lock API.

It is the fastest way to learn the library. Read it top to bottom, or copy the
parts you need.

---

## Contents

- [New here? Start in three steps](#new-here-start-in-three-steps)
- [What's new in Gen 1.1.0](#whats-new-in-gen-110)
- [Installation options](#installation-options)
- [Window](#window)
- [Tabs](#tabs)
- [Sections and groups](#sections-and-groups)
- [Elements](#elements)
- [Value handles](#value-handles)
- [Notifications](#notifications)
- [Appearance](#appearance)
- [Configs](#configs)
- [Player info](#player-info)
- [Show and hide keybind](#show-and-hide-keybind)
- [Traffic lights](#traffic-lights)
- [Minimise and the floating pill](#minimise-and-the-floating-pill)
- [Plugins](#plugins)
- [Rayfield Gen2 compatibility](#rayfield-gen2-compatibility)
- [Icons and animations](#icons-and-animations)
- [Executor support](#executor-support)
- [Verifying a build](#verifying-a-build)
- [Troubleshooting](#troubleshooting)

---

## Installation options

### Option A — load from a URL (recommended)

```lua
local NC = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/Naru571/NC/refs/heads/main/NC"
))()
```

`loadstring` compiles the downloaded source; the trailing `()` runs it and hands
you back the library table.

Raw GitHub URLs are cached by **both** the executor and GitHub's CDN. If you push
an update and still get the old build, bust the cache with a query string:

```lua
local NC = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/Naru571/NC/refs/heads/main/NC?t=" .. os.time()
))()
```

### Option B — load from disk

If the file is already on the executor's filesystem:

```lua
local NC = loadfile("NC.lua")()
```

### Option C — hardcoded source

For a script you intend to ship as a single file, paste the contents of `NC`
in place of the `loadstring` call. The library returns its table at the very
bottom of the file, so the same `local NC = ...()` pattern works.

### A loader that tells you what went wrong

A bare `loadstring` call fails silently or opaquely. This version explains the
failure instead:

```lua
local NC_SOURCE = "https://raw.githubusercontent.com/Naru571/NC/refs/heads/main/NC"

local function loadNC()
    local body, err = game:HttpGet(NC_SOURCE)
    if not body then
        error("[NC] could not download the library: " .. tostring(err), 0)
    end

    local head = body:sub(1, 300)

    -- raw.githubusercontent returns PLAIN TEXT for a missing file -- literally
    -- "404: Not Found" -- not an HTML error page. That text used to reach
    -- loadstring, which reported "expected identifier when parsing expression,
    -- got '404'" -- which reads like a bug in the library, not a bad URL.
    if head:match("^%s*404%s*:") or head:match("^%s*Not%s+Found") then
        error("[NC] the URL returned 404 - there is no file at that path.\n"
            .. "  Check the repo is public and the file name matches exactly:\n"
            .. "  " .. NC_SOURCE, 0)
    end

    if head:match("^%s*[Rr]ate%s+limit") or head:match("^%s*[Tt]oo%s+[Mm]any") then
        error("[NC] the host is rate-limiting the download. Wait a minute and "
            .. "run the script again.", 0)
    end

    -- Some hosts answer a bad path with an HTML error page instead.
    if head:match("^%s*<!DOCTYPE") or head:match("^%s*<html") or body:find("<body") then
        error("[NC] the URL returned a web page, not Lua.\n"
            .. "  Check the repo is public and the path is correct:\n"
            .. "  " .. NC_SOURCE, 0)
    end

    -- Last line of defence: if the body has nothing Lua-shaped in it, say so
    -- here rather than letting loadstring fail on line 1 with a message nobody
    -- can act on.
    if not body:find("local", 1, true) and not body:find("return", 1, true) then
        error("[NC] the download does not look like Lua source.\n"
            .. "  First 80 characters:\n  " .. head:sub(1, 80), 0)
    end

    local chunk, compileErr = loadstring(body, "@NC")
    if not chunk then
        error("[NC] the downloaded file has a syntax error: " .. tostring(compileErr), 0)
    end

    local ok, result = pcall(chunk)
    if not ok then
        error("[NC] the library threw while loading: " .. tostring(result), 0)
    end
    if type(result) ~= "table" then
        error("[NC] expected a library table, got " .. type(result), 0)
    end

    return result
end

local NC = loadNC()
```

---

## Window

### `NC:Init(config)`

| Key              | Type                     | Default  | Description                                                          |
| ---------------- | ------------------------ | -------- | -------------------------------------------------------------------- |
| `name`           | string                   | `"NC"`   | Window title. Aliases: `Name`, `title`, `Title`.                     |
| `subtitle`       | string                   | —        | Small text beside the title. Aliases: `Subtitle`, `LoadingSubtitle`. |
| `accent`         | Color3                   | `A855F7` | Starting accent. Derives the whole palette.                          |
| `build`          | function                 | —        | `function(window)` — your interface. Alias: `tabs`.                  |
| `tabs`           | table                    | —        | Declarative fallback: a list of tab prop tables.                     |
| `ready`          | function                 | —        | `function(window)` — runs after the window opens.                    |
| `autoSave`       | boolean                  | `false`  | Persist changes as they happen.                                      |
| `autoLoad`       | boolean                  | `true`   | Restore the saved config on boot.                                    |
| `autoLoadConfig` | string                   | —        | Which config to restore.                                             |
| `toggleKey`      | KeyCode \| string \| false | `N`    | Show/hide hotkey. `false` disables it.                               |
| `notifyOnLoad`   | boolean                  | `true`   | Toast confirming the interface loaded.                               |
| `gradientMode`   | boolean                  | `true`   | Gradient vs. flat surfaces.                                          |
| `transparency`   | number                   | `0`      | Surface transparency, `0`–`0.55`.                                    |
| `scale`          | number                   | `1`      | UI scale, `0.65`–`1.35`.                                             |
| `corner`         | number                   | `12`     | Corner radius, `0`–`24`.                                             |
| `glow`           | number                   | `1`      | Glow intensity, `0`–`1`.                                             |
| `animationSpeed` | number                   | `1`      | Duration multiplier, `0.5`–`2`.                                      |
| `pillDraggable`  | boolean                  | `false`  | Allow dragging the floating pill.                                    |
| `trafficLights`  | boolean                  | `true`   | macOS-style dots in the header.                                      |
| `allowRerun`     | boolean                  | `false`  | Skip the "already initialised" guard.                                |
| `guiName`        | string                   | `"NC"`   | `ScreenGui` instance name.                                           |
| `displayOrder`   | number                   | `999`    | `ScreenGui.DisplayOrder`.                                            |
| `protect`        | boolean                  | `false`  | Call `syn.protect_gui` if available.                                 |

The appearance keys above are the same settings exposed in the Settings panel, so
passing one is equivalent to the user having set it.

### Window methods

```lua
window:CreateTab({ name = "Main", icon = "bolt" })
window:CreateTab("Main", "bolt")     -- Rayfield positional form

window:buildSettings()               -- build/rebuild the settings panel
window:recolour()                    -- re-apply the theme to every element
window:open()                        -- animate in
window:close()                       -- animate out (runs the confirmation dialog)
window:minimise()                    -- fade out and show the floating pill
window:restore()                     -- bring it back
window:toggleMaximise()              -- green-dot zoom: grow / restore the window
```

There is no `window:toggle()` — the hotkey dispatches to `open`, `close` or
`restore` depending on the current state, so call whichever one you mean.

### Object model

```
NC
 └─ Window           draggable, header, nav rail, content, profile, controls
     └─ Tab          nav pill + scrolling page
         └─ Section  titled container
             └─ elements, and Groups (nested collapsible blocks)
```

---

## Tabs

A tab is a nav pill on the left rail plus a scrolling page. The first tab you
create becomes the landing tab.

```lua
local main    = window:CreateTab({ name = "Main",    icon = "bolt" })
local visuals = window:CreateTab({ name = "Visuals", icon = "eye" })
local player  = window:CreateTab({ name = "Player",  icon = "person" })
```

| Prop   | Type   | Description                                      |
| ------ | ------ | ------------------------------------------------ |
| `name` | string | Label on the rail.                               |
| `icon` | string | Glyph name (see [Icons](#icons-and-animations)). |

Tab methods:

```lua
tab:CreateSection({ name = "Movement" })
tab:select()                    -- switch to this tab programmatically
```

**Elements can be created directly on a tab.** They land in the tab's most recent
section — useful for porting scripts that do not use sections.

```lua
local tab = window:CreateTab({ name = "Main" })
tab:CreateSection({ name = "Movement" })
tab:CreateToggle({ name = "Speed", flag = "Speed" })   -- goes into "Movement"
```

---

## Sections and groups

A section is a titled container that holds elements. `description` renders under
the title; `icon` appears beside it.

```lua
local section = tab:CreateSection({
    name = "Movement",
    description = "Walk speed and jump power",
    icon = "bolt",
})
```

A **group** is a collapsible block nested inside a section. It exposes the same
constructors as a section, and can carry its own leading icon.

```lua
local group = section:CreateGroup({ name = "Box style", icon = "palette", collapsed = false })

group:CreateDropdown({ name = "Corner style", options = { "Corners", "Full box" } })
group:CreateSlider({ name = "Thickness", min = 1, max = 4, value = 1 })
```

| Prop        | Type    | Default | Description                            |
| ----------- | ------- | ------- | -------------------------------------- |
| `name`      | string  | —       | Group title. Aliases: `Name`, `title`. |
| `icon`      | string  | —       | Glyph shown before the title.          |
| `collapsed` | boolean | `false` | Start collapsed.                       |

```lua
group:toggle()      -- collapse or expand, with animation
group.collapsed     -- read the current state
```

---

## Elements

Every constructor below is also available on a `Tab` — elements land in the tab's
most recent section.

On a `Group`, everything is available **except** `CreateMultiDropdown`,
`CreateDropdownMulti` and `CreateGroup`: groups cannot nest, and the
multi-selects are section-level.

All of them accept these three shared props, supplied by the row builder:

| Prop          | Type   | Description                             |
| ------------- | ------ | --------------------------------------- |
| `name`        | string | Primary label. Aliases: `Name`, `Text`. |
| `description` | string | Secondary line under the label.         |
| `icon`        | string | Glyph shown in the row's leading box.   |

---

### Toggle

A switch. `CreateSwitch` is an alias.

```lua
section:CreateToggle({
    name = "Speed enabled",
    description = "Enables the override",
    icon = "bolt",
    flag = "SpeedEnabled",
    value = false,
    callback = function(state)
        print("enabled:", state)
    end,
})
```

| Prop       | Type     | Default | Description                                      |
| ---------- | -------- | ------- | ------------------------------------------------ |
| `value`    | boolean  | `false` | Initial state. Aliases: `Value`, `CurrentValue`. |
| `flag`     | string   | —       | Persistence key. Alias: `Flag`.                  |
| `callback` | function | —       | `function(state)` — fires on change.             |

---

### Button

A gradient plate with a press animation.

```lua
section:CreateButton({
    name = "Reset character",
    description = "Respawns you instantly",
    icon = "refresh",
    callback = function()
        local char = game.Players.LocalPlayer.Character
        if char then char:BreakJoints() end
    end,
})
```

| Prop       | Type     | Description                                       |
| ---------- | -------- | ------------------------------------------------- |
| `callback` | function | `function()` — fires on click. Alias: `Callback`. |
| `text`     | string   | Label. Alias: `Text`, `name`.                     |
| `width`    | number   | Optional fixed width in pixels.                   |

---

### Slider

Draggable track with a knob and a live readout. Values snap to `step`.

```lua
section:CreateSlider({
    name = "Walk speed",
    flag = "WalkSpeed",
    min = 16, max = 250, step = 1, value = 16,
    callback = function(value) print(value) end,
})
```

Percentage mode shows `0%`–`100%` instead of the raw number — handy for smoothing
factors:

```lua
section:CreateSlider({
    name = "Smoothing",
    description = "Higher is smoother but slower",
    flag = "AimSmoothing",
    min = 0, max = 1, step = 0.05, value = 0.35,
    percentage = true,
    callback = function(value) end,
})
```

| Prop         | Type     | Default | Description                                                 |
| ------------ | -------- | ------- | ----------------------------------------------------------- |
| `min`        | number   | `0`     | Minimum. Aliases: `Min`, `Minimum`.                         |
| `max`        | number   | `100`   | Maximum. Aliases: `Max`, `Maximum`.                         |
| `step`       | number   | `1`     | Increment. Alias: `Step`, `Increment`.                      |
| `value`      | number   | `min`   | Initial value. Aliases: `Value`, `Default`, `CurrentValue`. |
| `suffix`     | string   | —       | Appended to the readout, e.g. `"°"`. Alias: `Suffix`.       |
| `percentage` | boolean  | `false` | Render the readout as a percentage.                         |
| `callback`   | function | —       | `function(value)`.                                          |

`Range = { min, max }` is also understood for Rayfield compatibility.

---

### Dropdown

Single-select by default. Search auto-enables when there are more than 7 options —
multi-selects default to off, so pass `searchable = true` to force it on.

```lua
section:CreateDropdown({
    name = "Target priority",
    description = "Closest, lowest health, or first found",
    flag = "TargetPriority",
    options = {
        { label = "Closest",       value = "closest" },
        { label = "Lowest health", value = "lowest"  },
        { label = "First found",   value = "first"   },
    },
    default = "closest",
    callback = function(value) print(value) end,
})
```

Options may be plain strings — `options = { "Corners", "Full box", "Filled" }` —
in which case the label and the value are the same.

| Prop          | Type     | Default       | Description                                                        |
| ------------- | -------- | ------------- | ------------------------------------------------------------------ |
| `options`     | table    | `{}`          | Strings, or `{ label, value }` tables. Alias: `Options`.           |
| `default`     | any      | —             | Initially selected value. Aliases: `Default`, `CurrentOption`.     |
| `multi`       | boolean  | `false`       | Allow multiple selections. Aliases: `Multiple`, `MultipleOptions`. |
| `searchable`  | boolean  | auto          | Force the search field on or off.                                  |
| `placeholder` | string   | `"Select..."` | Text when nothing is chosen.                                       |
| `clearable`   | boolean  | `true`        | Offer a *Clear selection* row.                                     |
| `width`       | number   | —             | Optional fixed width.                                              |
| `callback`    | function | —             | `function(value)` — a table when `multi` is set.                   |

#### Multi-select

```lua
section:CreateMultiDropdown({
    name = "Select players",
    flag = "SelectedPlayers",
    options = { "Player1", "Player2", "Player3", "Player4" },
    default = {},
    callback = function(selected)
        for _, name in ipairs(selected) do print(name) end
    end,
})
```

`CreateDropdownMulti` is an alias. Selecting does **not** close the list, several
options stay highlighted with checkmarks, clicking again deselects, and a
*Clear selection* row is provided.

#### Compact positional form

```lua
section:Dropdown("Select players", { "Player1", "Player2" }, {
    Multi      = true,
    Default    = {},
    Flag       = "SelectedPlayers",
    Callback   = function(selected) end,
})
```

---

### Input

A text field with a clear button and a focus glow.

```lua
section:CreateInput({
    name = "Custom target",
    description = "Leave blank to auto-select",
    placeholder = "Username",
    flag = "CustomTarget",
    clearable = true,
    callback = function(text) print(text) end,
})
```

| Prop          | Type     | Default | Description                                                |
| ------------- | -------- | ------- | ---------------------------------------------------------- |
| `placeholder` | string   | —       | Ghost text. Aliases: `Placeholder`, `PlaceholderText`.     |
| `value`       | string   | `""`    | Initial text. Aliases: `Value`, `Default`, `CurrentValue`. |
| `numeric`     | boolean  | `false` | Restrict input to digits and `.`/`-`.                      |
| `multiline`   | boolean  | `false` | Allow newlines.                                            |
| `clearable`   | boolean  | `true`  | Show the clear button.                                     |
| `live`        | boolean  | `false` | Fire `callback` on every keystroke, not just Enter/blur.   |
| `onEnter`     | function | —       | `function(text)` — fires when Enter is pressed.            |
| `width`       | number   | —       | Optional fixed width.                                      |
| `callback`    | function | —       | `function(text)`.                                          |

---

### Paragraph

A block of wrapped, auto-height text. Writable at runtime.

```lua
local note = section:CreateParagraph({
    name = "Status",
    text = "Waiting for the game to load...",
})

note:Set("Ready.")
```

| Prop       | Type   | Description                                    |
| ---------- | ------ | ---------------------------------------------- |
| `text`     | string | Body text. Aliases: `Text`, `Content`, `name`. |
| `color`    | Color3 | Override the theme text colour.                |
| `textSize` | number | Override the font size.                        |

---

### Label

A non-interactive heading.

```lua
section:CreateLabel({ text = "Advanced", bold = true })
```

| Prop       | Type    | Description                      |
| ---------- | ------- | -------------------------------- |
| `text`     | string  | Label text.                      |
| `bold`     | boolean | Use a heavier font weight.       |
| `align`    | string  | `"left"`, `"center"`, `"right"`. |
| `color`    | Color3  | Override the theme colour.       |
| `textSize` | number | Override the font size.          |

---

### Divider

A thin rule with an optional caption.

```lua
section:CreateDivider("Danger zone")
section:CreateDivider()            -- bare rule
```

---

## Value handles

Every interactive element returns the same handle contract.

```lua
local handle = section:CreateToggle({ name = "Speed", flag = "Speed" })

handle.value                  -- read the current value directly
handle:Set(true)              -- set it; fires the callback
handle:Set(true, true)        -- set it; suppresses the callback
handle:Get()                  -- explicit read
handle:Lock("Unavailable")    -- block input and callback, show a reason
handle:Unlock()
handle:IsLocked()
```

| Member                      | Description                                                |
| --------------------------- | ---------------------------------------------------------- |
| `.value`                    | The current value.                                         |
| `:Set(value, skipCallback)` | Set the value. `skipCallback` suppresses the callback.     |
| `:Get()`                    | Read the value.                                            |
| `:Lock(reason)`             | Block interaction and show `reason` in place of the label. |
| `:Unlock()`                 | Remove the lock.                                           |
| `:IsLocked()`               | Whether it is currently locked.                            |
| `.flag`                     | The persistence key.                                       |

Locking is how a script disables a control that does not apply right now — for
example locking an aim slider while silent aim is off.

```lua
local slider = section:CreateSlider({ name = "Field of view", flag = "FOV", min = 10, max = 360 })

toggleHandle:Set(false)                     -- via the API
slider:Lock("Enable Silent aim first")
-- later
slider:Unlock()
```

**`flag` is the persistence key.** Anything with a flag is saved and restored by
the config system.

---

## Notifications

```lua
NC:Notify({ Title = "Success", Content = "Configuration saved", Duration = 3 })

NC:Notify({
    Title    = "Failed",
    Content  = "Could not reach the server",
    Type     = "error",
    Duration = 4,
})
```

| Prop       | Type   | Default  | Description                            |
| ---------- | ------ | -------- | -------------------------------------- |
| `Title`    | string | —        | Bold first line.                       |
| `Content`  | string | —        | Body text.                             |
| `Type`     | string | `"info"` | `success`, `info`, `warning`, `error`. |
| `Duration` | number | `3`      | Seconds on screen.                     |

Each type has its own icon, accent colour and draining progress bar. Click to
dismiss early. `NC:Toast(props)` is an alias.

**Stack cap.** At most `Theme.NotifyCap` cards sit on screen at once (default
`4`). Older ones park off-screen and slide in as newer ones expire. Set the cap to
`0` to stack without limit — exposed in Settings under **Behaviour → Limit
notifications**.

---

## Appearance

### Setters

```lua
NC:setAccent(Color3.fromHex("6366F1"))  -- recolours the entire interface
NC:setGradientMode(true)                -- gradients vs. flat colour
NC:setTransparency(0.15)                -- 0 – 0.55
NC:setScale(1.1)                        -- 0.65 – 1.35
NC:setCorner(12)                        -- 0 – 24
NC:setGlow(0.6)                         -- 0 – 1
NC:setAnimationSpeed(1.2)               -- 0.5 – 2
NC:setIconMode(false)                   -- icon-only rows in the nav rail
NC:setShowPillMode(true)                -- "Show NC" text on the floating pill
NC:setProfileVisible(true)              -- profile block in the window
NC:setTrafficLights(true)               -- macOS-style dots in the header
NC:setWindowIcon("rbxassetid://1234567")
```

Every setter takes a second `silent` argument that suppresses auto-save.

```lua
NC:setAccent(Color3.fromHex("10B981"), true)   -- do not write a config
```

**The logo follows the accent.** The drawn NC mark is recoloured with the theme —
the C and its beads take the accent, the N stays white for contrast. This covers
the header icon and the loading-screen mark, and it updates live while you drag
the colour wheel.

**Theme changes are smooth.** Repaints run in small per-frame slices through
`RunService` instead of one giant pass, so swapping accents mid-game no longer
hitches the framerate, even on large interfaces.

### Reacting to accent changes

```lua
NC:onAccent(function(color)
    print("accent is now", color)
end)
```

### Theme presets

Six named palettes, each an accent plus the gradient style that suits it. In the
Settings panel the chips wrap onto two rows, and whichever preset matches your
live accent shows a selection ring.

```lua
for _, preset in ipairs(NC:listThemePresets()) do
    print(preset.index, preset.name)   -- 1 Midnight, 2 Nord, ...
end

NC:applyThemePreset(3)                 -- Ember
```

| # | Name      | Accent   | Gradient |
| - | --------- | -------- | -------- |
| 1 | Midnight  | `A855F7` | on       |
| 2 | Nord      | `88C0D0` | off      |
| 3 | Ember     | `F97316` | on       |
| 4 | Solarized | `268BD2` | off      |
| 5 | Rosewood  | `E11D48` | on       |
| 6 | Verdant   | `22C55E` | off      |

Presets are **colour only** — they never touch glow or corner roundness, so
picking one cannot change the shape of the interface.

### The theme table

One table drives everything; no component hardcodes a colour. It is exposed as
`NC.Theme` for feature scripts.

```lua
Theme = {
    Background, BackgroundSecondary, BackgroundTertiary, Surface,
    Accent, AccentSecondary, AccentDeep, AccentSoft,
    Gradient, GradientAngle, GradientMode,
    Text, SubText, MutedText,
    Border, BorderBright, HeaderStroke,
    Success, Info, Warning, Error, ErrorStroke,
    Corner, CornerLarge, PillCorner,
    UIScale, Transparency, GlowIntensity, AnimationSpeed,
    NotifyCap, PerGameConfigs, PillDraggable,
}
```

`NC:setAccent` derives the full palette — secondary, deep, soft, border and
background — from a single HSV hue, so one colour picker recolours everything
without a restart.

The other internals are exposed too: `NC.State`, `NC.Glyph`, `NC.Anim`.

### Window icon

Accepts a bare numeric id, `rbxassetid://`, `rbxasset://`, an `http(s)://` URL, or
a `data:` URI. If the asset cannot be resolved it falls back to the drawn NC mark
rather than leaving a blank square. The same field is in the Settings panel under
**Window icon**.

---

## Configs

Save, load and delete named configurations. All values with a `flag` are included,
plus the theme colour, gradient mode, UI scale, transparency, corner radius, glow,
animation speed, window icon, traffic lights and profile visibility.

```lua
NC:saveConfig()                    -- save under the active name
NC:saveConfig("PvP")               -- save under a given name
NC:loadConfig("PvP")
NC:deleteConfig("PvP")
NC:listConfigs()                   -- { "PvP", "Farming", ... }
NC:queueAutoSave()                 -- debounced; only runs when auto-save is on
```

Auto-save is off unless you pass `autoSave = true` to `NC:Init` or the user
enables it in Settings.

### Export and import

Configs serialise to JSON, so they can be moved between machines or shared.

```lua
local json = NC:exportConfig("PvP")     -- returns a JSON string
NC:importConfig(json, "PvP (copy)")     -- returns true on success
NC:copyConfigToClipboard("PvP")         -- straight to setclipboard
```

### Per-game namespacing

By default configs are stored under a folder namespaced by `game.PlaceId`, so a
config for one game cannot collide with another. Turn it off to share one folder
across every game:

```lua
Theme.PerGameConfigs = false
```

This is exposed in Settings under **Behaviour → Per-game configs**.

Storage uses the executor filesystem (`writefile` / `readfile` / `isfile` /
`delfile` / `makefolder`), wrapped in `pcall` so a missing filesystem degrades to
an in-memory config rather than erroring.

---

## Player info

A button sits beside the profile block. Pressing it opens a card showing the local
player's details, with a copy button that puts the whole thing on the clipboard.
The card is fully themed — change the accent in Settings and the card, its
border, the avatar ring and the Copy button all follow.

```lua
local rows = NC:collectPlayerInfo()
for _, row in ipairs(rows) do
    print(row[1], row[2], row[3])   -- key, label, value
end

print(NC:playerInfoText())          -- plain-text form, used by the copy button
```

Rows: username, user ID, account age, time in the current place, HWID, executor,
platform, Premium status, Place ID and the **full** server Job ID. Long values
are only shortened visually on the row — the clipboard always receives the whole
string, so a copied server ID pastes complete.

**Privacy.** The eye toggle on the profile block hides your avatar and name *and*
hides the info button, closing the card if it was open — nothing personal stays
on screen. The choice is saved with your config and restored on load.

HWID resolution tries the usual executor globals and reports `Unavailable` if none
respond. It never throws.

---

## Show and hide keybind

The interface is toggled with **N** by default.

```lua
NC:Init({
    toggleKey = "N",          -- a KeyCode name
    -- toggleKey = Enum.KeyCode.RightShift,
    -- toggleKey = false,     -- disable the hotkey entirely
})
```

`NC_example.lua` sets this at the top of the file, so it is the natural place to
edit it.

```lua
NC:setToggleKey("K")            -- change it at runtime
NC:keyLabel(Enum.KeyCode.K)     -- "K" — display form
NC:captureToggleKey(function(key)
    print("new key:", NC:keyLabel(key))
end)
```

`captureToggleKey` is what the Settings panel uses: it listens for the next key
press and rebinds. In the UI it is **Settings → Show / hide key** — click the
button, then press a key. Escape cancels.

---

## Traffic lights

The window has macOS-style dots in the top-left of the header.

| Dot | Colour | Action |
| --- | ------ | ------ |
| Red | `FF5F57` | Closes the interface, through the same confirmation dialog as the X. |
| Yellow | `FEBC2E` | Minimises to the floating **Show NC** pill. |
| Green | `28C840` | Zooms the window between its normal size and a viewport-fitting one. |

How they behave:

- **Hover** springs a dot larger and brightens it; pressing squishes it and it
  bounces back. There are deliberately *no* ×/−/+ marks inside the dots — they
  are unreadable at 12px.
- **The dots explain themselves when you switch them on or off** in Settings:
  a notification spells out *"Red closes · Yellow minimises · Green zooms."*
  (No toasts on hover — those got noisy.)
- The dots **pop in with a staggered scale animation** when they appear.
- The zoomed size is remembered across minimise/close, and re-fits itself if
  your viewport resizes while zoomed.

While the dots are on, the top-right control bar parks its **expand**,
**minimise** and **close** plates — the dots already do those jobs — leaving a
clean search + settings bar. Turning the dots off brings all five plates back
(search, settings, expand, minimise, close), so you can zoom without the dots.
The logo and title slide to make room either way.

```lua
NC:Init({ trafficLights = false })      -- start with them off
NC:setTrafficLights(false)              -- or change it at runtime
```

In the panel it is the **"Mac-style traffic lights"** row, right below the
appearance sliders in Settings. The setting is saved with your config.

---

## Minimise and the floating pill

Minimising fades and shrinks the window out completely *before* the floating
`Show NC` pill appears, so the two never overlap. The window is hidden rather than
destroyed, so restoring is instant and state is preserved.

The pill has a pulsing halo, a drifting sheen, a slowly rotating sparkle and a
gentle float, brightens on hover, and is clamped so it can never be lost
offscreen. Clicking it restores the window.

**Dragging is off by default.** The pill is also the restore button, so a slightly
imprecise click used to move it — and once moved it stayed put, which read as the
pill teleporting on its own. Turn dragging on if you want it:

```lua
NC:Init({ pillDraggable = true })
-- or
NC:setPillDraggable(true)
```

Exposed in Settings under **Behaviour → Move the Show NC pill**.

---

## Plugins

A plugin adds its own controls to the Settings panel without editing the library.
Register a spec with a `build(host)` callback; `host` is a group inside the
settings panel, so it exposes the same constructors a section does.

```lua
NC:RegisterPlugin({
    name        = "Auto Farm",
    description = "Farming options",
    build = function(host)
        host:CreateToggle({
            name = "Enabled",
            flag = "AutoFarm",
            callback = function(state) end,
        })
        host:CreateSlider({
            name = "Radius",
            flag = "FarmRadius",
            min = 1, max = 50, value = 10,
        })
    end,
})
```

The group is created collapsed, so a plugin does not push the built-in settings
off the panel the moment it registers. Flags registered through a plugin are saved
and restored exactly like built-in ones.

```lua
NC:listPlugins()                 -- { { name, description, mounted }, ... }
NC:UnregisterPlugin("Auto Farm") -- destroys the group and its controls
NC:mountAllPlugins()             -- re-mount everything (called on settings build)
```

Registering before the settings panel exists is fine — the plugin mounts when the
panel is built.

---

## Rayfield Gen2 compatibility

Scripts written against Rayfield Gen2 run largely unchanged. `Name` / `Title`,
`Flag`, `Callback`, `CurrentValue`, `Range`, `Increment`, `Options`,
`CurrentOption`, `MultipleOptions`, `PlaceholderText`, `Content` and `Image` are
all understood, and elements may be created straight on a `Tab` — landing in its
most recent section — rather than on a `Section`.

```lua
local Window = NC:CreateWindow({
    Name = "NC",
    LoadingSubtitle = "Interface",
    ConfigurationSaving = { Enabled = true, FileName = "main" },
})

local Tab = Window:CreateTab("Main", "grid")   -- (Name, Icon)
Tab:CreateSection("Movement")

Tab:CreateToggle({
    Name = "Speed", CurrentValue = false, Flag = "Speed",
    Callback = print,
})

Tab:CreateSlider({
    Name = "Walk speed", Range = { 16, 250 }, Increment = 1,
    CurrentValue = 16, Flag = "WS", Callback = print,
})

NC:Notify({ Title = "Hi", Content = "Loaded", Duration = 3 })
```

| Rayfield key      | Maps to       |
| ----------------- | ------------- |
| `Name`, `Title`   | `name`        |
| `Description`     | `description` |
| `Icon`, `Image`   | `icon`        |
| `Flag`            | `flag`        |
| `Callback`        | `callback`    |
| `CurrentValue`    | `value`       |
| `Range = {a, b}`  | `min`, `max`  |
| `Increment`       | `step`        |
| `Options`         | `options`     |
| `CurrentOption`   | `default`     |
| `MultipleOptions` | `multi`       |
| `PlaceholderText` | `placeholder` |
| `Content`, `Text` | `text`        |

`NC:CreateWindow` runs the first loading stage inline, so the window already exists
when it returns and tabs can be added immediately. `NC.LoadConfiguration(name)` is
provided as the Rayfield-compatible loader.

---

## Icons and animations

### Icons

Every glyph is drawn from primitives — rotated rounded bars, stroked rings and
bead dots — in `Glyph.data`. **No Roblox asset ids are used**, so icons can never
fail to load, cannot be fingerprinted by anti-cheat, and work in secure mode.

Pass a glyph name as `icon` on a tab, section, group or element:

```lua
window:CreateTab({ name = "Visuals", icon = "eye" })
section:CreateToggle({ name = "ESP", icon = "sparkle" })
```

The full set — 35 glyphs, all drawn from primitives:

|           |            |             |           |
| --------- | ---------- | ----------- | --------- |
| `search`  | `settings` | `minimise`  | `close`   |
| `clear`   | `chevron`  | `check`     | `spinner` |
| `hash`    | `plus`     | `minus`     | `expand`  |
| `eye`     | `home`     | `bolt`      | `crosshair` |
| `visuals` | `person`   | `badge`     | `grid`    |
| `star`    | `palette`  | `save`      | `folder`  |
| `trash`   | `refresh`  | `sparkle`   | `info`    |
| `warning` | `error`    | `success`   | `image`   |
| `power`   | `sun`      | `moon`      |           |

An unknown name resolves to `grid` rather than erroring, so a typo will not break
the row.

### Animations

`Anim` wraps `TweenService` in one place, exposed as `NC.Anim`.

```lua
Anim.Ease  = { out, soft, gentle, spring, elastic, linear }
Anim.Dir   = { out, inward, both }
Anim.Speed = { instant, fast, normal, slow, reveal }

Anim.tween(instance, duration, properties, style, direction)
Anim.stagger(list, fn, step)
```

Durations are divided by `Theme.AnimationSpeed`, so one slider retimes the whole
interface. Everything animates: loading, window, tabs, sections, dropdowns,
toggles, sliders, search, settings, profile, notifications, minimise, restore,
the traffic lights and the floating pill. Nothing snaps.

---

## Executor support

NC is deliberately close to executor-agnostic. The only non-Roblox things it
touches are:

| Global                                                         | Use                                           | If missing                            |
| -------------------------------------------------------------- | --------------------------------------------- | ------------------------------------- |
| `gethui()`                                                     | Parent the GUI somewhere safer than `CoreGui` | Falls back to `PlayerGui` / `CoreGui` |
| `syn.protect_gui`                                              | Optional GUI protection                       | Skipped silently                      |
| `writefile` / `readfile` / `isfile` / `delfile` / `makefolder` | Config persistence                            | Config becomes in-memory              |
| `setclipboard`                                                 | Config copy button                            | Button reports failure                |

`loadstring` and `game:HttpGet` are used only by the loader example, not by the
library. Everything else — Frames, `TweenService`, `UICorner` / `UIStroke` /
`UIGradient`, `Draggable`, `:SetAttribute` — is core Roblox and works anywhere.

Every executor-specific call is wrapped in `pcall`, so an unsupported global
degrades a feature rather than breaking the interface.

---

## Verifying a build

`luacheck.py` parses the file with a real Lua VM (via `lupa`), because Python
AST-based Lua parsers give false positives:

```
python luacheck.py NC.lua        # -> OK NC.lua
```

It also warns when a Lua keyword is used as a table key or dot-accessed — the bug
class that `TweenService`'s `Enum.EasingDirection.In` invites (`Anim.Dir.in` is
illegal Lua; the key is `inward`).

Syntax checking alone does not prove a Roblox script runs. A file can pass the
parser and still throw on load from an invalid property name, a non-creatable
class, or a `local` captured before it was assigned. Test in a live environment.

---

## Troubleshooting

**Nothing happens when I run the script.**
Check the executor console for a `[NC]` warning. If stage 1 of the loading sequence
failed, later stages log `attempt to index nil` errors that bury the real cause —
the *first* warning is the one that matters.

**`[NC] syntax error: NC:1: expected identifier when parsing expression, got '404'`**
The URL returned `404: Not Found` — the file is not at that path. GitHub serves
that as **plain text**, so it slips past an HTML check and reaches `loadstring`,
which then blames line 1 of the library. Nothing is wrong with the library.

Check that the repo is public and the file name matches the URL exactly, including
capitalisation. Note the published file is named `NC` with no extension — a URL
ending `/NC.lua` will 404 against a file called `NC`. The hardened loader in
[Installation options](#installation-options) reports this directly.

**`loadstring` returns some other syntax error.**
The URL is serving something that is not Lua. Open it in a browser and confirm it
returns Lua source.

**An update I pushed is not showing up.**
The raw GitHub URL is cached by both the executor and GitHub's CDN. Append
`?t=` .. `os.time()`.

**My changes are not being saved.**
Auto-save is off by default. Pass `autoSave = true` to `NC:Init`, or enable it in
Settings. The executor must also support the filesystem API.

**I press the hotkey and nothing toggles.**
Another script may be binding the same key. Change it with `toggleKey` in
`NC:Init`, or set `toggleKey = false` and call `window:close()` / `window:open()`
yourself.

**A control is disabled and I cannot interact with it.**
It is locked. `handle:Unlock()` releases it.

**Everything renders as black or grey.**
A `UIGradient` multiplies with the frame's `BackgroundColor3`. If you are theming
by hand, set the base colour to white where the gradient supplies the colour.

**The window is lost offscreen.**
`NC:Init` centres it on build. To reset a dragged window, reload the interface —
or use the floating pill, which is clamped to the viewport by design.

**Changing the accent used to lag my game.**
Fixed in Gen 1.1.0 — theme repaints are now spread across frames. If you are on an
older build, update.

**Can I use this on mobile?**
Yes. The layout is touch-friendly. The hotkey is PC-only; on mobile use the
minimise button and the floating pill instead.
