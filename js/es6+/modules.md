# Модули в JavaScript

## Обзор

Модули в JavaScript позволяют разделять код на отдельные файлы и компоненты, обеспечивая лучшую организацию, повторное использование и изоляцию кода. Модули были официально добавлены в JavaScript в ES6 (ES2015).

## Типы экспорта

### Named Exports (именованные экспорты)

```javascript
// mathUtils.js
export const PI = 3.14159;

export function calculateArea(radius) {
    return PI * radius * radius;
}

export function calculateCircumference(radius) {
    return 2 * PI * radius;
}

// Можно также экспортировать в конце файла
const DEFAULT_RADIUS = 1;
function getDefaultArea() {
    return calculateArea(DEFAULT_RADIUS);
}

export { DEFAULT_RADIUS, getDefaultArea };
```

```javascript
// main.js
import { PI, calculateArea, calculateCircumference } from './mathUtils.js';

console.log(PI); // 3.14159
console.log(calculateArea(5)); // 78.53975
console.log(calculateCircumference(5)); // 31.4159
```

### Default Export (дефолтный экспорт)

```javascript
// calculator.js
class Calculator {
    constructor() {
        this.result = 0;
    }
    
    add(num) {
        this.result += num;
        return this;
    }
    
    multiply(num) {
        this.result *= num;
        return this;
    }
    
    getResult() {
        return this.result;
    }
}

export default Calculator;
```

```javascript
// main.js
import Calculator from './calculator.js';

const calc = new Calculator();
calc.add(5).multiply(3);
console.log(calc.getResult()); // 15
```

### Mixed Exports (смешанные экспорты)

```javascript
// utils.js
// Дефолтный экспорт
export default function greet(name) {
    return `Привет, ${name}!`;
}

// Именованные экспорты
export const MAX_USERS = 100;
export const formatDate = (date) => date.toISOString().split('T')[0];
export class User {
    constructor(name) {
        this.name = name;
    }
}
```

```javascript
// main.js
import greet, { MAX_USERS, formatDate, User } from './utils.js';

console.log(greet('Иван')); // "Привет, Иван!"
console.log(MAX_USERS); // 100
console.log(formatDate(new Date())); // "2023-01-01"
const user = new User('Мария');
```

## Типы импорта

### Именованный импорт

```javascript
// Импорт конкретных экспортированных значений
import { calculateArea, PI } from './mathUtils.js';

// Импорт с переименованием
import { calculateArea as areaCalc, PI as piConstant } from './mathUtils.js';

// Импорт всех именованных экспортированных значений
import * as math from './mathUtils.js';
console.log(math.PI); // 3.14159
console.log(math.calculateArea(3)); // 28.27431
```

### Комбинированный импорт

```javascript
// Импорт дефолтного и именованных экспортированных значений
import Calculator, { MAX_USERS, formatDate } from './utils.js';

// Импорт всего с переименованием
import Calculator, * as utils from './utils.js';
```

## Практические примеры

### Модуль для работы с API

```javascript
// api.js
const API_BASE_URL = 'https://api.example.com';

export async function fetchUserData(userId) {
    try {
        const response = await fetch(`${API_BASE_URL}/users/${userId}`);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        console.error('Ошибка при получении данных пользователя:', error);
        throw error;
    }
}

export async function createUser(userData) {
    try {
        const response = await fetch(`${API_BASE_URL}/users`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(userData),
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Ошибка при создании пользователя:', error);
        throw error;
    }
}

export { API_BASE_URL };
```

```javascript
// userService.js
import { fetchUserData, createUser, API_BASE_URL } from './api.js';

export class UserService {
    static async getUser(id) {
        return await fetchUserData(id);
    }
    
    static async registerUser(userData) {
        return await createUser(userData);
    }
    
    static getApiUrl() {
        return API_BASE_URL;
    }
}
```

### Модуль с конфигурацией

```javascript
// config.js
const development = {
    apiUrl: 'http://localhost:3000/api',
    debug: true,
    timeout: 5000
};

const production = {
    apiUrl: 'https://api.production.com',
    debug: false,
    timeout: 10000
};

const config = process.env.NODE_ENV === 'production' ? production : development;

export default config;
export { development, production };
```

```javascript
// apiClient.js
import config, { development } from './config.js';

class ApiClient {
    constructor() {
        this.baseUrl = config.apiUrl;
        this.timeout = config.timeout;
        this.debug = config.debug;
    }
    
    async request(endpoint, options = {}) {
        if (this.debug) {
            console.log(`Запрос к: ${this.baseUrl}${endpoint}`);
        }
        
        const url = `${this.baseUrl}${endpoint}`;
        const opts = {
            ...options,
            timeout: this.timeout
        };
        
        return await fetch(url, opts);
    }
}

export default ApiClient;
```

## Динамические импорты

```javascript
// Динамический импорт позволяет загружать модули по требованию
async function loadModule(modulePath) {
    const module = await import(modulePath);
    return module;
}

// Пример использования
async function initializeFeature() {
    if (someCondition) {
        const { heavyFunction } = await import('./heavyModule.js');
        heavyFunction();
    }
}

// Условная загрузка модуля
async function loadLanguage(lang) {
    let translations;
    
    switch (lang) {
        case 'en':
            translations = await import('./translations/en.js');
            break;
        case 'ru':
            translations = await import('./translations/ru.js');
            break;
        default:
            translations = await import('./translations/en.js');
    }
    
    return translations.default;
}
```

## Особенности модулей

1. **Файл как модуль**: Каждый файл JavaScript может быть модулем
2. **Лексическая область видимости**: Переменные в модуле не доступны глобально
3. **Строгий режим**: Модули всегда работают в strict mode
4. **Кэширование**: Модули кэшируются после первого импорта
5. **Top-level await**: В модулях доступен await на верхнем уровне (ES2022+)