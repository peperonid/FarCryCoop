# Локальная рабочая папка (этот ПК)

English version: [LOCAL.md](LOCAL.md)

`docs/workspace.md` описывает **исходную рабочую папку 2024 года** (`E:\FarCryCoop`). Диск `E:` на этом
ПК присутствует, но папки `FarCryCoop` на нём нет, поэтому эта структура — история. Фактическая рабочая
папка — этот репозиторий и папки игры, перечисленные ниже. Золотое правило 2 из [CLAUDE.md](../CLAUDE.md)
уже переписано на эти пути. Инструментарий REA / Ghidra / JDK при этом находится **вне** рабочей папки —
см. строки про инструментарий ниже, [workspace.ru.md](workspace.ru.md) и [docs/tools/rea.ru.md](tools/rea.ru.md).

| Путь | Назначение |
|---|---|
| `D:\Projects\Co-op Anything\FarCryCoop\` | Этот форк (корень репозитория: заметки, скрипты, `local/`) |
| `D:\Projects\Co-op Anything\coop-anything\` | Репозиторий-хаб (трекер задач, `games/<slug>/README.md`) |
| `D:\Projects\Co-op Anything\FarCry3Coop-bantila\` | Копия аналитической работы Bantila (заметки Ghidra по 1.05) |
| `H:\Uplay\Far Cry 3\` | Реальная установка Uplay — только чтение, не изменять (золотое правило 2 в `CLAUDE.md`; правило 2 в хабовом `AGENTS.md`) |
| `H:\FarCryCoop\TestFC3\` | Рабочая копия сборки 1.06: зонды дизасана, запуски меню, тестовые моды |
| `H:\FarCryCoop\games\far-cry-3\tools\` | Черновая копия анализных скриптов для рабочей копии |
| `local/` (вне git) | Приватное состояние владельца: резервные копии реестра, помощник меню, черновики в `local/scratch/` |
| `C:\tools\ghidra_12.1.4_PUBLIC\` | Ghidra 12.1.4, которую использует REA (`GHIDRA_INSTALL_DIR`); архив `ghidra_12.1.4_PUBLIC_20260921.zip` (544 МБ) лежит рядом |
| `C:\tools\ghidra_12.2_PUBLIC\` | Ghidra 12.2 (879 МБ) — REA 6.8.0 её **не** использует (политка только 12.1.x); ждёт решения владельца об удалении |
| `C:\tools\jdk-21.0.13+9\` | Eclipse Temurin JDK 21.0.13+9 (`JAVA_HOME`) |
| `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents\` | REA 6.8.0 (глобальная установка npm); MCP-сервер запускает `scripts\rea.mjs` |
| `C:\Users\USER\.pi\agent\mcp.json` | Конфиг MCP агента pi: запись сервера `rea` с `GHIDRA_INSTALL_DIR` и `JAVA_HOME`; там же резервные копии REA `mcp.json.rea.backup*` |
| `C:\Users\USER\.agents\skills\reverse-engineer-anything\` | Собственный навык REA, версия 49 (`cli_commands 175`, `mcp_tools 229`, `mcp_prompts 6`) |

## Отклонения от золотых правил

- `env.ps1` в корне репозитория нет, и `E:\FarCryCoop\env.ps1` на этом ПК тоже нет. `JAVA_HOME`
  (`C:\tools\jdk-21.0.13+9`) и `GHIDRA_INSTALL_DIR` (`C:\tools\ghidra_12.1.4_PUBLIC`) заданы
  **для пользователя** (`HKCU\Environment`) в `C:\tools`, а не направлены в рабочую папку. Настройки и
  кэш Ghidra остаются в стандартном для Ghidra месте: `support\launch.properties` в
  `C:\tools\ghidra_12.1.4_PUBLIC` не изменялся. `TEMP`/`TMP` — стандартные для Windows
  (`%USERPROFILE%\AppData\Local\Temp`), глобальный префикс npm — `%APPDATA%\npm`; проверено командами
  `reg query HKCU\Environment` и `npm root -g`.
- REA 6.8.0, его навык, Ghidra и JDK установлены на C: (`C:\tools`, `%APPDATA%\npm`, `.agents\skills`).
  Это противоречит золотому правилу 2 («не трогать C:») и требует решения владельца: перенести
  инструментарий в рабочую папку (`Tools\`, `Data\`) или переписать правило так, чтобы системное место
  для инструментов было разрешено.
- Сама регистрация REA — тоже запись на C:: `C:\Users\USER\.pi\agent\mcp.json` (2026-11-03, владелец
  разрешил; см. [docs/tools/rea.ru.md](tools/rea.ru.md)).
- Агентская программа пишет свои временные файлы и выводы делегатов в
  `C:\Users\<user>\AppData\Roaming\pi-desktop\` (скрипты `tmp\.ctx-mode-*`, файлы результатов делегатов). Это
  нарушает правило "не писать на C:"; нужно решение владельца (настройка программы или переопределение tmp).
- `H:\` используется для игровых установок и рабочей копии, потому что игровые папки находятся там.
- Черновики — в `local/scratch/` (вне git), никогда в `games/<slug>/`; анализные скрипты остаются в `tools/`
  или `games/<slug>/tools/` (оба вне git, см. `.gitignore`).
- `git add -A` в этом репозитории падает: в корне `D:\Projects\Co-op Anything` лежит неудаляемый файл `nul`
  (`error: invalid path 'NUL'`). Коммитить только с явными путями.
