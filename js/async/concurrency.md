# Конкурентность и параллелизм в асинхронном JavaScript

## Обзор

Конкурентность и параллелизм - важные концепции при работе с асинхронным JavaScript. Хотя JavaScript однопоточен в своей основе, он может эффективно обрабатывать множество асинхронных операций благодаря событийному циклу и веб-API.

## Понимание конкурентности в JavaScript

### Однопоточность vs Асинхронность

JavaScript работает в однопоточной среде, но может обрабатывать несколько асинхронных операций конкурентно:

```javascript
console.log('1. Начало выполнения');

// Эти операции будут выполнены конкурентно
setTimeout(() => console.log('4. Таймаут 1'), 1000);
setTimeout(() => console.log('3. Таймаут 2'), 500);

console.log('2. Синхронный код завершен');

// Выход: 1, 2, 3, 4
```

### Событийный цикл и очередь задач

```javascript
// Пример работы событийного цикла
console.log('1. Старт');

Promise.resolve().then(() => console.log('3. Microtask 1'));
Promise.resolve().then(() => console.log('4. Microtask 2'));

setTimeout(() => console.log('6. Callback 1'), 0);
setTimeout(() => console.log('7. Callback 2'), 0);

console.log('2. Синхронный код');

// Microtasks выполняются перед следующим макротаском
// Выход: 1, 2, 3, 4, 6, 7
```

## Управление конкурентностью

### Ограничение количества одновременных операций

```javascript
class ConcurrencyManager {
    constructor(maxConcurrent = 3) {
        this.maxConcurrent = maxConcurrent;
        this.running = 0;
        this.queue = [];
    }

    async add(task) {
        return new Promise((resolve, reject) => {
            this.queue.push({ task, resolve, reject });
            this.process();
        });
    }

    async process() {
        if (this.running >= this.maxConcurrent || this.queue.length === 0) {
            return;
        }

        this.running++;
        const { task, resolve, reject } = this.queue.shift();

        try {
            const result = await task();
            resolve(result);
        } catch (error) {
            reject(error);
        } finally {
            this.running--;
            this.process(); // Обработка следующей задачи
        }
    }

    async addAll(tasks) {
        return Promise.all(tasks.map(task => this.add(task)));
    }
}

// Использование менеджера конкурентности
const concurrencyManager = new ConcurrencyManager(2); // Только 2 одновременные операции

const urls = [
    'https://api.example.com/data1',
    'https://api.example.com/data2',
    'https://api.example.com/data3',
    'https://api.example.com/data4',
    'https://api.example.com/data5'
];

async function fetchAllData() {
    const fetchTasks = urls.map(url => () => fetch(url).then(res => res.json()));
    const results = await concurrencyManager.addAll(fetchTasks);
    console.log('Все данные получены:', results);
}
```

### Очередь задач с приоритетом

```javascript
class PriorityQueue {
    constructor() {
        this.queue = [];
    }

    add(task, priority = 0) {
        this.queue.push({ task, priority });
        // Сортировка по приоритету (высший приоритет первым)
        this.queue.sort((a, b) => b.priority - a.priority);
    }

    async process(maxConcurrent = 1) {
        const results = [];
        const running = new Set();

        while (this.queue.length > 0 || running.size > 0) {
            // Запуск задач до максимума
            while (this.queue.length > 0 && running.size < maxConcurrent) {
                const { task, priority } = this.queue.shift();
                const promise = task().then(
                    result => ({ result, priority, error: null }),
                    error => ({ result: null, priority, error })
                );
                
                running.add(promise);
                promise.then(() => running.delete(promise));
            }

            // Ожидание завершения хотя бы одной задачи
            if (running.size > 0) {
                const completed = await Promise.race(running);
                results.push(completed);
            }
        }

        return results;
    }
}

// Пример использования
const priorityQueue = new PriorityQueue();

// Добавление задач с разными приоритетами
priorityQueue.add(() => fetch('/api/high-priority').then(res => res.json()), 10);
priorityQueue.add(() => fetch('/api/low-priority').then(res => res.json()), 1);
priorityQueue.add(() => fetch('/api/medium-priority').then(res => res.json()), 5);

priorityQueue.process(3).then(results => {
    console.log('Результаты выполнения задач:', results);
});
```

## Параллельное выполнение операций

### Promise.all vs Promise.allSettled

```javascript
// Promise.all - все операции должны завершиться успешно
async function fetchAllRequired() {
    try {
        const [users, posts, comments] = await Promise.all([
            fetch('/api/users').then(res => res.json()),
            fetch('/api/posts').then(res => res.json()),
            fetch('/api/comments').then(res => res.json())
        ]);
        
        return { users, posts, comments };
    } catch (error) {
        console.error('Ошибка при загрузке данных:', error);
        throw error;
    }
}

// Promise.allSettled - продолжаем даже при ошибках
async function fetchAllOptional() {
    const results = await Promise.allSettled([
        fetch('/api/users').then(res => res.json()),
        fetch('/api/posts').then(res => res.json()),
        fetch('/api/comments').then(res => res.json())
    ]);

    const data = {};
    results.forEach((result, index) => {
        const keys = ['users', 'posts', 'comments'];
        if (result.status === 'fulfilled') {
            data[keys[index]] = result.value;
        } else {
            console.warn(`Ошибка при загрузке ${keys[index]}:`, result.reason);
            data[keys[index]] = null; // или значение по умолчанию
        }
    });

    return data;
}
```

### Управление параллельными запросами

```javascript
class ParallelRequestManager {
    constructor(options = {}) {
        this.maxConcurrent = options.maxConcurrent || 6;
        this.timeout = options.timeout || 10000;
        this.retryAttempts = options.retryAttempts || 3;
    }

    async executeInBatches(requests, batchSize = this.maxConcurrent) {
        const results = [];
        
        for (let i = 0; i < requests.length; i += batchSize) {
            const batch = requests.slice(i, i + batchSize);
            const batchResults = await Promise.allSettled(
                batch.map(request => this.executeWithRetry(request))
            );
            
            results.push(...batchResults);
        }
        
        return results;
    }

    async executeWithRetry(request, attempts = 0) {
        try {
            const controller = new AbortController();
            const timeoutId = setTimeout(() => controller.abort(), this.timeout);

            const response = await fetch(request.url, {
                ...request.options,
                signal: controller.signal
            });

            clearTimeout(timeoutId);

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            return await response.json();
        } catch (error) {
            clearTimeout(this.timeout);

            if (attempts < this.retryAttempts && !error.message.includes('AbortError')) {
                console.warn(`Попытка ${attempts + 1} не удалась, повтор...`);
                await this.delay(Math.pow(2, attempts) * 1000); // Экспоненциальная задержка
                return this.executeWithRetry(request, attempts + 1);
            }

            throw error;
        }
    }

    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// Использование менеджера параллельных запросов
const requestManager = new ParallelRequestManager({
    maxConcurrent: 4,
    timeout: 5000,
    retryAttempts: 2
});

const requests = [
    { url: '/api/users' },
    { url: '/api/posts' },
    { url: '/api/comments' },
    { url: '/api/likes' },
    { url: '/api/followers' },
    { url: '/api/messages' }
];

async function loadAllData() {
    try {
        const results = await requestManager.executeInBatches(requests, 3);
        
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Данные ${index} загружены:`, result.value);
            } else {
                console.error(`Ошибка при загрузке ${index}:`, result.reason);
            }
        });
    } catch (error) {
        console.error('Критическая ошибка:', error);
    }
}
```

## Паттерны конкурентности

### Semaphore (Семафор)

```javascript
class Semaphore {
    constructor(maxConcurrency) {
        this.maxConcurrency = maxConcurrency;
        this.currentConcurrency = 0;
        this.queue = [];
    }

    async acquire() {
        return new Promise((resolve) => {
            if (this.currentConcurrency < this.maxConcurrency) {
                this.currentConcurrency++;
                resolve();
            } else {
                this.queue.push(resolve);
            }
        });
    }

    release() {
        this.currentConcurrency--;
        
        if (this.queue.length > 0) {
            this.currentConcurrency++;
            const next = this.queue.shift();
            next();
        }
    }

    async execute(task) {
        await this.acquire();
        try {
            return await task();
        } finally {
            this.release();
        }
    }
}

// Использование семафора
const semaphore = new Semaphore(2); // Только 2 одновременные операции

async function limitedRequest(url) {
    return semaphore.execute(async () => {
        console.log(`Начало запроса к ${url}`);
        const response = await fetch(url);
        const data = await response.json();
        console.log(`Завершение запроса к ${url}`);
        return data;
    });
}

// Параллельные запросы с ограничением
const urls = [
    'https://api.example.com/data1',
    'https://api.example.com/data2',
    'https://api.example.com/data3',
    'https://api.example.com/data4'
];

Promise.all(urls.map(url => limitedRequest(url)))
    .then(results => console.log('Все результаты:', results));
```

### Pipeline

```javascript
class Pipeline {
    constructor(steps) {
        this.steps = steps;
    }

    async *processAsync(iterable) {
        let currentStream = iterable[Symbol.asyncIterator] ? 
            iterable : this.arrayToAsyncIterable(iterable);

        for (const step of this.steps) {
            const nextStream = this.applyStep(currentStream, step);
            currentStream = nextStream;
        }

        yield* currentStream;
    }

    async *applyStep(asyncIterable, step) {
        for await (const item of asyncIterable) {
            try {
                const result = await step(item);
                yield result;
            } catch (error) {
                console.error('Ошибка в шаге pipeline:', error);
                // В зависимости от стратегии, можно пропустить ошибку или пробросить
                yield { error: true, item, errorMessage: error.message };
            }
        }
    }

    arrayToAsyncIterable(array) {
        return {
            [Symbol.asyncIterator]() {
                let index = 0;
                return {
                    async next() {
                        if (index < array.length) {
                            return { value: array[index++], done: false };
                        }
                        return { done: true };
                    }
                };
            }
        };
    }
}

// Пример использования pipeline
const processingPipeline = new Pipeline([
    // Загрузка данных
    async (url) => {
        const response = await fetch(url);
        return response.json();
    },
    // Обработка данных
    async (data) => {
        return {
            ...data,
            processedAt: new Date().toISOString(),
            size: JSON.stringify(data).length
        };
    },
    // Валидация
    async (processedData) => {
        if (!processedData.id) {
            throw new Error('Отсутствует ID в данных');
        }
        return processedData;
    }
]);

async function processUrls(urls) {
    const results = [];
    for await (const result of processingPipeline.processAsync(urls)) {
        results.push(result);
    }
    return results;
}
```

## Продвинутые паттерны конкурентности

### Scatter-Gather

```javascript
class ScatterGather {
    constructor() {
        this.results = new Map();
        this.completed = new Set();
    }

    async scatter(tasks, timeout = 10000) {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), timeout);

        const promises = tasks.map(async (task, index) => {
            try {
                const result = await task();
                this.results.set(index, { success: true, data: result });
                this.completed.add(index);
            } catch (error) {
                this.results.set(index, { success: false, error });
                this.completed.add(index);
            }
        });

        await Promise.all(promises);
        clearTimeout(timeoutId);

        return this.gather();
    }

    gather() {
        const gathered = [];
        for (let i = 0; i < this.results.size; i++) {
            gathered.push(this.results.get(i));
        }
        return gathered;
    }

    getSuccessfulResults() {
        return Array.from(this.results.entries())
            .filter(([_, result]) => result.success)
            .map(([index, result]) => result.data);
    }

    getFailedResults() {
        return Array.from(this.results.entries())
            .filter(([_, result]) => !result.success)
            .map(([index, result]) => result.error);
    }
}

// Пример использования Scatter-Gather
const scatterGather = new ScatterGather();

const tasks = [
    () => fetch('/api/users').then(res => res.json()),
    () => fetch('/api/posts').then(res => res.json()),
    () => fetch('/api/comments').then(res => res.json()),
    () => fetch('/api/likes').then(res => res.json())
];

async function executeScatterGather() {
    try {
        const results = await scatterGather.scatter(tasks, 5000);
        
        console.log('Все результаты:', results);
        console.log('Успешные:', scatterGather.getSuccessfulResults());
        console.log('Неудачные:', scatterGather.getFailedResults());
    } catch (error) {
        console.error('Ошибка Scatter-Gather:', error);
    }
}
```

### Work Pool (Пул воркеров)

```javascript
class WorkerPool {
    constructor(workerFactory, size = 4) {
        this.workerFactory = workerFactory;
        this.size = size;
        this.workers = [];
        this.taskQueue = [];
        this.running = 0;
        
        this.initializeWorkers();
    }

    initializeWorkers() {
        for (let i = 0; i < this.size; i++) {
            this.workers.push({
                id: i,
                busy: false,
                worker: this.workerFactory()
            });
        }
    }

    async execute(task) {
        return new Promise((resolve, reject) => {
            this.taskQueue.push({ task, resolve, reject });
            this.processNext();
        });
    }

    async processNext() {
        if (this.running >= this.size || this.taskQueue.length === 0) {
            return;
        }

        const availableWorker = this.workers.find(w => !w.busy);
        if (!availableWorker) {
            return;
        }

        const { task, resolve, reject } = this.taskQueue.shift();
        availableWorker.busy = true;
        this.running++;

        try {
            const result = await task(availableWorker.worker);
            resolve(result);
        } catch (error) {
            reject(error);
        } finally {
            availableWorker.busy = false;
            this.running--;
            this.processNext(); // Обработка следующей задачи
        }
    }

    async executeAll(tasks) {
        return Promise.all(tasks.map(task => this.execute(task)));
    }
}

// Пример использования пула воркеров для обработки изображений
function createImageProcessor() {
    return {
        process: async (imageData) => {
            // Имитация обработки изображения
            await new Promise(resolve => setTimeout(resolve, Math.random() * 1000));
            return `Обработанное изображение: ${imageData}`;
        }
    };
}

const imageProcessorPool = new WorkerPool(createImageProcessor, 3);

const imageTasks = [
    (worker) => worker.process('image1.jpg'),
    (worker) => worker.process('image2.jpg'),
    (worker) => worker.process('image3.jpg'),
    (worker) => worker.process('image4.jpg'),
    (worker) => worker.process('image5.jpg')
];

async function processImages() {
    try {
        const results = await imageProcessorPool.executeAll(imageTasks);
        console.log('Результаты обработки изображений:', results);
    } catch (error) {
        console.error('Ошибка обработки изображений:', error);
    }
}
```

## Лучшие практики конкурентности

### Оптимизация загрузки данных

```javascript
class DataLoader {
    constructor(options = {}) {
        this.cache = new Map();
        this.pending = new Map();
        this.maxConcurrent = options.maxConcurrent || 6;
        this.ttl = options.ttl || 300000; // 5 минут
    }

    async load(keys) {
        // Разделение на кэшированные, ожидающие и новые запросы
        const cached = [];
        const pending = [];
        const toLoad = [];

        for (const key of keys) {
            if (this.cache.has(key) && !this.isExpired(key)) {
                cached.push({ key, data: this.cache.get(key).data });
            } else if (this.pending.has(key)) {
                pending.push({ key, promise: this.pending.get(key) });
            } else {
                toLoad.push(key);
            }
        }

        // Загрузка новых данных с ограничением по конкурентности
        let loaded = [];
        if (toLoad.length > 0) {
            loaded = await this.loadBatch(toLoad);
        }

        // Сбор всех результатов
        const allResults = [...cached, ...pending, ...loaded];
        return this.organizeResults(keys, allResults);
    }

    async loadBatch(keys) {
        const results = [];
        const concurrencyManager = new ConcurrencyManager(this.maxConcurrent);

        const loadTasks = keys.map(key => async () => {
            // Регистрация запроса как ожидающего
            const promise = this.fetchData(key);
            this.pending.set(key, promise);

            try {
                const data = await promise;
                this.cache.set(key, { data, timestamp: Date.now() });
                return { key, data };
            } finally {
                this.pending.delete(key);
            }
        });

        return concurrencyManager.addAll(loadTasks);
    }

    async fetchData(key) {
        // Имитация загрузки данных
        const response = await fetch(`/api/data/${key}`);
        return response.json();
    }

    isExpired(key) {
        const cached = this.cache.get(key);
        return Date.now() - cached.timestamp > this.ttl;
    }

    organizeResults(keys, results) {
        const resultMap = new Map(results.map(r => [r.key, r.data]));
        return keys.map(key => resultMap.get(key) || null);
    }
}

// Использование оптимизированного загрузчика данных
const dataLoader = new DataLoader({ maxConcurrent: 4, ttl: 300000 });

async function loadUserData(userIds) {
    const users = await dataLoader.load(userIds);
    console.log('Пользователи загружены:', users);
}
```