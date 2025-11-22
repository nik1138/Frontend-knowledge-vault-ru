---
aliases: ["Обнаружение Clickjacking", "Обнаружение атак фрейминга", "Анализ Clickjacking"]
tags: [security, clickjacking, detection, web-security, attack-analysis]
---

# Обнаружение Clickjacking

## Введение

Обнаружение Clickjacking — это процесс выявления уязвимостей, которые позволяют злоумышленникам замаскировать элементы управления другого сайта в iframe и заставить пользователей выполнять нежелательные действия. Clickjacking (или UI redress) представляет собой атаку, при которой пользователь обманом заставляется кликнуть по элементу, который не виден или замаскирован.

## Механизмы атаки Clickjacking

### 1. Базовая атака

Злоумышленник создает веб-страницу с невидимым iframe, поверх которого размещает свои элементы управления. Когда пользователь кликает по "безопасному" элементу, на самом деле он кликает по скрытому iframe.

### 2. Атака с использованием CSS

Злоумышленник использует CSS для изменения прозрачности, позиционирования и видимости элементов, чтобы скрыть уязвимый сайт и замаскировать клики.

## Методы обнаружения уязвимостей

### 1. Анализ HTTP-заголовков

Проверка наличия заголовков защиты:

```bash
# Проверка X-Frame-Options
curl -I https://target-site.com/page

# Проверка CSP frame-ancestors
curl -I https://target-site.com/page | grep -i 'content-security-policy'
```

### 2. Тестирование фрейминга

Создание тестовой HTML-страницы для проверки возможности фрейминга:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Clickjacking Test Page</title>
</head>
<body>
    <h1>Clickjacking Test</h1>
    <p>Если вы видите содержимое целевого сайта, значит, он уязвим к Clickjacking.</p>
    
    <div style="position: relative; height: 600px; width: 800px;">
        <iframe src="https://target-site.com/sensitive-page" 
                style="position: absolute; top: 0; left: 0; opacity: 0.1; width: 100%; height: 100%; z-index: 1;">
        </iframe>
        
        <div style="position: absolute; top: 300px; left: 300px; z-index: 2; background: white; padding: 10px;">
            <button>Click me!</button>
            <p>Эта кнопка может быть поверх важного элемента управления</p>
        </div>
    </div>
</body>
</html>
```

### 3. Автоматизированные инструменты

#### OWASP ZAP

```bash
# Запуск ZAP для сканирования на уязвимости Clickjacking
zap.sh -cmd -quickurl https://target-site.com -quickprogress -quickout report.html
```

#### Burp Suite

Использование встроенных сканеров Burp Suite для обнаружения уязвимостей фрейминга.

#### Скрипты для автоматического тестирования

```python
import requests
from urllib.parse import urlparse

def check_clickjacking_vulnerability(url):
    """
    Проверка уязвимости к Clickjacking
    """
    try:
        response = requests.get(url)
        
        # Проверка заголовков
        x_frame_options = response.headers.get('X-Frame-Options', '').upper()
        csp = response.headers.get('Content-Security-Policy', '').lower()
        
        vulnerabilities = []
        
        # Проверка X-Frame-Options
        if not x_frame_options or x_frame_options == 'ALLOW-FROM':
            vulnerabilities.append('X-Frame-Options отсутствует или использует ALLOW-FROM')
        
        # Проверка CSP
        if 'frame-ancestors' not in csp:
            vulnerabilities.append('Content-Security-Policy не содержит frame-ancestors')
        
        # Попытка фрейминга
        test_html = f"""
        <html>
        <head>
            <title>Clickjacking Test</title>
        </head>
        <body>
            <h1>Clickjacking Test</h1>
            <iframe width="900" height="600" src="{url}" frameborder="0"></iframe>
        </body>
        </html>
        """
        
        return {
            'url': url,
            'x_frame_options': x_frame_options,
            'csp': csp,
            'vulnerabilities': vulnerabilities,
            'test_page': test_html
        }
        
    except Exception as e:
        return {'url': url, 'error': str(e)}

# Пример использования
result = check_clickjacking_vulnerability('https://target-site.com')
print(result)
```

## Сигнатуры уязвимостей

### 1. Отсутствие заголовков защиты

```javascript
// Проверка отсутствия защиты на стороне клиента
function isVulnerableToClickjacking() {
    try {
        // Если страница может быть загружена в iframe, она уязвима
        return window.self !== window.top;
    } catch (e) {
        // Если не можем получить доступ к window.top, это может указывать на защиту
        return false;
    }
}
```

### 2. Неправильная настройка заголовков

```javascript
// Проверка на слабые настройки
function checkWeakFrameOptions(responseHeaders) {
    const xfo = responseHeaders['x-frame-options'];
    const csp = responseHeaders['content-security-policy'];
    
    const weakPatterns = [
        'ALLOW-FROM',  // Не поддерживается современными браузерами
        'ALLOWALL',    // Несуществующее значение
        ''             // Отсутствие заголовка
    ];
    
    return weakPatterns.some(pattern => 
        xfo && xfo.toUpperCase().includes(pattern)
    );
}
```

## Методы обнаружения на стороне сервера

### 1. Логирование подозрительных запросов

```javascript
// Middleware для обнаружения подозрительных запросов
function clickjackingDetection(req, res, next) {
    // Проверка заголовка Referer
    const referer = req.headers.referer;
    const origin = req.headers.origin;
    
    // Логирование запросов из неожиданных источников
    if (referer && !referer.includes(req.get('host'))) {
        console.warn(`Potential clickjacking attempt from: ${referer}`);
    }
    
    // Проверка заголовка X-Frame-Options у ответа
    res.on('finish', () => {
        const xfo = res.getHeader('X-Frame-Options');
        if (!xfo) {
            console.warn(`Missing X-Frame-Options for: ${req.originalUrl}`);
        }
    });
    
    next();
}

app.use(clickjackingDetection);
```

### 2. Анализ паттернов трафика

```javascript
// Анализ паттернов, характерных для clickjacking
class ClickjackingAnalyzer {
    constructor() {
        this.suspiciousPatterns = new Map();
    }
    
    analyzeRequest(req) {
        const key = this.generateRequestKey(req);
        const count = this.suspiciousPatterns.get(key) || 0;
        this.suspiciousPatterns.set(key, count + 1);
        
        // Проверка на подозрительные паттерны
        if (this.isSuspicious(req, count + 1)) {
            this.logSuspiciousActivity(req);
        }
    }
    
    generateRequestKey(req) {
        // Генерация ключа на основе IP, User-Agent и Referer
        return `${req.ip}-${req.get('User-Agent')}-${req.get('Referer')}`;
    }
    
    isSuspicious(req, count) {
        // Проверка на частые запросы с необычных рефереров
        return count > 10 && !req.get('Referer')?.includes(req.get('Host'));
    }
    
    logSuspiciousActivity(req) {
        console.log('Suspicious activity detected:', {
            ip: req.ip,
            url: req.originalUrl,
            userAgent: req.get('User-Agent'),
            referer: req.get('Referer')
        });
    }
}

const analyzer = new ClickjackingAnalyzer();
app.use((req, res, next) => {
    analyzer.analyzeRequest(req);
    next();
});
```

## Методы обнаружения на стороне клиента

### 1. Проверка контекста фрейминга

```javascript
// Проверка, находится ли страница во фрейме
function checkFramingContext() {
    if (window.self !== window.top) {
        // Страница находится во фрейме
        console.warn('Page is framed');
        
        // Проверка, является ли фрейминг ожидаемым
        const expectedParents = ['https://trusted-site.com'];
        const parentOrigin = document.referrer;
        
        if (!expectedParents.some(domain => parentOrigin.includes(domain))) {
            // Подозрительный фрейминг
            alert('This page appears to be framed unexpectedly!');
            return false;
        }
    }
    return true;
}

// Запуск проверки при загрузке
window.addEventListener('load', checkFramingContext);
```

### 2. Защита с помощью JavaScript

```javascript
// Защита от фрейминга с помощью JavaScript
(function() {
    if (window.top !== window.self) {
        // Если страница во фрейме, перенаправляем на себя
        window.top.location = window.self.location;
    }
})();
```

## Тестирование на уязвимость

### 1. Ручное тестирование

Шаги для ручного тестирования:
1. Открыть целевую страницу в браузере
2. Создать HTML-страницу с iframe, содержащим целевую страницу
3. Проверить, отображается ли содержимое во фрейме
4. Проверить наличие заголовков X-Frame-Options и CSP

### 2. Автоматизированное тестирование

```bash
#!/bin/bash
# Скрипт для автоматического тестирования Clickjacking

URLS=("https://example.com/page1" "https://example.com/page2")

for url in "${URLS[@]}"; do
    echo "Testing: $url"
    
    # Получение заголовков
    response=$(curl -s -I "$url")
    
    # Проверка X-Frame-Options
    xfo=$(echo "$response" | grep -i "X-Frame-Options" | head -n1)
    if [ -z "$xfo" ]; then
        echo "  ❌ X-Frame-Options отсутствует"
    else
        echo "  ✅ X-Frame-Options: $xfo"
    fi
    
    # Проверка CSP
    csp=$(echo "$response" | grep -i "Content-Security-Policy" | head -n1)
    if [ -z "$csp" ]; then
        echo "  ❌ Content-Security-Policy отсутствует"
    else
        if echo "$csp" | grep -i "frame-ancestors"; then
            echo "  ✅ CSP frame-ancestors присутствует"
        else
            echo "  ❌ CSP frame-ancestors отсутствует"
        fi
    fi
    
    echo "---"
done
```

## Лучшие практики обнаружения

> [!tip] Лучшие практики обнаружения Clickjacking
> 1. Регулярно проверяйте HTTP-заголовки безопасности
> 2. Используйте автоматизированные инструменты для сканирования
> 3. Тестируйте все чувствительные страницы
> 4. Анализируйте логи на предмет подозрительных паттернов
> 5. Проводите регулярные аудиты безопасности

### Комплексная проверка

```javascript
// Комплексный инструмент для проверки уязвимостей
class ClickjackingScanner {
    constructor() {
        this.results = [];
    }
    
    async scanEndpoint(url) {
        try {
            const response = await fetch(url, { method: 'HEAD' });
            const headers = Object.fromEntries(response.headers.entries());
            
            const result = {
                url: url,
                headers: headers,
                vulnerabilities: [],
                secure: true
            };
            
            // Проверка X-Frame-Options
            if (!headers['x-frame-options']) {
                result.vulnerabilities.push('Missing X-Frame-Options header');
                result.secure = false;
            } else {
                const xfo = headers['x-frame-options'].toUpperCase();
                if (xfo === 'ALLOW-FROM' || xfo === 'ALLOWALL') {
                    result.vulnerabilities.push('Weak X-Frame-Options value');
                    result.secure = false;
                }
            }
            
            // Проверка CSP
            if (!headers['content-security-policy']?.includes('frame-ancestors')) {
                result.vulnerabilities.push('Missing CSP frame-ancestors directive');
                result.secure = false;
            }
            
            this.results.push(result);
            return result;
        } catch (error) {
            console.error(`Error scanning ${url}:`, error);
            return null;
        }
    }
    
    async scanMultiple(urls) {
        const promises = urls.map(url => this.scanEndpoint(url));
        await Promise.all(promises);
        return this.results;
    }
    
    getReport() {
        const vulnerable = this.results.filter(r => !r.secure);
        const secure = this.results.filter(r => r.secure);
        
        return {
            total: this.results.length,
            vulnerable: vulnerable.length,
            secure: secure.length,
            details: this.results
        };
    }
}

// Использование
const scanner = new ClickjackingScanner();
const urls = [
    'https://example.com/login',
    'https://example.com/settings',
    'https://example.com/payment'
];

scanner.scanMultiple(urls).then(() => {
    console.log(scanner.getReport());
});
```

## Связь с другими аспектами безопасности

Обнаружение Clickjacking тесно связано с:
- [[X-Frame-Options]] — основной метод защиты
- [[Content-Security-Policy]] — альтернативный метод защиты
- [[HTTP-Security-Headers]] — общая концепция заголовков безопасности
- [[Тестирование-безопасности]] — методы проверки уязвимостей
- [[Защита-от-атак-на-уровне-браузера]] — комплексная защита

## Заключение

Обнаружение уязвимостей к Clickjacking требует комплексного подхода, включающего как автоматизированные инструменты, так и ручное тестирование. Регулярное сканирование и анализ HTTP-заголовков позволяют выявлять уязвимости до того, как они могут быть использованы злоумышленниками. Важно сочетать методы обнаружения с надежными мерами защиты для обеспечения безопасности веб-приложений.

## Дополнительные ресурсы

- OWASP Clickjacking Attack and Defense
- RFC 7034 (X-Frame-Options)
- Content Security Policy Documentation