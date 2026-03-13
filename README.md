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
| `upload-coverage` | boolean | `true` | Выводить отчёт покрытия |
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

### 3. `go-ci-complete.yml` — линтер + тесты

Объединяет `go-lint.yml` и `go-test.yml` в один вызов. Оба джоба запускаются параллельно.

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

## Пример: полный CI/CD pipeline

```yaml
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

Замените `your-org` на имя вашей GitHub-организации или аккаунта.
