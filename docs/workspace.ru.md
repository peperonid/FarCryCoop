# Рабочая папка

English version: [workspace.md](workspace.md)

Примечание: каноническая структура `E:\FarCryCoop` ниже — это **история предыдущего ПК**.
На этом ПК диск `E:` есть, но папки `FarCryCoop` на нём нет и файла `env.ps1` тоже нет. Фактическая
структура описана в [LOCAL.ru.md](LOCAL.ru.md) (+ [LOCAL.md](LOCAL.md)); реально установленный на этом
ПК инструментарий — в разделе «Инструментарий на этом ПК» ниже.

## Каноническая структура (история предыдущего ПК)

Вся работа шла в `E:\FarCryCoop` (золотое правило 2 в [CLAUDE.md](../CLAUDE.md), которое теперь указывает
на `D:\Projects\Co-op Anything\FarCryCoop`). Сначала загружали окружение: `. E:\FarCryCoop\env.ps1`.

| Папка | Содержимое |
|---|---|
| `Downloads\<имя>\` | Исходные архивы, у каждого своя папка, SHA-256 проверен |
| `Tools\` | Распакованные портативные инструменты |
| `Data\` | Настройки и кэши инструментов (настройки/кэш Ghidra, кэш npm, глобальный префикс npm) |
| `Temp\` | Временные файлы (сюда указывали `TEMP`/`TMP`) |
| `FarCryCoop\`, `FarCry<N>Coop\` | Репозитории, по одной папке на игру |

## Установленные инструменты (2026-10-18) — история предыдущего ПК

| Инструмент | Версия | Источник | SHA-256 (архив) |
|---|---|---|---|
| Node.js | v24.22.0 LTS (zip win-x64) | https://nodejs.org/dist/v24.22.0/ | `f9683a19877e12a9b167305c8db71040d812876359de21f0d2e97c3f321f80c3` |
| Eclipse Temurin JDK | 21.0.12.1+1 (zip x64) | https://github.com/adoptium/temurin21-binaries/releases/tag/jdk-21.0.12.1%2B1 | `f9d6e191ab098c0d416e7d588a24420a8621cd2f4720dab2459b8b7b2d2d8b4e` |
| Ghidra | 12.1.4 (`ghidra_12.1.4_PUBLIC_20260921.zip`) | https://github.com/NationalSecurityAgency/ghidra/releases/tag/Ghidra_12.1.4_build | `ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db` |

Контрольные суммы взяты из `SHASUMS256.txt` на nodejs.org, API Adoptium и релиза Ghidra на GitHub;
все три загрузки совпали.

В Ghidra там изменён `support\launch.properties` (оригинал сохранён как `launch.properties.orig`):
`JAVA_HOME_OVERRIDE`, `application.settingsdir`, `application.cachedir`, `application.tempdir`,
`java.io.tmpdir` указывали в `E:\FarCryCoop`. Проверка: `analyzeHeadless.bat` вывел справку и создал
папки настроек/кэша в `E:\FarCryCoop`. На этом ПК такого перенаправления нет:
`C:\tools\ghidra_12.1.4_PUBLIC\support\launch.properties` не изменён (его дата — 2026-09-21,
`JAVA_HOME_OVERRIDE=` по-прежнему пуст, файла `launch.properties.orig` рядом нет), а настройки и кэш
Ghidra находятся в стандартном для Ghidra месте (папка `C:\Users\USER\AppData\Roaming\ghidra` существует).

## Инструментарий на этом ПК (2026-11-03)

| Инструмент | Версия | Путь | Источник / подтверждение | SHA-256 (архив) |
|---|---|---|---|---|
| REA (`rea-agents`) | 6.8.0 | `C:\Users\USER\AppData\Roaming\npm\node_modules\rea-agents` (скрипт `scripts\rea.mjs`) | глобальная установка npm; `rea --version` -> 6.8.0 | не записан |
| Навык REA `reverse-engineer-anything` | 49 | `C:\Users\USER\.agents\skills\reverse-engineer-anything` | установлен командой `rea setup` | неприменимо |
| Ghidra | 12.1.4 (`ghidra_12.1.4_PUBLIC_20260921.zip`) | `C:\tools\ghidra_12.1.4_PUBLIC` | https://github.com/NationalSecurityAgency/ghidra/releases/tag/Ghidra_12.1.4_build | `ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db` (перепроверено 2026-11-03 командой `sha256sum` по архиву размера 569,649,598 байт) |
| Eclipse Temurin JDK | 21.0.13+9 | `C:\tools\jdk-21.0.13+9` | файл `release` в этой же JDK: `IMPLEMENTOR="Eclipse Adoptium"`, `IMPLEMENTOR_VERSION="Temurin-21.0.13+9"`, `JAVA_VERSION="21.0.13"`, `JAVA_VERSION_DATE="2026-10-20"` | не записан |
| Node.js | v22.23.4 | `C:\Program Files\nodejs\node.exe` | системная установка; удовлетворяет требованию REA Node 22.23.4+ | не записан |

`node -v` -> v22.23.4, `npm -v` -> 10.9.9, `java -version` -> Temurin-21.0.13+9 (build 21.0.13+9-LTS).
Для REA, JDK и Node.js на этом ПК проверены только строки версии и пути; хэшей архивов для них не
записано (в отличие от архива Ghidra, который лежит на диске).

В `C:\tools` также лежат `ghidra_12.1.4_PUBLIC_20260921.zip` (архив 544 МБ, оставлен) и
`ghidra_12.2_PUBLIC` (879 МБ). **Ghidra 12.2 REA не использует:** REA 6.8.0 принимает только линию
12.1.x, а 12.2 объявляет `application.java.min=25` (см. [docs/tools/rea.ru.md](tools/rea.ru.md)).
Решения о удалении владелец ещё не дал.

Пользовательские переменные окружения (`HKCU\Environment`) на этом ПК:

| Переменная | Значение |
|---|---|
| `GHIDRA_INSTALL_DIR` | `C:\tools\ghidra_12.1.4_PUBLIC` |
| `JAVA_HOME` | `C:\tools\jdk-21.0.13+9` |

Машинной переменной `JAVA_HOME` нет. `TEMP` и `TMP` — стандартные для Windows
(`%USERPROFILE%\AppData\Local\Temp`): ничего в рабочую папку не перенаправляется.

## Известные пределы правила «не трогать C:»

- Windows, PowerShell и системный `git.exe` (`C:\Program Files\Git`) находятся на C:. Они используются
  как установлены; наши собственные скрипты туда ничего не пишут.
- Запись в конфиг клиента REA выполнена 2026-11-03 в `C:\Users\USER\.pi\agent\mcp.json`: владелец
  разрешил эквивалент прежней записи в `%USERPROFILE%\.claude.json`. Путь `claude mcp add` на этом ПК не
  использовался, так как Claude Code на этом ПК не установлен. REA хранит собственные резервные копии
  (`mcp.json.rea.backup*`) в той же папке.
- REA 6.8.0 (глобальный префикс npm), его навык, Ghidra 12.1.4 и Temurin JDK 21.0.13+9 установлены на
  C: (`C:\tools`, `C:\Users\USER\AppData\Roaming\npm`, `C:\Users\USER\.agents\skills`), и
  `GHIDRA_INSTALL_DIR`/`JAVA_HOME` указывают в `C:\tools`. Это противоречит золотому правилу 2 (ничего
  на C:) и требует решения владельца: перенести инструментарий в рабочую папку (`Tools\`, `Data\`) или
  переписать правило так, чтобы системное место для инструментов было разрешено.
- Настройки и кэш Ghidra тоже не перенаправлены: `support\launch.properties` в
  `C:\tools\ghidra_12.1.4_PUBLIC` на этом ПК не изменялся, поэтому они остаются в стандартном для
  Ghidra месте.
- `E:\FarCryCoop` и `E:\FarCryCoop\env.ps1` на этом ПК не существуют, поэтому шаг «сначала загрузить
  окружение» из старой структуры здесь выполнить нельзя.
