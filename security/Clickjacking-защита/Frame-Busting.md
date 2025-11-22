---
aliases: [Frame Busting, Защита от фрейминга, Защита от Clickjacking]
tags: [security, clickjacking, frame-busting, javascript, web-security, frontend-security]
---

# Frame Busting: Защита от Clickjacking атак

## Введение в Frame Busting

Frame Busting - это техника веб-безопасности, предназначенная для предотвращения встраивания веб-страниц в iframe на сторонних сайтах. Эта защита была разработана для борьбы с атаками типа Clickjacking (или UI Redress), при которых злоумышленник создает вредоносную страницу, которая визуально маскирует интерфейс другого сайта, чтобы обмануть пользователя и заставить его выполнить нежелательные действия.

Clickjacking представляет собой серьезную угрозу для веб-приложений, особенно для тех, которые обрабатывают чувствительные операции, такие как банковские транзакции, изменение настроек аккаунта или выполнение административных действий. Frame Busting был одним из первых методов защиты от этих атак, хотя в настоящее время существуют более надежные альтернативы.

## Что такое Frame Busting и зачем он нужен

Frame Busting - это клиентская защита, реализуемая с помощью JavaScript, которая проверяет, загружена ли текущая страница в iframe, и если да, то пытается "взорвать" (bust) фрейм, перенаправив родительское окно на саму страницу. Основная цель Frame Busting - предотвратить встраивание критических страниц в iframe на вредоносных сайтах.

> [!warning] Важно понимать
> Frame Busting не является надежной защитой от Clickjacking и в настоящее время считается устаревшей техникой. Современные браузеры и HTTP-заголовки предоставляют более эффективные методы защиты.

### Основные цели Frame Busting:
- Предотвращение встраивания страниц в iframe
- Защита от Clickjacking атак
- Обеспечение прямого доступа к странице пользователями

## История и развитие метода

Frame Busting метод был разработан в начале 2000-х годов как ответ на появление атак типа Clickjacking. Первоначально это был простой JavaScript-код, который проверял, является ли текущая страница верхним окном браузера, и если нет, то перенаправлял родительское окно.

С развитием веб-технологий и появлением более изощренных методов обхода Frame Busting, разработчики начали создавать более сложные реализации с использованием различных событий, интервалов и обработчиков ошибок. Однако, несмотря на эти усилия, Frame Busting оставался уязвимым к различным обходам.

В 2009 году были представлены HTTP-заголовки X-Frame-Options, которые обеспечивали более надежную защиту от фрейминга на уровне браузера. Позже появилась более гибкая директива frame-ancestors в Content Security Policy, которая стала современным стандартом для защиты от Clickjacking.

## Методы реализации

### 1. Базовая реализация

Самая простая форма Frame Busting проверяет, является ли текущее окно верхним окном браузера:

```javascript
if (window !== window.top) {
    window.top.location = window.location;
}
```

### 2. Проверка через setInterval

Для повышения надежности Frame Busting может использовать интервалы для периодической проверки:

```javascript
(function() {
    if (window.top !== window.self) {
        window.top.location = window.self.location;
    }
    
    setInterval(function() {
        if (window.top !== window.self) {
            window.top.location = window.self.location;
        }
    }, 100);
})();
```

### 3. Обработка событий

Frame Busting может реагировать на различные события, чтобы убедиться, что защита активна:

```javascript
(function() {
    function bustFrame() {
        if (window.top !== window.self) {
            window.top.location = window.self.location;
        }
    }

    // Проверка при загрузке
    bustFrame();

    // Проверка при событиях
    window.addEventListener('load', bustFrame);
    window.addEventListener('focus', bustFrame);
    window.addEventListener('blur', bustFrame);
})();
```

### 4. Защищенная реализация с обработкой ошибок

Более надежная реализация включает обработку ошибок и защиту от обходов:

```javascript
(function() {
    var protected = false;
    
    function protect() {
        if (protected) return;
        
        try {
            if (window.self !== window.top) {
                protected = true;
                
                // Пытаемся перенаправить родительское окно
                window.top.location = window.self.location;
            }
        } catch (e) {
            // Если не удается получить доступ к window.top,
            // используем альтернативные методы
            document.body.style.display = 'none';
            document.location = document.location;
        }
    }

    // Несколько методов проверки
    protect();
    
    // Проверка через интервал
    var interval = setInterval(protect, 100);
    
    // Очистка интервала при полной загрузке
    window.addEventListener('load', function() {
        clearInterval(interval);
        protect();
    });
    
    // Проверка при фокусе
    window.addEventListener('focus', protect);
    window.addEventListener('blur', protect);
})();
```

## Примеры кода

### Пример 1: Простая проверка Frame Busting

```javascript
// Простой Frame Busting
if (top.location !== self.location) {
    top.location = self.location;
}
```

### Пример 2: Frame Busting с проверкой домена

```javascript
(function() {
    var allowedDomains = [
        'https://yourdomain.com',
        'https://www.yourdomain.com'
    ];
    
    var isAllowed = false;
    
    try {
        for (var i = 0; i < allowedDomains.length; i++) {
            if (top.location.hostname === new URL(allowedDomains[i]).hostname) {
                isAllowed = true;
                break;
            }
        }
    } catch (e) {
        // Ошибка доступа к top.location из-за разных доменов
        isAllowed = false;
    }
    
    if (!isAllowed && window.top !== window.self) {
        window.top.location = window.self.location;
    }
})();
```

### Пример 3: Frame Busting с обфускацией

```javascript
// Обфусцированный Frame Busting
(function() {
    var a = window;
    var b = a.top;
    var c = a.self;
    
    if (b !== c) {
        try {
            b.location = c.location;
        } catch (e) {
            // Альтернативный метод
            a.location = c.location;
        }
    }
})();
```

### Пример 4: Современный Frame Busting с использованием CSP

```javascript
// Frame Busting с проверкой CSP
(function() {
    // Проверяем, можно ли использовать CSP
    var supportsCSP = function() {
        return window.trustedTypes || 
               document.querySelector('meta[http-equiv="Content-Security-Policy"]') !== null;
    };
    
    if (!supportsCSP()) {
        // Используем Frame Busting как резервный вариант
        if (window.top !== window.self) {
            window.top.location = window.self.location;
        }
    }
})();
```

## Обходы Frame Busting

Несмотря на различные методы реализации, Frame Busting может быть обойден различными способами:

### 1. Использование sandbox атрибута

```html
<iframe src="target.html" sandbox="allow-forms allow-popups allow-same-origin"></iframe>
```

Sandbox атрибут может ограничить возможности скриптов внутри iframe, предотвращая выполнение Frame Busting кода.

### 2. Ограничение доступа к window.top

```javascript
// Злоумышленник может ограничить доступ к window.top
Object.defineProperty(window, 'top', {
    get: function() {
        return this;
    }
});
```

### 3. Использование двойного фрейминга

Злоумышленник может использовать промежуточный iframe, чтобы обойти Frame Busting:

```html
<!-- Злоумышленник использует двойное встраивание -->
<iframe src="intermediate.html">
  <!-- intermediate.html в свою очередь встраивает target.html -->
</iframe>
```

### 4. Блокировка JavaScript

Если JavaScript отключен в браузере, Frame Busting не будет работать.

### 5. Использование postMessage API

Злоумышленник может использовать postMessage для обхода Frame Busting:

```javascript
// В промежуточном фрейме
window.addEventListener('message', function(e) {
    if (e.data === 'bust') {
        // Злоумышленник может обойти Frame Busting через postMessage
    }
});
```

## Современные альтернативы

### 1. X-Frame-Options заголовок

[[X-Frame-Options]] - это HTTP-заголовок, который указывает браузеру, можно ли отображать страницу в iframe:

```
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com/
```

### 2. Content Security Policy frame-ancestors

Более гибкая альтернатива X-Frame-Options:

```
Content-Security-Policy: frame-ancestors 'none';
Content-Security-Policy: frame-ancestors 'self';
Content-Security-Policy: frame-ancestors https://example.com/;
```

### 3. Feature Policy

```
Feature-Policy: sync-xhr 'none';
```

## Сравнение с другими методами защиты

| Метод | Надежность | Совместимость | Сложность | Рекомендация |
|-------|------------|---------------|-----------|--------------|
| Frame Busting | Низкая | Высокая | Средняя | Не рекомендуется как основной метод |
| X-Frame-Options | Высокая | Высокая | Низкая | Рекомендуется |
| CSP frame-ancestors | Очень высокая | Высокая | Низкая | Наиболее рекомендуемый метод |
| SameSite cookies | Средняя | Высокая | Низкая | Дополнительная защита |

## Практические рекомендации

### 1. Используйте HTTP-заголовки как основную защиту

```http
Content-Security-Policy: frame-ancestors 'self';
X-Frame-Options: SAMEORIGIN
```

### 2. Комбинируйте методы защиты

Используйте несколько методов защиты одновременно для повышения безопасности:

- HTTP-заголовки (X-Frame-Options, CSP)
- Frame Busting как дополнительная мера
- Правильная обработка сессий и аутентификации

### 3. Тестируйте на разных браузерах

Проверяйте защиту на различных браузерах и версиях, так как поведение может отличаться.

### 4. Регулярно обновляйте методы защиты

Следите за новыми уязвимостями и методами обхода Frame Busting.

### 5. Не полагайтесь только на Frame Busting

Frame Busting не должен быть единственным методом защиты от Clickjacking.

## Тестирование Frame Busting

### 1. Тестирование в iframe

Создайте HTML-страницу, которая встраивает тестируемую страницу в iframe:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Тест Frame Busting</title>
</head>
<body>
    <h1>Тестирование Frame Busting</h1>
    <iframe src="https://yoursite.com/protected-page" width="800" height="600"></iframe>
</body>
</html>
```

### 2. Использование инструментов разработчика

Используйте инструменты разработчика браузера для проверки:

- Выполняется ли Frame Busting код
- Есть ли ошибки при выполнении
- Как браузер обрабатывает перенаправления

### 3. Автоматизированное тестирование

Создайте скрипты для автоматического тестирования Frame Busting:

```javascript
// Пример автоматизированного теста
function testFrameBusting() {
    var iframe = document.createElement('iframe');
    iframe.src = 'https://yoursite.com/protected-page';
    
    // Слушаем событие load
    iframe.onload = function() {
        // Проверяем, остался ли iframe на месте
        if (iframe.contentWindow.location.href) {
            console.log('Frame Busting не сработал');
        } else {
            console.log('Frame Busting сработал');
        }
    };
    
    document.body.appendChild(iframe);
}
```

## Возможные проблемы и побочные эффекты

### 1. Проблемы с производительностью

Frame Busting может создавать нагрузку на браузер из-за постоянных проверок:

```javascript
// Избегайте частых проверок
setInterval(function() {
    // Проверка каждые 100мс может быть избыточной
}, 100);
```

### 2. Конфликты с легитимными iframe

Frame Busting может мешать легитимному использованию iframe на собственном сайте.

### 3. Проблемы с SEO

Поисковые роботы могут интерпретировать Frame Busting как подозрительное поведение.

### 4. Проблемы с отладкой

Frame Busting может затруднять отладку и тестирование приложений.

### 5. Проблемы с пользовательским опытом

Частые перенаправления могут ухудшить пользовательский опыт.

## Лучшие практики

### 1. Используйте HTTP-заголовки как основную защиту

```javascript
// Серверный код для установки заголовков
app.use((req, res, next) => {
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('Content-Security-Policy', "frame-ancestors 'self'");
    next();
});
```

### 2. Ограничьте использование Frame Busting

Применяйте Frame Busting только для критически важных страниц:

```javascript
// Проверка, нужна ли защита для текущей страницы
if (window.location.pathname.includes('/sensitive-action')) {
    // Применяем Frame Busting только для чувствительных страниц
    (function() {
        if (window.top !== window.self) {
            window.top.location = window.self.location;
        }
    })();
}
```

### 3. Обеспечьте совместимость

Проверяйте, как Frame Busting работает с различными браузерами и расширениями:

```javascript
// Совместимая реализация Frame Busting
(function() {
    var isFrameBusted = false;
    
    function attemptBust() {
        if (isFrameBusted) return;
        
        try {
            if (window.self !== window.top) {
                window.top.location = window.self.location;
                isFrameBusted = true;
            }
        } catch (e) {
            // Обработка ошибок при ограничении доступа к window.top
            console.warn('Frame Busting blocked by browser security');
        }
    }
    
    // Проверка при загрузке
    attemptBust();
    
    // Проверка при фокусе
    window.addEventListener('focus', attemptBust);
})();
```

### 4. Документируйте использование Frame Busting

Опишите, где и зачем используется Frame Busting в вашем приложении:

```javascript
/**
 * Frame Busting для защиты от Clickjacking
 * Используется на страницах с чувствительными операциями
 * Не рекомендуется как основной метод защиты
 */
(function() {
    // Реализация Frame Busting
})();
```

### 5. Регулярно обновляйте методы защиты

Следите за новыми угрозами и методами обхода Frame Busting.

## Ссылки на другие связанные файлы

- [[Clickjacking]] - основная статья о Clickjacking атаках
- [[X-Frame-Options]] - HTTP-заголовок для предотвращения фрейминга
- [[Content-Security-Policy]] - политика безопасности контента
- [[Cross-Site-Request-Forgery]] - атаки подделки межсайтовых запросов
- [[SameSite]] - атрибуты куки для предотвращения CSRF
- [[Cross-Origin-Resource-Sharing]] - механизм CORS
- [[Session-Fixation]] - атаки фиксации сессии
- [[Web-Security-Testing]] - тестирование безопасности веб-приложений
- [[Browser-Security-Policies]] - политики безопасности браузеров
- [[Frontend-Security]] - общие вопросы безопасности на фронтенде
- [[HTTP-Security-Headers]] - важные HTTP-заголовки безопасности
- [[OWASP-Top-10]] - топ-10 уязвимостей веб-приложений
- [[Security-Testing-Tools]] - инструменты для тестирования безопасности
- [[Secure-Coding-Practices]] - безопасное программирование
- [[Web-Application-Security]] - общие вопросы безопасности веб-приложений