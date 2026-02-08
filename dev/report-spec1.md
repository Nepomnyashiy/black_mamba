# Отчёт по Spec-1: Local DevAgent + Web UI (Ansible-first)

## 1) Назначение
Зафиксировать состояние подготовки Spec-1 в репозитории и артефакты деплоя/документации.

## 2) Что реализовано в репозитории
1. **Каркас Ansible-деплоя**
   - Inventory, group vars и playbook для локального развёртывания.
   - Задачи: создание node layout, рендер `.env`, рендер `docker-compose.yml`, запуск `docker compose`, pull моделей, healthcheck.

2. **Шаблоны деплоя**
   - `env.j2` для генерации runtime `.env`.
   - `docker-compose.yml.j2` для Open WebUI с режимами `host/container` для Ollama.

3. **Документация**
   - `docs/runbook_local.md`: шаги запуска, проверка, обновление моделей и пути хранения.
   - `docs/security.md`: правила хранения секретов и исключений из Git.

4. **Спецификации**
   - Добавлены Spec-0 и Spec-1 в папку `specs/` для работы по Docs-as-Code.

5. **Git hygiene**
   - `.gitignore` обновлён, чтобы исключить vault-пароль, runtime `.env` и логи.

## 3) Статус соответствия DoD Spec-1
- **Web UI + Ollama**: инфраструктура подготовлена в репозитории (Ansible + compose). Фактический запуск должен быть выполнен оператором.
- **Модели**: предусмотрен pull через playbook; скачивание требует выполнения на целевой машине.
- **Secrets/Config**: добавлен vault-скелет и шаблон `.env`; требуется заполнение реальных секретов через `ansible-vault edit`.
- **CLI-агент `bm`**: не реализован в рамках текущего изменения (за пределами этого отчёта).

## 4) Следующие шаги
- Заполнить `deploy/secrets/vault.yml` реальными секретами через Ansible Vault.
- Запустить `ansible-playbook -i deploy/inventory.ini deploy/playbook.yml --ask-vault-pass` на целевом хосте.
- Проверить доступность `http://localhost:3000` и `http://127.0.0.1:11434/api/tags`.
