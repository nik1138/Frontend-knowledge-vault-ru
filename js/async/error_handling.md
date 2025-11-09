# Обработка ошибок в асинхронном JavaScript

## Обзор

Обработка ошибок в асинхронном JavaScript требует специального подхода, так как стандартные блоки try/catch не работают с асинхронными операциями, которые выполняются после возврата из функции.

## Продвинутые стратегии обработки ошибок

### Классификация асинхронных ошибок

```javascript
// Создание специфичных классов ошибок для лучшей обработки
class NetworkError extends Error {
    constructor(message, statusCode = null) {
        super(message);
        this.name = 'NetworkError';
        this.statusCode = statusCode;
        this.isRetryable = [500, 502, 503, 504].includes(statusCode) || !statusCode;
    }
}

class ValidationError extends Error {
    constructor(message, field = null) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
    }
}

class TimeoutError extends Error {
    constructor(message = 'Operation timed out') {
        super(message);
        this.name = 'TimeoutError';
    }
}

// Использование специфичных ошибок
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new NetworkError(
                `HTTP ${response.status}: ${response.statusText}`, 
                response.status
            );
        }
        
        const userData = await response.json();
        
        // Валидация данных
        if (!userData.email || !userData.email.includes('@')) {
            throw new ValidationError('Invalid email format', 'email');
        }
        
        return userData;
    } catch (error) {
        if (error instanceof NetworkError) {
            console.error('Сетевая ошибка:', error.message);
            if (error.isRetryable) {
                // Может потребоваться повторная попытка
                console.log('Ошибка может быть повторена');
            }
        } else if (error instanceof ValidationError) {
            console.error('Ошибка валидации:', error.message, 'в поле:', error.field);
        } else {
            console.error('Неизвестная ошибка:', error);
        }
        
        throw error; // Пробрасываем ошибку дальше
    }
}
```

### Централизованная обработка ошибок

```javascript
// Централизованный обработчик ошибок
class ErrorHandler {
    constructor() {
        this.handlers = new Map();
        this.globalHandler = null;
    }
    
    // Регистрация специфичного обработчика
    registerHandler(errorType, handler) {
        this.handlers.set(errorType, handler);
    }
    
    // Установка глобального обработчика
    setGlobalHandler(handler) {
        this.globalHandler = handler;
    }
    
    // Обработка ошибки
    async handle(error, context = {}) {
        // Поиск специфичного обработчика
        for (const [errorType, handler] of this.handlers) {
            if (error instanceof errorType) {
                return await handler(error, context);
            }
        }
        
        // Использование глобального обработчика
        if (this.globalHandler) {
            return await this.globalHandler(error, context);
        }
        
        // Если нет обработчиков, логируем ошибку
        console.error('Необработанная ошибка:', error, 'Контекст:', context);
        throw error;
    }
}

// Создание обработчика ошибок
const errorHandler = new ErrorHandler();

// Регистрация обработчиков
errorHandler.registerHandler(NetworkError, async (error, context) => {
    console.log(`Сетевая ошибка в ${context.operation}:`, error.message);
    
    // Логика обработки сетевой ошибки
    if (error.isRetryable && context.retryCount < 3) {
        console.log('Планируем повторную попытку...');
        return { retry: true, delay: Math.pow(2, context.retryCount) * 1000 };
    }
    
    // Отправка в систему мониторинга
    await logErrorToService(error, context);
    return { errorHandled: true, fallback: context.fallback || null };
});

errorHandler.registerHandler(ValidationError, async (error, context) => {
    console.log(`Ошибка валидации в ${context.operation}:`, error.message);
    
    // Отправка в систему мониторинга
    await logErrorToService(error, context);
    return { errorHandled: true, userMessage: error.message };
});

// Глобальный обработчик
errorHandler.setGlobalHandler(async (error, context) => {
    console.error('Неизвестная ошибка:', error);
    await logErrorToService(error, context);
    return { errorHandled: true, userMessage: 'Произошла неизвестная ошибка' };
});

// Вспомогательная функция для логирования ошибок
async function logErrorToService(error, context) {
    try {
        await fetch('/api/errors', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                message: error.message,
                stack: error.stack,
                name: error.name,
                context,
                timestamp: new Date().toISOString()
            })
        });
    } catch (loggingError) {
        console.error('Ошибка при логировании ошибки:', loggingError);
    }
}

// Использование обработчика ошибок
async function safeOperation(operation, context = {}) {
    try {
        return await operation();
    } catch (error) {
        const result = await errorHandler.handle(error, {
            ...context,
            operation: operation.name || 'unknown'
        });
        
        if (result.retry) {
            await new Promise(resolve => setTimeout(resolve, result.delay));
            context.retryCount = (context.retryCount || 0) + 1;
            return await safeOperation(operation, context);
        }
        
        if (result.fallback !== undefined) {
            return result.fallback;
        }
        
        throw error;
    }
}
```

### Обработка ошибок с контекстом

```javascript
// Класс для отслеживания контекста ошибки
class ErrorContext {
    constructor(initialContext = {}) {
        this.context = {
            timestamp: new Date().toISOString(),
            stack: new Error().stack,
            ...initialContext
        };
    }
    
    add(key, value) {
        this.context[key] = value;
        return this;
    }
    
    addAsync(key, asyncValueProvider) {
        return asyncValueProvider().then(value => {
            this.context[key] = value;
            return this;
        });
    }
    
    get() {
        return { ...this.context };
    }
    
    extend(additionalContext) {
        return new ErrorContext({ ...this.context, ...additionalContext });
    }
}

// Пример использования контекста ошибки
async function fetchUserWithRichContext(userId) {
    const context = new ErrorContext({ userId, operation: 'fetchUser' })
        .add('startTime', Date.now())
        .add('userAgent', navigator.userAgent);
    
    try {
        // Добавление асинхронного контекста
        await context.addAsync('userLocation', async () => {
            // Имитация получения геолокации
            return { lat: 55.7558, lng: 37.6176 };
        });
        
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new NetworkError(
                `HTTP ${response.status}: ${response.statusText}`,
                response.status
            );
        }
        
        const user = await response.json();
        context.add('endTime', Date.now());
        context.add('duration', Date.now() - context.get().startTime);
        
        console.log('Пользователь успешно получен:', user);
        return user;
    } catch (error) {
        context.add('error', error.message)
               .add('endTime', Date.now())
               .add('duration', Date.now() - context.get().startTime);
        
        console.error('Ошибка получения пользователя:', error.message, 'Контекст:', context.get());
        
        // Отправка полного контекста в систему мониторинга
        await logErrorWithContext(error, context.get());
        throw error;
    }
}

async function logErrorWithContext(error, context) {
    try {
        await fetch('/api/errors', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ error, context })
        });
    } catch (loggingError) {
        console.error('Ошибка при логировании:', loggingError);
    }
}
```

## Обработка ошибок с Promise

### Продвинутая обработка с Promise.allSettled

```javascript
// Обработка результатов Promise.allSettled с детализированной обработкой ошибок
async function fetchMultipleResourcesWithDetailedErrorHandling(urls) {
    const promises = urls.map(url => fetch(url).then(res => res.json()));
    const results = await Promise.allSettled(promises);
    
    const successfulResults = [];
    const failedResults = [];
    
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            successfulResults.push({
                url: urls[index],
                data: result.value,
                index
            });
        } else {
            failedResults.push({
                url: urls[index],
                error: result.reason,
                index,
                errorType: result.reason.constructor.name
            });
        }
    });
    
    // Обработка ошибок с разными стратегиями
    for (const failed of failedResults) {
        if (failed.error instanceof TypeError && failed.error.message.includes('fetch')) {
            console.warn(`Сетевая ошибка для ${failed.url}:`, failed.error.message);
            // Может быть попытка с резервного источника
        } else if (failed.error instanceof SyntaxError) {
            console.error(`Ошибка парсинга JSON для ${failed.url}:`, failed.error.message);
            // Может быть попытка с другого формата
        } else {
            console.error(`Неизвестная ошибка для ${failed.url}:`, failed.error);
        }
    }
    
    return {
        successful: successfulResults,
        failed: failedResults,
        all: results,
        hasErrors: failedResults.length > 0
    };
}

// Использование
async function loadDashboardData() {
    const urls = [
        '/api/users',
        '/api/posts', 
        '/api/analytics',
        '/api/settings'
    ];
    
    const results = await fetchMultipleResourcesWithDetailedErrorHandling(urls);
    
    console.log(`Успешно загружено: ${results.successful.length} из ${urls.length}`);
    
    if (results.hasErrors) {
        console.warn(`Неудачно: ${results.failed.length}`, results.failed);
        
        // Попытка восстановления с резервными данными
        for (const failed of results.failed) {
            try {
                const fallbackData = await getFallbackData(failed.url);
                results.successful.push({
                    url: failed.url,
                    data: fallbackData,
                    index: failed.index,
                    source: 'fallback'
                });
            } catch (fallbackError) {
                console.error(`Не удалось получить резервные данные для ${failed.url}:`, fallbackError);
            }
        }
    }
    
    return results.successful;
}

async function getFallbackData(url) {
    // В реальном приложении это может быть локальное хранилище или кэш
    const fallbackMap = {
        '/api/users': [],
        '/api/posts': [],
        '/api/analytics': { views: 0, visitors: 0 },
        '/api/settings': {}
    };
    
    return fallbackMap[url] || {};
}
```

### Обработка ошибок с транзакционной семантикой

```javascript
// Класс для транзакционной обработки асинхронных операций
class AsyncTransaction {
    constructor() {
        this.operations = [];
        this.rollbackOperations = [];
        this.completed = false;
    }
    
    addOperation(operation, rollbackOperation = null) {
        this.operations.push(operation);
        this.rollbackOperations.push(rollbackOperation);
        return this;
    }
    
    async execute() {
        const results = [];
        
        try {
            for (let i = 0; i < this.operations.length; i++) {
                const operation = this.operations[i];
                const result = await operation();
                results.push(result);
            }
            
            this.completed = true;
            return results;
        } catch (error) {
            console.error('Ошибка в транзакции, выполнение отката:', error);
            await this.rollback();
            throw error;
        }
    }
    
    async rollback() {
        // Выполнение операций отката в обратном порядке
        for (let i = this.rollbackOperations.length - 1; i >= 0; i--) {
            const rollbackOp = this.rollbackOperations[i];
            if (rollbackOp) {
                try {
                    await rollbackOp();
                    console.log(`Откат операции ${i} выполнен`);
                } catch (rollbackError) {
                    console.error(`Ошибка при откате операции ${i}:`, rollbackError);
                }
            }
        }
    }
}

// Пример использования транзакции
async function createUserWithProfileAndSettings(userId) {
    const transaction = new AsyncTransaction();
    
    // Добавление операций с соответствующими операциями отката
    transaction
        .addOperation(
            () => fetch('/api/users', {
                method: 'POST',
                body: JSON.stringify({ id: userId, name: 'New User' })
            }).then(res => res.json()),
            () => fetch(`/api/users/${userId}`, { method: 'DELETE' }) // Операция отката
        )
        .addOperation(
            () => fetch('/api/profiles', {
                method: 'POST', 
                body: JSON.stringify({ userId, preferences: {} })
            }).then(res => res.json()),
            () => fetch(`/api/profiles/${userId}`, { method: 'DELETE' })
        )
        .addOperation(
            () => fetch('/api/settings', {
                method: 'POST',
                body: JSON.stringify({ userId, theme: 'light' })
            }).then(res => res.json()),
            () => fetch(`/api/settings/${userId}`, { method: 'DELETE' })
        );
    
    try {
        const results = await transaction.execute();
        console.log('Пользователь успешно создан:', results);
        return results;
    } catch (error) {
        console.error('Не удалось создать пользователя:', error);
        throw error;
    }
}
```

### Обработка ошибок с retry-логикой

```javascript
// Продвинутая логика повторных попыток с различными стратегиями
class RetryStrategy {
    static exponential(baseDelay = 1000, maxDelay = 30000, factor = 2) {
        return (attempt) => {
            const delay = Math.min(baseDelay * Math.pow(factor, attempt - 1), maxDelay);
            // Добавляем jitter для предотвращения "thundering herd"
            return delay * (0.5 + Math.random() * 0.5);
        };
    }
    
    static linear(fixedDelay = 1000) {
        return () => fixedDelay;
    }
    
    static fibonacci(multiplier = 1000, maxDelay = 30000) {
        return (attempt) => {
            const fib = [1, 1];
            for (let i = 2; i < attempt; i++) {
                fib[i] = fib[i - 1] + fib[i - 2];
            }
            return Math.min(fib[attempt - 1] * multiplier, maxDelay);
        };
    }
}

class RetryableOperation {
    constructor(operation, options = {}) {
        this.operation = operation;
        this.maxAttempts = options.maxAttempts || 3;
        this.retryStrategy = options.retryStrategy || RetryStrategy.exponential();
        this.shouldRetry = options.shouldRetry || (() => true);
        this.onRetry = options.onRetry || (() => {});
    }
    
    async execute() {
        let lastError;
        
        for (let attempt = 1; attempt <= this.maxAttempts; attempt++) {
            try {
                return await this.operation();
            } catch (error) {
                lastError = error;
                
                if (attempt === this.maxAttempts || !this.shouldRetry(error, attempt)) {
                    throw error;
                }
                
                const delay = this.retryStrategy(attempt);
                console.log(`Попытка ${attempt} не удалась, следующая через ${delay}мс:`, error.message);
                
                await this.onRetry(error, attempt, delay);
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
        
        throw lastError;
    }
}

// Пример использования продвинутой retry-логики
async function fetchWithAdvancedRetry(url, options = {}) {
    const retryableOp = new RetryableOperation(
        () => fetch(url, options).then(res => {
            if (!res.ok) {
                throw new NetworkError(`HTTP ${res.status}`, res.status);
            }
            return res.json();
        }),
        {
            maxAttempts: 5,
            retryStrategy: RetryStrategy.exponential(500, 10000),
            shouldRetry: (error, attempt) => {
                // Повторяем только для определенных типов ошибок
                return error instanceof NetworkError && error.isRetryable && attempt < 5;
            },
            onRetry: async (error, attempt, delay) => {
                console.info(`Подготовка к попытке ${attempt + 1} через ${delay}мс`);
                // Можно добавить логирование или уведомления
            }
        }
    );
    
    return retryableOp.execute();
}

// Использование
async function loadCriticalData() {
    try {
        const data = await fetchWithAdvancedRetry('/api/critical-data');
        console.log('Критические данные загружены:', data);
        return data;
    } catch (error) {
        console.error('Не удалось загрузить критические данные:', error);
        // Могут быть дополнительные действия: уведомления, резервные источники и т.д.
        throw error;
    }
}

### Цепочки Promise и обработка ошибок

```javascript
// Обработка ошибок в цепочке Promise
function processUserData(userId) {
    return fetch(`/api/users/${userId}`)
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP ошибка! Статус: ${response.status}`);
            }
            return response.json();
        })
        .then(user => {
            // Может выбросить ошибку
            if (!user.email) {
                throw new Error('У пользователя отсутствует email');
            }
            return user;
        })
        .then(user => {
            // Дополнительная обработка
            return {
                ...user,
                emailDomain: user.email.split('@')[1]
            };
        })
        .catch(error => {
            console.error('Ошибка в цепочке обработки:', error);
            // Можно вернуть значение по умолчанию
            return { error: true, message: error.message };
        });
}
```

## Обработка ошибок с async/await

### Базовая обработка с try/catch

```javascript
// Обработка ошибок с async/await
async function fetchUserDataAsync(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка! Статус: ${response.status}`);
        }
        
        const user = await response.json();
        
        if (!user.email) {
            throw new Error('У пользователя отсутствует email');
        }
        
        return user;
    } catch (error) {
        console.error('Ошибка при получении данных пользователя:', error);
        throw error; // Пробрасываем ошибку дальше
    }
}

// Использование
async function handleUserRequest() {
    try {
        const user = await fetchUserDataAsync(123);
        console.log('Пользователь:', user);
    } catch (error) {
        console.error('Не удалось получить пользователя:', error);
        // Можно показать сообщение пользователю
        showErrorMessage('Не удалось загрузить данные пользователя');
    }
}
```

### Повторные попытки (retry) с обработкой ошибок

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await fetch(url, options);
            
            if (!response.ok) {
                throw new Error(`HTTP ошибка! Статус: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            lastError = error;
            console.warn(`Попытка ${attempt} не удалась:`, error.message);
            
            if (attempt < maxRetries) {
                // Ждем перед следующей попыткой (экспоненциальная задержка)
                await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
            }
        }
    }
    
    // Все попытки не удались
    throw new Error(`Не удалось выполнить запрос после ${maxRetries} попыток. Последняя ошибка: ${lastError.message}`);
}

// Использование
async function loadData() {
    try {
        const data = await fetchWithRetry('/api/data');
        console.log('Данные успешно загружены:', data);
    } catch (error) {
        console.error('Не удалось загрузить данные:', error.message);
    }
}
```

## Глобальная обработка ошибок

### Обработка неперехваченных Promise ошибок

```javascript
// Обработка неперехваченных ошибок Promise
window.addEventListener('unhandledrejection', event => {
    console.error('Неперехваченная ошибка Promise:', event.reason);
    
    // Предотвращаем стандартное поведение (вывод в консоль)
    event.preventDefault();
    
    // Можно отправить ошибку в систему логирования
    logErrorToService(event.reason);
});

// Обработка ошибок выполнения JavaScript
window.addEventListener('error', event => {
    console.error('Ошибка JavaScript:', event.error);
    
    // Можно отправить ошибку в систему логирования
    logErrorToService(event.error);
});

function logErrorToService(error) {
    // Отправка ошибки на сервер для анализа
    fetch('/api/errors', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            message: error.message,
            stack: error.stack,
            url: window.location.href,
            timestamp: new Date().toISOString()
        })
    }).catch(logError => {
        console.error('Не удалось отправить ошибку в систему логирования:', logError);
    });
}
```

## Практические примеры обработки ошибок

### Обработка ошибок в цепочке асинхронных операций

```javascript
class UserService {
    static async getUserWithFallback(userId) {
        // Сначала пытаемся получить данные из основного API
        try {
            return await this.fetchFromMainApi(userId);
        } catch (primaryError) {
            console.warn('Ошибка основного API:', primaryError.message);
            
            // Если основной API не работает, пробуем резервный
            try {
                console.log('Попытка получения данных из резервного источника...');
                return await this.fetchFromBackupApi(userId);
            } catch (backupError) {
                console.error('Ошибка резервного API:', backupError.message);
                
                // Если оба API не работают, пробуем кеш
                try {
                    console.log('Попытка получения данных из кеша...');
                    return await this.fetchFromCache(userId);
                } catch (cacheError) {
                    console.error('Ошибка получения данных из кеша:', cacheError.message);
                    
                    // Все источники не работают, пробрасываем ошибку
                    throw new Error(`Не удалось получить данные пользователя из всех источников. Основная ошибка: ${primaryError.message}`);
                }
            }
        }
    }
    
    static async fetchFromMainApi(userId) {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error(`Основной API: ${response.status}`);
        return response.json();
    }
    
    static async fetchFromBackupApi(userId) {
        const response = await fetch(`/backup-api/users/${userId}`);
        if (!response.ok) throw new Error(`Резервный API: ${response.status}`);
        return response.json();
    }
    
    static async fetchFromCache(userId) {
        // Имитация получения из кеша
        const cached = localStorage.getItem(`user_${userId}`);
        if (!cached) throw new Error('Данные не найдены в кеше');
        return JSON.parse(cached);
    }
}
```

### Создание универсального обработчика ошибок

```javascript
class ErrorHandler {
    static async handleAsyncOperation(operation, options = {}) {
        const {
            retries = 0,
            delay = 1000,
            onError = null,
            onSuccess = null,
            maxDelay = 10000
        } = options;
        
        let attempt = 0;
        
        while (true) {
            try {
                const result = await operation();
                
                if (onSuccess) {
                    onSuccess(result);
                }
                
                return result;
            } catch (error) {
                attempt++;
                
                if (onError) {
                    onError(error, attempt);
                }
                
                // Проверяем, нужно ли повторять попытку
                if (attempt <= retries) {
                    // Экспоненциальная задержка
                    const currentDelay = Math.min(delay * Math.pow(2, attempt - 1), maxDelay);
                    console.log(`Повторная попытка через ${currentDelay}мс...`);
                    await new Promise(resolve => setTimeout(resolve, currentDelay));
                } else {
                    // Все попытки исчерпаны
                    throw error;
                }
            }
        }
    }
}

// Использование универсального обработчика
async function fetchUserDataWithHandler(userId) {
    return await ErrorHandler.handleAsyncOperation(
        () => fetch(`/api/users/${userId}`).then(res => {
            if (!res.ok) throw new Error(`HTTP ${res.status}`);
            return res.json();
        }),
        {
            retries: 3,
            delay: 1000,
            onError: (error, attempt) => {
                console.warn(`Попытка ${attempt} не удалась:`, error.message);
            },
            onSuccess: (result) => {
                console.log('Данные успешно получены:', result);
            }
        }
    );
}
```

## Типы асинхронных ошибок

### Ошибки сети

```javascript
async function handleNetworkError() {
    try {
        const response = await fetch('/api/data');
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        return await response.json();
    } catch (error) {
        if (error instanceof TypeError && error.message.includes('fetch')) {
            // Ошибка сети (например, нет подключения)
            console.error('Ошибка сети:', error.message);
            return { error: 'network', message: 'Проверьте подключение к интернету' };
        } else if (error.message.includes('HTTP')) {
            // HTTP ошибка
            console.error('HTTP ошибка:', error.message);
            return { error: 'http', message: error.message };
        } else {
            // Другая ошибка
            console.error('Неизвестная ошибка:', error);
            return { error: 'unknown', message: error.message };
        }
    }
}
```

### Ошибки таймаута

```javascript
function timeoutPromise(promise, ms) {
    return new Promise((resolve, reject) => {
        const timer = setTimeout(() => {
            reject(new Error(`Таймаут операции: ${ms}мс`));
        }, ms);
        
        promise
            .then(value => {
                clearTimeout(timer);
                resolve(value);
            })
            .catch(error => {
                clearTimeout(timer);
                reject(error);
            });
    });
}

// Использование с таймаутом
async function fetchWithTimeout(url, timeoutMs = 5000) {
    try {
        const response = await timeoutPromise(fetch(url), timeoutMs);
        return await response.json();
    } catch (error) {
        if (error.message.includes('Таймаут')) {
            console.error('Таймаут запроса:', error.message);
            return { error: 'timeout', message: error.message };
        }
        throw error;
    }
}
```