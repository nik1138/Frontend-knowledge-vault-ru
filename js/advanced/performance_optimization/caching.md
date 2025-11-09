# Кэширование в JavaScript

## Введение

Кэширование - одна из самых эффективных техник оптимизации производительности в JavaScript. Правильное использование кэширования может значительно ускорить выполнение кода, уменьшить нагрузку на сервер и улучшить пользовательский опыт. В этом разделе мы рассмотрим различные виды кэширования и техники их реализации.

## Типы кэширования

### 1. Кэширование в памяти

Кэширование данных в оперативной памяти:

```javascript
// Кэширование в памяти
class MemoryCache {
  constructor(options = {}) {
    this.cache = new Map();
    this.maxSize = options.maxSize || 1000;
    this.defaultTTL = options.ttl || 5 * 60 * 1000; // 5 минут
    this.stats = {
      hits: 0,
      misses: 0,
      evictions: 0
    };
  }
  
  // Установка значения в кэш
  set(key, value, ttl = this.defaultTTL) {
    // Проверяем размер кэша и удаляем старые записи при необходимости
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      this.stats.evictions++;
    }
    
    const expiration = Date.now() + ttl;
    this.cache.set(key, {
      value,
      expiration
    });
  }
  
  // Получение значения из кэша
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      this.stats.misses++;
      return undefined;
    }
    
    // Проверяем срок действия
    if (Date.now() > item.expiration) {
      this.cache.delete(key);
      this.stats.misses++;
      return undefined;
    }
    
    this.stats.hits++;
    return item.value;
  }
  
  // Удаление значения из кэша
  delete(key) {
    return this.cache.delete(key);
  }
  
  // Очистка кэша
  clear() {
    this.cache.clear();
    this.stats.hits = 0;
    this.stats.misses = 0;
    this.stats.evictions = 0;
  }
  
  // Проверка наличия ключа
  has(key) {
    const item = this.cache.get(key);
    if (!item) return false;
    
    if (Date.now() > item.expiration) {
      this.cache.delete(key);
      return false;
    }
    
    return true;
  }
  
  // Получение статистики
  getStats() {
    const totalRequests = this.stats.hits + this.stats.misses;
    const hitRate = totalRequests > 0 ? (this.stats.hits / totalRequests) * 100 : 0;
    
    return {
      ...this.stats,
      hitRate: hitRate.toFixed(2) + '%',
      size: this.cache.size,
      maxSize: this.maxSize
    };
  }
  
  // LRU (Least Recently Used) реализация
  setLRU(key, value, ttl = this.defaultTTL) {
    // Удаляем ключ, если он уже существует
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Проверяем размер кэша
    if (this.cache.size >= this.maxSize) {
      // Удаляем самый старый элемент
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      this.stats.evictions++;
    }
    
    const expiration = Date.now() + ttl;
    this.cache.set(key, {
      value,
      expiration
    });
  }
}

// Пример использования кэша в памяти
class DataProcessor {
  constructor() {
    this.cache = new MemoryCache({ maxSize: 100, ttl: 10 * 60 * 1000 }); // 10 минут
  }
  
  async processData(dataId) {
    // Проверяем кэш
    const cached = this.cache.get(dataId);
    if (cached) {
      console.log(`Данные ${dataId} получены из кэша`);
      return cached;
    }
    
    // Загружаем данные (имитация)
    console.log(`Загрузка данных ${dataId}...`);
    const data = await this.fetchData(dataId);
    
    // Сохраняем в кэш
    this.cache.set(dataId, data);
    
    return data;
  }
  
  async fetchData(dataId) {
    // Имитация асинхронной загрузки данных
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          id: dataId,
          value: Math.random(),
          timestamp: Date.now()
        });
      }, 100);
    });
  }
  
  getCacheStats() {
    return this.cache.getStats();
  }
}
```

### 2. Кэширование HTTP запросов

Кэширование сетевых запросов:

```javascript
// Кэширование HTTP запросов
class HTTPCache {
  constructor() {
    this.cache = new Map();
    this.defaultTTL = 5 * 60 * 1000; // 5 минут
  }
  
  // Кэшированный fetch
  async cachedFetch(url, options = {}) {
    const cacheKey = this.generateCacheKey(url, options);
    const cached = this.getFromCache(cacheKey);
    
    if (cached) {
      return cached;
    }
    
    try {
      const response = await fetch(url, options);
      const data = await response.json();
      
      // Сохраняем в кэш
      this.setInCache(cacheKey, data, this.getTTLFromResponse(response));
      
      return data;
    } catch (error) {
      // В случае ошибки возвращаем кэшированные данные, если есть
      const fallback = this.getFromCache(cacheKey, true); // Игнорируем TTL
      if (fallback) {
        console.warn(`Использование кэшированных данных из-за ошибки: ${error.message}`);
        return fallback;
      }
      throw error;
    }
  }
  
  // Генерация ключа кэша
  generateCacheKey(url, options) {
    const method = options.method || 'GET';
    const body = options.body ? JSON.stringify(options.body) : '';
    return `${method}:${url}:${body}`;
  }
  
  // Получение TTL из заголовков ответа
  getTTLFromResponse(response) {
    const cacheControl = response.headers.get('Cache-Control');
    if (cacheControl) {
      const maxAgeMatch = cacheControl.match(/max-age=(\d+)/);
      if (maxAgeMatch) {
        return parseInt(maxAgeMatch[1]) * 1000; // Преобразуем в миллисекунды
      }
    }
    
    const expires = response.headers.get('Expires');
    if (expires) {
      const expireTime = new Date(expires).getTime();
      const now = Date.now();
      if (expireTime > now) {
        return expireTime - now;
      }
    }
    
    return this.defaultTTL;
  }
  
  // Получение из кэша
  getFromCache(key, ignoreTTL = false) {
    const item = this.cache.get(key);
    if (!item) return undefined;
    
    if (!ignoreTTL && Date.now() > item.expiration) {
      this.cache.delete(key);
      return undefined;
    }
    
    return item.data;
  }
  
  // Сохранение в кэш
  setInCache(key, data, ttl = this.defaultTTL) {
    const expiration = Date.now() + ttl;
    this.cache.set(key, {
      data,
      expiration
    });
  }
  
  // Инвалидация кэша
  invalidate(pattern) {
    if (!pattern) {
      this.cache.clear();
      return;
    }
    
    for (const key of this.cache.keys()) {
      if (key.includes(pattern)) {
        this.cache.delete(key);
      }
    }
  }
  
  // Получение статистики
  getStats() {
    return {
      size: this.cache.size,
      keys: Array.from(this.cache.keys())
    };
  }
}

// Расширенное кэширование HTTP с поддержкой стратегий
class AdvancedHTTPCache {
  constructor() {
    this.cache = new Map();
    this.staleWhileRevalidate = new Map();
  }
  
  // Стратегия Cache First
  async cacheFirst(url, options = {}) {
    const cacheKey = this.generateCacheKey(url, options);
    const cached = this.getFromCache(cacheKey);
    
    if (cached) {
      return cached;
    }
    
    // Загружаем из сети
    const data = await this.fetchFromNetwork(url, options, cacheKey);
    return data;
  }
  
  // Стратегия Network First
  async networkFirst(url, options = {}) {
    const cacheKey = this.generateCacheKey(url, options);
    
    try {
      const data = await this.fetchFromNetwork(url, options, cacheKey);
      return data;
    } catch (error) {
      // В случае ошибки возвращаем кэшированные данные
      const cached = this.getFromCache(cacheKey);
      if (cached) {
        console.warn('Использование кэшированных данных из-за ошибки сети');
        return cached;
      }
      throw error;
    }
  }
  
  // Стратегия Stale While Revalidate
  async staleWhileRevalidate(url, options = {}) {
    const cacheKey = this.generateCacheKey(url, options);
    const cached = this.getFromCache(cacheKey);
    
    // Возвращаем кэшированные данные немедленно
    if (cached) {
      // Асинхронно обновляем кэш
      this.updateCacheInBackground(url, options, cacheKey);
      return cached;
    }
    
    // Если нет кэша, загружаем из сети
    const data = await this.fetchFromNetwork(url, options, cacheKey);
    return data;
  }
  
  // Вспомогательные методы
  generateCacheKey(url, options) {
    const method = options.method || 'GET';
    const body = options.body ? JSON.stringify(options.body) : '';
    return `${method}:${url}:${body}`;
  }
  
  getFromCache(key) {
    const item = this.cache.get(key);
    if (!item) return undefined;
    
    if (Date.now() > item.expiration) {
      this.cache.delete(key);
      return undefined;
    }
    
    return item.data;
  }
  
  async fetchFromNetwork(url, options, cacheKey) {
    const response = await fetch(url, options);
    const data = await response.json();
    
    const ttl = this.getTTLFromResponse(response);
    const expiration = Date.now() + ttl;
    
    this.cache.set(cacheKey, {
      data,
      expiration
    });
    
    return data;
  }
  
  getTTLFromResponse(response) {
    const cacheControl = response.headers.get('Cache-Control');
    if (cacheControl) {
      const maxAgeMatch = cacheControl.match(/max-age=(\d+)/);
      if (maxAgeMatch) {
        return parseInt(maxAgeMatch[1]) * 1000;
      }
    }
    return 5 * 60 * 1000; // 5 минут по умолчанию
  }
  
  async updateCacheInBackground(url, options, cacheKey) {
    // Предотвращаем множественные обновления
    if (this.staleWhileRevalidate.has(cacheKey)) {
      return;
    }
    
    this.staleWhileRevalidate.set(cacheKey, true);
    
    try {
      const response = await fetch(url, options);
      const data = await response.json();
      
      const ttl = this.getTTLFromResponse(response);
      const expiration = Date.now() + ttl;
      
      this.cache.set(cacheKey, {
        data,
        expiration
      });
    } catch (error) {
      console.error('Ошибка при фоновом обновлении кэша:', error);
    } finally {
      this.staleWhileRevalidate.delete(cacheKey);
    }
  }
}
```

### 3. Кэширование с использованием Storage API

Кэширование в localStorage и sessionStorage:

```javascript
// Кэширование с использованием Storage API
class StorageCache {
  constructor(storage = localStorage, prefix = 'cache_') {
    this.storage = storage;
    this.prefix = prefix;
  }
  
  // Установка значения
  set(key, value, ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
    const expiration = Date.now() + ttl;
    const item = {
      value,
      expiration
    };
    
    try {
      this.storage.setItem(this.prefix + key, JSON.stringify(item));
    } catch (error) {
      console.error('Ошибка при сохранении в хранилище:', error);
    }
  }
  
  // Получение значения
  get(key) {
    try {
      const itemStr = this.storage.getItem(this.prefix + key);
      if (!itemStr) return undefined;
      
      const item = JSON.parse(itemStr);
      
      // Проверяем срок действия
      if (Date.now() > item.expiration) {
        this.storage.removeItem(this.prefix + key);
        return undefined;
      }
      
      return item.value;
    } catch (error) {
      console.error('Ошибка при получении из хранилища:', error);
      return undefined;
    }
  }
  
  // Удаление значения
  delete(key) {
    try {
      this.storage.removeItem(this.prefix + key);
      return true;
    } catch (error) {
      console.error('Ошибка при удалении из хранилища:', error);
      return false;
    }
  }
  
  // Очистка устаревших записей
  clearExpired() {
    const now = Date.now();
    let cleared = 0;
    
    for (let i = 0; i < this.storage.length; i++) {
      const key = this.storage.key(i);
      if (key && key.startsWith(this.prefix)) {
        try {
          const itemStr = this.storage.getItem(key);
          if (itemStr) {
            const item = JSON.parse(itemStr);
            if (now > item.expiration) {
              this.storage.removeItem(key);
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
  }
  
  // Получение всех ключей
  getKeys() {
    const keys = [];
    for (let i = 0; i < this.storage.length; i++) {
      const key = this.storage.key(i);
      if (key && key.startsWith(this.prefix)) {
        keys.push(key.substring(this.prefix.length));
      }
    }
    return keys;
  }
  
  // Очистка всего кэша
  clear() {
    const keys = this.getKeys();
    keys.forEach(key => {
      this.storage.removeItem(this.prefix + key);
    });
  }
}

// Гибридный кэш (память + хранилище)
class HybridCache {
  constructor(options = {}) {
    this.memoryCache = new Map();
    this.storageCache = new StorageCache(localStorage, options.prefix || 'hybrid_cache_');
    this.memoryMaxSize = options.memoryMaxSize || 100;
    this.defaultTTL = options.ttl || 5 * 60 * 1000;
  }
  
  // Установка значения
  set(key, value, ttl = this.defaultTTL) {
    // Сохраняем в памяти
    if (this.memoryCache.size >= this.memoryMaxSize) {
      const firstKey = this.memoryCache.keys().next().value;
      this.memoryCache.delete(firstKey);
    }
    
    const expiration = Date.now() + ttl;
    this.memoryCache.set(key, {
      value,
      expiration
    });
    
    // Сохраняем в хранилище
    this.storageCache.set(key, value, ttl);
  }
  
  // Получение значения
  get(key) {
    // Сначала проверяем память
    const memoryItem = this.memoryCache.get(key);
    if (memoryItem) {
      if (Date.now() <= memoryItem.expiration) {
        return memoryItem.value;
      } else {
        this.memoryCache.delete(key);
      }
    }
    
    // Затем проверяем хранилище
    const storageValue = this.storageCache.get(key);
    if (storageValue !== undefined) {
      // Перемещаем в память для ускорения следующего доступа
      const expiration = Date.now() + this.defaultTTL;
      if (this.memoryCache.size >= this.memoryMaxSize) {
        const firstKey = this.memoryCache.keys().next().value;
        this.memoryCache.delete(firstKey);
      }
      this.memoryCache.set(key, {
        value: storageValue,
        expiration
      });
      
      return storageValue;
    }
    
    return undefined;
  }
  
  // Удаление значения
  delete(key) {
    this.memoryCache.delete(key);
    this.storageCache.delete(key);
  }
  
  // Очистка
  clear() {
    this.memoryCache.clear();
    this.storageCache.clear();
  }
}
```

### 4. Кэширование функций (мемоизация)

Кэширование результатов функций:

```javascript
// Мемоизация функций
class Memoization {
  // Простая мемоизация
  static memoize(fn, resolver) {
    const cache = new Map();
    
    return function(...args) {
      const key = resolver ? resolver(...args) : JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn.apply(this, args);
      cache.set(key, result);
      return result;
    };
  }
  
  // Мемоизация с TTL
  static memoizeWithTTL(fn, ttl = 5 * 60 * 1000, resolver) {
    const cache = new Map();
    
    return function(...args) {
      const key = resolver ? resolver(...args) : JSON.stringify(args);
      const item = cache.get(key);
      
      if (item && Date.now() - item.timestamp < ttl) {
        return item.value;
      }
      
      const result = fn.apply(this, args);
      cache.set(key, {
        value: result,
        timestamp: Date.now()
      });
      
      return result;
    };
  }
  
  // Мемоизация с ограничением размера
  static memoizeWithLimit(fn, limit = 100, resolver) {
    const cache = new Map();
    
    return function(...args) {
      const key = resolver ? resolver(...args) : JSON.stringify(args);
      
      if (cache.has(key)) {
        // Перемещаем элемент в конец (LRU)
        const value = cache.get(key);
        cache.delete(key);
        cache.set(key, value);
        return value;
      }
      
      const result = fn.apply(this, args);
      
      // Ограничиваем размер кэша
      if (cache.size >= limit) {
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
      }
      
      cache.set(key, result);
      return result;
    };
  }
  
  // Асинхронная мемоизация
  static asyncMemoize(fn, resolver) {
    const cache = new Map();
    const promises = new Map();
    
    return async function(...args) {
      const key = resolver ? resolver(...args) : JSON.stringify(args);
      
      // Проверяем, есть ли уже выполняющийся промис
      if (promises.has(key)) {
        return await promises.get(key);
      }
      
      // Проверяем кэш
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      // Создаем новый промис
      const promise = fn.apply(this, args).then(result => {
        cache.set(key, result);
        promises.delete(key);
        return result;
      }).catch(error => {
        promises.delete(key);
        throw error;
      });
      
      promises.set(key, promise);
      return await promise;
    };
  }
}

// Примеры использования мемоизации
class MathUtils {
  // Медленная функция для демонстрации
  static expensiveCalculation(n) {
    console.log(`Вычисление для ${n}...`);
    
    // Имитация тяжелых вычислений
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.sin(n + i) * Math.cos(n - i);
    }
    
    return result;
  }
  
  // Мемоизированная версия
  static memoizedCalculation = Memoization.memoize(
    MathUtils.expensiveCalculation
  );
  
  // Асинхронная функция
  static async fetchUserData(userId) {
    console.log(`Загрузка данных пользователя ${userId}...`);
    
    // Имитация асинхронного запроса
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          id: userId,
          name: `User ${userId}`,
          email: `user${userId}@example.com`
        });
      }, 100);
    });
  }
  
  // Мемоизированная асинхронная функция
  static memoizedFetchUserData = Memoization.asyncMemoize(
    MathUtils.fetchUserData,
    (userId) => `user_${userId}` // Кастомный резолвер
  );
}

// Декоратор для мемоизации
function memoize(ttl, resolver) {
  return function(target, propertyName, descriptor) {
    const method = descriptor.value;
    const memoizedMethod = Memoization.memoizeWithTTL(method, ttl, resolver);
    
    descriptor.value = function(...args) {
      return memoizedMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

// Пример использования декоратора
class DataService {
  @memoize(10 * 60 * 1000) // 10 минут
  getUserData(userId) {
    console.log(`Загрузка данных пользователя ${userId}`);
    // Имитация загрузки данных
    return { id: userId, name: `User ${userId}` };
  }
  
  @memoize(5 * 60 * 1000, (userId, postId) => `${userId}_${postId}`)
  getUserPost(userId, postId) {
    console.log(`Загрузка поста ${postId} пользователя ${userId}`);
    return { userId, postId, content: `Content of post ${postId}` };
  }
}
```

## Практические рекомендации

1. **Выбирайте правильный тип кэша** - память, хранилище или HTTP кэш
2. **Устанавливайте разумные TTL** - чтобы данные не устаревали
3. **Ограничивайте размер кэша** - чтобы не переполнять память
4. **Используйте LRU алгоритм** - для эффективного управления памятью
5. **Реализуйте стратегии кэширования** - Cache First, Network First, Stale While Revalidate
6. **Мониторьте эффективность кэша** - с помощью статистики попаданий
7. **Обрабатывайте ошибки корректно** - используйте кэш как fallback
8. **Инвалидируйте кэш при необходимости** - при обновлении данных

Кэширование - мощный инструмент для оптимизации производительности JavaScript приложений. Правильное использование различных видов кэширования может значительно улучшить скорость работы приложения и пользовательский опыт.

#javascript #caching #performance #optimization #memoization #http-cache #storage-api