# Compass Heading Display — Developer Guide

**Version:** 1.0.0.0
**Author:** RocklandUSA Gaming
**Support:** [FSCORE.GG](https://fscore.gg)

---

## Overview

Compass Heading Display provides a GTA simulation-style horizontal compass bar for Farming Simulator 25. Beyond the visual compass, it exposes a **public API with 60+ functions** that any mod can use to display custom markers, query navigation data, and receive events — all with zero dependencies.

**API endpoint:** `g_currentMission.compassHeading`
**Registry endpoint:** `g_currentMission.compassHeadingRegistry`

---

## Quick Start — Zero-Dependency Integration

The golden rule: **never declare a dependency on Compass Heading.** Instead, check if the API exists at runtime. If it does, use it. If not, your mod works fine without it.

```lua
-- Safe pattern: works whether Compass Heading is installed or not
local function addMyMarker()
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch == nil then return end  -- Compass Heading not installed, silently skip

    ch.addMarker("my_mod_marker", {
        x = 500,
        z = -300,
        label = "Shop",
        category = "shop",
    })
end
```

---

## Table of Contents

1. [Adding Markers](#1-adding-markers)
2. [Marker Styles](#2-marker-styles)
3. [Categories](#3-categories)
4. [Animations](#4-animations)
5. [Temporary Markers & TTL](#5-temporary-markers--ttl)
6. [Provider System](#6-provider-system-dynamic-markers)
7. [Events & Callbacks](#7-events--callbacks)
8. [Navigation Utilities](#8-navigation-utilities)
9. [Settings Control](#9-settings-control)
10. [Distance & Proximity](#10-distance--proximity)
11. [Complete API Reference](#11-complete-api-reference)
12. [Real-World Examples](#12-real-world-examples)

---

## 1. Adding Markers

### Add a static marker
```lua
local ch = g_currentMission.compassHeading

ch.addMarker("gas_station_1", {
    x = 1200,            -- world X position
    z = -850,            -- world Z position
    label = "Gas",       -- text shown on compass (max 12 chars)
    color = {r=1, g=0.5, b=0, a=1},  -- orange (optional, defaults to category color)
    style = "diamond",   -- marker glyph style (optional, defaults to "tick")
    scale = 1.2,         -- size multiplier (optional, defaults to 1.0)
    priority = 5,        -- higher = drawn on top (optional, defaults to 0)
    category = "shop",   -- category name (optional, defaults to "default")
})
```

### Update an existing marker
```lua
ch.updateMarker("gas_station_1", {
    label = "Fuel",
    color = {r=0, g=1, b=0, a=1},
})
```

### Move a marker
```lua
ch.setMarkerPosition("gas_station_1", 1500, -900)
```

### Remove a marker
```lua
ch.removeMarker("gas_station_1")
```

### Check if a marker exists
```lua
if ch.hasMarker("gas_station_1") then
    local info = ch.getMarker("gas_station_1")
    print(info.label, info.x, info.z)
end
```

### Clear all markers (or by category)
```lua
ch.clearMarkers()              -- remove all standalone markers
ch.clearMarkers("emergency")   -- remove only emergency markers
```

---

## 2. Marker Styles

Each marker renders a text glyph on the compass bar. Available styles:

| Style       | Glyph | Best For |
|-------------|-------|----------|
| `tick`      | `\|`  | Players, generic points |
| `diamond`   | `◆`   | Objectives, POIs |
| `triangle`  | `V`   | Waypoints, drops |
| `circle`    | `O`   | Areas, zones |
| `square`    | `#`   | Buildings, structures |
| `chevron`   | `>>`  | Directional markers |
| `star`      | `*`   | Special, important |
| `cross`     | `+`   | Medical, aid |
| `dot`       | `.`   | Subtle, background |
| `pipe`      | `\|`  | Separator |
| `dash`      | `-`   | Divider |
| `arrow_up`  | `^`   | Upward indicators |
| `arrow_down`| `v`   | Downward indicators |

```lua
ch.addMarker("hospital", {
    x = 800, z = -400,
    label = "Hospital",
    style = "cross",
    category = "medical",
})

ch.setMarkerStyle("hospital", "star")  -- change style later
```

Get all available styles:
```lua
local styles = ch.getMarkerStyles()  -- returns list of style name strings
```

---

## 3. Categories

Categories provide automatic colour grouping and visibility control. Built-in defaults:

| Category     | Color        | Use Case |
|-------------|-------------|----------|
| `default`    | White       | General markers |
| `emergency`  | Red         | Fire, accidents, alerts |
| `delivery`   | Amber       | Pickups, dropoffs |
| `equipment`  | Light Blue  | Vehicles, tools |
| `farm`       | Green       | Farm locations |
| `activity`   | Purple      | Events, activities |
| `player`     | Cyan        | Player markers |
| `waypoint`   | Orange      | Map waypoints |
| `shop`       | Yellow      | Stores, dealerships |
| `medical`    | White+Red   | Hospitals, aid |
| `police`     | Blue        | Law enforcement |
| `fire`       | Red-Orange  | Fire stations |
| `mechanic`   | Steel       | Repair shops |
| `helper`     | Green       | AI hired workers |
| `job`        | Gold        | Job objectives |

If you add a marker with a category that doesn't exist yet, it is created automatically with the `default` colour.

### Toggle categories on/off
```lua
ch.setCategoryEnabled("emergency", false)   -- hide all emergency markers
ch.setCategoryEnabled("emergency", true)    -- show them again

if ch.isCategoryEnabled("delivery") then ... end
```

### Custom category colours
```lua
ch.setCategoryColor("my_category", {r=1, g=0, b=1, a=1})  -- magenta

local color = ch.getCategoryColor("emergency")  -- {r=..., g=..., b=..., a=...}
```

### List all categories
```lua
local cats = ch.getCategories()
for _, cat in ipairs(cats) do
    print(cat.name, cat.enabled, cat.color.r, cat.color.g, cat.color.b)
end
```

---

## 4. Animations

Markers support four animation modes that can be combined:

### Flash (square-wave visibility toggle)
```lua
ch.addMarker("alert", {
    x = 300, z = -200,
    label = "FIRE",
    category = "emergency",
    flash = true,        -- enable flashing
    flashHz = 3.0,       -- 3 flashes per second (default: 2.0)
})

-- Or toggle flash on an existing marker:
ch.setMarkerFlash("alert", true, 4.0)   -- enable at 4 Hz
ch.setMarkerFlash("alert", false)        -- disable
```

### Pulse (sine-wave alpha oscillation)
```lua
ch.addMarker("beacon", {
    x = 600, z = -100,
    label = "Beacon",
    pulse = true,
    pulseHz = 1.5,       -- oscillation speed (default: 1.0)
    pulseMin = 0.2,      -- minimum alpha (default: 0.3)
    pulseMax = 1.0,      -- maximum alpha (default: 1.0)
})

ch.setMarkerPulse("beacon", true, 2.0, 0.1, 1.0)
```

### Fade-in (alpha ramp on creation)
```lua
ch.addMarker("new_obj", {
    x = 400, z = -500,
    label = "New!",
    fadeIn = true,
    fadeInDur = 1.0,     -- fade duration in seconds (default: 0.5)
})
```

### Urgent (flash + colour shift toward red)
```lua
ch.addMarker("sos", {
    x = 900, z = -700,
    label = "SOS",
    category = "emergency",
    urgent = true,       -- combines flash + red colour shift
})

ch.setMarkerUrgent("sos", true)   -- enable
ch.setMarkerUrgent("sos", false)  -- disable
```

---

## 5. Temporary Markers & TTL

Markers can auto-remove after a set number of seconds:

```lua
-- Auto-removes after 30 seconds
ch.addTemporaryMarker("ping_1", {
    x = 500, z = -300,
    label = "Ping!",
    ttl = 30,            -- seconds until auto-removal
    fadeIn = true,
    category = "activity",
})

-- You can also set TTL on a regular marker:
ch.addMarker("temp", {
    x = 100, z = -100,
    label = "Temp",
    ttl = 60,  -- gone in 60 seconds
})
```

---

## 6. Provider System (Dynamic Markers)

For markers that change every frame (vehicle positions, moving NPCs), use the **provider system** instead of add/update/remove calls. A provider is a function that returns a list of points each frame.

### Register a provider
```lua
local reg = g_currentMission.compassHeadingRegistry
if reg then
    reg:register({
        id = "my_vehicle_tracker",
        label = "Vehicle Tracker",
        category = "equipment",
        getPoints = function()
            local points = {}
            -- Return current positions of all your tracked vehicles
            for _, veh in pairs(myTrackedVehicles) do
                points[#points + 1] = {
                    x = veh.posX,
                    z = veh.posZ,
                    label = veh.name,
                    color = {r=0.5, g=0.8, b=1, a=1},  -- optional per-point override
                }
            end
            return points
        end,
    })
end
```

### Unregister a provider
```lua
local reg = g_currentMission.compassHeadingRegistry
if reg then
    reg:unregister("my_vehicle_tracker")
end
```

### Query providers
```lua
local ch = g_currentMission.compassHeading

local providers = ch.getProviders()       -- list of {id, label, enabled, category}
local p = ch.getProvider("my_tracker")    -- raw provider object
local active = ch.isProviderActive("my_tracker")  -- true/false

-- Disable/enable a provider at runtime
ch.setProviderEnabled("my_tracker", false)
ch.setProviderEnabled("my_tracker", true)
```

---

## 7. Events & Callbacks

Subscribe to compass events to react when things happen:

```lua
local ch = g_currentMission.compassHeading

-- When a marker scrolls into the visible compass FOV
ch.onMarkerEnterView(function(id)
    print("Marker entered view: " .. id)
end)

-- When a marker scrolls out of view
ch.onMarkerExitView(function(id)
    print("Marker left view: " .. id)
end)

-- When compass is toggled on/off
ch.onCompassToggle(function(enabled)
    print("Compass is now " .. (enabled and "ON" or "OFF"))
end)

-- When a provider is registered/unregistered
ch.onProviderRegistered(function(id)
    print("New provider: " .. id)
end)

ch.onProviderUnregistered(function(id)
    print("Provider removed: " .. id)
end)

-- When any setting changes at runtime
ch.onSettingChanged(function(key, newValue, oldValue)
    print(key .. " changed from " .. tostring(oldValue) .. " to " .. tostring(newValue))
end)

-- When a marker is added or removed
ch.onMarkerAdded(function(id) print("Added: " .. id) end)
ch.onMarkerRemoved(function(id) print("Removed: " .. id) end)
ch.onMarkerExpired(function(id) print("Expired: " .. id) end)

-- Generic event subscription (any event name)
ch.on("compassToggle", function(enabled) ... end)
```

---

## 8. Navigation Utilities

These functions are available for any mod to use, even if you never add a marker:

```lua
local ch = g_currentMission.compassHeading

-- Current heading (0-359 degrees)
local heading = ch.getBearing()

-- Bearing from player to a world point
local bearing = ch.getBearingTo(1200, -800)

-- Distance from player to a point (meters)
local dist = ch.getDistanceTo(1200, -800)

-- Player's current world position
local x, z = ch.getPlayerPosition()

-- Format distance for display
local str = ch.formatDistance(1500)   -- "1.5km"
local str = ch.formatDistance(250)    -- "250m"

-- Cardinal direction string
local dir = ch.getCardinalDirection()          -- "NNE" (from current heading)
local dir = ch.getCardinalDirection(180)       -- "S"

-- System status
local ready = ch.isReady()         -- true if compass is loaded
local ver   = ch.getVersion()      -- "1.0.0.0"
local on    = ch.isEnabled()       -- true/false
```

---

## 9. Settings Control

Modify compass settings at runtime without editing the XML file:

```lua
local ch = g_currentMission.compassHeading

-- Enable/disable the entire compass
ch.setEnabled(true)
ch.setEnabled(false)

-- Field of view (30-360 degrees)
ch.setFOV(180)
local fov = ch.getFOV()

-- Bar position and size
ch.setBarPosition(0.5, 0.97)   -- centre X, top Y (normalized 0-1)
ch.setBarWidth(0.7)            -- bar width (normalized)
ch.setBackgroundOpacity(0.5)   -- background alpha

-- Get/set any setting by key
ch.setSetting("showTickMarks", false)
local val = ch.getSetting("showPlayerMarkers")

-- Get all settings as a table
local all = ch.getSettings()

-- Reload settings from disk
ch.reloadSettings()

-- Save current settings to disk
ch.saveSettings()
```

### Available setting keys

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | true | Master on/off |
| `barPositionX` | number | 0.500 | Horizontal centre (0-1) |
| `barWidth` | number | 0.680 | Bar width (0-1) |
| `barVerticalY` | number | 0.974 | Vertical position (0-1) |
| `fieldOfView` | number | 160 | Compass FOV in degrees |
| `backgroundOpacity` | number | 0.62 | Bar background alpha |
| `showCardinalLabels` | boolean | true | Show N/S/E/W |
| `showIntercardinalLabels` | boolean | true | Show NE/SE/SW/NW |
| `showMinorLabels` | boolean | true | Show NNE/ENE etc. |
| `showTickMarks` | boolean | true | Show 10-degree ticks |
| `showCenterIndicator` | boolean | true | Show gold centre line |
| `showDegreeReadout` | boolean | true | Show numeric heading |
| `showPlayerMarkers` | boolean | true | Show other players |
| `playerMarkerFilter` | string | "all" | "all" = every player, "farm" = same farm only |
| `showWaypointMarker` | boolean | true | Show map waypoint |
| `showTornadoMarker` | boolean | true | Show tornado tracker (flashing red) |
| `showHelperMarkers` | boolean | true | Show AI hired workers on compass |
| `helperMarkerFilter` | string | "all" | "all" = every farm, "farm" = same farm only |
| `colCenterIndicator` | {r,g,b,a} | gold | Centre indicator colour |
| `colDegreeReadout` | {r,g,b,a} | white 50% | Heading number colour |
| `colCardinal` | {r,g,b,a} | white | N/S/E/W label colour |
| `colIntercardinal` | {r,g,b,a} | light grey | NE/SE/SW/NW colour |
| `colMinor` | {r,g,b,a} | grey | Minor direction colour |
| `colPlayerMarker` | {r,g,b,a} | cyan | Fallback player colour |
| `colWaypointMarker` | {r,g,b,a} | orange | Waypoint marker colour |
| `colTornadoMarker` | {r,g,b,a} | red | Tornado marker colour |
| `colHelperMarker` | {r,g,b,a} | green | Default helper marker colour |
| `colHelperBlocked` | {r,g,b,a} | red | Blocked helper alert colour |

---

## 10. Built-in Tracking Features

Compass Heading includes several built-in tracking systems that work out of the box.
All are toggle-able via `settings.xml` or the runtime settings API.

### Player Tracking
Other players in multiplayer appear as named markers in their farm colour.
Players are tracked on foot and in vehicles. Dedicated server phantom players are filtered.

| Setting | Values | Effect |
|---|---|---|
| `showPlayerMarkers` | true/false | Toggle player markers |
| `playerMarkerFilter` | "all" / "farm" | Show all players or same-farm only |

### AI Helper Tracking
Active hired workers (AI helpers) appear as chevron (`>>`) markers colour-coded by farm.
The helper's character name is displayed (e.g. "Clara", "Hans"). If a helper gets blocked
by an obstacle, the marker changes to a flashing red exclamation mark (`!`) with urgent
colour shift — making it easy to spot which helper needs attention from across the map.

| Setting | Values | Effect |
|---|---|---|
| `showHelperMarkers` | true/false | Toggle helper markers |
| `helperMarkerFilter` | "all" / "farm" | Show all farms or same-farm only |
| `colHelperMarker` | {r,g,b,a} | Default helper colour (overridden by farm colour) |
| `colHelperBlocked` | {r,g,b,a} | Blocked helper alert colour |

Blocked detection uses two methods:
1. **AIJob.stop hook** — catches `AIMessageErrorBlockedByObject` events from the engine
2. **didNotMoveTimer** — reads the internal stuck timer from `spec_aiFieldWorker`

Blocked state persists for 30 seconds on the compass after the event fires.

### Tornado Tracking
When a vanilla FS25 twister spawns, a flashing red diamond marker labeled "TORNADO"
appears on the compass with live distance. The tornado node is scanned every 2 seconds
and auto-clears when the twister despawns.

| Setting | Values | Effect |
|---|---|---|
| `showTornadoMarker` | true/false | Toggle tornado tracking |
| `colTornadoMarker` | {r,g,b,a} | Tornado marker colour |

Test with the console command: `gsWeatherTwisterSpawn`

---

## 11. Distance & Proximity

### Find the nearest marker
```lua
local nearest = ch.getNearestMarker()            -- any category
local nearest = ch.getNearestMarker("emergency")  -- specific category

if nearest then
    print(nearest.id, nearest.label, nearest.distance)
end
-- Returns: {id, x, z, label, category, distance}
```

### Find all markers within range
```lua
local nearby = ch.getMarkersInRange(500)            -- 500m radius, any category
local nearby = ch.getMarkersInRange(200, "shop")    -- 200m, shops only

for _, m in ipairs(nearby) do
    print(m.label, m.distance)
end
-- Each entry: {id, x, z, label, category, distance}
```

### Count markers
```lua
local total = ch.getMarkerCount()                -- all standalone markers
local fires = ch.getMarkerCount("emergency")     -- just emergency markers
```

---

## 12. Complete API Reference

### System
| Function | Returns | Description |
|----------|---------|-------------|
| `isReady()` | boolean | True if compass is loaded |
| `getVersion()` | string | Version string |
| `setEnabled(bool)` | — | Enable/disable compass |
| `isEnabled()` | boolean | Current enabled state |

### Settings
| Function | Returns | Description |
|----------|---------|-------------|
| `getSetting(key)` | any | Get setting value |
| `setSetting(key, value)` | boolean | Set setting value |
| `getSettings()` | table | Get all settings |
| `setFOV(degrees)` | boolean | Set field of view (30-360) |
| `getFOV()` | number | Get field of view |
| `setBarPosition(x, y)` | — | Set bar position |
| `setBarWidth(w)` | — | Set bar width |
| `setBackgroundOpacity(a)` | — | Set background alpha |
| `reloadSettings()` | — | Reload from disk |
| `saveSettings()` | — | Save to disk |

### Markers
| Function | Returns | Description |
|----------|---------|-------------|
| `addMarker(id, opts)` | boolean | Add a marker |
| `addTemporaryMarker(id, opts)` | boolean | Add marker with TTL |
| `removeMarker(id)` | boolean | Remove a marker |
| `updateMarker(id, fields)` | boolean | Update marker fields |
| `getMarker(id)` | table/nil | Get marker data |
| `hasMarker(id)` | boolean | Check if marker exists |
| `clearMarkers(category?)` | number | Remove markers, returns count |
| `getMarkerCount(category?)` | number | Count markers |

### Marker Visual Shortcuts
| Function | Returns | Description |
|----------|---------|-------------|
| `setMarkerColor(id, {r,g,b,a})` | boolean | Change colour |
| `setMarkerLabel(id, text)` | boolean | Change label |
| `setMarkerScale(id, n)` | boolean | Change size |
| `setMarkerPriority(id, n)` | boolean | Change draw order |
| `setMarkerStyle(id, style)` | boolean | Change glyph |
| `setMarkerPosition(id, x, z)` | boolean | Move marker |
| `setMarkerFlash(id, bool, hz?)` | boolean | Toggle flash |
| `setMarkerPulse(id, bool, hz?, min?, max?)` | boolean | Toggle pulse |
| `setMarkerUrgent(id, bool)` | boolean | Toggle urgent mode |
| `getMarkerStyles()` | table | List all style names |
| `getDefaultCategories()` | table | List default categories |

### Categories
| Function | Returns | Description |
|----------|---------|-------------|
| `setCategoryEnabled(cat, bool)` | boolean | Toggle category visibility |
| `isCategoryEnabled(cat)` | boolean | Check if category visible |
| `setCategoryColor(cat, {r,g,b,a})` | boolean | Set category colour |
| `getCategoryColor(cat)` | table | Get category colour |
| `getCategories()` | table | List all categories |

### Providers
| Function | Returns | Description |
|----------|---------|-------------|
| `getProvider(id)` | table/nil | Get provider object |
| `getProviders()` | table | List all providers |
| `isProviderActive(id)` | boolean | Check if provider active |
| `setProviderEnabled(id, bool)` | boolean | Toggle provider |

### Events
| Function | Returns | Description |
|----------|---------|-------------|
| `on(event, callback)` | id | Subscribe to any event |
| `onMarkerEnterView(cb)` | id | Marker enters compass FOV |
| `onMarkerExitView(cb)` | id | Marker leaves compass FOV |
| `onCompassToggle(cb)` | id | Compass enabled/disabled |
| `onProviderRegistered(cb)` | id | Provider added |
| `onProviderUnregistered(cb)` | id | Provider removed |
| `onSettingChanged(cb)` | id | Setting changed |
| `onMarkerAdded(cb)` | id | Marker added |
| `onMarkerRemoved(cb)` | id | Marker removed |
| `onMarkerExpired(cb)` | id | Marker TTL expired |

### Navigation
| Function | Returns | Description |
|----------|---------|-------------|
| `getBearing()` | number/nil | Current heading 0-359 |
| `getBearingTo(x, z)` | number/nil | Bearing to world point |
| `getDistanceTo(x, z)` | number/nil | Distance to point (meters) |
| `getPlayerPosition()` | x, z / nil | Player world position |
| `formatDistance(meters)` | string | "250m" or "1.5km" |
| `getCardinalDirection(bearing?)` | string | "N", "NNE", "NE", etc. |

### Proximity
| Function | Returns | Description |
|----------|---------|-------------|
| `getNearestMarker(category?)` | table/nil | Nearest marker + distance |
| `getMarkersInRange(range, category?)` | table | Markers within radius |

---

## 13. Real-World Examples

### Fire Call System
```lua
-- A fire mod that shows flashing red markers at fire locations
local function onFireStarted(fireId, x, z)
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch == nil then return end

    ch.addMarker("fire_" .. fireId, {
        x = x, z = z,
        label = "FIRE",
        category = "emergency",
        urgent = true,         -- flashing red
        maxDistance = 5000,     -- hide beyond 5km
    })
end

local function onFireExtinguished(fireId)
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch == nil then return end

    ch.removeMarker("fire_" .. fireId)
end
```

### Delivery Job with Pickup and Dropoff
```lua
local function startDeliveryJob(pickupX, pickupZ, dropoffX, dropoffZ)
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch == nil then return end

    ch.addMarker("delivery_pickup", {
        x = pickupX, z = pickupZ,
        label = "Pickup",
        style = "triangle",
        category = "delivery",
        pulse = true,
        maxDistance = 2000,
    })

    ch.addMarker("delivery_dropoff", {
        x = dropoffX, z = dropoffZ,
        label = "Dropoff",
        style = "square",
        category = "delivery",
        maxDistance = 2000,
    })
end

local function onPickupComplete()
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch then ch.removeMarker("delivery_pickup") end
end

local function onDeliveryComplete()
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch then
        ch.removeMarker("delivery_dropoff")
        -- Show a temporary "Complete!" marker that fades in and auto-removes
        ch.addTemporaryMarker("delivery_done", {
            x = 0, z = 0,  -- will be at player pos
            label = "Done!",
            style = "star",
            category = "activity",
            fadeIn = true,
            ttl = 5,
        })
    end
end
```

### Vehicle Fleet Tracker (Provider)
```lua
-- Track all owned vehicles on the compass
local function setupFleetTracker()
    local reg = g_currentMission and g_currentMission.compassHeadingRegistry
    if reg == nil then return end

    reg:register({
        id = "fleet_tracker",
        label = "My Fleet",
        category = "equipment",
        getPoints = function()
            local points = {}
            local vehicles = g_currentMission.vehicles or {}
            for _, veh in pairs(vehicles) do
                if veh.rootNode and veh.rootNode ~= 0 then
                    local x, _, z = getWorldTranslation(veh.rootNode)
                    local name = veh:getName() or "Vehicle"
                    if #name > 8 then name = name:sub(1, 8) end
                    points[#points + 1] = {
                        x = x, z = z,
                        label = name,
                    }
                end
            end
            return points
        end,
    })
end

-- Cleanup in deleteMap
local function cleanup()
    local reg = g_currentMission and g_currentMission.compassHeadingRegistry
    if reg then reg:unregister("fleet_tracker") end
end
```

### Proximity Alert
```lua
-- Alert when player gets close to a danger zone
local function checkDangerProximity()
    local ch = g_currentMission and g_currentMission.compassHeading
    if ch == nil then return end

    local nearby = ch.getMarkersInRange(100, "emergency")
    if #nearby > 0 then
        print("WARNING: " .. #nearby .. " emergency markers within 100m!")
    end
end
```

### Using Navigation Utilities Without Markers
```lua
-- Any mod can use these without adding any markers
local ch = g_currentMission and g_currentMission.compassHeading
if ch then
    local heading = ch.getBearing()
    local dir = ch.getCardinalDirection()
    local x, z = ch.getPlayerPosition()
    local dist = ch.getDistanceTo(1000, -500)

    print(string.format("Heading: %d (%s) | Distance to target: %s",
        heading, dir, ch.formatDistance(dist)))
    -- Output: "Heading: 045 (NE) | Distance to target: 1.2km"
end
```

---

## Settings File

On first launch, a settings XML is created at:
```
Documents/My Games/FarmingSimulator2025/modSettings/FS25_CompassHeading/settings.xml
```

Players can edit this file to customize every aspect of the compass. Changes take effect on next map load, or mods can call `reloadSettings()` at runtime.

---

## Multiplayer

Compass Heading runs entirely client-side. No network events are sent. Each client independently:
- Reads player positions from `g_currentMission.playerSystem`
- Reads waypoint positions from the local map
- Renders markers added via the API

This means:
- Zero bandwidth impact
- No server-side code needed
- Works on dedicated servers and listen servers
- Player markers automatically show other connected players in their farm colour

---

## Support

- Website & API docs: [FSCORE.GG](https://fscore.gg)
- Issues & feedback: Discord (see FSCORE.GG for invite link)

---

*Compass Heading Display is part of the FSCore platform by RocklandUSA Gaming.*
