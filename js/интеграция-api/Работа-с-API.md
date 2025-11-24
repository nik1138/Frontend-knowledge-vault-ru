---
aliases: ["API Integration Basics", "JavaScript API", "Fetch API", "HTTP Requests in JS"]
tags: [javascript, api, http, frontend, web-development]
---

# Работа с API в JavaScript

## Общее понимание API

API (Application Programming Interface) - это интерфейс программирования приложений, который позволяет клиентским приложениям обмениваться данными с серверами. В контексте веб-разработки, API обычно представляют собой HTTP-интерфейсы, через которые приложения могут запрашивать и отправлять данные.

## Основные методы работы с API

### Fetch API

Fetch API предоставляет современный способ выполнения HTTP-запросов в браузере. Это нативный JavaScript API, который использует промисы (promises) для обработки асинхронных операций.

```javascript
// Пример GET-запроса
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Ошибка:', error));
```

### Async/Await

Современный синтаксис для работы с асинхронными операциями, делает код более читаемым:

```javascript
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

## Типы HTTP-запросов

### GET-запросы

Используются для получения данных с сервера:

```javascript
async function getData() {
  try {
    const response = await fetch('/api/users', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка при получении данных:', error);
  }
}
```

### POST-запросы

Используются для отправки данных на сервер:

```javascript
async function postData(payload) {
  try {
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка при отправке данных:', error);
  }
}
```

### PUT/PATCH-запросы

Используются для обновления существующих данных:

```javascript
async function updateData(id, updates) {
  try {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(updates)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка при обновлении данных:', error);
  }
}
```

### DELETE-запросы

Используются для удаления данных:

```javascript
async function deleteData(id) {
  try {
    const response = await fetch(`/api/users/${id}`, {
      method: 'DELETE',
      headers: {
        'Content-Type': 'application/json',
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.status;
  } catch (error) {
    console.error('Ошибка при удалении данных:', error);
  }
}
```

## Практические рекомендации

### Обработка CORS

При разработке в российских условиях часто сталкиваются с CORS-политикой. Важно понимать, как с ней работать:

```javascript
// При необходимости отправки credentials
fetch('https://api.example.com/data', {
  method: 'GET',
  credentials: 'include', // или 'same-origin', 'omit'
  headers: {
    'Content-Type': 'application/json',
  }
})
```

### Настройка таймаута запросов

В условиях нестабильного интернета в России важно устанавливать таймауты:

```javascript
function fetchWithTimeout(url, options = {}, timeout = 8000) {
  return Promise.race([
    fetch(url, options),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Request timeout')), timeout)
    )
  ]);
}

// Использование
fetchWithTimeout('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

### Параметры запроса

Для GET-запросов с параметрами рекомендуется использовать URLSearchParams:

```javascript
function getDataWithParams(params) {
  const url = new URL('/api/users', window.location.origin);
  
  // Добавляем параметры
  Object.keys(params).forEach(key => {
    url.searchParams.append(key, params[key]);
  });
  
  return fetch(url).then(response => response.json());
}

// Использование
getDataWithParams({ page: 1, limit: 10, sort: 'name' });
```

## Axios как альтернатива

Хотя Fetch API является нативным решением, в российской разработке часто используется Axios за его удобство:

```javascript
import axios from 'axios';

// GET-запрос
axios.get('/api/users')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));

// POST-запрос
axios.post('/api/users', { name: 'John', email: 'john@example.com' })
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

## Работа с заголовками

Важно корректно устанавливать заголовки для разных типов данных:

```javascript
const headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'X-Requested-With': 'XMLHttpRequest',
  // В российских проектах часто требуется указывать локализацию
  'Accept-Language': 'ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7',
};

fetch('/api/data', { headers });
```

## Заключение

Работа с API в JavaScript требует понимания асинхронности, HTTP-протокола и особенностей CORS. В российских условиях важно учитывать стабильность соединения, требования к безопасности и локализацию.

Для более сложных сценариев рекомендуется использовать специализированные библиотеки, такие как [[Axios]], [[GraphQL]] или [[REST-архитектура]].

См. также:
- [[Обработка-ответов]]
- [[Обработка-ошибок]]
- [[Авторизация-и-аутентификация]]
- [[Кэширование-запросов]]