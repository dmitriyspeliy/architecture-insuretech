# Задание 2. Масштабирование приложения в Kubernetes

В рамках задания был развёрнут тестовый сервис `scaletestapp` в локальном Kubernetes-кластере Minikube и проверена работа горизонтального автоскейлинга.

Были реализованы два сценария масштабирования:

1. **HPA по потреблению памяти**  
   Для приложения настроен `HorizontalPodAutoscaler`, который масштабирует количество pod'ов при превышении целевого значения использования памяти. Нагрузка создавалась через Locust. Результат масштабирования сохранён в `hpa-memory-result.txt`.

2. **HPA по RPS через Prometheus**  
   Для приложения настроен сбор метрик через Prometheus с endpoint `/metrics`. Затем через Prometheus Adapter метрика `http_requests_total` была преобразована в custom metric `http_requests_per_second`, на основе которой настроен HPA по RPS. Результаты проверки сохранены в `custom-metric-rps-result.json` и `hpa-rps-result.txt`.

Для локальной демонстрации в Minikube в `hpa-rps.yaml` используется тестовый порог:

```yaml
averageValue: "100m"