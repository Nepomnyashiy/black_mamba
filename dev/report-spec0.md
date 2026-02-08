# Отчёт по Spec-0: Образ локальной LLM

## Что удалось сделать
1. **Исправлена проблема с привязкой Ollama** — Ошибка `permission denied` на `/media/nsadmin/ufiles/ollama-models` была решена изменением `OLLAMA_MODELS` в `/etc/systemd/system/ollama.service.d/override.conf` на `/mnt/ufiles/ollama/models` и установкой прав доступа для пользователя `ollama`.

2. **Ollama запущена и работает** — Демон успешно запущен и слушает на `127.0.0.1:11434`:
   ```
   Active: active (running) since Sun 2026-02-08 22:44:23 MSK
   Listening on 127.0.0.1:11434 (version 0.11.10)
   ```

3. **Модели успешно загружены** — Все три модели загружены в `/mnt/ufiles/ollama/models` (итого 13G):
   - `llama3.1:8b` — универсальная модель (4.9 GB)
   - `qwen2.5-coder:7b` — кодинг-модель (4.7 GB)
   - `deepseek-coder:6.7b` — альтернативная кодинг-модель (3.8 GB)

4. **API доступен** — Команда `curl http://127.0.0.1:11434/api/tags` возвращает список моделей в JSON формате.

5. **Отчёты обновлены** — Созданы новые файлы:
   - `llm_readiness_2026-02-08_23-31-40.txt`
   - `llm_readiness_2026-02-08_23-31-40.md`

## Настройка VS Code Continue
Расширение Continue установлено (v1.2.14) и настроено через `~/.continue/config.yaml`:

```yaml
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: Llama 3.1 8B
    provider: ollama
    model: llama3.1:8b
    roles:
      - chat
      - edit
      - apply
  - name: Qwen2.5-Coder 7B
    provider: ollama
    model: qwen2.5-coder:7b
    roles:
      - autocomplete
      - edit
  - name: DeepSeek Coder 6.7B
    provider: ollama
    model: deepseek-coder:6.7b
    roles:
      - autocomplete
      - edit
  - name: Nomic Embed
    provider: ollama
    model: nomic-embed-text:latest
    roles:
      - embed
```

**Исправление:** Добавлены недостающие модели `qwen2.5-coder:7b` и `deepseek-coder:6.7b` в конфигурационный файл.

## Текущее состояние
- Ollama: ✅ Запущена и работает
- Модели: ✅ Все 3 модели загружены
- API: ✅ Доступен на 127.0.0.1:11434
- VS Code Continue: ✅ Установлен и настроен
- CUDA Toolkit: ⚠️ Не установлен (используется CUDA через драйвер)
