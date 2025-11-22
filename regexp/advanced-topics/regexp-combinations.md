---
aliases: [Multiple RegExps, RegExp Combinations, Pattern Chaining]
tags: [regexp, advanced, combinations, optimization]
---

# Комбинирование нескольких регулярных выражений

## Обзор

Часто одна регулярка не может справиться со всей сложностью задачи. В этой статье рассмотрим стратегии комбинирования нескольких регулярных выражений для решения сложных задач обработки текста.

## Последовательное применение регулярных выражений

```javascript
// Пример: очистка и нормализация пользовательского ввода
function normalizeUserInput(input) {
  // Шаг 1: Удалить HTML-теги
  let result = input.replace(/<[^>]*>/g, '');
  
  // Шаг 2: Заменить множественные пробелы на один
  result = result.replace(/\s+/g, ' ');
  
  // Шаг 3: Удалить начальные и конечные пробелы
  result = result.trim();
  
  // Шаг 4: Заменить опасные символы
  result = result.replace(/[<>]/g, '');
  
  return result;
}

const userInput = '  <p>Пользовательский   ввод с <script>опасным</script> содержимым</p>  ';
console.log(normalizeUserInput(userInput)); // Пользовательский ввод с опасным содержимым
```

## Использование массива регулярных выражений

```javascript
// Класс для последовательной обработки с помощью нескольких регулярных выражений
class RegExpPipeline {
  constructor() {
    this.patterns = [];
  }
  
  add(pattern, replacement) {
    this.patterns.push({ pattern, replacement });
    return this;
  }
  
  process(text) {
    return this.patterns.reduce((result, { pattern, replacement }) => {
      return result.replace(pattern, replacement);
    }, text);
  }
}

// Пример использования
const pipeline = new RegExpPipeline()
  .add(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '') // Удалить скрипты
  .add(/<[^>]*>/g, '') // Удалить остальные теги
  .add(/\s+/g, ' ') // Нормализовать пробелы
  .add(/^\s+|\s+$/g, ''); // Удалить крайние пробелы

const htmlText = '<div>Текст <script>alert("xss")</script> с <em>форматированием</em></div>';
console.log(pipeline.process(htmlText)); // Текст с форматированием
```

## Условное применение регулярных выражений

```javascript
// Функция для условной обработки на основе типа содержимого
function conditionalProcessing(text) {
  // Проверяем, содержит ли текст email
  if (/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/.test(text)) {
    // Обработка email
    return text.replace(/\b([A-Za-z0-9._%+-]+)@([A-Za-z0-9.-]+\.[A-Z|a-z]{2,})\b/g, 
                        (_, user, domain) => `${user[0]}***@${domain}`);
  }
  
  // Проверяем, содержит ли текст номер телефона
  if (/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/.test(text)) {
    // Обработка номера телефона
    return text.replace(/\b(\d{3})[-.]?(\d{3})[-.]?(\d{4})\b/g, 
                        (_, area, exchange, number) => `(${area}) ***-${number}`);
  }
  
  // Обработка по умолчанию
  return text;
}

console.log(conditionalProcessing('Контакт: john@example.com')); // Контакт: j***@example.com
console.log(conditionalProcessing('Телефон: 123-456-7890')); // Телефон: (123) ***-7890
console.log(conditionalProcessing('Простой текст')); // Простой текст
```

## Комбинирование регулярных выражений для валидации

```javascript
// Класс для сложной валидации с использованием нескольких регулярных выражений
class MultiPatternValidator {
  constructor() {
    this.validators = {
      email: /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/,
      phone: /^\+?[\d\s\-\(\)]{10,20}$/,
      username: /^[a-zA-Z0-9_]{3,30}$/,
      strongPassword: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/
    };
  }
  
  validate(type, value) {
    const validator = this.validators[type];
    if (!validator) {
      throw new Error(`Неизвестный тип валидации: ${type}`);
    }
    
    return validator.test(value);
  }
  
  validateMultiple(types, value) {
    return types.every(type => this.validate(type, value));
  }
  
  getFailedValidations(types, value) {
    return types.filter(type => !this.validate(type, value));
  }
}

// Пример использования
const validator = new MultiPatternValidator();
console.log(validator.validate('email', 'user@example.com')); // true
console.log(validator.validateMultiple(['username', 'email'], 'testuser')); // false
console.log(validator.getFailedValidations(['email', 'phone'], 'invalid')); // ['email', 'phone']
```

## Извлечение данных с помощью нескольких паттернов

```javascript
// Функция для извлечения различных типов данных из текста
function extractData(text) {
  const patterns = {
    emails: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g,
    phones: /\+?\d{1,4}?[-.\s]?\(?\d{1,3}?\)?[-.\s]?\d{1,4}[-.\s]?\d{1,4}[-.\s]?\d{1,9}/g,
    urls: /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g,
    dates: /\b\d{1,2}[\/\-]\d{1,2}[\/\-]\d{4}\b/g,
    ips: /\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b/g
  };
  
  const results = {};
  
  for (const [type, pattern] of Object.entries(patterns)) {
    results[type] = text.match(pattern) || [];
  }
  
  return results;
}

const mixedText = `
  Контакты: 
  Email: john@example.com, jane@test.org
  Телефон: +1-234-567-8900, (987) 654-3210
  Сайт: https://example.com
  Дата: 25/12/2023
  IP: 192.168.1.1
`;

console.log(extractData(mixedText));
```

## Комбинирование с условиями и логикой

```javascript
// Продвинутый парсер логов с комбинацией регулярных выражений
class LogParser {
  constructor() {
    this.patterns = {
      timestamp: /\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}/,
      level: /INFO|ERROR|WARN|DEBUG/,
      ip: /\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b/,
      url: /\/[a-zA-Z0-9\-_/]*/,
      status: /\d{3}/,
      userAgent: /"(?:[^"\\]|\\.)*"/
    };
  }
  
  parseLogLine(line) {
    const result = {};
    
    // Извлекаем метку времени
    const timestampMatch = line.match(this.patterns.timestamp);
    if (timestampMatch) result.timestamp = timestampMatch[0];
    
    // Извлекаем уровень лога
    const levelMatch = line.match(this.patterns.level);
    if (levelMatch) result.level = levelMatch[0];
    
    // Извлекаем IP-адрес
    const ipMatch = line.match(this.patterns.ip);
    if (ipMatch) result.ip = ipMatch[0];
    
    // Извлекаем URL
    const urlMatch = line.match(this.patterns.url);
    if (urlMatch) result.url = urlMatch[0];
    
    // Извлекаем статус
    const statusMatch = line.match(this.patterns.status);
    if (statusMatch) result.status = statusMatch[0];
    
    // Извлекаем User-Agent
    const userAgentMatch = line.match(this.patterns.userAgent);
    if (userAgentMatch) result.userAgent = userAgentMatch[0].slice(1, -1); // Убираем кавычки
    
    return result;
  }
  
  parseMultiple(lines) {
    return lines.map(line => this.parseLogLine(line)).filter(obj => Object.keys(obj).length > 0);
  }
}

// Пример использования
const logLines = [
  '2023-10-15 10:30:25 ERROR 192.168.1.100 /api/users 500 "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"',
  '2023-10-15 10:31:10 INFO 10.0.0.1 /home 200 "Mozilla/5.0 (Macintosh; Intel Mac OS X)"'
];

const parser = new LogParser();
console.log(parser.parseMultiple(logLines));
```

## Оптимизация комбинаций регулярных выражений

```javascript
// Класс для оптимизации выполнения нескольких регулярных выражений
class OptimizedRegExpProcessor {
  constructor() {
    // Компилируем регулярные выражения заранее
    this.compiledPatterns = new Map();
  }
  
  addPattern(name, pattern, flags = 'g') {
    this.compiledPatterns.set(name, new RegExp(pattern, flags));
    return this;
  }
  
  executePattern(name, text) {
    const pattern = this.compiledPatterns.get(name);
    if (!pattern) {
      throw new Error(`Шаблон ${name} не найден`);
    }
    
    return text.match(pattern) || [];
  }
  
  executeMultiple(names, text) {
    const results = {};
    for (const name of names) {
      results[name] = this.executePattern(name, text);
    }
    return results;
  }
  
  // Выполняет все шаблоны и возвращает результаты
  executeAll(text) {
    const results = {};
    for (const [name, pattern] of this.compiledPatterns) {
      results[name] = text.match(pattern) || [];
    }
    return results;
  }
}

// Пример использования
const processor = new OptimizedRegExpProcessor()
  .addPattern('emails', /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g)
  .addPattern('phones', /\+?\d{1,4}[-.\s]?\(?\d{1,3}\)?[-.\s]?\d{1,4}[-.\s]?\d{1,4}[-.\s]?\d{1,9}/g)
  .addPattern('urls', /https?:\/\/[^\s/$.?#].[^\s]*/g);

const text = 'Email: user@example.com, Phone: +1-234-567-8900, Site: https://example.com';
console.log(processor.executeAll(text));
```

## Использование регулярных выражений в цепочках обработки

```javascript
// Пример цепочки обработки данных
function processUserData(rawData) {
  return rawData
    // Удалить HTML-теги
    .replace(/<[^>]*>/g, '')
    // Удалить лишние пробелы
    .replace(/\s+/g, ' ')
    // Обрезать края
    .trim()
    // Заменить email на безопасный формат
    .replace(/\b([A-Za-z0-9._%+-]+)@([A-Za-z0-9.-]+\.[A-Z|a-z]{2,})\b/g, 
             (_, user, domain) => `${user[0]}***@${domain}`)
    // Заменить телефон на безопасный формат
    .replace(/\b(\d{3})[-.]?(\d{3})[-.]?(\d{4})\b/g, 
             (_, area, exchange, number) => `(${area}) ***-${number}`);
}

const rawUserData = '<p>Контакт: john.doe@example.com, тел: 123-456-7890</p>';
console.log(processUserData(rawUserData)); // Контакт: j***@example.com, тел: (123) ***-7890
```

## Заключение

Комбинирование нескольких регулярных выражений позволяет решать сложные задачи обработки текста, которые невозможно решить с помощью одного выражения. Правильная организация и оптимизация таких комбинаций может значительно повысить эффективность и читаемость кода.

## См. также

- [[RegExp Performance Tips]]
- [[Advanced Lookahead and Lookbehind]]
- [[Complex Parsing Strategies]]