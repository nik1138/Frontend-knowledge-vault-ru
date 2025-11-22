---
aliases: ["Асинхронные операции", "Обработка ошибок", "Async Operations"]
tags: [js, async, error-handling, practical-examples]
---

# Асинхронные операции и обработка ошибок

## Введение в асинхронность

Асинхронные операции позволяют выполнять задачи, которые занимают продолжительное время (например, сетевые запросы), не блокируя выполнение основного кода.

## Promise и его применение

### Основы Promise

```javascript
// Создание Promise
function delay(ms) {
  return new Promise((resolve, reject) => {
    if (ms < 0) {
      reject(new Error('Время задержки не может быть отрицательным'));
      return;
    }
    
    setTimeout(() => {
      resolve(`Задержка ${ms}мс завершена`);
    }, ms);
  });
}

// Использование Promise
delay(1000)
  .then(result => {
    console.log(result); // "Задержка 1000мс завершена"
    return delay(500);
  })
  .then(result => {
    console.log(result); // "Задержка 500мс завершена"
  })
  .catch(error => {
    console.error('Ошибка:', error.message);
  });

// Цепочка Promise
function fetchUserData(userId) {
  return fetch(`/api/users/${userId}`)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP ошибка! статус: ${response.status}`);
      }
      return response.json();
    })
    .then(userData => {
      console.log('Данные пользователя:', userData);
      return userData;
    })
    .catch(error => {
      console.error('Ошибка получения данных пользователя:', error);
      throw error; // Передаем ошибку дальше
    });
}
```

### Promise.all, Promise.race, Promise.allSettled

```javascript
// Promise.all - ждет выполнения всех Promise
async function fetchMultipleData() {
  try {
    const [users, posts, comments] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json()),
      fetch('/api/comments').then(r => r.json())
    ]);
    
    return { users, posts, comments };
  } catch (error) {
    console.error('Ошибка при загрузке данных:', error);
    throw error;
  }
}

// Promise.race - возвращает результат первого завершившегося Promise
async function fetchWithTimeout(url, timeout = 5000) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Таймаут запроса')), timeout);
  });
  
  const fetchPromise = fetch(url).then(r => r.json());
  
  try {
    const result = await Promise.race([fetchPromise, timeoutPromise]);
    return result;
  } catch (error) {
    console.error('Запрос не успел:', error.message);
    throw error;
  }
}

// Promise.allSettled - ждет завершения всех Promise (успешно или с ошибкой)
async function fetchWithPartialResults(urls) {
  const promises = urls.map(url => fetch(url).then(r => r.json()));
  const results = await Promise.allSettled(promises);
  
  const successful = results
    .filter(result => result.status === 'fulfilled')
    .map(result => result.value);
  
  const failed = results
    .filter(result => result.status === 'rejected')
    .map(result => result.reason);
  
  return { successful, failed };
}
```

## Async/Await

### Базовое использование

```javascript
// Асинхронная функция
async function getUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка! статус: ${response.status}`);
    }
    
    const userData = await response.json();
    console.log('Данные пользователя:', userData);
    
    // Загрузка дополнительных данных
    const postsResponse = await fetch(`/api/users/${userId}/posts`);
    const posts = await postsResponse.json();
    
    return { ...userData, posts };
  } catch (error) {
    console.error('Ошибка получения данных пользователя:', error);
    throw error;
  }
}

// Использование асинхронной функции
getUserData(123)
  .then(result => console.log('Результат:', result))
  .catch(error => console.error('Ошибка:', error));
```

### Параллельное выполнение с async/await

```javascript
// НЕ параллельное выполнение (медленно)
async function sequentialFetch() {
  const user = await fetch('/api/user').then(r => r.json());
  const posts = await fetch('/api/posts').then(r => r.json());
  const comments = await fetch('/api/comments').then(r => r.json());
  
  return { user, posts, comments };
}

// ПАРАЛЛЕЛЬное выполнение (быстро)
async function parallelFetch() {
  // Запускаем все запросы одновременно
  const userPromise = fetch('/api/user').then(r => r.json());
  const postsPromise = fetch('/api/posts').then(r => r.json());
  const commentsPromise = fetch('/api/comments').then(r => r.json());
  
  // Ждем завершения всех запросов
  const [user, posts, comments] = await Promise.all([
    userPromise,
    postsPromise,
    commentsPromise
  ]);
  
  return { user, posts, comments };
}
```

## Обработка ошибок

### try/catch в асинхронных функциях

```javascript
// Централизованная обработка ошибок
async function safeApiCall(url, options = {}) {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    });
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new ApiError(
        errorData.message || `HTTP ошибка! статус: ${response.status}`,
        response.status,
        errorData
      );
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof ApiError) {
      console.error(`API ошибка (${error.status}):`, error.message);
    } else if (error instanceof TypeError) {
      console.error('Ошибка сети:', error.message);
    } else {
      console.error('Неизвестная ошибка:', error);
    }
    
    throw error;
  }
}

// Кастомный класс ошибки
class ApiError extends Error {
  constructor(message, status, data = {}) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.data = data;
  }
}
```

### Повторные попытки (retry)

```javascript
// Функция с повторными попытками
async function retryAsync(asyncFn, maxRetries = 3, delay = 1000) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`Попытка ${attempt} из ${maxRetries}`);
      const result = await asyncFn();
      return result;
    } catch (error) {
      lastError = error;
      
      if (attempt === maxRetries) {
        console.error(`Все ${maxRetries} попыток не удались`);
        throw error;
      }
      
      // Экспоненциальный рост задержки
      const nextDelay = delay * Math.pow(2, attempt - 1);
      console.log(`Повторная попытка через ${nextDelay}мс...`);
      await new Promise(resolve => setTimeout(resolve, nextDelay));
    }
  }
  
  throw lastError;
}

// Использование с повторными попытками
async function unreliableApiCall() {
  const response = await fetch('/api/unreliable-endpoint');
  if (!response.ok) {
    throw new Error(`Ошибка: ${response.status}`);
  }
  return await response.json();
}

// Вызов с повторными попытками
retryAsync(unreliableApiCall, 3, 1000)
  .then(result => console.log('Успех:', result))
  .catch(error => console.error('Неудача после всех попыток:', error));
```

### Обработка ошибок в цепочке

```javascript
// Обработка ошибок на разных уровнях
class ApiService {
  constructor() {
    this.baseURL = '/api';
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    
    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ошибка: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      // Логирование ошибки
      console.error(`Ошибка запроса к ${url}:`, error.message);
      
      // Передача ошибки дальше для обработки на более высоком уровне
      throw error;
    }
  }
  
  async getUser(userId) {
    try {
      return await this.request(`/users/${userId}`);
    } catch (error) {
      // Конкретная обработка ошибки получения пользователя
      if (error.message.includes('404')) {
        console.warn(`Пользователь ${userId} не найден`);
        return null;
      }
      throw error; // Передаем другие ошибки дальше
    }
  }
  
  async getUsers() {
    try {
      return await this.request('/users');
    } catch (error) {
      // Обработка ошибки получения списка пользователей
      console.error('Ошибка получения списка пользователей:', error);
      return []; // Возвращаем пустой массив вместо ошибки
    }
  }
}
```

## Продвинутые паттерны

### Декоратор для обработки ошибок

```javascript
// Декоратор для автоматической обработки ошибок
function withErrorHandling(target, propertyName, descriptor) {
  const method = descriptor.value;
  
  descriptor.value = async function(...args) {
    try {
      return await method.apply(this, args);
    } catch (error) {
      console.error(`Ошибка в методе ${propertyName}:`, error);
      throw error;
    }
  };
  
  return descriptor;
}

// Использование декоратора
class UserService {
  @withErrorHandling
  async fetchUser(id) {
    const response = await fetch(`/api/users/${id}`);
    return await response.json();
  }
  
  @withErrorHandling
  async updateUser(id, data) {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
    return await response.json();
  }
}
```

### Асинхронная очередь задач

```javascript
class AsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async add(asyncFn) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        asyncFn,
        resolve,
        reject
      });
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    
    const { asyncFn, resolve, reject } = this.queue.shift();
    
    try {
      const result = await asyncFn();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process(); // Обрабатываем следующую задачу
    }
  }
}

// Использование очереди
const queue = new AsyncQueue(2); // Максимум 2 одновременные задачи

for (let i = 0; i < 5; i++) {
  queue.add(async () => {
    console.log(`Начало задачи ${i}`);
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log(`Завершение задачи ${i}`);
    return `Результат задачи ${i}`;
  }).then(result => console.log(result));
}
```

### Кэширование асинхронных результатов

```javascript
class CachedApiService {
  constructor() {
    this.cache = new Map();
    this.pendingRequests = new Map();
  }
  
  async getWithCache(key, fetchFn, ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
    // Проверяем кэш
    if (this.cache.has(key)) {
      const { data, timestamp } = this.cache.get(key);
      if (Date.now() - timestamp < ttl) {
        console.log(`Данные взяты из кэша: ${key}`);
        return data;
      } else {
        // Данные устарели, удаляем из кэша
        this.cache.delete(key);
      }
    }
    
    // Проверяем, есть ли уже выполняющийся запрос
    if (this.pendingRequests.has(key)) {
      console.log(`Ожидание результата уже выполняющегося запроса: ${key}`);
      return await this.pendingRequests.get(key);
    }
    
    // Создаем новый запрос
    const requestPromise = fetchFn().then(data => {
      // Сохраняем в кэш
      this.cache.set(key, {
        data,
        timestamp: Date.now()
      });
      // Удаляем из ожидающих
      this.pendingRequests.delete(key);
      return data;
    }).catch(error => {
      // Удаляем из ожидающих в случае ошибки
      this.pendingRequests.delete(key);
      throw error;
    });
    
    this.pendingRequests.set(key, requestPromise);
    
    return await requestPromise;
  }
  
  // Очистка устаревшего кэша
  clearExpired() {
    const now = Date.now();
    for (const [key, { timestamp }] of this.cache.entries()) {
      if (now - timestamp > 5 * 60 * 1000) { // 5 минут
        this.cache.delete(key);
      }
    }
  }
}

// Использование
const cachedApi = new CachedApiService();

async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return await response.json();
}

// Первый вызов - загрузит данные
cachedApi.getWithCache('user-123', () => fetchUser(123))
  .then(user => console.log('Пользователь:', user));

// Второй вызов - вернет из кэша
setTimeout(() => {
  cachedApi.getWithCache('user-123', () => fetchUser(123))
    .then(user => console.log('Пользователь из кэша:', user));
}, 1000);
```

## Практические примеры

### Загрузка данных с прогрессом

```javascript
class DataLoader {
  constructor() {
    this.progressCallbacks = [];
  }
  
  onProgress(callback) {
    this.progressCallbacks.push(callback);
  }
  
  async loadWithProgress(urls) {
    const results = [];
    const total = urls.length;
    let loaded = 0;
    
    for (const url of urls) {
      try {
        const response = await fetch(url);
        const data = await response.json();
        results.push(data);
        
        loaded++;
        const progress = (loaded / total) * 100;
        
        // Уведомляем подписчиков о прогрессе
        this.progressCallbacks.forEach(callback => 
          callback({ loaded, total, progress, currentUrl: url })
        );
      } catch (error) {
        console.error(`Ошибка загрузки ${url}:`, error);
        results.push(null); // Добавляем null в случае ошибки
        
        loaded++;
        const progress = (loaded / total) * 100;
        this.progressCallbacks.forEach(callback => 
          callback({ loaded, total, progress, currentUrl: url, error })
        );
      }
    }
    
    return results;
  }
}

// Использование
const loader = new DataLoader();

loader.onProgress(({ loaded, total, progress, currentUrl, error }) => {
  console.log(`Загружено: ${loaded}/${total} (${Math.round(progress)}%) - ${currentUrl}`);
  if (error) {
    console.error('Ошибка:', error);
  }
});

const urls = [
  '/api/data1',
  '/api/data2',
  '/api/data3'
];

loader.loadWithProgress(urls)
  .then(results => console.log('Все данные загружены:', results))
  .catch(error => console.error('Критическая ошибка:', error));
```

### Асинхронная валидация формы

```javascript
class AsyncFormValidator {
  constructor() {
    this.validators = new Map();
    this.errorMessages = new Map();
  }
  
  addValidator(fieldName, asyncValidator, errorMessage) {
    if (!this.validators.has(fieldName)) {
      this.validators.set(fieldName, []);
    }
    
    this.validators.get(fieldName).push({
      validate: asyncValidator,
      message: errorMessage
    });
  }
  
  async validateField(fieldName, value) {
    const fieldValidators = this.validators.get(fieldName) || [];
    const errors = [];
    
    for (const validator of fieldValidators) {
      try {
        const isValid = await validator.validate(value);
        if (!isValid) {
          errors.push(validator.message);
        }
      } catch (error) {
        console.error(`Ошибка валидации поля ${fieldName}:`, error);
        errors.push('Ошибка проверки поля');
      }
    }
    
    if (errors.length > 0) {
      this.errorMessages.set(fieldName, errors);
      return false;
    }
    
    this.errorMessages.delete(fieldName);
    return true;
  }
  
  async validateForm(formData) {
    const fieldNames = Object.keys(formData);
    const results = await Promise.allSettled(
      fieldNames.map(fieldName => 
        this.validateField(fieldName, formData[fieldName])
      )
    );
    
    const isValid = results.every(result => 
      result.status === 'fulfilled' && result.value
    );
    
    return {
      isValid,
      errors: Object.fromEntries(this.errorMessages.entries())
    };
  }
}

// Пример использования
const formValidator = new AsyncFormValidator();

// Добавление асинхронных валидаторов
formValidator.addValidator(
  'email',
  async (email) => {
    const response = await fetch(`/api/check-email?email=${encodeURIComponent(email)}`);
    const result = await response.json();
    return result.available;
  },
  'Email уже используется'
);

formValidator.addValidator(
  'username',
  async (username) => {
    const response = await fetch(`/api/check-username?username=${encodeURIComponent(username)}`);
    const result = await response.json();
    return result.available;
  },
  'Имя пользователя уже занято'
);

// Валидация формы
async function validateRegistrationForm(data) {
  const result = await formValidator.validateForm(data);
  
  if (!result.isValid) {
    console.log('Ошибки валидации:', result.errors);
    return { success: false, errors: result.errors };
  }
  
  return { success: true };
}
```

## Лучшие практики

### Обработка отмены запросов

```javascript
// Отмена асинхронных операций
class CancellableOperation {
  constructor() {
    this.abortController = new AbortController();
  }
  
  async performAsyncOperation(promiseFactory) {
    try {
      const result = await promiseFactory(this.abortController.signal);
      return result;
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('Операция была отменена');
        return null;
      }
      throw error;
    }
  }
  
  cancel() {
    this.abortController.abort();
  }
}

// Использование с fetch
async function fetchWithCancellation(url, signal) {
  const response = await fetch(url, { signal });
  return await response.json();
}

const operation = new CancellableOperation();

operation.performAsyncOperation(signal => 
  fetchWithCancellation('/api/data', signal)
).then(result => {
  if (result !== null) {
    console.log('Результат:', result);
  }
});

// Отмена операции через 5 секунд
setTimeout(() => {
  operation.cancel();
}, 5000);
```

### Управление состоянием загрузки

```javascript
// Класс для управления асинхронными состояниями
class AsyncState {
  constructor() {
    this.loading = false;
    this.error = null;
    this.data = null;
    this.subscribers = [];
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers() {
    this.subscribers.forEach(callback => callback({
      loading: this.loading,
      error: this.error,
      data: this.data
    }));
  }
  
  async execute(asyncFn) {
    this.loading = true;
    this.error = null;
    this.notifySubscribers();
    
    try {
      this.data = await asyncFn();
      this.loading = false;
      this.notifySubscribers();
      return this.data;
    } catch (error) {
      this.loading = false;
      this.error = error;
      this.notifySubscribers();
      throw error;
    }
  }
  
  reset() {
    this.loading = false;
    this.error = null;
    this.data = null;
    this.notifySubscribers();
  }
}

// Использование
const userState = new AsyncState();

userState.subscribe(({ loading, error, data }) => {
  if (loading) {
    console.log('Загрузка...');
  } else if (error) {
    console.error('Ошибка:', error);
  } else if (data) {
    console.log('Данные:', data);
  }
});

// Выполнение асинхронной операции
userState.execute(async () => {
  const response = await fetch('/api/user');
  return await response.json();
});
```

## Ссылки по теме

- [[js/async/promises]] - Promise
- [[js/async/async-await]] - Async/Await
- [[js/error-handling/best-practices]] - Лучшие практики обработки ошибок
- [[js/patterns/retry-mechanism]] - Механизм повторных попыток