---
aliases: ["CORS Security", "Cross-Origin Resource Sharing", "Безопасность CORS"]
tags: [security, web, cors, browser]
---

# Безопасность CORS (Cross-Origin Resource Sharing)

## Обзор

Cross-Origin Resource Sharing (CORS) - это механизм безопасности, реализованный в веб-браузерах, который контролирует, как веб-страницы из одного источника могут взаимодействовать с ресурсами другого источника. CORS необходим для предотвращения нежелательного доступа к приватным ресурсам, но при неправильной настройке может создать серьезные уязвимости.

## Что такое "Origin" (источник)

В контексте CORS, "origin" определяется как комбинация:

- Протокола (http, https)
- Домена (example.com)
- Порта (80, 443, 3000 и т.д.)

Примеры:
- `https://example.com:443` и `https://example.com:80` - разные источники
- `https://example.com` и `https://sub.example.com` - разные источники
- `https://example.com` и `http://example.com` - разные источники

## Как работает CORS

### Простые запросы

Некоторые запросы считаются "простыми" и не требуют предварительной проверки:

1. Методы: GET, POST, HEAD
2. Разрешенные заголовки: Accept, Accept-Language, Content-Language, Content-Type (с ограниченными значениями)
3. Content-Type: application/x-www-form-urlencoded, multipart/form-data, text/plain

### Предварительные запросы (Preflight)

Для более сложных запросов браузер сначала отправляет "preflight" запрос с методом OPTIONS, чтобы проверить, разрешено ли взаимодействие:

```http
OPTIONS /api/data HTTP/1.1
Host: api.example.com
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-Custom-Header
```

Сервер отвечает:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 86400
```

## Уязвимости, связанные с CORS

### Слишком широкие разрешения

```http
# НЕБЕЗОПАСНО - разрешает любой источник
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Это создает уязвимость, особенно при включенных credentials, так как позволяет любому сайту получить доступ к приватным данным пользователя.

### Уязвимость рефлекторного заголовка Origin

Некоторые серверы неправильно отражают заголовок Origin:

```javascript
// НЕБЕЗОПАСНЫЙ код
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', req.headers.origin);
  res.header('Access-Control-Allow-Credentials', true);
  next();
});
```

Злоумышленник может создать сайт, который отравляет заголовок Origin, и получить доступ к приватным ресурсам.

## Лучшие практики безопасности CORS

### 1. Явное указание разрешенных источников

```javascript
const allowedOrigins = [
  'https://example.com',
  'https://app.example.com',
  'https://staging.example.com'
];

app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
    res.header('Access-Control-Allow-Credentials', true);
  }
  next();
});
```

### 2. Избегайте использования wildcard с credentials

```http
# НЕБЕЗОПАСНО
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

# ПРАВИЛЬНО
Access-Control-Allow-Origin: https://trusted-site.com
Access-Control-Allow-Credentials: true
```

### 3. Проверка источника на сервере

Вместо доверия заголовку Origin, проверяйте источники на сервере:

```javascript
function isValidOrigin(origin) {
  const validOrigins = [
    /^https:\/\/example\.com(:\d+)?$/,
    /^https:\/\/app\.example\.com(:\d+)?$/
  ];
  
  return validOrigins.some(pattern => pattern.test(origin));
}
```

### 4. Минимизация разрешенных методов и заголовков

```javascript
// Указывайте только необходимые методы и заголовки
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type, Authorization
```

## Практические примеры настройки CORS

### В Express.js с использованием cors middleware

```javascript
const cors = require('cors');

// Безопасная настройка для production
const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = [
      'https://example.com',
      'https://app.example.com'
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('CORS не разрешен для этого источника'));
    }
  },
  credentials: true,
  optionsSuccessStatus: 200
};

app.use(cors(corsOptions));
```

### В Nginx

```nginx
location /api/ {
    # Проверка источника
    if ($http_origin ~* (https://example\.com|https://app\.example\.com)$ ) {
        set $cors "true";
    }
    
    if ($cors = "true") {
        add_header 'Access-Control-Allow-Origin' "$http_origin";
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
    }
    
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
    }
    
    proxy_pass http://backend;
}
```

## Тестирование безопасности CORS

### Проверка разрешенных источников

Создайте HTML-страницу на подозрительном домене:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CORS Test</title>
</head>
<body>
    <script>
        // Попытка доступа к API с подозрительного домена
        fetch('https://api.example.com/data', {
            method: 'GET',
            credentials: 'include'
        })
        .then(response => response.text())
        .then(data => console.log(data))
        .catch(error => console.error('Ошибка:', error));
    </script>
</body>
</html>
```

### Использование инструментов

- Burp Suite для тестирования CORS
- OWASP ZAP
- CORS Buster (инструмент для тестирования уязвимостей CORS)

## Обнаружение и предотвращение атак

### Логирование подозрительных запросов

```javascript
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const userAgent = req.headers['user-agent'];
  
  // Логирование необычных источников
  if (origin && !isValidOrigin(origin)) {
    console.warn(`Подозрительный CORS запрос от: ${origin} | User-Agent: ${userAgent}`);
  }
  
  next();
});
```

### Мониторинг аномалий

Отслеживайте:

- Необычно высокое количество CORS-запросов
- Запросы с неожиданных источников
- Частые preflight-запросы

## Заключение

CORS - мощный, но сложный механизм безопасности. Неправильная настройка может привести к серьезным уязвимостям, позволяющим злоумышленникам получить доступ к приватным данным пользователей. Всегда используйте явные разрешения, избегайте wildcard-шаблонов при работе с credentials, и регулярно тестируйте настройки безопасности.

## Связанные темы

- [[Функции-безопасности-браузера]]
- [[Безопасность-хранилища-браузера]]
- [[Лучшие-практики-безопасности-API]]
- [[Проверка-ввода]]