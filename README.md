# moon_roles

Автоматизирует полную установку [Aerokube Moon 2](https://aerokube.com/moon/).

## Что делается

Плейбук `helm_playbook.yaml` последовательно применяет роли:

| Роль | Что делает |
|------|-----------|
| `common` | Устанавливает базовые зависимости apt (curl, gnupg, ca-certificates, python3-yaml) |
| `proxy` | Прописывает HTTP/HTTPS прокси в `/etc/environment` с исключением для подсети Minikube `192.168.49.0/24` |
| `helm` | Скачивает и запускает официальный install-скрипт Helm, удаляет его после установки |
| `docker` | Добавляет официальный Docker-репозиторий, устанавливает docker-ce, добавляет пользователя в группу `docker` |
| `kubectl` | Определяет актуальную стабильную версию через API и скачивает бинарник |
| `minikube` | Запускает кластер с драйвером docker, пробрасывает порты 80/443, включает ingress-nginx и ждёт его готовности |
| `moon` | Добавляет Helm-репозиторий Aerokube, деплоит chart `aerokube/moon2` в namespace `moon` с кастомным `values.yaml` |
| `wiremock` | Добавляет Helm-репозиторий WireMock, создаёт ConfigMap со стабами из `files/mappings/`, деплоит chart `wiremock/wiremock` в namespace `wiremock-ns` |

## Переменные

| Переменная | Файл | Значение по умолчанию | Описание |
|-----------|------|-----------------------|----------|
| `moon_host` | `common/vars/main.yaml` | `moon.local` | Hostname для ingress Moon UI |
| `helm_script_url` | `common/vars/main.yaml` | `https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4` | URL install-скрипта Helm |
| `tmp_dir` | `common/vars/main.yaml` | `/tmp` | Временная директория |
| `proxy_url` | `proxy/vars/main.yaml` | `http://192.168.0.7:1081` | Адрес HTTP-прокси |
| `wiremock_namespace` | `wiremock/defaults/main.yaml` | `wiremock-ns` | Namespace для WireMock |
| `wiremock_release_name` | `wiremock/defaults/main.yaml` | `wiremock` | Имя Helm-релиза |
| `wiremock_chart_version` | `wiremock/defaults/main.yaml` | `1.11.0` | Версия Helm-чарта WireMock |
| `kubeconfig_path` | `wiremock/defaults/main.yaml` | `/home/margo/.kube/config` | Путь до kubeconfig |


## Запуск

```bash
ansible-playbook -i hosts helm_playbook.yaml --ask-become-pass
```

Только деплой Moon (если инфраструктура уже поднята):

```bash
ansible-playbook -i hosts helm_playbook.yaml --tags moon --ask-become-pass
```

Только деплой WireMock:

```bash
ansible-playbook -i hosts helm_playbook.yaml --tags wiremock --ask-become-pass
```

## WireMock стабы

JSON-файлы стабов хранятся в `roles/wiremock/files/mappings/`. При деплое они автоматически загружаются в ConfigMap и монтируются в под.

Проверить загруженные стабы:

```bash
curl http://wiremock.local/__admin/mappings
```

Добавить запись в `/etc/hosts` на локальной машине:

```
192.168.0.77    wiremock.local
```

## Браузеры

В `moon/values.yaml` преднастроены образы для трёх фреймворков:

- **Selenium**: Chrome, Firefox, Edge
- **Playwright**: Chrome, Chromium, Edge, Firefox, WebKit
