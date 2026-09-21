# Ошибка: git не виден в PATH сессии opencode — использовать полный путь MinGit

- **Дата:** 2026-09-21
- **Автор:** Разработчик-агент opencode (комплект opencode-kollegam)
- **Где:** работа с репозиториями (oshibki-kollegam) из сессии opencode

## Симптом

Команда `git pull` в папке базы упала: команда не найдена
(CommandNotFoundException). При этом в dev-registry заявлен Git 2.55.0.
Проверка стандартных путей `Program Files\Git\...` — файлов нет, поиск
шёл не там.

## Причина

Git (portable MinGit) установлен в `C:\Users\Admin\AppData\Local\Programs\
MinGit\cmd` и добавлен в **User PATH**, но сервер opencode был запущен
раньше и унаследовал старый PATH без MinGit. Та же причина, что и для
MCP/npx: процесс видит PATH только момента своего старта.

## Решение

Использовать **полный путь** к git, а не короткое имя:

```powershell
$git = "C:\Users\Admin\AppData\Local\Programs\MinGit\cmd\git.exe"
& $git -C <репозиторий> pull origin master
```

Проверено: `git pull` (Already up to date), `git push origin master`
(`ab73bfc master -> master`). Рекомендация: полностью перезапустить
opencode — тогда MinGit появится в PATH сессии и можно снова писать
просто `git`.

## Как предотвратить

1. При переносе комплекта на новую машину проверять git/gh/node ДО запуска
   opencode (или сразу после — перезапускать opencode).
2. В скриптах и командах использовать полные пути к инструментам,
   установленным portably (`Programs\MinGit\cmd\git.exe`,
   `Programs\gh\bin\gh.exe`, `Programs\nodejs\npx.cmd`, `~\.local\bin\uv.exe`).
3. Если полного пути нет под рукой — искать через `where.exe`/`Get-Command`,
   а не по стандартным каталогам `Program Files`.