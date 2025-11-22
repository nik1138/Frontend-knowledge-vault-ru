---
aliases: ["Хитрости работы со строками в JavaScript", "Методы строк JS"]
tags: [js, strings, tricks, frontend]
---

# Хитрости работы со строками в JavaScript

Работа со строками - важная часть разработки на JavaScript. Ниже приведены эффективные методы и паттерны для работы со строками, которые особенно полезны в frontend разработке.

## 1. Проверка на пустую строку

```javascript
// Проверка на null, undefined или пустую строку
function isEmpty(str) {
  return !str || str.length === 0;
}

// Проверка на "пустую" строку (с учетом пробелов)
function isBlank(str) {
  return !str || str.trim().length === 0;
}

console.log(isEmpty('')); // true
console.log(isBlank('   ')); // true
console.log(isEmpty(null)); // true
```

## 2. Удаление лишних пробелов

```javascript
const str = '  Hello World  ';

// Удаление пробелов с обеих сторон
console.log(str.trim()); // 'Hello World'

// Удаление пробелов только слева
console.log(str.trimStart()); // 'Hello World  '

// Удаление пробелов только справа
console.log(str.trimEnd()); // '  Hello World'

// Удаление всех лишних пробелов (включая внутренние)
const multipleSpaces = 'Hello    World    from    JS';
console.log(multipleSpaces.replace(/\s+/g, ' ')); // 'Hello World from JS'
```

## 3. Капитализация строк

```javascript
// Капитализация первой буквы
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

// Капитализация всех слов
function capitalizeWords(str) {
  return str.split(' ').map(word => 
    word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
  ).join(' ');
}

// Капитализация всех слов (альтернативный метод)
function capitalizeWordsAlt(str) {
  return str.replace(/\b\w/g, l => l.toUpperCase());
}

console.log(capitalize('hello world')); // 'Hello world'
console.log(capitalizeWords('hello world from js')); // 'Hello World From Js'
console.log(capitalizeWordsAlt('hello world from js')); // 'Hello World From Js'
```

## 4. Проверка на палиндром

```javascript
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

console.log(isPalindrome('A man a plan a canal Panama')); // true
console.log(isPalindrome('race a car')); // false
```

## 5. Форматирование строк с шаблонами

```javascript
// Использование шаблонных литералов
const name = 'Alice';
const age = 25;
const greeting = `Привет, меня зовут ${name} и мне ${age} лет.`;

// Условные шаблоны
function getGreeting(user) {
  return `Здравствуйте, ${user.name || 'незнакомец'}!`;
}

// Многострочные шаблоны
const htmlTemplate = `
  <div class="user-card">
    <h3>${name}</h3>
    <p>Возраст: ${age}</p>
  </div>
`;

// Форматирование с обработкой специальных символов
function escapeHtml(str) {
  const escapeMap = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  };
  
  return str.replace(/[&<>"'\/]/g, s => escapeMap[s]);
}
```

## 6. Поиск и замена с регулярными выражениями

```javascript
const text = 'JavaScript - это мощный язык программирования';

// Поиск с флагами
console.log(text.search(/javascript/i)); // 0 (регистронезависимый поиск)

// Замена с флагами
console.log(text.replace(/javascript/i, 'JS')); // 'JS - это мощный язык программирования'

// Замена всех вхождений
const repeated = 'Hello hello HELLO';
console.log(repeated.replace(/hello/gi, 'Hi')); // 'Hi Hi Hi'

// Замена с функцией обратного вызова
const numbers = 'Цена: $10, $20, $30';
const converted = numbers.replace(/\$(\d+)/g, (match, price) => `EUR ${price}`);
console.log(converted); // 'Цена: EUR 10, EUR 20, EUR 30'
```

## 7. Разделение строк с ограничениями

```javascript
const csvLine = 'John,Doe,30,Engineer,New York';

// Разделение только первых N элементов
const parts = csvLine.split(',', 3); // ['John', 'Doe', '30']

// Разделение с сохранением кавычек (упрощенный пример)
function splitRespectingQuotes(str, delimiter) {
  const regex = new RegExp(`(["'])(?:(?!\\1).)*\\1|[^${delimiter}]+`, 'g');
  return str.match(regex) || [];
}

// Пример использования
const quotedLine = 'John,"Doe, Jr.",30';
console.log(quotedLine.split(',')); // ['John', '"Doe', ' Jr."', '30']
console.log(splitRespectingQuotes(quotedLine, ',')); // ['John', '"Doe, Jr."', '30', '']
```

## 8. Подсчет вхождений подстроки

```javascript
function countOccurrences(str, substring) {
  return (str.match(new RegExp(substring.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), 'g')) || []).length;
}

const text = 'JavaScript - это JavaScript, и JavaScript снова';
console.log(countOccurrences(text, 'JavaScript')); // 3

// Альтернативный метод
function countOccurrencesAlt(str, substring) {
  let count = 0;
  let pos = 0;
  
  while ((pos = str.indexOf(substring, pos)) !== -1) {
    count++;
    pos++;
  }
  
  return count;
}
```

## 9. Обрезка строк до определенной длины

```javascript
function truncate(str, maxLength, suffix = '...') {
  if (str.length <= maxLength) return str;
  return str.substring(0, maxLength - suffix.length) + suffix;
}

console.log(truncate('Длинная строка для обрезки', 15)); // 'Длинная строк...'
console.log(truncate('Длинная строка для обрезки', 15, '... (читать далее)')); // 'Длинная строк... (читать далее)'

// Обрезка по словам
function truncateWords(str, maxLength, suffix = '...') {
  if (str.length <= maxLength) return str;
  
  const truncated = str.substring(0, maxLength - suffix.length);
  const lastSpaceIndex = truncated.lastIndexOf(' ');
  
  return (lastSpaceIndex > 0 ? truncated.substring(0, lastSpaceIndex) : truncated) + suffix;
}

console.log(truncateWords('Это длинная строка, которую нужно обрезать по словам', 30)); // 'Это длинная строку, которую...'
```

## 10. Преобразование строк в разные форматы

```javascript
// Кебаб-кейс (kebab-case)
function toKebabCase(str) {
  return str
    .replace(/([a-z])([A-Z])/g, '$1-$2') // вставляем дефис между camelCase
    .replace(/[^a-zA-Z0-9]+/g, '-') // заменяем неалфавитные символы дефисами
    .toLowerCase()
    .replace(/^-+|-+$/g, ''); // удаляем начальные и конечные дефисы
}

// Снейк-кейс (snake_case)
function toSnakeCase(str) {
  return str
    .replace(/([a-z])([A-Z])/g, '$1_$2')
    .replace(/[^a-zA-Z0-9]+/g, '_')
    .toLowerCase()
    .replace(/^_+|_+$/g, '');
}

// Кэмел-кейс (camelCase)
function toCamelCase(str) {
  return str
    .replace(/[^a-zA-Z0-9]+(.)/g, (_, chr) => chr.toUpperCase())
    .replace(/^[A-Z]/, match => match.toLowerCase());
}

// Паскаль-кейс (PascalCase)
function toPascalCase(str) {
  return str
    .replace(/(?:^\w|[A-Z]|\b\w)/g, word => word.toUpperCase())
    .replace(/[^a-zA-Z0-9]+/g, '');
}

console.log(toKebabCase('Hello World')); // 'hello-world'
console.log(toSnakeCase('Hello World')); // 'hello_world'
console.log(toCamelCase('Hello World')); // 'helloWorld'
console.log(toPascalCase('hello world')); // 'HelloWorld'
```

## 11. Проверка соответствия шаблону

```javascript
// Проверка на email
function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// Проверка на номер телефона (простой пример)
function isValidPhone(phone) {
  const phoneRegex = /^\+?[\d\s\-\(\)]{10,}$/;
  return phoneRegex.test(phone);
}

// Проверка на URL
function isValidUrl(url) {
  try {
    new URL(url);
    return true;
  } catch (_) {
    return false;
  }
}

// Проверка на UUID
function isValidUUID(uuid) {
  const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;
  return uuidRegex.test(uuid);
}

console.log(isValidEmail('user@example.com')); // true
console.log(isValidPhone('+7 (999) 123-45-67')); // true
console.log(isValidUrl('https://example.com')); // true
console.log(isValidUUID('550e8400-e29b-41d4-a716-446655440000')); // true
```

## 12. Работа с юникодом и эмодзи

```javascript
// Подсчет символов (включая эмодзи)
function countSymbols(str) {
  return [...str].length; // используем spread-оператор для правильного подсчета эмодзи
}

// Проверка наличия эмодзи
function containsEmoji(str) {
  const emojiRegex = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu;
  return emojiRegex.test(str);
}

// Удаление эмодзи
function removeEmojis(str) {
  const emojiRegex = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/gu;
  return str.replace(emojiRegex, '');
}

console.log(countSymbols('Hello 👋 World 🌍')); // 13 (вместо 15 при обычном str.length)
console.log(containsEmoji('Hello 👋 World')); // true
console.log(removeEmojis('Hello 👋 World 🌍')); // 'Hello  World '
```

## 13. Экранирование HTML

```javascript
function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };

  return text.replace(/[&<>"']/g, m => map[m]);
}

function unescapeHtml(text) {
  const map = {
    '&amp;': '&',
    '&lt;': '<',
    '&gt;': '>',
    '&quot;': '"',
    '&#039;': "'",
    '&apos;': "'"
  };

  return text.replace(/&amp;|&lt;|&gt;|&quot;|&#039;|&apos;/g, m => map[m]);
}

const userInput = '<script>alert("XSS")</script>';
console.log(escapeHtml(userInput)); // '&lt;script&gt;alert(&quot;XSS&quot;)&lt;&#x2F;script&gt;'
console.log(unescapeHtml(escapeHtml(userInput))); // '<script>alert("XSS")</script>'
```

## 14. Повторение строк

```javascript
// Повторение строки
function repeatString(str, count) {
  return str.repeat(count);
}

// Повторение с разделителем
function repeatStringWithSeparator(str, count, separator = '') {
  return Array(count).fill(str).join(separator);
}

console.log('='.repeat(20)); // '===================='
console.log(repeatStringWithSeparator('Hello', 3, ' ')); // 'Hello Hello Hello'
```

## 15. Сравнение строк с учетом регистра

```javascript
// Локализованное сравнение строк
function compareStrings(str1, str2, caseSensitive = true) {
  if (caseSensitive) {
    return str1.localeCompare(str2);
  }
  return str1.toLowerCase().localeCompare(str2.toLowerCase());
}

console.log(compareStrings('apple', 'Banana', false)); // отрицательное число (apple < banana)
console.log(compareStrings('apple', 'Banana', true)); // положительное число (apple > Banana)

// Сортировка массива строк с учетом регистра
const fruits = ['apple', 'Banana', 'cherry', 'Date'];
console.log(fruits.sort((a, b) => a.localeCompare(b))); // ['apple', 'Banana', 'cherry', 'Date']
console.log(fruits.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()))); // ['apple', 'Banana', 'cherry', 'Date']
```

Эти хитрости помогут вам эффективно работать со строками в JavaScript, улучшая производительность и безопасность ваших frontend приложений.

См. также:
- [[js/tricks/arrays-tricks]] - Работа с массивами
- [[js/tricks/numbers-tricks]] - Числа и математика
- [[js/tricks/objects-tricks]] - Работа с объектами