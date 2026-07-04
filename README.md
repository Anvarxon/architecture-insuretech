# Architecture InsureTech

Решения по проекту InsureTech разложены по директориям `Task1` - `Task6`.

## Состав

- `Task1` - технологическая архитектура to-be в draw.io.
- `Task2` - Kubernetes-манифесты, HPA, Prometheus adapter, Locust и проверочный лог окружения.
- `Task3` - анализ рисков и C4 container diagram для перехода на Event-Driven архитектуру.
- `Task4` - C4-схема для продажи ОСАГО онлайн.
- `Task5` - GraphQL-схема сервиса `client-info`.
- `Task6` - конфигурация Nginx с Rate Limiting и HTTP 429.

## Локальная проверка инструментов

Установлены:

- Minikube `v1.38.1`
- kubectl `v1.36.2`
- Python `3.12.10`
- pip `25.0.1`
- Locust `2.44.4`

Minikube не стартует в текущей не-elevated Windows-сессии из-за отсутствия доступного драйвера виртуализации. Подробность зафиксирована в `Task2/minikube-start-error.log`.
