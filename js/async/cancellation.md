# Отмена асинхронных операций в JavaScript

## Обзор

Отмена асинхронных операций - важная концепция в современном JavaScript, особенно при работе с долгими запросами, обработке пользовательского ввода или при навигации между страницами. До появления AbortController API отмена асинхронных операций была сложной задачей, требующей кастомных решений.

## AbortController API

### Основы AbortController

AbortController - это встроенный API, который позволяет создавать сигналы отмены для асинхронных операций:

```javascript
// Создание контроллера отмены
const controller = new AbortController();
const signal = controller.signal;

// Использование сигнала в fetch-запросе
fetch('/api/data', { signal })
    .then(response => response.json())
    .then(data => console.log('Данные:', data))
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Запрос был отменен');
        } else {
            console.error('Ошибка запроса:', error);
        }
    });

// Отмена операции через 2 секунды
setTimeout(() => {
    controller.abort();
    console.log('Операция отменена');
}, 2000);
```

### Отмена с обработчиком событий

```javascript
const controller = new AbortController();
const signal = controller.signal;

// Добавление обработчика события abort
signal.addEventListener('abort', () => {
    console.log('Операция была отменена!');
});

fetch('/api/data', { signal })
    .then(response => {
        if (!response.ok) throw new Error('Сервер вернул ошибку');
        return response.json();
    })
    .then(data => {
        if (signal.aborted) {
            console.log('Операция была отменена после получения данных');
            return;
        }
        console.log('Данные:', data);
    })
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Запрос был отменен пользователем');
        } else {
            console.error('Ошибка:', error);
        }
    });

// Отмена через 1.5 секунды
setTimeout(() => controller.abort(), 1500);
```

## Отмена Promise вручную

### Создание отменяемого Promise

```javascript
function createCancelablePromise(promiseFactory) {
    let hasCanceled = false;

    const wrappedPromise = new Promise((resolve, reject) => {
        promiseFactory()
            .then(val => hasCanceled ? reject({ isCanceled: true }) : resolve(val))
            .catch(error => hasCanceled ? reject({ isCanceled: true }) : reject(error));
    });

    return {
        promise: wrappedPromise,
        cancel() {
            hasCanceled = true;
        }
    };
}

// Использование отменяемого Promise
const { promise, cancel } = createCancelablePromise(() => 
    fetch('/api/data').then(res => res.json())
);

promise
    .then(data => console.log('Данные:', data))
    .catch(error => {
        if (error.isCanceled) {
            console.log('Операция была отменена');
        } else {
            console.error('Ошибка:', error);
        }
    });

// Отмена через 1 секунду
setTimeout(() => cancel(), 1000);
```

### Класс отменяемого Promise

```javascript
class CancelablePromise {
    constructor(executor) {
        this._isCanceled = false;
        this._promise = new Promise((resolve, reject) => {
            const rejector = (reason) => {
                if (!this._isCanceled) {
                    reject(reason);
                } else {
                    reject({ isCanceled: true });
                }
            };

            const resolver = (value) => {
                if (!this._isCanceled) {
                    resolve(value);
                } else {
                    reject({ isCanceled: true });
                }
            };

            const onCancel = (handler) => {
                this._onCancel = handler;
            };

            executor(resolver, rejector, onCancel);
        });
    }

    then(onFulfilled, onRejected) {
        return new CancelablePromise((resolve, reject) => {
            this._promise.then(
                (value) => {
                    if (onFulfilled) {
                        try {
                            resolve(onFulfilled(value));
                        } catch (error) {
                            reject(error);
                        }
                    } else {
                        resolve(value);
                    }
                },
                (error) => {
                    if (onRejected) {
                        try {
                            resolve(onRejected(error));
                        } catch (err) {
                            reject(err);
                        }
                    } else {
                        reject(error);
                    }
                }
            );
        });
    }

    catch(onRejected) {
        return this.then(undefined, onRejected);
    }

    finally(onFinally) {
        return this._promise.finally(onFinally);
    }

    cancel() {
        this._isCanceled = true;
        if (this._onCancel) {
            this._onCancel();
        }
    }

    get promise() {
        return this._promise;
    }
}

// Использование класса CancelablePromise
const cancelablePromise = new CancelablePromise((resolve, reject) => {
    const timeoutId = setTimeout(() => {
        resolve('Операция завершена');
    }, 3000);

    // Регистрация обработчика отмены
    this.onCancel(() => {
        clearTimeout(timeoutId);
        console.log('Таймаут отменен');
    });
});

cancelablePromise
    .then(result => console.log(result))
    .catch(error => {
        if (error.isCanceled) {
            console.log('Promise был отменен');
        } else {
            console.error('Ошибка:', error);
        }
    });

// Отмена через 1 секунду
setTimeout(() => cancelablePromise.cancel(), 1000);
```

## Отмена с использованием async/await

### Отмена fetch-запроса

```javascript
async function fetchWithCancellation(url, timeout = 5000) {
    const controller = new AbortController();
    
    // Установка таймаута для отмены
    const timeoutId = setTimeout(() => controller.abort(), timeout);

    try {
        const response = await fetch(url, { signal: controller.signal });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const data = await response.json();
        clearTimeout(timeoutId); // Отмена таймаута при успешном завершении
        
        return data;
    } catch (error) {
        clearTimeout(timeoutId);
        
        if (error.name === 'AbortError') {
            console.log('Запрос был отменен');
            throw new Error('Запрос был отменен по таймауту');
        }
        
        throw error;
    }
}

// Использование
async function loadUserData(userId) {
    try {
        const data = await fetchWithCancellation(`/api/users/${userId}`, 2000);
        console.log('Данные пользователя:', data);
    } catch (error) {
        console.error('Ошибка загрузки данных:', error.message);
    }
}
```

### Отмена последовательных операций

```javascript
class RequestManager {
    constructor() {
        this.activeControllers = new Map();
    }

    async makeRequest(id, url) {
        // Отмена предыдущего запроса с тем же ID
        if (this.activeControllers.has(id)) {
            this.activeControllers.get(id).abort();
            this.activeControllers.delete(id);
        }

        const controller = new AbortController();
        this.activeControllers.set(id, controller);

        try {
            const response = await fetch(url, { signal: controller.signal });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            const data = await response.json();
            this.activeControllers.delete(id); // Удаление после успешного завершения
            
            return data;
        } catch (error) {
            this.activeControllers.delete(id); // Удаление при ошибке
            
            if (error.name === 'AbortError') {
                console.log(`Запрос ${id} был отменен`);
                return null;
            }
            
            throw error;
        }
    }

    cancelRequest(id) {
        if (this.activeControllers.has(id)) {
            this.activeControllers.get(id).abort();
            this.activeControllers.delete(id);
        }
    }

    cancelAll() {
        for (const [id, controller] of this.activeControllers) {
            controller.abort();
        }
        this.activeControllers.clear();
    }
}

// Использование менеджера запросов
const requestManager = new RequestManager();

// Пример отмены предыдущих запросов при новом запросе
async function searchUsers(query) {
    // Каждый новый запрос отменяет предыдущий
    return await requestManager.makeRequest('search', `/api/users?q=${query}`);
}

// При вводе текста пользователем
document.getElementById('searchInput').addEventListener('input', async (event) => {
    const query = event.target.value.trim();
    if (query) {
        try {
            const results = await searchUsers(query);
            if (results) {
                displayResults(results);
            }
        } catch (error) {
            console.error('Ошибка поиска:', error);
        }
    }
});
```

## Отмена с debounce и throttle

### Отмена с debounce

```javascript
function debounceWithCancellation(fn, delay, signal) {
    let timeoutId = null;

    const debouncedFn = function(...args) {
        return new Promise((resolve, reject) => {
            if (timeoutId) {
                clearTimeout(timeoutId);
            }

            if (signal && signal.aborted) {
                reject(new Error('Операция отменена'));
                return;
            }

            timeoutId = setTimeout(() => {
                if (signal && signal.aborted) {
                    reject(new Error('Операция отменена'));
                } else {
                    try {
                        const result = fn.apply(this, args);
                        resolve(result);
                    } catch (error) {
                        reject(error);
                    }
                }
            }, delay);

            // Слушатель события отмены
            if (signal) {
                signal.addEventListener('abort', () => {
                    clearTimeout(timeoutId);
                    reject(new Error('Операция отменена'));
                }, { once: true });
            }
        });
    };

    return debouncedFn;
}

// Использование debounce с отменой
const debouncedSearch = debounceWithCancellation(
    async (query) => {
        const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
        return response.json();
    },
    300
);

// Пример использования в поиске
const controller = new AbortController();

document.getElementById('searchInput').addEventListener('input', async (event) => {
    try {
        const results = await debouncedSearch(event.target.value, controller.signal);
        displaySearchResults(results);
    } catch (error) {
        if (error.message === 'Операция отменена') {
            console.log('Поиск отменен');
        } else {
            console.error('Ошибка поиска:', error);
        }
    }
});

// Отмена всех операций при выходе со страницы
window.addEventListener('beforeunload', () => {
    controller.abort();
});
```

## Практические примеры отмены

### Отмена загрузки файлов

```javascript
class FileUploader {
    constructor() {
        this.activeUploads = new Map();
    }

    async uploadFile(file, onProgress = null) {
        const controller = new AbortController();
        const uploadId = `${file.name}-${Date.now()}`;
        
        this.activeUploads.set(uploadId, controller);

        const formData = new FormData();
        formData.append('file', file);

        try {
            const response = await fetch('/api/upload', {
                method: 'POST',
                body: formData,
                signal: controller.signal,
                onUploadProgress: onProgress ? (progressEvent) => {
                    const progress = Math.round((progressEvent.loaded * 100) / progressEvent.total);
                    onProgress(progress);
                } : undefined
            });

            if (!response.ok) {
                throw new Error(`Загрузка не удалась: ${response.status}`);
            }

            const result = await response.json();
            this.activeUploads.delete(uploadId);
            
            return result;
        } catch (error) {
            this.activeUploads.delete(uploadId);
            
            if (error.name === 'AbortError') {
                console.log('Загрузка файла была отменена');
                throw new Error('Загрузка была отменена пользователем');
            }
            
            throw error;
        }
    }

    cancelUpload(uploadId) {
        if (this.activeUploads.has(uploadId)) {
            this.activeUploads.get(uploadId).abort();
            this.activeUploads.delete(uploadId);
            console.log('Загрузка отменена');
        }
    }

    cancelAllUploads() {
        for (const [id, controller] of this.activeUploads) {
            controller.abort();
        }
        this.activeUploads.clear();
    }
}
```

### Отмена WebSocket-соединений

```javascript
class ManagedWebSocket {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
        this.reconnectDelay = 1000;
        this.shouldReconnect = true;
        this.controller = new AbortController();
    }

    async connect() {
        return new Promise((resolve, reject) => {
            this.ws = new WebSocket(this.url);
            
            this.ws.onopen = () => {
                this.reconnectAttempts = 0;
                console.log('WebSocket соединение установлено');
                resolve();
            };

            this.ws.onmessage = (event) => {
                // Проверяем, не отменено ли соединение
                if (this.controller.signal.aborted) {
                    return;
                }
                this.handleMessage(event.data);
            };

            this.ws.onerror = (error) => {
                console.error('Ошибка WebSocket:', error);
                reject(error);
            };

            this.ws.onclose = (event) => {
                if (this.controller.signal.aborted) {
                    console.log('WebSocket соединение закрыто по отмене');
                    return;
                }

                if (this.shouldReconnect && this.reconnectAttempts < this.maxReconnectAttempts) {
                    this.reconnectAttempts++;
                    console.log(`Попытка переподключения... (попытка ${this.reconnectAttempts})`);
                    
                    setTimeout(() => {
                        this.connect().catch(console.error);
                    }, this.reconnectDelay * this.reconnectAttempts);
                } else {
                    console.log('WebSocket соединение закрыто');
                }
            };
        });
    }

    handleMessage(data) {
        try {
            const message = JSON.parse(data);
            console.log('Получено сообщение:', message);
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
        }
    }

    send(message) {
        if (this.ws && this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(message));
        } else {
            console.error('WebSocket не подключен');
        }
    }

    disconnect() {
        this.shouldReconnect = false;
        this.controller.abort();
        
        if (this.ws) {
            this.ws.close();
        }
    }
}