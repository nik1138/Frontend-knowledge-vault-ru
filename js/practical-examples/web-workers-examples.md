---
aliases: ["Примеры использования Web Workers", "Web Workers в JavaScript"]
tags: [js, practical-examples, web-workers, frontend]
---

# Примеры использования Web Workers

Практические примеры использования Web Workers для выполнения вычислительно сложных задач в фоновом потоке без блокировки основного потока пользовательского интерфейса.

## 1. Основы Web Workers

### Создание простого Web Worker

Создадим файл `worker.js` для выполнения вычислений:

```javascript
// worker.js
// Обработка сообщений от основного потока
self.onmessage = function(e) {
    const data = e.data;
    
    switch(data.command) {
        case 'calculate':
            performCalculation(data.payload);
            break;
        case 'fibonacci':
            calculateFibonacci(data.n);
            break;
        case 'sort':
            performSort(data.array);
            break;
        default:
            console.log('Неизвестная команда:', data.command);
    }
};

// Выполнение сложных вычислений
function performCalculation(payload) {
    const startTime = Date.now();
    
    // Симуляция сложных вычислений
    let result = 0;
    for (let i = 0; i < payload.iterations; i++) {
        result += Math.sqrt(i) * Math.sin(i);
    }
    
    const endTime = Date.now();
    
    // Отправка результата в основной поток
    self.postMessage({
        command: 'calculationComplete',
        result: result,
        time: endTime - startTime,
        originalPayload: payload
    });
}

// Вычисление чисел Фибоначчи
function calculateFibonacci(n) {
    if (n <= 1) {
        self.postMessage({ command: 'fibonacciResult', n: n, result: n });
        return;
    }
    
    let a = 0, b = 1;
    for (let i = 2; i <= n; i++) {
        const temp = a + b;
        a = b;
        b = temp;
    }
    
    self.postMessage({ command: 'fibonacciResult', n: n, result: b });
}

// Сортировка массива
function performSort(array) {
    const startTime = Date.now();
    const sortedArray = [...array].sort((a, b) => a - b);
    const endTime = Date.now();
    
    self.postMessage({
        command: 'sortComplete',
        result: sortedArray,
        time: endTime - startTime
    });
}
```

### Использование Web Worker в основном потоке

```javascript
class WebWorkerManager {
    constructor(workerScriptPath) {
        this.worker = new Worker(workerScriptPath);
        this.setupMessageHandler();
        this.messageHandlers = new Map();
        this.messageId = 0;
    }
    
    setupMessageHandler() {
        this.worker.onmessage = (e) => {
            const data = e.data;
            
            if (data.id && this.messageHandlers.has(data.id)) {
                const handler = this.messageHandlers.get(data.id);
                handler.resolve(data);
                this.messageHandlers.delete(data.id);
            } else {
                // Обработка общих сообщений
                this.handleGeneralMessage(data);
            }
        };
        
        this.worker.onerror = (error) => {
            console.error('Ошибка Web Worker:', error);
        };
    }
    
    handleGeneralMessage(data) {
        switch(data.command) {
            case 'calculationComplete':
                console.log(`Вычисления завершены за ${data.time}мс`);
                break;
            case 'fibonacciResult':
                console.log(`F(${data.n}) = ${data.result}`);
                break;
            case 'sortComplete':
                console.log(`Сортировка завершена за ${data.time}мс`);
                break;
        }
    }
    
    sendMessage(command, payload) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.messageHandlers.set(id, { resolve, reject });
            
            this.worker.postMessage({
                id: id,
                command: command,
                payload: payload
            });
        });
    }
    
    terminate() {
        this.worker.terminate();
    }
}

// Пример использования
const workerManager = new WebWorkerManager('worker.js');

// Выполнение сложных вычислений
workerManager.sendMessage('calculate', {
    iterations: 10000000
}).then(result => {
    console.log('Результат вычислений:', result.result);
    console.log('Время выполнения:', result.time, 'мс');
});

// Вычисление числа Фибоначчи
workerManager.sendMessage('fibonacci', { n: 40 }).then(result => {
    console.log(`F(40) = ${result.result}`);
});
```

## 2. Продвинутые примеры Web Workers

### Обработка изображений

```javascript
// image-worker.js
self.onmessage = function(e) {
    const data = e.data;
    
    switch(data.command) {
        case 'processImage':
            processImage(data.imageData, data.operation, data.params);
            break;
        case 'resizeImage':
            resizeImage(data.imageData, data.newWidth, data.newHeight);
            break;
    }
};

// Изменение яркости изображения
function adjustBrightness(imageData, brightness) {
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
        data[i] = clamp(data[i] + brightness, 0, 255);     // R
        data[i + 1] = clamp(data[i + 1] + brightness, 0, 255); // G
        data[i + 2] = clamp(data[i + 2] + brightness, 0, 255); // B
    }
    
    return imageData;
}

// Изменение контрастности изображения
function adjustContrast(imageData, contrast) {
    const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
        data[i] = clamp(factor * (data[i] - 128) + 128, 0, 255);     // R
        data[i + 1] = clamp(factor * (data[i + 1] - 128) + 128, 0, 255); // G
        data[i + 2] = clamp(factor * (data[i + 2] - 128) + 128, 0, 255); // B
    }
    
    return imageData;
}

// Вспомогательная функция для ограничения значений
function clamp(value, min, max) {
    return Math.min(Math.max(value, min), max);
}

// Обработка изображения
function processImage(imageData, operation, params) {
    let result;
    
    switch(operation) {
        case 'brightness':
            result = adjustBrightness(imageData, params.value);
            break;
        case 'contrast':
            result = adjustContrast(imageData, params.value);
            break;
        default:
            result = imageData;
    }
    
    self.postMessage({
        command: 'imageProcessed',
        result: result,
        operation: operation
    });
}

// Изменение размера изображения (упрощенная реализация)
function resizeImage(imageData, newWidth, newHeight) {
    const oldWidth = imageData.width;
    const oldHeight = imageData.height;
    const newData = new Uint8ClampedArray(newWidth * newHeight * 4);
    
    // Простое масштабирование без интерполяции
    for (let y = 0; y < newHeight; y++) {
        for (let x = 0; x < newWidth; x++) {
            const oldX = Math.floor(x * oldWidth / newWidth);
            const oldY = Math.floor(y * oldHeight / newHeight);
            
            const oldIndex = (oldY * oldWidth + oldX) * 4;
            const newIndex = (y * newWidth + x) * 4;
            
            newData[newIndex] = imageData.data[oldIndex];       // R
            newData[newIndex + 1] = imageData.data[oldIndex + 1]; // G
            newData[newIndex + 2] = imageData.data[oldIndex + 2]; // B
            newData[newIndex + 3] = imageData.data[oldIndex + 3]; // A
        }
    }
    
    self.postMessage({
        command: 'imageResized',
        result: {
            data: newData,
            width: newWidth,
            height: newHeight
        }
    });
}
```

### Обработка больших массивов данных

```javascript
// array-worker.js
self.onmessage = function(e) {
    const data = e.data;
    
    switch(data.command) {
        case 'aggregate':
            performAggregation(data.array, data.operation);
            break;
        case 'filter':
            performFilter(data.array, data.filterFn);
            break;
        case 'map':
            performMap(data.array, data.mapFn);
            break;
        case 'search':
            performSearch(data.array, data.searchTerm);
            break;
    }
};

// Агрегация данных (сумма, среднее, мин, макс)
function performAggregation(array, operation) {
    let result;
    
    switch(operation) {
        case 'sum':
            result = array.reduce((sum, item) => sum + item, 0);
            break;
        case 'average':
            result = array.reduce((sum, item) => sum + item, 0) / array.length;
            break;
        case 'min':
            result = Math.min(...array);
            break;
        case 'max':
            result = Math.max(...array);
            break;
        case 'count':
            result = array.length;
            break;
    }
    
    self.postMessage({
        command: 'aggregationComplete',
        result: result,
        operation: operation
    });
}

// Фильтрация массива
function performFilter(array, filterFn) {
    // В реальном приложении filterFn нужно безопасно десериализовать
    // или передавать как строку и использовать new Function()
    const filterFunction = new Function('item', 'return ' + filterFn);
    const filteredArray = array.filter(item => filterFunction(item));
    
    self.postMessage({
        command: 'filterComplete',
        result: filteredArray
    });
}

// Маппинг массива
function performMap(array, mapFn) {
    const mapFunction = new Function('item', 'return ' + mapFn);
    const mappedArray = array.map(item => mapFunction(item));
    
    self.postMessage({
        command: 'mapComplete',
        result: mappedArray
    });
}

// Поиск в массиве
function performSearch(array, searchTerm) {
    const results = array.filter(item => 
        JSON.stringify(item).toLowerCase().includes(searchTerm.toLowerCase())
    );
    
    self.postMessage({
        command: 'searchComplete',
        result: results,
        searchTerm: searchTerm
    });
}
```

## 3. Многопоточная обработка с использованием нескольких Workers

```javascript
// Менеджер для управления несколькими Workers
class MultiWorkerManager {
    constructor(workerScriptPath, numWorkers = 4) {
        this.workers = [];
        this.availableWorkers = [];
        this.taskQueue = [];
        
        // Создаем указанное количество Workers
        for (let i = 0; i < numWorkers; i++) {
            const worker = new Worker(workerScriptPath);
            worker.id = i;
            worker.busy = false;
            
            worker.onmessage = (e) => {
                worker.busy = false;
                this.handleWorkerMessage(worker, e.data);
                this.processNextTask();
            };
            
            this.workers.push(worker);
            this.availableWorkers.push(worker);
        }
    }
    
    handleWorkerMessage(worker, data) {
        // Обработка сообщения от Worker
        console.log(`Worker ${worker.id} завершил задачу:`, data);
        
        // Здесь можно реализовать логику обработки результата
        if (data.onComplete) {
            data.onComplete(data.result);
        }
    }
    
    // Отправка задачи одному из доступных Workers
    assignTask(task) {
        if (this.availableWorkers.length > 0) {
            const worker = this.availableWorkers.pop();
            worker.busy = true;
            worker.postMessage(task);
        } else {
            // Если нет доступных Workers, добавляем в очередь
            this.taskQueue.push(task);
        }
    }
    
    // Обработка следующей задачи из очереди
    processNextTask() {
        if (this.taskQueue.length > 0 && this.availableWorkers.length > 0) {
            const task = this.taskQueue.shift();
            this.assignTask(task);
        }
    }
    
    // Разделение большой задачи на части для параллельной обработки
    parallelProcess(array, processor, chunkSize = 1000) {
        return new Promise((resolve, reject) => {
            const chunks = this.createChunks(array, chunkSize);
            const results = new Array(chunks.length);
            let completedTasks = 0;
            
            chunks.forEach((chunk, index) => {
                this.assignTask({
                    command: 'processChunk',
                    chunk: chunk,
                    processor: processor,
                    chunkIndex: index,
                    onComplete: (result) => {
                        results[index] = result;
                        completedTasks++;
                        
                        if (completedTasks === chunks.length) {
                            resolve(this.mergeResults(results));
                        }
                    }
                });
            });
        });
    }
    
    // Создание чанков из массива
    createChunks(array, chunkSize) {
        const chunks = [];
        for (let i = 0; i < array.length; i += chunkSize) {
            chunks.push(array.slice(i, i + chunkSize));
        }
        return chunks;
    }
    
    // Объединение результатов
    mergeResults(results) {
        return results.flat();
    }
    
    // Завершение всех Workers
    terminateAll() {
        this.workers.forEach(worker => worker.terminate());
    }
}
```

## 4. Примеры использования в реальных сценариях

### Обработка CSV-файлов

```javascript
// csv-worker.js
self.onmessage = function(e) {
    const data = e.data;
    
    switch(data.command) {
        case 'parseCSV':
            parseCSV(data.csvString, data.options);
            break;
        case 'transformData':
            transformData(data.rows, data.transformations);
            break;
        case 'generateCSV':
            generateCSV(data.rows, data.headers);
            break;
    }
};

// Парсинг CSV-строки
function parseCSV(csvString, options = {}) {
    const delimiter = options.delimiter || ',';
    const hasHeader = options.hasHeader !== false;
    
    const lines = csvString.split('\n').filter(line => line.trim() !== '');
    const headers = hasHeader ? lines[0].split(delimiter) : [];
    const rows = hasHeader ? lines.slice(1) : lines;
    
    const parsedData = rows.map(row => {
        const values = row.split(delimiter);
        if (hasHeader) {
            const obj = {};
            headers.forEach((header, index) => {
                obj[header.trim()] = values[index] ? values[index].trim() : '';
            });
            return obj;
        }
        return values;
    });
    
    self.postMessage({
        command: 'csvParsed',
        data: parsedData,
        headers: headers,
        rowCount: parsedData.length
    });
}

// Преобразование данных
function transformData(rows, transformations) {
    const transformedRows = rows.map(row => {
        const newRow = { ...row };
        
        transformations.forEach(transform => {
            if (transform.type === 'convertType') {
                newRow[transform.field] = convertType(newRow[transform.field], transform.targetType);
            } else if (transform.type === 'addComputedField') {
                newRow[transform.newField] = transform.computeFn(newRow);
            }
        });
        
        return newRow;
    });
    
    self.postMessage({
        command: 'dataTransformed',
        data: transformedRows
    });
}

// Преобразование типов данных
function convertType(value, targetType) {
    switch(targetType) {
        case 'number':
            return Number(value);
        case 'string':
            return String(value);
        case 'boolean':
            return value.toLowerCase() === 'true';
        case 'date':
            return new Date(value);
        default:
            return value;
    }
}

// Генерация CSV из данных
function generateCSV(rows, headers) {
    if (!rows || rows.length === 0) {
        self.postMessage({ command: 'csvGenerated', csv: '' });
        return;
    }
    
    const csvRows = [];
    
    // Добавляем заголовки
    if (headers) {
        csvRows.push(headers.join(','));
    } else {
        // Если заголовки не указаны, используем ключи первого объекта
        const firstRow = rows[0];
        const headerRow = Array.isArray(firstRow) ? 
            firstRow.map((_, index) => `Column${index}`) : 
            Object.keys(firstRow);
        csvRows.push(headerRow.join(','));
    }
    
    // Добавляем данные
    rows.forEach(row => {
        const values = Array.isArray(row) ? row : Object.values(row);
        const escapedValues = values.map(value => {
            // Экранируем значения, содержащие запятые или кавычки
            const str = String(value);
            return str.includes(',') || str.includes('"') || str.includes('\n') ?
                `"${str.replace(/"/g, '""')}"` : str;
        });
        csvRows.push(escapedValues.join(','));
    });
    
    self.postMessage({
        command: 'csvGenerated',
        csv: csvRows.join('\n')
    });
}
```

### Шифрование и хеширование данных

```javascript
// crypto-worker.js
self.onmessage = function(e) {
    const data = e.data;
    
    switch(data.command) {
        case 'hash':
            calculateHash(data.data, data.algorithm);
            break;
        case 'encrypt':
            encryptData(data.plainText, data.key, data.algorithm);
            break;
        case 'decrypt':
            decryptData(data.encryptedData, data.key, data.algorithm);
            break;
    }
};

// Вычисление хеша
async function calculateHash(data, algorithm = 'SHA-256') {
    try {
        const encoder = new TextEncoder();
        const dataBuffer = encoder.encode(data);
        const hashBuffer = await crypto.subtle.digest(algorithm, dataBuffer);
        const hashArray = Array.from(new Uint8Array(hashBuffer));
        const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
        
        self.postMessage({
            command: 'hashCalculated',
            algorithm: algorithm,
            hash: hashHex,
            originalDataLength: data.length
        });
    } catch (error) {
        self.postMessage({
            command: 'error',
            error: error.message
        });
    }
}

// Шифрование данных
async function encryptData(plainText, key, algorithm = 'AES-GCM') {
    try {
        // В реальном приложении ключ должен быть надежно сгенерирован и передан безопасно
        const encoder = new TextEncoder();
        const dataBuffer = encoder.encode(plainText);
        
        // Генерация случайного IV
        const iv = crypto.getRandomValues(new Uint8Array(12));
        
        // В реальном приложении: использовать crypto.subtle.importKey для импорта ключа
        // Здесь упрощенная реализация
        
        self.postMessage({
            command: 'dataEncrypted',
            encryptedData: btoa(String.fromCharCode(...dataBuffer)), // упрощенная "шифровка"
            iv: btoa(String.fromCharCode(...iv)),
            algorithm: algorithm
        });
    } catch (error) {
        self.postMessage({
            command: 'error',
            error: error.message
        });
    }
}

// Расшифровка данных
function decryptData(encryptedData, key, algorithm = 'AES-GCM') {
    try {
        const dataBuffer = new Uint8Array(atob(encryptedData).split('').map(c => c.charCodeAt(0)));
        const decoder = new TextDecoder();
        const plainText = decoder.decode(dataBuffer);
        
        self.postMessage({
            command: 'dataDecrypted',
            plainText: plainText,
            algorithm: algorithm
        });
    } catch (error) {
        self.postMessage({
            command: 'error',
            error: error.message
        });
    }
}
```

## 5. Обработка ошибок и отладка

```javascript
// worker-manager.js - Улучшенный менеджер с обработкой ошибок
class AdvancedWorkerManager {
    constructor(workerScriptPath, options = {}) {
        this.workerScriptPath = workerScriptPath;
        this.maxRetries = options.maxRetries || 3;
        this.timeout = options.timeout || 30000; // 30 секунд
        this.workers = [];
        this.activeTasks = new Map();
        this.taskId = 0;
        
        this.createWorkers(options.numWorkers || 4);
    }
    
    createWorkers(numWorkers) {
        for (let i = 0; i < numWorkers; i++) {
            const worker = this.createWorker();
            this.workers.push({
                worker: worker,
                id: i,
                busy: false,
                retryCount: 0
            });
        }
    }
    
    createWorker() {
        const worker = new Worker(this.workerScriptPath);
        
        worker.onmessage = (e) => {
            const task = this.activeTasks.get(e.data.taskId);
            if (task) {
                clearTimeout(task.timeoutId);
                task.resolve(e.data);
                this.markWorkerAvailable(worker);
                this.activeTasks.delete(e.data.taskId);
            }
        };
        
        worker.onerror = (error) => {
            console.error('Ошибка Worker:', error);
            // Попытка перезапуска Worker
            this.restartWorker(worker);
        };
        
        return worker;
    }
    
    async executeTask(command, payload, options = {}) {
        return new Promise((resolve, reject) => {
            const taskId = ++this.taskId;
            const worker = this.getAvailableWorker();
            
            if (!worker) {
                reject(new Error('Нет доступных Workers'));
                return;
            }
            
            // Устанавливаем таймаут
            const timeoutId = setTimeout(() => {
                reject(new Error(`Таймаут выполнения задачи ${command}`));
                this.markWorkerAvailable(worker.worker);
                this.activeTasks.delete(taskId);
            }, options.timeout || this.timeout);
            
            // Регистрируем задачу
            const task = {
                resolve,
                reject,
                timeoutId,
                worker: worker.worker
            };
            
            this.activeTasks.set(taskId, task);
            worker.busy = true;
            
            // Отправляем задачу
            worker.worker.postMessage({
                taskId: taskId,
                command: command,
                payload: payload
            });
        });
    }
    
    getAvailableWorker() {
        return this.workers.find(w => !w.busy);
    }
    
    markWorkerAvailable(worker) {
        const workerInfo = this.workers.find(w => w.worker === worker);
        if (workerInfo) {
            workerInfo.busy = false;
            workerInfo.retryCount = 0;
        }
    }
    
    restartWorker(worker) {
        const workerInfo = this.workers.find(w => w.worker === worker);
        if (workerInfo && workerInfo.retryCount < this.maxRetries) {
            workerInfo.retryCount++;
            workerInfo.worker.terminate();
            workerInfo.worker = this.createWorker();
        } else if (workerInfo) {
            console.error('Превышено количество попыток перезапуска Worker');
        }
    }
    
    terminateAll() {
        this.workers.forEach(workerInfo => {
            workerInfo.worker.terminate();
        });
        this.workers = [];
        this.activeTasks.clear();
    }
}

// Пример использования с обработкой ошибок
const advancedManager = new AdvancedWorkerManager('worker.js');

async function safeWorkerTask() {
    try {
        const result = await advancedManager.executeTask('calculate', {
            iterations: 1000000
        }, { timeout: 10000 });
        
        console.log('Результат:', result);
    } catch (error) {
        console.error('Ошибка выполнения задачи:', error.message);
    }
}
```

Web Workers позволяют выполнять тяжелые вычисления в фоновом потоке, не блокируя пользовательский интерфейс. Это особенно полезно для обработки больших объемов данных, сложных вычислений, шифрования и других ресурсоемких операций.

См. также:
- [[js/practical-examples/canvas-graphics-examples]] - Работа с canvas и графикой
- [[js/practical-examples/web-api-examples]] - Работа с Web API
- [[js/practical-examples/fetch-websocket-examples]] - Примеры с Fetch API и WebSocket