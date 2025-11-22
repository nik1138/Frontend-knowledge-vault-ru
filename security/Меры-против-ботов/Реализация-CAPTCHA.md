---
aliases: ["Реализация CAPTCHA", "CAPTCHA Implementation", "Captcha Systems"]
tags: [security, bot-protection, captcha, authentication]
created: 2024-11-18
updated: 2024-11-18
type: security
---

# Реализация-CAPTCHA

## Введение

CAPTCHA (Completely Automated Public Turing test to tell Computers and Humans Apart) - это метод проверки, что пользователь является человеком, а не автоматизированным ботом. Система CAPTCHA предоставляет задачи, которые легко решаются людьми, но сложно автоматизируются ботами.

## Типы CAPTCHA

### 1. Текстовая CAPTCHA

Классическая CAPTCHA, где пользователю нужно ввести искаженный текст:

```html
<div class="captcha-container">
  <div class="captcha-image">
    <img src="/api/captcha/image" alt="CAPTCHA" id="captcha-image">
    <button onclick="refreshCaptcha()">🔄</button>
  </div>
  <input type="text" id="captcha-input" placeholder="Введите текст с изображения">
  <button onclick="verifyCaptcha()">Проверить</button>
</div>

<script>
// Клиентская реализация работы с CAPTCHA
class CaptchaHandler {
  constructor() {
    this.captchaId = null;
    this.maxAttempts = 3;
    this.attempts = 0;
  }

  async loadCaptcha() {
    try {
      const response = await fetch('/api/captcha/generate');
      const data = await response.json();
      
      this.captchaId = data.id;
      document.getElementById('captcha-image').src = data.image;
    } catch (error) {
      console.error('Ошибка загрузки CAPTCHA:', error);
    }
  }

  async verifyCaptcha(input) {
    if (this.attempts >= this.maxAttempts) {
      alert('Превышено количество попыток. Пожалуйста, обновите страницу.');
      return false;
    }

    try {
      const response = await fetch('/api/captcha/verify', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          captchaId: this.captchaId,
          userInput: input
        })
      });

      const result = await response.json();
      
      if (result.success) {
        this.attempts = 0; // Сбросить счетчик при успешной проверке
        return true;
      } else {
        this.attempts++;
        this.loadCaptcha(); // Обновить CAPTCHA при неправильном ответе
        return false;
      }
    } catch (error) {
      console.error('Ошибка проверки CAPTCHA:', error);
      return false;
    }
  }
}

const captchaHandler = new CaptchaHandler();
captchaHandler.loadCaptcha();

async function verifyCaptcha() {
  const input = document.getElementById('captcha-input').value;
  const isValid = await captchaHandler.verifyCaptcha(input);
  
  if (isValid) {
    alert('CAPTCHA пройдена успешно!');
    // Продолжить выполнение операции
  } else {
    alert('Неправильный ответ. Попробуйте еще раз.');
  }
}

function refreshCaptcha() {
  captchaHandler.loadCaptcha();
}
</script>
```

### 2. Серверная реализация текстовой CAPTCHA

```javascript
const crypto = require('crypto');
const svgCaptcha = require('svg-captcha');
const redis = require('redis');

class CaptchaService {
  constructor(redisClient) {
    this.redis = redisClient;
    this.captchaExpiry = 300; // 5 минут
  }

  async generateCaptcha() {
    // Генерация случайного текста
    const captcha = svgCaptcha.create({
      size: 6, // количество символов
      ignoreChars: '0o1i', // исключить похожие символы
      noise: 2, // количество шумовых линий
      color: true,
      background: '#f0f0f0'
    });

    // Генерация уникального ID для CAPTCHA
    const captchaId = crypto.randomBytes(16).toString('hex');

    // Сохранение правильного ответа в Redis с TTL
    await this.redis.setex(
      `captcha:${captchaId}`, 
      this.captchaExpiry, 
      captcha.text.toLowerCase()
    );

    return {
      id: captchaId,
      image: `data:image/svg+xml;base64,${Buffer.from(captcha.data).toString('base64')}`
    };
  }

  async verifyCaptcha(captchaId, userInput) {
    if (!captchaId || !userInput) {
      return { success: false, message: 'Недостаточно данных для проверки' };
    }

    // Получение правильного ответа из Redis
    const correctAnswer = await this.redis.get(`captcha:${captchaId}`);
    
    if (!correctAnswer) {
      return { success: false, message: 'CAPTCHA устарела или не существует' };
    }

    // Проверка ответа (без учета регистра)
    const isValid = correctAnswer.toLowerCase() === userInput.toLowerCase();

    if (isValid) {
      // Удалить CAPTCHA из Redis после успешной проверки
      await this.redis.del(`captcha:${captchaId}`);
      return { success: true, message: 'CAPTCHA пройдена успешно' };
    } else {
      return { success: false, message: 'Неправильный ответ' };
    }
  }

  // Метод для очистки устаревших CAPTCHA (для обслуживания)
  async cleanupExpiredCaptcha() {
    // В Redis автоматически удаляются ключи по TTL, 
    // но можно реализовать дополнительную очистку при необходимости
    console.log('Очистка устаревших CAPTCHA...');
  }
}
```

### 3. reCAPTCHA v3

Современный подход, не требующий пользовательского взаимодействия:

```html
<!DOCTYPE html>
<html>
<head>
  <title>reCAPTCHA v3 Example</title>
  <script src="https://www.google.com/recaptcha/api.js?render=YOUR_SITE_KEY"></script>
</head>
<body>
  <form id="myForm">
    <input type="email" name="email" placeholder="Email" required>
    <input type="password" name="password" placeholder="Password" required>
    <button type="submit">Войти</button>
  </form>

  <script>
  // Настройка reCAPTCHA v3
  grecaptcha.ready(function() {
    document.getElementById('myForm').addEventListener('submit', function(event) {
      event.preventDefault();
      
      // Получение токена reCAPTCHA
      grecaptcha.execute('YOUR_SITE_KEY', {action: 'submit'})
        .then(function(token) {
          // Отправка формы с токеном
          submitForm(token);
        });
    });
  });

  async function submitForm(recaptchaToken) {
    const formData = new FormData(document.getElementById('myForm'));
    formData.append('recaptcha_token', recaptchaToken);

    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: formData
      });

      const result = await response.json();
      
      if (result.success) {
        // Успешная аутентификация
        window.location.href = '/dashboard';
      } else {
        alert('Ошибка входа: ' + result.message);
      }
    } catch (error) {
      console.error('Ошибка отправки формы:', error);
    }
  }
  </script>
  </body>
</html>
```

### 4. Серверная проверка reCAPTCHA

```javascript
const axios = require('axios');

class ReCaptchaService {
  constructor(secretKey) {
    this.secretKey = secretKey;
    this.verifyUrl = 'https://www.google.com/recaptcha/api/siteverify';
  }

  async verify(token, remoteip = null) {
    const data = {
      secret: this.secretKey,
      response: token,
      remoteip: remoteip
    };

    try {
      const response = await axios.post(this.verifyUrl, new URLSearchParams(data), {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      });

      const result = response.data;

      if (result.success) {
        // reCAPTCHA пройдена успешно
        return {
          success: true,
          score: result.score || 1, // для v3
          action: result.action,
          hostname: result.hostname
        };
      } else {
        // reCAPTCHA не пройдена
        return {
          success: false,
          message: result['error-codes']?.join(', ') || 'Verification failed'
        };
      }
    } catch (error) {
      console.error('Ошибка проверки reCAPTCHA:', error);
      return {
        success: false,
        message: 'Network error during verification'
      };
    }
  }

  // Метод для определения порога оценки
  isScoreAcceptable(score, threshold = 0.5) {
    return score >= threshold;
  }
}
```

## Альтернативные методы проверки

### 1. Invisible CAPTCHA

```html
<div class="g-recaptcha" data-sitekey="your_site_key" data-callback="onSubmit" data-size="invisible"></div>
<button id="submitBtn" onclick="onSubmit()">Отправить</button>

<script>
function onSubmit() {
  // Сделать форму невидимой и активировать invisible reCAPTCHA
  grecaptcha.execute();
}

function onSubmit(token) {
  // Форма будет отправлена автоматически после успешной проверки
  document.getElementById('myForm').submit();
}
</script>
```

### 2. Honeypot CAPTCHA

```html
<form id="contactForm">
  <!-- Видимые поля формы -->
  <input type="text" name="name" placeholder="Имя" required>
  <input type="email" name="email" placeholder="Email" required>
  <textarea name="message" placeholder="Сообщение" required></textarea>
  
  <!-- Скрытая ловушка для ботов -->
  <div style="display:none;">
    <label for="honeypot">Leave this field empty</label>
    <input type="text" name="honeypot" id="honeypot" value="">
  </div>
  
  <button type="submit">Отправить</button>
</form>

<script>
document.getElementById('contactForm').addEventListener('submit', function(e) {
  const honeypot = document.getElementById('honeypot');
  if (honeypot.value !== '') {
    e.preventDefault();
    console.log('Обнаружен бот по заполнению honeypot поля');
    // Логировать подозрительную активность
    return false;
  }
});
</script>
```

## Безопасная реализация

### 1. Защита от атак перебора

```javascript
class SecureCaptchaService {
  constructor(redisClient) {
    this.redis = redisClient;
    this.maxAttemptsPerIP = 5;
    this.blockTime = 300; // 5 минут
  }

  async validateCaptchaAttempts(ipAddress) {
    const key = `captcha_attempts:${ipAddress}`;
    const attempts = parseInt(await this.redis.get(key)) || 0;

    if (attempts >= this.maxAttemptsPerIP) {
      // IP заблокирован
      const ttl = await this.redis.ttl(key);
      throw new Error(`Слишком много попыток. Повторите через ${ttl} секунд`);
    }

    // Увеличить счетчик попыток
    await this.redis.incr(key);
    await this.redis.expire(key, this.blockTime);
  }

  async resetCaptchaAttempts(ipAddress) {
    const key = `captcha_attempts:${ipAddress}`;
    await this.redis.del(key);
  }
}
```

### 2. Проверка на стороне сервера

```javascript
// Middleware для проверки CAPTCHA
const captchaMiddleware = async (req, res, next) => {
  const { captchaId, captchaInput } = req.body;
  
  if (!captchaId || !captchaInput) {
    return res.status(400).json({
      error: 'Требуется ID и ввод CAPTCHA'
    });
  }

  try {
    const captchaService = new CaptchaService(redisClient);
    const result = await captchaService.verifyCaptcha(captchaId, captchaInput);

    if (!result.success) {
      return res.status(400).json({
        error: result.message
      });
    }

    // CAPTCHA пройдена, продолжить обработку запроса
    next();
  } catch (error) {
    console.error('Ошибка проверки CAPTCHA:', error);
    res.status(500).json({
      error: 'Внутренняя ошибка сервера'
    });
  }
};
```

## Лучшие практики

1. **Использование нескольких методов** - комбинирование различных подходов к защите
2. **Настройка порогов** - определение разумного баланса между безопасностью и удобством
3. **Обновление алгоритмов** - регулярное обновление методов для противодействия новым ботам
4. **Мониторинг эффективности** - отслеживание процента успешных прохождений CAPTCHA

## Связанные темы

- [[Обнаружение-ботов]]
- [[Ограничение-скорости]]
- [[Анализ-поведения]]

## Внешние ресурсы

- [Google reCAPTCHA Documentation](https://developers.google.com/recaptcha)
- [OWASP CAPTCHA Security Guidelines](https://owasp.org/www-community/controls/CAPTCHA)

> [!warning]
> Не храните ответы CAPTCHA в открытом виде в базе данных. Используйте безопасное хранение и короткие сроки действия.

> [!tip]
> Для повышения безопасности используйте комбинацию CAPTCHA с другими методами защиты, такими как [[Ограничение-скорости]] и [[Обнаружение-ботов]].