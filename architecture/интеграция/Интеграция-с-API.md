---
aliases: ["API Integration", "Интеграция с внешними API", "Работа с API"]
tags: [architecture, frontend, api, integration, http]
---

# Интеграция с API

## Обзор

Интеграция с API (Application Programming Interface) является одной из ключевых задач при разработке современных фронтенд-приложений. Это процесс взаимодействия клиентского приложения с внешними или внутренними сервисами через HTTP-протокол, позволяющий получать, отправлять и изменять данные.

## Основные концепции

Интеграция с API позволяет фронтенд-приложениям:
- Получать данные из внешних источников
- Отправлять пользовательские данные на серверы
- Аутентифицировать пользователей
- Синхронизировать состояние приложения с сервером
- Интегрироваться с различными сервисами (платежные системы, социальные сети, аналитика)

## Виды API

### REST API
Наиболее распространенный тип API, использующий HTTP-методы (GET, POST, PUT, DELETE) для выполнения операций CRUD.

### GraphQL API
Позволяет клиенту запрашивать только необходимые данные, что уменьшает объем передаваемой информации и улучшает производительность.

### SOAP API
Более старый протокол, часто используемый в корпоративных средах, основанный на XML.

## Практические рекомендации

### 1. Создание API-слоя

Создайте отдельный слой для работы с API, чтобы изолировать логику взаимодействия с сервером:

```javascript
// api/apiClient.js
class ApiClient {
  constructor(baseURL, defaultHeaders = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...defaultHeaders
    };
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: { ...this.defaultHeaders, ...options.headers },
      ...options
    };

    try {
      const response = await fetch(url, config);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('API request failed:', error);
      throw error;
    }
  }

  get(endpoint, headers = {}) {
    return this.request(endpoint, { method: 'GET', headers });
  }

  post(endpoint, data, headers = {}) {
    return this.request(endpoint, {
      method: 'POST',
      headers,
      body: JSON.stringify(data)
    });
  }

  put(endpoint, data, headers = {}) {
    return this.request(endpoint, {
      method: 'PUT',
      headers,
      body: JSON.stringify(data)
    });
  }

  delete(endpoint, headers = {}) {
    return this.request(endpoint, { method: 'DELETE', headers });
  }
}

export default ApiClient;
```

### 2. Обработка аутентификации

```javascript
// api/authenticatedApiClient.js
import ApiClient from './apiClient';

class AuthenticatedApiClient extends ApiClient {
  async request(endpoint, options = {}) {
    // Добавляем токен аутентификации
    const token = localStorage.getItem('authToken');
    
    const headers = {
      ...this.defaultHeaders,
      ...options.headers
    };

    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }

    return super.request(endpoint, { ...options, headers });
  }
}

export default AuthenticatedApiClient;
```

### 3. Управление ошибками

```javascript
// utils/errorHandler.js
export class ApiError extends Error {
  constructor(message, status, response) {
    super(message);
    this.status = status;
    this.response = response;
    this.name = 'ApiError';
  }
}

export const handleApiError = (error) => {
  if (error instanceof ApiError) {
    switch (error.status) {
      case 401:
        // Перенаправление на страницу входа
        window.location.href = '/login';
        break;
      case 403:
        console.error('Доступ запрещен');
        break;
      case 404:
        console.error('Ресурс не найден');
        break;
      case 500:
        console.error('Внутренняя ошибка сервера');
        break;
      default:
        console.error(`Ошибка API: ${error.message}`);
    }
  } else {
    console.error('Неизвестная ошибка:', error);
  }
};
```

### 4. Кэширование данных

```javascript
// utils/cache.js
class ApiCache {
  constructor(ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
    this.cache = new Map();
    this.ttl = ttl;
  }

  get(key) {
    const cached = this.cache.get(key);
    if (!cached) return null;

    if (Date.now() - cached.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return cached.data;
  }

  set(key, data) {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }

  clear() {
    this.cache.clear();
  }
}

export default ApiCache;
```

### 5. Использование в React-компонентах

```jsx
// components/UserProfile.jsx
import React, { useState, useEffect } from 'react';
import ApiClient from '../api/apiClient';
import { handleApiError } from '../utils/errorHandler';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const apiClient = new ApiClient('https://api.example.com');
        const userData = await apiClient.get(`/users/${userId}`);
        setUser(userData);
      } catch (err) {
        setError(err);
        handleApiError(err);
      } finally {
        setLoading(false);
      }
    };

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;

  return (
    <div>
      <h2>{user?.name}</h2>
      <p>{user?.email}</p>
    </div>
  );
};

export default UserProfile;
```

## Лучшие практики

1. **Используйте типизацию** (TypeScript) для API-ответов
2. **Валидируйте данные** перед отправкой на сервер
3. **Реализуйте повторные попытки** запросов при временных ошибках
4. **Логируйте ошибки** для последующего анализа
5. **Используйте CORS-совместимые заголовки** при необходимости
6. **Реализуйте отмену запросов** при размонтировании компонентов

## Безопасность

- Храните API-ключи и токены в безопасном месте
- Не отправляйте чувствительные данные в URL
- Используйте HTTPS для всех API-запросов
- Реализуйте защиту от CSRF-атак

## Связанные темы

- [[Интеграция с микросервисами]]
- [[Интеграция со сторонними библиотеками]]
- [[Тестирование интеграции]]
- [[Архитектура фронтенд-приложений]]

#frontend #api #integration #architecture