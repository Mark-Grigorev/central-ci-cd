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
