---
aliases: [CORS и безопасность запросов, CORS в HTML, Безопасность сетевых запросов]
tags: [html, networking, javascript, web-development, security, cors]
---

# CORS и безопасность запросов

## Введение в CORS

CORS (Cross-Origin Resource Sharing) - это механизм безопасности, реализованный в браузерах, который контролирует, как веб-страницы из одного источника могут запрашивать ресурсы из другого источника. Это важная часть безопасности веб-приложений, предотвращающая потенциальные атаки, такие как CSRF (Cross-Site Request Forgery).

### Что такое "origin" (источник)

Источник определяется как комбинация:
- Протокола (http, https)
- Домена (example.com)
- Порта (80, 443, 3000 и т.д.)

Примеры:
- `https://example.com:443` и `http://example.com:80` - разные источники (разные протоколы)
- `https://example.com` и `https://api.example.com` - разные источники (разные домены)
- `https://example.com:3000` и `https://example.com:8080` - разные источники (разные порты)

## Как работает CORS

### Простые запросы (Simple Requests)

Некоторые запросы считаются "простыми" и не требуют предварительной проверки CORS:

1. Методы: GET, POST, HEAD
2. Разрешенные заголовки: Accept, Accept-Language, Content-Language, Last-Event-ID, Content-Type (только с значениями application/x-www-form-urlencoded, multipart/form-data, text/plain)

```javascript
// Простой запрос - не требует preflight
fetch('https://api.example.com/data', {
  method: 'GET'
});

// Также простой запрос
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: 'key=value'
});
```

### Предварительные запросы (Preflight Requests)

Более сложные запросы требуют предварительной проверки с использованием метода OPTIONS:

```javascript
// Запрос с кастомным заголовком - требует preflight
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value'
  },
  body: JSON.stringify({data: 'example'})
});
```

Браузер автоматически отправит предварительный запрос:

```
OPTIONS /data HTTP/1.1
Host: api.example.com
Origin: https://mywebsite.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-Custom-Header, Content-Type
```

Сервер должен ответить с разрешающими заголовками:

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://mywebsite.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: X-Custom-Header, Content-Type
Access-Control-Max-Age: 86400
```

## Заголовки CORS

### Заголовки запроса

- `Origin` - указывает источник запроса
- `Access-Control-Request-Method` - метод, который будет использован в реальном запросе
- `Access-Control-Request-Headers` - заголовки, которые будут использованы в реальном запросе

### Заголовки ответа

- `Access-Control-Allow-Origin` - разрешенные источники (может быть "*" для всех или конкретный домен)
- `Access-Control-Allow-Methods` - разрешенные методы HTTP
- `Access-Control-Allow-Headers` - разрешенные заголовки
- `Access-Control-Allow-Credentials` - разрешение на отправку учетных данных
- `Access-Control-Max-Age` - время кэширования preflight-запроса
- `Access-Control-Expose-Headers` - заголовки, доступные клиентскому коду

## Настройка CORS на сервере

### Пример на Node.js (Express)

```javascript
// Простая настройка CORS
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
  
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
});

// Более безопасная настройка с конкретными доменами
const allowedOrigins = [
  'https://mywebsite.com',
  'https://admin.mywebsite.com',
  'http://localhost:3000'  // для разработки
];

app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
});
```

### Пример с использованием библиотеки cors в Express

```javascript
const cors = require('cors');

// Настройка CORS для конкретных доменов
const corsOptions = {
  origin: ['https://mywebsite.com', 'https://admin.mywebsite.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
};

app.use(cors(corsOptions));
```

## Практические примеры CORS-запросов

### Запрос с учетными данными

```javascript
// Запрос с отправкой cookies и аутентификационных данных
fetch('https://api.example.com/protected-data', {
  method: 'GET',
  credentials: 'include'  // важный параметр для отправки cookies
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Ошибка CORS:', error));
```

Для запросов с учетными данными сервер должен вернуть:

```
Access-Control-Allow-Origin: https://mywebsite.com
Access-Control-Allow-Credentials: true
```

> [!warning] Важно
> При использовании `credentials: 'include'`, заголовок `Access-Control-Allow-Origin` не может быть "*", должен быть указан конкретный домен.

### Обработка CORS-ошибок

```javascript
async function handleCORSRequest() {
  try {
    const response = await fetch('https://api.example.com/data');
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    if (error.name === 'TypeError' && error.message.includes('CORS')) {
      console.error('Ошибка CORS:', error);
      showCORSErrorMessage();
    } else {
      console.error('Другая ошибка:', error);
    }
    throw error;
  }
}

function showCORSErrorMessage() {
  alert('Ошибка доступа к удаленному ресурсу. Обратитесь к администратору.');
}
```

## Безопасность CORS-запросов

### Распространенные ошибки безопасности

1. **Разрешение всех источников**: `Access-Control-Allow-Origin: *`
   - Подходит только для публичных API без чувствительных данных
   - Никогда не используйте для запросов с учетными данными

2. **Неправильная обработка динамических источников**:
   ```javascript
   // НЕБЕЗОПАСНО
   res.header('Access-Control-Allow-Origin', req.headers.origin);
   ```

3. **Слишком широкие разрешения методов и заголовков**

### Безопасная реализация CORS

```javascript
// Безопасная проверка источника
function isValidOrigin(origin) {
  const allowedOrigins = [
    'https://mywebsite.com',
    'https://app.mywebsite.com',
    'https://admin.mywebsite.com',
    'http://localhost:3000',  // разработка
    'http://localhost:3001'   // тестирование
  ];
  
  return allowedOrigins.includes(origin);
}

app.use((req, res, next) => {
  const origin = req.headers.origin;
  
  if (origin && isValidOrigin(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  
  // Ограничение методов и заголовков
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
});
```

## Современные альтернативы CORS

### JSONP (JSON with Padding)

> [!warning] Устаревший метод
> JSONP использовался до появления CORS, но имеет серьезные ограничения безопасности и не поддерживает HTTP-методы кроме GET.

### Прокси-сервер

```javascript
// Вместо прямого запроса к внешнему API
// fetch('https://external-api.com/data') // может вызвать CORS

// Используем прокси на нашем сервере
fetch('/proxy/external-api/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

### Серверный прокси (на бэкенде)

```javascript
// Пример на Node.js
app.get('/proxy/external-data', async (req, res) => {
  try {
    const externalResponse = await fetch('https://external-api.com/data', {
      headers: {
        'Authorization': `Bearer ${process.env.EXTERNAL_API_KEY}`
      }
    });
    
    const data = await externalResponse.json();
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: 'Ошибка получения данных' });
  }
});
```

## Практические рекомендации для российских разработчиков (2025)

### Работа с российскими API и сервисами

При интеграции с российскими API учитывайте:

- Часто российские API имеют строгие ограничения CORS
- Требуют специфичные заголовки аутентификации
- Могут использовать внутренние прокси и шлюзы

```javascript
// Пример работы с Яндекс API
async function getYandexData() {
  try {
    const response = await fetch('https://api.yandex.ru/some-endpoint', {
      method: 'GET',
      headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.message.includes('CORS')) {
      // Для Яндекс API может потребоваться использование их SDK
      // или серверный прокси
      console.error('CORS ошибка при запросе к Яндекс API');
    }
    throw error;
  }
}
```

### Особенности российской инфраструктуры

- Часто используются внутренние прокси и корпоративные брандмауэры
- Возможны ограничения на доступ к иностранным API
- Важно учитывать требования законодательства (ФЗ-152 о персональных данных)

### Безопасность в российском контексте

```javascript
// Пример безопасной настройки CORS для российского проекта
const russianSafeCORS = (req, res, next) => {
  const origin = req.headers.origin;
  const userAgent = req.headers['user-agent'];
  
  // Проверка разрешенных источников
  const allowedOrigins = [
    'https://my-website.ru',
    'https://www.my-website.ru',
    'https://admin.my-website.ru',
    'https://lk.my-website.ru',
    'http://localhost:3000'  // для разработки
  ];
  
  if (origin && allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  
  // Ограничение методов
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  
  // Только необходимые заголовки
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
  
  // Без учетных данных по умолчанию
  // res.header('Access-Control-Allow-Credentials', 'true'); // включать только при необходимости
  
  // Защита от CSRF
  res.header('X-Content-Type-Options', 'nosniff');
  res.header('X-Frame-Options', 'DENY');
  res.header('X-XSS-Protection', '1; mode=block');
  
  if (req.method === 'OPTIONS') {
    // Кэширование preflight-запросов на 10 минут
    res.header('Access-Control-Max-Age', '600');
    res.sendStatus(200);
  } else {
    next();
  }
};
```

### Отладка CORS в российских условиях

При работе с российскими сервисами могут возникать специфичные проблемы:

1. **Блокировка запросов антивирусами или брандмауэрами**
2. **Проблемы с сертификатами SSL**
3. **Ограничения провайдеров интернет-услуг**

Для диагностики используйте:

```javascript
// Дополнительная диагностика CORS
async function diagnosticCORSRequest(url) {
  try {
    console.log('Попытка запроса к:', url);
    console.log('Origin:', window.location.origin);
    
    const response = await fetch(url, {
      method: 'OPTIONS', // preflight-запрос
      mode: 'cors'
    });
    
    console.log('CORS заголовки ответа:', {
      'Access-Control-Allow-Origin': response.headers.get('Access-Control-Allow-Origin'),
      'Access-Control-Allow-Methods': response.headers.get('Access-Control-Allow-Methods'),
      'Access-Control-Allow-Headers': response.headers.get('Access-Control-Allow-Headers')
    });
    
    return response;
  } catch (error) {
    console.error('CORS диагностика ошибка:', error);
    return null;
  }
}
```

## Современные возможности и лучшие практики

### CORS с использованием Fetch API

```javascript
// Явное указание режима CORS
const response = await fetch('https://api.example.com/data', {
  method: 'POST',
  mode: 'cors',  // явно указываем режим CORS
  credentials: 'same-origin',  // или 'include' для отправки cookies
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(data)
});
```

### Использование Subresource Integrity (SRI)

Для внешних скриптов и стилей:

```html
<script 
  src="https://cdn.example.com/library.js" 
  integrity="sha384-..." 
  crossorigin="anonymous">
</script>
```

### Content Security Policy (CSP)

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               connect-src 'self' https://api.example.com; 
               img-src 'self' https://images.example.com;">
```

## Отладка CORS-ошибок

### Использование DevTools

1. Откройте Network вкладку в DevTools
2. Найдите failed запрос
3. Проверьте заголовки запроса и ответа
4. Обратите внимание на ошибку в Console

### Частые ошибки и решения

| Ошибка | Причина | Решение |
|--------|---------|---------|
| "CORS header 'Access-Control-Allow-Origin' missing" | Сервер не возвращает CORS-заголовки | Настройте сервер для возврата заголовков |
| "The value of the 'Access-Control-Allow-Origin' header is invalid" | "*" используется с credentials | Укажите конкретный домен вместо "*" |
| "Request header field X is not allowed by Access-Control-Allow-Headers" | Заголовок не разрешен сервером | Добавьте заголовок в Access-Control-Allow-Headers |
| "CORS preflight channel did not succeed" | Preflight запрос отклонен | Проверьте настройки сервера для OPTIONS запросов |

## Полезные ресурсы

- [[Fetch-API-в-HTML]] - для понимания работы CORS с современным API
- [[XMLHttpRequest-в-HTML]] - для работы с CORS в унаследованном коде
- [[Обработка-ответов-и-ошибок]] - для правильной обработки CORS-ошибок
- [[FormData-и-загрузка-файлов]] - особенности CORS при загрузке файлов

## Заключение

CORS - важный механизм безопасности, который защищает пользователей от межсайтовых запросов. Правильная настройка CORS требует баланса между безопасностью и функциональностью. В российских условиях важно учитывать особенности инфраструктуры и требования законодательства при реализации CORS-политики.

> [!tip] Совет
> Всегда тестируйте CORS-политику в различных сценариях и используйте минимально необходимые разрешения для обеспечения безопасности.

> [!info] Примечание
> CORS - это браузерный механизм. Серверные приложения не подчиняются этим ограничениям, что делает серверный прокси эффективным решением для обхода CORS-ограничений.