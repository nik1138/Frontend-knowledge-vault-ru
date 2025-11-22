---
aliases: ["Безопасность WebSocket", "WebSocket Security", "WebSocket Protection"]
tags: [security, websocket, communication]
created: 2025-11-18
updated: 2025-11-18
---

# Безопасность WebSocket

WebSocket - это протокол двунаправленной связи между клиентом и сервером, который предоставляет возможность обмена данными в реальном времени. Безопасность WebSocket включает в себя защиту как от несанкционированного доступа, так и от различных атак, специфичных для этого протокола.

## Введение

WebSocket предоставляет постоянное соединение между клиентом и сервером, что отличает его от традиционного HTTP, где каждое соединение создается заново для каждого запроса. Эта особенность требует специального подхода к обеспечению безопасности соединения и передаваемых данных.

## Принципы работы WebSocket

### Процесс установления соединения

WebSocket соединение начинается с HTTP-запроса с "апгрейдом":

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

Ответ сервера:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
```

### Отличия от HTTP

- Постоянное соединение
- Двунаправленная передача данных
- Отсутствие автоматической аутентификации
- Неограниченное количество сообщений после установления соединения

## Угрозы безопасности WebSocket

### Подделка запросов

WebSocket соединение может быть инициировано с любого сайта:

```javascript
// Злонамеренный сайт может создать WebSocket соединение к вашему API
const ws = new WebSocket('wss://yourapi.com/chat');
ws.onopen = function() {
  // Злонамеренный код может отправлять сообщения от имени пользователя
  ws.send('{"action": "delete_account", "user_id": "victim_id"}');
};
```

### Утечка данных

После установления соединения данные передаются без дополнительной проверки:

- Доступ к чувствительной информации
- Перехват сообщений
- Нарушение конфиденциальности

### Атаки типа "отравление" соединений

Злоумышленник может поддерживать большое количество соединений:

- Перегрузка сервера
- Отказ в обслуживании
- Использование ресурсов

### Небезопасная передача данных

Незашифрованные сообщения могут быть перехвачены:

- Прослушивание трафика
- Изменение данных
- [[Анализ-логов]] сетевого трафика

## Меры безопасности

### Использование WSS (WebSocket Secure)

Всегда используйте зашифрованное соединение:

```javascript
// НЕБЕЗОПАСНО
const ws = new WebSocket('ws://example.com/socket');

// БЕЗОПАСНО
const ws = new WebSocket('wss://example.com/socket');
```

### Проверка Origin

Проверка заголовка Origin в handshake запросе:

```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ noServer: true });

// Проверка Origin при установлении соединения
wss.on('connection', (ws, req) => {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://trusted-site.com', 'https://another-trusted.com'];
  
  if (!allowedOrigins.includes(origin)) {
    ws.close(1008, 'Origin not allowed');
    return;
  }
  
  // Продолжение обработки соединения
  ws.on('message', (message) => {
    // Обработка сообщений
  });
});
```

### Аутентификация и авторизация

Проверка подлинности пользователя:

```javascript
// Пример аутентификации через токен в URL параметрах
const wss = new WebSocket.Server({ noServer: true });

wss.on('upgrade', (req, socket, head) => {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const token = url.searchParams.get('token');
  
  // Проверка токена
  if (!isValidToken(token)) {
    socket.destroy();
    return;
  }
  
  // Продолжение установления соединения
  wss.handleUpgrade(req, socket, head, (ws) => {
    wss.emit('connection', ws, req);
  });
});
```

### Ограничение соединений

Контроль количества соединений:

```javascript
const connections = new Map();
const MAX_CONNECTIONS_PER_IP = 10;

wss.on('connection', (ws, req) => {
  const clientIP = req.socket.remoteAddress;
  
  if (!connections.has(clientIP)) {
    connections.set(clientIP, 0);
  }
  
  if (connections.get(clientIP) >= MAX_CONNECTIONS_PER_IP) {
    ws.close(1008, 'Too many connections');
    return;
  }
  
  connections.set(clientIP, connections.get(clientIP) + 1);
  
  ws.on('close', () => {
    connections.set(clientIP, connections.get(clientIP) - 1);
    if (connections.get(clientIP) === 0) {
      connections.delete(clientIP);
    }
  });
});
```

## Архитектура безопасного WebSocket

### Слои безопасности

```
[Клиент] <--(WSS)--> [Прокси/Балансировщик] <--(WSS)--> [Сервер приложений]
    |                       |                            |
    |                   [Проверка Origin]            [Аутентификация]
    |                   [Ограничение]                [Авторизация]
    |                   [Фильтрация]                 [Валидация]
    |                   [Мониторинг]                 [Логирование]
```

### Проверка сообщений

Валидация всех входящих сообщений:

```javascript
class SecureWebSocket {
  constructor(ws) {
    this.ws = ws;
    this.setupMessageHandler();
  }
  
  setupMessageHandler() {
    this.ws.on('message', (data) => {
      try {
        // Парсинг JSON
        const message = JSON.parse(data);
        
        // Проверка структуры сообщения
        if (!this.isValidMessageStructure(message)) {
          this.ws.close(1008, 'Invalid message structure');
          return;
        }
        
        // Проверка типа сообщения
        if (!this.isValidMessageType(message.type)) {
          this.ws.close(1008, 'Invalid message type');
          return;
        }
        
        // Проверка размера сообщения
        if (data.length > MAX_MESSAGE_SIZE) {
          this.ws.close(1009, 'Message too large');
          return;
        }
        
        // Обработка сообщения
        this.processMessage(message);
        
      } catch (error) {
        this.ws.close(1003, 'Invalid message format');
      }
    });
  }
  
  isValidMessageStructure(message) {
    return message && 
           typeof message === 'object' && 
           message.hasOwnProperty('type') && 
           message.hasOwnProperty('data');
  }
  
  isValidMessageType(type) {
    const allowedTypes = ['chat', 'notification', 'command'];
    return allowedTypes.includes(type);
  }
  
  processMessage(message) {
    // Обработка валидного сообщения
    console.log(`Processing message: ${message.type}`);
  }
}
```

## Реализация безопасности

### Пример безопасного WebSocket сервера

```javascript
const WebSocket = require('ws');
const jwt = require('jsonwebtoken');
const rateLimit = require('ws-rate-limit');

// Настройка сервера с ограничением частоты
const wss = new WebSocket.Server({ 
  port: 8080,
  verifyClient: (info) => {
    // Проверка Origin
    const allowedOrigins = [
      'https://trusted-site.com',
      'https://localhost:3000'
    ];
    
    return allowedOrigins.includes(info.origin);
  }
});

// Ограничение частоты сообщений
rateLimit(wss, {
  timeLimit: 1000, // 1 секунда
  maxMsgs: 10,     // максимум 10 сообщений за секунду
  errorMsg: 'Rate limit exceeded',
  throttleMsg: 'Rate limit exceeded, throttling'
});

// Маппинг соединений с пользователями
const connections = new Map();

wss.on('connection', (ws, req) => {
  // Извлечение токена из URL параметров
  const url = new URL(`http://${req.headers.host}${req.url}`);
  const token = url.searchParams.get('token');
  
  try {
    // Проверка JWT токена
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    ws.userId = decoded.userId;
    
    // Сохранение соединения
    if (!connections.has(ws.userId)) {
      connections.set(ws.userId, new Set());
    }
    connections.get(ws.userId).add(ws);
    
    console.log(`User ${ws.userId} connected`);
    
    // Обработка сообщений
    ws.on('message', (message) => {
      try {
        const parsedMessage = JSON.parse(message);
        
        // Дополнительная проверка сообщения
        if (!isValidUserMessage(parsedMessage, ws.userId)) {
          ws.close(1008, 'Invalid message');
          return;
        }
        
        // Обработка сообщения
        handleMessage(parsedMessage, ws.userId);
      } catch (error) {
        console.error('Invalid message format:', error);
        ws.close(1003, 'Invalid message format');
      }
    });
    
    // Обработка закрытия соединения
    ws.on('close', () => {
      if (connections.has(ws.userId)) {
        connections.get(ws.userId).delete(ws);
        if (connections.get(ws.userId).size === 0) {
          connections.delete(ws.userId);
        }
      }
      console.log(`User ${ws.userId} disconnected`);
    });
    
  } catch (error) {
    console.error('Authentication failed:', error);
    ws.close(1008, 'Authentication failed');
  }
});

function isValidUserMessage(message, userId) {
  // Проверка структуры и содержания сообщения
  if (!message.type || !message.data) {
    return false;
  }
  
  // Проверка типа сообщения
  const allowedTypes = ['chat', 'status', 'command'];
  if (!allowedTypes.includes(message.type)) {
    return false;
  }
  
  // Проверка данных сообщения
  if (typeof message.data !== 'object') {
    return false;
  }
  
  // Проверка на наличие запрещенных полей
  const forbiddenFields = ['admin', 'system', 'root'];
  return !Object.keys(message.data).some(key => forbiddenFields.includes(key));
}

function handleMessage(message, userId) {
  // Обработка валидного сообщения
  switch (message.type) {
    case 'chat':
      // Отправка сообщения другим пользователям
      broadcastMessage(message, userId);
      break;
    case 'status':
      // Обновление статуса пользователя
      updateUserStatus(userId, message.data.status);
      break;
    default:
      console.log(`Unknown message type: ${message.type}`);
  }
}

function broadcastMessage(message, senderId) {
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN && client.userId !== senderId) {
      client.send(JSON.stringify(message));
    }
  });
}
```

### Клиентская безопасность

```javascript
class SecureWebSocketClient {
  constructor(url, token) {
    this.url = url;
    this.token = token;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectInterval = 1000;
  }
  
  connect() {
    // Добавление токена к URL
    const fullUrl = `${this.url}?token=${encodeURIComponent(this.token)}`;
    this.ws = new WebSocket(fullUrl);
    
    this.ws.onopen = () => {
      this.reconnectAttempts = 0;
      console.log('WebSocket connected');
    };
    
    this.ws.onmessage = (event) => {
      try {
        const message = JSON.parse(event.data);
        
        // Проверка сообщения от сервера
        if (this.isValidServerMessage(message)) {
          this.handleServerMessage(message);
        } else {
          console.error('Invalid server message received');
        }
      } catch (error) {
        console.error('Error parsing server message:', error);
      }
    };
    
    this.ws.onclose = (event) => {
      console.log(`WebSocket closed: ${event.code} - ${event.reason}`);
      
      // Попытка переподключения
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        setTimeout(() => {
          this.reconnectAttempts++;
          this.connect();
        }, this.reconnectInterval * this.reconnectAttempts);
      }
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }
  
  isValidServerMessage(message) {
    // Проверка структуры сообщения от сервера
    return message && typeof message === 'object' && 
           message.hasOwnProperty('type') && 
           message.hasOwnProperty('data');
  }
  
  handleServerMessage(message) {
    // Обработка сообщения от сервера
    switch (message.type) {
      case 'notification':
        this.showNotification(message.data);
        break;
      case 'update':
        this.handleUpdate(message.data);
        break;
      default:
        console.log(`Unknown server message type: ${message.type}`);
    }
  }
  
  send(message) {
    if (this.ws.readyState === WebSocket.OPEN) {
      // Проверка сообщения перед отправкой
      if (this.isValidClientMessage(message)) {
        this.ws.send(JSON.stringify(message));
      } else {
        console.error('Invalid client message');
      }
    } else {
      console.error('WebSocket is not open');
    }
  }
  
  isValidClientMessage(message) {
    // Проверка сообщения перед отправкой
    if (!message.type || !message.data) {
      return false;
    }
    
    // Проверка размера сообщения
    const serialized = JSON.stringify(message);
    return serialized.length <= 1024; // 1KB limit
  }
  
  showNotification(data) {
    // Отображение уведомления
    console.log('Notification:', data);
  }
  
  handleUpdate(data) {
    // Обработка обновления
    console.log('Update:', data);
  }
}
```

## Мониторинг и логирование

### Журналирование WebSocket активности

```javascript
const winston = require('winston');

const websocketLogger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'websocket-error.log', level: 'error' }),
    new winston.transports.File({ filename: 'websocket-combined.log' })
  ]
});

// Интеграция с WebSocket сервером
wss.on('connection', (ws, req) => {
  const clientId = generateClientId();
  
  websocketLogger.info('WebSocket connection established', {
    clientId: clientId,
    ip: req.socket.remoteAddress,
    userAgent: req.headers['user-agent'],
    origin: req.headers.origin
  });
  
  ws.on('message', (message) => {
    try {
      const parsed = JSON.parse(message);
      websocketLogger.info('Message received', {
        clientId: clientId,
        messageType: parsed.type,
        messageSize: message.length
      });
    } catch (error) {
      websocketLogger.error('Invalid message received', {
        clientId: clientId,
        error: error.message,
        rawMessage: message.toString()
      });
    }
  });
  
  ws.on('close', (code, reason) => {
    websocketLogger.info('WebSocket connection closed', {
      clientId: clientId,
      closeCode: code,
      closeReason: reason
    });
  });
});
```

### Интеграция с [[Сигнализация-безопасности]]

```javascript
// Функция для отправки сигналов безопасности
function sendSecurityAlert(alertType, details) {
  // Отправка сигнала в систему безопасности
  const alert = {
    timestamp: new Date().toISOString(),
    type: alertType,
    details: details,
    severity: 'high'
  };
  
  // Отправка в систему мониторинга
  sendToSecuritySystem(alert);
}

// Мониторинг аномальной активности
const messageCounters = new Map();

wss.on('connection', (ws, req) => {
  const clientId = req.socket.remoteAddress;
  messageCounters.set(clientId, { count: 0, lastReset: Date.now() });
  
  ws.on('message', (message) => {
    const counter = messageCounters.get(clientId);
    counter.count++;
    
    // Проверка на чрезмерную активность
    if (counter.count > 100 && (Date.now() - counter.lastReset) < 60000) { // 100 сообщений за минуту
      sendSecurityAlert('rate_limit_exceeded', {
        clientId: clientId,
        messageCount: counter.count,
        timeWindow: '60 seconds'
      });
    }
    
    // Сброс счетчика каждую минуту
    if (Date.now() - counter.lastReset > 60000) {
      counter.count = 1;
      counter.lastReset = Date.now();
    }
  });
});
```

## Лучшие практики

### Архитектурные рекомендации

- Использование WSS вместо WS
- Проверка Origin на сервере
- Аутентификация пользователей
- Ограничение количества соединений
- Валидация всех сообщений
- [[Управление-доступом]] к каналам

### Безопасность данных

- [[Шифрование-WebSocket]] для чувствительных данных
- Проверка целостности сообщений
- Ограничение размера сообщений
- [[Проверка-сообщений]] на наличие вредоносного содержимого
- [[Клиентское-шифрование]] для конфиденциальных данных

### Мониторинг и защита

- [[Инструменты-мониторинга-безопасности]]
- [[Анализ-логов]] WebSocket соединений
- [[Сигнализация-безопасности]] для аномальной активности
- Регулярные проверки безопасности
- [[Предотвращение-инъекций]] в WebSocket сообщениях

> [!tip] Совет
> Всегда используйте WSS (WebSocket Secure) для передачи конфиденциальных данных и в производственных средах.

> [!warning] Важно
> Проверяйте все сообщения, полученные через WebSocket, так как соединение остается открытым и может быть использовано для атак после установления.

## Заключение

Безопасность WebSocket требует комплексного подхода, включающего как защиту на уровне протокола, так и архитектурные меры безопасности. Правильная реализация обеспечивает безопасное двунаправленное соединение с возможностью реального времени обмена данными.