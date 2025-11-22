---
aliases: [CSRF токены, Защита от CSRF, CSRF защита]
tags: [security, web-security, csrf, tokens, protection]
---

# Анти-CSRF токены

## Введение в анти-CSRF токены

Cross-Site Request Forgery (CSRF) — это тип атаки, при которой злонамеренный сайт заставляет браузер пользователя выполнить нежелательное действие на доверенном сайте, где пользователь в настоящее время аутентифицирован. CSRF-атака возможна потому, что браузеры автоматически отправляют с каждым запросом аутентификационные данные (например, куки), независимо от того, откуда исходит запрос — с доверенного сайта или с вредоносного.

Анти-CSRF токены представляют собой уникальные, непредсказуемые значения, которые генерируются сервером и отправляются клиенту. При последующих запросах клиент должен предоставить действительный токен, чтобы подтвердить, что запрос исходит от доверенного источника. Это предотвращает выполнение нежелательных действий, инициированных с вредоносных сайтов.

### Пример CSRF-атаки

Представим, что пользователь аутентифицирован на банковском сайте и посещает вредоносный сайт. Вредоносный сайт может содержать изображение с URL, которое на самом деле является запросом на перевод денег:

```html
<img src="https://bank.example.com/transfer?to=attacker&amount=1000" width="0" height="0">
```

Если банк не защищен от CSRF, браузер пользователя автоматически отправит куки аутентификации вместе с этим запросом, и перевод средств может произойти без ведома пользователя.

### Зачем нужны анти-CSRF токены

Анти-CSRF токены решают эту проблему, обеспечивая, что запрос может быть успешно выполнен только если он содержит правильный токен. Вредоносный сайт не может получить этот токен из-за Same-Origin Policy, которая запрещает доступ к содержимому другого домена.

### Важность CSRF-защиты

CSRF-атаки могут привести к серьезным последствиям:
- Нежелательные финансовые транзакции
- Изменение конфиденциальных настроек пользователя
- Выполнение административных действий
- Раскрытие чувствительной информации

Поэтому внедрение надежной защиты от CSRF является критически важным элементом безопасности веб-приложений.

## Как работают анти-CSRF токены

Анти-CSRF токены работают на основе принципа "предъявления доказательства владения". Механизм защиты включает в себя следующие этапы:

### 1. Генерация токена

При первом посещении защищенной страницы или при начале сессии сервер генерирует уникальный, криптографически безопасный токен. Этот токен должен быть непредсказуемым и иметь достаточную длину, чтобы предотвратить угадывание.

### 2. Передача токена клиенту

Сервер передает сгенерированный токен клиенту (браузеру) одним из следующих способов:
- Встраивание в HTML-форму в виде скрытого поля (`<input type="hidden">`)
- Передача через HTTP-заголовок
- Включение в JavaScript переменную
- Установка в специальный HTTP-заголовок при AJAX-запросах

### 3. Сохранение токена на сервере

Сервер сохраняет токен в сессии пользователя или в безопасном хранилище, связанном с конкретной сессией или пользователем. Это позволяет серверу проверить подлинность токена при получении запроса.

### 4. Возврат токена при запросе

При отправке формы или выполнении чувствительного запроса клиент должен включить токен в запрос. Это может быть:
- Скрытое поле формы
- HTTP-заголовок (например, `X-CSRF-Token`)
- Параметр запроса
- Часть тела JSON-запроса

### 5. Проверка токена

При получении запроса сервер:
- Извлекает токен из запроса
- Сравнивает его с токеном, сохраненным в сессии
- Если токены совпадают, запрос считается легитимным
- Если токены не совпадают или токен отсутствует, запрос отклоняется

### Пример схемы работы

```
Клиент (браузер)                    Сервер
     |                                  |
     | ←−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−− |
     |        CSRF токен (ABC123)       |
     | −−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−→ |
     |     (с запросом, напр. POST)     |
     |                                  |
     |                                  |
     | ←−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−− |
     |        Ответ (успешно/ошибка)     |
     |                                  |
```

### Защита от атак

Этот механизм защищает от CSRF-атак, потому что:
- Вредоносный сайт не может получить токен из-за Same-Origin Policy
- Вредоносный сайт не может предугадать криптографически безопасный токен
- Даже если вредоносный сайт каким-то образом узнает токен, он действителен только для конкретной сессии
- Токены могут быть одноразовыми или иметь ограниченное время жизни

### Варианты реализации

Существуют различные стратегии использования CSRF-токенов:
- **Одноразовые токены**: токен используется один раз и затем генерируется новый
- **Многоразовые токены**: токен может использоваться несколько раз в течение сессии
- **Токены с истечением срока действия**: токены действительны только в течение определенного времени
- **Токены на основе сессии**: токен привязан к конкретной сессии пользователя

## Генерация и управление токенами

Генерация и управление CSRF-токенами — критически важные аспекты эффективной защиты от CSRF-атак. Рассмотрим подробно, как правильно реализовать эти процессы.

### Генерация CSRF-токенов

#### Требования к токенам

Для обеспечения безопасности CSRF-токены должны соответствовать следующим требованиям:

- **Криптографическая безопасность**: токены должны генерироваться с использованием криптографически безопасного генератора случайных чисел (CSPRNG)
- **Достаточная длина**: рекомендуется использовать токены длиной не менее 128 бит (16 байт), что эквивалентно 32 символам в шестнадцатеричном формате
- **Непредсказуемость**: токены не должны быть предсказуемыми или генерироваться на основе легко угадываемых данных
- **Уникальность**: каждый токен должен быть уникальным для своей сессии или запроса

#### Методы генерации

Существует несколько подходов к генерации CSRF-токенов:

**1. Случайные байты**

```javascript
// Пример на Node.js
const crypto = require('crypto');
const csrfToken = crypto.randomBytes(32).toString('hex');
```

**2. На основе соли и сессии**

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

**3. Использование библиотек**

Многие фреймворки предоставляют встроенные механизмы генерации CSRF-токенов:
- Django: `django.middleware.csrf.CsrfViewMiddleware`
- Express.js: `csurf` или `express-csrf`
- ASP.NET: `AntiForgery` класс

### Управление токенами

#### Хранение токенов

Существует несколько стратегий хранения CSRF-токенов:

**1. В сессии (наиболее распространенный подход)**

- Токен хранится на сервере в сессии пользователя
- Клиент получает только идентификатор, а токен проверяется на сервере
- Обеспечивает высокий уровень безопасности
- Требует серверного хранилища сессий

**2. В куки (с дополнительными мерами безопасности)**

- Токен хранится в куки на клиенте
- Требует дополнительных мер для предотвращения XSS-атак
- Может быть использован в stateless приложениях

**3. В URL (не рекомендуется)**

- Токен передается как параметр URL
- Уязвим для утечки через логи, рефереры и т.д.
- Не рекомендуется использовать

#### Стратегии управления

**1. Одноразовые токены**

- Каждый токен используется только один раз
- После использования генерируется новый токен
- Обеспечивает максимальную безопасность
- Требует больше ресурсов для управления

**2. Многоразовые токены**

- Один токен может использоваться несколько раз в течение сессии
- Менее ресурсоемкий подход
- Требует дополнительных мер безопасности

**3. Токены с истечением срока действия**

- Токены действительны только в течение определенного времени
- Предотвращает использование устаревших токенов
- Балансирует безопасность и удобство использования

#### Пример реализации управления токенами

```javascript
// Пример на Node.js с использованием Express и сессий
const session = require('express-session');
const crypto = require('crypto');

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

function validateCsrfToken(req, token) {
  return req.session.csrfToken === token;
}

// Middleware для генерации токена
app.use('/protected', (req, res, next) => {
  res.locals.csrfToken = generateCsrfToken(req);
  next();
});

// Middleware для проверки токена
app.post('/protected', (req, res, next) => {
  const token = req.body.csrfToken || req.headers['x-csrf-token'];
  if (!validateCsrfToken(req, token)) {
    return res.status(403).send('CSRF token mismatch');
  }
  // Генерируем новый токен для следующего запроса (если используем одноразовые)
  req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  next();
});
```

### Безопасность токенов

При управлении CSRF-токенами необходимо учитывать следующие аспекты безопасности:

- **Защита от XSS**: токены могут быть украдены через XSS-атаки, поэтому важно защищать приложение от них
- **Логирование**: токены не должны записываться в логи или отображаться в отладочной информации
- **Передача**: токены должны передаваться по защищенным каналам (HTTPS)
- **Хранение**: токены не должны храниться в открытом виде в базе данных или файлах

## Реализация на стороне сервера и клиента

Реализация защиты от CSRF требует координированного подхода между серверной и клиентской сторонами. Рассмотрим подробно, как реализовать каждый аспект защиты.

### Серверная реализация

#### 1. Генерация токенов

Сервер должен генерировать уникальные CSRF-токены для каждой сессии или запроса:

```javascript
// Пример генерации токена на Node.js
const crypto = require('crypto');

function generateCsrfToken() {
  return crypto.randomBytes(32).toString('hex');
}
```

```python
# Пример генерации токена на Python
import secrets

def generate_csrf_token():
    return secrets.token_urlsafe(32)
```

#### 2. Хранение токенов

Токены должны быть связаны с сессией пользователя:

```javascript
// Пример хранения токена в сессии на Node.js
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

function setCsrfToken(req) {
  if (!req.session.csrfToken) {
    req.session.csrfToken = generateCsrfToken();
  }
  return req.session.csrfToken;
}
```

#### 3. Встраивание токена в HTML

Сервер должен включать токен в формы и страницы:

```html
<!-- Встраивание токена в HTML-форму -->
<form method="POST" action="/transfer">
  <input type="hidden" name="csrf_token" value="{{csrf_token}}">
  <!-- остальные поля формы -->
  <button type="submit">Отправить</button>
</form>
```

#### 4. Проверка токенов

Сервер должен проверять токены при получении запросов:

```javascript
// Пример проверки токена на Node.js
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

```python
# Пример проверки токена на Python (Flask)
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

#### 5. Обновление токенов

При использовании одноразовых токенов, сервер должен генерировать новые токены:

```javascript
// Обновление токена после успешной проверки
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

### Клиентская реализация

#### 1. Получение токена

Клиент может получить токен несколькими способами:

**Из HTML-формы:**
```javascript
// Получение токена из скрытого поля формы
const csrfToken = document.querySelector('input[name="csrf_token"]').value;
```

**Из мета-тега:**
```html
<meta name="csrf-token" content="{{csrf_token}}">
```
```javascript
// Получение токена из мета-тега
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');
```

**Из HTTP-заголовка:**
```javascript
// Токен может быть передан в заголовке ответа сервера
const csrfToken = xhr.getResponseHeader('X-CSRF-Token');
```

#### 2. Включение токена в запросы

**Для обычных HTML-форм:**
Токен уже встроен в форму как скрытое поле.

**Для AJAX-запросов:**
```javascript
// Включение токена в AJAX-запросы
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

```javascript
// Установка токена для всех jQuery AJAX-запросов
$.ajaxSetup({
  beforeSend: function(xhr, settings) {
    if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader("X-CSRF-Token", $('meta[name=csrf-token]').attr('content'));
    }
  }
});
```

#### 3. Обновление токенов

При получении нового токена от сервера (например, в ответе на AJAX-запрос), клиент должен обновить его локально:

```javascript
// Обновление токена после AJAX-запроса
fetch('/api/action', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': currentToken
  },
  // ...
})
.then(response => {
  // Получаем новый токен из заголовка ответа
  const newToken = response.headers.get('X-CSRF-Token');
  if (newToken) {
    currentToken = newToken;
    // Обновляем токен в мета-теге или скрытом поле
    document.querySelector('meta[name="csrf-token"]').setAttribute('content', newToken);
  }
});
```

### Совместная реализация

#### 1. Middleware и библиотеки

Многие фреймворки предоставляют встроенные решения:

**Django:**
```python
# Django автоматически проверяет токены
from django.views.decorators.csrf import csrf_protect

@csrf_protect
def my_view(request):
    # Django проверит токен автоматически
    pass
```

**Express.js:**
```javascript
// Использование csurf middleware
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.get('/form', csrfProtection, (req, res) => {
  res.render('send', { csrfToken: req.csrfToken() });
});

app.post('/process', parseForm, csrfProtection, (req, res) => {
  res.send('Отправлено');
});
```

#### 2. Совместимость с SPA

Для одностраничных приложений (SPA) важно правильно управлять токенами:

```javascript
// Пример для React-приложения
import { useState, useEffect } from 'react';

function useCsrfToken() {
  const [token, setToken] = useState('');

  useEffect(() => {
    // Получаем токен с сервера при инициализации
    fetch('/api/csrf-token')
      .then(response => response.json())
      .then(data => setToken(data.token));
  }, []);

  return token;
}
```

#### 3. Обработка ошибок

Оба уровня должны корректно обрабатывать ошибки CSRF-проверки:

**Сервер:**
- Возвращать соответствующий HTTP-статус (обычно 403 Forbidden)
- Не раскрывать детали ошибки в целях безопасности

**Клиент:**
- Обрабатывать ошибки CSRF и, при необходимости, запрашивать новый токен
- Показывать пользователю понятное сообщение об ошибке

### Лучшие практики совместной реализации

1. **Единая стратегия**: согласовать подход к токенам между сервером и клиентом
2. **Безопасная передача**: всегда использовать HTTPS для передачи токенов
3. **Проверка на обоих уровнях**: не полагаться только на клиентскую проверку
4. **Регулярное обновление**: обновлять токены при важных действиях
5. **Тестирование**: тестировать защиту на наличие уязвимостей

## Примеры кода для различных языков/фреймворков

В этом разделе представлены примеры реализации CSRF-защиты с использованием различных языков программирования и фреймворков.

### Node.js (Express)

#### Установка и настройка

```bash
npm install express express-session csurf
```

#### Пример реализации

```javascript
const express = require('express');
const session = require('express-session');
const csrf = require('csurf');

const app = express();

// Настройка сессий
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

// Инициализация CSRF-защиты
const csrfProtection = csrf({ cookie: true });

// Промежуточный обработчик для генерации токена
app.use((req, res, next) => {
  res.locals.csrfToken = req.csrfToken();
  next();
});

// Маршрут для отображения формы
app.get('/form', csrfProtection, (req, res) => {
  res.send(`
    <form action="/process" method="POST">
      <input type="hidden" name="_csrf" value="${res.locals.csrfToken}">
      <input type="text" name="data" placeholder="Введите данные">
      <button type="submit">Отправить</button>
    </form>
  `);
});

// Маршрут для обработки формы
app.post('/process', csrfProtection, (req, res) => {
  res.send('Данные успешно обработаны!');
});

app.listen(3000);
```

### Python (Flask)

#### Установка и настройка

```bash
pip install Flask Flask-WTF
```

#### Пример реализации

```python
from flask import Flask, render_template, request, session
from flask_wtf.csrf import CSRFProtect
import secrets

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'

# Включение CSRF-защиты
csrf = CSRFProtect(app)

@app.route('/form', methods=['GET'])
def show_form():
    # Генерация CSRF-токена
    csrf_token = secrets.token_urlsafe(32)
    session['csrf_token'] = csrf_token
    return render_template('form.html', csrf_token=csrf_token)

@app.route('/process', methods=['POST'])
def process_form():
    # Проверка CSRF-токена
    token = request.form.get('csrf_token')
    if token != session.get('csrf_token'):
        return 'CSRF token mismatch', 403
    
    # Обработка формы
    data = request.form.get('data')
    return f'Данные успешно обработаны: {data}'

if __name__ == '__main__':
    app.run(debug=True)
```

Шаблон `form.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Форма с CSRF-защитой</title>
</head>
<body>
    <form method="POST" action="/process">
        <input type="hidden" name="csrf_token" value="{{ csrf_token }}">
        <input type="text" name="data" placeholder="Введите данные">
        <button type="submit">Отправить</button>
    </form>
</body>
</html>
```

### Python (Django)

Django имеет встроенную поддержку CSRF-защиты:

#### В шаблонах:

```html
<!-- form.html -->
<form method="post">
    {% csrf_token %}
    <input type="text" name="data" placeholder="Введите данные">
    <button type="submit">Отправить</button>
</form>
```

#### В представлениях:

```python
from django.shortcuts import render
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_protect

@csrf_protect
def my_view(request):
    if request.method == 'POST':
        data = request.POST.get('data')
        return HttpResponse(f'Данные успешно обработаны: {data}')
    return render(request, 'form.html')
```

#### Настройка в settings.py:

```python
# settings.py
MIDDLEWARE = [
    # ...
    'django.middleware.csrf.CsrfViewMiddleware',
    # ...
]

# Опциональные настройки
CSRF_COOKIE_SECURE = True  # Использовать только через HTTPS
CSRF_COOKIE_HTTPONLY = False  # Должен быть False, чтобы JavaScript мог получить токен
CSRF_USE_SESSIONS = True  # Хранить токены в сессиях
```

### PHP (с использованием сессий)

```php
<?php
session_start();

// Генерация CSRF-токена
function generate CSRFToken() {
    if (!isset($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }
    return $_SESSION['csrf_token'];
}

// Проверка CSRF-токена
function validateCSRFToken($token) {
    return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
}

// Страница с формой
if ($_SERVER['REQUEST_METHOD'] === 'GET') {
    $csrf_token = generateCSRFToken();
    echo '
    <form method="POST" action="process.php">
        <input type="hidden" name="csrf_token" value="' . $csrf_token . '">
        <input type="text" name="data" placeholder="Введите данные">
        <button type="submit">Отправить</button>
    </form>';
}

// Обработка формы
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!validateCSRFToken($_POST['csrf_token'])) {
        http_response_code(403);
        die('CSRF token mismatch');
    }
    
    $data = htmlspecialchars($_POST['data']);
    echo "Данные успешно обработаны: " . $data;
}
?>
```

### Java (Spring Boot)

#### Настройка

```java
// В классе конфигурации
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            );
        return http.build();
    }
}
```

#### Использование в контроллере

```java
@Controller
public class FormController {
    
    @GetMapping("/form")
    public String showForm(CsrfToken token, Model model) {
        // Токен автоматически добавляется в модель
        return "form";
    }
    
    @PostMapping("/process")
    @ResponseBody
    public String processForm(@RequestParam String data, @RequestBody CsrfToken token) {
        return "Данные успешно обработаны: " + data;
    }
}
```

#### Шаблон (Thymeleaf)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Форма с CSRF-защитой</title>
</head>
<body>
    <form th:action="@{/process}" method="post">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
        <input type="text" name="data" placeholder="Введите данные"/>
        <button type="submit">Отправить</button>
    </form>
</body>
</html>
```

### Ruby on Rails

Rails имеет встроенную поддержку CSRF-защиты:

#### В контроллере:

```ruby
class ApplicationController < ActionController::Base
  # Включение защиты от CSRF
  protect_from_forgery with: :exception
  
  # Или для API
  # protect_from_forgery with: :null_session
end
```

#### В представлениях:

```erb
<!-- form.html.erb -->
<%= form_with url: '/process', local: true do |form| %>
  <%= form.hidden_field :authenticity_token, value: form_authenticity_token %>
  <%= form.text_field :data, placeholder: "Введите данные" %>
  <%= form.submit "Отправить" %>
<% end %>

<!-- Или вручную -->
<%= form_tag('/process') do %>
  <%= hidden_field_tag :authenticity_token, form_authenticity_token %>
  <%= text_field_tag :data, nil, placeholder: "Введите данные" %>
  <%= submit_tag "Отправить" %>
<% end %>
```

### ASP.NET Core

#### Настройка

```csharp
// В Program.cs или Startup.cs
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";
    options.SuppressXFrameOptionsHeader = false;
});
```

#### В контроллере

```csharp
[ApiController]
public class FormController : ControllerBase
{
    [HttpGet("/form")]
    public IActionResult GetForm([FromServices] IAntiforgery antiforgery)
    {
        var token = antiforgery.GetAndStoreTokens(HttpContext);
        Response.Cookies.Append("XSRF-TOKEN", token.RequestToken);
        return Ok(new { token = token.RequestToken });
    }
    
    [HttpPost("/process")]
    [ValidateAntiForgeryToken]
    public IActionResult ProcessForm([FromBody] FormData data)
    {
        return Ok($"Данные успешно обработаны: {data.Value}");
    }
}

public class FormData
{
    public string Value { get; set; }
}
```

#### Использование на клиенте (JavaScript)

```javascript
// Получение токена
fetch('/form')
  .then(response => response.json())
  .then(data => {
    const token = data.token;
    
    // Отправка запроса с токеном
    fetch('/process', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'RequestVerificationToken': token
      },
      body: JSON.stringify({ value: 'some data' })
    });
  });
```

### Go (с использованием gorilla/csrf)

#### Установка

```bash
go get github.com/gorilla/csrf
```

#### Пример реализации

```go
package main

import (
    "fmt"
    "html/template"
    "net/http"
    
    "github.com/gorilla/csrf"
    "github.com/gorilla/mux"
)

const (
    key = "very-secret-key-32bytes-long"
)

func main() {
    r := mux.NewRouter()
    
    // Настройка CSRF-защиты
    csrfMiddleware := csrf.Protect([]byte(key), csrf.Secure(false))
    
    r.HandleFunc("/form", showForm).Methods("GET")
    r.HandleFunc("/process", processForm).Methods("POST")
    
    http.ListenAndServe(":8080", csrfMiddleware(r))
}

func showForm(w http.ResponseWriter, r *http.Request) {
    t, _ := template.New("form").Parse(`
        <html>
        <body>
            <form method="POST" action="/process">
                <input type="hidden" name="csrf_token" value="{{.}}">
                <input type="text" name="data" placeholder="Введите данные">
                <button type="submit">Отправить</button>
            </form>
        </body>
        </html>
    `)
    t.Execute(w, csrf.Token(r))
}

func processForm(w http.ResponseWriter, r *http.Request) {
    data := r.FormValue("data")
    fmt.Fprintf(w, "Данные успешно обработаны: %s", data)
}
```

Эти примеры показывают, как реализовать CSRF-защиту в различных технологических стеках. Важно следовать лучшим практикам безопасности при реализации защиты в вашем конкретном приложении.

## Практические рекомендации по использованию

Для эффективной и безопасной реализации защиты от CSRF с использованием токенов необходимо следовать ряду практических рекомендаций и лучших практик.

### Общие принципы

#### 1. Применяйте защиту ко всем чувствительным операциям

CSRF-защита должна применяться ко всем операциям, которые изменяют состояние системы или выполняют важные действия:

- Финансовые транзакции
- Изменение пользовательских данных
- Изменение настроек учетной записи
- Удаление данных
- Административные действия

#### 2. Используйте HTTPS повсеместно

Всегда используйте HTTPS для передачи CSRF-токенов, чтобы предотвратить их перехват через атаки типа "человек посередине" (MITM).

#### 3. Следуйте принципу наименьших привилегий

Ограничьте права, которые могут быть использованы через CSRF-уязвимости, чтобы минимизировать потенциальный ущерб.

### Генерация токенов

#### 4. Используйте криптографически безопасные генераторы

Всегда используйте криптографически безопасные генераторы случайных чисел:

```javascript
// Правильно - криптографически безопасный генератор
const token = crypto.randomBytes(32).toString('hex');

// Неправильно - предсказуемые числа
const token = Math.random().toString(36);
```

#### 5. Обеспечьте достаточную длину токенов

Рекомендуется использовать токены длиной не менее 128 бит (16 байт), что эквивалентно 32 символам в шестнадцатеричном формате или 22 символам в base64.

#### 6. Не используйте предсказуемые данные

Не генерируйте токены на основе предсказуемых данных, таких как:
- Временные метки
- IP-адреса
- ID пользователей
- Последовательные числа

### Хранение и передача токенов

#### 7. Храните токены на сервере

Храните CSRF-токены на сервере, а не на клиенте, чтобы избежать рисков, связанных с XSS-атаками:

> [!warning] Важно
> Если токены хранятся в куки или localStorage, они могут быть украдены через XSS-атаки.

#### 8. Используйте сессии для хранения токенов

Наиболее безопасный способ хранения CSRF-токенов - использование сессий:

```javascript
// Пример правильного хранения в сессии
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true
}));

function generateToken(req) {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  return req.session.csrfToken;
}
```

#### 9. Не передавайте токены в URL

Избегайте передачи CSRF-токенов в URL-параметрах, так как они могут:
- Сохраняться в логах сервера
- Отображаться в строке браузера
- Попадать в заголовки Referer
- Кэшироваться прокси-серверами

#### 10. Правильно обрабатывайте AJAX-запросы

Для AJAX-запросов используйте HTTP-заголовки для передачи токенов:

```javascript
// Получение токена из мета-тега
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

// Включение токена в AJAX-запрос
fetch('/api/action', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken
  },
  body: JSON.stringify(data)
});
```

### Проверка токенов

#### 11. Всегда проверяйте токены на сервере

Никогда не полагайтесь только на клиентскую проверку - все проверки должны происходить на сервере.

#### 12. Используйте безопасное сравнение строк

Для сравнения токенов используйте безопасные методы, устойчивые к атакам по времени:

```javascript
// Вместо обычного сравнения
if (receivedToken === storedToken) { ... }

// Используйте безопасное сравнение (в Node.js)
const crypto = require('crypto');
if (crypto.timingSafeEqual(Buffer.from(receivedToken), Buffer.from(storedToken))) { ... }

// Или в других языках используйте аналогичные функции
```

#### 13. Обрабатывайте ошибки корректно

При несовпадении токенов возвращайте соответствующий HTTP-статус (обычно 403 Forbidden) и не раскрывайте детали ошибки:

```javascript
app.post('/sensitive-action', (req, res) => {
  if (!validateToken(req.body.csrf_token)) {
    // Не раскрывать детали ошибки
    return res.status(403).send('Доступ запрещен');
  }
  // Обработка запроса
});
```

### Архитектурные рекомендации

#### 14. Используйте одноразовые токены для максимальной безопасности

Одноразовые токены обеспечивают максимальную защиту, хотя и требуют больше ресурсов:

```javascript
// Пример одноразового токена
app.post('/action', (req, res) => {
  if (!validateToken(req.body.csrf_token)) {
    return res.status(403).send('CSRF token mismatch');
  }
  // Удаляем использованный токен и генерируем новый
  req.session.csrfToken = generateNewToken();
  // Обработка запроса
});
```

#### 15. Рассмотрите использование SameSite-атрибута для куки

Хотя CSRF-токены обеспечивают надежную защиту, дополнительным слоем безопасности может быть атрибут SameSite для сессионных куки:

```javascript
// Установка куки с SameSite-атрибутом
app.use(session({
  cookie: {
    sameSite: 'lax', // или 'strict'
    secure: true
  }
}));
```

#### 16. Обновляйте токены при изменении важных параметров

Обновляйте CSRF-токены при:
- Аутентификации пользователя
- Изменении уровня привилегий
- Длительном периоде неактивности

### Тестирование и мониторинг

#### 17. Тестируйте защиту от CSRF

Регулярно тестируйте CSRF-защиту с помощью:
- Автоматизированных инструментов безопасности
- Ручного тестирования
- Пентестов

#### 18. Мониторьте попытки CSRF-атак

Реализуйте логирование и мониторинг подозрительных запросов, чтобы выявлять потенциальные атаки:

```javascript
// Пример логирования подозрительных запросов
app.post('/sensitive-action', (req, res) => {
  if (!validateToken(req.body.csrf_token)) {
    // Логирование подозрительного запроса
    console.warn('Подозрительный запрос CSRF', {
      ip: req.ip,
      userAgent: req.get('User-Agent'),
      timestamp: new Date()
    });
    return res.status(403).send('CSRF token mismatch');
  }
});
```

### Распространенные ошибки

#### 19. Избегайте следующих ошибок:

- **Хранение токенов в localStorage/sessionStorage** - уязвимо для XSS
- **Использование одинаковых токенов для всех пользователей** - не обеспечивает защиту
- **Отключение защиты для API-запросов** - API также уязвим к CSRF
- **Неправильная проверка токенов** - например, с использованием небезопасного сравнения
- **Передача токенов в URL** - может привести к утечке токенов

#### 20. Проверяйте совместимость с различными клиентами

Убедитесь, что ваша реализация CSRF-защиты совместима с:
- Мобильными приложениями
- Различными браузерами
- Сторонними интеграциями
- Автоматизированными инструментами

### Рекомендации для разных типов приложений

#### Для одностраничных приложений (SPA):

- Используйте заголовки для передачи токенов
- Реализуйте механизм обновления токенов
- Обрабатывайте ошибки CSRF в пользовательском интерфейсе

#### Для API:

- Используйте заголовки для передачи токенов
- Рассмотрите комбинацию CSRF-токенов с другими методами аутентификации
- Обеспечьте совместимость с различными клиентами API

#### Для мультидоменных приложений:

- Учитывайте особенности обмена токенами между доменами
- Рассмотрите использование CORS с соответствующими заголовками
- Используйте централизованное управление токенами при необходимости

## Ограничения и подводные камни

Хотя CSRF-токены являются эффективным методом защиты от атак типа Cross-Site Request Forgery, у них есть определенные ограничения и потенциальные подводные камни, которые разработчики должны учитывать при реализации.

### Ограничения CSRF-токенов

#### 1. Уязвимость к XSS-атакам

Самое главное ограничение CSRF-токенов — их уязвимость к XSS (Cross-Site Scripting) атакам:

- Если приложение уязвимо к XSS, злоумышленник может украсть CSRF-токены
- В JavaScript-коде токены могут быть доступны через DOM
- Это делает CSRF-защиту бесполезной

> [!danger] Критическая уязвимость
> XSS-атаки могут полностью обойти CSRF-защиту, поэтому важно защищать приложение от XSS как приоритетную задачу.

#### 2. Ограничения для API и мобильных приложений

CSRF-токены не всегда подходят для всех типов приложений:

- API-интерфейсы, используемые мобильными приложениями, не могут легко использовать CSRF-токены
- Требуется альтернативная аутентификация (токены, OAuth и т.д.)
- Сложности с междоменными запросами (CORS)

#### 3. Проблемы с масштабируемостью

- Необходимость хранения токенов в сессиях может создавать нагрузку на сервер
- Для распределенных приложений требуется централизованное хранилище сессий
- Возможны проблемы с синхронизацией токенов между инстансами

### Подводные камни при реализации

#### 4. Неправильное хранение токенов

**Хранение в localStorage/sessionStorage:**
```javascript
// НЕПРАВИЛЬНО - уязвимо для XSS
localStorage.setItem('csrfToken', token);
sessionStorage.setItem('csrfToken', token);
```

**Правильное хранение:**
- В сессиях на сервере
- В HTTP-куки с флагами HttpOnly и Secure

#### 5. Передача токенов в URL

Передача CSRF-токенов через URL-параметры создает уязвимости:

- Токены могут сохраняться в логах сервера
- Попадают в заголовки Referer
- Кэшируются браузерами и прокси
- Отображаются в адресной строке

#### 6. Неправильная генерация токенов

Использование небезопасных генераторов случайных чисел:

```javascript
// НЕПРАВИЛЬНО - предсказуемый токен
const weakToken = Math.random().toString(36);

// ПРАВИЛЬНО - криптографически безопасный токен
const secureToken = crypto.randomBytes(32).toString('hex');
```

#### 7. Отсутствие проверки на всех конечных точках

Разработчики могут забыть добавить проверку CSRF-токенов ко всем чувствительным маршрутам:

```javascript
// Пример: проверка CSRF отсутствует
app.post('/delete-account', (req, res) => {
  // Опасно! Нет проверки CSRF-токена
  deleteUser(req.session.userId);
  res.send('Account deleted');
});
```

#### 8. Проблемы с кэшированием страниц

Если страницы с CSRF-токенами кэшируются:
- Токены могут стать недействительными
- Пользователи могут получить устаревшие токены
- Возникают ошибки при отправке форм

#### 9. Проблемы с параллельными запросами

При использовании одноразовых токенов:
- Параллельные запросы могут привести к ошибкам
- Один запрос может использовать токен, а другой получит ошибку
- Требуется специальная обработка таких ситуаций

#### 10. Несовместимость с определенными архитектурами

- Некоторые архитектурные решения могут не поддерживать CSRF-токены
- Микросервисы могут требовать специального подхода
- Состоятельные и бессостоятельные архитектуры требуют разных решений

### Специфические подводные камни

#### 11. Проблемы с межсайтовыми запросами

- CSRF-токены не защищают от атак, происходящих с доверенного домена
- Поддомены могут иметь доступ к токенам
- Требуется дополнительная изоляция для поддоменов

#### 12. Проблемы с производительностью

- Генерация и проверка токенов добавляют накладные расходы
- Для высоконагруженных приложений это может быть значительным
- Требуется оптимизация процессов генерации и проверки

#### 13. Проблемы с тестированием

- CSRF-защита усложняет автоматизированное тестирование
- Необходимо учитывать токены в тестах
- Требуется специальная настройка тестовых окружений

#### 14. Проблемы с совместимостью

- Некоторые старые браузеры могут не поддерживать определенные методы
- Мобильные приложения могут требовать альтернативные подходы
- Интеграция с сторонними системами может быть сложной

### Сценарии, где CSRF-токены неэффективны

#### 15. Атаки через GET-запросы

CSRF-токены обычно не защищают от атак, использующих GET-запросы:
- GET-запросы не могут легко содержать токены
- Злоумышленники могут создать изображение или iframe с GET-запросом
- Для таких случаев необходимы другие меры защиты

#### 16. Атаки на уровне сети

- CSRF-токены не защищают от атак на уровне сети (например, DNS-отравления)
- Злоумышленник может подделать IP-адреса или использовать прокси
- Требуется дополнительная сетевая безопасность

### Меры по минимизации рисков

#### 17. Комбинирование с другими методами защиты

- Используйте CSRF-токены в сочетании с другими методами:
  - Проверка заголовков Origin/Referer
  - SameSite-атрибут куки
  - Content Security Policy
  - Проверка IP-адресов

#### 18. Регулярная проверка реализации

- Проводите регулярные аудиты безопасности
- Тестируйте защиту на уязвимости
- Обновляйте реализацию в соответствии с новыми угрозами

#### 19. Обработка ошибок CSRF

- Не раскрывайте детали ошибок CSRF-проверки
- Логируйте подозрительные запросы
- Обеспечьте корректное поведение при ошибках

#### 20. Обучение разработчиков

- Обучайте команду разработчиков безопасности
- Разрабатывайте стандарты кодирования с учетом безопасности
- Регулярно обновляйте знания о новых угрозах и методах защиты

### Заключение

CSRF-токены остаются важным инструментом защиты веб-приложений, но их эффективность зависит от правильной реализации и учета всех вышеупомянутых ограничений и подводных камней. Важно понимать, что CSRF-защита — это только один слой в многоуровневой системе безопасности.

## Альтернативы токенам

Хотя анти-CSRF токены являются одним из самых распространенных методов защиты от CSRF-атак, существуют и другие подходы, которые могут быть использованы как самостоятельно, так и в сочетании с CSRF-токенами для создания многоуровневой защиты.

### 1. SameSite-атрибут куки

SameSite — это атрибут HTTP-куки, который позволяет контролировать, когда куки отправляются с запросами на сайт. Это наиболее современный и эффективный способ защиты от CSRF-атак.

#### Значения атрибута SameSite:

- **Strict**: куки отправляются только при навигации на сайт изнутри (без переходов с других сайтов)
- **Lax**: куки отправляются при навигации с других сайтов, но только для безопасных HTTP-методов (GET, HEAD, OPTIONS, TRACE)
- **None**: куки отправляются всегда (требует флага Secure)

#### Примеры:

```javascript
// Установка куки с SameSite-атрибутом
res.cookie('session', sessionId, {
  secure: true,
  sameSite: 'lax', // или 'strict'
  httpOnly: true
});
```

```http
Set-Cookie: session=abc123; Secure; SameSite=Lax
```

#### Преимущества:
- Простота реализации
- Защита на уровне браузера
- Не требует изменения логики приложения
- Современный стандарт, поддерживаемый всеми актуальными браузерами

#### Недостатки:
- Ограниченная поддержка в старых браузерах
- Не защищает от XSS-атак
- Может повлиять на пользовательский опыт в некоторых сценариях

### 2. Проверка заголовков Origin и Referer

#### Заголовок Origin:
- Отправляется браузером с каждым CORS-запросом
- Содержит источник запроса (протокол, домен, порт)
- Не может быть изменен JavaScript

```javascript
// Пример проверки Origin
app.use((req, res, next) => {
  const origin = req.get('Origin');
  const expectedOrigin = 'https://yourdomain.com';
  
  if (origin && origin !== expectedOrigin) {
    return res.status(403).send('Origin mismatch');
  }
  next();
});
```

#### Заголовок Referer:
- Содержит URL страницы, с которой был сделан запрос
- Может быть подделан или отсутствовать в некоторых случаях

> [!warning] Ограничения
> Заголовки Origin/Referer могут быть отсутствовать или подделаны, особенно в старых браузерах или при определенных настройках конфиденциальности.

### 3. Double Submit Cookie Pattern

Этот подход заключается в отправке токена как в куки, так и в теле/заголовке запроса:

#### Принцип работы:
1. Сервер устанавливает куки с CSRF-токеном
2. Клиент отправляет тот же токен в теле запроса или в заголовке
3. Сервер сравнивает значения

```javascript
// Установка куки с токеном
app.use((req, res, next) => {
  if (!req.cookies.csrfToken) {
    const token = crypto.randomBytes(32).toString('hex');
    res.cookie('csrfToken', token, { httpOnly: false }); // Важно: httpOnly: false
  }
  next();
});

// Проверка токена
app.post('/api/action', (req, res, next) => {
  const cookieToken = req.cookies.csrfToken;
  const headerToken = req.get('X-CSRF-Token') || req.body.csrfToken;
  
  if (cookieToken !== headerToken) {
    return res.status(403).send('CSRF token mismatch');
  }
  next();
});
```

#### Преимущества:
- Не требует серверного хранения токенов
- Относительно простая реализация

#### Недостатки:
- Уязвим к XSS-атакам
- Требует установку куки доступной для JavaScript (httpOnly: false)

### 4. Synchronizer Token Pattern (другая реализация)

В отличие от стандартных CSRF-токенов, синхронизирующие токены могут быть привязаны к определенным действиям или формам:

```javascript
// Генерация токена для конкретного действия
function generateActionToken(action, userId) {
  return crypto.createHmac('sha256', secretKey)
    .update(`${action}:${userId}:${Date.now()}`)
    .digest('hex');
}
```

### 5. Проверка IP-адреса и User-Agent

Простая проверка соответствия IP-адреса и User-Agent строки сессии:

```javascript
// Пример проверки
app.use((req, res, next) => {
  if (req.session.userIp && req.session.userIp !== req.ip) {
    // Возможная CSRF-атака
    return res.status(403).send('IP address changed');
  }
  
  if (req.session.userAgent && req.session.userAgent !== req.get('User-Agent')) {
    // Возможная CSRF-атака
    return res.status(403).send('User-Agent changed');
  }
  
  next();
});
```

#### Недостатки:
- Неэффективна при использовании прокси или NAT
- Пользователи могут часто менять IP-адреса
- User-Agent может изменяться легитимно

### 6. Custom Request Headers

Требование наличия специфических заголовков, которые не могут быть установлены обычными HTML-формами:

```javascript
// Требование специфического заголовка
app.use('/api/*', (req, res, next) => {
  const requiredHeader = req.get('X-Requested-With');
  if (requiredHeader !== 'XMLHttpRequest') {
    return res.status(403).send('Invalid request header');
  }
  next();
});
```

#### Ограничения:
- Работает только для AJAX-запросов
- Не защищает обычные HTML-формы
- Современные браузеры могут ограничивать возможность установки кастомных заголовков

### 7. CAPTCHA для чувствительных операций

Использование CAPTCHA для критических действий:

```javascript
// Пример: проверка CAPTCHA для финансовых операций
app.post('/transfer', (req, res) => {
  if (req.body.amount > 1000) { // Если сумма больше порога
    const captchaValid = validateCaptcha(req.body.captcha);
    if (!captchaValid) {
      return res.status(400).send('Invalid CAPTCHA');
    }
  }
  // Обработка перевода
});
```

#### Преимущества:
- Высокий уровень защиты
- Не зависит от сессий или токенов

#### Недостатки:
- Ухудшает пользовательский опыт
- Требует дополнительной инфраструктуры

### 8. One-Time Tokens (OOT) для критических операций

Одноразовые токены для особенно важных действий:

```javascript
// Генерация одноразового токена для критической операции
app.post('/generate-oot', (req, res) => {
  const oneTimeToken = crypto.randomBytes(32).toString('hex');
  // Сохраняем токен в базе данных с меткой времени
  saveOneTimeToken(oneTimeToken, req.session.userId);
  res.json({ token: oneTimeToken });
});

// Использование одноразового токена
app.post('/critical-action', (req, res) => {
  const isValid = validateAndConsumeOneTimeToken(req.body.oot, req.session.userId);
  if (!isValid) {
    return res.status(403).send('Invalid or expired token');
  }
  // Выполнение критического действия
});
```

### 9. Content Security Policy (CSP)

Хотя CSP не является прямой защитой от CSRF, он может ограничить возможности атакующего:

```http
Content-Security-Policy: default-src 'self'; form-action 'self';
```

### 10. Сочетание нескольких методов

Наиболее эффективная защита достигается при использовании нескольких методов одновременно:

```javascript
// Комбинированный подход
app.use(session({
  cookie: { sameSite: 'lax', secure: true }
}));

// Проверка Origin для API-запросов
app.use('/api/', (req, res, next) => {
  if (req.get('Origin') && req.get('Origin') !== process.env.APP_ORIGIN) {
    return res.status(403).send('Invalid origin');
  }
  next();
});

// CSRF-токены для форм
app.use(csrfProtection);

// Дополнительная проверка для чувствительных операций
app.post('/sensitive-action', requireCaptcha, validateCsrfToken, (req, res) => {
  // Обработка запроса
});
```

### Выбор подходящего метода

При выборе метода защиты от CSRF следует учитывать:

1. **Тип приложения** (веб-сайт, API, мобильное приложение)
2. **Требования к совместимости** (старые браузеры, специфические клиенты)
3. **Уровень безопасности** требований
4. **Влияние на пользовательский опыт**
5. **Наличие других уязвимостей** (XSS, CORS и т.д.)

### Заключение

Хотя CSRF-токены остаются одним из наиболее надежных методов защиты, современные подходы, такие как SameSite-атрибут куки, часто более предпочтительны из-за своей простоты и эффективности. В идеале, защита от CSRF должна быть многоуровневой, сочетая несколько методов для максимальной безопасности.

## Ссылки на другие связанные файлы

- [[CSRF-защита]] - основной файл о защите от CSRF-атак
- [[XSS-защита]] - защита от межсайтового скриптинга, тесно связанная с CSRF-защитой
- [[HTTP-Security-Headers]] - безопасные HTTP-заголовки, включая SameSite и другие
- [[Cookies-безопасность]] - безопасное использование куки, важное для CSRF-защиты
- [[Сессии-и-аутентификация]] - управление сессиями, где хранятся CSRF-токены
- [[Content-Security-Policy]] - политика безопасности контента как дополнительный слой защиты
- [[Clickjacking-защита]] - защита от атак типа clickjacking, схожие угрозы с CSRF
- [[Управление-сессиями-и-аутентификацией]] - детали управления сессиями
- [[Безопасность-в-SPA]] - особенности безопасности для одностраничных приложений
- [[Безопасность-в-браузере]] - общие вопросы безопасности в браузере
- [[Безопасность-в-микросервисной-архитектуре]] - особенности CSRF-защиты в микросервисах
- [[Безопасность-в-облачных-средах]] - вопросы безопасности в облачных средах
- [[Безопасность-форм]] - безопасность обработки форм, где применяются CSRF-токены
- [[Безопасность-платежных-форм]] - специфические вопросы безопасности для платежных форм
- [[Защита-от-инъекций]] - общие вопросы веб-безопасности
- [[Основы-веб-безопасности]] - базовые принципы веб-безопасности
- [[Ограничение-доступа-к-API]] - вопросы безопасности API
- [[Тестирование-безопасности]] - методы тестирования безопасности
- [[Мониторинг-безопасности]] - мониторинг безопасности приложений
- [[Secure-Coding-Practices]] - безопасное программирование
- [[security]] - основной файл по безопасности
