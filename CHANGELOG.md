# Changelog

## [Unreleased]

### Added

- **`go-build.yml`** — workflow для сборки Go приложения: checkout, setup-go, сборка бинарника и загрузка артефакта (retention 7 дней).
- **`go-security-scan.yml`** — workflow для сканирования уязвимостей через `govulncheck`.
- **`go-ci-complete.yml`** теперь объединяет все четыре workflow (`go-lint`, `go-test`, `go-security-scan`, `go-build`) с умным графом зависимостей.

### Changed

- **README.md** обновлён: добавлены разделы для `go-build.yml` и `go-security-scan.yml`, описание `go-ci-complete.yml` актуализировано (граф зависимостей джобов, полный список параметров).
- Улучшены `description` у всех `inputs` в `go-security-scan.yml` (ранее отсутствовали).
