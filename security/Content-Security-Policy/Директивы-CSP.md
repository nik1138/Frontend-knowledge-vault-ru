---
aliases: ["CSP директивы", "Content Security Policy директивы", "Политики CSP"]
tags: [security, csp, content-security-policy, web-security, http-headers]
---

# Директивы Content Security Policy

## Введение

Content Security Policy (CSP) — это важный механизм безопасности, который помогает предотвратить атаки типа XSS, clickjacking и другие инъекции, контролируя, какие ресурсы могут быть загружены и выполнены на веб-странице. Директивы CSP определяют политику безопасности для различных типов контента.

## Основные директивы CSP

### 1. default-src

Базовая директива, которая применяется ко всем ресурсам, если для них не указана специфичная директива.

```
Content-Security-Policy: default-src 'self';
```

Это означает, что все ресурсы могут загружаться только с того же домена.

### 2. script-src

Контролирует, откуда могут загружаться и выполняться JavaScript-скрипты.

```
Content-Security-Policy: script-src 'self' 'unsafe-inline' 'unsafe-eval' https://trusted-cdn.com;
```

#### Значения для script-src:

- `'self'` — разрешает загрузку с того же домена
- `'unsafe-inline'` — разрешает inline-скрипты (не рекомендуется)
- `'unsafe-eval'` — разрешает eval() и подобные функции (не рекомендуется)
- `'strict-dynamic'` — позволяет динамическую загрузку скриптов
- URL — разрешает загрузку с указанных доменов

### 3. style-src

Контролирует, откуда могут загружаться CSS-стили.

```
Content-Security-Policy: style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
```

### 4. img-src

Контролирует, откуда могут загружаться изображения.

```
Content-Security-Policy: img-src 'self' data: https://images.example.com;
```

### 5. font-src

Контролирует, откуда могут загружаться шрифты.

```
Content-Security-Policy: font-src 'self' https://fonts.gstatic.com;
```

### 6. connect-src

Контролирует, к каким URL могут отправляться запросы через XHR, fetch и WebSocket.

```
Content-Security-Policy: connect-src 'self' https://api.example.com;
```

### 7. media-src

Контролирует, откуда могут загружаться аудио и видео.

```
Content-Security-Policy: media-src 'self' https://media.example.com;
```

### 8. frame-src

Контролирует, какие ресурсы могут быть встроены в iframe.

```
Content-Security-Policy: frame-src 'self' https://trusted-embed.com;
```

### 9. object-src

Контролирует, откуда могут загружаться плагины (Flash, Java и т.д.).

```
Content-Security-Policy: object-src 'none';
```

## Расширенные директивы

### 1. base-uri

Контролирует, какие URL могут использоваться в теге `<base>`.

```
Content-Security-Policy: base-uri 'self';
```

### 2. child-src

Устаревшая директива, заменена frame-src и worker-src.

### 3. form-action

Контролирует, на какие URL могут отправляться формы.

```
Content-Security-Policy: form-action 'self' https://payment.example.com;
```

### 4. frame-ancestors

Контролирует, на каких сайтах может отображаться страница во фрейме (альтернатива X-Frame-Options).

```
Content-Security-Policy: frame-ancestors 'none';  /* Запрещает фрейминг */
Content-Security-Policy: frame-ancestors 'self';  /* Разрешает только себе */
```

### 5. plugin-types

Контролирует, какие MIME-типы плагинов разрешены.

```
Content-Security-Policy: plugin-types application/pdf;
```

### 6. sandbox

Применяет ограничения к содержимому страницы, как будто она была загружена в sandboxed iframe.

```
Content-Security-Policy: sandbox allow-scripts allow-forms;
```

### 7. upgrade-insecure-requests

Автоматически обновляет HTTP-запросы до HTTPS.

```
Content-Security-Policy: upgrade-insecure-requests;
```

## Директивы отчетности

### 1. report-uri

Указывает URL для отправки отчетов о нарушениях CSP.

```
Content-Security-Policy: default-src 'self'; report-uri /csp-report-endpoint;
```

### 2. report-to

Современная альтернатива report-uri, использующая Reporting API.

```
Content-Security-Policy: default-src 'self'; report-to csp-endpoint;
```

## Значения директив

### 1. Ключевые слова

- `'self'` — тот же источник (домен, протокол, порт)
- `'none'` — ничего не разрешено
- `'unsafe-inline'` — inline-контент (стили, скрипты)
- `'unsafe-eval'` — eval() и подобные функции
- `'strict-dynamic'` — динамическая загрузка скриптов

### 2. Источники

- `https://example.com` — конкретный домен
- `*.example.com` — поддомены
- `https:` — любой HTTPS-ресурс
- `data:` — data: URL
- `blob:` — blob: URL
- `'nonce-*'` — скрипты с конкретным nonce
- `'sha256-*'` — скрипты с конкретным хешем

## Примеры политик CSP

### 1. Строгая политика для сайта без внешних ресурсов

```
Content-Security-Policy: 
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self';
  media-src 'self';
  object-src 'none';
  frame-ancestors 'none';
  form-action 'self';
  base-uri 'self';
```

### 2. Политика для сайта с внешними CDN

```
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://cdn.example.com https://analytics.example.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: blob: https:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.example.com;
  frame-src https://trusted-embed.com;
  object-src 'none';
```

### 3. Политика с nonce для inline-скриптов

```html
<!-- HTML с nonce -->
<script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">
  // Скрипт с nonce
</script>
```

```
Content-Security-Policy: 
  script-src 'self' 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa';
  style-src 'self' 'unsafe-inline';
```

### 4. Политика с хешированием

```
Content-Security-Policy: 
  script-src 'self' 'sha256-hq9b9mNk1a4Zdy7XxH2ZYZJUd4v3lQv2Qr6t5s8v7w=' 'sha256-abc123...';
```

## Эффективное использование директив

### 1. Начинайте с консервативной политики

```javascript
// Начальная политика для разработки
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', 
    "default-src 'self'; " +
    "script-src 'self'; " +
    "style-src 'self' 'unsafe-inline'; " +
    "img-src 'self' data:; " +
    "font-src 'self'; " +
    "connect-src 'self'; " +
    "object-src 'none'; " +
    "report-uri /csp-report"
  );
  next();
});
```

### 2. Пошаговое сужение политики

```javascript
// Пример постепенного ужесточения политики
const cspPolicies = {
  development: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval';",
  staging: "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';",
  production: "default-src 'self'; script-src 'self'; style-src 'self';"
};

app.use((req, res, next) => {
  const env = process.env.NODE_ENV || 'development';
  res.setHeader('Content-Security-Policy', cspPolicies[env]);
  next();
});
```

### 3. Использование nonce для динамических скриптов

```javascript
// Генерация nonce для каждого запроса
function generateNonce() {
  return crypto.randomBytes(16).toString('hex');
}

app.use((req, res, next) => {
  const nonce = generateNonce();
  res.locals.nonce = nonce;
  res.setHeader('Content-Security-Policy', 
    `default-src 'self'; script-src 'self' 'nonce-${nonce}'; style-src 'self' 'unsafe-inline'`
  );
  next();
});

// В шаблоне
app.get('/page', (req, res) => {
  res.render('page', { nonce: res.locals.nonce });
});
```

## Совместимость и браузерная поддержка

### Уровни CSP

- **CSP Level 1** — базовые директивы (широко поддерживается)
- **CSP Level 2** — дополнительные директивы (хорошая поддержка)
- **CSP Level 3** — современные возможности (ограниченная поддержка)

### Поддержка браузерами

- Chrome: с версии 25
- Firefox: с версии 23
- Safari: с версии 7
- Edge: с версии 12
- IE: частичная поддержка через X-Content-Security-Policy

## Лучшие практики

> [!tip] Лучшие практики использования CSP директив
> 1. Начинайте с консервативной политики и постепенно расширяйте
> 2. Используйте nonce или хеши вместо 'unsafe-inline'
> 3. Избегайте 'unsafe-eval' по возможности
> 4. Настройте отчетность для мониторинга нарушений
> 5. Регулярно обновляйте политику по мере изменений в приложении

### Пример безопасной настройки

```javascript
// Комплексная CSP политика
function setCSPHeaders(req, res, next) {
  // Генерация nonce для inline-скриптов
  const nonce = crypto.randomBytes(16).toString('base64');
  
  // Политика CSP
  const cspPolicy = [
    `default-src 'self'`,
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic' https:`,
    `style-src 'self' 'unsafe-inline' https://fonts.googleapis.com`,
    `img-src 'self' data: blob: https:`,
    `font-src 'self' https://fonts.gstatic.com`,
    `connect-src 'self' https://api.example.com`,
    `frame-ancestors 'none'`,
    `object-src 'none'`,
    `base-uri 'self'`,
    `report-uri /csp-report`
  ].join('; ');
  
  res.setHeader('Content-Security-Policy', cspPolicy);
  res.locals.nonce = nonce;
  
  next();
}

app.use(setCSPHeaders);
```

## Связь с другими аспектами безопасности

Директивы CSP тесно связаны с:
- [[Реализация-CSP]] — практическая реализация политик
- [[Отчеты-CSP]] — мониторинг и анализ нарушений
- [[Оценка-CSP]] — оценка эффективности политик
- [[HTTP-Security-Headers]] — общая концепция заголовков безопасности
- [[X-Frame-Options]] — альтернативная защита от фрейминга
- [[XSS-защита]] — предотвращение XSS-атак

## Заключение

Директивы Content Security Policy предоставляют мощный инструмент для ограничения источников контента на веб-странице. Правильная настройка директив позволяет значительно повысить безопасность веб-приложений, предотвращая различные типы атак. Важно тщательно планировать и тестировать политики CSP, чтобы обеспечить как безопасность, так и функциональность приложения.

## Дополнительные ресурсы

- CSP Level 3 Specification
- OWASP Content Security Policy
- Mozilla Developer Network - CSP
- CSP Evaluator Tool