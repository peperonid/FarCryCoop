# Local workspace (this PC)

Russian version: [LOCAL.ru.md](LOCAL.ru.md)

`docs/workspace.md` describes the **original 2024 workspace** (`E:\FarCryCoop`). That drive is not present on
this PC. The live workspace is this repository plus the game folders below. Golden rule 2 in
[CLAUDE.md](../CLAUDE.md) was already rewritten to point here.

| Path | Role |
|---|---|
| `D:\Projects\Co-op Anything\FarCryCoop\` | This fork (repo root, notes, scripts, `local/`) |
| `D:\Projects\Co-op Anything\coop-anything\` | Upstream hub repo (issue tracker, `games/<slug>/README.md`) |
| `D:\Projects\Co-op Anything\FarCry3Coop-bantila\` | Clone of the owner's earlier Bantila work (1.05 Ghidra notes) |
| `H:\Uplay\Far Cry 3\` | Real Uplay install — read-only, never modified (AGENTS.md rule 2) |
| `H:\FarCryCoop\TestFC3\` | Working copy of the 1.06 build: disassembly probes, menu runs, test mods |
| `H:\FarCryCoop\games\far-cry-3\tools\` | Scratch copy of analysis scripts used against the test copy |
| `local/` (ignored) | Owner-specific private state: registry backups, UI helper, `local/scratch/` drafts |

## Known deviations from the golden rules

- `env.ps1` is not present in this repo root, so TEMP/TMP, npm cache, `JAVA_HOME` and Ghidra settings are not
  redirected into the workspace.
- The agent harness writes its own temporary files and delegate output under
  `C:\Users\<user>\AppData\Roaming\pi-desktop\` (`tmp\.ctx-mode-*` scripts, delegate result files). This breaks
  the "never write to C:" rule; it needs an owner decision (harness setting or a workspace-local temp override).
- `H:\` is used for the game installs and the test copy because the game folders live there.
- Scratch artifacts go to `local/scratch/` (ignored), never into `games/<slug>/`; analysis scripts stay in
  `tools/` or `games/<slug>/tools/` (both ignored, see `.gitignore`).
- `git add -A` fails in this repo: an undeletable file named `nul` sits at `D:\Projects\Co-op Anything\nul`
  (`error: invalid path 'NUL'`). Always commit explicit paths.
