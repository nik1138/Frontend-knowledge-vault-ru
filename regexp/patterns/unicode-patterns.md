---
aliases: [Unicode Patterns, International Text, UTF-8 RegExps]
tags: [regexp, unicode, international, patterns]
---

# Паттерны для работы с юникодом

## Обзор

Регулярные выражения в JavaScript поддерживают работу с Unicode через флаг `u` и специальные последовательности. В этой статье рассмотрим паттерны для работы с международными текстами, эмодзи и другими юникодными символами.

## Базовые юникодные паттерны

```javascript
// Проверка, содержит ли строка кириллические символы
const cyrillicPattern = /[\u0400-\u04FF]/;

console.log(cyrillicPattern.test('Привет')); // true
console.log(cyrillicPattern.test('Hello')); // false

// Проверка на китайские иероглифы
const chinesePattern = /[\u4e00-\u9fff]/;

console.log(chinesePattern.test('你好')); // true
console.log(chinesePattern.test('Hello')); // false

// Проверка на японские символы (хирагана, катакана, кандзи)
const japanesePattern = /[\u3040-\u309f\u30a0-\u30ff\u4e00-\u9faf]/;

console.log(japanesePattern.test('こんにちは')); // true
console.log(japanesePattern.test('コンニチハ')); // true
console.log(japanesePattern.test('안녕하세요')); // false (корейский)
```

## Работа с эмодзи

```javascript
// Паттерн для поиска эмодзи
const emojiPattern = /[\u{1f600}-\u{1f64f}]|[\u{1f300}-\u{1f5ff}]|[\u{1f680}-\u{1f6ff}]|[\u{1f1e0}-\u{1f1ff}]|[\u{2600}-\u{26ff}]|[\u{2700}-\u{27bf}]/gu;

const textWithEmojis = 'Привет! 😊 Как дела? 🤔 Сегодня 🌞 солнечно!';
const emojis = textWithEmojis.match(emojiPattern);
console.log(emojis); // ['😊', '🤔', '🌞']

// Удаление эмодзи из текста
function removeEmojis(text) {
  return text.replace(emojiPattern, '');
}

console.log(removeEmojis(textWithEmojis)); // Привет!  Как дела?  Сегодня  солнечно!
```

## Подсчет символов с учетом суррогатных пар

```javascript
// Функция для подсчета реальных символов (с учетом эмодзи и других многосимвольных юникодных последовательностей)
function countSymbols(text) {
  return [...text.matchAll(/[\s\S]/gu)].length;
}

// Сравнение с обычным .length
const emojiText = 'Hello 👨‍👩‍👧‍👦 World! 🧠';
console.log('Длина .length:', emojiText.length); // 17 (неправильно)
console.log('Длина countSymbols:', countSymbols(emojiText)); // 14 (правильно)

// Правильное извлечение символов
function getSymbols(text) {
  return [...text.matchAll(/[\s\S]/gu)].map(match => match[0]);
}

console.log(getSymbols(emojiText)); // ['H', 'e', 'l', 'l', 'o', ' ', '👨‍👩‍👧‍👦', ' ', 'W', 'o', 'r', 'l', 'd', '! ', ' ', '🧠']
```

## Работа с различными письменностями

```javascript
// Паттерны для различных письменностей
const patterns = {
  arabic: /[\u0600-\u06FF]/,
  devanagari: /[\u0900-\u097F]/,
  thai: /[\u0E00-\u0E7F]/,
  korean: /[\uAC00-\uD7AF]/,
  greek: /[\u0370-\u03FF]/,
  hebrew: /[\u0590-\u05FF]/
};

function detectScript(text) {
  for (const [script, pattern] of Object.entries(patterns)) {
    if (pattern.test(text)) {
      return script;
    }
  }
  return 'latin';
}

console.log(detectScript('مرحبا')); // arabic
console.log(detectScript('नमस्ते')); // devanagari
console.log(detectScript('สวัสดี')); // thai
console.log(detectScript('안녕하세요')); // korean
console.log(detectScript('Γεια σας')); // greek
console.log(detectScript('שלום')); // hebrew
console.log(detectScript('Hello')); // latin
```

## Валидация и очистка текста с юникодом

```javascript
// Функция для валидации текста на принадлежность к определенному языку
function validateLanguage(text, language = 'cyrillic') {
  const patterns = {
    cyrillic: /^[\u0400-\u04FF\s\d\p{P}\p{S}]*$/u,
    latin: /^[a-zA-Z\s\d\p{P}\p{S}]*$/u,
    mixed: /^[\u0400-\u04FFa-zA-Z\s\d\p{P}\p{S}]*$/u
  };
  
  // Для использования \p{P} и \p{S} нужно флаг 'u' и поддержка браузером
  try {
    const pattern = patterns[language];
    if (!pattern) return false;
    return pattern.test(text);
  } catch (e) {
    // Альтернативная реализация без \p{} если не поддерживается
    if (language === 'cyrillic') {
      return /^[\u0400-\u04FFa-zA-Z\s\d.,!?;:\-_'"()]*$/.test(text);
    }
    return true;
  }
}

console.log(validateLanguage('Привет мир!')); // true
console.log(validateLanguage('Hello world!')); // false (для кириллицы)
```

## Извлечение текста определенной письменности

```javascript
// Функция для извлечения частей текста на определенном языке
function extractByScript(text, script) {
  const patterns = {
    cyrillic: /[\u0400-\u04FF]+/g,
    chinese: /[\u4e00-\u9fff]+/g,
    arabic: /[\u0600-\u06FF]+/g,
    japanese: /[\u3040-\u309f\u30a0-\u30ff\u4e00-\u9faf]+/g
  };
  
  const pattern = patterns[script];
  if (!pattern) return [];
  
  return text.match(pattern) || [];
}

const multilingualText = 'Привет! Hello! 你好! مرحبا! こんにちは!';
console.log(extractByScript(multilingualText, 'cyrillic')); // ['Привет']
console.log(extractByScript(multilingualText, 'chinese')); // ['你好']
console.log(extractByScript(multilingualText, 'arabic')); // ['مرحبا']
console.log(extractByScript(multilingualText, 'japanese')); // ['こんにちは']
```

## Работа с диакритическими знаками

```javascript
// Паттерн для поиска и удаления диакритических знаков
function removeAccents(text) {
  // Сначала нормализуем текст
  const normalized = text.normalize('NFD');
  // Затем удаляем диакритические знаки
  return normalized.replace(/[\u0300-\u036f]/g, '');
}

console.log(removeAccents('café')); // cafe
console.log(removeAccents('naïve')); // naive
console.log(removeAccents('résumé')); // resume
console.log(removeAccents('Москва')); // Москва (кириллица не меняется)

// Паттерн для поиска текста с диакритическими знаками
const accentedPattern = /\p{M}+/gu; // Маркировщики (диакритические знаки)

// Обратите внимание: \p{M} требует флага 'u' и поддержки браузером
try {
  const textWithAccents = 'café naïve résumé';
  console.log([...textWithAccents.matchAll(accentedPattern)].map(m => m[0]));
} catch (e) {
  console.log('Шаблон \\p{M} не поддерживается в этом окружении');
}
```

## Проверка на валидность юникодных строк

```javascript
// Функция для проверки, является ли строка валидной юникодной строкой
function isValidUnicode(str) {
  try {
    // Попытка нормализовать строку
    str.normalize('NFC');
    return true;
  } catch (e) {
    return false;
  }
}

// Функция для проверки на наличие проблемных юникодных последовательностей
function checkUnicodeSafety(text) {
  // Проверяем на управляющие символы (кроме перевода строки и табуляции)
  const controlPattern = /[\u0000-\u0008\u000B-\u001F\u007F-\u009F]/;
  
  // Проверяем на непарные суррогаты
  const unpairedSurrogatePattern = /[\uD800-\uDBFF](?![\uDC00-\uDFFF])|[\uDC00-\uDFFF](?<![\uD800-\uDBFF])/;
  
  return {
    hasControlChars: controlPattern.test(text),
    hasUnpairedSurrogates: unpairedSurrogatePattern.test(text),
    isValid: !controlPattern.test(text) && !unpairedSurrogatePattern.test(text)
  };
}

const testString = 'Hello\u0001World'; // Содержит управляющий символ
console.log(checkUnicodeSafety(testString));
// { hasControlChars: true, hasUnpairedSurrogates: false, isValid: false }
```

## Практический пример: валидатор многоязычных имен

```javascript
// Класс для валидации имен на различных языках
class MultilingualNameValidator {
  constructor() {
    // Паттерны для различных языков
    this.patterns = {
      cyrillic: /^[\u0400-\u04FF\s-]+$/u,
      latin: /^[a-zA-Z\s-]+$/u,
      chinese: /^[\u4e00-\u9fff\s-]+$/u,
      japanese: /^[\u3040-\u309f\u30a0-\u30ff\u4e00-\u9faf\s-]+$/u,
      arabic: /^[\u0600-\u06FF\s\u064B-\u065F\u0670\u06D6-\u06ED]+$/u,
      mixed: /^[\u0400-\u04FFa-zA-Z\u4e00-\u9fff\u3040-\u309f\u30a0-\u30ff\s-]+$/u
    };
  }
  
  validate(name, language = 'mixed') {
    const pattern = this.patterns[language];
    if (!pattern) {
      throw new Error(`Неизвестный язык: ${language}`);
    }
    
    // Убедимся, что имя не пустое и не содержит только пробелы
    if (!name || !name.trim()) {
      return { valid: false, error: 'Имя не может быть пустым' };
    }
    
    // Проверим длину
    const symbolCount = [...name.matchAll(/[\s\S]/gu)].length;
    if (symbolCount < 2 || symbolCount > 50) {
      return { valid: false, error: 'Имя должно быть от 2 до 50 символов' };
    }
    
    // Проверим соответствие паттерну
    if (!pattern.test(name)) {
      return { valid: false, error: `Имя содержит недопустимые символы для языка ${language}` };
    }
    
    // Проверим, не начинается и не заканчивается на пробел или дефис
    if (/^[\s-]|[\s-]$/.test(name)) {
      return { valid: false, error: 'Имя не должно начинаться или заканчиваться пробелом или дефисом' };
    }
    
    return { valid: true, error: null };
  }
  
  detectLanguage(name) {
    for (const [lang, pattern] of Object.entries(this.patterns)) {
      if (lang !== 'mixed' && pattern.test(name)) {
        return lang;
      }
    }
    return 'mixed';
  }
}

// Пример использования
const validator = new MultilingualNameValidator();

console.log(validator.validate('Иван Иванов')); // { valid: true, error: null }
console.log(validator.validate('John Doe')); // { valid: true, error: null }
console.log(validator.validate('田中太郎')); // { valid: true, error: null }
console.log(validator.validate('محمد أحمد')); // { valid: false, error: 'Имя содержит недопустимые символы для языка cyrillic' }

console.log(validator.detectLanguage('Иван Иванов')); // cyrillic
console.log(validator.detectLanguage('John Doe')); // latin
console.log(validator.detectLanguage('田中太郎')); // japanese
```

## Заключение

Работа с юникодом в регулярных выражениях требует понимания особенностей различных письменностей и суррогатных пар. Правильное использование юникодных паттернов позволяет создавать интернационализованные приложения, корректно обрабатывающие текст на различных языках.

## См. также

- [[Internationalization Best Practices]]
- [[Text Processing Patterns]]
- [[RegExp Performance Tips]]