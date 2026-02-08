# Runbook: локальный деплой Spec-1

## 1) Назначение
Описывает воспроизводимый запуск Spec-1 (Ollama + Open WebUI) через Ansible.

## 2) Предпосылки
- Ubuntu 24.04+
- Docker + Compose v2
- Ollama установлена и доступна (для `bm_ollama_mode: host`)
- `/mnt/ufiles` смонтирован и доступен

## 3) Подготовка Vault
Создайте/обновите файл секретов:

```bash
ansible-vault edit deploy/secrets/vault.yml
```

Ожидаемые ключи:
- `bm_master_key`
- `github_token`
- `gitlab_token`
- `telegram_bot_token`
- `jwt_secret`
- `webui_admin_password`

Пароль Vault хранится локально в `deploy/secrets/vault_password.txt` и не коммитится.

## 4) Деплой

```bash
ansible-playbook -i deploy/inventory.ini deploy/playbook.yml --ask-vault-pass
```

## 5) Проверка

```bash
curl http://127.0.0.1:11434/api/tags
```

Web UI:

```
http://localhost:3000
```

## 6) Обновление моделей

```bash
ollama pull llama3.1:8b
ollama pull qwen2.5-coder:7b
ollama pull deepseek-coder:6.7b
```

Если `bm_ollama_mode: container`:

```bash
docker exec ollama ollama pull llama3.1:8b
```

## 7) Пути данных
- Runtime/compose: `/mnt/ufiles/blackmamba/node/stack`
- Secrets: `/mnt/ufiles/blackmamba/node/secrets/.env`
- Logs: `/mnt/ufiles/blackmamba/node/logs`
- Models: `/mnt/ufiles/ollama/models`
