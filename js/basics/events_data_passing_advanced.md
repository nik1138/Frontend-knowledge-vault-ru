# Продвинутые концепции передачи данных и событий JavaScript

## Обзор

Передача данных и управление событиями - ключевые аспекты разработки интерактивных и масштабируемых JavaScript-приложений. В этой статье мы рассмотрим продвинутые паттерны передачи данных, современные подходы к работе с событиями, архитектурные решения и лучшие практики для создания надежных и производительных приложений.

## Продвинутые паттерны передачи данных

### Передача данных через замыкания

```javascript
// Использование замыканий для инкапсуляции и передачи данных
function createDataChannel(initialData) {
    let data = initialData;
    const listeners = [];
    
    return {
        // Получение данных
        get() {
            return { ...data }; // Возвращаем копию для безопасности
        },
        
        // Установка данных
        set(newData) {
            const oldData = { ...data };
            data = { ...newData };
            
            // Уведомляем слушателей об изменении
            listeners.forEach(listener => {
                try {
                    listener(data, oldData);
                } catch (error) {
                    console.error('Ошибка в слушателе канала данных:', error);
                }
            });
        },
        
        // Подписка на изменения
        subscribe(listener) {
            listeners.push(listener);
            
            // Возвращаем функцию отписки
            return () => {
                const index = listeners.indexOf(listener);
                if (index > -1) {
                    listeners.splice(index, 1);
                }
            };
        },
        
        // Создание производного канала
        derive(transformer) {
            const derivedChannel = createDataChannel(transformer(data));
            
            // Обновляем производный канал при изменении исходного
            const unsubscribe = this.subscribe((newData) => {
                derivedChannel.set(transformer(newData));
            });
            
            // Расширяем производный канал методом отписки
            const originalUnsubscribe = derivedChannel.unsubscribe;
            derivedChannel.unsubscribe = () => {
                unsubscribe();
                if (originalUnsubscribe) originalUnsubscribe();
            };
            
            return derivedChannel;
        }
    };
}

// Использование
const userChannel = createDataChannel({ id: 1, name: 'Иван', active: true });

// Подписка на изменения
const unsubscribe = userChannel.subscribe((newData, oldData) => {
    console.log('Данные пользователя изменились:', oldData, '->', newData);
});

// Создание производного канала
const userNameChannel = userChannel.derive(user => user.name);
userNameChannel.subscribe(name => {
    console.log('Имя пользователя:', name);
});

// Изменение данных
userChannel.set({ id: 1, name: 'Иван Петров', active: true });
// Выведет:
// Данные пользователя изменились: { id: 1, name: 'Иван', active: true } -> { id: 1, name: 'Иван Петров', active: true }
// Имя пользователя: Иван Петров
```

### Передача данных через прототипы

```javascript
// Использование прототипов для передачи данных
function createDataPrototype(initialData) {
    const proto = {
        update(patch) {
            Object.assign(this, patch);
            if (this.onUpdate) {
                this.onUpdate(this);
            }
            return this;
        },
        
        extend(newProps) {
            return Object.create(this, Object.getOwnPropertyDescriptors(newProps));
        },
        
        toJSON() {
            const obj = {};
            for (const key in this) {
                if (this.hasOwnProperty(key) && key !== 'toJSON' && key !== 'update' && key !== 'extend' && key !== 'onUpdate') {
                    obj[key] = this[key];
                }
            }
            return obj;
        }
    };
    
    const instance = Object.create(proto);
    Object.assign(instance, initialData);
    
    return instance;
}

// Использование
const baseUser = createDataPrototype({
    id: 1,
    name: 'Иван',
    role: 'user'
});

const adminUser = baseUser.extend({
    role: 'admin',
    permissions: ['read', 'write', 'delete']
});

console.log(baseUser.toJSON()); // { id: 1, name: 'Иван', role: 'user' }
console.log(adminUser.toJSON()); // { id: 1, name: 'Иван', role: 'admin', permissions: ['read', 'write', 'delete'] }

// Обновление данных
adminUser.onUpdate = (data) => console.log('Данные обновлены:', data.toJSON());
adminUser.update({ lastLogin: new Date() });
```

### Передача данных через Proxy

```javascript
// Продвинутое использование Proxy для передачи и отслеживания данных
function createTrackedData(initialData) {
    const observers = new Set();
    const changeLog = [];
    
    const trackedData = new Proxy({ ...initialData }, {
        get(target, prop, receiver) {
            if (prop === 'subscribe') {
                return (observer) => {
                    observers.add(observer);
                    return () => observers.delete(observer);
                };
            }
            
            if (prop === 'getChangeLog') {
                return () => [...changeLog];
            }
            
            if (prop === 'resetChangeLog') {
                return () => {
                    changeLog.length = 0;
                };
            }
            
            return Reflect.get(target, prop, receiver);
        },
        
        set(target, prop, value, receiver) {
            const oldValue = target[prop];
            const result = Reflect.set(target, prop, value, receiver);
            
            // Логируем изменение
            changeLog.push({
                property: prop,
                oldValue,
                newValue: value,
                timestamp: Date.now()
            });
            
            // Уведомляем наблюдателей
            observers.forEach(observer => {
                try {
                    observer(prop, value, oldValue, target);
                } catch (error) {
                    console.error('Ошибка в наблюдателе:', error);
                }
            });
            
            return result;
        }
    });
    
    return trackedData;
}

// Использование
const trackedUser = createTrackedData({ name: 'Иван', age: 30 });

// Подписка на изменения
const unsubscribe = trackedUser.subscribe((prop, newValue, oldValue, obj) => {
    console.log(`Свойство ${prop} измено: ${oldValue} -> ${newValue}`);
});

trackedUser.name = 'Иван Петров';
trackedUser.age = 31;

console.log('Лог изменений:', trackedUser.getChangeLog());
```

## Современные подходы к работе с событиями

### Продвинутая система событий

```javascript
// Продвинутая система событий с middleware и фильтрацией
class AdvancedEventEmitter {
    constructor() {
        this.events = new Map();
        this.middleware = new Map(); // middleware для конкретных событий
        this.globalMiddleware = []; // глобальный middleware
        this.maxListeners = 10; // предотвращение утечек
    }
    
    // Подписка на событие
    on(event, listener, options = {}) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        
        const listeners = this.events.get(event);
        
        if (listeners.length >= this.maxListeners) {
            console.warn(`Превышено максимальное количество слушателей (${this.maxListeners}) для события ${event}`);
        }
        
        listeners.push({ 
            fn: listener, 
            once: options.once || false,
            priority: options.priority || 0 // для сортировки
        });
        
        return this; // для цепочки вызовов
    }
    
    // Однократная подписка
    once(event, listener, options = {}) {
        return this.on(event, listener, { ...options, once: true });
    }
    
    // Вызов события
    async emit(event, ...args) {
        const listeners = this.events.get(event) || [];
        
        // Применяем middleware
        const context = { event, args, cancelled: false };
        
        // Глобальный middleware
        for (const middleware of this.globalMiddleware) {
            await middleware(context);
            if (context.cancelled) return false;
        }
        
        // Middleware для конкретного события
        const eventMiddleware = this.middleware.get(event) || [];
        for (const middleware of eventMiddleware) {
            await middleware(context);
            if (context.cancelled) return false;
        }
        
        // Сортируем слушателей по приоритету
        const sortedListeners = [...listeners].sort((a, b) => b.priority - a.priority);
        
        // Вызываем слушателей
        const results = [];
        for (const [index, listener] of sortedListeners.entries()) {
            try {
                const result = await Promise.resolve(listener.fn.apply(this, args));
                results.push(result);
                
                // Если это однократный слушатель, удаляем его
                if (listener.once) {
                    sortedListeners.splice(index, 1);
                }
            } catch (error) {
                console.error(`Ошибка в слушателе события ${event}:`, error);
            }
        }
        
        return results;
    }
    
    // Добавление middleware
    use(event, middleware) {
        if (typeof event === 'function') {
            // Глобальный middleware
            this.globalMiddleware.push(event);
        } else {
            // Middleware для конкретного события
            if (!this.middleware.has(event)) {
                this.middleware.set(event, []);
            }
            this.middleware.get(event).push(middleware);
        }
        return this;
    }
    
    // Удаление слушателя
    off(event, listener) {
        if (!this.events.has(event)) return this;
        
        const listeners = this.events.get(event);
        const index = listeners.findIndex(l => l.fn === listener);
        
        if (index > -1) {
            listeners.splice(index, 1);
        }
        
        return this;
    }
    
    // Удаление всех слушателей для события
    removeAllListeners(event) {
        if (event) {
            this.events.delete(event);
            this.middleware.delete(event);
        } else {
            this.events.clear();
            this.middleware.clear();
        }
        return this;
    }
    
    // Получение количества слушателей
    listenerCount(event) {
        return this.events.has(event) ? this.events.get(event).length : 0;
    }
}

// Использование продвинутой системы событий
const emitter = new AdvancedEventEmitter();

// Добавление глобального middleware
emitter.use(async (context) => {
    console.log(`Глобальный middleware: событие ${context.event}`);
    if (context.event === 'forbidden') {
        context.cancelled = true;
    }
});

// Добавление middleware для конкретного события
emitter.use('user:login', async (context) => {
    console.log('Middleware для входа пользователя:', context.args);
    // Можно добавить валидацию, логирование и т.д.
});

// Подписка с приоритетом
emitter.on('user:login', (user) => {
    console.log('Высокий приоритет - аутентификация:', user);
}, { priority: 10 });

emitter.on('user:login', (user) => {
    console.log('Низкий приоритет - логирование:', user);
}, { priority: 1 });

emitter.on('user:login', (user) => {
    console.log('Средний приоритет - обновление интерфейса:', user);
}, { priority: 5 });

// Вызов события
emitter.emit('user:login', { id: 1, name: 'Иван' });
```

### События с типизацией и валидацией

```javascript
// Система событий с типизацией данных
class TypedEventEmitter {
    constructor() {
        this.events = new Map();
        this.eventSchemas = new Map(); // схемы валидации для событий
    }
    
    // Регистрация схемы для события
    registerSchema(event, schema) {
        this.eventSchemas.set(event, schema);
        return this;
    }
    
    // Подписка на событие
    on(event, listener) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(listener);
        return this;
    }
    
    // Вызов события с валидацией
    emit(event, data) {
        // Проверяем схему, если она зарегистрирована
        const schema = this.eventSchemas.get(event);
        if (schema) {
            const validation = this.validate(data, schema);
            if (!validation.valid) {
                console.error(`Валидация события ${event} не пройдена:`, validation.errors);
                return false;
            }
        }
        
        const listeners = this.events.get(event) || [];
        const results = [];
        
        for (const listener of listeners) {
            try {
                results.push(listener(data));
            } catch (error) {
                console.error(`Ошибка в слушателе события ${event}:`, error);
            }
        }
        
        return results;
    }
    
    // Простая валидация (в реальном приложении используйте Joi, Yup или аналоги)
    validate(data, schema) {
        const errors = [];
        
        for (const [field, rules] of Object.entries(schema)) {
            const value = data[field];
            
            if (rules.required && (value === undefined || value === null)) {
                errors.push(`Поле ${field} обязательно`);
                continue;
            }
            
            if (value !== undefined && value !== null) {
                if (rules.type && typeof value !== rules.type) {
                    errors.push(`Поле ${field} должно быть типа ${rules.type}`);
                }
                
                if (rules.min !== undefined && typeof value === 'number' && value < rules.min) {
                    errors.push(`Поле ${field} должно быть не меньше ${rules.min}`);
                }
                
                if (rules.validator && !rules.validator(value)) {
                    errors.push(`Поле ${field} не прошло пользовательскую валидацию`);
                }
            }
        }
        
        return {
            valid: errors.length === 0,
            errors
        };
    }
}

// Использование системы с типизацией
const typedEmitter = new TypedEventEmitter();

// Регистрация схемы для события входа пользователя
typedEmitter.registerSchema('user:login', {
    userId: { type: 'number', required: true },
    timestamp: { type: 'number', required: true },
    userAgent: { type: 'string', required: true }
});

// Подписка на событие
typedEmitter.on('user:login', (data) => {
    console.log('Пользователь вошел:', data);
});

// Корректный вызов события
typedEmitter.emit('user:login', {
    userId: 123,
    timestamp: Date.now(),
    userAgent: 'Mozilla/5.0...'
});

// Вызов события с неверными данными
typedEmitter.emit('user:login', {
    userId: 'not a number', // ошибка: должно быть число
    timestamp: Date.now()
});
```

### События с промисами и асинхронной обработкой

```javascript
// Система событий с асинхронной обработкой и ожиданием
class AsyncEventEmitter {
    constructor() {
        this.events = new Map();
        this.waiting = new Map(); // ожидание событий
    }
    
    on(event, listener) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(listener);
        return this;
    }
    
    async emit(event, data) {
        const listeners = this.events.get(event) || [];
        const promises = [];
        
        for (const listener of listeners) {
            promises.push(Promise.resolve(listener(data)));
        }
        
        const results = await Promise.allSettled(promises);
        
        // Обработка результатов
        const successful = [];
        const errors = [];
        
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                successful.push(result.value);
            } else {
                errors.push({
                    listener: listeners[index],
                    error: result.reason
                });
            }
        });
        
        // Уведомляем ожидающие Promise
        const waitingResolvers = this.waiting.get(event);
        if (waitingResolvers) {
            waitingResolvers.forEach(resolve => resolve({ data, successful, errors }));
            this.waiting.set(event, []);
        }
        
        return { successful, errors };
    }
    
    // Ожидание события
    waitFor(event, timeout = 10000) {
        return new Promise((resolve, reject) => {
            if (!this.waiting.has(event)) {
                this.waiting.set(event, []);
            }
            
            this.waiting.get(event).push(resolve);
            
            // Таймаут
            setTimeout(() => {
                const waiting = this.waiting.get(event);
                if (waiting) {
                    const index = waiting.indexOf(resolve);
                    if (index > -1) {
                        waiting.splice(index, 1);
                        reject(new Error(`Таймаут ожидания события ${event}`));
                    }
                }
            }, timeout);
        });
    }
    
    // Ожидание с фильтром
    waitForFilter(event, filterFn, timeout = 10000) {
        return new Promise((resolve, reject) => {
            const timeoutId = setTimeout(() => {
                this.off(event, listener);
                reject(new Error(`Таймаут ожидания события ${event} с фильтром`));
            }, timeout);
            
            const listener = (data) => {
                if (filterFn(data)) {
                    clearTimeout(timeoutId);
                    this.off(event, listener);
                    resolve(data);
                }
            };
            
            this.on(event, listener);
        });
    }
    
    off(event, listener) {
        if (this.events.has(event)) {
            const listeners = this.events.get(event);
            const index = listeners.indexOf(listener);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        }
        return this;
    }
}

// Использование асинхронной системы событий
const asyncEmitter = new AsyncEventEmitter();

// Подписка на асинхронное событие
asyncEmitter.on('data:processed', async (data) => {
    // Симуляция асинхронной обработки
    await new Promise(resolve => setTimeout(resolve, 100));
    console.log('Данные обработаны:', data);
    return `processed_${data.id}`;
});

asyncEmitter.on('data:processed', (data) => {
    console.log('Второй обработчик:', data);
    return `second_handler_${data.id}`;
});

// Асинхронная отправка события
async function processData() {
    const result = await asyncEmitter.emit('data:processed', { id: 1, content: 'test' });
    console.log('Результаты обработки:', result);
}

processData();

// Ожидание события
async function waitForSpecificEvent() {
    try {
        console.log('Ожидание события...');
        const result = await asyncEmitter.waitFor('user:action');
        console.log('Событие получено:', result);
    } catch (error) {
        console.error('Ошибка ожидания:', error.message);
    }
}

// Ожидание с фильтром
async function waitForFilteredEvent() {
    try {
        console.log('Ожидание события с фильтром...');
        const result = await asyncEmitter.waitForFilter('user:action', 
            data => data.type === 'click' && data.target === 'button');
        console.log('Отфильтрованное событие получено:', result);
    } catch (error) {
        console.error('Ошибка ожидания с фильтром:', error.message);
    }
}
```

## Архитектурные паттерны передачи данных

### Шаблон "Событийный шин" (Event Bus)

```javascript
// Продвинутая реализация Event Bus
class EventBus {
    constructor(options = {}) {
        this.channels = new Map();
        this.middleware = options.middleware || [];
        this.logger = options.logger || console;
        this.enableLogging = options.enableLogging || false;
    }
    
    // Подписка на канал
    subscribe(channel, handler, options = {}) {
        if (!this.channels.has(channel)) {
            this.channels.set(channel, []);
        }
        
        const subscription = {
            handler,
            id: options.id || Symbol('subscription'),
            filter: options.filter || null,
            transform: options.transform || null,
            context: options.context || this
        };
        
        this.channels.get(channel).push(subscription);
        
        if (this.enableLogging) {
            this.logger.log(`Подписка на канал "${channel}" добавлена`);
        }
        
        // Возвращаем функцию отписки
        return () => this.unsubscribe(channel, subscription.id);
    }
    
    // Отписка от канала
    unsubscribe(channel, subscriptionId) {
        if (this.channels.has(channel)) {
            const channelHandlers = this.channels.get(channel);
            const index = channelHandlers.findIndex(sub => sub.id === subscriptionId);
            
            if (index > -1) {
                channelHandlers.splice(index, 1);
                
                if (this.enableLogging) {
                    this.logger.log(`Подписка на канал "${channel}" удалена`);
                }
            }
        }
    }
    
    // Публикация в канал
    async publish(channel, data, metadata = {}) {
        if (this.enableLogging) {
            this.logger.log(`Публикация в канал "${channel}":`, data);
        }
        
        // Применяем глобальный middleware
        let processedData = data;
        for (const middleware of this.middleware) {
            processedData = await Promise.resolve(middleware(processedData, channel, metadata));
        }
        
        // Получаем подписчиков
        const subscribers = this.channels.get(channel) || [];
        const promises = [];
        
        for (const subscription of subscribers) {
            // Применяем фильтр, если есть
            if (subscription.filter && !subscription.filter(processedData, metadata)) {
                continue;
            }
            
            // Применяем трансформацию, если есть
            let payload = processedData;
            if (subscription.transform) {
                payload = subscription.transform(processedData, metadata);
            }
            
            // Вызываем обработчик
            promises.push(
                Promise.resolve()
                    .then(() => subscription.handler.call(subscription.context, payload, metadata))
                    .catch(error => {
                        this.logger.error(`Ошибка в обработчике канала "${channel}":`, error);
                    })
            );
        }
        
        return Promise.allSettled(promises);
    }
    
    // Публикация с ожиданием ответа (Request-Reply паттерн)
    async request(channel, data, timeout = 5000) {
        return new Promise(async (resolve, reject) => {
            const timeoutId = setTimeout(() => {
                this.unsubscribe(replyChannel, handler);
                reject(new Error(`Таймаут запроса к каналу "${channel}"`));
            }, timeout);
            
            const replyChannel = `reply:${Date.now()}:${Math.random()}`;
            const handler = (response) => {
                clearTimeout(timeoutId);
                this.unsubscribe(replyChannel, handler);
                resolve(response);
            };
            
            // Подписываемся на ответ
            this.subscribe(replyChannel, handler);
            
            // Публикуем запрос с указанием канала ответа
            await this.publish(channel, {
                ...data,
                replyTo: replyChannel
            });
        });
    }
    
    // Очистка всех подписок
    clear() {
        this.channels.clear();
        if (this.enableLogging) {
            this.logger.log('Все подписки очищены');
        }
    }
}

// Использование Event Bus
const eventBus = new EventBus({ enableLogging: true });

// Подписка с фильтрацией
const userSubscription = eventBus.subscribe('user', (data, metadata) => {
    console.log('Пользовательские данные:', data);
}, {
    filter: (data) => data.type === 'profile-update',
    id: 'user-profile-handler'
});

// Подписка с трансформацией
const auditSubscription = eventBus.subscribe('user', (data) => {
    console.log('Аудит события:', data);
}, {
    transform: (data) => ({
        ...data,
        timestamp: new Date().toISOString(),
        processedBy: 'audit-handler'
    }),
    id: 'audit-handler'
});

// Публикация события
eventBus.publish('user', {
    type: 'profile-update',
    userId: 123,
    changes: { name: 'Иван Петров' }
}, {
    source: 'profile-editor',
    priority: 'high'
});
```

### Шаблон "Хранилище состояния" (State Store)

```javascript
// Продвинутое хранилище состояния с реактивностью
class ReactiveStore {
    constructor(initialState = {}, options = {}) {
        this.state = this.createProxyState(initialState);
        this.listeners = new Map(); // подписчики на конкретные пути
        this.globalListeners = new Set(); // глобальные подписчики
        this.middleware = options.middleware || [];
        this.devTools = options.devTools || false;
        this.history = options.history ? [] : null;
        this.maxHistory = options.maxHistory || 50;
    }
    
    createProxyState(obj, path = '') {
        const store = this;
        
        return new Proxy(obj, {
            get(target, prop) {
                const value = target[prop];
                
                if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
                    return store.createProxyState(value, path ? `${path}.${prop}` : String(prop));
                }
                
                return value;
            },
            
            set(target, prop, value) {
                const oldValue = target[prop];
                
                if (oldValue !== value) {
                    // Применяем middleware
                    let processedValue = value;
                    for (const mw of store.middleware) {
                        processedValue = mw({
                            type: 'SET',
                            path: path ? `${path}.${prop}` : String(prop),
                            value: processedValue,
                            oldValue
                        }) || processedValue;
                    }
                    
                    target[prop] = processedValue;
                    
                    // Уведомляем подписчиков
                    const fullPath = path ? `${path}.${prop}` : String(prop);
                    store.notifyListeners(fullPath, processedValue, oldValue);
                    store.notifyGlobalListeners(fullPath, processedValue, oldValue);
                    
                    // Сохраняем в историю
                    if (store.history !== null) {
                        store.history.push({
                            path: fullPath,
                            value: processedValue,
                            oldValue,
                            timestamp: Date.now()
                        });
                        
                        if (store.history.length > store.maxHistory) {
                            store.history.shift();
                        }
                    }
                }
                
                return true;
            }
        });
    }
    
    notifyListeners(path, newValue, oldValue) {
        // Уведомляем подписчиков на конкретный путь
        if (this.listeners.has(path)) {
            this.listeners.get(path).forEach(listener => {
                try {
                    listener(newValue, oldValue, path);
                } catch (error) {
                    console.error(`Ошибка в слушателе пути "${path}":`, error);
                }
            });
        }
        
        // Уведомляем подписчиков на родительские пути
        const pathParts = path.split('.');
        for (let i = pathParts.length - 1; i > 0; i--) {
            const parentPath = pathParts.slice(0, i).join('.');
            if (this.listeners.has(parentPath)) {
                this.listeners.get(parentPath).forEach(listener => {
                    try {
                        listener(this.getNestedValue(parentPath), oldValue, path);
                    } catch (error) {
                        console.error(`Ошибка в слушателе родительского пути "${parentPath}":`, error);
                    }
                });
            }
        }
    }
    
    notifyGlobalListeners(path, newValue, oldValue) {
        this.globalListeners.forEach(listener => {
            try {
                listener(this.state, path, newValue, oldValue);
            } catch (error) {
                console.error('Ошибка в глобальном слушателе:', error);
            }
        });
    }
    
    getNestedValue(path) {
        return path.split('.').reduce((obj, prop) => obj && obj[prop], this.state);
    }
    
    // Подписка на изменения
    subscribe(path, listener) {
        if (!this.listeners.has(path)) {
            this.listeners.set(path, new Set());
        }
        
        this.listeners.get(path).add(listener);
        
        // Возвращаем функцию отписки
        return () => {
            const listeners = this.listeners.get(path);
            if (listeners) {
                listeners.delete(listener);
                if (listeners.size === 0) {
                    this.listeners.delete(path);
                }
            }
        };
    }
    
    // Подписка на все изменения
    subscribeGlobal(listener) {
        this.globalListeners.add(listener);
        
        return () => {
            this.globalListeners.delete(listener);
        };
    }
    
    // Получение значения по пути
    get(path) {
        return path.split('.').reduce((obj, prop) => obj && obj[prop], this.state);
    }
    
    // Установка значения по пути
    set(path, value) {
        const pathParts = path.split('.');
        const lastProp = pathParts.pop();
        const parent = pathParts.reduce((obj, prop) => {
            if (!obj[prop]) obj[prop] = {};
            return obj[prop];
        }, this.state);
        
        parent[lastProp] = value;
    }
    
    // Выборка данных с мемоизацией
    createSelector(selectorFn) {
        let lastResult;
        let lastArgs = [];
        
        return (...args) => {
            const argsChanged = args.length !== lastArgs.length || 
                args.some((arg, index) => arg !== lastArgs[index]);
                
            if (!argsChanged && lastResult !== undefined) {
                return lastResult;
            }
            
            lastResult = selectorFn(this.state, ...args);
            lastArgs = args;
            
            return lastResult;
        };
    }
}

// Использование реактивного хранилища
const store = new ReactiveStore({
    user: {
        profile: {
            name: 'Иван',
            email: 'ivan@example.com'
        },
        preferences: {
            theme: 'light',
            notifications: true
        }
    },
    app: {
        status: 'ready',
        version: '1.0.0'
    }
});

// Подписка на конкретный путь
const nameUnsubscribe = store.subscribe('user.profile.name', (newName, oldName) => {
    console.log(`Имя пользователя изменилось: ${oldName} -> ${newName}`);
});

// Подписка на все изменения в профиле
const profileUnsubscribe = store.subscribe('user.profile', (newProfile) => {
    console.log('Профиль обновлен:', newProfile);
});

// Глобальная подписка
const globalUnsubscribe = store.subscribeGlobal((state, path, newValue, oldValue) => {
    console.log(`Глобальное изменение: ${path} ${oldValue} -> ${newValue}`);
});

// Изменение данных
store.set('user.profile.name', 'Иван Петров'); // Вызовет соответствующие подписки
store.set('user.preferences.theme', 'dark');

// Создание селектора с мемоизацией
const getUserDisplayName = store.createSelector((state) => {
    const name = state.user?.profile?.name;
    const email = state.user?.profile?.email;
    return name ? `${name} (${email})` : 'Гость';
});

console.log('Отображаемое имя:', getUserDisplayName()); // "Иван Петров (ivan@example.com)"
```

## Практические примеры из промышленной разработки

### Интеграция с DOM-событиями

```javascript
// Продвинутая система интеграции событий с DOM
class DOMEventIntegrator {
    constructor() {
        this.eventMap = new Map(); // DOM элемент -> [события]
        this.handlerMap = new Map(); // событие -> [обработчики]
        this.middleware = [];
    }
    
    // Подписка на DOM событие с обработчиком
    subscribeDOM(element, eventType, handler, options = {}) {
        const eventKey = `${element}[${eventType}]`;
        const middleware = options.middleware || [];
        
        const wrappedHandler = (event) => {
            // Применяем middleware
            let processedEvent = event;
            for (const mw of this.middleware.concat(middleware)) {
                processedEvent = mw(processedEvent, element, eventType) || processedEvent;
            }
            
            // Вызываем оригинальный обработчик
            return handler(processedEvent);
        };
        
        // Добавляем обработчик к DOM элементу
        element.addEventListener(eventType, wrappedHandler, options);
        
        // Сохраняем для возможной отписки
        if (!this.eventMap.has(element)) {
            this.eventMap.set(element, new Set());
        }
        this.eventMap.get(element).add({ eventType, handler: wrappedHandler, options });
        
        // Возвращаем функцию отписки
        return () => {
            element.removeEventListener(eventType, wrappedHandler, options);
            if (this.eventMap.has(element)) {
                const events = this.eventMap.get(element);
                const eventObj = Array.from(events).find(e => e.handler === wrappedHandler);
                if (eventObj) {
                    events.delete(eventObj);
                }
            }
        };
    }
    
    // Подписка на пользовательское событие
    subscribe(eventType, handler) {
        if (!this.handlerMap.has(eventType)) {
            this.handlerMap.set(eventType, []);
        }
        this.handlerMap.get(eventType).push(handler);
        
        return () => {
            const handlers = this.handlerMap.get(eventType);
            const index = handlers.indexOf(handler);
            if (index > -1) {
                handlers.splice(index, 1);
            }
        };
    }
    
    // Вызов пользовательского события
    emit(eventType, data) {
        const handlers = this.handlerMap.get(eventType) || [];
        const results = [];
        
        for (const handler of handlers) {
            try {
                results.push(handler(data));
            } catch (error) {
                console.error(`Ошибка в обработчике события ${eventType}:`, error);
            }
        }
        
        return results;
    }
    
    // Делегирование событий
    delegate(container, selector, eventType, handler) {
        const delegator = (event) => {
            if (event.target.matches(selector)) {
                handler(event);
            }
        };
        
        container.addEventListener(eventType, delegator);
        
        return () => {
            container.removeEventListener(eventType, delegator);
        };
    }
    
    // Очистка всех событий для элемента
    clearElementEvents(element) {
        if (this.eventMap.has(element)) {
            const events = this.eventMap.get(element);
            for (const event of events) {
                element.removeEventListener(event.eventType, event.handler, event.options);
            }
            this.eventMap.delete(element);
        }
    }
}

// Использование DOMEventIntegrator
const domIntegrator = new DOMEventIntegrator();

// Создание элементов для примера
const container = document.createElement('div');
container.innerHTML = '<button class="btn">Кнопка 1</button><button class="btn">Кнопка 2</button>';

// Делегирование событий
const delegateUnsubscribe = domIntegrator.delegate(container, '.btn', 'click', (event) => {
    console.log('Клик на кнопке:', event.target.textContent);
    domIntegrator.emit('button:clicked', { 
        text: event.target.textContent, 
        timestamp: Date.now() 
    });
});

// Подписка на пользовательское событие
const eventUnsubscribe = domIntegrator.subscribe('button:clicked', (data) => {
    console.log('Событие клика обработано:', data);
});

// Middleware для логирования
const loggingMiddleware = (event, element, eventType) => {
    console.log(`DOM событие: ${eventType} на элементе`, element);
    return event;
};

domIntegrator.middleware.push(loggingMiddleware);
```

### Система уведомлений на событиях

```javascript
// Продвинутая система уведомлений
class NotificationSystem {
    constructor() {
        this.eventBus = new EventBus();
        this.notifications = new Map();
        this.notificationQueue = [];
        this.processingQueue = false;
        this.maxNotifications = 10;
        this.defaultTimeout = 5000;
    }
    
    // Отправка уведомления
    send(notification) {
        // Валидация уведомления
        if (!this.validateNotification(notification)) {
            console.error('Невалидное уведомление:', notification);
            return false;
        }
        
        // Уникализация ID
        notification.id = notification.id || Date.now() + Math.random();
        notification.timestamp = notification.timestamp || Date.now();
        notification.expires = notification.expires || 
            (notification.timeout !== undefined ? 
             Date.now() + notification.timeout : 
             Date.now() + this.defaultTimeout);
        
        // Добавление в очередь
        this.notificationQueue.push(notification);
        
        // Запуск обработки очереди
        this.processQueue();
        
        // Вызов события
        this.eventBus.publish('notification:sent', notification);
        
        return notification.id;
    }
    
    validateNotification(notification) {
        return notification && 
               typeof notification.type === 'string' && 
               typeof notification.message === 'string';
    }
    
    // Обработка очереди уведомлений
    async processQueue() {
        if (this.processingQueue) return;
        
        this.processingQueue = true;
        
        while (this.notificationQueue.length > 0) {
            const notification = this.notificationQueue.shift();
            
            // Проверка на дубликаты
            if (this.notifications.has(notification.id)) continue;
            
            // Ограничение количества уведомлений
            if (this.notifications.size >= this.maxNotifications) {
                // Удаляем старейшее уведомление
                const oldestId = this.notifications.keys().next().value;
                this.remove(oldestId);
            }
            
            // Добавляем уведомление
            this.notifications.set(notification.id, notification);
            
            // Уведомляем через событие
            this.eventBus.publish(`notification:${notification.type}`, notification);
            this.eventBus.publish('notification:added', notification);
            
            // Автоматическое удаление по таймауту
            if (notification.expires > Date.now()) {
                setTimeout(() => {
                    this.remove(notification.id);
                }, notification.expires - Date.now());
            }
        }
        
        this.processingQueue = false;
    }
    
    // Удаление уведомления
    remove(id) {
        if (this.notifications.has(id)) {
            const notification = this.notifications.get(id);
            this.notifications.delete(id);
            
            this.eventBus.publish('notification:removed', { id, ...notification });
        }
    }
    
    // Получение уведомлений
    getNotifications(filter = null) {
        let notifications = Array.from(this.notifications.values());
        
        if (filter) {
            notifications = notifications.filter(filter);
        }
        
        return notifications.sort((a, b) => b.timestamp - a.timestamp);
    }
    
    // Подписка на события уведомлений
    on(event, handler) {
        return this.eventBus.subscribe(event, handler);
    }
    
    // Очистка всех уведомлений
    clear() {
        const ids = [...this.notifications.keys()];
        for (const id of ids) {
            this.remove(id);
        }
        this.eventBus.clear();
    }
}

// Использование системы уведомлений
const notifier = new NotificationSystem();

// Подписка на различные типы событий
notifier.on('notification:info', (notification) => {
    console.log('Информационное уведомление:', notification.message);
});

notifier.on('notification:error', (notification) => {
    console.error('Ошибочное уведомление:', notification.message);
});

notifier.on('notification:added', (notification) => {
    console.log('Уведомление добавлено:', notification);
});

notifier.on('notification:removed', (notification) => {
    console.log('Уведомление удалено:', notification.id);
});

// Отправка уведомлений
notifier.send({
    type: 'info',
    message: 'Добро пожаловать!',
    timeout: 3000
});

notifier.send({
    type: 'error',
    message: 'Произошла ошибка при загрузке данных'
});

notifier.send({
    type: 'success',
    message: 'Операция выполнена успешно',
    timeout: 2000
});
```

## Лучшие практики и рекомендации

### Оптимизация производительности

```javascript
// Оптимизация систем событий и передачи данных
class OptimizedEventSystem {
    constructor() {
        this.events = new Map();
        this.eventQueue = [];
        this.isProcessing = false;
        this.batchTimeout = null;
        this.batchInterval = 16; // ~60fps
    }
    
    // Пакетная обработка событий
    emit(event, data) {
        this.eventQueue.push({ event, data });
        
        if (!this.batchTimeout) {
            this.batchTimeout = setTimeout(() => this.processBatch(), this.batchInterval);
        }
    }
    
    processBatch() {
        if (this.isProcessing) return;
        
        this.isProcessing = true;
        const queue = [...this.eventQueue];
        this.eventQueue.length = 0;
        this.batchTimeout = null;
        
        // Группировка событий по типу
        const groupedEvents = queue.reduce((acc, item) => {
            if (!acc[item.event]) acc[item.event] = [];
            acc[item.event].push(item.data);
            return acc;
        }, {});
        
        // Обработка сгруппированных событий
        for (const [event, dataList] of Object.entries(groupedEvents)) {
            const listeners = this.events.get(event) || [];
            
            for (const listener of listeners) {
                try {
                    listener(dataList); // передаем массив данных
                } catch (error) {
                    console.error(`Ошибка в слушателе события ${event}:`, error);
                }
            }
        }
        
        this.isProcessing = false;
    }
    
    on(event, listener) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(listener);
    }
    
    off(event, listener) {
        if (this.events.has(event)) {
            const listeners = this.events.get(event);
            const index = listeners.indexOf(listener);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        }
    }
}

// Использование оптимизированной системы
const optimizedEmitter = new OptimizedEventSystem();

optimizedEmitter.on('user:action', (actions) => {
    console.log(`Обработано ${actions.length} действий пользователя`);
    // Пакетная обработка действий
    actions.forEach(action => console.log('Действие:', action));
});

// Отправка множества событий
for (let i = 0; i < 100; i++) {
    optimizedEmitter.emit('user:action', { type: 'click', target: `button${i}` });
}
// Все 100 событий будут обработаны в одном пакете
```

### Безопасность и валидация данных

```javascript
// Система с безопасной передачей данных
class SecureDataTransfer {
    constructor() {
        this.validators = new Map();
        this.filters = new Map();
        this.encoders = new Map();
    }
    
    // Регистрация валидатора
    addValidator(event, validator) {
        if (!this.validators.has(event)) {
            this.validators.set(event, []);
        }
        this.validators.get(event).push(validator);
        return this;
    }
    
    // Регистрация фильтра
    addFilter(event, filter) {
        if (!this.filters.has(event)) {
            this.filters.set(event, []);
        }
        this.filters.get(event).push(filter);
        return this;
    }
    
    // Регистрация кодировщика
    addEncoder(event, encoder) {
        this.encoders.set(event, encoder);
        return this;
    }
    
    // Безопасная передача данных
    transfer(event, data) {
        // Применение фильтров
        let filteredData = data;
        const filters = this.filters.get(event) || [];
        for (const filter of filters) {
            filteredData = filter(filteredData);
        }
        
        // Валидация
        const validators = this.validators.get(event) || [];
        for (const validator of validators) {
            if (!validator(filteredData)) {
                throw new Error(`Валидация данных для события ${event} не пройдена`);
            }
        }
        
        // Кодирование при необходимости
        const encoder = this.encoders.get(event);
        if (encoder) {
            filteredData = encoder(filteredData);
        }
        
        return filteredData;
    }
}

// Использование безопасной передачи данных
const secureTransfer = new SecureDataTransfer();

// Добавление валидаторов
secureTransfer.addValidator('user:profile-update', (data) => {
    return data && 
           typeof data.name === 'string' && 
           data.name.length >= 2 && 
           data.name.length <= 50;
});

secureTransfer.addValidator('user:profile-update', (data) => {
    return data.email && 
           typeof data.email === 'string' && 
           /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email);
});

// Добавление фильтров
secureTransfer.addFilter('user:profile-update', (data) => {
    // Очистка потенциально опасных данных
    const sanitized = { ...data };
    delete sanitized.__proto__;
    delete sanitized.constructor;
    return sanitized;
});

// Пример использования
try {
    const safeData = secureTransfer.transfer('user:profile-update', {
        name: 'Иван',
        email: 'ivan@example.com',
        dangerous: '__proto__: { admin: true }' // будет удален фильтром
    });
    console.log('Безопасные данные:', safeData);
} catch (error) {
    console.error('Ошибка безопасности:', error.message);
}
```

## Заключение

Передача данных и управление событиями - это сложные, но критически важные аспекты разработки JavaScript-приложений. Современные подходы включают в себя не только базовую передачу данных, но и продвинутые паттерны, такие как реактивные системы, middleware, валидация данных, оптимизация производительности и безопасность. Понимание этих концепций позволяет создавать более надежные, производительные и поддерживаемые приложения.

#programming #javascript #events #data-passing #eventbus #state-management #reactive-programming #middleware #validation #security #performance #web-development #advanced-concepts