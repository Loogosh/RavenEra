# RavenEra DLL API

Complete command reference for the three RavenEra libraries.

Everything the client exposes to Lua is reached through a single gate function:

```lua
RavenEra("Command", "argument")
```

Arguments are strings; results are Lua values (booleans, numbers, strings).
Commands whose name ends with `N` return real numbers instead of a formatted string
and should be preferred.

> **Threading note.** The gate may only be called from the client's Lua thread —
> that is, from addon code. Calling it from another thread is unsupported.

---

## 1. Core and diagnostics

| Command | Returns | Description |
|---|---|---|
| `Ping` | `true` | Sanity check: the DLL is loaded and the gate answers. |
| `Caps` | `15` | Capability bitmask (reserved for feature detection). |
| `GetVersion` / `ver` | `0, 1` | Gate protocol version. |
| `Pump` | — | Drains the internal work queue. |
| `Reset` | — | Resets the gate's internal state. |
| `ArmNextTarget` | `true` | Arms the secure bit for the next protected engine call. |

---

## 2. Object manager

| Command | Returns | Description |
|---|---|---|
| `ObjMgr` | `"begin/end/count"` | Addresses and size of the object container (diagnostics). |
| `ObjCount` | `number, string` | Number of live objects. |
| `ObjDbg` | `string` | Dumps the object container. |
| `ObjByGuid` | `"found=… vt=… pos=…"` | Object address, vtable and position for a GUID. Searches the container and memory, so the GUID does not have to be the current target. |
| `ScanObj` | `string` | Raw object scan (diagnostics, up to 64 MB). |
| `GetLocalPlayer` | `string` | Address of the local player object. |

---

## 3. Scanning

| Command | Returns | Description |
|---|---|---|
| `ScanUnits` | `n, "guid:dist:hp:max:lvl:kind …"` | Units within a radius. Optional flags appended to the radius: `p` include players, `d` include dead, `h` hostile only. |
| `ScanCorpses` | `n, "guid:dist:hasLoot …"` | Corpses within a radius, with a loot flag. |
| `ScanNameplates` | `"n=K 1:guid 2:guid …"` | All active nameplates and their units. |
| `ScanMarks` | `"n=K 1:guid 4:guid …"` | All active raid marks. |

`kind` is `n` (npc), `p` (player) or `d` (dead).

---

## 4. Unit data

All of these accept a bare hex GUID low (for example `"277b42"`).
`UnitPosN` additionally accepts unit tokens such as `player`, `target` or `pet`.

| Command | Returns | Description |
|---|---|---|
| `UnitPos` | `"r=1 x y z o"` | Position and orientation as text (debugging). |
| `UnitPosN` | `ok, x, y, z, orientation` | Position and orientation as numbers. |
| `UnitStats` | `"r=1 hp=… level=… armor=…"` | All stats as text. |
| `UnitStatsN` | `ok, hp, maxHp, power, maxPower, level, armor` | All stats as numbers. |
| `UnitHP` | `"r=1 hp=… max=… level=…"` | Health as text. |
| `UnitHPN` | `ok, hp, maxHp, level` | Health as numbers. |
| `UnitPowerN` | `ok, power, maxPower, powerType` | Power as numbers. `powerType`: 0 mana, 1 rage, 2 focus, 3 energy. |
| `UnitResist` | `"r=1 armor=… holy=… …"` | Armor and school resistances as text. |
| `UnitResistN` | `ok, armor, holy, fire, nature, frost, shadow, arcane` | Resistances as numbers. |
| `UnitReach` | `"ok=1 reach=… radius=…"` | Combat reach and hit radius as text. |
| `UnitReachN` | `ok, reach, radius` | Combat reach and hit radius as numbers. |
| `UnitDist` | `number` or `-1` | Distance between two GUIDs, passed as `"guidA,guidB"`. |
| `UnitBehindN` | `ok, behind, angleDeg` | Whether unit A stands behind unit B, plus the exact angle. |
| `UnitOwner` | `"ok=… lo=… hi=…"` | Owner GUID of a pet or totem. |
| `UnitNameN` | `string` or `nil` | Creature name by GUID, read straight from the creature record. Players return `nil`. |
| `UnitTypeN` | `id, name` | Creature type id and name. `1 Beast`, `2 Dragonkin`, `3 Demon`, `4 Elemental`, `5 Giant`, `6 Undead`, `7 Humanoid`, `8 Critter`, `9 Mechanical`, `10 Not specified`, `11 Totem`. |

---

## 5. Targets

| Command | Returns | Description |
|---|---|---|
| `TargetByGuid` | `bool` | Targets a unit by a full GUID string, for example `"Creature-0-1-0-0-62226-0000277B41"`. |
| `TargetByGuidLow` | `bool` | Targets a unit by a bare hex GUID low. |
| `SetTargetToken` | `bool` | Targets a unit by token (`target`, `pet`, `focus`, …). |
| `SetTargetCore` | `bool` | Low level engine target call. |
| `ClearTargetCore` | `bool` | Clears the target through the engine. The Lua `ClearTarget` is protected and silently refuses. |
| `ResolveToken` | `"r=1 lo=… hi=…"` | Resolves a unit token to its 128-bit GUID. `r=0` means the client cannot resolve it right now. |
| `UnitTarget` | `hex` or `nil` | Current target GUID low of the given unit. |
| `UnitLastTarget` | `hex` or `nil` | Previous target GUID low of the player. |
| `Assist` | `bool` | Targets whatever the given unit is targeting. |
| `LastTarget` | `bool` | Returns to the previous target. |

---

## 6. Focus

| Command | Returns | Description |
|---|---|---|
| `FocusByGuid` | `bool` | Sets the focus by GUID. |
| `SetFocusToken` | `bool` | Sets the focus by token. |
| `FocusDesc` | `string` | Raw focus descriptor (diagnostics). |
| `ClearFocus` | `bool` | Clears the focus. |

---

## 7. Hostility

| Command | Returns | Description |
|---|---|---|
| `IsHostile` | `bool` | Whether the unit is attackable by the player. Units owned by the player (own pet, totem) are excluded. |
| `NearestHostileN` | `guid, dist, hp, level` | Nearest hostile unit within a radius. |

---

## 8. Line of sight

| Command | Returns | Description |
|---|---|---|
| `Los` | `"ctx=… found=…"` | Line-of-sight context state (diagnostics). |
| `LosUnits` | `"insight=… hit=… d=… dist=…"` | Direct visibility between two GUIDs, passed as `"guidA,guidB"`. |

---

## 9. Tokens and marks

| Command | Returns | Description |
|---|---|---|
| `TokenForGuid` | `string` or `nil` | Token name for a GUID low, including `nameplateN` when the unit has a nameplate. |
| `UnitCandidateList` | `string` | All tokens separated by spaces: 58 fixed tokens plus the nameplates that actually exist. |
| `MarkGuid` | `hex` or `nil` | GUID low of the unit carrying raid mark `1..8`. |
| `NameplateIndex` | `number` or `-1` | Nameplate index of a GUID, 0-based. `-1` means the unit has no nameplate. |
| `NameplateGuid` | `hex` or `nil` | GUID low of nameplate `1..40`. |

---

## 10. Camera

| Command | Returns | Description |
|---|---|---|
| `CameraPosN` | `ok, x, y, z` | Camera position in the world. |
| `CameraAnglesN` | `ok, yaw, pitch, distance` | Camera orientation and distance from the player. Angles are in radians; a negative pitch means looking down. |
| `CameraAllN` | `ok, x, y, z, yaw, pitch, distance` | Everything in one call. |

The camera angles are derived from the camera position and the look-at point, so they
stay correct regardless of zoom or first-person mode.

---

## 11. Loot

| Command | Returns | Description |
|---|---|---|
| `Loot` | `1` / `0` | Takes a loot slot from an open loot window. Engine call that bypasses the Lua protection. |
| `LootProbe` | `"rc=… obj=… hasLoot=… canLoot=…"` | Loot flags for a GUID as text. |
| `LootProbeN` | `rc, hasLoot, canLoot, id` | Loot flags as numbers and booleans. `rc=0` means the object was not found. |
| `SetAutoloot` | `true` | Compatibility flag; logged only. |

---

## 12. Input

| Command | Returns | Description |
|---|---|---|
| `InteractKey` | `true` | Presses the interact binding through `SendInput`. This is the only way to open a loot window, because Lua-initiated interaction is blocked. |
| `InteractKeyMsg` | `true` | Same thing through `PostMessage`; a fallback when `SendInput` is filtered. |
| `MoveKeysUp` | `number` | Releases W/A/S/D/Space and returns the key mask that was held. |
| `MoveKeysDown` | `bool` | Restores the keys from a mask returned by `MoveKeysUp`. |

---

## 13. Files and logging

| Command | Returns | Description |
|---|---|---|
| `AppendFile` | `bool` | Appends text to a file. The payload is `"path\0text"`. |
| `ExportFile` | `bool` | Overwrites a file. |
| `ImportFile` | `string` or `nil, "open failed"` | Reads a file. |
| `CombatLogAdd` | `bool` | Adds an entry to the client combat log. |
| `TrackUnit` / `UntrackUnit` | `bool` | Compatibility stubs. |

---

## Return value conventions

| Convention | Meaning |
|---|---|
| `ok` (first value) | `true` when the call succeeded. Always check it. |
| `nil` with no second value | The command did not push anything; treat it as a miss. |
| `-1` | Explicit failure marker (distance, index). |
| `hex` | GUID low as a lowercase hexadecimal string without `0x`. |

## Known limitations

| Limitation | Detail |
|---|---|
| Radius scans | The result buffer holds roughly 20 units per call; `n` is the true count. |
| `UnitDist` and `LosUnits` | Accept only `"guidA,guidB"`, not tokens. |
| Creature data | `UnitNameN` and `UnitTypeN` work for creatures only; players return `nil`. |
| Corpses | A corpse must be loaded in the object container. Objects that are out of range are not in memory. |
| Descriptor updates | The client updates descriptor fields on the next frame, so wait a frame after changing the target before reading it. |
| Casting | Not provided. Use the built-in `UnitCastingInfo` and `UnitChannelInfo`. |
| Protected actions | Casting and interaction cannot be triggered from the DLL; input has to go through `SendInput`. |

---

## Example

```lua
-- minimal gate wrapper (the addon shipped separately is not part of this release)
local G = SetTaxiBenchmarkMode            -- the FrameScript slot claimed by SlotPrep
local function RavenEra(cmd, arg) return G(cmd, arg) end

local ok, x, y, z, o = RavenEra("UnitPosN", "player")
if ok then print(string.format("player at %.1f %.1f %.1f", x, y, z)) end

local n, list = RavenEra("ScanUnits", "30,h")
print(n .. " hostiles: " .. tostring(list))

local guid, dist = RavenEra("NearestHostileN", "40")
if guid then print("nearest hostile " .. guid .. " at " .. dist) end
```


---

# 14. HearthDB — embedded SQLite (separate gate)

`HearthDB.dll` claims its own FrameScript slot, so it has a **separate gate**:

```lua
local H = IsOnTournamentRealm        -- HearthDB gate
```

Its commands take several arguments, joined by `\x01` (`string.char(1)`) when calling
through Lua:

```lua
local SEP = string.char(1)
H("Query", handle .. SEP .. "select * from t where name = 'x'")
```

The addon that usually wraps this gate is **not required**. Everything below is callable
directly; the only thing an addon adds is parsing of the packed result string.

## 14.1 Connection

| Command | Returns | Description |
|---|---|---|
| `GetVersion` / `ver` | `major, minor, patch` | Engine version, currently `0, 4, 1`. |
| `Open` | `handle` or `nil, error` | Opens a database file relative to the client folder. |
| `OpenAddon` | `handle` or `nil, error` | Opens a database shipped inside an addon. Payload is `"addonName\x01path/in/addon"`. Opened read-only. |
| `Close` | — or `nil, error` | Closes a handle. |
| `Reset` | — | Resets the engine. |
| `Pump` | — | Drains the async queue. Call it from an `OnUpdate` handler if you use async commands. |
| `ClearPoison` | — | Clears a poisoned handle after a failed statement. |

Handles are small integers starting at 1.

## 14.2 Synchronous queries

| Command | Returns | Description |
|---|---|---|
| `Execute` | — or `nil, error` | Runs a statement that returns no rows: `INSERT`, `UPDATE`, `DELETE`, DDL. Payload is `"handle\x01sql"`. |
| `Query` | `packed` or `nil, error` | Runs a `SELECT` and returns a packed table with column names. |
| `QueryRaw` | `packed` or `nil, error` | Same, but without the named-column header semantics. |

## 14.3 Asynchronous queries

| Command | Returns | Description |
|---|---|---|
| `ExecuteAsync` | `ticket` or `nil, error` | Queues a statement. Returns a ticket number. |
| `QueryAsync` | `ticket` or `nil, error` | Queues a `SELECT`. |
| `QueryRawAsync` | `ticket` or `nil, error` | Queues a raw `SELECT`. |
| `GetResult` | `packed` / `rowsAffected` / `nil` / `nil, error` | Fetches the result of a ticket. Returns `nil` (one value) while the job is still pending, an integer for `INSERT`/`UPDATE`, or the packed string for a query. |

Async jobs run on a worker thread. Call `Pump` regularly to deliver ready results.

## 14.4 Packed result format

A query result is one Lua string:

```
S<ncols>;<len>:<colName><len>:<colName>...<nrows>;<len>:<cell><len>:<cell>...
```

* `S` marks the start of a packed table.
* Every field length is decimal digits followed by `:`, then exactly that many bytes.
* A length of `-1` means SQL `NULL` and no bytes follow.
* Rows are flattened: `nrows × ncols` cells in reading order.

Minimal parser:

```lua
local function parsePacked(s)
  if type(s) ~= "string" or s:byte(1) ~= 83 then return nil, "not packed" end
  local pos = 2
  local function number(stop)
    local p = s:find(stop, pos, true)
    if not p then return nil end
    local v = tonumber(s:sub(pos, p - 1))
    pos = p + 1
    return v
  end
  local ncols = number(";")
  local cols = {}
  for i = 1, ncols do
    local n = number(":")
    cols[i] = s:sub(pos, pos + n - 1)
    pos = pos + n
  end
  local nrows = number(";")
  local rows = {}
  for r = 1, nrows do
    local row = {}
    for c = 1, ncols do
      local n = number(":")
      if n >= 0 then
        row[cols[c]] = s:sub(pos, pos + n - 1)
        pos = pos + n
      end
    end
    rows[r] = row
  end
  return rows, cols
end
```

## 14.5 Example

```lua
local H = IsOnTournamentRealm        -- slot 0 gate
local SEP = string.char(1)

-- database shipped inside an addon, opened read-only
local h = H("OpenAddon", "HearthDB" .. SEP .. "data/smoke.db")
if not h then return end

-- synchronous query; the result is one packed string
local packed = H("Query", h .. SEP .. "select 1 as x")
local rows = parsePacked(packed)
if rows then print(rows[1].x) end          -- prints 1

H("Close", h)
```

Verified output for the query above: `S1;1:x1;1:1` — one column `x`, one row, value `1`.

## 14.6 HearthDB limitations

| Limitation | Detail |
|---|---|
| One writer | SQLite allows a single writer at a time; concurrent `Execute` calls return a busy error. |
| Paths | Relative to the client folder. Forward slashes work, backslashes must be escaped in Lua strings. |
| Read-only addon databases | `OpenAddon` opens read-only by design. |
| Async delivery | Results are delivered only while `Pump` is called; without it, `GetResult` keeps returning `nil`. |
| Threading | Same rule as the unit gate: call from the Lua thread only. |
