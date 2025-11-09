# Профилирование производительности в JavaScript

## Введение

Профилирование - это процесс измерения производительности приложения для выявления узких мест и оптимизации кода. Эффективное профилирование позволяет находить и устранять проблемы с производительностью до того, как они повлияют на пользователей.

## Инструменты профилирования

### Встроенные инструменты браузера

Браузеры предоставляют мощные инструменты для профилирования JavaScript:

```javascript
// Использование console.time для измерения времени выполнения
class ConsoleTimeProfiling {
  static measureFunctionExecution() {
    console.time('Функция выполнена');
    
    // Код для измерения
    for (let i = 0; i < 1000000; i++) {
      Math.sqrt(i);
    }
    
    console.timeEnd('Функция выполнена');
  }
  
  static measureMultipleOperations() {
    console.time('Все операции');
    
    console.time('Операция 1');
    // Первая операция
    const arr = new Array(100000).fill(0).map(() => Math.random());
    console.timeEnd('Операция 1');
    
    console.time('Операция 2');
    // Вторая операция
    const sorted = arr.sort((a, b) => a - b);
    console.timeEnd('Операция 2');
    
    console.time('Операция 3');
    // Третья операция
    const sum = sorted.reduce((acc, val) => acc + val, 0);
    console.timeEnd('Операция 3');
    
    console.timeEnd('Все операции');
  }
}

// Использование performance API
class PerformanceAPIProfiling {
  static measureCustomTiming() {
    performance.mark('start');
    
    // Код для измерения
    const data = [];
    for (let i = 0; i < 100000; i++) {
      data.push({ id: i, value: Math.random() });
    }
    
    performance.mark('end');
    performance.measure('data-generation', 'start', 'end');
    
    const measure = performance.getEntriesByName('data-generation')[0];
    console.log(`Генерация данных заняла ${measure.duration} мс`);
  }
  
  static measureFunctionPerformance(fn, ...args) {
    const start = performance.now();
    const result = fn.apply(this, args);
    const end = performance.now();
    
    console.log(`${fn.name} выполнена за ${end - start} мс`);
    return result;
  }
  
  static createPerformanceObserver() {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        console.log(`Производительность: ${entry.name} заняла ${entry.duration} мс`);
      }
    });
    
    observer.observe({ entryTypes: ['measure', 'navigation', 'resource'] });
    return observer;
  }
}
```

### Создание пользовательских инструментов профилирования

Разработка собственных инструментов для профилирования:

```javascript
// Пользовательский профайлер
class CustomProfiler {
  constructor() {
    this.measurements = new Map();
    this.activeTimers = new Map();
  }
  
  // Запуск таймера
  start(label) {
    if (this.activeTimers.has(label)) {
      console.warn(`Таймер '${label}' уже запущен`);
      return;
    }
    
    this.activeTimers.set(label, {
      startTime: performance.now(),
      startMemory: this.getMemoryUsage()
    });
  }
  
  // Остановка таймера
  end(label) {
    const timer = this.activeTimers.get(label);
    if (!timer) {
      console.warn(`Таймер '${label}' не найден`);
      return null;
    }
    
    const endTime = performance.now();
    const endMemory = this.getMemoryUsage();
    
    const measurement = {
      duration: endTime - timer.startTime,
      memoryDelta: {
        usedJSHeapSize: endMemory.usedJSHeapSize - timer.startMemory.usedJSHeapSize,
        totalJSHeapSize: endMemory.totalJSHeapSize - timer.startMemory.totalJSHeapSize
      },
      timestamp: new Date()
    };
    
    if (!this.measurements.has(label)) {
      this.measurements.set(label, []);
    }
    
    this.measurements.get(label).push(measurement);
    this.activeTimers.delete(label);
    
    return measurement;
  }
  
  // Получение использования памяти
  getMemoryUsage() {
    if (performance.memory) {
      return {
        usedJSHeapSize: performance.memory.usedJSHeapSize,
        totalJSHeapSize: performance.memory.totalJSHeapSize,
        jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
      };
    }
    return { usedJSHeapSize: 0, totalJSHeapSize: 0, jsHeapSizeLimit: 0 };
  }
  
  // Получение статистики по измерениям
  getStats(label) {
    const measurements = this.measurements.get(label);
    if (!measurements || measurements.length === 0) {
      return null;
    }
    
    const durations = measurements.map(m => m.duration);
    const avgDuration = durations.reduce((sum, d) => sum + d, 0) / durations.length;
    const minDuration = Math.min(...durations);
    const maxDuration = Math.max(...durations);
    
    return {
      count: measurements.length,
      avgDuration,
      minDuration,
      maxDuration,
      lastMeasurement: measurements[measurements.length - 1]
    };
  }
  
  // Логирование статистики
  logStats(label) {
    const stats = this.getStats(label);
    if (!stats) {
      console.log(`Нет измерений для '${label}'`);
      return;
    }
    
    console.log(`${label}:`);
    console.log(`  Вызовов: ${stats.count}`);
    console.log(`  Среднее время: ${stats.avgDuration.toFixed(2)} мс`);
    console.log(`  Минимальное время: ${stats.minDuration.toFixed(2)} мс`);
    console.log(`  Максимальное время: ${stats.maxDuration.toFixed(2)} мс`);
  }
  
  // Экспорт данных
  exportData() {
    const data = {};
    for (const [label, measurements] of this.measurements) {
      data[label] = measurements.map(m => ({
        duration: m.duration,
        memoryDelta: m.memoryDelta,
        timestamp: m.timestamp.toISOString()
      }));
    }
    return data;
  }
}

// Декоратор для профилирования методов
function profile(target, propertyName, descriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args) {
    const profiler = new CustomProfiler();
    profiler.start(propertyName);
    
    try {
      const result = method.apply(this, args);
      
      if (result instanceof Promise) {
        return result.then(resolvedResult => {
          profiler.end(propertyName);
          profiler.logStats(propertyName);
          return resolvedResult;
        }).catch(error => {
          profiler.end(propertyName);
          profiler.logStats(propertyName);
          throw error;
        });
      } else {
        profiler.end(propertyName);
        profiler.logStats(propertyName);
        return result;
      }
    } catch (error) {
      profiler.end(propertyName);
      profiler.logStats(propertyName);
      throw error;
    }
  };
  
  return descriptor;
}

// Пример использования декоратора
class DataProcessor {
  @profile
  processData(data) {
    // Имитация тяжелой обработки данных
    return data.map(item => ({
      ...item,
      processed: true,
      timestamp: Date.now()
    })).filter(item => item.value > 0.5);
  }
  
  @profile
  async asyncProcessData(data) {
    // Имитация асинхронной обработки
    return new Promise(resolve => {
      setTimeout(() => {
        const result = this.processData(data);
        resolve(result);
      }, 100);
    });
  }
}
```

## Профилирование памяти

Анализ использования памяти и утечек:

```javascript
// Профилирование памяти
class MemoryProfiling {
  // Мониторинг использования памяти
  static monitorMemoryUsage() {
    if (!performance.memory) {
      console.warn('Performance memory API не доступен');
      return;
    }
    
    const memory = performance.memory;
    console.log('Использование памяти:');
    console.log(`  Использовано: ${(memory.usedJSHeapSize / 1024 / 1024).toFixed(2)} MB`);
    console.log(`  Всего выделено: ${(memory.totalJSHeapSize / 1024 / 1024).toFixed(2)} MB`);
    console.log(`  Лимит: ${(memory.jsHeapSizeLimit / 1024 / 1024).toFixed(2)} MB`);
    
    return memory;
  }
  
  // Периодический мониторинг памяти
  static startMemoryMonitoring(interval = 5000) {
    console.log('Начало мониторинга памяти...');
    
    const intervalId = setInterval(() => {
      const memory = this.monitorMemoryUsage();
      if (memory) {
        // Проверка на утечки памяти
        if (memory.usedJSHeapSize > memory.jsHeapSizeLimit * 0.8) {
          console.warn('Высокое использование памяти! Возможна утечка.');
        }
      }
    }, interval);
    
    return intervalId;
  }
  
  // Анализ утечек памяти
  static detectMemoryLeaks() {
    const snapshots = [];
    let snapshotId = 0;
    
    return {
      takeSnapshot() {
        if (!performance.memory) return null;
        
        const snapshot = {
          id: ++snapshotId,
          timestamp: Date.now(),
          memory: { ...performance.memory },
          heapSize: performance.memory.usedJSHeapSize
        };
        
        snapshots.push(snapshot);
        
        // Ограничиваем количество снимков
        if (snapshots.length > 10) {
          snapshots.shift();
        }
        
        return snapshot;
      },
      
      analyzeTrend() {
        if (snapshots.length < 2) {
          return 'Недостаточно данных для анализа';
        }
        
        const first = snapshots[0];
        const last = snapshots[snapshots.length - 1];
        const timeDiff = (last.timestamp - first.timestamp) / 1000; // в секундах
        const memoryDiff = last.heapSize - first.heapSize;
        const memoryGrowthRate = memoryDiff / timeDiff; // байт/секунда
        
        if (memoryGrowthRate > 100000) { // 100KB/секунда
          return `Обнаружена утечка памяти: рост ${Math.round(memoryGrowthRate / 1024)} KB/секунда`;
        } else if (memoryGrowthRate > 0) {
          return `Небольшой рост памяти: ${Math.round(memoryGrowthRate / 1024)} KB/секунда`;
        } else {
          return 'Утечка памяти не обнаружена';
        }
      },
      
      getSnapshots() {
        return [...snapshots];
      }
    };
  }
  
  // Трекинг слушателей событий
  static createEventListenerTracker() {
    const listeners = new Map();
    
    // Перехватываем addEventListener
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function(type, listener, options) {
      const key = `${this.constructor.name}_${type}`;
      if (!listeners.has(key)) {
        listeners.set(key, new Set());
      }
      listeners.get(key).add({ target: this, listener, options });
      return originalAddEventListener.call(this, type, listener, options);
    };
    
    // Перехватываем removeEventListener
    const originalRemoveEventListener = EventTarget.prototype.removeEventListener;
    EventTarget.prototype.removeEventListener = function(type, listener, options) {
      const key = `${this.constructor.name}_${type}`;
      if (listeners.has(key)) {
        const set = listeners.get(key);
        for (const item of set) {
          if (item.target === this && item.listener === listener) {
            set.delete(item);
            break;
          }
        }
      }
      return originalRemoveEventListener.call(this, type, listener, options);
    };
    
    return {
      getListenerCount() {
        let total = 0;
        for (const set of listeners.values()) {
          total += set.size;
        }
        return total;
      },
      
      getListenersByType() {
        const result = {};
        for (const [key, set] of listeners) {
          result[key] = set.size;
        }
        return result;
      },
      
      logListeners() {
        console.log('Активные слушатели событий:');
        for (const [key, set] of listeners) {
          console.log(`  ${key}: ${set.size} слушателей`);
        }
      }
    };
  }
}
```

## Профилирование функций и методов

Детальный анализ производительности отдельных функций:

```javascript
// Профилирование функций
class FunctionProfiling {
  // Профилирование с помощью Function.prototype.call
  static profileFunction(fn, context, ...args) {
    const start = performance.now();
    const startMemory = performance.memory ? performance.memory.usedJSHeapSize : 0;
    
    try {
      const result = fn.apply(context, args);
      const end = performance.now();
      const endMemory = performance.memory ? performance.memory.usedJSHeapSize : 0;
      
      console.log(`${fn.name || 'Анонимная функция'}:`);
      console.log(`  Время выполнения: ${(end - start).toFixed(2)} мс`);
      console.log(`  Использование памяти: ${(endMemory - startMemory) / 1024} KB`);
      
      return {
        result,
        duration: end - start,
        memoryUsage: endMemory - startMemory
      };
    } catch (error) {
      const end = performance.now();
      console.log(`${fn.name || 'Анонимная функция'} завершилась с ошибкой:`);
      console.log(`  Время до ошибки: ${(end - start).toFixed(2)} мс`);
      throw error;
    }
  }
  
  // Бенчмаркинг функций
  static benchmarkFunction(fn, iterations = 1000, ...args) {
    const results = [];
    
    // Разогрев
    for (let i = 0; i < 10; i++) {
      fn.apply(null, args);
    }
    
    // Измерение
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      fn.apply(null, args);
      const end = performance.now();
      results.push(end - start);
    }
    
    const avg = results.reduce((sum, time) => sum + time, 0) / iterations;
    const min = Math.min(...results);
    const max = Math.max(...results);
    const median = results.sort((a, b) => a - b)[Math.floor(iterations / 2)];
    
    console.log(`Бенчмарк ${fn.name || 'функции'} (${iterations} итераций):`);
    console.log(`  Среднее: ${avg.toFixed(4)} мс`);
    console.log(`  Минимум: ${min.toFixed(4)} мс`);
    console.log(`  Максимум: ${max.toFixed(4)} мс`);
    console.log(`  Медиана: ${median.toFixed(4)} мс`);
    
    return { avg, min, max, median, results };
  }
  
  // Сравнение производительности двух функций
  static compareFunctions(fn1, fn2, iterations = 1000, ...args) {
    console.log('Сравнение производительности функций:');
    
    const benchmark1 = this.benchmarkFunction(fn1, iterations, ...args);
    const benchmark2 = this.benchmarkFunction(fn2, iterations, ...args);
    
    const ratio = benchmark1.avg / benchmark2.avg;
    
    console.log(`\nРезультаты:`);
    console.log(`${fn1.name} быстрее ${fn2.name} в ${ratio.toFixed(2)} раз(а)`);
    
    if (ratio > 1) {
      console.log(`${fn2.name} быстрее на ${((ratio - 1) * 100).toFixed(2)}%`);
    } else {
      console.log(`${fn1.name} быстрее на ${((1/ratio - 1) * 100).toFixed(2)}%`);
    }
    
    return {
      fn1: benchmark1,
      fn2: benchmark2,
      ratio
    };
  }
  
  // Примеры функций для сравнения
  static exampleFunctions() {
    // Медленная реализация
    function slowSum(arr) {
      let sum = 0;
      for (let i = 0; i < arr.length; i++) {
        sum += arr[i];
      }
      return sum;
    }
    
    // Быстрая реализация
    function fastSum(arr) {
      return arr.reduce((sum, val) => sum + val, 0);
    }
    
    // Тестовые данные
    const testData = new Array(10000).fill(0).map(() => Math.random());
    
    // Сравнение
    this.compareFunctions(slowSum, fastSum, 1000, testData);
  }
}
```

## Практические рекомендации

1. **Используйте встроенные инструменты** - Chrome DevTools, Firefox Developer Tools
2. **Измеряйте реальную производительность** - на реальных данных и устройствах
3. **Профилируйте регулярно** - во время разработки и перед релизом
4. **Анализируйте утечки памяти** - с помощью memory profiling
5. **Оптимизируйте критические пути** - определяйте самые медленные части кода
6. **Используйте бенчмаркинг** - для сравнения разных подходов
7. **Мониторьте производительность** - в production среде
8. **Документируйте результаты** - для отслеживания улучшений

Профилирование - важный этап в разработке высокопроизводительных приложений. Регулярное использование инструментов профилирования помогает выявлять и устранять проблемы с производительностью на ранних стадиях разработки.

#javascript #profiling #performance #memory #benchmarking #devtools