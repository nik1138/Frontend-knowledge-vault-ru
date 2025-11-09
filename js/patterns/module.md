# Паттерн Модуль в JavaScript

## Обзор

Паттерн Модуль (Module Pattern) позволяет создавать изолированные области видимости в JavaScript, обеспечивая инкапсуляцию данных и методов. Он используется для создания приватных и публичных членов, предотвращая загрязнение глобального пространства имен.

## Классическая реализация паттерна Модуль

### Анонимная самовызывающаяся функция (IIFE)

```javascript
const CalculatorModule = (function() {
    // Приватные переменные
    let result = 0;
    const history = [];
    
    // Приватные методы
    function addToHistory(operation) {
        history.push({
            operation: operation,
            result: result,
            timestamp: new Date()
        });
    }
    
    function validateNumber(num) {
        return typeof num === 'number' && !isNaN(num);
    }
    
    // Публичный API
    return {
        // Публичные методы
        add: function(num) {
            if (!validateNumber(num)) {
                throw new Error('Аргумент должен быть числом');
            }
            result += num;
            addToHistory(`add(${num})`);
            return this; // Для цепочки вызовов
        },
        
        subtract: function(num) {
            if (!validateNumber(num)) {
                throw new Error('Аргумент должен быть числом');
            }
            result -= num;
            addToHistory(`subtract(${num})`);
            return this;
        },
        
        multiply: function(num) {
            if (!validateNumber(num)) {
                throw new Error('Аргумент должен быть числом');
            }
            result *= num;
            addToHistory(`multiply(${num})`);
            return this;
        },
        
        divide: function(num) {
            if (!validateNumber(num)) {
                throw new Error('Аргумент должен быть числом');
            }
            if (num === 0) {
                throw new Error('Деление на ноль недопустимо');
            }
            result /= num;
            addToHistory(`divide(${num})`);
            return this;
        },
        
        getResult: function() {
            return result;
        },
        
        reset: function() {
            result = 0;
            history.length = 0; // Очистка истории
            return this;
        },
        
        getHistory: function() {
            return [...history]; // Возвращаем копию истории
        }
    };
})();

// Использование
CalculatorModule.add(10).multiply(2).subtract(5);
console.log(CalculatorModule.getResult()); // 15
console.log(CalculatorModule.getHistory()); // История операций

// Попытка доступа к приватным переменным
console.log(typeof CalculatorModule.result); // "undefined"
console.log(typeof CalculatorModule.history); // "undefined"
```

## Современная реализация с использованием ES6+

### Функция-фабрика

```javascript
function createCounter(initialValue = 0) {
    // Приватное состояние
    let count = initialValue;
    
    return {
        increment: function() {
            count++;
            return this;
        },
        
        decrement: function() {
            count--;
            return this;
        },
        
        getValue: function() {
            return count;
        },
        
        reset: function() {
            count = initialValue;
            return this;
        },
        
        // Метод для получения приватного состояния (опционально)
        _getInternalState: function() {
            return { count: count, initialValue: initialValue };
        }
    };
}

// Использование
const counter1 = createCounter(10);
const counter2 = createCounter(0);

counter1.increment().increment();
counter2.increment();

console.log(counter1.getValue()); // 12
console.log(counter2.getValue()); // 1

// Каждый экземпляр имеет собственное состояние
console.log(counter1._getInternalState()); // { count: 12, initialValue: 10 }
console.log(counter2._getInternalState()); // { count: 1, initialValue: 0 }
```

### Класс с приватными полями (ES2022+)

```javascript
class ApiClientModule {
    // Приватные поля (предварительно спецификация ES2022+)
    #apiKey;
    #baseUrl;
    #requestCount = 0;
    
    constructor(apiKey, baseUrl) {
        this.#apiKey = apiKey;
        this.#baseUrl = baseUrl;
    }
    
    async makeRequest(endpoint, options = {}) {
        this.#requestCount++;
        
        const url = `${this.#baseUrl}${endpoint}`;
        const headers = {
            ...options.headers,
            'Authorization': `Bearer ${this.#apiKey}`,
            'Content-Type': 'application/json'
        };
        
        try {
            const response = await fetch(url, {
                ...options,
                headers
            });
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.error('Ошибка запроса:', error);
            throw error;
        }
    }
    
    getRequestCount() {
        return this.#requestCount;
    }
    
    // Геттеры для доступа к приватным полям (если необходимо)
    getBaseUrl() {
        return this.#baseUrl;
    }
}

// Использование
const apiClient = new ApiClientModule('my-api-key', 'https://api.example.com');
// apiClient.#apiKey; // Ошибка: приватное поле недоступно извне
console.log(apiClient.getRequestCount()); // 0
```

## Практические примеры

### Модуль управления настройками

```javascript
const SettingsModule = (function() {
    // Приватные настройки
    const defaultSettings = {
        theme: 'light',
        language: 'ru',
        notifications: true,
        autoSave: true
    };
    
    let currentSettings = { ...defaultSettings };
    
    // Приватные методы
    function validateSetting(key, value) {
        const allowedKeys = Object.keys(defaultSettings);
        if (!allowedKeys.includes(key)) {
            throw new Error(`Недопустимый ключ настройки: ${key}`);
        }
        
        // Дополнительная валидация значений
        if (key === 'theme' && !['light', 'dark'].includes(value)) {
            throw new Error('Тема должна быть "light" или "dark"');
        }
        
        if (key === 'language' && !['ru', 'en', 'es'].includes(value)) {
            throw new Error('Язык должен быть "ru", "en" или "es"');
        }
        
        if (key === 'notifications' || key === 'autoSave') {
            if (typeof value !== 'boolean') {
                throw new Error(`${key} должен быть булевым значением`);
            }
        }
    }
    
    // Публичный API
    return {
        get: function(key) {
            return currentSettings[key];
        },
        
        set: function(key, value) {
            validateSetting(key, value);
            currentSettings[key] = value;
            return this;
        },
        
        getAll: function() {
            return { ...currentSettings }; // Возвращаем копию
        },
        
        reset: function() {
            currentSettings = { ...defaultSettings };
            return this;
        },
        
        onChange: function(callback) {
            // Упрощенная реализация подписки на изменения
            const originalSet = this.set;
            const self = this;
            
            this.set = function(key, value) {
                const oldValue = currentSettings[key];
                originalSet.call(self, key, value);
                
                if (callback && typeof callback === 'function') {
                    callback(key, value, oldValue);
                }
                
                return this;
            };
            
            return this;
        }
    };
})();

// Использование
SettingsModule.set('theme', 'dark').set('language', 'en');
console.log(SettingsModule.get('theme')); // 'dark'
console.log(SettingsModule.getAll()); // Все настройки

// Подписка на изменения
SettingsModule.onChange((key, newValue, oldValue) => {
    console.log(`Настройка ${key} изменилась с ${oldValue} на ${newValue}`);
});
```

### Модуль управления событиями

```javascript
const EventEmitterModule = (function() {
    const events = {};
    
    return {
        on: function(eventName, callback) {
            if (!events[eventName]) {
                events[eventName] = [];
            }
            
            if (typeof callback === 'function') {
                events[eventName].push(callback);
            }
            
            return this;
        },
        
        emit: function(eventName, data) {
            if (events[eventName]) {
                events[eventName].forEach(callback => {
                    try {
                        callback(data);
                    } catch (error) {
                        console.error(`Ошибка в обработчике события ${eventName}:`, error);
                    }
                });
            }
            return this;
        },
        
        off: function(eventName, callback) {
            if (events[eventName]) {
                if (callback) {
                    const index = events[eventName].indexOf(callback);
                    if (index > -1) {
                        events[eventName].splice(index, 1);
                    }
                } else {
                    events[eventName] = [];
                }
            }
            return this;
        },
        
        once: function(eventName, callback) {
            const self = this;
            const onceWrapper = function(...args) {
                callback.apply(this, args);
                self.off(eventName, onceWrapper);
            };
            
            return this.on(eventName, onceWrapper);
        }
    };
})();

// Использование
EventEmitterModule
    .on('userLogin', (userData) => console.log('Пользователь вошел:', userData))
    .on('userLogout', (userData) => console.log('Пользователь вышел:', userData))
    .emit('userLogin', { id: 1, name: 'Иван' });
```

## Преимущества и недостатки

### Преимущества:
- Инкапсуляция данных
- Защита приватных методов и переменных
- Предотвращение загрязнения глобального пространства
- Контроль над публичным API
- Возможность создания замыканий

### Недостатки:
- Сложность тестирования приватных методов
- Ограниченная возможность расширения
- Более сложный синтаксис по сравнению с обычными функциями
- Меньшая производительность из-за замыканий