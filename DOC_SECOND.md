# API Documentation

## Общее описание

API биллинговой системы предоставляет endpoints для управления подписками, тарифами, платежами и администрированием. Все запросы, кроме `/token` и `/register`, требуют JWT-аутентификации.

**Base URL:** `http://127.0.0.1:8001`


## Глобальные структуры данных

### Объект `Tariff`

| Поле | Тип | Описание |
|------|-----|----------|
| `id` | integer | Уникальный идентификатор |
| `name` | string | Название тарифа |
| `price` | float | Цена за период |
| `period_months` | integer | Период действия в месяцах |
| `trial_days` | integer | Длительность пробного периода (0 — нет) |
| `is_archived` | boolean | Архивирован ли тариф |
| `created_at` | datetime | Дата создания |

### Объект `Subscription`

| Поле | Тип | Описание |
|------|-----|----------|
| `id` | integer | Уникальный идентификатор |
| `user_id` | integer | ID пользователя |
| `tariff_id` | integer | ID выбранного тарифа |
| `status` | enum | Статус подписки |
| `start_date` | datetime | Дата начала |
| `trial_end_date` | datetime | Дата окончания пробного периода |
| `next_billing_date` | datetime | Дата следующего списания |
| `end_date` | datetime | Дата окончания подписки |
| `auto_renew` | boolean | Автопродление |
| `retry_count` | integer | Количество неудачных попыток |
| `next_retry_date` | datetime | Дата следующей попытки списания |
| `created_at` | datetime | Дата создания |
| `updated_at` | datetime | Дата последнего обновления |

### Объект `Payment`

| Поле | Тип | Описание |
|------|-----|----------|
| `id` | integer | Уникальный идентификатор |
| `subscription_id` | integer | ID подписки |
| `amount` | float | Сумма списания |
| `status` | enum | Статус платежа |
| `payment_date` | datetime | Дата платежа |
| `tariff_name` | string | Название тарифа на момент списания |
| `description` | string | Описание операции |

### Перечисления

**Статусы подписок (`subscription_status`):**
- `trial` — пробный период
- `active` — активна
- `paused` — приостановлена
- `cancelled` — отменена
- `payment_error` — ошибка оплаты
- `expired` — истекла

**Статусы платежей (`payment_status`):**
- `pending` — в обработке
- `paid` — оплачено
- `failed` — ошибка
- `prorated` — пропорциональный пересчёт


## Общие правила

### Аутентификация

Аутентификация осуществляется через JWT-токены.

**Алгоритм:**
1. Получить токен через `POST /token`
2. Передавать токен в заголовке: `Authorization: Bearer {token}`

**Ошибки:**
- `401 Unauthorized` — токен отсутствует или невалиден
- `403 Forbidden` — недостаточно прав (требуется роль администратора)

### Валидация

Все входные данные проходят серверную валидацию:
- Обязательные поля проверяются на наличие
- Числовые поля проверяются на допустимый диапазон
- Даты проверяются на корректность

**Формат дат:** `YYYY-MM-DD` или `YYYY-MM-DDTHH:MM:SS`


## Эндпоинты

### Аутентификация

#### Регистрация
POST /register

text

**Параметры (query):**
| Параметр | Тип | Обязательный | Описание |
|----------|-----|--------------|----------|
| `email` | string | ✅ | Email пользователя |
| `password` | string | ✅ | Пароль |

**Ответ (200):**
```json
{
  "message": "Пользователь создан",
  "user_id": 1
}
Ошибки:

400 — пользователь уже существует

Вход
text
POST /token
Параметры (form-data):

Параметр	Тип	Обязательный	Описание
username	string	✅	Email
password	string	✅	Пароль
Ответ (200):

json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
Ошибки:

401 — неверный email или пароль

Получить текущего пользователя
text
GET /users/me
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "id": 1,
  "email": "user@example.com",
  "is_admin": false
}
Тарифы
Получить список тарифов
text
GET /tariffs
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
[
  {
    "id": 1,
    "name": "Старт",
    "price": 490,
    "period_months": 1,
    "trial_days": 0,
    "is_archived": false,
    "created_at": "2026-06-18T10:00:00"
  }
]
Создать тариф
text
POST /tariffs
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Параметры (query):

Параметр	Тип	Обязательный	Описание
name	string	✅	Название
price	float	✅	Цена
period_months	integer	❌	Период (по умолчанию 1)
trial_days	integer	❌	Пробный период (по умолчанию 0)
Ответ (200):

json
{
  "id": 3,
  "name": "Корпоративный",
  "price": 4900,
  "period_months": 1,
  "trial_days": 0,
  "is_archived": false,
  "created_at": "2026-06-18T10:00:00"
}
Архивировать тариф
text
DELETE /tariffs/{tariff_id}
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Тариф архивирован"
}
Подписки
Оформить подписку
text
POST /subscriptions
Заголовки: Authorization: Bearer {token}

Параметры (query):

Параметр	Тип	Обязательный	Описание
tariff_id	integer	✅	ID тарифа
Ответ (200):

json
{
  "id": 1,
  "user_id": 1,
  "tariff_id": 2,
  "status": "trial",
  "start_date": "2026-06-18T10:00:00",
  "next_billing_date": "2026-07-02T10:00:00"
}
Ошибки:

400 — уже есть активная подписка

404 — тариф не найден

Получить текущую подписку
text
GET /subscriptions/me
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "id": 1,
  "user_id": 1,
  "tariff_id": 2,
  "tariff": {
    "id": 2,
    "name": "Бизнес",
    "price": 1490
  },
  "status": "active",
  "start_date": "2026-06-18T10:00:00",
  "next_billing_date": "2026-07-18T10:00:00",
  "auto_renew": true
}
Ошибки:

404 — подписка не найдена

Отменить подписку
text
PUT /subscriptions/{subscription_id}/cancel
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Подписка отменена",
  "subscription": { ... }
}
Приостановить подписку
text
PUT /subscriptions/{subscription_id}/pause
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Подписка приостановлена",
  "subscription": { ... }
}
Возобновить подписку
text
PUT /subscriptions/{subscription_id}/resume
Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Подписка возобновлена",
  "subscription": { ... }
}
Сменить тариф (с пропорциональным пересчётом)
text
PUT /subscriptions/{subscription_id}/change-tariff
Заголовки: Authorization: Bearer {token}

Параметры (query):

Параметр	Тип	Обязательный	Описание
new_tariff_id	integer	✅	ID нового тарифа
Ответ (200):

json
{
  "message": "Тариф успешно изменён",
  "subscription": { ... },
  "proration": {
    "refund": 326.67,
    "extra_charge": 993.33,
    "difference": 666.66,
    "days_remaining": 20
  }
}
Платежи
Получить историю платежей
text
GET /payments
Заголовки: Authorization: Bearer {token}

Параметры (query):

Параметр	Тип	Обязательный	Описание
subscription_id	integer	✅	ID подписки
start_date	string	❌	Начало периода (YYYY-MM-DD)
end_date	string	❌	Конец периода (YYYY-MM-DD)
Ответ (200):

json
[
  {
    "id": 1,
    "amount": 1490,
    "status": "paid",
    "payment_date": "2026-06-18T10:00:00",
    "tariff_name": "Бизнес",
    "description": "Ежемесячное списание"
  }
]
Администрирование
Все подписки
text
GET /admin/subscriptions
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Ответ (200): массив объектов Subscription

Отчёт по выручке
text
GET /admin/revenue
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Параметры (query):

Параметр	Тип	Обязательный	Описание
start_date	string	❌	Начало периода
end_date	string	❌	Конец периода
Ответ (200):

json
{
  "total_revenue": 14900,
  "payments_count": 10,
  "start_date": "2026-01-01",
  "end_date": "2026-06-18",
  "monthly": [
    { "month": "Jun 2026", "total": 5000, "count": 3 }
  ]
}
Ручная приостановка подписки (админ)
text
PUT /admin/subscriptions/{subscription_id}/pause
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Подписка приостановлена",
  "subscription": { ... }
}
Ручная отмена подписки (админ)
text
PUT /admin/subscriptions/{subscription_id}/cancel
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Подписка отменена",
  "subscription": { ... }
}
Запуск Dunning-процесса
text
POST /admin/run-dunning
Доступ: только администратор

Заголовки: Authorization: Bearer {token}

Ответ (200):

json
{
  "message": "Dunning-процесс запущен"
}
Коды ошибок
Код	Описание
200	Успешный запрос
400	Неверные параметры запроса
401	Требуется аутентификация
403	Недостаточно прав
404	Объект не найден
422	Ошибка валидации данных
500	Внутренняя ошибка сервера
