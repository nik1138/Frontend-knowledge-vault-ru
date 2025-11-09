# HTML Безопасность

Безопасность HTML-страниц - это критически важный аспект веб-разработки, который включает в себя защиту от различных типов атак, таких как XSS (межсайтовый скриптинг), CSRF (межсайтовая подделка запроса), clickjacking и других. Правильная реализация мер безопасности помогает защитить как пользователей, так и веб-приложения от потенциальных угроз.

## Основы веб-безопасности

### Основные угрозы безопасности

1. **XSS (Cross-Site Scripting)** - внедрение вредоносного JavaScript
2. **CSRF (Cross-Site Request Forgery)** - подделка межсайтовых запросов
3. **Clickjacking** - атака методом кликджекинга
4. **Insecure Direct Object References** - небезопасные прямые ссылки на объекты
5. **Security Misconfiguration** - неправильная конфигурация безопасности
6. **Sensitive Data Exposure** - утечка конфиденциальных данных
7. **Using Components with Known Vulnerabilities** - использование уязвимых компонентов
8. **Insufficient Logging & Monitoring** - недостаточное логирование и мониторинг

## XSS (Cross-Site Scripting) защита

### Типы XSS атак:

1. **Reflected XSS** - скрипт отражается от сервера
2. **Stored XSS** - скрипт сохраняется на сервере
3. **DOM-based XSS** - скрипт выполняется в DOM

### Защита от XSS в HTML

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Защита от XSS</title>
</head>
<body>
    <h1>Защита от XSS атак</h1>
    
    <form id="xss-protection-form">
        <div class="form-group">
            <label for="comment">Комментарий:</label>
            <textarea id="comment" name="comment" rows="4" cols="50"></textarea>
        </div>
        
        <button type="submit">Отправить комментарий</button>
    </form>
    
    <div id="comments-container">
        <h2>Комментарии</h2>
        <div id="comments-list"></div>
    </div>

    <script>
        class XSSProtection {
            constructor() {
                this.comments = [];
                this.form = document.getElementById('xss-protection-form');
                this.commentInput = document.getElementById('comment');
                this.commentsList = document.getElementById('comments-list');
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.handleCommentSubmission();
                });
            }
            
            handleCommentSubmission() {
                const commentText = this.commentInput.value.trim();
                
                if (commentText) {
                    // Правильная защита от XSS - экранирование HTML
                    const sanitizedComment = this.sanitizeHTML(commentText);
                    
                    this.addComment(sanitizedComment);
                    this.commentInput.value = '';
                }
            }
            
            sanitizeHTML(html) {
                // Создаем временный элемент для безопасного обработки
                const tempDiv = document.createElement('div');
                tempDiv.textContent = html; // Устанавливаем как текст, не как HTML
                return tempDiv.innerHTML;
            }
            
            addComment(comment) {
                const commentElement = document.createElement('div');
                commentElement.className = 'comment';
                
                // ИСПОЛЬЗУЕМ textContent для предотвращения XSS
                commentElement.textContent = comment;
                
                // Добавляем время комментария
                const timeElement = document.createElement('span');
                timeElement.className = 'comment-time';
                timeElement.textContent = ` (${new Date().toLocaleTimeString()})`;
                
                commentElement.appendChild(timeElement);
                this.commentsList.appendChild(commentElement);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new XSSProtection();
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        textarea {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-family: Arial, sans-serif;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.5rem 1rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .comment {
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 0.75rem;
            margin-bottom: 0.5rem;
        }
        
        .comment-time {
            font-size: 0.8em;
            color: #666;
            margin-left: 0.5rem;
        }
    </style>
</body>
</html>
```

### Санитизация данных на стороне клиента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Санитизация данных</title>
</head>
<body>
    <h1>Санитизация пользовательских данных</h1>
    
    <div class="sanitization-demo">
        <div class="input-section">
            <label for="unsafe-input">Введите потенциально опасный контент:</label>
            <textarea id="unsafe-input" rows="5" cols="60" placeholder="Введите HTML или JavaScript код для теста"></textarea>
            <button onclick="sanitizeAndDisplay()">Санировать и отобразить</button>
        </div>
        
        <div class="output-section">
            <h3>Безопасный вывод:</h3>
            <div id="safe-output" class="output-container"></div>
        </div>
    </div>

    <script>
        class ContentSanitizer {
            constructor() {
                this.outputContainer = document.getElementById('safe-output');
            }
            
            // Основная функция санитизации
            sanitizeContent(content) {
                if (typeof content !== 'string') return content;
                
                // Базовая санитизация данных
                return this.basicSanitize(content);
            }
            
            basicSanitize(content) {
                // Экранирование HTML тегов
                return content
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
            
            advancedSanitize(content) {
                // Более продвинутая санитизация
                const tempDiv = document.createElement('div');
                tempDiv.textContent = content;
                
                // Удаляем потенциально опасные теги
                const dangerousTags = [
                    'script', 'iframe', 'object', 'embed', 'form', 
                    'input', 'button', 'link', 'meta', 'base'
                ];
                
                dangerousTags.forEach(tag => {
                    const elements = tempDiv.querySelectorAll(tag);
                    elements.forEach(el => el.remove());
                });
                
                // Удаляем потенциально опасные атрибуты
                const allElements = tempDiv.querySelectorAll('*');
                allElements.forEach(el => {
                    const dangerousAttrs = ['onclick', 'onload', 'onerror', 'onmouseover', 'onfocus', 'onblur'];
                    dangerousAttrs.forEach(attr => {
                        if (el.hasAttribute(attr)) {
                            el.removeAttribute(attr);
                        }
                    });
                });
                
                return tempDiv.innerHTML;
            }
            
            // Санитизация URL
            sanitizeURL(url) {
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
            
            // Санитизация стилей
            sanitizeStyles(cssText) {
                // Блокируем потенциально опасные CSS свойства
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
            
            displayContent(content, isSafe = true) {
                if (isSafe) {
                    this.outputContainer.innerHTML = `<div class="safe-content">${content}</div>`;
                } else {
                    const sanitizedContent = this.sanitizeContent(content);
                    this.outputContainer.innerHTML = `<div class="sanitized-content">${sanitizedContent}</div>`;
                }
            }
        }
        
        const sanitizer = new ContentSanitizer();
        
        function sanitizeAndDisplay() {
            const userInput = document.getElementById('unsafe-input').value;
            sanitizer.displayContent(userInput, false);
        }
        
        // Примеры опасного контента для тестирования
        const dangerousExamples = [
            '<script>alert("XSS")</script>',
            '<img src="x" onerror="alert(\'XSS\')">',
            '<a href="javascript:alert(\'XSS\')">Кликни меня</a>',
            '<div onclick="alert(\'XSS\')">Кликабельный div</div>'
        ];
        
        // Демонстрация санитизации
        console.log('Примеры санитизации:');
        dangerousExamples.forEach(example => {
            console.log(`Оригинал: ${example}`);
            console.log(`Санированный: ${sanitizer.sanitizeContent(example)}`);
            console.log('---');
        });
    </script>
    
    <style>
        .sanitization-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .input-section, .output-section {
            margin-bottom: 20px;
        }
        
        .output-container {
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 4px;
            background-color: #f5f5f5;
            min-height: 100px;
        }
        
        .safe-content {
            color: #2e7d32;
        }
        
        .sanitized-content {
            color: #0277bd;
        }
    </style>
</body>
</html>
```

## Content Security Policy (CSP)

### CSP Header и атрибуты

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Content Security Policy</title>
    
    <!-- Пример CSP политики -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self'; 
                   script-src 'self' 'unsafe-inline' https://trusted-cdn.com; 
                   style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; 
                   img-src 'self' data: https:; 
                   font-src 'self' https://fonts.gstatic.com; 
                   connect-src 'self' https://api.example.com; 
                   frame-ancestors 'none'; 
                   base-uri 'self'; 
                   form-action 'self'; 
                   upgrade-insecure-requests;">
    
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
</head>
<body>
    <h1>Content Security Policy</h1>

    <div class="content">
        <p>Эта страница использует Content Security Policy для защиты от XSS атак.</p>
        
        <!-- Встроенные стили разрешены из-за 'unsafe-inline' в style-src -->
        <div style="color: #007acc; padding: 10px; background-color: #f8f9fa;">
            Стили с атрибутом style разрешены
        </div>
        
        <!-- Встроенный скрипт (только для демонстрации) -->
        <script>
            // Этот скрипт будет работать, потому что 'unsafe-inline' разрешен
            console.log('CSP тест: скрипт выполнен');
        </script>
    </div>

    <script>
        class CSPManager {
            constructor() {
                this.checkCSPSupport();
                this.setupSecurityMonitoring();
            }
            
            checkCSPSupport() {
                if (window.isSecureContext) {
                    console.log('Безопасный контекст (HTTPS или localhost)');
                } else {
                    console.warn('Не безопасный контекст - CSP может работать не полностью');
                }
            }
            
            setupSecurityMonitoring() {
                // Мониторинг нарушений CSP
                document.addEventListener('securitypolicyviolation', (e) => {
                    console.error('Нарушение CSP:', {
                        blockedURI: e.blockedURI,
                        violatedDirective: e.violatedDirective,
                        originalPolicy: e.originalPolicy
                    });
                    
                    // В реальном приложении можно отправить отчет на сервер
                    this.reportCSPViolation(e);
                });
            }
            
            reportCSPViolation(event) {
                // Отправка отчета о нарушении CSP
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/csp-report', JSON.stringify({
                        blockedURI: event.blockedURI,
                        violatedDirective: event.violatedDirective,
                        originalPolicy: event.originalPolicy,
                        timestamp: new Date().toISOString()
                    }));
                } else {
                    // Резервный метод для старых браузеров
                    fetch('/csp-report', {
                        method: 'POST',
                        body: JSON.stringify({
                            blockedURI: event.blockedURI,
                            violatedDirective: event.violatedDirective,
                            originalPolicy: event.originalPolicy,
                            timestamp: new Date().toISOString()
                        }),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CSPManager();
        });
    </script>
</body>
</html>
```

### Различные CSP политики для разных сценариев

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Различные CSP политики</title>
    
    <!-- Строгая CSP политика -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123' https://analytics.example.com;
                   style-src 'self' 'nonce-def456' https://fonts.googleapis.com;
                   img-src 'self' data: https:;
                   font-src 'self' https://fonts.gstatic.com;
                   connect-src 'self' https://api.example.com;
                   media-src 'self';
                   object-src 'none';
                   child-src 'self';
                   frame-ancestors 'none';
                   form-action 'self';
                   base-uri 'self';
                   upgrade-insecure-requests;">
    
    <!-- CSP для разработки -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self' 'unsafe-inline' 'unsafe-eval';
                   script-src 'self' 'unsafe-inline' 'unsafe-eval' http: https:;
                   style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
                   img-src 'self' data: blob: http: https:;
                   font-src 'self' 'unsafe-inline' https://fonts.gstatic.com;">
    
    <!-- CSP для API-сервера -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'none';
                   script-src 'self';
                   style-src 'self';
                   img-src 'self';
                   connect-src 'self' https://api.example.com;
                   font-src 'self';
                   media-src 'self';
                   object-src 'none';
                   frame-ancestors 'none';
                   form-action 'self';
                   base-uri 'self';">
</head>
<body>
    <h1>Различные CSP политики</h1>

    <!-- Использование nonce для разрешения конкретных скриптов -->
    <script nonce="abc123">
        // Этот скрипт разрешен благодаря nonce
        document.addEventListener('DOMContentLoaded', function() {
            console.log('Безопасный скрипт с nonce выполнен');
        });
    </script>
    
    <style nonce="def456">
        /* Этот стиль разрешен благодаря nonce */
        body {
            font-family: 'Roboto', sans-serif;
            background-color: #f5f5f5;
        }
    </style>
</body>
</html>
```

## CSRF (Cross-Site Request Forgery) защита

### CSRF токены

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF защита</title>
</head>
<body>
    <h1>Защита от CSRF атак</h1>
    
    <form id="csrf-protected-form" action="/transfer" method="post">
        <!-- CSRF токен как скрытое поле -->
        <input type="hidden" name="csrf_token" value="generated_csrf_token_here">
        
        <div class="form-group">
            <label for="amount">Сумма перевода:</label>
            <input type="number" id="amount" name="amount" required>
        </div>
        
        <div class="form-group">
            <label for="recipient">Получатель:</label>
            <input type="text" id="recipient" name="recipient" required>
        </div>
        
        <button type="submit">Выполнить перевод</button>
    </form>

    <script>
        class CSRFProtection {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.tokenInput = this.form.querySelector('input[name="csrf_token"]');
                this.setupValidation();
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    this.validateCSRFToken(e);
                });
            }
            
            validateCSRFToken(event) {
                const token = this.tokenInput.value;
                
                if (!token) {
                    event.preventDefault();
                    alert('Отсутствует CSRF токен. Форма не может быть отправлена.');
                    return false;
                }
                
                // Проверка токена (в реальном приложении - на сервере)
                if (!this.isValidToken(token)) {
                    event.preventDefault();
                    alert('Неверный CSRF токен. Попробуйте перезагрузить страницу.');
                    return false;
                }
                
                // Добавление токена в заголовок для AJAX запросов
                event.submitter.form.setAttribute('data-csrf-token', token);
                
                return true;
            }
            
            isValidToken(token) {
                // Простая проверка (в реальном приложении - более сложная)
                return token.length > 10 && /^[A-Za-z0-9-_]+$/.test(token);
            }
            
            // Метод для AJAX запросов с CSRF защитой
            async secureAjax(url, options = {}) {
                const csrfToken = this.tokenInput.value;
                
                const headers = {
                    'Content-Type': 'application/json',
                    'X-CSRF-Token': csrfToken,
                    'X-Requested-With': 'XMLHttpRequest',
                    ...options.headers
                };
                
                return fetch(url, {
                    ...options,
                    headers
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CSRFProtection('csrf-protected-form');
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        button {
            background-color: #dc3545;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

## Clickjacking защита

### X-Frame-Options и Frame Ancestors

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Защита от Clickjacking</title>
    
    <!-- Защита от clickjacking через CSP -->
    <meta http-equiv="Content-Security-Policy" content="frame-ancestors 'none';">
    
    <!-- Альтернативный способ через X-Frame-Options -->
    <meta http-equiv="X-Frame-Options" content="DENY">
</head>
<body>
    <h1>Защита от Clickjacking</h1>
    
    <div class="sensitive-content">
        <h2>Конфиденциальная информация</h2>
        <p>Этот контент не может быть встроен в iframe на других сайтах.</p>
    </div>
    
    <script>
        class ClickjackingProtection {
            constructor() {
                this.checkFrameAncestors();
                this.setupVisibilityMonitoring();
            }
            
            checkFrameAncestors() {
                // Проверка, находится ли страница в iframe
                if (window.self !== window.top) {
                    // Страница открыта во фрейме
                    console.warn('Страница открыта во фрейме');
                    
                    // Проверяем, разрешен ли этот фрейм
                    this.verifyFrameAncestors();
                }
            }
            
            verifyFrameAncestors() {
                try {
                    // Попытка доступа к родительскому окну
                    const parentOrigin = window.parent.location.origin;
                    const currentOrigin = window.location.origin;
                    
                    if (parentOrigin !== currentOrigin) {
                        console.error('Недопустимый родительский фрейм');
                        // В реальном приложении здесь может быть перенаправление
                        this.breakOutFrame();
                    }
                } catch (e) {
                    // Ошибка доступа к родительскому окну - это нормально для защищенных фреймов
                    console.log('Доступ к родительскому окну ограничен политикой безопасности');
                }
            }
            
            breakOutFrame() {
                // Попытка выхода из фрейма (в некоторых случаях)
                if (window.self !== window.top) {
                    window.top.location = window.location;
                }
            }
            
            setupVisibilityMonitoring() {
                // Мониторинг видимости страницы
                document.addEventListener('visibilitychange', () => {
                    if (document.visibilityState === 'visible') {
                        // Проверка после возвращения к странице
                        this.verifyPageContext();
                    }
                });
            }
            
            verifyPageContext() {
                // Проверка контекста отображения страницы
                if (window.self !== window.top) {
                    // Страница во фрейме - дополнительная проверка
                    this.enhancedFrameCheck();
                }
            }
            
            enhancedFrameCheck() {
                // Более тщательная проверка фрейма
                const allowedOrigins = [
                    window.location.origin,
                    'https://trusted-site.com'
                ];
                
                try {
                    const parentOrigin = window.parent.location.origin;
                    if (!allowedOrigins.includes(parentOrigin)) {
                        console.error('Недопустимый домен-родитель');
                        // Здесь может быть показ предупреждения пользователю
                        this.showSecurityWarning();
                    }
                } catch {
                    // Не можем определить домен родителя - это может быть нормально
                    console.log('Невозможно определить домен родителя');
                }
            }
            
            showSecurityWarning() {
                const warning = document.createElement('div');
                warning.className = 'security-warning';
                warning.innerHTML = `
                    <p>⚠️ Эта страница может быть открыта в небезопасном контексте.</p>
                    <p>Пожалуйста, убедитесь, что вы находитесь на доверенном сайте.</p>
                `;
                
                document.body.prepend(warning);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new ClickjackingProtection();
        });
    </script>
    
    <style>
        .sensitive-content {
            background-color: #fff3cd;
            border: 1px solid #ffeaa7;
            border-radius: 8px;
            padding: 1rem;
            margin: 1rem 0;
        }
        
        .security-warning {
            background-color: #f8d7da;
            color: #721c24;
            padding: 1rem;
            border-radius: 4px;
            margin: 1rem 0;
            text-align: center;
        }
    </style>
</body>
</html>
```

## Безопасные формы и обработка данных

### Создание безопасных форм

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасные формы</title>
</head>
<body>
    <h1>Безопасные формы и обработка данных</h1>
    
    <form id="secure-form" novalidate>
        <div class="form-group">
            <label for="secure-name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="text"
                   id="secure-name"
                   name="name"
                   required
                   minlength="2"
                   maxlength="50"
                   pattern="[A-Za-zА-Яа-яЁё\s\-]+"
                   autocomplete="name"
                   aria-describedby="name-help name-error">
            <div id="name-help" class="help-text">Только буквы, пробелы и дефисы</div>
            <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="email"
                   id="secure-email"
                   name="email"
                   required
                   autocomplete="email"
                   aria-describedby="email-help email-error">
            <div id="email-help" class="help-text">Действительный email адрес</div>
            <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="password"
                   id="secure-password"
                   name="password"
                   required
                   minlength="8"
                   aria-describedby="password-requirements password-error">
            <ul id="password-requirements" class="requirements-list">
                <li id="req-length">Не менее 8 символов</li>
                <li id="req-upper">С заглавной буквой</li>
                <li id="req-lower">Со строчной буквой</li>
                <li id="req-number">С цифрой</li>
                <li id="req-special">Со специальным символом</li>
            </ul>
            <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-confirm">Подтверждение пароля: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="password"
                   id="secure-confirm"
                   name="confirm_password"
                   required
                   aria-describedby="confirm-error">
            <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-bio">О себе:</label>
            <textarea id="secure-bio"
                      name="bio"
                      rows="4"
                      maxlength="500"
                      aria-describedby="bio-help bio-error"></textarea>
            <div id="bio-help" class="help-text">Краткое описание (максимум 500 символов)</div>
            <div id="bio-error" class="error-message" role="alert" aria-live="polite"></div>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="newsletter"
                       value="yes">
                Подписаться на рассылку
            </label>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="terms"
                       required
                       aria-describedby="terms-description">
                Я согласен с условиями использования
            </label>
            <div id="terms-description" class="help-text">Обязательное согласие для регистрации</div>
        </div>
        
        <button type="submit">Зарегистрироваться</button>
    </form>

    <script>
        class SecureFormHandler {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupValidation();
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                });
            }
            
            validateAndSubmit() {
                // Сброс ошибок
                this.clearAllErrors();
                
                let isValid = true;
                
                // Проверка всех обязательных полей
                const fields = this.form.querySelectorAll('input[required], textarea[required]');
                for (const field of fields) {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                }
                
                // Проверка сложных требований
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }
                
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }
                
                if (isValid) {
                    this.submitSecurely();
                }
            }
            
            validateField(field) {
                const fieldName = field.name.replace(/-/g, '');
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showFieldError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                if (field.pattern && field.value && !new RegExp(field.pattern).test(field.value)) {
                    this.showFieldError(field, errorElement, field.title || 'Неверный формат данных');
                    return false;
                }
                
                // Проверка на опасный контент
                if (this.containsMaliciousContent(field.value)) {
                    this.showFieldError(field, errorElement, 'Поле содержит недопустимый контент');
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            validatePasswordRequirements() {
                const password = document.getElementById('secure-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                // Обновление индикаторов требований
                Object.entries(requirements).forEach(([key, met]) => {
                    const element = document.getElementById(`req-${key}`);
                    if (element) {
                        element.classList.toggle('met', met);
                    }
                });
                
                const allMet = Object.values(requirements).every(req => req);
                
                if (!allMet && password) {
                    const errorElement = document.getElementById('password-error');
                    this.showFieldError(document.getElementById('secure-password'), errorElement, 'Пароль не соответствует требованиям');
                    return false;
                }
                
                return allMet;
            }
            
            validatePasswordMatch() {
                const password = document.getElementById('secure-password').value;
                const confirmPassword = document.getElementById('secure-confirm').value;
                const errorElement = document.getElementById('confirm-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    this.showFieldError(document.getElementById('secure-confirm'), errorElement, 'Пароли не совпадают');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            showFieldError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Устанавливаем фокус на поле с ошибкой
                if (!field.matches(':focus')) {
                    field.focus();
                }
            }
            
            clearAllErrors() {
                this.form.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                this.form.querySelectorAll('input, textarea').forEach(field => {
                    field.classList.remove('invalid', 'valid');
                    field.setAttribute('aria-invalid', 'false');
                });
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            containsMaliciousContent(content) {
                if (typeof content !== 'string') return false;
                
                const maliciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return maliciousPatterns.some(pattern => pattern.test(content));
            }
            
            async submitSecurely() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор и санитизация данных
                    const formData = new FormData(this.form);
                    const sanitizedData = {};
                    
                    for (const [key, value] of formData.entries()) {
                        if (key !== 'csrf_token') { // Исключаем токен из данных
                            sanitizedData[key] = this.sanitizeInput(value);
                        }
                    }
                    
                    // Добавление CSRF токена
                    const csrfToken = this.form.querySelector('input[name="csrf_token"]').value;
                    
                    const response = await fetch('/api/secure-register', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-CSRF-Token': csrfToken,
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Регистрация прошла успешно!');
                        this.form.reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка регистрации: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new SecureFormHandler('secure-form');
        });
    </script>
    
    <style>
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
        
        input:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.valid {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }
        
        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }
        
        .requirements-list li.met {
            color: #28a745;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
    </style>
</body>
</html>
```

## Subresource Integrity (SRI)

### Защита внешних ресурсов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Subresource Integrity</title>
    
    <!-- Защита внешних CSS файлов -->
    <link rel="stylesheet" 
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
          integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
          crossorigin="anonymous">
    
    <!-- Защита внешних шрифтов -->
    <link rel="stylesheet" 
          href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap"
          integrity="sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We"
          crossorigin="anonymous">
    
    <!-- Защита внешних JavaScript библиотек -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.6.0/dist/jquery.min.js"
            integrity="sha384-vtXRMe3mGCbOeY7l30aIg8H9p3GdeSe4IFlP6G8JMa7o7lXvnz3GFKzPxzJdPfGK"
            crossorigin="anonymous"></script>
</head>
<body>
    <h1>Subresource Integrity (SRI)</h1>
    
    <p>Эта страница использует SRI для защиты внешних ресурсов.</p>
    
    <!-- Кнопка для демонстрации работы jQuery -->
    <button id="jquery-test" onclick="testjQuery()">Тест jQuery</button>

    <script>
        class SRILoader {
            constructor() {
                this.loadedResources = new Set();
            }
            
            async loadScriptWithSRI(src, integrity, crossorigin = 'anonymous') {
                if (this.loadedResources.has(src)) {
                    console.log('Ресурс уже загружен:', src);
                    return Promise.resolve();
                }
                
                return new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = src;
                    script.integrity = integrity;
                    script.crossOrigin = crossorigin;
                    
                    script.onload = () => {
                        this.loadedResources.add(src);
                        console.log('Скрипт успешно загружен с SRI:', src);
                        resolve();
                    };
                    
                    script.onerror = () => {
                        console.error('Ошибка загрузки скрипта с SRI:', src);
                        reject(new Error(`Failed to load script with SRI: ${src}`));
                    };
                    
                    document.head.appendChild(script);
                });
            }
            
            async loadStylesheetWithSRI(href, integrity, crossorigin = 'anonymous') {
                if (this.loadedResources.has(href)) {
                    console.log('Стиль уже загружен:', href);
                    return Promise.resolve();
                }
                
                return new Promise((resolve, reject) => {
                    const link = document.createElement('link');
                    link.rel = 'stylesheet';
                    link.href = href;
                    link.integrity = integrity;
                    link.crossOrigin = crossorigin;
                    
                    link.onload = () => {
                        this.loadedResources.add(href);
                        console.log('Стиль успешно загружен с SRI:', href);
                        resolve();
                    };
                    
                    link.onerror = () => {
                        console.error('Ошибка загрузки стиля с SRI:', href);
                        reject(new Error(`Failed to load stylesheet with SRI: ${href}`));
                    };
                    
                    document.head.appendChild(link);
                });
            }
            
            // Генерация хеша для SRI (в реальном приложении на сервере)
            generateSRIHash(content) {
                return crypto.subtle.digest('SHA-384', new TextEncoder().encode(content))
                    .then(hash => {
                        return 'sha384-' + btoa(String.fromCharCode(...new Uint8Array(hash)));
                    });
            }
        }
        
        // Инициализация
        const sriLoader = new SRILoader();
        
        function testjQuery() {
            if (typeof $ !== 'undefined') {
                alert('jQuery успешно загружен с SRI!');
            } else {
                alert('jQuery не загружен');
            }
        }
        
        // Пример динамической загрузки с SRI
        sriLoader.loadScriptWithSRI(
            'https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js',
            'sha384-4hvpebDOcE5Tc6Z+hbsG4dIFlP6G8JMa7o7lXvnz3GFKzPxzJdPfGK'
        ).then(() => {
            console.log('Lodash загружен с SRI');
        }).catch(error => {
            console.error('Ошибка загрузки Lodash:', error);
        });
    </script>
    
    <style>
        #jquery-test {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

## Современные возможности безопасности

### Trusted Types API для защиты от XSS

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Trusted Types API</title>
    
    <!-- Включение Trusted Types политики -->
    <meta http-equiv="Content-Security-Policy" 
          content="require-trusted-types-for 'script';">
</head>
<body>
    <h1>Trusted Types API</h1>
    
    <div id="content-container">
        <p>Контент будет безопасно добавлен сюда</p>
    </div>
    
    <button id="inject-content">Добавить содержимое</button>

    <script>
        class TrustedTypesManager {
            constructor() {
                this.policy = null;
                this.contentContainer = document.getElementById('content-container');
                
                this.setupTrustedTypes();
            }
            
            setupTrustedTypes() {
                if (window.trustedTypes && window.trustedTypes.createPolicy) {
                    // Создаем политику Trusted Types
                    this.policy = window.trustedTypes.createPolicy('htmlPolicy', {
                        createHTML: (string, ...args) => {
                            // Санитизация HTML
                            const temp = document.createElement('div');
                            temp.textContent = string;
                            
                            // Проверка на опасные паттерны
                            if (this.containsDangerousContent(temp.innerHTML)) {
                                console.warn('Trusted Types: обнаружен опасный контент, очистка...');
                                return this.sanitizeHTML(temp.innerHTML);
                            }
                            
                            return temp.innerHTML;
                        },
                        
                        createScript: (string) => {
                            // Проверка скрипта
                            if (this.isSafeScript(string)) {
                                return string;
                            }
                            console.warn('Trusted Types: небезопасный скрипт заблокирован');
                            return '';
                        },
                        
                        createScriptURL: (url) => {
                            // Проверка URL скрипта
                            if (this.isSafeScriptURL(url)) {
                                return url;
                            }
                            console.warn('Trusted Types: небезопасный URL скрипта заблокирован');
                            return 'about:blank';
                        }
                    });
                    
                    console.log('Trusted Types политика создана и активна');
                } else {
                    console.warn('Trusted Types не поддерживается в этом браузере');
                    // Резервный вариант для старых браузеров
                    this.fallbackSanitization = true;
                }
            }
            
            containsDangerousContent(html) {
                if (typeof html !== 'string') return false;
                
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];
                
                return dangerousPatterns.some(pattern => pattern.test(html));
            }
            
            isSafeScript(script) {
                // Проверка на опасные паттерны
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /eval\(/i,
                    /setTimeout\([^)]*['"]/i,
                    /setInterval\([^)]*['"]/i,
                    /Function\(/i,
                    /document\.cookie/i,
                    /localStorage/i,
                    /sessionStorage/i
                ];
                
                return !dangerousPatterns.some(pattern => pattern.test(script));
            }
            
            isSafeScriptURL(url) {
                try {
                    const parsedUrl = new URL(url, window.location.origin);
                    
                    // Проверка протокола
                    const allowedProtocols = ['https:', 'http:', 'data:'];
                    if (!allowedProtocols.includes(parsedUrl.protocol)) {
                        return false;
                    }
                    
                    // Проверка домена
                    const allowedDomains = [
                        window.location.hostname,
                        'trusted-cdn.com',
                        'secure-scripts.com'
                    ];
                    
                    return allowedDomains.includes(parsedUrl.hostname) || 
                           allowedDomains.some(domain => parsedUrl.hostname.endsWith('.' + domain));
                } catch {
                    return false;
                }
            }
            
            sanitizeHTML(html) {
                // Резервная санитизация для старых браузеров
                const temp = document.createElement('div');
                temp.textContent = html;
                return temp.innerHTML;
            }
            
            injectTrustedContent() {
                const dangerousContent = '<p>Опасный контент <script>alert("XSS!")</script></p>';
                const safeContent = '<p>Безопасный контент без вредоносных скриптов</p>';
                
                if (this.policy) {
                    // Используем безопасный способ вставки HTML
                    this.contentContainer.innerHTML = this.policy.createHTML(safeContent);
                    
                    // Попытка вставить опасный контент будет заблокирована
                    try {
                        // Это вызовет ошибку в браузере с включенным Trusted Types
                        // this.contentContainer.innerHTML = this.policy.createHTML(dangerousContent);
                    } catch (error) {
                        console.error('Trusted Types предотвратил вставку опасного контента:', error);
                    }
                } else {
                    // Резервный способ для браузеров без Trusted Types
                    this.contentContainer.innerHTML = this.sanitizeHTML(safeContent);
                }
            }
        }
        
        // Инициализация
        const trustedTypesManager = new TrustedTypesManager();
        
        document.getElementById('inject-content').addEventListener('click', () => {
            trustedTypesManager.injectTrustedContent();
        });
    </script>
</body>
</html>
```

### Безопасные Web Components

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасные Web Components</title>
</head>
<body>
    <h1>Безопасные Web Components</h1>
    
    <secure-user-card 
        username="john_doe" 
        email="john@example.com" 
        avatar="avatar.jpg">
    </secure-user-card>
    
    <form-validator form-id="user-form"></form-validator>

    <script>
        // Безопасный кастомный элемент профиля пользователя
        class SecureUserCard extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'closed' }); // Используем closed для дополнительной безопасности
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['username', 'email', 'avatar'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    this[name] = this.sanitizeInput(newValue);
                    this.render();
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: inline-block;
                            font-family: Arial, sans-serif;
                        }
                        
                        .card {
                            border: 1px solid #ddd;
                            border-radius: 8px;
                            padding: 20px;
                            max-width: 300px;
                            background-color: white;
                            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                        }
                        
                        .avatar {
                            width: 80px;
                            height: 80px;
                            border-radius: 50%;
                            object-fit: cover;
                            margin-bottom: 15px;
                        }
                        
                        .username {
                            font-size: 1.2em;
                            font-weight: bold;
                            margin: 0 0 10px 0;
                        }
                        
                        .email {
                            color: #666;
                            margin: 0 0 15px 0;
                        }
                        
                        .actions {
                            display: flex;
                            gap: 10px;
                        }
                        
                        .action-btn {
                            padding: 8px 16px;
                            border: none;
                            border-radius: 4px;
                            cursor: pointer;
                            font-size: 0.9em;
                        }
                        
                        .message-btn {
                            background-color: #007acc;
                            color: white;
                        }
                        
                        .follow-btn {
                            background-color: #28a745;
                            color: white;
                        }
                    </style>
                    
                    <div class="card">
                        <img src="${this.safeUrl(this.avatar || 'default-avatar.png')}"
                             alt="Аватар ${this.escapeHTML(this.username)}"
                             class="avatar">
                        <h3 class="username">${this.escapeHTML(this.username || 'Анонимный пользователь')}</h3>
                        <p class="email">${this.escapeHTML(this.email || 'Email не указан')}</p>
                        <div class="actions">
                            <button class="action-btn message-btn" onclick="this.handleMessage()">Сообщение</button>
                            <button class="action-btn follow-btn" onclick="this.handleFollow()">Подписаться</button>
                        </div>
                    </div>
                `;
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                // Санитизация входных данных
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
            
            safeUrl(url) {
                try {
                    // Проверяем URL на безопасность
                    const parsedUrl = new URL(url, window.location.origin);
                    
                    // Разрешаем только безопасные протоколы
                    if (!['http:', 'https:', 'data:'].includes(parsedUrl.protocol)) {
                        return 'default-avatar.png';
                    }
                    
                    return parsedUrl.href;
                } catch (error) {
                    console.warn('Небезопасный URL:', url);
                    return 'default-avatar.png';
                }
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
            
            handleMessage() {
                // Безопасная обработка сообщения
                const username = this.getAttribute('username');
                if (username) {
                    // В реальном приложении здесь будет безопасный вызов API
                    console.log('Отправка сообщения пользователю:', username);
                }
            }
            
            handleFollow() {
                // Безопасная обработка подписки
                const username = this.getAttribute('username');
                if (username) {
                    // В реальном приложении здесь будет безопасный вызов API
                    console.log('Подписка на пользователя:', username);
                }
            }
        }
        
        // Безопасный валидатор форм
        class SecureFormValidator extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                this.form = null;
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['form-id'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'form-id') {
                        this.setupFormValidation(newValue);
                    }
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                        }
                        
                        .validation-results {
                            margin-top: 1rem;
                            padding: 0.75rem;
                            border-radius: 4px;
                            font-size: 0.9em;
                        }
                        
                        .validation-success {
                            background-color: #d4edda;
                            color: #155724;
                        }
                        
                        .validation-error {
                            background-color: #f8d7da;
                            color: #721c24;
                        }
                        
                        .error-list {
                            margin: 0;
                            padding-left: 1.5rem;
                            margin-top: 0.5rem;
                        }
                        
                        .error-item {
                            margin-bottom: 0.25rem;
                        }
                    </style>
                    
                    <div class="validation-results" 
                         id="validation-results" 
                         role="status" 
                         aria-live="polite"></div>
                `;
            }
            
            setupFormValidation(formId) {
                this.form = document.getElementById(formId);
                
                if (this.form) {
                    this.form.addEventListener('submit', (e) => {
                        e.preventDefault();
                        this.validateForm();
                    });
                    
                    // Валидация в реальном времени
                    this.form.addEventListener('input', (e) => {
                        if (e.target.matches('input[required], textarea[required]')) {
                            this.validateField(e.target);
                        }
                    });
                }
            }
            
            validateForm() {
                let isValid = true;
                const errors = [];
                
                const requiredFields = this.form.querySelectorAll('input[required], textarea[required]');
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                        errors.push({
                            field: field.name || field.id,
                            message: this.getFieldErrorMessage(field)
                        });
                    }
                });
                
                if (isValid) {
                    this.showSuccess('Форма валидна. Готова к отправке.');
                    // В реальном приложении здесь будет отправка формы
                } else {
                    this.showError(errors);
                }
                
                return isValid;
            }
            
            validateField(field) {
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                
                // Основные проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                return true;
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            getFieldErrorMessage(field) {
                if (field.hasAttribute('required') && !field.value.trim()) {
                    return 'Поле обязательно для заполнения';
                } else if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    return 'Введите действительный email адрес';
                } else if (field.minLength && field.value.length < field.minLength) {
                    return `Минимум ${field.minLength} символов`;
                } else if (field.maxLength && field.value.length > field.maxLength) {
                    return `Максимум ${field.maxLength} символов`;
                }
                return 'Неверный формат данных';
            }
            
            showError(errors) {
                const resultsElement = this.shadowRoot.getElementById('validation-results');
                resultsElement.className = 'validation-results validation-error';
                
                resultsElement.innerHTML = `
                    <strong>Найдены ошибки (${errors.length}):</strong>
                    <ul class="error-list">
                        ${errors.map(error => `
                            <li class="error-item">
                                ${error.field}: ${error.message}
                            </li>
                        `).join('')}
                    </ul>
                `;
            }
            
            showSuccess(message) {
                const resultsElement = this.shadowRoot.getElementById('validation-results');
                resultsElement.className = 'validation-results validation-success';
                resultsElement.innerHTML = `<strong>✓</strong> ${message}`;
            }
        }
        
        // Регистрация элементов
        customElements.define('secure-user-card', SecureUserCard);
        customElements.define('form-validator', SecureFormValidator);
    </script>
</body>
</html>
```

### Интеграция с WebSockets и безопасностью

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная интеграция с WebSockets</title>
</head>
<body>
    <h1>Безопасная интеграция с WebSockets</h1>
    
    <div class="websocket-security-container">
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
        class SecureWebSocketClient {
            constructor(url) {
                this.url = url;
                this.ws = null;
                this.isConnected = false;
                this.connectionAttempts = 0;
                this.maxConnectionAttempts = 3;
                this.connectionRetryDelay = 3000;
                
                this.messagesContainer = document.getElementById('messages-container');
                this.connectionStatus = document.getElementById('connection-status');
                this.messageInput = document.getElementById('message-input');
                this.sendBtn = document.getElementById('send-btn');
                
                this.setupSecureConnection();
            }
            
            setupSecureConnection() {
                document.getElementById('connect-btn').addEventListener('click', () => this.connectSecurely());
                document.getElementById('disconnect-btn').addEventListener('click', () => this.disconnectSecurely());
                document.getElementById('send-btn').addEventListener('click', () => this.sendMessage());
                
                this.messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter' && !this.messageInput.disabled) {
                        this.sendMessage();
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
                    
                    // В реальном приложении: this.ws = new WebSocket(this.url);
                    
                    // Для демонстрации создаем mock-соединение
                    this.ws = {
                        send: (message) => {
                            // Санитизация сообщения перед отправкой
                            const sanitizedMessage = this.sanitizeMessage(JSON.parse(message));
                            console.log('Отправлено:', sanitizedMessage);
                            
                            // Имитация получения сообщения от других пользователей
                            setTimeout(() => {
                                this.receiveMessage({
                                    user: 'Система',
                                    text: 'Сообщение доставлено',
                                    timestamp: new Date().toISOString()
                                });
                            }, 500);
                        },
                        close: () => {
                            this.isConnected = false;
                            this.updateConnectionStatus();
                        }
                    };
                    
                    // Имитация успешного подключения
                    setTimeout(() => {
                        this.isConnected = true;
                        this.connectionAttempts = 0;
                        this.updateConnectionStatus();
                        this.enableInputs();
                        
                        // Имитация получения сообщений
                        setInterval(() => {
                            if (Math.random() > 0.8) { // 20% шанс получить сообщение
                                const users = ['Алексей', 'Мария', 'Иван', 'Елена', 'Дмитрий'];
                                const messages = [
                                    'Привет всем!', 'Как дела?', 'Работаю над проектом', 'Кто на связи?',
                                    'Нужна помощь?', 'Все прошло отлично!', 'Планирую встречу', 'Обновил документацию'
                                ];
                                
                                this.receiveMessage({
                                    user: users[Math.floor(Math.random() * users.length)],
                                    text: messages[Math.floor(Math.random() * messages.length)],
                                    timestamp: new Date().toISOString()
                                });
                            }
                        }, 3000);
                    }, 1000);
                    
                    /*
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.connectionAttempts = 0;
                        this.updateConnectionStatus();
                        this.enableInputs();
                    };
                    
                    this.ws.onmessage = (event) => {
                        this.handleSecureMessage(event.data);
                    };
                    
                    this.ws.onclose = () => {
                        this.isConnected = false;
                        this.updateConnectionStatus();
                        this.disableInputs();
                        
                        // Автоматическое переподключение
                        if (event.code !== 1000) { // Нормальное закрытие
                            this.connectionAttempts++;
                            setTimeout(() => {
                                if (this.connectionAttempts < this.maxConnectionAttempts) {
                                    this.connectSecurely();
                                }
                            }, this.connectionRetryDelay);
                        }
                    };
                    
                    this.ws.onerror = (error) => {
                        console.error('WebSocket ошибка:', error);
                        this.updateConnectionStatus();
                    };
                    */
                } catch (error) {
                    console.error('Ошибка подключения к WebSocket:', error);
                    this.updateConnectionStatus();
                    this.disableInputs();
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
                        'secure-chat.example.com',
                        'api.trusted.com'
                    ];
                    
                    if (!this.isLocalhost(url)) {
                        return allowedDomains.some(domain => 
                            parsedUrl.hostname === domain || 
                            parsedUrl.hostname.endsWith('.' + domain)
                        );
                    }
                    
                    return true;
                } catch {
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
                    
                    // Санитизация содержимого
                    const sanitizedMessage = this.sanitizeMessage(message);
                    
                    this.displayMessage(sanitizedMessage, false);
                    
                } catch (error) {
                    console.error('Ошибка обработки сообщения:', error);
                    this.showSecurityError('Получено недопустимое сообщение');
                }
            }
            
            validateMessageStructure(message) {
                // Проверка обязательных полей
                const requiredFields = ['id', 'timestamp', 'content', 'user'];
                return requiredFields.every(field => field in message);
            }
            
            sanitizeMessage(message) {
                // Санитизация содержимого сообщения
                return {
                    ...message,
                    content: this.sanitizeInput(message.content),
                    user: this.sanitizeInput(message.user || 'Anonymous'),
                    timestamp: message.timestamp || new Date().toISOString()
                };
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
            
            sendMessage() {
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
                    userId: this.getUserId(),
                    sessionId: this.getSessionId()
                };
                
                try {
                    this.ws.send(JSON.stringify(message));
                    
                    // Отображение собственного сообщения
                    this.displayMessage({ ...message, isOwn: true }, true);
                    
                    this.messageInput.value = '';
                    
                } catch (error) {
                    console.error('Ошибка отправки сообщения:', error);
                    this.showSecurityError('Ошибка отправки сообщения');
                }
            }
            
            isMessageContentSafe(content) {
                if (typeof content !== 'string') return true;
                
                const maliciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return !maliciousPatterns.some(pattern => pattern.test(content));
            }
            
            displayMessage(message, isOwn) {
                const messageElement = document.createElement('div');
                messageElement.className = `message ${isOwn ? 'own' : 'received'}`;
                
                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageElement.innerHTML = `
                    <div class="message-header">
                        <span class="message-user">${this.escapeHTML(message.user || message.userId)}</span>
                        <span class="message-time">${time}</span>
                    </div>
                    <div class="message-content">${this.escapeHTML(message.content)}</div>
                `;
                
                this.messagesContainer.appendChild(messageElement);
                
                // Прокрутка к последнему сообщению
                this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus() {
                if (this.isConnected) {
                    this.connectionStatus.textContent = 'Подключено';
                    this.connectionStatus.className = 'connection-status connected';
                } else {
                    this.connectionStatus.textContent = 'Отключено';
                    this.connectionStatus.className = 'connection-status disconnected';
                }
            }
            
            enableInputs() {
                this.messageInput.disabled = false;
                this.sendBtn.disabled = false;
                document.getElementById('connect-btn').disabled = true;
                document.getElementById('disconnect-btn').disabled = false;
            }
            
            disableInputs() {
                this.messageInput.disabled = true;
                this.sendBtn.disabled = true;
                document.getElementById('connect-btn').disabled = false;
                document.getElementById('disconnect-btn').disabled = true;
            }
            
            disconnectSecurely() {
                if (this.ws) {
                    this.ws.close(1000, 'Нормальное закрытие соединения');
                }
            }
            
            showSecurityError(message) {
                const errorDiv = document.createElement('div');
                errorDiv.className = 'security-error';
                errorDiv.textContent = message;
                
                this.messagesContainer.prepend(errorDiv);
                
                // Автоматически удаляем ошибку через 5 секунд
                setTimeout(() => {
                    if (errorDiv.parentNode) {
                        errorDiv.remove();
                    }
                }, 5000);
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
            
            getUserId() {
                // В реальном приложении ID пользователя должен быть получен из сессии
                return localStorage.getItem('userId') || 'anonymous';
            }
            
            getSessionId() {
                // В реальном приложении ID сессии должен быть получен из сессии
                return sessionStorage.getItem('sessionId') || 'session_' + Date.now();
            }
        }
        
        // Инициализация (используем безопасный URL)
        const secureWS = new SecureWebSocketClient('wss://secure-chat.example.com/ws');
    </script>
    
    <style>
        .websocket-security-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .connection-controls {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        .connection-controls button {
            padding: 10px 20px;
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
        
        .connection-status {
            padding: 8px 12px;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .connection-status.connected {
            background-color: #d4edda;
            color: #155724;
        }
        
        .connection-status.disconnected {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        .message-controls {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        #message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        #send-btn {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
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
        
        .message.received {
            background-color: #f1f8e9;
        }
        
        .message-header {
            display: flex;
            justify-content: space-between;
            font-size: 0.8em;
            margin-bottom: 5px;
        }
        
        .message-user {
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

## Практические примеры и шаблоны

### Шаблон безопасной формы с многоуровневой защитой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123def456';
                   style-src 'self' 'unsafe-inline';
                   img-src 'self' data: https:;
                   connect-src 'self' https://api.trusted.com;
                   frame-ancestors 'none';
                   object-src 'none';
                   base-uri 'self';
                   form-action 'self';
                   upgrade-insecure-requests;">
    <title>Безопасная форма с многоуровневой защитой</title>
</head>
<body>
    <h1>Безопасная форма с многоуровневой защитой</h1>
    
    <form id="multi-layer-secure-form" novalidate>
        <!-- Уровень 1: CSRF защита -->
        <input type="hidden" name="csrf_token" value="GENERATED_CSRF_TOKEN_HERE">
        
        <!-- Уровень 2: XSS защита -->
        <div class="form-group">
            <label for="secure-name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="text"
                   id="secure-name"
                   name="name"
                   required
                   minlength="2"
                   maxlength="50"
                   pattern="[A-Za-zА-Яа-яЁё\s\-]+"
                   autocomplete="name"
                   aria-describedby="name-help name-error"
                   aria-invalid="false">
            <div id="name-help" class="help-text">Только буквы, пробелы и дефисы</div>
            <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="email"
                   id="secure-email"
                   name="email"
                   required
                   autocomplete="email"
                   aria-describedby="email-help email-error"
                   aria-invalid="false">
            <div id="email-help" class="help-text">Введите действительный email адрес</div>
            <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="password"
                   id="secure-password"
                   name="password"
                   required
                   minlength="8"
                   aria-describedby="password-requirements password-error"
                   aria-invalid="false">
            <ul id="password-requirements" class="requirements-list">
                <li id="req-length">Не менее 8 символов</li>
                <li id="req-upper">С заглавной буквой</li>
                <li id="req-lower">Со строчной буквой</li>
                <li id="req-number">С цифрой</li>
                <li id="req-special">Со специальным символом</li>
            </ul>
            <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-confirm">Подтверждение пароля: <span class="required" aria-label="обязательное поле">*</span></label>
            <input type="password"
                   id="secure-confirm"
                   name="confirm_password"
                   required
                   aria-describedby="confirm-error"
                   aria-invalid="false">
            <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-bio">О себе:</label>
            <textarea id="secure-bio"
                      name="bio"
                      rows="4"
                      maxlength="500"
                      aria-describedby="bio-help bio-error"></textarea>
            <div id="bio-help" class="help-text">Расскажите немного о себе (максимум 500 символов)</div>
            <div id="bio-error" class="error-message" role="alert" aria-live="polite"></div>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="newsletter"
                       value="yes">
                Подписаться на рассылку
            </label>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="terms"
                       required
                       aria-describedby="terms-description">
                Я согласен с условиями использования <span class="required" aria-label="обязательное поле">*</span>
            </label>
            <div id="terms-description" class="help-text">Обязательное поле для регистрации</div>
        </div>
        
        <button type="submit">Зарегистрироваться</button>
    </form>

    <script nonce="abc123def456">
        class MultiLayerSecurityForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.securityLayers = [];
                this.setupSecurityLayers();
                this.setupValidation();
            }
            
            setupSecurityLayers() {
                // Уровень 1: Защита от XSS
                this.securityLayers.push(new XSSProtectionLayer());
                
                // Уровень 2: Защита от CSRF
                this.securityLayers.push(new CSRFProtectionLayer());
                
                // Уровень 3: Защита от brute force
                this.securityLayers.push(new RateLimitingLayer());
                
                // Уровень 4: Проверка целостности данных
                this.securityLayers.push(new DataIntegrityLayer());
                
                // Уровень 5: Проверка безопасности ввода
                this.securityLayers.push(new InputSecurityLayer());
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                });
                
                // Валидация при потере фокуса
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input, textarea, select')) {
                        this.validateField(e.target);
                    }
                }, true);
            }
            
            validateAndSubmit() {
                // Сброс ошибок
                this.clearAllErrors();
                
                let isValid = true;
                
                // Проверка всех полей
                const fields = this.form.querySelectorAll('input[required], textarea[required]');
                for (const field of fields) {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                }
                
                // Проверка сложных требований
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }
                
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }
                
                // Проверка всех уровней безопасности
                for (const layer of this.securityLayers) {
                    if (!layer.validate(this.form)) {
                        isValid = false;
                    }
                }
                
                if (isValid) {
                    this.submitSecurely();
                } else {
                    this.handleValidationErrors();
                }
            }
            
            validateField(field) {
                const fieldName = field.name.replace(/-/g, '');
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showFieldError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                if (field.pattern && field.value && !new RegExp(field.pattern).test(field.value)) {
                    this.showFieldError(field, errorElement, field.title || 'Неверный формат данных');
                    return false;
                }
                
                // Проверка на опасный контент
                if (this.containsMaliciousContent(field.value)) {
                    this.showFieldError(field, errorElement, 'Поле содержит недопустимый контент');
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            validatePasswordRequirements() {
                const password = document.getElementById('secure-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                // Обновление индикаторов требований
                Object.entries(requirements).forEach(([key, met]) => {
                    const element = document.getElementById(`req-${key}`);
                    if (element) {
                        element.classList.toggle('met', met);
                    }
                });
                
                const allMet = Object.values(requirements).every(req => req);
                
                if (!allMet && password) {
                    const errorElement = document.getElementById('password-error');
                    this.showFieldError(document.getElementById('secure-password'), errorElement, 'Пароль не соответствует требованиям');
                    return false;
                }
                
                return allMet;
            }
            
            validatePasswordMatch() {
                const password = document.getElementById('secure-password').value;
                const confirmPassword = document.getElementById('secure-confirm').value;
                const errorElement = document.getElementById('confirm-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    this.showFieldError(document.getElementById('secure-confirm'), errorElement, 'Пароли не совпадают');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            showFieldError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Устанавливаем фокус на поле с ошибкой
                if (!field.matches(':focus')) {
                    field.focus();
                }
            }
            
            clearAllErrors() {
                this.form.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                this.form.querySelectorAll('input, textarea, select').forEach(field => {
                    field.classList.remove('invalid', 'valid');
                    field.setAttribute('aria-invalid', 'false');
                });
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            containsMaliciousContent(content) {
                if (typeof content !== 'string') return false;
                
                const maliciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return maliciousPatterns.some(pattern => pattern.test(content));
            }
            
            async submitSecurely() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор и санитизация данных
                    const formData = new FormData(this.form);
                    const sanitizedData = {};
                    
                    for (const [key, value] of formData.entries()) {
                        if (key !== 'csrf_token') { // Исключаем токен из данных
                            sanitizedData[key] = this.sanitizeInput(value);
                        }
                    }
                    
                    // Добавление дополнительных данных безопасности
                    sanitizedData.clientSecurity = {
                        timestamp: Date.now(),
                        userAgent: navigator.userAgent,
                        language: navigator.language
                    };
                    
                    const csrfToken = this.form.querySelector('input[name="csrf_token"]').value;
                    
                    const response = await fetch('/api/secure-register', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-CSRF-Token': csrfToken,
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Регистрация прошла успешно!');
                        this.form.reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка регистрации: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
            
            handleValidationErrors() {
                const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
                
                // Объявление для скринридеров
                this.announceToScreenReader(`Форма содержит ${errorCount} ошибок. Пожалуйста, исправьте их перед отправкой.`);
                
                // Установка фокуса на первое поле с ошибкой
                const firstErrorField = this.form.querySelector('.invalid');
                if (firstErrorField) {
                    firstErrorField.focus();
                }
            }
            
            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;
                
                document.body.appendChild(announcement);
                
                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 3000);
            }
        }
        
        // Классы уровней безопасности
        class XSSProtectionLayer {
            validate(form) {
                const inputs = form.querySelectorAll('input, textarea');
                for (const input of inputs) {
                    if (this.containsXSS(input.value)) {
                        console.warn('Обнаружен потенциальный XSS в поле:', input.name || input.id);
                        return false;
                    }
                }
                return true;
            }
            
            containsXSS(value) {
                if (typeof value !== 'string') return false;
                
                const xssPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];
                
                return xssPatterns.some(pattern => pattern.test(value));
            }
        }
        
        class CSRFProtectionLayer {
            validate(form) {
                const token = form.querySelector('input[name="csrf_token"]').value;
                if (!token || token.length < 32) {
                    console.warn('Неверный CSRF токен');
                    return false;
                }
                return true;
            }
        }
        
        class RateLimitingLayer {
            constructor() {
                this.attempts = 0;
                this.lastAttempt = 0;
                this.maxAttempts = 5;
                this.cooldown = 60000; // 1 минута
            }
            
            validate(form) {
                const now = Date.now();
                
                if (now - this.lastAttempt < this.cooldown) {
                    if (this.attempts >= this.maxAttempts) {
                        console.warn('Превышено количество попыток');
                        return false;
                    }
                } else {
                    this.attempts = 0;
                }
                
                this.attempts++;
                this.lastAttempt = now;
                
                return true;
            }
        }
        
        class DataIntegrityLayer {
            validate(form) {
                // Проверка целостности данных
                const formData = new FormData(form);
                
                for (const [key, value] of formData.entries()) {
                    if (typeof value === 'string' && this.hasDataIntegrityIssues(value)) {
                        console.warn('Нарушена целостность данных в поле:', key);
                        return false;
                    }
                }
                
                return true;
            }
            
            hasDataIntegrityIssues(value) {
                // Проверка на подозрительные паттерны
                return /(?:\b(?:exec|select|insert|update|delete|drop|create|alter|grant|revoke)\b)/i.test(value);
            }
        }
        
        class InputSecurityLayer {
            validate(form) {
                const inputs = form.querySelectorAll('input, textarea');
                for (const input of inputs) {
                    if (this.hasSecurityIssues(input.value)) {
                        console.warn('Обнаружены проблемы безопасности в поле:', input.name || input.id);
                        return false;
                    }
                }
                return true;
            }
            
            hasSecurityIssues(value) {
                if (typeof value !== 'string') return false;
                
                const securityPatterns = [
                    /<svg/i,
                    /<img[^>]*onload/i,
                    /<img[^>]*onerror/i,
                    /<link/i,
                    /<meta/i,
                    /document\.cookie/i,
                    /localStorage/i,
                    /sessionStorage/i
                ];
                
                return securityPatterns.some(pattern => pattern.test(value));
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MultiLayerSecurityForm('multi-layer-secure-form');
        });
    </script>
    
    <style>
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
        
        input:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.valid {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }
        
        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }
        
        .requirements-list li.met {
            color: #28a745;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .required {
            color: #d32f2f;
        }
        
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

### Безопасные элементы с веб-компонентами и Shadow DOM

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасные веб-компоненты с Shadow DOM</title>
</head>
<body>
    <h1>Безопасные веб-компоненты с Shadow DOM</h1>
    
    <secure-contact-form 
        title="Безопасная контактная форма"
        action="/api/contact">
    </secure-contact-form>
    
    <data-table-component 
        id="secure-data-table"
        caption="Таблица пользователей с безопасностью"
        role="table"
        aria-label="Безопасная таблица данных">
    </data-table-component>

    <script>
        // Безопасная контактная форма как веб-компонент
        class SecureContactForm extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'closed' }); // Используем closed для дополнительной безопасности
                
                this.render();
                this.setupSecurity();
            }
            
            static get observedAttributes() {
                return ['title', 'action'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    this[name] = newValue;
                    this.render();
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            font-family: Arial, sans-serif;
                        }
                        
                        .form-container {
                            max-width: 500px;
                            margin: 20px auto;
                            padding: 20px;
                            border: 1px solid #ddd;
                            border-radius: 8px;
                            background-color: white;
                            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                        }
                        
                        .form-title {
                            margin-top: 0;
                            color: #333;
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
                        
                        input:focus, textarea:focus {
                            outline: none;
                            border-color: #007acc;
                            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
                        }
                        
                        .error-message {
                            color: #dc3545;
                            font-size: 0.875rem;
                            margin-top: 0.25rem;
                            display: block;
                        }
                        
                        .submit-button {
                            background-color: #007acc;
                            color: white;
                            padding: 0.75rem 1.5rem;
                            border: none;
                            border-radius: 4px;
                            cursor: pointer;
                            font-size: 1rem;
                        }
                        
                        .submit-button:disabled {
                            background-color: #ccc;
                            cursor: not-allowed;
                        }
                    </style>
                    
                    <form class="secure-form" id="contact-form" novalidate>
                        <h2 class="form-title">${this.escapeHTML(this.getAttribute('title') || 'Контактная форма')}</h2>
                        
                        <div class="form-group">
                            <label for="name-input">Имя:</label>
                            <input type="text"
                                   id="name-input"
                                   name="name"
                                   required
                                   minlength="2"
                                   maxlength="50"
                                   aria-describedby="name-error"
                                   aria-invalid="false">
                            <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
                        </div>
                        
                        <div class="form-group">
                            <label for="email-input">Email:</label>
                            <input type="email"
                                   id="email-input"
                                   name="email"
                                   required
                                   aria-describedby="email-error"
                                   aria-invalid="false">
                            <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                        </div>
                        
                        <div class="form-group">
                            <label for="subject-input">Тема:</label>
                            <input type="text"
                                   id="subject-input"
                                   name="subject"
                                   required
                                   minlength="5"
                                   maxlength="100"
                                   aria-describedby="subject-error"
                                   aria-invalid="false">
                            <div id="subject-error" class="error-message" role="alert" aria-live="assertive"></div>
                        </div>
                        
                        <div class="form-group">
                            <label for="message-input">Сообщение:</label>
                            <textarea id="message-input"
                                      name="message"
                                      rows="5"
                                      required
                                      minlength="10"
                                      maxlength="1000"
                                      aria-describedby="message-error"
                                      aria-invalid="false"></textarea>
                            <div id="message-error" class="error-message" role="alert" aria-live="assertive"></div>
                        </div>
                        
                        <button type="submit" class="submit-button">Отправить</button>
                    </form>
                `;
            }
            
            setupSecurity() {
                const form = this.shadowRoot.getElementById('contact-form');
                
                form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                });
                
                // Валидация при потере фокуса
                form.addEventListener('blur', (e) => {
                    if (e.target.matches('input, textarea, select')) {
                        this.validateField(e.target);
                    }
                }, true);
            }
            
            validateAndSubmit() {
                // Сброс ошибок
                this.clearAllErrors();
                
                let isValid = true;
                
                // Проверка всех обязательных полей
                const fields = this.shadowRoot.querySelectorAll('input[required], textarea[required]');
                fields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });
                
                if (isValid) {
                    this.submitSecurely();
                }
            }
            
            validateField(field) {
                const fieldName = field.id.replace(/-input$/, '');
                const errorElement = this.shadowRoot.getElementById(`${fieldName}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showFieldError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                // Проверка на опасный контент
                if (this.containsMaliciousContent(field.value)) {
                    this.showFieldError(field, errorElement, 'Поле содержит недопустимый контент');
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            showFieldError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Устанавливаем фокус на поле с ошибкой
                if (!field.matches(':focus')) {
                    field.focus();
                }
            }
            
            clearAllErrors() {
                this.shadowRoot.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                this.shadowRoot.querySelectorAll('input, textarea, select').forEach(field => {
                    field.classList.remove('invalid', 'valid');
                    field.setAttribute('aria-invalid', 'false');
                });
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            containsMaliciousContent(content) {
                if (typeof content !== 'string') return false;
                
                const maliciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return maliciousPatterns.some(pattern => pattern.test(content));
            }
            
            async submitSecurely() {
                const submitBtn = this.shadowRoot.querySelector('.submit-button');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор и санитизация данных
                    const formData = new FormData(this.shadowRoot.getElementById('contact-form'));
                    const sanitizedData = {};
                    
                    for (const [key, value] of formData.entries()) {
                        sanitizedData[key] = this.sanitizeInput(value);
                    }
                    
                    // Добавление CSRF токена
                    const csrfToken = this.getCSRFToken();
                    
                    const action = this.getAttribute('action') || '/api/contact';
                    
                    const response = await fetch(action, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-CSRF-Token': csrfToken,
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Сообщение успешно отправлено!');
                        this.shadowRoot.getElementById('contact-form').reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка отправки: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
            
            getCSRFToken() {
                // В реальном приложении токен должен быть получен из безопасного источника
                return localStorage.getItem('csrf-token') || 'default-csrf-token';
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
        }
        
        // Безопасная таблица данных
        class SecureDataTable extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                this.data = [];
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['caption', 'data'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'data') {
                        this.data = JSON.parse(newValue);
                        this.renderTable();
                    }
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            font-family: Arial, sans-serif;
                        }
                        
                        table {
                            width: 100%;
                            border-collapse: collapse;
                            margin: 1rem 0;
                        }
                        
                        th, td {
                            padding: 0.75rem;
                            text-align: left;
                            border-bottom: 1px solid #ddd;
                        }
                        
                        th {
                            background-color: #f8f9fa;
                            font-weight: bold;
                            position: sticky;
                            top: 0;
                        }
                        
                        tr:hover {
                            background-color: #f5f5f5;
                        }
                        
                        .loading {
                            text-align: center;
                            padding: 2rem;
                            color: #666;
                        }
                        
                        .error {
                            color: #d32f2f;
                            background-color: #ffebee;
                            padding: 1rem;
                            border-radius: 4px;
                        }
                    </style>
                    
                    <table role="table" aria-label="${this.escapeHTML(this.getAttribute('aria-label') || 'Таблица данных')}">
                        <caption class="sr-only">${this.escapeHTML(this.getAttribute('caption') || 'Таблица данных')}</caption>
                        <thead id="table-header" role="rowgroup"></thead>
                        <tbody id="table-body" role="rowgroup"></tbody>
                    </table>
                `;
            }
            
            setData(data) {
                this.data = data;
                this.renderTable();
            }
            
            renderTable() {
                if (!this.data || this.data.length === 0) {
                    this.shadowRoot.getElementById('table-body').innerHTML = 
                        '<tr role="row"><td role="cell" colspan="100%" style="text-align: center;">Нет данных</td></tr>';
                    return;
                }
                
                // Определение колонок
                const columns = Object.keys(this.data[0]);
                
                // Рендер заголовков
                const headerRow = document.createElement('tr');
                headerRow.setAttribute('role', 'row');
                
                columns.forEach(col => {
                    const th = document.createElement('th');
                    th.setAttribute('role', 'columnheader');
                    th.setAttribute('scope', 'col');
                    th.textContent = this.formatColumnName(col);
                    headerRow.appendChild(th);
                });
                
                this.shadowRoot.getElementById('table-header').innerHTML = '';
                this.shadowRoot.getElementById('table-header').appendChild(headerRow);
                
                // Рендер тела таблицы
                const body = this.shadowRoot.getElementById('table-body');
                body.innerHTML = '';
                
                this.data.forEach((row, index) => {
                    const tr = document.createElement('tr');
                    tr.setAttribute('role', 'row');
                    
                    columns.forEach(col => {
                        const td = document.createElement('td');
                        td.setAttribute('role', 'cell');
                        
                        // Санитизация содержимого
                        const safeContent = this.sanitizeInput(row[col]);
                        td.textContent = safeContent;
                        
                        tr.appendChild(td);
                    });
                    
                    body.appendChild(tr);
                });
            }
            
            formatColumnName(name) {
                return name
                    .replace(/([A-Z])/g, ' $1')
                    .replace(/^./, str => str.toUpperCase());
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
        }
        
        // Регистрация элементов
        customElements.define('secure-contact-form', SecureContactForm);
        customElements.define('data-table-component', SecureDataTable);
        
        // Пример использования
        document.addEventListener('DOMContentLoaded', () => {
            // Загрузка данных для таблицы
            const table = document.getElementById('secure-data-table');
            const sampleData = [
                { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', role: 'Администратор' },
                { id: 2, name: 'Мария Петрова', email: 'maria@example.com', role: 'Модератор' },
                { id: 3, name: 'Алексей Сидоров', email: 'alexey@example.com', role: 'Пользователь' }
            ];
            
            table.setData(sampleData);
        });
    </script>
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
9. **axe-core** - проверка безопасности в контексте доступности
10. **Lighthouse** - комплексная проверка безопасности и производительности

### Проверка XSS уязвимостей:
1. Проверка всех мест вывода пользовательского контента
2. Проверка использования innerHTML вместо textContent
3. Проверка экранирования специальных символов
4. Проверка использования безопасных методов вставки HTML
5. Проверка санитизации пользовательского ввода
6. Проверка использования Trusted Types API
7. Проверка Content Security Policy
8. Проверка Subresource Integrity

### Проверка CSRF защиты:
1. Проверка наличия CSRF токенов
2. Проверка валидации токенов на сервере
3. Проверка использования SameSite cookie атрибутов
4. Проверка заголовков Origin и Referer
5. Проверка использования double-submit cookie паттерна
6. Проверка использования encrypted token паттерна
7. Проверка использования custom headers для аутентификации

### Проверка clickjacking защиты:
1. Проверка CSP frame-ancestors директивы
2. Проверка X-Frame-Options заголовка
3. Проверка возможности встраивания страниц в iframe
4. Проверка frame-busting скриптов
5. Проверка на использование iframe с атрибутом sandbox

### Проверка на уязвимости ввода:
1. Проверка экранирования HTML
2. Проверка валидации форматов данных
3. Проверка ограничения длины полей
4. Проверка фильтрации специальных символов
5. Проверка использования регулярных выражений
6. Проверка санитизации URL
7. Проверка обработки файлов (если применимо)
8. Проверка использования безопасных атрибутов

## Лучшие практики безопасности HTML

### 1. Правильная санитизация пользовательского ввода
```html
<!-- Правильно: безопасная обработка пользовательского ввода -->
<script>
function sanitizeInput(input) {
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

// Использование
const userInput = document.getElementById('user-input').value;
const sanitizedInput = sanitizeInput(userInput);
document.getElementById('output').textContent = sanitizedInput; // Всегда используем textContent
</script>

<!-- Плохо: использование innerHTML с пользовательским вводом -->
<div id="output"></div>
<script>
// НИКОГДА не используйте innerHTML с пользовательским вводом
document.getElementById('output').innerHTML = userInput; // Потенциальная XSS угроза!
</script>
```

### 2. Использование Content Security Policy
```html
<!-- Строгая CSP политика -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self';
               script-src 'self' 'nonce-abc123';
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;
               font-src 'self' https://fonts.gstatic.com;
               connect-src 'self' https://api.trusted.com;
               frame-ancestors 'none';
               object-src 'none';
               base-uri 'self';
               form-action 'self';
               upgrade-insecure-requests;">
```

### 3. Использование Subresource Integrity
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

### 4. Защита от CSRF
```html
<!-- CSRF токен в форме -->
<form action="/submit" method="post">
    <input type="hidden" name="csrf_token" value="GENERATED_SECURE_TOKEN">
    <input type="text" name="data" required>
    <button type="submit">Отправить</button>
</form>
```

### 5. Безопасные атрибуты
```html
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

<!-- Безопасные формы -->
<form action="https://secure.example.com/api/submit" 
      method="post" 
      target="_self">
    <input type="text" name="sensitive-data">
    <button type="submit">Отправить</button>
</form>
```

### 6. Безопасные кастомные элементы
```javascript
// Правильно: безопасный кастомный элемент
class SecureCustomElement extends HTMLElement {
    constructor() {
        super();
        
        // Используем closed Shadow DOM для изоляции
        this.attachShadow({ mode: 'closed' });
        
        // Экранируем пользовательские данные
        const title = this.escapeHTML(this.getAttribute('title') || '');
        const content = this.escapeHTML(this.getAttribute('content') || '');
        
        this.shadowRoot.innerHTML = `
            <div class="secure-element">
                <h3>${title}</h3>
                <p>${content}</p>
            </div>
        `;
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
    
    static get observedAttributes() {
        return ['title', 'content'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            this.render();
        }
    }
}
```

## Заключение

Безопасность HTML-страниц - это комплексный подход, который включает в себя защиту от различных типов атак, таких как XSS, CSRF, clickjacking и других. Правильная реализация мер безопасности помогает защитить как пользователей, так и веб-приложения от потенциальных угроз.

Ключевые аспекты безопасности HTML:
1. Использование Content Security Policy для предотвращения XSS
2. Subresource Integrity для проверки целостности внешних ресурсов
3. CSRF токены для защиты от подделки запросов
4. Правильная санитизация пользовательского ввода
5. Использование семантических элементов HTML
6. ARIA атрибуты для дополнительной информации
7. Управление фокусом и клавиатурной навигацией
8. Альтернативный текст для изображений
9. Доступные формы с понятными метками
10. Контрастность цветов
11. Совместимость с вспомогательными технологиями
12. Следование стандартам безопасности
13. Регулярное тестирование и проверка безопасности
14. Использование современных API безопасности
15. Защита данных пользователей
16. Обеспечение приватности
17. Защита от манипуляций с DOM
18. Правильное использование атрибутов безопасности
19. Защита от clickjacking и других визуальных атак
20. Обеспечение безопасности веб-компонентов

Эти практики обеспечивают создание веб-сайтов, которые безопасны, доступны и удобны для всех пользователей, независимо от их возможностей или используемых технологий.

Современные подходы к безопасности включают:
- Использование Content Security Policy для ограничения источников контента
- Subresource Integrity для проверки целостности внешних ресурсов
- Trusted Types для предотвращения XSS через DOM
- Proper input sanitization and validation
- Secure WebSocket connections with message validation
- Protected IndexedDB operations
- Safe DOM manipulation practices
- Modern web component patterns with security in mind
- Testing with automated tools and manual verification
- Following security best practices and standards
- Implementation of defense-in-depth strategies
- Use of secure headers and protocols
- Proper authentication and authorization mechanisms
- Regular security audits and updates

Эти методы позволяют создавать действительно безопасные веб-приложения, которые защищают данные пользователей и предотвращают различные типы атак.

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
- [[HTML/Доступность]]
- [[HTML/Совместимость]]
- [[HTML/Производительность]]

## Теги
#html #security #web-development #frontend #xss #csrf #csp #sri #clickjacking #web-components #javascript #dom-manipulation #input-sanitization #content-security-policy #subresource-integrity #trusted-types #websockets #indexeddb #security-headers #secure-coding #best-practices #security-auditing #security-monitoring #accessibility #performance #modern-web #security-tools #security-testing #security-patterns