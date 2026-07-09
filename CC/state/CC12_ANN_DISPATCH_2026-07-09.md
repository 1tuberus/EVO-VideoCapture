ДОБРЫЙ ДЕНЬ

Задача: довести Vapi-ассистента Anna до состояния "реально отвечает на звонки через
Telegram". Инфраструктура уже построена и почти работает — осталось 2 конкретных бага,
не архитектура.

Читай FIRST (уже в этом репо, ветка cc1-knowledge-transfer — fetch если ещё не подтянул):
1. CC/state/CC1_KNOWLEDGE_TRANSFER_2026-07-08.md — кто ты, кто владелец, доктрина работы
2. CC/video-transcriber-admin/ANN_VOICE_AGENT_MASTER_PLAN_MAX_2026-07-09.md — полная
   архитектура Ann: инфра на SF1, что живо, что сломано, оба бага разобраны построчно

TL;DR задачи (полный разбор в файле выше, §2):
- Vapi assistant Anna (ID 58839baa-8d98-488f-9493-a7709afac85b) — мозг+голос готовы,
  2 телефонных номера уже куплены и active (+14706251186, +18634170242).
- tg2sip (Docker-контейнер на SF1 38.107.237.60, живёт 2+ дня, реально ловит SIP-звонки
  СЕГОДНЯ) должен мостить Telegram (@ann_smm_evo) ↔ эти номера, но падает на 2 багах.

БАГ №1 — SIP-регистрация к Vapi = 403 Forbidden.
Причина: /opt/tg2sip/.env → SIP_PASSWORD=changeme — буквальный placeholder, никогда не
заменённый на настоящий SIP-креденшл. Найди в Vapi Dashboard/API актуальный способ
получить SIP-trunk credentials (BYO-SIP или их собственный registrar-пароль — проверь
свежую Vapi-документацию сам, не полагайся только на файл, дата могла устареть), впиши
реальный пароль, docker restart tg2sip, verify по логам — статус регистрации должен
стать успешным вместо 403.

БАГ №2 — Telegram-клиент (Pyrogram) внутри tg2sip никогда не логинится, висит на
интерактивном "Enter phone number or bot token" вечно (detached-контейнер, никто не
отвечает). Причина: /app/sessions/ на сервере ПУСТАЯ. TG_SESSION_STRING передана через
env, но Pyrogram её не подхватывает. Нужно: либо разобраться в исходнике tg2sip как он
реально ожидает получать сессию (может имя переменной другое — смотри их README/код,
не гадай), либо сгенерировать настоящий Pyrogram .session sqlite-файл локально один раз
интерактивно и положить в /opt/tg2sip/sessions/ под именем, которое ждёт config.yaml
(session_name: ann_smm_evo → обычно ann_smm_evo.session).

Доступы/ключи (НЕ коммить никуда, это чат-релей, не файл в репо):

SSH на SF1: ssh root@38.107.237.60 (ключ ~/.ssh/id_ed25519, если работаешь с той же
машины что и Roman; иначе Roman сам выполнит SSH-команды по твоей инструкции)
tg2sip живёт: /opt/tg2sip/ (⚠️ НЕ /opt/telegram-ai-voice/tg2sip/ — та копия orphaned,
контейнер её не использует, verified через docker inspect mounts)

VAPI_KEY_1=0536e09c-06e6-46d3-bce9-9e443d6f10aa
VAPI_KEY_2=655824f2-1078-4b0e-af4b-36a7451a0f11
ELEVENLABS_API_KEY=sk_4e3b2b79ff97c3b21ae0ccd973bcd84681605e9ffe7267a3

# /opt/tg2sip/.env на SF1 (текущее содержимое, баг №1 в SIP_PASSWORD)
TG_API_ID=30698862
TG_API_HASH=bcbaf0461ae3289b16a63a993db6aa9f
TG_SESSION_STRING=AgHUbW4APbw22HPkh_vFnLyNETeQsC7t-faXGeGBeocj5GmwSA3S5u4_-GS_9u6oYbl5RysakKgKf52dmQP8HRVhBPEyTA1f-67p6mD-JAws7SLQwViAgyW1BzUEf9zZd3FHZwKUdG3zuFSxxsv7WNPe2a13AzYcMEh2VgGqNPtwXA9TeDsdFeSzF679pKxyK3Kjfs2kwx4px5WqmX6tYdEn-IN2O61gpc5jKBVbrLoowjIXyHnB59VvwDtPI6kWV99Ip4L6T-mGp3AbTdLQzo0AuV8wUuwrEfGN6ih7VxaMQAoMBmw4VjZ3ZWgnh3UWSeIekqOyDzZavrskn6IxqfC1_BSKawAAAAAAAAAAAA
SIP_USERNAME=tg2sip
SIP_PASSWORD=changeme   ← БАГ №1, заменить на реальный
SIP_DOMAIN=sip.vapi.ai
SIP_REGISTRAR=sip.vapi.ai

Сделай по порядку:
1. Почини баг №1 (SIP-пароль) — docker restart tg2sip — verify логами (успешная
   регистрация вместо 403).
2. Почини баг №2 (Pyrogram-сессия) — verify: "Client has not been started yet" больше
   не появляется в логах при входящем звонке.
3. E2E-тест: реальный звонок на +14706251186 или +18634170242 → должен дойти в
   Telegram @ann_smm_evo → Anna отвечает голосом. Слушай реальный звук, не "должно
   работать".
4. Если найдёшь новый auth-глюч по пути (Vapi SIP-докой/Pyrogram-квиркой) — это
   saleable finding для RIG, не просто препятствие, задокументируй кратко в отчёте.
5. Отчитайся Роману коротко: что сделано, что verified, звонок реально прошёл или нет.

Не трогай: Poland-сервер (82.22.41.18, деплои запрещены), другие docker-контейнеры на
SF1 (n8n-pl-*, leadcatcher-*, eva-pl-*, ticket-cms, toploc-directus, hysteria2-sf1,
ss-sf1, xray-cc1) — не относятся к задаче. Секреты выше — только в .env на сервере,
никогда в git commit.

Работай в режиме максимальной автономности — не спрашивай подтверждений на штатные
шаги (docker restart, правка .env на сервере, SSH-команды) — только на реально
разрушительные/денежные действия. Короткие отчёты, verify реальным тестом на каждом шаге.
