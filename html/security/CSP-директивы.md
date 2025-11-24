---
aliases: [CSP, Content Security Policy, Политика безопасности содержимого]
tags: [html, security, csp, web-security, best-practices]
---

# CSP-директивы (Content Security Policy)

## Введение

Content Security Policy (CSP) - это важный механизм безопасности, который помогает предотвратить межсайтовый скриптинг (XSS), кликджекинг и другие атаки, связанные с внедрением вредоносного кода. CSP позволяет веб-сайтам объявлять разрешенные источники контента, которые браузер может загружать и выполнять.

## Основные директивы CSP

### `default-src`
Это основная директива, которая определяет политику для всех ресурсов, если для них не указана специфическая директива. Это базовое правило, от которого наследуются другие директивы.

```http
Content-Security-Policy: default-src 'self';
```

### `script-src`
Определяет разрешенные источники для выполнения JavaScript. Критически важна для предотвращения XSS-атак.

```http
Content-Security-Policy: script-src 'self' 'unsafe-inline' 'unsafe-eval';
```

> [!warning] Важно
> Использование `'unsafe-inline'` и `'unsafe-eval'` снижает эффективность CSP и не рекомендуется в продакшене.

### `style-src`
Определяет разрешенные источники для CSS-стилей.

```http
Content-Security-Policy: style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
```

### `img-src`
Указывает разрешенные источники для изображений.

```http
Content-Security-Policy: img-src 'self' data: https://*.example.com;
```

### `font-src`
Определяет разрешенные источники для шрифтов.

```http
Content-Security-Policy: font-src 'self' https://fonts.gstatic.com;
```

### `connect-src`
Ограничивает источники, к которым может обращаться JavaScript через XHR, Fetch API, WebSocket и EventSource.

```http
Content-Security-Policy: connect-src 'self' https://api.example.com;
```

### `frame-src`
Определяет разрешенные источники для встраивания iframe.

```http
Content-Security-Policy: frame-src https://example.com;
```

### `object-src`
Ограничивает загрузку плагинов (Flash, Java и т.д.).

```http
Content-Security-Policy: object-src 'none';
```

## Значения директив

### `'self'`
Разрешает загрузку ресурсов с того же источника (домен, протокол и порт).

### `'none'`
Запрещает загрузку соответствующего типа ресурсов.

### `'unsafe-inline'`
Разрешает встроенные скрипты и стили (не рекомендуется использовать).

### `'unsafe-eval'`
Разрешает использование `eval()` и подобных функций (не рекомендуется использовать).

### `'strict-dynamic'`
Механизм, позволяющий динамически загружать скрипты, разрешенные через `'nonce'` или `'hash'`.

### `'nonce-*'`
Разрешает выполнение скриптов с определенным nonce-значением.

### `'hash-*'`
Разрешает выполнение скриптов с определенным хешем.

## Практические рекомендации

### 1. Начните с разработки строгой политики

```http
Content-Security-Policy: 
  default-src 'none';
  script-src 'self' 'nonce-abc123';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self';
  frame-src 'none';
  object-src 'none';
```

### 2. Используйте Report-URI для мониторинга

```http
Content-Security-Policy-Report-Only: 
  default-src 'self';
  report-uri /csp-report;
```

### 3. Применение nonce и hash для inline-контента

Вместо `'unsafe-inline'` используйте:

```html
<script nonce="abc123">
  // Ваш JavaScript код
</script>
```

С политикой:
```
script-src 'self' 'nonce-abc123';
```

### 4. Проверка совместимости с российскими сервисами (2025)

В 2025 году при реализации CSP в российских веб-приложениях важно учитывать:

- **Локализованные CDN**: При использовании российских CDN-провайдеров (например, [[Яндекс.Облако]], [[Mail.ru Cloud Solutions]]) необходимо включать их домены в соответствующие директивы
- **Региональные домены**: Включение доменов российских сервисов в директивы (например, `mc.yandex.ru`, `sbercloud.ru`)
- **Соответствие требованиям**: Учитывайте требования российского законодательства в области информационной безопасности

### 5. Постепенное внедрение

Рекомендуется использовать сначала `Content-Security-Policy-Report-Only` для выявления проблем без блокировки контента:

```
Content-Security-Policy-Report-Only: 
  default-src 'self';
  script-src 'self';
  report-uri https://your-domain.ru/csp-report;
```

## Примеры политик для разных типов приложений

### Простое веб-приложение

```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline';
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: blob: https:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.yourdomain.com;
```

### Современное SPA-приложение

```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'nonce-abc123' 'strict-dynamic';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self' https://api.yourdomain.com;
  object-src 'none';
  base-uri 'none';
  form-action 'self';
```

## Учет российских реалий 2025 года

### 1. Работа с российскими аналитическими системами

При использовании российских систем аналитики (например, Яндекс.Метрика):

```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' mc.yandex.ru;
  img-src 'self' data: blob: mc.yandex.ru;
  connect-src 'self' mc.yandex.ru;
```

### 2. Интеграция с российскими платежными системами

При интеграции с российскими платежными системами (например, [[Платежные системы в России]]):

```http
Content-Security-Policy: 
  default-src 'self';
  frame-src 'self' secure.payments.ru;
  connect-src 'self' api.payments.ru;
```

### 3. Соответствие требованиям Рунета

С учетом особенностей российского сегмента интернета (Рунета), рекомендуется:

- Включать российские домены в соответствующие директивы
- Обеспечивать совместимость с российскими браузерами
- Учитывать особенности российских CDN-провайдеров

## Тестирование CSP

### Проверка в браузере

1. Откройте DevTools (F12)
2. Перейдите во вкладку Console
3. Найдите CSP-ошибки при загрузке страницы

### Использование инструментов

- [[CSP Evaluator]] - инструмент Google для проверки CSP
- [[Security Headers]] - проверка заголовков безопасности
- [[OWASP ZAP]] - инструмент тестирования безопасности

## Частые ошибки при реализации CSP

1. **Слишком широкие разрешения** - использование `'unsafe-inline'` и `'unsafe-eval'`
2. **Неполное покрытие** - отсутствие директив для всех типов ресурсов
3. **Неправильные домены** - ошибки в перечислении разрешенных источников
4. **Отсутствие тестирования** - не проверять политику перед продакшеном

## Совместимость с браузерами

| Браузер | Поддержка CSP |
|---------|---------------|
| Chrome | CSP Level 3 |
| Firefox | CSP Level 3 |
| Safari | CSP Level 2 |
| Edge | CSP Level 3 |

## Заключение

Content Security Policy - это мощный инструмент защиты веб-приложений от атак, связанных с внедрением кода. При правильной реализации CSP может значительно повысить безопасность вашего веб-приложения. В 2025 году особенно важно учитывать региональные особенности при реализации CSP в российских веб-приложениях.

## См. также

- [[XSS-атаки]]
- [[Безопасность веб-приложений]]
- [[HTTPS и SSL/TLS]]
- [[CORS-политика]]
- [[OWASP Top 10]]
- [[Платежные системы в России]]
- [[Яндекс.Облако]]
- [[Mail.ru Cloud Solutions]]