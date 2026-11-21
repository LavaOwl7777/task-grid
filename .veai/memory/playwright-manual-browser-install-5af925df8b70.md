---
name: "playwright-manual-browser-install"
description: "Playwright browser install: curl mirror playwright.download.prss.microsoft.com, tar, INSTALLATION_COMPLETE marker"
type: project
lastUpdated: 2026-11-21T20:59
---

# Playwright браузеры: установка вручную (сеть с нестабильным доступом к CDN)

**Факт:** в этой среде `npx playwright install` не работает: `cdn.playwright.dev` отвалывается по таймауту, а распаковка зависает на неопределённое время (20+ минут без вывода; убитый mid-extract установщик оставляет неполную папку без `chrome.exe`).

**Рабочий обходной путь (проверен 2026-11-20 для Playwright v1.50/билд 1161):**
1. Скачать архив curl'ом с зеркала `https://playwright.download.prss.microsoft.com/dbazure/download/playwright/builds/<tool>/<build>/<file>.zip` (авто-фолбэк установщика идёт именно туда; первый CDN — нет).
2. Сверить размер с `Content-Length` из HEAD-запроса.
3. `tar.exe -xf <zip> -C %LOCALAPPDATA%\ms-playwright\<tool>-<build>` (папка должна существовать ДО вызова tar — bsdtar падает с "could not chdir" иначе).
4. Создать пустой маркер-файл `INSTALLATION_COMPLETE` внутри папки — без него Playwright считает установку незавершённой.

**Какие компоненты нужны для `npm run test` (screenshot-тест на chromium, headless):** `chromium_headless_shell-<build>\chrome-win\headless_shell.exe`, `winldd-<build>\PrintDeps.exe`; полный `chromium-<build>` не обязателен, но пусть целый. Каждый недостающий компонент даёт свою ошибку `Executable doesn't exist at ...` — ставь по очереди.

**Why:** бинарники Playwright не в git и не в node_modules — при смене версии/машине установка повторится и снова зависнет стандартным путём.
**How to apply:** любая установка/обновление Playwright-браузеров на этой машине — сразу curl+tar+маркер, не тратить время на ожидание установщика.

**Подтверждено 2026-11-21 (task-grid, `@playwright/test 1.51.1` по package-lock):** номера сборок совпадают с 1.50 — chromium/chromium-headless-shell 1161, winldd 1007; ffmpeg 1011 (в дефолтном наборе `browsers.json`) скачан той же схемой: `ffmpeg-win64.zip` (~1.4 МБ, внутри `ffmpeg-win64.exe`). Внимание: в package.json `prepare` = `husky install && npx playwright install --with-deps` — обычный `npm install` повиснет на установщике; зависимости ставить через `npm install --ignore-scripts`.
