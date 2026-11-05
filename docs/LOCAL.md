# Local workspace (this PC)

Russian version: [LOCAL.ru.md](LOCAL.ru.md)

`docs/workspace.md` describes the **original 2024 workspace** (`E:\FarCryCoop`). Drive `E:` is present on
this PC but holds no `FarCryCoop` folder, so that layout is history. The live workspace is this repository
plus the game folders below. Golden rule 2 in [CLAUDE.md](../CLAUDE.md) was already rewritten to point
here. The REA / Ghidra / JDK toolchain, however, is **not** inside the workspace — see the toolchain rows
below, [workspace.md](workspace.md) and [docs/tools/rea.md](tools/rea.md).

| Path | Role |
|---|---|
| `D:\Projects\Co-op Anything\FarCryCoop\` | This fork (repo root, notes, scripts, `local/`) |
| `D:\Projects\Co-op Anything\coop-anything\` | Upstream hub repo (issue tracker, `games/<slug>/README.md`) |
| `D:\Projects\Co-op Anything\FarCry3Coop-bantila\` | Clone of the owner's earlier Bantila work (1.05 Ghidra notes) |
| `H:\Uplay\Far Cry 3\` | Real Uplay install — read-only, never modified (golden rule 2 in `CLAUDE.md`; rule 2 in the hub `AGENTS.md`) |
| `H:\FarCryCoop\TestFC3\` | Working copy of the 1.06 build: disassembly probes, menu runs, test mods |
| `H:\FarCryCoop\games\far-cry-3\tools\` | Scratch copy of analysis scripts used against the test copy |
| `local/` (ignored) | Owner-specific private state: registry backups, UI helper, `local/scratch/` drafts |
| `C:\tools\ghidra_12.1.4_PUBLIC\` | Ghidra 12.1.4 used by REA (`GHIDRA_INSTALL_DIR`); its archive `ghidra_12.1.4_PUBLIC_20260921.zip` (544 MB) is kept next to it |
| `C:\tools\ghidra_12.2_PUBLIC\` | Ghidra 12.2 (879 MB) — **not** used by REA 6.8.0 (12.1.x-only policy); awaiting the owner's delete decision |
| `C:\tools\jdk-21.0.13+9\` | Eclipse Temurin JDK 21.0.13+9 (`JAVA_HOME`) |
| `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents\` | REA 6.8.0 (global npm install); the MCP server runs `scripts\rea.mjs` |
| `C:\Users\USER\.pi\agent\mcp.json` | pi's MCP config: the `rea` server entry with `GHIDRA_INSTALL_DIR` and `JAVA_HOME`; REA's backups `mcp.json.rea.backup*` live there too |
| `C:\Users\USER\.agents\skills\reverse-engineer-anything\` | REA's own skill, version 49 (`cli_commands 175`, `mcp_tools 229`, `mcp_prompts 6`) |

## Known deviations from the golden rules

- `env.ps1` is not present in this repo root, and `E:\FarCryCoop\env.ps1` does not exist on this PC.
  `JAVA_HOME` (`C:\tools\jdk-21.0.13+9`) and `GHIDRA_INSTALL_DIR` (`C:\tools\ghidra_12.1.4_PUBLIC`) are
  set **user-wide** (`HKCU\Environment`) to `C:\tools` instead of being redirected into the workspace.
  Ghidra settings and cache stay in Ghidra's default location: `support\launch.properties` in
  `C:\tools\ghidra_12.1.4_PUBLIC` was not modified. `TEMP`/`TMP` are the Windows default
  (`%USERPROFILE%\AppData\Local\Temp`) and the npm global prefix is `%APPDATA%\npm` — checked with
  `reg query HKCU\Environment`, `npm root -g`.
- REA 6.8.0, its skill, Ghidra and the JDK are installed on C: (`C:\tools`, `%APPDATA%\npm`,
  `.agents\skills`). That contradicts golden rule 2 ("never touch C:") and needs the owner's decision:
  either move the toolchain under the workspace (`Tools\`, `Data\`) or rewrite the rule to allow a
  system toolchain location.
- The REA registration itself is also a write to C:: `C:\Users\USER\.pi\agent\mcp.json` (2026-11-03,
  owner approved; see [docs/tools/rea.md](tools/rea.md)).
- The agent harness writes its own temporary files and delegate output under
  `C:\Users\<user>\AppData\Roaming\pi-desktop\` (`tmp\.ctx-mode-*` scripts, delegate result files). This breaks
  the "never write to C:" rule; it needs an owner decision (harness setting or a workspace-local temp override).
- `H:\` is used for the game installs and the test copy because the game folders live there.
- Scratch artifacts go to `local/scratch/` (ignored), never into `games/<slug>/`; analysis scripts stay in
  `tools/` or `games/<slug>/tools/` (both ignored, see `.gitignore`).
- `git add -A` fails in this repo: an undeletable file named `nul` sits at `D:\Projects\Co-op Anything\nul`
  (`error: invalid path 'NUL'`). Always commit explicit paths.
