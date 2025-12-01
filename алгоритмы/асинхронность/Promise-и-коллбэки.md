---
aliases: ["Promise и коллбэки", "Promise", "Коллбэки", "Обработка асинхронных операций"]
tags: ["#programming", "#javascript", "#async", "#promise", "#callback", "#frontend", "#backend"]
---

# Promise-и-коллбэки

## Введение

Коллбэки и Promise - это два основных подхода к обработке асинхронных операций в JavaScript. Понимание этих концепций критически важно для разработчиков, особенно в 2025 году, когда асинхронное программирование стало стандартом для создания современных веб-приложений.

## Коллбэки (Callbacks)

Коллбэк - это функция, передаваемая в другую функцию в качестве аргумента, которая затем вызывается внутри внешней функции для выполнения определенной задачи.

### Простой коллбэк

```javascript
function greet(name, callback) {
    const greeting = `Привет, ${name}!`;
    callback(greeting);
}

greet('Иван', (message) => {
    console.log(message);
});
```

### Асинхронный коллбэк

```javascript
setTimeout(() => {
    console.log('Прошло 2 секунды');
}, 2000);
```

### Коллбэки с обработкой ошибок

```javascript
function fetchData(callback) {
    // Имитация асинхронной операции
    setTimeout(() => {
        const error = Math.random() > 0.5;
        if (error) {
            callback(new Error('Ошибка при загрузке данных'), null);
        } else {
            callback(null, { id: 1, name: 'Данные' });
        }
    }, 1000);
}

fetchData((error, data) => {
    if (error) {
        console.error('Ошибка:', error.message);
    } else {
        console.log('Данные:', data);
    }
});
```

### Проблема "callback hell"

Один из главных недостатков коллбэков - это "callback hell" или "pyramid of doom", когда вложенные коллбэки делают код трудночитаемым:

```javascript
// Пример callback hell
getData1(function(err, data1) {
    if (err) {
        console.error(err);
        return;
    }
    
    getData2(data1, function(err, data2) {
        if (err) {
            console.error(err);
            return;
        }
        
        getData3(data2, function(err, data3) {
            if (err) {
                console.error(err);
                return;
            }
            
            processData(data1, data2, data3, function(err, result) {
                if (err) {
                    console.error(err);
                    return;
                }
                
                console.log('Результат:', result);
            });
        });
    });
});
```

## Promise

Promise - это объект, представляющий окончательное завершение (или неудачу) асинхронной операции и её возвращаемое значение.

### Создание Promise

```javascript
const myPromise = new Promise((resolve, reject) => {
    // Асинхронная операция
    setTimeout(() => {
        const success = Math.random() > 0.5;
        if (success) {
            resolve('Операция выполнена успешно');
        } else {
            reject(new Error('Операция не удалась'));
        }
    }, 1000);
});

myPromise
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

### Состояния Promise

Promise может находиться в одном из трех состояний:
- **pending** (ожидание) - начальное состояние, ни выполнено, ни отклонено
- **fulfilled** (выполнено) - операция завершена успешно
- **rejected** (отклонено) - операция завершена с ошибкой

### Цепочка Promise

```javascript
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ id: userId, name: `Пользователь ${userId}` });
        }, 500);
    });
}

function fetchUserPosts(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve([
                { id: 1, userId, title: 'Пост 1' },
                { id: 2, userId, title: 'Пост 2' }
            ]);
        }, 500);
    });
}

fetchUserData(1)
    .then(user => {
        console.log('Пользователь:', user);
        return fetchUserPosts(user.id);
    })
    .then(posts => {
        console.log('Посты:', posts);
        return { user, posts };
    })
    .then(result => {
        console.log('Полный результат:', result);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

## Преобразование коллбэков в Promise

Многие старые API используют коллбэки. Их можно обернуть в Promise:

```javascript
// Функция с коллбэком
function legacyApiCall(data, callback) {
    // Имитация старого API
    setTimeout(() => {
        if (typeof data === 'string') {
            callback(null, `Обработано: ${data}`);
        } else {
            callback(new Error('Неверный формат данных'));
        }
    }, 1000);
}

// Обертка в Promise
function modernApiCall(data) {
    return new Promise((resolve, reject) => {
        legacyApiCall(data, (error, result) => {
            if (error) {
                reject(error);
            } else {
                resolve(result);
            }
        });
    });
}

// Использование
modernApiCall('данные')
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

## Встроенные методы Promise

### Promise.all

Выполняет все Promise параллельно и возвращает массив результатов:

```javascript
const promise1 = fetch('/api/data1');
const promise2 = fetch('/api/data2');
const promise3 = fetch('/api/data3');

Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log('Все результаты:', results);
    })
    .catch(error => {
        console.error('Ошибка в одном из Promise:', error);
    });
```

> [!warning] Важно
> Если хотя бы один из Promise в `Promise.all` отклоняется, то весь `Promise.all` отклоняется. Если это нежелательно, используйте `Promise.allSettled`.

### Promise.allSettled

Ожидает завершения всех Promise (как успешно, так и с ошибкой):

```javascript
Promise.allSettled([
    fetch('/api/valid-endpoint'),
    fetch('/api/invalid-endpoint'),
    Promise.resolve('Немедленный результат')
])
.then(results => {
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`Promise ${index} выполнен:`, result.value);
        } else {
            console.log(`Promise ${index} отклонен:`, result.reason);
        }
    });
});
```

### Promise.race

Возвращает результат первого завершившегося Promise:

```javascript
Promise.race([
    fetch('/api/fast'),
    fetch('/api/slow'),
    new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Таймаут')), 5000)
    )
])
.then(result => console.log('Первый результат:', result))
.catch(error => console.error('Ошибка или таймаут:', error));
```

### Promise.any

Возвращает результат первого успешно завершившегося Promise (ES2021):

```javascript
Promise.any([
    fetch('/api/server1').catch(() => { throw new Error('Сервер 1 недоступен'); }),
    fetch('/api/server2').catch(() => { throw new Error('Сервер 2 недоступен'); }),
    fetch('/api/server3').catch(() => { throw new Error('Сервер 3 недоступен'); })
])
.then(firstSuccessfulResult => {
    console.log('Первый успешный результат:', firstSuccessfulResult);
})
.catch(errors => {
    console.error('Все серверы недоступны:', errors);
});
```

## Современные практики использования Promise

### Использование async/await

Хотя это отдельная тема, важно понимать, что `async/await` - это синтаксический сахар над Promise:

```javascript
// С Promise
function fetchUserData(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => response.json())
        .then(user => user);
}

// С async/await
async function fetchUserData(userId) {
    const response = await fetch(`/api/users/${userId}`);
    const user = await response.json();
    return user;
}
```

### Обработка ошибок в Promise

```javascript
async function safeApiCall() {
    try {
        const response = await fetch('/api/data');
        if (!response.ok) {
            throw new Error(`HTTP ошибка! статус: ${response.status}`);
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка API вызова:', error);
        // Возвращаем значение по умолчанию или выбрасываем ошибку снова
        throw error;
    }
}
```

## Преимущества Promise перед коллбэками

1. **Лучшая обработка ошибок** - ошибки можно перехватывать в одном месте
2. **Избежание callback hell** - цепочки Promise делают код более читаемым
3. **Композиция** - легкое объединение нескольких асинхронных операций
4. **Стандартизация** - Promise соответствуют спецификации Promises/A+

## Практические рекомендации для российских разработчиков 2025 года

1. **Используйте Promise вместо коллбэков** - для новых проектов избегайте коллбэков в пользу Promise
2. **Обрабатывайте ошибки** - всегда добавляйте `.catch()` или используйте `try/catch` с `async/await`
3. **Используйте `Promise.all` для параллельных операций** - когда результаты не зависят друг от друга
4. **Используйте `Promise.allSettled` для необязательных операций** - когда нужно выполнить несколько операций, и не все должны быть успешными
5. **Добавляйте таймауты** - особенно при работе с внешними API, которые могут быть медленными

```javascript
// Пример с таймаутом для российских API
function fetchWithTimeout(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    return fetch(url, { signal: controller.signal })
        .then(response => {
            clearTimeout(timeoutId);
            return response;
        })
        .catch(error => {
            clearTimeout(timeoutId);
            if (error.name === 'AbortError') {
                throw new Error('Запрос превысил время ожидания');
            }
            throw error;
        });
}
```

## Заключение

Promise значительно улучшили обработку асинхронных операций в JavaScript, решив многие проблемы, связанные с коллбэками. В 2025 году знание и правильное использование Promise является обязательным навыком для любого JavaScript-разработчика.

## См. также

- [[Асинхронные-алгоритмы]]
- [[Обработка-ошибок-в-асинхронных-алгоритмах]]
- [[JavaScript современные возможности]]
- [[API запросы]]