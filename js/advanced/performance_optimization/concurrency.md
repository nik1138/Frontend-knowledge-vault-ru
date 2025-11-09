# Оптимизация параллелизма в JavaScript

## Введение

JavaScript - однопоточный язык, но с помощью асинхронных операций и Web Workers можно эффективно использовать параллелизм для улучшения производительности приложений. Правильная оптимизация параллелизма позволяет выполнять тяжелые вычисления, обрабатывать множество асинхронных операций и создавать отзывчивые интерфейсы. В этом разделе мы рассмотрим различные техники оптимизации параллелизма в JavaScript.

## Управление асинхронными операциями

Оптимизация выполнения множественных асинхронных операций:

```javascript
// Управление асинхронными операциями
class AsyncConcurrencyOptimization {
  // Ограниченный параллелизм
  static async limitedParallelExecution(tasks, limit) {
    const results = [];
    const executing = [];
    
    for (let i = 0; i < tasks.length; i++) {
      const task = tasks[i];
      const promise = task().then(result => {
        results[i] = result;
      });
      
      executing.push(promise);
      
      if (executing.length >= limit) {
        await Promise.race(executing);
        // Удаляем завершенные промисы
        const index = executing.findIndex(p => p === promise);
        if (index !== -1) {
          executing.splice(index, 1);
        }
      }
    }
    
    await Promise.all(executing);
    return results;
  }
  
  // Приоритезированный пул задач
  static createPriorityTaskPool() {
    const tasks = {
      high: [],
      normal: [],
      low: []
    };
    
    let isProcessing = false;
    
    return {
      add(task, priority = 'normal') {
        tasks[priority].push(task);
        if (!isProcessing) {
          this.process();
        }
      },
      
      async process() {
        if (isProcessing) return;
        
        isProcessing = true;
        
        try {
          // Обрабатываем задачи в порядке приоритета
          while (tasks.high.length > 0 || tasks.normal.length > 0 || tasks.low.length > 0) {
            let task;
            
            // Сначала высокий приоритет
            if (tasks.high.length > 0) {
              task = tasks.high.shift();
            } else if (tasks.normal.length > 0) {
              task = tasks.normal.shift();
            } else {
              task = tasks.low.shift();
            }
            
            if (task) {
              try {
                await task();
              } catch (error) {
                console.error('Ошибка выполнения задачи:', error);
              }
            }
            
            // Даем возможность выполнить другие задачи
            await new Promise(resolve => setTimeout(resolve, 0));
          }
        } finally {
          isProcessing = false;
        }
      }
    };
  }
  
  // Таймаут для асинхронных операций
  static async withTimeout(promise, timeout) {
    let timeoutId;
    
    const timeoutPromise = new Promise((_, reject) => {
      timeoutId = setTimeout(() => {
        reject(new Error('Операция превысила таймаут'));
      }, timeout);
    });
    
    try {
      const result = await Promise.race([promise, timeoutPromise]);
      clearTimeout(timeoutId);
      return result;
    } catch (error) {
      clearTimeout(timeoutId);
      throw error;
    }
  }
  
  // Retry механизм с экспоненциальной задержкой
  static async withRetry(operation, maxRetries = 3, baseDelay = 1000) {
    for (let i = 0; i <= maxRetries; i++) {
      try {
        return await operation();
      } catch (error) {
        if (i === maxRetries) {
          throw error;
        }
        
        const delay = baseDelay * Math.pow(2, i);
        console.log(`Повторная попытка через ${delay} мс...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  // Асинхронная очередь задач
  static createAsyncQueue(concurrency = 1) {
    const queue = [];
    let activeCount = 0;
    
    return {
      async add(asyncFunction) {
        return new Promise((resolve, reject) => {
          queue.push({
            asyncFunction,
            resolve,
            reject
          });
          
          this.process();
        });
      },
      
      async process() {
        if (activeCount >= concurrency || queue.length === 0) {
          return;
        }
        
        activeCount++;
        const { asyncFunction, resolve, reject } = queue.shift();
        
        try {
          const result = await asyncFunction();
          resolve(result);
        } catch (error) {
          reject(error);
        } finally {
          activeCount--;
          this.process();
        }
      },
      
      get pending() {
        return queue.length;
      },
      
      get running() {
        return activeCount;
      }
    };
  }
}

// Пример использования управления асинхронными операциями
class APIClient {
  constructor() {
    this.taskPool = AsyncConcurrencyOptimization.createPriorityTaskPool();
    this.requestQueue = AsyncConcurrencyOptimization.createAsyncQueue(6); // 6 одновременных запросов
  }
  
  // Приоритезированный API запрос
  async apiRequest(url, options = {}, priority = 'normal') {
    return new Promise((resolve, reject) => {
      this.taskPool.add(async () => {
        try {
          const response = await this.requestQueue.add(() => 
            fetch(url, options)
          );
          const data = await response.json();
          resolve(data);
        } catch (error) {
          reject(error);
        }
      }, priority);
    });
  }
  
  // Запрос с таймаутом и retry
  async reliableRequest(url, options = {}) {
    return await AsyncConcurrencyOptimization.withRetry(
      () => AsyncConcurrencyOptimization.withTimeout(
        fetch(url, options),
        5000 // 5 секунд таймаут
      ),
      3, // 3 попытки
      1000 // 1 секунда базовая задержка
    );
  }
  
  // Параллельные запросы с ограничением
  async parallelRequests(urls, limit = 3) {
    const tasks = urls.map(url => () => this.apiRequest(url));
    return await AsyncConcurrencyOptimization.limitedParallelExecution(tasks, limit);
  }
}
```

## Web Workers для тяжелых вычислений

Использование Web Workers для параллельных вычислений:

```javascript
// Web Workers для параллелизма
class WebWorkerOptimization {
  // Создание Web Worker из строки
  static createWorkerFromString(workerScript) {
    const blob = new Blob([workerScript], { type: 'application/javascript' });
    return new Worker(URL.createObjectURL(blob));
  }
  
  // Пул Web Workers
  static createWorkerPool(workerScript, poolSize = 4) {
    const workers = [];
    const taskQueue = [];
    let workerIndex = 0;
    
    // Создаем пул воркеров
    for (let i = 0; i < poolSize; i++) {
      const worker = this.createWorkerFromString(workerScript);
      worker.id = i;
      worker.busy = false;
      workers.push(worker);
    }
    
    return {
      async execute(data) {
        return new Promise((resolve, reject) => {
          const task = { data, resolve, reject };
          
          // Ищем свободного воркера
          const freeWorker = workers.find(worker => !worker.busy);
          
          if (freeWorker) {
            this.runTaskOnWorker(freeWorker, task);
          } else {
            // Добавляем в очередь
            taskQueue.push(task);
          }
        });
      },
      
      runTaskOnWorker(worker, task) {
        worker.busy = true;
        
        worker.onmessage = (event) => {
          worker.busy = false;
          task.resolve(event.data);
          this.processQueue();
        };
        
        worker.onerror = (error) => {
          worker.busy = false;
          task.reject(error);
          this.processQueue();
        };
        
        worker.postMessage(task.data);
      },
      
      processQueue() {
        if (taskQueue.length === 0) return;
        
        const freeWorker = workers.find(worker => !worker.busy);
        if (freeWorker) {
          const task = taskQueue.shift();
          this.runTaskOnWorker(freeWorker, task);
        }
      },
      
      terminate() {
        workers.forEach(worker => worker.terminate());
      },
      
      getStats() {
        return {
          totalWorkers: workers.length,
          busyWorkers: workers.filter(w => w.busy).length,
          queuedTasks: taskQueue.length
        };
      }
    };
  }
  
  // Распределенные вычисления
  static createDistributedComputing(workerScript) {
    const workerPool = this.createWorkerPool(workerScript, 4);
    
    return {
      async processLargeDataset(data, chunkSize = 1000) {
        // Разбиваем данные на чанки
        const chunks = [];
        for (let i = 0; i < data.length; i += chunkSize) {
          chunks.push(data.slice(i, i + chunkSize));
        }
        
        // Обрабатываем чанки параллельно
        const results = await Promise.all(
          chunks.map(chunk => workerPool.execute({ type: 'process', data: chunk }))
        );
        
        // Объединяем результаты
        return this.mergeResults(results);
      },
      
      mergeResults(results) {
        // Пример объединения результатов
        return results.reduce((acc, result) => {
          return {
            ...acc,
            ...result,
            total: (acc.total || 0) + (result.total || 0)
          };
        }, {});
      },
      
      getStats() {
        return workerPool.getStats();
      },
      
      terminate() {
        workerPool.terminate();
      }
    };
  }
  
  // Пример worker скрипта для тяжелых вычислений
  static getHeavyComputationWorkerScript() {
    return `
      self.onmessage = function(e) {
        const { type, data } = e.data;
        
        switch (type) {
          case 'process':
            const result = processChunk(data);
            self.postMessage(result);
            break;
          case 'sort':
            const sorted = data.sort((a, b) => a - b);
            self.postMessage(sorted);
            break;
          case 'calculate':
            const sum = data.reduce((acc, val) => acc + Math.sqrt(val), 0);
            self.postMessage({ sum, count: data.length });
            break;
          default:
            self.postMessage({ error: 'Неизвестный тип задачи' });
        }
      };
      
      function processChunk(chunk) {
        // Пример тяжелых вычислений
        const result = {
          processed: chunk.map(item => ({
            ...item,
            processed: true,
            timestamp: Date.now()
          })),
          total: chunk.length
        };
        
        // Имитация тяжелых вычислений
        let dummy = 0;
        for (let i = 0; i < 1000000; i++) {
          dummy += Math.sin(i) * Math.cos(i);
        }
        
        return result;
      }
    `;
  }
}

// Пример использования Web Workers
class DataProcessor {
  constructor() {
    this.computingPool = WebWorkerOptimization.createDistributedComputing(
      WebWorkerOptimization.getHeavyComputationWorkerScript()
    );
  }
  
  // Обработка больших наборов данных
  async processLargeDataset(data) {
    console.time('Обработка данных');
    
    try {
      const result = await this.computingPool.processLargeDataset(data, 500);
      console.timeEnd('Обработка данных');
      return result;
    } catch (error) {
      console.timeEnd('Обработка данных');
      throw error;
    }
  }
  
  // Параллельная сортировка
  async parallelSort(data) {
    const workerScript = `
      self.onmessage = function(e) {
        const sorted = e.data.sort((a, b) => a - b);
        self.postMessage(sorted);
      };
    `;
    
    const workerPool = WebWorkerOptimization.createWorkerPool(workerScript, 2);
    
    try {
      return await workerPool.execute(data);
    } finally {
      workerPool.terminate();
    }
  }
  
  // Получение статистики
  getStats() {
    return this.computingPool.getStats();
  }
}
```

## SharedArrayBuffer и Atomics

Синхронизация между потоками:

```javascript
// SharedArrayBuffer и Atomics для синхронизации
class SharedMemoryOptimization {
  // Создание shared memory буфера
  static createSharedBuffer(size) {
    if (typeof SharedArrayBuffer === 'undefined') {
      throw new Error('SharedArrayBuffer не поддерживается');
    }
    
    const buffer = new SharedArrayBuffer(size);
    const int32Array = new Int32Array(buffer);
    
    return {
      buffer,
      int32Array,
      // Используем первые элементы для синхронизации
      lockIndex: 0,
      dataStartIndex: 1
    };
  }
  
  // Простой mutex с использованием Atomics
  static createMutex(sharedArray, index) {
    return {
      async lock() {
        // Ждем, пока мьютекс не освободится
        while (Atomics.compareExchange(sharedArray, index, 0, 1) !== 0) {
          // Ждем уведомления
          Atomics.wait(sharedArray, index, 1);
        }
      },
      
      unlock() {
        // Освобождаем мьютекс
        Atomics.store(sharedArray, index, 0);
        // Уведомляем ожидающие потоки
        Atomics.notify(sharedArray, index, 1);
      }
    };
  }
  
  // Producer-Consumer паттерн
  static createProducerConsumer(bufferSize) {
    const shared = this.createSharedBuffer((bufferSize + 3) * 4); // +3 для метаданных
    const { int32Array } = shared;
    
    // Индексы для метаданных
    const headIndex = 0; // Индекс для головы очереди
    const tailIndex = 1; // Индекс для хвоста очереди
    const countIndex = 2; // Индекс для счетчика элементов
    
    return {
      producer: {
        async produce(data) {
          // Ждем, пока есть место в буфере
          while (Atomics.load(int32Array, countIndex) >= bufferSize) {
            Atomics.wait(int32Array, countIndex, bufferSize);
          }
          
          // Добавляем данные
          const tail = Atomics.load(int32Array, tailIndex);
          int32Array[shared.dataStartIndex + tail] = data;
          
          // Обновляем хвост и счетчик
          Atomics.store(int32Array, tailIndex, (tail + 1) % bufferSize);
          Atomics.add(int32Array, countIndex, 1);
          
          // Уведомляем потребителя
          Atomics.notify(int32Array, countIndex, 1);
        }
      },
      
      consumer: {
        async consume() {
          // Ждем, пока есть данные
          while (Atomics.load(int32Array, countIndex) <= 0) {
            Atomics.wait(int32Array, countIndex, 0);
          }
          
          // Получаем данные
          const head = Atomics.load(int32Array, headIndex);
          const data = int32Array[shared.dataStartIndex + head];
          
          // Обновляем голову и счетчик
          Atomics.store(int32Array, headIndex, (head + 1) % bufferSize);
          Atomics.sub(int32Array, countIndex, 1);
          
          // Уведомляем производителя
          Atomics.notify(int32Array, countIndex, 1);
          
          return data;
        }
      }
    };
  }
  
  // Worker с поддержкой shared memory
  static createSharedWorker(workerScript) {
    const workerScriptWithShared = `
      ${workerScript}
      
      // Добавляем обработчик сообщений для shared memory
      self.onmessage = function(e) {
        if (e.data.type === 'sharedBuffer') {
          self.sharedBuffer = e.data.buffer;
          self.sharedArray = new Int32Array(self.sharedBuffer);
        } else {
          // Передаем сообщение оригинальному обработчику
          if (typeof originalOnMessage !== 'undefined') {
            originalOnMessage(e);
          }
        }
      };
    `;
    
    return WebWorkerOptimization.createWorkerFromString(workerScriptWithShared);
  }
  
  // Пример worker скрипта с shared memory
  static getSharedMemoryWorkerScript() {
    return `
      let sharedArray;
      
      self.onmessage = function(e) {
        if (e.data.type === 'sharedBuffer') {
          sharedArray = new Int32Array(e.data.buffer);
        } else if (e.data.type === 'compute') {
          const { startIndex, endIndex } = e.data;
          let sum = 0;
          
          // Выполняем вычисления и записываем результат в shared memory
          for (let i = startIndex; i < endIndex; i++) {
            sum += sharedArray[i];
          }
          
          // Записываем результат
          sharedArray[e.data.resultIndex] = sum;
          
          self.postMessage({ type: 'done', resultIndex: e.data.resultIndex });
        }
      };
    `;
  }
}

// Пример использования shared memory
class ParallelCalculator {
  constructor() {
    this.workers = [];
    this.sharedBuffer = null;
    this.sharedArray = null;
  }
  
  async init(workerCount = 4) {
    // Создаем shared buffer
    this.sharedBuffer = new SharedArrayBuffer(10000 * 4); // 10000 32-битных чисел
    this.sharedArray = new Int32Array(this.sharedBuffer);
    
    // Создаем workers
    for (let i = 0; i < workerCount; i++) {
      const worker = SharedMemoryOptimization.createSharedWorker(
        SharedMemoryOptimization.getSharedMemoryWorkerScript()
      );
      
      // Передаем shared buffer worker'у
      worker.postMessage({
        type: 'sharedBuffer',
        buffer: this.sharedBuffer
      });
      
      this.workers.push(worker);
    }
  }
  
  // Параллельные вычисления
  async parallelSum(data) {
    // Заполняем shared array данными
    this.sharedArray.set(data.slice(0, Math.min(data.length, 10000)));
    
    const chunkSize = Math.ceil(data.length / this.workers.length);
    const results = new Array(this.workers.length);
    
    // Запускаем вычисления в workers
    const promises = this.workers.map((worker, index) => {
      return new Promise((resolve) => {
        worker.onmessage = (e) => {
          if (e.data.type === 'done') {
            resolve(e.data.resultIndex);
          }
        };
        
        const startIndex = index * chunkSize;
        const endIndex = Math.min(startIndex + chunkSize, data.length);
        
        worker.postMessage({
          type: 'compute',
          startIndex,
          endIndex,
          resultIndex: 10000 + index // Результаты записываем после данных
        });
      });
    });
    
    // Ждем завершения всех вычислений
    await Promise.all(promises);
    
    // Суммируем частичные результаты
    let total = 0;
    for (let i = 0; i < this.workers.length; i++) {
      total += this.sharedArray[10000 + i];
    }
    
    return total;
  }
  
  terminate() {
    this.workers.forEach(worker => worker.terminate());
  }
}
```

## Практические рекомендации

1. **Используйте ограниченный параллелизм** - чтобы не перегружать систему
2. **Применяйте Web Workers** - для тяжелых вычислений
3. **Реализуйте retry механизмы** - для надежности асинхронных операций
4. **Используйте таймауты** - для предотвращения зависания
5. **Применяйте приоритезацию задач** - для важных операций
6. **Используйте очереди задач** - для управления нагрузкой
7. **Применяйте SharedArrayBuffer** - для эффективной синхронизации
8. **Мониторьте производительность** - измеряйте время выполнения

Оптимизация параллелизма - ключ к созданию быстрых и отзывчивых веб-приложений. Правильное использование асинхронных операций и многопоточности может значительно улучшить производительность и пользовательский опыт.

#javascript #concurrency #optimization #performance #web-workers #async #parallel-processing #sharedarraybuffer