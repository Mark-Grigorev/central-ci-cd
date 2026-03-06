# Go Workflows for GitHub Actions

Централизованные GitHub Actions workflow для Go проектов.

## 📦 Доступные workflow

### 1. Go Lint (`go-lint.yml`)
Запускает golangci-lint для проверки кода.

**Использование:**
```yaml
jobs:
  lint:
    uses: your-org/go-workflows/.github/workflows/go-lint.yml@v1
    with:
      go-version: '1.23.2'      # опционально
      linter-version: 'v1.60.1'  # опционально
      working-directory: '.'      # опционально
```
### 2. Go Test (go-test.yml)
Запускает тесты с покрытием.

Использование:
```
yaml
jobs:
  test:
    uses: your-org/go-workflows/.github/workflows/go-test.yml@v1
    with:
      go-version: '1.23.2'
      os: 'ubuntu-latest'
      test-flags: '-v -race -cover'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

### 3. Go Complete CI (go-ci-complete.yml)
Объединяет линтер и тесты.

Использование:
```
yaml
jobs:
  ci:
    uses: your-org/go-workflows/.github/workflows/go-ci-complete.yml@v1
    with:
      go-version: '1.23.2'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

🎯 Использование в вашем Go проекте
```
name: CI/CD Pipeline

on: 
  push:
    branches: ["main", "feature*"]
  pull_request:
    branches: ["main", "feature*"]

jobs:
  # Используем готовый workflow из центрального репозитория
  ci:
    uses: your-org/go-workflows/.github/workflows/go-ci-complete.yml@v1
    with:
      go-version: '1.23.2'
      linter-version: 'v1.60.1'
      os: 'ubuntu-22.04'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```