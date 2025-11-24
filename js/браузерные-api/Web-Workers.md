---
aliases: [Веб-воркеры, Web Worker, Background Workers]
tags: [javascript, web-api, concurrency, performance, frontend, многопоточность]
---

# Web Workers

## Обзор

Web Workers - это API браузера, позволяющее запускать JavaScript в фоновом потоке, независимо от основного потока пользовательского интерфейса. Это позволяет выполнять тяжелые вычисления, обработку данных и другие ресурсоемкие операции без блокировки основного потока, что предотвращает "замораживание" интерфейса и обеспечивает плавность пользовательского опыта.

## Типы Web Workers

1. **Dedicated Workers** - привязаны к одному скрипту/окну
2. **Shared Workers** - могут использоваться несколькими скриптами/окнами одного домена
3. **Service Workers** - специализированный тип для кэширования и управления сетевыми запросами (рассматривается отдельно)

## Основы Dedicated Workers

### Создание и запуск Worker

```javascript
// main.js (основной поток)
const worker = new Worker('worker.js');

// Отправка сообщения в воркер
worker.postMessage({
    type: 'calculate',
    data: [1, 2, 3, 4, 5, /* ... */ 1000000]
});

// Обработка сообщений от воркера
worker.onmessage = function(e) {
    console.log('Результат из воркера:', e.data);
};

// Обработка ошибок
worker.onerror = function(error) {
    console.error('Ошибка в воркере:', error);
    error.preventDefault(); // Предотвращение выброса исключения
};
```

```javascript
// worker.js (файл воркера)
self.onmessage = function(e) {
    const { type, data } = e.data;
    
    if (type === 'calculate') {
        // Выполнение тяжелого вычисления
        const result = heavyCalculation(data);
        
        // Отправка результата в основной поток
        self.postMessage({
            type: 'result',
            data: result,
            originalDataLength: data.length
        });
    }
};

function heavyCalculation(numbers) {
    // Синтетический пример тяжелого вычисления
    return numbers.map(n => Math.sqrt(n * n * n)).reduce((a, b) => a + b, 0);
}
```

### Альтернативный способ обработки сообщений

```javascript
// main.js
worker.addEventListener('message', function(e) {
    const { type, data } = e.data;
    
    switch(type) {
        case 'result':
            handleCalculationResult(data);
            break;
        case 'progress':
            updateProgress(data.progress);
            break;
        case 'error':
            handleError(data.message);
            break;
    }
});

// worker.js
self.addEventListener('message', function(e) {
    const { type, data } = e.data;
    
    try {
        if (type === 'processLargeData') {
            processDataWithProgress(data);
        }
    } catch (error) {
        self.postMessage({
            type: 'error',
            message: error.message
        });
    }
});
```

## Практические примеры использования

### Обработка больших массивов данных

```javascript
// main.js
class DataProcessor {
    constructor() {
        this.worker = new Worker('data-processor-worker.js');
        this.callbacks = new Map();
        this.messageId = 0;
    }
    
    processLargeArray(data, options = {}) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            
            // Сохраняем колбэки для обработки результата
            this.callbacks.set(id, { resolve, reject });
            
            this.worker.postMessage({
                id,
                type: 'processArray',
                data,
                options
            });
        });
    }
    
    init() {
        this.worker.onmessage = (e) => {
            const { id, type, data, error } = e.data;
            
            const callback = this.callbacks.get(id);
            if (callback) {
                if (type === 'result') {
                    callback.resolve(data);
                } else if (type === 'error') {
                    callback.reject(new Error(error));
                }
                this.callbacks.delete(id);
            }
        };
    }
}

// Использование
const processor = new DataProcessor();
processor.init();

processor.processLargeArray(largeDataSet, { algorithm: 'sort' })
    .then(result => {
        console.log('Обработка завершена:', result);
        updateUI(result);
    })
    .catch(error => {
        console.error('Ошибка обработки:', error);
    });
```

```javascript
// data-processor-worker.js
self.onmessage = function(e) {
    const { id, type, data, options } = e.data;
    
    try {
        let result;
        
        switch(type) {
            case 'processArray':
                result = processArray(data, options);
                break;
            case 'filterData':
                result = filterData(data, options);
                break;
            default:
                throw new Error(`Неизвестный тип операции: ${type}`);
        }
        
        self.postMessage({
            id,
            type: 'result',
            data: result
        });
    } catch (error) {
        self.postMessage({
            id,
            type: 'error',
            error: error.message
        });
    }
};

function processArray(array, options) {
    // Тяжелые вычисления
    switch(options.algorithm) {
        case 'sort':
            return array.sort((a, b) => a - b);
        case 'aggregate':
            return array.reduce((acc, val) => {
                acc.sum += val;
                acc.count++;
                acc.avg = acc.sum / acc.count;
                return acc;
            }, { sum: 0, count: 0, avg: 0 });
        case 'transform':
            return array.map(item => ({
                ...item,
                processed: true,
                timestamp: Date.now()
            }));
        default:
            return array;
    }
}
```

### Генерация сложных графиков

```javascript
// chart-worker.js
self.onmessage = function(e) {
    const { type, data, options } = e.data;
    
    if (type === 'generateChart') {
        const chartData = generateChartData(data, options);
        
        self.postMessage({
            type: 'chartReady',
            imageData: chartData,
            dimensions: options.dimensions
        });
    }
};

function generateChartData(rawData, options) {
    // Генерация изображения графика
    const canvas = new OffscreenCanvas(options.width, options.height);
    const ctx = canvas.getContext('2d');
    
    // Рисование графика
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Ось X и Y
    ctx.strokeStyle = '#cccccc';
    ctx.beginPath();
    ctx.moveTo(50, 20);
    ctx.lineTo(50, canvas.height - 20);
    ctx.lineTo(canvas.width - 20, canvas.height - 20);
    ctx.stroke();
    
    // График
    const maxValue = Math.max(...rawData.map(d => d.value));
    const pointCount = rawData.length;
    const xStep = (canvas.width - 70) / (pointCount - 1);
    
    ctx.strokeStyle = '#4ECDC4';
    ctx.lineWidth = 2;
    ctx.beginPath();
    
    rawData.forEach((point, index) => {
        const x = 50 + index * xStep;
        const y = canvas.height - 20 - (point.value / maxValue) * (canvas.height - 40);
        
        if (index === 0) {
            ctx.moveTo(x, y);
        } else {
            ctx.lineTo(x, y);
        }
    });
    
    ctx.stroke();
    
    // Возврат данных изображения
    return canvas.transferToImageBitmap();
}
```

### Криптографические операции

```javascript
// crypto-worker.js
self.onmessage = function(e) {
    const { type, data, key } = e.data;
    
    switch(type) {
        case 'encrypt':
            const encrypted = encryptData(data, key);
            self.postMessage({
                type: 'encrypted',
                data: encrypted
            });
            break;
        case 'decrypt':
            const decrypted = decryptData(data, key);
            self.postMessage({
                type: 'decrypted',
                data: decrypted
            });
            break;
        case 'hash':
            const hash = generateHash(data);
            self.postMessage({
                type: 'hash',
                data: hash
            });
            break;
    }
};

function encryptData(data, key) {
    // Упрощенный пример шифрования (в реальных приложениях используйте Web Crypto API)
    return btoa(data.split('').map(char => 
        String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(0))
    ).join(''));
}

function decryptData(data, key) {
    return atob(data).split('').map(char => 
        String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(0))
    ).join('');
}

function generateHash(data) {
    // Простой хэш (в реальных приложениях используйте Web Crypto API)
    let hash = 0;
    for (let i = 0; i < data.length; i++) {
        const char = data.charCodeAt(i);
        hash = ((hash << 5) - hash) + char;
        hash |= 0; // Преобразование в 32-битное целое
    }
    return hash;
}
```

## Shared Workers

```javascript
// shared-worker.js
let connections = 0;

self.onconnect = function(e) {
    const port = e.ports[0];
    connections++;
    
    port.onmessage = function(e) {
        const { type, data } = e.data;
        
        switch(type) {
            case 'getSharedData':
                port.postMessage({
                    type: 'sharedData',
                    data: getSharedData(),
                    connections: connections
                });
                break;
            case 'updateSharedData':
                updateSharedData(data);
                // Оповещение всех подключений об изменении
                broadcastToAllConnections({
                    type: 'dataUpdated',
                    data: getSharedData()
                });
                break;
        }
    };
    
    // Отправка начальных данных
    port.postMessage({
        type: 'connected',
        connections: connections
    });
    
    port.start();
};

function broadcastToAllConnections(message) {
    // В реальной реализации нужно хранить все порты и отправлять сообщение каждому
    // Здесь упрощенный пример
}
```

```javascript
// main.js - использование Shared Worker
const sharedWorker = new SharedWorker('shared-worker.js');
const port = sharedWorker.port;

port.onmessage = function(e) {
    console.log('Сообщение от shared worker:', e.data);
};

port.postMessage({
    type: 'getSharedData',
    data: null
});

port.start();
```

## Обмен данными и производительность

### Structured Clone Algorithm

Web Workers использует Structured Clone Algorithm для передачи данных, который поддерживает:

- Примитивы (number, string, boolean)
- Объекты и массивы
- Даты
- RegExp
- Blob, File, FileList
- ArrayBuffer, ArrayBufferView
- ImageData
- Transferable объекты (ArrayBuffer, MessagePort, ImageBitmap)

```javascript
// Передача сложных объектов
const complexData = {
    users: [
        { id: 1, name: 'Иван', profile: { age: 30 } },
        { id: 2, name: 'Мария', profile: { age: 25 } }
    ],
    metadata: new Date(),
    settings: { theme: 'dark', lang: 'ru' }
};

worker.postMessage(complexData);

// Передача ArrayBuffer (эффективно для больших данных)
const buffer = new ArrayBuffer(1024 * 1024); // 1MB
const uint8Array = new Uint8Array(buffer);

// Заполнение данными
for (let i = 0; i < uint8Array.length; i++) {
    uint8Array[i] = i % 256;
}

// Передача с transfer, что делает буфер недоступным в основном потоке
worker.postMessage(buffer, [buffer]); // [buffer] - список передаваемых объектов
```

### Transferable Objects

```javascript
// main.js
const arrayBuffer = new ArrayBuffer(1024 * 1024);
const worker = new Worker('process-buffer.js');

// Передача буфера в воркер (очень эффективно)
worker.postMessage(arrayBuffer, [arrayBuffer]);

// Теперь arrayBuffer недоступен в основном потоке
// console.log(arrayBuffer.byteLength); // Ошибка!
```

```javascript
// process-buffer.js
self.onmessage = function(e) {
    const buffer = e.data;
    const uint8Array = new Uint8Array(buffer);
    
    // Обработка данных
    for (let i = 0; i < uint8Array.length; i++) {
        uint8Array[i] = uint8Array[i] * 2;
    }
    
    // Возврат обработанного буфера
    self.postMessage(buffer, [buffer]);
};
```

## Обработка ошибок и отладка

```javascript
// main.js
const worker = new Worker('worker.js');

worker.onerror = function(e) {
    console.error('Ошибка в воркере:', {
        message: e.message,
        filename: e.filename,
        lineno: e.lineno
    });
    
    // Логирование ошибки для анализа
    logErrorToServer({
        type: 'worker_error',
        message: e.message,
        filename: e.filename,
        lineno: e.lineno,
        timestamp: new Date().toISOString()
    });
};

worker.onmessageerror = function(e) {
    console.error('Ошибка десериализации сообщения:', e);
};

// worker.js
self.onerror = function(e) {
    // Отправка информации об ошибке в основной поток
    self.postMessage({
        type: 'system_error',
        message: e.message,
        filename: e.filename,
        lineno: e.lineno,
        colno: e.colno
    });
    
    // Предотвращение падения воркера
    e.preventDefault();
};
```

## Совместимость с браузерами

| Браузер | Dedicated Workers | Shared Workers | Service Workers |
|---------|-------------------|----------------|-----------------|
| Chrome | С 4.0 | С 4.0 | С 40.0 |
| Firefox | С 3.5 | С 29.0 | С 44.0 |
| Safari | С 4.0 | С 7.1 | С 11.1 |
| Edge | С 12.0 | С 12.0 | С 17.0 |
| Internet Explorer | С 10.0 | Нет | Нет |

## Ограничения и особенности

### Доступ к API в воркерах

> [!caution]
> В воркерах недоступны:
> - DOM
> - window, document, parent
> - alert, confirm
> - localStorage, sessionStorage
> - Множество других API, связанных с UI

```javascript
// worker.js - НЕЛЬЗЯ
// console.log(window.location); // Ошибка
// document.getElementById('element'); // Ошибка
// localStorage.setItem('key', 'value'); // Ошибка

// Доступные API в воркерах:
self.importScripts('helper.js'); // Импорт скриптов
fetch('https://api.example.com/data'); // Сетевые запросы
setTimeout(() => {}, 1000); // Таймеры
XMLHttpRequest; // Запросы
indexedDB; // Базы данных
```

### Безопасность

Web Workers подчиняются тем же правилам безопасности, что и основной скрипт:

- Ограничения Same-Origin Policy
- CORS для загрузки скриптов
- Ограничения на доступ к ресурсам

## Практические применения в российских реалиях 2025

1. **Финансовые приложения** - анализ рыночных данных в фоне
2. **Медицинские системы** - обработка изображений и анализ данных
3. **Научные вычисления** - статистический анализ и моделирование
4. **Обработка изображений/видео** - фильтры и преобразования
5. **Криптография** - шифрование и генерация ключей
6. **Игры** - ИИ, физика, генерация контента
7. **Аналитика** - обработка больших объемов данных

## Альтернативы и дополнения

- [[Service Workers]] - для кэширования и управления сетевыми запросами
- [[WebAssembly]] - для выполнения низкоуровневого кода с высокой производительностью
- [[Threads]] - экспериментальный API для настоящей многопоточности
- [[OffscreenCanvas]] - для рендеринга Canvas вне основного потока

## Рекомендации по использованию

1. **Используйте воркеры для тяжелых вычислений**, которые могут занять более 50мс
2. **Передавайте большие данные с использованием transferable objects** для повышения производительности
3. **Обрабатывайте ошибки** как в основном потоке, так и в воркерах
4. **Не создавайте слишком много воркеров** - это может привести к избыточному потреблению ресурсов
5. **Освобождайте ресурсы** при завершении работы с воркерами:

```javascript
// Завершение воркера
worker.terminate();

// В воркере можно использовать close()
// self.close(); // Закрытие воркера изнутри
```

## Заключение

Web Workers предоставляют мощный механизм для выполнения тяжелых вычислений в фоне, не блокируя пользовательский интерфейс. В 2025 году они особенно важны для создания отзывчивых веб-приложений, обрабатывающих большие объемы данных или выполняющих сложные вычисления.

Ключевые преимущества:
- Неблокирующее выполнение тяжелых операций
- Улучшенная производительность и пользовательский опыт
- Возможность обработки больших объемов данных
- Поддержка современными браузерами

При правильном использовании Web Workers могут значительно улучшить производительность веб-приложений, особенно в сценариях, требующих интенсивных вычислений или обработки данных.
