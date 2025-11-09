# Web Workers

Web Workers позволяют запускать JavaScript в фоновом потоке, не блокируя основной поток выполнения. Это позволяет выполнять тяжелые вычисления, обработку данных и другие ресурсоемкие операции без влияния на пользовательский интерфейс.

## Основы Web Workers

### Создание и запуск Web Worker

```javascript
// Основной поток (main.js)
// Проверка поддержки Web Workers
if (window.Worker) {
    // Создание нового воркера
    const worker = new Worker('worker.js');
    
    // Отправка сообщения воркеру
    worker.postMessage({
        command: 'calculate',
        data: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    });
    
    // Получение сообщения от воркера
    worker.onmessage = function(event) {
        console.log('Результат от воркера:', event.data);
    };
    
    // Обработка ошибок
    worker.onerror = function(error) {
        console.error('Ошибка воркера:', error);
    };
} else {
    console.log('Web Workers не поддерживаются в этом браузере');
}
```

```javascript
// Воркер (worker.js)
// Обработка сообщений от основного потока
self.onmessage = function(event) {
    const { command, data } = event.data;
    
    switch (command) {
        case 'calculate':
            const result = performCalculation(data);
            // Отправка результата обратно в основной поток
            self.postMessage(result);
            break;
            
        case 'sort':
            const sorted = performSort(data);
            self.postMessage(sorted);
            break;
            
        default:
            self.postMessage({ error: 'Неизвестная команда' });
    }
};

// Функция для выполнения тяжелых вычислений
function performCalculation(numbers) {
    // Имитация тяжелого вычисления
    let sum = 0;
    for (let i = 0; i < numbers.length; i++) {
        sum += numbers[i] * numbers[i]; // квадраты чисел
    }
    return sum;
}

function performSort(data) {
    // Сортировка с задержкой для демонстрации
    return data.sort((a, b) => a - b);
}
```

### Типы Web Workers

```javascript
// 1. Dedicated Worker (обычный воркер)
const dedicatedWorker = new Worker('dedicated-worker.js');

// 2. Inline Worker (воркер в виде строки)
const inlineWorkerCode = `
    self.onmessage = function(e) {
        self.postMessage('Hello from inline worker: ' + e.data);
    };
`;

const blob = new Blob([inlineWorkerCode], { type: 'application/javascript' });
const inlineWorker = new Worker(URL.createObjectURL(blob));

// 3. Module Worker (воркер с модулями)
const moduleWorker = new Worker('module-worker.js', { type: 'module' });
```

## Передача данных

### Structured Clone Algorithm

```javascript
// Пример передачи сложных данных
const worker = new Worker('data-worker.js');

// Можно передавать:
const complexData = {
    string: 'Hello',
    number: 42,
    boolean: true,
    array: [1, 2, 3],
    object: { nested: 'value' },
    date: new Date(),
    regexp: /pattern/gi,
    null: null,
    undefined: undefined
};

worker.postMessage(complexData);

// НЕЛЬЗЯ передавать:
// - DOM-элементы
// - Функции
// - Error объекты
// - Window объекты
// - ArrayBuffer (но можно передать его содержимое)
```

### Transferable Objects

```javascript
// Использование Transferable Objects для более эффективной передачи данных
function useTransferableObjects() {
    const worker = new Worker('transfer-worker.js');
    
    // Создание большого ArrayBuffer
    const buffer = new ArrayBuffer(1024 * 1024); // 1MB
    const uint8Array = new Uint8Array(buffer);
    
    // Заполнение данными
    for (let i = 0; i < uint8Array.length; i++) {
        uint8Array[i] = i % 256;
    }
    
    // Передача буфера с передачей владения (transfer)
    // После передачи, буфер в основном потоке становится неиспользуемым
    worker.postMessage(uint8Array, [uint8Array.buffer]);
    
    // uint8Array.buffer теперь равен null в основном потоке
    console.log('Буфер передан, исходный буфер:', uint8Array.buffer); // null
}

// В воркере (transfer-worker.js):
/*
self.onmessage = function(event) {
    const uint8Array = event.data;
    
    // Обработка данных
    for (let i = 0; i < uint8Array.length; i++) {
        uint8Array[i] = uint8Array[i] * 2;
    }
    
    // Возврат результата с передачей владения
    self.postMessage(uint8Array, [uint8Array.buffer]);
};
*/
```

## Практические примеры

### 1. Вычисления в фоне

```javascript
// Класс для выполнения вычислений в воркере
class CalculationWorker {
    constructor(workerScript) {
        this.worker = new Worker(workerScript);
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error } = event.data;
            
            const promise = this.pendingPromises.get(id);
            if (promise) {
                this.pendingPromises.delete(id);
                
                if (error) {
                    promise.reject(new Error(error));
                } else {
                    promise.resolve(result);
                }
            }
        };
    }
    
    executeCalculation(operation, data) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            this.worker.postMessage({ id, operation, data });
        });
    }
    
    // Вычисление чисел Фибоначчи
    fibonacci(n) {
        return this.executeCalculation('fibonacci', { n });
    }
    
    // Вычисление факториала
    factorial(n) {
        return this.executeCalculation('factorial', { n });
    }
    
    // Сортировка массива
    sortArray(array) {
        return this.executeCalculation('sort', { array });
    }
    
    // Поиск в массиве
    findInArray(array, searchValue) {
        return this.executeCalculation('find', { array, searchValue });
    }
    
    terminate() {
        this.worker.terminate();
    }
}

// Использование
const calcWorker = new CalculationWorker('calc-worker.js');

// Выполнение тяжелых вычислений в фоне
async function performCalculations() {
    try {
        const fibResult = await calcWorker.fibonacci(40);
        console.log('Число Фибоначчи:', fibResult);
        
        const factResult = await calcWorker.factorial(20);
        console.log('Факториал:', factResult);
        
        const sortedArray = await calcWorker.sortArray([5, 2, 8, 1, 9, 3]);
        console.log('Отсортированный массив:', sortedArray);
    } catch (error) {
        console.error('Ошибка вычислений:', error);
    }
}
```

```javascript
// calc-worker.js
self.onmessage = function(event) {
    const { id, operation, data } = event.data;
    
    try {
        let result;
        
        switch (operation) {
            case 'fibonacci':
                result = fibonacci(data.n);
                break;
                
            case 'factorial':
                result = factorial(data.n);
                break;
                
            case 'sort':
                result = data.array.sort((a, b) => a - b);
                break;
                
            case 'find':
                result = data.array.indexOf(data.searchValue);
                break;
                
            default:
                throw new Error(`Неизвестная операция: ${operation}`);
        }
        
        self.postMessage({ id, result });
    } catch (error) {
        self.postMessage({ id, error: error.message });
    }
};

function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

### 2. Обработка изображений

```javascript
// Класс для обработки изображений в воркере
class ImageProcessor {
    constructor() {
        this.worker = new Worker('image-processor.js');
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error } = event.data;
            
            const promise = this.pendingPromises.get(id);
            if (promise) {
                this.pendingPromises.delete(id);
                
                if (error) {
                    promise.reject(new Error(error));
                } else {
                    promise.resolve(result);
                }
            }
        };
    }
    
    // Изменение яркости изображения
    adjustBrightness(imageData, brightness) {
        return this.executeOperation('adjustBrightness', { imageData, brightness });
    }
    
    // Применение фильтра размытия
    applyBlur(imageData, radius) {
        return this.executeOperation('applyBlur', { imageData, radius });
    }
    
    // Конвертация в градации серого
    convertToGrayscale(imageData) {
        return this.executeOperation('convertToGrayscale', { imageData });
    }
    
    executeOperation(operation, data) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            // Передаем imageData как Transferable Object
            this.worker.postMessage({ id, operation, data }, [data.imageData.data.buffer]);
        });
    }
    
    destroy() {
        this.worker.terminate();
    }
}

// Использование
const imageProcessor = new ImageProcessor();

async function processImage(canvas) {
    const ctx = canvas.getContext('2d');
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    
    try {
        // Увеличение яркости
        const processedData = await imageProcessor.adjustBrightness(imageData, 50);
        
        // Обратное применение обработанных данных к canvas
        ctx.putImageData(processedData, 0, 0);
    } catch (error) {
        console.error('Ошибка обработки изображения:', error);
    }
}
```

```javascript
// image-processor.js
self.onmessage = function(event) {
    const { id, operation, data } = event.data;
    
    try {
        let result;
        
        switch (operation) {
            case 'adjustBrightness':
                result = adjustBrightness(data.imageData, data.brightness);
                break;
                
            case 'applyBlur':
                result = applyBlur(data.imageData, data.radius);
                break;
                
            case 'convertToGrayscale':
                result = convertToGrayscale(data.imageData);
                break;
                
            default:
                throw new Error(`Неизвестная операция: ${operation}`);
        }
        
        // Возвращаем результат как Transferable Object
        self.postMessage({ id, result }, [result.data.buffer]);
    } catch (error) {
        self.postMessage({ id, error: error.message });
    }
};

function adjustBrightness(imageData, brightness) {
    const data = imageData.data;
    const result = new ImageData(new Uint8ClampedArray(data), imageData.width, imageData.height);
    const resultData = result.data;
    
    for (let i = 0; i < resultData.length; i += 4) {
        resultData[i] = Math.min(255, Math.max(0, resultData[i] + brightness));     // R
        resultData[i + 1] = Math.min(255, Math.max(0, resultData[i + 1] + brightness)); // G
        resultData[i + 2] = Math.min(255, Math.max(0, resultData[i + 2] + brightness)); // B
    }
    
    return result;
}

function convertToGrayscale(imageData) {
    const data = imageData.data;
    const result = new ImageData(new Uint8ClampedArray(data), imageData.width, imageData.height);
    const resultData = result.data;
    
    for (let i = 0; i < resultData.length; i += 4) {
        const gray = 0.299 * resultData[i] + 0.587 * resultData[i + 1] + 0.114 * resultData[i + 2];
        resultData[i] = gray;     // R
        resultData[i + 1] = gray; // G
        resultData[i + 2] = gray; // B
    }
    
    return result;
}
```

### 3. Обработка файлов

```javascript
// Класс для обработки файлов в воркере
class FileProcessor {
    constructor() {
        this.worker = new Worker('file-processor.js');
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error, progress } = event.data;
            
            if (progress !== undefined) {
                // Обработка прогресса
                this.onProgress && this.onProgress(progress);
            } else {
                const promise = this.pendingPromises.get(id);
                if (promise) {
                    this.pendingPromises.delete(id);
                    
                    if (error) {
                        promise.reject(new Error(error));
                    } else {
                        promise.resolve(result);
                    }
                }
            }
        };
    }
    
    setOnProgress(callback) {
        this.onProgress = callback;
    }
    
    // Анализ CSV файла
    analyzeCSV(file) {
        return this.executeOperation('analyzeCSV', { file });
    }
    
    // Конвертация JSON в CSV
    jsonToCSV(jsonData) {
        return this.executeOperation('jsonToCSV', { jsonData });
    }
    
    // Обработка JSON данных
    processJSON(jsonData) {
        return this.executeOperation('processJSON', { jsonData });
    }
    
    executeOperation(operation, data) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            this.worker.postMessage({ id, operation, data });
        });
    }
    
    destroy() {
        this.worker.terminate();
    }
}

// Использование
const fileProcessor = new FileProcessor();
fileProcessor.setOnProgress((progress) => {
    console.log(`Прогресс обработки: ${progress}%`);
});

async function handleFileUpload(fileInput) {
    const file = fileInput.files[0];
    
    if (file.type === 'text/csv') {
        try {
            const analysis = await fileProcessor.analyzeCSV(file);
            console.log('Анализ CSV:', analysis);
        } catch (error) {
            console.error('Ошибка анализа CSV:', error);
        }
    }
}
```

## Продвинутые возможности

### Класс для управления несколькими воркерами

```javascript
// Пул воркеров для распределения нагрузки
class WorkerPool {
    constructor(workerScript, size = 4) {
        this.workers = [];
        this.taskQueue = [];
        this.workerStatus = []; // индекс воркера -> занято/свободно
        this.pendingTasks = new Map();
        this.messageId = 0;
        
        // Создание воркеров
        for (let i = 0; i < size; i++) {
            const worker = new Worker(workerScript);
            this.workers.push(worker);
            this.workerStatus.push(false); // свободен
            
            // Настройка обработчика сообщений
            worker.onmessage = (event) => {
                const { id, result, error } = event.data;
                const task = this.pendingTasks.get(id);
                
                if (task) {
                    this.pendingTasks.delete(id);
                    this.workerStatus[task.workerIndex] = false; // освобождаем воркер
                    task.resolve(error ? Promise.reject(new Error(error)) : result);
                    
                    // Проверяем очередь задач
                    this.processQueue();
                }
            };
        }
    }
    
    executeTask(taskData) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            const task = { id, taskData, resolve, reject };
            
            // Проверяем, есть ли свободные воркеры
            const availableWorkerIndex = this.workerStatus.findIndex(status => !status);
            
            if (availableWorkerIndex !== -1) {
                // Воркер доступен, отправляем задачу
                this.assignTaskToWorker(task, availableWorkerIndex);
            } else {
                // Нет свободных воркеров, добавляем в очередь
                this.taskQueue.push(task);
            }
        });
    }
    
    assignTaskToWorker(task, workerIndex) {
        this.pendingTasks.set(task.id, task);
        this.workerStatus[workerIndex] = true; // помечаем воркер как занятый
        
        // Отправляем задачу воркеру
        this.workers[workerIndex].postMessage({
            id: task.id,
            ...task.taskData
        });
    }
    
    processQueue() {
        if (this.taskQueue.length > 0) {
            const availableWorkerIndex = this.workerStatus.findIndex(status => !status);
            
            if (availableWorkerIndex !== -1) {
                const task = this.taskQueue.shift();
                this.assignTaskToWorker(task, availableWorkerIndex);
            }
        }
    }
    
    terminate() {
        this.workers.forEach(worker => worker.terminate());
        this.workers = [];
        this.pendingTasks.clear();
        this.taskQueue = [];
    }
}

// Использование пула воркеров
const workerPool = new WorkerPool('task-worker.js', 4);

// Выполнение нескольких задач параллельно
async function performParallelTasks() {
    const tasks = [
        workerPool.executeTask({ operation: 'calculate', data: [1, 2, 3] }),
        workerPool.executeTask({ operation: 'calculate', data: [4, 5, 6] }),
        workerPool.executeTask({ operation: 'calculate', data: [7, 8, 9] }),
        workerPool.executeTask({ operation: 'calculate', data: [10, 11, 12] })
    ];
    
    try {
        const results = await Promise.all(tasks);
        console.log('Результаты параллельных вычислений:', results);
    } catch (error) {
        console.error('Ошибка в одной из задач:', error);
    }
}
```

### Обмен сообщениями между воркерами

```javascript
// Создание воркера, который может создавать других воркеров
class SupervisorWorker {
    constructor() {
        this.worker = new Worker('supervisor-worker.js');
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error } = event.data;
            
            const promise = this.pendingPromises.get(id);
            if (promise) {
                this.pendingPromises.delete(id);
                
                if (error) {
                    promise.reject(new Error(error));
                } else {
                    promise.resolve(result);
                }
            }
        };
    }
    
    // Запуск задачи, которая будет выполняться с использованием других воркеров
    runDistributedTask(task) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            this.worker.postMessage({ id, task });
        });
    }
    
    destroy() {
        this.worker.terminate();
    }
}

// supervisor-worker.js
/*
// Воркер может создавать других воркеров
self.onmessage = function(event) {
    const { id, task } = event.data;
    
    switch (task.type) {
        case 'mapReduce':
            handleMapReduce(id, task.data);
            break;
            
        default:
            self.postMessage({ id, error: 'Неизвестный тип задачи' });
    }
};

function handleMapReduce(taskId, data) {
    const chunkSize = Math.ceil(data.length / 4);
    const workers = [];
    let completed = 0;
    let results = [];
    
    for (let i = 0; i < 4; i++) {
        const worker = new Worker('map-worker.js');
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, data.length);
        const chunk = data.slice(start, end);
        
        worker.onmessage = function(event) {
            results[i] = event.data.result;
            completed++;
            
            if (completed === 4) {
                const finalResult = results.reduce((acc, val) => acc + val, 0);
                self.postMessage({ id: taskId, result: finalResult });
            }
            
            worker.terminate();
        };
        
        worker.postMessage({ data: chunk });
        workers.push(worker);
    }
}
*/
```

## Лучшие практики

### 1. Управление памятью

```javascript
// Класс для эффективного управления памятью в воркерах
class MemoryEfficientWorker {
    constructor(workerScript) {
        this.worker = new Worker(workerScript);
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error } = event.data;
            
            const promise = this.pendingPromises.get(id);
            if (promise) {
                this.pendingPromises.delete(id);
                
                if (error) {
                    promise.reject(new Error(error));
                } else {
                    promise.resolve(result);
                }
            }
        };
    }
    
    // Использование Transferable Objects для больших данных
    processLargeData(arrayBuffer) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            // Используем transfer для эффективной передачи больших данных
            this.worker.postMessage({ id, buffer: arrayBuffer }, [arrayBuffer]);
        });
    }
    
    // Очистка ресурсов
    destroy() {
        this.worker.terminate();
        this.pendingPromises.clear();
    }
}
```

### 2. Обработка ошибок

```javascript
// Улучшенная обработка ошибок в воркерах
class ErrorHandlingWorker {
    constructor(workerScript) {
        this.worker = new Worker(workerScript);
        this.messageId = 0;
        this.pendingPromises = new Map();
        this.errorHandlers = [];
        
        this.worker.onmessage = (event) => {
            const { id, result, error, errorType } = event.data;
            
            const promise = this.pendingPromises.get(id);
            if (promise) {
                this.pendingPromises.delete(id);
                
                if (error) {
                    const workerError = new WorkerError(error, errorType);
                    promise.reject(workerError);
                    
                    // Вызов обработчиков ошибок
                    this.errorHandlers.forEach(handler => handler(workerError));
                } else {
                    promise.resolve(result);
                }
            }
        };
        
        this.worker.onerror = (error) => {
            console.error('Необработанная ошибка воркера:', error);
            this.errorHandlers.forEach(handler => handler(error));
        };
    }
    
    onError(handler) {
        this.errorHandlers.push(handler);
    }
    
    executeTask(task) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            this.worker.postMessage({ id, task });
        });
    }
    
    destroy() {
        this.worker.terminate();
    }
}

// Класс для ошибок воркера
class WorkerError extends Error {
    constructor(message, type = 'UnknownError') {
        super(message);
        this.name = 'WorkerError';
        this.type = type;
    }
}

// Использование
const errorWorker = new ErrorHandlingWorker('error-worker.js');
errorWorker.onError((error) => {
    console.error('Ошибка в воркере:', error);
    // Логика обработки ошибки
});
```

### 3. Мониторинг производительности

```javascript
// Класс для мониторинга производительности воркеров
class WorkerPerformanceMonitor {
    constructor(worker) {
        this.worker = worker;
        this.metrics = {
            tasksCompleted: 0,
            totalExecutionTime: 0,
            averageExecutionTime: 0,
            maxExecutionTime: 0,
            minExecutionTime: Infinity
        };
    }
    
    wrapTask(taskFunction) {
        return async (...args) => {
            const startTime = performance.now();
            
            try {
                const result = await taskFunction(...args);
                
                const endTime = performance.now();
                const executionTime = endTime - startTime;
                
                // Обновление метрик
                this.metrics.tasksCompleted++;
                this.metrics.totalExecutionTime += executionTime;
                this.metrics.averageExecutionTime = this.metrics.totalExecutionTime / this.metrics.tasksCompleted;
                this.metrics.maxExecutionTime = Math.max(this.metrics.maxExecutionTime, executionTime);
                this.metrics.minExecutionTime = Math.min(this.metrics.minExecutionTime, executionTime);
                
                return result;
            } catch (error) {
                throw error;
            }
        };
    }
    
    getMetrics() {
        return { ...this.metrics };
    }
    
    resetMetrics() {
        this.metrics = {
            tasksCompleted: 0,
            totalExecutionTime: 0,
            averageExecutionTime: 0,
            maxExecutionTime: 0,
            minExecutionTime: Infinity
        };
    }
}
```

## Современные возможности

### ES Modules в воркерах

```javascript
// Использование ES Modules в воркерах
const moduleWorker = new Worker('module-worker.js', { type: 'module' });

// module-worker.js
/*
import { heavyCalculation } from './calculations.js';
import { processData } from './data-processing.js';

self.onmessage = function(event) {
    const { operation, data } = event.data;
    
    try {
        let result;
        
        switch (operation) {
            case 'calculate':
                result = heavyCalculation(data);
                break;
                
            case 'process':
                result = processData(data);
                break;
                
            default:
                throw new Error(`Неизвестная операция: ${operation}`);
        }
        
        self.postMessage(result);
    } catch (error) {
        self.postMessage({ error: error.message });
    }
};
*/

// calculations.js
/*
export function heavyCalculation(data) {
    // Выполнение тяжелого вычисления
    return data.reduce((sum, value) => sum + value * value, 0);
}
*/
```

### SharedArrayBuffer и Atomics

```javascript
// Использование SharedArrayBuffer для совместного доступа к памяти
function createSharedWorker() {
    // Создание разделяемого буфера
    const sharedBuffer = new SharedArrayBuffer(1024);
    const sharedArray = new Int32Array(sharedBuffer);
    
    const worker = new Worker('shared-worker.js');
    
    // Отправка разделяемого буфера воркеру
    worker.postMessage({ buffer: sharedBuffer });
    
    // Основной поток может читать и писать в разделяемый буфер
    setInterval(() => {
        // Атомарная операция инкремента
        Atomics.add(sharedArray, 0, 1);
        console.log('Значение счетчика:', Atomics.load(sharedArray, 0));
    }, 1000);
    
    return worker;
}

// shared-worker.js
/*
self.onmessage = function(event) {
    const { buffer } = event.data;
    const sharedArray = new Int32Array(buffer);
    
    // Воркер также может читать и писать в разделяемый буфер
    setInterval(() => {
        Atomics.add(sharedArray, 0, 2);
    }, 1500);
};
*/
```

Web Workers предоставляют мощный способ выполнения тяжелых вычислений в фоновом потоке, не блокируя пользовательский интерфейс. При правильном использовании они могут значительно улучшить производительность и отзывчивость веб-приложений.

#javascript #web-workers #multithreading #frontend #web-development #performance