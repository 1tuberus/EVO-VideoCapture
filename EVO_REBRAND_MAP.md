# EVO-VIDEO — карта ребренда

Приватный форк **Cap** (AGPLv3) → **EVO-VIDEO** (RIG Ltd).
База: `cap-v0.5.9` · ветка `evo-video-v2` · исследование: `AIR_12/CC/state/EVO_VIDEOCAPTURE_RESEARCH_2026-06-30.md`

## Бренд
| Параметр | Значение |
|---|---|
| Имя продукта | **EVO-VIDEO** (прежнее рабочее — EVO-VideoCapture) |
| Bundle id | `pro.evotop.video` (dev: `.dev`) |
| Акцент | `#8B5CF6` — тот же фиолет, что в EVO-X и EVO Mac Cleaner Pro |
| Шрифт | Manrope, 6 подмножеств woff2 в комплекте (latin + cyrillic + greek + vietnamese) |
| Self-host домен | `vid.evotop.pro` (фаза 3) |

## Сделано (фаза 2, 2026-08-25)
- Ветка от свежего тега `cap-v0.5.9` — вместо разгребания 1245 коммитов отставания.
- `tauri.conf.json` + `tauri.prod.conf.json`: productName и identifier.
- `apps/desktop/src/styles/theme.css`: акцент перекрашен через шкалу `--blue-*` — **одно место вместо 128 упоминаний** в компонентах; светлая и тёмная темы отдельно.
- `evo-fonts.css` + `styles/fonts/*.woff2`: Manrope локально, без обращений к Google CDN.
- `NOTICE-EVO.md`: уведомление о модификации по §5a.

## Осталось
- Иконки: `apps/desktop/src-tauri/icons/*` + `app-icon.png` → бренд EVO.
- Строки «Cap» в UI-текстах и `package.json` (имена воркспейсов).
- Домены `cap.so` (168 файлов) — **только вместе с self-host**, иначе сломается шаринг.
- Профиль записи по умолчанию: HEVC + 30 к/с — ради этого всё и затевалось (замер 2026-08-25: даёт сжатие 22–33× при SSIM 0.9995).
- Гигиена: бинари `core` (54 МБ) и `apps/desktop/core` (66 МБ) закоммичены в upstream — вынести в `.gitignore`.

## Сборка
Tauri + Rust: **не на 8-гиговом Маке** — там своп. macOS-бинарь собирается только на macOS (ночью, `-j 2`), Windows-версия — на ПК (Xeon 14/28).
