# Профилирование производительности в JavaScript

## Обзор

Профилирование производительности - это процесс измерения и анализа производительности приложения для выявления узких мест и оптимизации. В JavaScript существуют различные инструменты и методы для профилирования производительности как в браузере, так и в Node.js.

## Инструменты профилирования

### Встроенные инструменты браузера

#### Performance API

```javascript
// Измерение времени выполнения функции
function measureFunctionPerformance(fn, name, iterations = 1) {
    const start = performance.now();
    
    for (let i = 0; i < iterations; i++) {
        fn();
    }
    
    const end = performance.now();
    const totalTime = end - start;
    const averageTime = totalTime / iterations;
    
    console.log(`${name}: Всего: ${totalTime.toFixed(2)}ms, Среднее: ${averageTime.toFixed(4)}ms (${iterations} итераций)`);
    
    return { totalTime, averageTime, iterations };
}

// Пример функции для тестирования
function expensiveOperation() {
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
        result += Math.sqrt(i) * Math.sin(i);
    }
    return result;
}

// Измерение
measureFunctionPerformance(expensiveOperation, 'expensiveOperation', 10);
```

#### Performance.mark и Performance.measure

```javascript
// Использование меток производительности
function profileWithMarks() {
    performance.mark('start-operation');
    
    // Выполняем операцию
    const result = expensiveOperation();
    
    performance.mark('end-operation');
    
    // Создаем измерение между метками
    performance.measure('operation-duration', 'start-operation', 'end-operation');
    
    // Получаем результаты
    const measures = performance.getEntriesByName('operation-duration');
    const measure = measures[0];
    
    console.log(`Операция заняла: ${measure.duration.toFixed(2)}ms`);
    
    // Очищаем метки
    performance.clearMarks();
    performance.clearMeasures();
    
    return result;
}

profileWithMarks();
```

### Консольные инструменты профилирования

```javascript
// Использование console.time/console.timeEnd
function profileWithConsoleTime() {
    console.time('expensive-operation');
    
    const result = expensiveOperation();
    
    console.timeEnd('expensive-operation'); // Выводит время в консоль
    
    return result;
}

profileWithConsoleTime();

// Использование console.profile
function profileWithConsoleProfile() {
    console.profile('My Profile');
    
    // Код для профилирования
    for (let i = 0; i < 1000; i++) {
        expensiveOperation();
    }
    
    console.profileEnd('My Profile');
}

profileWithConsoleProfile();
```

## Профилирование памяти

### Отслеживание использования памяти

```javascript
// Простой способ отслеживания памяти (если доступно)
function getMemoryUsage() {
    if (performance.memory) {
        return {
            used: Math.round(performance.memory.usedJSHeapSize / 1048576 * 100) / 100, // в МБ
            total: Math.round(performance.memory.totalJSHeapSize / 1048576 * 100) / 100, // в МБ
            limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576 * 100) / 100 // в МБ
        };
    }
    return null;
}

function monitorMemoryUsage() {
    const initialMemory = getMemoryUsage();
    console.log('Начальное использование памяти:', initialMemory);
    
    // Выполняем операции
    const largeArray = new Array(1000000).fill(0).map((_, i) => i * 2);
    
    const afterAllocation = getMemoryUsage();
    console.log('После выделения памяти:', afterAllocation);
    
    // Удаляем ссылки
    largeArray.length = 0;
    
    // Принудительно вызываем сборщик мусора (только в девелоперских инструментах с включенной опцией)
    if (window.gc) {
        window.gc();
    }
    
    const afterCleanup = getMemoryUsage();
    console.log('После очистки:', afterCleanup);
    
    return { initial: initialMemory, after: afterCleanup };
}

monitorMemoryUsage();
```

### Выявление утечек памяти

```javascript
// Пример кода, который может вызвать утечку памяти
class MemoryLeakExample {
    constructor() {
        this.data = new Array(1000000).fill('some data');
        this.eventHandlers = [];
        
        // Добавляем обработчики событий
        this.addHandler();
    }
    
    addHandler() {
        const handler = () => {
            // Этот обработчик держит ссылку на весь объект через замыкание
            console.log('Handler called', this.data.length);
        };
        
        document.addEventListener('click', handler);
        this.eventHandlers.push(handler);
    }
    
    // Метод для правильной очистки
    destroy() {
        // Удаляем обработчики событий
        this.eventHandlers.forEach(handler => {
            document.removeEventListener('click', handler);
        });
        
        // Очищаем массив
        this.data.length = 0;
        this.eventHandlers.length = 0;
    }
}

// Профилирование утечек памяти
function profileMemoryLeaks() {
    const objects = [];
    
    console.log('Начальное состояние памяти:', getMemoryUsage());
    
    // Создаем несколько объектов
    for (let i = 0; i < 100; i++) {
        objects.push(new MemoryLeakExample());
    }
    
    console.log('После создания 100 объектов:', getMemoryUsage());
    
    // Удаляем объекты без правильной очистки
    objects.length = 0;
    
    console.log('После удаления объектов (без очистки):', getMemoryUsage());
    
    // Вызываем сборку мусора
    if (window.gc) {
        window.gc();
    }
    
    console.log('После сборки мусора:', getMemoryUsage());
}

profileMemoryLeaks();
```

## Профилирование в Node.js

### Использование встроенных инструментов Node.js

```javascript
// performance-node.js
import { performance } from 'perf_hooks';
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

if (isMainThread) {
    // Основной поток
    async function profileNodeOperations() {
        // Профилирование синхронной операции
        performance.mark('start-sync');
        
        let result = 0;
        for (let i = 0; i < 10000000; i++) {
            result += Math.sqrt(i);
        }
        
        performance.mark('end-sync');
        performance.measure('sync-operation', 'start-sync', 'end-sync');
        
        const syncMeasure = performance.getEntriesByName('sync-operation')[0];
        console.log(`Синхронная операция: ${syncMeasure.duration.toFixed(2)}ms`);
        
        // Профилирование асинхронной операции
        performance.mark('start-async');
        
        const asyncResult = await new Promise(resolve => {
            setTimeout(() => {
                let sum = 0;
                for (let i = 0; i < 5000000; i++) {
                    sum += Math.random();
                }
                resolve(sum);
            }, 0);
        });
        
        performance.mark('end-async');
        performance.measure('async-operation', 'start-async', 'end-async');
        
        const asyncMeasure = performance.getEntriesByName('async-operation')[0];
        console.log(`Асинхронная операция: ${asyncMeasure.duration.toFixed(2)}ms`);
        
        // Профилирование с использованием Worker
        await profileWithWorker();
    }
    
    async function profileWithWorker() {
        return new Promise((resolve, reject) => {
            performance.mark('start-worker');
            
            const worker = new Worker(__filename, { workerData: { task: 'heavy-computation' } });
            
            worker.on('message', (result) => {
                performance.mark('end-worker');
                performance.measure('worker-operation', 'start-worker', 'end-worker');
                
                const workerMeasure = performance.getEntriesByName('worker-operation')[0];
                console.log(`Операция в Worker: ${workerMeasure.duration.toFixed(2)}ms`);
                console.log('Результат из Worker:', result);
                
                resolve(result);
            });
            
            worker.on('error', reject);
        });
    }
    
    profileNodeOperations();
    
} else {
    // Worker поток
    if (workerData.task === 'heavy-computation') {
        let result = 0;
        for (let i = 0; i < 10000000; i++) {
            result += Math.sin(i) * Math.cos(i);
        }
        parentPort.postMessage(result);
    }
}
```

### Использование Node.js профилировщика

```javascript
// Для запуска с профилировкой используйте:
// node --prof app.js
// node --prof-process isolate-*.log > processed.txt

// Пример функции для профилирования
function complexCalculation(n) {
    const result = [];
    
    for (let i = 0; i < n; i++) {
        const item = {
            id: i,
            data: new Array(i % 100).fill(i),
            calculated: Math.pow(i, 2) + Math.sqrt(i)
        };
        
        // Выполняем различные операции
        if (i % 2 === 0) {
            item.processed = item.data.reduce((sum, val) => sum + val, 0);
        } else {
            item.processed = item.data.map(x => x * 2).reduce((sum, val) => sum + val, 0);
        }
        
        result.push(item);
    }
    
    return result;
}

// Запуск функции для профилирования
const start = Date.now();
const calculationResult = complexCalculation(10000);
const end = Date.now();

console.log(`Расчет завершен за ${end - start}ms`);
console.log(`Получено ${calculationResult.length} элементов`);
```

## Профилирование DOM-операций

### Измерение производительности DOM-манипуляций

```javascript
// dom-performance.js
function profileDOMOperations() {
    const container = document.getElementById('container') || createTestContainer();
    
    // Профилирование создания элементов
    performance.mark('start-create-elements');
    
    const fragment = document.createDocumentFragment();
    for (let i = 0; i < 1000; i++) {
        const div = document.createElement('div');
        div.textContent = `Элемент ${i}`;
        div.className = `item item-${i}`;
        fragment.appendChild(div);
    }
    
    performance.mark('end-create-elements');
    performance.measure('create-elements', 'start-create-elements', 'end-create-elements');
    
    // Добавляем элементы в DOM
    performance.mark('start-append');
    container.appendChild(fragment);
    performance.mark('end-append');
    performance.measure('append-elements', 'start-append', 'end-append');
    
    // Профилирование изменения стилей
    performance.mark('start-style-changes');
    
    const items = container.querySelectorAll('.item');
    items.forEach((item, index) => {
        item.style.backgroundColor = index % 2 === 0 ? 'lightblue' : 'lightgreen';
        item.style.padding = '5px';
        item.style.margin = '2px';
    });
    
    performance.mark('end-style-changes');
    performance.measure('style-changes', 'start-style-changes', 'end-style-changes');
    
    // Вывод результатов
    const measures = performance.getEntriesByType('measure');
    measures.forEach(measure => {
        console.log(`${measure.name}: ${measure.duration.toFixed(2)}ms`);
    });
    
    return container;
}

function createTestContainer() {
    const container = document.createElement('div');
    container.id = 'container';
    container.style.width = '100%';
    document.body.appendChild(container);
    return container;
}

// Вызов функции профилирования
profileDOMOperations();
```

## Продвинутые техники профилирования

### Создание собственного профилировщика

```javascript
// custom-profiler.js
class CustomProfiler {
    constructor() {
        this.measurements = new Map();
        this.activeMarks = new Map();
    }
    
    start(name) {
        const startTime = performance.now();
        this.activeMarks.set(name, startTime);
        
        // Также создаем метку в Performance API
        performance.mark(`start-${name}`);
    }
    
    end(name) {
        const startTime = this.activeMarks.get(name);
        if (startTime === undefined) {
            console.warn(`Метка "${name}" не найдена`);
            return null;
        }
        
        const endTime = performance.now();
        const duration = endTime - startTime;
        
        // Создаем метку окончания и измерение
        performance.mark(`end-${name}`);
        performance.measure(name, `start-${name}`, `end-${name}`);
        
        // Сохраняем измерение
        if (!this.measurements.has(name)) {
            this.measurements.set(name, []);
        }
        this.measurements.get(name).push(duration);
        
        // Удаляем активную метку
        this.activeMarks.delete(name);
        
        return duration;
    }
    
    getStats(name) {
        const measurements = this.measurements.get(name);
        if (!measurements || measurements.length === 0) {
            return null;
        }
        
        const total = measurements.reduce((sum, val) => sum + val, 0);
        const average = total / measurements.length;
        const min = Math.min(...measurements);
        const max = Math.max(...measurements);
        
        return {
            count: measurements.length,
            total: total,
            average: average,
            min: min,
            max: max
        };
    }
    
    report() {
        console.log('\n=== Отчет профилировщика ===');
        for (const [name, stats] of this.measurements) {
            const stat = this.getStats(name);
            console.log(`${name}:`);
            console.log(`  Вызовов: ${stat.count}`);
            console.log(`  Всего: ${stat.total.toFixed(2)}ms`);
            console.log(`  Среднее: ${stat.average.toFixed(4)}ms`);
            console.log(`  Мин: ${stat.min.toFixed(4)}ms`);
            console.log(`  Макс: ${stat.max.toFixed(4)}ms`);
        }
        console.log('========================\n');
    }
    
    clear() {
        this.measurements.clear();
        this.activeMarks.clear();
        performance.clearMarks();
        performance.clearMeasures();
    }
}

// Использование кастомного профилировщика
const profiler = new CustomProfiler();

function exampleFunction() {
    profiler.start('example-function');
    
    // Имитация работы
    let sum = 0;
    for (let i = 0; i < 100000; i++) {
        sum += Math.sqrt(i);
    }
    
    profiler.end('example-function');
    return sum;
}

// Выполняем функцию несколько раз
for (let i = 0; i < 5; i++) {
    exampleFunction();
}

profiler.report();
```

### Профилирование асинхронных операций

```javascript
// async-profiler.js
class AsyncProfiler extends CustomProfiler {
    async measureAsync(name, asyncFn) {
        this.start(name);
        
        try {
            const result = await asyncFn();
            this.end(name);
            return result;
        } catch (error) {
            this.end(name);
            throw error;
        }
    }
    
    async measureParallel(name, ...asyncFns) {
        this.start(name);
        
        try {
            const results = await Promise.all(asyncFns.map(fn => fn()));
            this.end(name);
            return results;
        } catch (error) {
            this.end(name);
            throw error;
        }
    }
}

// Пример использования
async function simulateAsyncOperation(ms) {
    return new Promise(resolve => {
        setTimeout(() => resolve(`Результат через ${ms}мс`), ms);
    });
}

async function profileAsyncOperations() {
    const asyncProfiler = new AsyncProfiler();
    
    // Измеряем одиночную асинхронную операцию
    const result1 = await asyncProfiler.measureAsync(
        'single-async-operation',
        () => simulateAsyncOperation(100)
    );
    console.log('Результат:', result1);
    
    // Измеряем параллельные операции
    const results = await asyncProfiler.measureParallel(
        'parallel-operations',
        () => simulateAsyncOperation(50),
        () => simulateAsyncOperation(75),
        () => simulateAsyncOperation(100)
    );
    
    console.log('Параллельные результаты:', results);
    
    // Отчет
    asyncProfiler.report();
}

// Запуск асинхронного профилирования
profileAsyncOperations();
```

## Практические советы по профилированию

### Паттерн "Шумных соседей" (Noisy Neighbor)

```javascript
// bad-neighbors.js
class BadNeighbors {
    constructor() {
        // Эти свойства будут потреблять много памяти
        this.hugeArray = new Array(1000000).fill(0).map((_, i) => ({
            id: i,
            data: new Array(100).fill(`data_${i}`),
            timestamp: Date.now() + i
        }));
        
        // Эта функция будет часто вызываться и потреблять CPU
        this.intervalId = setInterval(() => {
            this.processData();
        }, 10);
    }
    
    processData() {
        // Выполняем тяжелые вычисления
        let result = 0;
        for (let i = 0; i < 100000; i++) {
            result += Math.sin(i) * Math.cos(i);
        }
        return result;
    }
    
    destroy() {
        clearInterval(this.intervalId);
        this.hugeArray.length = 0;
    }
}

// Профилирование "шумных соседей"
function profileBadNeighbors() {
    console.log('Начало профилирования "шумных соседей"');
    console.log('Память до:', getMemoryUsage());
    
    const badNeighbors = new BadNeighbors();
    
    // Ждем немного, чтобы дать "шумным соседям" поработать
    setTimeout(() => {
        console.log('Память во время работы:', getMemoryUsage());
        
        // Удаляем "шумных соседей"
        badNeighbors.destroy();
        
        if (window.gc) {
            window.gc();
        }
        
        setTimeout(() => {
            console.log('Память после очистки:', getMemoryUsage());
        }, 1000);
    }, 5000);
}

// profileBadNeighbors(); // Раскомментируйте для запуска
```

## Заключение

Профилирование производительности - важный этап оптимизации JavaScript-приложений. Правильное использование инструментов профилирования позволяет:

1. Выявлять узкие места в производительности
2. Обнаруживать утечки памяти
3. Оптимизировать алгоритмы и структуры данных
4. Повышать общую отзывчивость приложения
5. Улучшать пользовательский опыт