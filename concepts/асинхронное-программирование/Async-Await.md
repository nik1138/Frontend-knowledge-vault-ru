---
aliases: [Async/Await, Асинхронные функции, async await]
tags: [javascript, async-programming, async-await, frontend]
---

# Async/Await

## Описание

Async/await - это синтаксический сахар над промисами, введенный в ES2017, который позволяет писать асинхронный код так, как будто он синхронный. Это делает код более читаемым и легким для понимания.

## Применение

Async/await используется для упрощения работы с промисами и создания более последовательного и понятного кода при выполнении асинхронных операций.

### Основы async/await

```javascript
// Объявление асинхронной функции
async function fetchData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка при получении данных:', error);
        throw error;
    }
}

// Вызов асинхронной функции
fetchData()
    .then(data => console.log(data))
    .catch(error => console.error('Ошибка:', error));
```

### Использование с промисами

```javascript
// Асинхронная функция возвращает промис
async function processUserData(userId) {
    try {
        const user = await getUserById(userId);
        const posts = await getUserPosts(userId);
        const profile = await getUserProfile(userId);
        
        return {
            user,
            posts,
            profile
        };
    } catch (error) {
        console.error('Ошибка при обработке данных пользователя:', error);
        throw error;
    }
}
```

### Параллельное выполнение

```javascript
// НЕПРАВИЛЬНО - последовательное выполнение
async function badExample(userId) {
    const user = await getUserById(userId);      // Ждет 100ms
    const posts = await getUserPosts(userId);    // Ждет 200ms
    const profile = await getUserProfile(userId); // Ждет 150ms
    // Общее время: ~450ms
    
    return { user, posts, profile };
}

// ПРАВИЛЬНО - параллельное выполнение
async function goodExample(userId) {
    // Запускаем все промисы одновременно
    const userPromise = getUserById(userId);
    const postsPromise = getUserPosts(userId);
    const profilePromise = getUserProfile(userId);
    
    // Ждем завершения всех промисов
    const [user, posts, profile] = await Promise.all([
        userPromise,
        postsPromise,
        profilePromise
    ]);
    // Общее время: ~200ms (максимальное время среди промисов)
    
    return { user, posts, profile };
}
```

### Обработка ошибок

```javascript
// Глобальная обработка ошибок
async function apiCall(url) {
    try {
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка! Статус: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        // Логирование ошибки
        console.error('Ошибка API вызова:', error);
        
        // Передача ошибки дальше для обработки в вызывающем коде
        throw error;
    }
}

// Использование в цепочке
async function complexOperation() {
    try {
        const data1 = await apiCall('/api/data1');
        const data2 = await apiCall('/api/data2');
        const processed = await processData(data1, data2);
        
        return processed;
    } catch (error) {
        // Обработка ошибок на уровне операции
        console.error('Ошибка в сложной операции:', error);
        throw new Error('Не удалось выполнить сложную операцию');
    }
}
```

## Практические советы

1. **Используйте try/catch** для обработки ошибок в async функциях
2. **Параллельное выполнение** - запускайте независимые промисы одновременно с помощью Promise.all()
3. **Не используйте await без необходимости** - если промис не нужен для следующей операции, не ждите его
4. **Избегайте вложенных async функций** - используйте отдельные функции для лучшей читаемости
5. **Возвращайте значения из async функций** - они автоматически оборачиваются в промис

## Распространенные ошибки

```javascript
// ОШИБКА: Использование await вне async функции
function wrongExample() {
    const result = await someAsyncFunction(); // SyntaxError
}

// ОШИБКА: Неправильная обработка ошибок
async function wrongErrorHandling() {
    const response = await fetch('/api/data'); // Если ошибка - она не будет обработана
    const data = await response.json();
    return data;
}

// ПРАВИЛЬНО: Обработка ошибок
async function correctErrorHandling() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка при получении данных:', error);
        throw error;
    }
}
```

## Связанные темы

- [[Promises]] - основа, на которой построен async/await
- [[Callbacks]] - предшественник асинхронных подходов
- [[Event-Loop]] - как async/await интегрируется в модель событий JavaScript
- [[Обработка-ошибок-в-асинхронном-коде]] - особенности обработки ошибок с async/await