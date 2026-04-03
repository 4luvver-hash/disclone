# SETUP.md - Подробное руководство запуска

## Этап 1: Подготовка

### 1.1 Проверка системных требований

Windows PowerShell:
```powershell
# Проверь Docker
docker --version
docker-compose --version

# Проверь Node.js
node --version
npm --version
```

### 1.2 Клонирование и базовая конфигурация

```powershell
# Навигация в проект
cd C:\Users\User\Discord-clone

# Проверь, что все папки созданы
dir backend
dir frontend

# Проверь .env
cat .env
```

## Этап 2: Запуск с Docker Compose

### 2.1 Первый запуск (самый безопасный способ)

```powershell
# Убедись, что Docker Desktop запущен

# Перейди в корень проекта
cd C:\Users\User\Discord-clone

# Запусти контейнеры
docker-compose up --build

# Первый запуск займёт 2-3 минуты:
# - Fetch образов (PostgreSQL, Redis, Node)
# - npm install в контейнерах
# - Компиляция TypeScript
```

Проверь логи (искай эти сообщения):
```
postgres_1 | database system is ready to accept connections
redis_1 | Ready to accept connections
backend_1 | 🚀 Backend запущен на http://localhost:3000
frontend_1 | ✓ built in XXXms
```

### 2.2 Локальный выход из контейнеров

```powershell
# Если хочешь запустить без Docker (требует локального Node.js)

# Terminal 1 - PostgreSQL + Redis (в контейнерах)
docker-compose up postgres redis

# Terminal 2 - Backend (локально)
cd backend
npm install
npm run start:dev

# Terminal 3 - Frontend (локально)
cd frontend
npm install
npm run dev:vite

# Terminal 4 - Electron (локально)
cd frontend
npm run dev:electron
```

## Этап 3: Первый запуск приложения

### 3.1 Открытие приложения

1. Electron окно откроется автоматически (или вручную запусти `frontend > npm run dev:electron`)

2. Увидишь страницу логина:
   - Либо регистрируйся (создай новый аккаунт)
   - Либо используй тестовый аккаунт (если уже есть)

### 3.2 Регистрация тестового пользователя

С помощью curl (в новом PowerShell):

```powershell
# Регистрация
$body = @{
    username = "testuser"
    email = "test@test.com"
    password = "password123"
} | ConvertTo-Json

$response = Invoke-WebRequest -Uri "http://localhost:3000/auth/register" `
  -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body $body

$response.Content | ConvertFrom-Json

# Вывод (скопируй token):
# {
#   "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
#   "user": { "id": "...", "username": "testuser", ... }
# }
```

Или через приложение (проще):
1. Нажми "Зарегистрируйся"
2. Заполни форму
3. Готово! Автоматически перейдёшь в чат

### 3.3 Проверка подключения

- Мониторь веб-консоль (F12 в Electron окне):
  - Должно быть "Backend API on ws://localhost:3000/chat"
  - Статус подключения должен быть "connected"

- Проверь Network таб:
  - Есть WebSocket соединение `/socket.io/`
  - Статус 101 (upgrade)

## Этап 4: Тестирование функционала

### 4.1 Тестирование чата

Открой **2 окна Electron приложения**:

```powershell
# Terminal с Electron
npm run dev:electron

# Или запусти второй процесс
cd frontend
npx electron .
```

1. В первом окне: логинись как `user1`
2. Во втором окне: логинись как `user2`
3. Отправь сообщение из первого окна
4. Оно покажется во втором окне в реальном времени

### 4.2 Проверка индикатора печати

- Начни печатать в поле ввода
- Во втором окне увидишь "testuser печатает..."
- Прекратишь печатать - индикатор исчезнет

### 4.3 Проверка групування сообщений

- Отправь несколько сообщений подряд
- Они должны объединиться в одну группу (один аватар + время)
- Если подождёшь 5 минут и отправишь ещё - будет новая группа

## Этап 5: Отладка (Debug)

### 5.1 Логи Backend

Окно Docker Compose:
```
backend_1 | [Nest] 1 - 01/01/2024, 12:00:00 AM   LOG [NestFactory]
backend_1 | ✅ testuser подключился
backend_1 | message:send from testuser: "Привет"
```

### 5.2 Логи Frontend

Консоль Electron (F12):
```javascript
// Автоматические логи:
console.log('✅ Socket connected')
console.log('Новое сообщение:', message)
console.log('user123 печатает...')
```

### 5.3 Логи WebSocket

DevTools → Application → Cookies (для localStorage):
```
auth: {"token":"eyJ...","user":{"id":"...","username":"...","isOnline":true}}
```

### 5.4 Проверка БД

PostgreSQL (если хочешь заглянуть):

```bash
# Вход в контейнер
docker-compose exec postgres psql -U discord -d discord_db

# SQL команды
\dt                                    # Список таблиц
SELECT * FROM users;                 # Пользователи
SELECT * FROM messages LIMIT 5;      # Последние 5 сообщений
SELECT * FROM channels;              # Каналы
```

## Этап 6: Сборка для Production

### 6.1 Сборка Electron приложения (.exe)

```powershell
cd frontend

# Собрать фронтенд
npm run build:vite

# Собрать Electron приложение
npm run build:electron

# Готовый .exe будет в:
# frontend/release/MyChat Setup 1.0.0.exe

# Установка и запуск на чистой машине
# 1. Запусти This backend отдельно (Docker)
# 2. Установи .exe
# 3. Приложение запустится с настоящим окном
```

### 6.2 Для Mac / Linux

```bash
# Mac (.dmg)
npm run build:electron

# Linux (.AppImage)
npm run build:electron
```

## Типичные проблемы и решения

### Проблема: "Cannot connect to backend"

Решение:
```powershell
# Проверь, что backend запущен
docker-compose logs backend

# Проверь, что порт 3000 открыт
netstat -ano | findstr :3000

# Если не помогло, переподними контейнеры
docker-compose down
docker-compose up --build
```

### Проблема: "PostgreSQL connection timeout"

Решение:
```powershell
# Подожди 30 секунд при первом запуске

# Если остаётся проблема:
docker-compose restart postgres

# Проверь статус БД
docker-compose logs postgres | Select-String "ready to accept connections"
```

### Проблема: "Port 5173 is already in use"

Решение:
```powershell
# Найди процесс на порту 5173
netstat -ano | findstr :5173

# Убей процесс (замени PID на реальный процесс ID)
taskkill /PID <PID> /F

# Или измени порт в vite.config.ts
```

### Проблема: "Electron окно не открывается"

Решение:
```powershell
# Убедись что npm зависимости установлены
cd frontend
npm install

# Убедись что Vite dev server запущен (порт 5173)
npm run dev:vite

# В другом терминале запусти Electron
npm run dev:electron
```

### Проблема: "Cannot read properties of undefined (reading 'electronAPI')"

Решение:
- Это происходит если запустить React на обычном браузере вместо Electron
- Всегда используй `npm run dev:electron`, не открывай вручную http://localhost:5173

## Быстрые команды

```powershell
# === ОСТАНОВКА ===
# Ctrl+C в терминале с docker-compose

# === ОЧИСТКА ВСЕГО ===
docker-compose down
docker volume rm discord-clone_postgres_data

# === ПОЛНЫЙ ПЕРЕЗАПУСК ===
docker-compose down
docker-compose up --build

# === ПРОСМОТР ЛОГОВ ===
docker-compose logs backend -f     # Backend в реальном времени
docker-compose logs frontend -f    # Frontend
docker-compose logs postgres       # PostgreSQL

# === ОСТАНОВКА КОНКРЕТНОГО СЕРВИСА ===
docker-compose stop backend
docker-compose start backend

# === УДАЛЕНИЕ КОНТЕЙНЕРОВ ===
docker-compose rm
```

## Следующие шаги

1. **Изучи код**: Начни с [backend/src/gateway/chat.gateway.ts](../backend/src/gateway/chat.gateway.ts)

2. **Добавь свои каналы**: Отредактируй [backend/src/channels/channels.service.ts](../backend/src/channels/channels.service.ts)

3. **Кастомизируй интерфейс**: Измени цвета в [frontend/src/index.css](../frontend/src/index.css)

4. **Добавь функции**:
   - Личные сообщения (DM)
   - Редактирование/удаление сообщений
   - Загрузка файлов
   - Голосовые вызовы (не забудь про mediasoup!)

## Словарь терминов

- **WebSocket** - двусторонний канал связи (используется для чата в реальном времени)
- **Gateway** - в NestJS - обработчик WebSocket событий
- **TypeORM** - objektno-relatsionnyy mapper dlya bazy dannyh
- **JWT** - токен для аутентификации (полная форма: JSON Web Token)
- **Electron** - фреймворк для десктопных приложений (используется Discord, VSCode, Slack)
- **Tailwind CSS** - утилит-фёрст фреймворк для CSS

---

**Готов к следующему шагу?** Напиши "дальше" и перейдём к голосовым вызовам с mediasoup!
