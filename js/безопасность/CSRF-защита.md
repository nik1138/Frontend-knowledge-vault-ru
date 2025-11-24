---
aliases: [CSRF, Подделка межсайтовых запросов, Защита от CSRF]
tags: [javascript, безопасность, csrf, веб-безопасность]
---

# Защита от CSRF (Подделка межсайтовых запросов)

## Общее описание

CSRF (Cross-Site Request Forgery) - это тип атаки, при котором злоумышленник заставляет пользователя выполнить нежелательное действие в приложении, в котором пользователь аутентифицирован. В 2025 году CSRF остается серьезной угрозой для веб-приложений, особенно для банковских и финансовых сервисов, популярных в России.

## Как работает CSRF-атака

1. Пользователь аутентифицирован в доверенном веб-приложении
2. Пользователь посещает вредоносный сайт
3. Вредоносный сайт отправляет запрос к доверенному приложению от имени пользователя
4. Браузер автоматически отправляет куки аутентификации с запросом
5. Доверенное приложение обрабатывает запрос как легитимный

## Пример CSRF-атаки

```html
<!-- Вредоносная страница может содержать -->
<img src="https://bank.example.com/transfer?to=attacker&amount=10000" 
     width="0" height="0" style="display:none;">
```

При загрузке этой страницы, если пользователь аутентифицирован в банковском приложении, может произойти перевод средств без его ведома.

## Методы защиты от CSRF

### 1. CSRF-токены

Самый распространенный метод защиты - использование CSRF-токенов:

```javascript
// Пример генерации CSRF-токена на клиенте
function generateCSRFToken() {
    return Array.from(crypto.getRandomValues(new Uint8Array(32)))
        .map(b => b.toString(16).padStart(2, '0'))
        .join('');
}

// Пример отправки запроса с CSRF-токеном
async function makeSecureRequest(url, data) {
    const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
    
    const response = await fetch(url, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRF-Token': csrfToken
        },
        body: JSON.stringify(data),
        credentials: 'include' // Включает куки в запрос
    });
    
    return response;
}
```

### 2. Double Submit Cookie

Метод, при котором токен отправляется как в куки, так и в теле/заголовке запроса:

```javascript
// Проверка CSRF-токена на клиенте
function validateCSRFToken() {
    const cookieToken = getCookie('csrf-token');
    const headerToken = document.querySelector('meta[name="csrf-token"]').content;
    
    return cookieToken === headerToken;
}

function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
}
```

### 3. SameSite атрибут куки

Установка атрибута SameSite для куки предотвращает их отправку в межсайтовых запросах:

```javascript
// Установка куки с SameSite атрибутом
document.cookie = "sessionid=abc123; SameSite=Strict; Secure";
```

### 4. Проверка Referer заголовка

Хотя этот метод менее надежен, он может служить дополнительным уровнем защиты:

```javascript
// Проверка источника запроса
function isValidReferer(referer, origin) {
    if (!referer) return false;
    
    try {
        const refererUrl = new URL(referer);
        return refererUrl.origin === origin;
    } catch {
        return false;
    }
}
```

## Современные подходы к защите

### 1. Samesite Lax по умолчанию

В 2025 году большинство браузеров используют SameSite=Lax как значение по умолчанию, что значительно снижает риск CSRF-атак:

```javascript
// Современные браузеры устанавливают SameSite=Lax по умолчанию
// Но рекомендуется явно указывать для совместимости
document.cookie = "sessionid=abc123; SameSite=Lax; Secure";
```

### 2. Double-Check с помощью Origin заголовка

Дополнительная проверка заголовка Origin:

```javascript
// Проверка Origin заголовка
function validateOrigin(requestOrigin, expectedOrigin) {
    return requestOrigin && requestOrigin === expectedOrigin;
}
```

## Реализация защиты в JavaScript-приложениях

### Пример интеграции с формами

```javascript
// Автоматическое добавление CSRF-токена к формам
document.addEventListener('DOMContentLoaded', function() {
    const forms = document.querySelectorAll('form[method="post"]');
    
    forms.forEach(form => {
        const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
        const hiddenInput = document.createElement('input');
        hiddenInput.type = 'hidden';
        hiddenInput.name = '_csrf';
        hiddenInput.value = csrfToken;
        form.appendChild(hiddenInput);
    });
});
```

### Пример интеграции с fetch API

```javascript
// Обертка для безопасных запросов
class SecureApiClient {
    constructor(csrfToken) {
        this.csrfToken = csrfToken;
    }
    
    async request(url, options = {}) {
        const defaultHeaders = {
            'X-CSRF-Token': this.csrfToken,
            'Content-Type': 'application/json'
        };
        
        const config = {
            ...options,
            headers: {
                ...defaultHeaders,
                ...options.headers
            },
            credentials: 'include'
        };
        
        const response = await fetch(url, config);
        
        if (response.status === 403) {
            throw new Error('CSRF проверка не пройдена');
        }
        
        return response;
    }
}

// Использование
const apiClient = new SecureApiClient(document.querySelector('meta[name="csrf-token"]').content);
```

## CSRF в российском контексте 2025

В 2025 году в России особое внимание уделяется защите финансовых и государственных веб-приложений от CSRF-атак. Федеральная служба по техническому и экспортному контролю (ФСТЭК) рекомендует использовать многоуровневую защиту, включающую как токены, так и атрибуты куки.

Согласно отчетам российских CERT-организаций, CSRF-атаки составляют около 15% всех веб-уязвимостей, выявленных в отечественных информационных системах. Особенно актуальна защита для банковских приложений и электронного правительства.

## Лучшие практики

1. **Используйте CSRF-токены для всех изменяющих данных запросов** - POST, PUT, DELETE, PATCH
2. **Установите атрибут SameSite для сессионных куки** - предпочтительно Strict для чувствительных операций
3. **Регулярно обновляйте токены** - особенно после аутентификации или при длительных сессиях
4. **Используйте HTTPS** - предотвращает перехват токенов через MITM-атаки
5. **Проверяйте заголовки Origin и Referer** - как дополнительный уровень защиты
6. **Тестируйте приложение на CSRF-уязвимости** - с помощью инструментов автоматического тестирования

## Заключение

CSRF-атаки остаются актуальной угрозой в 2025 году, особенно для приложений, работающих с чувствительными данными. Эффективная защита требует комплексного подхода, включающего несколько методов защиты одновременно.

См. также: [[XSS-защита]], [[CSP]], [[Валидация-данных]], [[Защита-от-инъекций]]

> [!warning]
> Не полагайтесь только на один метод защиты CSRF. Комбинируйте несколько подходов для максимальной безопасности.