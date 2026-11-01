# RavenEra DLL API

Complete command reference for the RavenEra libraries (RavenEra.dll, HearthDB.dll).

## How you call it

```lua
SetTaxiBenchmarkMode("Command", "argument")   -- RavenEra.dll gate
GetTaxiBenchmarkMode("Command", "argument")   -- RavenEra.dll proxy entry point
IsOnTournamentRealm("Command", "argument")    -- HearthDB.dll gate
```

Those are the names you call. The loader takes over three script-table entries that no
addon in the 1.12/1.14 corpora uses and swaps their **function pointers**; the names stay
as they are, which is what makes loading reliable.

If you also install the `!RavenEraAPI` addon, it assigns shorter aliases on top of them:

| Alias (addon only) | Native name |
|---|---|
| `RavenEra(...)` | `SetTaxiBenchmarkMode` |
| `RavenEraGate(...)` | `SetTaxiBenchmarkMode` |
| `RavenEraProxy(...)` | `GetTaxiBenchmarkMode` |
| `HearthDBGate(...)` | `IsOnTournamentRealm` |

## The payload

The native gate takes **exactly two arguments**: the command name and one payload
string. Extra arguments are dropped, so several values are joined with `\x01`:

```lua
SetTaxiBenchmarkMode("ExportFile", path .. string.char(1) .. text)
```

With the addon, `RavenEra(cmd, a, b)` does that joining for you.

Commands ending in `N` return real numbers; the same command without `N` returns a
formatted string. Prefer the `N` variants.


> **Threading note.** The gate may only be called from the client's Lua thread —
> that is, from addon code. Calling it from another thread is unsupported.

---

## Parameter formats

Every command below uses these placeholders. Passing the wrong one is the most
common source of "the command returned nothing".

| Placeholder | Meaning | Example |
|---|---|---|
| `<low>` | **Low part of a GUID** — the last segment of `UnitGUID`, hexadecimal. Case-insensitive, `0x` optional. This is what most commands want. | `"277b29"` |
| `<guid>` | **Full GUID string**, or `<low>`. Both are accepted. | `"Creature-0-1-0-0-6466-0000003025"` |
| `<token>` | **Unit token** as used by the stock API. See the list below. | `"target"` |
| `<num>` | Decimal number. | `"1500"` |
| `<a>,<b>` | Two `<low>` values separated by a **comma**, no spaces. | `"242f,277b29"` |
| `<radius>` | Decimal radius in yards. | `"40"` |
| `<combo>` | Key combination with `+`. | `"CTRL+SHIFT+X"`, `"W"`, `"ENTER"` |
| `<path>` | Path relative to the game folder. `..` and absolute paths are rejected. | `"Logs\\out.txt"` |
| `<text>` | Free text; may contain `\x01` only as an internal separator. | `"line one\r\n"` |

### How to get a `<low>`

```lua
local low = UnitGUID("target"):match("(%x+)$")     --> "277b29"
```

`UnitGUID` returns `Creature-0-1-0-0-6466-0000003025`; `:match("(%x+)$")` keeps the
`3025` part. Never pass the full string to a `<low>` parameter — use `<guid>` or the
`-Low` command variant instead.

### Valid `<token>` values

`player`, `target`, `focus`, `mouseover`, `pet`, `targettarget`, `pettarget`,
`party1`…`party4`, `pet1`…`pet4`, `raid1`…`raid40`, `raidpet1`…`raidpet40`,
`boss1`…`boss8`, `nameplate1`…`nameplate40`, `arena1`…`arena5`.

`RavenEra("UnitCandidateList")` prints the tokens this client actually accepts.

### Which commands take which

| Parameter | Commands |
|---|---|
| `<low>` | `UnitHP`, `UnitHPN`, `UnitStats`, `UnitStatsN`, `UnitPowerN`, `UnitResist`, `UnitResistN`, `UnitReach`, `UnitReachN`, `UnitNameN`, `UnitTypeN`, `UnitOwner`, `UnitTarget`, `Assist`, `ReactionN`, `Reaction`, `IsHostile`, `LootProbe`, `LootProbeN`, `ObjByGuid`, `ScanObj`, `TokenForGuid` |
| `<low>` **or** `<token>` | `UnitPos`, `UnitPosN` |
| `<guid>` | `TargetByGuid`, `FocusByGuid` |
| `<low>` only | `TargetByGuidLow` |
| `<token>` **or** `<guid>` | `SetTargetCore`, `SetTargetToken`, `ResolveToken`, `SetFocusToken`, `NameplateIndex` |
| `<a>,<b>` | `UnitDist`, `UnitBehindN`, `LosUnits` |

---

## 1. Core and diagnostics

| Command | Returns | Description |
|---|---|---|
| `Ping` | `true` | Sanity check: the DLL is loaded and the gate answers. |
| `Caps` | `number` | Capability bitmask (reserved for feature detection). |
| `GetVersion` / `ver` | `0, 1` | Gate protocol version. |
| `ArmNextTarget` | `true` | Arms the secure bit for the next protected engine call. |
| `GetLocalPlayer` | `string` | Address of the local player object (hex). |
| `UnitCandidateList` | `string` | All unit tokens this client accepts, space separated. |

---

## 2. Object manager

| Command | Returns | Description |
|---|---|---|
| `ObjMgr` | `"begin/end/count"` | Addresses and size of the object container (diagnostics). |
| `ObjCount` | `number` | Number of live objects. |
| `ObjDbg` | `string` | Dumps the object container (diagnostics). |
| `ObjByGuid(<low>)` | `"found=… vt=… pos=…"` | Object address, vtable and position. Searches the container and memory, so the unit does not have to be your target. |
| `ScanObj(<low>)` | `"found=… pos=…"` | Same lookup, shorter output (diagnostics). |
| `TokenForGuid(<low>)` | `string` | Unit token for that GUID (`"player"`, `"nameplate3"`), when the client has one. |

---

## 3. Unit data (read-only)

| Command | Payload | Returns | Description |
|---|---|---|---|
| `UnitPos(<low\|token>)` | unit | `"r=1 x y z o"` | Position and orientation as text. |
| `UnitPosN(<low\|token>)` | unit | `ok, x, y, z, o` | Same as numbers. |
| `UnitStats(<low>)` | low | `"r=1 hp=… max=… pow=… maxpow=… level=… armor=…"` | Full stat block as text. |
| `UnitStatsN(<low>)` | low | `ok, hp, maxhp, power, maxpower, level` | Same as numbers. |
| `UnitHP(<low>)` | low | `"r=1 hp=… max=… level=…"` | Health and level as text. |
| `UnitHPN(<low>)` | low | `ok, hp, maxhp, level` | Health and level as numbers. |
| `UnitPowerN(<low>)` | low | `ok, power, maxpower` | Mana/rage/energy. |
| `UnitResist(<low>)` | low | `"r=1 armor=… holy=… fire=…" …` | Armor and the six resistances as text. |
| `UnitResistN(<low>)` | low | `ok, armor, holy, fire, nature, frost, shadow, arcane` | Same as numbers (8 values). |
| `UnitReach(<low>)` | low | `"ok=1 reach=… radius=…"` | Combat reach and hit radius as text. |
| `UnitReachN(<low>)` | low | `ok, reach, radius` | Same as numbers. |
| `UnitNameN(<low>)` | low | `string` | Name from the creature record (e.g. `Voidwalker` for a renamed pet). |
| `UnitTypeN(<low>)` | low | `number, string` | Creature type id and name (`3, "Demon"`). |
| `UnitOwner(<low>)` | low | `"ok=1 lo=… hi=…"` | Owner GUID of a pet or totem; `lo=0` when there is no owner. |
| `UnitTarget(<low>)` | low | `string` | Low GUID of **what that unit is targeting**. Same as the client's `targettarget`. |
| `UnitLastTarget()` | — | `string` | Low GUID of the last unit you had targeted. |
| `LastTarget()` | — | `true` | Re-targets the unit from `UnitLastTarget`. |
| `UnitDist(<a>,<b>)` | two lows | `number` | Distance in yards between two units; `-1` when either is not found. |
| `UnitBehindN(<a>,<b>)` | two lows | `ok, behind, angle` | Is `b` behind `a` (angle ≥ 90°). |

---

## 4. Reaction and hostility

| Command | Payload | Returns | Description |
|---|---|---|---|
| `ReactionN(<low>)` | low | `number` | Reaction `1`…`8`: 1 hated, 2 hostile, 3 unfriendly, 4 neutral, 5 friendly, 6+ honoured. |
| `Reaction(<low>)` | low | `string` | `"5 friendly"`, `"4 neutral"`, `"2 hostile"`. |
| `IsHostile(<low>)` | low | `bool` | `true` only for reaction ≤ 3. Neutral mobs and friendly pets return `false`. |

Rules used by the scans:

| Meaning | Rule |
|---|---|
| hostile | reaction ≤ 3 |
| attackable | reaction ≤ 4 (neutral is attackable, friendly is not) |
| friendly | reaction ≥ 5 |

The attackable **flag** in the descriptor is unreliable on its own: neutral NPCs such
as Gamon are attackable but do not carry it.

---

## 5. Targeting

| Command | Payload | Returns | Description |
|---|---|---|---|
| `TargetByGuid(<guid>)` | full GUID or low | `bool` | Sets your target. Accepts `Creature-…-HEX` or bare hex. |
| `TargetByGuidLow(<low>)` | low only | `bool` | Same, but strictly a low part. |
| `SetTargetCore(<token\|guid>)` | token or GUID | `bool` | Kernel-level target set; works where the stock API is protected. |
| `SetTargetToken(<token\|guid>)` | token or GUID | `bool` | Alias of `SetTargetCore`. |
| `Assist(<low>)` | low | `bool` | **Takes the target of that unit**: reads its target GUID and makes it yours. Fails when the unit has no target. |
| `ClearTargetCore()` | — | `true` | Clears the target. |
| `ResolveToken(<token\|guid>)` | token or GUID | `"r=1 lo=… hi=…"` | Resolves a token (or GUID) to the raw GUID halves. Empty payload resolves the current target. |

`Assist` needs the unit's **low**, not a token:

```lua
local low = UnitGUID("target"):match("(%x+)$")
print(RavenEra("Assist", low))     --> true, and your target becomes what that unit targets
```

If that unit has no target of its own, `Assist` returns `false` and your target does
not change.

---

## 6. Focus

| Command | Payload | Returns | Description |
|---|---|---|---|
| `SetFocusToken(<token\|guid>)` | token or GUID | `bool` | Sets the focus unit. |
| `FocusByGuid(<guid>)` | full GUID or low | `bool` | Same, taking a GUID. |
| `FocusDesc()` | — | `"lo=… hi=…"` | Focus GUID halves. |
| `ClearFocus()` | — | `true` | Clears the focus. |

---

## 7. Scanning

| Command | Payload | Returns | Description |
|---|---|---|---|
| `ScanUnits(<radius>[,p][,d][,h][,a])` | radius + flags | `n, list, "slots=… live=… inrange=… passed=…"` | All units in range. |
| `ScanCorpses(<radius>)` | radius | `n, list` | Lootable corpses only. |
| `NearestHostileN(<radius>)` | radius | `low, dist, hp, level` | Nearest unit with reaction ≤ 3. |
| `NearestAttackableN(<radius>)` | radius | `low, dist, hp, level` | Nearest unit with reaction ≤ 4. |
| `ScanNameplates()` | — | `n, list` | Visible nameplates (`1:guid 2:guid …`). |
| `ScanMarks()` | — | `n, list` | Raid marks (`4:guid …`). |
| `NameplateGuid(<num>)` | 1-based index | `string` | Low GUID behind that nameplate slot. |
| `NameplateIndex(<token\|guid>)` | token or GUID | `number` | Nameplate slot of a unit, `-1` when it has none. |
| `MarkGuid(<num>)` | 1…8 | `string` | Low GUID carrying that raid mark. |

### Flags

| Flag | Meaning |
|---|---|
| `p` | include players |
| `d` | include dead units |
| `h` | only hostile (reaction ≤ 3) |
| `a` | only attackable (reaction ≤ 4) |

### List format

Each entry is `guid:dist:hp:maxhp:level:kind`, separated by spaces:

```
277b29:12.1:288:288:12:n 2c4:16.0:847:847:25:n
```

`kind` is `n` (NPC), `p` (player) or `d` (dead).

The first return value is the number of entries; the third explains how many objects
were seen and why some were filtered out.

---

## 8. Loot

| Command | Payload | Returns | Description |
|---|---|---|---|
| `LootProbe(<low>)` | low | `"rc=1 obj=… id=… hasLoot=… canLoot=…"` | Loot state of a corpse as text. |
| `LootProbeN(<low>)` | low | `ok, hasLoot, canLoot, id` | Same as numbers. `canLoot` is unreliable — the client reports `0` even at 0.3 yards where looting works. |
| `Loot([<num>])` | slot, default 1 | `number` | Takes that loot slot of the open window. `1` = taken, `0` = nothing. |

The loot window must already be open — see `InteractIn`.

---

## 9. Interaction (the only synthetic input)

The library deliberately exposes **one** input action. There is no generic key press,
no key hold and no movement control.

| Command | Payload | Returns | Description |
|---|---|---|---|
| `InteractKey()` | — | `bool` | Presses the user's interact key, which fires `INTERACTTARGET`. |
| `InteractIn([<num>])` | delay ms, default `0` | `bool` | Same, after a delay, from a worker thread. |

Both commands refuse to press anything unless **all** of these hold:

| Condition | Checked |
|---|---|
| the current target exists | in the client's target slot |
| the target is a corpse | descriptor health is `0` |
| the corpse still has loot | the client's own "has loot" query |
| the corpse is within interact range | `<= 5` yards from the player |

Anything else returns `false` and no key is sent.

The key itself is fixed: `Ctrl+Shift+X`. It must be bound to `INTERACTTARGET`, and the
library can only ever trigger that one action, never an arbitrary key. The binding is
set through the proxy entry point, which reaches the client's own `SetBinding`:

```lua
GetTaxiBenchmarkMode("CTRL-SHIFT-X", "INTERACTTARGET", "bind")   --> 1
```

Run it once per session, or let the `!RavenEraAPI` addon do it on load. No addon is
required for it.

## 10. Camera

| Command | Returns | Description |
|---|---|---|
| `CameraPosN()` | `ok, x, y, z` | Camera position. |
| `CameraAnglesN()` | `ok, yaw, pitch, distance` | Camera orientation and distance to the target. |
| `CameraAllN()` | `ok, x, y, z, yaw, pitch, distance` | Everything in one call. |

---

## 11. Files

| Command | Payload | Returns | Description |
|---|---|---|---|
| `ExportFile(<path>\x01<text>)` | path + text | `bool` | Overwrites a file with `text`. |
| `AppendFile(<path>\x01<text>)` | path + text | `bool` | Appends to a file. |
| `ImportFile(<path>)` | path | `string` or `"open failed"` | Reads a whole file. |
| `CombatLogAdd(<line>[\x01<mode>])` | line + mode | `bool` | Writes a line into the combat log files. |

Paths are relative to the game folder; `..`, `/` and absolute paths are rejected by
`ExportFile` with `"unsafe path"`.

`CombatLogAdd` modes:

| Mode | Effect |
|---|---|
| *(none)* | Timestamped line into the session `Logs\WoWCombatLog-*.txt` and the root `WowCombatLog.txt`. |
| `raw` | Same, plus `WoWRawCombatLog.txt`. |
| `root` | Only the root `WowCombatLog.txt`, timestamped. |
| `plain` | No timestamp, root file only. |

---

## 12. Diagnostics

| Command | Payload | Returns | Description |
|---|---|---|---|
| `Los()` | — | `"ctx=… open=… ground=…"` | Line-of-sight context. |
| `LosUnits(<a>,<b>)` | two lows | `"insight=… hit=… d=… dist=…"` | Line of sight between two units. `hit=1, insight=0` means something blocks it. |

---

## Common mistakes

| Symptom | Cause | Fix |
|---|---|---|
| Command returns nothing | Wrong parameter format | Check the table above: `<low>` vs `<guid>` vs `<token>`. |
| `Assist` returns `false` | That unit has no target | It only works when the unit itself targets something. |
| `UnitTarget` returns nothing | Same | The unit is not targeting anything. |
| `Loot` returns `0` | No loot window open | Target a lootable corpse in range, then call `InteractIn` and wait ~1 s. |
| `ScanUnits` misses a unit | Flag filter or radius | Check the `h`/`a` flags and the radius. |
| `ExportFile` returns `"unsafe path"` | `..` or `/` in the path | Use backslashes relative to the game folder. |
| Second argument ignored | The gate takes only two arguments | Join extra values with `\x01`. |

## Status conventions

| Form | Meaning |
|---|---|
| `true` / `false` | Plain boolean result. |
| `1` / `0` | Numeric result. |
| no values | The unit or object was not found. |
| `ok` as first value | Boolean success of an `N` command; the rest is valid only when it is `true`. |
| `<cmd>N` vs `<cmd>` | Numbers vs formatted text. |
