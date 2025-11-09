# WebSocket

WebSocket - это протокол связи поверх TCP, который обеспечивает полнодуплексное (двунаправленное) соединение между веб-браузером и сервером. В отличие от HTTP, WebSocket позволяет отправлять и получать данные в реальном времени без перезапроса.

## Основы WebSocket

### Создание соединения

```javascript
// Создание WebSocket соединения
const ws = new WebSocket('ws://localhost:8080');

// Для защищенного соединения используем wss://
const secureWs = new WebSocket('wss://example.com');

// Обработчики событий
ws.onopen = function(event) {
    console.log('Соединение установлено');
    // Можно начинать отправлять данные
    ws.send('Привет, сервер!');
};

ws.onmessage = function(event) {
    console.log('Получено сообщение:', event.data);
    // Обработка полученных данных
};

ws.onclose = function(event) {
    console.log('Соединение закрыто:', event.code, event.reason);
};

ws.onerror = function(error) {
    console.error('Ошибка WebSocket:', error);
};
```

### Состояния соединения

```javascript
// Состояния WebSocket
const readyStates = {
    0: 'CONNECTING',  // соединение устанавливается
    1: 'OPEN',        // соединение установлено
    2: 'CLOSING',     // соединение закрывается
    3: 'CLOSED'       // соединение закрыто
};

function checkWebSocketState(webSocket) {
    console.log(`Состояние: ${readyStates[webSocket.readyState]}`);
    
    switch (webSocket.readyState) {
        case WebSocket.CONNECTING:
            console.log('Соединение устанавливается...');
            break;
        case WebSocket.OPEN:
            console.log('Соединение открыто, можно отправлять данные');
            break;
        case WebSocket.CLOSING:
            console.log('Соединение закрывается...');
            break;
        case WebSocket.CLOSED:
            console.log('Соединение закрыто');
            break;
    }
}
```

## Отправка и получение данных

### Отправка сообщений

```javascript
// Отправка текстовых сообщений
ws.send('Простое текстовое сообщение');
ws.send(JSON.stringify({ type: 'message', content: 'Привет!' }));

// Отправка бинарных данных
const buffer = new ArrayBuffer(16);
const view = new Uint8Array(buffer);
view[0] = 42;
ws.send(buffer); // ArrayBuffer

const blob = new Blob(['Hello, binary world!'], { type: 'text/plain' });
ws.send(blob); // Blob
```

### Получение сообщений

```javascript
ws.onmessage = function(event) {
    // Тип данных определяется автоматически
    console.log('Тип данных:', typeof event.data);
    
    // Для текстовых данных
    if (typeof event.data === 'string') {
        console.log('Текстовое сообщение:', event.data);
        
        try {
            const message = JSON.parse(event.data);
            console.log('Парсированное сообщение:', message);
        } catch (e) {
            console.log('Простое текстовое сообщение:', event.data);
        }
    }
    // Для бинарных данных (ArrayBuffer)
    else if (event.data instanceof ArrayBuffer) {
        console.log('Бинарные данные получены:', event.data);
        
        const view = new Uint8Array(event.data);
        console.log('Данные как байты:', Array.from(view));
    }
    // Для Blob
    else if (event.data instanceof Blob) {
        const reader = new FileReader();
        reader.onload = function() {
            console.log('Blob как текст:', reader.result);
        };
        reader.readAsText(event.data);
    }
};
```

## Класс для управления WebSocket

```javascript
class WebSocketManager {
    constructor(url, protocols = []) {
        this.url = url;
        this.protocols = protocols;
        this.ws = null;
        this.reconnectInterval = 5000; // 5 секунд
        this.maxReconnectAttempts = 5;
        this.reconnectAttempts = 0;
        this.messageQueue = [];
        this.eventHandlers = new Map();
        this.isManualClose = false;
    }
    
    connect() {
        try {
            this.ws = new WebSocket(this.url, this.protocols);
            
            this.ws.onopen = (event) => {
                console.log('WebSocket соединение установлено');
                this.reconnectAttempts = 0;
                this.isManualClose = false;
                
                // Отправка отложенных сообщений
                this.flushMessageQueue();
                
                // Вызов пользовательских обработчиков
                this.emit('open', event);
            };
            
            this.ws.onmessage = (event) => {
                this.handleMessage(event);
            };
            
            this.ws.onclose = (event) => {
                console.log('WebSocket соединение закрыто:', event.code, event.reason);
                
                // Вызов пользовательских обработчиков
                this.emit('close', event);
                
                // Попытка переподключения, если не было ручного закрытия
                if (!this.isManualClose && this.reconnectAttempts < this.maxReconnectAttempts) {
                    this.scheduleReconnect();
                }
            };
            
            this.ws.onerror = (error) => {
                console.error('Ошибка WebSocket:', error);
                this.emit('error', error);
            };
        } catch (error) {
            console.error('Ошибка создания WebSocket:', error);
            this.emit('error', error);
        }
    }
    
    scheduleReconnect() {
        this.reconnectAttempts++;
        console.log(`Попытка переподключения... (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
        
        setTimeout(() => {
            this.connect();
        }, this.reconnectInterval);
    }
    
    send(message) {
        if (!this.ws) {
            // Добавление в очередь до установления соединения
            this.messageQueue.push(message);
            return false;
        }
        
        if (this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(message));
            return true;
        } else if (this.ws.readyState === WebSocket.CONNECTING) {
            // Добавление в очередь при установлении соединения
            this.messageQueue.push(message);
            return false;
        } else {
            console.warn('WebSocket соединение не установлено');
            return false;
        }
    }
    
    flushMessageQueue() {
        while (this.messageQueue.length > 0) {
            const message = this.messageQueue.shift();
            this.send(message);
        }
    }
    
    handleMessage(event) {
        try {
            const data = JSON.parse(event.data);
            this.emit('message', data);
            
            // Обработка специальных типов сообщений
            if (data.type) {
                this.emit(`message:${data.type}`, data.payload);
            }
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
            // Вызов обработчика для неструктурированных сообщений
            this.emit('rawMessage', event.data);
        }
    }
    
    // Подписка на события
    on(event, handler) {
        if (!this.eventHandlers.has(event)) {
            this.eventHandlers.set(event, []);
        }
        this.eventHandlers.get(event).push(handler);
    }
    
    // Отписка от событий
    off(event, handler) {
        if (this.eventHandlers.has(event)) {
            const handlers = this.eventHandlers.get(event);
            const index = handlers.indexOf(handler);
            if (index > -1) {
                handlers.splice(index, 1);
            }
        }
    }
    
    // Вызов обработчиков событий
    emit(event, data) {
        if (this.eventHandlers.has(event)) {
            this.eventHandlers.get(event).forEach(handler => {
                try {
                    handler(data);
                } catch (error) {
                    console.error(`Ошибка в обработчике события ${event}:`, error);
                }
            });
        }
    }
    
    // Закрытие соединения
    close(code = 1000, reason = 'Normal closure') {
        this.isManualClose = true;
        if (this.ws && (this.ws.readyState === WebSocket.OPEN || this.ws.readyState === WebSocket.CONNECTING)) {
            this.ws.close(code, reason);
        }
    }
    
    // Получение состояния
    getReadyState() {
        if (!this.ws) return WebSocket.CLOSED;
        return this.ws.readyState;
    }
    
    // Проверка, открыто ли соединение
    isOpen() {
        return this.ws && this.ws.readyState === WebSocket.OPEN;
    }
}

// Использование
const wsManager = new WebSocketManager('ws://localhost:8080');

wsManager.on('open', () => {
    console.log('Соединение готово к отправке сообщений');
    wsManager.send({ type: 'auth', token: 'user-token' });
});

wsManager.on('message', (data) => {
    console.log('Получено сообщение:', data);
});

wsManager.on('message:chat', (chatMessage) => {
    console.log('Чат сообщение:', chatMessage);
});

wsManager.connect();
```

## Продвинутые возможности

### Аутентификация и авторизация

```javascript
class AuthenticatedWebSocketManager extends WebSocketManager {
    constructor(url, authToken) {
        super(url);
        this.authToken = authToken;
        this.isAuthenticated = false;
        this.authRetryCount = 0;
        this.maxAuthRetries = 3;
    }
    
    connect() {
        super.connect();
        
        // После открытия соединения отправляем токен аутентификации
        this.on('open', () => {
            this.authenticate();
        });
        
        // Обработка сообщений аутентификации
        this.on('message:auth', (authResponse) => {
            if (authResponse.success) {
                this.isAuthenticated = true;
                this.authRetryCount = 0;
                console.log('Аутентификация успешна');
                this.emit('authenticated');
            } else {
                console.error('Ошибка аутентификации:', authResponse.error);
                
                if (this.authRetryCount < this.maxAuthRetries) {
                    this.authRetryCount++;
                    setTimeout(() => this.authenticate(), 1000 * this.authRetryCount);
                } else {
                    this.emit('authFailed', authResponse.error);
                    this.close(4001, 'Authentication failed');
                }
            }
        });
    }
    
    authenticate() {
        if (this.isAuthenticated) return;
        
        this.send({
            type: 'auth',
            token: this.authToken
        });
    }
    
    send(message) {
        // Проверка аутентификации перед отправкой
        if (!this.isAuthenticated && message.type !== 'auth') {
            console.warn('Попытка отправки сообщения без аутентификации');
            return false;
        }
        
        return super.send(message);
    }
}

// Использование
const authWs = new AuthenticatedWebSocketManager('ws://chat.example.com', 'user-jwt-token');
authWs.on('authenticated', () => {
    console.log('Пользователь аутентифицирован, можно отправлять сообщения');
    authWs.send({ type: 'join', room: 'general' });
});
authWs.connect();
```

### Управление комнатами/каналами

```javascript
class ChannelWebSocketManager extends WebSocketManager {
    constructor(url) {
        super(url);
        this.joinedChannels = new Set();
        this.channelHandlers = new Map();
    }
    
    joinChannel(channelName) {
        if (this.joinedChannels.has(channelName)) {
            console.log(`Уже подключен к каналу: ${channelName}`);
            return;
        }
        
        this.send({
            type: 'join',
            channel: channelName
        });
    }
    
    leaveChannel(channelName) {
        if (!this.joinedChannels.has(channelName)) {
            return;
        }
        
        this.send({
            type: 'leave',
            channel: channelName
        });
    }
    
    subscribeToChannel(channelName, handler) {
        if (!this.channelHandlers.has(channelName)) {
            this.channelHandlers.set(channelName, []);
        }
        this.channelHandlers.get(channelName).push(handler);
    }
    
    handleMessage(event) {
        try {
            const data = JSON.parse(event.data);
            
            // Обработка сообщений каналов
            if (data.channel && this.channelHandlers.has(data.channel)) {
                const handlers = this.channelHandlers.get(data.channel);
                handlers.forEach(handler => handler(data.payload));
            }
            
            // Стандартная обработка
            super.handleMessage(event);
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
        }
    }
    
    onJoinSuccess(channelName) {
        this.joinedChannels.add(channelName);
        console.log(`Присоединен к каналу: ${channelName}`);
        this.emit('channelJoined', channelName);
    }
    
    onLeaveSuccess(channelName) {
        this.joinedChannels.delete(channelName);
        console.log(`Покинул канал: ${channelName}`);
        this.emit('channelLeft', channelName);
    }
}

// Использование
const channelWs = new ChannelWebSocketManager('ws://chat.example.com');
channelWs.connect();

channelWs.subscribeToChannel('general', (message) => {
    console.log('Сообщение в общем канале:', message);
});

channelWs.on('channelJoined', (channel) => {
    console.log(`Присоединен к каналу: ${channel}`);
});

channelWs.joinChannel('general');
```

## Практические примеры

### Чат-приложение

```javascript
class ChatClient {
    constructor(webSocketUrl) {
        this.ws = new ChannelWebSocketManager(webSocketUrl);
        this.users = new Map(); // userId -> userInfo
        this.messageHistory = []; // история сообщений
        this.maxHistorySize = 1000;
        
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        this.ws.on('open', () => {
            console.log('Чат-клиент подключен');
        });
        
        this.ws.on('message:userJoined', (userData) => {
            this.users.set(userData.id, userData);
            this.displaySystemMessage(`${userData.name} присоединился к чату`);
        });
        
        this.ws.on('message:userLeft', (userId) => {
            const user = this.users.get(userId);
            if (user) {
                this.users.delete(userId);
                this.displaySystemMessage(`${user.name} покинул чат`);
            }
        });
        
        this.ws.on('message:chat', (chatData) => {
            this.addMessageToHistory(chatData);
            this.displayMessage(chatData);
        });
        
        this.ws.on('message:userList', (userList) => {
            this.users.clear();
            userList.forEach(user => {
                this.users.set(user.id, user);
            });
            this.updateUserList();
        });
    }
    
    connect() {
        this.ws.connect();
    }
    
    sendMessage(text, channel = 'general') {
        if (!this.ws.isOpen()) {
            console.error('Соединение не установлено');
            return;
        }
        
        const message = {
            type: 'chat',
            text: text,
            timestamp: Date.now(),
            channel: channel
        };
        
        this.ws.send(message);
    }
    
    joinRoom(roomName) {
        this.ws.joinChannel(roomName);
    }
    
    addMessageToHistory(message) {
        this.messageHistory.push(message);
        
        // Ограничение размера истории
        if (this.messageHistory.length > this.maxHistorySize) {
            this.messageHistory = this.messageHistory.slice(-this.maxHistorySize);
        }
    }
    
    displayMessage(message) {
        const user = this.users.get(message.userId);
        const userName = user ? user.name : 'Неизвестный';
        
        console.log(`[${new Date(message.timestamp).toLocaleTimeString()}] ${userName}: ${message.text}`);
    }
    
    displaySystemMessage(text) {
        console.log(`[СИСТЕМА] ${text}`);
    }
    
    updateUserList() {
        console.log('Пользователи в чате:');
        for (const [id, user] of this.users) {
            console.log(`- ${user.name} (${user.id})`);
        }
    }
    
    disconnect() {
        this.ws.close();
    }
}

// Использование
const chat = new ChatClient('ws://chat.example.com');
chat.connect();
chat.joinRoom('general');

// Отправка сообщения
chat.sendMessage('Привет, мир!');
```

### Live-данные и мониторинг

```javascript
class LiveDataMonitor {
    constructor(webSocketUrl) {
        this.ws = new WebSocketManager(webSocketUrl);
        this.dataCallbacks = new Map();
        this.monitoringIntervals = new Map();
        
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        this.ws.on('open', () => {
            console.log('Соединение для мониторинга данных установлено');
            // Подписка на отслеживаемые данные
            this.resubscribe();
        });
        
        this.ws.on('message:dataUpdate', (data) => {
            this.handleDataUpdate(data);
        });
        
        this.ws.on('message:metricUpdate', (metric) => {
            this.handleMetricUpdate(metric);
        });
    }
    
    // Подписка на обновления данных
    subscribeToData(dataId, callback) {
        if (!this.dataCallbacks.has(dataId)) {
            this.dataCallbacks.set(dataId, []);
        }
        this.dataCallbacks.get(dataId).push(callback);
        
        // Отправка запроса на подписку
        if (this.ws.isOpen()) {
            this.ws.send({
                type: 'subscribe',
                dataId: dataId
            });
        }
    }
    
    // Отписка от обновлений данных
    unsubscribeFromData(dataId) {
        this.dataCallbacks.delete(dataId);
        
        if (this.ws.isOpen()) {
            this.ws.send({
                type: 'unsubscribe',
                dataId: dataId
            });
        }
    }
    
    // Подписка на метрики с интервалом
    subscribeToMetrics(interval = 1000, callback) {
        const intervalId = setInterval(() => {
            if (this.ws.isOpen()) {
                this.ws.send({
                    type: 'getMetrics'
                });
            }
        }, interval);
        
        this.monitoringIntervals.set(callback, intervalId);
        
        // Подписка на обновления метрик
        this.ws.on('message:metricUpdate', callback);
    }
    
    // Отписка от метрик
    unsubscribeFromMetrics(callback) {
        const intervalId = this.monitoringIntervals.get(callback);
        if (intervalId) {
            clearInterval(intervalId);
            this.monitoringIntervals.delete(callback);
        }
        
        this.ws.off('message:metricUpdate', callback);
    }
    
    handleDataUpdate(data) {
        if (this.dataCallbacks.has(data.dataId)) {
            const callbacks = this.dataCallbacks.get(data.dataId);
            callbacks.forEach(callback => {
                try {
                    callback(data.value, data.timestamp);
                } catch (error) {
                    console.error('Ошибка в обработчике обновления данных:', error);
                }
            });
        }
    }
    
    handleMetricUpdate(metric) {
        // Обработка метрик (загрузка CPU, память и т.д.)
        console.log('Метрики обновлены:', metric);
    }
    
    resubscribe() {
        // Повторная подписка после восстановления соединения
        for (const dataId of this.dataCallbacks.keys()) {
            this.ws.send({
                type: 'subscribe',
                dataId: dataId
            });
        }
    }
    
    connect() {
        this.ws.connect();
    }
    
    disconnect() {
        this.ws.close();
    }
}

// Использование
const monitor = new LiveDataMonitor('ws://metrics.example.com');
monitor.connect();

// Подписка на обновления цены акции
monitor.subscribeToData('stock_price_AAPL', (price, timestamp) => {
    console.log(`Цена AAPL: $${price} на ${new Date(timestamp).toLocaleTimeString()}`);
});

// Подписка на метрики сервера
monitor.subscribeToMetrics(5000, (metrics) => {
    console.log(`Загрузка CPU: ${metrics.cpu}%`);
    console.log(`Использование памяти: ${metrics.memory}%`);
});
```

## Обработка ошибок и восстановление

### Стратегии восстановления соединения

```javascript
class RobustWebSocketManager extends WebSocketManager {
    constructor(url, options = {}) {
        super(url);
        
        this.options = {
            reconnectInterval: 5000,
            maxReconnectInterval: 30000,
            reconnectDecay: 1.5,
            timeout: 10000,
            heartbeatInterval: 30000,
            ...options
        };
        
        this.heartbeatTimeout = null;
        this.lastHeartbeat = null;
    }
    
    connect() {
        this.isManualClose = false;
        super.connect();
    }
    
    scheduleReconnect() {
        this.reconnectAttempts++;
        
        // Экспоненциальный рост интервала переподключения
        const interval = Math.min(
            this.options.reconnectInterval * Math.pow(this.options.reconnectDecay, this.reconnectAttempts - 1),
            this.options.maxReconnectInterval
        );
        
        console.log(`Попытка переподключения через ${interval}мс... (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
        
        setTimeout(() => {
            this.connect();
        }, interval);
    }
    
    startHeartbeat() {
        if (this.heartbeatIntervalId) {
            clearInterval(this.heartbeatIntervalId);
        }
        
        this.heartbeatIntervalId = setInterval(() => {
            if (this.isOpen()) {
                // Отправка ping сообщения
                this.send({ type: 'ping' });
                
                // Установка таймаута для pong
                this.heartbeatTimeout = setTimeout(() => {
                    console.log('Таймаут heartbeat, переподключение...');
                    this.ws.close();
                }, this.options.timeout);
            }
        }, this.options.heartbeatInterval);
    }
    
    handlePong() {
        // Получен pong, сброс таймаута
        if (this.heartbeatTimeout) {
            clearTimeout(this.heartbeatTimeout);
            this.heartbeatTimeout = null;
        }
        
        this.lastHeartbeat = Date.now();
    }
    
    handleMessage(event) {
        try {
            const data = JSON.parse(event.data);
            
            // Обработка heartbeat сообщений
            if (data.type === 'pong') {
                this.handlePong();
                return;
            }
            
            super.handleMessage(event);
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
        }
    }
    
    on(event, handler) {
        // Особый обработчик для heartbeat
        if (event === 'pong') {
            const originalHandler = handler;
            handler = () => {
                this.handlePong();
                originalHandler();
            };
        }
        
        super.on(event, handler);
    }
    
    close(code = 1000, reason = 'Normal closure') {
        this.isManualClose = true;
        
        // Очистка heartbeat
        if (this.heartbeatIntervalId) {
            clearInterval(this.heartbeatIntervalId);
            this.heartbeatIntervalId = null;
        }
        
        if (this.heartbeatTimeout) {
            clearTimeout(this.heartbeatTimeout);
            this.heartbeatTimeout = null;
        }
        
        super.close(code, reason);
    }
}
```

## Лучшие практики

### 1. Управление памятью

```javascript
// Правильная очистка ресурсов
class ManagedWebSocket extends WebSocketManager {
    constructor(url) {
        super(url);
        this.eventListeners = new Map();
    }
    
    on(event, handler) {
        super.on(event, handler);
        
        if (!this.eventListeners.has(event)) {
            this.eventListeners.set(event, []);
        }
        this.eventListeners.get(event).push(handler);
    }
    
    destroy() {
        // Закрытие соединения
        this.close();
        
        // Очистка обработчиков событий
        this.eventListeners.clear();
        
        // Очистка других ресурсов
        if (this.ws) {
            this.ws.onopen = null;
            this.ws.onmessage = null;
            this.ws.onclose = null;
            this.ws.onerror = null;
        }
    }
}
```

### 2. Валидация сообщений

```javascript
// Валидация входящих сообщений
class ValidatedWebSocketManager extends WebSocketManager {
    constructor(url) {
        super(url);
        this.messageValidators = new Map();
    }
    
    addValidator(messageType, validator) {
        this.messageValidators.set(messageType, validator);
    }
    
    handleMessage(event) {
        try {
            const data = JSON.parse(event.data);
            
            // Валидация сообщения
            if (data.type && this.messageValidators.has(data.type)) {
                const validator = this.messageValidators.get(data.type);
                if (!validator(data)) {
                    console.error(`Невалидное сообщение типа ${data.type}:`, data);
                    return;
                }
            }
            
            super.handleMessage(event);
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
        }
    }
}

// Пример валидаторов
const ws = new ValidatedWebSocketManager('ws://example.com');
ws.addValidator('chat', (data) => {
    return typeof data.text === 'string' && 
           typeof data.userId === 'string' && 
           data.text.length <= 1000;
});
ws.addValidator('userUpdate', (data) => {
    return typeof data.userId === 'string' && 
           typeof data.profile === 'object';
});
```

### 3. Логирование и мониторинг

```javascript
// Система логирования WebSocket
class LoggedWebSocketManager extends WebSocketManager {
    constructor(url) {
        super(url);
        this.messageLog = [];
        this.maxLogSize = 1000;
    }
    
    send(message) {
        const logEntry = {
            type: 'outgoing',
            message: message,
            timestamp: Date.now()
        };
        this.addToLog(logEntry);
        
        return super.send(message);
    }
    
    handleMessage(event) {
        const logEntry = {
            type: 'incoming',
            message: event.data,
            timestamp: Date.now()
        };
        this.addToLog(logEntry);
        
        super.handleMessage(event);
    }
    
    addToLog(entry) {
        this.messageLog.push(entry);
        
        if (this.messageLog.length > this.maxLogSize) {
            this.messageLog = this.messageLog.slice(-this.maxLogSize);
        }
    }
    
    getLog() {
        return this.messageLog;
    }
    
    exportLog(filename = 'websocket-log.json') {
        const dataStr = JSON.stringify(this.messageLog, null, 2);
        const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
        
        const link = document.createElement('a');
        link.setAttribute('href', dataUri);
        link.setAttribute('download', filename);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }
}
```

WebSocket предоставляет мощный способ для создания приложений с реальным временем. При правильной реализации он позволяет создавать интерактивные и отзывчивые веб-приложения с минимальной задержкой передачи данных.

#javascript #websocket #realtime #network #frontend #web-development