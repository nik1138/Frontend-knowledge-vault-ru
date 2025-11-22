---
aliases: ["Дополнительные техники оптимизации JavaScript", "Продвинутые методы оптимизации"]
tags: [js, performance, optimization, frontend]
---

# Дополнительные техники оптимизации

Расширенные методы и техники оптимизации производительности JavaScript приложений, включая алгоритмические оптимизации, работу с памятью и эффективные паттерны программирования.

## 1. Алгоритмические оптимизации

### Оптимизация поиска и сортировки

```javascript
// Быстрый поиск в отсортированном массиве (бинарный поиск)
function binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        const midValue = arr[mid];
        
        if (midValue === target) {
            return mid; // Найден элемент
        } else if (midValue < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return -1; // Элемент не найден
}

// Поиск с использованием Set для O(1) времени
function fastArraySearch(arr, target) {
    const set = new Set(arr);
    return set.has(target);
}

// Сортировка подсчетом для целых чисел в ограниченном диапазоне
function countingSort(arr, max) {
    const count = new Array(max + 1).fill(0);
    const output = [];
    
    // Подсчет вхождений
    for (const num of arr) {
        count[num]++;
    }
    
    // Восстановление отсортированного массива
    for (let i = 0; i <= max; i++) {
        while (count[i] > 0) {
            output.push(i);
            count[i]--;
        }
    }
    
    return output;
}

// Примеры использования
const sortedArray = [1, 3, 5, 7, 9, 11, 13, 15];
console.log(binarySearch(sortedArray, 7)); // 3

const largeArray = Array.from({ length: 1000000 }, (_, i) => i);
const searchSet = new Set(largeArray);
console.time('Set search');
console.log(searchSet.has(999999)); // true
console.timeEnd('Set search');

console.time('Binary search');
console.log(binarySearch(sortedArray, 13)); // 6
console.timeEnd('Binary search');
```

### Эффективная работа с коллекциями

```javascript
// Оптимизированный способ объединения массивов
function efficientArrayMerge(arr1, arr2) {
    // Для небольших массивов используем spread
    if (arr1.length + arr2.length < 10000) {
        return [...arr1, ...arr2];
    }
    
    // Для больших массивов используем push.apply
    const result = new Array(arr1.length + arr2.length);
    let i = 0;
    
    for (let j = 0; j < arr1.length; j++) {
        result[i++] = arr1[j];
    }
    for (let j = 0; j < arr2.length; j++) {
        result[i++] = arr2[j];
    }
    
    return result;
}

// Оптимизированное объединение объектов
function efficientObjectMerge(...objects) {
    const result = {};
    
    // Используем Object.assign для простого случая
    if (objects.length === 2) {
        return Object.assign({}, objects[0], objects[1]);
    }
    
    // Для большего количества объектов используем цикл
    for (const obj of objects) {
        for (const key in obj) {
            if (obj.hasOwnProperty(key)) {
                result[key] = obj[key];
            }
        }
    }
    
    return result;
}

// Оптимизированное объединение массивов с удалением дубликатов
function mergeUniqueArrays(...arrays) {
    const seen = new Set();
    const result = [];
    
    for (const arr of arrays) {
        for (const item of arr) {
            if (!seen.has(item)) {
                seen.add(item);
                result.push(item);
            }
        }
    }
    
    return result;
}

// Группировка элементов с минимальным количеством итераций
function groupByOptimized(array, keyFunction) {
    const groups = new Map();
    
    for (const item of array) {
        const key = keyFunction(item);
        if (!groups.has(key)) {
            groups.set(key, []);
        }
        groups.get(key).push(item);
    }
    
    return Object.fromEntries(groups);
}

// Примеры использования
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
console.time('Efficient merge');
const merged = efficientArrayMerge(arr1, arr2);
console.timeEnd('Efficient merge');

const grouped = groupByOptimized(
    [{ type: 'A', value: 1 }, { type: 'B', value: 2 }, { type: 'A', value: 3 }],
    item => item.type
);
console.log(grouped); // { A: [{type: 'A', value: 1}, {type: 'A', value: 3}], B: [{type: 'B', value: 2}] }
```

## 2. Оптимизация памяти

### Управление ссылками и утечками памяти

```javascript
// Правильное управление событиями для предотвращения утечек
class EventManager {
    constructor() {
        this.eventListeners = new Map();
    }
    
    // Добавление обработчика с автоматическим удалением
    addListener(element, event, handler, autoCleanup = true) {
        const listener = { element, event, handler };
        element.addEventListener(event, handler);
        
        if (autoCleanup) {
            if (!this.eventListeners.has(element)) {
                this.eventListeners.set(element, []);
            }
            this.eventListeners.get(element).push(listener);
        }
        
        return () => this.removeListener(element, event, handler);
    }
    
    removeListener(element, event, handler) {
        element.removeEventListener(event, handler);
        
        if (this.eventListeners.has(element)) {
            const listeners = this.eventListeners.get(element);
            const index = listeners.findIndex(l => 
                l.event === event && l.handler === handler
            );
            if (index > -1) {
                listeners.splice(index, 1);
            }
        }
    }
    
    // Очистка всех обработчиков
    cleanup() {
        for (const [element, listeners] of this.eventListeners) {
            listeners.forEach(listener => {
                element.removeEventListener(listener.event, listener.handler);
            });
        }
        this.eventListeners.clear();
    }
}

// Класс с автоматической очисткой ресурсов
class ResourceHandler {
    constructor() {
        this.resources = new Set();
        this.timers = new Set();
        this.observer = null;
    }
    
    // Добавление ресурса для отслеживания
    addResource(resource) {
        this.resources.add(resource);
        return resource;
    }
    
    // Создание таймера с автоматическим отслеживанием
    setTimeout(callback, delay) {
        const timerId = setTimeout(() => {
            callback();
            this.timers.delete(timerId);
        }, delay);
        
        this.timers.add(timerId);
        return timerId;
    }
    
    setInterval(callback, interval) {
        const timerId = setInterval(callback, interval);
        this.timers.add(timerId);
        return timerId;
    }
    
    // Очистка всех ресурсов
    cleanup() {
        // Очистка таймеров
        for (const timerId of this.timers) {
            if (typeof timerId === 'number') {
                clearTimeout(timerId);
                clearInterval(timerId);
            }
        }
        this.timers.clear();
        
        // Очистка ресурсов
        for (const resource of this.resources) {
            if (resource && typeof resource.destroy === 'function') {
                resource.destroy();
            }
        }
        this.resources.clear();
        
        // Остановка observer'ов
        if (this.observer) {
            this.observer.disconnect();
            this.observer = null;
        }
    }
}

// Использование WeakMap для избежания утечек памяти
const elementData = new WeakMap();

function attachData(element, data) {
    elementData.set(element, data);
}

function getData(element) {
    return elementData.get(element);
}

// Пример использования
const handler = new ResourceHandler();
const cleanupBtn = document.createElement('button');
handler.addResource(cleanupBtn);

handler.setTimeout(() => {
    console.log('Таймер завершен');
}, 5000);
```

### Оптимизация создания объектов

```javascript
// Использование объектных пулов для частого создания/удаления объектов
class ObjectPool {
    constructor(createFn, resetFn, initialSize = 10) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.pool = [];
        
        // Инициализация пула
        for (let i = 0; i < initialSize; i++) {
            this.pool.push(this.createFn());
        }
    }
    
    acquire() {
        if (this.pool.length > 0) {
            return this.pool.pop();
        }
        return this.createFn();
    }
    
    release(obj) {
        this.resetFn(obj);
        this.pool.push(obj);
    }
    
    size() {
        return this.pool.length;
    }
}

// Пул для объектов частиц
const particlePool = new ObjectPool(
    () => ({ x: 0, y: 0, vx: 0, vy: 0, life: 0 }),
    (obj) => {
        obj.x = obj.y = obj.vx = obj.vy = 0;
        obj.life = 100;
    }
);

// Класс для создания частиц
class ParticleSystem {
    constructor() {
        this.particles = [];
    }
    
    createParticle(x, y, vx, vy) {
        const particle = particlePool.acquire();
        particle.x = x;
        particle.y = y;
        particle.vx = vx;
        particle.vy = vy;
        particle.life = 100;
        
        this.particles.push(particle);
        return particle;
    }
    
    update() {
        for (let i = this.particles.length - 1; i >= 0; i--) {
            const p = this.particles[i];
            p.x += p.vx;
            p.y += p.vy;
            p.life--;
            
            if (p.life <= 0) {
                particlePool.release(this.particles.splice(i, 1)[0]);
            }
        }
    }
}

// Использование конструкторов вместо литералов для больших объектов
function createLargeObjectConstructor() {
    function LargeObject(prop1, prop2, prop3, prop4, prop5) {
        this.prop1 = prop1;
        this.prop2 = prop2;
        this.prop3 = prop3;
        this.prop4 = prop4;
        this.prop5 = prop5;
    }
    
    return function(prop1, prop2, prop3, prop4, prop5) {
        return new LargeObject(prop1, prop2, prop3, prop4, prop5);
    };
}

const createLargeObject = createLargeObjectConstructor();
const obj1 = createLargeObject(1, 2, 3, 4, 5);
```

## 3. Оптимизация DOM-операций

### Эффективная работа с DOM

```javascript
// Батчевое обновление DOM
class DOMBatchUpdater {
    constructor() {
        this.updates = [];
        this.isBatching = false;
    }
    
    queueUpdate(updateFn) {
        this.updates.push(updateFn);
        
        if (!this.isBatching) {
            this.isBatching = true;
            requestAnimationFrame(() => this.processUpdates());
        }
    }
    
    processUpdates() {
        // Выполняем все обновления в одном цикле
        for (const update of this.updates) {
            update();
        }
        
        this.updates = [];
        this.isBatching = false;
    }
    
    // Метод для обновления нескольких элементов
    updateMultiple(elements, updateFn) {
        this.queueUpdate(() => {
            for (const element of elements) {
                updateFn(element);
            }
        });
    }
}

// Использование DocumentFragment для эффективного добавления элементов
function efficientAppend(parent, items) {
    const fragment = document.createDocumentFragment();
    
    for (const item of items) {
        const element = document.createElement('div');
        element.textContent = item;
        fragment.appendChild(element);
    }
    
    parent.appendChild(fragment);
}

// Виртуальный скроллинг для больших списков
class VirtualScroller {
    constructor(container, itemHeight, itemCount, renderItem) {
        this.container = container;
        this.itemHeight = itemHeight;
        this.itemCount = itemCount;
        this.renderItem = renderItem;
        this.visibleStart = 0;
        this.visibleEnd = 0;
        this.items = new Map();
        
        this.setupContainer();
        this.bindEvents();
        this.render();
    }
    
    setupContainer() {
        this.container.style.height = `${this.itemCount * this.itemHeight}px`;
        this.container.style.position = 'relative';
        this.container.style.overflow = 'auto';
    }
    
    bindEvents() {
        this.container.addEventListener('scroll', () => {
            this.updateVisibleRange();
            this.render();
        });
    }
    
    updateVisibleRange() {
        const scrollTop = this.container.scrollTop;
        const containerHeight = this.container.clientHeight;
        
        this.visibleStart = Math.floor(scrollTop / this.itemHeight);
        this.visibleEnd = Math.min(
            this.itemCount,
            Math.ceil((scrollTop + containerHeight) / this.itemHeight) + 1
        );
    }
    
    render() {
        this.updateVisibleRange();
        
        // Удаляем невидимые элементы
        for (const [index, element] of this.items) {
            if (index < this.visibleStart || index >= this.visibleEnd) {
                element.remove();
                this.items.delete(index);
            }
        }
        
        // Добавляем видимые элементы
        for (let i = this.visibleStart; i < this.visibleEnd; i++) {
            if (!this.items.has(i)) {
                const element = this.renderItem(i);
                element.style.position = 'absolute';
                element.style.top = `${i * this.itemHeight}px`;
                element.style.left = '0';
                element.style.width = '100%';
                
                this.container.appendChild(element);
                this.items.set(i, element);
            }
        }
    }
}

// Пример использования виртуального скроллинга
function createVirtualList(container, items) {
    return new VirtualScroller(
        container,
        50, // высота элемента
        items.length,
        (index) => {
            const div = document.createElement('div');
            div.textContent = items[index];
            div.style.height = '50px';
            div.style.borderBottom = '1px solid #ccc';
            return div;
        }
    );
}
```

### Оптимизация событий

```javascript
// Делегирование событий для динамических элементов
class EventDelegator {
    constructor(container) {
        this.container = container;
        this.listeners = new Map();
        this.setupDelegation();
    }
    
    setupDelegation() {
        // Поддерживаемые события
        const events = ['click', 'mouseover', 'mouseout', 'focus', 'blur'];
        
        for (const event of events) {
            this.container.addEventListener(event, (e) => {
                this.handleEvent(e, event);
            });
        }
    }
    
    on(selector, event, handler) {
        const key = `${event}:${selector}`;
        
        if (!this.listeners.has(key)) {
            this.listeners.set(key, []);
        }
        
        this.listeners.get(key).push(handler);
    }
    
    handleEvent(event, eventType) {
        let target = event.target;
        
        // Проверяем все зарегистрированные селекторы
        for (const [key, handlers] of this.listeners) {
            if (key.startsWith(eventType + ':')) {
                const selector = key.substring(eventType.length + 1);
                
                // Проверяем, соответствует ли элемент селектору
                if (target.matches && target.matches(selector)) {
                    for (const handler of handlers) {
                        handler(event);
                    }
                } else if (target.closest && target.closest(selector)) {
                    // Проверяем родительские элементы
                    for (const handler of handlers) {
                        handler(event);
                    }
                }
            }
        }
    }
}

// Оптимизация обработки событий с использованием throttling
function createOptimizedEventHandler(handler, delay = 16) {
    let timeoutId = null;
    let lastExecTime = 0;
    
    return function(...args) {
        const currentTime = Date.now();
        
        if (currentTime - lastExecTime >= delay) {
            handler.apply(this, args);
            lastExecTime = currentTime;
        } else {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => {
                handler.apply(this, args);
                lastExecTime = Date.now();
            }, delay - (currentTime - lastExecTime));
        }
    };
}

// Пример использования
const optimizedScrollHandler = createOptimizedEventHandler(() => {
    console.log('Scroll event handled');
}, 100); // Обработка не чаще раза в 100мс

// window.addEventListener('scroll', optimizedScrollHandler);
```

## 4. Оптимизация функций и замыканий

### Эффективная работа с функциями

```javascript
// Memoization для дорогостоящих вычислений
class Memoizer {
    constructor() {
        this.cache = new Map();
    }
    
    memoize(fn, keyFn = JSON.stringify) {
        return (...args) => {
            const key = keyFn(args);
            
            if (this.cache.has(key)) {
                return this.cache.get(key);
            }
            
            const result = fn.apply(this, args);
            this.cache.set(key, result);
            return result;
        };
    }
    
    clear() {
        this.cache.clear();
    }
}

const memoizer = new Memoizer();

// Пример дорогостоящей функции
const expensiveCalculation = memoizer.memoize((n) => {
    let result = 0;
    for (let i = 0; i < n; i++) {
        result += Math.sqrt(i) * Math.sin(i);
    }
    return result;
});

// Оптимизация вызовов методов
class OptimizedClass {
    constructor() {
        // Привязываем методы к экземпляру при создании
        this.boundMethod = this.method.bind(this);
        this.boundArrowMethod = this.arrowMethod.bind(this);
    }
    
    method(param) {
        return param * 2;
    }
    
    arrowMethod = (param) => {
        return param * 3;
    }
    
    // Используем мемоизацию для методов, которые часто вызываются с одинаковыми параметрами
    getMemoizedMethod() {
        if (!this._memoizedMethod) {
            this._memoizedMethod = memoizer.memoize(this.method.bind(this));
        }
        return this._memoizedMethod;
    }
}

// Оптимизация рекурсивных функций с мемоизацией
function fibonacciMemoized() {
    const cache = new Map();
    
    function fib(n) {
        if (cache.has(n)) {
            return cache.get(n);
        }
        
        let result;
        if (n <= 1) {
            result = n;
        } else {
            result = fib(n - 1) + fib(n - 2);
        }
        
        cache.set(n, result);
        return result;
    }
    
    return fib;
}

const fib = fibonacciMemoized();
console.time('Fibonacci with memoization');
console.log(fib(50)); // Вычисляется быстро благодаря мемоизации
console.timeEnd('Fibonacci with memoization');
```

### Работа с замыканиями

```javascript
// Оптимизация замыканий для избежания создания лишних функций
class ClosureOptimizer {
    constructor() {
        // Создаем общие функции-обработчики
        this.commonHandlers = {
            click: this.createHandler('click'),
            change: this.createHandler('change'),
            keyup: this.createHandler('keyup')
        };
    }
    
    createHandler(type) {
        return (event) => {
            // Общий обработчик для всех элементов одного типа
            console.log(`${type} event:`, event.target);
        };
    }
    
    // Используем общие замыкания вместо создания уникальных для каждого элемента
    attachCommonHandler(element, type) {
        element.addEventListener(type, this.commonHandlers[type]);
    }
    
    // Оптимизация с параметрами
    createParameterizedHandler(callback, params) {
        // Вместо создания новой функции для каждого вызова, используем один шаблон
        return function(event) {
            callback.call(this, event, params);
        };
    }
}

// Использование IIFE для избежания утечек памяти
function createOptimizedLoop() {
    const elements = document.querySelectorAll('.item');
    
    for (let i = 0; i < elements.length; i++) {
        // Используем IIFE для захвата правильного значения i
        (function(index) {
            elements[i].addEventListener('click', function() {
                console.log(`Clicked item ${index}`);
            });
        })(i);
    }
}

// Более эффективный способ - с использованием let
function createOptimizedLoopWithLet() {
    const elements = document.querySelectorAll('.item');
    
    for (let i = 0; i < elements.length; i++) {
        elements[i].addEventListener('click', function() {
            console.log(`Clicked item ${i}`); // let создает новую область видимости на каждой итерации
        });
    }
}
```

## 5. Асинхронные оптимизации

### Оптимизация асинхронных операций

```javascript
// Оптимизация Promise и async/await
class AsyncOptimizer {
    // Параллельное выполнение независимых операций
    async executeParallel(tasks) {
        const promises = tasks.map(task => this.safeExecute(task));
        return Promise.all(promises);
    }
    
    // Последовательное выполнение с возможностью отмены
    async executeSequential(tasks) {
        const results = [];
        
        for (const task of tasks) {
            try {
                const result = await this.safeExecute(task);
                results.push(result);
            } catch (error) {
                console.error('Task failed:', error);
                // В зависимости от требований, можно продолжить или прервать
                break;
            }
        }
        
        return results;
    }
    
    // Безопасное выполнение задачи с обработкой ошибок
    async safeExecute(task) {
        try {
            return await task();
        } catch (error) {
            console.error('Task execution failed:', error);
            throw error;
        }
    }
    
    // Ограничение количества одновременных запросов
    async executeWithConcurrencyLimit(tasks, limit = 3) {
        const results = new Array(tasks.length);
        const executing = [];
        
        for (let i = 0; i < tasks.length; i++) {
            const promise = this.safeExecute(tasks[i]).then(result => {
                results[i] = result;
            });
            
            executing.push(promise);
            
            if (executing.length >= limit) {
                await Promise.race(executing);
                executing.splice(executing.findIndex(p => p === promise), 1);
            }
        }
        
        await Promise.all(executing);
        return results;
    }
    
    // Таймаут для асинхронных операций
    async withTimeout(promise, timeoutMs) {
        const timeoutPromise = new Promise((_, reject) => {
            setTimeout(() => reject(new Error('Operation timed out')), timeoutMs);
        });
        
        return Promise.race([promise, timeoutPromise]);
    }
}

// Оптимизация загрузки данных
class DataLoader {
    constructor() {
        this.cache = new Map();
        this.pendingRequests = new Map();
    }
    
    // Загрузка с кэшированием и предотвращением дубликатов запросов
    async load(url) {
        // Проверяем кэш
        if (this.cache.has(url)) {
            return this.cache.get(url);
        }
        
        // Проверяем, есть ли уже ожидающий запрос
        if (this.pendingRequests.has(url)) {
            return this.pendingRequests.get(url);
        }
        
        // Создаем новый запрос
        const promise = fetch(url).then(response => response.json());
        
        // Сохраняем промис в pending
        this.pendingRequests.set(url, promise);
        
        try {
            const data = await promise;
            this.cache.set(url, data);
            this.pendingRequests.delete(url);
            return data;
        } catch (error) {
            this.pendingRequests.delete(url);
            throw error;
        }
    }
    
    // Предварительная загрузка данных
    async preload(urls) {
        const promises = urls.map(url => this.load(url));
        return Promise.all(promises);
    }
    
    clearCache() {
        this.cache.clear();
    }
}

// Пример использования
const asyncOptimizer = new AsyncOptimizer();
const dataLoader = new DataLoader();

// asyncOptimizer.executeWithConcurrencyLimit([
//     () => fetch('/api/data1').then(r => r.json()),
//     () => fetch('/api/data2').then(r => r.json()),
//     () => fetch('/api/data3').then(r => r.json()),
//     () => fetch('/api/data4').then(r => r.json()),
// ], 2); // Максимум 2 одновременных запроса
```

## 6. Профилирование и мониторинг

### Инструменты для измерения производительности

```javascript
// Утилита для профилирования функций
class PerformanceProfiler {
    constructor() {
        this.metrics = new Map();
        this.runningTimers = new Map();
    }
    
    // Измерение времени выполнения функции
    async measureAsync(name, fn) {
        const startTime = performance.now();
        let result;
        
        try {
            result = await fn();
        } finally {
            const endTime = performance.now();
            const duration = endTime - startTime;
            
            this.recordMetric(name, duration);
        }
        
        return result;
    }
    
    // Измерение синхронной функции
    measure(name, fn) {
        const startTime = performance.now();
        const result = fn();
        const endTime = performance.now();
        const duration = endTime - startTime;
        
        this.recordMetric(name, duration);
        return result;
    }
    
    // Запуск таймера
    startTimer(name) {
        this.runningTimers.set(name, performance.now());
    }
    
    // Остановка таймера
    stopTimer(name) {
        if (this.runningTimers.has(name)) {
            const startTime = this.runningTimers.get(name);
            const duration = performance.now() - startTime;
            this.recordMetric(name, duration);
            this.runningTimers.delete(name);
            return duration;
        }
        return null;
    }
    
    recordMetric(name, duration) {
        if (!this.metrics.has(name)) {
            this.metrics.set(name, {
                count: 0,
                total: 0,
                min: Infinity,
                max: 0
            });
        }
        
        const metric = this.metrics.get(name);
        metric.count++;
        metric.total += duration;
        metric.min = Math.min(metric.min, duration);
        metric.max = Math.max(metric.max, duration);
    }
    
    // Получение статистики
    getStats() {
        const stats = {};
        
        for (const [name, metric] of this.metrics) {
            stats[name] = {
                avg: metric.total / metric.count,
                min: metric.min,
                max: metric.max,
                count: metric.count,
                total: metric.total
            };
        }
        
        return stats;
    }
    
    // Сброс метрик
    reset() {
        this.metrics.clear();
        this.runningTimers.clear();
    }
    
    // Вывод отчета
    report() {
        const stats = this.getStats();
        console.table(stats);
    }
}

// Мониторинг производительности в реальном времени
class RealTimePerformanceMonitor {
    constructor(options = {}) {
        this.options = {
            interval: 1000, // 1 секунда
            reportCallback: console.log,
            ...options
        };
        
        this.observer = null;
        this.isEnabled = false;
    }
    
    start() {
        if (this.isEnabled) return;
        
        this.isEnabled = true;
        this.collectPerformanceData();
        
        // Используем PerformanceObserver для мониторинга
        if ('PerformanceObserver' in window) {
            this.observer = new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    if (entry.entryType === 'measure' || entry.entryType === 'mark') {
                        this.options.reportCallback({
                            type: entry.entryType,
                            name: entry.name,
                            duration: entry.duration || 0,
                            startTime: entry.startTime
                        });
                    }
                }
            });
            
            this.observer.observe({ entryTypes: ['measure', 'mark', 'navigation', 'paint'] });
        }
        
        // Регулярный опрос основных метрик
        this.pollInterval = setInterval(() => {
            if (this.isEnabled) {
                this.collectPerformanceData();
            }
        }, this.options.interval);
    }
    
    collectPerformanceData() {
        if (performance.memory) {
            this.options.reportCallback({
                type: 'memory',
                used: performance.memory.usedJSHeapSize,
                total: performance.memory.totalJSHeapSize,
                limit: performance.memory.jsHeapSizeLimit
            });
        }
        
        // Метрики производительности
        const perfData = {
            type: 'performance',
            timeOrigin: performance.timeOrigin,
            memory: performance.memory ? {
                used: performance.memory.usedJSHeapSize,
                total: performance.memory.totalJSHeapSize,
                limit: performance.memory.jsHeapSizeLimit
            } : null,
            timing: performance.timing ? {
                loadTime: performance.timing.loadEventEnd - performance.timing.navigationStart,
                domReady: performance.timing.domContentLoadedEventEnd - performance.timing.navigationStart,
                firstPaint: performance.getEntriesByName('first-paint')[0]?.startTime,
                domInteractive: performance.timing.domInteractive - performance.timing.navigationStart
            } : null
        };
        
        this.options.reportCallback(perfData);
    }
    
    stop() {
        this.isEnabled = false;
        
        if (this.pollInterval) {
            clearInterval(this.pollInterval);
        }
        
        if (this.observer) {
            this.observer.disconnect();
        }
    }
}

// Пример использования
const profiler = new PerformanceProfiler();

// profiler.startTimer('dataProcessing');
// // Выполняем какую-то работу
// const result = profiler.measure('arraySort', () => {
//     return largeArray.sort((a, b) => a - b);
// });
// profiler.stopTimer('dataProcessing');

// profiler.report();
```

Эти дополнительные техники оптимизации помогут вам создавать более производительные JavaScript приложения, улучшая как общую скорость работы, так и эффективность использования ресурсов.

См. также:
- [[js/performance/comparison-techniques]] - Сравнение производительности разных подходов
- [[js/performance/profiling-tips]] - Практические рекомендации по профилированию
- [[js/tricks/arrays-tricks]] - Хитрости работы с массивами
- [[js/tricks/objects-tricks]] - Работа с объектами