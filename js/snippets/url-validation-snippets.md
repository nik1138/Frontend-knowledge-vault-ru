---
aliases: ["Сниппеты для обработки и валидации URL", "URL валидация JS"]
tags: [js, snippets, url, validation, frontend]
---

# Сниппеты для обработки и валидации URL

Практические сниппеты для обработки, валидации и манипуляции URL в JavaScript. Эти сниппеты помогут вам эффективно работать с URL в ваших веб-приложениях.

## 1. Валидация URL

```javascript
// Проверка валидности URL с использованием встроенного класса URL
function isValidUrl(string) {
  try {
    new URL(string);
    return true;
  } catch (_) {
    return false;
  }
}

// Более строгая проверка URL
function isValidStrictUrl(string) {
  try {
    const url = new URL(string);
    return ['http:', 'https:'].includes(url.protocol);
  } catch (_) {
    return false;
  }
}

// Проверка URL с помощью регулярного выражения
function isValidUrlRegex(string) {
  const urlRegex = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/i;
  return urlRegex.test(string);
}

// Проверка валидности доменного имени
function isValidDomain(string) {
  const domainRegex = /^[a-zA-Z0-9][a-zA-Z0-9-]{1,61}[a-zA-Z0-9](\.[a-zA-Z]{2,})+$/;
  return domainRegex.test(string);
}

console.log(isValidUrl('https://example.com')); // true
console.log(isValidUrl('not-a-url')); // false
console.log(isValidStrictUrl('ftp://files.example.com')); // true
console.log(isValidStrictUrl('javascript:alert(1)')); // false
console.log(isValidDomain('example.com')); // true
```

## 2. Извлечение компонентов URL

```javascript
// Извлечение компонентов URL
function getUrlComponents(url) {
  try {
    const parsedUrl = new URL(url);
    return {
      protocol: parsedUrl.protocol,
      hostname: parsedUrl.hostname,
      port: parsedUrl.port,
      pathname: parsedUrl.pathname,
      search: parsedUrl.search,
      hash: parsedUrl.hash,
      origin: parsedUrl.origin,
      fullUrl: parsedUrl.href
    };
  } catch (error) {
    throw new Error('Invalid URL');
  }
}

// Извлечение параметров из строки запроса
function getUrlParams(url) {
  const urlObj = new URL(url);
  const params = {};
  
  for (const [key, value] of urlObj.searchParams) {
    // Если параметр встречается несколько раз, сохраняем как массив
    if (params[key]) {
      if (Array.isArray(params[key])) {
        params[key].push(value);
      } else {
        params[key] = [params[key], value];
      }
    } else {
      params[key] = value;
    }
  }
  
  return params;
}

// Получение значения конкретного параметра
function getUrlParam(url, paramName) {
  const urlObj = new URL(url);
  return urlObj.searchParams.get(paramName);
}

console.log(getUrlComponents('https://example.com:8080/path?query=1#section'));
// {
//   protocol: 'https:',
//   hostname: 'example.com',
//   port: '8080',
//   pathname: '/path',
//   search: '?query=1',
//   hash: '#section',
//   origin: 'https://example.com:8080',
//   fullUrl: 'https://example.com:8080/path?query=1#section'
// }

console.log(getUrlParams('https://example.com?name=Alice&age=25&tag=javascript&tag=frontend'));
// { name: 'Alice', age: '25', tag: ['javascript', 'frontend'] }

console.log(getUrlParam('https://example.com?name=Alice', 'name')); // 'Alice'
```

## 3. Манипуляции с параметрами URL

```javascript
// Добавление параметров к URL
function addUrlParams(url, params) {
  const urlObj = new URL(url);
  
  for (const [key, value] of Object.entries(params)) {
    if (Array.isArray(value)) {
      // Для массивов добавляем несколько параметров с одинаковым именем
      urlObj.searchParams.delete(key); // удаляем существующие
      value.forEach(v => urlObj.searchParams.append(key, v));
    } else {
      urlObj.searchParams.set(key, value);
    }
  }
  
  return urlObj.toString();
}

// Удаление параметров из URL
function removeUrlParams(url, paramNames) {
  const urlObj = new URL(url);
  
  if (Array.isArray(paramNames)) {
    paramNames.forEach(param => urlObj.searchParams.delete(param));
  } else {
    urlObj.searchParams.delete(paramNames);
  }
  
  return urlObj.toString();
}

// Обновление параметра в URL
function updateUrlParam(url, paramName, newValue) {
  const urlObj = new URL(url);
  urlObj.searchParams.set(paramName, newValue);
  return urlObj.toString();
}

// Получение URL без параметров
function getUrlWithoutParams(url) {
  const urlObj = new URL(url);
  urlObj.search = '';
  return urlObj.toString();
}

console.log(addUrlParams('https://example.com', { name: 'Alice', age: 25 }));
// 'https://example.com/?name=Alice&age=25'

console.log(removeUrlParams('https://example.com?name=Alice&age=25', ['name']));
// 'https://example.com/?age=25'

console.log(updateUrlParam('https://example.com?name=Alice', 'name', 'Bob'));
// 'https://example.com/?name=Bob'
```

## 4. Работа с путями URL

```javascript
// Объединение путей
function joinPaths(...paths) {
  return paths
    .join('/')
    .replace(/\/+/g, '/') // заменяем множественные слэши на один
    .replace(/^(https?:\/\/[^\/]+)\//g, '$1/') // сохраняем протокол и хост
    .replace(/\/$/, ''); // удаляем завершающий слэш
}

// Получение последнего сегмента пути
function getLastPathSegment(url) {
  const urlObj = new URL(url);
  const pathname = urlObj.pathname;
  const segments = pathname.split('/').filter(segment => segment !== '');
  return segments[segments.length - 1] || '';
}

// Получение всех сегментов пути
function getPathSegments(url) {
  const urlObj = new URL(url);
  return urlObj.pathname.split('/').filter(segment => segment !== '');
}

// Добавление слэша в начало пути, если его нет
function ensureLeadingSlash(path) {
  return path.startsWith('/') ? path : '/' + path;
}

// Удаление слэша в конце пути, если он есть
function removeTrailingSlash(path) {
  return path.endsWith('/') ? path.slice(0, -1) : path;
}

console.log(joinPaths('https://api.example.com', 'users', '123', 'posts'));
// 'https://api.example.com/users/123/posts'

console.log(getLastPathSegment('https://example.com/users/123/profile'));
// 'profile'

console.log(getPathSegments('https://example.com/api/v1/users'));
// ['api', 'v1', 'users']
```

## 5. Валидация специфичных URL

```javascript
// Проверка email URL (mailto:)
function isValidEmailUrl(url) {
  try {
    const urlObj = new URL(url);
    return urlObj.protocol === 'mailto:' && isValidEmail(urlObj.pathname);
  } catch (error) {
    return false;
  }
}

// Проверка телефонного URL (tel:)
function isValidPhoneUrl(url) {
  try {
    const urlObj = new URL(url);
    return urlObj.protocol === 'tel:' && /^[\d\-\+\(\)\s]+$/.test(urlObj.pathname);
  } catch (error) {
    return false;
  }
}

// Проверка URL изображения
function isValidImageUrl(url) {
  try {
    const urlObj = new URL(url);
    const imageExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp', '.bmp', '.svg'];
    return imageExtensions.some(ext => urlObj.pathname.toLowerCase().endsWith(ext));
  } catch (error) {
    // Если не можем распарсить как URL, проверим через регулярное выражение
    const imageRegex = /\.(jpg|jpeg|png|gif|webp|bmp|svg)(\?.*)?$/i;
    return imageRegex.test(url);
  }
}

// Проверка URL видео
function isValidVideoUrl(url) {
  try {
    const urlObj = new URL(url);
    const videoExtensions = ['.mp4', '.avi', '.mov', '.wmv', '.flv', '.webm', '.m3u8'];
    return videoExtensions.some(ext => urlObj.pathname.toLowerCase().endsWith(ext));
  } catch (error) {
    const videoRegex = /\.(mp4|avi|mov|wmv|flv|webm|m3u8)(\?.*)?$/i;
    return videoRegex.test(url);
  }
}

// Вспомогательная функция для проверки email
function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

console.log(isValidEmailUrl('mailto:test@example.com')); // true
console.log(isValidPhoneUrl('tel:+1234567890')); // true
console.log(isValidImageUrl('https://example.com/image.png')); // true
console.log(isValidVideoUrl('https://example.com/video.mp4')); // true
```

## 6. Сниппеты для работы с URL в браузере

```javascript
// Получение текущего URL
function getCurrentUrl() {
  return window.location.href;
}

// Получение параметров текущего URL
function getCurrentUrlParams() {
  const searchParams = new URLSearchParams(window.location.search);
  const params = {};
  
  for (const [key, value] of searchParams) {
    if (params[key]) {
      if (Array.isArray(params[key])) {
        params[key].push(value);
      } else {
        params[key] = [params[key], value];
      }
    } else {
      params[key] = value;
    }
  }
  
  return params;
}

// Изменение параметров текущего URL без перезагрузки страницы
function updateCurrentUrlParams(params) {
  const url = new URL(window.location);
  
  for (const [key, value] of Object.entries(params)) {
    url.searchParams.set(key, value);
  }
  
  window.history.pushState({}, '', url);
}

// Проверка, является ли URL внешним
function isExternalUrl(url) {
  try {
    const urlObj = new URL(url, window.location.origin);
    return urlObj.origin !== window.location.origin;
  } catch (error) {
    return false;
  }
}

// Получение домена из URL
function getDomain(url) {
  try {
    const urlObj = new URL(url);
    return urlObj.hostname;
  } catch (error) {
    return '';
  }
}

console.log(getCurrentUrl()); // текущий URL страницы
console.log(getCurrentUrlParams()); // параметры текущего URL
console.log(isExternalUrl('https://google.com')); // true (если текущий домен не google.com)
console.log(getDomain('https://subdomain.example.com/path')); // 'subdomain.example.com'
```

## 7. Сниппеты для очистки и нормализации URL

```javascript
// Нормализация URL (удаление лишних элементов)
function normalizeUrl(url) {
  try {
    const urlObj = new URL(url);
    
    // Удаление завершающего слэша, кроме корня
    if (urlObj.pathname !== '/' && urlObj.pathname.endsWith('/')) {
      urlObj.pathname = urlObj.pathname.slice(0, -1);
    }
    
    // Сортировка параметров для консистентности
    const searchParams = {};
    for (const [key, value] of urlObj.searchParams) {
      searchParams[key] = value;
    }
    
    urlObj.search = '';
    Object.keys(searchParams)
      .sort()
      .forEach(key => urlObj.searchParams.append(key, searchParams[key]));
    
    return urlObj.toString();
  } catch (error) {
    return url;
  }
}

// Очистка URL от фрагментов отслеживания
function cleanTrackingParams(url) {
  const trackingParams = [
    'utm_source', 'utm_medium', 'utm_campaign', 'utm_term', 'utm_content',
    'gclid', 'gclsrc', 'dclid', 'fbclid', 'ref', 'source', 'campaign'
  ];
  
  try {
    const urlObj = new URL(url);
    
    trackingParams.forEach(param => {
      urlObj.searchParams.delete(param);
    });
    
    return urlObj.toString();
  } catch (error) {
    return url;
  }
}

// Декодирование URL
function decodeUrl(url) {
  try {
    return decodeURIComponent(url);
  } catch (error) {
    return url;
  }
}

// Кодирование URL
function encodeUrl(url) {
  try {
    return encodeURIComponent(url);
  } catch (error) {
    return url;
  }
}

console.log(normalizeUrl('https://example.com/path/?b=2&a=1/'));
// 'https://example.com/path?a=1&b=2' (если путь не заканчивается на слэш)

console.log(cleanTrackingParams('https://example.com/page?utm_source=google&name=Alice'));
// 'https://example.com/page?name=Alice'
```

## 8. Сниппеты для безопасности URL

```javascript
// Проверка URL на потенциально опасные протоколы
function isSafeProtocol(url) {
  try {
    const urlObj = new URL(url);
    const safeProtocols = ['http:', 'https:', 'ftp:', 'mailto:', 'tel:'];
    return safeProtocols.includes(urlObj.protocol);
  } catch (error) {
    return false;
  }
}

// Санитизация URL (удаление потенциально опасных протоколов)
function sanitizeUrl(url) {
  const dangerousProtocols = ['javascript:', 'data:', 'vbscript:', 'file:'];
  
  for (const protocol of dangerousProtocols) {
    if (url.toLowerCase().startsWith(protocol)) {
      return '#';
    }
  }
  
  return url;
}

// Проверка URL на XSS
function hasXssAttempt(url) {
  const xssPatterns = [
    /javascript:/i,
    /vbscript:/i,
    /data:/i,
    /on\w+\s*=/i, // обработчики событий
    /<script/i,
    /javascript\s*:/i
  ];
  
  return xssPatterns.some(pattern => pattern.test(url));
}

// Проверка, что URL принадлежит доверенному домену
function isTrustedDomain(url, trustedDomains) {
  try {
    const urlObj = new URL(url);
    return trustedDomains.includes(urlObj.hostname) || 
           trustedDomains.some(domain => urlObj.hostname.endsWith('.' + domain));
  } catch (error) {
    return false;
  }
}

console.log(isSafeProtocol('javascript:alert(1)')); // false
console.log(sanitizeUrl('javascript:alert(1)')); // '#'
console.log(hasXssAttempt('javascript:alert(1)')); // true
console.log(isTrustedDomain('https://api.example.com', ['example.com'])); // true
```

Эти сниппеты помогут вам эффективно обрабатывать и валидировать URL в ваших JavaScript приложениях, обеспечивая безопасность и корректную работу с веб-адресами.

См. также:
- [[js/snippets/date-time-snippets]] - Сниппеты для работы с датами и временем
- [[js/snippets/number-formatting-snippets]] - Сниппеты для форматирования чисел и валют
- [[js/snippets/color-snippets]] - Сниппеты для работы с цветами