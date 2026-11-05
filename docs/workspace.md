# Workspace

Russian version: [workspace.ru.md](workspace.ru.md)

Note: the canonical `E:\FarCryCoop` layout below is the **history of the previous PC**. On this PC the
drive `E:` exists but has no `FarCryCoop` folder and no `env.ps1`. The live workspace layout is
documented in [LOCAL.md](LOCAL.md) (+ [LOCAL.ru.md](LOCAL.ru.md)); the toolchain that is actually
installed here is in "Toolchain on this PC" below.

## Canonical workspace layout (history of the previous PC)

All work happened in `E:\FarCryCoop` (golden rule 2 in [CLAUDE.md](../CLAUDE.md), which now points to
`D:\Projects\Co-op Anything\FarCryCoop`). Environment was loaded first: `. E:\FarCryCoop\env.ps1`.

| Folder | Contents |
|---|---|
| `Downloads\<name>\` | Original archives, one folder each, SHA-256 checked |
| `Tools\` | Unpacked portable tools |
| `Data\` | Tool settings and caches (Ghidra settings/cache, npm cache, npm global prefix) |
| `Temp\` | Temporary files (`TEMP`/`TMP` point here) |
| `FarCryCoop\`, `FarCry<N>Coop\` | Repositories, one folder per game |

## Installed tools (2026-10-18) — history of the previous PC

| Tool | Version | Source | SHA-256 (archive) |
|---|---|---|---|
| Node.js | v24.22.0 LTS (win-x64 zip) | https://nodejs.org/dist/v24.22.0/ | `f9683a19877e12a9b167305c8db71040d812876359de21f0d2e97c3f321f80c3` |
| Eclipse Temurin JDK | 21.0.12.1+1 (x64 zip) | https://github.com/adoptium/temurin21-binaries/releases/tag/jdk-21.0.12.1%2B1 | `f9d6e191ab098c0d416e7d588a24420a8621cd2f4720dab2459b8b7b2d2d8b4e` |
| Ghidra | 12.1.4 (`ghidra_12.1.4_PUBLIC_20260921.zip`) | https://github.com/NationalSecurityAgency/ghidra/releases/tag/Ghidra_12.1.4_build | `ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db` |

Checksums came from nodejs.org `SHASUMS256.txt`, the Adoptium API and the Ghidra GitHub release;
all three downloads matched.

Ghidra `support\launch.properties` was changed there (original kept as `launch.properties.orig`):
`JAVA_HOME_OVERRIDE`, `application.settingsdir`, `application.cachedir`, `application.tempdir`,
`java.io.tmpdir` pointed into `E:\FarCryCoop`. Check run: `analyzeHeadless.bat` printed its usage and
created its settings/cache folders under `E:\FarCryCoop`. No such redirect exists on this PC:
`C:\tools\ghidra_12.1.4_PUBLIC\support\launch.properties` is unmodified (its date is 2026-09-21,
`JAVA_HOME_OVERRIDE=` is still empty, no `launch.properties.orig` next to it), and Ghidra's settings and
cache live in Ghidra's default location (`C:\Users\USER\AppData\Roaming\ghidra` exists).

## Toolchain on this PC (2026-11-03)

| Tool | Version | Path | Source / evidence | SHA-256 (archive) |
|---|---|---|---|---|
| REA (`rea-agents`) | 6.8.0 | `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents` (script `scripts\rea.mjs`) | global npm install; `rea --version` -> 6.8.0 | not recorded |
| REA skill `reverse-engineer-anything` | 49 | `C:\Users\USER\.agents\skills\reverse-engineer-anything` | installed by `rea setup` | not applicable |
| Ghidra | 12.1.4 (`ghidra_12.1.4_PUBLIC_20260921.zip`) | `C:\tools\ghidra_12.1.4_PUBLIC` | https://github.com/NationalSecurityAgency/ghidra/releases/tag/Ghidra_12.1.4_build | `ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db` (re-checked 2026-11-03 with `sha256sum` on the 569,649,598-byte archive) |
| Eclipse Temurin JDK | 21.0.13+9 | `C:\tools\jdk-21.0.13+9` | `release` file in that JDK: `IMPLEMENTOR="Eclipse Adoptium"`, `IMPLEMENTOR_VERSION="Temurin-21.0.13+9"`, `JAVA_VERSION="21.0.13"`, `JAVA_VERSION_DATE="2026-10-20"` | not recorded |
| Node.js | v22.23.4 | `C:\Program Files\nodejs\node.exe` | system install; satisfies REA's Node 22.23.4+ requirement | not recorded |

`node -v` -> v22.23.4, `npm -v` -> 10.9.9. `C:\tools\jdk-21.0.13+9\bin\java -version` ->
`openjdk version "21.0.13" 2026-10-20 LTS` / `Temurin-21.0.13+9 (build 21.0.13+9-LTS)`. A plain `java`
resolved through PATH on this PC is the Oracle shim `C:\Program Files (x86)\Common Files\Oracle\Java\java8path\java.exe`
(`java version "1.8.0_511"`), so PATH does not select the JDK 21 — `JAVA_HOME` does, and that is what REA
reads (see [docs/tools/rea.md](tools/rea.md)). Only version strings and paths were verified for REA, the
JDK and Node.js on this PC; no archive hash was recorded for them (unlike the Ghidra archive, which is
still on disk).

`C:\tools` also holds `ghidra_12.1.4_PUBLIC_20260921.zip` (544 MB archive, kept) and
`ghidra_12.2_PUBLIC` (879 MB). **Ghidra 12.2 is not used by REA:** REA 6.8.0 accepts the 12.1.x line
only, and 12.2 declares `application.java.min=25` (see [docs/tools/rea.md](tools/rea.md)). The owner has
not yet decided whether to delete it.

User-level environment variables (`HKCU\Environment`) on this PC:

| Variable | Value |
|---|---|
| `GHIDRA_INSTALL_DIR` | `C:\tools\ghidra_12.1.4_PUBLIC` |
| `JAVA_HOME` | `C:\tools\jdk-21.0.13+9` |

There is no machine-level `JAVA_HOME`. `TEMP` and `TMP` are the Windows default
(`%USERPROFILE%\AppData\Local\Temp`): nothing is redirected into a workspace.

## Known limits of "never touch C:"

- Windows, PowerShell and the system `git.exe` (`C:\Program Files\Git`) live on C:. They are used as
  installed; our own scripts write nothing there.
- The REA client-config write happened on 2026-11-03 in `C:\Users\USER\.pi\agent\mcp.json`: the owner
  approved the equivalent of the old `%USERPROFILE%\.claude.json` write. `claude mcp add` was not used
  on this PC because Claude Code is not installed here. REA keeps its own backups
  (`mcp.json.rea.backup*`) in that same folder.
- REA 6.8.0 (npm global prefix), its skill, Ghidra 12.1.4 and Temurin JDK 21.0.13+9 are installed on C:
  (`C:\tools`, `C:\Users\USER\AppData\Roaming\npm`, `C:\Users\USER\.agents\skills`), and
  `GHIDRA_INSTALL_DIR`/`JAVA_HOME` point into `C:\tools`. That contradicts golden rule 2 (nothing on
  C:) and needs the owner's decision: either move the toolchain under the workspace (`Tools\`, `Data\`)
  or rewrite the rule to allow a system toolchain location.
- Ghidra settings and cache are not redirected either: `support\launch.properties` in
  `C:\tools\ghidra_12.1.4_PUBLIC` was not modified on this PC, so they stay in Ghidra's default
  location.
- `E:\FarCryCoop` and `E:\FarCryCoop\env.ps1` do not exist on this PC, so the "load the environment
  first" step of the old layout cannot be run here.
