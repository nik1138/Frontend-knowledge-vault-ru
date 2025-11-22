---
aliases: [Input Sanitization, Input Cleaning, Data Processing]
tags: [regexp, input, sanitization, frontend]
---

# Обработка и очистка пользовательского ввода

## Обзор

Обработка и очистка пользовательского ввода - важная часть безопасности и надежности веб-приложений. В этой статье рассмотрим различные подходы к очистке пользовательского ввода с помощью регулярных выражений.

## Очистка HTML-тегов из пользовательского ввода

```javascript
// Функция для удаления HTML-тегов из пользовательского ввода
function sanitizeHtmlTags(input) {
  // Удаляем все HTML-теги
  return input.replace(/<[^>]*>/g, '');
}

// Пример использования
const userInput = '<p>Привет, <script>alert("xss")</script> мир!</p>';
const sanitized = sanitizeHtmlTags(userInput);
console.log(sanitized); // Привет,  мир!
```

## Экранирование специальных символов

```javascript
// Функция для экранирования специальных регулярных выражений
function escapeRegExp(string) {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Пример использования
const userInput = 'Какой-то текст с . и * и ?';
const escaped = escapeRegExp(userInput);
console.log(escaped); // Какой-то текст с \. и \* и \?
```

## Удаление потенциально опасных JavaScript-сценариев

```javascript
// Паттерн для обнаружения и удаления JavaScript-сценариев
function removeJavaScript(input) {
  // Удаляем теги script
  let result = input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
  
  // Удаляем JavaScript-события
  result = result.replace(/\s+on\w+\s*=\s*["'][^"']*["']/gi, '');
  
  // Удаляем javascript: ссылки
  result = result.replace(/javascript:/gi, '');
  
  return result;
}

// Пример использования
const dangerousInput = '<img src="x" onerror="alert(1)"> <a href="javascript:alert(2)">Ссылка</a>';
const safeInput = removeJavaScript(dangerousInput);
console.log(safeInput); // <img src="x" > <a href="">Ссылка</a>
```

## Валидация и очистка URL

```javascript
// Паттерн для валидации и очистки URL
function sanitizeUrl(url) {
  // Проверяем, начинается ли URL с безопасной схемы
  const safeUrlPattern = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/i;
  
  if (safeUrlPattern.test(url)) {
    return url;
  }
  
  // Если URL не безопасный, возвращаем пустую строку или безопасный URL
  return '';
}

// Пример использования
console.log(sanitizeUrl('https://example.com')); // https://example.com
console.log(sanitizeUrl('javascript:alert(1)')); // (пустая строка)
console.log(sanitizeUrl('ftp://files.example.com')); // ftp://files.example.com
```

## Очистка текста от лишних пробелов и специальных символов

```javascript
// Функция для нормализации текста
function normalizeText(input) {
  // Заменяем множественные пробелы на один
  let result = input.replace(/\s+/g, ' ');
  
  // Удаляем начальные и конечные пробелы
  result = result.trim();
  
  // Удаляем или заменяем специальные символы (по необходимости)
  result = result.replace(/[^\w\sа-яёА-ЯЁ.,!?;:()\-]/g, '');
  
  return result;
}

// Пример использования
const messyText = '  Это    текст   с   множественными    пробелами    и   символами @#$%^&*()  ';
const normalized = normalizeText(messyText);
console.log(normalized); // Это текст с множественными пробелами и символами
```

## Валидация и очистка email-адресов

```javascript
// Паттерн для валидации email
const emailPattern = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;

function validateAndCleanEmail(email) {
  // Удаляем лишние пробелы
  email = email.trim();
  
  // Проверяем формат
  if (emailPattern.test(email)) {
    return { valid: true, email: email.toLowerCase() };
  }
  
  return { valid: false, email: null };
}

// Пример использования
console.log(validateAndCleanEmail('  USER@EXAMPLE.COM  ')); // { valid: true, email: 'user@example.com' }
console.log(validateAndCleanEmail('invalid-email')); // { valid: false, email: null }
```

## Очистка пользовательских имен и псевдонимов

```javascript
// Паттерн для валидации пользовательских имен
const usernamePattern = /^[a-zA-Z0-9_]{3,30}$/;

function sanitizeUsername(username) {
  // Удаляем лишние пробелы
  username = username.trim();
  
  // Проверяем формат
  if (usernamePattern.test(username)) {
    return { valid: true, username: username };
  }
  
  // Пытаемся очистить имя, если возможно
  let cleaned = username.replace(/[^a-zA-Z0-9_]/g, '');
  
  if (cleaned.length >= 3 && cleaned.length <= 30) {
    return { valid: true, username: cleaned };
  }
  
  return { valid: false, username: null };
}

// Пример использования
console.log(sanitizeUsername('valid_user123')); // { valid: true, username: 'valid_user123' }
console.log(sanitizeUsername('user@invalid')); // { valid: true, username: 'userinvalid' }
console.log(sanitizeUsername('ab')); // { valid: false, username: null }
```

## Очистка телефонных номеров

```javascript
// Паттерн для извлечения цифр из телефонного номера
function sanitizePhoneNumber(phone) {
  // Извлекаем только цифры
  const digitsOnly = phone.replace(/\D/g, '');
  
  // Проверяем длину
  if (digitsOnly.length === 11 && digitsOnly[0] === '7') {
    // Российский формат: +7 (XXX) XXX-XX-XX
    return `+7 (${digitsOnly.slice(1, 4)}) ${digitsOnly.slice(4, 7)}-${digitsOnly.slice(7, 9)}-${digitsOnly.slice(9, 11)}`;
  } else if (digitsOnly.length === 10) {
    // Американский формат: (XXX) XXX-XXXX
    return `(${digitsOnly.slice(0, 3)}) ${digitsOnly.slice(3, 6)}-${digitsOnly.slice(6, 10)}`;
  }
  
  return null; // Неверный формат
}

// Пример использования
console.log(sanitizePhoneNumber('+7 (999) 123-45-67')); // +7 (999) 123-45-67
console.log(sanitizePhoneNumber('89991234567')); // +7 (999) 123-45-67
console.log(sanitizePhoneNumber('123-456-7890')); // (123) 456-7890
```

## Санитизация пользовательских данных для SQL-запросов

```javascript
// Простая санитизация для предотвращения SQL-инъекций
function sanitizeForSql(input) {
  // Экранируем апострофы
  return input.replace(/'/g, "''");
}

// Паттерн для обнаружения потенциальных SQL-инъекций
const sqlInjectionPattern = /(\b(UNION|SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|DECLARE|CAST|CONVERT)\b|--|\/\*|\*\/|;)/gi;

function detectSqlInjection(input) {
  return sqlInjectionPattern.test(input);
}

// Пример использования
const userInput = "O'Reilly";
const sanitized = sanitizeForSql(userInput);
console.log(sanitized); // O''Reilly

console.log(detectSqlInjection("1' UNION SELECT * FROM users --")); // true
console.log(detectSqlInjection("normal input")); // false
```

## Класс для комплексной санитизации ввода

```javascript
class InputSanitizer {
  constructor() {
    this.rules = {
      // Опасные теги
      dangerousTags: /<(script|iframe|object|embed|form|input|button)[^>]*>.*?<\/\1>|<(script|iframe|object|embed|form|input|button)[^>]*\/?>/gi,
      
      // JavaScript-события
      jsEvents: /\s+on\w+\s*=\s*["'][^"']*["']/gi,
      
      // JavaScript-ссылки
      jsLinks: /javascript:/gi,
      
      // Data-URI
      dataUri: /data:/gi,
      
      // SQL-инъекции
      sqlInjection: /(\b(UNION|SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|DECLARE|CAST|CONVERT)\b|--|\/\*|\*\/|;)/gi
    };
  }
  
  sanitize(input, options = {}) {
    let result = input;
    
    // Опционально: удалить HTML-теги
    if (options.removeHtml) {
      result = result.replace(/<[^>]*>/g, '');
    } else {
      // Удалить опасные теги и атрибуты
      result = result.replace(this.rules.dangerousTags, '');
      result = result.replace(this.rules.jsEvents, '');
      result = result.replace(this.rules.jsLinks, '');
      result = result.replace(this.rules.dataUri, '');
    }
    
    // Удалить лишние пробелы
    if (options.normalizeWhitespace) {
      result = result.replace(/\s+/g, ' ').trim();
    }
    
    return result;
  }
  
  validate(input, type) {
    switch (type) {
      case 'email':
        return /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/.test(input);
      
      case 'username':
        return /^[a-zA-Z0-9_]{3,30}$/.test(input);
      
      case 'phone':
        return /^\+?[\d\s\-\(\)]{10,20}$/.test(input);
      
      case 'url':
        return /^https?:\/\/[^\s/$.?#].[^\s]*$/.test(input);
      
      default:
        return !this.rules.sqlInjection.test(input);
    }
  }
  
  containsMaliciousCode(input) {
    return this.rules.sqlInjection.test(input);
  }
}

// Пример использования
const sanitizer = new InputSanitizer();

// Санитизация HTML-ввода
const htmlInput = '<p>Текст</p><script>alert(1)</script>';
console.log(sanitizer.sanitize(htmlInput)); // <p>Текст</p>

// Валидация различных типов данных
console.log(sanitizer.validate('user@example.com', 'email')); // true
console.log(sanitizer.validate('validuser', 'username')); // true
console.log(sanitizer.containsMaliciousCode('1 UNION SELECT * FROM users')); // true
```

## Заключение

Правильная обработка и очистка пользовательского ввода - критически важная часть безопасности веб-приложений. Регулярные выражения могут быть эффективным инструментом для этой задачи, но они должны использоваться в сочетании с другими методами безопасности, такими как подготовленные выражения и строгая валидация на сервере.

## См. также

- [[Form Validation Patterns]]
- [[Frontend Security Best Practices]]
- [[Backend Input Validation]]