# ADR: Детализация решений для MVP мониторинга животных «АгроТех Х» с учётом корпоративной архитектуры (C3, C4)

### Название задачи:
Декомпозиция компонентов и сервисов с учётом платформенных решений компании (уровни C3 и C4)

### Автор:
Шаханов Михаил

### Дата:
17 июля 2025

---

## Основное решение: Централизованная аналитика с переиспользованием Kafka и TimescaleDB

### C3. Центральный сервер
```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

System_Boundary(centralServer, "Central Server") {
  Container(kafkaConsumer, "Kafka Consumer", "Spring Boot", "Получение данных с агентов через Kafka")
  Container(videoAnalytics, "Video Analytics", "Java + TensorFlow Serving", "Дополнительная аналитика")
  Container(alerting, "Alerting", "Spring Boot", "Оповещения операторов")
  Container(apiGateway, "API Gateway", "Spring Cloud Gateway", "API для мобильного приложения")
  ContainerDb(timescale, "TimescaleDB", "Исторические данные и метрики")
}

Rel(kafkaConsumer, videoAnalytics, "Отправка событий на анализ")
Rel(kafkaConsumer, alerting, "Тревожные события")
Rel(kafkaConsumer, timescale, "Сохраняет сырые данные")
Rel(videoAnalytics, timescale, "Результаты аналитики")
Rel(alerting, apiGateway, "Тревоги в UI")
@enduml
```

### C4. Структура Kafka Consumer Service
```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Container_Boundary(kafkaConsumer, "Kafka Consumer Service") {
  Container(metricsProcessor, "Metrics Processor", "Spring Boot", "Разбор метрик с Kafka")
  Container(anomalyDetector, "Anomaly Detector", "Java", "Детектирование аномалий по событиям")
  Container(writerTimescale, "TSDB Writer", "Spring Boot", "Запись в TimescaleDB")
}

Rel(metricsProcessor, anomalyDetector, "Отправляет метрики")
Rel(metricsProcessor, writerTimescale, "Исторические данные")
@enduml
```

---

## Альтернативное решение: Локальная аналитика с передачей агрегатов через Kafka

### C3. Edge Agent
```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

System_Boundary(edgeAgent, "Edge Agent") {
  Container(localVideoAnalytics, "Local Video Analytics", "Python + TensorRT", "AI анализ видео локально")
  Container(localAlerting, "Local Alerting", "Python", "Отправка тревог локально и в Kafka")
  Container(mqttClient, "MQTT Client", "Python", "Подключение сенсоров")
  Container(kafkaProxy, "Kafka Proxy", "Python", "Передача тревог и метрик в Kafka")
}

Rel(localVideoAnalytics, localAlerting, "Тревоги по видео")
Rel(localAlerting, kafkaProxy, "Тревоги в Kafka")
Rel(mqttClient, localAlerting, "Локальные события")
@enduml
```

### C4. Структура Local Video Analytics
```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

Container_Boundary(localVideoAnalytics, "Local Video Analytics") {
  Container(videoCapture, "Video Capture", "Python", "Чтение видеопотока")
  Container(aiInference, "YOLO Inference", "TensorRT", "Анализ поведения")
  Container(eventAggregator, "Event Aggregator", "Python", "Формирование тревог")
}

Rel(videoCapture, aiInference, "Передача кадров")
Rel(aiInference, eventAggregator, "Детектированные события")
@enduml
```

---

## Итог по Task3:
На уровнях компонентов и кода все взаимодействия вписываются в платформенные паттерны АгроПром Х — Kafka, TimescaleDB, локальные MQTT клиенты, Kafka Proxy. Основное решение быстрее запускается, альтернативное устойчиво к проблемам связи.

