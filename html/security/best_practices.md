# HTML Безопасность: Лучшие практики

Лучшие практики безопасности HTML - это проверенные методы и подходы, которые помогают создавать защищенные веб-приложения, защищающие как пользователей, так и системы от потенциальных угроз. Эти практики обеспечивают соответствие современным стандартам безопасности и создают надежную защиту от различных типов атак.

## Основы безопасности HTML

### Принципы безопасности

1. **Defense in Depth** - многоуровневая защита
2. **Least Privilege** - минимальные привилегии
3. **Fail Securely** - безопасное поведение при ошибках
4. **Secure Defaults** - безопасные настройки по умолчанию
5. **Complete Mediation** - полная проверка доступа
6. **Open Design** - открытый дизайн, секреты в ключах
7. **Separation of Privileges** - разделение привилегий
8. **Economy of Mechanism** - простота механизмов
9. **Least Common Mechanism** - минимальные общие механизмы
10. **Psychological Acceptability** - удобство для пользователей

### Безопасная разметка HTML

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Content Security Policy для предотвращения XSS -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123def456';
                   style-src 'self' 'unsafe-inline';
                   img-src 'self' data: blob: https:;
                   font-src 'self' https://fonts.gstatic.com;
                   connect-src 'self' https://api.trusted.com;
                   frame-ancestors 'none';
                   object-src 'none';
                   base-uri 'self';
                   form-action 'self';
                   upgrade-insecure-requests;">
    
    <title>Безопасная HTML разметка</title>
    
    <!-- Предзагрузка критических ресурсов -->
    <link rel="preload" href="critical.css" as="style">
    <link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
    
    <!-- Предзагрузка соединения -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://api.trusted.com">
    
    <!-- Безопасный favicon -->
    <link rel="icon" type="image/svg+xml" href="favicon.svg">
    <link rel="icon" type="image/png" sizes="32x32" href="favicon-32x32.png" integrity="sha384-..." crossorigin="anonymous">
</head>
<body>
    <header role="banner">
        <h1>Безопасный сайт</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main role="main">
        <article>
            <header>
                <h2>Безопасность HTML</h2>
                <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
            </header>
            
            <section aria-labelledby="security-principles-title">
                <h3 id="security-principles-title">Принципы безопасности</h3>
                <p>Современные веб-приложения должны следовать принципам безопасности...</p>
            </section>
        </article>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Безопасный сайт. Все права защищены.</p>
    </footer>
</body>
</html>
```

## Защита от XSS (Cross-Site Scripting)

### Типы XSS атак:

1. **Reflected XSS** - отражаемый XSS
2. **Stored XSS** - хранимый XSS
3. **DOM-based XSS** - XSS на основе DOM

### Основные методы защиты от XSS

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
            <textarea id="comment" name="comment" rows="4" cols="50" placeholder="Введите ваш комментарий"></textarea>
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
                    // ПРАВИЛЬНАЯ защита от XSS - санитизация
                    const sanitizedComment = this.sanitizeHTML(commentText);
                    
                    this.addComment(sanitizedComment);
                    this.commentInput.value = '';
                }
            }
            
            sanitizeHTML(html) {
                // Создаем временный элемент для безопасной обработки
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

### Расширенная санитизация пользовательского ввода

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Расширенная санитизация</title>
</head>
<body>
    <h1>Расширенная санитизация данных</h1>
    
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
        class AdvancedContentSanitizer {
            constructor() {
                this.outputContainer = document.getElementById('safe-output');
            }
            
            // Основная функция санитизации
            sanitizeContent(content) {
                if (typeof content !== 'string') return content;
                
                // Комплексная санитизация данных
                return this.comprehensiveSanitize(content);
            }
            
            comprehensiveSanitize(content) {
                // Удаляем потенциально опасные теги
                const dangerousTags = [
                    'script', 'iframe', 'object', 'embed', 'form', 
                    'input', 'button', 'link', 'meta', 'base', 
                    'style', 'frameset', 'frame', 'framesrc', 'bgsound'
                ];
                
                const tempDiv = document.createElement('div');
                tempDiv.textContent = content; // Устанавливаем как текст, не как HTML
                
                // Удаляем потенциально опасные элементы
                dangerousTags.forEach(tag => {
                    const elements = tempDiv.querySelectorAll(tag);
                    elements.forEach(el => el.remove());
                });
                
                // Удаляем потенциально опасные атрибуты
                const allElements = tempDiv.querySelectorAll('*');
                allElements.forEach(el => {
                    const dangerousAttrs = ['onclick', 'onload', 'onerror', 'onmouseover', 
                                           'onfocus', 'onblur', 'onsubmit', 'onchange', 
                                           'onkeypress', 'onkeydown', 'onkeyup'];
                    dangerousAttrs.forEach(attr => {
                        if (el.hasAttribute(attr)) {
                            el.removeAttribute(attr);
                        }
                    });
                });
                
                return tempDiv.textContent; // Возвращаем как текст, не как HTML
            }
            
            // Санитизация URL
            sanitizeURL(url) {
                try {
                    const parsedUrl = new URL(url, window.location.origin);
                    
                    // Проверка протокола
                    const allowedProtocols = ['http:', 'https:', 'mailto:', 'tel:', 'ftp:'];
                    if (!allowedProtocols.includes(parsedUrl.protocol)) {
                        throw new Error('Неподдерживаемый протокол');
                    }
                    
                    // Проверка домена
                    const allowedDomains = [
                        window.location.hostname,
                        'trusted-site.com',
                        'secure-api.com',
                        'cdn.example.com'
                    ];
                    
                    if (!allowedDomains.includes(parsedUrl.hostname) && 
                        !allowedDomains.some(domain => parsedUrl.hostname.endsWith('.' + domain))) {
                        throw new Error('Недопустимый домен');
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
                    /behavior:/i,
                    /include-source/i,
                    /binding/i
                ];
                
                if (dangerousCSS.some(pattern => pattern.test(cssText))) {
                    throw new Error('Обнаружены потенциально опасные CSS свойства');
                }
                
                return cssText;
            }
            
            // Санитизация с использованием DOMParser
            sanitizeWithDOMParser(content) {
                const parser = new DOMParser();
                const doc = parser.parseFromString(content, 'text/html');
                
                // Удаляем потенциально опасные элементы
                const dangerousElements = doc.querySelectorAll('script, iframe, object, embed, form');
                dangerousElements.forEach(el => el.remove());
                
                // Удаляем опасные атрибуты
                const allElements = doc.querySelectorAll('*');
                allElements.forEach(el => {
                    const dangerousAttrs = ['onclick', 'onload', 'onerror', 'onmouseover', 'onfocus', 'onblur'];
                    dangerousAttrs.forEach(attr => {
                        if (el.hasAttribute(attr)) {
                            el.removeAttribute(attr);
                        }
                    });
                });
                
                return doc.body.textContent;
            }
            
            // Санитизация с использованием регулярных выражений
            sanitizeWithRegex(content) {
                // Удаляем script теги
                let sanitized = content.replace(/<script[^>]*>[\s\S]*?<\/script>/gi, '');
                
                // Удаляем iframe теги
                sanitized = sanitized.replace(/<iframe[^>]*>[\s\S]*?<\/iframe>/gi, '');
                
                // Удаляем события
                sanitized = sanitized.replace(/on\w+\s*=\s*["'][^"']*["']/gi, '');
                
                // Экранируем HTML
                sanitized = sanitized
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;');
                
                return sanitized;
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
        
        const sanitizer = new AdvancedContentSanitizer();
        
        function sanitizeAndDisplay() {
            const userInput = document.getElementById('unsafe-input').value;
            sanitizer.displayContent(userInput, false);
        }
        
        // Примеры опасного контента для тестирования
        const dangerousExamples = [
            '<script>alert("XSS")</script>',
            '<img src="x" onerror="alert(\'XSS\')">',
            '<a href="javascript:alert(\'XSS\')">Кликни меня</a>',
            '<div onclick="alert(\'XSS\')">Кликабельный div</div>',
            '<iframe src="javascript:alert(\'XSS\')"></iframe>',
            '<svg onload="alert(\'XSS\')"></svg>',
            '<img src="x" onload="alert(\'XSS\')">'
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

### CSP для различных сценариев

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Content Security Policy</title>
    
    <!-- Строгая CSP политика для production -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123def456' https://analytics.example.com;
                   style-src 'self' 'nonce-ghi789' https://fonts.googleapis.com;
                   img-src 'self' data: blob: https:;
                   font-src 'self' https://fonts.gstatic.com;
                   connect-src 'self' https://api.trusted.com https://api.example.com;
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
                   font-src 'self' 'unsafe-inline' https://fonts.gstatic.com;
                   connect-src 'self' http: https:;">
    
    <!-- CSP Report Only для мониторинга -->
    <meta http-equiv="Content-Security-Policy-Report-Only" 
          content="default-src 'self';
                   report-uri /csp-report;">
    
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
</head>
<body>
    <h1>Content Security Policy</h1>

    <div class="content">
        <p>Эта страница использует Content Security Policy для защиты от XSS атак.</p>
        
        <!-- Встроенные стили с nonce -->
        <style nonce="ghi789">
            .content {
                color: #007acc;
            }
        </style>
        
        <!-- Встроенный скрипт с nonce -->
        <script nonce="abc123def456">
            // Этот скрипт будет работать благодаря nonce
            console.log('CSP тест: скрипт выполнен');
        </script>
    </div>

    <script>
        class CSPManager {
            constructor() {
                this.setupCSPMonitoring();
                this.checkCSPSupport();
            }
            
            setupCSPMonitoring() {
                // Мониторинг нарушений CSP
                document.addEventListener('securitypolicyviolation', (e) => {
                    console.error('Нарушение CSP:', {
                        blockedURI: e.blockedURI,
                        violatedDirective: e.violatedDirective,
                        originalPolicy: e.originalPolicy,
                        sourceFile: e.sourceFile,
                        lineno: e.lineno,
                        colno: e.colno
                    });
                    
                    // Отправка отчета о нарушении CSP
                    this.reportCSPViolation(e);
                });
            }
            
            checkCSPSupport() {
                if (window.isSecureContext) {
                    console.log('Безопасный контекст (HTTPS или localhost)');
                } else {
                    console.warn('Не безопасный контекст - CSP может работать не полностью');
                }
            }
            
            reportCSPViolation(event) {
                // Отправка отчета о нарушении CSP
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/csp-report', JSON.stringify({
                        blockedURI: event.blockedURI,
                        violatedDirective: event.violatedDirective,
                        originalPolicy: event.originalPolicy,
                        sourceFile: event.sourceFile,
                        lineno: event.lineno,
                        colno: event.colno,
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
                            sourceFile: event.sourceFile,
                            lineno: event.lineno,
                            colno: event.colno,
                            timestamp: new Date().toISOString()
                        }),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
            
            // Динамическое добавление скриптов с CSP
            addSecureScript(src, integrity, nonce) {
                return new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = src;
                    script.integrity = integrity;
                    script.crossOrigin = 'anonymous';
                    
                    if (nonce) {
                        script.nonce = nonce;
                    }
                    
                    script.onload = () => resolve();
                    script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
                    
                    document.head.appendChild(script);
                });
            }
            
            // Динамическое добавление стилей с CSP
            addSecureStylesheet(href, integrity, nonce) {
                return new Promise((resolve, reject) => {
                    const link = document.createElement('link');
                    link.rel = 'stylesheet';
                    link.href = href;
                    link.integrity = integrity;
                    link.crossOrigin = 'anonymous';
                    
                    if (nonce) {
                        link.setAttribute('nonce', nonce);
                    }
                    
                    link.onload = () => resolve();
                    link.onerror = () => reject(new Error(`Failed to load stylesheet: ${href}`));
                    
                    document.head.appendChild(link);
                });
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

### CSP для Web Components

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSP для Web Components</title>
    
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'unsafe-eval'; /* Необходимо для Web Components */
                   style-src 'self' 'unsafe-inline';
                   img-src 'self' data: https:;
                   connect-src 'self' https://api.trusted.com;
                   frame-ancestors 'none';
                   object-src 'none';
                   base-uri 'self';
                   form-action 'self';">
</head>
<body>
    <h1>CSP для Web Components</h1>
    
    <secure-user-card 
        username="john_doe" 
        email="john@example.com" 
        avatar="avatar.jpg">
    </secure-user-card>

    <script>
        // Безопасный кастомный элемент с CSP
        class SecureUserCard extends HTMLElement {
            constructor() {
                super();
                
                // Используем closed Shadow DOM для дополнительной безопасности
                this.attachShadow({ mode: 'closed' });
                
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
                            <button class="action-btn message-btn" @click="${this.handleMessage}">Сообщение</button>
                            <button class="action-btn follow-btn" @click="${this.handleFollow}">Подписаться</button>
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
        
        // Регистрация элемента
        customElements.define('secure-user-card', SecureUserCard);
    </script>
</body>
</html>
```

## CSRF (Cross-Site Request Forgery) защита

### CSRF токены и другие методы защиты

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
        <input type="hidden" name="csrf_token" value="GENERATED_SECURE_CSRF_TOKEN">
        
        <!-- Дополнительные данные безопасности -->
        <input type="hidden" name="timestamp" value="CURRENT_TIMESTAMP">
        <input type="hidden" name="user_agent" value="USER_AGENT_HASH">
        
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
                
                // Добавляем проверку Origin и Referer заголовков (на сервере)
                this.addSecurityHeaders();
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
                return token.length > 32 && /^[A-Za-z0-9-_]+$/.test(token);
            }
            
            addSecurityHeaders() {
                // Эти заголовки добавляются на сервере, но можно использовать для информации
                console.log('CSRF защита: заголовки безопасности добавлены');
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
            
            // Создание безопасного токена (в реальном приложении - на сервере)
            generateSecureToken() {
                const array = new Uint8Array(32);
                crypto.getRandomValues(array);
                return btoa(String.fromCharCode(...array));
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
                this.setupFrameBusting();
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
            
            setupFrameBusting() {
                // Frame busting скрипт (резервный метод)
                if (window.self !== window.top) {
                    try {
                        window.top.location = window.location;
                    } catch (e) {
                        // Если не можем изменить top.location, используем другую стратегию
                        document.body.style.display = 'none';
                    }
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
                    'https://trusted-site.com',
                    'https://partner-site.com'
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
            
            // Проверка целостности содержимого
            verifyContentIntegrity(content, expectedHash) {
                return this.generateSRIHash(content).then(actualHash => {
                    return actualHash === expectedHash;
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

### Trusted Types API

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
    
    <div class="content">
        <p>Эта страница использует Trusted Types для защиты от XSS.</p>
        
        <div id="dynamic-content-container">
            <button onclick="loadDynamicContent()">Загрузить динамический контент</button>
        </div>
    </div>

    <script>
        class TrustedTypesManager {
            constructor() {
                this.container = document.getElementById('dynamic-content-container');
                this.policy = null;
                
                this.setupTrustedTypes();
            }
            
            setupTrustedTypes() {
                if (window.trustedTypes && window.trustedTypes.createPolicy) {
                    // Создаем политику Trusted Types
                    this.policy = window.trustedTypes.createPolicy('htmlPolicy', {
                        createHTML: (string, ...args) => {
                            // Санитизация HTML перед вставкой
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
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
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
            
            loadDynamicContent() {
                const dangerousContent = '<p>Опасный контент <script>alert("XSS!")</script></p>';
                const safeContent = '<p>Безопасный контент без вредоносных скриптов</p>';
                
                if (this.policy) {
                    // Используем безопасный способ вставки HTML
                    this.container.innerHTML = this.policy.createHTML(safeContent);
                    
                    // Попытка вставить опасный контент будет заблокирована
                    try {
                        // Это вызовет ошибку в браузере с включенным Trusted Types
                        // this.container.innerHTML = this.policy.createHTML(dangerousContent);
                    } catch (error) {
                        console.error('Trusted Types предотвратил вставку опасного контента:', error);
                    }
                } else {
                    // Резервный способ для браузеров без Trusted Types
                    this.container.innerHTML = this.sanitizeHTML(safeContent);
                }
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
            
            connectedCallback() {
                const formId = this.getAttribute('form-id');
                if (formId) {
                    this.setupFormValidation(formId);
                }
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
                        if (e.target.matches('input[required], textarea[required], select[required]')) {
                            this.validateField(e.target);
                        }
                    });
                }
            }
            
            validateForm() {
                let isValid = true;
                const errors = [];
                
                const requiredFields = this.form.querySelectorAll('input[required], textarea[required], select[required]');
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
                
                // Проверка на опасный контент
                if (this.containsMaliciousContent(field.value)) {
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
                this.retryDelay = 3000;
                
                this.messagesContainer = document.getElementById('messages-container');
                this.connectionStatus = document.getElementById('connection-status');
                this.messageInput = document.getElementById('message-input');
                this.sendBtn = document.getElementById('send-btn');
                
                this.setupSecureConnection();
            }
            
            setupSecureConnection() {
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
                    if (e.key === 'Enter' && !this.messageInput.disabled) {
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
                    
                    // Создание безопасного соединения
                    this.ws = new WebSocket(this.url);
                    
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.connectionAttempts = 0;
                        this.updateConnectionStatus('Подключено', 'connected');
                        this.enableInputs();
                    };
                    
                    this.ws.onmessage = (event) => {
                        this.handleSecureMessage(event.data);
                    };
                    
                    this.ws.onclose = (event) => {
                        this.isConnected = false;
                        this.updateConnectionStatus('Отключено', 'disconnected');
                        this.disableInputs();
                        
                        // Автоматическое переподключение
                        if (event.code !== 1000) { // Нормальное закрытие
                            this.connectionAttempts++;
                            setTimeout(() => {
                                if (this.connectionAttempts < this.maxConnectionAttempts) {
                                    this.connectSecurely();
                                }
                            }, this.retryDelay);
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
                        'secure-chat.example.com',
                        'api.trusted.com'
                    ];
                    
                    if (!this.isLocalhost(url)) {
                        return allowedDomains.includes(parsedUrl.hostname) || 
                               allowedDomains.some(domain => parsedUrl.hostname.endsWith('.' + domain));
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
                const requiredFields = ['id', 'timestamp', 'content', 'user'];
                return requiredFields.every(field => field in message);
            }
            
            sanitizeMessageContent(message) {
                // Санитизация содержимого сообщения
                return {
                    ...message,
                    content: this.sanitizeInput(message.content),
                    user: this.sanitizeInput(message.user || 'Unknown')
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
                    .replace(/\//g, '&#x2F;')
                    .trim();
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
                    userId: this.getUserId(),
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
            
            displayMessage(message) {
                const messageElement = document.createElement('div');
                messageElement.className = `message ${message.isOwn ? 'own' : 'received'}`;
                
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
            
            updateConnectionStatus(status, type) {
                this.connectionStatus.textContent = status;
                this.connectionStatus.className = `connection-status ${type}`;
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
                    this.ws.close(1000, 'Нормальное закрытие');
                }
            }
            
            showSecurityError(message) {
                const errorDiv = document.createElement('div');
                errorDiv.className = 'security-error';
                errorDiv.textContent = message;
                
                this.messagesContainer.prepend(errorDiv);
                
                // Автоматически удаляем ошибку через 5 секунд
                setTimeout(() => {
                    if (this.messagesContainer.contains(errorDiv)) {
                        this.messagesContainer.removeChild(errorDiv);
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
        
        .connection-status.error {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        #message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        #send-btn {
            background-color: #007acc;
            color: white;
            border: none;
            padding: 10px 20px;
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
            <div id="name-help" class="help-text">Только буквы, пробелы и дефисы, 2-50 символов</div>
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
            
            async validateAndSubmit() {
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
                
                // Проверка всех уровней безопасности
                for (const layer of this.securityLayers) {
                    if (!await layer.validate(this.form)) {
                        isValid = false;
                    }
                }
                
                if (isValid) {
                    await this.submitSecurely();
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
                
                // Сообщение для скринридеров
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
            async validate(form) {
                const inputs = form.querySelectorAll('input, textarea');
                for (const input of inputs) {
                    if (this.containsXSS(input.value)) {
                        console.warn('Обнаружен потенциальный XSS');
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
            async validate(form) {
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
            
            async validate(form) {
                const now = Date.now();
                
                if (now - this.lastAttempt < this.cooldown) {
                    if (this.attempts >= this.maxAttempts) {
                        throw new Error('Превышено количество попыток. Попробуйте позже.');
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
            async validate(form) {
                // Проверка целостности данных
                const formData = new FormData(form);
                
                for (const [key, value] of formData.entries()) {
                    if (typeof value === 'string' && this.hasDataIntegrityIssues(value)) {
                        console.warn(`Нарушена целостность данных в поле ${key}`);
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
            async validate(form) {
                const inputs = form.querySelectorAll('input, textarea');
                for (const input of inputs) {
                    if (typeof input.value === 'string' && this.hasSecurityIssues(input.value)) {
                        console.warn(`Обнаружены проблемы безопасности в поле: ${input.name || input.id}`);
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

### Проверка clickjacking защиты:
1. Проверка CSP frame-ancestors директивы
2. Проверка X-Frame-Options заголовка
3. Проверка возможности встраивания страниц в iframe
4. Проверка frame-busting скриптов

### Проверка целостности данных:
1. Проверка санитизации ввода
2. Проверка валидации на сервере
3. Проверка использования безопасных методов обработки данных
4. Проверка на SQL-инъекции и другие уязвимости

## Лучшие практики безопасности HTML

### 1. Санитизация пользовательского ввода
```html
<script>
class InputSanitizer {
    static sanitizeHTML(input) {
        if (typeof input !== 'string') return input;
        
        // Создаем временный элемент для безопасной обработки
        const tempDiv = document.createElement('div');
        tempDiv.textContent = input; // Устанавливаем как текст, не как HTML
        return tempDiv.innerHTML;
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
    
    static deepSanitize(obj) {
        if (obj === null || typeof obj !== 'object') {
            return obj;
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

### 2. Правильное использование атрибутов безопасности
```html
<!-- Правильно: безопасное использование атрибутов -->
<form action="https://secure.example.com/api/submit" method="post" target="_self">
    <input type="hidden" name="csrf_token" value="SECURE_CSRF_TOKEN_HERE">
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

### 7. Современные возможности безопасности
```javascript
// Использование современных API для безопасности
class ModernSecurityFeatures {
    constructor() {
        this.checkSecurityAPIs();
        this.setupSecurityFeatures();
    }
    
    checkSecurityAPIs() {
        // Проверка поддержки современных API безопасности
        const securityFeatures = {
            csp: 'securitypolicyviolation' in document,
            sri: 'integrity' in document.createElement('script'),
            trustedTypes: 'trustedTypes' in window,
            fetch: 'fetch' in window,
            indexedDB: 'indexedDB' in window,
            webSockets: 'WebSocket' in window
        };
        
        console.log('Поддержка безопасности:', securityFeatures);
        return securityFeatures;
    }
    
    setupSecurityFeatures() {
        if ('trustedTypes' in window) {
            this.setupTrustedTypes();
        }
        
        if ('serviceWorker' in navigator) {
            this.setupServiceWorkerSecurity();
        }
    }
    
    setupTrustedTypes() {
        if (!window.trustedTypes.createPolicy) return;
        
        // Создание политики для санитизации
        const policy = window.trustedTypes.createPolicy('defaultPolicy', {
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
    }
    
    isSafeScript(script) {
        // Проверка на опасные паттерны
        const dangerousPatterns = [
            /<script/i,
            /javascript:/i,
            /on\w+\s*=/i,
            /eval\(/i,
            /Function\(/i
        ];
        
        return !dangerousPatterns.some(pattern => pattern.test(script));
    }
    
    setupServiceWorkerSecurity() {
        // Установка Service Worker для безопасности
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/sw.js')
                .then(registration => {
                    console.log('Service Worker зарегистрирован:', registration.scope);
                })
                .catch(error => {
                    console.error('Ошибка регистрации Service Worker:', error);
                });
        }
    }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
    new ModernSecurityFeatures();
});
```

## Заключение

Безопасность HTML-страниц - это комплексный подход, который включает в себя защиту от различных типов атак, таких как XSS, CSRF, clickjacking и других. Правильное использование современных возможностей безопасности, таких как CSP, SRI, CSRF токены и правильная санитизация данных, позволяет создавать защищенные веб-приложения, которые безопасны как для пользователей, так и для систем.

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
14. Использование современных веб-компонентов с безопасностью
15. Интеграция с Web Workers, WebSockets и другими API с безопасностью
16. Proper handling of user input and data validation
17. Implementation of defense-in-depth security strategies
18. Use of secure headers and protocols
19. Following security best practices and standards
20. Regular security audits and updates

Эти практики обеспечивают создание веб-сайтов, которые безопасны, доступны и удобны для всех пользователей, независимо от их возможностей или используемых технологий.

Современные подходы к безопасности включают:
- Использование Content Security Policy для ограничения источников контента
- Subresource Integrity для проверки целостности внешних ресурсов
- Trusted Types для предотвращения XSS через DOM
- Proper input sanitization and validation techniques
- Secure WebSocket connections with message validation
- Protected IndexedDB operations
- Optimized resource loading with security in mind
- Modern web component patterns with security built-in
- Testing with automated tools and manual verification
- Following security best practices and standards
- Implementation of defense-in-depth strategies
- Use of secure protocols and authentication methods
- Proper error handling and logging
- Regular security updates and patches

Эти методы позволяют создавать действительно безопасные веб-приложения, которые защищают данные пользователей и обеспечивают надежную защиту от потенциальных угроз.

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

Эти принципы помогают создавать веб-приложения, которые защищают данные пользователей и обеспечивают надежную защиту от потенциальных угроз.

## Следующие темы
- [[HTML/Доступность]]
- [[HTML/Производительность]]
- [[HTML/Совместимость]]

## Теги
#html #security #web-development #frontend #xss #csrf #csp #sri #clickjacking #web-components #javascript #dom-manipulation #input-sanitization #content-security-policy #subresource-integrity #trusted-types #websockets #indexeddb #security-headers #secure-coding #best-practices #security-auditing #security-monitoring #accessibility #performance #modern-web #security-tools #security-testing #security-patterns