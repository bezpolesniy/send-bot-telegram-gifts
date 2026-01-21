# 🏆 Telegram Gift Auction

Premium Real-Time Auction Platform для Telegram Mini Apps.

## 🚀 Этап 1 — Core Auction Engine

Базовая инфраструктура аукционов:

- ✅ MongoDB модели (User, Auction, Round, Bid, Balance)
- ✅ Двухбалансовая система (available/locked)
- ✅ Аукционная логика с многораундовой моделью
- ✅ Anti-sniping механизм
- ✅ REST API
- ✅ JWT авторизация

## � Требования

Перед установкой убедитесь, что у вас установлены:

### Обязательные
- **Node.js** v18 или выше ([скачать](https://nodejs.org/))
- **npm** v9+ (устанавливается вместе с Node.js)
- **Docker Desktop** ([скачать](https://www.docker.com/products/docker-desktop/))
  - Для Windows: Docker Desktop for Windows
  - Для macOS: Docker Desktop for Mac
  - Для Linux: Docker Engine

### Опциональные
- **Git** для клонирования репозитория ([скачать](https://git-scm.com/))
- **MongoDB Compass** для просмотра БД ([скачать](https://www.mongodb.com/products/compass))

### Проверка установки

```bash
# Проверить версию Node.js
node --version  # должно быть v18.0.0 или выше

# Проверить версию npm
npm --version   # должно быть 9.0.0 или выше

# Проверить Docker
docker --version
docker-compose --version
```

## �📦 Установка

### Шаг 1: Клонирование репозитория

```bash
git clone <repository-url>
cd telegram-gift-auction
```

### Шаг 2: Установка зависимостей

```bash
# Установить зависимости для backend
npm install

# Установить зависимости для frontend
cd client
npm install
cd ..
```

### Шаг 3: Настройка окружения

```bash
# Создать .env файл (скопировать из .env.example)
cp .env.example .env

# Открыть .env и настроить переменные окружения
# Минимальные настройки:
# - MONGODB_URI=mongodb://localhost:27017/telegram-auction
# - REDIS_URL=redis://localhost:6379
# - JWT_SECRET=your-secret-key-here
```

### Шаг 4: Запуск MongoDB и Redis через Docker

```bash
# Запустить MongoDB
docker run -d -p 27017:27017 --name mongodb mongo:latest

# Запустить Redis
docker run -d -p 6379:6379 --name redis redis:7.2

# Проверить, что контейнеры запущены
docker ps
```

### Шаг 5: Заполнение тестовыми данными

```bash
# Запустить seed для создания тестовых данных
npm run seed
```

### Шаг 6: Запуск приложения

```bash
# Вариант 1: Запустить через start.bat (Windows)
start.bat

# Вариант 2: Запустить вручную
# Backend (в одном терминале)
npm run dev

# Frontend (в другом терминале)
cd client
npm run dev
```

Приложение будет доступно:
- **Backend API**: http://localhost:3000
- **Frontend**: http://localhost:5173

## 🛑 Остановка приложения

```bash
# Остановить серверы через stop.bat (Windows)
stop.bat

# Остановить все Docker контейнеры
docker-compose down

# Или остановить контейнеры по отдельности
docker stop mongodb redis

# Удалить контейнеры (если нужно)
docker rm mongodb redis
```

## 🔧 Конфигурация

Настройки в `.env`:

```env
PORT=3000
MONGODB_URI=mongodb://localhost:27017/telegram-auction
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key

# Anti-sniping
ANTI_SNIPE_THRESHOLD_SECONDS=30
ANTI_SNIPE_EXTENSION_SECONDS=15
MAX_ANTI_SNIPE_EXTENSIONS=5
```

## 📡 API Endpoints

### Health
- `GET /api/health` - Проверка статуса API

### Users
- `POST /api/users/auth/telegram` - Регистрация/вход через Telegram
- `GET /api/users/me/profile` - Профиль текущего пользователя
- `GET /api/users/me/balance` - Баланс пользователя
- `POST /api/users/me/balance/add` - Пополнить баланс (тест)
- `GET /api/users/leaderboard` - Лидерборд

### Auctions
- `GET /api/auctions` - Все аукционы
- `GET /api/auctions/active` - Активные аукционы
- `GET /api/auctions/:id` - Информация об аукционе
- `POST /api/auctions` - Создать аукцион
- `POST /api/auctions/:id/start` - Запустить аукцион
- `GET /api/auctions/:id/leaderboard` - Лидерборд аукциона

### Bids
- `POST /api/bids` - Сделать ставку
- `GET /api/bids/round/:roundId/top` - Топ ставки раунда
- `GET /api/bids/round/:roundId/my-bid` - Моя ставка в раунде
- `GET /api/bids/history` - История ставок

## 🧪 Тестирование

```bash
# Авторизация (получить токен)
curl -X POST http://localhost:3000/api/users/auth/telegram \
  -H "Content-Type: application/json" \
  -d '{"telegramId": "111111111", "firstName": "Test"}'

# Получить активные аукционы
curl http://localhost:3000/api/auctions/active

# Сделать ставку (с токеном)
curl -X POST http://localhost:3000/api/bids \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"auctionId": "...", "roundId": "...", "amount": 100}'
```

## 💰 Двухбалансовая система

```
┌─────────────┐     ┌─────────────┐
│  Available  │ ──▶ │   Locked    │   Ставка
│   (можно    │     │ (заморожено │
│  тратить)   │     │  для ставки)│
└─────────────┘     └─────────────┘
       ▲                   │
       │                   ▼
       │            ┌─────────────┐
       └──────────  │   Outbid    │   Возврат
                    │  (перебит)  │
                    └─────────────┘
```

## 🛡️ Anti-Sniping

Если ставка приходит в последние 60 секунд:
- Раунд продлевается на 30 секунд
- Нельзя поставить ставку в последнюю секунду
- Неограниченное количество продлений

## 📂 Структура проекта

```
src/
├── config/          # Конфигурация
├── controllers/     # REST контроллеры
├── middleware/      # Auth, Error handling
├── models/          # MongoDB модели
├── routes/          # API роуты
├── services/        # Бизнес-логика
├── types/           # TypeScript типы
├── utils/           # Утилиты
├── scripts/         # Скрипты (seed)
└── index.ts         # Entry point
```

## 🔜 Следующие этапы

- **Этап 2**: WebSocket & Real-Time
- **Этап 3**: Social & Gamification
- **Этап 4**: Premium UI
- **Этап 5**: Auto-Bid Engine
- **Этап 6**: Telegram Bot Integration
- **Этап 7**: Load Testing
