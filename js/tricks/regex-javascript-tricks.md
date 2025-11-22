---
aliases: ["Регулярные выражения в JavaScript", "JS Regex"]
tags: [javascript, regex, frontend, tricks]
---

# Хитрости работы с регулярными выражениями в JavaScript

Регулярные выражения (regex) в JavaScript — мощный инструмент для поиска, замены и валидации строк. В этом документе собраны полезные трюки и паттерны для эффективной работы с регулярными выражениями.

## Основы регулярных выражений

Регулярные выражения в JavaScript можно создавать двумя способами:

```javascript
// Литерал регулярного выражения
const regex1 = /pattern/flags;

// Конструктор RegExp
const regex2 = new RegExp('pattern', 'flags');
```

### Флаги

- `g` - глобальный поиск (все вхождения)
- `i` - игнорирование регистра
- `m` - многострочный режим
- `s` - режим точки (точка соответствует символу новой строки)
- `u` - режим Unicode
- `y` - "липкий" режим

## Полезные паттерны и трюки

### 1. Валидация электронной почты

```javascript
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailRegex.test('user@example.com')); // true
console.log(emailRegex.test('invalid.email')); // false
```

> [!warning] Важно
> Полная валидация email через regex — сложная задача. Для реальных приложений рекомендуется использовать специализированные библиотеки.

### 2. Валидация URL

```javascript
const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;

console.log(urlRegex.test('https://www.example.com')); // true
console.log(urlRegex.test('http://subdomain.example.org/path')); // true
```

### 3. Извлечение чисел из строки

```javascript
const text = 'Цена: $29.99, скидка 15%, количество: 100 шт.';
const numbers = text.match(/\d+(\.\d+)?/g);
console.log(numbers); // ['29', '99', '15', '100']
```

### 4. Удаление лишних пробелов

```javascript
const text = '  Много    пробелов    и\tтабуляций\n\n';
const cleanText = text.replace(/\s+/g, ' ').trim();
console.log(cleanText); // 'Много пробелов и табуляций'
```

### 5. Замена с использованием групп захвата

```javascript
const text = 'Дата: 2023-11-13';
const reformatted = text.replace(/(\d{4})-(\d{2})-(\d{2})/, '$3.$2.$1');
console.log(reformatted); // 'Дата: 13.11.2023'
```

### 6. Проверка сложности пароля

```javascript
function validatePassword(password) {
  const checks = {
    length: password.length >= 8,
    uppercase: /[A-Z]/.test(password),
    lowercase: /[a-z]/.test(password),
    digit: /\d/.test(password),
    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
  };
  
  return Object.values(checks).filter(Boolean).length >= 4;
}

console.log(validatePassword('MyPass123!')); // true
console.log(validatePassword('weak')); // false
```

### 7. Поиск и замена с учетом контекста

```javascript
const text = 'JavaScript - отличный язык. JavaScript популярен.';
const replaced = text.replace(/\bJavaScript\b/g, '<em>JavaScript</em>');
console.log(replaced); // 'JavaScript - отличный язык. JavaScript популярен.'
```

### 8. Извлечение параметров из строки запроса

```javascript
function getQueryParams(url) {
  const regex = /[?&]([^=#]+)=([^&#]*)/g;
  const params = {};
  let match;
  
  while ((match = regex.exec(url)) !== null) {
    params[match[1]] = decodeURIComponent(match[2]);
  }
  
  return params;
}

const url = 'https://example.com?name=John&age=30&city=NYC';
console.log(getQueryParams(url)); // { name: 'John', age: '30', city: 'NYC' }
```

### 9. Валидация формата даты

```javascript
function validateDateFormat(dateString) {
  const dateRegex = /^\d{4}-\d{2}-\d{2}$/;
  if (!dateRegex.test(dateString)) return false;
  
  const date = new Date(dateString);
  return date instanceof Date && !isNaN(date);
}

console.log(validateDateFormat('2023-11-13')); // true
console.log(validateDateFormat('13-11-2023')); // false
```

### 10. Экранирование специальных символов

```javascript
function escapeRegExp(string) {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

const userInput = 'How much $ for a *?';
const safeRegex = new RegExp(escapeRegExp(userInput), 'g');
```

## Продвинутые трюки

### 1. Проверка симметричных тегов

```javascript
function hasBalancedTags(html) {
  const tagRegex = /<[^>]*>|[^<]+/g;
  const tags = [];
  let match;
  
  while ((match = tagRegex.exec(html)) !== null) {
    const part = match[0];
    if (part.startsWith('</')) {
      const tag = part.substring(2, part.length - 1);
      if (tags.length === 0 || tags.pop() !== tag) {
        return false;
      }
    } else if (part.startsWith('<') && !part.endsWith('/>')) {
      const tag = part.substring(1).split(' ')[0].replace('>', '');
      tags.push(tag);
    }
  }
  
  return tags.length === 0;
}

console.log(hasBalancedTags('<div><p>Hello</p></div>')); // true
console.log(hasBalancedTags('<div><p>Hello</div></p>')); // false
```

### 2. Извлечение ссылок из текста

```javascript
function extractLinks(text) {
  const linkRegex = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g;
  return text.match(linkRegex) || [];
}

const textWithLinks = 'Посетите https://example.com и http://test.org для получения информации.';
console.log(extractLinks(textWithLinks));
```

### 3. Проверка палиндрома с использованием regex

```javascript
function isPalindrome(str) {
  const cleanStr = str.toLowerCase().replace(/[\W_]/g, '');
  return cleanStr === cleanStr.split('').reverse().join('');
}

console.log(isPalindrome('A man, a plan, a canal: Panama')); // true
console.log(isPalindrome('race a car')); // false
```

## Практические советы

- Используйте `RegExp` конструктор при необходимости динамической генерации паттернов
- Для производительности кэшируйте регулярные выражения, если они используются многократно
- Избегайте catastrophic backtracking в сложных паттернах
- Используйте не захватывающие группы `(?:...)` для повышения производительности
- Тестируйте regex на различных входных данных

## Связанные темы

- [[javascript-validation-patterns]]
- [[string-manipulation-techniques]]
- [[frontend-security-basics]]