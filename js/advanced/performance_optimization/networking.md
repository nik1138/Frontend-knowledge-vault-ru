# Оптимизация сетевых операций в JavaScript

## Введение

Сетевые операции часто являются узким местом в веб-приложениях. Оптимизация сетевых запросов может значительно улучшить пользовательский опыт и общую производительность приложения. В этом разделе мы рассмотрим различные техники оптимизации сетевых операций в JavaScript.

## Оптимизация HTTP запросов

Эффективное использование HTTP протокола:

```javascript
// Оптимизация HTTP запросов
class HTTPOptimization {
  // Кэширование запросов
  static createCachedFetcher() {
    const cache = new Map();
    const cacheTimeout = 5 * 60 * 1000; // 5 минут
    
    return async function cachedFetch(url, options = {}) {
      const cacheKey = `${url}_${JSON.stringify(options)}`;
      const cached = cache.get(cacheKey);
      
      // Проверяем кэш
      if (cached && Date.now() - cached.timestamp < cacheTimeout) {
        return cached.response;
      }
      
      try {
        const response = await fetch(url, options);
        const data = await response.json();
        
        // Сохраняем в кэш
        cache.set(cacheKey, {
          response: data,
          timestamp: Date.now()
        });
        
        return data;
      } catch (error) {
        // В случае ошибки возвращаем кэшированные данные, если есть
        if (cached) {
          console.warn('Использование кэшированных данных из-за ошибки сети:', error);
          return cached.response;
        }
        throw error;
      }
    };
  }
  
  // Пакетирование запросов
  static createBatchFetcher(batchSize = 5, delay = 100) {
    const queue = [];
    let timeoutId = null;
    
    async function processBatch() {
      if (queue.length === 0) return;
      
      const batch = queue.splice(0, batchSize);
      const promises = batch.map(({ url, options, resolve, reject }) => 
        fetch(url, options).then(resolve).catch(reject)
      );
      
      try {
        await Promise.all(promises);
      } catch (error) {
        console.error('Ошибка при обработке пакета:', error);
      }
    }
    
    return function batchFetch(url, options = {}) {
      return new Promise((resolve, reject) => {
        queue.push({ url, options, resolve, reject });
        
        if (!timeoutId) {
          timeoutId = setTimeout(() => {
            processBatch();
            timeoutId = null;
          }, delay);
        }
      });
    };
  }
  
  // Retry механизм с экспоненциальной задержкой
  static async fetchWithRetry(url, options = {}, maxRetries = 3) {
    for (let i = 0; i <= maxRetries; i++) {
      try {
        const response = await fetch(url, options);
        
        // Проверяем статус ответа
        if (response.ok) {
          return await response.json();
        }
        
        // Для 4xx ошибок не пытаемся повторить
        if (response.status >= 400 && response.status < 500) {
          throw new Error(`Client error: ${response.status}`);
        }
        
        // Для остальных ошибок пробуем повторить
        if (i === maxRetries) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
      } catch (error) {
        if (i === maxRetries) {
          throw error;
        }
        
        // Экспоненциальная задержка
        const delay = Math.pow(2, i) * 1000;
        console.log(`Повторная попытка через ${delay} мс...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  // Таймаут для запросов
  static async fetchWithTimeout(url, options = {}, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      return await response.json();
    } catch (error) {
      clearTimeout(timeoutId);
      
      if (error.name === 'AbortError') {
        throw new Error('Запрос превысил таймаут');
      }
      
      throw error;
    }
  }
}
```

## Оптимизация загрузки ресурсов

Эффективная загрузка и кэширование ресурсов:

```javascript
// Оптимизация загрузки ресурсов
class ResourceOptimization {
  // Предзагрузка ресурсов
  static preloadResource(url, asType) {
    const link = document.createElement('link');
    link.rel = 'preload';
    link.href = url;
    link.as = asType;
    document.head.appendChild(link);
    
    return new Promise((resolve, reject) => {
      link.onload = () => resolve(link);
      link.onerror = () => reject(new Error(`Не удалось загрузить ${url}`));
    });
  }
  
  // Ленивая загрузка изображений
  static lazyLoadImages() {
    const images = document.querySelectorAll('img[data-src]');
    const imageObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          img.src = img.dataset.src;
          img.removeAttribute('data-src');
          observer.unobserve(img);
        }
      });
    });
    
    images.forEach(img => imageObserver.observe(img));
  }
  
  // Приоритезация загрузки ресурсов
  static prioritizeResourceLoading() {
    // Критические ресурсы загружаем с высоким приоритетом
    const criticalStyles = document.querySelectorAll('link[rel="stylesheet"][data-priority="critical"]');
    criticalStyles.forEach(link => {
      link.rel = 'stylesheet';
    });
    
    // Некритические ресурсы загружаем с низким приоритетом
    const nonCriticalScripts = document.querySelectorAll('script[data-priority="low"]');
    nonCriticalScripts.forEach(script => {
      script.async = true;
    });
  }
  
  // Кэширование ресурсов с Service Worker
  static registerServiceWorker() {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/sw.js')
        .then(registration => {
          console.log('Service Worker зарегистрирован:', registration);
        })
        .catch(error => {
          console.error('Ошибка регистрации Service Worker:', error);
        });
    }
  }
  
  // Оптимизация загрузки шрифтов
  static optimizeFontLoading() {
    // Используем font-display: swap
    const fontLinks = document.querySelectorAll('link[rel="stylesheet"][data-font]');
    
    fontLinks.forEach(link => {
      // Добавляем font-display: swap через CSS
      const style = document.createElement('style');
      style.textContent = `
        @font-face {
          font-display: swap;
        }
      `;
      document.head.appendChild(style);
    });
  }
}

// Пример Service Worker для кэширования
const serviceWorkerTemplate = `
const CACHE_NAME = 'app-cache-v1';
const urlsToCache = [
  '/',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Возвращаем кэшированный ресурс или загружаем новый
        return response || fetch(event.request);
      })
  );
});
`;
```

## Оптимизация WebSocket соединений

Эффективное использование WebSocket:

```javascript
// Оптимизация WebSocket соединений
class WebSocketOptimization {
  constructor(url) {
    this.url = url;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectDelay = 1000;
    this.messageQueue = [];
    this.isConnected = false;
    this.heartbeatInterval = null;
    
    this.connect();
  }
  
  connect() {
    try {
      this.ws = new WebSocket(this.url);
      
      this.ws.onopen = () => {
        console.log('WebSocket соединение установлено');
        this.isConnected = true;
        this.reconnectAttempts = 0;
        
        // Отправляем сообщения из очереди
        this.flushMessageQueue();
        
        // Начинаем heartbeat
        this.startHeartbeat();
      };
      
      this.ws.onmessage = (event) => {
        this.handleMessage(event.data);
      };
      
      this.ws.onclose = () => {
        console.log('WebSocket соединение закрыто');
        this.isConnected = false;
        this.stopHeartbeat();
        this.reconnect();
      };
      
      this.ws.onerror = (error) => {
        console.error('WebSocket ошибка:', error);
        this.stopHeartbeat();
      };
    } catch (error) {
      console.error('Ошибка создания WebSocket:', error);
      this.reconnect();
    }
  }
  
  reconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = this.reconnectDelay * this.reconnectAttempts;
      
      console.log(`Попытка переподключения через ${delay} мс...`);
      
      setTimeout(() => {
        this.connect();
      }, delay);
    } else {
      console.error('Максимальное количество попыток переподключения исчерпано');
    }
  }
  
  sendMessage(data) {
    if (this.isConnected) {
      this.ws.send(JSON.stringify(data));
    } else {
      // Добавляем в очередь, если нет соединения
      this.messageQueue.push(data);
    }
  }
  
  flushMessageQueue() {
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift();
      this.sendMessage(message);
    }
  }
  
  handleMessage(data) {
    try {
      const message = JSON.parse(data);
      // Обрабатываем сообщение
      console.log('Получено сообщение:', message);
    } catch (error) {
      console.error('Ошибка обработки сообщения:', error);
    }
  }
  
  startHeartbeat() {
    this.heartbeatInterval = setInterval(() => {
      this.sendMessage({ type: 'ping' });
    }, 30000); // Каждые 30 секунд
  }
  
  stopHeartbeat() {
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
      this.heartbeatInterval = null;
    }
  }
  
  close() {
    if (this.ws) {
      this.ws.close();
    }
    this.stopHeartbeat();
  }
}

// Использование оптимизированного WebSocket
class WebSocketClient {
  constructor() {
    this.ws = new WebSocketOptimization('ws://localhost:8080');
  }
  
  sendUserData(userData) {
    this.ws.sendMessage({
      type: 'user_data',
      data: userData
    });
  }
  
  sendChatMessage(message) {
    this.ws.sendMessage({
      type: 'chat_message',
      message: message,
      timestamp: Date.now()
    });
  }
}
```

## Оптимизация API запросов

Эффективная работа с REST API:

```javascript
// Оптимизация API запросов
class APIOptimization {
  constructor(baseURL, defaultOptions = {}) {
    this.baseURL = baseURL;
    this.defaultOptions = {
      headers: {
        'Content-Type': 'application/json',
        ...defaultOptions.headers
      },
      ...defaultOptions
    };
    
    // Кэш для GET запросов
    this.getCache = new Map();
    this.cacheTimeout = 5 * 60 * 1000; // 5 минут
    
    // Очередь для ограничения параллельных запросов
    this.requestQueue = [];
    this.activeRequests = 0;
    this.maxConcurrentRequests = 6;
  }
  
  // Ограниченные параллельные запросы
  async request(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.requestQueue.push({
        url,
        options,
        resolve,
        reject
      });
      
      this.processQueue();
    });
  }
  
  async processQueue() {
    if (this.activeRequests >= this.maxConcurrentRequests || this.requestQueue.length === 0) {
      return;
    }
    
    this.activeRequests++;
    const { url, options, resolve, reject } = this.requestQueue.shift();
    
    try {
      const response = await fetch(`${this.baseURL}${url}`, {
        ...this.defaultOptions,
        ...options
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const data = await response.json();
      resolve(data);
    } catch (error) {
      reject(error);
    } finally {
      this.activeRequests--;
      // Обрабатываем следующий запрос в очереди
      setTimeout(() => this.processQueue(), 0);
    }
  }
  
  // Кэшированный GET запрос
  async get(url, useCache = true) {
    if (useCache) {
      const cached = this.getCache.get(url);
      if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
        return cached.data;
      }
    }
    
    try {
      const data = await this.request(url, { method: 'GET' });
      
      if (useCache) {
        this.getCache.set(url, {
          data,
          timestamp: Date.now()
        });
      }
      
      return data;
    } catch (error) {
      // В случае ошибки возвращаем кэшированные данные, если есть
      if (useCache) {
        const cached = this.getCache.get(url);
        if (cached) {
          console.warn('Использование кэшированных данных из-за ошибки:', error);
          return cached.data;
        }
      }
      throw error;
    }
  }
  
  // POST запрос с retry
  async post(url, data, retries = 3) {
    const options = {
      method: 'POST',
      body: JSON.stringify(data)
    };
    
    for (let i = 0; i <= retries; i++) {
      try {
        return await this.request(url, options);
      } catch (error) {
        if (i === retries) {
          throw error;
        }
        
        // Ждем перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
  }
  
  // Пакетный запрос
  async batchGet(urls) {
    const promises = urls.map(url => this.get(url, false));
    const results = await Promise.allSettled(promises);
    
    const successful = {};
    const failed = {};
    
    results.forEach((result, index) => {
      const url = urls[index];
      if (result.status === 'fulfilled') {
        successful[url] = result.value;
      } else {
        failed[url] = result.reason;
      }
    });
    
    return { successful, failed };
  }
  
  // Очистка кэша
  clearCache() {
    this.getCache.clear();
  }
  
  // Получение статистики
  getStats() {
    return {
      cacheSize: this.getCache.size,
      queueLength: this.requestQueue.length,
      activeRequests: this.activeRequests,
      maxConcurrentRequests: this.maxConcurrentRequests
    };
  }
}

// Использование оптимизированного API клиента
class APIClient {
  constructor() {
    this.api = new APIOptimization('https://api.example.com');
  }
  
  async loadUserData(userId) {
    return await this.api.get(`/users/${userId}`);
  }
  
  async loadUserPosts(userId) {
    return await this.api.get(`/users/${userId}/posts`);
  }
  
  async loadUserComments(userId) {
    return await this.api.get(`/users/${userId}/comments`);
  }
  
  async loadUserProfile(userId) {
    // Параллельная загрузка связанных данных
    const [userData, posts, comments] = await Promise.all([
      this.loadUserData(userId),
      this.loadUserPosts(userId),
      this.loadUserComments(userId)
    ]);
    
    return {
      user: userData,
      posts,
      comments
    };
  }
}
```

## Оптимизация загрузки данных

Эффективная загрузка и обработка больших объемов данных:

```javascript
// Оптимизация загрузки данных
class DataLoadingOptimization {
  // Постраничная загрузка
  static async loadPaginatedData(apiEndpoint, pageSize = 50) {
    const allData = [];
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
      try {
        const response = await fetch(`${apiEndpoint}?page=${page}&limit=${pageSize}`);
        const data = await response.json();
        
        if (data.items && data.items.length > 0) {
          allData.push(...data.items);
          hasMore = data.items.length === pageSize;
          page++;
        } else {
          hasMore = false;
        }
        
        // Даем возможность выполнить другие задачи
        if (page % 5 === 0) {
          await new Promise(resolve => setTimeout(resolve, 0));
        }
      } catch (error) {
        console.error(`Ошибка загрузки страницы ${page}:`, error);
        hasMore = false;
      }
    }
    
    return allData;
  }
  
  // Инкрементальная загрузка
  static async loadIncrementalData(apiEndpoint, lastId = 0, batchSize = 100) {
    const newData = [];
    let currentId = lastId;
    let hasMore = true;
    
    while (hasMore) {
      try {
        const response = await fetch(
          `${apiEndpoint}?since=${currentId}&limit=${batchSize}`
        );
        const data = await response.json();
        
        if (data.items && data.items.length > 0) {
          newData.push(...data.items);
          currentId = data.items[data.items.length - 1].id;
          hasMore = data.items.length === batchSize;
        } else {
          hasMore = false;
        }
      } catch (error) {
        console.error('Ошибка инкрементальной загрузки:', error);
        hasMore = false;
      }
    }
    
    return {
      data: newData,
      lastId: currentId
    };
  }
  
  // Загрузка с прогрессом
  static async loadDataWithProgress(url, onProgress) {
    const response = await fetch(url);
    const contentLength = +response.headers.get('Content-Length');
    const reader = response.body.getReader();
    
    const chunks = [];
    let receivedLength = 0;
    
    while (true) {
      const { done, value } = await reader.read();
      
      if (done) {
        break;
      }
      
      chunks.push(value);
      receivedLength += value.length;
      
      if (onProgress && contentLength) {
        const progress = (receivedLength / contentLength) * 100;
        onProgress(progress);
      }
    }
    
    // Собираем все чанки в один Uint8Array
    const chunksAll = new Uint8Array(receivedLength);
    let position = 0;
    for (const chunk of chunks) {
      chunksAll.set(chunk, position);
      position += chunk.length;
    }
    
    // Преобразуем в строку и парсим JSON
    const result = new TextDecoder("utf-8").decode(chunksAll);
    return JSON.parse(result);
  }
  
  // Кэширование с инвалидацией
  static createSmartCache(invalidationTime = 10 * 60 * 1000) { // 10 минут
    const cache = new Map();
    
    return {
      async get(key, fetcher) {
        const cached = cache.get(key);
        const now = Date.now();
        
        if (cached && (now - cached.timestamp) < invalidationTime) {
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
      },
      
      getStats() {
        return {
          size: cache.size,
          keys: Array.from(cache.keys())
        };
      }
    };
  }
}
```

## Практические рекомендации

1. **Используйте кэширование** - для повторяющихся запросов
2. **Ограничивайте параллельные запросы** - чтобы не перегружать сервер
3. **Реализуйте retry механизмы** - для надежности
4. **Используйте таймауты** - для предотвращения зависания
5. **Оптимизируйте загрузку ресурсов** - с помощью предзагрузки и ленивой загрузки
6. **Применяйте пакетирование запросов** - для уменьшения количества HTTP запросов
7. **Используйте Service Worker** - для оффлайн кэширования
8. **Оптимизируйте WebSocket** - с переподключением и heartbeat

Оптимизация сетевых операций критически важна для создания быстрых и надежных веб-приложений. Следование этим принципам поможет улучшить пользовательский опыт и снизить нагрузку на сервер.

#javascript #networking #optimization #performance #http #websocket #api #caching