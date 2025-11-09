# Modules API

## Введение

Модульные системы в JavaScript позволяют организовывать код в независимые, переиспользуемые компоненты. В этой главе мы рассмотрим различные системы модулей: ES6 модули, CommonJS, AMD, а также современные подходы к организации кода и управлению зависимостями.

## Содержание

- [[ES6 Модули]]
- [[CommonJS]]
- [[AMD (Asynchronous Module Definition)]]
- [[UMD (Universal Module Definition)]]
- [[Динамические импорты]]
- [[Tree Shaking]]
- [[Модульные системы в браузере]]
- [[Работа с зависимостями]]
- [[Модульные паттерны]]

## ES6 Модули

ES6 модули (ECMAScript 2015) - это стандартный способ организации модулей в современном JavaScript. Они обеспечивают статическую структуру импортов/экспортов, что позволяет оптимизировать сборку и применять tree-shaking.

### Основы ES6 модулей

```javascript
// math-utils.js - экспорт функций
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export function multiply(a, b) {
    return a * b;
}

export function divide(a, b) {
    if (b === 0) {
        throw new Error('Деление на ноль');
    }
    return a / b;
}

// Именованный экспорт нескольких значений
export const PI = 3.14159;
export const E = 2.71828;

// Дефолтный экспорт (только один на модуль)
export default function calculate(operation, a, b) {
    switch (operation) {
        case 'add': return add(a, b);
        case 'subtract': return subtract(a, b);
        case 'multiply': return multiply(a, b);
        case 'divide': return divide(a, b);
        default: throw new Error('Неизвестная операция');
    }
}

// constants.js - экспорт констант
export const MAX_SAFE_INTEGER = Number.MAX_SAFE_INTEGER;
export const MIN_SAFE_INTEGER = Number.MIN_SAFE_INTEGER;
export const EPSILON = Number.EPSILON;

// Могут быть как именованные, так и дефолтный экспорт в одном модуле
export { add, subtract as sub, multiply as mul };
```

```javascript
// main.js - импорт модулей
import calculate, { add, subtract, multiply, divide, PI, E } from './math-utils.js';
import { MAX_SAFE_INTEGER, MIN_SAFE_INTEGER, EPSILON } from './constants.js';

// Использование импортированных функций
console.log(add(5, 3)); // 8
console.log(calculate('multiply', 4, 6)); // 24
console.log(PI); // 3.14159

// Импорт всех экспортируемых значений
import * as MathUtils from './math-utils.js';
console.log(MathUtils.add(10, 5)); // 15

// Импорт с переименованием
import { add as sum, subtract as diff } from './math-utils.js';
console.log(sum(7, 3)); // 10
```

### Продвинутые возможности ES6 модулей

```javascript
// async-utils.js - асинхронные операции
export async function fetchData(url) {
    const response = await fetch(url);
    if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
}

export async function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

export async function retryAsync(fn, maxRetries = 3, delayMs = 1000) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            await delay(delayMs * (i + 1)); // экспоненциальная задержка
        }
    }
}

// config.js - конфигурация
const defaultConfig = {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    retries: 3,
    debug: false
};

export { defaultConfig as config };

// Возможность изменять экспорт (в пределах модуля)
export function updateConfig(newConfig) {
    Object.assign(defaultConfig, newConfig);
}
```

```javascript
// api-client.js - клиент API
import { fetchData, retryAsync } from './async-utils.js';
import { config } from './config.js';

class APIClient {
    constructor(options = {}) {
        this.config = { ...config, ...options };
    }
    
    async get(endpoint) {
        return this.makeRequest('GET', endpoint);
    }
    
    async post(endpoint, data) {
        return this.makeRequest('POST', endpoint, data);
    }
    
    async put(endpoint, data) {
        return this.makeRequest('PUT', endpoint, data);
    }
    
    async delete(endpoint) {
        return this.makeRequest('DELETE', endpoint);
    }
    
    async makeRequest(method, endpoint, data = null) {
        const url = `${this.config.apiUrl}${endpoint}`;
        
        const request = async () => {
            const response = await fetch(url, {
                method,
                headers: {
                    'Content-Type': 'application/json',
                    ...this.config.headers
                },
                body: data ? JSON.stringify(data) : null
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            return response.json();
        };
        
        return retryAsync(request, this.config.retries);
    }
}

// Экспорт класса как дефолтного
export default APIClient;

// Также можно экспортировать экземпляр
export const defaultClient = new APIClient();
```

### Динамические импорты

```javascript
// dynamic-imports.js
export async function loadModule(modulePath) {
    try {
        const module = await import(modulePath);
        return module;
    } catch (error) {
        console.error(`Ошибка загрузки модуля ${modulePath}:`, error);
        throw error;
    }
}

// Условная загрузка модулей
export async function getCalculator(type) {
    switch (type) {
        case 'scientific':
            const { default: ScientificCalculator } = await import('./scientific-calculator.js');
            return new ScientificCalculator();
        case 'financial':
            const { default: FinancialCalculator } = await import('./financial-calculator.js');
            return new FinancialCalculator();
        default:
            const { default: BasicCalculator } = await import('./basic-calculator.js');
            return new BasicCalculator();
    }
}

// Ленивая загрузка функциональности
export class LazyFeatureLoader {
    constructor() {
        this.loadedFeatures = new Map();
    }
    
    async loadFeature(featureName) {
        if (this.loadedFeatures.has(featureName)) {
            return this.loadedFeatures.get(featureName);
        }
        
        try {
            let module;
            switch (featureName) {
                case 'charting':
                    module = await import('./charting-module.js');
                    break;
                case 'editor':
                    module = await import('./editor-module.js');
                    break;
                case 'analytics':
                    module = await import('./analytics-module.js');
                    break;
                default:
                    throw new Error(`Неизвестная функция: ${featureName}`);
            }
            
            this.loadedFeatures.set(featureName, module);
            return module;
        } catch (error) {
            console.error(`Ошибка загрузки функции ${featureName}:`, error);
            throw error;
        }
    }
    
    async useFeature(featureName, ...args) {
        const module = await this.loadFeature(featureName);
        
        if (module.default && typeof module.default === 'function') {
            return module.default(...args);
        }
        
        return module;
    }
}
```

## CommonJS

CommonJS - это система модулей, первоначально разработанная для Node.js. В отличие от ES6 модулей, она использует синтаксис require/exports и работает синхронно.

### Основы CommonJS

```javascript
// math-utils.js для CommonJS
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

function divide(a, b) {
    if (b === 0) {
        throw new Error('Деление на ноль');
    }
    return a / b;
}

// Именованный экспорт
module.exports = {
    add,
    subtract,
    multiply,
    divide,
    PI: 3.14159,
    E: 2.71828
};

// Также можно экспортировать по отдельности
// module.exports.add = add;
// module.exports.subtract = subtract;
```

```javascript
// advanced-math.js для CommonJS
const { add, multiply } = require('./math-utils');

// Дефолтный экспорт (в CommonJS это просто свойство module.exports)
function calculate(operation, a, b) {
    switch (operation) {
        case 'add': return add(a, b);
        case 'multiply': return multiply(a, b);
        default: throw new Error('Неизвестная операция');
    }
}

// Экспорт как дефолтного
module.exports = calculate;

// Или экспорт как именованного
module.exports.calculate = calculate;
module.exports.default = calculate;
```

```javascript
// main.js для CommonJS
const mathUtils = require('./math-utils');
const calculate = require('./advanced-math');

// Использование
console.log(mathUtils.add(5, 3)); // 8
console.log(mathUtils.PI); // 3.14159
console.log(calculate('multiply', 4, 6)); // 24

// Деструктуризация импорта
const { add, subtract } = require('./math-utils');
console.log(add(10, 5)); // 15
```

### Продвинутые возможности CommonJS

```javascript
// plugin-system.js для CommonJS
class PluginManager {
    constructor() {
        this.plugins = new Map();
        this.hooks = new Map();
    }
    
    // Регистрация плагина
    registerPlugin(name, pluginModule) {
        if (typeof pluginModule === 'string') {
            // Если передан путь к модулю, загружаем его
            pluginModule = require(pluginModule);
        }
        
        this.plugins.set(name, {
            module: pluginModule,
            name: name,
            enabled: true
        });
        
        // Регистрация хуков плагина
        if (pluginModule.hooks) {
            Object.entries(pluginModule.hooks).forEach(([hookName, handler]) => {
                if (!this.hooks.has(hookName)) {
                    this.hooks.set(hookName, []);
                }
                this.hooks.get(hookName).push(handler);
            });
        }
        
        return this;
    }
    
    // Вызов хуков
    async callHook(hookName, ...args) {
        const hooks = this.hooks.get(hookName) || [];
        const results = [];
        
        for (const hook of hooks) {
            try {
                const result = await Promise.resolve(hook(...args));
                results.push(result);
            } catch (error) {
                console.error(`Ошибка в хуке ${hookName}:`, error);
            }
        }
        
        return results;
    }
    
    // Получение плагина
    getPlugin(name) {
        return this.plugins.get(name);
    }
    
    // Включение/отключение плагина
    setPluginEnabled(name, enabled) {
        const plugin = this.plugins.get(name);
        if (plugin) {
            plugin.enabled = enabled;
        }
        return this;
    }
}

// Экспорт класса
module.exports = PluginManager;
module.exports.default = PluginManager;
```

```javascript
// plugin-example.js - пример плагина для CommonJS
function initialize(api) {
    console.log('Плагин инициализирован');
    
    // Регистрация хуков
    return {
        hooks: {
            'beforeRequest': (request) => {
                console.log('Перед запросом:', request.url);
            },
            'afterResponse': (response) => {
                console.log('После ответа:', response.status);
            }
        }
    };
}

// Экспорт инициализационной функции
module.exports = initialize;
```

## AMD (Asynchronous Module Definition)

AMD - это система модулей, разработанная для асинхронной загрузки модулей в браузере. Она была популярна до появления ES6 модулей.

### Основы AMD

```javascript
// math-utils.js для AMD
define(['require', 'exports'], function(require, exports) {
    function add(a, b) {
        return a + b;
    }
    
    function multiply(a, b) {
        return a * b;
    }
    
    // Экспорт
    exports.add = add;
    exports.multiply = multiply;
    exports.PI = 3.14159;
    
    // Возврат объекта также работает
    return {
        add: add,
        multiply: multiply,
        PI: 3.14159
    };
});
```

```javascript
// advanced-calculator.js для AMD
define(['./math-utils'], function(mathUtils) {
    function calculate(operation, a, b) {
        switch (operation) {
            case 'add':
                return mathUtils.add(a, b);
            case 'multiply':
                return mathUtils.multiply(a, b);
            default:
                throw new Error('Неизвестная операция');
        }
    }
    
    return {
        calculate: calculate
    };
});
```

```javascript
// main.js для AMD
require(['./advanced-calculator'], function(calc) {
    console.log(calc.calculate('add', 5, 3)); // 8
});
```

### Современное использование AMD

```javascript
// legacy-wrapper.js - обертка для работы с AMD в современных условиях
function createAMDModule(moduleName, dependencies, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(moduleName, dependencies, factory);
    } else if (typeof module === 'object' && module.exports) {
        // CommonJS
        const depModules = dependencies.map(dep => require(dep));
        module.exports = factory(...depModules);
    } else {
        // Глобальный объект
        window[moduleName] = factory();
    }
}

// Использование
createAMDModule('MyLibrary', ['dependency1', 'dependency2'], function(dep1, dep2) {
    return {
        doSomething: function() {
            return dep1.process(dep2.getData());
        }
    };
});
```

## UMD (Universal Module Definition)

UMD объединяет подходы AMD, CommonJS и глобальных переменных, позволяя модулю работать в разных окружениях.

```javascript
// umd-module.js - пример UMD модуля
(function (global, factory) {
    if (typeof exports === 'object' && typeof module !== 'undefined') {
        // CommonJS
        module.exports = factory();
    } else if (typeof define === 'function' && define.amd) {
        // AMD
        define(factory);
    } else {
        // Глобальный объект
        global.MyLibrary = factory();
    }
})(typeof globalThis !== 'undefined' ? globalThis : typeof window !== 'undefined' ? window : typeof global !== 'undefined' ? global : this, function () {
    'use strict';
    
    function MyLibrary() {
        this.version = '1.0.0';
    }
    
    MyLibrary.prototype.hello = function(name) {
        return `Привет, ${name || 'мир'}!`;
    };
    
    MyLibrary.prototype.add = function(a, b) {
        return a + b;
    };
    
    return MyLibrary;
});
```

## Tree Shaking

Tree Shaking - это техника оптимизации, при которой из бандла исключаются неиспользуемые экспорты.

```javascript
// utilities.js - модуль с множеством функций
export function isString(value) {
    return typeof value === 'string';
}

export function isNumber(value) {
    return typeof value === 'number';
}

export function isArray(value) {
    return Array.isArray(value);
}

export function isObject(value) {
    return value !== null && typeof value === 'object' && !Array.isArray(value);
}

export function isFunction(value) {
    return typeof value === 'function';
}

// Эти функции могут не использоваться, и будут исключены при tree shaking
export function complexCalculation(data) {
    // Сложные вычисления
    return data.reduce((sum, item) => sum + item.value, 0);
}

export function heavyDataProcessor(data) {
    // Тяжелая обработка данных
    return data.map(item => ({
        ...item,
        processed: true,
        timestamp: Date.now()
    }));
}

// Только используемые функции будут включены в финальный бандл
export default {
    isString,
    isNumber,
    isArray,
    isObject,
    isFunction
};
```

```javascript
// main.js - используем только часть функций
import { isString, isNumber } from './utilities.js';

// Только isString и isNumber будут включены в бандл
// Остальные функции будут исключены при tree shaking
console.log(isString('hello')); // true
console.log(isNumber(42)); // true
```

## Примеры из промышленной разработки

### Система модулей с горячей заменой

```javascript
// hot-module-replacement.js - система с поддержкой HMR
class ModuleSystem {
    constructor() {
        this.modules = new Map();
        this.dependencies = new Map();
        this.dependents = new Map();
        this.cache = new Map();
        this.hmrEnabled = false;
    }
    
    // Регистрация модуля
    define(moduleId, dependencies, factory) {
        this.modules.set(moduleId, {
            dependencies,
            factory,
            exports: null,
            loaded: false
        });
        
        // Отслеживание зависимостей
        dependencies.forEach(depId => {
            if (!this.dependents.has(depId)) {
                this.dependents.set(depId, new Set());
            }
            this.dependents.get(depId).add(moduleId);
        });
        
        this.dependencies.set(moduleId, new Set(dependencies));
    }
    
    // Получение модуля
    require(moduleId) {
        if (!this.modules.has(moduleId)) {
            throw new Error(`Модуль ${moduleId} не найден`);
        }
        
        const module = this.modules.get(moduleId);
        
        if (module.loaded) {
            return module.exports;
        }
        
        // Создание контекста модуля
        const moduleContext = {
            exports: {},
            require: (depId) => this.require(depId)
        };
        
        // Выполнение фабрики модуля
        const result = module.factory(moduleContext.require, moduleContext.exports, moduleContext);
        
        // Сохранение экспорта
        module.exports = result || moduleContext.exports;
        module.loaded = true;
        
        return module.exports;
    }
    
    // Обновление модуля (для HMR)
    async hotUpdate(moduleId, newFactory, newDependencies = []) {
        if (!this.hmrEnabled) {
            throw new Error('HMR отключен');
        }
        
        // Проверка зависимостей
        const oldModule = this.modules.get(moduleId);
        if (oldModule) {
            // Инвалидация зависимых модулей
            this.invalidateDependents(moduleId);
        }
        
        // Обновление модуля
        this.modules.set(moduleId, {
            dependencies: newDependencies,
            factory: newFactory,
            exports: null,
            loaded: false
        });
        
        // Перезагрузка модуля и его зависимостей
        await this.reloadModule(moduleId);
    }
    
    // Инвалидация зависимых модулей
    invalidateDependents(moduleId) {
        const dependents = this.dependents.get(moduleId) || new Set();
        
        for (const depId of dependents) {
            const depModule = this.modules.get(depId);
            if (depModule && depModule.loaded) {
                depModule.loaded = false;
                depModule.exports = null;
                this.invalidateDependents(depId); // Рекурсивно
            }
        }
    }
    
    // Перезагрузка модуля
    async reloadModule(moduleId) {
        const module = this.modules.get(moduleId);
        if (module) {
            module.loaded = false;
            // Модуль будет перезагружен при следующем require
        }
    }
    
    // Включение HMR
    enableHMR() {
        this.hmrEnabled = true;
    }
    
    // Получение информации о зависимостях
    getDependencyGraph() {
        return {
            modules: Array.from(this.modules.keys()),
            dependencies: Object.fromEntries(
                Array.from(this.dependencies.entries()).map(([id, deps]) => [id, Array.from(deps)])
            )
        };
    }
}

// Использование
const moduleSystem = new ModuleSystem();

// Регистрация модуля
moduleSystem.define('math', [], (require, exports) => {
    exports.add = (a, b) => a + b;
    exports.multiply = (a, b) => a * b;
});

moduleSystem.define('calculator', ['math'], (require, exports) => {
    const math = require('math');
    
    exports.calculate = (op, a, b) => {
        switch (op) {
            case 'add': return math.add(a, b);
            case 'multiply': return math.multiply(a, b);
            default: throw new Error('Unknown operation');
        }
    };
});

// Использование
const calc = moduleSystem.require('calculator');
console.log(calc.calculate('add', 5, 3)); // 8
```

### Модульная система с ленивой загрузкой

```javascript
// lazy-loading-module-system.js
class LazyModuleSystem {
    constructor() {
        this.modules = new Map();
        this.loadingPromises = new Map();
        this.resolvedModules = new Map();
        this.middleware = [];
    }
    
    // Регистрация модуля
    register(moduleId, loader, options = {}) {
        this.modules.set(moduleId, {
            loader,
            options,
            loaded: false,
            exports: null
        });
    }
    
    // Промежуточное ПО
    use(middlewareFn) {
        this.middleware.push(middlewareFn);
    }
    
    // Загрузка модуля
    async load(moduleId) {
        // Применение middleware
        for (const mw of this.middleware) {
            await mw('load', moduleId);
        }
        
        // Проверка кэша
        if (this.resolvedModules.has(moduleId)) {
            return this.resolvedModules.get(moduleId);
        }
        
        // Проверка на параллельные загрузки
        if (this.loadingPromises.has(moduleId)) {
            return this.loadingPromises.get(moduleId);
        }
        
        const moduleInfo = this.modules.get(moduleId);
        if (!moduleInfo) {
            throw new Error(`Модуль ${moduleId} не зарегистрирован`);
        }
        
        if (moduleInfo.loaded) {
            return moduleInfo.exports;
        }
        
        const loadPromise = this.loadModule(moduleInfo);
        this.loadingPromises.set(moduleId, loadPromise);
        
        try {
            const exports = await loadPromise;
            this.resolvedModules.set(moduleId, exports);
            this.loadingPromises.delete(moduleId);
            return exports;
        } catch (error) {
            this.loadingPromises.delete(moduleId);
            throw error;
        }
    }
    
    async loadModule(moduleInfo) {
        const exports = await moduleInfo.loader();
        moduleInfo.exports = exports;
        moduleInfo.loaded = true;
        return exports;
    }
    
    // Параллельная загрузка нескольких модулей
    async loadMultiple(moduleIds) {
        const promises = moduleIds.map(id => this.load(id));
        const results = await Promise.all(promises);
        return Object.fromEntries(moduleIds.map((id, index) => [id, results[index]]));
    }
    
    // Условная загрузка
    async loadIf(condition, moduleId) {
        if (condition) {
            return await this.load(moduleId);
        }
        return null;
    }
    
    // Загрузка с fallback
    async loadWithFallback(primaryId, fallbackId) {
        try {
            return await this.load(primaryId);
        } catch (error) {
            console.warn(`Ошибка загрузки ${primaryId}, используем fallback:`, error);
            return await this.load(fallbackId);
        }
    }
    
    // Очистка кэша
    clearCache() {
        this.resolvedModules.clear();
    }
    
    // Получение статуса модуля
    getModuleStatus(moduleId) {
        const moduleInfo = this.modules.get(moduleId);
        if (!moduleInfo) return 'not_registered';
        
        if (this.resolvedModules.has(moduleId)) return 'loaded';
        if (this.loadingPromises.has(moduleId)) return 'loading';
        return 'registered';
    }
}

// Использование
const lazySystem = new LazyModuleSystem();

// Регистрация модулей с ленивой загрузкой
lazySystem.register('heavy-component', async () => {
    // Имитация асинхронной загрузки тяжелого компонента
    await new Promise(resolve => setTimeout(resolve, 100));
    return {
        render: () => '<div>Тяжелый компонент</div>',
        init: () => console.log('Компонент инициализирован')
    };
});

lazySystem.register('utils', async () => {
    return {
        formatDate: (date) => date.toISOString(),
        capitalize: (str) => str.charAt(0).toUpperCase() + str.slice(1)
    };
});

// Загрузка по требованию
async function initializeApp() {
    // Загрузка утилит при старте
    const utils = await lazySystem.load('utils');
    
    // Загрузка тяжелого компонента только при необходимости
    const userAction = await getUserAction(); // асинхронная операция
    
    if (userAction === 'openModal') {
        const component = await lazySystem.load('heavy-component');
        component.init();
    }
}
```

### Управление зависимостями и версиями

```javascript
// dependency-manager.js
class DependencyManager {
    constructor() {
        this.packages = new Map();
        this.resolved = new Map();
        this.lockfile = null;
    }
    
    // Регистрация пакета
    registerPackage(name, version, resolver) {
        if (!this.packages.has(name)) {
            this.packages.set(name, new Map());
        }
        
        this.packages.get(name).set(version, {
            resolver,
            dependencies: new Map() // name -> version range
        });
    }
    
    // Добавление зависимости
    addDependency(parentName, parentVersion, depName, versionRange) {
        const packageMap = this.packages.get(parentName);
        if (!packageMap) {
            throw new Error(`Пакет ${parentName} не найден`);
        }
        
        const pkg = packageMap.get(parentVersion);
        if (pkg) {
            pkg.dependencies.set(depName, versionRange);
        }
    }
    
    // Разрешение зависимостей
    async resolve(name, versionRange = '*') {
        // Поиск подходящей версии
        const availableVersions = this.packages.get(name);
        if (!availableVersions) {
            throw new Error(`Пакет ${name} не найден`);
        }
        
        // Простая стратегия: использовать последнюю подходящую версию
        const suitableVersion = this.findSuitableVersion(
            Array.from(availableVersions.keys()), 
            versionRange
        );
        
        if (!suitableVersion) {
            throw new Error(`Версия ${versionRange} для пакета ${name} не найдена`);
        }
        
        const cacheKey = `${name}@${suitableVersion}`;
        if (this.resolved.has(cacheKey)) {
            return this.resolved.get(cacheKey);
        }
        
        const pkg = availableVersions.get(suitableVersion);
        const resolvedDeps = {};
        
        // Рекурсивное разрешение зависимостей
        for (const [depName, depRange] of pkg.dependencies) {
            resolvedDeps[depName] = await this.resolve(depName, depRange);
        }
        
        // Загрузка модуля с зависимостями
        const moduleExports = await pkg.resolver(resolvedDeps);
        
        this.resolved.set(cacheKey, moduleExports);
        return moduleExports;
    }
    
    findSuitableVersion(versions, range) {
        // Простая реализация для диапазонов
        if (range === '*') {
            return versions.sort().pop(); // последняя версия
        }
        
        if (range.startsWith('^')) {
            // Совместимость с мажорной версией
            const target = range.slice(1);
            return versions.find(v => this.isCompatible(v, target));
        }
        
        if (versions.includes(range)) {
            return range;
        }
        
        return null;
    }
    
    isCompatible(version, target) {
        // Простая проверка совместимости
        const [vMajor, vMinor] = version.split('.').map(Number);
        const [tMajor, tMinor] = target.split('.').map(Number);
        
        return vMajor === tMajor && vMinor >= tMinor;
    }
    
    // Создание lock-файла
    createLockfile() {
        const lockfile = {};
        
        for (const [name, versions] of this.packages) {
            lockfile[name] = {};
            for (const [version, pkg] of versions) {
                lockfile[name][version] = {
                    dependencies: Object.fromEntries(pkg.dependencies),
                    resolved: this.resolved.has(`${name}@${version}`)
                };
            }
        }
        
        return lockfile;
    }
}

// Использование
const depManager = new DependencyManager();

// Регистрация пакетов
depManager.registerPackage('lodash', '4.17.20', async () => {
    // В реальности это будет загрузка lodash
    return {
        map: (arr, fn) => arr.map(fn),
        filter: (arr, fn) => arr.filter(fn),
        reduce: (arr, fn, init) => arr.reduce(fn, init)
    };
});

depManager.registerPackage('my-utils', '1.0.0', async (deps) => {
    const _ = await deps.lodash;
    
    return {
        processData: (data) => _.map(data, item => ({ ...item, processed: true })),
        filterData: (data, condition) => _.filter(data, condition)
    };
});

depManager.addDependency('my-utils', '1.0.0', 'lodash', '^4.17.0');

// Разрешение зависимостей
async function usePackage() {
    const utils = await depManager.resolve('my-utils', '1.0.0');
    console.log(utils); // объект с функциями
}
```

## Лучшие практики

### 1. Структура модулей

```javascript
// Хорошая структура модуля
// user-service.js
import { API_BASE_URL } from './config.js';
import { validateUser } from './validators.js';
import { formatUser } from './formatters.js';

class UserService {
    static async getUser(id) {
        // реализация
    }
    
    static async createUser(userData) {
        // реализация
    }
}

export default UserService;
export { UserService };
```

### 2. Экспорт конфигурации

```javascript
// config.js - централизованная конфигурация
const config = {
    development: {
        apiUrl: 'http://localhost:3000',
        debug: true
    },
    production: {
        apiUrl: 'https://api.example.com',
        debug: false
    }
};

const currentEnv = process.env.NODE_ENV || 'development';
export default config[currentEnv];
```

### 3. Управление побочными эффектами

```javascript
// side-effects.js - избегаем побочных эффектов при импорте
let initialized = false;

export function initialize() {
    if (initialized) return;
    
    // Побочные эффекты здесь
    console.log('Модуль инициализирован');
    initialized = true;
}

// Только функции экспортируем, не вызываем код при импорте
export default { initialize };
```

## Безопасность

При работе с модульными системами важно учитывать безопасность:

```javascript
// secure-module-loader.js - безопасная загрузка модулей
class SecureModuleLoader {
    constructor() {
        this.allowedPaths = new Set();
        this.loadedModules = new Map();
    }
    
    // Добавление разрешенного пути
    addAllowedPath(path) {
        this.allowedPaths.add(path);
    }
    
    // Безопасная загрузка модуля
    async loadModule(modulePath) {
        // Проверка пути
        if (!this.isAllowedPath(modulePath)) {
            throw new Error(`Недопустимый путь модуля: ${modulePath}`);
        }
        
        // Проверка на попытки выхода за пределы разрешенных директорий
        if (this.containsParentTraversal(modulePath)) {
            throw new Error('Попытка выхода за пределы разрешенной директории');
        }
        
        // Загрузка модуля
        if (this.loadedModules.has(modulePath)) {
            return this.loadedModules.get(modulePath);
        }
        
        try {
            const module = await import(modulePath);
            this.loadedModules.set(modulePath, module);
            return module;
        } catch (error) {
            console.error(`Ошибка загрузки модуля ${modulePath}:`, error);
            throw error;
        }
    }
    
    isAllowedPath(path) {
        for (const allowedPath of this.allowedPaths) {
            if (path.startsWith(allowedPath)) {
                return true;
            }
        }
        return false;
    }
    
    containsParentTraversal(path) {
        return path.includes('../') || path.startsWith('..');
    }
    
    // Очистка кэша
    clearCache() {
        this.loadedModules.clear();
    }
}

// Использование
const secureLoader = new SecureModuleLoader();
secureLoader.addAllowedPath('/app/modules/');
secureLoader.addAllowedPath('/app/components/');

try {
    const module = await secureLoader.loadModule('/app/modules/user-service.js');
} catch (error) {
    console.error('Безопасная ошибка загрузки:', error.message);
}
```

## Теги

#javascript #modules #es6 #commonjs #amd #umd #import #export #tree-shaking #dependencies