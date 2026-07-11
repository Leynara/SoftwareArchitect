# 13. Базовая архитектура с учётом ограничений

---

## 13.1. Архитектурная цель

Базовая архитектура должна поддерживать:
- глобальный масштаб;
- разные пользовательские сценарии;
- чувствительные данные;
- интеграции с устройствами;
- интеграции с e-commerce;
- постепенное развитие продукта.

---

## 13.2. Основные архитектурные ограничения

### Ограничение 1. Глобальные пользователи
Система должна работать для пользователей в разных регионах и часовых поясах.

### Ограничение 2. Multi-cloud окружение
Компания уже использует разные облачные провайдеры, поэтому архитектура не должна жёстко зависеть от одного vendor-specific решения.

### Ограничение 3. Чувствительные данные
Данные о тренировках, здоровье и местоположении требуют особой защиты.

### Ограничение 4. Множество команд
Архитектура должна позволять параллельную разработку без постоянных блокировок между командами.

### Ограничение 5. Не все функции одинаково важны
Система должна позволять поэтапный запуск и отключение экспериментальных функций.

---

## 13.3. Логическая структура системы

### Client Layer
- iOS app
- Android app
- Web admin
- Partner integrations

### Edge Layer
- CDN
- API Gateway
- WAF
- Rate Limiting
- BFF

### Domain Services Layer
- Identity Service
- User Profile Service
- Training Service
- Device Integration Service
- Social Graph Service
- Activity Feed Service
- Challenge & Gamification Service
- Competition Service
- Recommendation Service
- Notification Service
- Promotion Service
- Commerce Integration Service

### Data Layer
- transactional DB;
- time-series storage;
- document storage;
- graph/social storage;
- cache;
- event log;
- data lake / warehouse.

### Platform Layer
- Kubernetes;
- service mesh;
- monitoring;
- logging;
- tracing;
- CI/CD;
- secret management;
- infrastructure as code.

---

## 13.4. Адресация атрибутов качества

### Масштабируемость
Обеспечивается через:
- stateless сервисы;
- horizontal autoscaling;
- event-driven обработку;
- кэширование;
- разделение read/write нагрузки;
- partitioning данных.

### Надёжность
Обеспечивается через:
- репликацию;
- retry;
- circuit breaker;
- outbox pattern;
- DLQ;
- graceful degradation.

### Безопасность
Обеспечивается через:
- OAuth2/OIDC;
- RBAC/ABAC;
- шифрование;
- аудит;
- consent management;
- data classification.
