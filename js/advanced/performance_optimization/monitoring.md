# Мониторинг производительности в JavaScript

## Введение

Мониторинг производительности - критически важный аспект разработки высокопроизводительных веб-приложений. Эффективный мониторинг позволяет выявлять узкие места, отслеживать деградацию производительности и обеспечивать оптимальный пользовательский опыт. В этом разделе мы рассмотрим различные техники и инструменты для мониторинга производительности в JavaScript.

## Клиентский мониторинг

Мониторинг производительности на стороне клиента:

```javascript
// Клиентский мониторинг производительности
class ClientPerformanceMonitoring {
  constructor() {
    this.metrics = new Map();
    this.observers = new Set();
    this.isMonitoring = false;
  }
  
  // Запуск мониторинга
  start() {
    if (this.isMonitoring) return;
    
    this.isMonitoring = true;
    this.setupPerformanceObservers();
    this.setupResourceMonitoring();
    this.setupUserExperienceMonitoring();
  }
  
  // Остановка мониторинга
  stop() {
    this.isMonitoring = false;
    this.cleanupObservers();
  }
  
  // Настройка Performance Observer
  setupPerformanceObservers() {
    // Наблюдатель за метриками навигации
    if ('PerformanceObserver' in window) {
      const navigationObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          if (entry.entryType === 'navigation') {
            this.recordNavigationMetrics(entry);
          }
        });
      });
      
      navigationObserver.observe({ entryTypes: ['navigation'] });
      this.observers.add(navigationObserver);
      
      // Наблюдатель за метриками paint
      const paintObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          this.recordPaintMetrics(entry);
        });
      });
      
      paintObserver.observe({ entryTypes: ['paint'] });
      this.observers.add(paintObserver);
      
      // Наблюдатель за метриками ресурсов
      const resourceObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          this.recordResourceMetrics(entry);
        });
      });
      
      resourceObserver.observe({ entryTypes: ['resource'] });
      this.observers.add(resourceObserver);
    }
  }
  
  // Запись метрик навигации
  recordNavigationMetrics(entry) {
    const metrics = {
      pageLoadTime: entry.loadEventEnd - entry.fetchStart,
      domContentLoadedTime: entry.domContentLoadedEventEnd - entry.fetchStart,
      firstByteTime: entry.responseStart - entry.requestStart,
      dnsLookupTime: entry.domainLookupEnd - entry.domainLookupStart,
      tcpConnectionTime: entry.connectEnd - entry.connectStart,
      requestResponseTime: entry.responseEnd - entry.requestStart,
      redirectTime: entry.redirectEnd - entry.redirectStart
    };
    
    this.metrics.set('navigation', metrics);
    this.notifyObservers('navigation', metrics);
  }
  
  // Запись метрик paint
  recordPaintMetrics(entry) {
    const metrics = {
      name: entry.name,
      startTime: entry.startTime
    };
    
    if (!this.metrics.has('paint')) {
      this.metrics.set('paint', []);
    }
    
    this.metrics.get('paint').push(metrics);
    this.notifyObservers('paint', metrics);
  }
  
  // Запись метрик ресурсов
  recordResourceMetrics(entry) {
    const metrics = {
      name: entry.name,
      duration: entry.duration,
      size: entry.transferSize,
      type: entry.initiatorType
    };
    
    if (!this.metrics.has('resources')) {
      this.metrics.set('resources', []);
    }
    
    this.metrics.get('resources').push(metrics);
    this.notifyObservers('resource', metrics);
  }
  
  // Мониторинг ресурсов
  setupResourceMonitoring() {
    // Мониторинг ошибок загрузки ресурсов
    window.addEventListener('error', (event) => {
      if (event.target !== window) {
        this.recordResourceError(event);
      }
    }, true);
    
    // Мониторинг медленных ресурсов
    setInterval(() => {
      this.checkSlowResources();
    }, 30000); // Проверяем каждые 30 секунд
  }
  
  // Запись ошибок ресурсов
  recordResourceError(event) {
    const error = {
      type: 'resource_error',
      target: event.target.tagName,
      url: event.target.src || event.target.href,
      timestamp: Date.now()
    };
    
    if (!this.metrics.has('errors')) {
      this.metrics.set('errors', []);
    }
    
    this.metrics.get('errors').push(error);
    this.notifyObservers('error', error);
  }
  
  // Проверка медленных ресурсов
  checkSlowResources() {
    if (performance.getEntriesByType) {
      const resources = performance.getEntriesByType('resource');
      const slowResources = resources.filter(resource => resource.duration > 5000); // Более 5 секунд
      
      if (slowResources.length > 0) {
        const metrics = {
          type: 'slow_resources',
          count: slowResources.length,
          resources: slowResources.map(r => ({
            name: r.name,
            duration: r.duration,
            type: r.initiatorType
          }))
        };
        
        this.notifyObservers('slow_resources', metrics);
      }
    }
  }
  
  // Мониторинг пользовательского опыта
  setupUserExperienceMonitoring() {
    // Мониторинг времени взаимодействия
    let firstInteractionTime = null;
    
    const interactionHandler = () => {
      if (!firstInteractionTime) {
        firstInteractionTime = performance.now();
        this.metrics.set('first_interaction', firstInteractionTime);
        this.notifyObservers('first_interaction', firstInteractionTime);
      }
    };
    
    document.addEventListener('click', interactionHandler, true);
    document.addEventListener('keydown', interactionHandler, true);
    
    // Мониторинг скролла
    let scrollCount = 0;
    let lastScrollTime = 0;
    
    window.addEventListener('scroll', () => {
      scrollCount++;
      const now = Date.now();
      
      if (now - lastScrollTime > 1000) { // Более 1 секунды между скроллами
        this.metrics.set('scroll_activity', scrollCount);
        lastScrollTime = now;
        scrollCount = 0;
      }
    }, { passive: true });
  }
  
  // Очистка наблюдателей
  cleanupObservers() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers.clear();
  }
  
  // Уведомление наблюдателей
  notifyObservers(type, data) {
    // В реальной реализации здесь будет логика уведомления
    console.log(`Метрика ${type}:`, data);
  }
  
  // Получение метрик
  getMetrics() {
    const result = {};
    for (const [key, value] of this.metrics) {
      result[key] = value;
    }
    return result;
  }
  
  // Экспорт метрик
  exportMetrics() {
    return JSON.stringify({
      timestamp: Date.now(),
      metrics: this.getMetrics(),
      userAgent: navigator.userAgent,
      url: location.href
    });
  }
}

// Расширенный мониторинг производительности
class AdvancedPerformanceMonitoring {
  constructor() {
    this.clientMonitor = new ClientPerformanceMonitoring();
    this.customMetrics = new Map();
    this.performanceThresholds = {
      pageLoad: 3000, // 3 секунды
      firstPaint: 1000, // 1 секунда
      resourceLoad: 2000 // 2 секунды
    };
  }
  
  // Запуск расширенного мониторинга
  start() {
    this.clientMonitor.start();
    this.setupCustomMetrics();
    this.setupPerformanceBudgets();
  }
  
  // Настройка пользовательских метрик
  setupCustomMetrics() {
    // Мониторинг времени выполнения функций
    this.instrumentFunctions();
    
    // Мониторинг памяти
    this.setupMemoryMonitoring();
    
    // Мониторинг FPS
    this.setupFPSMonitoring();
  }
  
  // Инструментирование функций
  instrumentFunctions() {
    // Пример инструментирования функции
    const originalFunction = window.someImportantFunction;
    if (originalFunction) {
      window.someImportantFunction = (...args) => {
        const startTime = performance.now();
        const result = originalFunction.apply(this, args);
        const endTime = performance.now();
        
        this.recordCustomMetric('function_execution', {
          name: 'someImportantFunction',
          duration: endTime - startTime,
          timestamp: Date.now()
        });
        
        return result;
      };
    }
  }
  
  // Мониторинг памяти
  setupMemoryMonitoring() {
    if (performance.memory) {
      setInterval(() => {
        const memory = performance.memory;
        this.recordCustomMetric('memory_usage', {
          used: memory.usedJSHeapSize,
          total: memory.totalJSHeapSize,
          limit: memory.jsHeapSizeLimit,
          timestamp: Date.now()
        });
      }, 5000); // Каждые 5 секунд
    }
  }
  
  // Мониторинг FPS
  setupFPSMonitoring() {
    let frameCount = 0;
    let lastTime = performance.now();
    let fps = 0;
    
    const measureFPS = () => {
      frameCount++;
      const now = performance.now();
      
      if (now - lastTime >= 1000) { // Каждую секунду
        fps = Math.round((frameCount * 1000) / (now - lastTime));
        this.recordCustomMetric('fps', {
          value: fps,
          timestamp: Date.now()
        });
        
        frameCount = 0;
        lastTime = now;
      }
      
      if (this.clientMonitor.isMonitoring) {
        requestAnimationFrame(measureFPS);
      }
    };
    
    requestAnimationFrame(measureFPS);
  }
  
  // Запись пользовательских метрик
  recordCustomMetric(name, data) {
    if (!this.customMetrics.has(name)) {
      this.customMetrics.set(name, []);
    }
    
    this.customMetrics.get(name).push(data);
    this.checkPerformanceThresholds(name, data);
  }
  
  // Проверка пороговых значений производительности
  checkPerformanceThresholds(metricName, data) {
    switch (metricName) {
      case 'function_execution':
        if (data.duration > 100) { // Более 100 мс
          this.reportPerformanceIssue('slow_function', {
            function: data.name,
            duration: data.duration
          });
        }
        break;
        
      case 'fps':
        if (data.value < 30) { // Меньше 30 FPS
          this.reportPerformanceIssue('low_fps', {
            fps: data.value
          });
        }
        break;
        
      case 'memory_usage':
        const usagePercent = (data.used / data.limit) * 100;
        if (usagePercent > 80) { // Более 80% памяти
          this.reportPerformanceIssue('high_memory_usage', {
            usage: usagePercent,
            used: data.used,
            limit: data.limit
          });
        }
        break;
    }
  }
  
  // Настройка бюджетов производительности
  setupPerformanceBudgets() {
    // Проверка бюджетов при загрузке страницы
    window.addEventListener('load', () => {
      setTimeout(() => {
        this.checkPerformanceBudgets();
      }, 1000); // Небольшая задержка для завершения метрик
    });
  }
  
  // Проверка бюджетов производительности
  checkPerformanceBudgets() {
    const navigationMetrics = this.clientMonitor.metrics.get('navigation');
    if (navigationMetrics) {
      if (navigationMetrics.pageLoadTime > this.performanceThresholds.pageLoad) {
        this.reportPerformanceIssue('page_load_budget_exceeded', {
          actual: navigationMetrics.pageLoadTime,
          budget: this.performanceThresholds.pageLoad
        });
      }
    }
    
    const paintMetrics = this.clientMonitor.metrics.get('paint');
    if (paintMetrics) {
      const firstPaint = paintMetrics.find(m => m.name === 'first-paint');
      if (firstPaint && firstPaint.startTime > this.performanceThresholds.firstPaint) {
        this.reportPerformanceIssue('first_paint_budget_exceeded', {
          actual: firstPaint.startTime,
          budget: this.performanceThresholds.firstPaint
        });
      }
    }
  }
  
  // Сообщение о проблемах производительности
  reportPerformanceIssue(type, details) {
    const issue = {
      type,
      details,
      timestamp: Date.now(),
      url: location.href
    };
    
    console.warn('Проблема производительности:', issue);
    
    // В реальной реализации здесь будет отправка в аналитику
    this.sendToAnalytics('performance_issue', issue);
  }
  
  // Отправка в аналитику
  sendToAnalytics(eventType, data) {
    // Имитация отправки в аналитику
    console.log(`Отправка в аналитику: ${eventType}`, data);
  }
  
  // Получение всех метрик
  getAllMetrics() {
    return {
      ...this.clientMonitor.getMetrics(),
      customMetrics: Object.fromEntries(this.customMetrics)
    };
  }
  
  // Генерация отчета о производительности
  generatePerformanceReport() {
    const metrics = this.getAllMetrics();
    
    return {
      timestamp: new Date().toISOString(),
      url: location.href,
      userAgent: navigator.userAgent,
      metrics: metrics,
      performanceIssues: this.getPerformanceIssues(),
      summary: this.generateSummary(metrics)
    };
  }
  
  // Получение проблем производительности
  getPerformanceIssues() {
    // В реальной реализации здесь будет логика получения проблем
    return [];
  }
  
  // Генерация сводки
  generateSummary(metrics) {
    const summary = {};
    
    if (metrics.navigation) {
      summary.pageLoadTime = metrics.navigation.pageLoadTime;
      summary.domContentLoadedTime = metrics.navigation.domContentLoadedTime;
    }
    
    if (metrics.paint) {
      const firstPaint = metrics.paint.find(m => m.name === 'first-paint');
      if (firstPaint) {
        summary.firstPaintTime = firstPaint.startTime;
      }
    }
    
    return summary;
  }
}

// Пример использования мониторинга производительности
class PerformanceDashboard {
  constructor() {
    this.monitor = new AdvancedPerformanceMonitoring();
    this.isRunning = false;
  }
  
  // Запуск мониторинга
  start() {
    if (this.isRunning) return;
    
    this.monitor.start();
    this.isRunning = true;
    console.log('Мониторинг производительности запущен');
  }
  
  // Остановка мониторинга
  stop() {
    if (!this.isRunning) return;
    
    this.monitor.clientMonitor.stop();
    this.isRunning = false;
    console.log('Мониторинг производительности остановлен');
  }
  
  // Получение текущих метрик
  getCurrentMetrics() {
    return this.monitor.getAllMetrics();
  }
  
  // Генерация отчета
  generateReport() {
    return this.monitor.generatePerformanceReport();
  }
  
  // Экспорт метрик
  exportMetrics() {
    const metrics = this.monitor.clientMonitor.exportMetrics();
    // В реальной реализации здесь будет сохранение в файл или отправка на сервер
    console.log('Экспорт метрик:', metrics);
    return metrics;
  }
}
```

## Серверный мониторинг

Мониторинг производительности на стороне сервера:

```javascript
// Серверный мониторинг производительности
class ServerPerformanceMonitoring {
  constructor() {
    this.metrics = new Map();
    this.requestTimings = new Map();
    this.errorCounts = new Map();
    this.slowRequests = [];
  }
  
  // Мониторинг HTTP запросов
  monitorHttpRequests(app) {
    app.use((req, res, next) => {
      const startTime = process.hrtime.bigint();
      const requestId = `${req.method}_${req.url}_${Date.now()}`;
      
      // Сохраняем время начала запроса
      this.requestTimings.set(requestId, {
        startTime,
        method: req.method,
        url: req.url,
        userAgent: req.get('User-Agent')
      });
      
      // Мониторинг завершения запроса
      const originalSend = res.send;
      res.send = (body) => {
        const endTime = process.hrtime.bigint();
        const duration = Number(endTime - startTime) / 1000000; // В миллисекундах
        
        // Записываем метрики
        this.recordRequestMetrics(req, res, duration);
        
        // Удаляем время запроса
        this.requestTimings.delete(requestId);
        
        return originalSend.call(res, body);
      };
      
      next();
    });
  }
  
  // Запись метрик запросов
  recordRequestMetrics(req, res, duration) {
    // Общие метрики запросов
    const method = req.method;
    const statusCode = res.statusCode;
    
    // Обновляем счетчики
    this.incrementMetric(`requests_${method}`);
    this.incrementMetric(`responses_${statusCode}`);
    
    // Записываем время выполнения
    this.recordTimingMetric('request_duration', duration);
    
    // Отслеживаем медленные запросы
    if (duration > 1000) { // Более 1 секунды
      this.recordSlowRequest(req, res, duration);
    }
    
    // Отслеживаем ошибки
    if (statusCode >= 400) {
      this.incrementErrorCount(`${method}_${req.url}`, statusCode);
    }
  }
  
  // Увеличение счетчика метрик
  incrementMetric(name, value = 1) {
    const current = this.metrics.get(name) || 0;
    this.metrics.set(name, current + value);
  }
  
  // Запись метрик времени выполнения
  recordTimingMetric(name, duration) {
    if (!this.metrics.has(`${name}_total`)) {
      this.metrics.set(`${name}_total`, 0);
      this.metrics.set(`${name}_count`, 0);
      this.metrics.set(`${name}_max`, 0);
      this.metrics.set(`${name}_min`, Infinity);
    }
    
    const total = this.metrics.get(`${name}_total`) + duration;
    const count = this.metrics.get(`${name}_count`) + 1;
    const max = Math.max(this.metrics.get(`${name}_max`), duration);
    const min = Math.min(this.metrics.get(`${name}_min`), duration);
    
    this.metrics.set(`${name}_total`, total);
    this.metrics.set(`${name}_count`, count);
    this.metrics.set(`${name}_max`, max);
    this.metrics.set(`${name}_min`, min);
    
    // Вычисляем среднее значение
    this.metrics.set(`${name}_avg`, total / count);
  }
  
  // Запись медленных запросов
  recordSlowRequest(req, res, duration) {
    this.slowRequests.push({
      method: req.method,
      url: req.url,
      duration,
      timestamp: Date.now(),
      userAgent: req.get('User-Agent'),
      statusCode: res.statusCode
    });
    
    // Ограничиваем размер массива
    if (this.slowRequests.length > 100) {
      this.slowRequests.shift();
    }
  }
  
  // Увеличение счетчика ошибок
  incrementErrorCount(endpoint, statusCode) {
    const key = `${endpoint}_${statusCode}`;
    const current = this.errorCounts.get(key) || 0;
    this.errorCounts.set(key, current + 1);
  }
  
  // Мониторинг базы данных
  monitorDatabaseQueries(db) {
    // Перехватываем методы базы данных
    const originalQuery = db.query;
    
    db.query = (sql, params, callback) => {
      const startTime = process.hrtime.bigint();
      
      return originalQuery.call(db, sql, params, (error, results) => {
        const endTime = process.hrtime.bigint();
        const duration = Number(endTime - startTime) / 1000000; // В миллисекундах
        
        // Записываем метрики запроса
        this.recordDatabaseMetrics(sql, duration, error);
        
        if (callback) {
          callback(error, results);
        }
      });
    };
  }
  
  // Запись метрик базы данных
  recordDatabaseMetrics(sql, duration, error) {
    // Общие метрики
    this.incrementMetric('database_queries');
    this.recordTimingMetric('database_query_duration', duration);
    
    // Медленные запросы
    if (duration > 500) { // Более 500 мс
      this.incrementMetric('slow_database_queries');
    }
    
    // Ошибки
    if (error) {
      this.incrementMetric('database_errors');
    }
    
    // Анализ SQL запросов
    const queryType = this.getQueryType(sql);
    this.incrementMetric(`database_${queryType}_queries`);
  }
  
  // Определение типа SQL запроса
  getQueryType(sql) {
    const upperSql = sql.trim().toUpperCase();
    if (upperSql.startsWith('SELECT')) return 'select';
    if (upperSql.startsWith('INSERT')) return 'insert';
    if (upperSql.startsWith('UPDATE')) return 'update';
    if (upperSql.startsWith('DELETE')) return 'delete';
    return 'other';
  }
  
  // Мониторинг памяти
  monitorMemoryUsage() {
    setInterval(() => {
      const memoryUsage = process.memoryUsage();
      
      this.recordMetric('memory_rss', memoryUsage.rss);
      this.recordMetric('memory_heap_total', memoryUsage.heapTotal);
      this.recordMetric('memory_heap_used', memoryUsage.heapUsed);
      this.recordMetric('memory_external', memoryUsage.external);
      
      // Проверяем утечки памяти
      this.checkMemoryLeaks(memoryUsage);
    }, 30000); // Каждые 30 секунд
  }
  
  // Запись метрик
  recordMetric(name, value) {
    this.metrics.set(name, value);
  }
  
  // Проверка утечек памяти
  checkMemoryLeaks(memoryUsage) {
    const heapUsed = memoryUsage.heapUsed;
    const heapTotal = memoryUsage.heapTotal;
    const usagePercent = (heapUsed / heapTotal) * 100;
    
    if (usagePercent > 85) { // Более 85% использования
      console.warn(`Высокое использование памяти: ${usagePercent.toFixed(2)}%`);
      this.incrementMetric('high_memory_usage_warnings');
    }
  }
  
  // Мониторинг CPU
  monitorCPUUsage() {
    let lastUsage = process.cpuUsage();
    
    setInterval(() => {
      const currentUsage = process.cpuUsage();
      const userDiff = currentUsage.user - lastUsage.user;
      const systemDiff = currentUsage.system - lastUsage.system;
      const totalDiff = userDiff + systemDiff;
      
      const cpuPercent = (totalDiff / 1000000) * 100; // Примерное значение
      
      this.recordMetric('cpu_usage_percent', cpuPercent);
      
      if (cpuPercent > 80) { // Более 80% использования
        console.warn(`Высокое использование CPU: ${cpuPercent.toFixed(2)}%`);
        this.incrementMetric('high_cpu_usage_warnings');
      }
      
      lastUsage = currentUsage;
    }, 5000); // Каждые 5 секунд
  }
  
  // Получение всех метрик
  getMetrics() {
    const result = {};
    
    for (const [key, value] of this.metrics) {
      result[key] = value;
    }
    
    // Добавляем вычисленные метрики
    if (result.request_duration_count > 0) {
      result.requests_per_second = result.request_duration_count / 60; // За последнюю минуту
    }
    
    return result;
  }
  
  // Получение медленных запросов
  getSlowRequests(limit = 10) {
    return this.slowRequests.slice(-limit);
  }
  
  // Получение ошибок
  getErrors() {
    const errors = [];
    for (const [key, count] of this.errorCounts) {
      errors.push({ endpoint: key, count });
    }
    return errors.sort((a, b) => b.count - a.count);
  }
  
  // Генерация отчета
  generateReport() {
    return {
      timestamp: new Date().toISOString(),
      metrics: this.getMetrics(),
      slowRequests: this.getSlowRequests(),
      errors: this.getErrors(),
      summary: this.generateSummary()
    };
  }
  
  // Генерация сводки
  generateSummary() {
    const metrics = this.getMetrics();
    
    return {
      totalRequests: metrics.requests_GET + metrics.requests_POST + (metrics.requests_PUT || 0) + (metrics.requests_DELETE || 0),
      averageResponseTime: metrics.request_duration_avg || 0,
      errorRate: metrics.database_errors ? (metrics.database_errors / (metrics.database_queries || 1)) * 100 : 0,
      memoryUsage: metrics.memory_heap_used || 0,
      cpuUsage: metrics.cpu_usage_percent || 0
    };
  }
}

// Пример использования серверного мониторинга
class ServerMonitor {
  constructor() {
    this.performanceMonitor = new ServerPerformanceMonitoring();
  }
  
  // Инициализация мониторинга
  init(app, db) {
    // Мониторинг HTTP запросов
    this.performanceMonitor.monitorHttpRequests(app);
    
    // Мониторинг базы данных
    if (db) {
      this.performanceMonitor.monitorDatabaseQueries(db);
    }
    
    // Мониторинг ресурсов
    this.performanceMonitor.monitorMemoryUsage();
    this.performanceMonitor.monitorCPUUsage();
    
    console.log('Серверный мониторинг инициализирован');
  }
  
  // Получение метрик
  getMetrics() {
    return this.performanceMonitor.getMetrics();
  }
  
  // Получение отчета
  getReport() {
    return this.performanceMonitor.generateReport();
  }
  
  // Экспорт метрик
  exportMetrics() {
    const report = this.getReport();
    // В реальной реализации здесь будет сохранение в файл или отправка в систему мониторинга
    console.log('Экспорт метрик сервера:', JSON.stringify(report, null, 2));
    return report;
  }
}
```

## Интеграция с системами мониторинга

Интеграция с популярными системами мониторинга:

```javascript
// Интеграция с системами мониторинга
class MonitoringIntegration {
  constructor() {
    this.integrations = new Map();
  }
  
  // Интеграция с Prometheus
  setupPrometheusIntegration() {
    const prometheusMetrics = new Map();
    
    return {
      // Регистрация метрики
      registerGauge(name, help, labels = []) {
        prometheusMetrics.set(name, {
          type: 'gauge',
          help,
          labels,
          values: new Map()
        });
      },
      
      // Установка значения метрики
      setGauge(name, value, labelValues = {}) {
        const metric = prometheusMetrics.get(name);
        if (metric) {
          const labelKey = JSON.stringify(labelValues);
          metric.values.set(labelKey, value);
        }
      },
      
      // Получение метрик в формате Prometheus
      getMetrics() {
        let result = '';
        
        for (const [name, metric] of prometheusMetrics) {
          result += `# HELP ${name} ${metric.help}\n`;
          result += `# TYPE ${name} ${metric.type}\n`;
          
          for (const [labelKey, value] of metric.values) {
            const labels = JSON.parse(labelKey);
            const labelString = Object.entries(labels)
              .map(([key, val]) => `${key}="${val}"`)
              .join(',');
            
            if (labelString) {
              result += `${name}{${labelString}} ${value}\n`;
            } else {
              result += `${name} ${value}\n`;
            }
          }
        }
        
        return result;
      }
    };
  }
  
  // Интеграция с New Relic
  setupNewRelicIntegration() {
    // Проверяем наличие New Relic агента
    if (typeof newrelic === 'undefined') {
      console.warn('New Relic agent not found');
      return null;
    }
    
    return {
      // Отправка пользовательских метрик
      recordCustomMetric(name, value) {
        newrelic.recordMetric(name, value);
      },
      
      // Отправка событий
      recordCustomEvent(eventType, attributes) {
        newrelic.recordCustomEvent(eventType, attributes);
      },
      
      // Отправка ошибок
      noticeError(error, customAttributes = {}) {
        newrelic.noticeError(error, customAttributes);
      },
      
      // Добавление атрибутов к транзакции
      addCustomParameter(name, value) {
        newrelic.addCustomParameter(name, value);
      }
    };
  }
  
  // Интеграция с DataDog
  setupDataDogIntegration() {
    return {
      // Отправка метрик
      sendMetric(metricName, points, tags = []) {
        // В реальной реализации здесь будет отправка в DataDog API
        console.log(`Отправка метрики в DataDog: ${metricName}`, { points, tags });
      },
      
      // Отправка событий
      sendEvent(title, text, tags = []) {
        console.log(`Отправка события в DataDog: ${title}`, { text, tags });
      },
      
      // Отправка логов
      sendLog(message, level = 'info', tags = []) {
        console.log(`Отправка лога в DataDog: ${level}`, { message, tags });
      }
    };
  }
  
  // Интеграция с Sentry
  setupSentryIntegration() {
    // Проверяем наличие Sentry
    if (typeof Sentry === 'undefined') {
      console.warn('Sentry not found');
      return null;
    }
    
    return {
      // Отправка ошибок
      captureException(error, context = {}) {
        Sentry.captureException(error, {
          contexts: {
            performance: context
          }
        });
      },
      
      // Отправка сообщений
      captureMessage(message, level = 'info', context = {}) {
        Sentry.captureMessage(message, {
          level,
          contexts: {
            performance: context
          }
        });
      },
      
      // Добавление контекста
      setContext(key, context) {
        Sentry.setContext(key, context);
      },
      
      // Добавление тегов
      setTag(key, value) {
        Sentry.setTag(key, value);
      }
    };
  }
  
  // Универсальная система отправки метрик
  setupUniversalMetricsSender() {
    const integrations = {
      prometheus: this.setupPrometheusIntegration(),
      newrelic: this.setupNewRelicIntegration(),
      datadog: this.setupDataDogIntegration(),
      sentry: this.setupSentryIntegration()
    };
    
    return {
      // Отправка метрики во все интеграции
      sendMetric(name, value, tags = {}) {
        // Prometheus
        if (integrations.prometheus) {
          integrations.prometheus.setGauge(name, value, tags);
        }
        
        // DataDog
        if (integrations.datadog) {
          integrations.datadog.sendMetric(name, value, Object.entries(tags).map(([k, v]) => `${k}:${v}`));
        }
        
        // New Relic
        if (integrations.newrelic) {
          integrations.newrelic.recordCustomMetric(name, value);
          Object.entries(tags).forEach(([key, val]) => {
            integrations.newrelic.addCustomParameter(key, val);
          });
        }
      },
      
      // Отправка ошибки во все интеграции
      sendError(error, context = {}) {
        // Sentry
        if (integrations.sentry) {
          integrations.sentry.captureException(error, context);
        }
        
        // DataDog
        if (integrations.datadog) {
          integrations.datadog.sendEvent('error', error.message, ['error']);
        }
      },
      
      // Отправка события
      sendEvent(eventType, data = {}) {
        // DataDog
        if (integrations.datadog) {
          integrations.datadog.sendEvent(eventType, JSON.stringify(data));
        }
        
        // New Relic
        if (integrations.newrelic) {
          integrations.newrelic.recordCustomEvent(eventType, data);
        }
      }
    };
  }
}

// Пример комплексного мониторинга
class ComprehensiveMonitoring {
  constructor() {
    this.clientMonitoring = new AdvancedPerformanceMonitoring();
    this.serverMonitoring = new ServerPerformanceMonitoring();
    this.integration = new MonitoringIntegration();
    this.metricsSender = this.integration.setupUniversalMetricsSender();
  }
  
  // Инициализация клиентского мониторинга
  initClientMonitoring() {
    this.clientMonitoring.start();
    
    // Отправка метрик в интеграции
    setInterval(() => {
      const metrics = this.clientMonitoring.getAllMetrics();
      
      // Отправляем основные метрики
      if (metrics.navigation && metrics.navigation.pageLoadTime) {
        this.metricsSender.sendMetric('page_load_time', metrics.navigation.pageLoadTime);
      }
      
      if (metrics.paint) {
        const firstPaint = metrics.paint.find(m => m.name === 'first-paint');
        if (firstPaint) {
          this.metricsSender.sendMetric('first_paint_time', firstPaint.startTime);
        }
      }
    }, 30000); // Каждые 30 секунд
  }
  
  // Инициализация серверного мониторинга
  initServerMonitoring(app, db) {
    this.serverMonitoring.monitorHttpRequests(app);
    
    if (db) {
      this.serverMonitoring.monitorDatabaseQueries(db);
    }
    
    this.serverMonitoring.monitorMemoryUsage();
    this.serverMonitoring.monitorCPUUsage();
    
    // Отправка метрик в интеграции
    setInterval(() => {
      const metrics = this.serverMonitoring.getMetrics();
      
      // Отправляем основные метрики
      Object.entries(metrics).forEach(([name, value]) => {
        this.metricsSender.sendMetric(`server_${name}`, value);
      });
    }, 60000); // Каждую минуту
  }
  
  // Обработка ошибок
  handleError(error, context = {}) {
    this.metricsSender.sendError(error, context);
  }
  
  // Отправка пользовательских событий
  trackEvent(eventType, data = {}) {
    this.metricsSender.sendEvent(eventType, data);
  }
  
  // Получение отчетов
  getReports() {
    return {
      client: this.clientMonitoring.generatePerformanceReport(),
      server: this.serverMonitoring.generateReport()
    };
  }
  
  // Экспорт всех данных
  exportAllData() {
    const reports = this.getReports();
    const exportData = {
      timestamp: Date.now(),
      reports,
      userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : 'server',
      url: typeof location !== 'undefined' ? location.href : 'server'
    };
    
    console.log('Экспорт всех данных мониторинга:', JSON.stringify(exportData, null, 2));
    return exportData;
  }
}
```

## Практические рекомендации

1. **Используйте Performance API** - для точных измерений
2. **Применяйте PerformanceObserver** - для автоматического сбора метрик
3. **Мониторьте ключевые метрики** - FCP, LCP, FID, CLS
4. **Используйте интеграцию с системами мониторинга** - Prometheus, DataDog, New Relic
5. **Отправляйте метрики регулярно** - для отслеживания трендов
6. **Мониторьте ошибки** - с контекстом производительности
7. **Используйте пользовательские метрики** - для бизнес-логики
8. **Настраивайте алерты** - для критических порогов

Мониторинг производительности - важный аспект создания высокопроизводительных веб-приложений. Правильная настройка мониторинга помогает выявлять проблемы на ранних стадиях и обеспечивает оптимальный пользовательский опыт.

#javascript #monitoring #performance #optimization #metrics #analytics #prometheus #sentry