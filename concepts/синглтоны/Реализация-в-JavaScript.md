---
aliases: ["JavaScript Singleton", "Синглтон в JavaScript", "Реализация Singleton"]
tags: [javascript, frontend, design-patterns]
---

# Реализация паттерна Синглтон в JavaScript

## Обзор

JavaScript предоставляет несколько способов реализации паттерна Синглтон благодаря своей гибкой природе. Ниже рассмотрены различные подходы к созданию синглтонов в JavaScript.

## Классическая реализация с ES6 классами

```javascript
class DatabaseConnection {
    constructor(connectionString) {
        if (DatabaseConnection.instance) {
            return DatabaseConnection.instance;
        }
        
        this.connectionString = connectionString;
        this.isConnected = false;
        
        DatabaseConnection.instance = this;
        return this;
    }
    
    connect() {
        if (!this.isConnected) {
            console.log(`Соединение с ${this.connectionString} установлено`);
            this.isConnected = true;
        }
        return this;
    }
    
    disconnect() {
        if (this.isConnected) {
            console.log('Соединение закрыто');
            this.isConnected = false;
        }
    }
}

// Использование
const db1 = new DatabaseConnection('mongodb://localhost:27017');
const db2 = new DatabaseConnection('mongodb://localhost:27018');

console.log(db1 === db2); // true - это один и тот же экземпляр
```

## Реализация с использованием замыкания

```javascript
const Logger = (function() {
    let instance;
    
    function createInstance() {
        return {
            logs: [],
            
            log(message) {
                const timestamp = new Date().toISOString();
                this.logs.push({ message, timestamp });
                console.log(`[${timestamp}] ${message}`);
            },
            
            getLogs() {
                return [...this.logs];
            },
            
            clearLogs() {
                this.logs = [];
            }
        };
    }
    
    return {
        getInstance: function() {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();

// Использование
const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();

logger1.log('Сообщение 1');
logger2.log('Сообщение 2');

console.log(logger1 === logger2); // true
console.log(logger1.getLogs()); // [{message: 'Сообщение 1', ...}, {message: 'Сообщение 2', ...}]
```

## Реализация с использованием объекта-модуля

```javascript
const ConfigManager = (() => {
    let instance = null;
    
    class ConfigManagerClass {
        constructor() {
            if (instance) {
                return instance;
            }
            
            this.config = {};
            instance = this;
            return this;
        }
        
        set(key, value) {
            this.config[key] = value;
        }
        
        get(key) {
            return this.config[key];
        }
        
        getAll() {
            return { ...this.config };
        }
        
        load(configObj) {
            Object.assign(this.config, configObj);
        }
    }
    
    return new ConfigManagerClass();
})();

// Использование
ConfigManager.set('apiUrl', 'https://api.example.com');
ConfigManager.set('timeout', 5000);

const config1 = ConfigManager;
const config2 = (() => {
    // Попытка создать новый экземпляр
    return new (ConfigManager.constructor)();
})();

console.log(config1 === config2); // true
console.log(ConfigManager.get('apiUrl')); // 'https://api.example.com'
```

## Реализация с ленивой инициализацией

```javascript
class LazySingleton {
    constructor() {
        if (LazySingleton._instance) {
            return LazySingleton._instance;
        }
        
        this.initialized = false;
        this.data = null;
        
        // Отложенная инициализация
        this.initialize();
        LazySingleton._instance = this;
        return this;
    }
    
    initialize() {
        if (!this.initialized) {
            console.log('Инициализация синглтона...');
            this.data = this.loadData();
            this.initialized = true;
        }
    }
    
    loadData() {
        // Симуляция загрузки данных
        return {
            timestamp: Date.now(),
            version: '1.0.0'
        };
    }
    
    getData() {
        this.initialize(); // Убедиться, что инициализация выполнена
        return this.data;
    }
}

// Использование
const instance1 = new LazySingleton();
const instance2 = new LazySingleton();

console.log(instance1 === instance2); // true
console.log(instance1.getData()); // Данные будут загружены только при первом вызове
```

## Реализация с поддержкой многопоточности (асинхронная безопасность)

```javascript
class ThreadSafeSingleton {
    constructor() {
        if (ThreadSafeSingleton._instance) {
            return ThreadSafeSingleton._instance;
        }
        
        this.initialized = false;
        this.initPromise = null;
        
        ThreadSafeSingleton._instance = this;
        return this;
    }
    
    async initialize() {
        if (this.initialized) {
            return this;
        }
        
        if (this.initPromise) {
            // Если инициализация уже в процессе, ждем её завершения
            return await this.initPromise;
        }
        
        this.initPromise = this._performInitialization();
        const result = await this.initPromise;
        this.initialized = true;
        return result;
    }
    
    async _performInitialization() {
        // Симуляция асинхронной инициализации
        await new Promise(resolve => setTimeout(resolve, 100));
        this.data = { initialized: true, timestamp: Date.now() };
        return this;
    }
    
    getData() {
        return this.data;
    }
}

// Пример использования в асинхронной среде
async function testThreadSafety() {
    const promises = [];
    
    // Создание нескольких экземпляров одновременно
    for (let i = 0; i < 5; i++) {
        promises.push((async () => {
            const instance = new ThreadSafeSingleton();
            return await instance.initialize();
        })());
    }
    
    const instances = await Promise.all(promises);
    
    // Проверка, что все экземпляры одинаковы
    const allSame = instances.every(inst => inst === instances[0]);
    console.log('Все экземпляры одинаковы:', allSame); // true
}
```

## Использование в браузерной среде

```javascript
// Глобальный менеджер состояния приложения
class AppStateManager {
    constructor() {
        if (AppStateManager._instance) {
            return AppStateManager._instance;
        }
        
        this.state = {
            user: null,
            theme: 'light',
            language: 'ru'
        };
        
        this.listeners = [];
        
        AppStateManager._instance = this;
        return this;
    }
    
    setState(newState) {
        this.state = { ...this.state, ...newState };
        this.notifyListeners();
    }
    
    getState() {
        return { ...this.state };
    }
    
    subscribe(callback) {
        this.listeners.push(callback);
        return () => {
            this.listeners = this.listeners.filter(listener => listener !== callback);
        };
    }
    
    notifyListeners() {
        this.listeners.forEach(callback => callback(this.state));
    }
}

// Использование в приложении
const appState = new AppStateManager();

// Установка начального состояния
appState.setState({ user: { name: 'John', id: 1 } });

// Подписка на изменения
const unsubscribe = appState.subscribe((state) => {
    console.log('Состояние изменилось:', state);
});

// Изменение состояния
appState.setState({ theme: 'dark' });
```

## Современные альтернативы

С развитием JavaScript и фреймворков, часто используются альтернативные подходы:

```javascript
// Использование ES6 модулей (естественный синглтон)
// config.js
let instance = null;

class ConfigManager {
    constructor() {
        if (instance) {
            return instance;
        }
        
        this.config = {};
        instance = this;
        return this;
    }
    
    set(key, value) {
        this.config[key] = value;
    }
    
    get(key) {
        return this.config[key];
    }
}

export default new ConfigManager(); // Модуль автоматически кэшируется

// Использование
import config from './config.js';
config.set('apiUrl', 'https://api.example.com');
```

## Заключение

JavaScript предоставляет гибкие возможности для реализации паттерна Синглтон. Выбор конкретного подхода зависит от требований проекта, особенностей среды выполнения и предпочтений команды разработчиков.

Важно помнить о потенциальных проблемах, связанных с тестируемостью и глобальным состоянием, и использовать синглтоны обоснованно.

## См. также

- [[Паттерн-синглтон]]
- [[Использование-в-React]]
- [[Использование-в-Vue]]
- [[Проблемы-и-альтернативы]]
- [[Модульный паттерн]]
- [[Фабричный паттерн]]