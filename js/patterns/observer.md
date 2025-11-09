# Паттерн Наблюдатель в JavaScript

## Обзор

Паттерн Наблюдатель (Observer Pattern) - это поведенческий паттерн проектирования, который позволяет объектам наблюдать за изменениями в других объектах и реагировать на них. Он реализует отношение "один ко многим", где один объект (субъект) имеет список зависимых объектов (наблюдателей), которые автоматически уведомляются об изменениях.

## Простая реализация паттерна Наблюдатель

### Базовая реализация

```javascript
// Класс субъекта (Subject)
class Subject {
    constructor() {
        this.observers = [];
    }
    
    // Добавление наблюдателя
    subscribe(observer) {
        if (!this.observers.includes(observer)) {
            this.observers.push(observer);
            console.log(`Наблюдатель добавлен. Всего наблюдателей: ${this.observers.length}`);
        }
    }
    
    // Удаление наблюдателя
    unsubscribe(observer) {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
            this.observers.splice(index, 1);
            console.log(`Наблюдатель удален. Всего наблюдателей: ${this.observers.length}`);
        }
    }
    
    // Уведомление всех наблюдателей
    notify(data) {
        console.log(`Уведомление ${this.observers.length} наблюдателей...`);
        this.observers.forEach(observer => {
            observer.update(data);
        });
    }
}

// Пример наблюдателя
class ConcreteObserver {
    constructor(name) {
        this.name = name;
    }
    
    update(data) {
        console.log(`Наблюдатель ${this.name} получил обновление:`, data);
    }
}

// Использование
const subject = new Subject();

const observer1 = new ConcreteObserver('Observer1');
const observer2 = new ConcreteObserver('Observer2');
const observer3 = new ConcreteObserver('Observer3');

subject.subscribe(observer1);
subject.subscribe(observer2);
subject.subscribe(observer3);

subject.notify({ message: 'Привет, наблюдатели!', timestamp: new Date() });

// Удаляем одного наблюдателя
subject.unsubscribe(observer2);

subject.notify({ message: 'Теперь наблюдатель 2 не будет уведомлен', timestamp: new Date() });
```

## Продвинутая реализация с различными типами событий

### Реализация с поддержкой различных событий

```javascript
// Улучшенная реализация Subject с поддержкой различных типов событий
class EventPublisher {
    constructor() {
        this.eventListeners = new Map(); // Карта событие -> [наблюдатели]
    }
    
    // Подписка на конкретное событие
    on(eventType, listener) {
        if (!this.eventListeners.has(eventType)) {
            this.eventListeners.set(eventType, []);
        }
        
        const listeners = this.eventListeners.get(eventType);
        if (!listeners.includes(listener)) {
            listeners.push(listener);
        }
    }
    
    // Отписка от события
    off(eventType, listener) {
        if (this.eventListeners.has(eventType)) {
            const listeners = this.eventListeners.get(eventType);
            const index = listeners.indexOf(listener);
            if (index > -1) {
                listeners.splice(index, 1);
            }
            
            // Удаляем пустой массив, если нет слушателей
            if (listeners.length === 0) {
                this.eventListeners.delete(eventType);
            }
        }
    }
    
    // Уведомление слушателей о событии
    emit(eventType, data) {
        if (this.eventListeners.has(eventType)) {
            const listeners = this.eventListeners.get(eventType);
            listeners.forEach(listener => {
                try {
                    listener(data);
                } catch (error) {
                    console.error(`Ошибка в слушателе события ${eventType}:`, error);
                }
            });
        }
    }
    
    // Подписка с автоматической отпиской после одного вызова
    once(eventType, listener) {
        const onceWrapper = (data) => {
            listener(data);
            this.off(eventType, onceWrapper);
        };
        
        this.on(eventType, onceWrapper);
    }
    
    // Получение количества слушателей для события
    listenerCount(eventType) {
        return this.eventListeners.has(eventType) 
            ? this.eventListeners.get(eventType).length 
            : 0;
    }
}

// Пример использования с различными событиями
const publisher = new EventPublisher();

// Определяем слушателей
const userLoginListener = (userData) => {
    console.log(`Пользователь ${userData.name} вошел в систему`);
};

const userLogoutListener = (userData) => {
    console.log(`Пользователь ${userData.name} вышел из системы`);
};

const systemNotificationListener = (notification) => {
    console.log(`Системное уведомление: ${notification.message}`);
};

// Подписываемся на события
publisher.on('userLogin', userLoginListener);
publisher.on('userLogout', userLogoutListener);
publisher.on('systemNotification', systemNotificationListener);

// Генерируем события
publisher.emit('userLogin', { name: 'Иван', id: 123 });
publisher.emit('systemNotification', { message: 'Система работает нормально' });
publisher.emit('userLogout', { name: 'Иван', id: 123 });

// Пример однократной подписки
publisher.once('oneTimeEvent', (data) => {
    console.log('Это событие будет обработано только один раз:', data);
});

publisher.emit('oneTimeEvent', { message: 'Первый вызов' });
publisher.emit('oneTimeEvent', { message: 'Второй вызов - не будет обработан' });
```

## Реализация с использованием Proxy

### Продвинутая реализация с автоматическим отслеживанием изменений

```javascript
// Реализация наблюдателя с использованием Proxy для автоматического отслеживания изменений
class ObservableObject {
    constructor(target, onChange) {
        this.target = target;
        this.onChange = onChange;
        this.observers = [];
        
        return new Proxy(target, {
            get: (obj, prop) => {
                // Если это вложенный объект, делаем его наблюдаемым тоже
                if (typeof obj[prop] === 'object' && obj[prop] !== null && !obj[prop].__isObservable) {
                    obj[prop] = this.makeObservable(obj[prop]);
                }
                return obj[prop];
            },
            
            set: (obj, prop, value) => {
                const oldValue = obj[prop];
                
                // Если значение - объект, делаем его наблюдаемым
                if (typeof value === 'object' && value !== null && !value.__isObservable) {
                    value = this.makeObservable(value);
                }
                
                obj[prop] = value;
                
                // Уведомляем наблюдателей об изменении
                if (this.onChange) {
                    this.onChange(prop, value, oldValue);
                }
                
                this.notifyObservers({ property: prop, newValue: value, oldValue: oldValue });
                
                return true;
            }
        });
    }
    
    makeObservable(obj) {
        if (obj.__isObservable) return obj;
        
        obj.__isObservable = true;
        return new ObservableObject(obj, this.onChange);
    }
    
    subscribe(observer) {
        this.observers.push(observer);
    }
    
    unsubscribe(observer) {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
            this.observers.splice(index, 1);
        }
    }
    
    notifyObservers(change) {
        this.observers.forEach(observer => observer(change));
    }
}

// Пример использования
const data = {
    user: {
        name: 'Иван',
        age: 30
    },
    settings: {
        theme: 'light',
        notifications: true
    }
};

const observableData = new ObservableObject(data, (property, newValue, oldValue) => {
    console.log(`Свойство "${property}" изменилось с "${oldValue}" на "${newValue}"`);
});

// Добавляем наблюдателей
observableData.subscribe((change) => {
    console.log('Наблюдатель 1: Изменение ->', change);
});

observableData.subscribe((change) => {
    console.log('Наблюдатель 2: Изменение ->', change);
});

// Тестируем изменения
observableData.user.name = 'Петр';
observableData.settings.theme = 'dark';
observableData.user.age = 31;
```

## Практические примеры использования

### Реализация системы уведомлений

```javascript
// Система уведомлений с использованием паттерна Наблюдатель
class NotificationSystem {
    constructor() {
        this.subscribers = new Map(); // тип уведомления -> [подписчики]
        this.history = []; // История уведомлений
    }
    
    // Подписка на уведомления определенного типа
    subscribe(notificationType, subscriber) {
        if (!this.subscribers.has(notificationType)) {
            this.subscribers.set(notificationType, []);
        }
        
        const subscribers = this.subscribers.get(notificationType);
        if (!subscribers.includes(subscriber)) {
            subscribers.push(subscriber);
            console.log(`Подписчик добавлен на уведомления типа: ${notificationType}`);
        }
    }
    
    // Отписка от уведомлений
    unsubscribe(notificationType, subscriber) {
        if (this.subscribers.has(notificationType)) {
            const subscribers = this.subscribers.get(notificationType);
            const index = subscribers.indexOf(subscriber);
            if (index > -1) {
                subscribers.splice(index, 1);
                console.log(`Подписчик отписан от уведомлений типа: ${notificationType}`);
            }
        }
    }
    
    // Отправка уведомления
    notify(notificationType, message, data = {}) {
        const notification = {
            id: Date.now(),
            type: notificationType,
            message: message,
            data: data,
            timestamp: new Date()
        };
        
        // Сохраняем в историю
        this.history.push(notification);
        
        // Уведомляем подписчиков
        if (this.subscribers.has(notificationType)) {
            const subscribers = this.subscribers.get(notificationType);
            subscribers.forEach(subscriber => {
                try {
                    subscriber.handleNotification(notification);
                } catch (error) {
                    console.error(`Ошибка при отправке уведомления подписчику:`, error);
                }
            });
        }
        
        return notification.id;
    }
    
    // Получение истории уведомлений
    getHistory(limit = 10) {
        return this.history.slice(-limit);
    }
}

// Классы подписчиков
class EmailNotifier {
    constructor(email) {
        this.email = email;
    }
    
    handleNotification(notification) {
        console.log(`Отправка email на ${this.email}: [${notification.type}] ${notification.message}`);
        // Здесь будет реальная логика отправки email
    }
}

class PushNotifier {
    constructor(deviceToken) {
        this.deviceToken = deviceToken;
    }
    
    handleNotification(notification) {
        console.log(`Отправка push-уведомления на устройство ${this.deviceToken}: ${notification.message}`);
        // Здесь будет реальная логика отправки push-уведомления
    }
}

class SMSNotifier {
    constructor(phone) {
        this.phone = phone;
    }
    
    handleNotification(notification) {
        console.log(`Отправка SMS на ${this.phone}: ${notification.message}`);
        // Здесь будет реальная логика отправки SMS
    }
}

// Использование системы уведомлений
const notificationSystem = new NotificationSystem();

// Создаем подписчиков
const emailNotifier = new EmailNotifier('ivan@example.com');
const pushNotifier = new PushNotifier('device-token-123');
const smsNotifier = new SMSNotifier('+79991234567');

// Подписываем на различные типы уведомлений
notificationSystem.subscribe('info', emailNotifier);
notificationSystem.subscribe('info', pushNotifier);
notificationSystem.subscribe('urgent', smsNotifier);
notificationSystem.subscribe('urgent', pushNotifier);

// Отправляем уведомления
notificationSystem.notify('info', 'Ваш профиль обновлен', { userId: 123 });
notificationSystem.notify('urgent', 'Неподтвержденная активность на вашем аккаунте', { userId: 123, ip: '192.168.1.1' });

// Показываем историю
console.log('История уведомлений:', notificationSystem.getHistory());
```

### Реализация системы событий для приложения

```javascript
// Система событий приложения (Event Bus)
class EventBus {
    constructor() {
        this.listeners = new Map();
        this.middleware = [];
    }
    
    // Добавление middleware
    use(middlewareFn) {
        this.middleware.push(middlewareFn);
    }
    
    // Подписка на событие
    on(event, callback) {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, []);
        }
        
        this.listeners.get(event).push(callback);
    }
    
    // Однократная подписка
    once(event, callback) {
        const onceCallback = (...args) => {
            callback(...args);
            this.off(event, onceCallback);
        };
        this.on(event, onceCallback);
    }
    
    // Отписка от события
    off(event, callback) {
        if (this.listeners.has(event)) {
            const callbacks = this.listeners.get(event);
            const index = callbacks.indexOf(callback);
            if (index > -1) {
                callbacks.splice(index, 1);
            }
        }
    }
    
    // Вызов события
    async emit(event, ...args) {
        // Применяем middleware
        for (const middleware of this.middleware) {
            const result = await Promise.resolve(middleware(event, args));
            if (result === false) {
                return; // Middleware остановил событие
            }
        }
        
        // Вызываем слушателей
        if (this.listeners.has(event)) {
            const callbacks = [...this.listeners.get(event)]; // Создаем копию на случай изменений во время выполнения
            for (const callback of callbacks) {
                try {
                    await Promise.resolve(callback(...args));
                } catch (error) {
                    console.error(`Ошибка в слушателе события ${event}:`, error);
                }
            }
        }
    }
    
    // Удаление всех слушателей для события
    removeAllListeners(event) {
        if (event) {
            this.listeners.delete(event);
        } else {
            this.listeners.clear();
        }
    }
}

// Пример использования EventBus
const eventBus = new EventBus();

// Добавляем middleware для логирования
eventBus.use(async (event, args) => {
    console.log(`Событие "${event}" вызвано с аргументами:`, args);
    return true; // Продолжить выполнение
});

// Подписываемся на события
eventBus.on('user:login', async (userData) => {
    console.log(`Пользователь ${userData.name} вошел в систему`);
    // Логика после входа пользователя
});

eventBus.on('user:logout', (userData) => {
    console.log(`Пользователь ${userData.name} вышел из системы`);
    // Логика при выходе пользователя
});

eventBus.once('app:initialized', () => {
    console.log('Приложение инициализировано');
});

// Вызываем события
await eventBus.emit('app:initialized');
await eventBus.emit('user:login', { name: 'Иван', id: 123 });
await eventBus.emit('user:logout', { name: 'Иван', id: 123 });
```

## Преимущества и недостатки паттерна Наблюдатель

### Преимущества:
- Поддерживает слабую связанность между субъектом и наблюдателями
- Позволяет настраивать зависимости между объектами во время выполнения
- Реализует широковещательную передачу (broadcast communication)
- Поддерживает работу с объектами, реализующими один и тот же интерфейс

### Недостатки:
- Может привести к неожиданному поведению, так как наблюдатели не знают о существовании друг друга
- Может вызвать утечки памяти, если не управлять подписками должным образом
- Сложно отлаживать, так как поток управления лежит в субъекте
- Наблюдатели реагируют на обновления в случайном порядке