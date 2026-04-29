# Go Workflows for GitHub Actions

Централизованные GitHub Actions workflow для Go проектов. Подключаются через `workflow_call` — достаточно добавить одну строку в ваш CI.

## Доступные workflow

### 1. `go-lint.yml` — проверка кода

Запускает [golangci-lint](https://github.com/golangci/golangci-lint).

| Параметр | Тип | По умолчанию | Описание |
|---|---|---|---|
| `go-version` | string | `1.23.2` | Версия Go |
| `linter-version` | string | `v1.60.1` | Версия golangci-lint |
| `linter-config-path` | string | `.golangci.yml` | Путь к конфигу линтера |
| `timeout` | string | `5m` | Таймаут линтера |
| `working-directory` | string | `.` | Рабочая директория |
| `ubuntu-version` | string | `22.04` | Версия Ubuntu раннера |

```yaml
jobs:
  lint:
    uses: your-org/central_ci_cd/.github/workflows/go-lint.yml@main
    with:
      go-version: '1.23.2'
      linter-version: 'v1.60.1'
      linter-config-path: '.golangci.yml'
      timeout: '5m'
      working-directory: '.'
```

---

### 2. `go-test.yml` — запуск тестов

Запускает `go test` с поддержкой отчёта покрытия.

| Параметр | Тип | По умолчанию | Описание |
|---|---|---|---|
| `go-version` | string | `1.23.2` | Версия Go |
| `test-flags` | string | `-v -race -coverprofile=coverage.txt -covermode=atomic` | Флаги `go test` |
| `upload-coverage` | boolean | `true` | Загружать отчёт покрытия в Codecov |
| `timeout` | string | `5m` | Таймаут тестов |
| `working-directory` | string | `.` | Рабочая директория |
| `ubuntu-version` | string | `22.04` | Версия Ubuntu раннера |

| Секрет | Обязательный | Описание |
|---|---|---|
| `CODECOV_TOKEN` | нет | Токен для Codecov |

```yaml
jobs:
  test:
    uses: your-org/central_ci_cd/.github/workflows/go-test.yml@main
    with:
      go-version: '1.23.2'
      test-flags: '-v -race -coverprofile=coverage.txt -covermode=atomic'
      upload-coverage: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

### 3. `go-build.yml` — сборка приложения

Собирает бинарный файл и загружает его как артефакт (хранится 7 дней).

| Параметр | Тип | По умолчанию | Описание |
|---|---|---|---|
| `go-version` | string | `1.23.2` | Версия Go |
| `build-flags` | string | `-v` | Флаги `go build` |
| `output-binary` | string | `app` | Имя выходного бинарного файла |
| `working-directory` | string | `.` | Рабочая директория |
| `ubuntu-version` | string | `22.04` | Версия Ubuntu раннера |

```yaml
jobs:
  build:
    uses: your-org/central_ci_cd/.github/workflows/go-build.yml@main
    with:
      go-version: '1.23.2'
      build-flags: '-v'
      output-binary: 'app'
      working-directory: '.'
```

---

### 4. `go-security-scan.yml` — сканирование уязвимостей

Запускает [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) для проверки известных уязвимостей в зависимостях.

| Параметр | Тип | По умолчанию | Описание |
|---|---|---|---|
| `go-version` | string | `1.23.2` | Версия Go |
| `working-directory` | string | `.` | Рабочая директория |
| `ubuntu-version` | string | `22.04` | Версия Ubuntu раннера |

```yaml
jobs:
  security:
    uses: your-org/central_ci_cd/.github/workflows/go-security-scan.yml@main
    with:
      go-version: '1.23.2'
      working-directory: '.'
```

---

### 5. `go-ci-complete.yml` — полный CI pipeline

Объединяет все четыре workflow в один вызов с умным управлением зависимостями:

- `lint` и `go_test` — запускаются параллельно
- `go_vuln_check` — запускается после успешного `lint`
- `go_build` — запускается после успешных `lint` + `go_test`

| Параметр | Тип | По умолчанию | Описание |
|---|---|---|---|
| `go-version` | string | `1.23.2` | Версия Go |
| `linter-version` | string | `v1.60.1` | Версия golangci-lint |
| `ubuntu-version` | string | `22.04` | Версия Ubuntu раннера |
| `working-directory` | string | `.` | Рабочая директория |

| Секрет | Обязательный | Описание |
|---|---|---|
| `CODECOV_TOKEN` | нет | Токен для Codecov |

```yaml
jobs:
  ci:
    uses: your-org/central_ci_cd/.github/workflows/go-ci-complete.yml@main
    with:
      go-version: '1.23.2'
      linter-version: 'v1.60.1'
      ubuntu-version: '22.04'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

### 6. `docker-publish.yml` — публикация Docker-образа в GHCR

Собирает Docker-образ и публикует его в [GitHub Container Registry](https://ghcr.io). Теги генерируются автоматически из git-тегов вида `v1.2.3`:

- `1.2.3`
- `1.2`
- `latest`

Сборка кешируется через GitHub Actions Cache (`cache-from/cache-to: type=gha`).

> **Примечание.** Workflow публикует образ, но не деплоит его — это артефакт релиза, а не полноценный CD. Для автоматического деплоя потребуется дополнительный шаг (SSH на сервер, обновление манифеста, вызов платформенного API и т.д.).

| Параметр | Тип | Обязательный | По умолчанию | Описание |
|---|---|---|---|---|
| `image-name` | string | да | — | Полное имя образа (например, `ghcr.io/your-org/app`) |
| `user-name` | string | да | — | Имя пользователя для авторизации в ghcr.io |
| `dockerfile` | string | нет | `Dockerfile` | Путь к Dockerfile |
| `build-args` | string | нет | `''` | Аргументы сборки (`ARG` в Dockerfile) |
| `ubuntu-version` | string | нет | `22.04` | Версия Ubuntu раннера |

| Секрет | Обязательный | Описание |
|---|---|---|
| `REGISTRY_TOKEN` | да | `secrets.GITHUB_TOKEN` с правом `packages: write` или PAT |

```yaml
jobs:
  publish:
    uses: your-org/central_ci_cd/.github/workflows/docker-publish.yml@main
    with:
      image-name: 'ghcr.io/your-org/your-app'
      user-name: 'your-org'
      dockerfile: 'Dockerfile'
    secrets:
      REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Пример: CI + публикация образа по тегу

Типовая схема: CI запускается на каждый пуш и PR, а публикация образа — только при создании git-тега.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, "feature*"]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: your-org/central_ci_cd/.github/workflows/go-ci-complete.yml@main
    with:
      go-version: '1.23.2'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    uses: your-org/central_ci_cd/.github/workflows/docker-publish.yml@main
    with:
      image-name: 'ghcr.io/your-org/your-app'
      user-name: 'your-org'
    secrets:
      REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Замените `your-org` на имя вашей GitHub-организации или аккаунта.
