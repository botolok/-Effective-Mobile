# Задание 3.2: Алгоритм создания пользователя

## Пошаговый алгоритм (бэкенд)

**Шаг 1. Приём запроса**

- Получен POST /api/v1/register с JSON-телом

**Шаг 2. Валидация входных данных**

- Проверить наличие обязательных полей: email, password, firstName, lastName
- Проверить формат email (regex)
- Проверить password: длина 8-64, наличие цифр, заглавных и строчных букв
- Проверить firstName и lastName: не пустые, только буквы, дефис, пробел
- Проверить phone (если передан): формат +XXXXXXXXXXXX

Если ошибка → ответ 400 Bad Request с указанием поля и причины

**Шаг 3. Проверка уникальности email**

- Выполнить запрос к БД: SELECT EXISTS(SELECT 1 FROM users WHERE email = :email)
- Если пользователь существует → ответ 409 Conflict

**Шаг 4. Хэширование пароля**

- Сгенерировать соль (bcrypt, cost factor = 12)
- Сохранить хэш, исходный пароль не хранить

**Шаг 5. Создание записи в БД**

- Сгенерировать UUID
- Записать: id, email, password_hash, first_name, last_name, phone, created_at = NOW(), status = 'pending_verification'

**Шаг 6. Генерация токена подтверждения email**

- JWT токен с payload: { email, exp: NOW() + 24h }
- Подписать секретным ключом

**Шаг 7. Отправка письма (асинхронно)**

- Поставить задачу в очередь (RabbitMQ / Redis)
- Не блокировать ответ пользователю

**Шаг 8. Возврат ответа**

- HTTP 201 Created
- Body: { userId, email, firstName, lastName, createdAt, status }

## Примеры запросов и ответов

### Успешный запрос

````json
{
  "email": "ivan@example.com",
  "password": "Password123",
  "firstName": "Иван",
  "lastName": "Петров",
  "phone": "+79001234567"
}

### Успешный ответ
```json
 {
  "timestamp": "2026-05-27T10:30:00Z",
  "status": 201,
  "message": "User registered successfully",
  "userId": 12345
}

### Ошибка дубликата
```json
{
  "timestamp": "2026-05-27T10:30:00Z",
  "status": 409,
  "error": "Conflict",
  "message": "User with email ivan@example.com already exists",
  "path": "/api/v1/register"
}


````
