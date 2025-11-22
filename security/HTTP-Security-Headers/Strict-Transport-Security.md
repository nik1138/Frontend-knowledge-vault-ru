---
aliases: ["HSTS", "Strict Transport Security", "Защита транспорта", "HTTPS-безопасность"]
tags: [security, hsts, https, transport-security, web-security, http-headers]
---

# Strict-Transport-Security

## Введение

Strict-Transport-Security (HSTS) — это HTTP-заголовок безопасности, который указывает браузеру использовать только HTTPS-соединения для взаимодействия с веб-сайтом. Этот заголовок защищает от атак типа "downgrade" и "cookie hijacking", автоматически перенаправляя HTTP-запросы на HTTPS даже до отправки запроса на сервер.

## Проблема, которую решает HSTS

### 1. Атаки типа "downgrade"

Злоумышленник может перехватить HTTP-запрос и не позволить браузеру перейти на HTTPS, оставляя соединение незашифрованным.

### 2. Атаки типа "cookie hijacking"

Если пользователь посещает HTTP-версию сайта, куки могут быть переданы в незашифрованном виде, что позволяет злоумышленнику украсть их.

### 3. Первый HTTP-запрос

Даже если сайт использует редирект с HTTP на HTTPS, первый HTTP-запрос остается уязвимым.

## Структура заголовка HSTS

```
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains; preload
```

### Параметры заголовка

#### 1. max-age

Обязательный параметр, указывающий время в секундах, на которое браузер должен запоминать политику HSTS.

```
Strict-Transport-Security: max-age=31536000  // 1 год
```

#### 2. includeSubDomains (опционально)

Указывает, что политика HSTS распространяется на все поддомены.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

#### 3. preload (опционально)

Указывает, что сайт хочет быть включенным в HSTS preload list браузеров.

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

## Реализация HSTS

### 1. В Express.js

```javascript
// Middleware для установки HSTS
app.use((req, res, next) => {
  if (req.secure || req.headers['x-forwarded-proto'] === 'https') {
    // Установка HSTS заголовка только для HTTPS-соединений
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  }
  next();
});

// Альтернативный вариант с условием
function hstsMiddleware(req, res, next) {
  // Убедиться, что соединение защищено
  if (req.protocol === 'https' || req.get('X-Forwarded-Proto') === 'https') {
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  }
  next();
}

app.use(hstsMiddleware);
```

### 2. В Apache

```
# В .htaccess или конфигурационном файле Apache
<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</IfModule>

# Альтернативно, с условием HTTPS
<If "%{HTTPS} == 'on'">
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</If>
```

### 3. В Nginx

```
# В конфигурационном файле Nginx
server {
    listen 443 ssl http2;
    # ... SSL конфигурация ...
    
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
}

# Альтернативно, в отдельном блоке
http {
    map $scheme $hsts_header {
        https "max-age=31536000; includeSubDomains; preload";
        default "";
    }
    
    server {
        listen 443 ssl;
        # ... SSL конфигурация ...
        
        add_header Strict-Transport-Security $hsts_header always;
    }
}
```

### 4. В ASP.NET

```csharp
// В web.config
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains; preload" />
    </customHeaders>
  </httpProtocol>
</system.webServer>

// Или в коде
if (Request.IsSecureConnection)
{
    Response.Headers.Add("Strict-Transport-Security", 
        "max-age=31536000; includeSubDomains; preload");
}
```

## Практические примеры настройки

### 1. Базовая настройка

```javascript
// Простая настройка HSTS
app.use((req, res, next) => {
  if (process.env.NODE_ENV === 'production' && req.secure) {
    res.setHeader('Strict-Transport-Security', 'max-age=31536000');
  }
  next();
});
```

### 2. Расширенная настройка

```javascript
// Расширенная настройка с проверками
class HSTSMiddleware {
  constructor(options = {}) {
    this.maxAge = options.maxAge || 31536000; // 1 год
    this.includeSubDomains = options.includeSubDomains !== false;
    this.preload = options.preload || false;
    this.setIfUnset = options.setIfUnset !== false;
  }
  
  generateHeader() {
    const directives = [`max-age=${this.maxAge}`];
    
    if (this.includeSubDomains) {
      directives.push('includeSubDomains');
    }
    
    if (this.preload) {
      directives.push('preload');
    }
    
    return directives.join('; ');
  }
  
  middleware() {
    return (req, res, next) => {
      // Проверка, что соединение HTTPS
      const isSecure = req.secure || 
                      req.headers['x-forwarded-proto'] === 'https' ||
                      (req.connection.encrypted || req.socket.encrypted);
      
      if (isSecure) {
        // Установка заголовка HSTS
        res.setHeader('Strict-Transport-Security', this.generateHeader());
      }
      
      next();
    };
  }
}

// Использование
const hsts = new HSTSMiddleware({
  maxAge: 31536000, // 1 год
  includeSubDomains: true,
  preload: true
});

app.use(hsts.middleware());
```

### 3. Условная настройка по окружению

```javascript
// Установка HSTS в зависимости от окружения
function conditionalHSTS(req, res, next) {
  let maxAge;
  
  switch (process.env.NODE_ENV) {
    case 'production':
      maxAge = 31536000; // 1 год
      break;
    case 'staging':
      maxAge = 604800; // 1 неделя
      break;
    default:
      maxAge = 300; // 5 минут для разработки
  }
  
  if (req.secure || req.headers['x-forwarded-proto'] === 'https') {
    const directives = [`max-age=${maxAge}`];
    
    if (process.env.NODE_ENV === 'production') {
      directives.push('includeSubDomains', 'preload');
    }
    
    res.setHeader('Strict-Transport-Security', directives.join('; '));
  }
  
  next();
}

app.use(conditionalHSTS);
```

## Значения max-age

### Рекомендуемые значения

- **31536000** (1 год) — стандартное значение для production
- **15768000** (6 месяцев) — для тестирования
- **604800** (1 неделя) — для начального внедрения

### Важность правильного выбора max-age

```javascript
// Функция для расчета max-age
function calculateMaxAge(years = 1) {
  return years * 365 * 24 * 60 * 60;
}

// Примеры
const oneYear = calculateMaxAge(1);      // 31536000
const sixMonths = calculateMaxAge(0.5);  // 15768000
const twoYears = calculateMaxAge(2);     // 63072000
```

## Включение в HSTS Preload List

### Требования для включения

1. Сервер должен поддерживать HTTPS на порту 443
2. Должен быть установлен заголовок HSTS с:
   - max-age не менее 31536000 секунд (1 год)
   - includeSubDomains директива
   - preload директива
3. Домен и все поддомены должны обслуживать HTTPS
4. Должен быть редирект с HTTP на HTTPS

### Процесс включения

```javascript
// Пример настройки для preload
app.use((req, res, next) => {
  if (req.secure) {
    // Заголовок для HSTS preload
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  } else {
    // Редирект с HTTP на HTTPS
    res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

## Проверка HSTS

### 1. Проверка с помощью curl

```bash
# Проверка HSTS заголовка
curl -I https://yourdomain.com

# Ожидаемый результат:
# Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### 2. Автоматизированная проверка

```javascript
// Инструмент для проверки HSTS
async function checkHSTS(domain) {
  try {
    const response = await fetch(`https://${domain}`, { method: 'HEAD' });
    const hstsHeader = response.headers.get('Strict-Transport-Security');
    
    if (!hstsHeader) {
      return {
        hasHSTS: false,
        error: 'HSTS header is missing'
      };
    }
    
    // Парсинг заголовка
    const directives = hstsHeader.split(';').map(d => d.trim());
    const maxAgeMatch = hstsHeader.match(/max-age=(\d+)/);
    const maxAge = maxAgeMatch ? parseInt(maxAgeMatch[1]) : null;
    
    return {
      hasHSTS: true,
      maxAge: maxAge,
      includeSubDomains: directives.includes('includeSubDomains'),
      preload: directives.includes('preload'),
      isValidForPreload: maxAge >= 31536000 && 
                        directives.includes('includeSubDomains') &&
                        directives.includes('preload')
    };
  } catch (error) {
    return {
      hasHSTS: false,
      error: error.message
    };
  }
}

// Использование
checkHSTS('example.com').then(result => {
  console.log('HSTS Check Result:', result);
});
```

### 3. Проверка в браузере

```javascript
// Проверка HSTS поддержки в браузере (клиентский код)
function checkHSTSSupport() {
  // Проверка через Image
  return new Promise((resolve) => {
    const img = new Image();
    let hstsSupported = false;
    
    img.onload = () => {
      // Если изображение загружено успешно, HSTS может быть включен
      hstsSupported = true;
    };
    
    img.onerror = () => {
      // Ошибка может указывать на HSTS
      hstsSupported = true;
    };
    
    // Попытка загрузки изображения с HTTP
    img.src = 'http://example.com/check-hsts.png';
    
    // Таймаут для определения результата
    setTimeout(() => {
      resolve(hstsSupported);
    }, 1000);
  });
}
```

## Тестирование и отладка

### 1. Тестирование HSTS в Jest

```javascript
const request = require('supertest');
const app = require('../app');

describe('HSTS Security Header', () => {
  test('should include Strict-Transport-Security header for HTTPS', async () => {
    // Имитация HTTPS-запроса
    const response = await request(app)
      .get('/api/data')
      .set('X-Forwarded-Proto', 'https')
      .expect(200);
    
    expect(response.headers).toHaveProperty('strict-transport-security');
    
    const hsts = response.headers['strict-transport-security'];
    expect(hsts).toMatch(/max-age=\d+/);
    expect(hsts).toContain('includeSubDomains');
    expect(hsts).toContain('preload');
  });
  
  test('should not include HSTS header for HTTP', async () => {
    const response = await request(app)
      .get('/api/data')
      .set('X-Forwarded-Proto', 'http')
      .expect(200);
    
    expect(response.headers).not.toHaveProperty('strict-transport-security');
  });
});
```

### 2. Инструменты для тестирования

```javascript
// Инструмент для комплексной проверки HSTS
class HSTSValidator {
  constructor(domain) {
    this.domain = domain;
  }
  
  async validate() {
    const results = {
      basic: await this.checkBasicHSTS(),
      preloadRequirements: await this.checkPreloadRequirements(),
      security: await this.checkSecurityAspects()
    };
    
    return {
      isValid: this.calculateValidity(results),
      results,
      recommendations: this.generateRecommendations(results)
    };
  }
  
  async checkBasicHSTS() {
    const response = await fetch(`https://${this.domain}`, { method: 'HEAD' });
    const hsts = response.headers.get('Strict-Transport-Security');
    
    return {
      present: !!hsts,
      value: hsts,
      maxAge: this.extractMaxAge(hsts),
      hasIncludeSubDomains: hsts?.includes('includeSubDomains'),
      hasPreload: hsts?.includes('preload')
    };
  }
  
  extractMaxAge(hstsHeader) {
    if (!hstsHeader) return null;
    const match = hstsHeader.match(/max-age=(\d+)/);
    return match ? parseInt(match[1]) : null;
  }
  
  async checkPreloadRequirements() {
    const basicCheck = await this.checkBasicHSTS();
    
    return {
      meetsMaxAgeRequirement: basicCheck.maxAge >= 31536000,
      hasIncludeSubDomains: basicCheck.hasIncludeSubDomains,
      hasPreload: basicCheck.hasPreload,
      allRequirementsMet: basicCheck.maxAge >= 31536000 &&
                         basicCheck.hasIncludeSubDomains &&
                         basicCheck.hasPreload
    };
  }
  
  calculateValidity(results) {
    return results.basic.present && 
           results.basic.maxAge >= 31536000 &&
           results.basic.hasIncludeSubDomains;
  }
  
  generateRecommendations(results) {
    const recommendations = [];
    
    if (!results.basic.present) {
      recommendations.push('Add Strict-Transport-Security header');
    }
    
    if (results.basic.maxAge < 31536000) {
      recommendations.push('Increase max-age to at least 31536000 seconds (1 year)');
    }
    
    if (!results.basic.hasIncludeSubDomains) {
      recommendations.push('Add includeSubDomains directive');
    }
    
    if (!results.basic.hasPreload && results.preloadRequirements.allRequirementsMet) {
      recommendations.push('Consider adding preload directive');
    }
    
    return recommendations;
  }
}

// Использование
const validator = new HSTSValidator('example.com');
validator.validate().then(result => {
  console.log('HSTS Validation Result:', result);
});
```

## Возможные проблемы и решения

### 1. Проблема с локальной разработкой

```javascript
// Условная установка HSTS для разных окружений
function safeHSTSMiddleware(req, res, next) {
  // Не устанавливать HSTS в локальной разработке
  if (process.env.NODE_ENV === 'development') {
    // Можно установить короткий max-age для тестирования
    if (req.secure) {
      res.setHeader('Strict-Transport-Security', 'max-age=300'); // 5 минут
    }
  } else if (process.env.NODE_ENV === 'production') {
    // Полная настройка для production
    if (req.secure) {
      res.setHeader('Strict-Transport-Security', 
        'max-age=31536000; includeSubDomains; preload');
    }
  }
  
  next();
}
```

### 2. Проблема с обратной совместимостью

```javascript
// Проверка поддержки HSTS в браузере клиента
function hstsWithFallback(req, res, next) {
  const userAgent = req.get('User-Agent');
  
  // Проверка минимальной версии браузера
  const supportsHSTS = this.checkBrowserSupport(userAgent);
  
  if (supportsHSTS && req.secure) {
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  }
  
  next();
}

// Метод для проверки поддержки
checkBrowserSupport(userAgent) {
  // Проверка версии Chrome
  const chromeMatch = userAgent.match(/Chrome\/(\d+)/);
  if (chromeMatch && parseInt(chromeMatch[1]) >= 4) return true;
  
  // Проверка версии Firefox
  const firefoxMatch = userAgent.match(/Firefox\/(\d+)/);
  if (firefoxMatch && parseInt(firefoxMatch[1]) >= 4) return true;
  
  // Проверка версии Safari
  const safariMatch = userAgent.match(/Version\/(\d+)/);
  if (safariMatch && parseInt(safariMatch[1]) >= 7) return true;
  
  // Проверка версии IE/Edge
  const edgeMatch = userAgent.match(/Edge\/(\d+)/);
  if (edgeMatch && parseInt(edgeMatch[1]) >= 12) return true;
  
  return false;
}
```

### 3. Проблема с редиректами

```javascript
// Обработка HTTP -> HTTPS редиректов
app.use((req, res, next) => {
  if (!req.secure && req.get('X-Forwarded-Proto') !== 'https') {
    // Редирект на HTTPS
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  
  if (req.secure) {
    // Установка HSTS только для HTTPS
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  }
  
  next();
});
```

## Лучшие практики

> [!tip] Лучшие практики HSTS
> 1. Всегда устанавливайте HSTS только через HTTPS
> 2. Используйте max-age не менее 1 года в production
> 3. Добавляйте includeSubDomains для полной защиты
> 4. Рассмотрите возможность preload для максимальной защиты
> 5. Тестируйте настройки перед включением preload

### Рекомендуемая конфигурация

```javascript
// Рекомендуемая HSTS конфигурация
function recommendedHSTS(req, res, next) {
  if (req.secure) {
    // Политика HSTS для production
    res.setHeader('Strict-Transport-Security', 
      'max-age=31536000; includeSubDomains; preload');
  } else {
    // Обязательный редирект на HTTPS
    res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
}

app.use(recommendedHSTS);
```

### Постепенное внедрение

```javascript
// Постепенное внедрение HSTS
class GradualHSTS {
  constructor(options = {}) {
    this.stages = {
      1: { maxAge: 300, description: 'Initial testing (5 minutes)' },      // 5 минут
      2: { maxAge: 86400, description: 'Extended testing (1 day)' },      // 1 день  
      3: { maxAge: 604800, description: 'Production ready (1 week)' },    // 1 неделя
      4: { maxAge: 2592000, description: 'Long term (1 month)' },         // 1 месяц
      5: { maxAge: 31536000, description: 'Full deployment (1 year)' }    // 1 год
    };
    
    this.currentStage = options.startingStage || 1;
    this.autoAdvance = options.autoAdvance || false;
  }
  
  middleware() {
    return (req, res, next) => {
      if (req.secure && this.currentStage in this.stages) {
        const directives = [`max-age=${this.stages[this.currentStage].maxAge}`];
        
        if (this.currentStage >= 3) {
          directives.push('includeSubDomains');
        }
        
        if (this.currentStage >= 5) {
          directives.push('preload');
        }
        
        res.setHeader('Strict-Transport-Security', directives.join('; '));
      }
      
      next();
    };
  }
  
  advanceStage() {
    if (this.currentStage < Object.keys(this.stages).length) {
      this.currentStage++;
      console.log(`HSTS stage advanced to: ${this.currentStage} (${this.stages[this.currentStage].description})`);
    }
  }
}

// Использование
const gradualHSTS = new GradualHSTS({ startingStage: 1 });
app.use(gradualHSTS.middleware());

// Автоматическое продвижение через определенное время
if (gradualHSTS.autoAdvance) {
  setInterval(() => {
    gradualHSTS.advanceStage();
  }, 7 * 24 * 60 * 60 * 1000); // каждую неделю
}
```

## Совместимость с браузерами

### Поддержка HSTS:

- Chrome: с версии 4.0
- Firefox: с версии 4.0
- Safari: с версии 7.0
- Internet Explorer: с версии 11.0
- Edge: с версии 12.0

## Безопасность и риски

### Потенциальные риски

1. **Злоупотребление HSTS**: Если домен больше не использует HTTPS, пользователи не смогут получить доступ к нему в течение срока max-age
2. **Неправильная конфигурация**: Ошибки в конфигурации могут сделать сайт недоступным

### Меры предосторожности

```javascript
// Безопасная реализация HSTS с возможностью отключения
class SafeHSTS {
  constructor(options = {}) {
    this.maxAge = options.maxAge || 31536000;
    this.includeSubDomains = options.includeSubDomains !== false;
    this.preload = options.preload || false;
    
    // Флаг для безопасного отключения
    this.enabled = true;
  }
  
  disable() {
    this.enabled = false;
  }
  
  enable() {
    this.enabled = true;
  }
  
  middleware() {
    return (req, res, next) => {
      if (this.enabled && req.secure) {
        const directives = [`max-age=${this.maxAge}`];
        
        if (this.includeSubDomains) {
          directives.push('includeSubDomains');
        }
        
        if (this.preload) {
          directives.push('preload');
        }
        
        res.setHeader('Strict-Transport-Security', directives.join('; '));
      }
      
      next();
    };
  }
}
```

## Связь с другими аспектами безопасности

Strict-Transport-Security тесно связан с:
- [[Обзор-заголовков-безопасности]] — общая концепция заголовков безопасности
- [[X-Content-Type-Options]] — дополнительные заголовки безопасности
- [[Content-Security-Policy]] — комплексная защита
- [[Распространенные-уязвимости]] — защита от атак на транспортный уровень
- [[Тестирование-безопасности]] — методы проверки HSTS

## Заключение

Strict-Transport-Security является критически важным заголовком безопасности, который обеспечивает защиту от downgrade-атак и гарантирует, что все соединения с веб-сайтом будут зашифрованы. Правильная настройка HSTS требует внимательного подхода к выбору параметров, особенно при включении в preload list. Рекомендуется начинать с короткого max-age и постепенно увеличивать его после тестирования.

## Дополнительные ресурсы

- RFC 6797 (HTTP Strict Transport Security)
- OWASP Transport Layer Protection Cheat Sheet
- HSTS Preload List Submission
- Mozilla Security Guidelines