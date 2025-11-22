---
aliases: [HTML Processing, Tag Extraction, Attribute Parsing]
tags: [regexp, html, frontend, parsing]
---

# Обработка HTML тегов и атрибутов

## Обзор

Регулярные выражения часто используются для обработки HTML-тегов и извлечения их атрибутов. В этой статье рассмотрим различные подходы к работе с HTML-тегами с помощью регулярных выражений.

> [!warning] 
> Важно понимать, что регулярные выражения не подходят для полного парсинга HTML. Для сложных задач лучше использовать специализированные HTML-парсеры. Однако для простых задач извлечения данных регулярные выражения могут быть эффективны.

## Извлечение тегов без атрибутов

```javascript
// Простое извлечение тегов без атрибутов
const html = '<p>Текст параграфа</p><div>Содержимое div</div>';
const simpleTagRegex = /<(\w+)>(.*?)<\/\1>/g;

let match;
while ((match = simpleTagRegex.exec(html)) !== null) {
  console.log(`Тег: ${match[1]}, Содержимое: ${match[2]}`);
}
// Вывод:
// Тег: p, Содержимое: Текст параграфа
// Тег: div, Содержимое: Содержимое div
```

## Извлечение тегов с атрибутами

```javascript
// Извлечение тегов с атрибутами
const htmlWithAttrs = '<div class="container" id="main">Контент</div><img src="image.jpg" alt="Описание">';
const tagWithAttrsRegex = /<(\w+)([^>]*?)>(.*?)<\/\1>|<(\w+)([^>]*?)\s*\/>/gs;

let match;
while ((match = tagWithAttrsRegex.exec(htmlWithAttrs)) !== null) {
  if (match[1]) {
    // Открывающий и закрывающий тег
    console.log(`Тег: ${match[1]}, Атрибуты: ${match[2]}, Содержимое: ${match[3]}`);
  } else {
    // Одиночный тег (например, img)
    console.log(`Одиночный тег: ${match[4]}, Атрибуты: ${match[5]}`);
  }
}
```

## Извлечение конкретных атрибутов

```javascript
// Извлечение конкретного атрибута (например, src из img)
const imgSrcRegex = /<img[^>]*src\s*=\s*["']([^"']*)["'][^>]*>/gi;

const htmlImages = '<img src="image1.jpg" alt="Первое изображение"><img src="image2.png" class="thumbnail">';
const sources = htmlImages.match(imgSrcRegex).map(img => {
  const match = img.match(/src\s*=\s*["']([^"']*)["']/i);
  return match ? match[1] : null;
});

console.log(sources); // ['image1.jpg', 'image2.png']
```

## Извлечение всех атрибутов тега

```javascript
// Извлечение всех атрибутов тега
function extractAttributes(tag) {
  const attrRegex = /(\w+)\s*=\s*["']([^"']*)["']/g;
  const attributes = {};
  let attrMatch;
  
  while ((attrMatch = attrRegex.exec(tag)) !== null) {
    attributes[attrMatch[1]] = attrMatch[2];
  }
  
  return attributes;
}

const tag = '<div class="container" id="main" data-value="123">';
console.log(extractAttributes(tag)); 
// { class: 'container', id: 'main', 'data-value': '123' }
```

## Удаление HTML-тегов

```javascript
// Удаление всех HTML-тегов
function stripHtmlTags(html) {
  return html.replace(/<[^>]*>/g, '');
}

const htmlWithTags = '<p>Это <strong>жирный</strong> текст</p>';
console.log(stripHtmlTags(htmlWithTags)); // Это жирный текст
```

## Замена тегов с сохранением содержимого

```javascript
// Замена тегов (например, h1 на h2)
function replaceTag(html, oldTag, newTag) {
  const regex = new RegExp(`<${oldTag}([^>]*?)>(.*?)<\/${oldTag}>`, 'gi');
  return html.replace(regex, `<${newTag}$1>$2</${newTag}>`);
}

const originalHtml = '<h1 class="title">Заголовок</h1>';
const newHtml = replaceTag(originalHtml, 'h1', 'h2');
console.log(newHtml); // <h2 class="title">Заголовок</h2>
```

## Проверка корректности HTML-тегов

```javascript
// Проверка корректности HTML-тегов (упрощенная проверка)
function isValidHtml(html) {
  // Проверяем наличие парных тегов
  const openingTags = html.match(/<([a-zA-Z][a-zA-Z0-9]*)[^>]*>/g) || [];
  const closingTags = html.match(/<\/([a-zA-Z][a-zA-Z0-9]*)>/g) || [];
  
  // Простая проверка количества открытия и закрытия тегов
  const openTagNames = openingTags.map(tag => tag.match(/<([a-zA-Z][a-zA-Z0-9]*)/)[1]);
  const closeTagNames = closingTags.map(tag => tag.match(/<\/([a-zA-Z][a-zA-Z0-9]*)/)[1]);
  
  const tagCount = {};
  
  openTagNames.forEach(tag => {
    tagCount[tag] = (tagCount[tag] || 0) + 1;
  });
  
  closeTagNames.forEach(tag => {
    tagCount[tag] = (tagCount[tag] || 0) - 1;
  });
  
  // Проверяем, что все теги имеют пару
  return Object.values(tagCount).every(count => count === 0);
}

console.log(isValidHtml('<div><p>Текст</p></div>')); // true
console.log(isValidHtml('<div><p>Текст</div>')); // false
```

## Извлечение ссылок из HTML

```javascript
// Извлечение всех ссылок из HTML
function extractLinks(html) {
  const linkRegex = /<a[^>]*href\s*=\s*["']([^"']*)["'][^>]*>(.*?)<\/a>/gi;
  const links = [];
  let match;
  
  while ((match = linkRegex.exec(html)) !== null) {
    links.push({
      url: match[1],
      text: match[2].replace(/<[^>]*>/g, '') // удаляем вложенные теги из текста
    });
  }
  
  return links;
}

const htmlWithLinks = '<p>Посетите <a href="https://example.com">наш сайт</a> или <a href="https://github.com">GitHub</a></p>';
console.log(extractLinks(htmlWithLinks));
// [
//   { url: 'https://example.com', text: 'наш сайт' },
//   { url: 'https://github.com', text: 'GitHub' }
// ]
```

## Заключение

Регулярные выражения могут быть полезны для простой обработки HTML-тегов, но помните, что они не заменяют полноценные HTML-парсеры. Используйте их с осторожностью и только для простых задач извлечения данных.

## См. также

- [[HTML Parsing Best Practices]]
- [[Advanced Regexp Patterns]]
- [[RegExp Performance Tips]]