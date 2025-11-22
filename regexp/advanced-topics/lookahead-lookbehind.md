---
aliases: [Lookahead, Lookbehind, Advanced Anchoring]
tags: [regexp, advanced, lookahead, lookbehind]
---

# Lookahead и lookbehind продвинутого уровня

## Обзор

Lookahead и lookbehind - мощные функции регулярных выражений, которые позволяют проверять наличие или отсутствие шаблона без включения его в результат. В этой статье рассмотрим продвинутые применения этих функций.

## Positive Lookahead

```javascript
// Positive lookahead - проверяет, что шаблон следует за текущей позицией, но не включает его в совпадение
const text = 'Пароль123! и пароль456# и секрет789$';

// Найти все слова "пароль", за которыми следует число и специальный символ
const positiveLookaheadPattern = /пароль(?=\d+[!#$])/gi;
const matches = text.match(positiveLookaheadPattern);
console.log(matches); // ['пароль', 'пароль', 'пароль']

// Более сложный пример: извлечение слов перед определенными шаблонами
const complexText = 'Цена: 100$, Скидка: 20%, Налог: 5eur, Сумма: 1000₽';
const currencyPattern = /\d+(?=\$|%|eur|₽)/g;
console.log(complexText.match(currencyPattern)); // ['100', '20', '5', '1000']
```

## Negative Lookahead

```javascript
// Negative lookahead - проверяет, что шаблон НЕ следует за текущей позицией
const text = 'file.txt, file.bak, image.jpg, document.pdf, temp.tmp';

// Найти все файлы, которые НЕ являются резервными копиями (.bak) или временными (.tmp)
const notBackupPattern = /[\w]+(?!\.bak|\.tmp)\.\w+/g;
console.log(text.match(notBackupPattern)); // ['file.txt', 'image.jpg', 'document.pdf']

// Пример: избежать совпадения с определенными словами
const sentence = 'Я люблю JavaScript, Python, но не Java и не C++';
const notJavaPattern = /\b\w+(?!Script| не Java)\b/gi;
console.log(sentence.match(notJavaPattern)); // Совпадения будут содержать слова, за которыми не следует 'Script' или ' не Java'
```

## Positive Lookbehind

```javascript
// Positive lookbehind - проверяет, что шаблон предшествует текущей позиции
// Доступно в современных браузерах (не поддерживается в IE)

// Найти числа, перед которыми стоит символ валюты
const priceText = '$100, €200, ¥300, 400р, $500';
try {
  const positiveLookbehindPattern = /(?<=\$)\d+/g;
  console.log(priceText.match(positiveLookbehindPattern)); // ['100', '500']
} catch (e) {
  console.log('Positive lookbehind не поддерживается в этом окружении');
}

// Найти числа, за которыми следует символ валюты
try {
  const lookbehindForCurrency = /(?<=\$|€|¥)\d+/g;
  console.log(priceText.match(lookbehindForCurrency)); // ['100', '200', '300', '500']
} catch (e) {
  console.log('Positive lookbehind не поддерживается в этом окружении');
}
```

## Negative Lookbehind

```javascript
// Negative lookbehind - проверяет, что шаблон НЕ предшествует текущей позиции
const versionText = 'Версия 1.0.0, v2.0.0, номер 3.0.0, release 4.0.0';

try {
  // Найти числа версий, которые НЕ начинаются с 'v'
  const negativeLookbehindPattern = /(?<!v)\d+\.\d+\.\d+/g;
  console.log(versionText.match(negativeLookbehindPattern)); // ['1.0.0', '3.0.0', '4.0.0']
} catch (e) {
  console.log('Negative lookbehind не поддерживается в этом окружении');
}
```

## Комплексные примеры с lookahead и lookbehind

```javascript
// Пример: извлечение email-адресов, исключая те, которые находятся в тегах комментариев
const textWithComments = `
  Контакт: user@example.com
  <!-- Скрытый: admin@hidden.com -->
  Поддержка: support@company.org
  <!-- Заметка: info@internal.net -->
  Обратная связь: feedback@public.com
`;

try {
  // Найти email-адреса, которые НЕ находятся между <!-- и -->
  const emailPattern = /(?<!<!--.*?)[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(?!.*?-->)/g;
  const emails = textWithComments.match(emailPattern);
  console.log(emails); // ['user@example.com', 'support@company.org', 'feedback@public.com']
} catch (e) {
  console.log('Lookbehind не поддерживается, используем альтернативный подход');
  
  // Альтернативный подход без lookbehind
  const cleanText = textWithComments.replace(/<!--[\s\S]*?-->/g, '');
  const emailPatternAlt = /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g;
  console.log(cleanText.match(emailPatternAlt));
}
```

## Использование lookahead для проверки сложных условий пароля

```javascript
// Проверка сложного пароля с несколькими условиями
function validateComplexPassword(password) {
  // Пароль должен содержать:
  // - не менее 8 символов
  // - хотя бы одну заглавную букву
  // - хотя бы одну строчную букву
  // - хотя бы одну цифру
  // - хотя бы один специальный символ
  const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  
  return passwordRegex.test(password);
}

console.log(validateComplexPassword('Password123!')); // true
console.log(validateComplexPassword('weakpass')); // false
console.log(validateComplexPassword('STRONGPASS123!')); // true
```

## Использование lookahead для извлечения слов с определенными условиями

```javascript
// Найти слова, которые содержат как минимум 2 гласные
const sentence = 'Программирование это интересно и полезно';
const vowelPattern = /\b\w*(?=(?:\w*[аеёиоуыэюяАЕЁИОУЫЭЮЯ]\w*){2,})\w*\b/gi;
console.log(sentence.match(vowelPattern)); // ['Программирование', 'интересно', 'полезно']
```

## Использование lookbehind для извлечения значений с префиксами

```javascript
// Извлечение значений после определенных префиксов
const data = 'Имя: Иван, Возраст: 30, Город: Москва, Рейтинг: 4.5';
try {
  // Извлечь все значения после двоеточия
  const valuesPattern = /(?<=:\s)[^,]+/g;
  console.log(data.match(valuesPattern)); // ['Иван', '30', 'Москва', '4.5']
} catch (e) {
  // Альтернативный подход
  const valuesPatternAlt = /:\s([^,]+)/g;
  const matches = [...data.matchAll(valuesPatternAlt)];
  console.log(matches.map(m => m[1])); // ['Иван', '30', 'Москва', '4.5']
}
```

## Продвинутые примеры с комбинацией lookahead и lookbehind

```javascript
// Пример: извлечение чисел, которые не являются годами (4 цифры между пробелами)
const text = 'В 2023 году было 15 событий, 200 событий за всё время, дата 1999-01-01';
try {
  // Найти числа, которые НЕ являются 4-значными годами
  const nonYearNumbers = /(?<!\d)(?!\d{4}(?!\d))\d+(?!\d)/g;
  console.log(text.match(nonYearNumbers)); // ['15', '200', '01', '01']
} catch (e) {
  console.log('Комплексный lookbehind не поддерживается');
}

// Альтернативный подход
const numbers = text.match(/\d+/g);
const years = text.match(/\b\d{4}\b/g);
const nonYearNumbers = numbers.filter(num => !years.includes(num));
console.log('Числа, не являющиеся годами:', nonYearNumbers.filter(num => num !== '1999')); // ['15', '200', '01', '01', '01']
```

## Практическое применение в валидации

```javascript
// Класс для сложной валидации с использованием lookahead
class AdvancedValidator {
  static patterns = {
    // Пароль: 8+ символов, заглавная, строчная, цифра, специальный символ
    strongPassword: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
    
    // Имя пользователя: не начинается с цифры, содержит буквы и цифры
    validUsername: /^(?!\d)[a-zA-Z0-9_]{3,30}$/,
    
    // Номер телефона: определенные форматы
    phone: /^(?=\+?1?-?\.?\s?\(?[0-9]{3}\)?[\s.-]?[0-9]{3}[\s.-]?[0-9]{4}$)/,
    
    // Валидный URL: начинается с http:// или https://
    validUrl: /^(?=https?:\/\/)/
  };
  
  static validate(type, value) {
    const pattern = this.patterns[type];
    if (!pattern) return false;
    
    return pattern.test(value);
  }
  
  // Валидация с объяснением
  static validateWithExplanation(type, value) {
    switch (type) {
      case 'strongPassword':
        const checks = [
          { test: /(?=.*[a-z])/, message: 'Должна быть хотя бы одна строчная буква' },
          { test: /(?=.*[A-Z])/, message: 'Должна быть хотя бы одна заглавная буква' },
          { test: /(?=.*\d)/, message: 'Должна быть хотя бы одна цифра' },
          { test: /(?=.*[@$!%*?&])/, message: 'Должен быть хотя бы один специальный символ' },
          { test: /^.{8,}/, message: 'Должно быть не менее 8 символов' }
        ];
        
        const failedChecks = checks.filter(check => !check.test(value));
        return {
          valid: failedChecks.length === 0,
          messages: failedChecks.map(check => check.message)
        };
      
      default:
        return {
          valid: this.validate(type, value),
          messages: []
        };
    }
  }
}

// Пример использования
console.log(AdvancedValidator.validate('strongPassword', 'MyPass123!')); // true
console.log(AdvancedValidator.validateWithExplanation('strongPassword', 'weak')); 
// { valid: false, messages: ['Должна быть хотя бы одна заглавная буква', 'Должна быть хотя бы одна цифра', 'Должен быть хотя бы один специальный символ', 'Должно быть не менее 8 символов'] }
```

## Заключение

Lookahead и lookbehind - мощные инструменты для сложных задач сопоставления. Они позволяют проверять контекст без включения его в результат, что делает регулярные выражения более гибкими и точными. Однако стоит учитывать поддержку этими функциями в различных окружениях.

## См. также

- [[Recursive Patterns and Bracket Balancing]]
- [[RegExp Performance Tips]]
- [[Complex Parsing Strategies]]