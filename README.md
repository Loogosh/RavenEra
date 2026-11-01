# RavenEra

A low-level compatibility layer that extends the client's scripting API.
It gives addon authors access to data and actions the client
does not expose through its own scripting API: exact unit positions, distances, line of
sight, target and focus control, raid marks, nameplates, creature names and types,
the camera, loot handling, hardware input, and an embedded SQLite engine.

Everything is reached through gate functions that live in the DLLs:

```lua
SetTaxiBenchmarkMode("Command", "argument")   -- unit, world, loot and input commands
GetTaxiBenchmarkMode("Command", "argument")   -- binding control and native calls
IsOnTournamentRealm("Command", "argument")    -- SQLite commands
```

Those names belong to the client; the loader takes over three unused script-table entries
and swaps their function pointers, keeping the names so loading stays reliable. With the
`!RavenEraAPI` addon installed you can use the shorter aliases `RavenEra(...)`,
`RavenEraGate(...)`, `RavenEraProxy(...)` and `HearthDBGate(...)`.

See **[API.md](API.md)** for the full command reference, including the SQL API.

**Version 1.1**

---

## Scope

* **One client build only.** The libraries read hardcoded memory offsets of one specific
  client build. Any other build, language pack or client patch will fail.
* **Not for commercial use.** Personal, educational and non-commercial addon development only.
* **No warranty.** The libraries read and write client memory. They can crash the
  client, and no support is provided.

---

## Contents

| File | Role |
|---|---|
| `version.dll` | Proxy loader. Loads the other libraries and forwards the real `version.dll` exports. |
| `SlotPrep.dll` | Reserves script-engine slots early in startup so the gates can be registered. |
| `RavenEra.dll` | World and unit gate: every `RavenEra(...)` command in `API.md`. |
| `HearthDB.dll` | SQLite engine: opens databases, runs queries, returns packed tables. |

`HearthDB.dll` is independent: it claims its own slot and works without `RavenEra.dll`.
It needs `version.dll` and `SlotPrep.dll` only to be loaded and to claim a slot.

The libraries expose commands, not a ready-made addon. The full list is in `API.md`.

---

## Installation

1. Close the game client.
2. Copy the four DLLs next to the game executable, into the same folder.
3. Start the client through your normal launcher.

The gates are registered as script-engine functions, so **no file is patched** — the
executable on disk stays untouched.

Verify the installation from chat:

```
/run print(SetTaxiBenchmarkMode("Ping"))       -- unit gate, prints true
/run print(IsOnTournamentRealm("GetVersion"))  -- SQL gate, prints 0 4 1
```

---

## Detection and risk

There is **no technical measure that guarantees a client-side tool cannot be detected**.
Anyone claiming otherwise is wrong. What this project does and does not do:

| Aspect | Status |
|---|---|
| Modifies the executable on disk | **No.** The executable is never patched. |
| Adds files next to the executable | Yes — the four DLLs. |
| Loads at startup under a system library name | Yes, `version.dll` is a proxy. |
| Reads client memory | Yes. Every data command is a read of client structures. |
| Writes client memory | **Yes, briefly and narrowly.** The loader rewrites three script-table function pointers at startup, and the client's own "protected action allowed" flag is set and restored around target/focus/assist/cast calls (`ArmNextTarget`, `SetTargetCore`). No game data is written. |
| Sends synthetic input | One key press only: `InteractKey` / `InteractIn`, and only when the current target is a dead, lootable corpse within 5 yards. There is no generic key press, no key hold and no movement control. The key must be bound to `INTERACTTARGET`; bind it once per session with `GetTaxiBenchmarkMode("CTRL-SHIFT-X", "INTERACTTARGET", "bind")`, no addon needed. |
| Automates gameplay by itself | **No.** The library has no timers of its own, no combat logic and no background loop; every command runs only when an addon calls it. |

Practical precautions, in order of importance:

1. **Do not automate combat or other player actions.** Input automation is the behaviour
   most likely to be flagged, and it is not what this library is for.
2. **Keep the binaries unmodified.** Do not add strings, renames or extra exports — a
   scanner looks for known signatures, and custom builds make things worse, not better.

None of these is a guarantee. They only reduce the surface.

---

## What is not included

This release contains binaries and documentation only.

* No source code.
* No addons. The gates are called directly from your own script.
* No support for other client builds.

---

## Notes and caveats

* **Single threaded.** The gates must be called from the client's script thread, which in
  practice means from addon code and event handlers. Do not call them from other threads.
* **Protected actions.** Casting and interaction are protected by the client. The library
  cannot trigger them directly; `InteractKey` sends a real key press, which the client
  accepts.
* **Memory layout.** Every offset is specific to the supported build. A client update will break
  the libraries and they will need to be rebuilt.
* **Stability.** Memory reads are guarded, but a wrong pointer can still crash the
  client. Test addons in a safe location first.
* **SQLite files.** `H("Open", path)` resolves the path relative to the client folder.
  Use `H("OpenAddon", addonName .. string.char(1) .. path)` for files shipped inside an addon.

---

## License

Non-commercial use only. See [LICENSE](LICENSE).

---

## Disclaimer

This project is not affiliated with, endorsed by, or sponsored by the game publisher or
any server operator. All trademarks belong to their respective owners. Use at your own
risk.
