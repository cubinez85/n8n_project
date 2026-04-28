# 📋 Project Prompt: AI-Automation Platform на базе n8n

## 🎯 Цель проекта
Создать единую платформу автоматизации для генерации контента (изображения, видео, аудио) и управления бизнес-процессами с помощью low-code workflow-движка **n8n**, интегрированного с локальными AI-моделями.

http://n8n.cubinez.ru/home/workflows
---

## 🏗 Архитектура системы

```
┌─────────────────────────────────────────────────────┐
│                    CLIENT LAYER                      │
│  • Веб-интерфейс n8n (http://<IP>:8082)             │
│  • Telegram-бот для пользователей                    │
│  • Webhook API для внешних систем                    │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│                 PROXY LAYER (Nginx)                  │
│  • Порт 80/443 → проксирование на сервисы           │
│  • SSL-терминация (опционально)                      │
│  • Rate limiting, CORS, базовая защита               │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│              ORCHESTRATION LAYER (n8n)               │
│  • Docker-контейнер с PostgreSQL (постоянное хранилище)│
│  • Workflow-движок с REST API и Webhook-триггерами  │
│  • Интеграции: Email, Telegram, HTTP, Code, FS      │
└─────────────────┬───────────────────────────────────┘
                  │
    ┌─────────────┴─────────────┐
    │                           │
┌───▼───────────┐     ┌────────▼────────┐
│  AI: ComfyUI  │     │  AI: SadTalker  │
│  (WSL-2 / Docker)  │  (Docker + GPU)  │
│  • Генерация изображений    │  • Анимация аватаров    │
│  • Image-to-image           │  • Lip-sync (фото + аудио)│
│  • AnimateDiff              │  • API на FastAPI/Uvicorn│
└─────────────┘     └─────────────────┘
```

---

## 📦 Компоненты и технологии

| Компонент | Версия / Образ | Назначение | Порт |
|-----------|---------------|------------|------|
| **n8n** | `n8n-io/n8n:latest` | Оркестрация воркфлоу, API, UI | `5678` (внутр.) |
| **PostgreSQL** | `postgres:13-alpine` | Хранение воркфлоу, пользователей, логов | `5432` (внутр.) |
| **Nginx** | `nginx:alpine` (на хосте) | Reverse proxy, SSL, маршрутизация | `80` → `8082` |
| **ComfyUI** | manual install (WSL-2) | Генерация изображений, анимация | `8188` |
| **SadTalker** | custom Docker image | Говорящие аватары (фото + аудио → видео) | `8000` → `8082` |
| **SMTP** | внешний сервер `95.174.94.246:25` | Отправка уведомлений, отчётов | — |

---

## 🔑 Ключевые возможности

### 🤖 Автоматизация контента
- [ ] Генерация изображений по текстовому промпту (ComfyUI + Stable Diffusion)
- [ ] Создание говорящих аватаров: `фото + текст → TTS → SadTalker → видео`
- [ ] Пакетная обработка: очередь задач, повторные попытки, логирование

### 📡 Интеграции
- [ ] **Telegram**: приём команд, отправка результатов, интерактивные кнопки
- [ ] **Email**: уведомления об ошибках, отправка отчётов, подтверждение действий
- [ ] **Webhook API**: внешний запуск воркфлоу из других систем (сайт, мобильное приложение)
- [ ] **PostgreSQL**: чтение/запись данных, аналитика, хранение метаданных

### ⚙️ Управление и мониторинг
- [ ] Веб-интерфейс n8n для визуального редактирования воркфлоу
- [ ] Логирование выполнений в БД + отправка алертов при сбоях
- [ ] Переменные окружения для безопасного хранения секретов (`.env`)
- [ ] Автозапуск через `systemd` + `docker compose restart: always`

---

## 🗂 Структура проекта на сервере

```
~/
├── n8n-docker/
│   ├── docker-compose.yml      # n8n + postgres
│   ├── .env                    # секреты, настройки БД, SMTP
│   ├── nginx/default.conf      # конфиг reverse proxy
│   ├── postgres/               # volume: данные PostgreSQL
│   └── n8n_/                   # volume: данные n8n (воркфлоу, ключи)
│
├── ComfyUI/                    # WSL-2, генерация изображений
│   ├── input/                  # исходные файлы
│   ├── output/                 # результаты
│   ├── custom_nodes/           # расширения (AnimateDiff, etc.)
│   └── venv/                   # Python virtualenv
│
├── SadTalker/                  # Docker, анимация аватаров
│   ├── docker-compose.yml
│   ├── .dockerignore           # исключение моделей из сборки
│   ├── input/                  # фото + аудио для генерации
│   ├── results/                # готовые видео
│   ├── checkpoints/            # модели (монтируются из хоста)
│   └── gfpgan/weights/         # энхансер лиц
│
└── scripts/                    # вспомогательные утилиты
    ├── backup-n8n.sh           # бэкап БД и воркфлоу
    ├── update-all.sh           # обновление образов и кода
    └── monitor-resources.sh    # мониторинг CPU/RAM/VRAM
```

---

## 🔐 Безопасность и секреты

```env
# .env — НЕ КОММИТИТЬ В РЕПОЗИТОРИЙ!
# ===================================
# PostgreSQL
POSTGRES_PASSWORD=REDACTED

# n8n
N8N_ENCRYPTION_KEY=REDACTED
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=REDACTED

# SMTP
SMTP_PASSWORD=REDACTED

# Telegram
TELEGRAM_BOT_TOKEN=REDACTED

# API Keys (для внешних сервисов, если используются)
FAL_AI_KEY=REDACTED
REPLICATE_TOKEN=REDACTED
```

**Рекомендации:**
- ✅ Использовать `.env` + `docker-compose config` для проверки
- ✅ Ограничить доступ к портам через `ufw` / cloud firewall
- ✅ Включить 2FA в интерфейсе n8n (если поддерживается)
- ✅ Регулярно обновлять образы: `docker compose pull && docker compose up -d`

---

## 🔄 Типовые сценарии (Use Cases)

### 🎬 Сценарий 1: «Говорящий аватар по запросу»
```
1. Пользователь отправляет фото + текст в Telegram-бота
2. n8n: Webhook → TTS (edge-tts) → SadTalker API → ffmpeg (склеивание звука)
3. Результат: видео отправляется обратно в Telegram + сохраняется в БД
4. Уведомление об успехе/ошибке — на email администратора
```

### 🎨 Сценарий 2: «Генерация баннера для сайта»
```
1. Менеджер заполняет форму на сайте (текст, стиль, размер)
2. n8n: Webhook → ComfyUI API (workflow JSON) → ожидание генерации
3. Результат: изображение загружается на S3 / отправляется на email
4. Метаданные (промпт, модель, время) сохраняются в PostgreSQL
```

### 📊 Сценарий 3: «Ежедневный отчёт + алерты»
```
1. Schedule-триггер в n8n (каждый день в 09:00)
2. Запрос к БД / внешнему API → агрегация данных
3. Формирование отчёта (текст + график через Code-ноду)
4. Отправка: Email (SMTP) + Telegram + сохранение в БД
5. При ошибке в любом шаге — мгновенный алерт админу
```

---

## 🚀 Команды для управления

```bash
# ===== n8n =====
cd ~/n8n-docker
docker compose up -d              # запуск
docker compose logs -f n8n        # логи
docker compose exec n8n bash      # доступ в контейнер
docker compose pull && docker compose up -d  # обновление

# ===== SadTalker =====
cd ~/SadTalker
docker compose up -d              # запуск API
docker compose logs -f            # мониторинг
docker exec -it sadtalker-sadtalker-1 bash  # доступ в контейнер

# ===== Резервное копирование =====
# Бэкап PostgreSQL
docker exec n8n_postgres pg_dump -U n8n n8n > backup_$(date +%F).sql

# Бэкап воркфлоу n8n (через API)
curl -u admin:password http://localhost:8082/api/v1/workflows \
  | jq '.data' > workflows_backup_$(date +%F).json

# Синхронизация на внешний носитель
rsync -avz ~/n8n-docker/postgres/ user@backup-server:/backups/n8n-db/
```

---

## 📈 Масштабирование и развитие

### Краткосрочно (1-2 недели)
- [ ] Добавить очередь задач (Redis + Bull) для долгих генераций
- [ ] Реализовать API-ключи и rate limiting для webhook
- [ ] Настроить автоматический бэкап в облако (S3 / Yandex Object Storage)

### Среднесрочно (1 месяц)
- [ ] Вынести ComfyUI/SadTalker на выделенный GPU-сервер (Cloud.ru / RunPod)
- [ ] Добавить поддержку очередей приоритетов (premium / free пользователи)
- [ ] Внедрить метрики (Prometheus + Grafana) для мониторинга загрузки

### Долгосрочно (3+ месяца)
- [ ] Микросервисная архитектура: отдельный сервис для TTS, для генерации, для отправки
- [ ] Поддержка мульти-тенантности (разные пользователи / проекты)
- [ ] Интеграция с LLM (локальная Mistral / API) для умной обработки промптов

---

## 🆘 Диагностика проблем

```bash
# 1. Проверка здоровья сервисов
docker compose ps                    # статус контейнеров
curl http://localhost:8082/health    # n8n API
curl http://localhost:8000/health    # SadTalker API

# 2. Логи
docker compose logs -f n8n           # ошибки оркестрации
docker compose logs -f sadtalker     # ошибки генерации
sudo journalctl -u nginx -f          # ошибки прокси

# 3. Ресурсы
docker stats                         # CPU/RAM контейнеров
nvidia-smi                           # загрузка GPU (на хосте)
free -h && df -h                     # память и диск

# 4. Сеть
ss -tlnp | grep -E '8082|8000|8188'  # кто слушает порты
curl -v http://localhost:8000/generate  # тест API с детализацией
```

---

## 📄 Лицензия и источники

- **n8n**: [Fair-code License](https://n8n.io/legal/licensing) — бесплатно для внутреннего использования
- **ComfyUI**: MIT License — https://github.com/comfyanonymous/ComfyUI
- **SadTalker**: MIT License — https://github.com/OpenTalker/SadTalker
- **Модели**: проверять лицензии каждой модели (SD 1.5, GFPGAN, SadTalker checkpoints)

---

> 💡 **Совет для команды**:  
> Храните этот документ в `README.md` корня проекта.  
> Обновляйте раздел «Сценарии» при добавлении новых воркфлоу.  
> Используйте `#project-prompt` в чатах для быстрого поиска.

---

**Готово!** 🎯  
Этот промт можно:
- Использовать как `README.md` для репозитория
- Отправить новому разработчику для онбординга
- Применить как ТЗ для расширения функционала
- Загрузить в LLM для генерации документации / кода / тестов


