---
aliases: [Промисы, Promise, Асинхронные промисы]
tags: [javascript, async-programming, promises, frontend]
---

# Promises (Промисы)

## Описание

Promise (промис) - это объект, представляющий результат асинхронной операции. Промисы были введены в JavaScript для решения проблемы "Callback Hell" и упрощения обработки асинхронных операций.

## Состояния промиса

Промис может находиться в одном из трех состояний:
- **Pending (Ожидание)** - начальное состояние, промис не завершен
- **Fulfilled (Выполнен)** - операция завершена успешно
- **Rejected (Отклонен)** - операция завершена с ошибкой

## Применение

Промисы позволяют более читаемо и последовательно обрабатывать асинхронные операции, особенно при цепочечных вызовах.

### Создание промиса

```javascript
// Создание промиса
const myPromise = new Promise((resolve, reject) => {
    // Асинхронная операция
    setTimeout(() => {
        const success = true;
        
        if (success) {
            resolve('Операция выполнена успешно!');
        } else {
            reject(new Error('Произошла ошибка'));
        }
    }, 1000);
});
```

### Использование промиса

```javascript
// Использование .then() для обработки результата
myPromise
    .then(result => {
        console.log(result); // 'Операция выполнена успешно!'
    })
    .catch(error => {
        console.error(error.message); // Обработка ошибки
    });
```

### Цепочки промисов

```javascript
// Цепочка промисов
fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => {
        console.log(data);
        return processData(data);
    })
    .then(processedData => {
        console.log('Обработанные данные:', processedData);
        return saveData(processedData);
    })
    .then(result => {
        console.log('Данные сохранены:', result);
    })
    .catch(error => {
        console.error('Ошибка в цепочке:', error);
    });
```

## Методы промисов

### Promise.all()

Выполняет все промисы параллельно и возвращает массив результатов, или отклоняется, если хотя бы один промис отклонен.

```javascript
const promise1 = fetch('/api/users');
const promise2 = fetch('/api/posts');
const promise3 = fetch('/api/comments');

Promise.all([promise1, promise2, promise3])
    .then(responses => {
        console.log('Все запросы выполнены:', responses);
    })
    .catch(error => {
        console.error('Ошибка в одном из запросов:', error);
    });
```

### Promise.allSettled()

Ждет завершения всех промисов (независимо от успеха или ошибки) и возвращает результаты всех.

```javascript
Promise.allSettled([promise1, promise2, promise3])
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Промис ${index} выполнен:`, result.value);
            } else {
                console.log(`Промис ${index} отклонен:`, result.reason);
            }
        });
    });
```

### Promise.race()

Возвращает результат первого завершившегося промиса (независимо от успеха или ошибки).

```javascript
const timeout = new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Таймаут')), 5000)
);

Promise.race([
    fetch('/api/data'),
    timeout
])
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Запрос не выполнен:', error.message));
```

### Promise.any()

Возвращает результат первого успешно выполненного промиса, или AggregateError, если все промисы отклонены.

```javascript
Promise.any([
    fetch('/api/server1'),
    fetch('/api/server2'),
    fetch('/api/server3')
])
.then(response => response.json())
.then(data => console.log('Данные с первого доступного сервера:', data))
.catch(error => console.error('Все серверы недоступны:', error));
```

## Практические советы

1. **Используйте .catch()** для обработки ошибок в цепочках промисов
2. **Избегайте лишних вложений** - используйте возврат промисов для построения цепочек
3. **Не забывайте обрабатывать ошибки** - даже если вы не ожидаете их
4. **Используйте Promise.all() с осторожностью** - если один промис отклоняется, все остальные также отклоняются
5. **Рассмотрите async/await** для более читаемого кода

## Связанные темы

- [[Callbacks]] - предшественник промисов в асинхронном программировании
- [[Async-Await]] - более удобный синтаксис для работы с промисами
- [[Event-Loop]] - как промисы интегрируются в модель событий JavaScript
- [[Обработка-ошибок-в-асинхронном-коде]] - особенности обработки ошибок в промисах