---
aliases: [DOM Based XSS, DOM XSS, XSS на основе DOM, Межсайтовый скриптинг DOM]
tags: [security, xss, dom, injection]
---

# DOM-Based XSS

## Обзор

DOM-Based XSS (Cross-Site Scripting на основе DOM) - это тип межсайтового скриптинга, при котором вредоносный скрипт внедряется через манипуляции с DOM (Document Object Model) в браузере жертвы, а не через серверную часть. В отличие от традиционных форм XSS, DOM-Based XSS полностью происходит на стороне клиента и может обойти многие серверные методы защиты.

## Различие между типами XSS

### Reflected XSS
- Вредоносный скрипт передается через URL или форму
- Сервер включает его в ответ
- Выполняется при получении ответа

### Stored XSS
- Вредоносный скрипт сохраняется на сервере
- Выполняется при каждом доступе к зараженному контенту

### DOM-Based XSS
- Вредоносный скрипт обрабатывается исключительно на клиенте
- Никогда не отправляется на сервер
- Выполняется при манипуляциях с DOM

## Механизмы DOM-Based XSS

### 1. Чтение из уязвимых источников

DOM-Based XSS происходит, когда приложение читает данные из потенциально небезопасных источников:

```javascript
// Уязвимый код - чтение из URL
function getParameterByName(name) {
    const url = window.location.href;
    const regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)');
    const results = regex.exec(url);
    
    if (results && results[2]) {
        return decodeURIComponent(results[2].replace(/\+/g, ' '));
    }
    return null;
}

// Использование уязвимого параметра
const userInput = getParameterByName('search');
document.getElementById('searchResult').innerHTML = userInput; // Уязвимость!
```

### 2. Запись в уязвимые приемники

Приемники (sinks) - это методы DOM, которые могут выполнить вредоносный код:

```javascript
// Опасные методы и свойства
document.write(userInput);           // Уязвимо
document.writeln(userInput);         // Уязвимо
element.innerHTML = userInput;       // Уязвимо
element.outerHTML = userInput;       // Уязвимо
element.insertAdjacentHTML(...);     // Уязвимо
eval(userInput);                     // Уязвимо
setTimeout(userInput, ...);          // Уязвимо
setInterval(userInput, ...);         // Уязвимо
location.href = userInput;           // Уязвимо
window.location = userInput;         // Уязвимо
```

## Распространенные источники (Sources)

### 1. URL-параметры
```javascript
// window.location, document.URL, document.documentURI
const url = window.location.href;
const search = window.location.search;
const hash = window.location.hash;
```

### 2. Реферер
```javascript
// document.referrer
const referrer = document.referrer;
```

### 3. Данные формы
```javascript
// Значения из input, textarea и т.д.
const userInput = document.getElementById('inputField').value;
```

### 4. Данные из localStorage/sessionStorage
```javascript
// Получение данных из хранилища
const storedData = localStorage.getItem('userInput');
const sessionData = sessionStorage.getItem('formData');
```

## Примеры уязвимостей

### 1. Уязвимость через URL-хэш
```html
<!DOCTYPE html>
<html>
<head>
    <title>Уязвимая страница</title>
</head>
<body>
    <div id="content"></div>
    <script>
        // Уязвимый код
        document.getElementById('content').innerHTML = location.hash.substring(1);
    </script>
</body>
</html>
```

Злоумышленник может использовать URL вида:
```
http://example.com/page.html#<img src=x onerror=alert('XSS')>
```

### 2. Уязвимость через localStorage
```javascript
// Уязвимый код
function displayStoredMessage() {
    const message = localStorage.getItem('userMessage');
    document.getElementById('messageBox').innerHTML = message;
}

// Потенциально вредоносный код может установить:
localStorage.setItem('userMessage', '<img src=x onerror=alert(1)>');
```

### 3. Уязвимость через событие
```javascript
// Уязвимый обработчик события
document.getElementById('button').onclick = function() {
    const userInput = document.getElementById('input').value;
    eval(userInput); // Крайне уязвимо!
};
```

## Типичные сценарии атак

### 1. Атака через URL-параметры
```javascript
// Пример уязвимого кода
function processUrl() {
    const params = new URLSearchParams(window.location.search);
    const redirect = params.get('redirect');
    
    if (redirect) {
        // Уязвимость: перенаправление может быть подделано
        window.location = redirect;
    }
}

// Злоумышленник может использовать:
// http://example.com/page?redirect=javascript:alert('XSS')
```

### 2. Атака через JSON.parse
```javascript
// Уязвимый код
function parseUserData() {
    const userData = localStorage.getItem('userPreferences');
    const parsed = JSON.parse(userData); // Может быть манипулирован
    
    // Использование parsed данных без проверки
    document.getElementById('profile').innerHTML = parsed.name;
}
```

### 3. Атака через шаблонизатор
```javascript
// Уязвимый клиентский шаблонизатор
function renderTemplate(template, data) {
    // Простая замена переменных - потенциально уязвимо
    let result = template;
    for (const key in data) {
        result = result.replace(new RegExp(`{{${key}}}`, 'g'), data[key]);
    }
    return result;
}

// Использование
const template = '<div>{{username}}</div>';
const userData = { username: '<img src=x onerror=alert(1)>' };
document.getElementById('container').innerHTML = renderTemplate(template, userData);
```

## Защитные меры

### 1. Санитизация ввода

```javascript
// Безопасная санитизация HTML
function sanitizeHTML(str) {
    const temp = document.createElement('div');
    temp.textContent = str;
    return temp.innerHTML;
}

// Использование безопасного метода
const userInput = getParameterByName('search');
document.getElementById('searchResult').textContent = sanitizeHTML(userInput);
```

### 2. Использование безопасных методов DOM

```javascript
// Вместо innerHTML используйте textContent для текста
element.textContent = userInput; // Безопасно для текста

// Или создавайте элементы программно
function safeCreateElement(tag, content) {
    const element = document.createElement(tag);
    element.textContent = content;
    return element;
}

// Использование
const div = safeCreateElement('div', userInput);
document.getElementById('container').appendChild(div);
```

### 3. Проверка и валидация данных

```javascript
// Валидация URL перед использованием
function isValidUrl(string) {
    try {
        const url = new URL(string);
        // Проверка протокола
        return url.protocol === 'https:' || url.protocol === 'http:';
    } catch (_) {
        return false;
    }
}

// Использование
const redirectUrl = new URLSearchParams(window.location.search).get('redirect');
if (redirectUrl && isValidUrl(redirectUrl)) {
    window.location = redirectUrl;
}
```

### 4. Использование библиотек санитизации

```javascript
// Использование DOMPurify для санитизации
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(userInput);
document.getElementById('content').innerHTML = clean;
```

## Современные подходы к защите

### 1. Content Security Policy (CSP)

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self'; object-src 'none';">
```

CSP может предотвратить выполнение внедренных скриптов.

### 2. Strict CSP Nonce

```html
<meta http-equiv="Content-Security-Policy" 
      content="script-src 'nonce-r4nd0mNumb3r';">
```

### 3. Subresource Integrity

```html
<script src="https://example.com/script.js" 
        integrity="sha384-..." 
        crossorigin="anonymous">
</script>
```

## Автоматизированное обнаружение

### 1. Инструменты статического анализа

Для обнаружения потенциальных уязвимостей DOM-Based XSS можно использовать инструменты:

```javascript
// Пример проверки на наличие уязвимых паттернов
function analyzeDOMManipulation(code) {
    const vulnerablePatterns = [
        /\.innerHTML\s*=/,
        /\.outerHTML\s*=/,
        /eval\s*\(/,
        /setTimeout\s*\([^"]/,  // setTimeout с нестроковым первым параметром
        /setInterval\s*\([^"]/, // setInterval с нестроковым первым параметром
        /Function\s*\(/,
        /document\.write/,
        /document\.writeln/
    ];
    
    for (const pattern of vulnerablePatterns) {
        if (pattern.test(code)) {
            console.warn('Potential DOM XSS vulnerability found:', pattern);
        }
    }
}
```

### 2. Runtime Detection

```javascript
// Пример системы обнаружения XSS во время выполнения
class XSSDetector {
    constructor() {
        this.originalInnerHTML = Element.prototype.innerHTML;
        this.setupProtection();
    }
    
    setupProtection() {
        const self = this;
        
        Object.defineProperty(Element.prototype, 'innerHTML', {
            set: function(value) {
                if (self.containsScript(value)) {
                    console.error('Potential XSS attempt detected:', value);
                    // Логирование или другие меры реагирования
                    return;
                }
                self.originalInnerHTML.call(this, value);
            },
            get: function() {
                return self.originalInnerHTML.call(this);
            }
        });
    }
    
    containsScript(html) {
        const scriptRegex = /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi;
        const eventHandlers = [
            /on\w+\s*=/gi,  // Обработчики событий
            /javascript:/gi, // JavaScript URL
            /data:/gi        // Data URL
        ];
        
        if (scriptRegex.test(html)) return true;
        
        for (const handler of eventHandlers) {
            if (handler.test(html)) return true;
        }
        
        return false;
    }
}

// Инициализация детектора
new XSSDetector();
```

## Тестирование на DOM-Based XSS

### 1. Ручное тестирование

```javascript
// Payloads для тестирования
const testPayloads = [
    '<img src=x onerror=alert(1)>',
    'javascript:alert(1)',
    '<svg onload=alert(1)>',
    '"><script>alert(1)</script>',
    'javascript://%0aalert(1)',
    '<iframe src="javascript:alert(1)">'
];

// Тестирование параметров URL
function testUrlParameter(paramName, payload) {
    const url = new URL(window.location);
    url.searchParams.set(paramName, payload);
    console.log('Testing payload:', url.toString());
}
```

### 2. Автоматизированное тестирование

```javascript
// Пример автоматизированного тестирования
class DOMXSSTester {
    constructor() {
        this.payloads = [
            '<img src=x onerror=alert("DOM-XSS-TEST")>',
            '"><script>alert("DOM-XSS-TEST")</script>',
            'javascript:alert("DOM-XSS-TEST")'
        ];
        this.sources = ['hash', 'search', 'referrer'];
    }
    
    async runTests() {
        for (const source of this.sources) {
            for (const payload of this.payloads) {
                await this.testSource(source, payload);
            }
        }
    }
    
    testSource(source, payload) {
        // Тестирование конкретного источника с пейлоадом
        console.log(`Testing ${source} with payload: ${payload}`);
        // Реализация тестирования
    }
}
```

## Лучшие практики

### 1. Принцип минимальных привилегий

```javascript
// Ограничьте возможности скриптов
function createSandboxedEnvironment() {
    // Ограничение доступа к опасным API
    const sandbox = {
        safeMethods: {
            createElement: document.createElement.bind(document),
            getElementById: document.getElementById.bind(document),
            addEventListener: window.addEventListener.bind(window)
        }
    };
    return sandbox;
}
```

### 2. Использование шаблонизаторов с автоматической эскейпингом

```javascript
// Использование безопасного шаблонизатора
const template = Handlebars.compile('<div>{{username}}</div>');
// Handlebars автоматически экранирует HTML
const html = template({ username: userInput });
document.getElementById('container').innerHTML = html;
```

### 3. Регулярный аудит кода

Проводите регулярные проверки на наличие потенциальных уязвимостей DOM-Based XSS.

## Заключение

DOM-Based XSS представляет собой сложную и часто недооцениваемую угрозу безопасности веб-приложений. В отличие от традиционных форм XSS, она полностью происходит на стороне клиента и может обойти многие серверные методы защиты. Понимание механизмов таких атак и реализация соответствующих мер защиты критически важны для обеспечения безопасности веб-приложений.

> [!tip] Совет
> Всегда проверяйте и санитизируйте данные из любых источников перед их использованием в DOM.

> [!warning] Важно
> DOM-Based XSS может обойти серверные методы защиты, требуя клиентских мер безопасности.

> [!note] Примечание
> Используйте сочетание CSP, санитизации и безопасного программирования для полной защиты от XSS атак.