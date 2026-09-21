# Ошибка: officecli — официальный установщик отдаёт битый файл (mirror); качать с GitHub

- **Дата:** 2026-09-21
- **Автор:** Разработчик-агент opencode (комплект opencode-kollegam)
- **Где:** установка OfficeCLI для MCP-сервера officecli (opencode)

## Симптом

Команда из официальной инструкции `irm https://d.officecli.ai/install.ps1 | iex`
зависла; по таймауту выяснилось: в `%TEMP%\officecli.exe` скачался файл
размером **16 КБ вместо 33 МБ** — бинарник не установлен, `officecli.exe` не
найден. Установщик сначала качает с mirror `d.officecli.ai`, и mirror отдал
битый/неполный файл (fallback на GitHub не сработал в срок).

## Причина

Официальный install.ps1 (iOfficeAI/OfficeCLI) качает с mirror `d.officecli.ai`
первым, GitHub — только fallback. Mirror периодически отдаёт повреждённый
файл (HTML-заглушку или обрезанный ответ); скрипт при этом не проверяет
размер, а проверка контрольной суммы идёт уже после полной загрузки.

## Решение

Качать **напрямую с GitHub** (работает стабильно):

```powershell
# Windows x64, версия v1.0.151 (актуальная на 2026-09-21), immutable URL:
Invoke-WebRequest -Uri "https://github.com/iOfficeAI/OfficeCLI/releases/download/v1.0.151/officecli-win-x64.exe" -OutFile "$env:LOCALAPPDATA\OfficeCLI\officecli.exe" -TimeoutSec 240
# или «всегда последняя»: .../releases/latest/download/officecli-win-x64.exe

& "$env:LOCALAPPDATA\OfficeCLI\officecli.exe" --version   # ожидаемо: 1.0.151
# размер файла: 33 439 656 байт (~33 МБ); SHA256 (v1.0.151):
# 57cd0e597514a4948ca034ea187637a18ad5b44a6e6729dfb7799414e06a083f
# проверка: Get-FileHash ... -Algorithm SHA256
```

Дальше как обычно: добавить `%LOCALAPPDATA%\OfficeCLI` в User PATH и включить
MCP в `opencode.jsonc` (`"command": ["C:\\Users\\Admin\\AppData\\Local\\OfficeCLI\\officecli.exe", "mcp"], "enabled": true`).
Результат: `mcp connected server=officecli tools=1`.

## Как предотвратить

1. При установке officecli предлагать коллегам **сразу ссылку на скачивание
   через GitHub**, а не `irm ... | iex`:
   `https://github.com/iOfficeAI/OfficeCLI/releases/latest/download/officecli-win-x64.exe`
2. После скачивания проверять размер (~33 МБ) и `officecli --version`.
3. Если установщик (mirror) отдал файл меньше ~10 МБ — не использовать,
   качать с GitHub.