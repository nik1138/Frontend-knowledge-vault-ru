---
aliases: ["Безопасность CORS", "Cross-Origin Resource Sharing", "CORS защита"]
tags: ["#security", "#cors", "#api-security", "#web-security"]
---

# Безопасность CORS

## Введение

Cross-Origin Resource Sharing (CORS) - это механизм безопасности, который позволяет веб-приложениям с одного домена получать доступ к ресурсам с другого домена. Безопасная настройка CORS критически важна для предотвращения атак типа "cross-site request forgery" (CSRF) и других уязвимостей, связанных с кросс-доменными запросами.

## Как работает CORS

CORS работает через специальные HTTP-заголовки, которые позволяют серверу указать, какие домены, методы и заголовки разрешены для кросс-доменных запросов.

### Простые запросы
- GET, POST, HEAD методы
- Ограниченный набор заголовков
- Не требуют preflight-запросов

### Preflight-запросы
- Запросы с нестандартными методами (PUT, DELETE, PATCH)
- Запросы с кастомными заголовками
- Требуют предварительной проверки OPTIONS-запросом

## Безопасная настройка CORS

### 1. Базовая безопасная настройка

```javascript
const cors = require('cors');

// Безопасная настройка CORS
const corsOptions = {
    origin: function (origin, callback) {
        // В продакшене использовать белый список доменов
        const allowedOrigins = [
            'https://yourapp.com',
            'https://www.yourapp.com',
            'https://staging.yourapp.com',
            process.env.FRONTEND_URL // из переменных окружения
        ];
        
        // Если origin отсутствует (сервер-серверный запрос), разрешить
        if (!origin) {
            callback(null, true);
            return;
        }
        
        if (allowedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true, // Разрешить отправку куки и аутентификационных заголовков
    optionsSuccessStatus: 200, // Успешный ответ для OPTIONS запросов
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD'],
    allowedHeaders: [
        'Content-Type',
        'Authorization',
        'X-Requested-With',
        'Accept',
        'Origin'
    ],
    exposedHeaders: [
        'X-Total-Count',
        'X-RateLimit-Remaining',
        'X-RateLimit-Reset'
    ]
};

app.use(cors(corsOptions));
```

### 2. Динамическая настройка CORS

```javascript
// Класс для динамического управления CORS
class DynamicCORSManager {
    constructor() {
        this.allowedOrigins = new Set();
        this.allowedOrigins.add('https://yourapp.com');
        this.allowedOrigins.add('https://www.yourapp.com');
        
        // Загрузка из конфигурации
        this.loadConfiguration();
    }
    
    loadConfiguration() {
        // Загрузка разрешенных доменов из конфигурации или БД
        const configuredOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
        configuredOrigins.forEach(origin => this.allowedOrigins.add(origin.trim()));
    }
    
    addOrigin(origin) {
        this.allowedOrigins.add(origin);
    }
    
    removeOrigin(origin) {
        this.allowedOrigins.delete(origin);
    }
    
    isOriginAllowed(origin) {
        if (!origin) return true; // Сервер-серверные запросы
        
        // Проверка точного совпадения
        if (this.allowedOrigins.has(origin)) {
            return true;
        }
        
        // Проверка с подстановкой (например, *.yourapp.com)
        for (const allowedOrigin of this.allowedOrigins) {
            if (this.matchesWildcard(allowedOrigin, origin)) {
                return true;
            }
        }
        
        return false;
    }
    
    matchesWildcard(pattern, origin) {
        if (pattern.startsWith('*.') && origin.endsWith(pattern.substring(1))) {
            return true;
        }
        return false;
    }
    
    getMiddleware() {
        return (req, res, next) => {
            const origin = req.get('Origin');
            
            if (this.isOriginAllowed(origin)) {
                res.header('Access-Control-Allow-Origin', origin);
            } else {
                res.status(403).json({ error: 'CORS not allowed' });
                return;
            }
            
            res.header('Access-Control-Allow-Credentials', 'true');
            res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
            res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
            res.header('Access-Control-Max-Age', '86400'); // 24 часа
            
            if (req.method === 'OPTIONS') {
                res.sendStatus(200);
                return;
            }
            
            next();
        };
    }
}

const corsManager = new DynamicCORSManager();
app.use(corsManager.getMiddleware());
```

## Уязвимости и атаки через CORS

### 1. Уязвимость "Origin Reflection"

```javascript
// ПЛОХО - отражение любого origin
app.use(cors({
    origin: true // ОПАСНО! Разрешает любой origin
}));

// ПЛОХО - регулярное выражение с уязвимостью
app.use(cors({
    origin: /.*\.yourapp\.com$/ // Может быть обойдено через evil-yourapp.com
}));

// ХОРОШО - явный белый список
app.use(cors({
    origin: [
        'https://yourapp.com',
        'https://www.yourapp.com'
    ]
}));
```

### 2. Уязвимость "Null Origin"

```javascript
// Небезопасная обработка null origin
function unsafeOriginCheck(origin) {
    if (!origin) return true; // ОПАСНО! null origin может быть использован в атаках
    return allowedOrigins.includes(origin);
}

// Безопасная обработка null origin
function safeOriginCheck(origin) {
    if (!origin) {
        // Проверить, что это действительно сервер-серверный запрос
        const isServerToServer = req.ip === 'internal-server-ip';
        return isServerToServer;
    }
    return allowedOrigins.includes(origin);
}
```

### 3. Уязвимость "Wildcard Origin"

```javascript
// ПЛОХО - wildcard разрешает все домены
app.use(cors({
    origin: '*' // ОПАСНО! Особенно с credentials: true
}));

// ПЛОХО - wildcard с credentials
app.use(cors({
    origin: '*',
    credentials: true // ОПАСНО! Это создает уязвимость
}));

// ХОРОШО - явные домены с credentials
app.use(cors({
    origin: ['https://yourapp.com', 'https://www.yourapp.com'],
    credentials: true
}));
```

## Практические примеры безопасной настройки

### 1. Настройка для разных окружений

```javascript
// Конфигурация CORS для разных окружений
const corsConfig = {
    development: {
        origin: [
            'http://localhost:3000',
            'http://localhost:3001',
            'http://127.0.0.1:3000'
        ],
        credentials: true
    },
    staging: {
        origin: [
            'https://staging.yourapp.com',
            'https://test.yourapp.com'
        ],
        credentials: true
    },
    production: {
        origin: [
            'https://yourapp.com',
            'https://www.yourapp.com'
        ],
        credentials: true
    }
};

const environment = process.env.NODE_ENV || 'development';
app.use(cors(corsConfig[environment]));
```

### 2. Условная настройка CORS

```javascript
// Условная настройка CORS на основе маршрутов
function conditionalCors(req, res, next) {
    // Для публичных API не требовать строгой CORS
    if (req.path.startsWith('/api/public/')) {
        res.header('Access-Control-Allow-Origin', '*');
        res.header('Access-Control-Allow-Methods', 'GET, POST');
    } 
    // Для чувствительных API - строгая проверка
    else if (req.path.startsWith('/api/private/')) {
        const origin = req.get('Origin');
        if (isTrustedOrigin(origin)) {
            res.header('Access-Control-Allow-Origin', origin);
        } else {
            return res.status(403).json({ error: 'CORS not allowed' });
        }
    }
    // Для остальных API - стандартная проверка
    else {
        if (!isTrustedOrigin(req.get('Origin'))) {
            return res.status(403).json({ error: 'CORS not allowed' });
        }
    }
    
    res.header('Access-Control-Allow-Credentials', 'true');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    if (req.method === 'OPTIONS') {
        res.sendStatus(200);
        return;
    }
    
    next();
}

function isTrustedOrigin(origin) {
    const trustedOrigins = [
        'https://yourapp.com',
        'https://www.yourapp.com',
        process.env.FRONTEND_URL
    ];
    
    return !origin || trustedOrigins.includes(origin);
}

app.use('/api/', conditionalCors);
```

### 3. CORS с проверкой токенов

```javascript
// CORS с дополнительной проверкой токенов
class SecureCORSWithTokens {
    constructor() {
        this.trustedOrigins = new Set([
            'https://yourapp.com',
            'https://www.yourapp.com'
        ]);
    }
    
    middleware() {
        return (req, res, next) => {
            const origin = req.get('Origin');
            const token = req.headers.authorization?.split(' ')[1];
            
            // Проверка origin
            if (origin && !this.trustedOrigins.has(origin)) {
                return res.status(403).json({ error: 'CORS not allowed' });
            }
            
            // Для критических операций требовать токен
            if (this.isCriticalEndpoint(req) && !token) {
                return res.status(401).json({ error: 'Authorization required' });
            }
            
            // Установка CORS заголовков
            res.header('Access-Control-Allow-Origin', origin);
            res.header('Access-Control-Allow-Credentials', 'true');
            res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
            res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
            res.header('Access-Control-Expose-Headers', 'X-Total-Count, X-RateLimit-Remaining');
            
            if (req.method === 'OPTIONS') {
                res.sendStatus(200);
                return;
            }
            
            next();
        };
    }
    
    isCriticalEndpoint(req) {
        return req.path.includes('/admin/') || 
               req.path.includes('/payment/') || 
               req.method === 'DELETE';
    }
}

const secureCORS = new SecureCORSWithTokens();
app.use('/api/', secureCORS.middleware());
```

## Продвинутые техники

### 1. CORS с динамической политикой

```javascript
// Система динамической политики CORS
class DynamicCORSPolicy {
    constructor() {
        this.policies = new Map();
        this.loadPolicies();
    }
    
    loadPolicies() {
        // Загрузка политик из конфигурации
        this.policies.set('/api/public/*', {
            origins: ['*'],
            methods: ['GET', 'POST'],
            credentials: false
        });
        
        this.policies.set('/api/user/*', {
            origins: [
                'https://yourapp.com',
                'https://www.yourapp.com',
                process.env.MOBILE_APP_ORIGIN
            ],
            methods: ['GET', 'POST', 'PUT', 'DELETE'],
            credentials: true
        });
        
        this.policies.set('/api/admin/*', {
            origins: ['https://admin.yourapp.com'],
            methods: ['GET', 'POST', 'PUT', 'DELETE'],
            credentials: true
        });
    }
    
    getPolicyForPath(path) {
        for (const [pattern, policy] of this.policies) {
            if (this.pathMatchesPattern(path, pattern)) {
                return policy;
            }
        }
        return null;
    }
    
    pathMatchesPattern(path, pattern) {
        // Простая проверка с подстановкой
        if (pattern.endsWith('/*')) {
            const basePath = pattern.slice(0, -2); // убираем '/*'
            return path.startsWith(basePath);
        }
        return path === pattern;
    }
    
    middleware() {
        return (req, res, next) => {
            const policy = this.getPolicyForPath(req.path);
            
            if (!policy) {
                return res.status(403).json({ error: 'CORS policy not found' });
            }
            
            const origin = req.get('Origin');
            
            // Проверка origin в зависимости от политики
            if (policy.origins[0] !== '*') { // Не wildcard
                if (!policy.origins.includes(origin)) {
                    return res.status(403).json({ error: 'CORS not allowed' });
                }
            }
            
            // Установка заголовков в соответствии с политикой
            if (policy.origins[0] !== '*') {
                res.header('Access-Control-Allow-Origin', origin);
            } else {
                res.header('Access-Control-Allow-Origin', '*');
            }
            
            res.header('Access-Control-Allow-Methods', policy.methods.join(', '));
            res.header('Access-Control-Allow-Credentials', policy.credentials.toString());
            res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
            
            if (req.method === 'OPTIONS') {
                res.sendStatus(200);
                return;
            }
            
            next();
        };
    }
}

const dynamicCORS = new DynamicCORSPolicy();
app.use('/api/', dynamicCORS.middleware());
```

### 2. CORS с аудитом и мониторингом

```javascript
// CORS с системой аудита
class AuditedCORS {
    constructor() {
        this.auditLog = [];
        this.maxLogSize = 10000;
    }
    
    middleware() {
        return (req, res, next) => {
            const origin = req.get('Origin');
            const userAgent = req.get('User-Agent');
            const ip = req.ip;
            const method = req.method;
            const path = req.path;
            
            // Логирование попытки CORS
            this.logCORSAttempt({
                origin,
                userAgent,
                ip,
                method,
                path,
                timestamp: new Date().toISOString()
            });
            
            // Проверка и установка CORS заголовков
            if (this.isOriginAllowed(origin)) {
                res.header('Access-Control-Allow-Origin', origin);
                res.header('Access-Control-Allow-Credentials', 'true');
                res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
                res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
                
                // Логирование успешной попытки
                this.logCORSResult({
                    origin,
                    ip,
                    path,
                    result: 'ALLOWED',
                    timestamp: new Date().toISOString()
                });
            } else {
                // Логирование отклоненной попытки
                this.logCORSResult({
                    origin,
                    ip,
                    path,
                    result: 'BLOCKED',
                    timestamp: new Date().toISOString()
                });
                
                return res.status(403).json({ error: 'CORS not allowed' });
            }
            
            if (req.method === 'OPTIONS') {
                res.sendStatus(200);
                return;
            }
            
            next();
        };
    }
    
    isOriginAllowed(origin) {
        if (!origin) return true; // Сервер-серверные запросы
        
        const allowedOrigins = [
            'https://yourapp.com',
            'https://www.yourapp.com',
            process.env.FRONTEND_URL
        ];
        
        return allowedOrigins.includes(origin);
    }
    
    logCORSAttempt(attempt) {
        this.auditLog.push({
            type: 'cors_attempt',
            ...attempt
        });
        
        // Ограничение размера лога
        if (this.auditLog.length > this.maxLogSize) {
            this.auditLog = this.auditLog.slice(-this.maxLogSize);
        }
    }
    
    logCORSResult(result) {
        this.auditLog.push({
            type: 'cors_result',
            ...result
        });
        
        // Анализ подозрительной активности
        this.analyzeSuspiciousActivity(result);
    }
    
    analyzeSuspiciousActivity(result) {
        // Простой анализ: много попыток с одного IP
        const recentAttempts = this.auditLog
            .filter(log => log.type === 'cors_result' && 
                         log.ip === result.ip && 
                         Date.now() - new Date(log.timestamp).getTime() < 60000) // за последнюю минуту
            .length;
        
        if (recentAttempts > 10) {
            console.warn('SUSPICIOUS CORS ACTIVITY:', {
                ip: result.ip,
                count: recentAttempts,
                origin: result.origin
            });
        }
    }
    
    getAuditReport() {
        return {
            totalAttempts: this.auditLog.length,
            allowedRequests: this.auditLog.filter(log => log.result === 'ALLOWED').length,
            blockedRequests: this.auditLog.filter(log => log.result === 'BLOCKED').length,
            suspiciousIPs: this.getSuspiciousIPs()
        };
    }
    
    getSuspiciousIPs() {
        // Группировка по IP и подсчет попыток
        const ipCounts = {};
        this.auditLog.forEach(log => {
            if (log.ip) {
                ipCounts[log.ip] = (ipCounts[log.ip] || 0) + 1;
            }
        });
        
        return Object.entries(ipCounts)
            .filter(([ip, count]) => count > 100) // больше 100 попыток
            .map(([ip, count]) => ({ ip, count }));
    }
}

const auditedCORS = new AuditedCORS();
app.use('/api/', auditedCORS.middleware());

// Эндпоинт для получения отчета аудита
app.get('/admin/cors-audit', (req, res) => {
    if (req.user?.role !== 'admin') {
        return res.status(403).json({ error: 'Admin access required' });
    }
    
    res.json(auditedCORS.getAuditReport());
});
```

## Лучшие практики

### 1. Минимизация разрешенных доменов

```javascript
// Хорошая практика: минимальный набор разрешенных доменов
const secureCORSPolicy = {
    origin: [
        'https://yourapp.com',        // основной домен
        'https://www.yourapp.com'     // www версия
    ],
    credentials: true,
    methods: ['GET', 'POST'], // только необходимые методы
    allowedHeaders: ['Content-Type', 'Authorization'] // только необходимые заголовки
};
```

### 2. Использование Content Security Policy в дополнение к CORS

```javascript
// Дополнительная защита через CSP
app.use((req, res, next) => {
    res.setHeader('Content-Security-Policy', 
        "default-src 'self'; " +
        "frame-ancestors 'self'; " +
        "connect-src 'self' https://api.yourapp.com;"
    );
    next();
});
```

### 3. Проверка реальных сценариев использования

```javascript
// Проверка CORS в реальных сценариях
class CORSPolicyValidator {
    static validateConfiguration(config) {
        const errors = [];
        
        // Проверка на wildcard с credentials
        if (config.origin === '*' && config.credentials === true) {
            errors.push('Wildcard origin with credentials is dangerous');
        }
        
        // Проверка на небезопасные регулярные выражения
        if (typeof config.origin === 'object' && config.origin.constructor === RegExp) {
            const regexStr = config.origin.toString();
            if (regexStr.includes('.*')) {
                errors.push('Unsafe regex pattern detected');
            }
        }
        
        // Проверка разрешенных методов
        const unsafeMethods = ['TRACE', 'TRACK'];
        if (config.methods && config.methods.some(m => unsafeMethods.includes(m))) {
            errors.push('Unsafe HTTP methods detected');
        }
        
        return {
            valid: errors.length === 0,
            errors
        };
    }
}

// Использование валидатора
const corsValidation = CORSPolicyValidator.validateConfiguration(corsOptions);
if (!corsValidation.valid) {
    console.error('CORS Configuration Errors:', corsValidation.errors);
    process.exit(1);
}
```

## Тестирование CORS безопасности

### 1. Автоматизированное тестирование

```javascript
// Тестирование безопасности CORS
describe('CORS Security', () => {
    test('should block requests from untrusted origins', async () => {
        const response = await request(app)
            .get('/api/data')
            .set('Origin', 'https://evil-site.com');
        
        expect(response.status).toBe(403);
        expect(response.body.error).toBe('CORS not allowed');
    });
    
    test('should allow requests from trusted origins', async () => {
        const response = await request(app)
            .get('/api/data')
            .set('Origin', 'https://yourapp.com');
        
        expect(response.status).toBe(200);
        expect(response.headers['access-control-allow-origin']).toBe('https://yourapp.com');
    });
    
    test('should handle preflight requests correctly', async () => {
        const response = await request(app)
            .options('/api/data')
            .set('Origin', 'https://yourapp.com')
            .set('Access-Control-Request-Method', 'POST')
            .set('Access-Control-Request-Headers', 'Content-Type');
        
        expect(response.status).toBe(200);
        expect(response.headers['access-control-allow-origin']).toBe('https://yourapp.com');
        expect(response.headers['access-control-allow-methods']).toContain('POST');
    });
    
    test('should not allow wildcard origin with credentials', async () => {
        // Проверка, что такая конфигурация не используется
        const config = corsConfig[process.env.NODE_ENV];
        expect(config.origin).not.toEqual('*');
        expect(config.credentials).toBe(true);
    });
    
    test('should handle null origin safely', async () => {
        const response = await request(app)
            .get('/api/data')
            .set('Origin', 'null');
        
        // Зависит от конкретной реализации
        // В безопасной реализации null origin должен быть отклонен
        // или обработан особым образом
    });
    
    test('should expose only safe headers', async () => {
        const response = await request(app)
            .get('/api/data')
            .set('Origin', 'https://yourapp.com');
        
        const exposedHeaders = response.headers['access-control-expose-headers'];
        expect(exposedHeaders).not.toContain('authorization');
        expect(exposedHeaders).not.toContain('cookie');
    });
});
```

## Распространенные ошибки

### 1. Использование wildcard origin

```javascript
// ПЛОХО - универсальное разрешение
app.use(cors({
    origin: '*',  // Опасно!
    credentials: true  // Особенно опасно в сочетании с wildcard
}));

// ХОРОШО - явные домены
app.use(cors({
    origin: ['https://yourapp.com', 'https://www.yourapp.com'],
    credentials: true
}));
```

### 2. Неправильная обработка регулярных выражений

```javascript
// ПЛОХО - уязвимо для обхода
app.use(cors({
    origin: /\.yourapp\.com$/  // Может быть обойдено через evil.yourapp.com.evil.com
}));

// ХОРОШО - точное совпадение
app.use(cors({
    origin: ['https://yourapp.com', 'https://www.yourapp.com']
}));
```

### 3. Избыточные разрешенные заголовки

```javascript
// ПЛОХО - разрешение чувствительных заголовков
app.use(cors({
    allowedHeaders: ['Content-Type', 'Authorization', 'Cookie', 'X-Forwarded-For']
}));

// ХОРОШО - минимально необходимые заголовки
app.use(cors({
    allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## Связанные темы

- [[Лучшие-практики-безопасности-API]]
- [[Ограничение-скорости]]
- [[HTTP-Security-Headers]]
- [[Тестирование-безопасности]]
- [[Мониторинг-безопасности]]