---
aliases: [XSS, Межсайтовый скриптинг, Защита от XSS]
tags: [security, xss, frontend-security, injection]
---

# XSS-защита

## Введение

Межсайтовый скриптинг (Cross-Site Scripting, XSS) — это тип уязвимости, при котором злоумышленник внедряет вредоносный скрипт в веб-страницу, который затем выполняется в браузере других пользователей. XSS-атаки могут привести к краже сессионных cookies, перенаправлению на вредоносные сайты или выполнению нежелательных действий от имени пользователя.

## Типы XSS-атак

### 1. Отраженный XSS (Reflected XSS)
Злоумышленник отправляет пользователю специально сформированную ссылку с вредоносным скриптом. При переходе по ссылке скрипт выполняется в браузере жертвы.

### 2. Хранимый XSS (Stored XSS)
Вредоносный скрипт сохраняется на сервере (например, в комментариях, сообщениях) и отображается всем пользователям, просматривающим контент.

### 3. DOM-based XSS
Тип XSS, при котором вредоносный скрипт манипулирует DOM-деревом на стороне клиента без взаимодействия с сервером.

## Примеры XSS-атак

### Отраженный XSS
```html
<!-- Небезопасный код на сервере -->
<?php echo $_GET['name']; ?>
```

Злоумышленник может создать ссылку:
```
http://example.com/page.php?name=<script>alert('XSS')</script>
```

При переходе пользователь выполнит вредоносный скрипт.

### DOM-based XSS
```javascript
// Небезопасный код на клиенте
document.getElementById('output').innerHTML = window.location.hash.substring(1);
```

Если пользователь перейдет по ссылке:
```
http://example.com/page.html#<script>alert('XSS')</script>
```

Скрипт будет выполнен.

## Методы защиты

### 1. Экранирование вывода данных

Важно экранировать специальные символы при выводе пользовательских данных:

```javascript
// Небезопасный код
function renderUserInput(input) {
  document.getElementById('output').innerHTML = input;
}

// Безопасный код
function escapeHtml(unsafe) {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

function renderUserInput(input) {
  document.getElementById('output').textContent = input; // Используем textContent вместо innerHTML
}
```

### 2. Использование безопасных методов вставки HTML

Вместо `innerHTML` используйте безопасные методы:

```javascript
// Небезопасно
element.innerHTML = userInput;

// Безопасно
element.textContent = userInput;

// Или безопасная вставка через DOM-элементы
const textNode = document.createTextNode(userInput);
element.appendChild(textNode);
```

### 3. Использование фреймворков с встроенной защитой

Современные фреймворки обеспечивают встроенную защиту от XSS:

#### React
```jsx
// React автоматически экранирует данные при использовании JSX
function UserProfile({ username }) {
  return <div>{username}</div>; // username автоматически экранируется
}
```

#### Vue.js
```vue
<template>
  <!-- Vue автоматически экранирует данные -->
  <div>{{ userInput }}</div>
</template>
```

### 4. Content Security Policy (CSP)

CSP позволяет ограничить источники, из которых могут загружаться ресурсы:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://trusted-cdn.com; object-src 'none';">
```

CSP может предотвратить выполнение inline-скриптов и блокировать XSS-атаки.

### 5. Правильная обработка URL

При работе с URL, особенно при использовании `window.location`, необходимо быть осторожным:

```javascript
// Небезопасно
function updateContent() {
  const hash = window.location.hash.substring(1);
  document.getElementById('content').innerHTML = hash;
}

// Безопасно
function updateContent() {
  const hash = window.location.hash.substring(1);
  document.getElementById('content').textContent = hash;
}
```

## Практические рекомендации для фронтенд-разработчиков

### 1. Используйте безопасные методы рендеринга
- Избегайте `innerHTML`, `outerHTML`, `document.write()`
- Используйте `textContent`, `createElement()`, `appendChild()`
- Используйте шаблонные литералы с осторожностью

### 2. Валидируйте и фильтруйте пользовательский ввод
- Проверяйте типы данных
- Используйте регулярные выражения для проверки форматов
- Ограничивайте длину вводимых данных

### 3. Используйте библиотеки для очистки HTML
```javascript
// Пример использования DOMPurify для очистки HTML
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(dirty);
document.getElementById('content').innerHTML = clean;
```

### 4. Не доверяйте данным из URL
- Всегда валидируйте параметры URL
- Используйте URLSearchParams для безопасной обработки параметров

```javascript
// Безопасная обработка параметров URL
const urlParams = new URLSearchParams(window.location.search);
const name = urlParams.get('name');
if (name && /^[a-zA-Z0-9_]+$/.test(name)) {
  document.getElementById('output').textContent = name;
}
```

### 5. Используйте httpOnly cookies
Для хранения сессионных токенов используйте httpOnly cookies, которые недоступны из JavaScript:

```javascript
// На сервере установите httpOnly cookie
res.cookie('session', sessionId, { httpOnly: true, secure: true });
```

## Проверка на наличие XSS

### 1. Ручное тестирование
- Попробуйте ввести `<script>alert('XSS')</script>` в различные поля ввода
- Проверьте все места вывода пользовательских данных
- Проверьте URL-параметры и хэш

### 2. Автоматизированные инструменты
- OWASP ZAP
- Burp Suite
- XSStrike

### 3. Проверка кода
- Используйте статические анализаторы кода
- Проверяйте использование небезопасных методов (innerHTML и т.д.)

## Примеры безопасного кода

### Безопасная вставка HTML с разрешенными тегами
```javascript
function sanitizeAndInsert(htmlString) {
  // Создаем временный элемент для парсинга
  const temp = document.createElement('div');
  temp.textContent = htmlString;
  
  // Извлекаем только разрешенные теги
  const allowedTags = ['b', 'i', 'em', 'strong'];
  const sanitized = document.createElement('div');
  
  for (const child of temp.childNodes) {
    if (child.nodeType === Node.TEXT_NODE) {
      sanitized.appendChild(child.cloneNode());
    } else if (allowedTags.includes(child.tagName.toLowerCase())) {
      sanitized.appendChild(child.cloneNode(true));
    }
  }
  
  return sanitized.innerHTML;
}
```

### Безопасная обработка данных формы
```javascript
function handleFormSubmit(event) {
  event.preventDefault();
  
  const formData = new FormData(event.target);
  const userInput = formData.get('userInput');
  
  // Валидация и очистка данных
  if (typeof userInput === 'string' && userInput.length < 1000) {
    // Экранируем и вставляем данные
    document.getElementById('output').textContent = userInput;
  } else {
    console.error('Некорректный ввод данных');
  }
}
```

## Заключение

XSS-атаки остаются одной из самых распространенных угроз в веб-безопасности. Понимание механизмов этих атак и применение соответствующих методов защиты позволяет значительно повысить безопасность веб-приложений. Главное правило - никогда не доверять пользовательским данным и всегда экранировать их при выводе.

## См. также
- [[Основы-веб-безопасности]]
- [[CSRF-защита]]
- [[Защита-от-инъекций]]
- [[HTTP-заголовки-безопасности]]