# Spec-1: Black Mamba — Local Developer Agent + Web UI (Ansible-first) v1.0

**Статус:** Ready-to-Implement (под реализацию)  
**Роль:** внутренний инструмент разработки и ИТ-основа компании (Black Mamba)  
**Фокус Spec-1:** локальный запуск **Ollama + Web UI** и внедрение **CLI-агента `bm`** (файлы/Git/команды) с правильным хранением конфигов и секретов через **Ansible Vault**.

---

## 0) Ключевая концепция (исправлено)

### 0.1 Где ведётся разработка
Разработка Black Mamba идёт **в рабочем репозитории**, например:

- `~/Black_MAMBA/`  (или текущая директория проекта)

В репозитории хранятся:
- исходники CLI-агента (`bm`)
- спецификации (`specs/`)
- тесты
- шаблоны деплоя (Ansible playbooks + templates)
- документация/runbook

### 0.2 Где хранится инфраструктура узла (локальная настройка машины)
Тяжёлые/секретные/рантайм-артефакты хранятся на большом диске **/mnt/ufiles**:

- `/mnt/ufiles/ollama/models` — **модели Ollama** (обязательно)
- `/mnt/ufiles/blackmamba/node/stack` — **docker compose runtime** + volumes web UI (обязательно)
- `/mnt/ufiles/blackmamba/node/secrets/.env` — **секреты** (0600) (обязательно)
- `/mnt/ufiles/blackmamba/node/logs/` — аудит/логи (обязательно)
- `/mnt/ufiles/blackmamba/node/workspaces/` — опционально: песочницы/клоны проектов (по желанию)

> В Spec-2 мы оформим это как отдельную фичу **Node Layout / Storage Policy** (переносимость путей).  
> В Spec-1 пути фиксируем как “локальная политика узла”.

---

## 1) Цель Spec-1

Поднять локально “рабочую станцию Black Mamba”, где:
- LLM крутится локально через Ollama;
- есть Web UI для быстрых чатов с моделями;
- есть CLI-агент `bm`, который работает с файлами, Git и командами;
- секреты/переменные хранятся безопасно (Ansible Vault);
- всё поднимается воспроизводимо (Ansible).

---

## 2) Definition of Done (DoD)

Spec-1 завершён, если:

### 2.1 Web UI
- Web UI доступен по `http://localhost:3000`
- Web UI подключён к Ollama и видит модели в списке
- Ollama API доступен: `curl http://127.0.0.1:11434/api/tags`

### 2.2 Модели
- модели скачаны и лежат **на /mnt/ufiles**:
  - `llama3.1:8b`
  - `qwen2.5-coder:7b`
  - `deepseek-coder:6.7b`
- команда `ollama list` (внутри контейнера или на хосте — по выбранному режиму) показывает эти модели

### 2.3 Secrets/Config
- секреты зашифрованы в `deploy/secrets/vault.yml` (Ansible Vault, можно коммитить)
- на диск развёрнут файл `/mnt/ufiles/blackmamba/node/secrets/.env` с правами `0600`
- `.env`, пароль Vault и любые decrypted файлы **не попадают в git**

### 2.4 CLI-агент `bm`
- `bm doctor` показывает OK по: Ollama, наличие моделей, доступ к нужным путям, права на логи
- `bm --repo <path> "создай файл README.md"` создаёт файл
- `bm --repo <path> "запусти pytest"` запускает тесты
- `bm --repo <path> "сделай commit"` делает коммит
- `bm --repo <path> "git push"` требует подтверждение и работает при наличии токена

### 2.5 Аудит
- на каждую задачу создаётся лог в `/mnt/ufiles/blackmamba/node/logs/<task_id>/`
  - `events.jsonl` (структурные события)
  - `summary.md` (человеческий отчёт)
  - `diff.patch` (если были изменения)

---

## 3) Предпосылки (requirements)

- Ubuntu 24.04+
- NVIDIA GPU (RTX 4060 8GB) + драйвер (nvidia-smi OK)
- Docker + Compose v2
- `/mnt/ufiles` смонтирован и доступен на запись
- VS Code + (Cline или Continue) — опционально, но рекомендовано

---

## 4) Выбор Web UI (варианты)

### 4.1 Рекомендуемый по умолчанию: Open WebUI
Причины:
- минимальная настройка
- хорошо работает с Ollama
- легко контейнеризировать

### 4.2 Альтернативы (переключаемые шаблоном)
- AnythingLLM — удобные “workspaces” и документы (сложнее сетевые нюансы)
- LibreChat — мощный интерфейс, но конфиг сложнее

**В Spec-1 фиксируем:** Open WebUI как default, но оставляем опцию смены UI через переменную `BM_UI_FLAVOR`.

---

## 5) Репозиторий разработки (структура)

Внутри `~/Black_MAMBA/`:

```
Black_MAMBA/
  specs/
    Spec-0_*.md
    Spec-1_Black_Mamba_Local_DevAgent_WebUI_Ansible.md   <-- этот документ
  deploy/
    inventory.ini
    playbook.yml
    group_vars/
      all.yml
      local.yml
    secrets/
      vault.yml                 # ansible-vault encrypted (COMMIT OK)
      vault_password.txt        # local only (NO COMMIT)
    templates/
      docker-compose.yml.j2
      env.j2
  agent/
    pyproject.toml
    src/blackmamba_cli/
      __init__.py
      main.py                   # entrypoint bm
      doctor.py
      ollama_client.py
      tools/
        fs_tools.py
        git_tools.py
        cmd_tools.py
      policy/
        policy.yaml
        guards.py
      logging/
        audit.py
  .gitignore
  README.md
```

---

## 6) Node layout (жёстко для Spec-1)

На хосте должны существовать каталоги:

- `/mnt/ufiles/ollama/models`
- `/mnt/ufiles/blackmamba/node/stack`
- `/mnt/ufiles/blackmamba/node/secrets`
- `/mnt/ufiles/blackmamba/node/logs`
- `/mnt/ufiles/blackmamba/node/workspaces` (опционально)

Права:
- `secrets/` → `0700`
- `.env` → `0600`
- `logs/` → `0775`

---

## 7) Secrets & Config (Ansible-first)

### 7.1 Что является секретом (Vault)
`deploy/secrets/vault.yml` (ansible-vault encrypted):

- `bm_master_key`
- `github_token` (PAT)
- `gitlab_token` (PAT)
- `telegram_bot_token` (заготовка под Spec-3)
- `jwt_secret` (если понадобится)
- (опционально) `webui_admin_password` (если UI поддерживает)

### 7.2 Что НЕ секрет (в git)
`deploy/group_vars/*.yml`:
- пути node layout
- выбранный UI flavor
- allowlists/denylists
- параметры моделей (ctx, temp)

### 7.3 Генерация runtime env
Ansible рендерит:
- `/mnt/ufiles/blackmamba/node/secrets/.env`

---

## 8) Деплой через Ansible (Web UI + Ollama)

### 8.1 Playbook отвечает за:
- создание директорий и прав
- рендер `.env`
- рендер `docker-compose.yml` в `/mnt/ufiles/blackmamba/node/stack/`
- `docker compose up -d`
- healthchecks
- (опционально) `ollama pull` моделей

### 8.2 Режим Ollama
Два режима, выбирается переменной `BM_OLLAMA_MODE`:

- `host` — Ollama установлена на хосте (как у тебя сейчас)
- `container` — Ollama запускается контейнером (в compose)

**Spec-1 рекомендует:** `host`, если Ollama уже стоит и работает.  
**Если хотим полностью контейнерно:** `container`.

---

## 9) CLI-агент `bm` (требования)

### 9.1 Ввод команд (куда вводить запросы)
CLI — основной вход для “управляемых действий”:

```bash
bm doctor
bm --repo . "сгенерируй README и структуру"
bm --repo . "исправь тесты и сделай commit"
bm --repo . "подготовь ветку и сделай push"
```

### 9.2 Как `bm` использует LLM
`bm` отправляет запросы в Ollama API:
- `http://127.0.0.1:11434` (host mode)
- или `http://ollama:11434` (container network, если bm бежит в контейнере — в Spec-1 bm бежит на хосте)

### 9.3 Tool layer (обязательно)
- FS: list/read/write/search/mkdir
- Git: status/diff/branch/add/commit/push
- Cmd: run_cmd (pytest/ruff/mypy) с ограничениями

### 9.4 Политики безопасности (обязательно)
- allowlist директорий (repo + workspaces)
- denylist опасных команд
- подтверждение для: `git push`, массовых удалений, подозрительных команд
- “dry-run” режим для команд (по умолчанию показываем команду перед выполнением)

### 9.5 Логирование
- JSONL события: tool calls, stdout/stderr, решения
- summary.md: человекочитаемый отчёт
- diff.patch: изменения файлов

---

## 10) Модели и параметры (под RTX 4060 8GB)

### 10.1 Набор моделей
- `llama3.1:8b` — архитектура/спеки/документация
- `qwen2.5-coder:7b` — основная модель кодинга
- `deepseek-coder:6.7b` — запасной кодер

### 10.2 Контекст/температура (старт)
- context: 4096–8192
- temp:
  - кодинг: 0.1–0.3
  - тексты: 0.3–0.7

---

## 11) Backlog: задачи (максимально чётко)

### Milestone M0 — Repo hardening (git hygiene)
**Цель:** подготовить репозиторий разработки к безопасной работе с секретами.

**Задачи:**
- M0.1 Добавить `.gitignore` для:
  - `deploy/secrets/vault_password.txt`
  - `.env`
  - любые decrypted файлы
  - логи/артефакты
- M0.2 Создать `specs/` и положить Spec-0/Spec-1
- M0.3 Создать `deploy/` каркас

**Acceptance:**
- `git status` не показывает секретов/паролей/`.env`

---

### Milestone M1 — Ansible Vault + Secrets rendering
**Цель:** секреты живут в vault и разворачиваются на /mnt/ufiles с правильными правами.

**Задачи:**
- M1.1 `ansible-vault create deploy/secrets/vault.yml`
- M1.2 создать `deploy/templates/env.j2`
- M1.3 playbook task: создать node dirs + права
- M1.4 playbook task: рендер `.env` в `/mnt/ufiles/blackmamba/node/secrets/.env` (0600)

**Acceptance:**
- `.env` существует, права 0600
- в git его нет

---

### Milestone M2 — Web UI stack (Open WebUI) + healthchecks
**Цель:** поднять web-интерфейс в контейнере и подключить к Ollama.

**Задачи:**
- M2.1 `deploy/templates/docker-compose.yml.j2`:
  - сервис `open-webui`
  - подключение к Ollama (host или container mode)
  - volume для данных UI в `/mnt/ufiles/blackmamba/node/stack/openwebui-data`
- M2.2 playbook task: рендер compose в `/mnt/ufiles/blackmamba/node/stack/docker-compose.yml`
- M2.3 playbook task: `docker compose up -d`
- M2.4 healthcheck:
  - `GET http://localhost:3000`
  - `GET http://127.0.0.1:11434/api/tags`

**Acceptance:**
- UI открывается и видит модели (после pull)

---

### Milestone M3 — Model management (pull + storage verification)
**Цель:** модели скачиваются и хранятся на `/mnt/ufiles/ollama/models`.

**Задачи:**
- M3.1 playbook task: `ollama pull` требуемых моделей (в нужном режиме)
- M3.2 проверка: размер каталога моделей, наличие manifests/blobs
- M3.3 документировать “как добавлять модель” (runbook)

**Acceptance:**
- модели видны в `api/tags` и в UI

---

### Milestone M4 — CLI-агент `bm` (MVP)
**Цель:** рабочий CLI-агент для разработки.

**Задачи:**
- M4.1 создать python пакет `agent/`:
  - entrypoint `bm`
  - конфиг loader (.env + config.yaml)
- M4.2 `ollama_client.py`: chat call + streaming (опционально)
- M4.3 tools:
  - `fs_tools.py`
  - `git_tools.py`
  - `cmd_tools.py`
- M4.4 policy:
  - `policy.yaml` (allowlist dirs, denylist cmds)
  - `guards.py` (confirmations)
- M4.5 audit logger:
  - `audit.py` → пишет `events.jsonl`, `summary.md`, `diff.patch`
- M4.6 `bm doctor`:
  - проверка Ollama
  - проверка моделей
  - проверка прав и путей
  - проверка токенов (наличие, без вывода значений)

**Acceptance:**
- `bm doctor` OK
- `bm --repo . "создай README.md"` работает
- `bm --repo . "сделай commit"` работает

---

### Milestone M5 — GitHub/GitLab push (guarded)
**Цель:** безопасный push в удалённые репозитории.

**Задачи:**
- M5.1 allowlist репозиториев (org/repo)
- M5.2 `git_push` требует подтверждение (Y/n)
- M5.3 токены берутся только из `.env` (Vault-derived)
- M5.4 audit: сохранять remote URL, branch, hash commit

**Acceptance:**
- `bm --repo . "push"` делает push только после подтверждения

---

## 12) Runbook (минимум для Spec-1)

Должны появиться документы:
- `docs/runbook_local.md`:
  - как запустить ansible deploy
  - как обновить модели
  - как добавить токен (через vault)
  - где смотреть логи
- `docs/security.md`:
  - allowlist dirs
  - denylist cmds
  - правила подтверждения

---

## 13) Команды запуска (ожидаемые)

### 13.1 Деплой
В репозитории разработки:

```bash
cd ~/Black_MAMBA
ansible-playbook -i deploy/inventory.ini deploy/playbook.yml --ask-vault-pass
```

### 13.2 Проверка
```bash
curl http://127.0.0.1:11434/api/tags
# открыть http://localhost:3000
```

### 13.3 CLI
```bash
bm doctor
bm --repo . "создай README.md и добавь секцию про запуск"
```

---

## 14) Ограничения Spec-1 (чтобы не расползалось)
- Без RAG-БД (pgvector) — это Spec-2
- Без Telegram — Spec-3
- Без server-ops — Spec-3/4

---

## 15) Следующая спецификация (Spec-2)
- Node Layout / Storage Policy (переносимые пути)
- Postgres + pgvector
- embeddings pipeline
- RAG tool для `bm` и Web UI
- пакет “project templates” (генератор структуры проектов)
