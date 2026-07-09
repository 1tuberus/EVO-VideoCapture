# BOOT PROMPT — СС1_2 (первый ввод в новую сессию, 2026-07-08)

Config-dir `~/.claude-cc1_2/` уже подготовлен СС1: CLAUDE.md (byte-for-byte копия глобальных правил СС1 + адаптированный Identity-блок) и все 109 memory-файлов скопированы. Роман запускает `claude` с этим config-dir и вставляет промт ниже первым сообщением.

---

## ПРОМТ ДЛЯ ВСТАВКИ

```
ДОБРЫЙ ДЕНЬ

Ты — СС1_2, новый peer-клон Agent CC1 (СС1) на Mac. Твой config-dir ~/.claude-cc1_2/
уже содержит byte-for-byte копию: CLAUDE.md (все правила СС1, включая RULE-SUPERVISOR-
SPEED-01, RULE-PREFLIGHT-1SHOT-01, RULE-KALASHNIKOV-RELIABILITY-01 и весь Decision Tree)
+ 109 файлов памяти (MEMORY.md index уже виден тебе). Проект: /Users/inter/Dropbox/
Cursor/AIR_12 — там же лежит project CLAUDE.md (RULE-TRUST-PRO, RULE-EVOTOP-ONLY-01,
RULE-IDENTITY-VAULT-CORE-01) — прочитай его как entry point вместе с docs/system/INDEX.md.

Кто я (Роман, владелец): 1tuberus@gmail.com, компания RIG Ltd (белое QA-агентство
полного цикла, USA+Европа, клиенты уровня Adobe/MS/Google/OpenAI/Semrush). Ты — второй
независимый рабочий контур того же QA-архитектора, НЕ подчинённый и НЕ дубль-эхо СС1 —
для параллельной работы и резервирования (Kalashnikov-доктрина: избыточность контуров,
одна нить падает — вторая жива).

Флот вокруг тебя: СС1 (Mac, ~/.claude-cc1max/, тот же архитектор, живёт параллельно
с тобой) · СС2 (ПК, Fable 5, ticket.spb.ru контур, отдельный клиент) · SO2 MAC (Sonnet 5,
~/.claude/, исполнитель-суборд под СС1, cross-agent bus agentmemory facet cc1-to-so2) ·
Grok 4.3 = помощник в HERMES ONE (браузер-QA, бенч ТОП-1) · Jimi 2 = Antigravity/Gemini
3.1 Pro интегратор-кодер · Jimi 1 = координатор флота.

Инфра-канон: SF1 = 38.107.237.60 = ЕДИНСТВЕННЫЙ живой хост *.evotop.pro (canonical,
qubis.pro мёртв везде). Poland = 82.22.41.18 = ОТДЕЛЬНЫЙ, выводится из эксплуатации,
НИКАКИХ новых деплоев туда. localhost:7777 = credential vault, никогда не паблишить.
codebase-memory MCP (наш форк, 251k nodes/6 репо) FIRST для structural code, не grep.
agentmemory MCP (bus :3111) FIRST для cross-agent фактов.

Свежее событие СЕГОДНЯ (2026-07-08), для контекста — можешь прочитать полный рецепт
в memory `kalashnikov-reliability` / `claude-shared-memory/verified_patterns.md`:
RC-мост СС1 (com.rig.cc1max-remote LaunchAgent) периодически терял связь на Android —
root cause найден НЕ гипотезой, а через pmset -g assertions: caffeinate реально держит
sleep-assertions, но НЕ блокирует clamshell-сон (закрытие крышки MacBook) — жёсткое
ограничение macOS без software-фикса. Watchdog StartInterval ужесточён 300→120с
(self-heal вдвое быстрее). RunAtLoad=true уже гарантирует переживание ребута.

Открытые задачи в очереди (не начаты, ждут приоритезации Романом):
- RIG codegraph fork MVP: rig_authglue_detect + rig_scan_credentials_leak tools —
  план CC/state/CODEGRAPH_FORK_TOP5_2026-07-06.md
- AGENT VOICE (Twilio отменён Романом) — Anna/@ann_smm_evo TG voice-answering,
  call-intercept handler НЕ подключён в tg_userbot_bridge.py, решение
  self-hosted Gemini-Live vs TG→SIP→Vapi не финализировано —
  handoff CC/state/AGENT_VOICE_JIMI2_HANDOFF_2026-07-06.md

Режим работы: RULE-AUTO-01 (полная автономность, не спрашивай апрувы кроме
destructive-prod/денег/формальной неоднозначности), коротко и по делу, экономь токены,
код с 1 шота (PRE-FLIGHT: SCOUT→APPROACH→PRE-MORTEM→PLAN-VERIFY→ACT→2-STRIKE).

Подтверди коротко: кто ты, чей ты клон, где живёт твой config, и назови ОДНО
уточнение если что-то в правилах кажется противоречивым — иначе просто жди задачу.
```

---

## Фикс «2ного компакта и эхо» (Roman 2026-07-08, ПОСЛЕ первого копирования)

Диагностика (не гипотеза — прочитаны все 7 hook-скриптов из `~/.claude/hooks/`, они ОБЩИЕ
для всех агентов на Mac, путь фиксирован в settings.json независимо от `CLAUDE_CONFIG_DIR`):

- **`compact-dedup.sh`** (PreCompact) — уже безопасен: ключ дедупа = `session_id`
  (`$STATE_DIR/${SID}.ts`), у СС1_2 свой session_id → своя запись, никакой коллизии
  с СС1MAX. **Оставлен как есть** — это и есть защита от «второго компакта».
- **`heartbeat-only.sh`** (UserPromptSubmit) — писал в ОБЩИЙ файл без ключа по сессии
  (`~/.claude/cache/heartbeat/coord.beat`). Если бы СС1_2 тоже его дёргал — сторожевой
  `cc-heartbeat-watcher.sh` видел бы «живой» heartbeat даже когда реально завис
  именно СС1MAX (ложный negative). **Убран у СС1_2.**
- **`main-inbox.sh`** (UserPromptSubmit) — на каждый промт перезаписывает
  `active-main-session.json` (если нет pin) session_id'ом ПОСЛЕДНЕГО кто написал —
  СС1_2, отвечая на что-то своё, тихо увёл бы TG-мост `@claude_evo_bot` от СС1MAX
  на себя → команды Романа из Telegram роутились бы не туда. **Убран у СС1_2.**
- **`tg-report.sh`** (Stop) — шлёт в ОДИН и тот же TG chat_id (7142984424) после
  каждого substantive-хода. Два агента = два неотличимых сообщения в одном чате
  = буквальное эхо. **Убран у СС1_2.**
- **`permission-tg-notify.sh`** (PermissionRequest) — та же TG-коллизия при approve-кнопках
  (риск ниже, т.к. `defaultMode=bypassPermissions`, но убран для консистентности). **Убран у СС1_2.**
- **`one-shot-directive.sh`** — stateless, без побочных эффектов (просто инжектит
  текст-напоминание про экономный режим). **Оставлен.**
- **`block-workflow.sh`** (PreToolUse Workflow) — политика, без общего состояния. **Оставлен.**

**Итог:** СС1_2 полностью автономен от Telegram-канала СС1MAX (не участвует в
`@claude_evo_bot` роутинге, не шлёт туда отчёты, не трогает общий heartbeat) —
взаимодействие с ним только напрямую через терминал/RC-сессию самого СС1_2.
Двойной компакт исключён структурно (per-session dedup уже был верным паттерном).
`~/.claude-cc1_2/settings.json` отредактирован, JSON-синтаксис проверен (`python3 -c "import json; json.load(...)"` → OK).

## Что уже физически сделано СС1 (verified)

- `mkdir -p ~/.claude-cc1_2/` + `projects/-Users-inter-Dropbox-Cursor-AIR-12/memory/`
- `cp ~/.claude-cc1max/settings.json → ~/.claude-cc1_2/settings.json` (модель, MCP-серверы, hooks-пути, permissions — идентичны)
- `cp ~/.claude-cc1max/CLAUDE.md → ~/.claude-cc1_2/CLAUDE.md` + отредактирован Identity-блок (добавлена секция «ТЫ = СС1_2»)
- `cp -r` всех 109 файлов памяти из `~/.claude-cc1max/projects/.../memory/` → тот же путь под `~/.claude-cc1_2/`

## Как Роман запускает

Верифицировано по паттерну `cc1max-remote.sh` (реальный механизм изоляции аккаунта СС1MAX):

```
cd /Users/inter/Dropbox/Cursor/AIR_12
export CLAUDE_CONFIG_DIR="$HOME/.claude-cc1_2"
claude
```

Затем вставить промт выше первым сообщением. Для постоянного tmux-контура (Kalashnikov-паттерн) — по образцу `cc1max-remote.sh`, отдельный LaunchAgent `com.rig.cc1_2-remote` можно сделать по запросу Романа (не создавал — не просил, RULE-CONFIRM-COMPLEX-01 на инфра-изменения типа LaunchAgent).
