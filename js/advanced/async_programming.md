# Асинхронное программирование в JavaScript

## Введение в асинхронность

Асинхронное программирование - это подход к написанию кода, который позволяет выполнять операции без блокировки основного потока выполнения. Это особенно важно в JavaScript, который работает в однопоточной среде, где длительные операции могут "заморозить" интерфейс пользователя.

## Callbacks

### Основы callbacks

```javascript
// Синхронная функция
function syncOperation(data) {
  return data * 2;
}

console.log(syncOperation(5)); // 10
console.log("Это выполнится после");

// Асинхронная функция с callback
function asyncOperation(data, callback) {
  setTimeout(() => {
    const result = data * 2;
    callback(null, result); // null - ошибка, result - данные
  }, 1000);
}

asyncOperation(5, (error, result) => {
  if (error) {
    console.error("Ошибка:", error);
  } else {
    console.log("Результат:", result); // 10
  }
});

console.log("Это выполнится до результата");
```

### Проблема callback hell

```javascript
// Callback hell - пирамида вложенности
function getUser(userId, callback) {
  setTimeout(() => {
    callback(null, { id: userId, name: "Иван" });
  }, 100);
}

function getUserPosts(userId, callback) {
  setTimeout(() => {
    callback(null, [{ id: 1, title: "Пост 1" }, { id: 2, title: "Пост 2" }]);
  }, 100);
}

function getPostComments(postId, callback) {
  setTimeout(() => {
    callback(null, [{ id: 1, text: "Комментарий 1" }]);
  }, 100);
}

// Пирамида вложенности
getUser(1, (error, user) => {
  if (error) {
    console.error(error);
    return;
  }
  
  console.log("Пользователь:", user);
  
  getUserPosts(user.id, (error, posts) => {
    if (error) {
      console.error(error);
      return;
    }
    
    console.log("Посты:", posts);
    
    if (posts.length > 0) {
      getPostComments(posts[0].id, (error, comments) => {
        if (error) {
          console.error(error);
          return;
        }
        
        console.log("Комментарии:", comments);
      });
    }
  });
});
```

## Promises

### Создание и использование Promise

```javascript
// Создание Promise
function fetchData(url) {
  return new Promise((resolve, reject) => {
    // Имитация асинхронной операции
    setTimeout(() => {
      if (url) {
        resolve(`Данные с ${url}`);
      } else {
        reject(new Error("Неверный URL"));
      }
    }, 1000);
  });
}

// Использование Promise
fetchData("https://api.example.com")
  .then(data => {
    console.log("Успех:", data);
    return data.toUpperCase();
  })
  .then(upperData => {
    console.log("Преобразованные данные:", upperData);
  })
  .catch(error => {
    console.error("Ошибка:", error.message);
  })
  .finally(() => {
    console.log("Операция завершена");
  });
```

### Методы Promise

#### Promise.resolve() и Promise.reject()

```javascript
// Создание разрешенного Promise
const resolvedPromise = Promise.resolve("Успешное значение");
resolvedPromise.then(value => console.log(value)); // "Успешное значение"

// Создание отклоненного Promise
const rejectedPromise = Promise.reject(new Error("Ошибка"));
rejectedPromise.catch(error => console.error(error.message)); // "Ошибка"
```

#### Promise.all()

```javascript
// Выполнение нескольких Promise параллельно
const promises = [
  fetchData("https://api1.example.com"),
  fetchData("https://api2.example.com"),
  fetchData("https://api3.example.com")
];

Promise.all(promises)
  .then(results => {
    console.log("Все данные получены:", results);
    // ["Данные с https://api1.example.com", "Данные с https://api2.example.com", "Данные с https://api3.example.com"]
  })
  .catch(error => {
    console.error("Ошибка в одном из запросов:", error);
  });
```

#### Promise.allSettled()

```javascript
// Ожидание завершения всех Promise независимо от результата
const promises = [
  Promise.resolve("Успех 1"),
  Promise.reject(new Error("Ошибка 1")),
  Promise.resolve("Успех 2")
];

Promise.allSettled(promises)
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === "fulfilled") {
        console.log(`Promise ${index}: успех - ${result.value}`);
      } else {
        console.log(`Promise ${index}: ошибка - ${result.reason.message}`);
      }
    });
  });
```

#### Promise.race()

```javascript
// Первый завершившийся Promise определяет результат
const promises = [
  new Promise(resolve => setTimeout(() => resolve("Первый"), 1000)),
  new Promise(resolve => setTimeout(() => resolve("Второй"), 2000)),
  new Promise(resolve => setTimeout(() => resolve("Третий"), 3000))
];

Promise.race(promises)
  .then(result => {
    console.log("Первый завершенный:", result); // "Первый"
  });

// Таймаут для Promise
function withTimeout(promise, timeout) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error("Таймаут")), timeout)
    )
  ]);
}
```

## Async/Await

### Основы async/await

```javascript
// Асинхронная функция
async function fetchUserData(userId) {
  try {
    // Ожидание разрешения Promise
    const user = await fetchData(`https://api.example.com/users/${userId}`);
    console.log("Пользователь:", user);
    
    const posts = await fetchData(`https://api.example.com/users/${userId}/posts`);
    console.log("Посты:", posts);
    
    return { user, posts };
  } catch (error) {
    console.error("Ошибка при получении данных:", error.message);
    throw error; // Пробрасываем ошибку дальше
  }
}

// Использование асинхронной функции
fetchUserData(123)
  .then(result => {
    console.log("Все данные получены:", result);
  })
  .catch(error => {
    console.error("Ошибка:", error.message);
  });
```

### Параллельное выполнение

```javascript
// Последовательное выполнение (медленно)
async function sequentialExecution() {
  const start = Date.now();
  
  const user = await fetchData("https://api.example.com/user");
  const posts = await fetchData("https://api.example.com/posts");
  const comments = await fetchData("https://api.example.com/comments");
  
  console.log("Последовательно:", Date.now() - start, "мс");
  return { user, posts, comments };
}

// Параллельное выполнение (быстро)
async function parallelExecution() {
  const start = Date.now();
  
  // Все запросы запускаются одновременно
  const [user, posts, comments] = await Promise.all([
    fetchData("https://api.example.com/user"),
    fetchData("https://api.example.com/posts"),
    fetchData("https://api.example.com/comments")
  ]);
  
  console.log("Параллельно:", Date.now() - start, "мс");
  return { user, posts, comments };
}
```

### Обработка ошибок в async/await

```javascript
// Обработка ошибок для отдельных операций
async function handleIndividualErrors() {
  try {
    const user = await fetchData("https://api.example.com/user");
    console.log("Пользователь:", user);
  } catch (error) {
    console.error("Ошибка получения пользователя:", error.message);
  }
  
  try {
    const posts = await fetchData("https://api.example.com/posts");
    console.log("Посты:", posts);
  } catch (error) {
    console.error("Ошибка получения постов:", error.message);
  }
}

// Использование try/catch с Promise.all
async function handleAllErrors() {
  try {
    const [user, posts] = await Promise.all([
      fetchData("https://api.example.com/user"),
      fetchData("https://api.example.com/posts")
    ]);
    
    console.log("Все данные получены:", { user, posts });
  } catch (error) {
    console.error("Ошибка в одном из запросов:", error.message);
  }
}
```

## Практические примеры

### Работа с API

```javascript
// Класс для работы с API
class ApiClient {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL;
    this.defaultOptions = {
      timeout: 5000,
      headers: {
        'Content-Type': 'application/json'
      },
      ...options
    };
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = { ...this.defaultOptions, ...options };
    
    // Создание контроллера для таймаута
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), config.timeout);
    
    try {
      const response = await fetch(url, {
        ...config,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      return await response.json();
    } catch (error) {
      clearTimeout(timeoutId);
      
      if (error.name === 'AbortError') {
        throw new Error('Таймаут запроса');
      }
      
      throw error;
    }
  }
  
  async get(endpoint) {
    return this.request(endpoint, { method: 'GET' });
  }
  
  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  async put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  async delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

// Использование
const api = new ApiClient('https://jsonplaceholder.typicode.com');

async function fetchUserPosts(userId) {
  try {
    const [user, posts] = await Promise.all([
      api.get(`/users/${userId}`),
      api.get(`/users/${userId}/posts`)
    ]);
    
    return {
      user: user,
      posts: posts,
      totalPosts: posts.length
    };
  } catch (error) {
    console.error('Ошибка при получении данных:', error.message);
    throw error;
  }
}

fetchUserPosts(1)
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### Цепочка асинхронных операций

```javascript
// Утилита для последовательного выполнения асинхронных операций
async function asyncSequence(operations) {
  const results = [];
  
  for (const operation of operations) {
    try {
      const result = await operation();
      results.push(result);
    } catch (error) {
      results.push({ error: error.message });
    }
  }
  
  return results;
}

// Утилита для параллельного выполнения с ограничением количества одновременных операций
async function asyncPool(poolLimit, array, iteratorFn) {
  const ret = [];
  const executing = [];
  
  for (const item of array) {
    const p = Promise.resolve().then(() => iteratorFn(item, array));
    ret.push(p);
    
    if (poolLimit <= array.length) {
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      
      if (executing.length >= poolLimit) {
        await Promise.race(executing);
      }
    }
  }
  
  return Promise.all(ret);
}

// Пример использования
const urls = [
  'https://api1.example.com',
  'https://api2.example.com',
  'https://api3.example.com',
  'https://api4.example.com',
  'https://api5.example.com'
];

// Ограничение - максимум 2 одновременных запроса
asyncPool(2, urls, async (url) => {
  const response = await fetch(url);
  return response.json();
})
.then(results => console.log('Все данные получены:', results))
.catch(error => console.error('Ошибка:', error));
```

### Retry механизм

```javascript
// Асинхронная функция с повторными попытками
async function retryAsyncOperation(operation, maxRetries = 3, delay = 1000) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`Попытка ${attempt}/${maxRetries}`);
      return await operation();
    } catch (error) {
      lastError = error;
      console.log(`Попытка ${attempt} не удалась:`, error.message);
      
      if (attempt < maxRetries) {
        console.log(`Ожидание ${delay}мс перед следующей попыткой...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2; // Экспоненциальное увеличение задержки
      }
    }
  }
  
  throw new Error(`Все ${maxRetries} попыток не удались. Последняя ошибка: ${lastError.message}`);
}

// Пример использования
async function unreliableOperation() {
  // Имитация нестабильной операции
  if (Math.random() < 0.7) {
    throw new Error("Случайная ошибка");
  }
  return "Успех!";
}

retryAsyncOperation(unreliableOperation, 5, 500)
  .then(result => console.log("Результат:", result))
  .catch(error => console.error("Все попытки не удались:", error.message));
```

### Асинхронные генераторы

```javascript
// Асинхронный генератор
async function* asyncNumberGenerator(start, end, delay = 1000) {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, delay));
    yield i;
  }
}

// Использование асинхронного генератора
async function useAsyncGenerator() {
  console.log("Начало генерации...");
  
  for await (const number of asyncNumberGenerator(1, 5, 500)) {
    console.log("Сгенерировано:", number);
  }
  
  console.log("Генерация завершена");
}

useAsyncGenerator();

// Асинхронный итератор для обработки потоковых данных
async function* fetchPaginatedData(url, maxPages = 10) {
  let page = 1;
  let hasNext = true;
  
  while (hasNext && page <= maxPages) {
    try {
      const response = await fetch(`${url}?page=${page}`);
      const data = await response.json();
      
      yield data;
      
      // Проверка наличия следующей страницы
      hasNext = data.hasNextPage || false;
      page++;
    } catch (error) {
      console.error(`Ошибка при загрузке страницы ${page}:`, error.message);
      break;
    }
  }
}

// Использование
async function processAllPages() {
  for await (const pageData of fetchPaginatedData('https://api.example.com/data')) {
    console.log('Обработка страницы:', pageData);
    // Обработка данных страницы
  }
}
```

## Лучшие практики

### 1. Всегда обрабатывайте ошибки

```javascript
// Плохо: без обработки ошибок
async function badExample() {
  const data = await fetchData("https://api.example.com");
  return data;
}

// Хорошо: с обработкой ошибок
async function goodExample() {
  try {
    const data = await fetchData("https://api.example.com");
    return data;
  } catch (error) {
    console.error("Ошибка при получении данных:", error.message);
    // Логика восстановления или проброс ошибки
    throw new Error(`Не удалось получить данные: ${error.message}`);
  }
}
```

### 2. Используйте Promise.all для параллельных операций

```javascript
// Плохо: последовательное выполнение
async function badParallel() {
  const user = await fetchUser(1);
  const posts = await fetchUserPosts(1);
  const comments = await fetchUserComments(1);
  return { user, posts, comments };
}

// Хорошо: параллельное выполнение
async function goodParallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),
    fetchUserPosts(1),
    fetchUserComments(1)
  ]);
  return { user, posts, comments };
}
```

### 3. Используйте таймауты

```javascript
// Утилита для добавления таймаута к Promise
function withTimeout(promise, timeoutMs) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Таймаут')), timeoutMs)
    )
  ]);
}

// Использование
async function fetchWithTimeout() {
  try {
    const data = await withTimeout(fetchData("https://api.example.com"), 5000);
    return data;
  } catch (error) {
    if (error.message === 'Таймаут') {
      console.error('Запрос превысил время ожидания');
    }
    throw error;
  }
}
```

## Современные возможности

### Top-level await (ES2022)

```javascript
// В модулях можно использовать await на верхнем уровне
// config.js
const config = await fetch('/api/config').then(res => res.json());
export default config;

// main.js
import config from './config.js';
console.log('Конфигурация загружена:', config);
```

### Promise.any()

```javascript
// Возвращает первый успешно завершившийся Promise
const promises = [
  fetch('https://fast-api.example.com/data'),
  fetch('https://backup-api.example.com/data'),
  fetch('https://fallback-api.example.com/data')
];

try {
  const response = await Promise.any(promises);
  const data = await response.json();
  console.log('Данные получены с первого доступного API:', data);
} catch (error) {
  console.error('Все API недоступны:', error.errors);
}
```

## Заключение

Асинхронное программирование является ключевой частью современной разработки на JavaScript. Понимание Promise, async/await и различных паттернов асинхронного программирования позволяет создавать эффективные, отзывчивые и надежные приложения. Правильное использование этих механизмов помогает избежать блокировки основного потока, обрабатывать ошибки и оптимизировать производительность приложений.

#javascript #programming #async #promises #async-await #web-development