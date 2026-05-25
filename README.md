# AI Radar

Ежедневный дайджест AI-новостей из Telegram-каналов. Раз в день в 09:00 (Екатеринбург, UTC+5) бот присылает в личку топ-5 постов с саммари и оценкой важности.

## Стек

- **Telethon** — чтение каналов через MTProto
- **Groq** (llama-3.3-70b) — анализ и саммаризация
- **python-telegram-bot** — отправка дайджеста
- **SQLite + SQLAlchemy** — дедупликация постов
- **Xray 25.3.6** (VLESS+Reality) — прокси для Telegram с VPS
- **Docker + cron** — расписание

---

## Установка с нуля

### 1. Получи ключи

| Что нужно | Где взять |
|---|---|
| `TG_API_ID` / `TG_API_HASH` | https://my.telegram.org → API development tools |
| `TG_BOT_TOKEN` | @BotFather в Telegram → `/newbot` |
| `TG_USER_CHAT_ID` | написать боту @userinfobot |
| `GROQ_API_KEY` | https://console.groq.com |
| Данные Xray-сервера | у тебя уже есть (UUID, public key, IP, port) |

### 2. Клонируй и настрой

```bash
git clone <repo> ai-radar
cd ai-radar
cp .env.example .env
# Заполни .env своими данными
```

### 3. Сгенерируй Telegram-сессию (один раз, локально)

```bash
pip install telethon
python scripts/generate_session.py
```

Скопируй выведенную строку в `.env` как `TG_SESSION_STRING=...`

### 4. Сгенерируй конфиг Xray

```bash
pip install python-dotenv
python scripts/generate_xray_config.py
```

Создаст файл `xray/config.json` с твоими данными.

### 5. Запусти

```bash
docker compose up -d --build
```

Проверь логи:

```bash
docker compose logs -f bot
tail -f logs/cron.log
```

---

## Ручной запуск (для теста)

```bash
docker compose exec bot python -m src.main
```

---

## Расписание

Cron-задача в контейнере: `0 4 * * *` (04:00 UTC = 09:00 Yekaterinburg).

Если твой часовой пояс другой — скорректируй час в `crontab`:
- UTC+3 (Москва): `0 6 * * *`
- UTC+6 (Омск): `0 3 * * *`

---

## Структура проекта

```
ai-radar/
├── src/
│   ├── config.py           # настройки из .env
│   ├── sources/telegram.py # сбор постов через Telethon
│   ├── analyzer/groq.py    # анализ через Groq API
│   ├── storage/            # SQLite + SQLAlchemy
│   ├── delivery/           # отправка через Telegram Bot API
│   └── main.py             # точка входа
├── scripts/
│   ├── generate_session.py     # генерация TG_SESSION_STRING
│   └── generate_xray_config.py # генерация xray/config.json
├── xray/
│   └── config.json.template    # шаблон конфига Xray
├── tests/test_smoke.py
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── crontab
```

---

## Критерии работоспособности

- После `docker compose up -d` система работает без вмешательства
- В 09:00 в личку прилетает дайджест
- При повторном запуске за один день дайджест не дублируется
- Если Groq API упал — ошибка в логах, посты будут проанализированы при следующем запуске
- `docker compose logs bot` показывает прогресс каждого шага

---

## Тесты

```bash
pip install -e ".[dev]"
pytest
```
