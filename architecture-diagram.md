# Диаграмма архитектуры

## High-level architecture

```mermaid
flowchart TB
    %% Clients
    subgraph Clients["Клиентский слой"]
        Mobile["Mobile App<br/>iOS / Android"]
        Web["Web / Admin UI"]
        Devices["Wearables & Sensors<br/>HR, GPS, Oxygen"]
    end

    %% Edge
    subgraph Edge["Edge / Access слой"]
        CDN["CDN"]
        WAF["WAF"]
        APIGW["API Gateway"]
        BFF["BFF for Mobile"]
    end

    %% Core services
    subgraph Core["Core domain services"]
        Auth["Identity & Access Service"]
        User["User Profile Service"]
        Training["Training Service"]
        DeviceIntegration["Device Integration Service"]
        Social["Social Graph Service"]
        Feed["Activity Feed Service"]
        Gamification["Challenge & Gamification Service"]
        Competition["Competition Service"]
        Recommendation["Recommendation Service"]
        Notification["Notification Service"]
        Promotion["Promotion & Content Service"]
        Commerce["Commerce Integration Service"]
        Analytics["Analytics / Insights Service"]
    end

    %% Data layer
    subgraph Data["Data layer"]
        UserDB[("User / Profile DB<br/>Relational DB")]
        TrainingDB[("Training DB<br/>Document / Wide-column")]
        TimeSeries[("Telemetry Storage<br/>Time-series")]
        SocialDB[("Social Graph Storage")]
        Cache[("Cache<br/>Redis")]
        EventBus[("Event Bus<br/>Kafka / PubSub")]
        Warehouse[("Data Warehouse / Lake")]
        ObjectStorage[("Object Storage")]
    end

    %% External systems
    subgraph External["Внешние системы"]
        Ecommerce["Existing E-commerce Apps"]
        FitnessPlatforms["Fitness Platforms<br/>Apple Health / Google Fit"]
        PartnerDevices["Partner Device APIs"]
        Marketing["Marketing / CRM Systems"]
    end

    Mobile --> CDN
    Web --> CDN
    CDN --> WAF
    WAF --> APIGW
    APIGW --> BFF

    Devices --> DeviceIntegration
    FitnessPlatforms --> DeviceIntegration
    PartnerDevices --> DeviceIntegration

    BFF --> Auth
    BFF --> User
    BFF --> Training
    BFF --> Social
    BFF --> Feed
    BFF --> Recommendation
    BFF --> Competition

    Auth --> UserDB
    User --> UserDB
    Training --> TrainingDB
    Training --> TimeSeries
    Social --> SocialDB
    Feed --> Cache
    Competition --> Cache

    Training --> EventBus
    DeviceIntegration --> EventBus
    Social --> EventBus
    Gamification --> EventBus
    Competition --> EventBus

    EventBus --> Analytics
    EventBus --> Recommendation
    EventBus --> Notification
    EventBus --> Gamification
    EventBus --> Promotion
    EventBus --> Warehouse

    Analytics --> Warehouse
    Recommendation --> Warehouse
    Recommendation --> UserDB
    Notification --> Mobile
    Promotion --> Marketing
    Commerce --> Ecommerce
    Recommendation --> Commerce

    TrainingDB --> Warehouse
    TimeSeries --> Warehouse
    ObjectStorage --> Warehouse
```

## Основной поток: завершение тренировки

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant API as API Gateway / BFF
    participant Training as Training Service
    participant Bus as Event Bus
    participant Analytics as Analytics Service
    participant Game as Gamification Service
    participant Rec as Recommendation Service
    participant Notify as Notification Service
    participant Commerce as Commerce Integration

    App->>API: Отправка завершённой тренировки
    API->>Training: Save workout
    Training->>Training: Валидация и сохранение
    Training->>Bus: publish training.completed

    Bus-->>Analytics: training.completed
    Analytics->>Analytics: Пересчёт прогресса

    Bus-->>Game: training.completed
    Game->>Game: Проверка достижений

    Bus-->>Rec: training.completed
    Rec->>Rec: Обновление рекомендаций

    Bus-->>Notify: achievement / progress events
    Notify-->>App: Push / in-app notification

    Rec->>Commerce: Контекстная рекомендация товара
```

## Легенда

| Элемент | Назначение |
|---|---|
| API Gateway | Единая точка входа, маршрутизация, rate limiting |
| BFF | Адаптация API под мобильный клиент |
| Event Bus | Асинхронное взаимодействие сервисов |
| Training Service | Центральный сервис записи тренировок |
| Analytics Service | Расчёт прогресса и агрегатов |
| Recommendation Service | Персональные рекомендации тренировок, инвентаря и промо |
| Commerce Integration Service | Интеграция с существующими e-commerce приложениями |
| Data Warehouse | Аналитика, BI, ML и отчётность |
