# Task1. Проектирование технологической архитектуры

## Решение

Целевая архитектура построена как multi-AZ deployment в Yandex Cloud.

Пользовательский frontend и статические ресурсы отдаются через CDN, чтобы время загрузки страниц меньше зависело от региона пользователя.

B2C и B2B трафик разделён на уровне DNS, edge-routing, ingress и backend deployments:
- B2C: www.insuretech.ru → CDN/WAF → ALB → B2C Ingress → core-app-b2c
- B2B: api.insuretech.ru → WAF → API Gateway → ALB → B2B Ingress → core-app-b2b

Такое разделение нужно, чтобы партнёрский API-трафик не влиял на пользовательский сайт.

Backend-сервисы развёрнуты в одном региональном Managed Kubernetes-кластере, распределённом по трём зонам доступности. Приложения масштабируются горизонтально через HPA, worker nodes — через Cluster Autoscaler.

Для хранения данных используется PostgreSQL HA Cluster:
- primary в одной зоне доступности;
- synchronous standby во второй зоне;
- asynchronous standby в третьей зоне;
- WAL archive и backups в Object Storage;
- PITR включён.

Шардирование БД не применяется, так как текущий объём данных составляет 50 GB. Для такого объёма достаточно HA-репликации, индексов, кэша и оптимизации запросов.

## Соответствие требованиям

| Требование | Решение |
|---|---|
| Availability 99.9% | 3 AZ, ALB health checks, Kubernetes replicas, PostgreSQL HA |
| RTO 45 мин | automatic failover на уровне Kubernetes и PostgreSQL |
| RPO 15 мин | synchronous replica, WAL archive, PITR |
| Одинаковая загрузка страниц | CDN + Object Storage для frontend/static |
| Рост нагрузки | HPA, Cluster Autoscaler, горизонтальное масштабирование |
| Изоляция B2B | API Gateway, отдельный ingress и core-app-b2b |
| Шардирование | Не применяется, объём данных 50 GB |