# №14: Основные архитектурные представления (Architecture Views)

---

**Назначение раздела:** Архитектурные представления описывают систему с разных точек зрения (viewpoints) для различных групп стейкхолдеров. В соответствии с заданием представлены пять основных представлений: функциональное, информационное, многозадачность (concurrency), инфраструктурное и безопасность. Каждое представление фокусируется на определённых аспектах системы и использует язык и уровень детализации, понятный соответствующей аудитории.

---

## 1. Функциональное представление (Functional View)

**Назначение:** Описывает, как система реализует функциональные требования, какие компоненты существуют, как они взаимодействуют и какие интерфейсы предоставляют.

### 1.1. Домены и их ответственность

| Домен | Ответственность | Ключевые функции | Взаимодействие с другими доменами |
|:---|:---|:---|:---|
| **Identity** | Аутентификация и управление доступом | Регистрация, вход (OAuth 2.0), MFA, управление сессиями, RBAC | ↔ Все домены (через API Gateway) |
| **User Profile** | Управление профилями, настройками и инвентарём | Профили, настройки приватности, инвентарь (обувь, снаряды), предпочтения | ↔ Identity, ↔ Training, ↔ Recommendation, ↔ Commerce |
| **Training** | Трекинг тренировок, хранение и анализ активности | Запись тренировок (GPS/пульс), история, календарь, сравнение с собой/регионом/профи | ↔ Device Integration, ↔ Recommendation, ↔ Gamification, ↔ Activity Feed |
| **Device Integration** | Подключение и интеграция внешних устройств и сервисов | BLE/ANT+ устройства, импорт из Garmin/Strava/Apple Health/Google Fit | ↔ Training, ↔ User Profile |
| **Social** | Социальные функции, связи между пользователями | Группы по интересам, друзья, подписки, чаты, поиск людей | ↔ Activity Feed, ↔ Notification, ↔ Gamification |
| **Activity Feed** | Лента активности и обновлений | Лента друзей, лента групп, публикации, комментарии, репосты | ↔ Social, ↔ Training, ↔ Gamification, ↔ Notification |
| **Gamification** | Игровые механики и мотивация | Уровни и XP, достижения (ачивки), виртуальные награды, индивидуальные челленджи | ↔ Training, ↔ Social, ↔ Competition, ↔ Notification |
| **Competition** | Массовые соревнования | Командные челленджи, глобальные рейтинги, турниры, сравнение с профессиональными спортсменами | ↔ Gamification, ↔ Training, ↔ Social |
| **Recommendation** | Персонализированные рекомендации и AI-коучинг | Генерация тренировочных планов, динамическая корректировка, рекомендации по замене инвентаря | ↔ Training, ↔ User Profile, ↔ Device Integration, ↔ Commerce, ↔ Promotion |
| **Notification** | Уведомления пользователей | Push-уведомления (Firebase/APNS), email-уведомления, in-app уведомления | ↔ Social, ↔ Activity Feed, ↔ Gamification, ↔ Competition, ↔ Promotion |
| **Promotion** | Промоакции, новости и B2B-платформа | Региональные промоакции, новости спорта, админ-панель для партнёров, публичное API, календарь событий | ↔ Notification, ↔ Recommendation, ↔ Commerce, ↔ Identity |
| **Commerce** | Монетизация и интеграция с продажами | Подписки, платежи (Stripe/Apple Pay/Google Pay), бесшовная интеграция с legacy e-commerce, промокоды | ↔ User Profile, ↔ Recommendation, ↔ Promotion, ↔ Identity |

---

### 1.2. Взаимодействие доменов (Основные потоки)

| Поток | Описание | Участники (домены) | Протокол |
|:---|:---|:---|:---|
| **Запись тренировки** | Пользователь записывает тренировку с датчиками | Client → Identity → Training → Device Integration | REST/WebSocket |
| **Генерация AI-плана** | Пользователь запрашивает персонализированный план | Client → Identity → Recommendation → Training | REST |
| **Обновление инвентаря** | Тренировка завершена → обновление пробега | Training → Recommendation → User Profile | Асинхронный (Kafka) |
| **Начисление XP** | Тренировка завершена → начисление опыта | Training → Gamification | Асинхронный (Kafka) |
| **Публикация в ленте** | Тренировка завершена → обновление ленты друзей | Training → Activity Feed | Асинхронный (Kafka) |
| **Уведомление друзей** | Достижение побито → уведомление друзей | Gamification → Notification → Social | Асинхронный (Kafka) |
| **Рекомендация инвентаря** | Пробег превысил лимит → рекомендация | User Profile → Recommendation → Commerce | Синхронный (REST) |
| **Покупка инвентаря** | Пользователь покупает товар | Client → Commerce → Anti-Corruption Layer (legacy) | REST / SOAP |
| **Создание промоакции** | Партнёр создаёт региональную акцию | Client → Promotion | REST |
| **Регистрация на событие** | Пользователь регистрируется на забег | Client → Promotion → Commerce | REST |
| **Участие в челлендже** | Пользователь участвует в соревновании | Client → Competition → Training | REST |
| **Поиск пользователей** | Пользователь ищет людей рядом | Client → Social → Activity Feed | REST |

---

### 1.3. Матрица взаимодействия доменов

| → | Identity | User Profile | Training | Device Integration | Social | Activity Feed | Gamification | Competition | Recommendation | Notification | Promotion | Commerce |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Identity** | — | ✅ | — | — | — | — | — | — | — | — | ✅ | ✅ |
| **User Profile** | ✅ | — | ✅ | — | — | — | — | — | ✅ | — | — | ✅ |
| **Training** | — | ✅ | — | ✅ | — | ✅ | ✅ | ✅ | ✅ | — | — | — |
| **Device Integration** | — | — | ✅ | — | — | — | — | — | — | — | — | — |
| **Social** | — | — | — | — | — | ✅ | ✅ | ✅ | — | ✅ | — | — |
| **Activity Feed** | — | — | ✅ | — | ✅ | — | ✅ | — | — | ✅ | — | — |
| **Gamification** | — | — | ✅ | — | ✅ | ✅ | — | ✅ | — | ✅ | — | — |
| **Competition** | — | — | ✅ | — | ✅ | — | ✅ | — | — | ✅ | — | — |
| **Recommendation** | — | ✅ | ✅ | ✅ | — | — | — | — | — | — | ✅ | ✅ |
| **Notification** | — | — | — | — | ✅ | ✅ | ✅ | ✅ | — | — | ✅ | — |
| **Promotion** | ✅ | — | — | — | — | — | — | — | ✅ | ✅ | — | ✅ |
| **Commerce** | ✅ | ✅ | — | — | — | — | — | — | ✅ | — | ✅ | — |

**Легенда:** ✅ — взаимодействие присутствует, — — взаимодействие отсутствует.

---

## 2. Информационное представление (Information View)

**Назначение:** Описывает структуру данных, модели сущностей, потоки данных и способы хранения в разрезе доменов.

### 2.1. Основные сущности по доменам

| Домен | Сущности | Хранилище | Ключевые атрибуты |
|:---|:---|:---|:---|
| **Identity** | Пользователь (учётная запись), Роль, Сессия, Токен | PostgreSQL, Redis | id, email, password_hash, roles, refresh_token, last_login |
| **User Profile** | Профиль, Инвентарь, Настройки приватности, Предпочтения | PostgreSQL | id, user_id, имя, возраст, вес, рост, цели, приватность, инвентарь (тип, модель, пробег) |
| **Training** | Тренировка, Сегмент_трека, Статистика, Рекорд | TimescaleDB, PostgreSQL | id, user_id, тип, дата, дистанция, темп, пульс, GPS-трек, VO2max |
| **Device Integration** | Устройство, Сессия_устройства, Импорт_данных | PostgreSQL | id, user_id, тип_устройства, протокол (BLE/ANT+), статус, последняя_синхронизация |
| **Social** | Группа, Дружба, Сообщение, Чат, Участник_группы | PostgreSQL, Neo4j | id, название, описание, тип (открытая/закрытая), статус_дружбы, сообщения |
| **Activity Feed** | Пост, Лента, Комментарий, Реакция | PostgreSQL, Elasticsearch | id, user_id, тип_события (тренировка/достижение), контент, timestamp |
| **Gamification** | Уровень, XP, Достижение, Виртуальная_награда, Челлендж (индивидуальный) | PostgreSQL | id, user_id, уровень, xp, достижения (тип, название, дата), баланс_монет |
| **Competition** | Командный_челлендж, Рейтинг, Соревнование, Участник_команды | PostgreSQL | id, название, команды, рейтинг, призовой_фонд, дата_начала/конца |
| **Recommendation** | AI_план, Рекомендация, Инвентарная_рекомендация | PostgreSQL, DWH/BigQuery | id, user_id, тип_рекомендации, параметры, модель_версия |
| **Notification** | Уведомление, Шаблон_уведомления, Настройка_уведомлений | PostgreSQL, Firebase | id, user_id, тип (push/email/in-app), статус (отправлено/прочитано), контент |
| **Promotion** | Промоакция, Новость, Событие, Партнёр, Статистика_акции | PostgreSQL, Elasticsearch | id, партнёр, название, регион, период, целевая_аудитория, просмотры, конверсия |
| **Commerce** | Подписка, Платёж, Заказ, Промокод, Интеграция_с_legacy | PostgreSQL | id, user_id, тип_подписки, платёж, статус_заказа, интеграционные_данные |

---

## 2.2. Потоки данных между доменами

| Поток | Источник → Назначение | Формат | Частота | Задержка | Домен-источник | Домен-назначение |
|:---|:---|:---|:---|:---|:---|:---|
| **Запись тренировки** | Device Integration → Training → TimescaleDB | JSON | Высокая (до 5M/день) | < 200 мс | Device Integration | Training |
| **Обновление инвентаря** | Training → Recommendation → User Profile | Событие | Высокая | < 1 мин (асинхронно) | Training | User Profile |
| **Начисление XP** | Training → Gamification | Событие | Высокая | < 1 мин | Training | Gamification |
| **Публикация в ленте** | Training → Activity Feed | Событие | Высокая | < 1 мин | Training | Activity Feed |
| **Уведомления** | Gamification → Notification → Social | Событие | Средняя | < 5 мин | Gamification | Notification, Social |
| **Поиск пользователей** | Client → Social → Elasticsearch | Запрос-ответ | Средняя | < 300 мс | Client | Social |
| **Рекомендация инвентаря** | User Profile → Recommendation → Commerce | Запрос-ответ | Низкая | < 1 с | User Profile | Recommendation, Commerce |
| **Генерация AI-плана** | Client → Recommendation → Training | Запрос-ответ | Низкая | < 2 с | Client | Recommendation, Training |
| **Создание промоакции** | Client → Promotion → Commerce | Запрос-ответ | Низкая | < 1 с | Client | Promotion, Commerce |
| **Регистрация на событие** | Client → Promotion → Commerce | Запрос-ответ | Средняя | < 1 с | Client | Promotion, Commerce |
| **Участие в челлендже** | Client → Competition → Training | Запрос-ответ | Средняя | < 500 мс | Client | Competition, Training |

---

## 3. Представление многозадачности (Concurrency View)

**Назначение:** Описывает, как система обрабатывает параллельные запросы, конкурентный доступ к ресурсам и обеспечивает согласованность данных в разрезе доменов.

### 3.1. Стратегии обработки параллелизма по доменам

| Домен | Стратегия | Реализация | Обоснование |
|:---|:---|:---|:---|
| **Training** | Асинхронная обработка через Kafka + партиционирование TimescaleDB | Запись → Kafka → потребители | Разгрузка критического пути; гарантированная доставка; масштабируемость |
| **Recommendation** | Асинхронная обработка; CQRS для чтения | Отдельные модели для чтения | Снижение нагрузки на OLTP-БД; быстрые запросы |
| **Social** | Графовая БД (Neo4j) для быстрых запросов связей; кеширование в Redis | Neo4j + Redis | Высоконагруженные запросы связей (друзья, группы) |
| **Gamification** | Оптимистическая блокировка для обновления XP/уровней | Версионирование в PostgreSQL | Предотвращение конфликтов при одновременных обновлениях |
| **Commerce** | Saga Pattern (Choreography) для распределённых транзакций | События через Kafka; компенсирующие операции | Сохранение согласованности без 2PC |
| **Notification** | Асинхронная отправка через очереди | Kafka → Firebase/APNS | Отправка не блокирует основные потоки |
| **All domains** | Горизонтальное масштабирование (stateless сервисы) | Kubernetes HPA, множество реплик | Обработка до 10,000 RPS |

### 3.2. Обработка пиковых нагрузок по доменам

| Домен | Механизм | Реализация | Когда применяется |
|:---|:---|:---|:---|
| **Training** | Rate Limiting + Autoscaling | API Gateway + Kubernetes HPA | При пиковых нагрузках (марафоны) |
| **Recommendation** | Асинхронная генерация + кеширование | Kafka очередь + Redis | При высокой нагрузке на AI-вычисления |
| **Social** | Кеширование графовых запросов | Redis + Neo4j | При частых запросах друзей/групп |
| **All domains** | Circuit Breaker | Istio / Resilience4j | При сбоях внешних API |
| **All domains** | Bulkhead (изоляция пулов потоков) | Разделение ресурсов по доменам | Защита от каскадных отказов |
| **All domains** | Graceful Degradation | Отключение некритичных функций (Recommendation при перегрузке) | При превышении 80% ёмкости |

---

## 4. Инфраструктурное представление (Infrastructure View)

**Назначение:** Описывает физическую инфраструктуру, развертывание компонентов, сетевое взаимодействие и среду выполнения в разрезе доменов.

---

## Основной функциональный поток
Тренировка является центральным событием системы:

```text
Workout completed
      ↓
Training Service
      ↓
Event Bus
      ↓
Analytics / Gamification / Notification / Recommendation
```

```mermaid
flowchart LR
    User["Пользователь"] --> App["Mobile App / Web"]
    App --> APIGW["API Gateway / BFF"]

    APIGW --> Identity["Identity Service"]
    APIGW --> Profile["User Profile Service"]
    APIGW --> Training["Training Service"]
    APIGW --> Social["Social Graph Service"]
    APIGW --> Feed["Activity Feed Service"]
    APIGW --> Competition["Competition Service"]

    Training --> Analytics["Analytics / Insights Service"]
    Training --> Gamification["Gamification Service"]
    Training --> Recommendation["Recommendation Service"]

    Social --> Feed
    Gamification --> Notification["Notification Service"]
    Recommendation --> Commerce["Commerce Integration Service"]
    Recommendation --> Promotion["Promotion Service"]

    Notification --> App
    Commerce --> Ecommerce["Existing E-commerce Apps"]
```

---

# 14.2. Информационное представление

Информационное представление описывает основные данные системы.

## Основные сущности

| Сущность | Описание |
|---|---|
| User | Пользователь |
| Profile | Спортивный профиль |
| Workout | Тренировка |
| WorkoutMetrics | Метрики тренировки |
| Device | Устройство |
| InventoryItem | Инвентарь |
| Group | Социальная группа |
| Challenge | Челлендж |
| Competition | Соревнование |
| Achievement | Достижение |
| Promotion | Промоакция |
| Recommendation | Рекомендация |

## Классы данных по чувствительности

| Класс | Примеры | Защита |
|---|---|---|
| Public | публичные достижения | базовая |
| Personal | имя, регион | доступ по ролям |
| Sensitive | здоровье, маршруты | шифрование, аудит |
| Commercial | промо, заказы | контроль доступа |
| Operational | логи, метрики | retention policy |

## Подход к хранению
- транзакционные данные — relational DB;
- телеметрия — time-series / wide-column;
- события — event log;
- аналитика — warehouse;
- кэш — Redis;
- социальные связи — graph-like model.

```mermaid
erDiagram
    USER ||--|| PROFILE : has
    USER ||--o{ WORKOUT : performs
    USER ||--o{ DEVICE : connects
    USER ||--o{ INVENTORY_ITEM : owns
    USER ||--o{ ACHIEVEMENT : receives
    USER ||--o{ RECOMMENDATION : receives
    USER ||--o{ GROUP_MEMBERSHIP : participates

    GROUP ||--o{ GROUP_MEMBERSHIP : includes
    WORKOUT ||--o{ WORKOUT_METRICS : contains
    WORKOUT ||--o{ ROUTE_POINT : has
    WORKOUT ||--o{ EVENT : produces

    CHALLENGE ||--o{ WORKOUT : includes
    COMPETITION ||--o{ CHALLENGE : contains
    PROMOTION ||--o{ RECOMMENDATION : used_in

    USER {
        string id
        string email
        string region
        string timezone
    }

    PROFILE {
        string user_id
        string sport_level
        string preferences
        string privacy_settings
    }

    WORKOUT {
        string id
        string user_id
        string type
        datetime started_at
        datetime finished_at
    }

    WORKOUT_METRICS {
        string workout_id
        float distance
        float pace
        int heart_rate
        int calories
    }

    DEVICE {
        string id
        string user_id
        string provider
        string type
    }

    INVENTORY_ITEM {
        string id
        string user_id
        string type
        int usage_distance
    }
```

---

# 14.3. Представление многозадачности / concurrency

Это представление показывает, как система справляется с параллельными запросами, асинхронными процессами и пиковыми нагрузками.

## Основные источники параллелизма
- одновременная запись тренировок;
- массовые соревнования;
- рассылка уведомлений;
- обновление лент активности;
- пересчёт достижений;
- загрузка данных с устройств.

## Основные паттерны

### 1. Idempotency
Повторная отправка тренировки не должна создавать дубликат.

### 2. Outbox Pattern
Если сервис сохранил тренировку, событие о ней не должно потеряться.

### 3. Retry + DLQ
Ошибочные события не теряются, а попадают в отдельную очередь.

### 4. CQRS для тяжёлых read-сценариев
Leaderboard и аналитика читаются из подготовленных моделей.

### 5. Backpressure
При пиках система ограничивает скорость обработки, чтобы не упасть полностью.

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant API as API Gateway
    participant Training as Training Service
    participant DB as Training DB
    participant Outbox as Outbox
    participant Bus as Event Bus
    participant Analytics as Analytics Service
    participant Game as Gamification Service
    participant Notify as Notification Service
    participant Feed as Activity Feed Service

    App->>API: Submit workout
    API->>Training: Save workout request
    Training->>Training: Validate + idempotency check
    Training->>DB: Persist workout
    Training->>Outbox: Save training.completed event
    Training-->>API: Success response
    API-->>App: Workout saved

    Outbox->>Bus: Publish training.completed

    par Async processing
        Bus-->>Analytics: training.completed
        Analytics->>Analytics: Update aggregates
    and
        Bus-->>Game: training.completed
        Game->>Game: Check achievements
    and
        Bus-->>Feed: training.completed
        Feed->>Feed: Update activity feed
    and
        Bus-->>Notify: training.completed / achievement
        Notify-->>App: Push notification
    end
```
---

# 14.4. Инфраструктурное представление

## Основные элементы
- Kubernetes clusters;
- managed databases;
- Kafka/PubSub;
- CDN;
- API Gateway;
- object storage;
- monitoring stack;
- secret manager.

## Среды
- dev;
- test;
- staging;
- production.

## Развёртывание
- CI/CD pipeline;
- blue-green или canary deployments;
- infrastructure as code;
- automated rollback.

## Multi-region подход
Для MVP достаточно одного основного региона и disaster recovery.  
Для глобального масштаба — несколько регионов с локализацией данных.

```mermaid
flowchart TB
    subgraph Internet["Internet"]
        Users["Users"]
        Devices["Wearables / Devices"]
    end

    subgraph Edge["Edge Layer"]
        DNS["Global DNS"]
        CDN["CDN"]
        WAF["WAF"]
        APIGW["API Gateway"]
    end

    subgraph RegionA["Primary Cloud Region"]
        subgraph K8S["Kubernetes Cluster"]
            BFF["BFF Pods"]
            Identity["Identity Service Pods"]
            Profile["Profile Service Pods"]
            Training["Training Service Pods"]
            Social["Social Service Pods"]
            Recommendation["Recommendation Pods"]
            Notification["Notification Pods"]
            Competition["Competition Pods"]
        end

        Kafka["Managed Kafka / PubSub"]
        Redis["Redis Cache"]
        SQL["Relational DB"]
        TSDB["Time-series / Training DB"]
        Warehouse["Data Warehouse"]
        ObjectStorage["Object Storage"]
        Monitoring["Monitoring / Logging / Tracing"]
        Secrets["Secret Manager"]
    end

    subgraph DR["Disaster Recovery Region"]
        BackupDB["Replicated DB / Backups"]
        BackupStorage["Backup Object Storage"]
    end

    Users --> DNS --> CDN --> WAF --> APIGW --> BFF
    Devices --> APIGW

    BFF --> Identity
    BFF --> Profile
    BFF --> Training
    BFF --> Social
    BFF --> Recommendation
    BFF --> Competition

    Training --> TSDB
    Profile --> SQL
    Identity --> SQL
    Social --> SQL
    Competition --> Redis
    Training --> Kafka
    Kafka --> Recommendation
    Kafka --> Notification
    Kafka --> Warehouse

    K8S --> Monitoring
    K8S --> Secrets

    SQL --> BackupDB
    ObjectStorage --> BackupStorage
```
---

# 14.5. Представление безопасности

## Основные угрозы
- кража аккаунта;
- утечка маршрутов;
- несанкционированный доступ к данным здоровья;
- злоупотребление социальными функциями;
- компрометация API;
- утечка токенов устройств.

## Меры безопасности
- OAuth2/OIDC;
- MFA для чувствительных операций;
- TLS everywhere;
- encryption at rest;
- RBAC/ABAC;
- audit logs;
- secrets management;
- WAF;
- rate limiting;
- privacy settings;
- consent management.

## Privacy by default
По умолчанию чувствительные данные не должны быть публичными:
- точные маршруты скрыты;
- активность видна только выбранной аудитории;
- текущая геолокация не раскрывается без явного согласия.

```mermaid
flowchart TB
    User["Пользователь"] --> App["Mobile App"]

    App --> TLS["TLS / HTTPS"]
    TLS --> WAF["WAF + Rate Limiting"]
    WAF --> APIGW["API Gateway"]

    APIGW --> Auth["Identity & Access Service"]
    Auth --> Token["OAuth2 / OIDC Tokens"]

    APIGW --> Policy["Authorization Layer<br/>RBAC / ABAC"]

    Policy --> Profile["User Profile Service"]
    Policy --> Training["Training Service"]
    Policy --> Social["Social Graph Service"]
    Policy --> Commerce["Commerce Integration Service"]
    Policy --> Analytics["Analytics Service"]

    Profile --> DataClass["Data Classification"]
    Training --> DataClass
    Social --> DataClass
    Commerce --> DataClass
    Analytics --> DataClass

    DataClass --> Encryption["Encryption at Rest"]
    Profile --> Audit["Audit Logs"]
    Training --> Audit
    Social --> Audit
    Commerce --> Audit
    Analytics --> Audit

    Profile --> Consent["Consent Management"]
    Training --> Masking["Geo Data Masking"]

    Encryption --> DB["Databases / Storage"]
    Audit --> SIEM["Security Monitoring / SIEM"]

    Secrets["Secret Manager"] --> Profile
    Secrets --> Training
    Secrets --> Social
    Secrets --> Commerce
    Secrets --> Analytics
```
