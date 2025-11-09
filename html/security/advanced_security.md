# HTML Безопасность: Продвинутые возможности

Продвинутые возможности безопасности HTML включают в себя сложные методы защиты, современные подходы к предотвращению атак и интеграцию с различными веб-API для обеспечения максимальной безопасности веб-приложений.

## Content Security Policy (CSP) - продвинутые возможности

### CSP Nonce и Hash

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Продвинутый CSP с Nonce и Hash</title>
    
    <!-- CSP с использованием nonce -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123def456' 'strict-dynamic' https:;
                   style-src 'self' 'nonce-ghi789' 'unsafe-inline';
                   img-src 'self' data: blob: https:;
                   font-src 'self' https://fonts.gstatic.com;
                   connect-src 'self' https://api.example.com;
                   object-src 'none';
                   base-uri 'self';
                   form-action 'self';
                   frame-ancestors 'none';
                   upgrade-insecure-requests;">
</head>
<body>
    <h1>Продвинутый CSP</h1>
    
    <!-- Скрипт с nonce -->
    <script nonce="abc123def456">
        // Этот скрипт разрешен благодаря nonce
        console.log('Безопасный скрипт выполнен с nonce');
        
        // Используем строгую CSP политику
        document.addEventListener('DOMContentLoaded', function() {
            // Динамическое создание элементов
            const element = document.createElement('div');
            element.textContent = 'Динамически созданный контент';
            element.style.color = '#007acc';
            
            document.body.appendChild(element);
        });
    </script>
    
    <!-- Стили с nonce -->
    <style nonce="ghi789">
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }
        
        h1 {
            color: #007acc;
            text-align: center;
        }
        
        .content {
            max-width: 800px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
    </style>
    
    <div class="content">
        <p>Эта страница использует продвинутые возможности CSP для защиты от XSS.</p>
    </div>
</body>
</html>
```

### CSP Report Only и отчеты о нарушениях

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSP Report Only и отчеты</title>
    
    <!-- CSP в режиме отчета (не блокирует, только отслеживает) -->
    <meta http-equiv="Content-Security-Policy-Report-Only" 
          content="default-src 'self';
                   script-src 'self' 'unsafe-inline';
                   report-uri /csp-report;
                   report-to /csp-report-endpoint;">
</head>
<body>
    <h1>Отслеживание CSP нарушений</h1>
    
    <div class="content">
        <p>Эта страница отслеживает CSP нарушения без блокировки контента.</p>
        
        <!-- Встроенный скрипт для демонстрации отчета -->
        <script>
            console.log('Этот скрипт будет вызывать CSP нарушение в реальной политике');
            
            // Отслеживание нарушений CSP
            document.addEventListener('securitypolicyviolation', function(e) {
                console.warn('CSP Violation:', {
                    blockedURI: e.blockedURI,
                    violatedDirective: e.violatedDirective,
                    originalPolicy: e.originalPolicy
                });
                
                // Отправка отчета о нарушении
                sendCSPReport({
                    blockedURI: e.blockedURI,
                    violatedDirective: e.violatedDirective,
                    originalPolicy: e.originalPolicy,
                    sourceFile: e.sourceFile,
                    lineno: e.lineno,
                    colno: e.colno,
                    timestamp: new Date().toISOString()
                });
            });
            
            function sendCSPReport(report) {
                // Отправка отчета о нарушении CSP
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/csp-violation-report', JSON.stringify(report));
                } else {
                    fetch('/csp-violation-report', {
                        method: 'POST',
                        body: JSON.stringify(report),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
        </script>
    </div>
</body>
</html>
```

## Современные возможности безопасности

### Trusted Types API

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Trusted Types API</title>
    
    <!-- Включение Trusted Types -->
    <meta http-equiv="Content-Security-Policy" 
          content="require-trusted-types-for 'script';">
</head>
<body>
    <h1>Trusted Types API</h1>
    
    <div id="content-container">
        <p>Контент будет добавлен сюда безопасно</p>
    </div>
    
    <button id="inject-content">Добавить содержимое</button>

    <script>
        class TrustedTypesManager {
            constructor() {
                this.policy = null;
                this.contentContainer = document.getElementById('content-container');
                this.injectBtn = document.getElementById('inject-content');
                
                this.setupTrustedTypes();
                this.setupEventListeners();
            }
            
            setupTrustedTypes() {
                if (window.trustedTypes && window.trustedTypes.createPolicy) {
                    // Создаем политику Trusted Types
                    this.policy = window.trustedTypes.createPolicy('defaultPolicy', {
                        createHTML: (string) => {
                            // Санитизация HTML
                            const temp = document.createElement('div');
                            temp.textContent = string;
                            return temp.innerHTML;
                        },
                        createScript: (string) => {
                            // Проверка скрипта
                            if (this.isSafeScript(string)) {
                                return string;
                            }
                            console.warn('Небезопасный скрипт заблокирован');
                            return '';
                        }
                    });
                    
                    console.log('Trusted Types политика создана');
                } else {
                    console.warn('Trusted Types не поддерживается в этом браузере');
                }
            }
            
            isSafeScript(script) {
                // Проверка на потенциально опасные паттерны
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /eval\(/i,
                    /setTimeout\([^)]*['"]/i,
                    /setInterval\([^)]*['"]/i
                ];
                
                return !dangerousPatterns.some(pattern => pattern.test(script));
            }
            
            setupEventListeners() {
                this.injectBtn.addEventListener('click', () => {
                    this.injectTrustedContent();
                });
            }
            
            injectTrustedContent() {
                const unsafeContent = '<p>Опасный контент <script>alert("XSS!")</script></p>';
                const safeContent = '<p>Безопасный контент</p>';
                
                if (this.policy) {
                    // Используем безопасный способ вставки HTML
                    this.contentContainer.innerHTML = this.policy.createHTML(safeContent);
                    
                    // Попытка вставить опасный контент будет заблокирована
                    try {
                        // Это вызовет ошибку в браузере с включенным Trusted Types
                        // this.contentContainer.innerHTML = this.policy.createHTML(unsafeContent);
                    } catch (error) {
                        console.error('Trusted Types предотвратил вставку опасного контента:', error);
                    }
                } else {
                    // Резервный способ для браузеров без Trusted Types
                    this.contentContainer.innerHTML = this.sanitizeContent(safeContent);
                }
            }
            
            sanitizeContent(content) {
                // Резервная санитизация
                const temp = document.createElement('div');
                temp.textContent = content;
                return temp.innerHTML;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new TrustedTypesManager();
        });
    </script>
</body>
</html>
```

### Subresource Integrity (SRI) с различными форматами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Subresource Integrity - продвинутые возможности</title>
    
    <!-- CSS с SRI -->
    <link rel="stylesheet" 
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
          integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
          crossorigin="anonymous">
    
    <!-- Шрифты с SRI -->
    <link rel="stylesheet" 
          href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap"
          integrity="sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We"
          crossorigin="anonymous">
    
    <!-- JavaScript с SRI -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.6.0/dist/jquery.min.js"
            integrity="sha384-vtXRMe3mGCbOeY7l30aIg8H9p3GdeSe4IFlP6G8JMa7o7lXvnz3GFKzPxzJdPfGK"
            crossorigin="anonymous"></script>
</head>
<body>
    <h1>Subresource Integrity (SRI) - продвинутые возможности</h1>
    
    <div class="container">
        <p>Эта страница использует SRI для проверки целостности внешних ресурсов.</p>
        
        <!-- Динамическая загрузка ресурсов с SRI -->
        <button id="load-external-script">Загрузить внешний скрипт с SRI</button>
    </div>

    <script>
        class SRIManager {
            constructor() {
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('load-external-script').addEventListener('click', () => {
                    this.loadExternalScriptWithSRI();
                });
            }
            
            async loadExternalScriptWithSRI() {
                const scripts = [
                    {
                        src: 'https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js',
                        integrity: 'sha384-1UUG8jeO41IU3zRyH6yfW2RZ5+4rG9zQh7KU3p4c7XNQ11r7Dy+674rG75YXfQ+'
                    },
                    {
                        src: 'https://cdn.jsdelivr.net/npm/axios@0.27.2/dist/axios.min.js',
                        integrity: 'sha384-odGZc5CdIZ0XZhZV1s+3JpDcgTchW8N+j7eBY6VR1XGmR4KJ8QA4AcMqCcRd8M'
                    }
                ];
                
                for (const script of scripts) {
                    try {
                        await this.loadScriptWithSRI(script.src, script.integrity);
                        console.log('Скрипт успешно загружен с SRI:', script.src);
                    } catch (error) {
                        console.error('Ошибка загрузки скрипта с SRI:', script.src, error);
                    }
                }
            }
            
            loadScriptWithSRI(src, integrity) {
                return new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = src;
                    script.integrity = integrity;
                    script.crossOrigin = 'anonymous';
                    
                    script.onload = () => {
                        console.log('Скрипт загружен с SRI:', src);
                        resolve();
                    };
                    
                    script.onerror = () => {
                        console.error('Ошибка загрузки скрипта с SRI:', src);
                        reject(new Error(`Failed to load script with SRI: ${src}`));
                    };
                    
                    document.head.appendChild(script);
                });
            }
            
            loadStylesheetWithSRI(href, integrity) {
                return new Promise((resolve, reject) => {
                    const link = document.createElement('link');
                    link.rel = 'stylesheet';
                    link.href = href;
                    link.integrity = integrity;
                    link.crossOrigin = 'anonymous';
                    
                    link.onload = () => {
                        console.log('Стиль загружен с SRI:', href);
                        resolve();
                    };
                    
                    link.onerror = () => {
                        console.error('Ошибка загрузки стиля с SRI:', href);
                        reject(new Error(`Failed to load stylesheet with SRI: ${href}`));
                    };
                    
                    document.head.appendChild(link);
                });
            }
            
            // Генерация SRI хеша для локальных файлов (для демонстрации)
            generateSRIHash(content) {
                if (window.crypto && window.crypto.subtle) {
                    return crypto.subtle.digest('SHA-384', new TextEncoder().encode(content))
                        .then(hash => {
                            return 'sha384-' + btoa(String.fromCharCode(...new Uint8Array(hash)));
                        });
                } else {
                    // Резервный метод для старых браузеров
                    return new Promise(resolve => resolve('sha384-fallback-hash-not-available'));
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new SRIManager();
        });
    </script>
</body>
</html>
```

### Web Authentication API (WebAuthn)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Web Authentication API (WebAuthn)</title>
</head>
<body>
    <h1>Web Authentication API</h1>
    
    <div class="auth-container">
        <h2>Безопасная аутентификация</h2>
        
        <div class="auth-options">
            <button id="register-btn">Зарегистрировать устройство</button>
            <button id="login-btn">Войти с устройства</button>
        </div>
        
        <div id="auth-status" aria-live="polite"></div>
    </div>

    <script>
        class WebAuthnManager {
            constructor() {
                this.setupAuthButtons();
            }
            
            setupAuthButtons() {
                document.getElementById('register-btn').addEventListener('click', () => {
                    this.registerCredential();
                });
                
                document.getElementById('login-btn').addEventListener('click', () => {
                    this.loginWithCredential();
                });
            }
            
            async registerCredential() {
                if (!this.isWebAuthnSupported()) {
                    this.showAuthError('Web Authentication API не поддерживается в вашем браузере');
                    return;
                }
                
                try {
                    this.showAuthStatus('Начинаем регистрацию...');
                    
                    // Параметры регистрации
                    const challenge = this.generateChallenge();
                    const userId = this.generateUserId();
                    
                    const publicKey = {
                        challenge: challenge,
                        rp: {
                            name: 'Пример сайта',
                            id: window.location.hostname
                        },
                        user: {
                            id: userId,
                            name: 'user@example.com',
                            displayName: 'Имя пользователя'
                        },
                        pubKeyCredParams: [
                            { alg: -7, type: 'public-key' } // ES256
                        ],
                        timeout: 60000,
                        attestation: 'direct',
                        authenticatorSelection: {
                            authenticatorAttachment: 'platform',
                            requireResidentKey: false,
                            userVerification: 'preferred'
                        }
                    };
                    
                    const credential = await navigator.credentials.create({
                        publicKey: publicKey
                    });
                    
                    this.showAuthSuccess('Устройство успешно зарегистрировано');
                    console.log('Регистрация завершена:', credential);
                    
                    // Отправка данных на сервер для сохранения
                    await this.sendRegistrationData(credential);
                    
                } catch (error) {
                    this.showAuthError(`Ошибка регистрации: ${error.message}`);
                    console.error('WebAuthn регистрация ошибка:', error);
                }
            }
            
            async loginWithCredential() {
                if (!this.isWebAuthnSupported()) {
                    this.showAuthError('Web Authentication API не поддерживается в вашем браузере');
                    return;
                }
                
                try {
                    this.showAuthStatus('Начинаем аутентификацию...');
                    
                    // Параметры аутентификации
                    const challenge = this.generateChallenge();
                    
                    const publicKey = {
                        challenge: challenge,
                        timeout: 60000,
                        rpId: window.location.hostname,
                        allowCredentials: [], // В реальном приложении - список сохраненных cred
                        userVerification: 'preferred'
                    };
                    
                    const assertion = await navigator.credentials.get({
                        publicKey: publicKey
                    });
                    
                    this.showAuthSuccess('Успешная аутентификация');
                    console.log('Аутентификация завершена:', assertion);
                    
                    // Отправка данных на сервер для проверки
                    await this.sendAuthenticationData(assertion);
                    
                } catch (error) {
                    this.showAuthError(`Ошибка аутентификации: ${error.message}`);
                    console.error('WebAuthn аутентификация ошибка:', error);
                }
            }
            
            isWebAuthnSupported() {
                return 'credentials' in navigator && 
                       'create' in navigator.credentials && 
                       'get' in navigator.credentials;
            }
            
            generateChallenge() {
                // Генерация случайного challenge
                const array = new Uint8Array(32);
                crypto.getRandomValues(array);
                return array;
            }
            
            generateUserId() {
                // Генерация ID пользователя
                const array = new Uint8Array(16);
                crypto.getRandomValues(array);
                return array;
            }
            
            async sendRegistrationData(credential) {
                try {
                    const response = await fetch('/api/register-credential', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({
                            id: credential.id,
                            rawId: Array.from(new Uint8Array(credential.rawId)),
                            type: credential.type,
                            response: {
                                attestationObject: Array.from(new Uint8Array(credential.response.attestationObject)),
                                clientDataJSON: Array.from(new Uint8Array(credential.response.clientDataJSON))
                            }
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error('Ошибка сервера при регистрации');
                    }
                    
                    const result = await response.json();
                    console.log('Данные регистрации отправлены:', result);
                } catch (error) {
                    console.error('Ошибка отправки данных регистрации:', error);
                }
            }
            
            async sendAuthenticationData(assertion) {
                try {
                    const response = await fetch('/api/authenticate', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({
                            id: assertion.id,
                            rawId: Array.from(new Uint8Array(assertion.rawId)),
                            type: assertion.type,
                            response: {
                                authenticatorData: Array.from(new Uint8Array(assertion.response.authenticatorData)),
                                clientDataJSON: Array.from(new Uint8Array(assertion.response.clientDataJSON)),
                                signature: Array.from(new Uint8Array(assertion.response.signature)),
                                userHandle: assertion.response.userHandle ? 
                                           Array.from(new Uint8Array(assertion.response.userHandle)) : null
                            }
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error('Ошибка сервера при аутентификации');
                    }
                    
                    const result = await response.json();
                    console.log('Данные аутентификации отправлены:', result);
                } catch (error) {
                    console.error('Ошибка отправки данных аутентификации:', error);
                }
            }
            
            showAuthStatus(message) {
                const statusElement = document.getElementById('auth-status');
                statusElement.innerHTML = `<div class="status-info">${message}</div>`;
                statusElement.setAttribute('role', 'status');
                statusElement.setAttribute('aria-live', 'polite');
            }
            
            showAuthSuccess(message) {
                const statusElement = document.getElementById('auth-status');
                statusElement.innerHTML = `<div class="status-success">✓ ${message}</div>`;
                statusElement.setAttribute('role', 'status');
                statusElement.setAttribute('aria-live', 'polite');
            }
            
            showAuthError(message) {
                const statusElement = document.getElementById('auth-status');
                statusElement.innerHTML = `<div class="status-error">✗ ${message}</div>`;
                statusElement.setAttribute('role', 'alert');
                statusElement.setAttribute('aria-live', 'assertive');
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new WebAuthnManager();
        });
    </script>
    
    <style>
        .auth-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
        .auth-options {
            margin: 20px 0;
        }
        
        .auth-options button {
            padding: 10px 20px;
            margin-right: 10px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .status-info {
            padding: 10px;
            background-color: #e3f2fd;
            color: #0d47a1;
            border-radius: 4px;
        }
        
        .status-success {
            padding: 10px;
            background-color: #e8f5e8;
            color: #1b5e20;
            border-radius: 4px;
        }
        
        .status-error {
            padding: 10px;
            background-color: #ffebee;
            color: #b71c1c;
            border-radius: 4px;
        }
    </style>
</body>
</html>
```

### Безопасная работа с WebSockets

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная работа с WebSockets</title>
</head>
<body>
    <h1>Безопасная работа с WebSockets</h1>
    
    <div class="websocket-security">
        <div class="connection-controls">
            <button id="connect-btn">Подключиться</button>
            <button id="disconnect-btn" disabled>Отключиться</button>
            <span id="connection-status">Отключено</span>
        </div>
        
        <div class="message-controls">
            <input type="text" id="message-input" placeholder="Введите сообщение..." disabled>
            <button id="send-btn" disabled>Отправить</button>
        </div>
        
        <div id="messages-container" class="messages-container"></div>
    </div>

    <script>
        class SecureWebSocketManager {
            constructor(url) {
                this.url = url;
                this.ws = null;
                this.isConnected = false;
                this.connectionAttempts = 0;
                this.maxConnectionAttempts = 3;
                this.retryInterval = 3000;
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('connect-btn').addEventListener('click', () => {
                    this.connectSecurely();
                });
                
                document.getElementById('disconnect-btn').addEventListener('click', () => {
                    this.disconnectSecurely();
                });
                
                document.getElementById('send-btn').addEventListener('click', () => {
                    this.sendMessageSecurely();
                });
                
                document.getElementById('message-input').addEventListener('keypress', (e) => {
                    if (e.key === 'Enter' && !document.getElementById('message-input').disabled) {
                        this.sendMessageSecurely();
                    }
                });
            }
            
            connectSecurely() {
                if (this.connectionAttempts >= this.maxConnectionAttempts) {
                    this.showSecurityError('Превышено максимальное количество попыток подключения');
                    return;
                }
                
                try {
                    // Проверка безопасности URL
                    if (!this.isValidSecureWebSocketURL(this.url)) {
                        throw new Error('Небезопасный WebSocket URL');
                    }
                    
                    this.ws = new WebSocket(this.url);
                    
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.connectionAttempts = 0;
                        this.updateConnectionStatus('Подключено', 'success');
                        this.enableInputs();
                    };
                    
                    this.ws.onmessage = (event) => {
                        this.handleSecureMessage(event.data);
                    };
                    
                    this.ws.onclose = (event) => {
                        this.isConnected = false;
                        this.updateConnectionStatus('Отключено', 'warning');
                        this.disableInputs();
                        
                        // Автоматическое переподключение
                        if (event.code !== 1000) { // Нормальное закрытие
                            setTimeout(() => {
                                if (this.connectionAttempts < this.maxConnectionAttempts) {
                                    this.connectionAttempts++;
                                    this.connectSecurely();
                                }
                            }, this.retryInterval);
                        }
                    };
                    
                    this.ws.onerror = (error) => {
                        console.error('WebSocket ошибка:', error);
                        this.updateConnectionStatus('Ошибка', 'error');
                        this.disableInputs();
                    };
                    
                } catch (error) {
                    console.error('Ошибка подключения к WebSocket:', error);
                    this.showSecurityError(`Ошибка подключения: ${error.message}`);
                }
            }
            
            isValidSecureWebSocketURL(url) {
                try {
                    const parsedUrl = new URL(url);
                    
                    // Проверка протокола (только wss для безопасности)
                    if (parsedUrl.protocol !== 'wss:' && !this.isLocalhost(url)) {
                        console.warn('Используйте wss:// для безопасного WebSocket соединения');
                        return false;
                    }
                    
                    // Проверка домена
                    const allowedDomains = [
                        window.location.hostname,
                        'api.example.com',
                        'secure-chat.example.com'
                    ];
                    
                    if (!this.isLocalhost(url)) {
                        return allowedDomains.includes(parsedUrl.hostname);
                    }
                    
                    return true;
                } catch (error) {
                    console.error('Ошибка проверки URL:', error);
                    return false;
                }
            }
            
            isLocalhost(url) {
                try {
                    const parsedUrl = new URL(url);
                    return ['localhost', '127.0.0.1', '[::1]'].includes(parsedUrl.hostname);
                } catch {
                    return false;
                }
            }
            
            handleSecureMessage(data) {
                try {
                    // Проверка целостности сообщения
                    const message = JSON.parse(data);
                    
                    if (!this.validateMessageStructure(message)) {
                        console.error('Невалидная структура сообщения:', message);
                        return;
                    }
                    
                    // Санитизация содержимого сообщения
                    const sanitizedMessage = this.sanitizeMessageContent(message);
                    
                    this.displayMessage(sanitizedMessage);
                    
                } catch (error) {
                    console.error('Ошибка обработки сообщения:', error);
                    this.showSecurityError('Получено недопустимое сообщение');
                }
            }
            
            validateMessageStructure(message) {
                // Проверка обязательных полей
                const requiredFields = ['id', 'timestamp', 'content', 'sender'];
                return requiredFields.every(field => field in message);
            }
            
            sanitizeMessageContent(message) {
                // Санитизация содержимого сообщения
                return {
                    ...message,
                    content: this.sanitizeInput(message.content),
                    sender: this.sanitizeInput(message.sender || 'Unknown')
                };
            }
            
            sendMessageSecurely() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;
                
                const content = this.messageInput.value.trim();
                
                // Проверка безопасности перед отправкой
                if (!this.isMessageContentSafe(content)) {
                    this.showSecurityError('Сообщение содержит недопустимый контент');
                    return;
                }
                
                const message = {
                    content: content,
                    timestamp: new Date().toISOString(),
                    userId: this.getCurrentUserId(),
                    sessionId: this.getSessionId()
                };
                
                try {
                    this.ws.send(JSON.stringify(message));
                    
                    // Отображение собственного сообщения
                    this.displayMessage({ ...message, isOwn: true });
                    
                    this.messageInput.value = '';
                    
                } catch (error) {
                    console.error('Ошибка отправки сообщения:', error);
                    this.showSecurityError('Ошибка отправки сообщения');
                }
            }
            
            isMessageContentSafe(content) {
                // Проверка на потенциально опасный контент
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return !dangerousPatterns.some(pattern => pattern.test(content));
            }
            
            displayMessage(message) {
                const messagesContainer = document.getElementById('messages-container');
                const messageElement = document.createElement('div');
                messageElement.className = `message ${message.isOwn ? 'own' : 'other'}`;
                
                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageElement.innerHTML = `
                    <div class="message-header">
                        <span class="message-sender">${this.escapeHTML(message.sender || message.userId)}</span>
                        <span class="message-time">${time}</span>
                    </div>
                    <div class="message-content">${this.escapeHTML(message.content)}</div>
                `;
                
                messagesContainer.appendChild(messageElement);
                
                // Прокрутка к последнему сообщению
                messagesContainer.scrollTop = messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus(status, type) {
                const statusElement = document.getElementById('connection-status');
                statusElement.textContent = status;
                statusElement.className = `connection-status ${type}`;
            }
            
            enableInputs() {
                document.getElementById('message-input').disabled = false;
                document.getElementById('send-btn').disabled = false;
                document.getElementById('connect-btn').disabled = true;
                document.getElementById('disconnect-btn').disabled = false;
            }
            
            disableInputs() {
                document.getElementById('message-input').disabled = true;
                document.getElementById('send-btn').disabled = true;
                document.getElementById('connect-btn').disabled = false;
                document.getElementById('disconnect-btn').disabled = true;
            }
            
            disconnectSecurely() {
                if (this.ws) {
                    this.ws.close(1000, 'Нормальное закрытие');
                }
            }
            
            showSecurityError(message) {
                const messagesContainer = document.getElementById('messages-container');
                const errorDiv = document.createElement('div');
                errorDiv.className = 'security-error';
                errorDiv.textContent = message;
                
                messagesContainer.prepend(errorDiv);
                
                // Автоматически удаляем ошибку через 5 секунд
                setTimeout(() => {
                    if (messagesContainer.contains(errorDiv)) {
                        messagesContainer.removeChild(errorDiv);
                    }
                }, 5000);
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .trim();
            }
            
            escapeHTML(str) {
                if (typeof str !== 'string') return str;
                
                return str
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;');
            }
            
            getCurrentUserId() {
                // В реальном приложении ID пользователя должен быть получен из сессии
                return localStorage.getItem('userId') || 'anonymous';
            }
            
            getSessionId() {
                // В реальном приложении ID сессии должен быть получен из сессии
                return sessionStorage.getItem('sessionId') || 'session_' + Date.now();
            }
        }
        
        // Инициализация (в реальном приложении использовать реальный WebSocket URL)
        // const wsManager = new SecureWebSocketManager('wss://secure-chat.example.com/ws');
        
        // Для демонстрации создаем mock-менеджер
        class MockWebSocketManager {
            constructor(url) {
                this.url = url;
                this.isConnected = false;
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('connect-btn').addEventListener('click', () => {
                    this.mockConnect();
                });
                
                document.getElementById('disconnect-btn').addEventListener('click', () => {
                    this.mockDisconnect();
                });
                
                document.getElementById('send-btn').addEventListener('click', () => {
                    this.mockSendMessage();
                });
            }
            
            mockConnect() {
                this.isConnected = true;
                this.updateConnectionStatus('Подключено (Mock)', 'success');
                this.enableInputs();
                
                // Имитация получения сообщений
                setInterval(() => {
                    if (this.isConnected && Math.random() > 0.8) {
                        const users = ['Алексей', 'Мария', 'Иван', 'Елена', 'Дмитрий'];
                        const messages = [
                            'Привет всем!', 'Как дела?', 'Работаю над проектом', 'Кто на связи?',
                            'Нужна помощь?', 'Все прошло отлично!', 'Планирую встречу', 'Обновил документацию'
                        ];
                        
                        this.mockReceiveMessage({
                            sender: users[Math.floor(Math.random() * users.length)],
                            content: messages[Math.floor(Math.random() * messages.length)],
                            timestamp: new Date().toISOString()
                        });
                    }
                }, 3000);
            }
            
            mockDisconnect() {
                this.isConnected = false;
                this.updateConnectionStatus('Отключено', 'warning');
                this.disableInputs();
            }
            
            mockSendMessage() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;
                
                const content = this.messageInput.value.trim();
                this.mockReceiveMessage({
                    sender: 'Вы',
                    content: content,
                    timestamp: new Date().toISOString(),
                    isOwn: true
                });
                
                this.messageInput.value = '';
            }
            
            mockReceiveMessage(message) {
                const messagesContainer = document.getElementById('messages-container');
                const messageElement = document.createElement('div');
                messageElement.className = `message ${message.isOwn ? 'own' : 'other'}`;
                
                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageElement.innerHTML = `
                    <div class="message-header">
                        <span class="message-sender">${message.sender}</span>
                        <span class="message-time">${time}</span>
                    </div>
                    <div class="message-content">${message.content}</div>
                `;
                
                messagesContainer.appendChild(messageElement);
                messagesContainer.scrollTop = messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus(status, type) {
                document.getElementById('connection-status').textContent = status;
                document.getElementById('connection-status').className = `connection-status ${type}`;
            }
            
            enableInputs() {
                document.getElementById('message-input').disabled = false;
                document.getElementById('send-btn').disabled = false;
                document.getElementById('connect-btn').disabled = true;
                document.getElementById('disconnect-btn').disabled = false;
            }
            
            disableInputs() {
                document.getElementById('message-input').disabled = true;
                document.getElementById('send-btn').disabled = true;
                document.getElementById('connect-btn').disabled = false;
                document.getElementById('disconnect-btn').disabled = true;
            }
        }
        
        // Инициализация mock-версии
        document.addEventListener('DOMContentLoaded', () => {
            new MockWebSocketManager('mock://localhost:8080/ws');
        });
    </script>
    
    <style>
        .websocket-security {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .connection-controls, .message-controls {
            margin-bottom: 20px;
        }
        
        .connection-controls button, .message-controls button {
            padding: 10px 20px;
            margin-right: 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        #connect-btn {
            background-color: #28a745;
            color: white;
        }
        
        #disconnect-btn {
            background-color: #dc3545;
            color: white;
        }
        
        #send-btn {
            background-color: #007acc;
            color: white;
        }
        
        .connection-status {
            padding: 10px 15px;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .connection-status.success {
            background-color: #d4edda;
            color: #155724;
        }
        
        .connection-status.warning {
            background-color: #fff3cd;
            color: #856404;
        }
        
        .connection-status.error {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        #message-input {
            width: calc(100% - 120px);
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .messages-container {
            height: 400px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            overflow-y: auto;
            background-color: #f9f9f9;
        }
        
        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            max-width: 80%;
        }
        
        .message.own {
            background-color: #e3f2fd;
            margin-left: auto;
            text-align: right;
        }
        
        .message.other {
            background-color: #f1f8e9;
        }
        
        .message-header {
            display: flex;
            justify-content: space-between;
            font-size: 0.8em;
            margin-bottom: 5px;
        }
        
        .message-sender {
            font-weight: bold;
        }
        
        .message-time {
            color: #666;
        }
        
        .message-content {
            margin: 0;
            line-height: 1.5;
        }
        
        .security-error {
            background-color: #f8d7da;
            color: #721c24;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
            text-align: center;
        }
    </style>
</body>
</html>
```

### Безопасная интеграция с IndexedDB

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная интеграция с IndexedDB</title>
</head>
<body>
    <h1>Безопасная интеграция с IndexedDB</h1>
    
    <div class="secure-db-container">
        <div class="form-container">
            <h2>Добавить защищенное сообщение</h2>
            <form id="secure-message-form">
                <div class="form-group">
                    <label for="message-title">Заголовок:</label>
                    <input type="text" 
                           id="message-title" 
                           name="title" 
                           required
                           aria-describedby="title-help">
                    <div id="title-help" class="help-text">Заголовок сообщения</div>
                </div>
                
                <div class="form-group">
                    <label for="message-content">Содержимое:</label>
                    <textarea id="message-content" 
                              name="content" 
                              rows="4" 
                              required
                              aria-describedby="content-help"></textarea>
                    <div id="content-help" class="help-text">Содержимое сообщения</div>
                </div>
                
                <div class="form-group">
                    <label for="message-tags">Теги:</label>
                    <input type="text" 
                           id="message-tags" 
                           name="tags" 
                           placeholder="tag1, tag2, tag3"
                           aria-describedby="tags-help">
                    <div id="tags-help" class="help-text">Теги через запятую</div>
                </div>
                
                <button type="submit">Сохранить сообщение</button>
            </form>
        </div>
        
        <div class="messages-container">
            <h2>Сохраненные сообщения</h2>
            <div class="filters">
                <input type="text" id="search-messages" placeholder="Поиск сообщений...">
                <select id="filter-by-tag">
                    <option value="">Все теги</option>
                </select>
            </div>
            <div id="messages-list"></div>
        </div>
    </div>

    <script>
        class SecureIndexedDBManager {
            constructor(dbName = 'SecureMessagesDB', version = 1) {
                this.dbName = dbName;
                this.version = version;
                this.db = null;
                
                this.setupSecureDB();
            }
            
            async setupSecureDB() {
                try {
                    const request = indexedDB.open(this.dbName, this.version);
                    
                    request.onerror = (event) => {
                        console.error('Ошибка открытия базы данных:', event.target.error);
                    };
                    
                    request.onsuccess = (event) => {
                        this.db = event.target.result;
                        console.log('Безопасная база данных открыта');
                        this.loadMessages();
                    };
                    
                    request.onupgradeneeded = (event) => {
                        this.db = event.target.result;
                        
                        // Создаем безопасное хранилище
                        if (!this.db.objectStoreNames.contains('messages')) {
                            const store = this.db.createObjectStore('messages', { 
                                keyPath: 'id', 
                                autoIncrement: true 
                            });
                            
                            // Создаем индексы для безопасных операций поиска
                            store.createIndex('title', 'title', { unique: false });
                            store.createIndex('timestamp', 'timestamp', { unique: false });
                            store.createIndex('tags', 'tags', { unique: false });
                            
                            // Защита от небезопасных операций
                            store.transaction.oncomplete = () => {
                                console.log('Объектное хранилище создано безопасно');
                            };
                        }
                    };
                } catch (error) {
                    console.error('Ошибка настройки безопасной базы данных:', error);
                }
            }
            
            async addSecureMessage(messageData) {
                if (!this.db) {
                    await this.setupSecureDB();
                }
                
                // Санитизация данных перед сохранением
                const sanitizedData = {
                    ...messageData,
                    title: this.sanitizeInput(messageData.title),
                    content: this.sanitizeInput(messageData.content),
                    tags: messageData.tags ? messageData.tags.split(',').map(tag => this.sanitizeInput(tag.trim())) : [],
                    timestamp: new Date().toISOString(),
                    created: Date.now()
                };
                
                // Проверка на безопасность
                if (!this.validateSecureData(sanitizedData)) {
                    throw new Error('Данные содержат потенциально опасный контент');
                }
                
                const transaction = this.db.transaction(['messages'], 'readwrite');
                const store = transaction.objectStore('messages');
                
                return new Promise((resolve, reject) => {
                    const request = store.add(sanitizedData);
                    
                    request.onsuccess = () => {
                        console.log('Безопасное сообщение добавлено с ID:', request.result);
                        this.loadMessages(); // Обновляем список
                        resolve(request.result);
                    };
                    
                    request.onerror = (event) => {
                        console.error('Ошибка добавления сообщения:', event.target.error);
                        reject(event.target.error);
                    };
                });
            }
            
            async getSecureMessages(filter = {}) {
                if (!this.db) {
                    await this.setupSecureDB();
                }
                
                const transaction = this.db.transaction(['messages'], 'readonly');
                const store = transaction.objectStore('messages');
                
                return new Promise((resolve, reject) => {
                    const request = store.getAll();
                    
                    request.onsuccess = () => {
                        let messages = request.result;
                        
                        // Фильтрация безопасных данных
                        if (filter.search) {
                            messages = messages.filter(msg => 
                                msg.title.toLowerCase().includes(filter.search.toLowerCase()) ||
                                msg.content.toLowerCase().includes(filter.search.toLowerCase())
                            );
                        }
                        
                        if (filter.tag) {
                            messages = messages.filter(msg => 
                                msg.tags && msg.tags.includes(filter.tag)
                            );
                        }
                        
                        // Сортировка по времени создания (новые первыми)
                        messages.sort((a, b) => b.created - a.created);
                        
                        resolve(messages);
                    };
                    
                    request.onerror = (event) => {
                        console.error('Ошибка получения сообщений:', event.target.error);
                        reject(event.target.error);
                    };
                });
            }
            
            async updateSecureMessage(id, updates) {
                if (!this.db) {
                    await this.setupSecureDB();
                }
                
                const transaction = this.db.transaction(['messages'], 'readwrite');
                const store = transaction.objectStore('messages');
                
                const request = store.get(id);
                
                request.onsuccess = () => {
                    const message = request.result;
                    
                    // Обновляем только разрешенные поля
                    const allowedFields = ['title', 'content', 'tags'];
                    for (const [key, value] of Object.entries(updates)) {
                        if (allowedFields.includes(key)) {
                            message[key] = this.sanitizeInput(value);
                        }
                    }
                    
                    // Проверка безопасности
                    if (!this.validateSecureData(message)) {
                        throw new Error('Обновленные данные содержат потенциально опасный контент');
                    }
                    
                    const updateRequest = store.put(message);
                    updateRequest.onsuccess = () => {
                        console.log('Сообщение обновлено безопасно');
                        this.loadMessages(); // Обновляем список
                    };
                    updateRequest.onerror = (event) => {
                        console.error('Ошибка обновления сообщения:', event.target.error);
                    };
                };
                
                request.onerror = (event) => {
                    console.error('Ошибка получения сообщения для обновления:', event.target.error);
                };
            }
            
            async deleteSecureMessage(id) {
                if (!this.db) {
                    await this.setupSecureDB();
                }
                
                const transaction = this.db.transaction(['messages'], 'readwrite');
                const store = transaction.objectStore('messages');
                
                return new Promise((resolve, reject) => {
                    const request = store.delete(id);
                    
                    request.onsuccess = () => {
                        console.log('Сообщение удалено безопасно');
                        this.loadMessages(); // Обновляем список
                        resolve();
                    };
                    
                    request.onerror = (event) => {
                        console.error('Ошибка удаления сообщения:', event.target.error);
                        reject(event.target.error);
                    };
                });
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                // Комплексная санитизация
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .replace(/javascript:/gi, '') // Удаление опасных протоколов
                    .replace(/on\w+\s*=/gi, '') // Удаление событий
                    .trim();
            }
            
            validateSecureData(data) {
                // Проверка на опасный контент
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                const dataString = JSON.stringify(data);
                return !dangerousPatterns.some(pattern => pattern.test(dataString));
            }
            
            async loadMessages() {
                try {
                    const messages = await this.getSecureMessages();
                    this.displayMessages(messages);
                    this.updateTagFilters(messages);
                } catch (error) {
                    console.error('Ошибка загрузки сообщений:', error);
                    document.getElementById('messages-list').innerHTML = '<p>Ошибка загрузки сообщений</p>';
                }
            }
            
            displayMessages(messages) {
                const container = document.getElementById('messages-list');
                
                if (messages.length === 0) {
                    container.innerHTML = '<p>Нет сохраненных сообщений</p>';
                    return;
                }
                
                container.innerHTML = messages.map(message => `
                    <div class="message-card" data-id="${message.id}">
                        <div class="message-header">
                            <h3>${this.escapeHTML(message.title)}</h3>
                            <time datetime="${message.timestamp}">${new Date(message.timestamp).toLocaleString('ru-RU')}</time>
                        </div>
                        <div class="message-content">${this.escapeHTML(message.content)}</div>
                        <div class="message-tags">
                            ${message.tags ? message.tags.map(tag => `
                                <span class="tag">${this.escapeHTML(tag)}</span>
                            `).join('') : ''}
                        </div>
                        <div class="message-actions">
                            <button onclick="secureDB.editMessage(${message.id})">Редактировать</button>
                            <button onclick="secureDB.deleteMessage(${message.id})">Удалить</button>
                        </div>
                    </div>
                `).join('');
            }
            
            updateTagFilters(messages) {
                const tagFilter = document.getElementById('filter-by-tag');
                const allTags = new Set();
                
                messages.forEach(message => {
                    if (message.tags) {
                        message.tags.forEach(tag => allTags.add(tag));
                    }
                });
                
                // Очищаем текущие опции
                tagFilter.innerHTML = '<option value="">Все теги</option>';
                
                // Добавляем новые опции
                allTags.forEach(tag => {
                    const option = document.createElement('option');
                    option.value = tag;
                    option.textContent = tag;
                    tagFilter.appendChild(option);
                });
            }
            
            escapeHTML(str) {
                if (typeof str !== 'string') return str;
                
                return str
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;');
            }
            
            async editMessage(id) {
                // В реальном приложении здесь будет форма редактирования
                const messages = await this.getSecureMessages();
                const message = messages.find(msg => msg.id === id);
                
                if (message) {
                    document.getElementById('message-title').value = message.title;
                    document.getElementById('message-content').value = message.content;
                    document.getElementById('message-tags').value = message.tags ? message.tags.join(', ') : '';
                    
                    // Добавляем скрытое поле для ID
                    let idField = document.querySelector('#secure-message-form input[name="edit-id"]');
                    if (!idField) {
                        idField = document.createElement('input');
                        idField.type = 'hidden';
                        idField.name = 'edit-id';
                        idField.value = id;
                        document.getElementById('secure-message-form').appendChild(idField);
                    } else {
                        idField.value = id;
                    }
                    
                    // Изменяем текст кнопки
                    document.querySelector('#secure-message-form button[type="submit"]').textContent = 'Обновить сообщение';
                }
            }
            
            async deleteMessage(id) {
                if (confirm('Удалить это сообщение?')) {
                    try {
                        await this.deleteSecureMessage(id);
                        console.log('Сообщение удалено');
                    } catch (error) {
                        console.error('Ошибка удаления сообщения:', error);
                        alert('Ошибка при удалении сообщения');
                    }
                }
            }
        }
        
        class SecureMessageApp {
            constructor() {
                this.db = new SecureIndexedDBManager();
                this.form = document.getElementById('secure-message-form');
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.handleFormSubmit();
                });
                
                document.getElementById('search-messages').addEventListener('input', (e) => {
                    this.searchMessages(e.target.value);
                });
                
                document.getElementById('filter-by-tag').addEventListener('change', (e) => {
                    this.filterByTag(e.target.value);
                });
            }
            
            async handleFormSubmit() {
                const title = document.getElementById('message-title').value.trim();
                const content = document.getElementById('message-content').value.trim();
                const tagsInput = document.getElementById('message-tags').value.trim();
                
                if (!title || !content) {
                    alert('Пожалуйста, заполните все обязательные поля');
                    return;
                }
                
                const tags = tagsInput ? tagsInput.split(',').map(tag => tag.trim()) : [];
                
                const messageData = {
                    title: title,
                    content: content,
                    tags: tags
                };
                
                const editId = this.form.querySelector('input[name="edit-id"]');
                
                try {
                    if (editId && editId.value) {
                        // Обновление сообщения
                        await this.db.updateSecureMessage(parseInt(editId.value), messageData);
                        editId.remove(); // Удаляем поле редактирования
                        this.form.querySelector('button[type="submit"]').textContent = 'Сохранить сообщение';
                    } else {
                        // Добавление нового сообщения
                        await this.db.addSecureMessage(messageData);
                    }
                    
                    // Очищаем форму
                    this.form.reset();
                    
                } catch (error) {
                    console.error('Ошибка обработки сообщения:', error);
                    alert('Ошибка при сохранении сообщения: ' + error.message);
                }
            }
            
            async searchMessages(query) {
                const messages = await this.db.getSecureMessages({ search: query });
                this.db.displayMessages(messages);
            }
            
            async filterByTag(tag) {
                const messages = await this.db.getSecureMessages({ tag: tag });
                this.db.displayMessages(messages);
            }
        }
        
        // Инициализация приложения
        document.addEventListener('DOMContentLoaded', () => {
            new SecureMessageApp();
        });
    </script>
    
    <style>
        .secure-db-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            display: grid;
            grid-template-columns: 1fr 2fr;
            gap: 30px;
        }
        
        .form-container {
            background-color: #f8f9fa;
            padding: 20px;
            border-radius: 8px;
        }
        
        .messages-container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        textarea {
            resize: vertical;
            min-height: 100px;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .filters {
            display: flex;
            gap: 15px;
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 1px solid #eee;
        }
        
        .filters input, .filters select {
            flex: 1;
        }
        
        .message-card {
            border: 1px solid #eee;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            background-color: #fafafa;
        }
        
        .message-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        
        .message-header h3 {
            margin: 0;
            color: #333;
        }
        
        .message-content {
            margin-bottom: 10px;
            line-height: 1.5;
        }
        
        .message-tags {
            margin-bottom: 10px;
        }
        
        .tag {
            display: inline-block;
            background-color: #e3f2fd;
            color: #0d47a1;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 0.8em;
            margin-right: 5px;
        }
        
        .message-actions {
            text-align: right;
        }
        
        .message-actions button {
            padding: 5px 10px;
            margin-left: 5px;
            font-size: 0.9em;
        }
        
        .message-actions button:last-child {
            background-color: #dc3545;
        }
    </style>
</body>
</html>
```

## Проверка и тестирование безопасности

### Инструменты для проверки безопасности:
1. **OWASP ZAP** - автоматизированное сканирование уязвимостей
2. **Snyk** - проверка уязвимостей в зависимостях
3. **npm audit** - проверка безопасности npm пакетов
4. **Chrome DevTools Security Panel** - проверка безопасности
5. **Mozilla Observatory** - анализ конфигурации безопасности сайта
6. **Security Headers** - проверка HTTP заголовков безопасности
7. **CSP Evaluator** - проверка Content Security Policy
8. **Retire.js** - поиск уязвимых JavaScript библиотек
9. **Lighthouse** - комплексная проверка безопасности и доступности
10. **axe-core** - проверка безопасности в контексте доступности

### Проверка XSS уязвимостей:
1. Проверка всех мест вывода пользовательского контента
2. Проверка использования innerHTML вместо textContent
3. Проверка экранирования специальных символов
4. Проверка использования безопасных методов вставки HTML
5. Проверка на наличие вредоносных скриптов
6. Проверка санитизации данных
7. Проверка CSP политики
8. Проверка SRI для внешних ресурсов

### Проверка CSRF защиты:
1. Проверка наличия CSRF токенов
2. Проверка валидации токенов на сервере
3. Проверка использования SameSite cookie атрибутов
4. Проверка заголовков Origin и Referer
5. Проверка использования double-submit cookie паттерна
6. Проверка использования encrypted token паттерна

### Проверка clickjacking защиты:
1. Проверка CSP frame-ancestors директивы
2. Проверка X-Frame-Options заголовка
3. Проверка возможности встраивания страниц в iframe
4. Проверка frame-busting скриптов

## Лучшие практики безопасности HTML

### 1. Санитизация пользовательского ввода
```html
<script>
class InputSanitizer {
    static sanitizeHTML(input) {
        if (typeof input !== 'string') return input;
        
        // Экранирование HTML
        return input
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#x27;')
            .replace(/\//g, '&#x2F;')
            .trim();
    }
    
    static sanitizeURL(url) {
        try {
            const parsedUrl = new URL(url, window.location.origin);
            
            // Проверка протокола
            const allowedProtocols = ['http:', 'https:', 'mailto:', 'tel:'];
            if (!allowedProtocols.includes(parsedUrl.protocol)) {
                throw new Error('Неподдерживаемый протокол');
            }
            
            return parsedUrl.href;
        } catch (error) {
            console.warn('Небезопасный URL:', url);
            return '#';
        }
    }
    
    static sanitizeCSS(cssText) {
        // Блокируем опасные CSS свойства
        const dangerousCSS = [
            /expression/i,
            /javascript:/i,
            /url\(\s*['"]?\s*javascript:/i,
            /behavior:/i
        ];
        
        if (dangerousCSS.some(pattern => pattern.test(cssText))) {
            throw new Error('Обнаружены потенциально опасные CSS свойства');
        }
        
        return cssText;
    }
    
    static deepSanitize(obj) {
        if (obj === null || typeof obj !== 'object') {
            return this.sanitizeValue(obj);
        }
        
        if (Array.isArray(obj)) {
            return obj.map(item => this.deepSanitize(item));
        }
        
        const sanitized = {};
        for (const [key, value] of Object.entries(obj)) {
            sanitized[key] = this.deepSanitize(value);
        }
        return sanitized;
    }
    
    static sanitizeValue(value) {
        if (typeof value === 'string') {
            return this.sanitizeHTML(value);
        }
        return value;
    }
}
</script>
```

### 2. Использование безопасных атрибутов
```html
<!-- Правильно: безопасное использование атрибутов -->
<form action="/secure-endpoint" method="post">
    <input type="hidden" name="csrf_token" value="generated_secure_token">
    <input type="text" name="data" required>
    <button type="submit">Отправить</button>
</form>

<!-- Безопасные ссылки -->
<a href="https://trusted-site.com" 
   rel="noopener noreferrer" 
   target="_blank">Доверенный сайт</a>

<!-- Безопасные iframe -->
<iframe src="https://trusted-video.com" 
        sandbox="allow-scripts allow-same-origin allow-popups"
        loading="lazy"
        referrerpolicy="no-referrer-when-downgrade">
</iframe>
```

### 3. Защита от clickjacking
```html
<!-- Защита через CSP -->
<meta http-equiv="Content-Security-Policy" content="frame-ancestors 'none';">

<!-- Альтернативный способ через X-Frame-Options -->
<meta http-equiv="X-Frame-Options" content="DENY">
```

### 4. Использование безопасных протоколов
```html
<!-- Всегда используйте HTTPS для важных данных -->
<form action="https://secure.example.com/api/submit" method="post">
    <input type="text" name="sensitive-data">
    <button type="submit">Отправить</button>
</form>
```

### 5. Subresource Integrity для внешних ресурсов
```html
<!-- Защита внешних ресурсов с SRI -->
<script src="https://cdn.example.com/library.js"
        integrity="sha384-..."
        crossorigin="anonymous"></script>

<link rel="stylesheet" 
      href="https://fonts.example.com/font.css"
      integrity="sha384-..."
      crossorigin="anonymous">
```

### 6. Content Security Policy
```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self';
               script-src 'self' 'nonce-abc123' https://trusted-cdn.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
               img-src 'self' data: https:;
               font-src 'self' https://fonts.gstatic.com;
               connect-src 'self' https://api.example.com;
               frame-ancestors 'none';
               object-src 'none';
               base-uri 'self';
               form-action 'self';
               upgrade-insecure-requests;">
```

## Заключение

Безопасность HTML-страниц - это комплексный подход, который включает в себя правильное использование семантических элементов, ARIA-атрибутов, управление фокусом, клавиатурную навигацию и другие техники. Создание доступных веб-сайтов и приложений позволяет использовать их всем пользователям, включая тех, кто использует вспомогательные технологии.

Ключевые аспекты безопасности HTML:
1. Использование семантических элементов
2. Правильная структура документа
3. ARIA атрибуты для дополнительной информации
4. Управление фокусом и клавиатурной навигацией
5. Альтернативный текст для изображений
6. Доступные формы с понятными метками
7. Контрастность цветов
8. Совместимость с вспомогательными технологиями
9. Следование стандартам WCAG
10. Регулярное тестирование доступности
11. Использование современных возможностей безопасности (CSP, SRI, Trusted Types)
12. Безопасная обработка пользовательского ввода
13. Защита от XSS, CSRF и других атак
14. Интеграция с безопасными веб-API
15. Использование безопасных протоколов и методов передачи данных

Эти практики обеспечивают создание веб-сайтов, которые безопасны, доступны и удобны для всех пользователей, независимо от их возможностей или используемых технологий.

Современные подходы к безопасности включают:
- Использование Content Security Policy для ограничения источников контента
- Subresource Integrity для проверки целостности внешних ресурсов
- Trusted Types для предотвращения XSS
- Proper input sanitization and validation
- Secure WebSocket connections with message validation
- Protected IndexedDB operations
- Safe DOM manipulation practices
- Modern web component patterns with security in mind
- Testing with automated tools and manual verification
- Following security best practices and standards

Эти методы позволяют создавать действительно безопасные веб-приложения, которые обеспечивают защиту данных пользователей и предотвращают различные типы атак.

Ключевые принципы безопасности:
1. Defense in depth - многоуровневая защита
2. Principle of least privilege - минимальные привилегии
3. Fail securely - безопасное поведение при ошибках
4. Secure defaults - безопасные настройки по умолчанию
5. Complete mediation - полная проверка доступа
6. Open design - открытый дизайн, секреты в ключах
7. Separation of privileges - разделение привилегий
8. Economy of mechanism - простота механизмов
9. Least common mechanism - минимальные общие механизмы
10. Psychological acceptability - удобство для пользователей

Эти принципы помогают создавать безопасные веб-приложения, которые защищают данные пользователей и обеспечивают надежную защиту от потенциальных угроз.

## Следующие темы
- [[HTML/Производительность]]
- [[HTML/Доступность]]
- [[HTML/Совместимость]]

## Теги
#html #security #web-development #frontend #xss #csrf #csp #sri #clickjacking #web-components #javascript #dom-manipulation #input-sanitization #content-security-policy #subresource-integrity #trusted-types #webauthn #indexeddb #websockets #security-headers #secure-coding #best-practices #security-auditing #security-monitoring #security-tools #security-standards