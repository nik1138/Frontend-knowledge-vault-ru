---
aliases: [Промисы, Promise, Fetch API, Асинхронные запросы]
tags: [programming, javascript, html, асинхронность, promise, fetch, best-practices, 2025]
---

# Promise и Fetch

## Введение

Promise (Промис) - это объект, представляющий результат асинхронной операции. Fetch API - современный способ выполнения HTTP-запросов, возвращающий Promise. Эти технологии стали стандартом для асинхронных операций в веб-разработке 2025 года.

## Promise: основы

Promise - это объект, который может находиться в одном из трех состояний:

- `pending` - начальное состояние, операция еще не завершена
- `fulfilled` - операция завершена успешно
- `rejected` - операция завершена с ошибкой

```javascript
const myPromise = new Promise((resolve, reject) => {
    // Асинхронная операция
    setTimeout(() => {
        const success = Math.random() > 0.5;
        if (success) {
            resolve('Операция выполнена успешно');
        } else {
            reject('Произошла ошибка');
        }
    }, 1000);
});
```

## Методы Promise

### .then()

Используется для обработки успешного выполнения промиса:

```javascript
myPromise
    .then(result => {
        console.log('Успешно:', result);
        return result.toUpperCase();
    })
    .then(uppercaseResult => {
        console.log('Преобразованный результат:', uppercaseResult);
    });
```

### .catch()

Обрабатывает ошибки:

```javascript
myPromise
    .then(result => {
        console.log('Результат:', result);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

### .finally()

Выполняется независимо от результата:

```javascript
myPromise
    .then(result => {
        console.log('Успешно:', result);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    })
    .finally(() => {
        console.log('Операция завершена');
    });
```

## Цепочки промисов

Promise позволяют строить цепочки асинхронных операций:

```javascript
function getUserData(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return response.json();
        })
        .then(userData => {
            return fetch(`/api/users/${userId}/posts`)
                .then(postsResponse => postsResponse.json())
                .then(posts => {
                    return { ...userData, posts };
                });
        });
}

getUserData(123)
    .then(completeUserData => {
        console.log('Полные данные пользователя:', completeUserData);
    })
    .catch(error => {
        console.error('Ошибка получения данных пользователя:', error);
    });
```

## Fetch API

Fetch API предоставляет интерфейс для выполнения сетевых запросов. В отличие от XMLHttpRequest, fetch возвращает Promise.

### Простой GET-запрос

```javascript
fetch('/api/data')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
    })
    .then(data => {
        console.log('Данные получены:', data);
    })
    .catch(error => {
        console.error('Ошибка запроса:', error);
    });
```

### POST-запрос с данными

```javascript
const postData = {
    title: 'Новый пост',
    content: 'Содержимое поста',
    author: 'Иван Петров'
};

fetch('/api/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + token
    },
    body: JSON.stringify(postData)
})
.then(response => response.json())
.then(result => {
    console.log('Пост создан:', result);
})
.catch(error => {
    console.error('Ошибка создания поста:', error);
});
```

## Современные практики работы с Fetch

### Проверка ответа

Важно проверять статус ответа перед обработкой данных:

```javascript
async function fetchWithErrorHandling(url) {
    try {
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка при выполнении запроса:', error);
        throw error;
    }
}
```

### Использование AbortController для отмены запросов

```javascript
const controller = new AbortController();
const signal = controller.signal;

// Отмена запроса через 5 секунд
setTimeout(() => controller.abort(), 5000);

fetch('/api/data', { signal })
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Запрос был отменен');
        } else {
            console.error('Ошибка:', error);
        }
    });
```

## Методы для работы с несколькими промисами

### Promise.all()

Выполняется, когда все промисы завершены успешно:

```javascript
const urls = [
    '/api/users',
    '/api/posts',
    '/api/comments'
];

Promise.all(urls.map(url => fetch(url)))
    .then(responses => Promise.all(responses.map(response => response.json())))
    .then(data => {
        const [users, posts, comments] = data;
        console.log('Все данные:', { users, posts, comments });
    })
    .catch(error => {
        console.error('Ошибка при загрузке данных:', error);
    });
```

### Promise.allSettled()

Ждет завершения всех промисов, независимо от результата:

```javascript
const requests = [
    fetch('/api/users'),
    fetch('/api/posts'),
    fetch('/api/invalid') // Этот запрос завершится с ошибкой
];

Promise.allSettled(requests)
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Запрос ${index} успешен`, result.value);
            } else {
                console.log(`Запрос ${index} неудачен`, result.reason);
            }
        });
    });
```

### Promise.race()

Выполняется при завершении первого промиса:

```javascript
Promise.race([
    fetch('/api/fast'),
    new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Таймаут')), 5000)
    )
])
.then(response => response.json())
.catch(error => {
    if (error.message === 'Таймаут') {
        console.log('Запрос занял слишком много времени');
    } else {
        console.error('Ошибка запроса:', error);
    }
});
```

## Практические рекомендации

### 1. Обработка ошибок

Всегда проверяйте статус HTTP-ответа и обрабатывайте ошибки:

```javascript
// Хорошо
async function safeFetch(url) {
    try {
        const response = await fetch(url);
        if (!response.ok) {
            throw new Error(`Ошибка ${response.status}: ${response.statusText}`);
        }
        return await response.json();
    } catch (error) {
        console.error('Ошибка при выполнении запроса:', error);
        throw error;
    }
}
```

### 2. Повторные попытки (retry)

Реализация повторных попыток для неудачных запросов:

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
    for (let i = 0; i <= maxRetries; i++) {
        try {
            const response = await fetch(url, options);
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return response;
        } catch (error) {
            if (i === maxRetries) {
                throw error; // Все попытки исчерпаны
            }
            console.log(`Попытка ${i + 1} не удалась, повтор...`);
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1))); // Экспоненциальная задержка
        }
    }
}
```

### 3. Управление загрузкой данных

Используйте Promise.all для параллельной загрузки независимых данных:

```javascript
async function loadDashboardData() {
    const [users, posts, stats] = await Promise.all([
        fetch('/api/users').then(r => r.json()),
        fetch('/api/posts').then(r => r.json()),
        fetch('/api/stats').then(r => r.json())
    ]);
    
    return { users, posts, stats };
}
```

## Совместимость и поддержка

В 2025 году все современные браузеры поддерживают Promise и Fetch API. Для поддержки старых браузеров используйте полифиллы:

```html
<script src="https://cdn.jsdelivr.net/npm/promise-polyfill@8/dist/polyfill.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/whatwg-fetch@3.6.2/dist/fetch.umd.js"></script>
```

## Заключение

Promise и Fetch API являются стандартом для асинхронных операций в современной веб-разработке. Они обеспечивают читаемый, понятный способ обработки асинхронных операций и HTTP-запросов. В сочетании с [[Async-await]] и правильной [[Обработка-ошибок]] они позволяют создавать надежные и эффективные веб-приложения.

## Ключевые тезисы

- Promise предоставляет способ работы с асинхронными операциями
- Fetch API - современный способ выполнения HTTP-запросов
- Всегда проверяйте статус HTTP-ответа
- Используйте AbortController для отмены запросов
- Применяйте Promise.all, Promise.allSettled и Promise.race для работы с несколькими промисами