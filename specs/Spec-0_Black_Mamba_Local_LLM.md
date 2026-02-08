# Spec-0: Black Mamba — Local LLM + VS Code (Continue) Bootstrap

## 0. Назначение
Цель Spec‑0 — получить **работающую локальную нейросеть** и **IDE‑интеграцию**,
чтобы Black Mamba мог помогать писать код, генерировать документацию и выполнять инженерные задачи.

Результат Spec‑0:
- Ollama работает как локальный LLM runtime;
- Модели скачиваются на большой диск `/mnt/ufiles`;
- Установлены и проверены модели:
  - `llama3.1:8b` — универсальная
  - `qwen2.5-coder:7b` — кодинг
  - `deepseek-coder:6.7b` — кодинг (альтернатива)
- VS Code подключён к Ollama через Continue и умеет чат/редактирование/автодополнение.

---

## 1. Предпосылки
- Ubuntu 24.04+
- NVIDIA GPU с установленным драйвером
- Смонтированный диск `/mnt/ufiles`
- Установленный VS Code

---

## 2. Хранение моделей
Все модели Ollama хранятся на большом диске:

```
/mnt/ufiles/ollama/models
```

### 2.1 Подготовка каталогов
```bash
sudo mkdir -p /mnt/ufiles/ollama/models
sudo chown -R "$USER":"$USER" /mnt/ufiles/ollama
sudo chmod -R 775 /mnt/ufiles/ollama
```

### 2.2 Переменная окружения
Добавить в `~/.bashrc`:
```bash
export OLLAMA_MODELS=/mnt/ufiles/ollama/models
```

---

## 3. Ollama

### 3.1 Проверка
```bash
ollama --version
```

### 3.2 Запуск
```bash
sudo systemctl enable --now ollama
```

### 3.3 Проверка API
```bash
curl http://127.0.0.1:11434/api/tags
```

---

## 4. Установка моделей

### 4.1 Универсальная модель
```bash
ollama pull llama3.1:8b
```

### 4.2 Кодинг-модель (основная)
```bash
ollama pull qwen2.5-coder:7b
```

### 4.3 Кодинг-модель (альтернатива)
```bash
ollama pull deepseek-coder:6.7b
```

### 4.4 Проверка
```bash
ollama list
du -sh /mnt/ufiles/ollama/models
```

---

## 5. Быстрый тест
```bash
ollama run llama3.1:8b
ollama run qwen2.5-coder:7b
ollama run deepseek-coder:6.7b
```

---

## 6. VS Code + Continue

### 6.1 Установка
VS Code → Extensions → **Continue** → Install

### 6.2 Подключение
- Provider: Ollama
- URL: `http://127.0.0.1:11434`
- Models:
  - Chat: `llama3.1:8b`
  - Coding: `qwen2.5-coder:7b`

---

## 7. Правила работы
- Архитектура, спецификации, тексты — `llama3.1`
- Код, рефакторинг, тесты — `qwen2.5-coder`
- Контекст в IDE держать 4–8K токенов

---

## 8. Definition of Done
- Ollama запущена
- Все модели скачаны в `/mnt/ufiles`
- VS Code + Continue успешно работает с локальной LLM

---

## 9. Следующий шаг
**Spec‑1:** Python API + инструменты (git, files, commands) — начало Black Mamba.
