# EVO-VideoCapture — карта ребренда (каркас Ф1, исполнение в Ф2+)

Приватный форк **Cap** (AGPLv3) → **EVO-VideoCapture** (RIG Ltd). Источник истины по плану:
`AIR_12/CC/state/EVO_VIDEOCAPTURE_RESEARCH_2026-06-30.md`.

## Бренд-решения (фиксированы)
| Параметр | Значение |
|---|---|
| Имя продукта | **EVO-VideoCapture** |
| Bundle id (desktop) | `pro.evotop.videocapture` (было `so.cap.desktop[.dev]`) |
| Дизайн-токены | Manrope (шрифт), фиолетовый `#8B5CF6` (как EVO-X/EVO-RD) |
| Self-host домен | `vid.evotop.pro` (share-ссылки/бэкенд) |
| Хранилище | свой S3-совместимый бакет на нашей инфре (Польша/SF1) — НЕ cap.so SaaS |
| Версия | начнём с `1.0.0` (наш счёт) |

## Поверхность ребренда (замерено 2026-06-30)
- `cap.so` встречается в **168 файлах** → заменить на `evotop.pro` / `vid.evotop.pro`.
- bundle `so.cap` в **11 файлах** → `pro.evotop.videocapture`.
- «Cap Software» в **7 файлах** → «RIG Ltd» (с сохранением AGPL-атрибуции, см. `NOTICE-EVO.md`).

## Файлы Ф2 (инкрементально, `cargo check`/lint после каждого шага; СБОРКА на CI/ПК, НЕ на 8 ГБ Маке)
1. **Идентичность desktop:** `apps/desktop/src-tauri/tauri.conf.json` (`productName`, `identifier`),
   плюс prod-конфиг/билд-скрипт (productName сейчас `Cap - Development` → ищем prod-override в `scripts/` и CI).
2. **Иконки:** `apps/desktop/src-tauri/icons/*` + корневой `app-icon.png` → бренд EVO (фиолет, object-fit, без растяга).
3. **package.json names:** корневой (`name: cap`) + воркспейсы `apps/*/package.json`, `packages/*`.
4. **Rust crates:** `Cargo.toml` package names (где светят в UI/бинарях; внутренние crate-имена можно не трогать).
5. **Web/бренд-строки:** `apps/web` (тексты, метаданные, лого, домены cap.so→evotop.pro).
6. **Дизайн-токены:** тема Tailwind/CSS — основной цвет → `#8B5CF6`, шрифт → Manrope.
7. **Self-host (Ф3):** отвязать от cap.so SaaS — endpoints/ENV на `vid.evotop.pro` + свой S3.

## AGPL-комплаенс (выполнять параллельно)
- `LICENSE` — НЕ удалять/НЕ менять (AGPLv3).
- `NOTICE-EVO.md` — пометка модификации + дата (§5a) — ✅ добавлено в Ф1.
- При раздаче бинаря — отдавать соответствующий исходник; при публичном модиф. web-сервисе — ссылка «Source» (§13).

## Гигиена репо (Ф2)
- ⚠️ В upstream закоммичены бинари `core` (54 МБ) и `apps/desktop/core` (66 МБ) > GitHub soft-limit 50 МБ.
  В Ф2: убрать из индекса + `.gitignore` (это build-артефакты, не исходник).

## Статус
- **Ф1 — done (2026-06-30):** приватный форк `1tuberus/EVO-VideoCapture`, ветка `evo-rebrand`,
  LICENSE прочитан (AGPLv3), `NOTICE-EVO.md` + эта карта. Сборка НЕ запускалась.
- **Ф2 — next:** ребренд по карте выше (по согласованию с Романом; сборку — на CI/ПК).
