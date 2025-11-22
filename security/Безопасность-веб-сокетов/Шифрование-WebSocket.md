---
aliases: ["Шифрование WebSocket", "WebSocket Encryption", "WSS Security"]
tags: [security, websocket, encryption, wss]
created: 2025-11-18
updated: 2025-11-18
---

# Шифрование WebSocket

Шифрование WebSocket обеспечивает конфиденциальность и целостность данных, передаваемых через WebSocket соединения. Это критически важно для защиты чувствительной информации от перехвата и модификации в процессе передачи.

## Введение

WebSocket соединения могут использовать как незашифрованный протокол (ws://), так и зашифрованный (wss://). Использование шифрования через WSS (WebSocket Secure) является обязательным требованием для любых приложений, обрабатывающих конфиденциальные данные.

## Протоколы и стандарты

### WSS (WebSocket Secure)

WSS использует TLS/SSL для шифрования соединения:

- Основан на TLS 1.2 или выше
- Использует тот же порт, что и HTTPS (443 по умолчанию)
- Требует действительный SSL-сертификат
- Обеспечивает конфиденциальность и целостность данных

### TLS версии

Рекомендуемые версии TLS:

- TLS 1.3 (предпочтительно)
- TLS 1.2 (широко поддерживается)
- Избегать TLS 1.0 и 1.1 из-за уязвимостей

## Архитектура шифрования

### Установление зашифрованного соединения

```
[Клиент] --(HTTPS Handshake)--> [Сервер]
[Сервер] --(TLS Handshake)--> [Клиент]
[Клиент] --(WebSocket Upgrade)--> [Сервер]
[Сервер] --(Соединение установлено)--> [Клиент]
```

### Уровни шифрования

```
[Приложение] --> [WebSocket сообщения]
[WebSocket] --> [TLS шифрование]
[TCP] --> [Сеть]
```

## Реализация WSS

### Серверная реализация

```javascript
const https = require('https');
const WebSocket = require('ws');
const fs = require('fs');

// Создание HTTPS сервера с SSL сертификатами
const server = https.createServer({
  cert: fs.readFileSync('/path/to/certificate.pem'),
  key: fs.readFileSync('/path/to/private-key.pem'),
  // Дополнительные настройки безопасности TLS
  secureOptions: crypto.constants.SSL_OP_NO_SSLv2 | 
                 crypto.constants.SSL_OP_NO_SSLv3 |
                 crypto.constants.SSL_OP_NO_TLSv1 |
                 crypto.constants.SSL_OP_NO_TLSv1_1
});

// Создание WebSocket сервера поверх HTTPS
const wss = new WebSocket.Server({ server });

wss.on('connection', (ws, req) => {
  console.log('Зашифрованное WebSocket соединение установлено');
  
  ws.on('message', (message) => {
    console.log('Получено зашифрованное сообщение:', message);
    // Обработка сообщения
  });
  
  ws.on('close', () => {
    console.log('Соединение закрыто');
  });
});

server.listen(443, () => {
  console.log('WSS сервер запущен на порту 443');
});
```

### Клиентская реализация

```javascript
// Использование зашифрованного соединения
const ws = new WebSocket('wss://secure-websocket.example.com/chat');

ws.onopen = function(event) {
  console.log('Зашифрованное соединение установлено');
  
  // Отправка зашифрованного сообщения
  ws.send(JSON.stringify({
    type: 'auth',
    data: { token: 'secure-token' }
  }));
};

ws.onmessage = function(event) {
  // Сообщение получено в зашифрованном виде
  const message = JSON.parse(event.data);
  console.log('Получено зашифрованное сообщение:', message);
};

ws.onerror = function(error) {
  console.error('Ошибка WSS:', error);
};
```

## Дополнительное шифрование на уровне приложения

### Клиентское шифрование сообщений

```javascript
const crypto = require('crypto');

class SecureWebSocketClient {
  constructor(url, encryptionKey) {
    this.url = url;
    this.encryptionKey = encryptionKey;
    this.iv = crypto.randomBytes(16); // Вектор инициализации
  }
  
  async connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('Соединение установлено');
    };
    
    this.ws.onmessage = async (event) => {
      // Расшифровка полученного сообщения
      const decryptedMessage = await this.decryptMessage(event.data);
      console.log('Расшифрованное сообщение:', decryptedMessage);
    };
  }
  
  async encryptMessage(message) {
    const cipher = crypto.createCipher('aes-256-cbc', this.encryptionKey);
    let encrypted = cipher.update(JSON.stringify(message), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    // Возвращаем зашифрованное сообщение вместе с IV
    return JSON.stringify({
      encrypted: encrypted,
      iv: this.iv.toString('hex')
    });
  }
  
  async decryptMessage(encryptedData) {
    const data = JSON.parse(encryptedData);
    const decipher = crypto.createDecipher('aes-256-cbc', this.encryptionKey);
    let decrypted = decipher.update(data.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return JSON.parse(decrypted);
  }
  
  async send(message) {
    if (this.ws.readyState === WebSocket.OPEN) {
      const encryptedMessage = await this.encryptMessage(message);
      this.ws.send(encryptedMessage);
    }
  }
}
```

### Серверная реализация дополнительного шифрования

```javascript
const WebSocket = require('ws');
const crypto = require('crypto');

class SecureWebSocketServer {
  constructor(server) {
    this.wss = new WebSocket.Server({ server });
    this.encryptionKeys = new Map(); // userId -> key
    this.setupConnectionHandlers();
  }
  
  setupConnectionHandlers() {
    this.wss.on('connection', (ws, req) => {
      // Аутентификация пользователя и получение ключа шифрования
      const userId = this.authenticateUser(req);
      if (!userId) {
        ws.close(1008, 'Authentication required');
        return;
      }
      
      // Получение ключа шифрования для пользователя
      const encryptionKey = this.getEncryptionKeyForUser(userId);
      ws.encryptionKey = encryptionKey;
      
      ws.on('message', async (message) => {
        try {
          // Попытка расшифровки сообщения
          const decryptedMessage = await this.decryptMessage(message, ws.encryptionKey);
          console.log(`Расшифрованное сообщение от ${userId}:`, decryptedMessage);
          
          // Обработка расшифрованного сообщения
          await this.processMessage(ws, decryptedMessage, userId);
        } catch (error) {
          console.error('Ошибка расшифровки:', error);
          ws.close(1003, 'Invalid encrypted message');
        }
      });
    });
  }
  
  async decryptMessage(encryptedData, key) {
    try {
      const data = JSON.parse(encryptedData);
      const decipher = crypto.createDecipher('aes-256-cbc', key);
      let decrypted = decipher.update(data.encrypted, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      return JSON.parse(decrypted);
    } catch (error) {
      throw new Error('Failed to decrypt message');
    }
  }
  
  async processMessage(ws, message, userId) {
    // Обработка расшифрованного сообщения
    switch (message.type) {
      case 'chat':
        await this.broadcastChatMessage(message, userId, ws.encryptionKey);
        break;
      default:
        console.log(`Неизвестный тип сообщения: ${message.type}`);
    }
  }
  
  async broadcastChatMessage(message, senderId, encryptionKey) {
    // Шифрование сообщения для отправки другим пользователям
    const encryptedMessage = await this.encryptMessage(message, encryptionKey);
    
    this.wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN && client !== ws) {
        client.send(encryptedMessage);
      }
    });
  }
  
  async encryptMessage(message, key) {
    const cipher = crypto.createCipher('aes-256-cbc', key);
    let encrypted = cipher.update(JSON.stringify(message), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return JSON.stringify({
      encrypted: encrypted,
      iv: crypto.randomBytes(16).toString('hex')
    });
  }
  
  authenticateUser(req) {
    // Реализация аутентификации пользователя
    // Возвращает userId или null
    return 'user123'; // Заглушка
  }
  
  getEncryptionKeyForUser(userId) {
    // Получение или генерация ключа шифрования для пользователя
    if (!this.encryptionKeys.has(userId)) {
      const key = crypto.randomBytes(32); // 256-bit key
      this.encryptionKeys.set(userId, key);
    }
    
    return this.encryptionKeys.get(userId);
  }
}
```

## Настройка SSL/TLS

### Генерация самоподписного сертификата

```bash
# Генерация приватного ключа
openssl genrsa -out server-key.pem 2048

# Генерация CSR (Certificate Signing Request)
openssl req -new -key server-key.pem -out server-csr.pem

# Генерация самоподписанного сертификата
openssl x509 -req -in server-csr.pem -signkey server-key.pem -out server-cert.pem -days 365
```

### Настройка продвинутого TLS

```javascript
const tls = require('tls');
const fs = require('fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  
  // Настройки безопасности TLS
  secureProtocol: 'TLSv1_2_method', // Использовать только TLS 1.2+
  
  // Запрет слабых шифров
  ciphers: [
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES256-SHA384',
    'ECDHE-RSA-AES128-SHA256'
  ].join(':'),
  
  honorCipherOrder: true,
  
  // Настройки для защиты от атак
  rejectUnauthorized: false, // В продакшене должно быть true
  requestCert: true,        // Запрашивать клиентский сертификат
  secureOptions: crypto.constants.SSL_OP_NO_SSLv2 |
                 crypto.constants.SSL_OP_NO_SSLv3 |
                 crypto.constants.SSL_OP_NO_TLSv1 |
                 crypto.constants.SSL_OP_NO_TLSv1_1
};
```

## Практические примеры

### Полностью защищенный WebSocket сервер

```javascript
const https = require('https');
const WebSocket = require('ws');
const fs = require('fs');
const crypto = require('crypto');

class SecureWebSocketServer {
  constructor(config) {
    this.config = config;
    this.clients = new Map(); // userId -> Set<WebSocket>
    this.encryptionKeys = new Map(); // userId -> key
    this.rateLimiter = new RateLimiter();
    
    this.createServer();
    this.createWebSocketServer();
  }
  
  createServer() {
    this.server = https.createServer({
      cert: fs.readFileSync(this.config.certPath),
      key: fs.readFileSync(this.config.keyPath),
      // Безопасные настройки TLS
      secureOptions: crypto.constants.SSL_OP_NO_SSLv2 |
                     crypto.constants.SSL_OP_NO_SSLv3 |
                     crypto.constants.SSL_OP_NO_TLSv1 |
                     crypto.constants.SSL_OP_NO_TLSv1_1,
      ciphers: [
        'ECDHE-RSA-AES256-GCM-SHA384',
        'ECDHE-RSA-AES128-GCM-SHA256',
        'ECDHE-RSA-AES256-SHA384',
        'ECDHE-RSA-AES128-SHA256'
      ].join(':'),
      honorCipherOrder: true
    });
  }
  
  createWebSocketServer() {
    this.wss = new WebSocket.Server({ 
      server: this.server,
      // Настройки безопасности WebSocket
      verifyClient: (info) => {
        // Проверка Origin
        return this.config.allowedOrigins.includes(info.origin);
      }
    });
    
    this.setupWebSocketHandlers();
  }
  
  setupWebSocketHandlers() {
    this.wss.on('connection', async (ws, req) => {
      try {
        // Аутентификация пользователя
        const authResult = await this.authenticateConnection(req);
        if (!authResult.success) {
          ws.close(1008, authResult.reason);
          return;
        }
        
        const userId = authResult.userId;
        ws.userId = userId;
        
        // Генерация ключа шифрования для пользователя
        const encryptionKey = await this.getOrCreateEncryptionKey(userId);
        ws.encryptionKey = encryptionKey;
        
        // Добавление клиента к списку
        if (!this.clients.has(userId)) {
          this.clients.set(userId, new Set());
        }
        this.clients.get(userId).add(ws);
        
        console.log(`Пользователь ${userId} подключился с защищенным соединением`);
        
        // Установка обработчиков
        this.setupClientHandlers(ws);
        
      } catch (error) {
        console.error('Ошибка подключения:', error);
        ws.close(1001, 'Connection error');
      }
    });
  }
  
  setupClientHandlers(ws) {
    ws.on('message', async (message) => {
      try {
        // Проверка ограничений
        if (!await this.rateLimiter.check(ws.userId)) {
          ws.close(1008, 'Rate limit exceeded');
          return;
        }
        
        // Проверка размера сообщения
        if (message.length > this.config.maxMessageSize) {
          ws.close(1009, 'Message too large');
          return;
        }
        
        // Попытка расшифровки сообщения
        const decryptedMessage = await this.decryptMessage(message, ws.encryptionKey);
        
        // Проверка структуры сообщения
        if (!this.validateMessageStructure(decryptedMessage)) {
          ws.close(1008, 'Invalid message structure');
          return;
        }
        
        // Обработка сообщения
        await this.processMessage(ws, decryptedMessage);
        
      } catch (error) {
        console.error('Ошибка обработки сообщения:', error);
        ws.close(1003, 'Invalid message format');
      }
    });
    
    ws.on('close', () => {
      if (this.clients.has(ws.userId)) {
        this.clients.get(ws.userId).delete(ws);
        if (this.clients.get(ws.userId).size === 0) {
          this.clients.delete(ws.userId);
        }
      }
      console.log(`Пользователь ${ws.userId} отключился`);
    });
    
    ws.on('error', (error) => {
      console.error(`Ошибка WebSocket для пользователя ${ws.userId}:`, error);
    });
  }
  
  async authenticateConnection(req) {
    // Извлечение токена из URL
    const url = new URL(req.url, `https://${req.headers.host}`);
    const token = url.searchParams.get('token');
    
    if (!token) {
      return { success: false, reason: 'Token required' };
    }
    
    try {
      // Проверка JWT токена
      const decoded = jwt.verify(token, this.config.jwtSecret);
      return { success: true, userId: decoded.userId };
    } catch (error) {
      return { success: false, reason: 'Invalid token' };
    }
  }
  
  async getOrCreateEncryptionKey(userId) {
    if (!this.encryptionKeys.has(userId)) {
      // Генерация нового ключа шифрования
      const key = crypto.randomBytes(32); // 256-bit ключ
      this.encryptionKeys.set(userId, key);
    }
    
    return this.encryptionKeys.get(userId);
  }
  
  async decryptMessage(encryptedData, key) {
    try {
      const data = JSON.parse(encryptedData);
      const decipher = crypto.createDecipher('aes-256-cbc', key);
      let decrypted = decipher.update(data.encrypted, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      return JSON.parse(decrypted);
    } catch (error) {
      throw new Error('Failed to decrypt message');
    }
  }
  
  validateMessageStructure(message) {
    // Проверка структуры сообщения
    return message && 
           typeof message === 'object' && 
           message.hasOwnProperty('type') && 
           message.hasOwnProperty('data') &&
           typeof message.type === 'string';
  }
  
  async processMessage(ws, message) {
    // Обработка зашифрованного и проверенного сообщения
    switch (message.type) {
      case 'chat':
        await this.handleChatMessage(ws, message);
        break;
      case 'command':
        await this.handleCommand(ws, message);
        break;
      default:
        console.log(`Неизвестный тип сообщения: ${message.type}`);
    }
  }
  
  async handleChatMessage(ws, message) {
    // Шифрование и отправка сообщения другим пользователям
    const broadcastMessage = {
      type: 'chat',
      data: {
        userId: ws.userId,
        message: message.data.message,
        timestamp: Date.now()
      }
    };
    
    // Шифрование сообщения для отправки
    const encryptedMessage = await this.encryptMessage(broadcastMessage, ws.encryptionKey);
    
    this.wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN && 
          client !== ws && 
          client.userId !== ws.userId) {
        client.send(encryptedMessage);
      }
    });
  }
  
  async encryptMessage(message, key) {
    const cipher = crypto.createCipher('aes-256-cbc', key);
    let encrypted = cipher.update(JSON.stringify(message), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return JSON.stringify({
      encrypted: encrypted
    });
  }
  
  start(port) {
    this.server.listen(port, () => {
      console.log(`Защищенный WebSocket сервер запущен на порту ${port}`);
    });
  }
}

// Использование
const server = new SecureWebSocketServer({
  certPath: '/path/to/certificate.pem',
  keyPath: '/path/to/private-key.pem',
  jwtSecret: process.env.JWT_SECRET,
  allowedOrigins: ['https://trusted-site.com'],
  maxMessageSize: 1024 * 1024 // 1MB
});

server.start(443);
```

### Клиентская сторона с шифрованием

```javascript
class SecureWebSocketClient {
  constructor(url, token, encryptionKey) {
    this.url = `${url}?token=${encodeURIComponent(token)}`;
    this.encryptionKey = encryptionKey;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectInterval = 1000;
  }
  
  async connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      this.reconnectAttempts = 0;
      console.log('Защищенное соединение установлено');
    };
    
    this.ws.onmessage = async (event) => {
      try {
        // Расшифровка полученного сообщения
        const decryptedMessage = await this.decryptMessage(event.data);
        this.handleMessage(decryptedMessage);
      } catch (error) {
        console.error('Ошибка расшифровки сообщения:', error);
      }
    };
    
    this.ws.onclose = (event) => {
      console.log(`Соединение закрыто: ${event.code} - ${event.reason}`);
      
      // Автоматическое переподключение
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        setTimeout(() => {
          this.reconnectAttempts++;
          this.connect();
        }, this.reconnectInterval * this.reconnectAttempts);
      }
    };
    
    this.ws.onerror = (error) => {
      console.error('Ошибка WSS:', error);
    };
  }
  
  async encryptMessage(message) {
    const cipher = crypto.createCipher('aes-256-cbc', this.encryptionKey);
    let encrypted = cipher.update(JSON.stringify(message), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return JSON.stringify({
      encrypted: encrypted
    });
  }
  
  async decryptMessage(encryptedData) {
    const data = JSON.parse(encryptedData);
    const decipher = crypto.createDecipher('aes-256-cbc', this.encryptionKey);
    let decrypted = decipher.update(data.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return JSON.parse(decrypted);
  }
  
  async send(message) {
    if (this.ws.readyState === WebSocket.OPEN) {
      try {
        const encryptedMessage = await this.encryptMessage(message);
        this.ws.send(encryptedMessage);
      } catch (error) {
        console.error('Ошибка шифрования сообщения:', error);
      }
    } else {
      console.error('WebSocket соединение не открыто');
    }
  }
  
  handleMessage(message) {
    // Обработка расшифрованного сообщения
    switch (message.type) {
      case 'chat':
        this.displayChatMessage(message.data);
        break;
      case 'notification':
        this.showNotification(message.data);
        break;
      default:
        console.log('Получено сообщение:', message);
    }
  }
  
  displayChatMessage(data) {
    console.log(`${data.userId}: ${data.message}`);
  }
  
  showNotification(data) {
    console.log('Уведомление:', data);
  }
}
```

## Безопасность шифрования

### Управление ключами

```javascript
class KeyManager {
  constructor() {
    this.keys = new Map(); // userId -> { key, createdAt, lastUsed }
    this.keyRotationInterval = 24 * 60 * 60 * 1000; // 24 часа
  }
  
  generateKey() {
    return crypto.randomBytes(32); // 256-bit ключ
  }
  
  async getOrCreateKey(userId) {
    if (!this.keys.has(userId)) {
      const key = this.generateKey();
      this.keys.set(userId, {
        key: key,
        createdAt: Date.now(),
        lastUsed: Date.now()
      });
    }
    
    const keyData = this.keys.get(userId);
    keyData.lastUsed = Date.now();
    
    // Проверка необходимости ротации ключа
    if (Date.now() - keyData.createdAt > this.keyRotationInterval) {
      await this.rotateKey(userId);
    }
    
    return keyData.key;
  }
  
  async rotateKey(userId) {
    const oldKey = this.keys.get(userId);
    const newKey = this.generateKey();
    
    // Уведомление пользователя о ротации ключа
    // (в реальном приложении нужно безопасно передать новый ключ)
    
    this.keys.set(userId, {
      key: newKey,
      createdAt: Date.now(),
      lastUsed: Date.now()
    });
    
    console.log(`Ключ для пользователя ${userId} обновлен`);
  }
}
```

### Проверка целостности

```javascript
class MessageIntegrity {
  constructor(secret) {
    this.secret = secret;
  }
  
  createSignature(data) {
    const hmac = crypto.createHmac('sha256', this.secret);
    hmac.update(JSON.stringify(data));
    return hmac.digest('hex');
  }
  
  verifySignature(data, signature) {
    const expectedSignature = this.createSignature(data);
    return crypto.timingSafeEqual(
      Buffer.from(signature, 'hex'),
      Buffer.from(expectedSignature, 'hex')
    );
  }
  
  createSignedMessage(message) {
    const signature = this.createSignature(message);
    return {
      message: message,
      signature: signature
    };
  }
  
  verifySignedMessage(signedMessage) {
    const { message, signature } = signedMessage;
    return this.verifySignature(message, signature);
  }
}
```

## Лучшие практики

### Архитектурные рекомендации

- Использование WSS для всех производственных сред
- Дополнительное шифрование чувствительных данных
- Регулярная ротация ключей
- [[Управление-ключами]] для шифрования

### Безопасность

- Проверка SSL-сертификатов
- Защита от downgrade атак
- Использование безопасных алгоритмов шифрования
- [[Сигнализация-безопасности]] для аномального трафика

### Производительность

- Кэширование ключей шифрования
- Асинхронная обработка шифрования
- [[Анализ-логов]] производительности шифрования
- [[Инструменты-мониторинга-безопасности]]

## Современные тенденции

### Post-Quantum Cryptography

Подготовка к квантовым компьютерам:

- Исследование устойчивых к квантовым атакам алгоритмов
- Планирование перехода на квантово-устойчивые методы
- [[Клиентское-шифрование]] с учетом будущих угроз

### Zero Trust для WebSocket

- Проверка на каждом уровне
- Минимальные привилегии
- Постоянная аутентификация
- [[Методы-контроля-доступа]] в Zero Trust архитектуре

> [!tip] Совет
> Всегда используйте WSS в производственных средах и рассмотрите дополнительное шифрование для особенно чувствительных данных.

> [!warning] Важно
> Регулярно обновляйте SSL-сертификаты и следите за уязвимостями в TLS реализациях.

## Заключение

Шифрование WebSocket соединений является критически важным аспектом безопасности современных веб-приложений. Правильная реализация обеспечивает защиту конфиденциальных данных и создает надежную основу для безопасного обмена информацией в реальном времени.