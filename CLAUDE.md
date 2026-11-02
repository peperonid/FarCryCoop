# Project rules for Claude / Правила проекта для Claude

## Golden rule / Золотое правило

**EN:** ALWAYS write documentation and every other human-readable project text (README, notes,
guides, comments in docs, issue/PR descriptions) in BOTH Russian and English. No exceptions.
Use a pair of files `NAME.md` (English) + `NAME.ru.md` (Russian), or both languages in one file
for short texts. When one language version changes, update the other in the same commit.

**RU:** ВСЕГДА пиши документацию и любые другие тексты проекта для людей (README, заметки,
руководства, комментарии в документах, описания issue/PR) на ДВУХ языках: русском и английском.
Без исключений. Используй пару файлов `NAME.md` (английский) + `NAME.ru.md` (русский) или оба
языка в одном файле для коротких текстов. Если меняется одна языковая версия, обнови другую в
том же коммите.

## Golden rule 2: workspace / Золотое правило 2: рабочая папка

**EN:** Work ONLY inside `D:\Projects\Co-op Anything\FarCryCoop`. NEVER read, write or download anything on drive C: —
no exceptions. NEVER download into `%TEMP%`. Before running any tool, load `. D:\Projects\Co-op Anything\FarCryCoop\env.ps1`
(it points TEMP/TMP, npm cache, JAVA_HOME, Ghidra to `D:\Projects\Co-op Anything\FarCryCoop`). Layout:
`Downloads\<name>\` — verified archives (one folder each); `Tools\` — unpacked tools;
`Data\` — tool settings and caches; `Temp\` — temporary files. The game at
`D:\SteamLibrary\steamapps\common\FarCry` is read-only (AGENTS.md rule 2); work on copies in
`D:\Projects\Co-op Anything\FarCryCoop`. If a step would need drive C:, stop and ask the owner.

**RU:** Работать ТОЛЬКО внутри `D:\Projects\Co-op Anything\FarCryCoop`. НИКОГДА ничего не читать, не писать и не скачивать
на диск C: — без исключений. НИКОГДА не качать в `%TEMP%`. Перед запуском любых инструментов
загружать `. D:\Projects\Co-op Anything\FarCryCoop\env.ps1` (он направляет TEMP/TMP, кэш npm, JAVA_HOME, Ghidra в
`D:\Projects\Co-op Anything\FarCryCoop`). Структура: `Downloads\<имя>\` — проверенные архивы (у каждого своя папка);
`Tools\` — распакованные инструменты; `Data\` — настройки и кэши инструментов; `Temp\` — временные
файлы. Игра в `D:\SteamLibrary\steamapps\common\FarCry` — только чтение (правило 2 AGENTS.md);
работать с копиями в `D:\Projects\Co-op Anything\FarCryCoop`. Если шаг требует диска C:, остановиться и спросить владельца.

## Commits / Коммиты

**EN:** Commit messages follow Conventional Commits (https://www.conventionalcommits.org/en/v1.0.0/)
and are written in English only: `<type>(<optional scope>): <description>`, e.g.
`docs(far-cry): add research notes`. Types: feat, fix, docs, chore, refactor, test, build, ci, perf, style, revert.

**RU:** Сообщения коммитов пишутся по Conventional Commits и только на английском:
`<тип>(<область, необязательно>): <описание>`, например `docs(far-cry): add research notes`.

## Tools / Инструменты

**EN:** Reverse engineering uses REA (https://github.com/morluto/rea) as an MCP server, registered
locally on the owner's PC: see [docs/tools/rea.md](docs/tools/rea.md).

**RU:** Для реверс-инжиниринга используется REA (https://github.com/morluto/rea) как MCP-сервер,
подключённый локально на ПК владельца: см. [docs/tools/rea.ru.md](docs/tools/rea.ru.md).

## Communication / Общение

**EN:** Chat with the owner in Russian.

**RU:** С владельцем общаться по-русски.

## Session rules / Правила сессии

**EN:** Tool calls: one tool block per turn, one tool name per block, arguments under ~4 KB. Long probe
scripts are written to a file and run with a short command (`python -I tools/<script>.py`), never pasted
as tool input; read files with line ranges, not whole. This rule exists because two turns this session
printed half-mangled tool markup as plain text and stopped.
Working paths: the live workspace is the folder named in `docs/LOCAL.md` (`local/` is git-ignored and
owner-specific); the game install is never modified. `D:\Projects\Co-op Anything\FarCryCoop` in rule 2 above, and the same path
in [docs/workspace.md](docs/workspace.md), is the original 2024 workspace and is no longer present on
this PC.

**RU:** Вызовы инструментов: один блок на ход, один инструмент в блоке, аргументы не больше ~4 КБ.
Длинные probe-скрипты писать в файл и запускать короткой командой (`python -I tools/<script>.py`),
не вставлять код в вызов; читать файлы частями (строки по номерам), не целиком. Правило появилось
из-за двух ходов, где разметка вызовов выводилась как обычный текст и работа останавливалась.
Рабочие пути: рабочая папка указана в `docs/LOCAL.md` (`local/` в git не попадает и принадлежит
владельцу); установку игры не изменять. `D:\Projects\Co-op Anything\FarCryCoop` из правила 2 выше — рабочий каталог 2024 года,
на этом ПК его нет.
