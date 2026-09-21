# Ошибка: MCP memory/sequentialthinking падали «Connection closed» — npx не находил node в PATH

- **Дата:** 2026-09-21
- **Автор:** Разработчик-агент opencode (комплект opencode-kollegam)
- **Где:** подключение MCP-серверов в `opencode.jsonc`

## Симптом

В прогоне «тест разработчика» MCP memory и sequentialthinking не
подключались: в opencode.log `mcp connect failed ... status.error="Connection
closed"`, инструменты в каталоге отсутствовали. При этом npm-пакеты
существуют (2026.8.31), `npx` по полному пути работает.

## Причина

Сервер opencode был запущен **до** установки Node.js; унаследованный PATH
не содержал папку nodejs. Команда `cmd /c npx -y @modelcontextprotocol/...`
падала («"node" не является командой»): npx при запуске пакета ищет node
в PATH процесса, а процесс opencode имеет PATH момента своего старта.

## Решение

В `opencode.jsonc` для обоих серверов задать **полный путь к `npx.cmd`**
и `environment.PATH` с nodejs (не полагаясь на унаследованный PATH):

```jsonc
"command": ["cmd", "/c", "C:\\Users\\Admin\\AppData\\Local\\Programs\\nodejs\\npx.cmd",
            "-y", "@modelcontextprotocol/server-memory"],
"environment": { "PATH": "C:\\Users\\Admin\\AppData\\Local\\Programs\\nodejs;C:\\Windows\\System32;C:\\Windows" }
```

Проверено: `mcp connected server=memory tools=9`, `server=sequentialthinking tools=1`.
Та же схема сработала для officecli (абсолютный путь к exe) и fetch/time
(uv-шимы в `~/.local/bin`).

## Как предотвратить

1. Ошибка не только про npx: **opencode-сервер наследует PATH момента запуска** —
   инструменты, установленные позже (node, git, uv, gh), он не видит.
2. В конфиге MCP/командах использовать **абсолютные пути** и при необходимости
   `environment.PATH`, а не короткие имена.
3. После установки новых инструментов — полностью перезапускать opencode,
   чтобы сервер подхватил обновлённый PATH.