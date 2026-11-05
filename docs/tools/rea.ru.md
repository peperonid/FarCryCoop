# REA (Reverse Engineer Anything)

English version: [rea.md](rea.md)

- Репозиторий: https://github.com/morluto/rea (MIT). Пакет npm: `rea-agents`.
- Что это: MCP-сервер + CLI, дающий агенту инструменты реверс-инжиниринга (нативные бинарники через
  Ghidra / IDA / Hopper, JavaScript/Electron, .NET, сайты). Анализ выполняется локально.
- Проверенная версия: 6.8.0 (2026-11-03; `rea --version` -> `6.8.0`, установка `rea-agents` глобально
  через npm).
- История (предыдущий ПК): 6.7.0 (`server.json`, `package.json`, коммит `a25eb5fb`, 2026-10-18).

## Почему REA НЕ прописан в `.mcp.json` этого репозитория

Документация REA (`docs/installation.md`, раздел «One project») советует не использовать
`--scope project`, который пишет `.mcp.json` в репозиторий, откуда его можно закоммитить или скопировать.
Вместо этого — пользовательский конфиг самого агента. Кроме того, REA имеет смысл только на ПК владельца:
там файлы игры и Ghidra, а не в облачной сессии. На этом ПК регистрация находится в `mcp.json` агента
**pi**, а не в репозитории.

## Установка и регистрация на этом ПК (2026-11-03)

Требования из `docs/windows-ghidra-p0.md` (поддержка Windows **экспериментальная**, только чтение):

- Windows 10+ x64, копия игры на локальном диске NTFS;
- Node.js 22.x (>=22.23.4), 24.x (>=24.22.1) или 26.12.1+;
- Ghidra 12.1.x, установленная вами (проверено на 12.1.4) — REA её никогда не ставит;
- 64-битный полный JDK 21 в пределах версий Java, объявленных самой установкой Ghidra.

Путь, который использован здесь. Claude Code на этом ПК не установлен, поэтому REA зарегистрирован для
агента **pi**:

```powershell
npm install -g rea-agents      # 6.8.0; обёртки %APPDATA%\npm\rea и %APPDATA%\npm\rea.cmd
rea --version                  # 6.8.0
rea setup --client pi --yes    # без --yes тот же запрос задаётся интерактивно
rea doctor --client pi --provider ghidra
```

`rea setup` переписал только запись `rea` в `C:\Users\<user>\.pi\agent\mcp.json`; серверы
`coop-anything` и `supermemory` в этом же файле не тронуты. Там же он поставил собственный навык REA.
Рядом с конфигом созданы резервные копии: `mcp.json.rea.backup` (снимок без записи `rea`) и
`mcp.json.rea.backup.<64-hex>.recovery` (снимок с записью `rea`, но без блока `env`). Их отметки
времени — 2026-11-02 17:23 и 2026-11-02 17:59; рабочий конфиг записан 2026-11-03 11:40, то есть блок
`env` появился только после установки Ghidra/JDK на этом ПК.

## Зарегистрировано на этом ПК (2026-11-03)

Проверенное содержимое `C:\Users\USER\.pi\agent\mcp.json`:

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

Инструменты, на которые указывает эта запись (проверено на этом ПК):

| Компонент | Версия | Путь |
|---|---|---|
| REA `rea-agents` | 6.8.0 | `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents` (скрипт `scripts\rea.mjs`) |
| Навык REA `reverse-engineer-anything` | 49 | `C:\Users\USER\.agents\skills\reverse-engineer-anything` |
| Ghidra | 12.1.4 | `C:\tools\ghidra_12.1.4_PUBLIC` |
| Eclipse Temurin JDK | 21.0.13+9 | `C:\tools\jdk-21.0.13+9` |
| Node.js | v22.23.4 | `C:\Program Files\nodejs\node.exe` |

`rea doctor --client pi --provider ghidra`: `healthy: true`, `environment_healthy: true`; провайдер
`ghidra` — `configured: true`, `available: true`, `providerVersion 12.1.4`; все 9 проверок ghidra в
порядке: установка, версия 12.1.4, headless `support\analyzeHeadless.bat`, `native_decompiler`
(`Ghidra\Features\Decompiler\os\win_x86_64\decompile.exe` сообщает `pe x86_64`, что подходит для
win32/x64), java (`21.0.13; 64-bit; JDK; C:\tools\jdk-21.0.13+9; accepted a 64-bit full JDK 21 or
newer`), `native_authority`. `rea doctor --client pi` тоже здоров и сообщает каталог
`cli_commands 175`, `mcp_tools 229`, `mcp_prompts 6`, навык `installed_version 49` /
`installed_tool_count 229` / `state aligned`, регистрацию `pi` -> `C:\Users\USER\.pi\agent\mcp.json`
со статусом `state: aligned`.

- ida, jadx и kuna помечены как `missing_analysis_engine`: REA их никогда не скачивает; нужны
  `REA_IDA_MCP_CONFIG`, `REA_JADX_MCP_JAR` и `REA_KUNA_COMMAND`. Для этого проекта они не нужны.

## Политика версий и одна ловушка (проверено на этом ПК)

- REA 6.8.0 принимает только линию Ghidra **12.1.x**: в `rea-agents\dist\ghidra\GhidraReleasePolicy.js`
  заданы `SUPPORTED_GHIDRA_VERSION = "12.1.4"` и `SUPPORTED_GHIDRA_JAVA_MAJOR = 21`. Ghidra 12.2
  (`Ghidra\application.properties`: `application.version=12.2`, `application.java.min=25`) REA 6.8.0
  отвергает — именно из-за этого расхождения и перепроверялась документация. `GHIDRA_INSTALL_DIR` должен
  указывать на `C:\tools\ghidra_12.1.4_PUBLIC`.
- Java принимается в пределах тех границ, которые объявляет сама установка Ghidra
  (`application.java.min` / `application.java.max`); у Ghidra 12.1.4 `application.java.min=21`,
  `application.java.max=` (пусто).
- Ловушка: проверки провайдера ghidra читают окружение **процесса CLI**. Изменения `setx` / HKCU не
  видны уже запущенным процессам, поэтому после смены переменных нужно перезапустить оболочку и агента
  (в том числе MCP-сервер `rea` в pi). `rea setup` копирует обнаруженное окружение провайдера в запись
  клиента, поэтому `mcp.json` несёт `GHIDRA_INSTALL_DIR` и `JAVA_HOME` независимо от окружения ОС.
- Пользовательские переменные (`HKCU\Environment`) на этом ПК:
  `GHIDRA_INSTALL_DIR=C:\tools\ghidra_12.1.4_PUBLIC`, `JAVA_HOME=C:\tools\jdk-21.0.13+9`.
  Машинной переменной `JAVA_HOME` нет.

## Вариант для Claude Code (Claude Code на этом ПК не установлен)

Проверено здесь: `where claude` не находит программу, `C:\Users\USER\.claude` и
`C:\Users\USER\.claude.json` не существуют. Этот раздел и два исторических раздела ниже — запись
предыдущего ПК, а не рабочий путь на этом ПК.

```powershell
Remove-Item Env:CLAUDE_CONFIG_DIR -ErrorAction SilentlyContinue
claude mcp add --scope local rea -- cmd.exe /d /c npx -y rea-agents@6.7.0 mcp
$env:GHIDRA_INSTALL_DIR = "C:\tools\ghidra_12.1.4_PUBLIC"   # ваш путь
$env:JAVA_HOME = "C:\Program Files\Java\jdk-21"              # ваш путь
npx -y rea-agents@6.7.0 doctor --json
claude mcp list
```

Локальная область хранит запись в `%USERPROFILE%\.claude.json` только для этой папки. Навык (skill) REA
при этом не ставится. Удаление: `claude mcp remove --scope local rea` из той же папки.
Альтернатива: `npx rea-agents setup` регистрирует REA для всех проектов (user scope) и ставит навык.

## Зарегистрирован на ПК владельца (2026-10-18) — история предыдущего ПК

Владелец разрешил одну запись в `C:\Users\<user>\.claude.json` (локальная область, только эта папка). Все пути инструментов вели в `E:\FarCryCoop` (такой папки на этом ПК нет):

```powershell
$R='E:\FarCryCoop'
claude mcp add --scope local rea -e "GHIDRA_INSTALL_DIR=$R\Tools\ghidra_12.1.4_PUBLIC" -e "JAVA_HOME=$R\Tools\jdk-21.0.12.1+1" `
  -e "TEMP=$R\Temp" -e "TMP=$R\Temp" -e "npm_config_cache=$R\Data\npm-cache" `
  -e "XDG_CONFIG_HOME=$R\Data\xdg-config" -e "XDG_CACHE_HOME=$R\Data\xdg-cache" `
  -- cmd.exe /d /c "$R\Tools\node-v24.22.0-win-x64\npx.cmd" -y rea-agents@6.7.0 mcp
```

Имя сервера должно идти до опций `-e` (`-e` принимает несколько значений). `claude mcp get rea` показывал `Connected`; инструменты появлялись в новой сессии Claude Code. `rea doctor`: Ghidra 12.1.4 и JDK 21 в порядке; Hopper, IDA, jadx, Kuna не установлены (не нужны).

## Перенесён в пользовательскую область (2026-10-22) — история предыдущего ПК

Решение владельца: по репозиторию на игру, поэтому REA зарегистрирован для всех папок. Локальная
запись удалена (`claude mcp remove rea -s local`), та же команда выполнена с `--scope user` вместо
`--scope local`. Тот же файл `C:\Users\<user>\.claude.json`, теперь его общий раздел; все пути
по-прежнему вели в `E:\FarCryCoop`. `claude mcp get rea` из `E:\FarCryCoop\FarCry3Coop`:
`Scope: User config`, `Connected`. Удаление: `claude mcp remove rea -s user`. 2026-11-03 владелец
разрешил эквивалентную запись на этом ПК, и она попала в `C:\Users\USER\.pi\agent\mcp.json`
(см. раздел «Зарегистрировано на этом ПК» выше).

## Правила, которые нас касаются

- Золотое правило 2 в `CLAUDE.md` (и правило 2 в хабовом `AGENTS.md`): анализируем только **копию** папки игры, не установку.
- `DISCLAIMER.md` REA: проект не поддерживает «circumventing DRM, license checks, anti-cheat systems or
  other technical protection measures» (обход DRM, проверок лицензии, античитов и других мер защиты) и
  «developing game cheats or tampering with online services» (создание читов и вмешательство в
  онлайн-сервисы). Та же граница, что и правило 10 («Stay fair») в хабовом `AGENTS.md`
  (`D:\Projects\Co-op Anything\coop-anything\AGENTS.md`).
- Windows Ghidra P0 работает только на чтение: REA не меняет байты исполняемых файлов.

## Неизвестно / догадка

- Догадка, не проверено: `FC_m64d3d12.dll` защищена Denuvo и VMProtect, поэтому статический анализ
  в Ghidra может показать мало полезного кода. Неизвестно, пока не попробуем на копии владельца.
