# Promise в JavaScript

## Обзор

Promise (обещание) - это объект, представляющий окончательное завершение или неудачу асинхронной операции. Promise предоставляет способ взаимодействия с асинхронными операциями без блокировки основного потока выполнения.

## Состояния Promise

Promise может находиться в одном из трех состояний:

- `pending` (ожидание) - начальное состояние, операция еще не завершена
- `fulfilled` (выполнено) - операция завершена успешно
- `rejected` (отклонено) - операция завершена с ошибкой

## Продвинутые концепции Promise

### Промисификация функций

Промисификация - это процесс преобразования функций, использующих колбэки, в функции, возвращающие Promise:

```javascript
// Промисификация функции с колбэком
function promisify(asyncFunction) {
    return function(...args) {
        return new Promise((resolve, reject) => {
            asyncFunction.call(this, ...args, (error, result) => {
                if (error) {
                    reject(error);
                } else {
                    resolve(result);
                }
            });
        });
    };
}

// Пример использования с Node.js API
const fs = require('fs');
const readFile = promisify(fs.readFile);

readFile('file.txt', 'utf8')
    .then(content => console.log(content))
    .catch(error => console.error(error));
```

### Promise.finally()

Метод `finally()` позволяет зарегистрировать колбэк, который выполнится независимо от того, был ли Promise разрешен или отклонен:

```javascript
let isLoading = false;

function fetchUserData(userId) {
    isLoading = true;
    
    return fetch(`/api/users/${userId}`)
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            return response.json();
        })
        .finally(() => {
            isLoading = false;
            console.log('Загрузка завершена');
        });
}

// Использование
fetchUserData(123)
    .then(user => console.log('Пользователь:', user))
    .catch(error => console.error('Ошибка:', error));
```

### Цепочки Promise и возврат значений

В цепочке Promise возвращаемое значение из `.then()` становится значением следующего Promise:

```javascript
// Пример сложной цепочки с преобразованием данных
fetch('/api/users')
    .then(response => response.json())                    // Возвращает Promise с JSON
    .then(users => users.filter(user => user.active))    // Возвращает отфильтрованный массив
    .then(activeUsers => ({                               // Возвращает объект с метаданными
        count: activeUsers.length,
        users: activeUsers,
        timestamp: Date.now()
    }))
    .then(data => {
        console.log('Обработанные данные:', data);
        return data; // Возвращаем данные для следующего .then()
    })
    .then(processedData => {
        // Здесь мы получаем результат предыдущего .then()
        console.log('Финальные данные:', processedData);
    })
    .catch(error => {
        console.error('Ошибка в цепочке:', error);
    });
```

### Продвинутая обработка ошибок в цепочках

```javascript
// Стратегии обработки ошибок в цепочках Promise
function advancedErrorHandling() {
    return fetch('/api/data')
        .then(response => {
            if (!response.ok) {
                // Создание специфичной ошибки для конкретного типа проблемы
                if (response.status === 404) {
                    throw new Error('Данные не найдены');
                } else if (response.status === 403) {
                    throw new Error('Нет доступа к данным');
                } else {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
            }
            return response.json();
        })
        .then(data => {
            // Проверка структуры данных
            if (!data || !Array.isArray(data.items)) {
                throw new Error('Неверный формат данных');
            }
            return data.items;
        })
        .catch(error => {
            // Логирование ошибки
            console.error('Ошибка получения данных:', error);
            
            // Возврат значения по умолчанию вместо проброса ошибки
            return []; // Позволяет цепочке продолжиться
        })
        .then(items => {
            // Эта часть выполнится даже при ошибке выше
            console.log('Количество элементов:', items.length);
            return items;
        });
}

// Использование
advancedErrorHandling()
    .then(items => console.log('Финальный результат:', items))
    .catch(error => console.error('Непредвиденная ошибка:', error));
```

### Создание Promise с обработкой ошибок

```javascript
// Создание Promise с встроенной обработкой ошибок
function createSafePromise(executor) {
    return new Promise((resolve, reject) => {
        try {
            executor(resolve, reject);
        } catch (error) {
            reject(error);
        }
    });
}

// Пример использования
const safePromise = createSafePromise((resolve, reject) => {
    // Опасная операция, которая может выбросить исключение
    const riskyOperation = () => {
        if (Math.random() > 0.5) {
            resolve('Успех!');
        } else {
            throw new Error('Произошла ошибка');
        }
    };
    
    riskyOperation();
});

safePromise
    .then(result => console.log(result))
    .catch(error => console.error('Обработанная ошибка:', error.message));
```

## Стратегии обработки ошибок в Promise

### Стратегия "Fail Fast"

```javascript
// Прерывание цепочки при первой ошибке
function failFastStrategy() {
    return fetch('/api/users')
        .then(response => {
            if (!response.ok) throw new Error('Ошибка получения пользователей');
            return response.json();
        })
        .then(users => {
            if (users.length === 0) throw new Error('Нет пользователей');
            return users;
        })
        .then(users => fetch(`/api/profiles?userIds=${users.map(u => u.id).join(',')}`))
        .then(response => {
            if (!response.ok) throw new Error('Ошибка получения профилей');
            return response.json();
        });
}
```

### Стратегия "Graceful Degradation"

```javascript
// Продолжение выполнения с возвратом значений по умолчанию
function gracefulDegradationStrategy() {
    return fetch('/api/users')
        .then(response => {
            if (!response.ok) {
                console.warn('Ошибка получения пользователей, используем кэш');
                return getCachedUsers(); // Функция возвращает Promise
            }
            return response.json();
        })
        .then(users => {
            if (!users || users.length === 0) {
                console.warn('Нет пользователей, возвращаем пустой массив');
                return [];
            }
            return users;
        })
        .catch(error => {
            console.error('Критическая ошибка, возвращаем минимальные данные:', error);
            return getMinimalFallbackData(); // Функция возвращает Promise
        });
}

function getCachedUsers() {
    return new Promise(resolve => {
        const cached = localStorage.getItem('users');
        resolve(cached ? JSON.parse(cached) : []);
    });
}

function getMinimalFallbackData() {
    return Promise.resolve([{ id: 0, name: 'Гость', fallback: true }]);
}
```

### Стратегия "Retry with Backoff"

```javascript
// Повторные попытки с экспоненциальной задержкой
function fetchWithRetry(url, options = {}, maxRetries = 3) {
    let attempts = 0;
    
    function attempt() {
        attempts++;
        
        return fetch(url, options)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
                return response;
            })
            .catch(error => {
                if (attempts >= maxRetries) {
                    throw new Error(`Не удалось выполнить запрос после ${maxRetries} попыток: ${error.message}`);
                }
                
                // Экспоненциальная задержка: 1с, 2с, 4с и т.д.
                const delay = Math.pow(2, attempts - 1) * 1000;
                console.log(`Попытка ${attempts} не удалась, следующая через ${delay}мс:`, error.message);
                
                return new Promise(resolve => setTimeout(resolve, delay)).then(attempt);
            });
    }
    
    return attempt();
}

// Использование
fetchWithRetry('/api/data', {}, 3)
    .then(response => response.json())
    .then(data => console.log('Данные получены:', data))
    .catch(error => console.error('Неудача после всех попыток:', error));
```

### Стратегия "Timeout"

```javascript
// Добавление таймаута к Promise
function withTimeout(promise, timeoutMs) {
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error(`Таймаут операции: ${timeoutMs}мс`)), timeoutMs);
    });
    
    return Promise.race([promise, timeoutPromise]);
}

// Пример использования
withTimeout(fetch('/api/slow-endpoint'), 5000)
    .then(response => response.json())
    .then(data => console.log('Данные:', data))
    .catch(error => {
        if (error.message.includes('Таймаут')) {
            console.error('Операция превысила время ожидания');
        } else {
            console.error('Другая ошибка:', error);
        }
    });
```

### Стратегия "Circuit Breaker"

```javascript
// Паттерн Circuit Breaker для предотвращения каскадных сбоев
class CircuitBreaker {
    constructor(threshold = 5, timeout = 60000) {
        this.threshold = threshold;      // Количество неудач перед отключением
        this.timeout = timeout;          // Время до повторной попытки
        this.failureCount = 0;           // Счетчик неудач
        this.lastFailureTime = null;     // Время последней неудачи
        this.state = 'CLOSED';           // CLOSED, OPEN, HALF_OPEN
    }
    
    async call(promiseFactory) {
        // Проверка необходимости переключения в HALF_OPEN состояние
        if (this.state === 'OPEN' && Date.now() - this.lastFailureTime > this.timeout) {
            this.state = 'HALF_OPEN';
        }
        
        // Если цепь разомкнута, сразу возвращаем ошибку
        if (this.state === 'OPEN') {
            throw new Error('Circuit breaker is OPEN');
        }
        
        try {
            const result = await promiseFactory();
            
            // Успешное выполнение сбрасывает счетчик
            if (this.state === 'HALF_OPEN') {
                console.log('Circuit breaker CLOSED after successful call');
            }
            
            this.failureCount = 0;
            this.state = 'CLOSED';
            
            return result;
        } catch (error) {
            this.failureCount++;
            this.lastFailureTime = Date.now();
            
            // Если порог превышен, переключаем в OPEN состояние
            if (this.failureCount >= this.threshold) {
                this.state = 'OPEN';
                console.log('Circuit breaker OPENED due to failures');
            } else if (this.state === 'HALF_OPEN') {
                // Неудача в HALF_OPEN состоянии возвращает в OPEN
                this.state = 'OPEN';
                console.log('Circuit breaker OPENED after failed attempt');
            }
            
            throw error;
        }
    }
}

// Использование Circuit Breaker
const circuitBreaker = new CircuitBreaker(3, 10000); // 3 неудачи, 10с таймаут

async function protectedCall(url) {
    return circuitBreaker.call(() => fetch(url).then(res => res.json()));
}

// Пример использования
protectedCall('/api/unstable-endpoint')
    .then(data => console.log('Данные:', data))
    .catch(error => console.error('Ошибка:', error.message));
```

### Комбинирование стратегий

```javascript
// Комбинирование нескольких стратегий обработки ошибок
class RobustPromiseHandler {
    constructor(options = {}) {
        this.maxRetries = options.maxRetries || 3;
        this.timeout = options.timeout || 10000;
        this.circuitBreaker = new CircuitBreaker(
            options.cbThreshold || 5, 
            options.cbTimeout || 60000
        );
    }
    
    async execute(promiseFactory) {
        // Обертываем в Circuit Breaker
        return this.circuitBreaker.call(async () => {
            let lastError;
            
            // Повторные попытки
            for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
                try {
                    // Добавляем таймаут
                    return await withTimeout(promiseFactory(), this.timeout);
                } catch (error) {
                    lastError = error;
                    console.warn(`Попытка ${attempt} не удалась:`, error.message);
                    
                    if (attempt < this.maxRetries) {
                        // Задержка между попытками
                        await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
                    }
                }
            }
            
            throw lastError;
        });
    }
}

// Использование комбинированного обработчика
const robustHandler = new RobustPromiseHandler({
    maxRetries: 3,
    timeout: 5000,
    cbThreshold: 3,
    cbTimeout: 30000
});

async function robustFetch(url) {
    return robustHandler.execute(() => fetch(url).then(res => res.json()));
}

// Пример использования
robustFetch('/api/reliable-endpoint')
    .then(data => console.log('Надежно полученные данные:', data))
    .catch(error => console.error('Не удалось получить данные:', error));
}

## Создание Promise

### Основной синтаксис

```javascript
const myPromise = new Promise((resolve, reject) => {
    // Асинхронная операция
    setTimeout(() => {
        const success = Math.random() > 0.5;
        
        if (success) {
            resolve("Операция выполнена успешно!");
        } else {
            reject(new Error("Операция завершена с ошибкой!"));
        }
    }, 1000);
});

// Использование Promise
myPromise
    .then(result => {
        console.log(result); // Вывод: "Операция выполнена успешно!" или ошибка
    })
    .catch(error => {
        console.error(error.message); // Вывод: "Операция завершена с ошибкой!"
    });
```

### Пример с фетч-запросом

```javascript
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        fetch(`https://api.example.com/users/${userId}`)
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return response.json();
            })
            .then(userData => {
                resolve(userData);
            })
            .catch(error => {
                reject(error);
            });
    });
}

// Использование
fetchUserData(123)
    .then(user => {
        console.log('Пользователь:', user);
    })
    .catch(error => {
        console.error('Ошибка загрузки пользователя:', error);
    });
```

## Цепочки Promise

```javascript
function step1() {
    return new Promise(resolve => {
        setTimeout(() => {
            console.log('Шаг 1 завершен');
            resolve('Результат шага 1');
        }, 1000);
    });
}

function step2(result) {
    return new Promise(resolve => {
        setTimeout(() => {
            console.log('Шаг 2 завершен, получено:', result);
            resolve('Результат шага 2');
        }, 1000);
    });
}

function step3(result) {
    return new Promise(resolve => {
        setTimeout(() => {
            console.log('Шаг 3 завершен, получено:', result);
            resolve('Результат шага 3');
        }, 1000);
    });
}

// Цепочка Promise
step1()
    .then(result1 => step2(result1))
    .then(result2 => step3(result2))
    .then(result3 => {
        console.log('Все шаги завершены, финальный результат:', result3);
    })
    .catch(error => {
        console.error('Ошибка в цепочке:', error);
    });
```

## Методы Promise

### Promise.all()

```javascript
// Выполняет несколько Promise параллельно
const promise1 = fetch('/api/users');
const promise2 = fetch('/api/posts');
const promise3 = new Promise(resolve => setTimeout(() => resolve('Задача 3'), 1000));

Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log('Все Promise выполнены:', results);
        // results[0] - ответ от /api/users
        // results[1] - ответ от /api/posts
        // results[2] - 'Задача 3'
    })
    .catch(error => {
        console.error('Один из Promise завершился ошибкой:', error);
    });
```

### Promise.allSettled()

```javascript
// Ждет завершения всех Promise, независимо от успеха или ошибки
const promises = [
    Promise.resolve('Успешный результат'),
    Promise.reject(new Error('Ошибка')),
    Promise.resolve('Еще один успешный результат')
];

Promise.allSettled(promises)
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Promise ${index} выполнено:`, result.value);
            } else {
                console.log(`Promise ${index} отклонено:`, result.reason.message);
            }
        });
    });
```

### Promise.race()

```javascript
// Возвращает результат первого завершившегося Promise
const fastPromise = new Promise(resolve => setTimeout(() => resolve('Быстрый'), 100));
const slowPromise = new Promise(resolve => setTimeout(() => resolve('Медленный'), 1000));

Promise.race([fastPromise, slowPromise])
    .then(result => {
        console.log(result); // 'Быстрый'
    });
```

### Promise.any()

```javascript
// Возвращает результат первого успешно завершившегося Promise (ES2021+)
const promises = [
    Promise.reject(new Error('Ошибка 1')),
    Promise.reject(new Error('Ошибка 2')),
    Promise.resolve('Успех!'),
    Promise.resolve('Еще один успех')
];

Promise.any(promises)
    .then(result => {
        console.log(result); // 'Успех!'
    })
    .catch(errors => {
        console.error('Все Promise завершились ошибкой:', errors);
    });
```

## Обработка ошибок

```javascript
// Правильная обработка ошибок
function riskyOperation() {
    return new Promise((resolve, reject) => {
        // Симуляция асинхронной операции с возможной ошибкой
        setTimeout(() => {
            const shouldFail = Math.random() < 0.3;
            
            if (shouldFail) {
                reject(new Error('Произошла ошибка в операции'));
            } else {
                resolve('Операция выполнена успешно');
            }
        }, 500);
    });
}

// Использование с обработкой ошибок
riskyOperation()
    .then(result => {
        console.log('Успех:', result);
        return processResult(result); // Может вернуть другой Promise
    })
    .then(processedResult => {
        console.log('Обработанный результат:', processedResult);
    })
    .catch(error => {
        console.error('Произошла ошибка:', error.message);
        // Можно выполнить действия при ошибке
        logError(error);
    });

function processResult(result) {
    return new Promise(resolve => {
        setTimeout(() => resolve(`Обработано: ${result}`), 300);
    });
}

function logError(error) {
    console.error('Логирование ошибки:', error);
}
```

## Практические примеры

### Загрузка данных с повторными попытками

```javascript
function fetchWithRetry(url, maxRetries = 3) {
    return new Promise((resolve, reject) => {
        let attempts = 0;
        
        function attempt() {
            attempts++;
            
            fetch(url)
                .then(response => {
                    if (!response.ok) {
                        throw new Error(`HTTP error! status: ${response.status}`);
                    }
                    return response.json();
                })
                .then(data => {
                    resolve(data);
                })
                .catch(error => {
                    if (attempts < maxRetries) {
                        console.log(`Попытка ${attempts} не удалась, повтор...`);
                        setTimeout(attempt, 1000 * attempts); // Увеличиваем задержку
                    } else {
                        reject(new Error(`Не удалось выполнить запрос после ${maxRetries} попыток: ${error.message}`));
                    }
                });
        }
        
        attempt();
    });
}

// Использование
fetchWithRetry('https://api.example.com/data')
    .then(data => console.log('Данные получены:', data))
    .catch(error => console.error('Не удалось получить данные:', error));
```