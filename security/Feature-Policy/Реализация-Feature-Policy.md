---
aliases: ["Реализация Feature Policy", "Implementing Feature Policy"]
tags: [security, feature-policy, web-security, implementation, browser-security]
---

# Реализация Feature Policy

Реализация Feature Policy (ранее Permission Policy) - это процесс настройки и внедрения политики контроля доступа к браузерным функциям в веб-приложениях. Правильная реализация позволяет ограничить использование потенциально опасных API и уменьшить атакующую поверхность приложения.

## Введение в реализацию Feature Policy

Feature Policy - это механизм, позволяющий веб-разработчикам контролировать, какие браузерные функции доступны для веб-страницы и встроенных фреймов. Он заменил более ранний механизм "Feature Policy" и предоставляет более гибкий способ управления доступом к функциям.

Реализация включает в себя:
- Определение требований безопасности
- Настройку HTTP-заголовков
- Настройку атрибутов для iframe
- Тестирование и мониторинг политик

## Подготовка к реализации

### Оценка текущего состояния

Перед внедрением Feature Policy необходимо:

1. Определить, какие функции используются в приложении
2. Оценить необходимость каждой функции
3. Идентифицировать потенциальные риски безопасности
4. Составить список доверенных источников

```bash
# Пример анализа использования функций в коде
grep -r "getUserMedia\|geolocation\|webkitRTCPeerConnection" src/
```

### Определение требований безопасности

Создайте матрицу требований для каждой функции:

| Функция | Необходима? | Уровень риска | Доверенные источники |
|---------|-------------|---------------|----------------------|
| camera | Да | Высокий | self, trusted-camera.com |
| geolocation | Да | Средний | self |
| microphone | Нет | Высокий | none |

## Способы реализации

### 1. HTTP-заголовок Feature-Policy

Наиболее распространенный способ - использование HTTP-заголовка:

```http
Feature-Policy: 
  camera 'self' https://trusted-camera.com;
  microphone 'self';
  geolocation 'self' https://trusted-location.com;
  fullscreen 'self';
  payment 'self' https://trusted-payment.com
```

#### Пример настройки в различных серверах

**Apache (.htaccess):**
```apache
Header always set Feature-Policy "camera 'self'; microphone 'none'; geolocation 'self'"
```

**Nginx:**
```nginx
add_header Feature-Policy "camera 'self'; microphone 'none'; geolocation 'self'" always;
```

**Node.js/Express:**
```javascript
app.use((req, res, next) => {
  res.setHeader('Feature-Policy', 
    "camera 'self'; microphone 'none'; geolocation 'self'; fullscreen 'self'"
  );
  next();
});
```

### 2. HTML-тег meta

Для случаев, когда невозможно настроить HTTP-заголовок:

```html
<meta http-equiv="Feature-Policy" 
      content="camera 'self'; microphone 'none'; geolocation 'self'">
```

### 3. Атрибут allow для iframe

Для контроля функций во встроенных фреймах:

```html
<iframe 
  src="https://third-party.com" 
  allow="geolocation 'self'; camera 'none'; microphone 'none'">
</iframe>
```

## Практические примеры реализации

### Пример 1: Безопасное веб-приложение

```http
Feature-Policy: 
  accelerometer 'none';
  ambient-light-sensor 'none';
  autoplay 'self';
  camera 'self' https://trusted-camera.com;
  encrypted-media 'self';
  fullscreen 'self';
  geolocation 'self';
  gyroscope 'none';
  magnetometer 'none';
  microphone 'self';
  midi 'none';
  payment 'self' https://trusted-payment.com;
  usb 'none'
```

### Пример 2: Приложение с встроенными фреймами

```html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="Feature-Policy" 
        content="camera 'none'; microphone 'none'; geolocation 'none'">
</head>
<body>
  <iframe 
    src="https://map-service.com" 
    allow="geolocation 'self' https://map-service.com; fullscreen 'self'">
  </iframe>
  
  <iframe 
    src="https://payment-gateway.com" 
    allow="payment 'self' https://payment-gateway.com">
  </iframe>
</body>
</html>
```

### Пример 3: API для динамической настройки политик

```javascript
// Серверный API для генерации политик
function generateFeaturePolicy(trustedDomains = {}) {
  const policies = {
    'camera': `'self' ${trustedDomains.camera || ''}`.trim(),
    'microphone': `'self' ${trustedDomains.microphone || ''}`.trim(),
    'geolocation': `'self' ${trustedDomains.geolocation || ''}`.trim(),
    'payment': `'self' ${trustedDomains.payment || ''}`.trim(),
    'fullscreen': `'self'`,
    'accelerometer': `'none'`,
    'gyroscope': `'none'`
  };

  return Object.entries(policies)
    .filter(([_, value]) => value)
    .map(([feature, value]) => `${feature} ${value}`)
    .join('; ');
}

// Использование
const policy = generateFeaturePolicy({
  camera: 'https://trusted-camera.com',
  payment: 'https://trusted-payment.com'
});

res.setHeader('Feature-Policy', policy);
```

## Распространенные политики

### Политика для публичного сайта

```http
Feature-Policy: 
  camera 'none';
  microphone 'none';
  geolocation 'none';
  accelerometer 'none';
  gyroscope 'none';
  magnetometer 'none';
  usb 'none';
  payment 'self' https://trusted-payment.com;
  fullscreen 'self';
  sync-xhr 'none'
```

### Политика для внутреннего приложения

```http
Feature-Policy: 
  camera 'self';
  microphone 'self';
  geolocation 'self';
  accelerometer 'self';
  gyroscope 'self';
  payment 'self';
  fullscreen 'self';
  sync-xhr 'self'
```

### Политика для лендинга

```http
Feature-Policy: 
  camera 'none';
  microphone 'none';
  geolocation 'none';
  payment 'self' https://payment-provider.com;
  fullscreen 'self'
```

## Лучшие практики реализации

### 1. Использование принципа наименьших привилегий

Предоставляйте только минимально необходимые права:

```http
# НЕПРАВИЛЬНО: слишком широкие разрешения
Feature-Policy: camera *; microphone *

# ПРАВИЛЬНО: минимальные необходимые разрешения
Feature-Policy: camera 'self' https://trusted.com; microphone 'self'
```

### 2. Сегментация политик по разделам приложения

Разные разделы приложения могут иметь разные требования к политикам:

```javascript
// Пример динамической настройки политик в зависимости от маршрута
app.use((req, res, next) => {
  let policy;
  
  if (req.path.startsWith('/admin')) {
    policy = "camera 'none'; microphone 'none'; geolocation 'none'";
  } else if (req.path.startsWith('/video-call')) {
    policy = "camera 'self'; microphone 'self'; fullscreen 'self'";
  } else {
    policy = "camera 'none'; microphone 'none'; geolocation 'self'";
  }
  
  res.setHeader('Feature-Policy', policy);
  next();
});
```

### 3. Использование HTTPS

Feature Policy должна использоваться только на HTTPS-сайтах:

```javascript
// Проверка протокола перед установкой политик
if (window.location.protocol === 'https:') {
  // Установка политик
} else {
  console.warn('Feature Policy должна использоваться только по HTTPS');
}
```

### 4. Постепенное внедрение

Начинайте с критически важных функций и постепенно расширяйте политики:

```http
# Этап 1: Запрет самых чувствительных функций
Feature-Policy: camera 'none'; microphone 'none'

# Этап 2: Добавление остальных ограничений
Feature-Policy: 
  camera 'none'; 
  microphone 'none';
  geolocation 'self';
  accelerometer 'none'

# Этап 3: Полная политика
Feature-Policy: 
  camera 'none'; 
  microphone 'none';
  geolocation 'self';
  accelerometer 'none';
  gyroscope 'none';
  magnetometer 'none';
  usb 'none'
```

## Тестирование и отладка

### Проверка HTTP-заголовков

Используйте инструменты разработчика для проверки заголовков:

```bash
# Проверка через curl
curl -I https://yoursite.com

# Или через браузерные инструменты
# Network -> Headers -> Response Headers
```

### Проверка доступа к функциям

Создайте тестовую страницу для проверки политик:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Тест Feature Policy</title>
</head>
<body>
  <button id="testCamera">Тест камеры</button>
  <button id="testGeolocation">Тест геолокации</button>
  
  <script>
    document.getElementById('testCamera').addEventListener('click', async () => {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        console.log('Доступ к камере разрешен');
        stream.getTracks().forEach(track => track.stop());
      } catch (error) {
        console.log('Доступ к камере запрещен:', error.message);
      }
    });
    
    document.getElementById('testGeolocation').addEventListener('click', () => {
      navigator.geolocation.getCurrentPosition(
        position => console.log('Геолокация разрешена:', position),
        error => console.log('Геолокация запрещена:', error.message)
      );
    });
  </script>
</body>
</html>
```

### Мониторинг ошибок

Настройте логирование ошибок, связанных с политиками:

```javascript
// Логирование ошибок политик
window.addEventListener('error', (event) => {
  if (event.message.includes('Feature Policy')) {
    console.error('Ошибка Feature Policy:', event.error);
    // Отправка в систему мониторинга
    analytics.track('feature_policy_error', {
      message: event.message,
      url: window.location.href
    });
  }
});
```

## Совместимость и резервные механизмы

### Поддержка браузерами

Feature Policy поддерживается современными браузерами:
- Chrome 60+
- Firefox 69+
- Safari 14+
- Edge 79+

### Резервные механизмы для старых браузеров

```html
<!-- Резервный механизм для старых браузеров -->
<script>
if (!window.FeaturePolicy) {
  // Использование альтернативных методов для старых браузеров
  console.warn('Feature Policy не поддерживается в этом браузере');
  // Дополнительные проверки безопасности
}
</script>
```

### Проверка поддержки конкретных функций

```javascript
// Проверка поддержки конкретных функций
function checkFeatureSupport() {
  const features = {
    camera: 'navigator.mediaDevices' in window,
    geolocation: 'geolocation' in navigator,
    microphone: 'navigator.mediaDevices' in window
  };
  
  return features;
}
```

## Интеграция с другими системами безопасности

### Совместимость с CSP

Feature Policy должна быть совместима с Content Security Policy:

```http
Content-Security-Policy: default-src 'self'; media-src 'self' blob:; 
Feature-Policy: camera 'self'; microphone 'self'
```

### Совместимость с CORS

Убедитесь, что политики CORS совместимы с Feature Policy:

```http
Access-Control-Allow-Origin: https://trusted.com
Feature-Policy: camera 'self' https://trusted.com
```

## Мониторинг и аудит

### Автоматический аудит политик

Создайте скрипт для регулярной проверки политик:

```javascript
// Скрипт аудита политик
function auditFeaturePolicy() {
  const requiredFeatures = ['camera', 'microphone', 'geolocation'];
  const policyHeader = document.querySelector('meta[http-equiv="Feature-Policy"]');
  
  if (!policyHeader) {
    console.warn('Feature Policy не установлена');
    return false;
  }
  
  const policy = policyHeader.getAttribute('content');
  const policyFeatures = policy.split(';').map(p => p.trim().split(' ')[0]);
  
  const missing = requiredFeatures.filter(f => !policyFeatures.includes(f));
  if (missing.length > 0) {
    console.warn('Отсутствуют политики для функций:', missing);
  }
  
  return true;
}
```

### Отчеты о нарушениях

Настройте отчеты о нарушениях политик:

```javascript
// Обработка нарушений политик
navigator.permissions.query({name: 'camera'}).then(result => {
  result.onchange = () => {
    if (result.state === 'prompt') {
      // Возможное нарушение политики
      console.warn('Запрошено разрешение, несмотря на политику');
    }
  };
});
```

## Расширенные сценарии

### Динамическая настройка политик

```javascript
// Динамическая настройка политик на основе пользовательских данных
function setDynamicPolicy(userRole, trustedDomains) {
  const basePolicies = {
    'fullscreen': `'self'`,
    'sync-xhr': `'self'`
  };
  
  // Политики в зависимости от роли пользователя
  if (userRole === 'admin') {
    basePolicies['camera'] = `'self' ${trustedDomains.camera || ''}`.trim();
    basePolicies['microphone'] = `'self' ${trustedDomains.microphone || ''}`.trim();
  } else {
    basePolicies['camera'] = `'none'`;
    basePolicies['microphone'] = `'none'`;
  }
  
  return Object.entries(basePolicies)
    .map(([feature, value]) => `${feature} ${value}`)
    .join('; ');
}
```

### Политики для микросервисов

```javascript
// Пример политик для микросервисной архитектуры
const servicePolicies = {
  'auth-service': "geolocation 'none'; camera 'none'; microphone 'none'",
  'video-service': "camera 'self'; microphone 'self'; fullscreen 'self'",
  'payment-service': "payment 'self' https://payment-gateway.com"
};

function getPolicyForService(serviceName) {
  return servicePolicies[serviceName] || servicePolicies['default'];
}
```

## Заключение

Реализация Feature Policy - важный шаг в обеспечении безопасности веб-приложений. Правильная настройка политик позволяет значительно уменьшить риски, связанные с использованием мощных браузерных функций. Важно подходить к реализации системно, начиная с анализа требований и заканчивая тестированием и мониторингом.

Для успешной реализации рекомендуется:
- Провести тщательный анализ используемых функций
- Применить принцип наименьших привилегий
- Проверить совместимость с различными браузерами
- Настроить мониторинг и аудит политик

Для дальнейшего изучения рекомендуется ознакомиться с [[Обзор-Feature-Policy]], [[Контроль-функций]] и [[Безопасность-браузерных-функций]].

## См. также

- [[Обзор-Feature-Policy]]
- [[Контроль-функций]]
- [[Безопасность-браузерных-функций]]
- [[Content Security Policy]]
- [[Permissions API]]

#security #feature-policy #implementation #web-security #browser-security