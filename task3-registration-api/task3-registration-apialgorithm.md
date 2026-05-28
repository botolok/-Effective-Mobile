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

{
"email": "ivan@example.com",
"password": "Password123",
"firstName": "Иван",
"lastName": "Петров",
"phone": "+79001234567"
}

### Успешный ответ (201)

{
"userId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
"email": "ivan@example.com",
"firstName": "Иван",
"lastName": "Петров",
"createdAt": "2026-05-27T10:30:00Z",
"status": "pending_verification"
}

### Ошибка валидации (400)

{
"timestamp": "2026-05-27T10:30:00Z",
"status": 400,
"error": "Bad Request",
"message": "Password must contain at least one uppercase letter",
"path": "/api/v1/register"
}
