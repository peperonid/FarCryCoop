# REA (Reverse Engineer Anything)

Russian version: [rea.ru.md](rea.ru.md)

- Repository: https://github.com/morluto/rea (MIT). npm package: `rea-agents`.
- What it is: one MCP server + CLI that gives an agent reverse-engineering tools (native binaries via
  Ghidra / IDA / Hopper, JavaScript/Electron, .NET, websites). Analysis runs locally.
- Version checked: 6.8.0 (2026-11-03; `rea --version` -> `6.8.0`, global npm install of `rea-agents`).
- History (previous PC): 6.7.0 (`server.json`, `package.json`, commit `a25eb5fb`, 2026-10-18).

## Why it is NOT registered in this repository's `.mcp.json`

REA's own docs (`docs/installation.md`, "One project") say: avoid `--scope project`, which writes
`.mcp.json` into the repository where it can be committed or copied. Use the agent's own user-level
config instead. It also only makes sense on the owner's PC: the game files and Ghidra are there, not in
the cloud session. On this PC the registration lives in the **pi** agent's `mcp.json`, not in the repo.

## Install and register on this PC (2026-11-03)

Requirements from `docs/windows-ghidra-p0.md` (Windows support is **experimental**, read-only):

- Windows 10+ x64, game copy on a local NTFS drive;
- Node.js 22.x (>=22.23.4), 24.x (>=24.22.1) or 26.12.1+;
- Ghidra 12.1.x installed by you (verified with 12.1.4) — REA never installs it;
- 64-bit full JDK 21 inside the Java range declared by that Ghidra installation.

Route used here. Claude Code is not installed on this PC, so REA is registered for the **pi** agent:

```powershell
npm install -g rea-agents      # 6.8.0; shims at %APPDATA%\npm\rea and %APPDATA%\npm\rea.cmd
rea --version                  # 6.8.0
rea setup --client pi --yes    # without --yes it asks the same approval interactively
rea doctor --client pi --provider ghidra
```

`rea setup` rewrote only the `rea` entry in `C:\Users\<user>\.pi\agent\mcp.json`; the `coop-anything`
and `supermemory` servers in the same file were left untouched. It also installed REA's own skill. It
wrote backups next to the config: `mcp.json.rea.backup` (snapshot without the `rea` entry) and
`mcp.json.rea.backup.<64-hex>.recovery` (snapshot with the `rea` entry but without the `env` block).
Their timestamps are 2026-11-02 17:23 and 2026-11-02 17:59; the live config was written 2026-11-03
11:40, i.e. the `env` block appeared only after the Ghidra/JDK install on this PC.

## Registered on this PC (2026-11-03)

Verified content of `C:\Users\USER\.pi\agent\mcp.json`:

```json
"rea": {
  "type": "stdio",
  "command": "C:\\Program Files\\nodejs\\node.exe",
  "args": [
    "C:\\Users\\USER\\AppData\\Roaming\\npm\\node_modules\\rea-agents\\scripts\\rea.mjs",
    "mcp"
  ],
  "env": {
    "GHIDRA_INSTALL_DIR": "C:\\tools\\ghidra_12.1.4_PUBLIC",
    "JAVA_HOME": "C:\\tools\\jdk-21.0.13+9"
  }
}
```

Toolchain that entry points to (verified on this PC):

| Component | Version | Path |
|---|---|---|
| REA `rea-agents` | 6.8.0 | `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents` (script `scripts\rea.mjs`) |
| REA skill `reverse-engineer-anything` | 49 | `C:\Users\USER\.agents\skills\reverse-engineer-anything` |
| Ghidra | 12.1.4 | `C:\tools\ghidra_12.1.4_PUBLIC` |
| Eclipse Temurin JDK | 21.0.13+9 | `C:\tools\jdk-21.0.13+9` |
| Node.js | v22.23.4 | `C:\Program Files\nodejs\node.exe` |

`rea doctor --client pi --provider ghidra`: `healthy: true`, `environment_healthy: true`; provider
`ghidra` `configured: true`, `available: true`, `providerVersion 12.1.4`; all 9 ghidra checks ok —
installation, version 12.1.4, headless `support\analyzeHeadless.bat`, `native_decompiler`
(`Ghidra\Features\Decompiler\os\win_x86_64\decompile.exe` reports `pe x86_64`, matches win32/x64),
java (`21.0.13; 64-bit; JDK; C:\tools\jdk-21.0.13+9; accepted a 64-bit full JDK 21 or newer`),
`native_authority`. `rea doctor --client pi` is healthy as well and reports catalog `cli_commands 175`,
`mcp_tools 229`, `mcp_prompts 6`, skill `installed_version 49` / `installed_tool_count 229` /
`state aligned`, registration `pi` -> `C:\Users\USER\.pi\agent\mcp.json` `state: aligned`.

- ida, jadx and kuna report `missing_analysis_engine`: REA never downloads them; they need
  `REA_IDA_MCP_CONFIG`, `REA_JADX_MCP_JAR` and `REA_KUNA_COMMAND`. Not needed for this project.

## Version policy and one pitfall (verified on this PC)

- REA 6.8.0 accepts the Ghidra **12.1.x** line only: `rea-agents\dist\ghidra\GhidraReleasePolicy.js`
  sets `SUPPORTED_GHIDRA_VERSION = "12.1.4"` and `SUPPORTED_GHIDRA_JAVA_MAJOR = 21`. Ghidra 12.2
  (`Ghidra\application.properties`: `application.version=12.2`, `application.java.min=25`) is rejected
  by REA 6.8.0 — that mismatch is what started this documentation pass. `GHIDRA_INSTALL_DIR` must point
  at `C:\tools\ghidra_12.1.4_PUBLIC`.
- Java is accepted within that Ghidra installation's own bounds (`application.java.min` /
  `application.java.max`); Ghidra 12.1.4 declares `application.java.min=21`, `application.java.max=`
  (empty).
- Pitfall: the ghidra provider checks read the **CLI process** environment. `setx` / HKCU changes are
  not visible to processes that are already running, so after changing the variables restart the shell
  and the agent (the pi `rea` MCP server included). `rea setup` copies the detected provider env into
  the client registration, so `mcp.json` carries `GHIDRA_INSTALL_DIR` and `JAVA_HOME` regardless of the
  OS environment.
- User-level (`HKCU\Environment`) variables on this PC: `GHIDRA_INSTALL_DIR=C:\tools\ghidra_12.1.4_PUBLIC`,
  `JAVA_HOME=C:\tools\jdk-21.0.13+9`. There is no machine-level `JAVA_HOME`.

## Claude Code variant (Claude Code is not installed on this PC)

Verified here: `where claude` fails, and `C:\Users\USER\.claude` and `C:\Users\USER\.claude.json` do
not exist. This section and the two history sections below are the record of the previous PC, not a
runnable route on this one.

```powershell
Remove-Item Env:CLAUDE_CONFIG_DIR -ErrorAction SilentlyContinue
claude mcp add --scope local rea -- cmd.exe /d /c npx -y rea-agents@6.7.0 mcp
$env:GHIDRA_INSTALL_DIR = "C:\tools\ghidra_12.1.4_PUBLIC"   # your path
$env:JAVA_HOME = "C:\Program Files\Java\jdk-21"              # your path
npx -y rea-agents@6.7.0 doctor --json
claude mcp list
```

Local scope stores the entry in `%USERPROFILE%\.claude.json` for this folder only. It does not install
REA's skill. Remove with `claude mcp remove --scope local rea` from the same folder.
Alternative: `npx rea-agents setup` registers REA for every project (user scope) and installs the skill.

## Registered on the owner's PC (2026-10-18) — history of the previous PC

Owner approved the one write to `C:\Users\<user>\.claude.json` (local scope, this folder only). All tool paths pointed to `E:\FarCryCoop` (that folder does not exist on this PC):

```powershell
$R='E:\FarCryCoop'
claude mcp add --scope local rea -e "GHIDRA_INSTALL_DIR=$R\Tools\ghidra_12.1.4_PUBLIC" -e "JAVA_HOME=$R\Tools\jdk-21.0.12.1+1" `
  -e "TEMP=$R\Temp" -e "TMP=$R\Temp" -e "npm_config_cache=$R\Data\npm-cache" `
  -e "XDG_CONFIG_HOME=$R\Data\xdg-config" -e "XDG_CACHE_HOME=$R\Data\xdg-cache" `
  -- cmd.exe /d /c "$R\Tools\node-v24.22.0-win-x64\npx.cmd" -y rea-agents@6.7.0 mcp
```

The server name must come before the `-e` options (`-e` takes several values). `claude mcp get rea` showed `Connected`; the tools appeared in a new Claude Code session. `rea doctor`: Ghidra 12.1.4 and JDK 21 healthy; Hopper, IDA, jadx, Kuna not installed (not needed).

## Moved to user scope (2026-10-22) — history of the previous PC

Owner decision: one repository per game, so REA was registered for every folder. The local entry was
removed (`claude mcp remove rea -s local`) and the same command was run with `--scope user` instead
of `--scope local`. Same file `C:\Users\<user>\.claude.json`, now its user section; all paths still
pointed to `E:\FarCryCoop`. `claude mcp get rea` from `E:\FarCryCoop\FarCry3Coop`: `Scope: User config`,
`Connected`. Remove with `claude mcp remove rea -s user`. On 2026-11-03 the same approval was given
for the equivalent write on this PC, which landed in `C:\Users\USER\.pi\agent\mcp.json` (see
"Registered on this PC" above).

## Rules that apply to us

- Golden rule 2 in `CLAUDE.md` (and rule 2 in the hub `AGENTS.md`): analyse only a **copy** of the game folder, never the install.
- REA's `DISCLAIMER.md` says the project does not support "circumventing DRM, license checks,
  anti-cheat systems or other technical protection measures" or "developing game cheats or tampering
  with online services". Same boundary as rule 10 ("Stay fair") in the hub `AGENTS.md`
  (`D:\Projects\Co-op Anything\coop-anything\AGENTS.md`).
- Windows Ghidra P0 is read-only: REA does not change executable bytes.

## Unknown / guess

- Guess, not verified: `FC_m64d3d12.dll` is protected by Denuvo and VMProtect, so static Ghidra
  analysis may show little useful code. Unknown until tried on the owner's copy.
