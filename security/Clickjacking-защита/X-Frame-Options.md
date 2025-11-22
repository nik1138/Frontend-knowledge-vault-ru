---
aliases: ["X-Frame-Options заголовок", "Защита от фрейминга", "Фреймирование"]
tags: [security, x-frame-options, clickjacking, web-security, http-headers]
---

# X-Frame-Options

## Введение

X-Frame-Options — это HTTP-заголовок безопасности, который позволяет контролировать, может ли страница быть отображена в iframe на других сайтах. Этот заголовок был разработан для предотвращения атак типа clickjacking (также известных как UI redress), при которых злоумышленник замаскировывает элементы управления другого сайта в своем iframe.

## Типы значений X-Frame-Options

### 1. DENY

Полностью запрещает отображение страницы в iframe на любом сайте, включая тот же домен.

```
X-Frame-Options: DENY
```

Это наиболее безопасное значение, которое предотвращает любое встраивание страницы в iframe.

### 2. SAMEORIGIN

Разрешает отображение страницы в iframe только в том случае, если родительская страница находится в том же домене.

```
X-Frame-Options: SAMEORIGIN
```

Это значение позволяет использовать iframe для внутренних страниц, но защищает от внешнего фрейминга.

### 3. ALLOW-FROM uri

Разрешает отображение страницы в iframe только на указанном домене. Однако этот вариант не поддерживается во всех браузерах.

```
X-Frame-Options: ALLOW-FROM https://trusted-site.com
```

> [!warning] Поддержка ALLOW-FROM
> Значение ALLOW-FROM не поддерживается в последних версиях Chrome и Firefox, поэтому его использование не рекомендуется.

## Реализация X-Frame-Options

### В Express.js

```javascript
// Middleware для установки X-Frame-Options
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  next();
});

// Или для конкретного маршрута
app.get('/sensitive-page', (req, res) => {
  res.setHeader('X-Frame-Options', 'DENY');
  res.render('sensitive-page');
});
```

### В Apache

```
# В .htaccess или конфигурационном файле Apache
Header always set X-Frame-Options "SAMEORIGIN"
```

### В Nginx

```
# В конфигурационном файле Nginx
add_header X-Frame-Options "SAMEORIGIN" always;
```

### В ASP.NET

```csharp
// В web.config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <add name="X-Frame-Options" value="SAMEORIGIN" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

## Примеры использования

### Защита чувствительных страниц

```javascript
// Защита страницы с изменением пароля
app.get('/change-password', (req, res) => {
  res.setHeader('X-Frame-Options', 'DENY');
  res.render('change-password');
});

// Защита платежной формы
app.get('/payment', (req, res) => {
  res.setHeader('X-Frame-Options', 'DENY');
  res.render('payment-form');
});
```

### Разрешение внутреннего фрейминга

```javascript
// Разрешение фрейминга для внутренних страниц
app.use('/dashboard', (req, res, next) => {
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  next();
});
```

## Совместимость с браузерами

X-Frame-Options поддерживается всеми современными браузерами:

- Chrome: с версии 4.0
- Firefox: с версии 3.6.9
- Safari: с версии 4.0
- Internet Explorer: с версии 8.0
- Opera: с версии 10.50

## Ограничения X-Frame-Options

### 1. Отсутствие гибкости

Заголовок X-Frame-Options предоставляет только базовые опции и не позволяет задавать сложные правила фрейминга.

### 2. Отсутствие поддержки ALLOW-FROM

Как упоминалось выше, значение ALLOW-FROM не поддерживается в современных браузерах.

### 3. Невозможность динамической настройки

Заголовок должен быть установлен на уровне сервера и не может быть изменен динамически на стороне клиента.

## Альтернатива: Content Security Policy (CSP)

Content Security Policy предоставляет более гибкую альтернативу X-Frame-Options через директиву frame-ancestors:

```
Content-Security-Policy: frame-ancestors 'self' https://trusted-site.com;
```

### Сравнение X-Frame-Options и CSP

| Особенность | X-Frame-Options | CSP frame-ancestors |
|-------------|------------------|---------------------|
| Поддержка браузеров | Широкая, включая старые | Современные браузеры |
| Гибкость | Ограниченная | Высокая |
| Совместимость | Отличная | Хорошая |
| Настройка исключений | Ограниченная | Расширенная |

### Использование обоих заголовков

Для максимальной совместимости рекомендуется использовать оба заголовка:

```javascript
app.use((req, res, next) => {
  // Совместимость со старыми браузерами
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  
  // Современная защита
  res.setHeader('Content-Security-Policy', "frame-ancestors 'self'");
  
  next();
});
```

## Практические примеры

### Защита админ-панели

```javascript
// Middleware для защиты административных страниц
function protectAdminPages(req, res, next) {
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Content-Security-Policy', "frame-ancestors 'none'");
  next();
}

app.use('/admin', protectAdminPages);
```

### Условная защита

```javascript
// Защита в зависимости от типа запроса
app.use((req, res, next) => {
  // Для AJAX-запросов разрешаем фрейминг (если необходимо)
  if (req.xhr || req.headers['x-requested-with'] === 'XMLHttpRequest') {
    // Не устанавливаем X-Frame-Options для AJAX
  } else {
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  }
  next();
});
```

## Тестирование X-Frame-Options

### Проверка заголовка

```bash
# Проверка с помощью curl
curl -I https://yoursite.com/sensitive-page

# Ожидаемый результат:
# X-Frame-Options: DENY
# или
# X-Frame-Options: SAMEORIGIN
```

### Тестирование в браузере

Создайте HTML-страницу для тестирования:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Тест X-Frame-Options</title>
</head>
<body>
    <h1>Тест X-Frame-Options</h1>
    <iframe src="https://yoursite.com/sensitive-page" width="800" height="600"></iframe>
</body>
</html>
```

Если X-Frame-Options установлен правильно, iframe не будет отображаться для защищенных страниц.

## Лучшие практики

> [!tip] Лучшие практики X-Frame-Options
> 1. Используйте DENY для чувствительных страниц
> 2. Используйте SAMEORIGIN для внутренних страниц, которые могут быть во фреймах
> 3. Комбинируйте с CSP для лучшей совместимости
> 4. Регулярно тестируйте настройки безопасности

### Рекомендуемая конфигурация

```javascript
// Универсальный middleware для X-Frame-Options
function xFrameOptions(options = {}) {
  const defaultPolicy = options.policy || 'SAMEORIGIN';
  const paths = options.paths || {};
  
  return (req, res, next) => {
    let policy = defaultPolicy;
    
    // Проверка специфичных маршрутов
    for (const [path, pathPolicy] of Object.entries(paths)) {
      if (req.path.startsWith(path)) {
        policy = pathPolicy;
        break;
      }
    }
    
    res.setHeader('X-Frame-Options', policy);
    next();
  };
}

// Использование
app.use(xFrameOptions({
  policy: 'SAMEORIGIN',
  paths: {
    '/admin': 'DENY',
    '/payment': 'DENY',
    '/api': 'DENY'
  }
}));
```

## Связь с другими аспектами безопасности

X-Frame-Options тесно связан с:
- [[Обнаружение-Clickjacking]] — методы обнаружения атак
- [[Content-Security-Policy]] — более гибкая альтернатива
- [[HTTP-Security-Headers]] — общая концепция заголовков безопасности
- [[Защита-от-атак-на-уровне-браузера]] — комплексная защита
- [[Тестирование-безопасности]] — методы проверки защиты

## Заключение

X-Frame-Options остается важным инструментом для предотвращения атак clickjacking, несмотря на появление более гибких альтернатив в виде CSP. Правильное использование этого заголовка в сочетании с другими мерами безопасности обеспечивает надежную защиту веб-приложений от фрейминга.

## Дополнительные ресурсы

- RFC 7034 (HTTP Header Field X-Frame-Options)
- OWASP Clickjacking Defense Cheat Sheet
- Mozilla Developer Network - X-Frame-Options