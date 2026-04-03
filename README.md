# Discord Clone - Self-Hosted Messenger

Полнофункциональный мессенджер с архитектурой Discord, построенный с использованием NestJS, React, Electron, PostgreSQL и WebSockets.

## Архитектура

```
┌─────────────────┐
│   Electron App  │ (Frontend: React + TypeScript)
│   (Главное окно)│
└────────┬────────┘
         │ HTTP/WebSocket
┌────────▼────────────────────────┐
│   NestJS Backend (Node.js)      │
│  - JWT Auth                     │
│  - WebSocket Chat Gateway       │
│  - REST API                     │
└────┬───┬─────────────┬──────────┘
     │   │             │
┌────▼─┐ │ ┌──────────┴──────────┐
│ PostgreSQL │ │     Redis       │
│ (Основная  │ │ (Сессии, Кэш)  │
│  база 
)    │ │                       │
└────────┘ └────────────────────┘
```

## Стек технологий

### Backend
- **NestJS** - модульный фреймворк на Node.js
- **PostgreSQL 16** - основная база данных
- **Redis 7** - кеширование и сессии
- **TypeORM** - ORM для работы с БД
- **Socket.IO** - WebSocket для реального времени
- **Passport + JWT** - аутентификация
- **bcrypt** - хеширование паролей

### Frontend
- **React 18** - UI фреймворк с TypeScript
- **Vite** - сборка и dev server
- **Electron** - обёртка для десктопного приложения
- **Socket.IO Client** - подключение к WebSocket
- **Zustand** - управление состоянием
- **Tailwind CSS** - стилизация
- **react-router** - маршрутизация

## Быстрый старт

### Требования
- Docker & Docker Compose
- Node.js 20+ (если запускать локально)
- npm или yarn

### Способ 1: С Docker Compose (Рекомендуется)

```bash
# 1. Клонируй репо и перейди в проект
cd Discord-clone

# 2. Скопируй .env.example в .env
cp .env.example .env

# 3. (Опционально) Сгенерируй новый JWT_SECRET
# Замени JWT_SECRET на длинную случайную строку (минимум 32 символа)

# 4. Запусти все сервисы
docker-compose up --build

# Первый запуск займёт 2-3 минуты (загрузка образов и установка зависимостей)
```

После запуска:
- **Backend API**: http://localhost:3000
- **Frontend Dev Server**: http://localhost:5173
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379

### Способ 2: Локальный запуск

#### Запуск только Docker зависимостей

```bash
# Запусти только БД и Redis
docker-compose up postgres redis

# В отдельных терминалах:

# Terminal 1 - Backend
cd backend
npm install
npm run start:dev

# Terminal 2 - Frontend (Vite)
cd frontend
npm install
npm run dev:vite

# Terminal 3 - Electron
cd frontend
npm run dev:electron
```

## Использование

### Регистрация и вход

1. Заполни форму регистрации (никнейм, email, пароль)
2. После регистрации автоматически вернёшься в чат
3. Чтобы выйти, кликни иконку выхода в левой панели

### Отправка сообщений

- Напиши текст в поле ввода внизу
- **Enter** - отправить сообщение
- **Shift + Enter** - перевод строки

### Переключение каналов

- Кликай на названия каналов в левой панели
- Поддерживаемые типы:
  - **Текстовые каналы** - обычный чат
  - **Голосовые каналы** - заготовка для видео/аудио (будет добавлено в следующих этапах)

### Индикатор набора текста

Видишь "Пользователь печатает..." - это значит кто-то прямо сейчас набирает сообщение.

## API Endpoints

### Аутентификация

```bash
# Регистрация
POST /auth/register
Body: { username, email, password }

# Вход
POST /auth/login
Body: { email, password }

# Проверка токена
GET /auth/me
Headers: Authorization: Bearer <token>
```

### Каналы

```bash
# Все каналы
GET /channels

# Канал по ID
GET /channels/:id
```

### Пользователи

```bash
# Текущий пользователь
GET /users/me
Headers: Authorization: Bearer <token>

# Пользователь по ID
GET /users/:id
```

## WebSocket События

### Подключение
```javascript
socket.emit('channel:join', { channelId: 'general' }, (res) => {
  console.log('История сообщений:', res.history);
});
```

### Отправка сообщения
```javascript
socket.emit('message:send', { 
  channelId: 'general', 
  content: 'Привет!' 
});

// Слушаем новые сообщения
socket.on('message:new', (message) => {
  console.log('Новое сообщение:', message);
});
```

### Индикатор набора текста
```javascript
socket.emit('typing:start', { channelId: 'general' });
setTimeout(() => {
  socket.emit('typing:stop', { channelId: 'general' });
}, 2000);

socket.on('typing:update', ({ userId, username, typing }) => {
  console.log(`${username} печатает: ${typing}`);
});
```

## Структура проекта

```
Discord-clone/
├── backend/                    # NestJS Backend
│   ├── src/
│   │   ├── auth/              # Аутентификация
│   │   ├── users/             # Пользователи
│   │   ├── channels/          # Каналы
│   │   ├── messages/          # Сообщения
│   │   ├── gateway/           # WebSocket Gateway
│   │   ├── app.module.ts      # Главный модуль
│   │   └── main.ts            # Entry point
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/                   # React + Electron
│   ├── src/
│   │   ├── pages/             # Страницы (Login, Chat)
│   │   ├── components/        # Компоненты (TitleBar)
│   │   ├── hooks/             # React hooks (useSocket)
│   │   ├── store/             # Zustand stores (auth, chat)
│   │   ├── main.tsx           # Entry point
│   │   └── index.css          # Tailwind
│   ├── electron/
│   │   ├── main.js            # Главный процесс Electron
│   │   └── preload.js         # Preload script (безопасность)
│   ├── index.html
│   ├── Dockerfile
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── package.json
│
├── docker-compose.yml         # Оркестрация контейнеров
├── .env                       # Переменные окружения
└── README.md
```

## Переменные окружения (.env)

```env
# Database
DATABASE_URL=postgres://discord:discord_pass@postgres:5432/discord_db
REDIS_URL=redis://redis:6379

# JWT
JWT_SECRET=замени_на_длинную_случайную_строку_минимум_32_символа
JWT_EXPIRES_IN=7d

# Server
PORT=3000
FRONTEND_URL=http://localhost:5173

# Mediasoup (для будущих голосовых вызовов)
MEDIASOUP_ANNOUNCED_IP=127.0.0.1
MEDIASOUP_MIN_PORT=40000
MEDIASOUP_MAX_PORT=40100
```

## Команды

### Backend

```bash
cd backend

# Установка зависимостей
npm install

# Development с hot reload
npm run start:dev

# Production сборка
npm run build
npm run start:prod

# Линтинг
npm run lint
```

### Frontend

```bash
cd frontend

# Установка зависимостей
npm install

# Development
npm run dev:vite          # Только Vite
npm run dev:electron      # Только Electron
npm run dev               # Оба вместе (рекомендуется)

# Production сборка
npm run build:vite        # Собрать фронтенд
npm run build:electron    # Собрать Electron приложение
npm run build             # Полная сборка

# Preview
npm run preview
```

## Решение проблем

### "postgres connection refused"
- Убедись, что PostgreSQL контейнер запущен: `docker-compose ps`
- Переподними контейнеры: `docker-compose restart postgres`

### Frontend не подключается к backend
- Проверь, что backend запущен на http://localhost:3000
- Убедись, что CORS включён (должен быть по умолчанию)
- Проверь сетевые вкладку в DevTools (F12)

### "JWT_SECRET is not set"
- Добавь JWT_SECRET в .env файл (минимум 32 символа)

### Electron не запускается
- Убедись, что у тебя установлены зависимости: `npm install`
- Проверь, что Vite dev сервер запущен на порту 5173

## Следующие этапы развития

- ✅ Этап 1: Базовый сервер + Авторизация
- ✅ Этап 2: Electron приложение + WebSocket чат
- [ ] Этап 3: Голосовые и видео вызовы (mediasoup)
- [ ] Этап 4: Стрим экрана
- [ ] Этап 5: Файлы и медиа
- [ ] Этап 6: Уведомления
- [ ] Этап 7: Deploy (Docker, VPS)

## Лицензия

MIT

---

**Автор**: Discord Clone Tutorial
**Язык**: TypeScript / JavaScript
**Версия**: 1.0.0
