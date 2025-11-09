# Streams API

## Введение

Streams API предоставляет мощный интерфейс для работы с потоками данных. Он позволяет обрабатывать данные по мере их поступления, а не ждать полной загрузки. В этой главе мы рассмотрим все аспекты Streams API: ReadableStream, WritableStream, TransformStream, а также их применение для обработки больших объемов данных, загрузки файлов и других сценариев.

## Содержание

- [[ReadableStream]]
- [[WritableStream]]
- [[TransformStream]]
- [[Работа с HTTP-запросами]]
- [[Обработка файлов]]
- [[Кастомные потоки]]
- [[Производительность потоков]]
- [[Обработка ошибок]]

## ReadableStream

ReadableStream - это интерфейс для потоков данных, которые могут быть прочитаны. Он позволяет читать данные асинхронно по мере их доступности.

### Основы ReadableStream

```javascript
// Создание ReadableStream
class ReadableStreamExample {
    // Простой ReadableStream с генератором данных
    static createSimpleStream() {
        return new ReadableStream({
            start(controller) {
                // Начальная инициализация
                console.log('Stream started');
                
                // Добавление первых данных
                controller.enqueue('Hello');
                controller.enqueue(' ');
                controller.enqueue('World');
                
                // Завершение потока
                controller.close();
            }
        });
    }
    
    // ReadableStream с асинхронной генерацией данных
    static createAsyncStream() {
        let counter = 0;
        const maxCount = 5;
        
        return new ReadableStream({
            start(controller) {
                this.pushData(controller);
            },
            
            pull(controller) {
                setTimeout(() => {
                    if (counter < maxCount) {
                        controller.enqueue(`Data chunk ${counter + 1}`);
                        counter++;
                    } else {
                        controller.close();
                    }
                }, 1000);
            },
            
            cancel(reason) {
                console.log('Stream cancelled:', reason);
            }
        });
    }
    
    // ReadableStream из существующих данных
    static createFromData(data) {
        let index = 0;
        
        return new ReadableStream({
            pull(controller) {
                if (index < data.length) {
                    controller.enqueue(data[index]);
                    index++;
                } else {
                    controller.close();
                }
            },
            
            cancel() {
                console.log('Stream cancelled');
            }
        });
    }
    
    // Асинхронное чтение из потока
    static async readStream(stream) {
        const reader = stream.getReader();
        const chunks = [];
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) {
                    console.log('Stream finished');
                    break;
                }
                
                console.log('Received chunk:', value);
                chunks.push(value);
            }
        } finally {
            reader.releaseLock();
        }
        
        return chunks;
    }
    
    // Чтение с использованием for await
    static async readStreamWithForAwait(stream) {
        const reader = stream.getReader();
        const chunks = [];
        
        try {
            for await (const chunk of reader) {
                console.log('Chunk received:', chunk);
                chunks.push(chunk);
            }
        } finally {
            reader.releaseLock();
        }
        
        return chunks;
    }
}

// Использование
async function example1() {
    const stream = ReadableStreamExample.createSimpleStream();
    const chunks = await ReadableStreamExample.readStream(stream);
    console.log('All chunks:', chunks);
}

async function example2() {
    const asyncStream = ReadableStreamExample.createAsyncStream();
    const chunks = await ReadableStreamExample.readStreamWithForAwait(asyncStream);
    console.log('Async chunks:', chunks);
}
```

### Продвинутый ReadableStream

```javascript
// Класс для работы с продвинутыми ReadableStream
class AdvancedReadableStream {
    // ReadableStream с контролем скорости (throttling)
    static createThrottledStream(data, chunkSize = 1024, delay = 100) {
        let offset = 0;
        
        return new ReadableStream({
            async pull(controller) {
                if (offset < data.length) {
                    const end = Math.min(offset + chunkSize, data.length);
                    const chunk = data.slice(offset, end);
                    
                    controller.enqueue(chunk);
                    offset = end;
                    
                    // Задержка для контроля скорости
                    await new Promise(resolve => setTimeout(resolve, delay));
                } else {
                    controller.close();
                }
            },
            
            cancel() {
                console.log('Throttled stream cancelled');
            }
        });
    }
    
    // ReadableStream с буферизацией
    static createBufferedStream(data, bufferSize = 4) {
        const chunks = [];
        
        // Разделение данных на чанки
        for (let i = 0; i < data.length; i += bufferSize) {
            chunks.push(data.slice(i, i + bufferSize));
        }
        
        let index = 0;
        
        return new ReadableStream({
            pull(controller) {
                if (index < chunks.length) {
                    controller.enqueue(chunks[index]);
                    index++;
                } else {
                    controller.close();
                }
            }
        });
    }
    
    // ReadableStream с обработкой ошибок
    static createErrorHandlingStream(data, errorRate = 0.1) {
        let index = 0;
        
        return new ReadableStream({
            pull(controller) {
                // Симуляция ошибки
                if (Math.random() < errorRate) {
                    controller.error(new Error('Simulated error'));
                    return;
                }
                
                if (index < data.length) {
                    controller.enqueue(data[index]);
                    index++;
                } else {
                    controller.close();
                }
            }
        });
    }
    
    // Чтение с обработкой ошибок
    static async readWithErrorHandling(stream) {
        const reader = stream.getReader();
        const chunks = [];
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                chunks.push(value);
            }
        } catch (error) {
            console.error('Stream error:', error);
            throw error;
        } finally {
            reader.releaseLock();
        }
        
        return chunks;
    }
    
    // Чтение с предельным временем ожидания
    static async readWithTimeout(stream, timeout = 5000) {
        const reader = stream.getReader();
        const chunks = [];
        
        const timeoutPromise = new Promise((_, reject) => {
            setTimeout(() => reject(new Error('Stream read timeout')), timeout);
        });
        
        const readPromise = (async () => {
            try {
                while (true) {
                    const { done, value } = await reader.read();
                    
                    if (done) break;
                    
                    chunks.push(value);
                }
            } finally {
                reader.releaseLock();
            }
            
            return chunks;
        })();
        
        return Promise.race([readPromise, timeoutPromise]);
    }
}

// Использование продвинутых возможностей
async function advancedExample() {
    const data = new Uint8Array(100).map((_, i) => i);
    
    // Поток с контролем скорости
    const throttledStream = AdvancedReadableStream.createThrottledStream(data, 10, 200);
    console.log('Reading throttled stream...');
    await AdvancedReadableStream.readWithErrorHandling(throttledStream);
    
    // Поток с буферизацией
    const bufferedStream = AdvancedReadableStream.createBufferedStream(data, 5);
    console.log('Reading buffered stream...');
    await AdvancedReadableStream.readWithErrorHandling(bufferedStream);
}
```

## WritableStream

WritableStream - это интерфейс для потоков данных, в которые можно писать. Он позволяет асинхронно записывать данные в различные источники.

### Основы WritableStream

```javascript
// Класс для работы с WritableStream
class WritableStreamExample {
    // Простой WritableStream
    static createSimpleStream() {
        const chunks = [];
        
        return new WritableStream({
            write(chunk) {
                console.log('Writing chunk:', chunk);
                chunks.push(chunk);
            },
            
            close() {
                console.log('Stream closed');
                console.log('All chunks:', chunks);
            },
            
            abort(reason) {
                console.log('Stream aborted:', reason);
            }
        });
    }
    
    // WritableStream с буферизацией
    static createBufferedStream(bufferSize = 3) {
        const buffer = [];
        
        return new WritableStream({
            write(chunk, controller) {
                buffer.push(chunk);
                
                if (buffer.length >= bufferSize) {
                    this.processBuffer(buffer, controller);
                    buffer.length = 0; // Очистка буфера
                }
            },
            
            close() {
                // Обработка оставшихся данных в буфере
                if (buffer.length > 0) {
                    this.processBuffer(buffer);
                }
            },
            
            processBuffer(buffer, controller) {
                console.log('Processing buffer:', buffer);
                // Здесь может быть асинхронная обработка
            }
        });
    }
    
    // WritableStream с обработкой ошибок
    static createErrorHandlingStream() {
        return new WritableStream({
            async write(chunk) {
                // Симуляция асинхронной обработки
                await new Promise(resolve => setTimeout(resolve, 100));
                
                // Симуляция ошибки
                if (chunk === 'ERROR') {
                    throw new Error('Processing error');
                }
                
                console.log('Processed chunk:', chunk);
            },
            
            close() {
                console.log('Stream closed successfully');
            },
            
            abort(reason) {
                console.error('Stream aborted:', reason);
            }
        });
    }
    
    // Запись в поток
    static async writeToStream(stream, data) {
        const writer = stream.getWriter();
        
        try {
            for (const chunk of data) {
                await writer.write(chunk);
                console.log('Chunk written:', chunk);
            }
            
            await writer.close();
            console.log('Stream closed');
        } catch (error) {
            console.error('Write error:', error);
            await writer.abort(error);
        }
    }
    
    // Запись с контролем давления
    static async writeToStreamWithBackpressure(stream, data) {
        const writer = stream.getWriter();
        
        try {
            for (const chunk of data) {
                // Ожидание, если буфер переполнен
                if (writer.desiredSize <= 0) {
                    console.log('Backpressure detected, waiting...');
                    await writer.ready;
                }
                
                await writer.write(chunk);
                console.log('Chunk written, desired size:', writer.desiredSize);
            }
            
            await writer.close();
        } catch (error) {
            console.error('Write error:', error);
            await writer.abort(error);
        }
    }
}

// Использование WritableStream
async function writableExample() {
    // Простой поток
    const simpleStream = WritableStreamExample.createSimpleStream();
    await WritableStreamExample.writeToStream(simpleStream, ['Hello', ' ', 'World', '!']);
    
    // Поток с буферизацией
    const bufferedStream = WritableStreamExample.createBufferedStream(2);
    await WritableStreamExample.writeToStream(bufferedStream, ['A', 'B', 'C', 'D', 'E']);
    
    // Поток с обработкой ошибок
    const errorStream = WritableStreamExample.createErrorHandlingStream();
    await WritableStreamExample.writeToStream(errorStream, ['Data1', 'Data2', 'ERROR', 'Data3']);
}
```

### Продвинутый WritableStream

```javascript
// Класс для продвинутых WritableStream
class AdvancedWritableStream {
    // WritableStream для записи в файл (в браузере через File System Access API)
    static createFileWriterStream(fileHandle) {
        return new WritableStream({
            async start(controller) {
                this.writer = await fileHandle.createWritable();
            },
            
            async write(chunk, controller) {
                await this.writer.write(chunk);
            },
            
            async close() {
                await this.writer.close();
            },
            
            async abort(reason) {
                await this.writer.close();
            }
        });
    }
    
    // WritableStream для отправки данных на сервер
    static createUploadStream(url, options = {}) {
        let uploadBuffer = [];
        let uploadPromise = null;
        
        return new WritableStream({
            async write(chunk) {
                uploadBuffer.push(chunk);
                
                // Отправка данных пакетами
                if (uploadBuffer.length >= (options.batchSize || 10)) {
                    await this.sendBatch(uploadBuffer, url);
                    uploadBuffer = [];
                }
            },
            
            async close() {
                if (uploadBuffer.length > 0) {
                    await this.sendBatch(uploadBuffer, url);
                }
            },
            
            async abort(reason) {
                console.log('Upload aborted:', reason);
            },
            
            async sendBatch(batch, url) {
                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/octet-stream',
                            ...options.headers
                        },
                        body: new Blob(batch)
                    });
                    
                    if (!response.ok) {
                        throw new Error(`Upload failed: ${response.status}`);
                    }
                    
                    console.log(`Uploaded batch of ${batch.length} chunks`);
                } catch (error) {
                    console.error('Batch upload error:', error);
                    throw error;
                }
            }
        });
    }
    
    // WritableStream с логированием
    static createLoggingStream(logLevel = 'info') {
        return new WritableStream({
            write(chunk) {
                const timestamp = new Date().toISOString();
                const message = `[${timestamp}] ${logLevel.toUpperCase()}: ${chunk}`;
                
                switch (logLevel.toLowerCase()) {
                    case 'error':
                        console.error(message);
                        break;
                    case 'warn':
                        console.warn(message);
                        break;
                    case 'debug':
                        console.debug(message);
                        break;
                    default:
                        console.log(message);
                }
            }
        });
    }
    
    // WritableStream с кэшированием
    static createCachingStream(cacheKey, options = {}) {
        const cache = options.cache || new Map();
        
        return new WritableStream({
            write(chunk) {
                // Добавление в кэш
                if (!cache.has(cacheKey)) {
                    cache.set(cacheKey, []);
                }
                
                const chunks = cache.get(cacheKey);
                chunks.push(chunk);
                
                // Ограничение размера кэша
                if (options.maxSize && chunks.length > options.maxSize) {
                    chunks.shift(); // Удаление старейшего элемента
                }
            },
            
            close() {
                console.log(`Cache updated for key: ${cacheKey}`);
            }
        });
    }
    
    // WritableStream с трансформацией данных
    static createTransformingStream(transformFn) {
        return new WritableStream({
            async write(chunk) {
                const transformed = await Promise.resolve(transformFn(chunk));
                console.log('Transformed chunk:', transformed);
                // Здесь можно отправить в другой поток или сохранить
            }
        });
    }
}

// Использование продвинутых WritableStream
async function advancedWritableExample() {
    // Логирующий поток
    const logStream = AdvancedWritableStream.createLoggingStream('info');
    const logWriter = logStream.getWriter();
    
    await logWriter.write('Application started');
    await logWriter.write('Processing data...');
    await logWriter.close();
    
    // Поток с трансформацией
    const transformStream = AdvancedWritableStream.createTransformingStream(
        chunk => chunk.toUpperCase()
    );
    const transformWriter = transformStream.getWriter();
    
    await transformWriter.write('hello');
    await transformWriter.write('world');
    await transformWriter.close();
}
```

## TransformStream

TransformStream - это интерфейс, который объединяет ReadableStream и WritableStream для преобразования данных.

### Основы TransformStream

```javascript
// Класс для работы с TransformStream
class TransformStreamExample {
    // Простой TransformStream для преобразования регистра
    static createCaseTransformStream() {
        return new TransformStream({
            transform(chunk, controller) {
                // Преобразование в верхний регистр
                const transformed = chunk.toString().toUpperCase();
                controller.enqueue(transformed);
            }
        });
    }
    
    // TransformStream для фильтрации данных
    static createFilterStream(predicate) {
        return new TransformStream({
            transform(chunk, controller) {
                if (predicate(chunk)) {
                    controller.enqueue(chunk);
                }
                // Если предикат возвращает false, чанк отбрасывается
            }
        });
    }
    
    // TransformStream для агрегации данных
    static createAggregateStream(chunkSize = 3) {
        let buffer = [];
        
        return new TransformStream({
            transform(chunk, controller) {
                buffer.push(chunk);
                
                if (buffer.length >= chunkSize) {
                    const aggregated = buffer.join(' ');
                    controller.enqueue(aggregated);
                    buffer = [];
                }
            },
            
            flush(controller) {
                // Отправка оставшихся данных
                if (buffer.length > 0) {
                    const aggregated = buffer.join(' ');
                    controller.enqueue(aggregated);
                }
            }
        });
    }
    
    // TransformStream для кодирования/декодирования
    static createEncodingStream(encoding = 'utf-8') {
        return new TransformStream({
            transform(chunk, controller) {
                let encoded;
                
                if (encoding === 'base64') {
                    encoded = btoa(chunk);
                } else if (encoding === 'utf-8') {
                    encoded = encodeURIComponent(chunk);
                } else {
                    encoded = chunk;
                }
                
                controller.enqueue(encoded);
            }
        });
    }
    
    // Трансформация с асинхронной обработкой
    static createAsyncTransformStream(asyncProcessor) {
        return new TransformStream({
            async transform(chunk, controller) {
                try {
                    const result = await asyncProcessor(chunk);
                    controller.enqueue(result);
                } catch (error) {
                    controller.error(error);
                }
            }
        });
    }
    
    // Цепочка трансформаций
    static createPipeline(...transforms) {
        let stream = transforms[0];
        
        for (let i = 1; i < transforms.length; i++) {
            stream = stream.pipeThrough(transforms[i]);
        }
        
        return stream;
    }
    
    // Тестирование трансформаций
    static async testTransform(stream, input) {
        const writer = stream.writable.getWriter();
        const reader = stream.readable.getReader();
        
        // Запись данных
        for (const chunk of input) {
            await writer.write(chunk);
        }
        await writer.close();
        
        // Чтение результата
        const result = [];
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            result.push(value);
        }
        
        return result;
    }
}

// Использование TransformStream
async function transformExample() {
    // Преобразование регистра
    const caseStream = TransformStreamExample.createCaseTransformStream();
    const caseResult = await TransformStreamExample.testTransform(caseStream, ['hello', 'world']);
    console.log('Case transform result:', caseResult); // ['HELLO', 'WORLD']
    
    // Фильтрация
    const filterStream = TransformStreamExample.createFilterStream(chunk => chunk.length > 3);
    const filterResult = await TransformStreamExample.testTransform(filterStream, ['hi', 'hello', 'a', 'world']);
    console.log('Filter result:', filterResult); // ['hello', 'world']
    
    // Агрегация
    const aggregateStream = TransformStreamExample.createAggregateStream(2);
    const aggregateResult = await TransformStreamExample.testTransform(aggregateStream, ['a', 'b', 'c', 'd', 'e']);
    console.log('Aggregate result:', aggregateResult); // ['a b', 'c d', 'e']
    
    // Цепочка трансформаций
    const pipeline = TransformStreamExample.createPipeline(
        TransformStreamExample.createCaseTransformStream(),
        TransformStreamExample.createFilterStream(chunk => chunk.length > 3)
    );
    
    const pipelineResult = await TransformStreamExample.testTransform(pipeline, ['hello', 'hi', 'world']);
    console.log('Pipeline result:', pipelineResult); // ['HELLO', 'WORLD']
}
```

### Продвинутый TransformStream

```javascript
// Класс для продвинутых TransformStream
class AdvancedTransformStream {
    // TransformStream для сжатия данных
    static createCompressionStream() {
        return new TransformStream({
            transform(chunk, controller) {
                // Простая "компрессия" - удаление повторяющихся символов
                const compressed = chunk.toString()
                    .replace(/(.)\1+/g, '$1'); // Замена последовательностей на один символ
                controller.enqueue(compressed);
            }
        });
    }
    
    // TransformStream для шифрования
    static createEncryptionStream(key) {
        return new TransformStream({
            transform(chunk, controller) {
                // Простое шифрование (в реальном приложении используйте криптографические библиотеки)
                const encrypted = chunk.toString().split('')
                    .map(char => String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(0)))
                    .join('');
                controller.enqueue(encrypted);
            }
        });
    }
    
    // TransformStream для валидации данных
    static createValidationStream(schema) {
        return new TransformStream({
            transform(chunk, controller) {
                const isValid = this.validateChunk(chunk, schema);
                
                if (isValid) {
                    controller.enqueue(chunk);
                } else {
                    controller.error(new Error(`Validation failed for chunk: ${chunk}`));
                }
            },
            
            validateChunk(chunk, schema) {
                // Простая валидация (в реальном приложении используйте Joi, Yup и т.д.)
                if (schema.type === 'string' && typeof chunk === 'string') {
                    return schema.minLength ? chunk.length >= schema.minLength : true;
                }
                return false;
            }
        });
    }
    
    // TransformStream для логирования
    static createLoggingTransform(prefix = 'TRANSFORM') {
        return new TransformStream({
            transform(chunk, controller) {
                console.log(`[${prefix}] Processing:`, chunk);
                controller.enqueue(chunk);
            }
        });
    }
    
    // TransformStream для буферизации с задержкой
    static createBufferedDelayStream(delay = 1000, bufferSize = 5) {
        let buffer = [];
        let timeoutId = null;
        
        return new TransformStream({
            transform(chunk, controller) {
                buffer.push(chunk);
                
                // Если буфер полон, отправляем немедленно
                if (buffer.length >= bufferSize) {
                    this.flushBuffer(controller);
                } else if (!timeoutId) {
                    // Иначе ждем задержку
                    timeoutId = setTimeout(() => {
                        this.flushBuffer(controller);
                        timeoutId = null;
                    }, delay);
                }
            },
            
            flush(controller) {
                if (timeoutId) {
                    clearTimeout(timeoutId);
                    this.flushBuffer(controller);
                }
            },
            
            flushBuffer(controller) {
                if (buffer.length > 0) {
                    const batch = buffer.join(' | ');
                    controller.enqueue(batch);
                    buffer = [];
                }
            }
        });
    }
    
    // TransformStream для нормализации данных
    static createNormalizationStream() {
        return new TransformStream({
            transform(chunk, controller) {
                // Нормализация строки: удаление лишних пробелов, приведение к нижнему регистру
                const normalized = chunk.toString()
                    .trim()
                    .toLowerCase()
                    .replace(/\s+/g, ' ');
                controller.enqueue(normalized);
            }
        });
    }
    
    // Комбинированный TransformStream
    static createMultiTransformStream(transforms) {
        return new TransformStream({
            transform(chunk, controller) {
                let result = chunk;
                
                for (const transform of transforms) {
                    result = transform(result);
                }
                
                controller.enqueue(result);
            }
        });
    }
}

// Использование продвинутых TransformStream
async function advancedTransformExample() {
    // Буферизованный поток с задержкой
    const delayedStream = AdvancedTransformStream.createBufferedDelayStream(500, 3);
    const writer = delayedStream.writable.getWriter();
    const reader = delayedStream.readable.getReader();
    
    // Запись данных
    writer.write('Chunk 1');
    writer.write('Chunk 2');
    writer.write('Chunk 3'); // Вызовет немедленную отправку
    writer.write('Chunk 4');
    
    setTimeout(async () => {
        writer.write('Chunk 5');
        await writer.close();
    }, 1000);
    
    // Чтение результатов
    setTimeout(async () => {
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            console.log('Delayed result:', value);
        }
    }, 1500);
    
    // Нормализующий поток
    const normalizeStream = AdvancedTransformStream.createNormalizationStream();
    const normalizeResult = await TransformStreamExample.testTransform(normalizeStream, ['  HELLO  ', '  WORLD  ']);
    console.log('Normalize result:', normalizeResult); // ['hello', 'world']
}
```

## Примеры из промышленной разработки

### Обработка файлов с использованием Streams

```javascript
// Класс для обработки файлов с использованием Streams
class FileStreamProcessor {
    constructor() {
        this.processedChunks = 0;
        this.totalSize = 0;
    }
    
    // Чтение файла с использованием Streams
    async processFileWithStreams(file) {
        // Создание readable stream из файла
        const fileStream = file.stream();
        
        // Создание цепочки обработки
        const processingStream = new TransformStream({
            async transform(chunk, controller) {
                // Обработка чанка данных
                const processedChunk = await this.processChunk(chunk);
                controller.enqueue(processedChunk);
                
                this.processedChunks++;
                this.totalSize += chunk.length;
                
                // Отчет о прогрессе
                this.reportProgress();
            },
            
            flush() {
                console.log(`Обработка завершена. Всего обработано чанков: ${this.processedChunks}`);
            }
        });
        
        // Создание writable stream для результата
        const writeStream = new WritableStream({
            write(chunk) {
                console.log(`Записан чанк размером: ${chunk.length} байт`);
                // Здесь можно сохранить в другой файл или отправить на сервер
            }
        });
        
        // Подключение всех потоков
        return fileStream
            .pipeThrough(processingStream)
            .pipeTo(writeStream);
    }
    
    async processChunk(chunk) {
        // Симуляция обработки данных
        await new Promise(resolve => setTimeout(resolve, 10));
        
        // В реальном приложении здесь может быть:
        // - валидация данных
        // - преобразование формата
        // - сжатие
        // - шифрование
        // - валидация структуры
        
        return chunk; // Возвращаем обработанный чанк
    }
    
    reportProgress() {
        const progress = {
            chunksProcessed: this.processedChunks,
            totalSize: this.totalSize,
            averageChunkSize: this.totalSize / this.processedChunks
        };
        
        console.log('Прогресс обработки:', progress);
    }
    
    // Чтение CSV файла с использованием Streams
    async processCsvStream(file) {
        const fileStream = file.stream();
        const lines = [];
        
        const csvProcessor = new TransformStream({
            start() {
                this.currentLine = '';
            },
            
            transform(chunk, controller) {
                const text = new TextDecoder().decode(chunk);
                const parts = (this.currentLine + text).split('\n');
                
                // Последняя часть может быть неполной строкой
                this.currentLine = parts.pop() || '';
                
                for (const line of parts) {
                    if (line.trim()) {
                        controller.enqueue(this.parseCsvLine(line));
                    }
                }
            },
            
            flush(controller) {
                if (this.currentLine.trim()) {
                    controller.enqueue(this.parseCsvLine(this.currentLine));
                }
            },
            
            parseCsvLine(line) {
                // Простой парсер CSV (в реальном приложении используйте библиотеку)
                return line.split(',').map(field => field.trim());
            }
        });
        
        const readable = fileStream.pipeThrough(csvProcessor);
        const reader = readable.getReader();
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                lines.push(value);
                console.log('Обработана строка CSV:', value);
            }
        } finally {
            reader.releaseLock();
        }
        
        return lines;
    }
    
    // Обработка JSON Lines (JSONL) файла
    async processJsonlStream(file) {
        const fileStream = file.stream();
        const records = [];
        
        const jsonlProcessor = new TransformStream({
            start() {
                this.currentLine = '';
            },
            
            transform(chunk, controller) {
                const text = new TextDecoder().decode(chunk);
                const parts = (this.currentLine + text).split('\n');
                
                this.currentLine = parts.pop() || '';
                
                for (const line of parts) {
                    if (line.trim()) {
                        try {
                            const json = JSON.parse(line);
                            controller.enqueue(json);
                        } catch (error) {
                            console.error('Ошибка парсинга JSON:', error);
                            // В зависимости от требований, можно пропустить или выбросить ошибку
                        }
                    }
                }
            },
            
            flush(controller) {
                if (this.currentLine.trim()) {
                    try {
                        const json = JSON.parse(this.currentLine);
                        controller.enqueue(json);
                    } catch (error) {
                        console.error('Ошибка парсинга последней строки JSON:', error);
                    }
                }
            }
        });
        
        const readable = fileStream.pipeThrough(jsonlProcessor);
        const reader = readable.getReader();
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                records.push(value);
                console.log('Обработан JSONL запись:', value);
            }
        } finally {
            reader.releaseLock();
        }
        
        return records;
    }
}

// Использование FileProcessor
async function fileProcessingExample() {
    // Предположим, у нас есть файл из File API
    // const fileInput = document.getElementById('fileInput');
    // const file = fileInput.files[0];
    
    // const processor = new FileStreamProcessor();
    
    // Обработка обычного файла
    // await processor.processFileWithStreams(file);
    
    // Обработка CSV файла
    // const csvData = await processor.processCsvStream(file);
    // console.log('CSV данные:', csvData);
    
    // Обработка JSONL файла
    // const jsonlData = await processor.processJsonlStream(file);
    // console.log('JSONL данные:', jsonlData);
}
```

### HTTP-запросы с использованием Streams

```javascript
// Класс для работы с HTTP-запросами через Streams
class HttpStreamClient {
    constructor() {
        this.requestQueue = [];
        this.isProcessing = false;
    }
    
    // Потоковая загрузка данных
    async streamDownload(url, options = {}) {
        const response = await fetch(url, options);
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const reader = response.body.getReader();
        const stream = new ReadableStream({
            async start(controller) {
                await this.readStream(reader, controller);
            },
            
            cancel() {
                reader.cancel();
            }
        });
        
        return stream;
    }
    
    async readStream(reader, controller) {
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) {
                    controller.close();
                    break;
                }
                
                controller.enqueue(value);
                
                // Опционально: обработка прогресса
                if (typeof this.onProgress === 'function') {
                    this.onProgress(value.length);
                }
            }
        } catch (error) {
            controller.error(error);
        } finally {
            await reader.releaseLock();
        }
    }
    
    // Потоковая отправка данных
    async streamUpload(url, readableStream, options = {}) {
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/octet-stream',
                ...options.headers
            },
            body: readableStream
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return response;
    }
    
    // Загрузка файла с отчетом о прогрессе
    async downloadWithProgress(url, onProgress) {
        const response = await fetch(url);
        const contentLength = response.headers.get('Content-Length');
        const total = parseInt(contentLength, 10);
        let loaded = 0;
        
        const reader = response.body.getReader();
        const chunks = [];
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                chunks.push(value);
                loaded += value.length;
                
                if (onProgress && total) {
                    const progress = Math.round((loaded / total) * 100);
                    onProgress(progress, loaded, total);
                }
            }
        } finally {
            await reader.releaseLock();
        }
        
        // Сборка полного результата
        const fullBuffer = new Uint8Array(loaded);
        let offset = 0;
        
        for (const chunk of chunks) {
            fullBuffer.set(chunk, offset);
            offset += chunk.length;
        }
        
        return fullBuffer;
    }
    
    // Потоковая обработка ответа
    async processResponseStream(response) {
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let buffer = '';
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                buffer += decoder.decode(value, { stream: true });
                
                // Обработка полных строк
                const lines = buffer.split('\n');
                buffer = lines.pop() || ''; // Последняя строка может быть неполной
                
                for (const line of lines) {
                    if (line.trim()) {
                        await this.processLine(line);
                    }
                }
            }
            
            // Обработка оставшейся строки
            if (buffer.trim()) {
                await this.processLine(buffer);
            }
        } finally {
            await reader.releaseLock();
        }
    }
    
    async processLine(line) {
        // Обработка отдельной строки из потока
        console.log('Обработана строка:', line);
        
        // Здесь может быть:
        // - парсинг JSON
        // - валидация данных
        // - сохранение в БД
        // - отправка в другую систему
    }
    
    // Server-Sent Events через Streams
    async connectSSE(url, onMessage) {
        const response = await fetch(url);
        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let buffer = '';
        
        try {
            while (true) {
                const { done, value } = await reader.read();
                
                if (done) break;
                
                buffer += decoder.decode(value, { stream: true });
                
                const lines = buffer.split('\n');
                buffer = lines.pop() || '';
                
                for (const line of lines) {
                    if (line.startsWith('data: ')) {
                        const data = line.slice(6).trim();
                        if (data) {
                            const parsed = JSON.parse(data);
                            onMessage(parsed);
                        }
                    }
                }
            }
        } finally {
            await reader.releaseLock();
        }
    }
}

// Использование HTTP клиента
async function httpClientExample() {
    const client = new HttpStreamClient();
    
    // Потоковая загрузка
    const downloadStream = await client.streamDownload('https://api.example.com/large-file');
    
    // Обработка потока
    const reader = downloadStream.getReader();
    let totalBytes = 0;
    
    try {
        while (true) {
            const { done, value } = await reader.read();
            
            if (done) break;
            
            totalBytes += value.length;
            console.log(`Получено ${value.length} байт, всего: ${totalBytes}`);
            
            // Здесь можно обрабатывать данные по мере получения
        }
    } finally {
        reader.releaseLock();
    }
    
    // Загрузка с прогрессом
    await client.downloadWithProgress(
        'https://api.example.com/download',
        (progress, loaded, total) => {
            console.log(`Прогресс: ${progress}% (${loaded}/${total} байт)`);
        }
    );
    
    // Подключение к SSE
    await client.connectSSE('https://api.example.com/events', (data) => {
        console.log('Получено сообщение:', data);
    });
}
```

### Кастомные потоки для специфических задач

```javascript
// Классы для кастомных потоков
class CustomStreams {
    // Поток для обработки логов
    static createLogProcessingStream() {
        return new TransformStream({
            transform(chunk, controller) {
                const logEntry = {
                    timestamp: new Date().toISOString(),
                    level: 'INFO',
                    message: chunk.toString(),
                    processed: true
                };
                
                controller.enqueue(JSON.stringify(logEntry) + '\n');
            }
        });
    }
    
    // Поток для агрегации метрик
    static createMetricsAggregationStream(windowSize = 10) {
        let buffer = [];
        let sum = 0;
        
        return new TransformStream({
            transform(chunk, controller) {
                const value = parseFloat(chunk);
                if (!isNaN(value)) {
                    buffer.push(value);
                    sum += value;
                    
                    if (buffer.length >= windowSize) {
                        const average = sum / buffer.length;
                        const metrics = {
                            average,
                            count: buffer.length,
                            windowSize,
                            timestamp: new Date().toISOString()
                        };
                        
                        controller.enqueue(JSON.stringify(metrics) + '\n');
                        buffer = [];
                        sum = 0;
                    }
                }
            },
            
            flush(controller) {
                if (buffer.length > 0) {
                    const average = sum / buffer.length;
                    const metrics = {
                        average,
                        count: buffer.length,
                        windowSize: buffer.length,
                        timestamp: new Date().toISOString()
                    };
                    
                    controller.enqueue(JSON.stringify(metrics) + '\n');
                }
            }
        });
    }
    
    // Поток для фильтрации чувствительных данных
    static createDataSanitizationStream() {
        const sensitivePatterns = [
            /\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b/g, // кредитные карты
            /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g, // email
            /\b\d{3}-\d{2}-\d{4}\b/g // SSN
        ];
        
        return new TransformStream({
            transform(chunk, controller) {
                let processed = chunk.toString();
                
                for (const pattern of sensitivePatterns) {
                    processed = processed.replace(pattern, '[REDACTED]');
                }
                
                controller.enqueue(processed);
            }
        });
    }
    
    // Поток для компрессии данных
    static createCompressionStream() {
        // В реальном приложении использовалась бы библиотека типа pako
        // или встроенные API сжатия
        
        return new TransformStream({
            transform(chunk, controller) {
                // Простая "компрессия" для демонстрации
                const original = chunk.toString();
                const compressed = this.simpleCompress(original);
                controller.enqueue(compressed);
            },
            
            simpleCompress(text) {
                // Простая замена повторяющихся символов
                return text.replace(/(.)\1{2,}/g, '$1$1'); // замена 3+ на 2
            }
        });
    }
    
    // Поток для шифрования данных
    static createEncryptionStream(password) {
        const encoder = new TextEncoder();
        const decoder = new TextDecoder();
        
        return new TransformStream({
            transform(chunk, controller) {
                const data = chunk.toString();
                const encrypted = this.encrypt(data, password);
                controller.enqueue(encrypted);
            },
            
            encrypt(text, password) {
                const data = encoder.encode(text);
                const key = encoder.encode(password);
                
                // Простое XOR шифрование (НЕ использовать в продакшене!)
                const encrypted = new Uint8Array(data.length);
                for (let i = 0; i < data.length; i++) {
                    encrypted[i] = data[i] ^ key[i % key.length];
                }
                
                // Кодирование в base64 для удобства
                return btoa(String.fromCharCode(...encrypted));
            }
        });
    }
}

// Использование кастомных потоков
async function customStreamsExample() {
    // Обработка логов
    const logStream = CustomStreams.createLogProcessingStream();
    const logWriter = logStream.writable.getWriter();
    
    await logWriter.write('Application started');
    await logWriter.write('User logged in');
    await logWriter.write('Data processed');
    await logWriter.close();
    
    // Агрегация метрик
    const metricsStream = CustomStreams.createMetricsAggregationStream(3);
    const metricsWriter = metricsStream.writable.getWriter();
    
    for (let i = 1; i <= 10; i++) {
        await metricsWriter.write(i.toString());
    }
    await metricsWriter.close();
    
    // Санитизация данных
    const sanitizeStream = CustomStreams.createDataSanitizationStream();
    const sanitizeWriter = sanitizeStream.writable.getWriter();
    const sanitizeReader = sanitizeStream.readable.getReader();
    
    await sanitizeWriter.write('Email: user@example.com and card: 1234-5678-9012-3456');
    await sanitizeWriter.close();
    
    const { value: sanitized } = await sanitizeReader.read();
    console.log('Sanitized:', sanitized);
}
```

## Лучшие практики

### 1. Обработка ошибок в потоках

```javascript
// Лучшие практики для обработки ошибок в потоках
class StreamErrorHandling {
    // Создание потока с надежной обработкой ошибок
    static createRobustStream(errorHandler) {
        return new TransformStream({
            async transform(chunk, controller) {
                try {
                    // Основная логика обработки
                    const result = await this.processChunk(chunk);
                    controller.enqueue(result);
                } catch (error) {
                    if (errorHandler) {
                        const handled = await errorHandler(error, chunk);
                        if (handled) {
                            controller.enqueue(handled);
                        } else {
                            controller.error(error);
                        }
                    } else {
                        controller.error(error);
                    }
                }
            }
        });
    }
    
    static async processChunk(chunk) {
        // Симуляция асинхронной обработки
        await new Promise(resolve => setTimeout(resolve, 10));
        
        if (chunk === 'ERROR') {
            throw new Error('Simulated processing error');
        }
        
        return chunk.toString().toUpperCase();
    }
    
    // Обертка для безопасного чтения потока
    static async safeReadStream(stream, options = {}) {
        const { 
            maxRetries = 3, 
            retryDelay = 1000, 
            onRetry,
            onError
        } = options;
        
        let attempts = 0;
        
        while (attempts <= maxRetries) {
            try {
                const reader = stream.getReader();
                const chunks = [];
                
                try {
                    while (true) {
                        const { done, value } = await reader.read();
                        
                        if (done) break;
                        
                        chunks.push(value);
                    }
                } finally {
                    reader.releaseLock();
                }
                
                return chunks;
            } catch (error) {
                attempts++;
                
                if (attempts > maxRetries) {
                    if (onError) onError(error);
                    throw error;
                }
                
                if (onRetry) onRetry(error, attempts);
                
                // Ожидание перед повторной попыткой
                await new Promise(resolve => setTimeout(resolve, retryDelay * attempts));
                
                // Пересоздание потока если возможно
                if (typeof stream.start === 'function') {
                    stream = new ReadableStream(stream.start);
                }
            }
        }
    }
}
```

## Безопасность

При работе с Streams API важно учитывать безопасность:

```javascript
// Безопасность Streams API
class SecureStreamProcessing {
    // Безопасная обработка пользовательского ввода
    static createSecureInputStream(allowedTypes = ['string', 'number']) {
        return new TransformStream({
            transform(chunk, controller) {
                // Проверка типа данных
                const type = typeof chunk;
                if (!allowedTypes.includes(type)) {
                    throw new Error(`Недопустимый тип данных: ${type}`);
                }
                
                // Санитизация строк
                if (type === 'string') {
                    const sanitized = this.sanitizeString(chunk);
                    controller.enqueue(sanitized);
                } else {
                    controller.enqueue(chunk);
                }
            },
            
            sanitizeString(str) {
                // Удаление потенциально опасных символов
                return str
                    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
                    .replace(/javascript:/gi, '')
                    .replace(/vbscript:/gi, '')
                    .replace(/on\w+\s*=/gi, '');
            }
        });
    }
    
    // Ограничение размера данных
    static createSizeLimitedStream(maxSize = 10 * 1024 * 1024) { // 10MB
        let totalSize = 0;
        
        return new TransformStream({
            transform(chunk, controller) {
                const chunkSize = typeof chunk === 'string' ? 
                    new Blob([chunk]).size : 
                    chunk.byteLength || chunk.length || 0;
                
                totalSize += chunkSize;
                
                if (totalSize > maxSize) {
                    controller.error(new Error(`Превышен максимальный размер: ${maxSize} байт`));
                    return;
                }
                
                controller.enqueue(chunk);
            }
        });
    }
    
    // Проверка MIME-типов для файлов
    static createMimeTypeValidatorStream(allowedTypes) {
        return new TransformStream({
            transform(chunk, controller) {
                // В реальном приложении проверка MIME-типа файла
                // происходит при его загрузке, а не в потоке
                controller.enqueue(chunk);
            }
        });
    }
}

// Использование безопасных потоков
async function secureStreamExample() {
    const secureStream = SecureStreamProcessing.createSecureInputStream(['string']);
    const writer = secureStream.writable.getWriter();
    const reader = secureStream.readable.getReader();
    
    try {
        await writer.write('Safe string');
        await writer.write('<script>alert("xss")</script>');
        await writer.close();
        
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            console.log('Safe chunk:', value);
        }
    } catch (error) {
        console.error('Stream processing error:', error);
    }
}
```

## Теги

#javascript #streams #readablestream #writablestream #transformstream #web-api #async #data-processing