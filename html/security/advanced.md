# Безопасность HTML: защита от XSS, CSRF и других угроз

## Введение

Безопасность HTML-приложений является критически важной частью разработки веб-приложений. Неправильная обработка пользовательского ввода, отсутствие надлежащей валидации и санитизации могут привести к серьезным уязвимостям, включая межсайтовый скриптинг (XSS), подделку межсайтовых запросов (CSRF), внедрение HTML и другие атаки. В этой статье мы рассмотрим продвинутые методы защиты HTML-приложений.

## Типы угроз безопасности HTML

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

#### Stored XSS (Хранимый XSS)
Происходит, когда вредоносный скрипт сохраняется на сервере (в базе данных, комментариях, профилях пользователей и т.д.) и отображается другим пользователям.

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

#### DOM-based XSS (XSS на основе DOM)
Происходит, когда вредоносный скрипт манипулирует DOM-деревом на стороне клиента без участия сервера.

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

### 2. Подделка межсайтовых запросов (CSRF)

CSRF-атака заставляет пользователя выполнить нежелательное действие на веб-сайте, на котором он аутентифицирован.

```html
<!-- Без CSRF токена - уязвимо -->
<form action="/transfer" method="POST">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="to" value="attacker@example.com">
    <input type="submit" value="Получить подарок!">
</form>
```

**Безопасная реализация:**
```html
<form action="/transfer" method="POST">
    <!-- CSRF токен для защиты от подделки межсайтовых запросов -->
    <input type="hidden" name="csrf_token" value="generated_csrf_token">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="to" value="user@example.com">
    <input type="submit" value="Перевести">
</form>
```

### 3. Внедрение HTML/JavaScript через атрибуты

```html
<!-- Уязвимый код -->
<a href="<?php echo $_GET['url']; ?>">Перейти</a>

<!-- Возможная атака -->
<!-- http://example.com/page?url=javascript:alert('XSS') -->
```

**Безопасная реализация:**
```php
<?php
function validateUrl($url) {
    // Проверяем, что URL начинается с http:// или https://
    return filter_var($url, FILTER_VALIDATE_URL) !== false && 
           (strpos($url, 'http://') === 0 || strpos($url, 'https://') === 0);
}

$safeUrl = validateUrl($_GET['url']) ? $_GET['url'] : 'https://example.com';
?>
<a href="<?php echo htmlspecialchars($safeUrl, ENT_QUOTES, 'UTF-8'); ?>">Перейти</a>
```

## Защита от XSS-атак

### 1. Экранирование и санитизация

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

### 3. Безопасная вставка контента

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

### 1. Trusted Types API

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

### 2. Subresource Integrity (SRI)

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

### 3. Referrer Policy

Политика реферера помогает контролировать, какая информация о реферере отправляется с запросами.

```html
<!-- В виде мета-тега -->
<meta name="referrer" content="no-referrer">

<!-- Или как HTTP заголовок -->
<!-- Referrer-Policy: no-referrer -->
```

## Защита форм

### 1. CSRF-токены

```html
<form action="/submit" method="POST" id="secure-form">
    <!-- CSRF токен для защиты от подделки межсайтовых запросов -->
    <input type="hidden" name="csrf_token" value="generated_csrf_token">
    
    <label for="email">Email:</label>
    <input 
        type="email" 
        id="email" 
        name="email" 
        required
        autocomplete="email">
    
    <label for="message">Сообщение:</label>
    <textarea id="message" name="message" required></textarea>
    
    <button type="submit">Отправить</button>
</form>
```

```javascript
// Добавление CSRF токена к AJAX запросам
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

### 2. Валидация на стороне клиента и сервера

```html
<form id="validation-form" novalidate>
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
    
    <label for="password">Пароль:</label>
    <input 
        type="password" 
        id="password" 
        name="password" 
        required
        minlength="8">
    
    <button type="submit">Отправить</button>
</form>
```

```javascript
// Клиентская валидация с дополнительными проверками
document.getElementById('validation-form').addEventListener('submit', function(e) {
    const email = document.getElementById('email');
    const phone = document.getElementById('phone');
    const password = document.getElementById('password');
    
    let isValid = true;
    
    // Проверка email
    if (!email.validity.valid) {
        showError(email, 'Пожалуйста, введите корректный email');
        isValid = false;
    }
    
    // Проверка телефона
    if (!phone.validity.valid) {
        showError(phone, 'Пожалуйста, введите корректный телефон в формате 123-456-7890');
        isValid = false;
    }
    
    // Проверка пароля
    if (!validatePassword(password.value)) {
        showError(password, 'Пароль должен содержать минимум 8 символов');
        isValid = false;
    }
    
    if (!isValid) {
        e.preventDefault();
    }
});

function showError(input, message) {
    // Удалить предыдущие ошибки
    const existingError = input.parentNode.querySelector('.error-message');
    if (existingError) {
        existingError.remove();
    }
    
    // Создать новый элемент ошибки
    const error = document.createElement('div');
    error.className = 'error-message';
    error.textContent = message;
    error.style.color = 'red';
    error.style.fontSize = '0.875rem';
    error.style.marginTop = '0.25rem';
    
    input.parentNode.appendChild(error);
}

function validatePassword(password) {
    return password.length >= 8;
}
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

## Защита от clickjacking

```html
<!-- Защита от clickjacking -->
<meta http-equiv="X-Frame-Options" content="DENY">
<!-- Или -->
<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
```

## Примеры из промышленной разработки

### 1. Безопасная система комментариев

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

### 2. Защита от overposting атак

```html
<!DOCTYPE html>
<html>
<head>
    <title>Безопасная форма профиля</title>
</head>
<body>
    <form id="profile-form" action="/profile/update" method="POST">
        <!-- CSRF токен -->
        <input type="hidden" name="csrf_token" value="generated_token">
        
        <!-- Только разрешенные поля для редактирования -->
        <div>
            <label for="first-name">Имя:</label>
            <input type="text" id="first-name" name="first_name" value="Иван" required>
        </div>
        
        <div>
            <label for="last-name">Фамилия:</label>
            <input type="text" id="last-name" name="last_name" value="Иванов" required>
        </div>
        
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" value="ivan@example.com" required>
        </div>
        
        <!-- Скрытое поле с ID пользователя (не изменяемое пользователем) -->
        <input type="hidden" name="user_id" value="123" readonly>
        
        <!-- Поля, которые не должны быть изменяемы через эту форму -->
        <!-- <input type="hidden" name="is_admin" value="false"> - НЕ ДОБАВЛЯТЬ! -->
        
        <button type="submit">Сохранить</button>
    </form>
</body>
</html>
```

### 3. Безопасная загрузка файлов

```html
<!DOCTYPE html>
<html>
<head>
    <title>Безопасная загрузка файлов</title>
</head>
<body>
    <form id="file-upload-form" enctype="multipart/form-data">
        <input type="hidden" name="csrf_token" value="generated_token">
        
        <label for="avatar">Загрузить аватар:</label>
        <input 
            type="file" 
            id="avatar" 
            name="avatar" 
            accept="image/*" 
            required>
        
        <div class="file-info" id="file-info"></div>
        <button type="submit">Загрузить</button>
    </form>

    <script>
        document.getElementById('avatar').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                // Проверка типа файла на клиенте
                const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
                if (!allowedTypes.includes(file.type)) {
                    alert('Разрешены только изображения (JPEG, PNG, GIF)');
                    e.target.value = '';
                    return;
                }
                
                // Проверка размера файла
                const maxSize = 5 * 1024 * 1024; // 5MB
                if (file.size > maxSize) {
                    alert('Файл слишком большой. Максимальный размер: 5MB');
                    e.target.value = '';
                    return;
                }
                
                // Отображение информации о файле
                document.getElementById('file-info').innerHTML = `
                    <p>Имя файла: ${file.name}</p>
                    <p>Тип: ${file.type}</p>
                    <p>Размер: ${(file.size / 1024 / 1024).toFixed(2)} MB</p>
                `;
            }
        });
    </script>
</body>
</html>
```

## Лучшие практики безопасности HTML

### 1. Валидация и санитизация на сервере

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

// Безопасная вставка данных в HTML
echo "<p>Добро пожаловать, " . htmlspecialchars($user['name'], ENT_QUOTES, 'UTF-8') . "!</p>";
?>
```

### 2. Безопасная работа с URL

```javascript
// Безопасная функция для построения URL
function buildSafeUrl(base, path, params = {}) {
    const url = new URL(base);
    
    // Проверяем, что путь не содержит опасных символов
    const safePath = path.replace(/(\.\.\/|\.\.\\)/g, '');
    url.pathname = safePath;
    
    // Добавляем параметры безопасности
    Object.keys(params).forEach(key => {
        if (typeof params[key] === 'string') {
            // Санитизируем параметры
            const sanitizedValue = encodeURIComponent(params[key]);
            url.searchParams.set(key, sanitizedValue);
        }
    });
    
    return url.toString();
}

// Использование
const safeUrl = buildSafeUrl('https://api.example.com', '/users', {
    id: '123',
    name: 'John Doe'
});
```

### 3. Защита от overposting

```html
<!-- Правильный подход: явно указываем разрешенные поля -->
<form method="POST" action="/user/update">
    <input type="hidden" name="csrf_token" value="token">
    
    <!-- Разрешенные поля -->
    <input type="text" name="first_name" value="Иван">
    <input type="text" name="last_name" value="Иванов">
    <input type="email" name="email" value="ivan@example.com">
    
    <!-- Поля, которые не должны быть изменяемы -->
    <!-- Не включаем is_admin, role, created_at и т.д. -->
    
    <button type="submit">Сохранить</button>
</form>
```

## Ссылки на связанны темы
- [[Безопасность HTML - основы защиты веб-приложений]] - Основы безопасности HTML
- [[html/forms/security]] - Безопасность форм
- [[security/best-practices]] - Лучшие практики безопасности
- [[js/security]] - Безопасность JavaScript

#programming #html #security #xss #csrf #csp #best-practices #web-security