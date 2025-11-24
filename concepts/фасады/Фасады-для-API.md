---
aliases: [Фасады для API, API Facades, API Wrapper]
tags: [programming, design-patterns, facade, api, architecture, frontend]
---

# Фасады для API

Паттерн **Фасад** особенно полезен при работе с внешними API, где он служит в качестве слоя абстракции, упрощающего взаимодействие с внешними сервисами. Фасады для API скрывают сложность выполнения HTTP-запросов, обработки ответов, управления аутентификацией и обработки ошибок.

## Общее описание

Фасады для API создают абстрактный слой между приложением и внешними сервисами. Они инкапсулируют логику выполнения запросов, преобразования данных и обработки ошибок, предоставляя простой интерфейс для взаимодействия с API.

## Преимущества использования фасадов для API

### 1. Упрощение взаимодействия

Фасады скрывают сложность выполнения HTTP-запросов, работы с Promise/async-await и преобразования данных:

```javascript
// Без фасада - сложный код
const response = await fetch('/api/users', {
  method: 'GET',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  }
});
const data = await response.json();
if (!response.ok) {
  throw new Error(data.message || 'Ошибка запроса');
}

// С фасадом - простой код
const users = await apiClient.getUsers();
```

### 2. Централизация логики

Все операции с API сосредоточены в одном месте, что упрощает поддержку и обновление:

```javascript
class APIClient {
  constructor(baseURL, token) {
    this.baseURL = baseURL;
    this.token = token;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`,
        ...options.headers
      },
      ...options
    };
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || `HTTP ${response.status}`);
    }
    
    return response.json();
  }
  
  async getUsers() {
    return this.request('/users');
  }
  
  async getUser(id) {
    return this.request(`/users/${id}`);
  }
  
  async createUser(userData) {
    return this.request('/users', {
      method: 'POST',
      body: JSON.stringify(userData)
    });
  }
}
```

### 3. Обработка ошибок

Фасады позволяют централизованно обрабатывать ошибки API:

```javascript
class APIClient {
  async request(endpoint, options = {}) {
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.token}`,
          ...options.headers
        },
        ...options
      });
      
      const data = await response.json();
      
      if (!response.ok) {
        throw new APIError(data.message || 'Неизвестная ошибка', response.status);
      }
      
      return data;
    } catch (error) {
      if (error instanceof APIError) {
        // Обработка специфичных ошибок API
        this.handleError(error);
      } else {
        // Обработка сетевых ошибок
        console.error('Сетевая ошибка:', error);
      }
      throw error;
    }
  }
  
  handleError(error) {
    switch (error.status) {
      case 401:
        // Перенаправление на страницу авторизации
        window.location.href = '/login';
        break;
      case 403:
        // Отображение сообщения о недостатке прав
        showNotification('Недостаточно прав для выполнения операции');
        break;
      case 429:
        // Обработка ограничения по частоте запросов
        showNotification('Слишком много запросов, попробуйте позже');
        break;
      default:
        showNotification('Произошла ошибка, попробуйте позже');
    }
  }
}
```

## Примеры реализации фасадов для API

### 1. Базовый фасад для REST API

```javascript
class RESTAPIFacade {
  constructor(config) {
    this.baseURL = config.baseURL;
    this.defaultHeaders = config.headers || {};
    this.timeout = config.timeout || 10000;
  }
  
  async request(endpoint, options = {}) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);
    
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        headers: {
          ...this.defaultHeaders,
          ...options.headers
        },
        signal: controller.signal,
        ...options
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      clearTimeout(timeoutId);
      
      if (error.name === 'AbortError') {
        throw new Error('Запрос превысил время ожидания');
      }
      
      throw error;
    }
  }
  
  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url, { method: 'GET' });
  }
  
  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' }
    });
  }
  
  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' }
    });
  }
  
  delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

// Использование
const api = new RESTAPIFacade({
  baseURL: 'https://api.example.com',
  headers: {
    'Authorization': 'Bearer your-token'
  }
});

const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'John', email: 'john@example.com' });
```

### 2. Фасад для GraphQL API

```javascript
class GraphQLAPIFacade {
  constructor(endpoint, token) {
    this.endpoint = endpoint;
    this.token = token;
  }
  
  async query(query, variables = {}) {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      },
      body: JSON.stringify({
        query,
        variables
      })
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    const result = await response.json();
    
    if (result.errors) {
      throw new Error(result.errors[0].message);
    }
    
    return result.data;
  }
  
  // Методы для конкретных операций
  async getUsers() {
    const query = `
      query GetUsers {
        users {
          id
          name
          email
        }
      }
    `;
    return this.query(query);
  }
  
  async createUser(userData) {
    const mutation = `
      mutation CreateUser($input: UserInput!) {
        createUser(input: $input) {
          id
          name
          email
        }
      }
    `;
    return this.query(mutation, { input: userData });
  }
}
```

### 3. Фасад с кэшированием

```javascript
class CachedAPIFacade {
  constructor(apiClient) {
    this.api = apiClient;
    this.cache = new Map();
    this.cacheTimeout = 5 * 60 * 1000; // 5 минут
  }
  
  async getWithCache(endpoint, cacheKey, options = {}) {
    const now = Date.now();
    
    // Проверка кэша
    if (this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey);
      if (now - cached.timestamp < this.cacheTimeout) {
        console.log('Данные взяты из кэша');
        return cached.data;
      } else {
        // Удаление устаревшего кэша
        this.cache.delete(cacheKey);
      }
    }
    
    // Загрузка новых данных
    const data = await this.api.request(endpoint, options);
    
    // Сохранение в кэш
    this.cache.set(cacheKey, {
      data,
      timestamp: now
    });
    
    return data;
  }
  
  async getUsers() {
    return this.getWithCache('/users', 'users');
  }
  
  async getUser(id) {
    return this.getWithCache(`/users/${id}`, `user_${id}`);
  }
  
  // Очистка кэша при изменении данных
  async updateUser(id, userData) {
    const result = await this.api.put(`/users/${id}`, userData);
    // Удаление устаревших данных из кэша
    this.cache.delete(`user_${id}`);
    this.cache.delete('users');
    return result;
  }
}
```

## Лучшие практики при создании фасадов для API

### 1. Обработка аутентификации

```javascript
class AuthenticatedAPIFacade {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.token = null;
  }
  
  setToken(token) {
    this.token = token;
  }
  
  async ensureToken() {
    if (!this.token) {
      // Получение токена, возможно через refresh
      this.token = await this.refreshToken();
    }
  }
  
  async request(endpoint, options = {}) {
    await this.ensureToken();
    
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      headers: {
        'Authorization': `Bearer ${this.token}`,
        ...options.headers
      },
      ...options
    });
    
    if (response.status === 401) {
      // Попытка обновить токен и повторить запрос
      this.token = null;
      await this.ensureToken();
      
      const retryResponse = await fetch(`${this.baseURL}${endpoint}`, {
        headers: {
          'Authorization': `Bearer ${this.token}`,
          ...options.headers
        },
        ...options
      });
      
      if (!retryResponse.ok) {
        throw new Error('Аутентификация не удалась');
      }
      
      return retryResponse.json();
    }
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return response.json();
  }
}
```

### 2. Логирование и мониторинг

```javascript
class MonitoredAPIFacade {
  constructor(apiClient) {
    this.api = apiClient;
  }
  
  async request(endpoint, options = {}) {
    const startTime = Date.now();
    
    try {
      console.log(`Запрос: ${options.method || 'GET'} ${endpoint}`);
      const result = await this.api.request(endpoint, options);
      const duration = Date.now() - startTime;
      
      console.log(`Успешный запрос: ${endpoint}, время: ${duration}ms`);
      this.logMetrics(endpoint, duration, 'success');
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      console.error(`Ошибка запроса: ${endpoint}, время: ${duration}ms, ошибка:`, error);
      this.logMetrics(endpoint, duration, 'error');
      
      throw error;
    }
  }
  
  logMetrics(endpoint, duration, status) {
    // Отправка метрик в систему мониторинга
    if (window.gtag) {
      window.gtag('event', 'api_request', {
        custom_parameter_endpoint: endpoint,
        custom_parameter_duration: duration,
        custom_parameter_status: status
      });
    }
  }
}
```

### 3. Обработка разных типов ответов

```javascript
class FlexibleAPIFacade {
  async request(endpoint, options = {}) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      headers: {
        'Authorization': `Bearer ${this.token}`,
        ...options.headers
      },
      ...options
    });
    
    if (!response.ok) {
      throw new APIError(await response.text(), response.status);
    }
    
    // Определение типа содержимого
    const contentType = response.headers.get('content-type');
    
    if (contentType && contentType.includes('application/json')) {
      return response.json();
    } else if (contentType && contentType.includes('text/')) {
      return response.text();
    } else {
      return response.blob();
    }
  }
}
```

## Интеграция с фронтенд-фреймворками

### С React

```jsx
// Хук для использования API фасада
function useAPI() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const apiCall = useCallback(async (apiMethod, ...args) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiMethod(...args);
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);
  
  return { loading, error, apiCall };
}

// Компонент, использующий API фасад
function UserList() {
  const [users, setUsers] = useState([]);
  const { loading, error, apiCall } = useAPI();
  
  useEffect(() => {
    apiCall(apiClient.getUsers)
      .then(setUsers)
      .catch(console.error);
  }, []);
  
  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Заключение

Фасады для API являются важным инструментом в арсенале фронтенд-разработчика. Они позволяют скрыть сложность взаимодействия с внешними сервисами и предоставляют простой, последовательный интерфейс для работы с данными. При правильной реализации фасады API улучшают качество кода, упрощают отладку и облегчают сопровождение приложений.

## Связанные темы

- [[Паттерн-фасад]]
- [[Фасады-в-библиотеках]]
- [[Фасады-в-React]]
- [[HTTP-запросы]]
- [[Асинхронное-программирование]]