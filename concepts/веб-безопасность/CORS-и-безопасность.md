---
aliases: ["CORS", "Cross-Origin Resource Sharing", "Безопасность CORS"]
tags: ["#security", "#cors", "#web-security", "#frontend-security", "#http"]
---

# CORS и безопасность

**Cross-Origin Resource Sharing (CORS)** — это механизм, который позволяет веб-странице запрашивать ресурсы с другого домена, отличного от домена, с которого была загружена страница. Это важный аспект безопасности веб-приложений, который помогает предотвратить нежелательные межсайтовые запросы.

## Что такое CORS?

CORS — это спецификация W3C, которая позволяет серверу указать, какие домены могут получать доступ к его ресурсам. Это предотвращает выполнение запросов с других доменов без разрешения, что защищает от межсайтовых атак.

## Как работает CORS?

Когда браузер делает запрос к другому домену, он отправляет предварительный запрос (preflight) с методом `OPTIONS`, чтобы проверить, разрешен ли такой запрос. Если сервер отвечает с соответствующими заголовками, браузер выполняет основной запрос.

### Заголовки CORS

- `Access-Control-Allow-Origin`: указывает, какие домены могут получить доступ к ресурсам.
- `Access-Control-Allow-Methods`: указывает, какие HTTP-методы разрешены.
- `Access-Control-Allow-Headers`: указывает, какие заголовки могут быть использованы в запросе.
- `Access-Control-Allow-Credentials`: указывает, можно ли включать учетные данные в запрос.

## Практические примеры

### Неправильная настройка CORS

```javascript
// НЕБЕЗОПАСНО: разрешение доступа для всех доменов
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type');
  next();
});
```

### Правильная настройка CORS

```javascript
// БЕЗОПАСНО: разрешение доступа только для доверенных доменов
const allowedOrigins = ['https://trusted-domain.com', 'https://another-trusted.com'];

app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  next();
});
```

## Уязвимости и риски

### 1. Чрезмерно широкие разрешения

Разрешение доступа для всех доменов (`*`) может привести к утечке данных или выполнению нежелательных действий.

### 2. Неправильная обработка учетных данных

Если `Access-Control-Allow-Credentials` установлен в `true`, но `Access-Control-Allow-Origin` равен `*`, это может привести к краже учетных данных.

### 3. Атаки через CORS

Атакующий может использовать уязвимости CORS для доступа к приватным данным или выполнения действий от имени пользователя.

## Рекомендации по безопасности

### 1. Используйте точные домены

Вместо `*` указывайте конкретные домены, которым разрешен доступ:

```javascript
// Правильно
res.header('Access-Control-Allow-Origin', 'https://trusted-domain.com');
```

### 2. Проверяйте заголовки Origin

Всегда проверяйте значение заголовка `Origin` и сверяйтесь с белым списком доверенных доменов:

```javascript
const isValidOrigin = allowedOrigins.includes(req.headers.origin);
if (!isValidOrigin) {
  return res.status(403).send('Forbidden');
}
```

### 3. Не разрешайте учетные данные с wildcard

Если вы используете `Access-Control-Allow-Credentials: true`, не устанавливайте `Access-Control-Allow-Origin: *`:

```javascript
// Правильно
res.header('Access-Control-Allow-Origin', 'https://trusted-domain.com');
res.header('Access-Control-Allow-Credentials', 'true');
```

### 4. Минимизируйте разрешенные методы и заголовки

Разрешайте только те методы и заголовки, которые действительно необходимы:

```javascript
res.header('Access-Control-Allow-Methods', 'GET, POST');
res.header('Access-Control-Allow-Headers', 'Content-Type');
```

## Примеры атак и защиты

### Атака с использованием поддельного Origin

```javascript
// Атакующий может подделать заголовок Origin
// Сервер должен проверять его и не доверять без проверки
fetch('https://victim.com/api/data', {
  method: 'GET',
  headers: {
    'Origin': 'https://attacker.com'  // Поддельный Origin
  }
});
```

### Защита от поддельного Origin

```javascript
// Сервер должен проверять Origin и сверяться с белым списком
const trustedOrigins = ['https://trusted.com'];
const requestOrigin = req.headers.origin;

if (!trustedOrigins.includes(requestOrigin)) {
  return res.status(403).send('Forbidden');
}
```

## Связанные темы

- [[OWASP-Top-10]]
- [[Защита-от-XSS]]
- [[Защита-от-CSRF]]
- [[HTTPS-и-шифрование]]

## Заключение

CORS — мощный инструмент, но его неправильная настройка может привести к серьезным уязвимостям. Фронтенд-разработчики должны понимать, как работает CORS, и сотрудничать с бэкенд-разработчиками для обеспечения безопасной настройки.

> [!tip] Совет
> Используйте инструменты разработчика в браузере для проверки CORS-заголовков и отладки проблем с межсайтовыми запросами.

> [!warning] Важно
> Никогда не полагайтесь только на CORS для защиты чувствительных данных. Это дополнительный слой безопасности, но не замена аутентификации и авторизации.