---
tags: [javascript, security, web-security, best-practices]
aliases: [JS Security Fundamentals, JavaScript Security Basics]
---

# Основы безопасности JavaScript

## Введение

Безопасность JavaScript критически важна для защиты веб-приложений от различных угроз. Понимание фундаментальных принципов безопасности позволяет разработчикам создавать более защищенные приложения и предотвращать атаки.

## Распространенные уязвимости JavaScript

### 1. Внедрение скриптов (XSS)

**Межсайтовый скриптинг (XSS)** - это тип атаки, при котором злоумышленник внедряет вредоносный скрипт в веб-страницу, который затем выполняется в браузере других пользователей.

```javascript
// НЕБЕЗОПАСНЫЙ код
const userInput = document.getElementById('userInput').value;
document.getElementById('output').innerHTML = userInput; // Уязвимость XSS

// БЕЗОПАСНЫЙ код
const userInput = document.getElementById('userInput').value;
document.getElementById('output').textContent = userInput; // Безопасный способ
```

### 2. Внедрение команд (Command Injection)

В Node.js особенно важно избегать выполнения команд с непроверенными пользовательскими данными:

```javascript
// НЕБЕЗОПАСНЫЙ код
const { exec } = require('child_process');
exec(`ls ${userInput}`, (error, stdout) => {
  // Уязвимость к внедрению команд
});

// БЕЗОПАСНЫЙ код
const { spawn } = require('child_process');
spawn('ls', [userInput]); // Параметры передаются отдельно
```

### 3. Уязвимости, связанные с обработкой данных

Неправильная обработка пользовательских данных может привести к различным проблемам, включая обход аутентификации и повышение привилегий.

## Валидация входных данных

Валидация входных данных - это первый и один из самых важных уровней защиты приложения. Все входные данные, поступающие от пользователей, должны быть проверены перед обработкой.

### Проверка на стороне клиента

```javascript
function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function sanitizeInput(input) {
  // Удаление потенциально опасных символов
  return input.replace(/[<>'"&]/g, function(match) {
    return {
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#x27;',
      '&': '&amp;'
    }[match];
  });
}
```

### Проверка на стороне сервера

Даже при наличии клиентской валидации, серверная проверка обязательна, так как клиентский код может быть обойден:

```javascript
// Пример валидации в Node.js с использованием Joi
const Joi = require('joi');

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(120)
});

const { error, value } = userSchema.validate(userData);
```

## Практики безопасного кодирования

### 1. Использование безопасных библиотек

Используйте проверенные библиотеки для обработки данных, таких как DOMPurify для очистки HTML:

```javascript
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(dirty);
document.getElementById('content').innerHTML = clean;
```

### 2. Избегайте eval() и других опасных функций

Функции `eval()`, `Function()`, `setTimeout()`/`setInterval()` с передачей строк несут в себе серьезные риски безопасности:

```javascript
// НЕБЕЗОПАСНО
eval(userInput);

// ЛУЧШЕ
const result = new Function('return ' + userInput)();
```

### 3. Используйте строгий режим

```javascript
'use strict';

// Это помогает избежать некоторых распространенных ошибок
```

## Политика безопасности содержимого (CSP)

**Content Security Policy (CSP)** - это важный механизм защиты от XSS и других атак, позволяющий ограничить источники, с которых могут загружаться ресурсы.

### Установка CSP заголовка

```javascript
// В Node.js с Express
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'");
  next();
});
```

### Пример CSP заголовка

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://apis.example.com; object-src 'none'; style-src 'self' 'unsafe-inline'; font-src 'self' fonts.example.com; img-src 'self' data: https:; media-src 'self'; frame-src 'none'; connect-src 'self' ws://localhost:3000/;
```

## Политика одинакового источника (Same-Origin Policy)

**Same-Origin Policy** - это важный механизм безопасности, который ограничивает взаимодействие между документами/скриптами с разных источников. Источник определяется комбинацией протокола, домена и порта.

### Нарушение политики

Попытка доступа к ресурсам с другого источника без соответствующих разрешений будет заблокирована браузером.

## CORS (Cross-Origin Resource Sharing)

**CORS** - это механизм, который позволяет веб-страницам запрашивать ресурсы с другого домена, при соблюдении определенных условий.

### Настройка CORS в Node.js

```javascript
const cors = require('cors');

// Разрешить всем
app.use(cors());

// Или с ограничениями
app.use(cors({
  origin: ['https://trusted-site.com', 'https://another-trusted-site.com'],
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## Безопасное хранение данных

### На стороне клиента

- **localStorage/sessionStorage**: Не безопасны для хранения чувствительных данных
- **Cookies**: Используйте флаги `HttpOnly`, `Secure`, `SameSite`

```javascript
// Установка безопасных куки
document.cookie = "sessionId=abc123; Secure; HttpOnly; SameSite=Strict";
```

### На стороне сервера

- Храните чувствительные данные в зашифрованном виде
- Используйте безопасные методы аутентификации (JWT с правильной реализацией, сессии с CSRF защитой)

## Предотвращение атак внедрения

### SQL-инъекции в Node.js

```javascript
// Используйте подготовленные выражения
const stmt = db.prepare('SELECT * FROM users WHERE id = ?');
const result = stmt.get(userId);
```

### HTML-инъекции

См. раздел валидации входных данных и использования библиотек очистки.

## Безопасность серверного JavaScript (Node.js)

### Управление зависимостями

```bash
npm audit
npm audit fix
```

### Проверка уязвимостей

Используйте инструменты для проверки уязвимостей в зависимостях, такие как `npm audit` или сторонние сервисы.

### Безопасная обработка файлов

```javascript
const path = require('path');
const uploadPath = path.join(__dirname, 'uploads', path.basename(fileName));
// Проверка расширения файла и его содержимого
```

## OWASP Top 10 в контексте JavaScript

1. **Инъекции (A1)** - Предотвращение XSS, SQL-инъекций
2. **Нарушение контроля доступа (A2)** - Правильная аутентификация и авторизация
3. **Межсайтовый скриптинг (XSS) (A3)** - Очистка и валидация вывода
4. **Небезопасная десериализация (A4)** - Проверка данных при десериализации
5. **Неправильная настройка безопасности (A5)** - Правильные заголовки, настройки CSP
6. **Уязвимости в компонентах с известными уязвимостями (A6)** - Обновление зависимостей
7. **Аутентификация, подверженная нападению (A7)** - Использование проверенных библиотек аутентификации
8. **Небезопасное хранение данных (A8)** - Шифрование чувствительных данных
9. **Небезопасные реализации перенаправлений и переходов (A9)** - Проверка URL-адресов
10. **Недостаточная защита от автоматизированных атак (A10)** - Защита от ботов и автоматизированных атак

## Внутренние связи

- [[js/best-practices]] - Лучшие практики программирования на JavaScript
- [[js/security/cors]] - Подробное руководство по CORS
- [[js/security/xss-protection]] - Защита от XSS атак
- [[nodejs/security]] - Безопасность в Node.js
- [[web-security/fundamentals]] - Общие принципы веб-безопасности
- [[html/security]] - Безопасность HTML
- [[http/security-headers]] - HTTP заголовки безопасности

## Заключение

Безопасность JavaScript требует комплексного подхода, включающего как клиентскую, так и серверную защиту. Важно следить за последними уязвимостями, использовать проверенные библиотеки и следовать лучшим практикам безопасного программирования.

## Ключевые тезисы

- Всегда проверяйте и очищайте пользовательский ввод
- Используйте CSP для предотвращения XSS
- Правильно настраивайте CORS
- Следите за безопасностью зависимостей
- Храните чувствительные данные безопасно
- Регулярно обновляйте библиотеки и инструменты