Infrastructure & Deployment Stack: Cloud-Native GitOps

Проект представляет собой реализацию современного производственного (Production-ready) цикла управления инфраструктурой и доставки приложений в облаке Yandex Cloud.

1. Infrastructure as Code (IaC)Terraform: Используется для управления жизненным циклом облачных ресурсов (Provisioning).
Создание сети (VPC, Subnets) и вычислительных ресурсов (Managed Service for Kubernetes).
Управление безопасностью через IAM (Service Accounts, Roles).
Использование Yandex KMS для криптографических операций (Auto-unseal Vault).
Terraform Helm Provider: Используется исключительно для инициализации "фундамента" — установки ArgoCD.

2. GitOps & Continuous Delivery (CD)
ArgoCD: Центральный контроллер доставки. Реализует паттерн "App of Apps".
Синхронизация состояния кластера с Git-репозиторием (Single Source of Truth).
Автоматический мониторинг дрейфа конфигурации (Self-healing).
Helm: Пакетный менеджер. Все компоненты (Ingress, Vault, App) упакованы в чарты для управления зависимостями и параметризации через values.yaml.

3. Secret Management & Security
HashiCorp Vault (Raft HA): Хранилище секретов в режиме высокой доступности с интегрированным хранилищем Raft.
Интеграция с Yandex KMS для автоматического разпечатывания (Auto-unseal).
External Secrets Operator (ESO): Автоматизирует синхронизацию секретов из Vault в нативные Kubernetes Secrets.
Метод аутентификации: Kubernetes Auth Method (через ServiceAccounts).
Cert-Manager: Автоматизация выпуска TLS-сертификатов от Let's Encrypt через HTTP-01/DNS-01 челленджи.

4. Infrastructure Services
Ingress Nginx Controller: Точка входа трафика в кластер с поддержкой балансировки и терминации SSL.
NFS Server Provisioner: Обеспечение общего хранилища (Shared Storage) для распределенных компонентов приложения (ReadWriteMany).

5. Continuous Integration (CI)
GitLab CI:
Build Stage: Сборка Docker-образов с использованием Kaniko/Docker-in-Docker.
Registry: Хранение артефактов в Yandex Container Registry.
Update Stage: Автоматическое обновление тегов образов в GitOps-репозитории.

6. Observability (Full Monitoring Stack)
Prometheus & Grafana: Сбор метрик с инфраструктуры и приложения, визуализация состояния через дашборды.
Loki & Promtail: Централизованный сбор и агрегация логов со всех подов кластера.

Итоговая схема потока данных (Workflow):
Code Push -> GitLab CI собирает образ и пушит в Registry.
GitOps Update -> GitLab CI правит версию чарта в Infra-репозитории.
Sync -> ArgoCD видит изменения и обновляет ресурсы в K8s.Runtime -> App запрашивает секреты через ESO из Vault, Ingress направляет трафик, а Prometheus собирает метрики.


Cloud-Native Production Infrastructure

Проект автоматизированного развертывания отказоустойчивой инфраструктуры в Yandex Cloud с использованием методологии GitOps и полного цикла CI/CD.

Архитектура стека

Core Infrastructure
Cloud: Yandex Cloud (Managed Service for Kubernetes)
IaC: Terraform (VPC, Кворум узлов, KMS, IAM)
GitOps: ArgoCD (реализация паттерна App-of-Apps)
Ingress: Nginx Ingress Controller + Cert-Manager (Let's Encrypt TLS)

Security & Secrets
Vault: HashiCorp Vault (HA mode + Raft)
Auto-unseal: Интеграция с Yandex KMS
Sync: External Secrets Operator (ESO) для доставки секретов в K8s
Auth: Kubernetes ServiceAccount Auth

Observability
Metrics: Prometheus & Grafana (мониторинг ресурсов и здоровья кластера)
Logs: Loki & Promtail (агрегация логов)

Поток доставки (CI/CD Workflow)
Develop: Разработчик пушит код в GitLab.
Build: GitLab CI собирает Docker-образ и пушит его в Yandex Container Registry.
Deploy: GitLab CI обновляет версию образа в GitOps-репозитории.
Sync: ArgoCD обнаруживает расхождение (drift) и синхронизирует состояние кластера с Git.
Inject: ESO забирает необходимые переменные окружения из Vault и создает K8s Secrets.

Структура репозиториев1. [Infrastructure-Repo] (Terraform)Содержит описание "железа" и базовую установку ArgoCD.

terraform/
├── main.tf          # Описание кластера и Helm-релиза ArgoCD
├── variables.tf     # Переменные окружения облака
└── providers.tf     # Настройки провайдеров YC и Helm

2. [GitOps-Repo] (ArgoCD & Helm)

Единый источник истины для всех сервисов внутри кластера.

apps/                # Манифесты ArgoCD Application
  ├── vault.yaml
  ├── ingress.yaml
  └── my-app.yaml

values/              # Конфигурации для Helm-чартов
  ├── vault-values.yaml
  └── app-values.yaml

Быстрый старт
Развертывание фундамента:
bashcd terraform && terraform apply

Инициализация Vault (единоразово):
bashkubectl exec -ti vault-0 -n vault -- vault operator init

Запуск GitOps:
Применить root-app.yaml в кластер, после чего ArgoCD автоматически развернет все остальные компоненты.

Что можно улучшить в этом README:
Добавить схему архитектуры (сделанную в draw.io или Excalidraw).
Добавить скриншоты Grafana Dashboards и ArgoCD UI.
