# Лучшие практики управления памятью в JavaScript

## Введение

Следование лучшим практикам управления памятью помогает создавать более эффективные, стабильные и производительные JavaScript приложения. Эти практики охватывают все аспекты работы с памятью - от проектирования архитектуры до конкретных техник оптимизации.

## 1. Явная очистка ресурсов

Паттерн "Disposable" для явной очистки ресурсов:

```javascript
// Паттерн "Disposable" для явной очистки
class DisposableResource {
  constructor() {
    this.isDisposed = false;
    this.data = new Array(100000).fill(0);
  }
  
  processData() {
    if (this.isDisposed) {
      throw new Error('Ресурс уже освобожден');
    }
    // Обработка данных
    return this.data.length;
  }
  
  dispose() {
    if (!this.isDisposed) {
      this.data = null;
      this.isDisposed = true;
      console.log('Ресурс освобожден');
    }
  }
  
  // Symbol.dispose для использования с using (Stage 3)
  [Symbol.dispose]() {
    this.dispose();
  }
}

// Использование с явной очисткой
const resource = new DisposableResource();
try {
  const result = resource.processData();
  console.log('Результат:', result);
} finally {
  resource.dispose();
}

// В будущем с using (если будет поддерживаться):
// using resource = new DisposableResource();
// const result = resource.processData();
```

## 2. Использование WeakRef для слабых ссылок

WeakRef позволяет создавать ссылки на объекты без предотвращения их сборки:

```javascript
// Использование WeakRef для предотвращения удержания объектов
class CacheWithWeakRefs {
  constructor() {
    this.cache = new Map();
    this.finalizationRegistry = new FinalizationRegistry((key) => {
      console.log(`Объект с ключом ${key} собран сборщиком мусора`);
      this.cache.delete(key);
    });
  }
  
  set(key, value) {
    const weakRef = new WeakRef(value);
    this.cache.set(key, weakRef);
    this.finalizationRegistry.register(value, key, value);
  }
  
  get(key) {
    const weakRef = this.cache.get(key);
    if (weakRef) {
      const value = weakRef.deref();
      if (value) {
        return value;
      } else {
        // Объект собран сборщиком мусора
        this.cache.delete(key);
      }
    }
    return undefined;
  }
}
```

## 3. Оптимизация DOM операций

Эффективная работа с DOM для предотвращения утечек:

```javascript
// Эффективная работа с DOM для предотвращения утечек
class DOMManager {
  constructor() {
    this.eventListeners = new Map();
  }
  
  addEventListener(element, event, handler) {
    element.addEventListener(event, handler);
    
    if (!this.eventListeners.has(element)) {
      this.eventListeners.set(element, []);
    }
    
    this.eventListeners.get(element).push({ event, handler });
  }
  
  removeAllListeners(element) {
    const listeners = this.eventListeners.get(element);
    if (listeners) {
      listeners.forEach(({ event, handler }) => {
        element.removeEventListener(event, handler);
      });
      this.eventListeners.delete(element);
    }
  }
  
  destroy() {
    for (const element of this.eventListeners.keys()) {
      this.removeAllListeners(element);
    }
  }
}
```

## 4. Управление жизненным циклом объектов

Правильное управление жизненным циклом объектов:

```javascript
// Менеджер жизненного цикла объектов
class LifecycleManager {
  constructor() {
    this.managedObjects = new Set();
    this.cleanupCallbacks = new Map();
  }
  
  add(object, cleanupCallback) {
    this.managedObjects.add(object);
    if (cleanupCallback) {
      this.cleanupCallbacks.set(object, cleanupCallback);
    }
  }
  
  remove(object) {
    const cleanupCallback = this.cleanupCallbacks.get(object);
    if (cleanupCallback) {
      cleanupCallback(object);
      this.cleanupCallbacks.delete(object);
    }
    this.managedObjects.delete(object);
  }
  
  cleanupAll() {
    for (const object of this.managedObjects) {
      this.remove(object);
    }
  }
  
  // Автоматическая очистка при сборке менеджера
  destroy() {
    this.cleanupAll();
  }
}

// Использование
const lifecycleManager = new LifecycleManager();

const largeObject = {
  data: new Array(1000000).fill(0),
  cleanup: function() {
    this.data = null;
    console.log('Объект очищен');
  }
};

lifecycleManager.add(largeObject, (obj) => obj.cleanup());

// При завершении работы
// lifecycleManager.destroy();
```

## 5. Использование паттерна Observer с автоматической отпиской

Паттерн Observer с автоматическим управлением подписками:

```javascript
// Observer с автоматической отпиской
class AutoUnsubscribeObserver {
  constructor() {
    this.subscribers = new Set();
    this.finalizationRegistry = new FinalizationRegistry((unsubscribe) => {
      unsubscribe();
    });
  }
  
  subscribe(callback) {
    const unsubscribe = () => {
      this.subscribers.delete(callback);
    };
    
    this.subscribers.add(callback);
    this.finalizationRegistry.register(callback, unsubscribe, callback);
    
    return unsubscribe;
  }
  
  notify(data) {
    for (const callback of this.subscribers) {
      try {
        callback(data);
      } catch (error) {
        console.error('Ошибка в подписчике:', error);
      }
    }
  }
}
```

## 6. Оптимизация работы с большими данными

Техники для эффективной работы с большими объемами данных:

```javascript
// Оптимизация работы с большими данными
class LargeDataOptimizer {
  // Использование генераторов для ленивой обработки
  static *processDataGenerator(data, processor) {
    for (const item of data) {
      yield processor(item);
    }
  }
  
  // Пакетная обработка с ограничением памяти
  static async processInBatches(data, batchSize, processor) {
    const results = [];
    
    for (let i = 0; i < data.length; i += batchSize) {
      const batch = data.slice(i, i + batchSize);
      const processedBatch = batch.map(processor);
      results.push(...processedBatch);
      
      // Освобождение памяти между пакетами
      if (i % (batchSize * 10) === 0) {
        await new Promise(resolve => setTimeout(resolve, 0));
      }
    }
    
    return results;
  }
  
  // Использование Transform Streams для обработки потоков данных
  static createTransformStream(transformer) {
    return new TransformStream({
      transform(chunk, controller) {
        const transformed = transformer(chunk);
        controller.enqueue(transformed);
      }
    });
  }
}
```

## 7. Мониторинг и профилирование

Регулярный мониторинг для предотвращения проблем:

```javascript
// Система мониторинга с автоматическими предупреждениями
class MemoryMonitoringSystem {
  constructor() {
    this.thresholds = {
      memoryGrowth: 0.1, // 10% рост памяти
      gcFrequency: 5000, // 5 секунд между GC
      leakDetection: 10000 // 10 секунд для детектирования утечек
    };
    this.lastGcTime = Date.now();
    this.memoryHistory = [];
  }
  
  checkMemoryUsage() {
    if (performance.memory) {
      const current = {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        timestamp: Date.now()
      };
      
      this.memoryHistory.push(current);
      
      // Ограничиваем историю
      if (this.memoryHistory.length > 100) {
        this.memoryHistory.shift();
      }
      
      // Проверяем на утечки
      this.detectLeaks();
      
      // Проверяем частоту GC
      this.checkGcFrequency();
    }
  }
  
  detectLeaks() {
    if (this.memoryHistory.length < 10) return;
    
    const recent = this.memoryHistory.slice(-10);
    const initial = recent[0].used;
    const current = recent[recent.length - 1].used;
    const growth = (current - initial) / initial;
    
    if (growth > this.thresholds.memoryGrowth) {
      console.warn('Возможная утечка памяти обнаружена!', {
        growth: (growth * 100).toFixed(2) + '%',
        initial: this.formatBytes(initial),
        current: this.formatBytes(current)
      });
    }
  }
  
  checkGcFrequency() {
    const now = Date.now();
    const timeSinceLastGc = now - this.lastGcTime;
    
    if (timeSinceLastGc < this.thresholds.gcFrequency) {
      console.warn('Частые сборки мусора обнаружены!', {
        interval: timeSinceLastGc + 'ms'
      });
    }
    
    this.lastGcTime = now;
  }
  
  formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
}
```

## 8. Архитектурные рекомендации

### Использование модульной архитектуры

```javascript
// Модуль с автоматической очисткой
const DataModule = (() => {
  let privateData = null;
  let cleanupTimer = null;
  
  const initialize = () => {
    privateData = new Array(100000).fill(0);
    cleanupTimer = setTimeout(cleanup, 30000); // Автоматическая очистка через 30 секунд
  };
  
  const cleanup = () => {
    if (privateData) {
      privateData = null;
      if (cleanupTimer) {
        clearTimeout(cleanupTimer);
        cleanupTimer = null;
      }
      console.log('Модуль очищен');
    }
  };
  
  const processData = () => {
    if (!privateData) {
      initialize();
    }
    return privateData.length;
  };
  
  // Public API
  return {
    processData,
    cleanup
  };
})();

// Использование
// const result = DataModule.processData();
// DataModule.cleanup(); // Явная очистка
```

## Заключение

Следование этим лучшим практикам поможет создавать JavaScript приложения, которые эффективно используют память, не страдают от утечек и обеспечивают хорошую производительность. Ключевые принципы:

1. **Явное управление ресурсами** - всегда очищайте то, что создали
2. **Слабые ссылки** - используйте WeakMap, WeakSet, WeakRef когда это возможно
3. **Мониторинг** - регулярно проверяйте использование памяти
4. **Оптимизация** - применяйте техники оптимизации для больших данных
5. **Архитектура** - проектируйте приложения с учетом управления памятью

#javascript #memory-management #best-practices #performance #architecture