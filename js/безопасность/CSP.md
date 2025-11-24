---
aliases: [CSP, Content Security Policy, Политика безопасности контента]
tags: [javascript, безопасность, csp, веб-безопасность]
---

# Content Security Policy (CSP)

## Общее описание

Content Security Policy (CSP) - это важный механизм безопасности, который помогает предотвратить XSS, clickjacking и другие атаки, контролируя, какие ресурсы может загружать и выполнять веб-страница. В 2025 году CSP стал критически важным элементом безопасности современных веб-приложений.

## Как работает CSP

CSP позволяет веб-сайтам объявлять политику разрешений для контента, который может быть загружен и выполнено на странице. Политика передается браузеру через HTTP-заголовок или тег `<meta>` и включает директивы, контролирующие различные типы ресурсов.

## Основные директивы CSP

### 1. script-src
Контролирует, откуда могут загружаться и выполняться JavaScript-скрипты:

```html
<!-- Разрешает скрипты только с текущего домена -->
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self';">

<!-- Разрешает скрипты с текущего домена и trusted-cdn.com -->
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'self' https://trusted-cdn.com;">
```

### 2. style-src
Контролирует источники CSS-стилей:

```html
<!-- Разрешает стили только с текущего домена -->
<meta http-equiv="Content-Security-Policy" 
      content="style-src 'self';">
```

### 3. img-src
Контролирует источники изображений:

```html
<!-- Разрешает изображения с текущего домена и gravatar.com -->
<meta http-equiv="Content-Security-Policy" 
      content="img-src 'self' https://secure.gravatar.com;">
```

### 4. connect-src
Контролирует источники для AJAX-запросов, WebSocket-соединений и EventSource:

```html
<!-- Разрешает AJAX-запросы только на текущий домен -->
<meta http-equiv="Content-Security-Policy" 
      content="connect-src 'self';">
```

## Практические примеры CSP

### Строгая политика для веб-приложения

```html
<meta http-equiv="Content-Security-Policy" 
      content="
          default-src 'self';
          script-src 'self' 'unsafe-inline' https://analytics.example.com;
          style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
          img-src 'self' data: https:;
          font-src 'self' https://fonts.gstatic.com;
          connect-src 'self' https://api.example.com;
          frame-ancestors 'none';
          base-uri 'self';
          form-action 'self';
      ">
```

### CSP для API-ориентированного приложения

```javascript
// Пример настройки CSP в Express.js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      connectSrc: ["'self'", "https://api.example.com"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  })
);
```

## CSP в контексте JavaScript-приложений

### Защита от XSS через CSP

```javascript
// Неправильный способ (уязвим к XSS)
function vulnerableFunction(userInput) {
    eval(userInput); // CSP с 'unsafe-eval' запретит это
}

// Правильный способ
function safeFunction(userInput) {
    // Использование безопасных методов вместо eval
    const parsed = JSON.parse(userInput);
    return parsed;
}
```

### Совместимость с современными фреймворками

```javascript
// Для React-приложений может потребоваться 'unsafe-inline' для стилей
// Лучше использовать nonce или hash
const crypto = require('crypto');
const nonce = crypto.randomBytes(16).toString('hex');

// В шаблоне:
`<meta http-equiv="Content-Security-Policy" 
       content="style-src 'self' 'nonce-${nonce}';">`

// В React-компоненте:
<div style={{color: 'red'}} nonce={nonce}>Стилизованный элемент</div>
```

## Отчеты о нарушениях CSP

CSP может отправлять отчеты о нарушениях политики для мониторинга и отладки:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; report-uri /csp-violation-report-endpoint;">
```

```javascript
// Обработка отчетов о нарушениях
app.post('/csp-violation-report-endpoint', (req, res) => {
    const cspReport = req.body['csp-report'];
    console.log('CSP Violation:', cspReport);
    
    // Логирование нарушений для анализа
    logCSPViolation({
        blockedUri: cspReport['blocked-uri'],
        violatedDirective: cspReport['violated-directive'],
        originalPolicy: cspReport['original-policy']
    });
});
```

## CSP3 и новые возможности

В 2025 году CSP3 включает дополнительные возможности:

### 1. Trusted Types
Предотвращает XSS за счет принуждения к использованию безопасных API:

```html
<meta http-equiv="Content-Security-Policy" 
      content="trusted-types sanitizer dompurify; require-trusted-types-for 'script';">
```

```javascript
// Использование Trusted Types
function createSafeHTML(input) {
    const escaped = escapeHtml(input);
    return trustedTypes.createPolicy('default', {
        createHTML: (string) => escaped
    }).createHTML(escaped);
}
```

### 2. Директива sandbox
Ограничивает возможности iframe:

```html
<meta http-equiv="Content-Security-Policy" 
      content="sandbox allow-scripts allow-same-origin;">
```

## Практические рекомендации

### 1. Постепенное внедрение CSP

```javascript
// Начните с отчетов, не блокируя ничего
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      reportUri: "/csp-report"
    },
    reportOnly: true // Только отчеты, без блокировки
  })
);
```

### 2. Использование nonce для динамических скриптов

```javascript
// Генерация nonce для каждого запроса
function generateCSPNonce() {
    return crypto.randomBytes(16).toString('hex');
}

// В middleware
app.use((req, res, next) => {
    res.locals.nonce = generateCSPNonce();
    next();
});

// В шаблоне
`<script nonce="${res.locals.nonce}">...</script>`
```

### 3. Строгая политика для чувствительных страниц

```html
<!-- Для страниц входа и платежей -->
<meta http-equiv="Content-Security-Policy" 
      content="
          default-src 'none';
          script-src 'self';
          style-src 'self';
          img-src 'self' data:;
          connect-src 'self';
          frame-ancestors 'none';
      ">
```

## CSP в российском контексте 2025

В 2025 году в России требования к защите веб-приложений ужесточаются, особенно для государственных и финансовых сервисов. ФСТЭК и ФСБ рекомендуют использовать CSP как один из ключевых элементов защиты веб-приложений.

Согласно рекомендациям, CSP должен быть внедрен во все веб-приложения, обрабатывающие персональные данные или финансовую информацию. В банковском секторе CSP обязателен для защиты от XSS-атак, направленных на кражу финансовой информации.

## Тестирование CSP

```javascript
// Пример автоматизированного тестирования CSP
const puppeteer = require('puppeteer');

async function testCSP(pageUrl) {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    
    // Слушатель событий CSP
    page.on('console', msg => {
        if (msg.text().includes('Content Security Policy')) {
            console.log('CSP Violation:', msg.text());
        }
    });
    
    await page.goto(pageUrl);
    await browser.close();
}
```

## Заключение

CSP является важным слоем защиты веб-приложений, особенно в сочетании с другими методами безопасности. Правильная настройка CSP может значительно снизить риск XSS-атак и других уязвимостей, связанных с выполнением нежелательного контента.

См. также: [[XSS-защита]], [[CSRF-защита]], [[Валидация-данных]], [[Защита-от-инъекций]]

> [!tip]
> Начинайте с отчетов о нарушениях (report-only mode), чтобы понять, какие ресурсы использует ваше приложение, прежде чем внедрять строгую политику.