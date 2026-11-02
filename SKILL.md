---
name: figma-mcp-setup
description: Одноразовая настройка Figma MCP в OpenCode-агентах (MultiTool, OpenCode). Используй, когда пользователь кинул в чат файл/ссылку с инструкцией по подключению Figma(MCP) (например figma-mcp-instruction.html) или попросил «подключи фигму», «настрой MCP фигмы», «сделай чтобы работали инструменты figma_*», «почему не работает фигма MCP», "set up Figma MCP", "connect Figma" — даже если он не называет MCP явно. Это служебная разовая операция: проверяет, что в конфиге есть секция mcp.figma с url https://mcp.figma.com/mcp, а в mcp-auth.json лежит свежий OAuth-токен; при нехватке переносит токен из соседнего конфига того же движка и правит файлы, затем просит перезапустить приложение. Работает полностью автономно: сам читает и правит конфиг, от пользователя остаётся только штатное окно UAC. Всё на Windows PowerShell, без внешних зависимостей. Do NOT use для разбора дизайна/layers Figma, для работы с Figma-файлами (то инструменты figma_*), или для подключения MCP других сервисов (Wildberries, Яндекс и т.п.) — здесь только Figma.
---

# Настройка Figma MCP (полный автопилот)

Пользователь кинул файл/ссылку с инструкцией по подключению Figma MCP или
просит, чтобы инструменты `figma_*` работали в чатах. Твоя задача — довести
настройку до рабочего состояния, самому внося правки, и оставить одиночный
понятный шаг пользователю (обычно только «перезапусти приложение»).

## Ключевая идея (почему всё так, а не иначе)

Агенты на движке OpenCode читают конфиг MCP в формате `opencode.json` и хранят
OAuth-токены в том же формате `mcp-auth.json`. Figma подключается как
`type: "remote"`, `url: https://mcp.figma.com/mcp` — это **не** локальный сервер
и **не** фигмовский PAT (`figd_...`), а OAuth. Установка токенов уже может
существовать в соседнем конфиге того же движка (например, у пользователя стоит
и MultiTool, и чистый OpenCode) — ищи готовые токены там и переноси, прежде чем
просить что-то вводить руками.

Алгоритм: определить хост → проверить конфиг → проверить токен и его срок →
починить недостающее → попросить перезапуск.

## Шаги

### 1. Определи окружение (какой агент/хост)

Найди, какой конфиг использует текущий агент. Обычно это один из двух
движков, оба читают одинаковый формат:

- MultiTool → `%USERPROFILE%\.config\gigatool\` и `%USERPROFILE%\.local\share\gigatool\`
- OpenCode → `%USERPROFILE%\.config\opencode\` и `%USERPROFILE%\.local\share\opencode\`

Проверь через `Test-Path`, какие каталоги реально существуют и какой соответствует
текущему агенту (по его имени в чате, по работающей папке, по отсутствию/наличию
файлов). Формально оба пути — просто `%USERPROFILE%\.config\<host>\opencode.json`
и `%USERPROFILE%\.local\share\<host>\mcp-auth.json`, где `<host>` =
`gigatool` | `opencode`. Запиши выбранные:

```
CONFIG = %USERPROFILE%\.config\gigatool\opencode.json   # пример MultiTool
AUTH   = %USERPROFILE%\.local\share\gigatool\mcp-auth.json
```

Рядом часто лежит конфиг «соседнего» движка того же формата (эталон из
основного OpenCode) — он пригодится как источник уже готового токена:

```
OCONFIG = %USERPROFILE%\.config\opencode\opencode.json
OAUTH   = %USERPROFILE%\.local\share\opencode\mcp-auth.json
```

> `%USERPROFILE%` — домашняя папка Windows текущего пользователя (обычно
> `C:\Users\<имя-пользователя>`); в PowerShell это `$env:USERPROFILE`.
> **Никогда не зашивай в скрипты конкретное имя пользователя или хост.**

Если у текущего агента своя папка-хост (не `gigatool`/`opencode`) — возьми её,
формат файлов тот же. Не смотри только один вариант: сначала определи хост,
потом решай.

### 2. Проверь секцию `mcp` в конфиге агента

Прочитай `CONFIG` (это JSON). В нём должен быть объект:

```json
"mcp": {
  "figma": {
    "type": "remote",
    "url": "https://mcp.figma.com/mcp",
    "enabled": true
  }
}
```

Если секции `mcp` нет вообще или нет ключа `figma` — добавь/вправь. При правке
не трогай остальные ключи (`permission` и прочее), сохраняй их без изменений.

### 3. Проверь и, если надо, перенеси OAuth-токен

`AUTH` должен содержать объект `figma` с полями `clientInfo` (clientId,
clientSecret) и `tokens` (accessToken, refreshToken, expiresAt). Ключ —
`figma` (в нижнем регистре, так движок ожидает).

- Токен есть и `expiresAt` в будущем → ок, ничего не делай.
- Токена нет, `expiresAt` в прошлом ИЛИ в `AUTH` пустовато → посмотри `OAUTH`
  (соседний конфиг). Если там есть валидный свежий `figma` (тот же json-формат) —
  скопируй блок `figma` целиком (clientInfo+tokens) в `AUTH`.
- Свежего токена нигде нет → сообщи пользователю, что нужно пройти OAuth заново
  (браузер → «Allow»), и не прави конфиг до этого. Подскажи путь: обычно
  `node dist/index.js auth figma --url https://mcp.figma.com/mcp` из
  `mcp-auth-helper` (sdaoudi/mcp-auth-helper на GitHub).

> ВАЖНО про соседние ключи конфига: если в `AUTH` рядом с `figma` есть отдельный
> ключ со `codeVerifier` (например `Figma`) — сохрани его / восстанови при
> перезаписи; он связан с OAuth-потоком, удалять его не нужно. Пиши файл в UTF-8.

### 4. Запись в файл, заблокированный запущенным приложением

Пока приложение запущено, `AUTH` может держаться открытым, и запись падает с
`Access to the path ... is denied`, а прав на запись у обычной сессии может не
быть (ACL: `Users: RX`). Тогда:

1. Проверь, действительно ли файл заблокирован:
   `[System.IO.File]::Open($p, Open, ReadWrite, ReadWrite)` — если исключение,
   продолжай с пунктом 2.
2. Пиши через elevated-запуск (UAC) — надёжнее всего и не требует прав рукой.
   Запиши правильный json из временного `.ps1`, запущенного от имени
   администратора, через `Start-Process ... -Verb RunAs -Wait`. Пример:

   ```powershell
   $script = @'
   $gtPath = Join-Path $env:USERPROFILE ".local\share\gigatool\mcp-auth.json"  # git. подставь свой <host>: gigatool / opencode / иное
   # ... собери $newJson как строку JSON с блоком figma ...
   Set-Content -Path $gtPath -Value $newJson -Encoding UTF8 -Force
   $check = Get-Content $gtPath -Raw
   if ($check -match '"expiresAt":\s*([0-9.]+)') {
     $e=[double]$Matches[1]
     Write-Output ("OK: токен Figma обновлён, срок до " + [DateTimeOffset]::FromUnixTimeMilliseconds([long]($e*1000)).ToLocalTime())
   }
   '@
   $tempScript = [System.IO.Path]::GetTempFileName() + '.ps1'
   Set-Content -Path $tempScript -Value $script
   Start-Process -FilePath 'powershell' -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File `"$tempScript`"" -Verb RunAs -Wait
   Remove-Item $tempScript -Force -ErrorAction SilentlyContinue
   ```

   UAC-запрос выскочит в окне («Разрешить»), пользователь жмёт — без закрытия
   приложения. После — перечитай `AUTH` и убедись, что `expiresAt` правильный.

> Запись в `CONFIG` (opencode.json) обычно не заблокирована и не требует UAC —
> правится обычным edit/write. UAC нужен прежде всего для `AUTH`.

### 5. Финиш и проверка

- Скажи пользователю перезапустить агент/приложение. После перезапуска инструменты
  `figma_*` должны появиться; проверить можно через `figma_whoami` (в текущем чате
  они могут появиться только в новой сессии).
- Кратко отчитайся: что было сделано (конфиг / токен / оба), что перезапустить,
  как проверить.

## Чек-лист результата

- [ ] Определён хост (`gigatool` / `opencode` / иной) и правильные `CONFIG`/`AUTH`.
- [ ] В `CONFIG` есть `mcp.figma` (remote, url верный, enabled: true).
- [ ] В `AUTH` есть блок `figma` с clientInfo+tokens, `expiresAt` в будущем.
- [ ] При перезаписи сохранён `codeVerifier`-ключ, если был.
- [ ] Файлы записаны (конфиг — обычной правкой; AUTH — под UAC, если нужно).
- [ ] Пользователь получил одну инструкцию: перезапустить приложение.