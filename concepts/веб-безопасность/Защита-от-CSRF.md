---
aliases: ["CSRF", "Cross-Site Request Forgery", "Защита от подделывания межсайтовых запросов"]
tags: ["#security", "#csrf", "#web-security", "#frontend-security", "#authentication"]
---

# Защита от CSRF

**Cross-Site Request Forgery (CSRF)** — это тип атаки, при котором злоумышленник заставляет пользователя выполнить нежелательное действие на сайте, на котором пользователь аутентифицирован. Это может привести к изменению данных, выполнению транзакций или другим нежелательным действиям без ведома пользователя.

## Как работает CSRF-атака?

CSRF-атака основывается на том, что браузер автоматически отправляет куки и другие учетные данные вместе с каждым запросом к домену, на котором пользователь аутентифицирован. Злоумышленник может создать страницу, которая отправляет запрос к целевому сайту, и если пользователь посетит эту страницу, запрос будет выполнен от его имени.

### Пример CSRF-атаки

```html
<!-- Злоумышленник создает страницу с формой -->
<html>
  <body>
    <h1>Вы выиграли миллион долларов!</h1>
    <form action="https://victim.com/transfer" method="POST">
      <input type="hidden" name="to" value="attacker@evil.com" />
      <input type="hidden" name="amount" value="1000000" />
      <input type="submit" value="Забрать приз!" />
    </form>
    <script>
      // Автоматически отправляем форму при загрузке страницы
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Если пользователь посетит эту страницу, будучи аутентифицированным на `victim.com`, запрос на перевод средств будет выполнен от его имени.

## Методы защиты от CSRF

### 1. CSRF-токены

Самый распространенный метод защиты — использование CSRF-токенов. Сервер генерирует уникальный токен для каждой сессии или запроса и включает его в формы и AJAX-запросы. При получении запроса сервер проверяет наличие и правильность токена.

#### Реализация на фронтенде

```javascript
// Получаем CSRF-токен из meta-тега или заголовка
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

// Добавляем токен в каждую форму
document.querySelectorAll('form').forEach(form => {
  const tokenInput = document.createElement('input');
  tokenInput.type = 'hidden';
  tokenInput.name = '_csrf';
  tokenInput.value = csrfToken;
  form.appendChild(tokenInput);
});

// Добавляем токен в AJAX-запросы
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken
  },
  body: JSON.stringify({
    to: 'recipient@example.com',
    amount: 100
  })
});
```

### 2. Проверка заголовка Origin и Referer

Сервер может проверять заголовки `Origin` и `Referer`, чтобы убедиться, что запрос исходит с доверенного домена.

```javascript
// Пример проверки Origin на сервере
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const trustedOrigins = ['https://trusted.com', 'https://sub.trusted.com'];
  
  if (origin && !trustedOrigins.includes(origin)) {
    return res.status(403).send('Forbidden');
  }
  
  next();
});
```

### 3. SameSite-атрибут куки

Атрибут `SameSite` для куки предотвращает отправку куки в межсайтовых запросах. Возможные значения:
- `Strict`: куки отправляются только в рамках одного сайта
- `Lax`: куки отправляются в некоторых случаях межсайтовых запросов (например, при переходе по ссылке)
- `None`: куки отправляются во всех случаях (требует `Secure`)

```javascript
// Установка куки с SameSite-атрибутом
res.cookie('session', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'  // или 'lax'
});
```

### 4. Double Submit Cookie

Метод, при котором CSRF-токен сохраняется как куки и отправляется в теле запроса или заголовке. Сервер сравнивает токен из куки с токеном из запроса.

```javascript
// Фронтенд: получаем токен из куки и отправляем в заголовке
const csrfToken = document.cookie
  .split('; ')
  .find(row => row.startsWith('csrfToken='))
  .split('=')[1];

fetch('/api/action', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken
  },
  body: JSON.stringify(data)
});
```

## Практические рекомендации

### 1. Используйте CSRF-токены для всех изменяющих данных запросов

Все POST, PUT, DELETE и другие методы, изменяющие данные, должны использовать CSRF-токены.

### 2. Не полагайтесь только на проверку Origin/Referer

Эти заголовки могут быть отсутствовать или подделаны, поэтому они не обеспечивают полной защиты.

### 3. Обновляйте токены при изменении сессии

При каждом входе/выходе пользователя или изменении сессии генерируйте новый CSRF-токен.

### 4. Используйте SameSite-атрибут для сессионных куки

Это дополнительный уровень защиты, особенно эффективный против CSRF-атак.

## Пример реализации защиты в приложении

```javascript
// Генерация CSRF-токена на сервере (Node.js + Express)
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Отправка токена клиенту
app.get('/form', (req, res) => {
  res.send(`
    <form action="/submit" method="POST">
      <input type="hidden" name="_csrf" value="${req.csrfToken()}">
      <input type="text" name="data">
      <button type="submit">Отправить</button>
    </form>
  `);
});

// Защита маршрута
app.post('/submit', csrfProtection, (req, res) => {
  // Обработка данных
  res.send('Данные успешно отправлены');
});
```

## Связанные темы

- [[OWASP-Top-10]]
- [[CORS-и-безопасность]]
- [[Защита-от-XSS]]
- [[HTTPS-и-шифрование]]

## Заключение

CSRF — серьезная угроза безопасности, особенно для приложений с аутентификацией. Фронтенд-разработчики должны сотрудничать с бэкенд-разработчиками для реализации эффективной защиты, включая использование CSRF-токенов, правильную настройку куки и проверку заголовков.

> [!tip] Совет
> Используйте библиотеки и фреймворки, которые автоматически обеспечивают защиту от CSRF, такие как `csurf` для Express или встроенные механизмы в Django, Rails и других.

> [!warning] Важно
> Не используйте GET-запросы для изменяющих данных действий. Всегда используйте POST, PUT, PATCH или DELETE для действий, изменяющих состояние сервера.