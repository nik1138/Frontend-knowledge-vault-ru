# Передача данных и события в JavaScript

## Обзор

Передача данных и обработка событий - ключевые концепции в JavaScript, которые позволяют создавать интерактивные и динамические приложения. В этой статье мы рассмотрим различные методы передачи данных между функциями, объектами и компонентами, а также современные подходы к работе с событиями.

## Передача данных между функциями

### Передача по значению и по ссылке

В JavaScript примитивные типы передаются по значению, а объекты - по ссылке:

```javascript
// Передача примитивов (по значению)
function modifyPrimitive(value) {
    value = 100;
    console.log('Внутри функции:', value); // 100
}

let num = 5;
modifyPrimitive(num);
console.log('После функции:', num); // 5 (не изменилось)

// Передача объектов (по ссылке)
function modifyObject(obj) {
    obj.name = 'Измененный';
    obj.age = 100;
    console.log('Внутри функции:', obj); // { name: 'Измененный', age: 100 }
}

let person = { name: 'Иван', age: 30 };
modifyObject(person);
console.log('После функции:', person); // { name: 'Измененный', age: 100 } (изменилось!)

// Изменение ссылки vs изменение содержимого
function changeReference(obj) {
    obj = { name: 'Новый объект' }; // Изменяем ссылку, но не оригинальный объект
    console.log('Внутри функции:', obj); // { name: 'Новый объект' }
}

let original = { name: 'Оригинал' };
changeReference(original);
console.log('После функции:', original); // { name: 'Оригинал' } (не изменилось!)
```

### Копирование объектов

```javascript
// Поверхностное копирование
const originalObj = { a: 1, b: { c: 2 } };
const shallowCopy = { ...originalObj }; // Spread оператор
const shallowCopy2 = Object.assign({}, originalObj); // Object.assign

// Изменение вложенного объекта повлияет на обе копии
shallowCopy.b.c = 100;
console.log(originalObj.b.c); // 100 (изменилось в оригинале!)

// Глубокое копирование вручную
function deepCopy(obj) {
    if (obj === null || typeof obj !== 'object') return obj;
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof Array) return obj.map(item => deepCopy(item));
    if (typeof obj === 'object') {
        const copiedObj = {};
        for (let key in obj) {
            if (obj.hasOwnProperty(key)) {
                copiedObj[key] = deepCopy(obj[key]);
            }
        }
        return copiedObj;
    }
}

const deepOriginal = { a: 1, b: { c: 2 } };
const deepCopied = deepCopy(deepOriginal);
deepCopied.b.c = 100;
console.log(deepOriginal.b.c); // 2 (не изменилось в оригинале!)

// Глубокое копирование с JSON (ограничения)
const jsonOriginal = { a: 1, b: { c: 2 }, fn: () => {} };
const jsonCopy = JSON.parse(JSON.stringify(jsonOriginal));
// jsonCopy.fn будет undefined, так как функции не сериализуются
```

### Передача аргументов

```javascript
// Rest параметры
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// Деструктуризация аргументов
function createUser({ name, age, email = 'no-email' }) {
    return { name, age, email };
}

const user = createUser({ name: 'Иван', age: 30 });
console.log(user); // { name: 'Иван', age: 30, email: 'no-email' }

// Spread оператор для передачи аргументов
function multiply(a, b, c) {
    return a * b * c;
}

const numbers = [2, 3, 4];
console.log(multiply(...numbers)); // 24
```

## Современные методы передачи данных

### Callback-функции

```javascript
// Простой callback
function processData(data, callback) {
    // Симуляция обработки данных
    setTimeout(() => {
        const processed = data.map(item => item * 2);
        callback(null, processed);
    }, 100);
}

processData([1, 2, 3], (error, result) => {
    if (error) {
        console.error('Ошибка:', error);
    } else {
        console.log('Результат:', result); // [2, 4, 6]
    }
});

// Callback с замыканием для хранения состояния
function createCounter() {
    let count = 0;
    
    return function(callback) {
        count++;
        callback(count);
    };
}

const counter = createCounter();
counter(result => console.log('Счетчик:', result)); // Счетчик: 1
counter(result => console.log('Счетчик:', result)); // Счетчик: 2
```

### Promise и асинхронная передача данных

```javascript
// Создание Promise для передачи данных
function fetchData(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, name: `Пользователь ${id}` });
            } else {
                reject(new Error('Неверный ID'));
            }
        }, 100);
    });
}

// Использование Promise
fetchData(1)
    .then(user => {
        console.log('Получен пользователь:', user);
        return user.name; // Передаем данные дальше
    })
    .then(name => {
        console.log('Имя пользователя:', name);
        return name.length; // Передаем дальше
    })
    .then(length => {
        console.log('Длина имени:', length);
    })
    .catch(error => {
        console.error('Ошибка:', error.message);
    });

// Async/await для передачи данных
async function processUserData(userId) {
    try {
        const user = await fetchData(userId);
        const profile = await fetchUserProfile(user.id);
        const permissions = await fetchUserPermissions(user.id);
        
        return {
            ...user,
            profile,
            permissions
        };
    } catch (error) {
        console.error('Ошибка обработки данных пользователя:', error);
        throw error;
    }
}

async function fetchUserProfile(userId) {
    return new Promise(resolve => {
        setTimeout(() => resolve({ bio: 'Профиль пользователя' }), 100);
    });
}

async function fetchUserPermissions(userId) {
    return new Promise(resolve => {
        setTimeout(() => resolve(['read', 'write']), 100);
    });
}
```

### Генераторы для передачи данных

```javascript
// Генераторы для пошаговой передачи данных
function* dataProcessor(data) {
    for (const item of data) {
        const processed = yield item * 2;
        console.log(`Обработан элемент: ${processed}`);
    }
}

const processor = dataProcessor([1, 2, 3]);
let result = processor.next();
while (!result.done) {
    console.log(`Получено: ${result.value}`);
    result = processor.next(result.value); // Передаем данные обратно в генератор
}
```

## События и передача данных через события

### Основы событий

```javascript
// Создание и использование пользовательских событий
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }
    
    emit(event, ...args) {
        if (this.events[event]) {
            this.events[event].forEach(listener => listener(...args));
        }
    }
    
    off(event, listener) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(l => l !== listener);
        }
    }
}

// Использование
const emitter = new EventEmitter();

emitter.on('dataReceived', (data) => {
    console.log('Получены данные:', data);
});

emitter.on('dataReceived', (data) => {
    console.log('Второй обработчик:', data.length);
});

emitter.emit('dataReceived', { id: 1, name: 'Иван', items: [1, 2, 3] });
// Выведет:
// Получены данные: { id: 1, name: 'Иван', items: [1, 2, 3] }
// Второй обработчик: 3
```

### Передача данных через DOM-события

```javascript
// Создание события с данными
function createDataEvent(type, data) {
    return new CustomEvent(type, {
        detail: data,
        bubbles: true,
        cancelable: true
    });
}

// Пример использования
const button = document.createElement('button');
button.textContent = 'Нажми меня';

// Добавляем обработчик
button.addEventListener('dataEvent', (event) => {
    console.log('Полученные данные:', event.detail);
});

// Вызываем событие с данными
const data = { userId: 123, action: 'click', timestamp: Date.now() };
button.dispatchEvent(createDataEvent('dataEvent', data));

// Использование dataTransfer для drag and drop
function setupDragAndDrop() {
    const draggable = document.querySelector('.draggable');
    const dropZone = document.querySelector('.drop-zone');
    
    draggable.addEventListener('dragstart', (e) => {
        e.dataTransfer.setData('text/plain', JSON.stringify({
            id: 123,
            type: 'item',
            name: 'Перетаскиваемый элемент'
        }));
    });
    
    dropZone.addEventListener('drop', (e) => {
        e.preventDefault();
        const data = JSON.parse(e.dataTransfer.getData('text/plain'));
        console.log('Перетащенные данные:', data);
    });
    
    dropZone.addEventListener('dragover', (e) => {
        e.preventDefault();
    });
}
```

### Современные паттерны событий

```javascript
// Event Bus для передачи данных между компонентами
class EventBus {
    constructor() {
        this.listeners = new Map();
    }
    
    subscribe(event, callback) {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        this.listeners.get(event).add(callback);
        
        // Возвращаем функцию отписки
        return () => {
            this.listeners.get(event).delete(callback);
        };
    }
    
    publish(event, data) {
        if (this.listeners.has(event)) {
            this.listeners.get(event).forEach(callback => {
                try {
                    callback(data);
                } catch (error) {
                    console.error(`Ошибка в обработчике события ${event}:`, error);
                }
            });
        }
    }
}

// Использование EventBus
const eventBus = new EventBus();

// Подписка на события
const unsubscribe = eventBus.subscribe('user:login', (userData) => {
    console.log('Пользователь вошел:', userData);
    updateUI(userData);
});

eventBus.subscribe('user:logout', () => {
    console.log('Пользователь вышел');
    resetUI();
});

// Публикация событий
eventBus.publish('user:login', { id: 1, name: 'Иван', role: 'admin' });

// Отписка
unsubscribe(); // Больше не будет получать события 'user:login'

// События с промисами (ожидание события)
function waitForEvent(element, eventType, timeout = 5000) {
    return new Promise((resolve, reject) => {
        const timeoutId = setTimeout(() => {
            element.removeEventListener(eventType, handler);
            reject(new Error(`Таймаут ожидания события ${eventType}`));
        }, timeout);
        
        const handler = (event) => {
            clearTimeout(timeoutId);
            element.removeEventListener(eventType, handler);
            resolve(event);
        };
        
        element.addEventListener(eventType, handler);
    });
}

// Использование
async function handleButtonClick() {
    const button = document.querySelector('#myButton');
    try {
        const event = await waitForEvent(button, 'click', 10000);
        console.log('Кнопка была нажата:', event);
    } catch (error) {
        console.error('Кнопка не была нажата вовремя:', error.message);
    }
}
```

## Практические примеры из промышленной разработки

### Система уведомлений на событиях

```javascript
// Система уведомлений
class NotificationSystem {
    constructor() {
        this.eventBus = new EventBus();
        this.notifications = [];
    }
    
    subscribe(event, callback) {
        return this.eventBus.subscribe(event, callback);
    }
    
    showNotification(type, message, data = {}) {
        const notification = {
            id: Date.now() + Math.random(),
            type,
            message,
            data,
            timestamp: new Date()
        };
        
        this.notifications.push(notification);
        this.eventBus.publish('notification:created', notification);
        
        // Автоматическое удаление уведомления через 5 секунд
        setTimeout(() => {
            this.removeNotification(notification.id);
        }, 5000);
    }
    
    removeNotification(id) {
        this.notifications = this.notifications.filter(n => n.id !== id);
        this.eventBus.publish('notification:removed', { id });
    }
    
    getNotifications() {
        return [...this.notifications];
    }
}

// Использование
const notifier = new NotificationSystem();

// Подписка на создание уведомлений
notifier.subscribe('notification:created', (notification) => {
    console.log(`Новое уведомление [${notification.type}]: ${notification.message}`);
    renderNotification(notification);
});

// Отправка уведомления
notifier.showNotification('info', 'Добро пожаловать!', { userId: 123 });
notifier.showNotification('error', 'Ошибка при загрузке данных');
```

### Передача данных в сложных приложениях

```javascript
// Контекстная передача данных
class DataContext {
    constructor() {
        this.data = new Map();
        this.listeners = new Map(); // key -> [callbacks]
    }
    
    set(key, value) {
        const oldValue = this.data.get(key);
        this.data.set(key, value);
        
        // Уведомляем слушателей об изменении
        const listeners = this.listeners.get(key) || [];
        listeners.forEach(callback => {
            try {
                callback(value, oldValue, key);
            } catch (error) {
                console.error(`Ошибка в слушателе данных для ключа ${key}:`, error);
            }
        });
    }
    
    get(key) {
        return this.data.get(key);
    }
    
    subscribe(key, callback) {
        if (!this.listeners.has(key)) {
            this.listeners.set(key, []);
        }
        this.listeners.get(key).push(callback);
        
        // Возвращаем функцию отписки
        return () => {
            const listeners = this.listeners.get(key);
            const index = listeners.indexOf(callback);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        };
    }
    
    // Получение данных с автоматической подпиской (для реактивности)
    observe(key, callback) {
        const currentValue = this.get(key);
        if (currentValue !== undefined) {
            callback(currentValue);
        }
        return this.subscribe(key, callback);
    }
}

// Использование
const context = new DataContext();

// Устанавливаем начальные данные
context.set('user', { id: 1, name: 'Иван' });
context.set('theme', 'dark');

// Подписываемся на изменения
const userUnsubscribe = context.subscribe('user', (newUser, oldUser) => {
    console.log('Пользователь изменился:', oldUser, '->', newUser);
});

const themeUnsubscribe = context.subscribe('theme', (newTheme) => {
    document.body.className = newTheme;
    console.log('Тема изменена на:', newTheme);
});

// Изменяем данные
context.set('user', { id: 1, name: 'Иван Петров' });
context.set('theme', 'light');
```

## Лучшие практики

### Безопасная передача данных

```javascript
// Валидация передаваемых данных
function validateAndProcess(data, schema) {
    const errors = [];
    
    for (const [key, validator] of Object.entries(schema)) {
        if (!(key in data)) {
            errors.push(`Отсутствует обязательное поле: ${key}`);
            continue;
        }
        
        if (!validator(data[key])) {
            errors.push(`Неверный формат поля ${key}: ${data[key]}`);
        }
    }
    
    if (errors.length > 0) {
        throw new Error(`Ошибки валидации: ${errors.join(', ')}`);
    }
    
    return data;
}

// Схема валидации
const userSchema = {
    id: value => typeof value === 'number' && value > 0,
    name: value => typeof value === 'string' && value.length > 0,
    email: value => typeof value === 'string' && value.includes('@'),
    age: value => typeof value === 'number' && value >= 0 && value < 150
};

// Использование
try {
    const userData = { id: 1, name: 'Иван', email: 'ivan@example.com', age: 30 };
    const validData = validateAndProcess(userData, userSchema);
    console.log('Валидные данные:', validData);
} catch (error) {
    console.error('Ошибка валидации:', error.message);
}
```

### Оптимизация передачи больших объемов данных

```javascript
// Использование Transferable Objects для передачи больших данных
function transferLargeData(largeArrayBuffer) {
    // В веб-воркере
    const worker = new Worker('worker.js');
    
    // Передача ArrayBuffer без копирования
    worker.postMessage(largeArrayBuffer, [largeArrayBuffer]);
    // ArrayBuffer теперь недоступен в основном потоке
}

// Паттерн "обертка" для контроля передачи данных
class DataWrapper {
    constructor(data) {
        this._data = data;
        this._accessCount = 0;
    }
    
    get data() {
        this._accessCount++;
        return this._data;
    }
    
    getAccessCount() {
        return this._accessCount;
    }
    
    // Создание копии данных (контролируемое клонирование)
    clone() {
        this._accessCount++;
        if (typeof this._data === 'object' && this._data !== null) {
            return JSON.parse(JSON.stringify(this._data));
        }
        return this._data;
    }
}

// Использование
const largeData = { items: new Array(1000000).fill(0).map((_, i) => i) };
const wrapped = new DataWrapper(largeData);

// Передаем обертку вместо оригинальных данных
function processWrappedData(wrapper) {
    console.log('Количество доступов:', wrapper.getAccessCount());
    return wrapper.data.items.length;
}
```

## Заключение

Передача данных и работа с событиями - фундаментальные аспекты разработки на JavaScript. Понимание различных методов передачи данных, особенностей передачи по значению и по ссылке, а также современных паттернов событий позволяет создавать более надежные и эффективные приложения. Важно выбирать подходящий метод передачи данных в зависимости от конкретной задачи и объема передаваемой информации.

#programming #javascript #data-passing #events #callbacks #promises #async #reactive #eventbus #web-development