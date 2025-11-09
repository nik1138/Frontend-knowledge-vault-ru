# Оптимизация хранилищ в JavaScript

## Введение

Эффективное использование клиентских хранилищ (localStorage, sessionStorage, IndexedDB) играет важную роль в создании быстрых и отзывчивых веб-приложений. Правильная оптимизация хранилищ может значительно улучшить производительность, уменьшить время загрузки и повысить пользовательский опыт. В этом разделе мы рассмотрим различные техники оптимизации хранилищ в JavaScript.

## Оптимизация localStorage и sessionStorage

Эффективное использование Web Storage API:

```javascript
// Оптимизация localStorage и sessionStorage
class StorageOptimization {
  // Базовый кэш с TTL
  static createTTLStorage(storage = localStorage, prefix = 'cache_') {
    return {
      set(key, value, ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
        const expiration = Date.now() + ttl;
        const item = {
          value,
          expiration
        };
        
        try {
          storage.setItem(prefix + key, JSON.stringify(item));
        } catch (error) {
          console.error('Ошибка при сохранении в хранилище:', error);
          // Обработка ошибок переполнения
          this.clearExpired();
          try {
            storage.setItem(prefix + key, JSON.stringify(item));
          } catch (retryError) {
            console.error('Не удалось сохранить данные даже после очистки:', retryError);
          }
        }
      },
      
      get(key) {
        try {
          const itemStr = storage.getItem(prefix + key);
          if (!itemStr) return undefined;
          
          const item = JSON.parse(itemStr);
          
          // Проверяем срок действия
          if (Date.now() > item.expiration) {
            storage.removeItem(prefix + key);
            return undefined;
          }
          
          return item.value;
        } catch (error) {
          console.error('Ошибка при получении из хранилища:', error);
          return undefined;
        }
      },
      
      delete(key) {
        try {
          storage.removeItem(prefix + key);
          return true;
        } catch (error) {
          console.error('Ошибка при удалении из хранилища:', error);
          return false;
        }
      },
      
      clearExpired() {
        const now = Date.now();
        let cleared = 0;
        
        for (let i = 0; i < storage.length; i++) {
          const key = storage.key(i);
          if (key && key.startsWith(prefix)) {
            try {
              const itemStr = storage.getItem(key);
              if (itemStr) {
                const item = JSON.parse(itemStr);
                if (now > item.expiration) {
                  storage.removeItem(key);
                  cleared++;
                  i--; // Корректируем индекс после удаления
                }
              }
            } catch (error) {
              console.error('Ошибка при очистке устаревших записей:', error);
            }
          }
        }
        
        return cleared;
      },
      
      getKeys() {
        const keys = [];
        for (let i = 0; i < storage.length; i++) {
          const key = storage.key(i);
          if (key && key.startsWith(prefix)) {
            keys.push(key.substring(prefix.length));
          }
        }
        return keys;
      },
      
      clear() {
        const keys = this.getKeys();
        keys.forEach(key => {
          storage.removeItem(prefix + key);
        });
      },
      
      getSize() {
        let totalSize = 0;
        for (let i = 0; i < storage.length; i++) {
          const key = storage.key(i);
          if (key && key.startsWith(prefix)) {
            const value = storage.getItem(key);
            totalSize += key.length + (value ? value.length : 0);
          }
        }
        return totalSize;
      }
    };
  }
  
  // Компрессия данных перед сохранением
  static createCompressedStorage(storage = localStorage, prefix = 'compressed_') {
    return {
      set(key, value) {
        try {
          // Преобразуем в строку
          const jsonString = JSON.stringify(value);
          
          // Простая компрессия (в реальных приложениях используйте библиотеки)
          const compressed = this.compressString(jsonString);
          
          storage.setItem(prefix + key, compressed);
        } catch (error) {
          console.error('Ошибка при сжатии и сохранении:', error);
        }
      },
      
      get(key) {
        try {
          const compressed = storage.getItem(prefix + key);
          if (!compressed) return undefined;
          
          const decompressed = this.decompressString(compressed);
          return JSON.parse(decompressed);
        } catch (error) {
          console.error('Ошибка при получении и распаковке:', error);
          return undefined;
        }
      },
      
      compressString(str) {
        // Простая реализация RLE (Run-Length Encoding)
        let compressed = '';
        let currentChar = str[0];
        let count = 1;
        
        for (let i = 1; i <= str.length; i++) {
          if (i === str.length || str[i] !== currentChar) {
            compressed += count + currentChar;
            if (i < str.length) {
              currentChar = str[i];
              count = 1;
            }
          } else {
            count++;
          }
        }
        
        // Возвращаем сжатую строку, если она короче оригинала
        return compressed.length < str.length ? compressed : str;
      },
      
      decompressString(compressed) {
        // Декомпрессия RLE
        let decompressed = '';
        let i = 0;
        
        while (i < compressed.length) {
          let count = '';
          while (i < compressed.length && /\d/.test(compressed[i])) {
            count += compressed[i];
            i++;
          }
          
          if (count && i < compressed.length) {
            const char = compressed[i];
            decompressed += char.repeat(parseInt(count));
            i++;
          } else if (i < compressed.length) {
            decompressed += compressed[i];
            i++;
          }
        }
        
        return decompressed;
      }
    };
  }
  
  // Батчинг операций
  static createBatchStorage(storage = localStorage) {
    const batchOperations = [];
    let batchTimeout = null;
    
    return {
      set(key, value) {
        batchOperations.push({ type: 'set', key, value });
        this.scheduleBatch();
      },
      
      delete(key) {
        batchOperations.push({ type: 'delete', key });
        this.scheduleBatch();
      },
      
      scheduleBatch() {
        if (batchTimeout) {
          clearTimeout(batchTimeout);
        }
        
        batchTimeout = setTimeout(() => {
          this.executeBatch();
        }, 10); // Выполняем батч через 10 мс
      },
      
      executeBatch() {
        batchOperations.forEach(operation => {
          try {
            if (operation.type === 'set') {
              storage.setItem(operation.key, JSON.stringify(operation.value));
            } else if (operation.type === 'delete') {
              storage.removeItem(operation.key);
            }
          } catch (error) {
            console.error('Ошибка при выполнении батч операции:', error);
          }
        });
        
        batchOperations.length = 0;
        batchTimeout = null;
      }
    };
  }
  
  // Мониторинг использования хранилища
  static createStorageMonitor(storage = localStorage) {
    const originalSetItem = storage.setItem;
    const originalRemoveItem = storage.removeItem;
    const usageStats = new Map();
    
    // Перехватываем методы
    storage.setItem = function(key, value) {
      const size = key.length + (typeof value === 'string' ? value.length : JSON.stringify(value).length);
      usageStats.set(key, { size, lastAccess: Date.now() });
      return originalSetItem.call(this, key, value);
    };
    
    storage.removeItem = function(key) {
      usageStats.delete(key);
      return originalRemoveItem.call(this, key);
    };
    
    return {
      getStats() {
        let totalSize = 0;
        let itemCount = 0;
        
        for (const [key, stats] of usageStats) {
          totalSize += stats.size;
          itemCount++;
        }
        
        return {
          totalSize,
          itemCount,
          averageItemSize: itemCount > 0 ? Math.round(totalSize / itemCount) : 0,
          usageStats: Array.from(usageStats.entries())
        };
      },
      
      getLargestItems(limit = 5) {
        return Array.from(usageStats.entries())
          .sort((a, b) => b[1].size - a[1].size)
          .slice(0, limit);
      },
      
      getOldestItems(limit = 5) {
        return Array.from(usageStats.entries())
          .sort((a, b) => a[1].lastAccess - b[1].lastAccess)
          .slice(0, limit);
      }
    };
  }
}

// Пример использования оптимизированного хранилища
class OptimizedCache {
  constructor() {
    this.ttlStorage = StorageOptimization.createTTLStorage(localStorage, 'app_cache_');
    this.compressedStorage = StorageOptimization.createCompressedStorage(localStorage, 'app_compressed_');
    this.batchStorage = StorageOptimization.createBatchStorage(localStorage);
    this.monitor = StorageOptimization.createStorageMonitor(localStorage);
  }
  
  // Кэширование данных с TTL
  cacheData(key, data, ttl = 5 * 60 * 1000) {
    this.ttlStorage.set(key, data, ttl);
  }
  
  // Получение кэшированных данных
  getCachedData(key) {
    return this.ttlStorage.get(key);
  }
  
  // Сохранение больших данных с компрессией
  saveLargeData(key, data) {
    this.compressedStorage.set(key, data);
  }
  
  // Получение сжатых данных
  getLargeData(key) {
    return this.compressedStorage.get(key);
  }
  
  // Пакетное сохранение настроек
  saveSettings(settings) {
    Object.entries(settings).forEach(([key, value]) => {
      this.batchStorage.set(`setting_${key}`, value);
    });
  }
  
  // Получение статистики использования
  getStorageStats() {
    return this.monitor.getStats();
  }
}
```

## Оптимизация IndexedDB

Эффективное использование IndexedDB для больших объемов данных:

```javascript
// Оптимизация IndexedDB
class IndexedDBOptimization {
  constructor(dbName = 'AppDB', version = 1) {
    this.dbName = dbName;
    this.version = version;
    this.db = null;
  }
  
  // Инициализация базы данных
  async init(stores = []) {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        stores.forEach(storeConfig => {
          if (!db.objectStoreNames.contains(storeConfig.name)) {
            const store = db.createObjectStore(storeConfig.name, storeConfig.options);
            
            // Создаем индексы
            if (storeConfig.indexes) {
              storeConfig.indexes.forEach(index => {
                store.createIndex(index.name, index.keyPath, index.options);
              });
            }
          }
        });
      };
    });
  }
  
  // Батчинг операций
  async batchOperations(operations) {
    const transaction = this.db.transaction(
      operations.map(op => op.store),
      'readwrite'
    );
    
    const promises = operations.map(op => {
      return new Promise((resolve, reject) => {
        const store = transaction.objectStore(op.store);
        let request;
        
        switch (op.type) {
          case 'add':
            request = store.add(op.data, op.key);
            break;
          case 'put':
            request = store.put(op.data, op.key);
            break;
          case 'delete':
            request = store.delete(op.key);
            break;
          default:
            reject(new Error(`Неизвестный тип операции: ${op.type}`));
            return;
        }
        
        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
      });
    });
    
    transaction.oncomplete = () => console.log('Батч операций завершен');
    transaction.onerror = (event) => console.error('Ошибка батч операций:', event.target.error);
    
    return Promise.all(promises);
  }
  
  // Курсор с пагинацией
  async cursorWithPagination(storeName, pageSize = 100, processBatch) {
    const transaction = this.db.transaction(storeName, 'readonly');
    const store = transaction.objectStore(storeName);
    const request = store.openCursor();
    
    let batch = [];
    let count = 0;
    
    return new Promise((resolve, reject) => {
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          batch.push(cursor.value);
          count++;
          
          if (batch.length >= pageSize) {
            // Обрабатываем батч
            processBatch(batch);
            batch = [];
          }
          
          cursor.continue();
        } else {
          // Обрабатываем последний батч
          if (batch.length > 0) {
            processBatch(batch);
          }
          resolve(count);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Индексированный поиск
  async indexedSearch(storeName, indexName, query) {
    const transaction = this.db.transaction(storeName, 'readonly');
    const store = transaction.objectStore(storeName);
    const index = store.index(indexName);
    
    return new Promise((resolve, reject) => {
      const request = index.getAll(query);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Кэширование запросов
  createQueryCache() {
    const cache = new Map();
    const ttl = 5 * 60 * 1000; // 5 минут
    
    return {
      async get(key, fetcher) {
        const cached = cache.get(key);
        const now = Date.now();
        
        if (cached && (now - cached.timestamp) < ttl) {
          return cached.data;
        }
        
        try {
          const data = await fetcher();
          cache.set(key, {
            data,
            timestamp: now
          });
          return data;
        } catch (error) {
          // В случае ошибки возвращаем кэшированные данные, если есть
          if (cached) {
            console.warn('Использование кэшированных данных из-за ошибки:', error);
            return cached.data;
          }
          throw error;
        }
      },
      
      invalidate(key) {
        cache.delete(key);
      },
      
      clear() {
        cache.clear();
      }
    };
  }
  
  // Оптимизация производительности
  async optimizePerformance() {
    // Компактификация базы данных
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName);
      
      request.onsuccess = () => {
        const db = request.result;
        db.close();
        
        // Закрываем текущее соединение
        if (this.db) {
          this.db.close();
        }
        
        // Открываем новое соединение с увеличенной версией
        const newVersion = this.version + 1;
        const newRequest = indexedDB.open(this.dbName, newVersion);
        
        newRequest.onsuccess = () => {
          this.db = newRequest.result;
          this.version = newVersion;
          resolve();
        };
        
        newRequest.onerror = () => reject(newRequest.error);
        
        newRequest.onupgradeneeded = (event) => {
          // Никаких изменений схемы, просто компактификация
          console.log('Компактификация базы данных завершена');
        };
      };
      
      request.onerror = () => reject(request.error);
    });
  }
}

// Пример использования оптимизированного IndexedDB
class DataManager {
  constructor() {
    this.db = new IndexedDBOptimization('AppData', 1);
    this.queryCache = this.db.createQueryCache();
  }
  
  async init() {
    await this.db.init([
      {
        name: 'users',
        options: { keyPath: 'id' },
        indexes: [
          { name: 'byName', keyPath: 'name' },
          { name: 'byEmail', keyPath: 'email' }
        ]
      },
      {
        name: 'posts',
        options: { keyPath: 'id' },
        indexes: [
          { name: 'byUserId', keyPath: 'userId' },
          { name: 'byDate', keyPath: 'date' }
        ]
      }
    ]);
  }
  
  // Батч добавление пользователей
  async addUsersBatch(users) {
    const operations = users.map(user => ({
      type: 'add',
      store: 'users',
      data: user
    }));
    
    return await this.db.batchOperations(operations);
  }
  
  // Пагинация пользователей
  async getUsersPaginated(pageSize, processBatch) {
    return await this.db.cursorWithPagination('users', pageSize, processBatch);
  }
  
  // Поиск пользователей по имени
  async searchUsersByName(name) {
    return await this.db.indexedSearch('users', 'byName', name);
  }
  
  // Кэшированный запрос пользователей
  async getCachedUsers() {
    return await this.queryCache.get('all_users', async () => {
      const transaction = this.db.db.transaction('users', 'readonly');
      const store = transaction.objectStore('users');
      
      return new Promise((resolve, reject) => {
        const request = store.getAll();
        request.onsuccess = () => resolve(request.result);
        request.onerror = () => reject(request.error);
      });
    });
  }
}
```

## Гибридные хранилища

Комбинирование разных типов хранилищ для оптимальной производительности:

```javascript
// Гибридные хранилища
class HybridStorage {
  constructor(options = {}) {
    this.memoryCache = new Map();
    this.localStorage = options.localStorage || localStorage;
    this.sessionStorage = options.sessionStorage || sessionStorage;
    this.indexedDB = options.indexedDB;
    
    this.memoryMaxSize = options.memoryMaxSize || 100;
    this.defaultTTL = options.ttl || 5 * 60 * 1000;
    this.prefix = options.prefix || 'hybrid_';
  }
  
  // Установка значения
  async set(key, value, options = {}) {
    const ttl = options.ttl || this.defaultTTL;
    const storageType = options.storage || 'memory';
    const expiration = Date.now() + ttl;
    
    const item = {
      value,
      expiration,
      storageType
    };
    
    // Сохраняем в памяти
    if (storageType === 'memory') {
      if (this.memoryCache.size >= this.memoryMaxSize) {
        const firstKey = this.memoryCache.keys().next().value;
        this.memoryCache.delete(firstKey);
      }
      this.memoryCache.set(key, item);
    }
    
    // Сохраняем в localStorage
    if (storageType === 'local' || options.persist) {
      try {
        this.localStorage.setItem(
          this.prefix + key,
          JSON.stringify({ value, expiration })
        );
      } catch (error) {
        console.error('Ошибка при сохранении в localStorage:', error);
      }
    }
    
    // Сохраняем в IndexedDB для больших данных
    if (storageType === 'indexeddb' && this.indexedDB) {
      try {
        const transaction = this.indexedDB.transaction('cache', 'readwrite');
        const store = transaction.objectStore('cache');
        await new Promise((resolve, reject) => {
          const request = store.put({ key, value, expiration });
          request.onsuccess = () => resolve();
          request.onerror = () => reject(request.error);
        });
      } catch (error) {
        console.error('Ошибка при сохранении в IndexedDB:', error);
      }
    }
  }
  
  // Получение значения
  async get(key) {
    // Сначала проверяем память
    const memoryItem = this.memoryCache.get(key);
    if (memoryItem) {
      if (Date.now() <= memoryItem.expiration) {
        return memoryItem.value;
      } else {
        this.memoryCache.delete(key);
      }
    }
    
    // Затем проверяем localStorage
    try {
      const itemStr = this.localStorage.getItem(this.prefix + key);
      if (itemStr) {
        const item = JSON.parse(itemStr);
        if (Date.now() <= item.expiration) {
          // Перемещаем в память для ускорения следующего доступа
          if (this.memoryCache.size >= this.memoryMaxSize) {
            const firstKey = this.memoryCache.keys().next().value;
            this.memoryCache.delete(firstKey);
          }
          this.memoryCache.set(key, {
            value: item.value,
            expiration: item.expiration,
            storageType: 'memory'
          });
          return item.value;
        } else {
          this.localStorage.removeItem(this.prefix + key);
        }
      }
    } catch (error) {
      console.error('Ошибка при получении из localStorage:', error);
    }
    
    // Наконец проверяем IndexedDB
    if (this.indexedDB) {
      try {
        const transaction = this.indexedDB.transaction('cache', 'readonly');
        const store = transaction.objectStore('cache');
        const result = await new Promise((resolve, reject) => {
          const request = store.get(key);
          request.onsuccess = () => resolve(request.result);
          request.onerror = () => reject(request.error);
        });
        
        if (result && Date.now() <= result.expiration) {
          // Перемещаем в память
          if (this.memoryCache.size >= this.memoryMaxSize) {
            const firstKey = this.memoryCache.keys().next().value;
            this.memoryCache.delete(firstKey);
          }
          this.memoryCache.set(key, {
            value: result.value,
            expiration: result.expiration,
            storageType: 'memory'
          });
          return result.value;
        }
      } catch (error) {
        console.error('Ошибка при получении из IndexedDB:', error);
      }
    }
    
    return undefined;
  }
  
  // Удаление значения
  async delete(key) {
    this.memoryCache.delete(key);
    
    try {
      this.localStorage.removeItem(this.prefix + key);
    } catch (error) {
      console.error('Ошибка при удалении из localStorage:', error);
    }
    
    if (this.indexedDB) {
      try {
        const transaction = this.indexedDB.transaction('cache', 'readwrite');
        const store = transaction.objectStore('cache');
        await new Promise((resolve, reject) => {
          const request = store.delete(key);
          request.onsuccess = () => resolve();
          request.onerror = () => reject(request.error);
        });
      } catch (error) {
        console.error('Ошибка при удалении из IndexedDB:', error);
      }
    }
  }
  
  // Очистка устаревших записей
  async clearExpired() {
    const now = Date.now();
    let cleared = 0;
    
    // Очистка памяти
    for (const [key, item] of this.memoryCache) {
      if (now > item.expiration) {
        this.memoryCache.delete(key);
        cleared++;
      }
    }
    
    // Очистка localStorage
    for (let i = 0; i < this.localStorage.length; i++) {
      const key = this.localStorage.key(i);
      if (key && key.startsWith(this.prefix)) {
        try {
          const itemStr = this.localStorage.getItem(key);
          if (itemStr) {
            const item = JSON.parse(itemStr);
            if (now > item.expiration) {
              this.localStorage.removeItem(key);
              cleared++;
              i--; // Корректируем индекс
            }
          }
        } catch (error) {
          console.error('Ошибка при очистке localStorage:', error);
        }
      }
    }
    
    // Очистка IndexedDB
    if (this.indexedDB) {
      try {
        const transaction = this.indexedDB.transaction('cache', 'readwrite');
        const store = transaction.objectStore('cache');
        const request = store.openCursor();
        
        request.onsuccess = (event) => {
          const cursor = event.target.result;
          if (cursor) {
            if (now > cursor.value.expiration) {
              cursor.delete();
              cleared++;
            }
            cursor.continue();
          }
        };
      } catch (error) {
        console.error('Ошибка при очистке IndexedDB:', error);
      }
    }
    
    return cleared;
  }
  
  // Получение статистики
  async getStats() {
    let localStorageSize = 0;
    let localStorageItems = 0;
    
    for (let i = 0; i < this.localStorage.length; i++) {
      const key = this.localStorage.key(i);
      if (key && key.startsWith(this.prefix)) {
        const value = this.localStorage.getItem(key);
        localStorageSize += key.length + (value ? value.length : 0);
        localStorageItems++;
      }
    }
    
    let indexedDBSize = 0;
    let indexedDBItems = 0;
    
    if (this.indexedDB) {
      try {
        const transaction = this.indexedDB.transaction('cache', 'readonly');
        const store = transaction.objectStore('cache');
        const request = store.count();
        
        indexedDBItems = await new Promise((resolve, reject) => {
          request.onsuccess = () => resolve(request.result);
          request.onerror = () => reject(request.error);
        });
        
        // Приблизительный размер (в реальности нужно суммировать все записи)
        indexedDBSize = indexedDBItems * 1024; // Примерно 1KB на запись
      } catch (error) {
        console.error('Ошибка при получении статистики IndexedDB:', error);
      }
    }
    
    return {
      memory: {
        size: this.memoryCache.size,
        maxSize: this.memoryMaxSize
      },
      localStorage: {
        size: localStorageSize,
        items: localStorageItems
      },
      indexedDB: {
        size: indexedDBSize,
        items: indexedDBItems
      }
    };
  }
}

// Пример использования гибридного хранилища
class ApplicationStorage {
  constructor() {
    this.hybridStorage = new HybridStorage({
      memoryMaxSize: 50,
      ttl: 10 * 60 * 1000, // 10 минут
      prefix: 'app_'
    });
  }
  
  // Сохранение пользовательских настроек в localStorage
  async saveUserSettings(settings) {
    await this.hybridStorage.set('user_settings', settings, {
      storage: 'local',
      ttl: 24 * 60 * 60 * 1000 // 24 часа
    });
  }
  
  // Получение пользовательских настроек
  async getUserSettings() {
    return await this.hybridStorage.get('user_settings');
  }
  
  // Кэширование API данных в памяти
  async cacheAPIData(key, data) {
    await this.hybridStorage.set(key, data, {
      storage: 'memory',
      ttl: 5 * 60 * 1000 // 5 минут
    });
  }
  
  // Получение кэшированных данных
  async getCachedData(key) {
    return await this.hybridStorage.get(key);
  }
  
  // Сохранение больших данных в IndexedDB
  async saveLargeData(key, data) {
    if (this.hybridStorage.indexedDB) {
      await this.hybridStorage.set(key, data, {
        storage: 'indexeddb',
        ttl: 60 * 60 * 1000 // 1 час
      });
    } else {
      // Fallback на localStorage с компрессией
      await this.hybridStorage.set(key, data, {
        storage: 'local',
        ttl: 30 * 60 * 1000 // 30 минут
      });
    }
  }
  
  // Очистка устаревших данных
  async cleanup() {
    const cleared = await this.hybridStorage.clearExpired();
    console.log(`Очищено ${cleared} устаревших записей`);
  }
  
  // Получение статистики использования
  async getStorageStats() {
    return await this.hybridStorage.getStats();
  }
}
```

## Практические рекомендации

1. **Используйте TTL** - для автоматической очистки устаревших данных
2. **Применяйте компрессию** - для больших объемов данных
3. **Используйте батчинг** - для групповых операций
4. **Реализуйте гибридные хранилища** - комбинируя память, localStorage и IndexedDB
5. **Мониторьте использование** - отслеживайте размер и частоту доступа
6. **Обрабатывайте ошибки** - особенно переполнение хранилищ
7. **Используйте индексирование** - в IndexedDB для быстрого поиска
8. **Очищайте устаревшие данные** - регулярно для предотвращения переполнения

Оптимизация хранилищ - важный аспект создания производительных веб-приложений. Правильное использование различных типов хранилищ и техник оптимизации может значительно улучшить пользовательский опыт и производительность приложения.

#javascript #storage #optimization #performance #localstorage #sessionstorage #indexeddb #caching