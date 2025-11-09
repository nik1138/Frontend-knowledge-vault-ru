# Безопасность HTML приложений: XSS, защита данных и лучшие практики

## Введение

Безопасность HTML-приложений является критически важной частью разработки веб-приложений. Одной из наиболее распространенных угроз является межсайтовый скриптинг (XSS), который позволяет злоумышленникам внедрять вредоносный JavaScript-код в доверенные веб-сайты. В этой статье мы рассмотрим различные аспекты безопасности HTML-приложений.

## Типы XSS-атак

### Reflected XSS (Отраженный XSS)

Reflected XSS происходит, когда вредоносный скрипт "отражается" от веб-сервера в HTTP-ответе, например, в результатах поиска или ошибке.

```html
<!-- Потенциально уязвимый код -->
<!DOCTYPE html>
<html>
<head>
    <title>Поиск</title>
</head>
<body>
    <h1>Результаты поиска для: <?php echo $_GET['q']; ?></h1>
    <!-- НЕБЕЗОПАСНО: напрямую выводим пользовательский ввод -->
</body>
</html>
```

**Атака:**
```
http://example.com/search?q=<script>alert('XSS')</script>
```

**Безопасная реализация:**
```php
<!DOCTYPE html>
<html>
<head>
    <title>Поиск</title>
</head>
<body>
    <h1>Результаты поиска для: <?php echo htmlspecialchars($_GET['q'], ENT_QUOTES, 'UTF-8'); ?></h1>
    <!-- БЕЗОПАСНО: экранируем специальные символы -->
</body>
</html>
```

### Stored XSS (Хранимый XSS)

Stored XSS возникает, когда вредоносный скрипт сохраняется на сервере (в базе данных, комментариях, профилях пользователей и т.д.) и отображается другим пользователям.

```html
<!-- Уязвимый комментарий -->
<div class="comment">
    <p><?php echo $comment_text; ?></p>
</div>
```

**Безопасная реализация:**
```php
<div class="comment">
    <p><?php echo htmlspecialchars($comment_text, ENT_QUOTES, 'UTF-8'); ?></p>
</div>
```

### DOM-based XSS (XSS на основе DOM)

DOM-based XSS происходит, когда вредоносный скрипт манипулирует DOM-деревом на стороне клиента без участия сервера.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Пример DOM XSS</title>
</head>
<body>
    <script>
        // НЕБЕЗОПАСНО: используем location.hash напрямую
        document.write('Привет, ' + location.hash.substring(1));
    </script>
</body>
</html>
```

**Атака:**
```
http://example.com/page#<script>alert('XSS')</script>
```

**Безопасная реализация:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Безопасный пример</title>
</head>
<body>
    <script>
        // БЕЗОПАСНО: используем безопасные методы
        const hashValue = location.hash.substring(1);
        const safeValue = document.createTextNode(hashValue);
        document.body.appendChild(safeValue);
    </script>
</body>
</html>
```

## Защита от XSS-атак

### Экранирование и санитизация

#### HTML-экранирование
```javascript
// Функция для безопасного экранирования HTML
function escapeHtml(text) {
    const map = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#039;',
        '/': '&#x2F;'
    };
    
    return text.replace(/[&<>"'\/]/g, function(s) {
        return map[s];
    });
}

// Использование
const userInput = '<script>alert("XSS")</script>';
const safeOutput = escapeHtml(userInput);
console.log(safeOutput); // &lt;script&gt;alert("XSS")&lt;/script&gt;
```

#### Санитизация HTML
```javascript
// Пример использования DOMPurify для санитизации HTML
// <script src="https://cdn.jsdelivr.net/npm/dompurify@2.3.3/dist/purify.min.js"></script>

const dirty = '<p>Привет <img src="x" onerror="alert(1)"> мир</p>';
const clean = DOMPurify.sanitize(dirty);
console.log(clean); // <p>Привет <img src="x"> мир</p>

// С конфигурацией
const cleanWithConfig = DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    ALLOWED_ATTR: ['href', 'src', 'alt']
});
```

### Content Security Policy (CSP)

CSP - это важный механизм защиты, который помогает предотвратить XSS-атаки путем ограничения источников, из которых могут загружаться ресурсы.

```html
<!-- В виде HTTP заголовка -->
<!-- Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; -->

<!-- В виде мета-тега -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' https://trusted-cdn.com; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data: https:; 
               font-src 'self'; 
               connect-src 'self'; 
               frame-ancestors 'none'; 
               base-uri 'self';">
```

#### Примеры CSP директив

```html
<!-- Строгая политика (рекомендуется) -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self';
               script-src 'self' 'nonce-r4nd0mNumb3r' 'strict-dynamic';
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;
               font-src 'self';
               connect-src 'self';
               object-src 'none';
               base-uri 'self';
               form-action 'self';
               frame-ancestors 'none';
               block-all-mixed-content;
               upgrade-insecure-requests;">

<!-- Менее строгая политика для разработки -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self';
               script-src 'self' 'unsafe-inline' 'unsafe-eval';
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;
               font-src 'self';">
```

### Безопасная вставка контента

#### Безопасное использование innerHTML
```javascript
// НЕБЕЗОПАСНО
element.innerHTML = userInput;

// БЕЗОПАСНО: использование textContent
element.textContent = userInput;

// БЕЗОПАСНО: использование insertAdjacentHTML с санитизацией
const cleanHTML = DOMPurify.sanitize(userInput);
element.insertAdjacentHTML('beforeend', cleanHTML);

// БЕЗОПАСНО: создание элементов программно
function createSafeElement(tag, textContent) {
    const element = document.createElement(tag);
    element.textContent = textContent;
    return element;
}
```

#### Безопасная работа с URL
```javascript
// Небезопасная функция
function redirectTo(url) {
    window.location.href = url; // Уязвимо для XSS
}

// Безопасная функция
function safeRedirect(url) {
    // Проверяем, что URL начинается с доверенного домена
    const trustedDomains = ['example.com', 'trusted-site.com'];
    const parsedUrl = new URL(url, window.location.origin);
    
    if (trustedDomains.includes(parsedUrl.hostname)) {
        window.location.href = url;
    } else {
        console.error('Недоверенный домен:', parsedUrl.hostname);
    }
}

// Альтернатива: использование белого списка
function validateAndRedirect(url) {
    const allowedUrls = [
        '/dashboard',
        '/profile',
        '/settings',
        'https://trusted-site.com/page'
    ];
    
    if (allowedUrls.includes(url) || allowedUrls.some(allowed => url.startsWith(allowed))) {
        window.location.href = url;
    } else {
        console.error('URL не в белом списке:', url);
    }
}
```

## Современные методы защиты

### Trusted Types API

Trusted Types - это современный API, который помогает предотвратить XSS, требуя, чтобы контент, вставляемый в DOM, был "доверенным".

```html
<script>
// Настройка Trusted Types
if (window.trustedTypes && window.trustedTypes.createPolicy) {
    window.trustedTypes.createPolicy('default', {
        createHTML: (string) => {
            // Санитизация HTML перед вставкой
            return DOMPurify.sanitize(string);
        },
        createScript: (string) => {
            // В идеале, не использовать вставку скриптов
            console.warn('Попытка вставить скрипт:', string);
            return '';
        },
        createScriptURL: (string) => {
            // Проверка URL перед загрузкой скрипта
            const url = new URL(string, window.location.href);
            if (url.hostname === 'trusted-cdn.com') {
                return string;
            }
            console.error('Недоверенный URL скрипта:', string);
            return 'javascript:void(0)';
        }
    });
}
</script>
```

### Subresource Integrity (SRI)

SRI позволяет браузеру проверять, что файлы, загружаемые с CDN, не были изменены.

```html
<!-- Пример использования SRI -->
<script src="https://cdn.example.com/library.js" 
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC" 
        crossorigin="anonymous"></script>

<link rel="stylesheet" href="https://cdn.example.com/style.css" 
      integrity="sha384-cg4uRpy/6+5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5c5" 
      crossorigin="anonymous">
```

### Referrer Policy

Политика реферера помогает контролировать, какая информация о реферере отправляется с запросами.

```html
<!-- В виде мета-тега -->
<meta name="referrer" content="no-referrer">

<!-- Или как HTTP заголовок -->
<!-- Referrer-Policy: no-referrer -->
```

## Защита данных в HTML

### Защита чувствительной информации

```html
<!-- Правильное использование autocomplete -->
<form>
    <input type="text" name="username" autocomplete="username">
    <input type="password" name="password" autocomplete="current-password">
    <input type="text" name="credit-card" autocomplete="cc-number">
    <input type="text" name="credit-card-name" autocomplete="cc-name">
    <input type="text" name="credit-card-expiry" autocomplete="cc-exp">
    <input type="text" name="credit-card-csc" autocomplete="cc-csc">
</form>

<!-- Защита от автозаполнения чувствительных полей -->
<input type="text" name="secret-answer" autocomplete="off" spellcheck="false">
```

### Безопасное хранение данных

```javascript
// НЕБЕЗОПАСНО: хранение чувствительных данных в localStorage
localStorage.setItem('authToken', 'jwt-token-here');

// БЕЗОПАСНЕЕ: использование httpOnly cookies
// Установка через HTTP заголовок на сервере:
// Set-Cookie: authToken=jwt-token-here; HttpOnly; Secure; SameSite=Strict

// Использование sessionStorage для временных данных
sessionStorage.setItem('tempData', JSON.stringify(sensitiveData));

// Шифрование данных перед хранением (клиентская сторона)
function encryptData(data, key) {
    // В реальном приложении используйте криптографически стойкие методы
    return btoa(JSON.stringify(data)); // Простой пример, НЕ для продакшена
}
```

## Лучшие практики безопасности

### Валидация и санитизация на сервере

```php
<?php
// Пример безопасной обработки пользовательского ввода на PHP
function sanitizeInput($input) {
    // Удаляем теги HTML
    $input = strip_tags($input);
    
    // Экранируем специальные символы
    $input = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
    
    // Удаляем лишние пробелы
    $input = trim($input);
    
    return $input;
}

// Использование подготовленных выражений для SQL
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([sanitizeInput($_POST['email'])]);
$user = $stmt->fetch();
?>
```

### Безопасные HTTP заголовки

```html
<!-- Пример безопасных HTTP заголовков -->
<!-- X-Content-Type-Options: nosniff -->
<!-- X-Frame-Options: DENY -->
<!-- X-XSS-Protection: 1; mode=block -->
<!-- Strict-Transport-Security: max-age=31536000; includeSubDomains; preload -->

<!-- В виде мета-тегов (альтернатива) -->
<meta http-equiv="X-Content-Type-Options" content="nosniff">
<meta http-equiv="X-Frame-Options" content="DENY">
<meta http-equiv="X-XSS-Protection" content="1; mode=block">
```

### Безопасная работа с формами

```html
<form action="/submit" method="POST" 
      id="secure-form"
      autocomplete="on"
      novalidate>
    <!-- CSRF токен для защиты от подделки межсайтовых запросов -->
    <input type="hidden" name="csrf_token" value="generated_csrf_token">
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <label for="message">Сообщение:</label>
    <textarea id="message" name="message" required></textarea>
    
    <button type="submit">Отправить</button>
</form>
```

```javascript
// Добавление CSRF токена к AJAX запросам
function addCSRFToken(xhr) {
    const token = document.querySelector('input[name="csrf_token"]').value;
    xhr.setRequestHeader('X-CSRF-Token', token);
}

// Использование fetch с CSRF токеном
document.getElementById('secure-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const formData = new FormData(e.target);
    const csrfToken = document.querySelector('input[name="csrf_token"]').value;
    
    try {
        const response = await fetch('/submit', {
            method: 'POST',
            headers: {
                'X-CSRF-Token': csrfToken,
                'Accept': 'application/json'
            },
            body: formData
        });
        
        if (response.ok) {
            console.log('Форма успешно отправлена');
        }
    } catch (error) {
        console.error('Ошибка отправки формы:', error);
    }
});
```

## Современные инструменты и библиотеки

### Библиотеки для санитизации

```javascript
// Пример использования различных библиотек санитизации

// 1. DOMPurify (рекомендуется)
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(dirtyHTML);

// 2. sanitize-html (на стороне сервера)
// const sanitizeHtml = require('sanitize-html');
// 
// const clean = sanitizeHtml(dirtyHTML, {
//     allowedTags: ['p', 'br', 'strong', 'em', 'u'],
//     allowedAttributes: {
//         'a': ['href']
//     }
// });

// 3. Custom sanitizer
class HTMLSanitizer {
    constructor(allowedTags = [], allowedAttributes = {}) {
        this.allowedTags = new Set(allowedTags);
        this.allowedAttributes = allowedAttributes;
    }
    
    sanitize(html) {
        const temp = document.createElement('div');
        temp.innerHTML = html;
        
        this.cleanNode(temp);
        
        return temp.innerHTML;
    }
    
    cleanNode(node) {
        if (node.nodeType === Node.ELEMENT_NODE) {
            // Удаляем запрещенные теги
            if (!this.allowedTags.has(node.tagName.toLowerCase())) {
                node.remove();
                return;
            }
            
            // Очищаем атрибуты
            const attrs = Array.from(node.attributes);
            attrs.forEach(attr => {
                if (!this.isAllowedAttribute(node.tagName, attr.name)) {
                    node.removeAttribute(attr.name);
                }
            });
            
            // Рекурсивно обрабатываем дочерние элементы
            Array.from(node.childNodes).forEach(child => this.cleanNode(child));
        }
    }
    
    isAllowedAttribute(tag, attribute) {
        if (this.allowedAttributes[tag]) {
            return this.allowedAttributes[tag].includes(attribute);
        }
        return false;
    }
}

// Использование
const sanitizer = new HTMLSanitizer(['p', 'strong', 'em'], {
    'p': ['class'],
    'strong': [],
    'em': []
});
```

## Примеры из промышленной разработки

### Комплексная система защиты комментариев

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';">
    <title>Безопасные комментарии</title>
</head>
<body>
    <div id="comments-container">
        <form id="comment-form">
            <textarea 
                id="comment-text" 
                name="comment" 
                placeholder="Написать комментарий..." 
                maxlength="1000"
                required></textarea>
            <button type="submit">Отправить</button>
        </form>
        
        <div id="comments-list"></div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/dompurify@2.3.3/dist/purify.min.js"></script>
    <script>
        class SecureCommentSystem {
            constructor() {
                this.form = document.getElementById('comment-form');
                this.textarea = document.getElementById('comment-text');
                this.list = document.getElementById('comments-list');
                this.init();
            }
            
            init() {
                this.form.addEventListener('submit', this.handleSubmit.bind(this));
            }
            
            async handleSubmit(event) {
                event.preventDefault();
                
                const commentText = this.textarea.value.trim();
                
                if (!this.validateComment(commentText)) {
                    alert('Комментарий не соответствует требованиям');
                    return;
                }
                
                // Санитизация перед отправкой
                const sanitizedComment = this.sanitizeComment(commentText);
                
                try {
                    const response = await this.submitComment(sanitizedComment);
                    if (response.ok) {
                        this.addCommentToUI(sanitizedComment);
                        this.textarea.value = '';
                    }
                } catch (error) {
                    console.error('Ошибка отправки комментария:', error);
                    alert('Ошибка при отправке комментария');
                }
            }
            
            validateComment(comment) {
                // Проверка длины
                if (comment.length < 1 || comment.length > 1000) {
                    return false;
                }
                
                // Проверка на подозрительные паттерны
                const suspiciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];
                
                return !suspiciousPatterns.some(pattern => pattern.test(comment));
            }
            
            sanitizeComment(comment) {
                // Двойная защита: санитизация на клиенте
                return DOMPurify.sanitize(comment, {
                    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'blockquote'],
                    ALLOWED_ATTR: []
                });
            }
            
            async submitComment(comment) {
                const formData = new FormData();
                formData.append('comment', comment);
                formData.append('csrf_token', this.getCSRFToken());
                
                return fetch('/api/comments', {
                    method: 'POST',
                    body: formData,
                    headers: {
                        'X-Requested-With': 'XMLHttpRequest'
                    }
                });
            }
            
            addCommentToUI(comment) {
                const commentElement = document.createElement('div');
                commentElement.className = 'comment';
                
                // Используем textContent для дополнительной безопасности
                const textElement = document.createElement('p');
                textElement.textContent = comment;
                
                commentElement.appendChild(textElement);
                this.list.prepend(commentElement);
            }
            
            getCSRFToken() {
                return document.querySelector('meta[name="csrf-token"]')?.content || '';
            }
        }
        
        // Инициализация системы
        document.addEventListener('DOMContentLoaded', () => {
            new SecureCommentSystem();
        });
    </script>
</body>
</html>
```

### Безопасная система ввода с автодополнением

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасный автокомплит</title>
    <style>
        .autocomplete-container {
            position: relative;
            display: inline-block;
            width: 100%;
        }
        
        .autocomplete-input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        
        .autocomplete-results {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 1px solid #ccc;
            border-top: none;
            max-height: 200px;
            overflow-y: auto;
            z-index: 1000;
            display: none;
        }
        
        .autocomplete-item {
            padding: 10px;
            cursor: pointer;
            border-bottom: 1px solid #eee;
        }
        
        .autocomplete-item:hover {
            background-color: #f5f5f5;
        }
    </style>
</head>
<body>
    <div class="autocomplete-container">
        <input 
            type="text" 
            id="search-input" 
            class="autocomplete-input" 
            placeholder="Поиск..."
            autocomplete="off">
        <div id="autocomplete-results" class="autocomplete-results"></div>
    </div>

    <script>
        class SecureAutocomplete {
            constructor(inputId, resultsId) {
                this.input = document.getElementById(inputId);
                this.resultsContainer = document.getElementById(resultsId);
                this.timeout = null;
                this.init();
            }
            
            init() {
                this.input.addEventListener('input', this.handleInput.bind(this));
                this.input.addEventListener('keydown', this.handleKeydown.bind(this));
                
                // Скрытие результатов при клике вне поля
                document.addEventListener('click', (e) => {
                    if (!this.input.contains(e.target) && !this.resultsContainer.contains(e.target)) {
                        this.hideResults();
                    }
                });
            }
            
            handleInput() {
                const query = this.input.value.trim();
                
                if (query.length < 2) {
                    this.hideResults();
                    return;
                }
                
                // Используем debounce для уменьшения запросов
                clearTimeout(this.timeout);
                this.timeout = setTimeout(() => {
                    this.search(query);
                }, 300);
            }
            
            handleKeydown(event) {
                const items = this.resultsContainer.querySelectorAll('.autocomplete-item');
                const activeItem = this.resultsContainer.querySelector('.autocomplete-item.active');
                
                switch (event.key) {
                    case 'ArrowDown':
                        event.preventDefault();
                        this.activateNextItem(items, activeItem);
                        break;
                    case 'ArrowUp':
                        event.preventDefault();
                        this.activatePreviousItem(items, activeItem);
                        break;
                    case 'Enter':
                        event.preventDefault();
                        if (activeItem) {
                            this.selectItem(activeItem);
                        }
                        break;
                    case 'Escape':
                        this.hideResults();
                        break;
                }
            }
            
            activateNextItem(items, activeItem) {
                if (items.length === 0) return;
                
                const currentIndex = activeItem ? Array.from(items).indexOf(activeItem) : -1;
                const nextIndex = (currentIndex + 1) % items.length;
                
                if (activeItem) activeItem.classList.remove('active');
                items[nextIndex].classList.add('active');
            }
            
            activatePreviousItem(items, activeItem) {
                if (items.length === 0) return;
                
                const currentIndex = activeItem ? Array.from(items).indexOf(activeItem) : 0;
                const prevIndex = currentIndex === 0 ? items.length - 1 : currentIndex - 1;
                
                if (activeItem) activeItem.classList.remove('active');
                items[prevIndex].classList.add('active');
            }
            
            async search(query) {
                // Экранируем запрос для безопасности
                const sanitizedQuery = this.escapeHtml(query);
                
                try {
                    const response = await fetch(`/api/search?q=${encodeURIComponent(sanitizedQuery)}`);
                    const results = await response.json();
                    
                    this.displayResults(results);
                } catch (error) {
                    console.error('Ошибка поиска:', error);
                    this.hideResults();
                }
            }
            
            displayResults(results) {
                if (!results || results.length === 0) {
                    this.hideResults();
                    return;
                }
                
                // Очищаем контейнер
                this.resultsContainer.innerHTML = '';
                
                // Создаем элементы результатов
                results.forEach(result => {
                    const item = document.createElement('div');
                    item.className = 'autocomplete-item';
                    
                    // Используем textContent для безопасности
                    item.textContent = result.name || result.title || result;
                    
                    // Добавляем данные как атрибуты, а не HTML
                    item.dataset.value = result.value || result;
                    
                    item.addEventListener('click', () => {
                        this.selectItem(item);
                    });
                    
                    this.resultsContainer.appendChild(item);
                });
                
                this.resultsContainer.style.display = 'block';
            }
            
            selectItem(item) {
                const value = item.dataset.value;
                this.input.value = value;
                this.hideResults();
                
                // Вызываем событие для внешнего кода
                this.input.dispatchEvent(new CustomEvent('autocomplete-select', {
                    detail: { value: value }
                }));
            }
            
            hideResults() {
                this.resultsContainer.style.display = 'none';
                
                // Убираем активный элемент
                const activeItem = this.resultsContainer.querySelector('.autocomplete-item.active');
                if (activeItem) {
                    activeItem.classList.remove('active');
                }
            }
            
            escapeHtml(text) {
                const div = document.createElement('div');
                div.textContent = text;
                return div.innerHTML;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new SecureAutocomplete('search-input', 'autocomplete-results');
        });
    </script>
</body>
</html>
```

## Ссылки на связанные темы
- [[html/validation]] - Валидация HTML
- [[js/security]] - Безопасность JavaScript
- [[security/best-practices]] - Лучшие практики безопасности
- [[css/security]] - Безопасность CSS

#programming #html #security #xss #best-practices #web-security