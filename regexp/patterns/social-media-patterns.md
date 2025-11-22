---
aliases: [Social Media Patterns, Hashtag Extraction, Mention Parsing]
tags: [regexp, social, media, patterns]
---

# Паттерны для извлечения хэштегов, упоминаний и ссылок

## Обзор

В социальных сетях и приложениях обработки текста часто требуется извлекать хэштеги, упоминания пользователей и ссылки. В этой статье рассмотрим эффективные паттерны для этих целей.

## Извлечение хэштегов

```javascript
// Простой паттерн для извлечения хэштегов
const hashtagPattern = /#[a-zA-Zа-яёА-ЯЁ0-9_]+/g;

const textWithHashtags = 'Программирование #JavaScript #Frontend #WebDev это интересно!';
const hashtags = textWithHashtags.match(hashtagPattern);
console.log(hashtags); // ['#JavaScript', '#Frontend', '#WebDev']

// Улучшенный паттерн, поддерживающий юникод
const unicodeHashtagPattern = /#[^\s\u200C-\u200D\u0300-\u036F\p{M}]+/gu;

try {
  const multilingualText = '#Привет #JavaScript #العربية #日本語 #한국어';
  console.log(multilingualText.match(unicodeHashtagPattern)); // ['#Привет', '#JavaScript', '#العربية', '#日本語', '#한국어']
} catch (e) {
  // Альтернатива для окружений, не поддерживающих \p{}
  const altHashtagPattern = /#[^\s!@#$%^&*(),.?":{}|<>[\]\\;'`~]+/g;
  console.log('Используем альтернативный паттерн');
}
```

## Извлечение упоминаний пользователей

```javascript
// Паттерн для извлечения упоминаний пользователей (никнеймы)
const mentionPattern = /@[a-zA-Z0-9_]{1,30}(?!\w)/g;

const textWithMentions = 'Привет @john_doe, смотри что @jane_smith показала в #JavaScript';
const mentions = textWithMentions.match(mentionPattern);
console.log(mentions); // ['@john_doe', '@jane_smith']

// Улучшенный паттерн для разных платформ
function getMentionPattern(platform = 'default') {
  const patterns = {
    default: /@[a-zA-Z0-9_]{1,30}(?!\w)/g,
    twitter: /@[a-zA-Z0-9_]{1,15}(?!\w)/g,  // Twitter ограничивает 15 символами
    github: /@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,38}[a-zA-Z0-9])?(?!\w)/g  // GitHub: 1-39 символов, может содержать дефисы
  };
  
  return patterns[platform] || patterns.default;
}

console.log('Twitter mentions:', 'Привет @verylongtwitterusername123'.match(getMentionPattern('twitter'))); // null (слишком длинный)
console.log('GitHub mentions:', 'Код на @user-name'.match(getMentionPattern('github'))); // ['@user-name']
```

## Извлечение URL-ссылок

```javascript
// Паттерн для извлечения URL-ссылок
const urlPattern = /https?:\/\/(?:[-\w.])+(?:\:[0-9]+)?(?:\/(?:[\w\/_.])*(?:\?(?:[\w&=%.])*)?(?:\#(?:[\w.])*)?)?/g;

const textWithUrls = 'Посетите https://example.com или https://subdomain.site.org/path?param=value#section';
const urls = textWithUrls.match(urlPattern);
console.log(urls); // ['https://example.com', 'https://subdomain.site.org/path?param=value#section']

// Более комплексный паттерн для URL
const comprehensiveUrlPattern = /(?:https?|ftp):\/\/[\n\S]+?(?=\s|[^\n\S]|$)/gi;

const complexText = 'Сайт: https://example.com, Email: mailto:test@example.com, и ftp://files.example.com';
console.log(complexText.match(comprehensiveUrlPattern)); // ['https://example.com', 'ftp://files.example.com']
```

## Комплексный парсер социальных элементов

```javascript
// Класс для извлечения всех социальных элементов
class SocialMediaParser {
  constructor() {
    this.patterns = {
      hashtag: /#[a-zA-Zа-яёА-ЯЁ0-9_]+/g,
      mention: /@[a-zA-Z0-9_]{1,30}(?!\w)/g,
      url: /https?:\/\/(?:[-\w.])+(?:\:[0-9]+)?(?:\/(?:[\w\/_.])*(?:\?(?:[\w&=%.])*)?(?:\#(?:[\w.])*)?)?/g,
      email: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g
    };
  }
  
  parse(text) {
    const result = {
      hashtags: text.match(this.patterns.hashtag) || [],
      mentions: text.match(this.patterns.mention) || [],
      urls: text.match(this.patterns.url) || [],
      emails: text.match(this.patterns.email) || [],
      all: []
    };
    
    // Создаем массив всех элементов с позициями
    for (const [type, pattern] of Object.entries(this.patterns)) {
      let match;
      while ((match = pattern.exec(text)) !== null) {
        result.all.push({
          type,
          value: match[0],
          start: match.index,
          end: match.index + match[0].length
        });
      }
      // Сбрасываем lastIndex для следующего использования
      pattern.lastIndex = 0;
    }
    
    // Сортируем по позиции в тексте
    result.all.sort((a, b) => a.start - b.start);
    
    return result;
  }
  
  // Заменить элементы в тексте
  replaceElements(text, replacements = {}) {
    // Сначала заменяем URL, чтобы не испортить другие паттерны
    if (replacements.urls !== undefined) {
      text = text.replace(this.patterns.url, replacements.urls);
    }
    
    // Затем хэштеги
    if (replacements.hashtags !== undefined) {
      text = text.replace(this.patterns.hashtag, replacements.hashtags);
    }
    
    // Затем упоминания
    if (replacements.mentions !== undefined) {
      text = text.replace(this.patterns.mention, replacements.mentions);
    }
    
    // И email
    if (replacements.emails !== undefined) {
      text = text.replace(this.patterns.email, replacements.emails);
    }
    
    return text;
  }
  
  // Очистить текст от социальных элементов
  cleanText(text) {
    return this.replaceElements(text, '');
  }
}

// Пример использования
const parser = new SocialMediaParser();
const socialText = 'Привет @user1, смотри #JavaScript статью на https://example.com/article #coding @user2 отправил email на admin@example.com';

const parsed = parser.parse(socialText);
console.log(parsed);
// {
//   hashtags: ['#JavaScript', '#coding'],
//   mentions: ['@user1', '@user2'],
//   urls: ['https://example.com/article'],
//   emails: ['admin@example.com'],
//   all: [...]
// }

console.log('Очищенный текст:', parser.cleanText(socialText));
// Привет , смотри  статью на  отправил email на 
```

## Обработка хэштегов с учетом регистра и дубликатов

```javascript
// Функция для нормализации и извлечения уникальных хэштегов
function extractUniqueHashtags(text, options = {}) {
  const {
    caseSensitive = false,
    minLength = 2,
    maxLength = 50
  } = options;
  
  const hashtagPattern = /#[a-zA-Zа-яёА-ЯЁ0-9_]+/g;
  const matches = text.match(hashtagPattern) || [];
  
  // Удаляем символ # и фильтруем по длине
  let hashtags = matches
    .map(tag => tag.substring(1)) // удаляем #
    .filter(tag => tag.length >= minLength && tag.length <= maxLength);
  
  // Если не чувствительны к регистру, приводим к одному регистру
  if (!caseSensitive) {
    hashtags = hashtags.map(tag => tag.toLowerCase());
  }
  
  // Удаляем дубликаты
  return [...new Set(hashtags)];
}

const text = '#JavaScript #javascript #JS #Frontend #frontend #JavaScript';
console.log(extractUniqueHashtags(text)); // ['javascript', 'js', 'frontend']
console.log(extractUniqueHashtags(text, { caseSensitive: true })); // ['JavaScript', 'javascript', 'JS', 'Frontend', 'frontend']
```

## Валидация хэштегов

```javascript
// Класс для валидации хэштегов
class HashtagValidator {
  static validate(tag, options = {}) {
    const {
      minLength = 1,
      maxLength = 50,
      allowUnicode = true,
      allowNumbers = true,
      allowUnderscores = true
    } = options;
    
    // Удаляем # если присутствует
    if (tag.startsWith('#')) {
      tag = tag.substring(1);
    }
    
    // Проверяем длину
    if (tag.length < minLength || tag.length > maxLength) {
      return { valid: false, error: `Длина хэштега должна быть от ${minLength} до ${maxLength} символов` };
    }
    
    // Составляем паттерн валидации
    let pattern = '^[';
    
    if (allowUnicode) {
      pattern += 'a-zA-Zа-яёА-ЯЁ';
    } else {
      pattern += 'a-zA-Z';
    }
    
    if (allowNumbers) {
      pattern += '0-9';
    }
    
    if (allowUnderscores) {
      pattern += '_';
    }
    
    pattern += ']+$';
    
    const regex = new RegExp(pattern);
    
    if (!regex.test(tag)) {
      return { valid: false, error: 'Хэштег содержит недопустимые символы' };
    }
    
    return { valid: true, error: null };
  }
  
  static validateMultiple(tags, options) {
    return tags.map(tag => ({
      tag,
      ...this.validate(tag, options)
    }));
  }
}

console.log(HashtagValidator.validate('#JavaScript')); // { valid: true, error: null }
console.log(HashtagValidator.validate('#bad-tag!', { allowUnderscores: false })); // { valid: false, error: 'Хэштег содержит недопустимые символы' }
console.log(HashtagValidator.validate('#a')); // { valid: false, error: 'Длина хэштега должна быть от 1 до 50 символов' }
```

## Обработка упоминаний с проверкой валидности

```javascript
// Класс для обработки упоминаний
class MentionProcessor {
  static extractAndValidate(text, validatorFn = null) {
    const mentionPattern = /@[a-zA-Z0-9_]{1,30}(?!\w)/g;
    const matches = text.match(mentionPattern) || [];
    
    return matches.map(mention => {
      const username = mention.substring(1); // удаляем @
      
      const result = {
        full: mention,
        username: username,
        isValid: this.validateUsername(username)
      };
      
      // Если предоставлена пользовательская функция валидации
      if (validatorFn && typeof validatorFn === 'function') {
        result.customValidation = validatorFn(username);
      }
      
      return result;
    });
  }
  
  static validateUsername(username) {
    // Базовая валидация: 1-30 символов, только буквы, цифры и подчеркивания
    const pattern = /^[a-zA-Z0-9_]{1,30}$/;
    return pattern.test(username);
  }
  
  static normalizeUsername(username) {
    // Приведение к нижнему регистру и удаление лишних символов
    return username.toLowerCase().replace(/[^a-zA-Z0-9_]/g, '');
  }
}

const mentionText = 'Сообщение для @valid_user и @invalid-user!';
const processedMentions = MentionProcessor.extractAndValidate(mentionText);
console.log(processedMentions);
// [
//   { full: '@valid_user', username: 'valid_user', isValid: true },
//   { full: '@invalid-user', username: 'invalid-user', isValid: false }
// ]
```

## Извлечение и валидация ссылок

```javascript
// Класс для извлечения и валидации URL
class UrlExtractor {
  static extractUrls(text) {
    // Более полный паттерн для URL
    const urlPattern = /(?:https?|ftp):\/\/[^\s/$.?#].[^\s]*/gi;
    return text.match(urlPattern) || [];
  }
  
  static isValidUrl(string) {
    try {
      const url = new URL(string);
      return ['http:', 'https:', 'ftp:'].includes(url.protocol);
    } catch (_) {
      return false;
    }
  }
  
  static extractAndValidateUrls(text) {
    const urls = this.extractUrls(text);
    
    return urls.map(url => ({
      url,
      isValid: this.isValidUrl(url),
      protocol: new URL(url).protocol.replace(':', ''),
      hostname: new URL(url).hostname
    }));
  }
  
  static sanitizeUrls(text) {
    const urls = this.extractUrls(text);
    
    return urls.reduce((result, url) => {
      if (this.isValidUrl(url)) {
        return result; // Оставляем валидные URL
      } else {
        // Заменяем невалидные URL на пустую строку или безопасный вариант
        return result.replace(url, '');
      }
    }, text);
  }
}

const textWithUrls = 'Полезные ресурсы: https://example.com и http://invalid.url и https://good-site.org/page';
const extractedUrls = UrlExtractor.extractAndValidateUrls(textWithUrls);
console.log(extractedUrls);
// [
//   { url: 'https://example.com', isValid: true, protocol: 'https', hostname: 'example.com' },
//   { url: 'http://invalid.url', isValid: true, protocol: 'http', hostname: 'invalid.url' },
//   { url: 'https://good-site.org/page', isValid: true, protocol: 'https', hostname: 'good-site.org' }
// ]
```

## Комплексная обработка социальных медиа текста

```javascript
// Комплексный класс для обработки текста социальных медиа
class SocialTextProcessor {
  constructor() {
    this.parser = new SocialMediaParser();
  }
  
  process(text, options = {}) {
    const {
      extractOnly = false,
      validate = true,
      transform = null
    } = options;
    
    // Извлекаем все элементы
    const elements = this.parser.parse(text);
    
    if (extractOnly) {
      return elements;
    }
    
    // Валидация (если требуется)
    if (validate) {
      elements.hashtags = elements.hashtags.filter(tag => 
        HashtagValidator.validate(tag).valid
      );
      
      elements.mentions = elements.mentions.filter(mention => 
        MentionProcessor.validateUsername(mention.substring(1))
      );
      
      elements.urls = elements.urls.filter(url => 
        UrlExtractor.isValidUrl(url)
      );
    }
    
    // Трансформация (если требуется)
    if (transform && typeof transform === 'function') {
      text = transform(text, elements);
    }
    
    return {
      originalText: text,
      elements,
      cleanText: this.parser.cleanText(text)
    };
  }
  
  // Создание HTML-ссылок из социальных элементов
  createHtmlLinks(text) {
    // Заменяем URL на HTML-ссылки
    text = text.replace(
      /https?:\/\/[^\s/$.?#].[^\s]*/gi,
      url => `<a href="${url}" target="_blank">${url}</a>`
    );
    
    // Заменяем хэштеги на ссылки
    text = text.replace(
      /#[a-zA-Zа-яёА-ЯЁ0-9_]+/g,
      tag => `<a href="/search?q=${encodeURIComponent(tag)}">${tag}</a>`
    );
    
    // Заменяем упоминания на ссылки
    text = text.replace(
      /@[a-zA-Z0-9_]{1,30}(?!\w)/g,
      mention => `<a href="/user/${mention.substring(1)}">${mention}</a>`
    );
    
    return text;
  }
}

// Пример использования
const processor = new SocialTextProcessor();
const socialPost = 'Изучаю #JavaScript с @teacher! Ресурсы: https://mdn.com и https://javascript.info';

const result = processor.process(socialPost);
console.log(result);
// {
//   originalText: 'Изучаю #JavaScript с @teacher! Ресурсы: https://mdn.com и https://javascript.info',
//   elements: { hashtags: ['#JavaScript'], mentions: ['@teacher'], urls: ['https://mdn.com', 'https://javascript.info'], emails: [], all: [...] },
//   cleanText: 'Изучаю  с ! Ресурсы:  и '
// }

console.log(processor.createHtmlLinks(socialPost));
// Изучаю <a href="/search?q=%23JavaScript">#JavaScript</a> с <a href="/user/teacher">@teacher</a>! Ресурсы: <a href="https://mdn.com" target="_blank">https://mdn.com</a> и <a href="https://javascript.info" target="_blank">https://javascript.info</a>
```

## Заключение

Эффективное извлечение и обработка хэштегов, упоминаний и ссылок позволяет создавать богатые интерактивные интерфейсы для социальных приложений. Правильная валидация и нормализация этих элементов улучшает пользовательский опыт и безопасность приложения.

## См. также

- [[Text Processing Patterns]]
- [[Input Sanitization Techniques]]
- [[Social Media APIs]]