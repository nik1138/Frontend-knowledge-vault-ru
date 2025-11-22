---
aliases: ["Безопасность Server-Sent Events", "SSE Security", "Server-Sent Events Security"]
tags: [security, sse, real-time, web-security]
---

# Безопасность Server-Sent Events

## Введение

Server-Sent Events (SSE) — это веб-технология, позволяющая серверу асинхронно отправлять данные клиенту через HTTP-соединение. Несмотря на свою простоту и эффективность для передачи данных в одном направлении (сервер → клиент), SSE имеет свои уязвимости и требует специфических мер безопасности. Понимание и реализация безопасности SSE критически важны для защиты данных и предотвращения различных атак.

## Основные концепции SSE

### Что такое Server-Sent Events

Server-Sent Events — это технология веб-реального времени, которая позволяет веб-странице получать обновления от сервера автоматически. SSE использует обычное HTTP-соединение и предоставляет простой API для получения потока данных от сервера к клиенту.

```javascript
// Пример использования SSE на клиенте
const eventSource = new EventSource('/events');

eventSource.onmessage = function(event) {
  console.log('Получено сообщение:', event.data);
};

eventSource.onerror = function(event) {
  console.error('Ошибка SSE:', event);
};
```

### Преимущества SSE

1. **Простота реализации** — проще, чем WebSocket
2. **Автоматическое восстановление соединения** — встроенный механизм
3. **Использование HTTP** — работает с существующей инфраструктурой
4. **Односторонняя передача** — подходит для уведомлений и обновлений

### Ограничения и риски безопасности

1. **Односторонняя связь** — только сервер → клиент
2. **Отсутствие встроенной аутентификации** — требует дополнительной реализации
3. **Потенциальные уязвимости при неправильной реализации**
4. **Ограничения CORS** — требуют специальной настройки

## Основные угрозы безопасности SSE

### 1. Атаки подделки запросов (CSRF)

SSE-соединения могут быть инициированы с любого домена, если не настроена защита CORS:

```javascript
// Плохо: отсутствует проверка происхождения
app.get('/events', (req, res) => {
  // Любой сайт может подключиться к этому эндпоинту
  setupSSEConnection(req, res);
});

// Хорошо: проверка происхождения
app.get('/events', (req, res) => {
  const origin = req.headers.origin;
  if (!isValidOrigin(origin)) {
    res.status(403).send('Forbidden');
    return;
  }
  setupSSEConnection(req, res);
});
```

### 2. Утечка данных

Незащищенные SSE-эндпоинты могут передавать конфиденциальные данные любому подключившемуся пользователю:

```javascript
// Плохо: передача данных без аутентификации
app.get('/events', (req, res) => {
  // Передаем данные всем подключившимся
  sendUserDataToAll(res);
});

// Хорошо: проверка аутентификации и авторизации
app.get('/events', authenticateToken, (req, res) => {
  // Передаем данные только авторизованному пользователю
  sendUserDataToUser(req.user.id, res);
});
```

### 3. Атаки переполнения ресурсов

Массовое подключение к SSE-эндпоинтам может привести к исчерпанию ресурсов сервера:

```javascript
// Плохо: нет ограничений на количество соединений
app.get('/events', (req, res) => {
  // Каждый запрос создает новое соединение
  setupSSEConnection(req, res);
});

// Хорошо: ограничение количества соединений
app.get('/events', (req, res) => {
  if (!connectionLimiter.isAllowed(req.ip)) {
    res.status(429).send('Too many connections');
    return;
  }
  setupSSEConnection(req, res);
});
```

## Методы аутентификации и авторизации

### 1. Токены в заголовках

```javascript
// Клиентская сторона
const eventSource = new EventSource('/events', {
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('token')}`
  }
});

// Серверная сторона
app.get('/events', (req, res) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    res.status(401).send('Unauthorized');
    return;
  }

  const token = authHeader.substring(7);
  const user = verifyToken(token);
  
  if (!user) {
    res.status(401).send('Unauthorized');
    return;
  }

  setupSSEConnection(req, res, user);
});
```

### 2. Проверка сессии

```javascript
// Использование сессии для аутентификации
app.get('/events', (req, res) => {
  if (!req.session || !req.session.userId) {
    res.status(401).send('Unauthorized');
    return;
  }

  setupSSEConnection(req, res, { userId: req.session.userId });
});
```

### 3. Валидация JWT-токенов

```javascript
const jwt = require('jsonwebtoken');

function authenticateSSE(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    res.status(401).send('Unauthorized');
    return;
  }

  const token = authHeader.substring(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).send('Invalid token');
  }
}

app.get('/events', authenticateSSE, (req, res) => {
  setupSSEConnection(req, res, req.user);
});
```

## Безопасная реализация SSE-сервера

### Базовый безопасный SSE-сервер

```javascript
class SecureSSEServer {
  constructor(app) {
    this.app = app;
    this.connections = new Map();
    this.connectionLimiter = new ConnectionLimiter();
    this.auditLogger = new AuditLogger();
    this.setupRoutes();
  }

  setupRoutes() {
    // Основной эндпоинт для SSE с аутентификацией
    this.app.get('/api/sse/events', authenticateJWT, (req, res) => {
      this.handleSSEConnection(req, res, req.user);
    });

    // Эндпоинт для конкретного пользователя
    this.app.get('/api/sse/user/:userId', authenticateJWT, (req, res) => {
      // Проверяем, что пользователь запрашивает свои собственные события
      if (req.user.id !== req.params.userId) {
        res.status(403).send('Forbidden');
        return;
      }
      this.handleSSEConnection(req, res, req.user);
    });
  }

  handleSSEConnection(req, res, user) {
    // Проверяем лимит соединений для пользователя
    if (!this.connectionLimiter.isConnectionAllowed(user.id)) {
      res.status(429).send('Too many connections');
      return;
    }

    // Устанавливаем заголовки SSE
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': process.env.CLIENT_ORIGIN || '*',
      'Access-Control-Allow-Credentials': 'true',
      'Access-Control-Allow-Headers': 'Authorization',
      'Access-Control-Expose-Headers': 'Authorization'
    });

    const connectionId = this.generateConnectionId();
    const connection = {
      id: connectionId,
      userId: user.id,
      res,
      createdAt: Date.now(),
      lastActivity: Date.now(),
      subscriptions: new Set()
    };

    this.connections.set(connectionId, connection);
    
    // Логируем подключение
    this.auditLogger.logEvent('SSE_CONNECTED', {
      userId: user.id,
      connectionId,
      ip: req.ip
    });

    // Отправляем событие подключения
    this.sendSSEEvent(res, 'connected', {
      connectionId,
      userId: user.id,
      timestamp: new Date().toISOString()
    });

    // Обработка закрытия соединения
    req.on('close', () => {
      this.handleConnectionClose(connectionId);
    });

    // Обработка ошибок
    req.on('error', (error) => {
      console.error('Ошибка SSE-соединения:', error);
      this.handleConnectionClose(connectionId);
    });

    // Проверка активности
    this.startActivityMonitoring(connection, req);
  }

  generateConnectionId() {
    return `sse_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  sendSSEEvent(res, eventType, data) {
    try {
      if (res.writable) {
        res.write(`event: ${eventType}\n`);
        res.write(`data: ${JSON.stringify(data)}\n\n`);
      }
    } catch (error) {
      console.error('Ошибка отправки SSE-события:', error);
    }
  }

  handleConnectionClose(connectionId) {
    const connection = this.connections.get(connectionId);
    if (connection) {
      // Удаляем соединение
      this.connections.delete(connectionId);
      
      // Уменьшаем счетчик соединений для пользователя
      this.connectionLimiter.decrementConnection(connection.userId);
      
      // Логируем отключение
      this.auditLogger.logEvent('SSE_DISCONNECTED', {
        userId: connection.userId,
        connectionId,
        duration: Date.now() - connection.createdAt
      });
    }
  }

  startActivityMonitoring(connection, req) {
    const activityInterval = setInterval(() => {
      if (Date.now() - connection.lastActivity > 30 * 60 * 1000) { // 30 минут
        connection.res.end();
        clearInterval(activityInterval);
        this.handleConnectionClose(connection.id);
      } else {
        // Отправляем keep-alive сообщение
        this.sendSSEEvent(connection.res, 'keepalive', {});
        connection.lastActivity = Date.now();
      }
    }, 30000); // Проверка каждые 30 секунд

    // Очищаем интервал при закрытии соединения
    req.on('close', () => {
      clearInterval(activityInterval);
    });
  }

  // Метод для отправки событий конкретному пользователю
  sendEventToUser(userId, eventType, data) {
    for (const [connectionId, connection] of this.connections) {
      if (connection.userId === userId && connection.res.writable) {
        this.sendSSEEvent(connection.res, eventType, data);
        connection.lastActivity = Date.now();
      }
    }
  }

  // Метод для отправки событий подписчикам определенного канала
  sendEventToChannel(channel, eventType, data) {
    for (const [connectionId, connection] of this.connections) {
      if (connection.subscriptions.has(channel) && connection.res.writable) {
        this.sendSSEEvent(connection.res, eventType, data);
        connection.lastActivity = Date.now();
      }
    }
  }
}
```

### Управление подписками

```javascript
class SubscriptionManager {
  constructor() {
    this.userSubscriptions = new Map(); // userId -> Set<channels>
    this.channelSubscribers = new Map(); // channel -> Set<connectionIds>
  }

  subscribe(connectionId, userId, channel) {
    // Добавляем подписку пользователя
    if (!this.userSubscriptions.has(userId)) {
      this.userSubscriptions.set(userId, new Set());
    }
    this.userSubscriptions.get(userId).add(channel);

    // Добавляем подписчика канала
    if (!this.channelSubscribers.has(channel)) {
      this.channelSubscribers.set(channel, new Set());
    }
    this.channelSubscribers.get(channel).add(connectionId);
  }

  unsubscribe(connectionId, userId, channel) {
    // Удаляем подписку пользователя
    const userChannels = this.userSubscriptions.get(userId);
    if (userChannels) {
      userChannels.delete(channel);
      if (userChannels.size === 0) {
        this.userSubscriptions.delete(userId);
      }
    }

    // Удаляем подписчика канала
    const channelSubscribers = this.channelSubscribers.get(channel);
    if (channelSubscribers) {
      channelSubscribers.delete(connectionId);
      if (channelSubscribers.size === 0) {
        this.channelSubscribers.delete(channel);
      }
    }
  }

  getSubscribedChannels(userId) {
    const channels = this.userSubscriptions.get(userId);
    return channels ? Array.from(channels) : [];
  }

  getChannelSubscribers(channel) {
    const subscribers = this.channelSubscribers.get(channel);
    return subscribers ? Array.from(subscribers) : [];
  }
}
```

## Защита от атак

### 1. Ограничение частоты подключений

```javascript
class ConnectionLimiter {
  constructor() {
    this.connectionCounts = new Map(); // userId -> count
    this.maxConnectionsPerUser = 5;
    this.timeWindow = 60 * 1000; // 1 минута
  }

  isConnectionAllowed(userId) {
    const now = Date.now();
    let count = this.connectionCounts.get(userId) || 0;

    // Сбрасываем счетчик, если прошло достаточно времени
    if (now - (this.lastReset || 0) > this.timeWindow) {
      this.connectionCounts.clear();
      this.lastReset = now;
      count = 0;
    }

    if (count >= this.maxConnectionsPerUser) {
      return false;
    }

    this.connectionCounts.set(userId, count + 1);
    return true;
  }

  decrementConnection(userId) {
    const count = this.connectionCounts.get(userId) || 0;
    if (count > 0) {
      this.connectionCounts.set(userId, count - 1);
    }
  }
}
```

### 2. Валидация и санитизация данных

```javascript
class SSEDataValidator {
  static validateEventData(data) {
    // Проверяем размер данных
    const dataSize = JSON.stringify(data).length;
    if (dataSize > 100 * 1024) { // 100 КБ
      throw new Error('Слишком большой размер данных');
    }

    // Проверяем структуру данных
    if (typeof data !== 'object' || data === null) {
      throw new Error('Неверный формат данных');
    }

    // Санитизация потенциально опасных полей
    const sanitizedData = this.sanitizeData(data);
    
    return sanitizedData;
  }

  static sanitizeData(data) {
    const sanitized = { ...data };
    
    // Удаляем потенциально опасные поля
    if (typeof sanitized.content === 'string') {
      sanitized.content = sanitized.content
        .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
        .replace(/javascript:/gi, '')
        .replace(/on\w+\s*=/gi, '');
    }

    // Ограничиваем длину строк
    for (const [key, value] of Object.entries(sanitized)) {
      if (typeof value === 'string' && value.length > 1000) {
        sanitized[key] = value.substring(0, 1000);
      }
    }

    return sanitized;
  }

  static validateEventType(eventType) {
    // Проверяем, что тип события соответствует требованиям
    const validEventTypes = [
      'notification', 'update', 'message', 'status', 
      'error', 'info', 'warning', 'success'
    ];

    if (!validEventTypes.includes(eventType)) {
      throw new Error(`Недопустимый тип события: ${eventType}`);
    }

    return true;
  }
}
```

### 3. Защита от XSS в SSE-данных

```javascript
class SSEXSSProtection {
  static sanitizeForSSE(data) {
    // Преобразуем данные в безопасный формат для SSE
    const jsonString = JSON.stringify(data);
    
    // Экранируем потенциально опасные символы
    const sanitizedString = jsonString
      .replace(/[\u0000-\u0008\u000B-\u000C\u000E-\u001F\u007F-\u009F]/g, '') // Удаляем управляющие символы
      .replace(/\n/g, '\\n') // Заменяем переводы строк
      .replace(/\r/g, '\\r') // Заменяем возвраты каретки
      .replace(/\0/g, '\\0'); // Заменяем нулевые байты

    try {
      return JSON.parse(sanitizedString);
    } catch (error) {
      throw new Error('Невозможно санитизировать данные для SSE');
    }
  }

  static escapeSSEData(data) {
    // Экранирование данных для предотвращения XSS
    if (typeof data === 'string') {
      return data
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;');
    }
    
    if (typeof data === 'object') {
      const result = {};
      for (const [key, value] of Object.entries(data)) {
        result[key] = this.escapeSSEData(value);
      }
      return result;
    }
    
    return data;
  }
}
```

## CORS и безопасность происхождения

### Настройка CORS для SSE

```javascript
// Безопасная настройка CORS для SSE
const cors = require('cors');

// Конфигурация CORS для SSE-эндпоинтов
const sseCorsOptions = {
  origin: function (origin, callback) {
    // Проверяем, что происхождение в белом списке
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [
      'https://example.com',
      'https://app.example.com'
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Не разрешенное происхождение'));
    }
  },
  credentials: true,
  optionsSuccessStatus: 200
};

// Применение CORS к SSE-эндпоинтам
app.use('/api/sse', cors(sseCorsOptions));

// Альтернативный подход - ручная проверка
app.get('/api/sse/events', (req, res) => {
  const origin = req.headers.origin;
  const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
  
  if (origin && !allowedOrigins.includes(origin)) {
    res.status(403).send('Forbidden');
    return;
  }
  
  // Устанавливаем заголовки CORS
  res.setHeader('Access-Control-Allow-Origin', origin);
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Allow-Headers', 'Authorization');
  
  setupSSEConnection(req, res);
});
```

## Шифрование данных в SSE

### Шифрование чувствительных данных

```javascript
class SSEEncryption {
  constructor() {
    this.encryptionKeys = new Map();
  }

  async encryptData(data, userId) {
    const key = await this.getEncryptionKey(userId);
    
    if (!key) {
      // Если ключ не найден, возвращаем данные как есть (для публичных событий)
      return data;
    }

    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encodedData = new TextEncoder().encode(JSON.stringify(data));
    
    const encryptedData = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      key,
      encodedData
    );

    return {
      encrypted: Array.from(new Uint8Array(encryptedData)),
      iv: Array.from(iv),
      encrypted: true
    };
  }

  async decryptData(encryptedData, userId) {
    if (!encryptedData.encrypted) {
      return encryptedData;
    }

    const key = await this.getEncryptionKey(userId);
    const iv = new Uint8Array(encryptedData.iv);
    const data = new Uint8Array(encryptedData.encrypted);
    
    const decryptedData = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv },
      key,
      data
    );

    const decoder = new TextDecoder();
    return JSON.parse(decoder.decode(decryptedData));
  }

  async getEncryptionKey(userId) {
    let key = this.encryptionKeys.get(userId);
    
    if (!key) {
      // Генерируем ключ для пользователя
      key = await crypto.subtle.generateKey(
        { name: 'AES-GCM', length: 256 },
        true,
        ['encrypt', 'decrypt']
      );
      this.encryptionKeys.set(userId, key);
    }

    return key;
  }
}
```

## Мониторинг и аудит

### Система аудита SSE-соединений

```javascript
class SSEAuditLogger {
  constructor() {
    this.events = [];
    this.maxEvents = 10000;
  }

  logEvent(type, details) {
    const event = {
      id: crypto.randomUUID(),
      type,
      details,
      timestamp: new Date().toISOString()
    };

    this.events.push(event);

    // Ограничиваем размер лога
    if (this.events.length > this.maxEvents) {
      this.events = this.events.slice(-this.maxEvents);
    }

    // Логируем в консоль (в реальном приложении - в систему логирования)
    console.log(`SSE Audit: ${type}`, details);

    // Проверяем на подозрительные события
    this.checkForSuspiciousActivity(event);
  }

  checkForSuspiciousActivity(event) {
    switch (event.type) {
      case 'SSE_CONNECTED':
        // Проверяем частые подключения с одного IP
        if (this.isFrequentConnection(event.details.ip)) {
          this.logSecurityAlert('FREQUENT_SSE_CONNECTIONS', event.details);
        }
        break;
        
      case 'SSE_ERROR':
        // Проверяем ошибки аутентификации
        if (event.details.error?.includes('Unauthorized')) {
          this.logSecurityAlert('SSE_AUTHENTICATION_FAILED', event.details);
        }
        break;
    }
  }

  isFrequentConnection(ip) {
    const recentEvents = this.events.filter(
      event => event.details.ip === ip && 
               event.type === 'SSE_CONNECTED' && 
               Date.now() - new Date(event.timestamp).getTime() < 60000 // За последнюю минуту
    );
    
    return recentEvents.length > 10; // Более 10 подключений за минуту
  }

  logSecurityAlert(type, details) {
    console.warn(`SSE Security Alert: ${type}`, details);
    
    // Здесь можно отправить уведомление администратору
    // или зарегистрировать событие в системе безопасности
  }

  getRecentEvents(userId = null, limit = 100) {
    let events = this.events;

    if (userId) {
      events = events.filter(event => event.details.userId === userId);
    }

    return events.slice(-limit).reverse();
  }
}
```

## Практические примеры безопасной реализации

### Пример полной безопасной SSE-системы

```javascript
// Комплексный пример безопасного SSE-сервера
class CompleteSecureSSEServer {
  constructor(app) {
    this.app = app;
    this.connections = new Map();
    this.connectionLimiter = new ConnectionLimiter();
    this.validator = new SSEDataValidator();
    this.xssProtection = new SSEXSSProtection();
    this.auditLogger = new SSEAuditLogger();
    this.encryption = new SSEEncryption();
    this.subscriptionManager = new SubscriptionManager();
    
    this.setupRoutes();
    this.setupCleanup();
  }

  setupRoutes() {
    this.app.get('/api/sse/notifications', authenticateJWT, (req, res) => {
      this.handleSecureSSEConnection(req, res);
    });
  }

  async handleSecureSSEConnection(req, res) {
    const user = req.user;
    
    // Проверяем лимит соединений
    if (!this.connectionLimiter.isConnectionAllowed(user.id)) {
      this.auditLogger.logEvent('SSE_CONNECTION_LIMIT_EXCEEDED', {
        userId: user.id,
        ip: req.ip
      });
      res.status(429).send('Too many connections');
      return;
    }

    // Проверяем права доступа к SSE
    if (!this.hasSSEPermission(user)) {
      res.status(403).send('Forbidden');
      return;
    }

    // Устанавливаем безопасные заголовки
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': process.env.CLIENT_ORIGIN || 'https://example.com',
      'Access-Control-Allow-Credentials': 'true'
    });

    const connectionId = this.generateConnectionId();
    const connection = {
      id: connectionId,
      userId: user.id,
      res,
      createdAt: Date.now(),
      lastActivity: Date.now(),
      permissions: user.permissions
    };

    this.connections.set(connectionId, connection);
    this.connectionLimiter.incrementConnection(user.id);

    // Логируем подключение
    this.auditLogger.logEvent('SSE_CONNECTED', {
      userId: user.id,
      connectionId,
      ip: req.ip
    });

    // Отправляем подтверждение подключения
    this.sendSSEEvent(res, 'connected', {
      connectionId,
      timestamp: new Date().toISOString()
    });

    // Обработка закрытия
    req.on('close', () => {
      this.handleConnectionClose(connectionId);
    });

    req.on('error', (error) => {
      this.auditLogger.logEvent('SSE_ERROR', {
        userId: user.id,
        connectionId,
        error: error.message
      });
      this.handleConnectionClose(connectionId);
    });

    // Мониторинг активности
    this.startActivityMonitoring(connection, req);
  }

  async sendSecureEvent(userId, eventType, data, encrypt = false) {
    // Валидируем данные
    data = this.validator.validateEventData(data);
    
    // Санитизируем данные
    data = this.xssProtection.escapeSSEData(data);
    
    // При необходимости шифруем
    if (encrypt) {
      data = await this.encryption.encryptData(data, userId);
    }

    // Отправляем событие пользователю
    for (const [connectionId, connection] of this.connections) {
      if (connection.userId === userId && connection.res.writable) {
        this.sendSSEEvent(connection.res, eventType, data);
        connection.lastActivity = Date.now();
      }
    }
  }

  sendSSEEvent(res, eventType, data) {
    try {
      if (res.writable) {
        res.write(`event: ${eventType}\n`);
        res.write(`data: ${JSON.stringify(data)}\n\n`);
      }
    } catch (error) {
      console.error('Ошибка отправки SSE-события:', error);
    }
  }

  handleConnectionClose(connectionId) {
    const connection = this.connections.get(connectionId);
    if (connection) {
      this.connections.delete(connectionId);
      this.connectionLimiter.decrementConnection(connection.userId);
      
      this.auditLogger.logEvent('SSE_DISCONNECTED', {
        userId: connection.userId,
        connectionId,
        duration: Date.now() - connection.createdAt
      });
    }
  }

  hasSSEPermission(user) {
    // Проверяем, есть ли у пользователя разрешение на SSE
    return user.permissions.includes('sse_access');
  }

  setupCleanup() {
    // Регулярная очистка устаревших соединений
    setInterval(() => {
      const now = Date.now();
      for (const [connectionId, connection] of this.connections) {
        if (now - connection.lastActivity > 30 * 60 * 1000) { // 30 минут
          connection.res.end();
          this.handleConnectionClose(connectionId);
        }
      }
    }, 60000); // Каждую минуту
  }

  generateConnectionId() {
    return `secure_sse_${Date.now()}_${crypto.randomUUID().substr(0, 8)}`;
  }

  startActivityMonitoring(connection, req) {
    const activityInterval = setInterval(() => {
      if (Date.now() - connection.lastActivity > 25 * 60 * 1000) { // 25 минут
        this.sendSSEEvent(connection.res, 'keepalive', {});
        connection.lastActivity = Date.now();
      }
    }, 30000); // Каждые 30 секунд

    req.on('close', () => {
      clearInterval(activityInterval);
    });
  }
}
```

## Практические рекомендации

### Лучшие практики безопасности SSE

1. **Всегда требуйте аутентификацию** — не разрешайте анонимные SSE-подключения
2. **Ограничивайте количество соединений** — предотвращайте атаки переполнения
3. **Валидируйте все данные** — проверяйте и санитизируйте отправляемые данные
4. **Используйте HTTPS** — шифруйте все SSE-соединения
5. **Реализуйте мониторинг** — отслеживайте подозрительную активность
6. **Ограничивайте права доступа** — используйте систему разрешений

### Обработка ошибок и исключений

```javascript
// Централизованная обработка ошибок SSE
class SSEErrorHandler {
  static handleError(error, context) {
    const errorEvent = {
      type: 'SSE_ERROR',
      error: error.message,
      context,
      timestamp: new Date().toISOString()
    };

    console.error('Ошибка SSE:', errorEvent);

    // Логируем в систему аудита
    auditLogger.logEvent('SSE_ERROR', errorEvent);

    // В зависимости от типа ошибки принимаем меры
    if (error.message.includes('Authentication')) {
      this.handleAuthenticationError(context);
    } else if (error.message.includes('Rate limit')) {
      this.handleRateLimitError(context);
    } else if (error.message.includes('XSS')) {
      this.handleXSSError(context);
    }
  }

  static handleAuthenticationError(context) {
    // Блокируем IP при частых ошибках аутентификации
    rateLimiter.blockIP(context.ip);
  }

  static handleRateLimitError(context) {
    // Увеличиваем уровень подозрения
    console.warn(`Подозрительная активность от ${context.ip}`);
  }

  static handleXSSError(context) {
    // Логируем потенциальную XSS-атаку
    securityLogger.logAttack('XSS_ATTEMPT', context);
  }
}
```

## Связь с другими аспектами безопасности

Безопасность Server-Sent Events тесно связана с:
- [[Безопасность-данных-в-реальном-времени]] — общие принципы безопасности реального времени
- [[Реальное-время-шифрование]] — шифрование данных в реальном времени
- [[Предотвращение-злоупотреблений]] — защита от злоупотреблений
- [[Ограничение-скорости]] — контроль частоты подключений
- [[HTTP-Security-Headers]] — заголовки безопасности HTTP

## Заключение

Безопасность Server-Sent Events требует комплексного подхода, включающего аутентификацию, авторизацию, валидацию данных, защиту от XSS и мониторинг активности. При правильной реализации SSE может быть безопасным и эффективным способом передачи данных в реальном времени. Ключевыми элементами безопасной реализации являются: строгая аутентификация, ограничение ресурсов, валидация данных и надежный аудит всех операций.

## Дополнительные ресурсы

- MDN Web Docs: Server-Sent Events
- OWASP: Testing for Server-Sent Events
- HTML5 SSE Security Guidelines
- Real-Time Web Security Best Practices