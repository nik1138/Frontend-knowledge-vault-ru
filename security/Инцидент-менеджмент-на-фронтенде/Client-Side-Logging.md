---
aliases: [Клиентское логирование, Логирование на фронтенде, Frontend Logging]
tags: [security, logging, monitoring, frontend, incident-management, observability]
---

# Клиентское логирование и безопасность

## Введение в клиентское логирование

Клиентское логирование — это процесс сбора, регистрации и анализа событий, ошибок и действий пользователей на стороне клиента (браузера или мобильного приложения). В отличие от серверного логирования, клиентское логирование предоставляет информацию о реальном опыте пользователя и позволяет выявлять проблемы, которые происходят непосредственно в браузере.

Клиентское логирование играет важную роль в современных веб-приложениях, особенно в одностраничных приложениях (SPA), где значительная часть логики выполняется на стороне клиента. Это позволяет разработчикам получить представление о реальной производительности, обнаруживать ошибки, которые не могут быть воспроизведены в тестовой среде, и отслеживать поведение пользователей.

### Основные особенности клиентского логирования

- **Реальное время взаимодействия**: Логи содержат информацию о действиях пользователей в реальном времени
- **Ошибки в браузере**: Регистрация исключений JavaScript, ошибок сети и других клиентских ошибок
- **Производительность**: Отслеживание времени загрузки, FPS, времени выполнения операций
- **Поведение пользователей**: Логирование кликов, навигации, заполнения форм и других действий

## Зачем нужно логирование на клиенте

Логирование на клиенте необходимо по нескольким важным причинам:

### 1. Отладка в продакшене

Клиентские ошибки часто не могут быть воспроизведены в тестовой среде из-за различий в:
- Браузерах и их версиях
- Операционных системах
- Расширениях браузера
- Сетевых условиях
- Геолокации пользователя

### 2. Мониторинг пользовательского опыта

Логирование позволяет:
- Отслеживать реальную производительность приложения
- Выявлять узкие места в пользовательском интерфейсе
- Анализировать поведение пользователей
- Определять наиболее популярные функции приложения

### 3. Обнаружение инцидентов безопасности

Клиентское логирование может помочь в обнаружении:
- Попыток XSS-атак
- Подозрительной активности пользователей
- Нарушений политик безопасности
- Несанкционированного доступа к ресурсам

### 4. Улучшение качества приложения

Собранные логи позволяют:
- Быстро реагировать на ошибки
- Принимать обоснованные решения о развитии приложения
- Повышать стабильность и надежность системы

## Типы событий для логирования

### Ошибки JavaScript

```javascript
window.addEventListener('error', (event) => {
  logError({
    message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
    stack: event.error?.stack
  });
});

window.addEventListener('unhandledrejection', (event) => {
  logError({
    type: 'unhandledrejection',
    reason: event.reason,
    promise: event.promise
  });
});
```

### Ошибки сети

```javascript
// Логирование сетевых ошибок
fetch('/api/data')
  .then(response => {
    if (!response.ok) {
      logNetworkError({
        url: response.url,
        status: response.status,
        statusText: response.statusText
      });
    }
    return response.json();
  })
  .catch(error => {
    logNetworkError({
      url: '/api/data',
      error: error.message
    });
  });
```

### Действия пользователей

```javascript
// Логирование кликов
document.addEventListener('click', (event) => {
  logUserAction({
    type: 'click',
    element: event.target.tagName,
    id: event.target.id,
    class: event.target.className,
    timestamp: Date.now()
  });
});

// Логирование заполнения форм
document.addEventListener('input', (event) => {
  if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA') {
    logUserAction({
      type: 'input',
      field: event.target.name || event.target.id,
      timestamp: Date.now()
    });
  }
});
```

### Производительность

```javascript
// Логирование производительности
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    logPerformance({
      name: entry.name,
      duration: entry.duration,
      entryType: entry.entryType
    });
  }
});

observer.observe({ entryTypes: ['measure', 'navigation', 'paint'] });
```

### Безопасность

```javascript
// Логирование подозрительной активности
document.addEventListener('paste', (event) => {
  const paste = (event.clipboardData || window.clipboardData).getData('text');
  if (paste.includes('<script>') || paste.includes('javascript:')) {
    logSecurityEvent({
      type: 'suspicious-paste',
      content: paste.substring(0, 100),
      timestamp: Date.now()
    });
  }
});
```

## Безопасность логов

### Уязвимости, связанные с логированием

Клиентское логирование может стать источником уязвимостей, если:

- В логи попадают конфиденциальные данные пользователя
- Логи отправляются без шифрования
- Логи содержат информацию о структуре приложения
- Логи могут быть использованы для анализа поведения пользователей

### Защита от утечки данных

```javascript
// Функция фильтрации конфиденциальных данных
function sanitizeLogData(data) {
  const sanitized = { ...data };
  
  // Удаление чувствительных полей
  delete sanitized.password;
  delete sanitized.token;
  delete sanitized.creditCard;
  
  // Обрезка длинных строк
  for (const key in sanitized) {
    if (typeof sanitized[key] === 'string' && sanitized[key].length > 100) {
      sanitized[key] = sanitized[key].substring(0, 100) + '...';
    }
  }
  
  return sanitized;
}
```

### Шифрование логов

```javascript
// Пример шифрования логов перед отправкой
async function encryptLog(logData) {
  const encoder = new TextEncoder();
  const data = encoder.encode(JSON.stringify(logData));
  
  const iv = window.crypto.getRandomValues(new Uint8Array(12));
  const key = await window.crypto.subtle.importKey(
    'raw',
    encoder.encode('your-encryption-key'),
    { name: 'AES-GCM' },
    false,
    ['encrypt']
  );
  
  const encrypted = await window.crypto.subtle.encrypt(
    { name: 'AES-GCM', iv: iv },
    key,
    data
  );
  
  return {
    encrypted: Array.from(new Uint8Array(encrypted)),
    iv: Array.from(iv)
  };
}
```

## Стандарты и форматы логирования

### Структурированные логи

Логи должны быть структурированы в формате JSON для облегчения парсинга и анализа:

```json
{
  "timestamp": "2023-10-15T10:30:00.000Z",
  "level": "error",
  "message": "Network request failed",
  "context": {
    "url": "/api/users",
    "method": "GET",
    "status": 500,
    "userAgent": "Mozilla/5.0...",
    "userId": "12345"
  },
  "meta": {
    "version": "1.2.3",
    "environment": "production",
    "session": "abc123"
  }
}
```

### RFC 5424 (Syslog)

Хотя RFC 5424 в основном применяется к серверным системам, его принципы могут быть адаптированы для клиентского логирования:

- Уровни важности (severity levels)
- Стандартизированные форматы времени
- Единообразная структура сообщений

### Спецификации для веб-приложений

- **Web Performance API**: Стандарты для логирования производительности
- **Navigation Timing API**: Временные метки навигации
- **Resource Timing API**: Временные метки загрузки ресурсов

## Уровни логирования

### TRACE
Самый подробный уровень, содержит детали выполнения операций. Используется редко на клиенте из-за объема данных.

### DEBUG
Подробная информация для диагностики проблем. Может включать информацию о внутреннем состоянии приложения.

### INFO
Информационные сообщения о нормальной работе приложения, например, успешная загрузка данных.

### WARN
Предупреждения о потенциальных проблемах, которые не прерывают работу приложения.

### ERROR
Ошибки, которые не прерывают работу приложения, но требуют внимания.

### FATAL
Критические ошибки, приводящие к остановке приложения.

### Пример реализации уровней логирования

```javascript
class Logger {
  constructor(level = 'INFO') {
    this.level = level;
    this.levels = {
      'TRACE': 0,
      'DEBUG': 1,
      'INFO': 2,
      'WARN': 3,
      'ERROR': 4,
      'FATAL': 5
    };
  }

  log(level, message, context = {}) {
    if (this.levels[level] >= this.levels[this.level]) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level,
        message,
        context,
        meta: {
          version: APP_VERSION,
          userAgent: navigator.userAgent,
          url: window.location.href
        }
      };
      
      // Отправка лога
      this.sendLog(logEntry);
    }
  }

  trace(message, context) { this.log('TRACE', message, context); }
  debug(message, context) { this.log('DEBUG', message, context); }
  info(message, context) { this.log('INFO', message, context); }
  warn(message, context) { this.log('WARN', message, context); }
  error(message, context) { this.log('ERROR', message, context); }
  fatal(message, context) { this.log('FATAL', message, context); }
}
```

## Защита конфиденциальных данных

### Правила обработки чувствительной информации

1. **Никогда не логировать пароли**
2. **Фильтровать токены аутентификации**
3. **Анонимизировать персональные данные**
4. **Удалять номера кредитных карт**
5. **Не логировать содержимое форм с чувствительной информацией**

### Пример фильтрации данных

```javascript
const SENSITIVE_FIELDS = [
  'password', 'token', 'auth', 'secret', 'key', 
  'creditCard', 'cvv', 'ssn', 'email', 'phone'
];

function filterSensitiveData(obj, path = '') {
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }

  const result = Array.isArray(obj) ? [] : {};

  for (const [key, value] of Object.entries(obj)) {
    const currentPath = path ? `${path}.${key}` : key;
    
    if (SENSITIVE_FIELDS.some(field => 
      currentPath.toLowerCase().includes(field) || 
      key.toLowerCase().includes(field)
    )) {
      result[key] = '[FILTERED]';
    } else if (typeof value === 'object' && value !== null) {
      result[key] = filterSensitiveData(value, currentPath);
    } else {
      result[key] = value;
    }
  }

  return result;
}
```

### Проверка данных перед логированием

```javascript
function isValidForLogging(data) {
  // Проверка на наличие чувствительных данных
  const jsonStr = JSON.stringify(data);
  
  // Проверка на регулярные выражения для чувствительных данных
  const patterns = [
    /\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/, // кредитные карты
    /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/, // email
    /\b\d{3}-\d{2}-\d{4}\b/ // SSN
  ];
  
  return !patterns.some(pattern => pattern.test(jsonStr));
}
```

## Анонимизация данных

### Методы анонимизации

1. **Хеширование идентификаторов**
2. **Обезличивание персональных данных**
3. **Использование псевдонимов**
4. **Удаление прямых идентификаторов**

### Пример анонимизации пользовательских данных

```javascript
// Функция анонимизации пользовательских данных
function anonymizeUserData(userData) {
  return {
    userId: hash(userData.id), // хеширование ID
    email: userData.email ? hash(userData.email) : undefined,
    name: userData.name ? maskName(userData.name) : undefined,
    location: anonymizeLocation(userData.location),
    // остальные поля
  };
}

// Хеширование с использованием SHA-256
async function hash(data) {
  const encoder = new TextEncoder();
  const digest = await crypto.subtle.digest('SHA-256', encoder.encode(data));
  return Array.from(new Uint8Array(digest))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}

// Маскировка имени
function maskName(name) {
  if (name.length <= 2) return '*'.repeat(name.length);
  return name.charAt(0) + '*'.repeat(name.length - 2) + name.charAt(name.length - 1);
}

// Анонимизация геолокации
function anonymizeLocation(location) {
  if (!location) return location;
  
  // Округление координат до 2 знаков после запятой для анонимизации
  return {
    lat: Math.round(location.lat * 100) / 100,
    lng: Math.round(location.lng * 100) / 100
  };
}
```

### Генерация анонимных сессий

```javascript
function generateAnonymousSessionId() {
  // Генерация уникального ID без привязки к пользователю
  const timestamp = Date.now().toString(36);
  const random = Math.random().toString(36).substr(2, 5);
  return `${timestamp}-${random}`;
}
```

## Отправка логов на сервер

### Методы отправки

1. **Fetch API** - для обычных HTTP-запросов
2. **Beacon API** - для отправки данных при закрытии вкладки
3. **Image ping** - для отправки данных без CORS-ограничений
4. **WebSocket** - для потоковой отправки

### Beacon API для отправки логов

```javascript
// Использование Beacon API для отправки логов при закрытии страницы
window.addEventListener('beforeunload', () => {
  if (pendingLogs.length > 0) {
    navigator.sendBeacon(
      '/api/logs',
      JSON.stringify({ logs: pendingLogs })
    );
  }
});
```

### Отложенная отправка логов

```javascript
class LogBuffer {
  constructor(batchSize = 10, flushInterval = 5000) {
    this.buffer = [];
    this.batchSize = batchSize;
    this.flushInterval = flushInterval;
    this.timer = null;
    
    // Регулярная отправка накопленных логов
    this.timer = setInterval(() => {
      this.flush();
    }, this.flushInterval);
  }
  
  add(log) {
    this.buffer.push(log);
    
    if (this.buffer.length >= this.batchSize) {
      this.flush();
    }
  }
  
  flush() {
    if (this.buffer.length > 0) {
      const logsToSend = [...this.buffer];
      this.buffer = [];
      
      // Отправка логов на сервер
      this.sendLogs(logsToSend);
    }
  }
  
  async sendLogs(logs) {
    try {
      const response = await fetch('/api/logs/batch', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getAuthToken()}`
        },
        body: JSON.stringify({ logs })
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      console.error('Failed to send logs:', error);
      // Возврат логов в буфер для повторной отправки
      this.buffer.unshift(...logs);
    }
  }
}
```

### Обработка ошибок отправки

```javascript
async function sendLogWithRetry(log, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch('/api/logs', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(log)
      });
      
      if (response.ok) {
        return true;
      }
    } catch (error) {
      if (i === maxRetries - 1) {
        // Сохранение в локальное хранилище для последующей отправки
        saveLogLocally(log);
        return false;
      }
      
      // Ожидание перед повторной попыткой
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

## Хранение и архивация логов

### Локальное хранение логов

```javascript
class LocalLogStorage {
  constructor(maxSize = 1000) {
    this.maxSize = maxSize;
    this.storageKey = 'client_logs';
  }
  
  add(log) {
    const logs = this.getLogs();
    logs.push({
      ...log,
      storedAt: Date.now()
    });
    
    // Ограничение размера хранилища
    if (logs.length > this.maxSize) {
      logs.splice(0, logs.length - this.maxSize);
    }
    
    localStorage.setItem(this.storageKey, JSON.stringify(logs));
  }
  
  getLogs() {
    try {
      const stored = localStorage.getItem(this.storageKey);
      return stored ? JSON.parse(stored) : [];
    } catch {
      return [];
    }
  }
  
  clear() {
    localStorage.removeItem(this.storageKey);
  }
}
```

### Архивация логов

```javascript
// Архивация логов по датам
function archiveLogs() {
  const logs = getLogs();
  const today = new Date().toISOString().split('T')[0];
  const archiveKey = `logs_${today}`;
  
  // Сохранение в IndexedDB для больших объемов
  const request = indexedDB.open('LogArchive', 1);
  
  request.onupgradeneeded = function(event) {
    const db = event.target.result;
    if (!db.objectStoreNames.contains('logs')) {
      const store = db.createObjectStore('logs', { keyPath: 'timestamp' });
      store.createIndex('date', 'date', { unique: false });
    }
  };
  
  request.onsuccess = function(event) {
    const db = event.target.result;
    const transaction = db.transaction(['logs'], 'readwrite');
    const store = transaction.objectStore('logs');
    
    logs.forEach(log => store.add(log));
  };
}
```

## Мониторинг логов в реальном времени

### WebSocket для потоковой передачи логов

```javascript
class RealTimeLogMonitor {
  constructor(wsUrl) {
    this.wsUrl = wsUrl;
    this.ws = null;
    this.reconnectInterval = 5000;
    this.maxReconnectAttempts = 5;
    this.reconnectAttempts = 0;
  }
  
  connect() {
    this.ws = new WebSocket(this.wsUrl);
    
    this.ws.onopen = () => {
      console.log('Connected to log monitoring service');
      this.reconnectAttempts = 0;
    };
    
    this.ws.onmessage = (event) => {
      const log = JSON.parse(event.data);
      this.processLog(log);
    };
    
    this.ws.onclose = () => {
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        setTimeout(() => {
          this.reconnectAttempts++;
          this.connect();
        }, this.reconnectInterval);
      }
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }
  
  processLog(log) {
    // Обработка полученного лога
    console.log('Received log:', log);
    
    // Проверка на критические ошибки
    if (log.level === 'FATAL' || 
        (log.level === 'ERROR' && log.context?.critical)) {
      this.handleCriticalLog(log);
    }
  }
  
  handleCriticalLog(log) {
    // Уведомление о критической ошибке
    if ('Notification' in window && Notification.permission === 'granted') {
      new Notification('Критическая ошибка в приложении', {
        body: log.message,
        icon: '/error-icon.png'
      });
    }
  }
}
```

### Визуализация логов в реальном времени

```javascript
// Пример компонента для отображения логов в реальном времени
class LogViewer {
  constructor(containerId) {
    this.container = document.getElementById(containerId);
    this.maxEntries = 100;
  }
  
  addLog(log) {
    const logElement = document.createElement('div');
    logElement.className = `log-entry log-${log.level.toLowerCase()}`;
    logElement.innerHTML = `
      <span class="timestamp">${new Date(log.timestamp).toLocaleTimeString()}</span>
      <span class="level">${log.level}</span>
      <span class="message">${log.message}</span>
      <details>
        <summary>Детали</summary>
        <pre>${JSON.stringify(log.context, null, 2)}</pre>
      </details>
    `;
    
    this.container.prepend(logElement);
    
    // Ограничение количества отображаемых записей
    while (this.container.children.length > this.maxEntries) {
      this.container.removeChild(this.container.lastChild);
    }
  }
}
```

## Анализ логов

### Статистика ошибок

```javascript
class LogAnalyzer {
  constructor() {
    this.errorStats = new Map();
    this.performanceStats = new Map();
  }
  
  analyzeError(log) {
    if (log.level === 'ERROR' || log.level === 'FATAL') {
      const errorKey = this.getErrorKey(log);
      const stats = this.errorStats.get(errorKey) || {
        count: 0,
        firstSeen: log.timestamp,
        lastSeen: log.timestamp,
        users: new Set()
      };
      
      stats.count++;
      stats.lastSeen = log.timestamp;
      if (log.context?.userId) {
        stats.users.add(log.context.userId);
      }
      
      this.errorStats.set(errorKey, stats);
    }
  }
  
  getErrorKey(log) {
    // Создание ключа для группировки одинаковых ошибок
    return `${log.message}_${log.context?.url || ''}_${log.context?.stack?.substring(0, 50) || ''}`;
  }
  
  getTopErrors(limit = 10) {
    return Array.from(this.errorStats.entries())
      .sort(([, a], [, b]) => b.count - a.count)
      .slice(0, limit)
      .map(([key, stats]) => ({
        error: key,
        ...stats,
        userCount: stats.users.size
      }));
  }
}
```

### Анализ производительности

```javascript
// Анализ производительности на основе логов
function analyzePerformanceLogs(logs) {
  const performanceMetrics = {
    avgLoadTime: 0,
    maxLoadTime: 0,
    minLoadTime: Infinity,
    slowPages: [],
    resourceLoadTimes: []
  };
  
  const loadTimes = logs
    .filter(log => log.context?.type === 'performance' && log.context?.metric === 'loadTime')
    .map(log => log.context.value);
  
  if (loadTimes.length > 0) {
    performanceMetrics.avgLoadTime = loadTimes.reduce((a, b) => a + b, 0) / loadTimes.length;
    performanceMetrics.maxLoadTime = Math.max(...loadTimes);
    performanceMetrics.minLoadTime = Math.min(...loadTimes);
  }
  
  return performanceMetrics;
}
```

## Оповещения на основе логов

### Правила срабатывания оповещений

```javascript
class AlertSystem {
  constructor() {
    this.rules = [
      {
        name: 'High Error Rate',
        condition: (logs) => {
          const errorCount = logs.filter(log => log.level === 'ERROR').length;
          return errorCount > 10; // более 10 ошибок за интервал
        },
        severity: 'HIGH',
        message: 'Высокий уровень ошибок в приложении'
      },
      {
        name: 'Slow Performance',
        condition: (logs) => {
          const slowLoads = logs.filter(log => 
            log.context?.type === 'performance' && 
            log.context?.metric === 'loadTime' && 
            log.context?.value > 5000 // более 5 секунд
          );
          return slowLoads.length > 5;
        },
        severity: 'MEDIUM',
        message: 'Медленная загрузка страниц'
      }
    ];
  }
  
  checkAlerts(logs) {
    const triggeredAlerts = [];
    
    for (const rule of this.rules) {
      if (rule.condition(logs)) {
        triggeredAlerts.push({
          rule: rule.name,
          message: rule.message,
          severity: rule.severity,
          timestamp: new Date().toISOString()
        });
      }
    }
    
    return triggeredAlerts;
  }
}
```

### Отправка оповещений

```javascript
async function sendAlert(alert) {
  try {
    await fetch('/api/alerts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(alert)
    });
  } catch (error) {
    console.error('Failed to send alert:', error);
  }
}
```

## Лучшие практики

### 1. Минимизация объема логов

- Логируйте только значимые события
- Используйте фильтрацию для избежания дубликатов
- Ограничивайте частоту отправки логов

### 2. Защита конфиденциальности

- Обязательно фильтруйте чувствительные данные
- Используйте анонимизацию пользовательских данных
- Шифруйте логи при передаче

### 3. Эффективная структура логов

- Используйте структурированный формат (JSON)
- Включайте контекстную информацию
- Добавляйте уникальные идентификаторы для отслеживания

### 4. Управление ресурсами

- Ограничивайте размер буфера логов
- Используйте эффективные методы отправки
- Учитывайте влияние на производительность

### 5. Тестирование системы логирования

- Тестируйте систему логирования отдельно
- Проверяйте обработку ошибок отправки
- Обеспечьте отказоустойчивость

## Связанные материалы

- [[Инцидент-менеджмент-на-фронтенде]] - Общая информация об управлении инцидентами на фронтенде
- [[Отладка-безопасности]] - Методы отладки и анализа безопасности
- [[Мониторинг-безопасности]] - Системы мониторинга безопасности
- [[Безопасность-в-SPA]] - Особенности безопасности в одностраничных приложениях
- [[Secure-Storage]] - Безопасное хранение данных в браузере
- [[XSS-защита]] - Защита от межсайтового скриптинга
- [[CSRF-защита]] - Защита от подделки межсайтовых запросов
- [[HTTP-Security-Headers]] - Заголовки безопасности HTTP
- [[Content-Security-Policy]] - Политика безопасности контента
- [[Управление-сессиями-и-аутентификацией]] - Управление сессиями и аутентификацией
- [[Обработка-персональных-данных]] - Работа с персональными данными
- [[Шифрование-на-клиенте]] - Методы шифрования на клиенте
- [[Безопасность-в-облачных-средах]] - Безопасность в облачных средах
- [[Мониторинг-безопасности-в-продакшене]] - Мониторинг безопасности в продакшене
- [[Идентификация-и-отслеживание-угроз]] - Методы идентификации и отслеживания угроз

## Заключение

Клиентское логирование является важным инструментом для обеспечения безопасности и надежности веб-приложений. При правильной реализации оно позволяет:

- Быстро выявлять и устранять ошибки
- Обнаруживать потенциальные угрозы безопасности
- Повышать качество пользовательского опыта
- Обеспечивать прозрачность работы приложения

Важно помнить, что система логирования сама по себе должна быть безопасной и не становиться источником уязвимостей. Правильная защита конфиденциальных данных, шифрование передаваемой информации и анонимизация пользовательских данных являются ключевыми аспектами безопасного клиентского логирования.
