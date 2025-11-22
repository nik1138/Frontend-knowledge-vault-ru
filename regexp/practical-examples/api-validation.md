---
aliases: ["Регулярные выражения в API валидации", "JSON Schema и регулярки", "Валидация API"]
tags: ["#regexp", "#api", "#json-schema", "#validation", "#backend", "#frontend"]
---

# Регулярные выражения для работы с API (валидация JSON-схем)

## Введение

При работе с API регулярные выражения играют важную роль в валидации данных как на клиентской, так и на серверной стороне. JSON Schema позволяет интегрировать регулярные выражения в схемы данных для проверки формата полей, что особенно полезно при валидации строковых значений.

## JSON Schema и регулярные выражения

### 1. Использование свойства `pattern`

В JSON Schema регулярные выражения добавляются с помощью свойства `pattern`, которое проверяет соответствие строки заданному регулярному выражению:

```json
{
  "type": "object",
  "properties": {
    "email": {
      "type": "string",
      "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    "phone": {
      "type": "string",
      "pattern": "^\\+?[1-9]\\d{1,14}$"
    },
    "username": {
      "type": "string",
      "pattern": "^[a-zA-Z0-9_]{3,20}$"
    }
  },
  "required": ["email", "username"]
}
```

### 2. Пример валидации на клиентской стороне

```javascript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';

const ajv = new Ajv();
addFormats(ajv);

const userSchema = {
  type: "object",
  properties: {
    email: {
      type: "string",
      format: "email",
      pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    phone: {
      type: "string",
      pattern: "^\\+?[1-9]\\d{1,14}$"
    },
    password: {
      type: "string",
      pattern: "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$"
    }
  },
  required: ["email", "password"]
};

const validate = ajv.compile(userSchema);

const userData = {
  email: "user@example.com",
  phone: "+79991234567",
  password: "MySecurePassword123!"
};

const valid = validate(userData);
if (!valid) {
  console.log('Ошибки валидации:', validate.errors);
} else {
  console.log('Данные валидны');
}
```

### 3. Валидация на серверной стороне (Node.js)

```javascript
const express = require('express');
const Ajv = require('ajv');
const addFormats = require('ajv-formats');

const app = express();
app.use(express.json());

const ajv = new Ajv();
addFormats(ajv);

const registrationSchema = {
  type: "object",
  properties: {
    email: {
      type: "string",
      format: "email",
      pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    password: {
      type: "string",
      pattern: "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$"
    },
    phone: {
      type: "string",
      pattern: "^\\+?[1-9]\\d{1,14}$"
    }
  },
  required: ["email", "password"]
};

const validate = ajv.compile(registrationSchema);

app.post('/register', (req, res) => {
  const valid = validate(req.body);
  
  if (!valid) {
    return res.status(400).json({
      error: 'Некорректные данные',
      details: validate.errors
    });
  }
  
  // Обработка корректных данных
  console.log('Регистрация пользователя:', req.body);
  res.status(201).json({ message: 'Пользователь успешно зарегистрирован' });
});
```

## Часто используемые паттерны в API валидации

### 1. Email адреса

```json
{
  "type": "string",
  "format": "email",
  "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
}
```

### 2. Номера телефонов

```json
{
  "type": "string",
  "pattern": "^\\+?[1-9]\\d{1,14}$"
}
```

Для российских номеров:

```json
{
  "type": "string",
  "pattern": "^\\+?7[0-9]{10}$"
}
```

### 3. Пароли

```json
{
  "type": "string",
  "pattern": "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$"
}
```

### 4. URL адреса

```json
{
  "type": "string",
  "format": "uri",
  "pattern": "^https?://[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}(/.*)?$"
}
```

### 5. Идентификаторы

```json
{
  "type": "string",
  "pattern": "^[a-zA-Z0-9_-]+$"
}
```

### 6. Коды и токены

```json
{
  "type": "string",
  "pattern": "^[A-Za-z0-9-_]{20,}$"
}
```

## Практические примеры

### 1. Сложная схема профиля пользователя

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "User Profile",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^[a-f0-9]{24}$"
    },
    "username": {
      "type": "string",
      "pattern": "^[a-zA-Z0-9_]{3,20}$"
    },
    "email": {
      "type": "string",
      "format": "email",
      "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    "phone": {
      "type": "string",
      "pattern": "^\\+?7[0-9]{10}$"
    },
    "website": {
      "type": "string",
      "format": "uri",
      "pattern": "^https?://[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}(/.*)?$"
    },
    "socialMedia": {
      "type": "object",
      "properties": {
        "twitter": {
          "type": "string",
          "pattern": "^@?[a-zA-Z0-9_]{1,15}$"
        },
        "linkedin": {
          "type": "string",
          "format": "uri"
        }
      }
    },
    "bio": {
      "type": "string",
      "maxLength": 500
    }
  },
  "required": ["id", "username", "email"],
  "additionalProperties": false
}
```

### 2. Валидация данных продукта в интернет-магазине

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Product",
  "properties": {
    "id": {
      "type": "string",
      "pattern": "^[A-Z0-9]{8,12}$"
    },
    "name": {
      "type": "string",
      "minLength": 3,
      "maxLength": 100
    },
    "sku": {
      "type": "string",
      "pattern": "^[A-Z0-9-]+$"
    },
    "price": {
      "type": "number",
      "minimum": 0
    },
    "imageUrl": {
      "type": "string",
      "format": "uri",
      "pattern": "^https://.*\\.(jpg|jpeg|png|gif|webp)$"
    },
    "category": {
      "type": "string",
      "enum": ["electronics", "clothing", "books", "home", "sports"]
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^[a-zA-Z0-9_-]+$"
      }
    }
  },
  "required": ["id", "name", "price", "category"],
  "additionalProperties": false
}
```

### 3. Валидация банковских данных

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Banking Details",
  "properties": {
    "cardNumber": {
      "type": "string",
      "pattern": "^\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}$"
    },
    "cvv": {
      "type": "string",
      "pattern": "^\\d{3,4}$"
    },
    "expiryDate": {
      "type": "string",
      "pattern": "^(0[1-9]|1[0-2])/\\d{2}$"
    },
    "iban": {
      "type": "string",
      "pattern": "^[A-Z]{2}\\d{2}[A-Z0-9]{4}\\d{4}\\d{4}\\d{4}$"
    },
    "bic": {
      "type": "string",
      "pattern": "^[A-Z]{6}[A-Z0-9]{2}([A-Z0-9]{3})?$"
    }
  },
  "required": ["cardNumber", "cvv", "expiryDate"]
}
```

## Валидация на фронтенде с использованием регулярных выражений

### 1. Интеграция с формами в React

```jsx
import React, { useState } from 'react';
import Ajv from 'ajv';

const ajv = new Ajv();

const userSchema = {
  type: "object",
  properties: {
    email: {
      type: "string",
      format: "email",
      pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    phone: {
      type: "string",
      pattern: "^\\+?7[0-9]{10}$"
    }
  },
  required: ["email"]
};

const validate = ajv.compile(userSchema);

const UserForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    phone: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
    
    // Валидация в реальном времени
    const tempData = { ...formData, [name]: value };
    const valid = validate(tempData);
    
    if (!valid) {
      const fieldErrors = validate.errors
        .filter(error => error.instancePath.includes(name))
        .map(error => error.message);
      
      setErrors({
        ...errors,
        [name]: fieldErrors[0] || ''
      });
    } else {
      setErrors({
        ...errors,
        [name]: ''
      });
    }
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const valid = validate(formData);
    
    if (valid) {
      console.log('Отправка данных:', formData);
      // Отправка данных на сервер
    } else {
      const newErrors = {};
      validate.errors.forEach(error => {
        const field = error.instancePath.replace('/', '');
        newErrors[field] = error.message;
      });
      setErrors(newErrors);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="tel"
          name="phone"
          value={formData.phone}
          onChange={handleChange}
          placeholder="Телефон"
        />
        {errors.phone && <span className="error">{errors.phone}</span>}
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};

export default UserForm;
```

### 2. Валидация в Vue.js

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <input
        v-model="formData.email"
        @input="validateField('email')"
        placeholder="Email"
      />
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <input
        v-model="formData.phone"
        @input="validateField('phone')"
        placeholder="Телефон"
      />
      <span v-if="errors.phone" class="error">{{ errors.phone }}</span>
    </div>
    
    <button type="submit" :disabled="!isFormValid">Отправить</button>
  </form>
</template>

<script>
import Ajv from 'ajv';

const ajv = new Ajv();

const userSchema = {
  type: "object",
  properties: {
    email: {
      type: "string",
      format: "email",
      pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    },
    phone: {
      type: "string",
      pattern: "^\\+?7[0-9]{10}$"
    }
  },
  required: ["email"]
};

const validate = ajv.compile(userSchema);

export default {
  data() {
    return {
      formData: {
        email: '',
        phone: ''
      },
      errors: {}
    };
  },
  computed: {
    isFormValid() {
      return validate(this.formData) && Object.keys(this.errors).length === 0;
    }
  },
  methods: {
    validateField(fieldName) {
      const tempData = { ...this.formData };
      const valid = validate(tempData);
      
      if (!valid) {
        const fieldErrors = validate.errors
          .filter(error => error.instancePath.includes(fieldName))
          .map(error => error.message);
        
        this.$set(this.errors, fieldName, fieldErrors[0] || '');
      } else {
        this.$delete(this.errors, fieldName);
      }
    },
    
    submitForm() {
      const valid = validate(this.formData);
      
      if (valid) {
        console.log('Отправка данных:', this.formData);
        // Отправка данных на сервер
      } else {
        const newErrors = {};
        validate.errors.forEach(error => {
          const field = error.instancePath.replace('/', '');
          newErrors[field] = error.message;
        });
        this.errors = newErrors;
      }
    }
  }
};
</script>
```

## Лучшие практики

### 1. Комбинирование с другими методами валидации

```javascript
// Комбинирование JSON Schema с дополнительной валидацией
const validateUser = (userData) => {
  // Валидация схемы
  const schemaValid = validate(userData);
  if (!schemaValid) {
    return { valid: false, errors: validate.errors };
  }
  
  // Дополнительная валидация
  if (userData.password && userData.password.length < 12) {
    return { 
      valid: false, 
      errors: [{ message: 'Пароль должен быть не менее 12 символов' }] 
    };
  }
  
  return { valid: true };
};
```

### 2. Кэширование скомпилированных схем

```javascript
// Кэширование схем для повышения производительности
const compiledSchemas = new Map();

const getValidator = (schema) => {
  const schemaKey = JSON.stringify(schema);
  
  if (!compiledSchemas.has(schemaKey)) {
    const validator = ajv.compile(schema);
    compiledSchemas.set(schemaKey, validator);
  }
  
  return compiledSchemas.get(schemaKey);
};
```

### 3. Использование готовых библиотек форматов

```javascript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';

const ajv = new Ajv();
addFormats(ajv); // Добавляет стандартные форматы валидации

// Теперь можно использовать стандартные форматы
const schema = {
  type: "object",
  properties: {
    email: { type: "string", format: "email" },
    date: { type: "string", format: "date" },
    uri: { type: "string", format: "uri" }
  }
};
```

## Заключение

Регулярные выражения в JSON Schema обеспечивают мощный инструмент для валидации формата данных в API. Их использование позволяет задавать строгие правила для строковых значений, что улучшает надежность приложения и предотвращает ошибки, связанные с некорректным форматом данных.

При правильном применении регулярных выражений в схемах API можно достичь высокой степени валидации данных как на клиентской, так и на серверной стороне, обеспечивая целостность и безопасность приложения.