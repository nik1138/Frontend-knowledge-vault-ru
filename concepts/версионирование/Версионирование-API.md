---
aliases: ["API Versioning", "Версионирование API", "REST API Versioning"]
tags: ["#api-design", "#versioning", "#backend", "#frontend-development"]
---

# Версионирование API

## Определение

**Версионирование API** — это практика управления изменениями в интерфейсе программирования приложений (API), позволяющая поддерживать обратную совместимость с существующими клиентами при добавлении новых функций или изменении поведения API.

## Зачем нужно версионирование API

### Основные причины

1. **Обратная совместимость**: существующие клиенты продолжают работать без изменений
2. **Эволюция функций**: добавление новых возможностей без нарушения существующего функционала
3. **Исправление ошибок**: возможность внесения изменений в поведение API
4. **Улучшение безопасности**: обновление методов аутентификации и шифрования
5. **Оптимизация производительности**: изменения структуры данных или способов передачи

## Стратегии версионирования API

### 1. Версионирование в URL

#### Подход с префиксом версии

```
GET /api/v1/users
GET /api/v2/users
POST /api/v1/products
POST /api/v2/products
```

**Преимущества:**
- Простота реализации
- Ясность для разработчиков
- Легкость отладки

**Недостатки:**
- Нарушает REST принципы (версия в пути, а не в содержимом)
- Сложнее поддерживать несколько версий одновременно

#### Подход с поддоменами

```
GET https://v1.api.example.com/users
GET https://v2.api.example.com/users
```

### 2. Версионирование через HTTP заголовки

```
Accept: application/vnd.myapi.v1+json
Accept: application/vnd.myapi.v2+json
```

**Преимущества:**
- Соответствует REST принципам
- Не загрязняет URL
- Гибкость в управлении версиями

**Недостатки:**
- Сложнее для кэширования
- Меньше поддержки в инструментах
- Меньше интуитивности для новых разработчиков

### 3. Версионирование через параметры запроса

```
GET /api/users?version=1
GET /api/users?version=2
```

**Преимущества:**
- Простота реализации
- Легкость тестирования

**Недостатки:**
- Нарушает REST принципы
- Может конфликтовать с другими параметрами
- Меньше контроля над кэшированием

### 4. Версионирование через заголовок Accept-Version

```
Accept-Version: 1.0.0
Accept-Version: 2.0.0
```

**Преимущества:**
- Явное указание версии
- Легкость реализации на сервере

**Недостатки:**
- Меньше распространено
- Не все клиенты поддерживают

## Практические примеры

### Пример с URL версионированием

```javascript
// Frontend запросы
const apiClient = {
  v1: {
    getUsers: () => fetch('/api/v1/users'),
    getUser: (id) => fetch(`/api/v1/users/${id}`),
    createUser: (user) => fetch('/api/v1/users', {
      method: 'POST',
      body: JSON.stringify(user)
    })
  },
  v2: {
    getUsers: () => fetch('/api/v2/users'),
    getUser: (id) => fetch(`/api/v2/users/${id}`),
    createUser: (user) => fetch('/api/v2/users', {
      method: 'POST',
      body: JSON.stringify(user)
    }),
    updateUser: (id, user) => fetch(`/api/v2/users/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(user)
    })
  }
};
```

### Пример с заголовками Accept

```javascript
// Frontend запросы с заголовками
const makeRequest = (endpoint, version) => {
  return fetch(endpoint, {
    headers: {
      'Accept': `application/vnd.myapi.v${version}+json`,
      'Content-Type': 'application/json'
    }
  });
};

// Использование
const response = makeRequest('/api/users', 2);
```

## Практические рекомендации для фронтенд-разработчиков

### 1. Управление версиями API в клиенте

```javascript
class ApiClient {
  constructor(baseURL, defaultVersion = 'v1') {
    this.baseURL = baseURL;
    this.defaultVersion = defaultVersion;
  }

  async request(endpoint, options = {}, version = this.defaultVersion) {
    const url = `${this.baseURL}/api/${version}${endpoint}`;
    
    const response = await fetch(url, {
      headers: {
        'Accept': `application/vnd.myapi.${version}+json`,
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    });

    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }

    return response.json();
  }

  // Методы для разных версий
  getUsersV1() {
    return this.request('/users');
  }

  getUsersV2() {
    return this.request('/users', {}, 'v2');
  }
}
```

### 2. Обработка разных версий API

```javascript
// Адаптер для разных версий API
const adaptUserResponse = (userData, version) => {
  switch(version) {
    case 'v1':
      return {
        id: userData.id,
        name: userData.name,
        email: userData.email
      };
    case 'v2':
      return {
        id: userData.id,
        firstName: userData.firstName,
        lastName: userData.lastName,
        email: userData.email,
        createdAt: userData.created_at
      };
    default:
      return userData;
  }
};
```

### 3. Тестирование разных версий

```javascript
// Тестирование с разными версиями API
describe('API Client', () => {
  test('should handle v1 response format', async () => {
    const client = new ApiClient('https://api.example.com');
    const data = await client.getUsersV1();
    
    expect(data[0]).toHaveProperty('name');
    expect(data[0]).not.toHaveProperty('firstName');
  });

  test('should handle v2 response format', async () => {
    const client = new ApiClient('https://api.example.com');
    const data = await client.getUsersV2();
    
    expect(data[0]).toHaveProperty('firstName');
    expect(data[0]).toHaveProperty('lastName');
  });
});
```

## Стратегии миграции API

### 1. Поддержка нескольких версий одновременно

- Позволяет клиентам постепенно переходить на новую версию
- Требует больше ресурсов для поддержки
- Упрощает процесс миграции для клиентов

### 2. Постепенное отключение старых версий

- Объявление устаревания (deprecation) старых версий
- Предоставление срока для миграции
- Отключение поддержки после срока

### 3. Использование промежуточных версий

```javascript
// Пример промежуточной версии для плавной миграции
app.get('/api/v1.5/users', (req, res) => {
  // Возвращает данные в формате v1, но с новыми полями
  const users = getUsers();
  const adaptedUsers = users.map(user => ({
    ...user,
    // Новые поля возвращаются, но не обязательны
    newField: user.newField || null
  }));
  
  res.json(adaptedUsers);
});
```

## Лучшие практики

### 1. Четкая документация

```yaml
# Пример OpenAPI спецификации для разных версий
openapi: 3.0.0
info:
  title: My API
  version: 2.0.0
  description: Version 2 of the API with enhanced user management
```

### 2. Использование HTTP кодов для указания устаревания

```
HTTP/1.1 200 OK
X-API-Version: 2.0.0
X-API-Deprecated: true
X-API-Deprecated-Date: 2023-12-31
```

### 3. Мониторинг использования версий

```javascript
// Логирование версий API для аналитики
const logApiUsage = (version, endpoint, userId) => {
  console.log(`API Version ${version} used for ${endpoint} by user ${userId}`);
  // Отправка в систему аналитики
};
```

## Распространенные ошибки

### 1. Отсутствие версионирования

- API изменяется без уведомления клиентов
- Ломается совместимость с существующими приложениями
- Приводит к непредсказуемому поведению

### 2. Частые мажорные обновления

- Клиенты не успевают за изменениями
- Увеличивается нагрузка на поддержку
- Снижается доверие к API

### 3. Неправильное определение моментов для новой версии

- Новые поля в ответе не требуют новой версии
- Изменение семантики метода требует новой версии
- Добавление нового метода может не требовать новой версии

## Современные подходы

### GraphQL и версионирование

GraphQL уменьшает необходимость в версионировании благодаря:

- Гибкости запросов (клиент запрашивает только нужные поля)
- Возможности постепенного добавления полей
- Использованию директив устаревания

```graphql
type User {
  id: ID!
  name: String! @deprecated(reason: "Use firstName and lastName instead")
  firstName: String!
  lastName: String!
  email: String!
}
```

### JSON Schema для управления изменениями

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {"type": "string"},
    "name": {"type": "string"},
    "email": {"type": "string", "format": "email"}
  },
  "required": ["id", "name", "email"],
  "version": "2.0.0"
}
```

## Связанные концепции

- [[Семантическое-версионирование]]
- [[Управление-версиями-зависимостей]]
- [[Миграция-между-версиями]]
- [[Версионирование-компонентов]]
- [[Совместимость]]
- [[Обратная-совместимость]]
- [[API Design]]

## Заключение

Версионирование API — важный аспект разработки стабильных и надежных веб-приложений. Правильная стратегия версионирования позволяет эволюционировать API, не нарушая работу существующих клиентов. При выборе стратегии важно учитывать специфику проекта, ожидаемую частоту изменений и требования к совместимости.