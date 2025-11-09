# Асинхронные оптимизации в JavaScript

## Введение

Асинхронные операции - неотъемлемая часть веб-разработки, но они также могут быть источником проблем с производительностью. Правильная оптимизация асинхронного кода помогает создавать более отзывчивые и эффективные приложения.

## Использование Web Workers

Web Workers позволяют выполнять тяжелые вычисления в фоновом потоке, не блокируя основной поток:

```javascript
// Оптимизация тяжелых вычислений с Web Workers
class WorkerOptimization {
  static createWorker() {
    // Создание Web Worker из строки (в реальном приложении используйте отдельный файл)
    const workerScript = `
      self.onmessage = function(e) {
        const { data, operation } = e.data;
        
        let result;
        switch (operation) {
          case 'sort':
            result = data.sort((a, b) => a - b);
            break;
          case 'calculate':
            result = data.reduce((sum, val) => sum + Math.sqrt(val), 0);
            break;
          default:
            result = data;
        }
        
        self.postMessage({ result, operation });
      };
    `;
    
    const blob = new Blob([workerScript], { type: 'application/javascript' });
    return new Worker(URL.createObjectURL(blob));
  }
  
  static async processDataWithWorker(data, operation) {
    return new Promise((resolve, reject) => {
      const worker = this.createWorker();
      
      worker.onmessage = function(e) {
        resolve(e.data.result);
        worker.terminate();
      };
      
      worker.onerror = function(error) {
        reject(error);
        worker.terminate();
      };
      
      worker.postMessage({ data, operation });
    });
  }
  
  static async processDataInMainThread(data, operation) {
    switch (operation) {
      case 'sort':
        return data.sort((a, b) => a - b);
      case 'calculate':
        return data.reduce((sum, val) => sum + Math.sqrt(val), 0);
      default:
        return data;
    }
  }
  
  static async benchmark() {
    const largeData = new Array(100000).fill(0).map(() => Math.random() * 1000);
    
    console.time('Main thread sorting');
    const sorted1 = await this.processDataInMainThread([...largeData], 'sort');
    console.timeEnd('Main thread sorting');
    
    console.time('Worker sorting');
    const sorted2 = await this.processDataWithWorker([...largeData], 'sort');
    console.timeEnd('Worker sorting');
  }
}
```

## Оптимизация Promise

Эффективное использование Promise и асинхронных операций:

```javascript
// Оптимизация Promise
class PromiseOptimization {
  // Параллельное выполнение с Promise.all
  static async parallelExecution(tasks) {
    try {
      const results = await Promise.all(tasks);
      return results;
    } catch (error) {
      console.error('Одна из задач завершилась с ошибкой:', error);
      throw error;
    }
  }
  
  // Выполнение с ограничением параллелизма
  static async limitedParallelExecution(tasks, limit) {
    const results = [];
    
    for (let i = 0; i < tasks.length; i += limit) {
      const batch = tasks.slice(i, i + limit);
      const batchResults = await Promise.all(batch);
      results.push(...batchResults);
    }
    
    return results;
  }
  
  // Использование Promise.allSettled для обработки всех результатов
  static async processAllTasks(tasks) {
    const results = await Promise.allSettled(tasks);
    
    const successful = results
      .filter(result => result.status === 'fulfilled')
      .map(result => result.value);
    
    const failed = results
      .filter(result => result.status === 'rejected')
      .map(result => result.reason);
    
    return { successful, failed };
  }
  
  // Отменяемые Promise
  static createCancellablePromise(promiseFactory) {
    let cancelled = false;
    
    const promise = new Promise((resolve, reject) => {
      promiseFactory()
        .then(result => {
          if (!cancelled) {
            resolve(result);
          }
        })
        .catch(error => {
          if (!cancelled) {
            reject(error);
          }
        });
    });
    
    promise.cancel = () => {
      cancelled = true;
    };
    
    return promise;
  }
}
```

## Оптимизация асинхронных циклов

Правильная обработка асинхронных операций в циклах:

```javascript
// Оптимизация асинхронных циклов
class AsyncLoopOptimization {
  // Последовательное выполнение
  static async sequentialExecution(tasks) {
    const results = [];
    for (const task of tasks) {
      try {
        const result = await task();
        results.push(result);
      } catch (error) {
        console.error('Ошибка в задаче:', error);
        results.push(null);
      }
    }
    return results;
  }
  
  // Параллельное выполнение с ограничением
  static async parallelWithLimit(tasks, limit) {
    const results = new Array(tasks.length);
    const executing = [];
    
    for (let i = 0; i < tasks.length; i++) {
      const task = tasks[i];
      const promise = task().then(result => {
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
  
  // Использование async/await с массивами
  static async processArrayAsync(arr, processor) {
    // Не блокируем основной поток
    const results = [];
    for (let i = 0; i < arr.length; i++) {
      results.push(await processor(arr[i]));
      
      // Даем возможность выполнить другие задачи
      if (i % 100 === 0) {
        await new Promise(resolve => setTimeout(resolve, 0));
      }
    }
    return results;
  }
}
```

## Оптимизация сетевых запросов

Эффективная работа с сетевыми запросами:

```javascript
// Оптимизация сетевых запросов
class NetworkOptimization {
  // Кэширование запросов
  static requestCache = new Map();
  
  static async cachedFetch(url, options = {}) {
    const cacheKey = `${url}_${JSON.stringify(options)}`;
    
    // Проверяем кэш
    if (this.requestCache.has(cacheKey)) {
      const cached = this.requestCache.get(cacheKey);
      if (Date.now() - cached.timestamp < 60000) { // 1 минута
        return cached.data;
      }
    }
    
    // Выполняем запрос
    try {
      const response = await fetch(url, options);
      const data = await response.json();
      
      // Сохраняем в кэш
      this.requestCache.set(cacheKey, {
        data,
        timestamp: Date.now()
      });
      
      return data;
    } catch (error) {
      console.error('Ошибка запроса:', error);
      throw error;
    }
  }
  
  // Пакетирование запросов
  static async batchRequests(requests, batchSize = 5) {
    const results = [];
    
    for (let i = 0; i < requests.length; i += batchSize) {
      const batch = requests.slice(i, i + batchSize);
      const batchPromises = batch.map(req => fetch(req.url, req.options));
      const batchResults = await Promise.all(batchPromises);
      
      results.push(...batchResults);
    }
    
    return results;
  }
  
  // Retry механизм
  static async fetchWithRetry(url, options = {}, maxRetries = 3) {
    for (let i = 0; i <= maxRetries; i++) {
      try {
        const response = await fetch(url, options);
        if (response.ok) {
          return await response.json();
        }
        
        if (i === maxRetries) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
      } catch (error) {
        if (i === maxRetries) {
          throw error;
        }
        
        // Ждем перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
  }
}
```

## Использование async/await эффективно

Правильное использование async/await:

```javascript
// Эффективное использование async/await
class AsyncAwaitOptimization {
  // Параллельное выполнение независимых операций
  static async parallelIndependentOperations() {
    // Плохо: последовательное выполнение
    // const user = await fetchUser();
    // const permissions = await fetchPermissions();
    // const settings = await fetchSettings();
    
    // Хорошо: параллельное выполнение
    const [user, permissions, settings] = await Promise.all([
      this.fetchUser(),
      this.fetchPermissions(),
      this.fetchSettings()
    ]);
    
    return { user, permissions, settings };
  }
  
  // Обработка ошибок в async/await
  static async handleErrors() {
    try {
      const data = await this.fetchData();
      return data;
    } catch (error) {
      // Логируем ошибку
      console.error('Ошибка при получении данных:', error);
      
      // Возвращаем значение по умолчанию
      return [];
    }
  }
  
  // Использование finally для очистки
  static async withCleanup() {
    let resource = null;
    
    try {
      resource = await this.acquireResource();
      const data = await this.processData(resource);
      return data;
    } finally {
      if (resource) {
        await this.releaseResource(resource);
      }
    }
  }
  
  // Имитация асинхронных операций
  static fetchUser() {
    return new Promise(resolve => setTimeout(() => resolve({ id: 1, name: 'User' }), 100));
  }
  
  static fetchPermissions() {
    return new Promise(resolve => setTimeout(() => resolve(['read', 'write']), 150));
  }
  
  static fetchSettings() {
    return new Promise(resolve => setTimeout(() => resolve({ theme: 'dark' }), 200));
  }
  
  static fetchData() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (Math.random() > 0.5) {
          resolve([1, 2, 3, 4, 5]);
        } else {
          reject(new Error('Network error'));
        }
      }, 100);
    });
  }
  
  static acquireResource() {
    return new Promise(resolve => setTimeout(() => resolve({ id: 'resource-1' }), 50));
  }
  
  static processData(resource) {
    return new Promise(resolve => setTimeout(() => resolve([resource.id, 'processed']), 100));
  }
  
  static releaseResource(resource) {
    return new Promise(resolve => setTimeout(() => resolve(), 25));
  }
}
```

## Практические рекомендации

1. **Используйте Web Workers** - для тяжелых вычислений
2. **Применяйте Promise.all** - для параллельного выполнения независимых операций
3. **Ограничивайте параллелизм** - чтобы не перегружать систему
4. **Кэшируйте результаты** - для повторяющихся запросов
5. **Используйте retry механизмы** - для надежности
6. **Обрабатывайте ошибки правильно** - с async/await
7. **Избегайте блокировки основного потока** - разбивайте тяжелые операции
8. **Используйте finally** - для очистки ресурсов

Асинхронные оптимизации помогают создавать более отзывчивые приложения и эффективно использовать системные ресурсы. Следование этим принципам улучшит пользовательский опыт и производительность приложений.

#javascript #async #optimization #performance #web-workers #promises