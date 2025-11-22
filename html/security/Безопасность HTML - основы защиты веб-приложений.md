# Безопасность HTML: основы защиты веб-приложений

## Введение

Безопасность HTML-приложений является фундаментальной частью разработки веб-приложений. Неправильная обработка пользовательского ввода, отсутствие надлежащей валидации и санитизации могут привести к серьезным уязвимостям, включая межсайтовый скриптинг (XSS), подделку межсайтовых запросов (CSRF) и другие атаки.

## Основные угрозы безопасности HTML

### 1. Межсайтовый скриптинг (XSS)

XSS - это тип инъекционной атаки, при которой злоумышленник внедряет вредоносный скрипт в доверенный веб-сайт. Существует три основных типа XSS:

#### Reflected XSS (Отраженный XSS)
Происходит, когда вредоносный скрипт "отражается" от веб-сервера в HTTP-ответе.

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

#### Stored XSS (Хранимый XSS)
Происходит, когда вредоносный скрипт сохраняется на сервере и отображается другим пользователям.

```html
<!-- Уязвимый комментарий -->
<div class="comment">
    <p><?php echo $comment_text; ?></p>
</div>
```

#### DOM-based XSS (XSS на основе DOM)
Происходит, когда вредоносный скрипт манипулирует DOM-деревом на стороне клиента.

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

### 2. Подделка межсайтовых запросов (CSRF)

CSRF-атака заставляет пользователя выполнить нежелательное действие на веб-сайте, на котором он аутентифицирован.

```html
<!-- Без CSRF токена -->
<form action="/transfer" method="POST">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="to" value="attacker@example.com">
    <input type="submit" value="Получить подарок!">
</form>
```

### 3. Внедрение HTML/JavaScript через атрибуты

```html
<!-- Уязвимый код -->
<a href="<?php echo $_GET['url']; ?>">Перейти</a>

<!-- Возможная атака -->
<!-- http://example.com/page?url=javascript:alert('XSS') -->
```

## Защита от XSS-атак

### 1. Экранирование и санитизация

#### HTML-экранирование
```php
<!-- Безопасный PHP код -->
<h1>Результаты поиска для: <?php echo htmlspecialchars($_GET['q'], ENT_QUOTES, 'UTF-8'); ?></h1>
```

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
```

### 2. Content Security Policy (CSP)

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

### 3. Безопасная вставка контента

```javascript
// НЕБЕЗОПАСНО
element.innerHTML = userInput;

// БЕЗОПАСНО: использование textContent
element.textContent = userInput;

// БЕЗОПАСНО: создание элементов программно
function createSafeElement(tag, textContent) {
    const element = document.createElement(tag);
    element.textContent = textContent;
    return element;
}

// БЕЗОПАСНО: санитизация перед вставкой
const cleanHTML = DOMPurify.sanitize(userInput);
element.insertAdjacentHTML('beforeend', cleanHTML);
```

## Защита форм

### 1. CSRF-токены

```html
<form action="/submit" method="POST">
    <!-- CSRF токен для защиты от подделки межсайтовых запросов -->
    <input type="hidden" name="csrf_token" value="generated_csrf_token">
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <button type="submit">Отправить</button>
</form>
```

### 2. Валидация на стороне клиента и сервера

```html
<form id="secure-form" novalidate>
    <label for="email">Email:</label>
    <input 
        type="email" 
        id="email" 
        name="email" 
        required
        pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
        autocomplete="email">
    
    <label for="phone">Телефон:</label>
    <input 
        type="tel" 
        id="phone" 
        name="phone" 
        required
        pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}">
    
    <button type="submit">Отправить</button>
</form>
```

```javascript
// Клиентская валидация
document.getElementById('secure-form').addEventListener('submit', function(e) {
    const email = document.getElementById('email');
    const phone = document.getElementById('phone');
    
    if (!email.validity.valid || !phone.validity.valid) {
        e.preventDefault();
        alert('Пожалуйста, исправьте ошибки в форме');
    }
});
```

## Безопасные HTTP заголовки

### 1. Заголовки для предотвращения XSS

```html
<!-- Заголовки безопасности -->
<meta http-equiv="X-Content-Type-Options" content="nosniff">
<meta http-equiv="X-Frame-Options" content="DENY">
<meta http-equiv="X-XSS-Protection" content="1; mode=block">
```

### 2. Strict Transport Security

```html
<!-- HTTPS только -->
<meta http-equiv="Strict-Transport-Security" content="max-age=31536000; includeSubDomains; preload">
```

## Защита данных в HTML

### 1. Безопасное хранение данных

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

### 2. Работа с cookies

```html
<!-- Безопасные атрибуты cookie -->
<!-- Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict -->
```

## Современные методы защиты

### 1. Trusted Types API

```javascript
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
        }
    });
}
```

### 2. Subresource Integrity (SRI)

```html
<!-- Пример использования SRI -->
<script src="https://cdn.example.com/library.js" 
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC" 
        crossorigin="anonymous"></script>
```

## Лучшие практики безопасности HTML

### 1. Валидация и санитизация на сервере

```php
<?php
// Пример безопасной обработки пользовательского ввода
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

### 2. Безопасная работа с URL

```javascript
// Безопасная функция редиректа
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
```

### 3. Защита от clickjacking

```html
<!-- Защита от clickjacking -->
<meta http-equiv="X-Frame-Options" content="DENY">
<!-- Или -->
<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
```

## Примеры безопасных паттернов

### 1. Безопасная система комментариев

```html
<!DOCTYPE html>
<html>
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

### 2. Безопасная форма авторизации

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Безопасная авторизация</title>
</head>
<body>
    <form id="login-form" action="/login" method="POST">
        <input type="hidden" name="csrf_token" value="generated_csrf_token">
        
        <div>
            <label for="username">Имя пользователя:</label>
            <input 
                type="text" 
                id="username" 
                name="username" 
                required
                autocomplete="username"
                autocapitalize="none"
                autocorrect="off">
        </div>
        
        <div>
            <label for="password">Пароль:</label>
            <input 
                type="password" 
                id="password" 
                name="password" 
                required
                autocomplete="current-password">
        </div>
        
        <button type="submit">Войти</button>
    </form>
</body>
</html>
```

## Ссылки на связанные темы
- [[html/security/xss]] - Подробнее о XSS
- [[html/forms/security]] - Безопасность форм
- [[security/best-practices]] - Лучшие практики безопасности
- [[js/security]] - Безопасность JavaScript

#programming #html #security #xss #csrf #csp #best-practices