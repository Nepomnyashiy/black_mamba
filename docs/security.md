# Security: секреты и политики (Spec-1)

## 1) Секреты
Секреты хранятся в `deploy/secrets/vault.yml` (Ansible Vault).
Runtime `.env` рендерится в:

```
/mnt/ufiles/blackmamba/node/secrets/.env
```

Права:
- каталог `secrets/` — `0700`
- файл `.env` — `0600`

## 2) Исключения из Git
В репозитории должны быть исключены:
- `deploy/secrets/vault_password.txt`
- `.env` и любые decrypted файлы
- логи и артефакты

## 3) Политики CLI-агента (ожидаемо)
- Allowlist директорий: репозитории + `/mnt/ufiles/blackmamba/node/workspaces`
- Denylist команд: destructive ops (`rm -rf /`, `mkfs`, etc.)
- Подтверждения: `git push`, массовые удаления, подозрительные команды
