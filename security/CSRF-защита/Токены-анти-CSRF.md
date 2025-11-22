---
aliases: [CSRF токены, Защита от CSRF, Anti-CSRF tokens, CSRF protection]
tags: [security, web-security, csrf, tokens, protection, authentication]
---

# Токены анти-CSRF

## Введение в токены анти-CSRF

Cross-Site Request Forgery (CSRF) — это тип атаки, при которой злонамеренный сайт заставляет браузер аутентифицированного пользователя выполнить нежелательное действие на доверенном сайте. CSRF-атака возможна потому, что браузеры автоматически отправляют с каждым запросом аутентификационные данные (например, куки), независимо от источника запроса.

Токены анти-CSRF представляют собой уникальные, непредсказуемые значения, которые генерируются сервером и отправляются клиенту. При последующих запросах клиент должен предоставить действительный токен, чтобы подтвердить, что запрос исходит от доверенного источника. Это предотвращает выполнение нежелательных действий, инициированных с вредоносных сайтов.

> [!info] Важно
> Токены анти-CSRF работают на основе принципа "предъявления доказательства владения", требуя от клиента предоставить токен, который он мог получить только через легитимный процесс.

## Зачем нужны токены анти-CSRF

Токены анти-CSRF решают фундаментальную проблему безопасности веб-приложений, связанную с доверием браузера к веб-сайтам. Без этой защиты приложения уязвимы к атакам, при которых вредоносные сайты могут заставить браузер пользователя выполнять действия от его имени.

### Основные причины использования

1. **Защита от поддельных запросов**: Предотвращение выполнения нежелательных действий на доверенном сайте
2. **Сохранение целостности данных**: Обеспечение того, что изменения происходят только по инициативе пользователя
3. **Соответствие требованиям безопасности**: Многие стандарты безопасности требуют защиты от CSRF
4. **Доверие пользователей**: Защита конфиденциальных действий пользователя

### Сценарии, требующие защиты

- Финансовые транзакции
- Изменение настроек учетной записи
- Отправка сообщений
- Удаление данных
- Доступ к конфиденциальной информации

## Типы токенов

### Одноразовые токены

Одноразовые токены (one-time tokens) генерируются для каждого запроса и становятся недействительными после использования. Этот подход обеспечивает максимальную безопасность, так как даже если токен будет перехвачен, его нельзя будет использовать повторно.

**Преимущества:**
- Максимальная защита от повторного использования
- Минимизация риска при утечке токена
- Эффективная защита от атак "отправь еще раз"

**Недостатки:**
- Более сложная реализация
- Повышенная нагрузка на сервер
- Потенциальные проблемы с параллельными запросами

### Многоразовые токены

Многоразовые токены (multi-use tokens) действительны в течение определенного периода или сессии. Они могут использоваться для нескольких запросов до истечения срока действия или смены сессии.

**Преимущества:**
- Простота реализации
- Меньшая нагрузка на сервер
- Лучшая производительность

**Недостатки:**
- Меньший уровень безопасности по сравнению с одноразовыми токенами
- Повышенный риск при утечке токена
- Потенциальная уязвимость к атакам с повторным использованием

### Токены с истечением срока действия

Этот тип токенов объединяет преимущества одноразовых и многоразовых токенов, ограничивая срок действия токена определенным временем (например, 30 минут или 24 часа).

## Генерация токенов

### Требования к безопасности

Для обеспечения эффективной защиты CSRF-токены должны соответствовать следующим требованиям безопасности:

- **Криптографическая безопасность**: токены должны генерироваться с использованием криптографически безопасного генератора случайных чисел (CSPRNG)
- **Достаточная длина**: рекомендуется использовать токены длиной не менее 128 бит (16 байт), что эквивалентно 32 символам в шестнадцатеричном формате
- **Непредсказуемость**: токены не должны быть предсказуемыми или генерироваться на основе легко угадываемых данных
- **Уникальность**: каждый токен должен быть уникальным для своей сессии или запроса

### Методы генерации

#### Случайные байты

Самый распространенный и безопасный подход к генерации CSRF-токенов:

```javascript
// Пример на Node.js
const crypto = require('crypto');
const csrfToken = crypto.randomBytes(32).toString('hex');
```

```python
# Пример на Python
import secrets
csrf_token = secrets.token_urlsafe(32)
```

```php
// Пример на PHP
$csrfToken = bin2hex(random_bytes(32));
```

#### На основе соли и сессии

Более сложный подход, при котором токен генерируется на основе данных сессии:

```python
# Пример на Python
import hmac
import hashlib
import secrets

def generate_csrf_token(session_id, secret_key):
    salt = secrets.token_urlsafe(16)
    token = hmac.new(
        secret_key.encode(),
        (session_id + salt).encode(),
        hashlib.sha256
    ).hexdigest()
    return token
```

#### Использование библиотек

Многие фреймворки предоставляют встроенные механизмы генерации CSRF-токенов:
- Django: `django.middleware.csrf.CsrfViewMiddleware`
- Express.js: `csurf` или `express-csrf`
- ASP.NET: `AntiForgery` класс

## Хранение токенов

### В сессии (наиболее распространенный подход)

Самый безопасный способ хранения CSRF-токенов - на сервере в сессии пользователя. Токен хранится на сервере и связан с конкретной сессией пользователя.

**Преимущества:**
- Высокий уровень безопасности
- Невозможность доступа к токену из клиентского кода
- Простая инвалидация токенов

**Недостатки:**
- Требует серверного хранилища сессий
- Может создавать нагрузку на сервер при большом количестве пользователей

```javascript
// Пример на Node.js
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

function generateCsrfToken(req) {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  return req.session.csrfToken;
}
```

### В куки (с дополнительными мерами безопасности)

Хранение токенов в куки на клиенте требует дополнительных мер для предотвращения XSS-атак:

```javascript
// Пример безопасного хранения в куки
res.cookie('csrf-token', token, {
  httpOnly: false,  // Должен быть false, чтобы JavaScript мог получить токен
  secure: true,     // Только по HTTPS
  sameSite: 'strict' // Защита от CSRF для куки
});
```

### В URL (не рекомендуется)

Передача токенов как параметров URL крайне не рекомендуется из-за риска утечки через логи, рефереры и т.д.

## Передача токенов на клиент

### Встраивание в HTML-формы

Самый распространенный способ передачи токена - встраивание его в HTML-форму в виде скрытого поля:

```html
<form method="POST" action="/transfer">
  <input type="hidden" name="csrf_token" value="ABC123DEF456...">
  <!-- остальные поля формы -->
  <button type="submit">Отправить</button>
</form>
```

### Через HTTP-заголовки

Токены могут передаваться через HTTP-заголовки, что особенно полезно для AJAX-запросов:

```javascript
// Установка токена в заголовок
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken
  },
  body: JSON.stringify({
    amount: 100,
    recipient: 'user@example.com'
  })
});
```

### Через мета-теги

Токен может быть передан через мета-тег в HTML-документе:

```html
<meta name="csrf-token" content="ABC123DEF456...">
```

```javascript
// Получение токена из мета-тега
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');
```

### В JavaScript переменных

Токен может быть встроен в JavaScript-код на этапе генерации страницы:

```html
<script>
  window.csrfToken = "{{csrf_token}}";
</script>
```

## Проверка токенов на сервере

### Алгоритм проверки

Процесс проверки CSRF-токенов включает в себя следующие шаги:

1. Извлечение токена из запроса (из формы, заголовка или параметра)
2. Получение ожидаемого токена из сессии или другого безопасного хранилища
3. Сравнение токенов с использованием безопасного метода (например, `hash_equals` в PHP)
4. Принятие решения о продолжении обработки запроса

### Примеры проверки

#### На Node.js

```javascript
function validateCsrfToken(req, res, next) {
  const token = req.body.csrf_token || req.headers['x-csrf-token'];
  if (req.session.csrfToken !== token) {
    return res.status(403).send('CSRF token mismatch');
  }
  next();
}

app.post('/transfer', validateCsrfToken, (req, res) => {
  // Обработка запроса
});
```

#### На Python (Flask)

```python
from flask import request, session, abort

def validate_csrf_token():
    token = request.form.get('csrf_token') or request.headers.get('X-CSRF-Token')
    if token != session.get('csrf_token'):
        abort(403)

@app.route('/transfer', methods=['POST'])
def transfer():
    validate_csrf_token()
    # Обработка запроса
```

#### На PHP

```php
function validateCSRFToken($token) {
    return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!validateCSRFToken($_POST['csrf_token'])) {
        http_response_code(403);
        die('CSRF token mismatch');
    }
    // Обработка запроса
}
```

### Обновление токенов

При использовании одноразовых токенов сервер должен генерировать новые токены после успешной проверки:

```javascript
function validateAndRefreshCsrfToken(req, res, next) {
  const token = req.body.csrf_token || req.headers['x-csrf-token'];
  if (req.session.csrfToken !== token) {
    return res.status(403).send('CSRF token mismatch');
  }
  // Генерируем новый токен для следующего запроса
  req.session.csrfToken = generateCsrfToken();
  next();
}
```

## Примеры реализации

### Простая реализация на Node.js

```javascript
const express = require('express');
const session = require('express-session');
const crypto = require('crypto');

const app = express();

app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

// Генерация CSRF-токена
function generateCsrfToken(req) {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  return req.session.csrfToken;
}

// Проверка CSRF-токена
function validateCsrfToken(req, res, next) {
  const token = req.body.csrf_token || req.headers['x-csrf-token'];
  if (req.session.csrfToken !== token) {
    return res.status(403).send('CSRF token mismatch');
  }
  next();
}

// Middleware для вставки токена в шаблоны
app.use((req, res, next) => {
  res.locals.csrfToken = generateCsrfToken(req);
  next();
});

// Маршрут для формы
app.get('/form', (req, res) => {
  res.send(`
    <form method="POST" action="/submit">
      <input type="hidden" name="csrf_token" value="${res.locals.csrfToken}">
      <input type="text" name="data" placeholder="Введите данные">
      <button type="submit">Отправить</button>
    </form>
  `);
});

// Маршрут для обработки формы
app.post('/submit', validateCsrfToken, (req, res) => {
  res.send('Форма успешно отправлена!');
});

app.listen(3000);
```

### Реализация с использованием Express и csurf

```javascript
const express = require('express');
const session = require('express-session');
const csrf = require('csurf');

const app = express();

app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

const csrfProtection = csrf({ cookie: true });

app.use((req, res, next) => {
  res.locals.csrfToken = req.csrfToken();
  next();
});

app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: res.locals.csrfToken });
});

app.post('/process', csrfProtection, (req, res) => {
  res.send('Форма обработана');
});
```

### Реализация на Django

```python
# views.py
from django.shortcuts import render
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_protect

@csrf_protect
def my_view(request):
    if request.method == 'POST':
        # Обработка формы
        data = request.POST.get('data')
        return HttpResponse(f'Данные: {data}')
    return render(request, 'form.html')
```

```html
<!-- form.html -->
<form method="post">
    {% csrf_token %}
    <input type="text" name="data" placeholder="Введите данные">
    <button type="submit">Отправить</button>
</form>
```

## Плюсы и минусы подхода

### Плюсы

- **Высокий уровень безопасности**: Эффективная защита от CSRF-атак
- **Гибкость**: Возможность адаптации под различные сценарии использования
- **Широкая поддержка**: Поддерживается большинством современных фреймворков
- **Совместимость**: Работает с различными типами запросов и форм

### Минусы

- **Сложность реализации**: Требует тщательной проработки всех аспектов
- **Зависимость от сессий**: В большинстве случаев требует серверного хранения сессий
- **Потенциальные проблемы с производительностью**: Дополнительная нагрузка на сервер
- **Неэффективность против XSS**: Токены могут быть украдены при XSS-атаках

## Возможные уязвимости в реализации

### Неправильная генерация токенов

Использование предсказуемых или слабых токенов делает защиту бессмысленной:

```javascript
// ПЛОХО: Предсказуемые токены
const weakToken = Date.now().toString(); // Легко предсказуемо

// ХОРОШО: Криптографически безопасные токены
const strongToken = crypto.randomBytes(32).toString('hex');
```

### Утечка токенов через URL

Передача токенов в URL может привести к их утечке через логи, рефереры и т.д.

### Неправильная проверка токенов

Небезопасное сравнение токенов может привести к обходу защиты:

```php
// ПЛОХО: Небезопасное сравнение
if ($_POST['token'] == $_SESSION['token']) { /* ... */ }

// ХОРОШО: Безопасное сравнение
if (hash_equals($_SESSION['token'], $_POST['token'])) { /* ... */ }
```

### Отсутствие защиты от XSS

Если приложение уязвимо к XSS-атакам, токены могут быть украдены через клиентский код.

### Параллельные запросы

Неправильная обработка параллельных запросов может привести к проблемам с одноразовыми токенами.

## Лучшие практики

### Безопасная генерация токенов

- Использовать криптографически безопасные генераторы случайных чисел
- Обеспечить достаточную длину токенов (не менее 128 бит)
- Использовать непредсказуемые значения

### Правильное хранение

- Хранить токены в сессии на сервере
- Не хранить токены в логах или отладочной информации
- Использовать безопасные соединения (HTTPS) для передачи токенов

### Проверка и валидация

- Использовать безопасные методы сравнения токенов
- Проверять токены для всех чувствительных операций
- Обрабатывать ошибки корректно, не раскрывая детали безопасности

### Обновление и инвалидация

- Обновлять токены при важных событиях (например, изменение пароля)
- Инвалидировать токены при завершении сессии
- Использовать токены с ограниченным сроком действия

### Комплексная защита

- Комбинировать CSRF-токены с другими методами защиты (SameSite куки, CSP и т.д.)
- Регулярно обновлять и тестировать защиту
- Обучать разработчиков лучшим практикам безопасности

## Альтернативы токенам

### SameSite куки

Атрибут SameSite для куки автоматически защищает от CSRF-атак на уровне браузера:

```javascript
// Установка куки с атрибутом SameSite
res.cookie('sessionid', sessionId, {
  sameSite: 'strict',  // или 'lax'
  secure: true,
  httpOnly: true
});
```

### Double Submit Cookie

Метод, при котором токен отправляется как в куки, так и в теле запроса, и сервер сравнивает их:

```javascript
// Установка токена в куки
res.cookie('csrf-token', token, { sameSite: 'strict', secure: true });

// Клиент отправляет токен в заголовке
fetch('/api/action', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': token
  }
});
```

### Referer/Origin заголовки

Проверка заголовков Referer или Origin для определения источника запроса:

```javascript
function validateOrigin(req) {
  const origin = req.headers.origin;
  const host = req.headers.host;
  
  if (origin && new URL(origin).host !== host) {
    return false; // Подозрительный источник
  }
  return true;
}
```

### Усиленная аутентификация

Для критических операций требовать дополнительную аутентификацию (например, повторный ввод пароля или SMS-код):

```javascript
// Проверка дополнительной аутентификации для критических операций
function requireAdditionalAuth(req, operation) {
  if (isCriticalOperation(operation)) {
    return checkAdditionalAuth(req);
  }
  return true;
}
```

## Связанные материалы

Для более полного понимания темы рекомендуется ознакомиться с следующими материалами:

- [[CSRF-защита]]
- [[Типы-CSRF-атак]]
- [[SameSite-куки]]
- [[Double-Submit-Cookie]]
- [[Методы-защиты-от-CSRF]]
- [[XSS-защита]]
- [[HTTP-Security-Headers]]
- [[Content-Security-Policy]]
- [[Управление-сессиями-и-аутентификацией]]
- [[Защита-от-инъекций]]
- [[Secure-Storage]]
- [[Clickjacking-защита]]
- [[Ограничение-доступа-к-API]]
- [[Безопасность-в-веб-приложениях-с-AI-ML]]
- [[Безопасность-в-SPA]]
- [[Интеграция-с-OAuth]]
- [[Безопасность-в-OIDC]]
- [[Управление-доступом]]
- [[Тестирование-безопасности]]
- [[Аудит-безопасности]]