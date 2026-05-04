# mirror — Backend

Ты пишешь backend для приложения **mirror** — планер для предпринимателей с Telegram-ботом и AI-агентом.

Прочитай этот файл целиком. Здесь всё: спека, этапы, правила. Других документов нет.

После прочтения — кратко (5-7 строк) скажи что понял и предложи начать STAGE 0. Не пиши код пока я не сказал «ок, начинай».

---

## Эталоны фронта (ОБЯЗАТЕЛЬНО)

Готовые эталоны всех экранов фронта лежат на GitHub:

**https://github.com/andrewsatorim/Planer**

В архиве `V18.zip` (в одной из веток репо) — 238 HTML-файлов всех экранов приложения mirror в финальном дизайне (mobile/desktop, dark/light, все состояния).

**Правила использования:**

1. **Каждый раз когда делаешь endpoint** — открывай соответствующий эталон экрана и смотри какие поля и в каком виде нужны фронту. Это источник истины для:
   - Имён полей в response (если на экране показывается «доход месяца» — endpoint должен отдавать поле под этим смыслом)
   - Состояний (open / in_progress / done / archived — все должны иметь свой эталон)
   - Структуры view-эндпоинтов (day/week/month/horizon — какие данные группируются вместе)
   - Empty-states (что показываем когда нет данных)

2. **Соответствие endpoint → экран:**
   - `/tasks/day/:date` → `screens/day/mobile_dark.html`, `screens/day/desktop_dark.html`
   - `/tasks/week/:start` → `screens/week/*`
   - `/tasks/month/:y/:m` → `screens/month/*`
   - `/finance/summary` → `screens/finance/mobile_dark.html`
   - `/finance/pipeline` → `screens/finance/pipeline_mobile_dark.html`
   - `/directions` → `screens/direction/*` + `screens/profile/directions_management_*`
   - `/me/stats` → `screens/profile/mobile_dark.html`
   - `/archive/*` → `screens/archive/*`
   - `/auth/*` → `screens/auth/*`
   - И так далее — каждой группе endpoints соответствует папка экранов

3. **Любые визуальные доработки** (например текст ошибок, формат вывода данных в API ответах) делать **в соответствии с дизайном эталонов**. Не выдумывай свои тексты — смотри что уже есть на экране и используй те же формулировки.

4. **Бренд-правила из эталонов** (закреплены, не нарушать):
   - `mirror` всегда lowercase
   - `«Код 1847»` всегда в кавычках-ёлочках
   - Числа: `247к ₽` (cyrillic строчная к для тысяч), `1.84M ₽` (latin uppercase M для миллионов)
   - ToV: на «ты», без архаизмов и пафоса
   - Стоп-слова в любых текстах от backend: VIP, элитный, лакшери, эксклюзив, премиальный (если попадаются в коде/email-шаблонах/логах для юзера — заменять)

5. **Перед каждым этапом** который касается endpoints — клонируй или обнови репо с эталонами, найди соответствующие экраны, посмотри что там. Только потом пиши код.

---

## Правила работы

1. Работа строго по этапам ниже. Сейчас STAGE 0. К следующему этапу не переходить пока я не сказал «дальше» или «ок, дальше».

2. В конце каждого этапа должны быть выполнены ВСЕ acceptance criteria:
   - `pytest` проходит
   - `ruff check .` — 0 ошибок
   - `mypy app` — 0 ошибок
   - `PROGRESS.md` обновлён (журнал решений и времени)

3. Стек закреплён, не меняй: FastAPI 0.115+, Python 3.12, SQLAlchemy 2.0 async, Pydantic v2, PostgreSQL 16, Redis 7, **uv** (не poetry, не pip), Docker.

4. Стиль кода:
   - Python 3.12 syntax (типы через `|`, не `Optional`)
   - SQLAlchemy 2.0 async (Mapped, mapped_column)
   - Pydantic v2 (model_validator, ConfigDict)
   - line-length 100
   - docstrings на русском, имена на английском
   - magic numbers → const

5. Безопасность: никогда не логируй токены, пароли, refresh_tokens, magic_link tokens, полное содержимое AI-запросов. Только метрики: tokens count, duration, model.

6. Git: ты делаешь только `git add` и `git commit` после каждого этапа с conventional commit message (feat/fix/chore/test/docs/refactor). **Push НЕ делаешь** — это моя работа.

7. Если что-то непонятно или противоречит спеке — **спрашивай меня**. Не додумывай. Не пиши TODO-заглушки.

8. Money везде в копейках (int). Никаких float.

9. Все timestamps в UTC. due_date и due_time хранятся в TZ юзера без конвертации.

10. Команды от меня:
    - **«дальше»** / **«ок, дальше»** / **«next»** → следующий этап
    - **«стоп»** → остановись, жди
    - **«покажи план»** → опиши план этапа без выполнения
    - **«где мы»** → текущий этап + что сделано
    - **«проверь»** → прогони acceptance criteria
    - **«коммит»** → git add + commit (без push)

---

## Обзор продукта

**mirror** — планер для предпринимателей которые ведут несколько направлений жизни параллельно.

**Фичи:**
- До 7 активных направлений (студия, эзотерика, трейдинг и т.п.)
- Цели (год, месяц), задачи с приоритетами и сроками
- Финансы по направлениям: транзакции, pipeline сделок, налоги
- Архив с восстановлением и hard-delete
- Telegram-бот с AI-агентом (создание задач из текста)

**Тарифы:**
- Базовый: **790 ₽/мес** или **6 990 ₽/год**
- С AI-агентом: **1 900 ₽/мес** или **16 900 ₽/год**
- Триал 7 дней без карты

**AI двухуровневый:**
- **Tier 1 — DeepSeek** (через OpenAI-compat API): extract_task, parse_command, reminders text
- **Tier 2 — OpenAI GPT-4o-mini**: weekly/monthly summary, reflection

Все вызовы AI идут через `services/ai/router.py` (AIRouter), никогда напрямую.

---

## Стек

| Компонент | Что |
|---|---|
| Web | FastAPI 0.115+ |
| Python | 3.12 |
| ORM | SQLAlchemy 2.0 async |
| Driver | asyncpg |
| Миграции | Alembic |
| Валидация | Pydantic v2 |
| БД | PostgreSQL 16 (Railway managed) |
| Cache + jobs | Redis 7 (Railway managed) |
| Cron | APScheduler 3.x |
| Email | Resend |
| AI Tier 1 | DeepSeek |
| AI Tier 2 | OpenAI GPT-4o-mini |
| Telegram | aiogram 3.x (webhook) |
| Платежи | ЮKassa |
| Тесты | pytest + pytest-asyncio + httpx |
| Линтер | ruff |
| Type check | mypy strict |
| Pre-commit | pre-commit |
| Docker | multi-stage |
| Logs | structlog (JSON в проде) |
| Errors | Sentry |
| Package manager | uv |

---

## Архитектурные принципы

1. Multi-tenancy с первого дня — все таблицы имеют `user_id`, фильтрация в каждом запросе
2. Soft-delete для tasks, transactions, deals, directions — `deleted_at`
3. Hard-delete через 30 дней (cron) или явно через `permanent` endpoint
4. AI как сервис через `AIRouter` — провайдер per-operation через env
5. Pricing как данные — таблица `pricing_plans` с `active_from/active_to`
6. Idempotency для денежных операций — `idempotency_key` UNIQUE
7. Rate limiting на публичных endpoints
8. Все timestamps в UTC, фронт конвертирует
9. Money в копейках (int)
10. Audit log на критичные действия

---

## Схема БД (16 таблиц)

### users
```
id              uuid pk default gen_random_uuid()
email           text unique not null
telegram_id     bigint unique nullable
telegram_username text nullable
name            text nullable
avatar_url      text nullable
locale          text default 'ru' not null
timezone        text default 'Europe/Moscow' not null
created_at      timestamptz default now() not null
last_login_at   timestamptz nullable
deleted_at      timestamptz nullable
```

### pricing_plans
```
id              uuid pk
code            text not null            -- 'basic_monthly', etc.
name            text not null            -- 'Базовый'
period          text not null            -- 'monthly' | 'yearly'
price_kop       int not null             -- в копейках: 79000 = 790₽
features        jsonb not null           -- {"directions_max":7,"ai_agent":false}
active_from     timestamptz not null
active_to       timestamptz nullable
created_at      timestamptz default now()
unique(code, active_from)
```

### subscriptions
```
id              uuid pk
user_id         uuid fk → users(id) on delete cascade
plan_id         uuid fk → pricing_plans(id)
status          text not null            -- 'trialing'|'active'|'past_due'|'canceled'|'expired'
trial_ends_at   timestamptz nullable
current_period_start timestamptz not null
current_period_end   timestamptz not null
cancel_at_period_end boolean default false
canceled_at     timestamptz nullable
created_at      timestamptz default now()
```
**Индекс:** `(user_id, status)`, `(current_period_end) WHERE status='active'`

### payments
```
id              uuid pk
user_id         uuid fk
subscription_id uuid fk nullable
amount_kop      int not null
currency        text default 'RUB' not null
status          text not null            -- 'pending'|'succeeded'|'failed'|'refunded'
provider        text not null            -- 'yookassa'
provider_id     text not null
idempotency_key text unique not null
metadata        jsonb default '{}'
created_at      timestamptz default now()
updated_at      timestamptz nullable
```

### directions
```
id              uuid pk
user_id         uuid fk on delete cascade
name            text not null
description     text nullable
order_index     int not null
is_main         boolean default false not null
status          text default 'active' not null  -- 'active'|'archived'
archived_at     timestamptz nullable
created_at      timestamptz default now()
```
**Индексы:** `(user_id, status, order_index)`, `unique partial (user_id) where is_main = true`

### goals
```
id              uuid pk
user_id         uuid fk on delete cascade
direction_id    uuid fk → directions(id) nullable
title           text not null
description     text nullable
target_amount_kop int nullable
period_type     text not null            -- 'year' | 'month'
period_start    date not null
period_end      date not null
status          text default 'active' not null  -- 'active'|'achieved'|'archived'
achieved_at     timestamptz nullable
created_at      timestamptz default now()
```

### tasks
```
id              uuid pk
user_id         uuid fk on delete cascade
direction_id    uuid fk → directions(id) nullable
goal_id         uuid fk → goals(id) nullable
title           text not null
notes           text nullable
priority        text default 'normal' not null  -- 'low'|'normal'|'high'
estimated_minutes int nullable
status          text default 'open' not null    -- 'open'|'in_progress'|'done'|'archived'
due_date        date nullable
due_time        time nullable
started_at      timestamptz nullable
completed_at    timestamptz nullable
completion_note text nullable
last_reminded_at timestamptz nullable
source          text default 'manual' not null  -- 'manual'|'telegram'|'ai_agent'
created_at      timestamptz default now()
deleted_at      timestamptz nullable
```
**Индексы:**
- `(user_id, status, due_date) WHERE deleted_at IS NULL`
- `(user_id, direction_id) WHERE deleted_at IS NULL`
- `(user_id, due_date) WHERE status='open' AND deleted_at IS NULL`

### task_time_logs
```
id              uuid pk
task_id         uuid fk → tasks(id) on delete cascade
user_id         uuid fk on delete cascade
started_at      timestamptz not null
ended_at        timestamptz nullable
duration_seconds int nullable
created_at      timestamptz default now()
```

### transactions
```
id              uuid pk
user_id         uuid fk on delete cascade
direction_id    uuid fk → directions(id) nullable
type            text not null            -- 'income'|'expense'|'tax'
amount_kop      int not null
title           text not null
notes           text nullable
related_task_id uuid fk → tasks(id) nullable
related_deal_id uuid fk → deals(id) nullable
transaction_date date not null
created_at      timestamptz default now()
deleted_at      timestamptz nullable
```

### deals
```
id              uuid pk
user_id         uuid fk on delete cascade
direction_id    uuid fk → directions(id)
title           text not null
client_name     text nullable
notes           text nullable
amount_kop      int not null
stage           text not null            -- 'lead'|'qualified'|'proposal'|'closed_won'|'closed_lost'
probability     int not null check (probability between 0 and 100)
expected_close_date date nullable
closed_at       timestamptz nullable
created_at      timestamptz default now()
deleted_at      timestamptz nullable
```

### monthly_budgets
```
id              uuid pk
user_id         uuid fk on delete cascade
direction_id    uuid fk → directions(id) on delete cascade
period          date not null            -- первый день месяца
planned_income_kop int not null
created_at      timestamptz default now()
unique(user_id, direction_id, period)
```

### notifications_settings
```
user_id         uuid pk fk → users(id) on delete cascade
daily_summary_enabled  boolean default true
daily_summary_time     time default '21:00'
weekly_review_enabled  boolean default true
weekly_review_dow      smallint default 0  -- 0=воскресенье
task_reminder_minutes  int default 15
overdue_enabled        boolean default true
goal_deadline_enabled  boolean default true
delivery_channel       text default 'telegram'  -- 'telegram'|'email'|'app_only'
quiet_hours_start      time default '23:00'
quiet_hours_end        time default '08:00'
updated_at             timestamptz nullable
```

### auth_magic_links
```
token           text pk                   -- urlsafe_b64, 32 bytes
email           text not null
created_at      timestamptz default now()
expires_at      timestamptz not null
used_at         timestamptz nullable
ip              inet nullable
user_agent      text nullable
```

### auth_sessions
```
id              uuid pk
user_id         uuid fk on delete cascade
refresh_token_hash text not null         -- sha256 hash
device_info     text nullable
ip              inet nullable
expires_at      timestamptz not null
revoked_at      timestamptz nullable
created_at      timestamptz default now()
```

### ai_messages
```
id              uuid pk
user_id         uuid fk on delete cascade
provider        text not null            -- 'deepseek'|'openai'
model           text not null
operation       text not null            -- 'extract_task'|'weekly_summary'|...
input_tokens    int not null
output_tokens   int not null
total_cost_usd  numeric(10,6) not null
duration_ms     int not null
created_task_id uuid fk → tasks(id) nullable
status          text not null            -- 'success'|'error'
error_message   text nullable
created_at      timestamptz default now()
```

### audit_log
```
id              uuid pk
user_id         uuid fk nullable
action          text not null            -- 'subscription.changed', 'account.deleted'
entity_type     text nullable
entity_id       uuid nullable
metadata        jsonb default '{}'
ip              inet nullable
created_at      timestamptz default now()
```

---

## API Endpoints

### Auth — `/api/v1/auth/`
```
POST   /magic-link/request       { email } → 200 {sent: true}
POST   /magic-link/verify        { token } → 200 {access_token, refresh_token, user}
POST   /telegram/login           { tg_init_data } → 200 {access, refresh, user}
POST   /refresh                  { refresh_token } → 200 {access, refresh}
POST   /logout                   → 204
DELETE /account                  { confirmation_phrase } → 204
```

### Users — `/api/v1/me/`
```
GET    /                         → user + active subscription
PATCH  /                         { name?, avatar_url?, timezone?, locale? }
GET    /stats                    → {tasks_done, goals_achieved, focus_seconds, streak_days}
GET    /notifications            → settings
PATCH  /notifications            settings
GET    /export                   → zip (streaming)
```

### Directions — `/api/v1/directions/`
```
GET    /
POST   /                         { name, description?, is_main? }
PATCH  /:id                      { name?, description?, is_main? }
POST   /:id/archive
POST   /:id/restore
DELETE /:id                      (только если archived)
PATCH  /reorder                  { order: [id1, id2, ...] }
```

### Goals — `/api/v1/goals/`
```
GET    /                         ?period=year|month&direction_id=...
POST   /                         { direction_id?, title, target_amount_kop?, period_type, period_start, period_end }
PATCH  /:id
POST   /:id/achieve
DELETE /:id                      (soft archive)
```

### Tasks — `/api/v1/tasks/`
```
GET    /                         ?status=&direction_id=&due_date_from=&due_date_to=
POST   /                         { direction_id?, goal_id?, title, priority?, due_date?, due_time?, estimated_minutes?, notes? }
GET    /:id
PATCH  /:id
POST   /:id/start
POST   /:id/pause
POST   /:id/complete             { completion_note? }
POST   /:id/restore
DELETE /:id

GET    /day/:date                → задачи дня в TZ юзера + summary
GET    /week/:start_date         → задачи + цели + фокус недели
GET    /month/:year/:month       → задачи + цели + summary
```

### Finance — `/api/v1/finance/`
```
GET    /summary?year=2026        → target/fact/tempo/forecast
GET    /transactions             ?period=&type=&direction_id=
POST   /transactions             { direction_id, type, amount_kop, title, transaction_date, notes? }
PATCH  /transactions/:id
DELETE /transactions/:id

GET    /pipeline                 → all deals grouped by stage + metrics
POST   /deals                    { direction_id, title, client_name?, amount_kop, stage, probability, expected_close_date? }
PATCH  /deals/:id
POST   /deals/:id/move           { stage }
POST   /deals/:id/close-won
POST   /deals/:id/close-lost
DELETE /deals/:id

GET    /budgets?year=2026
PUT    /budgets                  { direction_id, period, planned_income_kop } upsert

POST   /export?format=csv&period=month → CSV streaming
```

### Archive — `/api/v1/archive/`
```
GET    /                         ?period=quarter&type=tasks|goals|deals|transactions
GET    /tags                     → tag cloud
GET    /year-ago                 → задача из прошлого года в этот день
GET    /:type/:id
POST   /:type/:id/restore
DELETE /:type/:id/permanent      { confirmation_phrase: "удалить навсегда" }
```

### Subscription — `/api/v1/subscription/`
```
GET    /plans                    → активные тарифы
GET    /                         → моя подписка + status
POST   /change                   { plan_code, promo_code?, idempotency_key } → confirmation_url
POST   /cancel
POST   /resume
POST   /promo/apply              { code }

POST   /webhook/yookassa         (signature verify, idempotent)
```

### Telegram — `/api/v1/telegram/`
```
POST   /webhook/:secret          aiogram webhook
POST   /link                     { tg_init_data, login_token }
DELETE /link
```

### Health — `/api/v1/health/`
```
GET    /live                     → 200
GET    /ready                    → checks DB + Redis
GET    /version                  → git sha, build time
```

---

## Telegram-бот flow

**Команды:**
```
/start                            привязка / приветствие
/today                            задачи сегодня
/week                             фокус недели + список
/help                             помощь
[свободный текст]                 → AI-агент (Tier 1: DeepSeek) → создание задачи
```

**Создание задачи через AI:**

1. Юзер: «завтра в 10 встреча с Алексеем по брендингу студии, важно»
2. Бот проверяет: привязан? тариф включает agent?
3. `ai_router.extract_task(message, context)` → Tier 1 (DeepSeek)
4. Возвращает structured `ExtractedTask`:
   ```json
   {
     "is_task": true,
     "title": "Встреча с Алексеем по брендингу",
     "direction_hint": "студия",
     "due_date": "2026-05-05",
     "due_time": "10:00",
     "priority": "high",
     "estimated_minutes": null
   }
   ```
5. Backend ищет direction по hint (fuzzy)
6. Создаёт task через CRUD, source='telegram'
7. Бот отвечает с ссылкой на задачу

**Напоминания:**
- APScheduler каждые 5 минут — проверка задач с `due_time` через 15 минут
- Уважает quiet_hours юзера
- `last_reminded_at` чтобы не дублировать
- Текст напоминания через Tier 1 (DeepSeek)

**Сводки:**
- Cron в `daily_summary_time` юзера → Tier 2 (GPT-4o-mini) генерирует summary
- Воскресенье вечером → недельный обзор Tier 2

---

## AI-слой архитектура

```
services/ai/
├── __init__.py
├── base.py                     AIProvider abstract base + AIResult
├── router.py                   AIRouter — маршрутизация per-operation
├── providers/
│   ├── __init__.py
│   ├── deepseek.py             DeepSeekProvider (OpenAI-compat)
│   └── openai.py               OpenAIProvider
├── prompts/
│   ├── __init__.py
│   ├── task_extraction.py
│   ├── command_parsing.py
│   ├── reminder_text.py
│   ├── weekly_summary.py
│   ├── monthly_summary.py
│   └── reflection.py
├── schemas.py                  ExtractedTask, ParsedCommand, etc.
└── exceptions.py               AIError, AITimeoutError, AIRateLimitError
```

**AIRouter:**
```python
class AIRouter:
    OPERATION_TIER = {
        'extract_task':         'tier1',
        'parse_command':        'tier1',
        'generate_reminder':    'tier1',
        'simple_response':      'tier1',
        'weekly_summary':       'tier2',
        'monthly_summary':      'tier2',
        'reflection':           'tier2',
        'analyze_finance':      'tier2',
    }
```

**Конфиг через env:**
```
AI_TIER1_PROVIDER=deepseek
DEEPSEEK_API_KEY=sk-...
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
DEEPSEEK_MODEL=deepseek-chat

AI_TIER2_PROVIDER=openai
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini
```

**Системный промпт `task_extraction` (для DeepSeek):**

```
Ты ассистент в планере mirror. Преобрази свободный текст пользователя в структуру задачи.

Текущая дата: {{today}} ({{today_dow}})
Часовой пояс пользователя: {{timezone}}

Активные направления пользователя:
{{directions_list}}

Извлеки JSON со следующими полями (в строгом JSON, без markdown):
- is_task (bool): false если сообщение не является задачей (вопрос, болтовня, мат)
- title (str): краткая формулировка без даты, времени и приоритета. Без точки в конце
- direction_hint (str|null): название направления если упомянуто
- due_date (str|null): ISO YYYY-MM-DD. «сегодня», «завтра», «послезавтра», «в среду», «через 3 дня», «15 числа»
- due_time (str|null): HH:MM. «утром»→09:00, «днём»→14:00, «вечером»→19:00
- priority (str): «high» если есть «срочно», «важно», «критично», «!!!»; «low» если «потом», «когда-нибудь»; иначе «normal»
- estimated_minutes (int|null): если указано «на час», «на 30 минут»

Если поле не определено — используй null.

Верни ТОЛЬКО JSON. Никаких пояснений.
```

Используй JSON mode для structured output: `response_format={"type": "json_object"}`.

---

## Pricing

**Seed данные (миграция STAGE 1):**

```python
plans = [
    {"code": "basic_monthly",  "name": "Базовый",      "period": "monthly",
     "price_kop":   79000, "features": {"directions_max": 7, "ai_agent": False}},
    {"code": "basic_yearly",   "name": "Базовый",      "period": "yearly",
     "price_kop":  699000, "features": {"directions_max": 7, "ai_agent": False}},
    {"code": "agent_monthly",  "name": "С AI-агентом", "period": "monthly",
     "price_kop":  190000, "features": {"directions_max": 7, "ai_agent": True}},
    {"code": "agent_yearly",   "name": "С AI-агентом", "period": "yearly",
     "price_kop": 1690000, "features": {"directions_max": 7, "ai_agent": True}},
]
```

**Триал:**
- 7 дней при регистрации (без карты)
- Default plan: `basic_monthly`
- За день до окончания → email + TG
- После → `status='past_due'`, paywall flag в API responses

**Прорейтинг при смене тарифа:**
```
basic_monthly (790₽) → agent_monthly (1900₽) посреди месяца:
1. unused_days = days_left_in_period
2. credit = (790 / 30) * unused_days
3. new_charge = (1900 / 30) * unused_days - credit
4. → платёж в ЮKassa на new_charge
```

Понижение тарифа — в начале следующего периода.

---

## Структура репозитория

```
mirror-backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── deps.py
│   ├── core/
│   │   ├── security.py
│   │   ├── exceptions.py
│   │   ├── pagination.py
│   │   └── timezone.py
│   ├── db/
│   │   ├── base.py
│   │   ├── session.py
│   │   └── models/
│   ├── schemas/
│   ├── services/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── directions/
│   │   ├── goals/
│   │   ├── tasks/
│   │   ├── time_tracking/
│   │   ├── finance/
│   │   ├── subscription/
│   │   ├── ai/
│   │   ├── telegram/
│   │   └── notifications/
│   ├── api/v1/
│   │   ├── auth.py
│   │   ├── users.py
│   │   ├── directions.py
│   │   ├── goals.py
│   │   ├── tasks.py
│   │   ├── finance.py
│   │   ├── archive.py
│   │   ├── subscription.py
│   │   ├── telegram.py
│   │   └── health.py
│   └── jobs/
│       ├── reminders.py
│       ├── daily_summary.py
│       ├── weekly_review.py
│       ├── trial_expiry.py
│       └── purge_deleted.py
├── alembic/
├── tests/
│   ├── conftest.py
│   ├── factories.py
│   ├── api/
│   ├── services/
│   └── jobs/
├── scripts/
│   ├── seed_pricing.py
│   └── check_health.py
├── .env.example
├── .gitignore
├── .pre-commit-config.yaml
├── pyproject.toml
├── ruff.toml
├── mypy.ini
├── alembic.ini
├── Dockerfile
├── docker-compose.yml
├── railway.toml
├── README.md
├── CLAUDE.md  (этот файл)
└── PROGRESS.md  (журнал, ты его создашь и обновляешь)
```

---

## Безопасность

- JWT access TTL: 15 минут
- JWT refresh TTL: 30 дней
- Magic-link TTL: 15 минут, одноразовый
- Refresh token хранится как sha256 hash
- Rate limit: `/auth/*` 5 req/min/IP, `/telegram/webhook` 100 req/min
- Telegram webhook secret в URL
- ЮKassa signature verification обязательна
- CORS: только наш домен
- HTTPS обязателен
- Все секреты только в env
- Pydantic валидация на каждом endpoint

---

## Money handling

Все суммы в **копейках** (int). На границах API:
```python
class TransactionRead(BaseModel):
    amount_kop: int  # 79000
    
    @computed_field
    @property
    def amount_rub(self) -> str:
        return f"{self.amount_kop / 100:.2f}"  # "790.00"
```

В БД — int. В API ответах — int_kop. Format в рубли — на frontend.

---

## Timezones

- БД: `timestamptz` (UTC)
- `due_date`, `due_time` — в TZ юзера, без конвертации
- Cron-напоминания: вычисляем «через 15 минут» в локальном времени, конвертируем в UTC для запроса

---

## Что НЕ делаем в MVP

- ❌ Командные аккаунты (only single-user)
- ❌ OAuth (только magic-link + Telegram)
- ❌ Push notifications (только TG + email)
- ❌ WebSockets реалтайм
- ❌ Mobile apps
- ❌ Импорт из Notion/Trello/Calendar
- ❌ PDF export (только CSV)
- ❌ Сложная аналитика и графики
- ❌ Stripe (только ЮKassa)

Идёт в `TODO_AFTER_MVP.md`.

---

# ЭТАПЫ РАЗРАБОТКИ (16 этапов)

## STAGE 0 — Bootstrap

**Цель:** скелет проекта, инструменты, docker-compose.

**Делаем:**
- Структура папок (см. выше)
- `pyproject.toml` через uv (НЕ poetry, НЕ pip)
- `ruff.toml` (line-length 100, target Python 3.12)
- `mypy.ini` (strict)
- `.pre-commit-config.yaml`
- `Dockerfile` (multi-stage, slim, non-root user)
- `docker-compose.yml` (Postgres 16-alpine + Redis 7-alpine, healthchecks, volumes)
- `.env.example` со всеми переменными
- `.gitignore`
- `app/main.py` с одним endpoint `GET /api/v1/health/live` → `{"status": "ok"}`
- `app/config.py` (pydantic-settings)
- `README.md` (3 команды для запуска)

**Зависимости в pyproject.toml:**
- fastapi
- uvicorn[standard]
- pydantic
- pydantic-settings
- structlog

**Dev:**
- pytest
- pytest-asyncio
- httpx
- ruff
- mypy
- pre-commit

**Acceptance criteria:**
- [ ] `docker compose up -d` поднимает Postgres + Redis
- [ ] `uv run uvicorn app.main:app --reload` стартует
- [ ] `curl http://localhost:8000/api/v1/health/live` → 200
- [ ] `curl http://localhost:8000/docs` → Swagger
- [ ] `ruff check .` — 0 ошибок
- [ ] `mypy app` — 0 ошибок
- [ ] `pre-commit run --all-files` — passes
- [ ] README объясняет 3-командный запуск

**Что показать перед началом:**
1. Список файлов которые планируешь создать
2. Жди моего «ок» прежде чем создавать

**Коммиты:**
- `chore: init project structure`
- `chore: add ruff/mypy/pre-commit configs`
- `chore: add docker-compose for local dev`
- `feat: add health endpoint`

---

## STAGE 1 — DB Schema + Alembic

**Цель:** все 16 моделей + миграция + seed pricing.

**Делаем:**
- Все модели из секции «Схема БД» выше в `app/db/models/`
- SQLAlchemy 2.0 async синтаксис: Mapped, mapped_column
- UUID через `postgresql.UUID(as_uuid=True), default=uuid.uuid4`
- timestamptz через `DateTime(timezone=True), server_default=func.now()`
- jsonb через JSONB
- Все индексы (включая partial `unique partial WHERE is_main = true`)
- Все foreign keys с правильным on delete cascade
- `alembic init alembic` с async config (`async_engine_from_config`)
- Первая миграция через `alembic revision --autogenerate -m "initial schema"`
- `scripts/seed_pricing.py` — вставка 4 базовых тарифов
- Pytest fixture `db_session` в `tests/conftest.py` — async session с rollback

**Сначала покажи мне ОДНУ модель (users)** — проверим стиль. После моего «ок» делай остальные.

**Acceptance criteria:**
- [ ] Все 16 таблиц созданы (`\dt` в psql)
- [ ] `alembic upgrade head` работает на чистой БД
- [ ] `alembic downgrade base` работает (полный откат)
- [ ] `python scripts/seed_pricing.py` вставляет 4 тарифа
- [ ] `SELECT count(*) FROM pricing_plans WHERE active_to IS NULL` → 4
- [ ] mypy не ругается на модели
- [ ] Smoke-тест: создание User через ORM, чтение
- [ ] Constraint test: попытка двух is_main у одного юзера → IntegrityError

**Smoke check после миграций:**
```sql
\dt
SELECT code, period, price_kop FROM pricing_plans ORDER BY price_kop;
```
Должно вывести:
```
basic_monthly  | monthly |   79000
basic_yearly   | yearly  |  699000
agent_monthly  | monthly |  190000
agent_yearly   | yearly  | 1690000
```

---

## STAGE 2 — Auth (Magic-link)

**Цель:** регистрация и логин через email magic-link, JWT, refresh.

**Перед стартом покажи мне:**
1. Структуру `services/auth/` (какие файлы)
2. Логику timing-safe verify токена

**Делаем:**

`services/auth/magic_link.py`:
- `generate_token()` → urlsafe_b64 (32 bytes)
- `create_magic_link(email)` → создаёт User если новый, создаёт AuthMagicLink, возвращает токен
- `verify_magic_link(token)` → проверка expires_at, used_at; помечает used_at; возвращает user_id
- timing-safe сравнение через `secrets.compare_digest`

`services/auth/jwt.py`:
- `issue_access_token(user_id)` — TTL 15 мин
- `issue_refresh_token(user_id, device_info, ip)` — TTL 30 дней, hash в БД
- `verify_access_token(token)` → user_id
- `rotate_refresh_token(refresh_token)` — атомарно: revoke старый, выдать новый

`services/auth/email.py`:
- `send_magic_link(email, token)` через Resend
- В `ENVIRONMENT=dev`: вместо отправки — `log.info("dev_magic_link", token=token, dev_magic_link=True)`

`api/v1/auth.py` — все endpoints (см. секцию API выше).

`deps.py`:
- `get_current_user(token = Depends(oauth2_scheme))` — валидирует JWT, тянет User из БД, кеширует в Redis на 1 минуту

Rate limiting (slowapi):
- `/auth/magic-link/request`: 5/min/IP → 6-й запрос → 429

Mock Resend в тестах через `unittest.mock.patch`.

**Acceptance criteria:**
- [ ] `POST /auth/magic-link/request` с новым email → создаёт User, 200
- [ ] Тот же endpoint с существующим email → НЕ создаёт нового, 200 (security: не палим existence)
- [ ] `POST /auth/magic-link/verify` с валидным токеном → access + refresh + user
- [ ] Просроченный токен → 401
- [ ] Использованный токен → 401 (one-time)
- [ ] `POST /auth/refresh` обновляет токены
- [ ] Logout инвалидирует refresh
- [ ] Rate limit: 6-й запрос → 429
- [ ] 12+ тестов

**Manual check:**
```bash
curl -X POST http://localhost:8000/api/v1/auth/magic-link/request \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com"}'

# Найти токен в логах (structlog json: ищи "dev_magic_link":true)
curl -X POST http://localhost:8000/api/v1/auth/magic-link/verify \
  -H "Content-Type: application/json" \
  -d '{"token":"<token>"}'
```

---

## STAGE 3 — Users + Directions

**Цель:** профиль и направления.

**Делаем:**

`services/users/`:
- `get_user(user_id)`
- `update_user(user_id, name?, avatar_url?, timezone?, locale?)` — только эти поля
- `get_stats(user_id)` → tasks_done, goals_achieved, focus_seconds, streak_days (на этом этапе нули — это ок)

`services/directions/`:
- `list_directions(user_id)` — все active + archived, отсортированные по order_index
- `create_direction(user_id, name, description?, is_main?)` — проверка лимита из `subscription.plan.features.directions_max` (НЕ хардкод 7)
- `update_direction(direction_id, user_id, ...)`
- `archive_direction` / `restore_direction`
- `delete_direction` — только если archived и нет связанных tasks
- `reorder_directions(user_id, order: list[uuid])` — атомарно
- `set_main(direction_id, user_id)` — атомарно: снимает is_main с других + ставит на этот

`is_main` constraint — НЕ через триггер, через partial unique index в миграции.

API из секции выше.

Multi-tenancy: каждый запрос фильтрует по user_id. Юзер A пытается обратиться к direction юзера B → 404 (не 403).

**Acceptance criteria:**
- [ ] `GET /me/` → user + activeSubscription
- [ ] `PATCH /me/` обновляет name/timezone/locale
- [ ] `GET /me/stats` → нули
- [ ] `POST /directions/` создаёт, проверяет лимит
- [ ] При превышении лимита → 422
- [ ] `is_main: true` снимает флаг с других (атомарно)
- [ ] `PATCH /directions/reorder` меняет order_index пакетом
- [ ] Multi-tenancy: юзер A не видит directions юзера B → 404
- [ ] Все queries фильтруют по user_id

---

## STAGE 4 — Goals + Tasks

**Цель:** цели + задачи + view-эндпоинты для day/week/month.

**Сначала кратко опиши план реализации view-эндпоинтов** — особенно как обработаешь timezone юзера.

**Делаем:**

`services/goals/`:
- CRUD
- Validation: `goal.direction_id` должна быть active direction юзера
- `achieve_goal(goal_id)` → status='achieved', achieved_at=now()

`services/tasks/`:
- CRUD
- Validation: `task.goal` должна быть в той же direction что и `task.direction`
- `list_tasks` с фильтрами: status, direction_id, goal_id, due_date_from, due_date_to, priority
- `get_day_tasks(user_id, date)` — задачи на дату в TZ юзера, time-блоки
- `get_week_tasks(user_id, start_date)` — задачи + цели + фокус (топ-3 high priority открытых)
- `get_month_tasks(user_id, year, month)` — задачи + цели + summary по направлениям

**Timezone:**
- `due_date` и `due_time` хранятся в TZ юзера без конвертации
- `/tasks/day/2026-05-04` — это 2026-05-04 целиком в TZ юзера
- `created_at`, `completed_at`, `started_at` — UTC
- Фильтрация по due_date — прямое сравнение DATE без TZ

**Оптимизация запросов:**
- В list-эндпоинтах: `selectinload` для direction и goal
- N+1 проверка: `sqlalchemy.event` для подсчёта queries
- Один list-запрос на 50 задач = максимум 3 SQL queries

**Acceptance criteria:**
- [ ] CRUD goals и tasks
- [ ] Goal validation: `task.goal.direction != task.direction` → 422
- [ ] Day view: задача с due_date=сегодня попадает; due_date=вчера не попадает
- [ ] Day view с TZ: юзер в Europe/Moscow (UTC+3) запрашивает 2026-05-04 → берёт задачи в его локальном 2026-05-04
- [ ] Week view содержит фокус (3 high priority открытых)
- [ ] Month view содержит summary по направлениям
- [ ] Фильтр status=open
- [ ] Фильтр direction_id
- [ ] Soft-delete не показывает в list
- [ ] Archived не показывает в list
- [ ] N+1: count test (max 3 queries)
- [ ] 25+ тестов

---

## STAGE 5 — Time tracking

**Цель:** трекинг времени по задачам.

**Главное:** только одна задача in_progress у юзера. Старт новой → автопауза текущей. **Атомарно в одной транзакции БД.**

**Делаем:**

`services/time_tracking/`:
- `start_task(task_id, user_id)` — атомарно:
  1. Найти текущую in_progress задачу юзера
  2. Если есть — закрыть её TaskTimeLog (ended_at=now(), duration_seconds=...) и status='open'
  3. Открыть новый TaskTimeLog
  4. status='in_progress', started_at=now() если ещё не было
- `pause_task(task_id, user_id)` — закрывает текущий лог, status='open'
- `complete_task(task_id, user_id, completion_note?)` — закрывает лог, status='done', completed_at=now()
- `get_total_time(task_id)` — SUM(duration_seconds) по всем логам

`/me/stats.focus_seconds` — SUM(duration_seconds) всех логов юзера.

**Acceptance criteria:**
- [ ] start новой когда нет in_progress → создаётся лог
- [ ] start когда уже есть in_progress → автопауза + новый лог
- [ ] pause создаёт ended_at
- [ ] complete создаёт ended_at и status=done
- [ ] complete с completion_note сохраняет note
- [ ] duration_seconds правильно считается
- [ ] total_time_seconds на задаче с несколькими логами = SUM
- [ ] focus_seconds в /me/stats отражает все логи юзера
- [ ] Concurrent старт двух задач (asyncio.gather) → одна побеждает
- [ ] 10+ тестов

---

## STAGE 6 — Finance: transactions + budgets

**Цель:** учёт финансов.

**Сначала покажи мне формулу `/finance/summary`** — как считаешь tempo и forecast.

**Делаем:**

`services/finance/transactions/`:
- CRUD transactions
- Soft-delete (deleted_at)
- list с фильтрами: type, direction_id, period (month/year/all), date_from/to

`services/finance/budgets/`:
- `list_budgets(user_id, year)`
- `upsert_budget(user_id, direction_id, period, planned_income_kop)` — INSERT ... ON CONFLICT ... UPDATE

`services/finance/summary/`:
- `get_yearly_summary(user_id, year)`:
  - `target_kop` = SUM(goals.target_amount_kop WHERE period_type='year' AND period_start in year)
  - `fact_kop` = SUM(transactions.amount_kop WHERE type='income' AND year=year AND deleted_at IS NULL)
  - `months_passed` = текущий месяц года, если запрашиваемый = текущий; иначе 12
  - `tempo_kop` = fact_kop / months_passed (если > 0)
  - `forecast_kop` = tempo_kop * 12
  - `delta_to_target_kop` = target_kop - forecast_kop

**Acceptance criteria:**
- [ ] CRUD transactions
- [ ] 4 income по 100к в Q1 2026, target 1.5M → fact=400к, tempo=100к, forecast=1.2M, delta=300к
- [ ] Soft-delete не учитывается в summary
- [ ] Запрос за прошлый год → tempo на 12 месяцев
- [ ] Запрос за текущий год → tempo на прошедшие месяцы
- [ ] Budgets upsert: первый раз INSERT, второй UPDATE
- [ ] Multi-tenancy

---

## STAGE 7 — Finance: Pipeline (Deals)

**Цель:** воронка сделок.

**Делаем:**

`services/finance/deals/`:
- CRUD
- `move_deal_stage(deal_id, user_id, new_stage)` — валидация переходов:
  - lead → qualified
  - qualified → proposal
  - proposal → closed_won | closed_lost
  - closed_won/closed_lost → нельзя обратно
  - При нарушении → 422

- `close_won(deal_id, user_id)` — атомарно в одной транзакции БД:
  1. Обновить deal: stage='closed_won', closed_at=now(), probability=100
  2. Создать transaction: type='income', amount_kop=deal.amount_kop, direction_id=deal.direction_id, related_deal_id=deal.id, transaction_date=today, title=f"Сделка: {deal.title}"
  3. Если ошибка — откат всей транзакции

- `close_lost(deal_id, user_id)` — stage='closed_lost', closed_at=now(), probability=0

`services/finance/pipeline/`:
- `get_pipeline(user_id, direction_id?)` → группировка по stage:
  - `{stage: {count, total_kop, weighted_kop, deals: [...]}}`
  - `weighted_kop` = SUM(amount_kop * probability / 100)

- `get_pipeline_metrics(user_id)` → `{total_pipeline_kop, total_weighted_kop, avg_check_kop, conversion_rate}`

**Acceptance criteria:**
- [ ] CRUD deals
- [ ] Move stage валидные переходы работают
- [ ] Move в closed → попытка вернуть → 422
- [ ] close_won создаёт transaction атомарно
- [ ] Тест на rollback: мокаем создание transaction чтобы упало → deal не должен измениться
- [ ] Pipeline grouping
- [ ] Pipeline weighted = SUM(amount * probability/100)
- [ ] Multi-tenancy

---

## STAGE 8 — Archive

**Цель:** архив с restore и hard-delete.

**Делаем:**

`services/archive/`:
- `list_archived(user_id, period: 'month'|'quarter'|'year'|'all', type: 'tasks'|'goals'|'deals'|'transactions')` — unified API
- `get_year_ago(user_id)` — задача из прошлого года в этот же день (random если несколько)
- `get_tag_cloud(user_id)` → направления + сезоны (зима/весна/лето/осень) с count
- `restore_item(type, id, user_id)`:
  - tasks: status='archived' → 'open'
  - goals: status='archived' → 'active'
  - deals: deleted_at=NULL
  - transactions: deleted_at=NULL
- `permanent_delete(type, id, user_id, confirmation_phrase)` — hard delete:
  - confirmation_phrase должен быть точно "удалить навсегда" (case-sensitive)
  - Без правильной фразы → 422
  - Связанные транзакции/сделки НЕ удаляются (related_task_id → NULL)

**Acceptance criteria:**
- [ ] list по type
- [ ] list по period
- [ ] restore tasks
- [ ] restore goals
- [ ] restore deals (deleted_at=NULL)
- [ ] permanent_delete с правильной фразой → 204, физически удалено
- [ ] permanent_delete без фразы → 422
- [ ] permanent_delete с "Удалить навсегда" (большая) → 422
- [ ] permanent_delete task с related transaction → транзакция остаётся, related_task_id=NULL
- [ ] year_ago возвращает задачу
- [ ] tag_cloud показывает направления + сезоны

---

## STAGE 9 — Subscriptions + ЮKassa

**Цель:** платная подписка через ЮKassa.

**Перед стартом покажи мне:**
1. Как хранишь ЮKassa secret (env переменные)
2. Как тестируешь webhook (signed payload mock)
3. Логику триал-периода
4. Поведение при истёкшем триале (response code, paywall flag)

**Делаем:**

`services/subscription/`:
- `get_active_subscription(user_id)`
- `create_trial(user_id)` — при регистрации, status='trialing', trial_ends_at=now+7d, plan=basic_monthly
- `change_plan(user_id, plan_code, idempotency_key, promo_code?)` — расчёт прорейтинга, создание payment в ЮKassa, return confirmation_url
- `cancel(user_id)` — cancel_at_period_end=True
- `resume(user_id)` — cancel_at_period_end=False

**Прорейтинг (basic_monthly → agent_monthly посреди периода):**
- `unused_days = (current_period_end - now).days`
- `daily_old = old_plan.price_kop / 30`
- `daily_new = new_plan.price_kop / 30`
- `credit = daily_old * unused_days`
- `charge = daily_new * unused_days - credit`

`services/payments/yookassa.py`:
- `create_payment(amount_kop, description, return_url, idempotency_key, metadata)`
- `verify_webhook_signature(headers, body)` — HMAC SHA1 с YOOKASSA_SECRET_KEY
- `handle_webhook(event)` — idempotent через `payment.provider_id`
  - payment.succeeded → продлить subscription, AuditLog
  - payment.canceled → status='past_due', notify
  - payment.refunded → handle (твоё решение, обоснуй)

`jobs/trial_expiry.py`:
- Daily cron в 09:00 UTC
- Найти subscriptions с trial_ends_at в течение 24 часов → нотификация
- Отдельно: trial_ends_at < now() AND status='trialing' → status='past_due'

`deps.py`:
- `check_subscription_active(user)` — для защиты endpoints
- При status='past_due' → 402 Payment Required с paywall flag

**Acceptance criteria:**
- [ ] `/subscription/plans` → 4 активных
- [ ] При регистрации создаётся trial subscription
- [ ] `/subscription/change` создаёт payment в ЮKassa (мок), возвращает confirmation_url
- [ ] Webhook с правильной signature и payment.succeeded → продлевает subscription
- [ ] Webhook с неправильной signature → 401
- [ ] Webhook с тем же event_id повторно → noop
- [ ] Прорейтинг: basic_monthly → agent_monthly через 15 дней → правильная сумма charge
- [ ] Триал истёк → status='past_due', endpoints возвращают 402
- [ ] cancel: cancel_at_period_end=True, доступ остаётся до конца периода
- [ ] resume отменяет cancel
- [ ] Мок ЮKassa: класс FakeYooKassaClient с predefined responses

---

## STAGE 10 — Telegram: webhook + login

**Цель:** Telegram-бот, привязка аккаунта, базовые команды (без AI).

**Делаем:**

`services/telegram/`:
- `bot.py` — aiogram Bot и Dispatcher setup
- `handlers/start.py` — /start
- `handlers/today.py` — /today
- `handlers/week.py` — /week
- `handlers/help.py` — /help
- `handlers/text.py` — обработчик текста (на этом этапе → инструкция, AI пока не подключаем)
- `middleware.py` — проверка привязки + тарифа

**Webhook:**
- POST `/api/v1/telegram/webhook/:secret`
- Secret сравнивается с `TELEGRAM_WEBHOOK_SECRET` через `secrets.compare_digest`
- Подделка → 401
- Парсит aiogram.types.Update, диспатчит

**Команды:**

`/start` без привязки:
- Генерирует одноразовый токен (TTL 15 мин)
- Отвечает: «Привет! Чтобы привязать аккаунт mirror, открой:\nhttps://mirror.app/link?token={token}»

`/start` с привязкой:
- «С возвращением, {name}! Используй /today, /week, /help»

`/today` без привязки → инструкция «привяжи аккаунт»
`/today` с привязкой → список задач сегодня

`/week` — задачи недели + фокус

`/help` — список команд

Любой текст без привязки → /start hint
Любой текст с привязкой и БЕЗ AI-агента в тарифе → «AI-агент доступен в тарифе «С AI-агентом» (1 900 ₽/мес)»
Любой текст с привязкой и С AI-агентом → «(STAGE 11) пока заглушка»

**Endpoint POST /api/v1/telegram/link:**
- Принимает `{login_token, tg_init_data}`
- Валидирует tg_init_data (HMAC от Telegram bot token)
- Проверяет login_token (одноразовый, по нему находит email юзера)
- Привязывает: `User.telegram_id`, `telegram_username`
- Возвращает 204

**Acceptance criteria:**
- [ ] Webhook с неправильным secret → 401
- [ ] Webhook с правильным secret и /start → 200, бот ответил
- [ ] /today без привязки → инструкция
- [ ] /today с привязкой → задачи (мок 3 задачи в БД, проверяем что они в ответе)
- [ ] Любой текст без AI-тарифа → сообщение про тариф
- [ ] Link endpoint: правильный login_token + tg_init_data → 204, telegram_id set
- [ ] Link с invalid tg_init_data signature → 401
- [ ] Тесты используют моки `aiogram.Bot.send_message`

---

## STAGE 11 — AI слой (двухуровневый)

**Цель:** AIRouter + DeepSeek (Tier 1) + OpenAI (Tier 2).

**Делаем:**

`services/ai/base.py`:
- Абстракт `AIProvider`:
  - `extract_task(message: str, context: TaskContext) -> ExtractedTask`
  - `parse_command(message: str, context: CommandContext) -> ParsedCommand`
  - `generate_reminder_text(task: Task) -> str`
  - `generate_weekly_summary(data: WeekData) -> str`
  - `generate_monthly_summary(data: MonthData) -> str`
  - `generate_reflection(prompt: str, data: dict) -> str`

`services/ai/providers/deepseek.py`:
- `DeepSeekProvider(AIProvider)`
- Использует `AsyncOpenAI` клиент с `base_url=https://api.deepseek.com/v1`, `api_key=DEEPSEEK_API_KEY`
- model='deepseek-chat'
- JSON mode для structured output
- Логирует в `ai_messages`: provider, model, operation, tokens, duration, cost

`services/ai/providers/openai.py`:
- `OpenAIProvider(AIProvider)`
- AsyncOpenAI с api_key=OPENAI_API_KEY
- model='gpt-4o-mini'
- Аналогично deepseek

`services/ai/router.py`:
- `AIRouter`:
  - `tier1: AIProvider` — из `AI_TIER1_PROVIDER` env (deepseek)
  - `tier2: AIProvider` — из `AI_TIER2_PROVIDER` env (openai)
  - Все методы дублируют интерфейс AIProvider, маршрутизируют

`services/ai/prompts/task_extraction.py`:
- Системный промпт ровно как в секции «AI-слой архитектура» выше

`services/ai/schemas.py`:
- `ExtractedTask` (Pydantic):
  - is_task: bool
  - title: str | None
  - direction_hint: str | None
  - due_date: date | None
  - due_time: time | None
  - priority: Literal['low', 'normal', 'high'] = 'normal'
  - estimated_minutes: int | None
- `TaskContext`: user_directions, today, today_dow, timezone

`services/ai/exceptions.py`:
- AIError, AITimeoutError, AIRateLimitError, AIInvalidResponseError

**Direction matching (fuzzy):**
- В `services/directions/match.py`:
  - `find_direction_by_hint(user_id, hint: str) -> Direction | None`
  - Используй `rapidfuzz` для fuzzy match по name (учитывает падежи)
  - Threshold 70% — иначе None

**Интеграция в Telegram (handlers/text.py):**
- Если юзер привязан и AI-агент в тарифе:
  - text → `ai_router.extract_task(text, context)`
  - is_task=False → «Не похоже на задачу. Попробуй сформулировать иначе.»
  - is_task=True:
    - find direction по hint
    - create task с source='telegram'
    - Ответ: «✓ задача создана\n\n{direction.name} · {due_date} {due_time} · {priority}\n«{title}»»

**FakeAIProvider для тестов:**
- В `tests/fakes/ai.py`
- Принимает predefined_responses в конструкторе
- Все методы возвращают из предзаданного списка

**Acceptance criteria:**
- [ ] AIRouter с конфигом из env
- [ ] extract_task через DeepSeek работает на тестовых сообщениях:
  - «завтра в 10 встреча с Алексеем по брендингу студии важно» → high priority, due_time='10:00', direction_hint='студия', due_date=tomorrow
  - «купить молоко» → title='Купить молоко', остальное None
  - «срочно сдать отчёт по налогам в среду» → high priority, due_date=ближайшая среда
  - «что у нас сегодня?» → is_task=False
- [ ] Direction matching: «студия»/«студии»/«в студии» → одна direction
- [ ] Hint не найден → задача без direction_id
- [ ] Лог в ai_messages: provider, model, tokens, duration, cost_usd
- [ ] Все тесты с FakeAIProvider
- [ ] Один integration test под `pytest --integration` для реального DeepSeek API
- [ ] Stub для Tier 2 методов (generate_*_summary) — реализован но не используется в jobs (это в STAGE 12)

---

## STAGE 12 — Notifications + AI summaries

**Цель:** напоминания + ежедневные/недельные сводки через AI Tier 2.

**Делаем:**

APScheduler в lifespan FastAPI (startup/shutdown).

`jobs/reminders.py`:
- Каждые 5 минут (cron `*/5`)
- Найти tasks где:
  - status='open'
  - due_date = today (в TZ юзера)
  - due_time IS NOT NULL
  - now() в TZ юзера >= due_time - 15 минут И now() < due_time
  - last_reminded_at IS NULL OR last_reminded_at < (due_time - 15 minutes)
- Для каждой:
  - Проверить quiet_hours (с учётом перехода через полночь)
  - Сгенерировать текст через `ai_router.generate_reminder_text(task)` (Tier 1 = DeepSeek)
  - Отправить в TG (если delivery_channel='telegram')
  - Установить last_reminded_at = now()

`jobs/daily_summary.py`:
- Каждый день, динамически берёт время из `notifications_settings.daily_summary_time` для каждого юзера
- Реализация: cron каждый час → найти юзеров чьё время сейчас, отправить
- Текст через `ai_router.generate_daily_summary(data)` → Tier 2 = OpenAI gpt-4o-mini
- data: `{tasks_done_today, tasks_open, tasks_tomorrow, ...}`

`jobs/weekly_review.py`:
- Каждое воскресенье в weekly_review_time (по умолчанию 19:00 в TZ юзера)
- `ai_router.generate_weekly_summary(data)` → Tier 2
- data: `{goals_achievements, tasks_summary, focus_minutes, finance_summary, ...}`

**Quiet hours логика:**
- now_local = current time in TZ
- Если start < end: блокируем когда start <= now_local < end
- Если start > end (через полночь): блокируем когда now_local >= start ИЛИ now_local < end

**Промпты (services/ai/prompts/):**
- `daily_summary.py` — кратко, с цифрами, без воды
- `weekly_summary.py` — анализ + предложения
- `reminder_text.py` — короткое напоминание, разнообразие формулировок

API:
- `/me/notifications` — GET и PATCH

**Acceptance criteria:**
- [ ] Reminder за 15 минут до due_time приходит в TG (с time mocking через freezegun)
- [ ] Quiet hours блокируют отправку
- [ ] Quiet hours через полночь (23:00..08:00) тоже работают
- [ ] last_reminded_at защищает от дубликатов
- [ ] Daily summary вызывает Tier 2 (через AIRouter, FakeAIProvider в тестах)
- [ ] Weekly summary вызывает Tier 2
- [ ] Можно отключить через PATCH /me/notifications {daily_summary_enabled: false}

---

## STAGE 13 — Export + GDPR

**Цель:** экспорт данных + правильное удаление аккаунта.

**Делаем:**

`services/export/`:
- `export_user_data(user_id) -> AsyncIterator[bytes]` — streaming zip
- Внутри zip: directions.csv, goals.csv, tasks.csv, transactions.csv, deals.csv, ai_messages.csv
- CSV: UTF-8 с BOM (`\ufeff`), заголовки на русском, разделитель `;`

API:
- `GET /api/v1/me/export` → StreamingResponse с media_type='application/zip', filename='mirror_export_{date}.zip'

`services/auth/account_delete.py`:
- `delete_account(user_id, confirmation_phrase)`:
  - confirmation_phrase должен быть «удалить аккаунт навсегда» (case-sensitive)
  - Без правильной фразы → 422
  - Soft-delete: User.deleted_at = now()
  - Audit log
  - Все sessions revoke
  - Уведомление на email + magic-link для отмены

`DELETE /api/v1/auth/account` → endpoint

`jobs/purge_deleted.py`:
- Daily cron в 03:00 UTC
- Найти users где deleted_at < now() - 30 days
- Hard DELETE (cascade удалит всё связанное)
- Audit log: «account.purged»

`cancel_account_deletion(user_id)`:
- При login через magic-link — если user.deleted_at IS NOT NULL и < 30 days назад → восстановить (deleted_at=NULL)

**Acceptance criteria:**
- [ ] Export содержит все CSV
- [ ] CSV открывается в Excel (BOM, ; разделитель)
- [ ] Account delete с правильной фразой → 204, deleted_at set
- [ ] Account delete с неправильной → 422
- [ ] Login через magic-link на удалённый в 30 дней → восстанавливает
- [ ] Login после purge → создаётся новый юзер
- [ ] Purge job удаляет старые
- [ ] Purge не удаляет недавние (< 30 дней)

---

## STAGE 14 — Production deploy на Railway

**Цель:** деплой на Railway, домен, мониторинг.

**Делаем:**

`railway.toml`:
```toml
[build]
builder = "DOCKERFILE"
dockerfilePath = "Dockerfile"

[deploy]
startCommand = "uvicorn app.main:app --host 0.0.0.0 --port $PORT"
preDeployCommand = "alembic upgrade head"
healthcheckPath = "/api/v1/health/ready"
healthcheckTimeout = 30
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 10
```

`Dockerfile` (multi-stage):
- Stage 1 (builder): python:3.12-slim + uv + sync dependencies
- Stage 2 (runtime): python:3.12-slim + copy from builder + non-root user
- EXPOSE 8000
- HEALTHCHECK через curl /api/v1/health/live

`services/monitoring/sentry.py`:
- `init_sentry(dsn, environment, traces_sample_rate=0.1)`
- before_send hook для фильтрации PII:
  - удалять request.cookies, request.headers.authorization
  - удалять любые поля содержащие 'token', 'password', 'secret'

`api/v1/health/ready`:
- Проверка БД: SELECT 1
- Проверка Redis: ping
- Если что-то fail → 503

Логи в проде через structlog — JSON формат:
- `ENVIRONMENT=production` → json renderer
- dev → ConsoleRenderer

**Чеклист от тебя пользователю (опубликовать в README):**

Что нужно сделать в Railway dashboard руками:
1. New Project → Deploy from GitHub Repo → mirror-backend
2. Add Service → Database → PostgreSQL
3. Add Service → Database → Redis
4. Service Variables (на основном сервисе):
   - DATABASE_URL = ${{ Postgres.DATABASE_URL }}
   - REDIS_URL = ${{ Redis.REDIS_URL }}
   - JWT_SECRET = (openssl rand -hex 32)
   - DEEPSEEK_API_KEY = ...
   - OPENAI_API_KEY = ...
   - TELEGRAM_BOT_TOKEN = ...
   - TELEGRAM_WEBHOOK_SECRET = (openssl rand -hex 32)
   - YOOKASSA_SHOP_ID = ...
   - YOOKASSA_SECRET_KEY = ...
   - RESEND_API_KEY = ...
   - SENTRY_DSN = ...
   - ENVIRONMENT = production
5. Generate Domain (Railway public URL)
6. Set webhook на Telegram: `curl https://api.telegram.org/bot{token}/setWebhook?url=https://{railway_url}/api/v1/telegram/webhook/{secret}`

**Acceptance criteria:**
- [ ] Локальная сборка Docker работает
- [ ] /health/ready проверяет БД и Redis
- [ ] Sentry инициализируется
- [ ] before_send фильтрует authorization header
- [ ] structlog JSON renderer в production
- [ ] Smoke test на проде (от пользователя): magic-link → verify → создать direction

---

## STAGE 15 — Полировка + документация

**Цель:** причесать всё перед релизом.

**Делаем:**

1. Просмотр всех endpoints — единый формат ошибок:
- Все ошибки через `ApiError(status_code, message, error_code, details?)`
- В Swagger каждый endpoint показывает возможные коды
- Стандартизированный response: `{error: {code, message, details}}`

2. OpenAPI документация:
- В каждом endpoint: summary, description, response_model
- Примеры в Pydantic schemas через `Field(example=...)`
- Tags для группировки

3. README.md обновить:
- Как поднять локально (3 команды)
- Как запустить тесты
- Как контрибьютить
- Архитектура (диаграмма ASCII)

4. CHANGELOG.md:
- v1.0.0 — Initial release
- Список фич

5. Security audit:
- Все endpoints под auth (whitelist: health, auth/*, telegram/webhook/*)
- Rate limits на публичных endpoints
- SQL injection: все queries через ORM
- XSS в email шаблонах: escape всё
- Secrets: только в env

6. TODO_AFTER_MVP.md — собрать все TODO из кода

7. Performance:
- `pytest --durations=20` — топ медленных
- Тест > 1 сек → оптимизировать

**Acceptance criteria:**
- [ ] Swagger UI читабельный с примерами
- [ ] README актуальный
- [ ] CHANGELOG.md создан
- [ ] Все TODO в коде либо решены, либо в TODO_AFTER_MVP.md
- [ ] Tests + ruff + mypy проходят

Конец. Backend готов к production.

---

# КОНЕЦ CLAUDE.md

Прочитал? Скажи кратко что понял, и предложи начать STAGE 0.
