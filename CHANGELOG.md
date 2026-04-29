# Changelog

## [Unreleased]

### Added

- **`docker-publish.yml`** — workflow для сборки и публикации Docker-образа в GitHub Container Registry (ghcr.io). Теги формируются автоматически из git-тегов по semver (`1.2.3`, `1.2`, `latest`). Поддерживает кастомный Dockerfile, `build-args` и кеширование через GHA cache.

### Changed

- **README.md** обновлён: добавлен раздел для `docker-publish.yml` с таблицей параметров и секретов, добавлена заметка о том, что workflow является артефактом релиза, а не полноценным CD; раздел с примером переработан — теперь показаны два отдельных файла (`ci.yml` на каждый пуш/PR и `release.yml` на тег `v*`).

---

## [0.1.0] — 2025-01-01

### Added

- **`go-build.yml`** — workflow для сборки Go приложения: checkout, setup-go, сборка бинарника и загрузка артефакта (retention 7 дней).
- **`go-security-scan.yml`** — workflow для сканирования уязвимостей через `govulncheck`.
- **`go-ci-complete.yml`** — объединяет все четыре workflow (`go-lint`, `go-test`, `go-security-scan`, `go-build`) с умным графом зависимостей.

### Changed

- **README.md** обновлён: добавлены разделы для `go-build.yml` и `go-security-scan.yml`, описание `go-ci-complete.yml` актуализировано (граф зависимостей джобов, полный список параметров).
- Улучшены `description` у всех `inputs` в `go-security-scan.yml` (ранее отсутствовали).
