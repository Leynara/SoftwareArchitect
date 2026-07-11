# 14. Основные архитектурные представления

---

# 14.1. Функциональное представление

Функциональное представление показывает, какие бизнес-возможности реализует система.

## Основные функциональные домены

| Домен | Назначение |
|---|---|
| Identity | Аутентификация и доступ |
| User Profile | Профиль, настройки, инвентарь |
| Training | Тренировки и активность |
| Device Integration | Подключение внешних устройств |
| Social | Друзья, группы, связи |
| Activity Feed | Лента активности |
| Gamification | Достижения, челленджи |
| Competition | Массовые соревнования |
| Recommendation | Рекомендации |
| Notification | Уведомления |
| Promotion | Промо и новости |
| Commerce | Интеграция с продажами |

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
