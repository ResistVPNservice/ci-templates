# ci-templates

Переиспользуемые GitHub Actions для всех репозиториев ResistVPN. Сервис подключает CI одной строкой — логика пайплайна живёт здесь, а не копируется в 10 репозиториев.

| Workflow | Для чего | Что делает |
|---|---|---|
| [`python-service.yml`](.github/workflows/python-service.yml) | микросервисы `tg_*` | ruff → pytest → gitleaks → Docker-образ в GHCR → Trivy → новая версия в `deploy/envs/staging` |
| [`python-library.yml`](.github/workflows/python-library.yml) | `contracts`, `service-kit`, `tg_kit` | ruff → pytest → проверка совместимости контрактов → gitleaks → GitHub Release по тегу |

```yaml
# .github/workflows/ci.yml в сервисе
jobs:
  ci:
    uses: ResistVPNservice/ci-templates/.github/workflows/python-service.yml@v1
    with: { image: tg-user-service }
    secrets: inherit
    permissions: { contents: read, packages: write }
```

## Теги образов

| Событие | Теги в `ghcr.io/resistvpnservice/<сервис>` |
|---|---|
| push в `main` | `sha-<7 символов коммита>`, `main` |
| тег `v1.4.0` | `sha-…`, `1.4.0` |

В прод уходит ровно тот образ, который прошёл тесты и staging, — без пересборки.

## Секреты организации (Settings → Secrets and variables → Actions)

| Секрет | Зачем | Права |
|---|---|---|
| `DEPLOY_TOKEN` | автокоммит новой версии в `deploy` | fine-grained PAT: Contents — Read and write, только `deploy` |

Без `DEPLOY_TOKEN` всё работает, просто автовыкатка на staging пропускается.

Библиотеки (`contracts`, `service-kit`, `tg_kit`) публичные — CI и `docker build` ставят их из git без токена.

## Версии

Сервисы ссылаются на `@v1`. При изменении пайплайна: тег `v1.x.y` и перенос `v1` на него
(`git tag -f v1 && git push -f origin v1`). Ломающие изменения — `v2`.

Чтобы приватные репозитории могли использовать эти workflow: Settings → Actions → General → Access →
«Accessible from repositories in the 'ResistVPNservice' organization».
