# Производительность JavaScript: оптимизация, профилирование и лучшие практики

## Введение

Производительность JavaScript критически важна для создания быстрых, отзывчивых и эффективных веб-приложений. В этой статье мы рассмотрим ключевые аспекты оптимизации JavaScript-кода, методы профилирования и лучшие практики для достижения максимальной производительности.

## Основы производительности JavaScript

### 1. Понимание движка JavaScript

JavaScript-движки (V8, SpiderMonkey, JavaScriptCore) выполняют несколько этапов обработки кода:

1. **Парсинг** - преобразование кода в AST (Abstract Syntax Tree)
2. **Компиляция** - преобразование в байт-код
3. **Оптимизация** - JIT (Just-In-Time) компиляция и оптимизации
4. **Выполнение** - выполнение скомпилированного кода

```javascript
// Пример, демонстрирующий влияние на оптимизации
// Плохо: изменение типа переменной
function badExample() {
    let value = 1;        // number
    value = 'string';     // string - нарушает оптимизации
    value = true;         // boolean - еще больше усложняет
    return value;
}

// Хорошо: предсказуемые типы
function goodExample() {
    let numberValue = 1;
    let stringValue = 'string';
    let booleanValue = true;
    return { numberValue, stringValue, booleanValue };
}

// Плохо: изменение формы объекта
function badObjectExample() {
    const obj = { a: 1, b: 2 };
    obj.c = 3; // Добавление свойства после создания
    return obj;
}

// Хорошо: предсказуемая форма объекта
function goodObjectExample() {
    return { a: 1, b: 2, c: 3 }; // Все свойства определены сразу
}
```

### 2. Измерение производительности

```javascript
// Использование Performance API
function measureFunctionPerformance(fn, ...args) {
    const start = performance.now();
    const result = fn.apply(null, args);
    const end = performance.now();
    
    console.log(`${fn.name} выполнена за ${(end - start).toFixed(2)}ms`);
    return result;
}

// Сравнение производительности
function performanceComparison() {
    const iterations = 1000000;
    
    // Тест 1: цикл for
    console.time('for loop');
    for (let i = 0; i < iterations; i++) {
        // пустое тело
    }
    console.timeEnd('for loop');
    
    // Тест 2: цикл while
    console.time('while loop');
    let j = 0;
    while (j < iterations) {
        j++;
    }
    console.timeEnd('while loop');
    
    // Тест 3: метод массива
    console.time('array.forEach');
    new Array(iterations).fill(0).forEach(() => {});
    console.timeEnd('array.forEach');
}

// Более точное измерение с PerformanceObserver
class PerformanceMonitor {
    constructor() {
        this.entries = [];
        this.init();
    }
    
    init() {
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    this.entries.push(entry);
                    console.log(`${entry.name}: ${entry.duration}ms`);
                }
            });
            
            observer.observe({ entryTypes: ['measure', 'navigation'] });
        }
    }
    
    measure(name, fn) {
        performance.mark(`${name}-start`);
        const result = fn();
        performance.mark(`${name}-end`);
        performance.measure(name, `${name}-start`, `${name}-end`);
        return result;
    }
}
```

## Оптимизация алгоритмов и структур данных

### 1. Выбор правильных структур данных

```javascript
// Массивы vs Объекты (Map) для поиска
function arrayVsMapComparison() {
    const array = [];
    const map = new Map();
    const iterations = 100000;
    
    // Заполнение данных
    for (let i = 0; i < iterations; i++) {
        array.push({ id: i, value: `value${i}` });
        map.set(i, `value${i}`);
    }
    
    // Поиск в массиве (O(n))
    console.time('Array search');
    const foundInArray = array.find(item => item.id === 50000);
    console.timeEnd('Array search');
    
    // Поиск в Map (O(1))
    console.time('Map search');
    const foundInMap = map.get(50000);
    console.timeEnd('Map search');
    
    // Вставка
    console.time('Array insert');
    array.push({ id: iterations, value: 'new' });
    console.timeEnd('Array insert');
    
    console.time('Map insert');
    map.set(iterations, 'new');
    console.timeEnd('Map insert');
}

// Использование Set для уникальных значений
function uniqueValuesExample() {
    // Плохо: с использованием массива
    function getUniqueWithArray(arr) {
        const unique = [];
        for (const item of arr) {
            if (!unique.includes(item)) {
                unique.push(item);
            }
        }
        return unique;
    }
    
    // Хорошо: с использованием Set
    function getUniqueWithSet(arr) {
        return [...new Set(arr)];
    }
    
    const largeArray = Array.from({ length: 100000 }, () => Math.floor(Math.random() * 10000));
    
    console.time('Array method');
    getUniqueWithArray(largeArray);
    console.timeEnd('Array method');
    
    console.time('Set method');
    getUniqueWithSet(largeArray);
    console.timeEnd('Set method');
}

// Использование WeakMap и WeakSet
class EfficientCache {
    constructor() {
        this.cache = new Map();
        this.references = new WeakMap(); // Автоматическая очистка
    }
    
    set(key, value) {
        this.cache.set(key, value);
        this.references.set(value, key); // Слабая ссылка
    }
    
    get(key) {
        return this.cache.get(key);
    }
    
    // Очистка устаревших ссылок
    cleanup() {
        // WeakMap автоматически очищает недоступные объекты
    }
}
```

### 2. Оптимизация алгоритмов

```javascript
// Плохо: O(n²) алгоритм
function badNestedLoop(arr1, arr2) {
    const result = [];
    for (let i = 0; i < arr1.length; i++) {
        for (let j = 0; j < arr2.length; j++) {
            if (arr1[i] === arr2[j]) {
                result.push(arr1[i]);
            }
        }
    }
    return result;
}

// Хорошо: O(n) алгоритм с использованием Set
function goodOptimized(arr1, arr2) {
    const set2 = new Set(arr2);
    return arr1.filter(item => set2.has(item));
}

// Оптимизация сортировки
function sortingOptimizations() {
    const largeArray = Array.from({ length: 100000 }, () => ({
        id: Math.floor(Math.random() * 100000),
        value: Math.random()
    }));
    
    // Плохо: каждый раз вычисляем длину
    function badSort(arr) {
        for (let i = 0; i < arr.length; i++) { // length вычисляется каждый раз
            for (let j = 0; j < arr.length - i - 1; j++) {
                if (arr[j].id > arr[j + 1].id) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                }
            }
        }
    }
    
    // Хорошо: кешируем длину
    function goodSort(arr) {
        const len = arr.length; // Кешируем длину
        for (let i = 0; i < len; i++) {
            for (let j = 0; j < len - i - 1; j++) {
                if (arr[j].id > arr[j + 1].id) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                }
            }
        }
    }
    
    // Лучше: использовать встроенные методы
    function bestSort(arr) {
        return arr.sort((a, b) => a.id - b.id);
    }
    
    // Измерение производительности
    const arr1 = [...largeArray];
    const arr2 = [...largeArray];
    const arr3 = [...largeArray];
    
    console.time('Bad sort');
    badSort(arr1);
    console.timeEnd('Bad sort');
    
    console.time('Good sort');
    goodSort(arr2);
    console.timeEnd('Good sort');
    
    console.time('Best sort');
    bestSort(arr3);
    console.timeEnd('Best sort');
}
```

## Оптимизация DOM-операций

### 1. Эффективная работа с DOM

```javascript
// Плохо: частые DOM-операции
function badDOMOperations() {
    const container = document.getElementById('container');
    
    // Плохо: каждый appendChild вызывает перерисовку
    for (let i = 0; i < 1000; i++) {
        const div = document.createElement('div');
        div.textContent = `Item ${i}`;
        container.appendChild(div); // Вызывает перерисовку каждый раз
    }
}

// Хорошо: буферизация DOM-операций
function goodDOMOperations() {
    const container = document.getElementById('container');
    const fragment = document.createDocumentFragment();
    
    // Создаем все элементы в памяти
    for (let i = 0; i < 1000; i++) {
        const div = document.createElement('div');
        div.textContent = `Item ${i}`;
        fragment.appendChild(div);
    }
    
    // Один раз добавляем в DOM
    container.appendChild(fragment);
}

// Оптимизация доступа к DOM
function DOMAccessOptimizations() {
    const container = document.getElementById('container');
    
    // Плохо: частый доступ к DOM
    function badExample() {
        for (let i = 0; i < 100; i++) {
            container.style.left = i + 'px';  // Частый доступ к DOM
            container.style.top = i + 'px';   // Частый доступ к DOM
        }
    }
    
    // Хорошо: группировка изменений
    function goodExample() {
        const styles = [];
        for (let i = 0; i < 100; i++) {
            styles.push(`${i}px`);
        }
        
        // Применяем изменения за раз
        container.style.transform = `translate(${styles[styles.length - 1]}, ${styles[styles.length - 1]})`;
    }
    
    // Лучше: использование CSS-классов
    function bestExample() {
        // Определяем анимацию в CSS
        container.classList.add('moving-animation');
    }
}
```

### 2. Оптимизация событий

```javascript
// Плохо: много обработчиков событий
function badEventHandling() {
    const buttons = document.querySelectorAll('.button');
    
    // Плохо: создаем отдельный обработчик для каждого элемента
    buttons.forEach(button => {
        button.addEventListener('click', function() {
            console.log('Кнопка нажата:', this.textContent);
        });
    });
}

// Хорошо: делегирование событий
function goodEventHandling() {
    const container = document.getElementById('buttons-container');
    
    // Один обработчик для всех кнопок
    container.addEventListener('click', function(event) {
        if (event.target.matches('.button')) {
            console.log('Кнопка нажата:', event.target.textContent);
        }
    });
}

// Оптимизация обработчиков событий
class EventOptimizedComponent {
    constructor(element) {
        this.element = element;
        this.boundHandlers = new Map();
        this.abortController = new AbortController();
        this.init();
    }
    
    init() {
        // Привязываем обработчики к экземпляру
        this.boundHandlers.set('click', this.handleClick.bind(this));
        this.boundHandlers.set('mouseover', this.handleMouseOver.bind(this));
        
        // Добавляем обработчики с контроллером отмены
        this.element.addEventListener('click', this.boundHandlers.get('click'), {
            signal: this.abortController.signal
        });
        
        this.element.addEventListener('mouseover', this.boundHandlers.get('mouseover'), {
            signal: this.abortController.signal
        });
    }
    
    handleClick(event) {
        console.log('Клик обработан');
    }
    
    handleMouseOver(event) {
        console.log('Наведение мыши');
    }
    
    destroy() {
        // Отмена всех обработчиков
        this.abortController.abort();
        
        // Очистка привязанных обработчиков
        this.boundHandlers.clear();
        this.element = null;
    }
}

// Использование requestAnimationFrame для анимаций
class SmoothAnimation {
    constructor(element) {
        this.element = element;
        this.isActive = false;
        this.startTime = null;
        this.duration = 1000; // 1 секунда
    }
    
    animate(timestamp) {
        if (!this.startTime) {
            this.startTime = timestamp;
        }
        
        const elapsed = timestamp - this.startTime;
        const progress = Math.min(elapsed / this.duration, 1);
        
        // Вычисляем новое положение
        const newPosition = progress * 300; // 300px
        this.element.style.transform = `translateX(${newPosition}px)`;
        
        if (progress < 1 && this.isActive) {
            requestAnimationFrame(this.animate.bind(this));
        } else {
            this.isActive = false;
        }
    }
    
    start() {
        this.isActive = true;
        this.startTime = null;
        requestAnimationFrame(this.animate.bind(this));
    }
    
    stop() {
        this.isActive = false;
    }
}
```

## Оптимизация памяти

### 1. Управление памятью

```javascript
// Избегаем утечек памяти
class MemoryOptimizedClass {
    constructor() {
        this.data = new Map();
        this.eventListeners = new Set();
        this.timers = new Set();
        this.weakRefs = new WeakMap();
    }
    
    // Правильная очистка
    destroy() {
        // Очистка данных
        this.data.clear();
        
        // Удаление обработчиков событий
        this.eventListeners.forEach(listener => {
            if (listener.element && listener.handler) {
                listener.element.removeEventListener(listener.type, listener.handler);
            }
        });
        this.eventListeners.clear();
        
        // Очистка таймеров
        this.timers.forEach(timerId => clearTimeout(timerId));
        this.timers.clear();
        
        // Слабые ссылки очищаются автоматически
        this.weakRefs = null;
        
        // Очистка ссылок
        this.data = null;
    }
    
    // Использование WeakMap для метаданных
    addMetadata(obj, metadata) {
        this.weakRefs.set(obj, metadata); // Автоматическая очистка при удалении obj
    }
    
    getMetadata(obj) {
        return this.weakRefs.get(obj);
    }
}

// Оптимизация массивов
function arrayOptimizations() {
    // Плохо: частое изменение размера массива
    function badArrayUsage() {
        const arr = [];
        for (let i = 0; i < 100000; i++) {
            arr.push(i); // Может вызывать перераспределение памяти
        }
        return arr;
    }
    
    // Хорошо: предварительное выделение памяти
    function goodArrayUsage() {
        const arr = new Array(100000); // Предварительное выделение
        for (let i = 0; i < 100000; i++) {
            arr[i] = i;
        }
        return arr;
    }
    
    // Лучше: использовать push с несколькими элементами
    function bestArrayUsage() {
        const arr = [];
        const batchSize = 1000;
        
        for (let i = 0; i < 100000; i += batchSize) {
            const batch = Array.from({ length: batchSize }, (_, j) => i + j);
            arr.push(...batch);
        }
        
        return arr;
    }
}

// Использование Object Pool для часто создаваемых объектов
class ObjectPool {
    constructor(createFn, resetFn) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.pool = [];
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
    
    get size() {
        return this.pool.length;
    }
}

// Пример использования пула объектов
const vectorPool = new ObjectPool(
    () => ({ x: 0, y: 0, z: 0 }),
    (obj) => { obj.x = 0; obj.y = 0; obj.z = 0; }
);

function useVectorPool() {
    const vector = vectorPool.acquire();
    vector.x = 10;
    vector.y = 20;
    vector.z = 30;
    
    // Используем вектор...
    console.log(vector);
    
    // Возвращаем в пул
    vectorPool.release(vector);
}
```

### 2. Оптимизация строк

```javascript
// Оптимизация строковых операций
function stringOptimizations() {
    // Плохо: конкатенация в цикле
    function badStringConcatenation(count) {
        let result = '';
        for (let i = 0; i < count; i++) {
            result += `Item ${i}, `; // Создает новые строки каждый раз
        }
        return result;
    }
    
    // Хорошо: использование массива
    function goodStringConcatenation(count) {
        const parts = [];
        for (let i = 0; i < count; i++) {
            parts.push(`Item ${i}, `);
        }
        return parts.join('');
    }
    
    // Лучше: использование template literals с join
    function bestStringConcatenation(count) {
        return Array.from({ length: count }, (_, i) => `Item ${i}, `).join('');
    }
    
    // Для больших объемов данных
    function efficientStringBuilding(count) {
        const buffer = [];
        const chunkSize = 1000;
        
        for (let i = 0; i < count; i += chunkSize) {
            const chunk = [];
            const end = Math.min(i + chunkSize, count);
            
            for (let j = i; j < end; j++) {
                chunk.push(`Item ${j}, `);
            }
            
            buffer.push(chunk.join(''));
        }
        
        return buffer.join('');
    }
}

// Использование String.intern() аналога
class StringInterner {
    constructor() {
        this.cache = new Map();
    }
    
    intern(str) {
        if (!this.cache.has(str)) {
            this.cache.set(str, str);
        }
        return this.cache.get(str);
    }
    
    get cacheSize() {
        return this.cache.size;
    }
}

// Оптимизация регулярных выражений
class RegexOptimizations {
    constructor() {
        // Компилируем регулярные выражения заранее
        this.emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        this.phoneRegex = /^\+?[\d\s\-\(\)]{10,15}$/;
        this.urlRegex = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/;
    }
    
    validateEmail(email) {
        return this.emailRegex.test(email); // Быстрая проверка
    }
    
    validatePhone(phone) {
        return this.phoneRegex.test(phone.replace(/\s/g, '')); // Быстрая проверка
    }
    
    validateURL(url) {
        return this.urlRegex.test(url); // Быстрая проверка
    }
}
```

## Профилирование и отладка производительности

### 1. Использование DevTools

```javascript
// Создание измеряемых точек в коде
class PerformanceProfiler {
    constructor() {
        this.measurements = new Map();
        this.markers = new Set();
    }
    
    start(name) {
        performance.mark(`${name}-start`);
        this.markers.add(`${name}-start`);
    }
    
    end(name) {
        performance.mark(`${name}-end`);
        performance.measure(name, `${name}-start`, `${name}-end`);
        this.markers.add(`${name}-end`);
        
        const measure = performance.getEntriesByName(name)[0];
        console.log(`${name}: ${measure.duration.toFixed(2)}ms`);
        
        return measure.duration;
    }
    
    report() {
        const measures = performance.getEntriesByType('measure');
        measures.forEach(measure => {
            console.log(`${measure.name}: ${measure.duration.toFixed(2)}ms`);
        });
    }
    
    clear() {
        performance.clearMarks();
        performance.clearMeasures();
        this.markers.clear();
    }
}

// Использование профилировщика
function profileCode() {
    const profiler = new PerformanceProfiler();
    
    profiler.start('complex-operation');
    
    // Выполняем сложную операцию
    const result = complexOperation();
    
    profiler.end('complex-operation');
    profiler.report();
    
    return result;
}

// Функция для измерения
function complexOperation() {
    let sum = 0;
    for (let i = 0; i < 1000000; i++) {
        sum += Math.sqrt(i) * Math.sin(i);
    }
    return sum;
}

// Сбор статистики по производительности
class PerformanceStats {
    constructor() {
        this.stats = {
            functionCalls: new Map(),
            memoryUsage: [],
            executionTimes: new Map()
        };
    }
    
    trackFunction(fn, name) {
        const self = this;
        return function(...args) {
            const start = performance.now();
            const startMemory = performance.memory ? performance.memory.usedJSHeapSize : 0;
            
            const result = fn.apply(this, args);
            
            const end = performance.now();
            const endMemory = performance.memory ? performance.memory.usedJSHeapSize : 0;
            
            // Обновляем статистику
            if (!self.stats.functionCalls.has(name)) {
                self.stats.functionCalls.set(name, 0);
                self.stats.executionTimes.set(name, []);
            }
            
            self.stats.functionCalls.set(name, self.stats.functionCalls.get(name) + 1);
            self.stats.executionTimes.get(name).push(end - start);
            
            // Логируем, если функция выполняется слишком долго
            if (end - start > 100) { // больше 100ms
                console.warn(`Медленная функция ${name}: ${(end - start).toFixed(2)}ms`);
            }
            
            return result;
        };
    }
    
    getStats() {
        const result = {};
        for (let [name, times] of this.stats.executionTimes) {
            const avg = times.reduce((a, b) => a + b, 0) / times.length;
            const max = Math.max(...times);
            const min = Math.min(...times);
            
            result[name] = {
                calls: this.stats.functionCalls.get(name),
                avg: avg.toFixed(2),
                min: min.toFixed(2),
                max: max.toFixed(2)
            };
        }
        return result;
    }
}
```

### 2. Автоматическое профилирование

```javascript
// Декоратор для автоматического профилирования
function profile(target, propertyName, descriptor) {
    const method = descriptor.value;
    
    descriptor.value = function(...args) {
        const start = performance.now();
        const result = method.apply(this, args);
        const end = performance.now();
        
        console.log(`${propertyName} выполнена за ${(end - start).toFixed(2)}ms`);
        
        return result;
    };
    
    return descriptor;
}

// Использование декоратора (экспериментально)
class ProfiledClass {
    @profile
    slowMethod() {
        // Медленный код
        let sum = 0;
        for (let i = 0; i < 1000000; i++) {
            sum += i;
        }
        return sum;
    }
    
    @profile
    anotherMethod(data) {
        return data.map(x => x * 2);
    }
}

// Функция для профилирования асинхронных операций
async function profileAsyncOperation(operation, name) {
    const start = performance.now();
    
    try {
        const result = await operation();
        const end = performance.now();
        
        console.log(`${name} выполнена за ${(end - start).toFixed(2)}ms`);
        return result;
    } catch (error) {
        const end = performance.now();
        console.error(`${name} завершена с ошибкой за ${(end - start).toFixed(2)}ms:`, error);
        throw error;
    }
}

// Пример использования
async function example() {
    await profileAsyncOperation(async () => {
        // Асинхронная операция
        await new Promise(resolve => setTimeout(resolve, 1000));
        return 'result';
    }, 'delayed-operation');
}
```

## Современные техники оптимизации

### 1. Web Workers

```javascript
// Основной поток
class WorkerManager {
    constructor() {
        this.workers = [];
        this.workerCount = navigator.hardwareConcurrency || 4;
    }
    
    createWorker() {
        const workerCode = `
            self.onmessage = function(e) {
                const { data, operation } = e.data;
                
                let result;
                switch(operation) {
                    case 'sort':
                        result = data.sort((a, b) => a - b);
                        break;
                    case 'calculate':
                        result = data.reduce((sum, num) => sum + Math.sqrt(num), 0);
                        break;
                    case 'filter':
                        result = data.filter(item => item > 100);
                        break;
                    default:
                        result = data;
                }
                
                self.postMessage(result);
            };
        `;
        
        const blob = new Blob([workerCode], { type: 'application/javascript' });
        return new Worker(URL.createObjectURL(blob));
    }
    
    async processInWorker(data, operation) {
        return new Promise((resolve, reject) => {
            const worker = this.createWorker();
            
            worker.onmessage = function(e) {
                resolve(e.data);
                worker.terminate();
            };
            
            worker.onerror = function(error) {
                reject(error);
                worker.terminate();
            };
            
            worker.postMessage({ data, operation });
        });
    }
    
    async parallelProcess(data, operation) {
        const chunkSize = Math.ceil(data.length / this.workerCount);
        const chunks = [];
        
        for (let i = 0; i < data.length; i += chunkSize) {
            chunks.push(data.slice(i, i + chunkSize));
        }
        
        const promises = chunks.map(chunk => 
            this.processInWorker(chunk, operation)
        );
        
        const results = await Promise.all(promises);
        return results.flat();
    }
}

// Использование
async function workerExample() {
    const manager = new WorkerManager();
    const largeArray = Array.from({ length: 1000000 }, () => Math.floor(Math.random() * 1000));
    
    console.time('Worker sort');
    const sorted = await manager.parallelProcess(largeArray, 'sort');
    console.timeEnd('Worker sort');
}
```

### 2. WebAssembly

```javascript
// Пример интеграции WebAssembly для вычислений
class WasmCalculator {
    constructor(wasmModule) {
        this.wasmInstance = null;
        this.module = wasmModule;
        this.initialize();
    }
    
    async initialize() {
        // Загрузка и компиляция WebAssembly модуля
        this.wasmInstance = await WebAssembly.instantiateStreaming(
            fetch('calculator.wasm')
        );
    }
    
    fibonacci(n) {
        if (this.wasmInstance) {
            return this.wasmInstance.instance.exports.fibonacci(n);
        }
        // Резервная реализация на JavaScript
        return this.jsFibonacci(n);
    }
    
    matrixMultiply(matrixA, matrixB) {
        if (this.wasmInstance) {
            return this.wasmInstance.instance.exports.matrixMultiply(matrixA, matrixB);
        }
        // Резервная реализация на JavaScript
        return this.jsMatrixMultiply(matrixA, matrixB);
    }
    
    jsFibonacci(n) {
        if (n <= 1) return n;
        return this.jsFibonacci(n - 1) + this.jsFibonacci(n - 2);
    }
    
    jsMatrixMultiply(a, b) {
        const result = new Array(a.length);
        for (let i = 0; i < a.length; i++) {
            result[i] = new Array(b[0].length).fill(0);
            for (let j = 0; j < b[0].length; j++) {
                for (let k = 0; k < b.length; k++) {
                    result[i][j] += a[i][k] * b[k][j];
                }
            }
        }
        return result;
    }
}
```

## Лучшие практики производительности

### 1. Структура и организация кода

```javascript
// Использование модулей для оптимизации
// utils/performance.js
export class PerformanceOptimizer {
    constructor() {
        this.memoCache = new Map();
        this.throttleCache = new Map();
    }
    
    // Мемоизация
    memoize(fn, keyFn = (...args) => JSON.stringify(args)) {
        return (...args) => {
            const key = keyFn(...args);
            if (this.memoCache.has(key)) {
                return this.memoCache.get(key);
            }
            
            const result = fn(...args);
            this.memoCache.set(key, result);
            return result;
        };
    }
    
    // Троттлинг
    throttle(func, limit) {
        let inThrottle;
        return function(...args) {
            const context = this;
            if (!inThrottle) {
                func.apply(context, args);
                inThrottle = true;
                setTimeout(() => inThrottle = false, limit);
            }
        };
    }
    
    // Дебаунсинг
    debounce(func, delay) {
        let timeoutId;
        return function(...args) {
            const context = this;
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => func.apply(context, args), delay);
        };
    }
}

// Использование
const optimizer = new PerformanceOptimizer();

const expensiveCalculation = (x, y) => {
    // Медленное вычисление
    return Math.pow(x, y) + Math.sqrt(x * y);
};

const memoizedCalc = optimizer.memoize(expensiveCalculation);
console.log(memoizedCalc(10, 5)); // Вычисляет
console.log(memoizedCalc(10, 5)); // Возвращает из кеша
```

### 2. Асинхронные оптимизации

```javascript
// Оптимизация асинхронных операций
class AsyncOptimizer {
    constructor() {
        this.cache = new Map();
        this.pendingRequests = new Map();
    }
    
    // Кеширование асинхронных операций
    async cachedFetch(url, options = {}) {
        const cacheKey = `${url}_${JSON.stringify(options)}`;
        
        // Проверяем кеш
        if (this.cache.has(cacheKey)) {
            return this.cache.get(cacheKey);
        }
        
        // Проверяем, есть ли уже ожидающий запрос
        if (this.pendingRequests.has(cacheKey)) {
            return this.pendingRequests.get(cacheKey);
        }
        
        // Создаем новый запрос
        const promise = fetch(url, options)
            .then(response => response.json())
            .then(data => {
                // Сохраняем в кеш
                this.cache.set(cacheKey, data);
                this.pendingRequests.delete(cacheKey);
                return data;
            })
            .catch(error => {
                this.pendingRequests.delete(cacheKey);
                throw error;
            });
        
        this.pendingRequests.set(cacheKey, promise);
        return promise;
    }
    
    // Параллельные запросы с ограничением
    async limitedParallel(promises, limit = 5) {
        const results = [];
        const executing = [];
        
        for (const [index, promise] of promises.entries()) {
            const queuePromise = Promise.resolve().then(() => promise());
            
            results[index] = queuePromise;
            
            if (promises.length >= limit) {
                const e = queuePromise.then(() => {
                    executing.splice(executing.indexOf(e), 1);
                });
                executing.push(e);
                
                if (executing.length >= limit) {
                    await Promise.race(executing);
                }
            }
        }
        
        return Promise.all(results);
    }
    
    // Пакетная обработка
    batchProcess(items, batchSize = 100) {
        const batches = [];
        for (let i = 0; i < items.length; i += batchSize) {
            batches.push(items.slice(i, i + batchSize));
        }
        
        return batches.map(batch => async () => {
            // Обработка пачки
            return Promise.all(batch.map(item => this.processItem(item)));
        });
    }
    
    async processItem(item) {
        // Обработка отдельного элемента
        return item;
    }
}

// Использование
async function asyncOptimizationExample() {
    const optimizer = new AsyncOptimizer();
    
    // Кешированные запросы
    const data1 = await optimizer.cachedFetch('/api/data');
    const data2 = await optimizer.cachedFetch('/api/data'); // Из кеша
    
    // Пакетная обработка
    const items = Array.from({ length: 1000 }, (_, i) => ({ id: i, value: `item${i}` }));
    const batches = optimizer.batchProcess(items, 50);
    
    const results = await Promise.all(batches.map(batch => batch()));
    console.log('Обработано:', results.flat().length);
}
```

## Примеры из промышленной разработки

### 1. Оптимизация рендеринга

```javascript
// Оптимизированный виртуальный DOM
class VirtualDOM {
    constructor() {
        this.componentRegistry = new Map();
        this.componentInstances = new Map();
    }
    
    createElement(type, props, ...children) {
        return {
            type,
            props: props || {},
            children: children.flat(),
            key: props?.key || null
        };
    }
    
    render(vnode, container) {
        const element = this.createDOMElement(vnode);
        container.appendChild(element);
        return element;
    }
    
    createDOMElement(vnode) {
        if (typeof vnode.type === 'function') {
            // Компонент
            return this.createComponent(vnode);
        }
        
        if (typeof vnode === 'string' || typeof vnode === 'number') {
            // Текстовый узел
            return document.createTextNode(vnode);
        }
        
        // DOM элемент
        const element = document.createElement(vnode.type);
        
        // Установка атрибутов
        this.setProps(element, vnode.props);
        
        // Добавление детей
        vnode.children.forEach(child => {
            if (child) {
                element.appendChild(this.createDOMElement(child));
            }
        });
        
        return element;
    }
    
    setProps(element, props) {
        Object.keys(props).forEach(propName => {
            const value = props[propName];
            
            if (propName === 'children') return;
            
            if (propName.startsWith('on')) {
                // Обработчик событий
                const eventType = propName.toLowerCase().substring(2);
                element.addEventListener(eventType, value);
            } else if (propName === 'style') {
                // Стили
                Object.keys(value).forEach(styleProp => {
                    element.style[styleProp] = value[styleProp];
                });
            } else if (propName === 'className') {
                // Класс
                element.className = value;
            } else {
                // Обычные атрибуты
                element.setAttribute(propName, value);
            }
        });
    }
    
    diff(oldVNode, newVNode, parentElement) {
        // Простая реализация diff алгоритма
        if (!oldVNode && newVNode) {
            parentElement.appendChild(this.createDOMElement(newVNode));
        } else if (oldVNode && !newVNode) {
            parentElement.removeChild(parentElement.firstChild);
        } else if (oldVNode.type !== newVNode.type) {
            parentElement.replaceChild(
                this.createDOMElement(newVNode),
                parentElement.firstChild
            );
        }
        // Здесь могла бы быть более сложная логика diff
    }
}

// Использование
const vdom = new VirtualDOM();

function App() {
    return vdom.createElement('div', { className: 'app' },
        vdom.createElement('h1', null, 'Привет, мир!'),
        vdom.createElement('p', null, 'Это оптимизированный рендеринг')
    );
}

// Рендеринг
const container = document.getElementById('app');
vdom.render(App(), container);
```

### 2. Оптимизация поиска

```javascript
// Оптимизированный поисковый индекс
class SearchIndex {
    constructor() {
        this.index = new Map(); // термин -> [документы]
        this.documents = new Map(); // id -> документ
        this.stemmer = new PorterStemmer(); // для стемминга
    }
    
    addDocument(id, content) {
        this.documents.set(id, content);
        
        // Токенизация и индексация
        const tokens = this.tokenize(content);
        const uniqueTokens = new Set(tokens);
        
        for (const token of uniqueTokens) {
            if (!this.index.has(token)) {
                this.index.set(token, new Set());
            }
            this.index.get(token).add(id);
        }
    }
    
    tokenize(text) {
        // Простая токенизация
        return text
            .toLowerCase()
            .replace(/[^\w\s]/g, ' ')
            .split(/\s+/)
            .filter(token => token.length > 2)
            .map(token => this.stemmer.stem(token));
    }
    
    search(query) {
        const queryTokens = this.tokenize(query);
        const resultSets = queryTokens.map(token => this.index.get(token) || new Set());
        
        // Пересечение всех результатов
        let results = resultSets.length > 0 ? resultSets[0] : new Set();
        for (let i = 1; i < resultSets.length; i++) {
            results = new Set([...results].filter(x => resultSets[i].has(x)));
        }
        
        // Ранжирование по релевантности
        const scoredResults = [...results].map(docId => {
            const docContent = this.documents.get(docId);
            const relevance = this.calculateRelevance(queryTokens, docContent);
            return { id: docId, relevance, content: docContent };
        });
        
        return scoredResults
            .sort((a, b) => b.relevance - a.relevance)
            .slice(0, 10); // Топ 10 результатов
    }
    
    calculateRelevance(queryTokens, docContent) {
        let score = 0;
        const docTokens = new Set(this.tokenize(docContent));
        
        for (const token of queryTokens) {
            if (docTokens.has(token)) {
                score += 1;
            }
        }
        
        return score / queryTokens.length;
    }
}

// Простой стеммер Портера (упрощенная версия)
class PorterStemmer {
    stem(word) {
        // Упрощенная реализация стемминга
        if (word.endsWith('ing')) return word.slice(0, -3);
        if (word.endsWith('ed')) return word.slice(0, -2);
        if (word.endsWith('ly')) return word.slice(0, -2);
        return word;
    }
}

// Использование
const searchIndex = new SearchIndex();

// Добавление документов
searchIndex.addDocument(1, 'JavaScript performance optimization techniques');
searchIndex.addDocument(2, 'Advanced JavaScript patterns and practices');
searchIndex.addDocument(3, 'Web performance and optimization guide');

// Поиск
const results = searchIndex.search('javascript optimization');
console.log('Результаты поиска:', results);
```

## Ссылки на связанные темы
- [[js/best-practices]] - Лучшие практики JavaScript
- [[js/es6+/performance]] - Современные оптимизации ES6+
- [[browser-performance]] - Производительность браузера
- [[web-workers]] - Web Workers

#programming #javascript #performance #optimization #profiling #best-practices