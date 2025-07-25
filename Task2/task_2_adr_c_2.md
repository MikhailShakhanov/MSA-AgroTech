# ADR: Контейнерные диаграммы MVP мониторинга животных для «АгроТех Х» с учётом корпоративной платформы

### Название задачи:
Проработка архитектуры уровня контейнеров (C2) с интеграцией в цифровую платформу компании

### Автор:
Шаханов Михаил

### Дата:
17 июля 2025

---

## Основное решение: Централизованная аналитика через Kafka и корпоративные хранилища

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Person(operator, "Оператор фермы")
Person(mobileApp, "Мобильное приложение")

System_Boundary(system, "Система мониторинга животных") {
  Container(edgeAgent, "Edge Agent", "Python/Go", "Собирает данные с камер, сенсоров, публикует в Kafka")
  Container(centralServer, "Central Server", "Spring Boot", "Обработка событий и аналитика")
  Container(mobileAPI, "API Gateway", "Spring Cloud Gateway", "Единая точка входа")
  ContainerDb(timescale, "TimescaleDB", "Исторические данные")
  Container_Ext(kafka, "Kafka", "Брокер сообщений компании")
}

Rel(edgeAgent, kafka, "Публикация видеоаналитики и событий")
Rel(kafka, centralServer, "Подписка на события")
Rel(centralServer, timescale, "Сохранение метрик")
Rel(centralServer, mobileAPI, "REST API для операторов")
Rel(mobileAPI, operator, "Доступ к событиям через Web-интерфейс")
Rel(mobileApp, mobileAPI, "REST API для мобильного доступа")
@enduml
```

## Альтернативное решение: Локальная аналитика с использованием Kafka для метрик

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Person(operator, "Оператор фермы")
Person(mobileApp, "Мобильное приложение")

System_Boundary(system, "Система мониторинга животных") {
  Container(edgeAgent, "Edge Agent with Local AI", "Python + TensorRT", "Локальная аналитика с публикацией тревог")
  Container_Ext(kafka, "Kafka", "Брокер компании")
  Container(centralServer, "Central Server", "Spring Boot", "Агрегация и API")
  ContainerDb(timescale, "TimescaleDB", "Исторические данные")
  Container(mobileAPI, "API Gateway", "Spring Cloud Gateway")
}

Rel(edgeAgent, kafka, "Публикация метрик и тревог")
Rel(kafka, centralServer, "Подписка")
Rel(centralServer, timescale, "Сохранение")
Rel(centralServer, mobileAPI, "REST API")
Rel(mobileAPI, operator, "Просмотр тревог через Web")
Rel(mobileApp, mobileAPI, "REST API для мобильного доступа")
@enduml
```

## Компромиссы и риски
- Основное решение быстрее внедряется, лучше вписывается в существующий ландшафт;
- Альтернативное решение улучшает автономность в отрыве от центра, но дороже из-за edge AI;
- Оба варианта интегрируются с Kafka, TSDB и BI-платформой компании.

---

## Итог по Task2:
Обе архитектуры учитывают корпоративную инфраструктуру компании: Kafka, TimescaleDB и BI-решения. Основное решение проще масштабируется, альтернативное даёт более высокую устойчивость на фермах без связи.

