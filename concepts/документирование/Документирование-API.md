---
aliases: [API-документирование, Документирование-REST-API, Swagger-документирование]
tags: [api, documentation, rest, swagger, openapi, backend]
---

# Документирование API

## Обзор

Документирование API (Application Programming Interface) — это процесс создания описания интерфейсов, которые позволяют различным программным системам взаимодействовать друг с другом. Качественная документация API критически важна для разработчиков, которые будут использовать ваш API.

## Важность документирования API

Правильная документация API необходима для:
- [[Интеграция сервисов]] - упрощение процесса интеграции с другими системами
- [[Разработка клиентских приложений]] - предоставление информации для разработчиков фронтенда
- [[Поддержка и развитие API]] - упрощение внесения изменений и улучшений
- [[Безопасность API]] - четкое определение разрешений и ограничений

## Типы API документации

### 1. Интерактивная документация

Интерактивная документация позволяет пользователям тестировать API прямо из документации:

```yaml
# Пример OpenAPI спецификации (Swagger)
openapi: 3.0.0
info:
  title: User Management API
  version: 1.0.0
  description: API для управления пользователями
paths:
  /users:
    get:
      summary: Получить список пользователей
      description: Возвращает список всех пользователей с возможностью фильтрации
      parameters:
        - name: limit
          in: query
          description: Максимальное количество возвращаемых пользователей
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 10
        - name: offset
          in: query
          description: Количество пропускаемых записей
          required: false
          schema:
            type: integer
            default: 0
      responses:
        '200':
          description: Список пользователей
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  total:
                    type: integer
        '401':
          description: Неавторизованный доступ
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

### 2. REST API документация

Пример документирования REST API с примерами запросов и ответов:

#### Получение пользователя

**GET** `/api/users/{id}`

Получает информацию о пользователе по его идентификатору.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|----------|-----|--------------|----------|
| id | string | Да | Уникальный идентификатор пользователя |

**Пример запроса:**

```
GET /api/users/123
Authorization: Bearer {token}
```

**Пример ответа (200 OK):**

```json
{
  "id": "123",
  "name": "Иван Петров",
  "email": "ivan@example.com",
  "createdAt": "2023-01-15T10:30:00Z",
  "updatedAt": "2023-02-20T14:45:00Z"
}
```

**Коды ошибок:**

- `401` - Неавторизованный доступ
- `404` - Пользователь не найден
- `500` - Внутренняя ошибка сервера

### 3. GraphQL API документация

Для GraphQL API документация включает схему и примеры запросов:

```graphql
# Схема
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: String!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
}

# Пример запроса
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    posts {
      id
      title
      content
    }
  }
}
```

## Стандарты документирования API

### OpenAPI (Swagger)

OpenAPI Specification (OAS) — это стандарт для описания REST API. Он позволяет:

- Автоматически генерировать интерактивную документацию
- Создавать клиентские SDK для различных языков программирования
- Валидировать API запросы и ответы

### RAML

RESTful API Modeling Language (RAML) — это альтернатива OpenAPI, ориентированная на REST API:

```raml
#%RAML 1.0
title: User Management API
version: v1
baseUri: https://api.example.com

/users:
  get:
    description: Get all users
    queryParameters:
      limit:
        type: integer
        required: false
        default: 10
    responses:
      200:
        body:
          application/json:
            example: |
              {
                "users": [],
                "total": 0
              }
```

### API Blueprint

API Blueprint — это язык описания API на основе Markdown:

```apib
FORMAT: 1A

# User Management API

## Users [/users]

### Get All Users [GET]

+ Request (application/json)

+ Response 200 (application/json)
    + Attributes (object)
        + users (array[User])
        + total (number)

## Data Structures

### User (object)
+ id: 123 (string)
+ name: Иван Петров (string)
+ email: ivan@example.com (string)
```

## Лучшие практики документирования API

### 1. Четкое описание ресурсов

Каждый endpoint должен иметь понятное описание:

```
POST /api/users - Создает нового пользователя
GET /api/users - Возвращает список пользователей
GET /api/users/{id} - Возвращает информацию о конкретном пользователе
PUT /api/users/{id} - Обновляет информацию о пользователе
DELETE /api/users/{id} - Удаляет пользователя
```

### 2. Примеры запросов и ответов

Всегда включайте примеры:

```javascript
// Пример запроса на создание пользователя
POST /api/users
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "Иван Петров",
  "email": "ivan@example.com",
  "role": "user"
}

// Пример успешного ответа
HTTP/1.1 201 Created
{
  "id": "456",
  "name": "Иван Петров",
  "email": "ivan@example.com",
  "role": "user",
  "createdAt": "2023-01-15T10:30:00Z"
}
```

### 3. Описание ошибок

Документируйте возможные ошибки и коды ответов:

| Код | Описание | Причина |
|-----|----------|---------|
| 400 | Некорректный запрос | Неправильный формат данных |
| 401 | Неавторизованный доступ | Отсутствует или невалидный токен |
| 403 | Доступ запрещен | Недостаточно прав для выполнения операции |
| 404 | Ресурс не найден | Запрашиваемый ресурс не существует |
| 422 | Необрабатываемый запрос | Валидация данных не пройдена |
| 500 | Внутренняя ошибка сервера | Неожиданная ошибка на сервере |

### 4. Версионирование API

Документируйте версии API:

```
# Версия 1.0 (текущая)
- Поддерживает основные операции с пользователями
- Доступен по адресу /api/v1/

# Версия 2.0 (в разработке)
- Добавлена поддержка ролей пользователей
- Улучшена фильтрация данных
- Доступен по адресу /api/v2/
```

## Инструменты для документирования API

### Swagger UI

Интерактивная документация, генерируемая из OpenAPI спецификации:

- Визуальный интерфейс для тестирования API
- Возможность отправки реальных запросов
- Автоматическая генерация документации

### Postman

Инструмент для тестирования и документирования API:

- Возможность создания коллекций запросов
- Генерация документации на основе коллекций
- Совместная работа над документацией

### Redoc

Альтернативный инструмент для отображения OpenAPI спецификаций:

- Чистый и современный интерфейс
- Поддержка всех возможностей OpenAPI 3.0
- Интеграция с GitHub Pages

## Автоматическое документирование

Многие фреймворки позволяют автоматически генерировать документацию API:

### Для Node.js (Express)

```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User Management API',
      version: '1.0.0',
    },
  },
  apis: ['./routes/*.js'], // файлы с JSDoc комментариями
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));
```

### Для Python (FastAPI)

```python
from fastapi import FastAPI

app = FastAPI(
    title="User Management API",
    description="API для управления пользователями",
    version="1.0.0"
)

@app.get("/users/{user_id}")
async def get_user(user_id: str):
    """
    Получить информацию о пользователе
    
    - **user_id**: уникальный идентификатор пользователя
    - Возвращает информацию о пользователе
    """
    return {"user_id": user_id}
```

## Связанные темы

- [[Документирование-кода]]
- [[Документирование-компонентов]]
- [[Генерация-документации]]
- [[Типы-документации]]

## Заключение

Качественная документация API — ключ к успешному использованию вашего сервиса. Она должна быть актуальной, понятной и содержать все необходимые сведения для интеграции. Использование стандартов и автоматических инструментов значительно упрощает процесс создания и поддержания документации.