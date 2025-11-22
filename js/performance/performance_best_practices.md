---
tags: [javascript, frontend, performance, optimization, best-practices]
aliases: [оптимизация js производительности, лучшие практики производительности, производительность веб-приложений]
---

# Оптимизация производительности JavaScript - Лучшие практики для Frontend разработки

## Введение

Производительность веб-приложений напрямую влияет на пользовательский опыт, SEO и бизнес-метрики. В этой статье рассмотрим комплексный подход к оптимизации производительности JavaScript приложений, включая лучшие практики, инструменты и методы измерения.

## 1. Фундаментальные принципы оптимизации

### Измеряйте перед оптимизацией

```javascript
// Создание системы мониторинга производительности
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observers = [];
  }
  
  // Измерение времени выполнения функции
  measureFunction(name, fn) {
    return function(...args) {
      const start = performance.now();
      const result = fn.apply(this, args);
      const end = performance.now();
      
      this.recordMetric(`${name}_execution_time`, end - start);
      return result;
    };
  }
  
  // Асинхронное измерение
  async measureAsyncFunction(name, asyncFn) {
    return async function(...args) {
      const start = performance.now();
      const result = await asyncFn.apply(this, args);
      const end = performance.now();
      
      this.recordMetric(`${name}_execution_time`, end - start);
      return result;
    };
  }
  
  // Замер FPS
  measureFPS() {
    let lastTime = performance.now();
    let frameCount = 0;
    let fps = 0;
    
    const measure = (currentTime) => {
      frameCount++;
      
      if (currentTime - lastTime >= 1000) {
        fps = frameCount;
        frameCount = 0;
        lastTime = currentTime;
        
        this.recordMetric('fps', fps);
      }
      
      requestAnimationFrame(measure);
    };
    
    requestAnimationFrame(measure);
    return () => {}; // Возвращаем функцию остановки
  }
  
  // Замер использования памяти
  measureMemory() {
    if ('memory' in performance) {
      const memory = performance.memory;
      this.recordMetric('used_heap_size', memory.usedJSHeapSize);
      this.recordMetric('total_heap_size', memory.totalJSHeapSize);
      this.recordMetric('heap_limit', memory.jsHeapSizeLimit);
    }
  }
  
  recordMetric(name, value) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    
    const values = this.metrics.get(name);
    values.push({
      value,
      timestamp: performance.now()
    });
    
    // Ограничиваем размер истории
    if (values.length > 1000) {
      values.shift();
    }
    
    // Уведомляем наблюдателей
    this.observers.forEach(observer => {
      observer(name, value);
    });
  }
  
  subscribe(observer) {
    this.observers.push(observer);
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  getMetrics() {
    return Object.fromEntries(this.metrics);
  }
  
  // Получение статистики по метрике
  getMetricStats(name) {
    const values = this.metrics.get(name) || [];
    if (values.length === 0) return null;
    
    const numericValues = values.map(v => v.value);
    return {
      min: Math.min(...numericValues),
      max: Math.max(...numericValues),
      avg: numericValues.reduce((a, b) => a + b, 0) / numericValues.length,
      count: numericValues.length
    };
  }
}

// Использование монитора производительности
const perfMonitor = new PerformanceMonitor();

// Подписка на метрики
perfMonitor.subscribe((name, value) => {
  if (name === 'fps' && value < 30) {
    console.warn(`Низкий FPS: ${value}`);
  }
});

// Измерение функции
const measuredFunction = perfMonitor.measureFunction('dataProcessing', (data) => {
  return data.map(item => item * 2);
});
```

### Понимание критических путей рендеринга

```javascript
// Оптимизация критического пути рендеринга
class RenderingOptimizer {
  constructor() {
    this.criticalResources = new Set();
    this.nonCriticalResources = new Set();
  }
  
  // Загрузка критических ресурсов
  loadCriticalResource(url, type = 'script') {
    return new Promise((resolve, reject) => {
      let element;
      
      if (type === 'script') {
        element = document.createElement('script');
        element.src = url;
      } else if (type === 'style') {
        element = document.createElement('link');
        element.rel = 'stylesheet';
        element.href = url;
      }
      
      element.onload = resolve;
      element.onerror = reject;
      
      // Добавляем в начало head для приоритета
      document.head.insertBefore(element, document.head.firstChild);
      
      this.criticalResources.add(url);
    });
  }
  
  // Отложенная загрузка некритических ресурсов
  loadNonCriticalResource(url, type = 'script') {
    // Загружаем после загрузки DOM
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', () => {
        this.loadResource(url, type);
      });
    } else {
      return this.loadResource(url, type);
    }
  }
  
  async loadResource(url, type = 'script') {
    if (type === 'script') {
      return import(url); // Динамический импорт для модулей
    } else if (type === 'style') {
      const link = document.createElement('link');
      link.rel = 'stylesheet';
      link.href = url;
      document.head.appendChild(link);
      return new Promise((resolve, reject) => {
        link.onload = resolve;
        link.onerror = reject;
      });
    }
  }
  
  // Оптимизация критического CSS
  inlineCriticalCSS(css) {
    const style = document.createElement('style');
    style.textContent = css;
    document.head.appendChild(style);
  }
  
  // Отложенная загрузка шрифтов
  loadFont(fontFamily, url) {
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'font';
    link.type = 'font/woff2';
    link.href = url;
    link.crossOrigin = 'anonymous';
    
    document.head.appendChild(link);
    
    // После загрузки подключаем шрифт
    link.onload = () => {
      const fontStyle = document.createElement('style');
      fontStyle.textContent = `
        @font-face {
          font-family: '${fontFamily}';
          src: url('${url}') format('woff2');
          font-display: swap;
        }
      `;
      document.head.appendChild(fontStyle);
    };
  }
}

// Использование оптимизатора рендеринга
const renderer = new RenderingOptimizer();

// Загрузка критических ресурсов
renderer.loadCriticalResource('/js/critical.js', 'script');
renderer.inlineCriticalCSS(`
  body { font-family: Arial, sans-serif; }
  .header { display: flex; }
`);

// Загрузка некритических ресурсов
renderer.loadNonCriticalResource('/js/analytics.js', 'script');
renderer.loadFont('CustomFont', '/fonts/custom.woff2');
```

## 2. Оптимизация работы с DOM

### Эффективная работа с DOM элементами

```javascript
// Класс для эффективной работы с DOM
class EfficientDOMManager {
  constructor() {
    this.elementCache = new Map();
    this.batchOperations = [];
    this.isBatching = false;
  }
  
  // Кэширование элементов
  getElement(selector) {
    if (!this.elementCache.has(selector)) {
      this.elementCache.set(selector, document.querySelector(selector));
    }
    return this.elementCache.get(selector);
  }
  
  // Пакетная обработка DOM операций
  batch(operations) {
    if (!this.isBatching) {
      this.isBatching = true;
      requestAnimationFrame(() => {
        operations.forEach(op => op());
        this.isBatching = false;
      });
    } else {
      // Если уже в режиме пакетной обработки, просто добавляем операции
      operations.forEach(op => op());
    }
  }
  
  // Эффективное обновление списка
  updateList(container, items, renderItem) {
    // Используем DocumentFragment для минимизации перерисовок
    const fragment = document.createDocumentFragment();
    
    items.forEach((item, index) => {
      const element = renderItem(item, index);
      fragment.appendChild(element);
    });
    
    // Очищаем контейнер и добавляем фрагмент за один раз
    container.innerHTML = '';
    container.appendChild(fragment);
  }
  
  // Дифф-алгоритм для минимальных изменений DOM
  diffAndUpdateList(container, newList, renderItem, keyFn) {
    const oldElements = Array.from(container.children);
    const oldKeys = oldElements.map(el => el.dataset.key);
    const newKeys = newList.map(item => keyFn(item));
    
    // Находим элементы для добавления, удаления и перемещения
    const toRemove = oldKeys.filter(key => !newKeys.includes(key));
    const toAdd = newKeys.filter(key => !oldKeys.includes(key));
    
    // Удаляем лишние элементы
    oldElements.forEach((el, index) => {
      if (toRemove.includes(el.dataset.key)) {
        el.remove();
      }
    });
    
    // Добавляем новые элементы
    newList.forEach((item, index) => {
      const key = keyFn(item);
      if (toAdd.includes(key)) {
        const element = renderItem(item, index);
        element.dataset.key = key;
        
        // Вставляем на правильную позицию
        const nextElement = container.children[index];
        if (nextElement) {
          container.insertBefore(element, nextElement);
        } else {
          container.appendChild(element);
        }
      }
    });
  }
  
  // Использование CSS-трансформаций вместо изменения layout свойств
  moveElement(element, x, y) {
    // Используем transform вместо left/top для избежания relayout
    element.style.transform = `translate(${x}px, ${y}px)`;
  }
  
  // Оптимизация перерисовки с использованием will-change
  prepareForAnimation(element, properties) {
    element.style.willChange = properties.join(', ');
    
    // Сброс после анимации
    setTimeout(() => {
      element.style.willChange = 'auto';
    }, 1000); // Примерное время анимации
  }
  
  // Использование CSS containment для изоляции изменений
  isolateChanges(element) {
    element.style.contain = 'layout style paint';
  }
}

// Использование эффективного DOM менеджера
const domManager = new EfficientDOMManager();

// Рендер списка с минимальными изменениями
function renderUserList(users) {
  const container = domManager.getElement('#user-list');
  
  domManager.diffAndUpdateList(
    container,
    users,
    (user, index) => {
      const li = document.createElement('li');
      li.innerHTML = `<span>${user.name}</span> <span>${user.email}</span>`;
      return li;
    },
    user => user.id
  );
}
```

### Оптимизация обработчиков событий

```javascript
// Менеджер событий с делегированием
class EventManager {
  constructor() {
    this.eventListeners = new Map();
    this.delegatedHandlers = new Map();
  }
  
  // Делегирование событий для динамических элементов
  delegateEvent(container, event, selector, handler) {
    const key = `${event}-${selector}`;
    
    if (!this.delegatedHandlers.has(key)) {
      const delegatedHandler = (e) => {
        const target = e.target.closest(selector);
        if (target) {
          handler.call(target, e, target);
        }
      };
      
      container.addEventListener(event, delegatedHandler);
      this.delegatedHandlers.set(key, {
        container,
        event,
        handler: delegatedHandler
      });
    }
  }
  
  // Эффективное управление слушателями
  addListener(element, event, handler, options = {}) {
    const key = this.generateListenerKey(element, event, handler);
    
    if (!this.eventListeners.has(key)) {
      element.addEventListener(event, handler, options);
      this.eventListeners.set(key, { element, event, handler, options });
    }
  }
  
  removeListener(element, event, handler) {
    const key = this.generateListenerKey(element, event, handler);
    const listener = this.eventListeners.get(key);
    
    if (listener) {
      element.removeEventListener(event, handler, listener.options);
      this.eventListeners.delete(key);
    }
  }
  
  removeAllListeners() {
    for (const [key, listener] of this.eventListeners) {
      const { element, event, handler, options } = listener;
      element.removeEventListener(event, handler, options);
    }
    this.eventListeners.clear();
  }
  
  generateListenerKey(element, event, handler) {
    return `${element.tagName}-${event}-${handler.name || 'anonymous'}`;
  }
  
  // Использование passive listeners для улучшения скролла
  addPassiveListener(element, event, handler) {
    element.addEventListener(event, handler, { passive: true });
  }
  
  // Использование capture phase для оптимизации
  addCaptureListener(element, event, handler) {
    element.addEventListener(event, handler, { capture: true });
  }
}

// Использование менеджера событий
const eventManager = new EventManager();

// Делегирование для динамических кнопок
eventManager.delegateEvent(
  document,
  'click',
  '.dynamic-button',
  (e, button) => {
    console.log('Клик на динамическую кнопку:', button.textContent);
  }
);

// Passive scroll listener для плавного скролла
eventManager.addPassiveListener(
  window,
  'scroll',
  () => {
    // Обработка скролла без блокировки
    updateScrollPosition();
  }
);
```

## 3. Оптимизация асинхронных операций

### Управление асинхронными операциями

```javascript
// Менеджер асинхронных операций с оптимизациями
class AsyncOperationManager {
  constructor() {
    this.activeOperations = new Map();
    this.operationQueue = [];
    this.maxConcurrent = 6; // Ограничение количества одновременных операций
    this.retryConfig = {
      maxRetries: 3,
      baseDelay: 1000,
      backoffMultiplier: 2
    };
  }
  
  // Выполнение операции с ограничением количества
  async execute(operation, priority = 'normal') {
    return new Promise((resolve, reject) => {
      const operationWrapper = {
        operation,
        resolve,
        reject,
        priority,
        timestamp: Date.now()
      };
      
      // Добавляем в очередь в зависимости от приоритета
      if (priority === 'high') {
        this.operationQueue.unshift(operationWrapper);
      } else {
        this.operationQueue.push(operationWrapper);
      }
      
      this.processQueue();
    });
  }
  
  async processQueue() {
    while (this.activeOperations.size < this.maxConcurrent && this.operationQueue.length > 0) {
      const operationWrapper = this.operationQueue.shift();
      this.executeOperation(operationWrapper);
    }
  }
  
  async executeOperation(operationWrapper) {
    const { operation, resolve, reject } = operationWrapper;
    const operationId = Symbol('operation');
    
    this.activeOperations.set(operationId, operationWrapper);
    
    try {
      const result = await this.executeWithRetry(operation);
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.activeOperations.delete(operationId);
      this.processQueue(); // Продолжаем обработку очереди
    }
  }
  
  // Выполнение с повторными попытками
  async executeWithRetry(operation, retriesLeft = this.retryConfig.maxRetries) {
    try {
      return await operation();
    } catch (error) {
      if (retriesLeft > 0 && this.shouldRetry(error)) {
        const delay = this.calculateDelay(this.retryConfig.maxRetries - retriesLeft);
        await this.delay(delay);
        return this.executeWithRetry(operation, retriesLeft - 1);
      }
      throw error;
    }
  }
  
  shouldRetry(error) {
    // Проверяем, можно ли повторить операцию
    return error.name === 'NetworkError' || 
           error.status >= 500 || 
           error.message.includes('timeout');
  }
  
  calculateDelay(attempt) {
    return this.retryConfig.baseDelay * Math.pow(this.retryConfig.backoffMultiplier, attempt);
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Отмена операции
  cancel(operationId) {
    const operation = this.activeOperations.get(operationId);
    if (operation) {
      // Реализация отмены через AbortController или аналогичный механизм
      operation.cancelled = true;
      operation.reject(new Error('Operation cancelled'));
      this.activeOperations.delete(operationId);
    }
  }
  
  // Получение статистики
  getStats() {
    return {
      activeOperations: this.activeOperations.size,
      queuedOperations: this.operationQueue.length,
      maxConcurrent: this.maxConcurrent
    };
  }
}

// Использование менеджера асинхронных операций
const asyncManager = new AsyncOperationManager();

// Пример использования
async function fetchUserData(userId) {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

// Выполнение с управлением
asyncManager.execute(() => fetchUserData(123), 'high')
  .then(data => console.log('Данные пользователя:', data))
  .catch(error => console.error('Ошибка:', error));
```

### Кеширование и предзагрузка данных

```javascript
// Продвинутый кеш с политиками инвалидации
class AdvancedCache {
  constructor(options = {}) {
    this.cache = new Map();
    this.metadata = new Map();
    this.maxSize = options.maxSize || 1000;
    this.defaultTTL = options.defaultTTL || 5 * 60 * 1000; // 5 минут
    this.evictionPolicy = options.evictionPolicy || 'lru'; // 'lru', 'fifo', 'lfu'
  }
  
  set(key, value, options = {}) {
    const ttl = options.ttl || this.defaultTTL;
    const timestamp = Date.now();
    
    // Проверяем размер кеша
    if (this.cache.size >= this.maxSize) {
      this.evict();
    }
    
    this.cache.set(key, value);
    this.metadata.set(key, {
      timestamp,
      ttl,
      accessCount: 0,
      tags: options.tags || []
    });
  }
  
  get(key) {
    const metadata = this.metadata.get(key);
    
    // Проверяем TTL
    if (metadata && Date.now() - metadata.timestamp > metadata.ttl) {
      this.delete(key);
      return undefined;
    }
    
    // Обновляем статистику доступа для LRU/LFU
    if (metadata) {
      this.metadata.set(key, {
        ...metadata,
        accessCount: metadata.accessCount + 1
      });
    }
    
    return this.cache.get(key);
  }
  
  has(key) {
    const value = this.get(key);
    return value !== undefined;
  }
  
  delete(key) {
    this.cache.delete(key);
    this.metadata.delete(key);
  }
  
  // Инвалидация по тегам
  invalidateByTag(tag) {
    for (const [key, metadata] of this.metadata) {
      if (metadata.tags.includes(tag)) {
        this.delete(key);
      }
    }
  }
  
  // Инвалидация по шаблону
  invalidateByPattern(pattern) {
    for (const key of this.cache.keys()) {
      if (key.includes(pattern)) {
        this.delete(key);
      }
    }
  }
  
  // Очистка устаревших записей
  clearExpired() {
    const now = Date.now();
    for (const [key, metadata] of this.metadata) {
      if (now - metadata.timestamp > metadata.ttl) {
        this.delete(key);
      }
    }
  }
  
  // Алгоритмы вытеснения
  evict() {
    if (this.evictionPolicy === 'lru') {
      this.evictLRU();
    } else if (this.evictionPolicy === 'fifo') {
      this.evictFIFO();
    } else if (this.evictionPolicy === 'lfu') {
      this.evictLFU();
    }
  }
  
  evictLRU() {
    let oldestKey;
    let oldestTime = Infinity;
    
    for (const [key, metadata] of this.metadata) {
      if (metadata.timestamp < oldestTime) {
        oldestTime = metadata.timestamp;
        oldestKey = key;
      }
    }
    
    if (oldestKey) {
      this.delete(oldestKey);
    }
  }
  
  evictFIFO() {
    // FIFO просто удаляет самую старую запись
    const firstKey = this.cache.keys().next().value;
    if (firstKey) {
      this.delete(firstKey);
    }
  }
  
  evictLFU() {
    let leastFrequentKey;
    let leastFrequent = Infinity;
    
    for (const [key, metadata] of this.metadata) {
      if (metadata.accessCount < leastFrequent) {
        leastFrequent = metadata.accessCount;
        leastFrequentKey = key;
      }
    }
    
    if (leastFrequentKey) {
      this.delete(leastFrequentKey);
    }
  }
  
  // Получение статистики
  getStats() {
    return {
      size: this.cache.size,
      hits: 0, // Реализовать при добавлении подсчета хитов
      misses: 0
    };
  }
  
  // Очистка всего кеша
  clear() {
    this.cache.clear();
    this.metadata.clear();
  }
}

// Использование продвинутого кеша
const dataCache = new AdvancedCache({
  maxSize: 500,
  defaultTTL: 10 * 60 * 1000, // 10 минут
  evictionPolicy: 'lru'
});

// Кеширование данных пользователя
async function getCachedUserData(userId) {
  const cacheKey = `user-${userId}`;
  
  // Проверяем кеш
  let userData = dataCache.get(cacheKey);
  if (userData) {
    return userData;
  }
  
  // Загружаем данные
  userData = await fetch(`/api/users/${userId}`).then(r => r.json());
  
  // Кешируем с тегом для инвалидации
  dataCache.set(cacheKey, userData, {
    tags: ['users'],
    ttl: 15 * 60 * 1000 // 15 минут
  });
  
  return userData;
}

// Инвалидация при обновлении данных
function invalidateUserCache(userId) {
  dataCache.delete(`user-${userId}`);
}
```

## 4. Оптимизация памяти и предотвращение утечек

### Управление памятью и утечками

```javascript
// Менеджер ресурсов для предотвращения утечек
class ResourceManager {
  constructor() {
    this.resources = new Map();
    this.cleanupCallbacks = new Set();
    this.weakRefs = new Set();
  }
  
  // Регистрация ресурса с функцией очистки
  register(resource, cleanupFn, owner) {
    const resourceId = Symbol('resource');
    
    this.resources.set(resourceId, {
      resource,
      cleanupFn,
      owner,
      timestamp: Date.now()
    });
    
    return resourceId;
  }
  
  // Регистрация слабой ссылки
  registerWeakRef(target, cleanupFn) {
    const ref = new WeakRef(target);
    this.weakRefs.add({ ref, cleanupFn });
    
    return { ref, cleanupFn };
  }
  
  // Добавление функции очистки
  addCleanupCallback(callback) {
    this.cleanupCallbacks.add(callback);
  }
  
  // Очистка ресурсов
  cleanup(owner = null) {
    for (const [resourceId, resourceData] of this.resources) {
      if (!owner || resourceData.owner === owner) {
        resourceData.cleanupFn(resourceData.resource);
        this.resources.delete(resourceId);
      }
    }
    
    // Вызов общих функций очистки
    this.cleanupCallbacks.forEach(callback => callback());
  }
  
  // Автоматическая очистка неиспользуемых ресурсов
  cleanupUnused() {
    // Очистка WeakRefs
    this.weakRefs.forEach(({ ref, cleanupFn }) => {
      if (!ref.deref()) {
        cleanupFn();
        this.weakRefs.delete({ ref, cleanupFn });
      }
    });
    
    // Очистка обычных ресурсов
    this.resources.forEach((resourceData, resourceId) => {
      if (!resourceData.owner || !document.contains(resourceData.owner)) {
        resourceData.cleanupFn(resourceData.resource);
        this.resources.delete(resourceId);
      }
    });
  }
  
  // Получение отчета о памяти
  getMemoryReport() {
    return {
      registeredResources: this.resources.size,
      cleanupCallbacks: this.cleanupCallbacks.size,
      weakRefs: this.weakRefs.size
    };
  }
}

// Использование менеджера ресурсов
const resourceManager = new ResourceManager();

// Пример компонента с управлением ресурсами
class OptimizedComponent {
  constructor(container) {
    this.container = container;
    this.eventListeners = [];
    this.timers = [];
    this.observer = null;
    
    // Регистрируем компонент в менеджере ресурсов
    this.resourceId = resourceManager.register(
      this,
      () => this.destroy(),
      this.container
    );
    
    this.init();
  }
  
  init() {
    // Добавляем обработчики событий
    const clickHandler = () => console.log('Клик');
    this.container.addEventListener('click', clickHandler);
    this.eventListeners.push({ element: this.container, event: 'click', handler: clickHandler });
    
    // Добавляем таймер
    const timer = setInterval(() => {
      this.update();
    }, 1000);
    this.timers.push(timer);
    
    // Добавляем Intersection Observer
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.onVisible();
        }
      });
    });
    this.observer.observe(this.container);
  }
  
  update() {
    // Обновление компонента
  }
  
  onVisible() {
    // Действия при появлении в области видимости
  }
  
  destroy() {
    // Удаляем обработчики событий
    this.eventListeners.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    this.eventListeners = [];
    
    // Очищаем таймеры
    this.timers.forEach(timer => clearInterval(timer));
    this.timers = [];
    
    // Отключаем observer
    if (this.observer) {
      this.observer.disconnect();
      this.observer = null;
    }
    
    // Очищаем DOM
    if (this.container && this.container.parentNode) {
      this.container.parentNode.removeChild(this.container);
    }
    
    this.container = null;
  }
  
  // Метод для ручной очистки
  cleanup() {
    resourceManager.cleanup(this.container);
  }
}

// Глобальная очистка неиспользуемых ресурсов
setInterval(() => {
  resourceManager.cleanupUnused();
}, 30000); // Каждые 30 секунд
```

## 5. Инструменты и методы профилирования

### Система мониторинга производительности

```javascript
// Комплексная система мониторинга
class PerformanceMonitoringSystem {
  constructor() {
    this.metrics = {
      navigation: {},
      resource: {},
      paint: {},
      userTiming: {},
      memory: {},
      custom: {}
    };
    
    this.reporters = [];
    this.thresholds = {
      fcp: 1800,    // First Contentful Paint
      lcp: 2500,    // Largest Contentful Paint
      fid: 100,     // First Input Delay
      cls: 0.1,     // Cumulative Layout Shift
      ttfb: 600     // Time to First Byte
    };
    
    this.init();
  }
  
  init() {
    // Сбор метрик загрузки страницы
    this.collectNavigationMetrics();
    
    // Сбор метрик ресурсов
    this.collectResourceMetrics();
    
    // Сбор метрик отрисовки
    this.collectPaintMetrics();
    
    // Мониторинг памяти
    this.monitorMemory();
    
    // Мониторинг пользовательских событий
    this.monitorUserInteractions();
  }
  
  collectNavigationMetrics() {
    if ('performance' in window) {
      const navEntries = performance.getEntriesByType('navigation');
      if (navEntries.length > 0) {
        const navEntry = navEntries[0];
        this.metrics.navigation = {
          dns: navEntry.domainLookupEnd - navEntry.domainLookupStart,
          connect: navEntry.connectEnd - navEntry.connectStart,
          request: navEntry.responseStart - navEntry.requestStart,
          response: navEntry.responseEnd - navEntry.responseStart,
          dom: navEntry.domContentLoadedEventEnd - navEntry.navigationStart,
          load: navEntry.loadEventEnd - navEntry.navigationStart,
          ttfb: navEntry.responseStart - navEntry.navigationStart
        };
      }
    }
  }
  
  collectResourceMetrics() {
    if ('performance' in window) {
      const resourceEntries = performance.getEntriesByType('resource');
      this.metrics.resource = resourceEntries.map(entry => ({
        name: entry.name,
        duration: entry.duration,
        size: entry.transferSize,
        type: entry.initiatorType
      }));
    }
  }
  
  collectPaintMetrics() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          if (entry.entryType === 'paint') {
            this.metrics.paint[entry.name] = entry.startTime;
          } else if (entry.entryType === 'largest-contentful-paint') {
            this.metrics.paint.lcp = entry.startTime;
          } else if (entry.entryType === 'layout-shift') {
            if (!entry.hadRecentInput) {
              this.metrics.paint.cls = (this.metrics.paint.cls || 0) + entry.value;
            }
          }
        });
      });
      
      observer.observe({
        entryTypes: ['paint', 'largest-contentful-paint', 'layout-shift']
      });
    }
  }
  
  monitorMemory() {
    if ('memory' in performance) {
      setInterval(() => {
        this.metrics.memory = {
          used: performance.memory.usedJSHeapSize,
          total: performance.memory.totalJSHeapSize,
          limit: performance.memory.jsHeapSizeLimit,
          timestamp: Date.now()
        };
      }, 5000);
    }
  }
  
  monitorUserInteractions() {
    let interactionCount = 0;
    let interactionDelays = [];
    
    const measureInteraction = (event) => {
      interactionCount++;
      const start = performance.now();
      
      // Измеряем задержку до следующего кадра
      requestAnimationFrame(() => {
        const delay = performance.now() - start;
        interactionDelays.push(delay);
        
        // Если задержка слишком большая, это FID
        if (delay > this.thresholds.fid) {
          this.metrics.custom.fid = delay;
        }
      });
    };
    
    ['click', 'keydown', 'touchstart'].forEach(eventType => {
      document.addEventListener(eventType, measureInteraction, { passive: true });
    });
  }
  
  // Добавление пользовательских метрик
  addCustomMetric(name, value) {
    this.metrics.custom[name] = value;
    this.checkThresholds(name, value);
  }
  
  // Проверка пороговых значений
  checkThresholds(metricName, value) {
    if (this.thresholds[metricName] !== undefined) {
      if (value > this.thresholds[metricName]) {
        this.reportIssue(`${metricName.toUpperCase()} превышает порог: ${value} > ${this.thresholds[metricName]}`);
      }
    }
  }
  
  // Добавление репортера
  addReporter(reporter) {
    this.reporters.push(reporter);
  }
  
  // Отчет о проблеме
  reportIssue(message) {
    this.reporters.forEach(reporter => {
      reporter(message, this.metrics);
    });
  }
  
  // Получение полного отчета
  getReport() {
    return {
      metrics: this.metrics,
      thresholds: this.thresholds,
      issues: []
    };
  }
  
  // Отправка отчета на сервер
  async sendReport() {
    try {
      await fetch('/api/performance-report', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(this.getReport())
      });
    } catch (error) {
      console.error('Ошибка отправки отчета производительности:', error);
    }
  }
}

// Использование системы мониторинга
const perfSystem = new PerformanceMonitoringSystem();

// Добавление кастомного репортера
perfSystem.addReporter((message, metrics) => {
  console.warn('Проблема производительности:', message);
  
  // Отправка в систему мониторинга
  if (typeof gtag !== 'undefined') {
    gtag('event', 'performance_issue', {
      event_category: 'Performance',
      event_label: message,
      value: 1
    });
  }
});

// Пример пользовательской метрики
function measureComponentRenderTime(componentName, renderFunction) {
  const start = performance.now();
  const result = renderFunction();
  const end = performance.now();
  
  perfSystem.addCustomMetric(`${componentName}_render_time`, end - start);
  
  return result;
}
```

## 6. Практические рекомендации и лучшие практики

### Сводка лучших практик

```javascript
// Шаблон для оптимизированного компонента
class BestPracticeComponent {
  constructor(element) {
    this.element = element;
    this.state = {};
    this.cache = new Map();
    this.isMounted = false;
    
    this.boundMethods = this.bindMethods();
    this.init();
  }
  
  bindMethods() {
    // Привязываем методы для предотвращения создания новых функций
    return {
      handleResize: this.handleResize.bind(this),
      handleScroll: this.handleScroll.bind(this),
      handleClick: this.handleClick.bind(this)
    };
  }
  
  init() {
    this.isMounted = true;
    
    // Используем passive listeners
    window.addEventListener('resize', this.boundMethods.handleResize, { passive: true });
    window.addEventListener('scroll', this.boundMethods.handleScroll, { passive: true });
    
    // Делегируем события
    this.element.addEventListener('click', this.boundMethods.handleClick);
  }
  
  handleResize() {
    // Используем debounce для избежания частых вызовов
    if (!this.resizeHandler) {
      this.resizeHandler = debounce(() => {
        this.onResize();
      }, 100);
    }
    this.resizeHandler();
  }
  
  handleScroll() {
    // Используем throttle для оптимизации
    if (!this.scrollHandler) {
      this.scrollHandler = throttle(() => {
        this.onScroll();
      }, 16); // ~60 FPS
    }
    this.scrollHandler();
  }
  
  handleClick(event) {
    const target = event.target.closest('.action-button');
    if (target) {
      this.performAction(target.dataset.action);
    }
  }
  
  async performAction(action) {
    // Кешируем результаты
    const cacheKey = `action-${action}`;
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }
    
    try {
      const result = await this.executeAction(action);
      this.cache.set(cacheKey, result);
      return result;
    } catch (error) {
      console.error('Ошибка выполнения действия:', error);
      throw error;
    }
  }
  
  async executeAction(action) {
    // Реализация действия
    const response = await fetch(`/api/actions/${action}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ timestamp: Date.now() })
    });
    
    return response.json();
  }
  
  onResize() {
    // Обработка изменения размера
    const rect = this.element.getBoundingClientRect();
    this.updateLayout(rect.width, rect.height);
  }
  
  onScroll() {
    // Обработка скролла
    const scrollY = window.scrollY;
    this.updateScrollState(scrollY);
  }
  
  updateLayout(width, height) {
    // Оптимизированное обновление макета
    if (this.lastWidth !== width || this.lastHeight !== height) {
      this.lastWidth = width;
      this.lastHeight = height;
      
      // Используем CSS transforms вместо изменения layout свойств
      this.element.style.transform = `scale(${width / 1000})`;
    }
  }
  
  updateScrollState(scrollY) {
    // Обновление состояния на основе скролла
    if (Math.abs(this.lastScrollY - scrollY) > 10) {
      this.lastScrollY = scrollY;
      
      // Потенциальная подгрузка данных
      if (scrollY > document.body.scrollHeight - window.innerHeight - 100) {
        this.loadMoreData();
      }
    }
  }
  
  async loadMoreData() {
    // Загрузка дополнительных данных с кешированием
    const cacheKey = 'more-data';
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }
    
    const response = await fetch('/api/more-data');
    const data = await response.json();
    
    this.cache.set(cacheKey, data);
    this.renderAdditionalData(data);
    
    return data;
  }
  
  renderAdditionalData(data) {
    // Эффективный рендер новых данных
    const fragment = document.createDocumentFragment();
    
    data.forEach(item => {
      const element = this.createItemElement(item);
      fragment.appendChild(element);
    });
    
    this.element.appendChild(fragment);
  }
  
  createItemElement(item) {
    const div = document.createElement('div');
    div.className = 'item';
    div.innerHTML = `<h3>${item.title}</h3><p>${item.description}</p>`;
    return div;
  }
  
  destroy() {
    if (!this.isMounted) return;
    
    this.isMounted = false;
    
    // Удаляем все слушатели
    window.removeEventListener('resize', this.boundMethods.handleResize);
    window.removeEventListener('scroll', this.boundMethods.handleScroll);
    this.element.removeEventListener('click', this.boundMethods.handleClick);
    
    // Очищаем кеш
    this.cache.clear();
    
    // Удаляем элемент
    if (this.element && this.element.parentNode) {
      this.element.parentNode.removeChild(this.element);
    }
    
    this.element = null;
  }
}

// Утилиты для оптимизации
const OptimizationUtils = {
  // Дебаунс с опцией immediate
  debounce(func, wait, immediate = false) {
    let timeout;
    return function executedFunction(...args) {
      const later = () => {
        timeout = null;
        if (!immediate) func.apply(this, args);
      };
      const callNow = immediate && !timeout;
      clearTimeout(timeout);
      timeout = setTimeout(later, wait);
      if (callNow) func.apply(this, args);
    };
  },
  
  // Троттлинг
  throttle(func, limit) {
    let inThrottle;
    return function(...args) {
      if (!inThrottle) {
        func.apply(this, args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  },
  
  // Проверка видимости элемента
  isElementInViewport(el) {
    const rect = el.getBoundingClientRect();
    return (
      rect.top >= 0 &&
      rect.left >= 0 &&
      rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
      rect.right <= (window.innerWidth || document.documentElement.clientWidth)
    );
  }
};
```

## Заключение

Оптимизация производительности JavaScript для фронтенд разработки - это комплексная задача, требующая понимания как основ языка, так и специфики работы браузера. Ключевые аспекты включают:

1. Измерение производительности перед оптимизацией
2. Эффективную работу с DOM
3. Правильное управление асинхронными операциями
4. Оптимизацию памяти и предотвращение утечек
5. Использование современных инструментов и подходов

Постоянное мониторирование и профилирование позволяют поддерживать высокую производительность приложения на протяжении всего цикла разработки.

## См. также

- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
- [[promise_practical_examples.md]]
- [[frontend_patterns_examples.md]]
- [[common_tasks_solutions.md]]
- [[optimization_tricks_advanced.md]]