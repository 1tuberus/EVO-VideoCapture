# ANN / VOICE AGENT — MASTER PLAN MAX (2026-07-09, REVISION 2 — CORRECTED)

Полная передача знаний по проекту «Ann» (Энн) для агента, продолжающего доработки —
читать этот файл ВМЕСТО повторного ресёрча репозитория. Всё ниже — verified 2026-07-09
(живой SSH на SF1, live Vapi API, docker logs/config), не пересказ старых доков.

🔴 **REVISION 2:** Первая версия этого файла (та же дата, ниже по времени) ошибочно
назвала Vapi-трек «вероятно запаркованным» на основе устаревшей (2-3 дня) памяти,
не перепроверив live-состояние. Roman поправил: **Vapi-путь — АКТИВНЫЙ и главный.**
Live-проверка (curl к Vapi API + docker-логи tg2sip) подтвердила: у Anna уже 2 реальных
телефонных номера (куплены 2026-07-06), и SIP-мост к ним построен и запущен — просто
падает на двух конкретных багах (§2). Секция про PyTgCalls-прямой-в-Gemini (§4) —
это ПАРАЛЛЕЛЬНЫЙ, вероятно исследовательский трек, не главная задача.

## 0. TL;DR

- **Домен:** [videotrans.evotop.pro](https://videotrans.evotop.pro/) → SF1 `38.107.237.60`,
  `/opt/videotrans/`, systemd `videotrans.service` — **ACTIVE прямо сейчас** (verified).
- **«Ann» = Vapi.ai assistant «Anna»** (ID `58839baa-8d98-488f-9493-a7709afac85b`), мозг+голос
  готовы, **у неё уже 2 живых телефонных номера** (`+14706251186`, `+18634170242`, status
  active, куплены 2026-07-06 — verified live GET прямо сейчас). Telegram-мост строится через
  **`tg2sip`** (Docker-контейнер на SF1, SIP-шлюз Telegram↔SIP, использует аккаунт
  `@ann_smm_evo`) — **контейнер работает 2+ дня, реально получает звонки, но падает на
  2 конкретных, точно найденных багах (§2). Это и есть единственная реальная
  незавершённая работа — не архитектура, а именно эти 2 бага.**
- Для контекста, НЕ путать: есть ещё «Марина» (§3, отдельный demo/dashboard-агент,
  Pipecat+LiveKit) и параллельный PyTgCalls-прямой-в-Gemini трек (§4, вероятно
  экспериментальный, не факт что нужен) — оба НЕ являются задачей «доделать Ann».

## 1. Инфраструктура (verified live 2026-07-09)

| Компонент | Статус | Где |
|---|---|---|
| `videotrans.service` (Node.js admin, server.js) | 🟢 active | SF1 systemd |
| `tg_userbot_bridge.py` (Telethon userbot) | 🟢 процесс жив (root, venv) | SF1, **НЕ под systemd** — просто голый процесс, при краше НЕ перезапустится сам (Kalashnikov-доктрина нарушена, нужен unit) |
| docker `tg2sip` | 🟢 up 2 дня | SF1 — SIP↔Telegram шлюз уже развёрнут |
| docker `videotrans-livekit-server-1` | 🟢 up 2 дня, порты 7880/7881/7882 | SF1 |
| docker `videotrans-redis-1` | 🟢 up 2 дня, порт 8379 | SF1 — телеметрия Марины |
| TG-сессии | 5 файлов в `/opt/videotrans/voice_worker/sessions/`: `ann_smm_evo` (бот) + 4 тестовых аккаунта-звонильщика (testalexkorob/testandrey/testromancpa/testromancpl) | SF1 |

Деплой: `rsync -avz --exclude node_modules/ --exclude workspace/` из
`CC/video-transcriber-admin/` → `root@38.107.237.60:/opt/videotrans/`, затем
`systemctl restart videotrans`. Полная инструкция: `DEPLOY_INSTRUCTIONS_V2.7.md` (этот же каталог).

**Секреты (пути + env-var имена, НЕ значения в этом файле — не коммитить). Реальные
значения переданы Роману отдельно в чате (не в git — этот файл может уйти в публичный
репозиторий), Роман передаёт их СС12 напрямую текстом, не файлом в публичном репо:**
- TG session-strings/api_id/api_hash (`ann_smm_evo` + tg2sip): `voice_worker/deploy_sessions.py` (gitignored) + `/opt/tg2sip/.env` + `/opt/videotrans/voice_worker/sessions/*.json` на сервере.
- `GEMINI_API_KEY` + `GEMINI_API_KEYS_POOL` (10-ключевой пул) + `SUPADATA_API_KEY`: только в env живого процесса на SF1 (`/proc/<pid>/environ`), файла `.env` для voice_worker НЕ существует — как именно они туда попадают (какой wrapper их экспортирует) НЕ найдено, требует розыска перед тем как переносить деплой на systemd-unit (§2, пункт 6).
- Vapi (`VAPI_KEY_1`, `VAPI_KEY_2`) + ElevenLabs: `CC/secrets/vapi_2026-07-06.env`.
- tg2sip SIP-пароль: `SIP_PASSWORD=changeme` — **это буквальный placeholder, не настоящий пароль** (см. §2 баг №1).

## 2. Vapi «Anna» + tg2sip мост — ГЛАВНАЯ, АКТИВНАЯ задача (это и есть «Ann»)

**Roman подтвердил и live-проверка согласна:** реальный путь — Vapi.ai assistant Anna,
подключаемая к Telegram через `tg2sip` (SIP-шлюз). НЕ через PyTgCalls-напрямую-в-Gemini
(это §4, отдельный параллельный трек).

**Живое состояние Vapi (verified GET прямо сейчас, 2026-07-09):**
```
Assistant ID: 58839baa-8d98-488f-9493-a7709afac85b, name: Anna
serverUrl: None (не нужен для SIP-пути)
Phone numbers (оба status=active, куплены 2026-07-06):
  +14706251186  (id dfb9cad0-db3a-4b5e-9350-6e8fd5647ad0)
  +18634170242  (id 22b263be-d7dc-4e09-8cfe-94324cd9ec34)
```
System-prompt/voice-конфиг (Sarah, eleven_flash_v2_5, deepgram/en) — канон, не менять
без approve Романа, полный текст: `CC/state/AGENT_VOICE_JIMI2_HANDOFF_2026-07-06.md` §5.

**tg2sip — что это и где (verified через docker inspect + logs + config.yaml на SF1):**
- Контейнер `tg2sip` (image `tg2sip-gateway`), исходники смонтированы с хоста
  `/opt/tg2sip/` (⚠️ ЕСТЬ дубль-папка `/opt/telegram-ai-voice/tg2sip/` — НЕ используется
  контейнером, mounts подтверждают `/opt/tg2sip/{sessions,config}` — не путай, не трогай
  вторую папку, она orphaned).
- `/opt/tg2sip/config/config.yaml`: `telegram.session_name: ann_smm_evo`,
  `inbound_routes: "*": "sip:58839baa-8d98-488f-9493-a7709afac85b@sip.vapi.ai"`
  (маршрут TG→SIP уже прописан на правильный Assistant ID).
- Контейнер поднят 2026-07-06 23:30, живёт **уже 2+ дня, реально принимает SIP-звонки
  СЕГОДНЯ** (логи 2026-07-09 09:11 и 09:57 — Vapi шлёт тестовые SIP INVITE).

**БАГ №1 — SIP-регистрация к Vapi = 403 Forbidden (первая строка в логах контейнера):**
```
registering as sip:tg2sip@sip.vapi.ai → SIP registration failed, status=403 (Forbidden)
```
Причина: `SIP_PASSWORD=changeme` в `/opt/tg2sip/.env` — **буквальный placeholder,
никогда не заменённый на настоящий SIP-креденшл от Vapi.** Нужно: в Vapi Dashboard /
API найти актуальный способ получить SIP-trunk креды для номера/ассистента (BYO-SIP
или их собственный registrar-пароль — проверить текущую Vapi-документацию, могло
измениться), вписать в `.env`, `docker restart tg2sip`, смотреть логи на
`status=200`/успешную регистрацию вместо 403.

**БАГ №2 — Telegram-клиент внутри tg2sip никогда не логинится (второй, независимый баг):**
```
Welcome to Pyrogram (version 2.0.106)
Enter phone number or bot token:      ← висит здесь НАВСЕГДА, никто не отвечает
...
"cannot resolve TG target '+442088199562': Client has not been started yet"
```
Причина: `docker exec tg2sip ls /app/sessions/` → **пусто** (только `.`/`..`). Хотя
`TG_SESSION_STRING` (готовая строка сессии `ann_smm_evo`) передана через env-переменную,
Pyrogram внутри контейнера её не подхватывает и падает в интерактивный login-prompt —
который в detached-контейнере никто не может ответить, так клиент висит вечно.
**Нужно:** либо (a) разобраться в исходнике tg2sip как он ожидает получать сессию
(может это не `TG_SESSION_STRING`, а другое имя переменной/формат — проверь их
README/исходники, не гадай), либо (b) сгенерировать настоящий Pyrogram `.session`
sqlite-файл локально (через `Client.start()` один раз интерактивно с реальным
номером/кодом) и положить его в `/opt/tg2sip/sessions/` с именем, которое ждёт
`session_name: ann_smm_evo` из config.yaml (обычно `ann_smm_evo.session`).

**Итог:** оба бага независимы, чинятся отдельно, ни один не архитектурный — это
конфигурационные проблемы в уже написанном и задеплоенном мосте. После фикса обоих —
`docker restart tg2sip` → тестовый звонок с любого телефона на `+14706251186` или
`+18634170242` → должен дойти до Telegram `@ann_smm_evo` → Anna отвечает.

**Реальный E2E-тест (после фиксов):** позвонить на любой из 2 номеров Anna → слушать,
доходит ли звонок и отвечает ли голос — не «должно работать», слушать реальный звук.

## 3. Подсистема «Марина» — НЕ путать с задачей (для контекста, не трогать без запроса)

`voice_worker/main.py` — Pipecat pipeline (LiveKit transport + Deepgram STT + Groq LLM
llama3-70b + ElevenLabs TTS + Silero VAD), публикует latency-метрики (TTFB, VAD duration)
в Redis `voice_agent_live_metrics`, читает `voice_db.js`/`voice_api.js` → SSE → дашборд
"VOICE AGENT" в UI (`public/`, вкладка с осциллографом Chart.js). Это demo/telemetry-агент
для browser/LiveKit-комнаты, НЕ для реальных TG-звонков и НЕ связан с Vapi Anna.
`.env.example` показывает нужные переменные (`LIVEKIT_*`, `DEEPGRAM_API_KEY`,
`OPENAI_API_KEY`=Groq-ключ, `ELEVENLABS_API_KEY`, `TTS_VOICE_ID` — сейчас placeholder
`your_marina_clone_voice_id`, не настоящий голос). Инфра (LiveKit+Redis) уже живая на
SF1 (см. §1), но НЕ является частью задачи «Ann».

## 4. Параллельный трек: PyTgCalls-напрямую-в-Gemini (НЕ приоритет, но не удалять)

**Файлы:** `voice_worker/tg_userbot_bridge.py` (Telethon+PyTgCalls) + `voice_worker/gemini_ws.py`
(готовый Gemini Multimodal Live duplex-мост, FIFO-based) + `voice_worker/diag_agent.py`.
Живой процесс на SF1 (PID жив с Jul08, `ppid=1` — **НЕ под systemd**, осиротевший процесс,
переживёт крэш только пока сам не упадёт). Обрабатывает пока только голосовые СООБЩЕНИЯ
(`event.message.voice`), не звонки — строка 40 буквально `# To handle calls, we would
add PyTgCalls handlers here`. `gemini_ws.py` готов (system-prompt Энн внутри), но нигде
не импортируется из `tg_userbot_bridge.py` — не подключён.

Известные гочи (если этот трек когда-то доведут до конца): модель `models/gemini-2.0-flash`
(строка 53 `gemini_ws.py`) требует проверки на актуальность Bidi-поддержки (Jimi2
2026-07-08: `gemini-2.0-flash-exp` её лишился, нужен `gemini-2.5-flash-native-audio-latest`);
FIFO/SIGPIPE фикс из `AG12/verified_patterns.md:719-722` (ffprobe рвёт FIFO-writer) —
в `gemini_ws.py` этого фикса нет; `PyTgCalls(client)` создавать только внутри `async def
main()`. Тупиковая ветка (не повторять, 3ч потрачено): `AG12/src/voice_agent/ann_group_call.py`.
Возможный дубль/более новая версия: `AG12/src/voice_agent/system1_telegram_native.py` —
не сверено с `tg_userbot_bridge.py`, при желании продолжать этот трек — сверь сначала.

**Roman: если §2 (Vapi+tg2sip) закрывает задачу — этот трек, вероятно, можно заморозить
как исследовательский; если нужен собственный voice-стек без Vapi — тогда этот трек
и есть план Б.**

## 5. Safety / что не трогать

- Секреты (`deploy_sessions.py`, `.env`, `CC/secrets/*`) — verified gitignored, НЕ коммитить.
  Перед любым `git add` рядом — `git check-ignore -v <file>`.
- SF1 = канон-хост, Poland (`82.22.41.18`) — деплои запрещены (сервер выводится).
- Не трогай другие docker-контейнеры на SF1 (n8n-pl-*, leadcatcher-*, eva-pl-*,
  ticket-cms, toploc-directus, hysteria2-sf1, ss-sf1, xray-cc1) — не относятся к задаче.
- RULE-AGENT-APPEND-ONLY: правки в общие доки/память — только дописывать.

## Related

`CURRENT_STATE.md` · `PROJECT_ENCYCLOPEDIA.md` · `MANUAL.md` · `DEPLOY_INSTRUCTIONS_V2.7.md`
(этот же каталог) · `CC/state/AGENT_VOICE_JIMI2_HANDOFF_2026-07-06.md` ·
`CC/state/VOICE_AI_TOP5_OSS_2026-07-06.md` · `AG12/verified_patterns.md:719-730` ·
memory `vapi-anna-assistant` / `agent-voice-twilio-cancelled` · task #102.
