---
aliases: ["Регулярные выражения для валидации JSON-схем", "Regex for JSON Schema Validation"]
tags: [regexp, json, validation, api, schema]
---

# Регулярные выражения для работы с API (валидация JSON-схем)

## Общие принципы валидации JSON с использованием регулярных выражений

Хотя JSON Schema предоставляет встроенные механизмы валидации, регулярные выражения остаются мощным инструментом для проверки формата строковых значений в JSON-данных. Они особенно полезны для проверки форматов email, URL, телефонных номеров, артикулов и других специфических строковых паттернов.

## Использование регулярных выражений в JSON Schema

### 1. Валидация email в JSON Schema

```json
{
  "type": "object",
  "properties": {
    "email": {
      "type": "string",
      "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    }
  },
  "required": ["email"]
}
```

### 2. Валидация URL в JSON Schema

```json
{
  "type": "object",
  "properties": {
    "website": {
      "type": "string",
      "pattern": "^https?:\\/\\/(www\\.)?[-a-zA-Z0-9@:%._\\+~#=]{1,256}\\.[a-zA-Z0-9()]{1,6}\\b([-a-zA-Z0-9()@:%_\\+.~#?&//=]*)$"
    }
  },
  "required": ["website"]
}
```

### 3. Валидация телефонного номера

```json
{
  "type": "object",
  "properties": {
    "phone": {
      "type": "string",
      "pattern": "^\\+?[1-9]\\d{1,14}$"
    },
    "localPhone": {
      "type": "string",
      "pattern": "^\\(?\\d{3}\\)?[\\s-]?\\d{3}[\\s-]?\\d{4}$"
    }
  }
}
```

## Практические примеры валидации API данных

### 1. Валидация профиля пользователя

```javascript
// Схема валидации профиля пользователя
const userProfileSchema = {
  type: 'object',
  properties: {
    id: {
      type: 'string',
      pattern: '^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$', // UUID
      description: 'Уникальный идентификатор пользователя'
    },
    email: {
      type: 'string',
      format: 'email',
      pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'
    },
    username: {
      type: 'string',
      pattern: '^[a-zA-Z0-9_]{3,20}$', // 3-20 символов, только буквы, цифры и подчеркивание
      minLength: 3,
      maxLength: 20
    },
    phone: {
      type: 'string',
      pattern: '^\\+?[1-9][\\d\\s\\-\\(\\)]{7,15}$' // Международный формат телефона
    },
    website: {
      type: 'string',
      pattern: '^https?:\\/\\/(www\\.)?[-a-zA-Z0-9@:%._\\+~#=]{1,256}\\.[a-zA-Z0-9()]{1,6}\\b([-a-zA-Z0-9()@:%_\\+.~#?&//=]*)$'
    },
    postalCode: {
      type: 'string',
      pattern: '^[A-Za-z0-9\\s\\-]{3,10}$' // Почтовый индекс (универсальный формат)
    }
  },
  required: ['id', 'email', 'username'],
  additionalProperties: false
};

// Функция валидации с использованием Ajv (JSON Schema Validator)
const Ajv = require('ajv');
const ajv = new Ajv();

const validateUserProfile = ajv.compile(userProfileSchema);

// Пример использования
const userData = {
  id: '550e8400-e29b-41d4-a716-446655440000',
  email: 'user@example.com',
  username: 'john_doe',
  phone: '+1-555-123-4567',
  website: 'https://example.com',
  postalCode: '12345'
};

const valid = validateUserProfile(userData);
if (!valid) {
  console.log('Ошибки валидации:', validateUserProfile.errors);
} else {
  console.log('Данные пользователя валидны');
}
```

### 2. Валидация данных продукта

```javascript
const productSchema = {
  type: 'object',
  properties: {
    id: {
      type: 'string',
      pattern: '^PROD-[A-Z0-9]{8}$' // Артикул в формате PROD-XXXXXXXX
    },
    name: {
      type: 'string',
      minLength: 1,
      maxLength: 200
    },
    sku: {
      type: 'string',
      pattern: '^[A-Z0-9\\-_]{5,50}$' // Артикул (SKU) - 5-50 символов
    },
    price: {
      type: 'number',
      minimum: 0.01
    },
    currency: {
      type: 'string',
      pattern: '^[A-Z]{3}$' // Код валюты (например, USD, EUR)
    },
    imageUrl: {
      type: 'string',
      format: 'uri',
      pattern: '^https?:\\/\\/(www\\.)?[-a-zA-Z0-9@:%._\\+~#=]{1,256}\\.[a-zA-Z0-9()]{1,6}\\b([-a-zA-Z0-9()@:%_\\+.~#?&//=]*)\\.(jpg|jpeg|png|gif|webp)$'
    },
    tags: {
      type: 'array',
      items: {
        type: 'string',
        pattern: '^[a-zA-Z0-9_-]+$' // Теги - только буквы, цифры, дефисы и подчеркивания
      }
    },
    description: {
      type: 'string',
      maxLength: 10000
    }
  },
  required: ['id', 'name', 'sku', 'price', 'currency'],
  additionalProperties: false
};

const validateProduct = ajv.compile(productSchema);
```

### 3. Валидация данных заказа

```javascript
const orderSchema = {
  type: 'object',
  properties: {
    orderId: {
      type: 'string',
      pattern: '^ORD-[A-Z0-9]{10}$' // Номер заказа в формате ORD-XXXXXXXXXX
    },
    orderDate: {
      type: 'string',
      format: 'date-time',
      pattern: '^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}(\\.\\d{3})?Z?$' // ISO 8601
    },
    paymentId: {
      type: 'string',
      pattern: '^PAY-[A-Z0-9]{12}$' // ID платежа
    },
    cardNumber: {
      type: 'string',
      pattern: '^\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}$' // Номер карты
    },
    cardCvv: {
      type: 'string',
      pattern: '^\\d{3,4}$' // CVV код
    },
    cardExpiry: {
      type: 'string',
      pattern: '^(0[1-9]|1[0-2])\\/\\d{2}$' // Срок действия (MM/YY)
    },
    billingAddress: {
      type: '$ref',
      $ref: '#/definitions/address'
    },
    shippingAddress: {
      type: '$ref',
      $ref: '#/definitions/address'
    }
  },
  required: ['orderId', 'orderDate', 'paymentId'],
  definitions: {
    address: {
      type: 'object',
      properties: {
        street: { type: 'string' },
        city: { type: 'string', pattern: '^[A-Za-z\\s\\-\'\.]{2,50}$' },
        state: { type: 'string', pattern: '^[A-Z]{2}$' }, // Аббревиатура штата
        postalCode: {
          type: 'string',
          pattern: '^\\d{5}(-\\d{4})?$|^([A-Z]\\d[A-Z])\\s*(\\d[A-Z]\\d)$' // US ZIP или Canadian postal code
        },
        country: { type: 'string', pattern: '^[A-Z]{2}$' } // Код страны
      },
      required: ['street', 'city', 'postalCode', 'country']
    }
  }
};

const validateOrder = ajv.compile(orderSchema);
```

## Проверка данных API с использованием регулярных выражений

### 1. Middleware для Express.js

```javascript
const express = require('express');
const app = express();

// Регулярные выражения для валидации
const patterns = {
  email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
  phone: /^\+?[1-9]\d{1,14}$/,
  uuid: /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i,
  url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
  username: /^[a-zA-Z0-9_]{3,20}$/
};

// Мидлвар для валидации входящих данных
const validateInput = (req, res, next) => {
  const errors = [];
  
  // Валидация email в теле запроса
  if (req.body.email && !patterns.email.test(req.body.email)) {
    errors.push('Некорректный формат email');
  }
  
  // Валидация телефона
  if (req.body.phone && !patterns.phone.test(req.body.phone)) {
    errors.push('Некорректный формат телефона');
  }
  
  // Валидация URL
  if (req.body.website && !patterns.url.test(req.body.website)) {
    errors.push('Некорректный формат URL');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors
    });
  }
  
  next();
};

app.use(express.json());
app.use(validateInput);

app.post('/api/users', (req, res) => {
  // Обработка запроса после валидации
  res.json({ message: 'User created successfully' });
});
```

### 2. Валидация параметров запроса

```javascript
// Валидация параметров запроса
const validateQueryParams = (req, res, next) => {
  const { email, userId, page, limit } = req.query;
  const errors = [];

  if (email && !/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(email)) {
    errors.push('Некорректный email в параметрах запроса');
  }

  if (userId && !/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i.test(userId)) {
    errors.push('Некорректный формат ID пользователя');
  }

  if (page && !/^\d+$/.test(page)) {
    errors.push('Параметр page должен быть числом');
  }

  if (limit && !/^(10|20|50|100)$/.test(limit)) {
    errors.push('Параметр limit может быть только 10, 20, 50 или 100');
  }

  if (errors.length > 0) {
    return res.status(400).json({
      error: 'Query validation failed',
      details: errors
    });
  }

  next();
};

app.get('/api/users', validateQueryParams, (req, res) => {
  // Обработка запроса
  res.json({ message: 'Users retrieved successfully' });
});
```

### 3. Валидация заголовков запроса

```javascript
// Валидация специфических заголовков
const validateHeaders = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  const userAgent = req.headers['user-agent'];
  const contentType = req.headers['content-type'];
  
  const errors = [];
  
  // Валидация API ключа (обычно UUID или специфический формат)
  if (apiKey && !/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i.test(apiKey)) {
    errors.push('Некорректный формат API ключа');
  }
  
  // Валидация user-agent (проверка на подозрительные паттерны)
  if (userAgent && /sql|union|select|drop|insert|update|delete|exec|execute|script/i.test(userAgent)) {
    errors.push('Подозрительный User-Agent заголовок');
  }
  
  // Валидация Content-Type
  if (contentType && !/^application\/(json|xml)|text\/plain/.test(contentType)) {
    errors.push('Неподдерживаемый тип контента');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      error: 'Header validation failed',
      details: errors
    });
  }
  
  next();
};
```

## Валидация в библиотеках

### 1. Валидация с использованием Joi

```javascript
const Joi = require('joi');

// Схема с регулярными выражениями
const userSchema = Joi.object({
  email: Joi.string()
    .email()
    .pattern(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/)
    .required(),
    
  username: Joi.string()
    .pattern(/^[a-zA-Z0-9_]{3,20}$/)
    .required(),
    
  phone: Joi.string()
    .pattern(/^\+?[1-9]\d{1,14}$/)
    .optional(),
    
  website: Joi.string()
    .uri()
    .pattern(/^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/)
    .optional(),
    
  postalCode: Joi.string()
    .pattern(/^[A-Za-z0-9\s\-]{3,10}$/)
    .optional()
});

// Валидация данных
const validateUser = (userData) => {
  const { error, value } = userSchema.validate(userData);
  
  if (error) {
    return {
      isValid: false,
      errors: error.details.map(detail => detail.message)
    };
  }
  
  return {
    isValid: true,
    data: value
  };
};
```

### 2. Валидация с использованием Yup

```javascript
import * as Yup from 'yup';

// Регулярные выражения
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
const phoneRegex = /^\+?[1-9]\d{1,14}$/;
const usernameRegex = /^[a-zA-Z0-9_]{3,20}$/;

const userSchema = Yup.object({
  email: Yup.string()
    .matches(emailRegex, 'Некорректный email')
    .required('Email обязателен'),
    
  username: Yup.string()
    .matches(usernameRegex, 'Некорректный формат username')
    .required('Username обязателен'),
    
  phone: Yup.string()
    .matches(phoneRegex, 'Некорректный формат телефона')
    .nullable(),
    
  website: Yup.string()
    .matches(
      /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
      'Некорректный URL'
    )
    .nullable()
});

// Валидация
const validateUser = async (userData) => {
  try {
    const validatedData = await userSchema.validate(userData, { abortEarly: false });
    return { isValid: true, data: validatedData };
  } catch (error) {
    return {
      isValid: false,
      errors: error.inner.map(err => ({
        field: err.path,
        message: err.message
      }))
    };
  }
};
```

## Валидация в TypeScript

### 1. Строгая валидация с типами

```typescript
interface User {
  id: string;
  email: string;
  username: string;
  phone?: string;
  website?: string;
}

// Регулярные выражения для валидации
const patterns = {
  email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
  uuid: /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i,
  username: /^[a-zA-Z0-9_]{3,20}$/,
  phone: /^\+?[1-9]\d{1,14}$/,
  url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/
};

// Функция валидации с типами
const validateUser = (data: Partial<User>): data is User => {
  if (!data.id || !patterns.uuid.test(data.id)) {
    console.error('Invalid user ID format');
    return false;
  }
  
  if (!data.email || !patterns.email.test(data.email)) {
    console.error('Invalid email format');
    return false;
  }
  
  if (!data.username || !patterns.username.test(data.username)) {
    console.error('Invalid username format');
    return false;
  }
  
  if (data.phone && !patterns.phone.test(data.phone)) {
    console.error('Invalid phone format');
    return false;
  }
  
  if (data.website && !patterns.url.test(data.website)) {
    console.error('Invalid website format');
    return false;
  }
  
  return true;
};

// Использование
const userData: Partial<User> = {
  id: '550e8400-e29b-41d4-a716-446655440000',
  email: 'user@example.com',
  username: 'john_doe',
  phone: '+1-555-123-4567',
  website: 'https://example.com'
};

if (validateUser(userData)) {
  console.log('User data is valid:', userData);
} else {
  console.log('User data validation failed');
}
```

### 2. Валидация с использованием Zod

```typescript
import { z } from 'zod';

// Регулярные выражения
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
const phoneRegex = /^\+?[1-9]\d{1,14}$/;
const usernameRegex = /^[a-zA-Z0-9_]{3,20}$/;

// Схема валидации с Zod
const UserSchema = z.object({
  id: z.string().regex(/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i, 'Invalid UUID format'),
  email: z.string().email().regex(emailRegex, 'Invalid email format'),
  username: z.string().regex(usernameRegex, 'Invalid username format').min(3).max(20),
  phone: z.string().regex(phoneRegex, 'Invalid phone format').optional(),
  website: z.string().url().optional(),
  postalCode: z.string().regex(/^[A-Za-z0-9\s\-]{3,10}$/, 'Invalid postal code').optional()
});

// Валидация данных
const validateUserWithZod = (data: unknown) => {
  try {
    const validatedUser = UserSchema.parse(data);
    return { success: true, data: validatedUser };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return {
        success: false,
        errors: error.errors.map(err => ({
          field: err.path.join('.'),
          message: err.message
        }))
      };
    }
    throw error;
  }
};

// Использование
const result = validateUserWithZod({
  id: '550e8400-e29b-41d4-a716-446655440000',
  email: 'user@example.com',
  username: 'john_doe'
});

if (result.success) {
  console.log('Valid user:', result.data);
} else {
  console.log('Validation errors:', result.errors);
}
```

## Практические рекомендации

### 1. Производительность
- Кэшируйте скомпилированные регулярные выражения
- Используйте более специфичные паттерны для лучшей производительности
- Избегайте чрезмерно сложных регулярных выражений

### 2. Безопасность
- Валидируйте входные данные перед обработкой
- Используйте белые списки (whitelist) вместо черных списков (blacklist)
- Экранируйте специальные символы при построении регулярных выражений динамически

### 3. Поддерживаемость
- Используйте именованные константы для сложных регулярных выражений
- Документируйте сложные паттерны
- Рассмотрите использование библиотек для сложной валидации

## Заключение

Регулярные выражения в валидации JSON-схем API обеспечивают мощный инструмент для проверки формата строковых данных. Они особенно полезны для проверки специфических форматов, таких как email, URL, телефонные номера и пользовательские идентификаторы.

Ключевые моменты:
- Используйте регулярные выражения для проверки формата строк в JSON-схемах
- Комбинируйте с другими методами валидации для полной проверки данных
- Рассмотрите использование специализированных библиотек для сложной валидации
- Обеспечьте безопасность и производительность при валидации входных данных