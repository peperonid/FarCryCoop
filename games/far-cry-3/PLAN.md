# Far Cry 3 — campaign co-op for 4 players: plan

Russian version: [PLAN.ru.md](PLAN.ru.md)

Status: plan, forked from Bantila/FarCryCoop 2026-11-01. Branch `game/far-cry-3`.

## Goal

Four players pass the whole single-player story of Far Cry 3 together, with cutscenes, side
activities (outposts, towers, hunting, Path of the Hunter, Wanted Dead, trials, races) and
collectibles (relics, letters, memory cards). Friends join through a **Steam invite**.

**Game versions:** both supported: **1.05** (Steam depot 220241, manifest `7362101836779063707`,
DLL build 2018-05-02, last version with multiplayer, what scubrah's mod is built on) and **1.06**
(current Steam / Ubisoft Connect, build 2022-10-03). Each version has its own addresses; a shared
address map is built by `scripts/fc3_addrmap.py`. Ubisoft servers for FC3 are shut down, so joining
goes only through our own transport.

## Architecture

**Transport A (engine netcode)** — the primary path, confirmed viable by Bantila's Phase 1 RE:

- The engine's `CFCXPlayerService` is a network-replicated component with a 4-slot player table.
  In single-player, slot 0 is the local player. `GetNumberOfPlayers()`, `GetPlayerInSlot()`,
  `CoopTeleportPlayer` all exist and are Lua-exposed.
- **Direct connect:** the game has a developer quick-launch path:
  `-host -login <name> -coop` (host) and `-client -login <name> -ip <addr>` (client).
  Session type 3 = local coop, no Ubisoft services needed.
- **Join path:** Storm LAN discovery (`storm::echo::lan`) exists in the engine but the PC game
  layer never uses it (only `_RdV` online providers). Options:
  - **A1:** ASI hooks the `_RdV` providers to answer locally; session created/joined by direct
    address via `-host`/`-client` or Storm LAN over a localhost relay (same pattern as Far Cry 1).
  - **A2:** local emulator of the RendezVous service.
- **Fallback (Transport B):** own UDP netcode in the ASI. Every player runs single-player; the ASI
  sends the host's state; joined players are puppets. More work, full control.

Parts in both cases:

- `FarCry3Coop.asi` loaded by Ultimate ASI Loader; hooks in `FC3.dll` / `FC3_d3d11.dll` by address,
  found with REA/Ghidra. Pattern: the Far Cry (2004) `FarCryCoopNet.asi`.
- Script layer: changed mission scripts in our own archive; original files are not replaced.
- **Steam invite:** the game has no `steam_api.dll`, so the Far Cry (2004) Steam bridge (Spacewar,
  AppID 480) is reused: "Join" in the Steam friends list, connection relayed through localhost.
  Direct IP stays as a fallback.
- Launcher and lobby: the Far Cry (2004) launcher (Rust + Tauri) gets a game profile, not a fork.

## Phases

0. **Game identity.** Done 2026-10-22 (Bantila), see [NOTES.md](NOTES.md).
1. **Research (read-only).** Done 2026-10-24 (Bantila). Unpacked data, found mission scripts,
   network layer, player service, co-op Lua API. See NOTES.md for full details.
2. **Spike, choose A or B.** Two game windows on one PC (host left, joined player right): both walk
   the island, see each other, shoot the same target. Transport A first: use `-host`/`-client`
   with session type 3 (local coop). If the island loads in a co-op session with story content, A
   is chosen. If story maps refuse to load in a co-op session, fall back to B.
   - **Check in game:** two windows, both players walk the island, see each other, shoot.
3. **Story scripts in co-op.** The single-player Domino scripts use `GetLocalPlayer` and know
   nothing about other players. Plan: patch mission scripts in our archive so that scripted events
   (forced animations, takedowns, vehicle seats, chases) apply to the host; other players are
   moved or watch. Non-scripted segments are free for everyone.
   - **Check in game:** the first three story missions together.
4. **Cutscenes.** The Vaas intro and other cutscenes need to play on all players. The engine's
   cutscene system (xbg/seq files) needs a replication layer: host triggers, all players play.
   - **Check in game:** the Vaas intro cutscene on all players.
5. **Side activities.** Outposts, towers, hunting, Path of the Hunter, Wanted Dead, trials, races.
   These are mostly area-based and should work once the co-op session is running. May need script
   patches for multi-player triggers.
   - **Check in game:** take an outpost together.
6. **Collectibles and progress.** Relics, letters, memory cards. A relic taken by a joined player
   must count and disappear for the host. Save format: the host's save is the source of truth.
   - **Check in game:** a relic taken by a joined player counts and disappears for the host.
7. **4-player scaling.** Four windows, one full mission. Check performance, lag, desync.
   - **Check in game:** 4 windows, one full mission.
8. **Steam invite and launcher.** Join through a Steam invite via the Spacewar bridge. The Rust+
   Tauri launcher gets an FC3 profile: game path, version detect, player list, "Start".
   - **Check in game:** join through a Steam invite.

## Key addresses (1.05, from Bantila's Phase 1)

| Symbol | Address | Notes |
|---|---|---|
| `GetLocalPlayerId` (Lua) | `0x11fbdcc4` (string), fn not yet found | Two xrefs: `0x111ce912` (console cmd), one in a large fn |
| `GetNumberOfPlayers` | `FUN_10681070` | No arg, returns count |
| `GetPlayerInSlot` | `FUN_10680ef0` | Slot to id string |
| `CoopGetCharacterName` | `FUN_10680f90` | Entity to string |
| `CoopTeleportPlayer` | reg xref `0x11b1d252` | Callback not decoded |
| `CFCXPlayerService` lookup | `FUN_1039e1f0` | Walks component array, type from `FUN_1039e0d0` |
| Slot-0 fallback | `FUN_11305eb0` | Uses local-player provider `DAT_1280437c` |
| Session object creation | `FUN_10260770` | Not decompiled |
| `SpawnCoopAI` / `OnSpawnPlayer_%u` | `0x11fc1c64` | Not decoded |
| Net registration | `FUN_11839b60`, `FUN_10468480` | `NetObject.FCXPlayerService`, hide/show messages |
| `-host` launcher | `FUN_106518ba` | Needs `-login <name>`, reads `-host <map>`, game mode switches |
| `-client` launcher | `FUN_10651d10` | Needs `-login <name>` and `-ip <addr>` |
| `-offline` handler | `FUN_10680f27` | Picks a message mode |
| "Not enough players" check | `0x108c4b09` | NOP'd by mod 262 (calls at `0x108c4c6b`, `0x108c5c36`) |
| Debug main menu | `-offline` switch | Shows "COOP Story" / "Adversarial" / "UPlay" |
| Central game object | `DAT_12804acc` | 4529 references, `+0x68` = component container |
| Net config (by hash) | `02E1D1848EC84585.Profile.xml` | Ports 9000-9004, NAT traversal, LAN discovery |

## Risks

- Story scripts expect one Jason: forced animations, takedowns, vehicle seats, chases. Plan: the
  host is Jason in scripted parts; others are moved or watch for a short time.
- The save may be tied to the Ubisoft account: joined players may need state sync instead of a
  copied save.
- If the engine's network play needs Ubisoft servers for matchmaking, transport A needs direct
  join (as in Far Cry 2004) or the project falls back to B.
- 1.06 removed multiplayer code: different addresses, possibly missing functions. 1.05 is the
  primary target; 1.06 support is secondary.

## Prior art

- **Bantila/FarCryCoop** — the project this fork is based on. Phase 0-1 done, Phase 2 pending.
- **Far Cry 1 (2004) co-op** (same project, 0.2 beta) — working reference for ASI + launcher +
  Steam bridge architecture.
- **scubrah "Open World Co-op"** (Nexus 220) — alpha, no story missions. Proves co-op works on
  the FC3 island. ~2 GB download suggests a repacked world archive.
- **Nexus mod 262 "Solo Offline Coop"** — NOPs the "not enough players" check, changes playlist
  `minPlayersInRound` to 1. Reference for offline session startup.


## Candidate hook points for the ASI (added 2026-11-07, agent; 1.06 addresses — map to 1.05 before use)

From the read-only RE in `NOTES.md` (sections "Co-op-id install path, verified from raw bytes" and
"Store-writer attribution"). These are **candidates for the Phase 2 spike, not decisions**; 1.05 is
the primary target, so every address below is a 1.06 build address that must be re-located in 1.05
with `scriptsc3_addrmap.py` before any patch. The owner approves the actual hook before it is written.

Confirmed facts (1.06, `H:\FarCryCoop\TestFC3in\FC3.dll`, image base `0x10000000`):

- The co-op game/mode id is the hash `0x43521e6f`. It is also a literal in the mode-id table at
  `.data 0x1216a00c`, `0x1216aa40`, `0x1216aee4`, `0x12173ff4`.
- Two runtime globals hold the installed co-op id: `[0x12891ca0]` and `[0x12891ca8]` (both BSS, zero
  at load; `[0x12891ca4]`/`[0x12891cac]` are init stamps). They are written by the install routine
  `0x11279910` (stores at `0x1127995f` and `0x11279adf`).
- `0x11279910` has no direct trigger: it is reached by a virtual dispatch — vmethod `0x11a58700`
  (a slot in the dispatch table at `.rdata 0x1200ad00`) → caller `0x1127a0f0`/`0x11279fc0` → `0x11279910`.
- The ten globals `[0x12807738..0x1280775c]` are primed to `-1` at module init by `0x11b2a610` (an
  entry of the init table at `.rdata 0x11bd2b04` — NOT a vtable; see NOTES 2026-11-07). The only
  runtime writer of those globals is `0x117884f0`, which derives them from the packed field
  `[0x1217c688]`.

Candidate hooks to evaluate in Phase 2 (transport A: force a local co-op session and see if the
island loads with story content):

- **H1 — force the co-op id at startup.** Either (a) NOP the install guard and write `0x43521e6f`
  into `[0x12891ca0]`/`[0x12891ca8]` from the ASI after the engine is up, or (b) hook the install
  vmethod chain (`0x1200ad00` dispatch → `0x11279910`) and force the id. Chosen over reverse-driving
  the whole dispatch: writing the two globals is the smallest observable change.
- **H2 — (doubtful, likely drop).** `0x11b2a610` is a one-shot module-init stub (not a per-session
  method), so it does not run on session teardown — a reload re-priming `[0x12807738..0x1280775c]`
  is not the threat assumed when this hook was drafted. The co-op id globals `[0x12891ca0]/[0x12891ca8]`
  are a different set. Re-check only if a runtime trace shows `[0x12891ca0]` actually being cleared.

Open before either hook is real (tracked in NOTES): the caller of vmethod `0x11a58700` (owner of the
`.rdata 0x1200ad00` table is resolved — `CFCXPlaylistService`; `0x11bd2b04` is resolved — an init
table, not a vtable). Until the `0x11a58700` caller is known, the hooks are write-after-init
experiments, not stable detours.


## Hook status after sessions 5–6 (added 2026-11-08, agent; supersedes the H1 framing above)

NOTES 2026-11-08 (sessions 5 and 6, probes `p300`–`p315`) changed the read on the co-op switch. This
addendum overrides the H1 framing in the section above; the addresses there stay valid, the *choice of
variable to poke* is what is now wrong.

- **H1 is demoted: writing `[0x12891ca0]`/`[0x12891ca8]` (`0x43521e6f`) cannot switch the live mode.**
  Those two globals are function-local `static const` query-key constants (init-stamp bytes
  `[0x12891ca4]`/`[0x12891cac]`), each referenced by exactly 3 sites in `.text`, all inside the install
  routines `0x11279910` / `0x11279a2b`. No gameplay/dispatch code reads them by address. Scribbling them
  from an ASI only edits the constants the installer uses as lookup keys, so H1 is **inconclusive by
  construction** (the engine already keeps `0x43521e6f` there). Safe to leave as a diagnostic, but it is
  not the lever.
- **The live current-mode id is the mode-manager field `[this + 0x4c]`** (`this` in `ebx`), not a global.
  Dispatch at `FC3.dll 0x113405a0` reads it and compares against six mode-hash ids —
  `0xab4fc8e2, 0x43521e6f`(CO-OP)`, 0x9991daec, 0x6c2b3868, 0xf66ffb87, 0x2100ced7` (three chains at
  `0x113405a0`, `0x11340622`, `0x11340a25`). All six route to the main-menu / `IngameToLobby` block;
  `0x2100ced7` is the one id that also drives per-world loading textures (`CustomMap`,
  `UI\Multiplayer\Textures\Menu\Loading\Worlds\`).
- **The live co-op session state is the playlist-service container `[service + 0x8c0]`**, service =
  `CFCXPlaylistService` singleton (instance global `0x12863890`, getter `0x103d1640`). The co-op session is
  a `PrivateCoopPlaylist` (`0x11fab710`) entry in that container. `0x43521e6f` is referenced at 11 `.text`
  sites (the real consumers) — see NOTES session 5.

Revised Phase-2 lead (supersedes "force the co-op id at startup"):

- **H3 — new primary.** Runtime trace (owner's debugger, no static alternative found): break on the writer
  of mode-manager `[this + 0x4c]` to learn who sets the live mode and on what, and walk `[service + 0x8c0]`
  to read the live playlist container. This is the only path to forcing a local co-op session; the offline
  global pokes are exhausted.
- **H1 — keep only as a safe diagnostic.** Leave the install-globals write in the probe purely to confirm
  the chain runs; it is not expected to change behaviour.
- **Still blocked:** naming the five non-coop mode-hash ids needs the custom Dunia id-hash recovered
  (FNV/djb2/sdbm/murmur all rejected) or the `GameModesConfig.xml` name↔id table; not offline-crackable.


## Hook status after session 7 (added 2026-11-08, agent; refines H3 above - now static, not trace-only)

Session 7 exported pseudocode from the offline Ghidra project `FC3_106` (see NOTES.md session 7). This
upgrades H3: the switch mechanism is now pinned statically; the runtime trace shrinks to just grabbing
the mode-manager `this` pointer.

- **The "switch to mode" virtual method is `modeManager->vtable[+0x5c]`** (called as
  `(**(code **)(*param_1 + 0x5c))(&id)` inside `FUN_11340560`, body `[0x11340560,0x113409cd]`). The
  dispatcher reads `[this+0x4c]` (confirms the session 5-6 finding), remaps any of the six ids to the
  **lobby id `0x63ddbbcb`** when the destination string is `FUN_11788030("FCXMainMenu")`, then calls
  that virtual. `[this+0x74]` gates the per-world loading-texture branch.
- **Populate a co-op session without the UI:** `FUN_1127a0f0(service)` (service `this`) builds the
  `[service+0x8c0]` container via `FUN_1025f3f0`, then runs the installer `FUN_11279910`, which looks
  the co-op entry up **by key `0x43521e6f`** through `FUN_113224e0` and Adds it into the container with
  `container->vtable[+0x5c]`. Confirms `[service+0x8c0]` is live (populated at runtime, not dead).
- **Correction to the singleton note:** `FUN_103d1640` returns `&DAT_12863890` - the instance is the
  object AT `0x12863890` (C++ magic-static guarded by `0x1286388c`), NOT a pointer stored there. So
  `service this = module.base + (0x12863890 - 0x10000000)` directly; do not `readPointer()` it. Fixed in
  `scripts/fc3_h3_trace.js`.

Revised Phase-2 levers (both now static):
- **H3a - switch mode:** write `[modeManager+0x4c] = 0x43521e6f` and call `modeManager->vtable[+0x5c]
  (0x43521e6f)`. Need only the live `modeManager this` (one Frida read of `ebx` at `0x113405a0`).
- **H3b - populate session:** call `FUN_1127a0f0` (or the finer `FUN_113224e0`(0x43521e6f) then
  `container->vtable[+0x5c]`) on the playlist service to inject the co-op entry.
- Still blocked: naming the five sibling mode-hash ids (custom hash not recovered).

## Hook status after session 8 (added 2026-11-08, agent; OVERRIDES H3a above)

Sessions 8/8b/8c captured the live mode-manager (Frida on the DX9 `farcry3.exe`, working copy only) and
decompiled the switch virtual. Two hard results change H3a:

- **The live mode-manager primary vtable is `0x1200e5f8`; the dispatcher `FUN_11340560` is its `+0x60` slot,
  and `vtable[+0x5c]` is `FUN_1030f8a0`.** `FUN_1030f8a0` decompiles to a two-line id-swizzle:
  `[this+0x4c]=[this+0x48]; [this+0x48]=*param_2`. It loads no world and builds no session.
  **H3a (poke `[this+0x4c]` + call `vtable[+0x5c]`) is therefore a no-op for entering co-op** - harmless
  (no crash risk, so the live `switch go` test is unnecessary) but it cannot start a co-op session.
  The "switch" only records a target id into `[this+0x48]`; the real transition lives in the consumers of
  `[this+0x48]` and in whoever invokes the dispatcher-virtual `vtable[+0x60]` with `(param_2=&destId, ...)`.
- **The live manager is only reliably found via the dispatch hook** (`--watch` during a transition):
  the `[this+0x4c]`/`[this+0x48]` ids are transient and never one of the six; the object is NOT findable by
  a vtable-pointer scan (its primary vptr `0x1200e5f8` was never scanned; `0x1200e620` is a secondary
  subobject at primary+0x28). `[service+0x8c0]` (co-op playlist) stays `0x0` even inside a loaded single-player
  world, and the co-op UI is gated by the Ubisoft network service, which is unavailable offline (the game
  shows "the network game service is no longer available" and aborts the mode change).

Revised Phase-2 direction:
- **Do not pursue H3a.** The id-setter does not enter co-op.
- **H3b (populate) alone is likely insufficient too**: even with `[service+0x8c0]` built, no offline caller
  drives `[this+0x48]` into a co-op world because the netplay prerequisite is missing.
- **New primary lead H4: capture a real co-op join ONLINE** (owner plays a genuine 2-player co-op join while
  the trace is attached), record the dispatcher-virtual call sequence + the playlist-service/netplay state
  that accompanies it, and reconstruct that sequence for offline/4p. Static xrefs can't reach the transition
  entry (indirect vtable call), so runtime evidence of a *successful* join is the only reliable map.
- Secondary (optional): find consumers of `[this+0x48]` offline to see whether any single function performs
  the world/session bring-up given a co-op target id.
