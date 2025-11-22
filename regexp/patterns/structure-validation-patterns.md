---
aliases: [JSON Validation, XML Validation, Structure Validation]
tags: [regexp, json, xml, validation, patterns]
---

# Паттерны для валидации JSON и XML структур

## Обзор

Хотя регулярные выражения не являются идеальным инструментом для парсинга JSON и XML (для этого лучше использовать специализированные парсеры), они могут быть полезны для базовой валидации, извлечения данных и проверки структуры. В этой статье рассмотрим паттерны для работы с этими форматами.

## Базовая валидация JSON

```javascript
// Простая проверка, является ли строка JSON (начинается и заканчивается правильными символами)
function isPossibleJson(str) {
  const trimmed = str.trim();
  return (trimmed.startsWith('{') && trimmed.endsWith('}')) || 
         (trimmed.startsWith('[') && trimmed.endsWith(']'));
}

console.log(isPossibleJson('{"name": "John"}')); // true
console.log(isPossibleJson('[1, 2, 3]')); // true
console.log(isPossibleJson('not json')); // false

// Более строгая проверка JSON с помощью регулярного выражения
function isValidJsonStructure(str) {
  try {
    // Сначала проверяем структуру с помощью регулярного выражения
    const jsonPattern = /^\s*(\{.*\}|\[.*\])\s*$/s;
    if (!jsonPattern.test(str)) {
      return false;
    }
    
    // Затем используем JSON.parse для окончательной проверки
    JSON.parse(str);
    return true;
  } catch (e) {
    return false;
  }
}

console.log(isValidJsonStructure('{"name": "John", "age": 30}')); // true
console.log(isValidJsonStructure('{"name": "John", "age":}')); // false
```

## Извлечение значений из JSON с помощью регулярных выражений

```javascript
// Функция для извлечения значения по ключу из JSON-строки
function extractJsonValue(jsonStr, key) {
  // Паттерн для извлечения значения по ключу
  // Работает с простыми значениями (строки, числа, булевы значения)
  const pattern = new RegExp(`"${key}"\\s*:\\s*("[^"]*"|-?\\d+(?:\\.\\d+)?|true|false|null)`, 'i');
  const match = jsonStr.match(pattern);
  
  if (match) {
    let value = match[1];
    
    // Удаляем кавычки, если это строка
    if (value.startsWith('"') && value.endsWith('"')) {
      value = value.slice(1, -1);
    }
    
    return value;
  }
  
  return undefined;
}

const jsonString = '{"name": "John", "age": 30, "active": true, "score": 95.5}';
console.log(extractJsonValue(jsonString, 'name')); // John
console.log(extractJsonValue(jsonString, 'age')); // 30
console.log(extractJsonValue(jsonString, 'active')); // true
console.log(extractJsonValue(jsonString, 'score')); // 95.5
```

## Извлечение всех пар ключ-значение

```javascript
// Функция для извлечения всех пар ключ-значение из JSON
function extractAllKeyValuePairs(jsonStr) {
  // Паттерн для извлечения всех пар ключ-значение
  const kvPattern = /"([^"]+)"\s*:\s*("[^"]*"|-?\d+(?:\.\d+)?|true|false|null)/g;
  const pairs = {};
  let match;
  
  while ((match = kvPattern.exec(jsonStr)) !== null) {
    let value = match[2];
    
    // Удаляем кавычки у строковых значений
    if (value.startsWith('"') && value.endsWith('"')) {
      value = value.slice(1, -1);
    }
    
    pairs[match[1]] = value;
  }
  
  return pairs;
}

const complexJson = '{"user": "john", "role": "admin", "count": 42, "verified": true}';
console.log(extractAllKeyValuePairs(complexJson));
// { user: 'john', role: 'admin', count: '42', verified: 'true' }
```

## Извлечение вложенных объектов

```javascript
// Функция для извлечения вложенных JSON-объектов
function extractNestedObjects(jsonStr) {
  const results = [];
  let braceCount = 0;
  let start = -1;
  
  for (let i = 0; i < jsonStr.length; i++) {
    if (jsonStr[i] === '{') {
      if (braceCount === 0) {
        start = i;
      }
      braceCount++;
    } else if (jsonStr[i] === '}') {
      braceCount--;
      if (braceCount === 0 && start !== -1) {
        results.push(jsonStr.substring(start, i + 1));
      }
    }
  }
  
  return results;
}

const nestedJson = '{"user": {"name": "John", "details": {"age": 30}}, "config": {"debug": true}}';
console.log(extractNestedObjects(nestedJson));
// ['{"name": "John", "details": {"age": 30}}', '{"age": 30}', '{"debug": true}']
```

## Валидация XML-структуры

```javascript
// Простая проверка XML-структуры
function isPossibleXml(str) {
  // Проверяем, начинается ли строка с XML объявления или открывающего тега
  const xmlStartPattern = /<\?xml[^>]*\?>|<\w+/i;
  return xmlStartPattern.test(str);
}

console.log(isPossibleXml('<?xml version="1.0"?><root></root>')); // true
console.log(isPossibleXml('<data><item>value</item></data>')); // true
console.log(isPossibleXml('not xml')); // false

// Проверка на основную структуру XML (упрощенная)
function hasValidXmlStructure(str) {
  // Проверяем наличие парных тегов
  const openingTags = (str.match(/<[^>]*>/g) || []).filter(tag => !tag.startsWith('</') && !tag.endsWith('/>'));
  const closingTags = (str.match(/<\/[^>]*>/g) || []);
  
  // Простая проверка количества открывающих и закрывающих тегов
  return openingTags.length === closingTags.length && openingTags.length > 0;
}

const validXml = '<root><child>data</child></root>';
const invalidXml = '<root><child>data</root>'; // Несоответствие тегов
console.log(hasValidXmlStructure(validXml)); // true
console.log(hasValidXmlStructure(invalidXml)); // false
```

## Извлечение значений из XML

```javascript
// Функция для извлечения значения по тегу из XML
function extractXmlValue(xmlStr, tag) {
  const pattern = new RegExp(`<${tag}[^>]*>(.*?)<\/${tag}>`, 'i');
  const match = xmlStr.match(pattern);
  return match ? match[1] : undefined;
}

const xmlString = '<user><name>John</name><age>30</age><role>admin</role></user>';
console.log(extractXmlValue(xmlString, 'name')); // John
console.log(extractXmlValue(xmlString, 'age')); // 30
console.log(extractXmlValue(xmlString, 'role')); // admin

// Извлечение значения с атрибутами
function extractXmlValueWithAttrs(xmlStr, tag) {
  const pattern = new RegExp(`<${tag}([^>]*)>(.*?)<\/${tag}>`, 'i');
  const match = xmlStr.match(pattern);
  if (match) {
    return {
      value: match[2],
      attributes: parseXmlAttributeString(match[1])
    };
  }
  return undefined;
}

function parseXmlAttributeString(attrStr) {
  if (!attrStr) return {};
  
  const attrs = {};
  const attrPattern = /(\w+)\s*=\s*["']([^"']*)["']/g;
  let match;
  
  while ((match = attrPattern.exec(attrStr)) !== null) {
    attrs[match[1]] = match[2];
  }
  
  return attrs;
}

const xmlWithAttrs = '<user id="123" type="admin"><name>John</name></user>';
console.log(extractXmlValueWithAttrs(xmlWithAttrs, 'user'));
// { value: '<name>John</name>', attributes: { id: '123', type: 'admin' } }
```

## Извлечение всех элементов XML

```javascript
// Функция для извлечения всех элементов XML
function extractAllXmlElements(xmlStr) {
  const elementPattern = /<(\w+)([^>]*)>(.*?)<\/\1>|<(\w+)([^>]*)\s*\/>/gs;
  const elements = [];
  let match;
  
  while ((match = elementPattern.exec(xmlStr)) !== null) {
    if (match[1]) {
      // Открывающий и закрывающий тег
      elements.push({
        tag: match[1],
        attributes: parseXmlAttributeString(match[2]),
        content: match[3],
        selfClosing: false
      });
    } else {
      // Самозакрывающийся тег
      elements.push({
        tag: match[4],
        attributes: parseXmlAttributeString(match[5]),
        content: '',
        selfClosing: true
      });
    }
  }
  
  return elements;
}

const sampleXml = '<root><item id="1">First</item><item id="2">Second</item><br/></root>';
console.log(extractAllXmlElements(sampleXml));
```

## Проверка валидности структуры JSON с помощью регулярных выражений

```javascript
// Класс для проверки структуры JSON
class JsonStructureValidator {
  // Проверка, является ли JSON объектом
  static isJsonObject(jsonStr) {
    const trimmed = jsonStr.trim();
    return trimmed.startsWith('{') && trimmed.endsWith('}');
  }
  
  // Проверка, является ли JSON массивом
  static isJsonArray(jsonStr) {
    const trimmed = jsonStr.trim();
    return trimmed.startsWith('[') && trimmed.endsWith(']');
  }
  
  // Проверка на наличие определенных полей
  static hasRequiredFields(jsonStr, requiredFields) {
    return requiredFields.every(field => {
      const pattern = new RegExp(`"${field}"\\s*:`);
      return pattern.test(jsonStr);
    });
  }
  
  // Подсчет количества пар ключ-значение
  static countKeyValuePairs(jsonStr) {
    const kvPattern = /"([^"]+)"\s*:\s*("[^"]*"|-?\d+(?:\.\d+)?|true|false|null|\{|\[)/g;
    let count = 0;
    let match;
    
    while ((match = kvPattern.exec(jsonStr)) !== null) {
      count++;
    }
    
    return count;
  }
  
  // Проверка на валидность строк JSON
  static validateJsonStrings(jsonStr) {
    // Паттерн для поиска строк в JSON (между двойными кавычками)
    const stringPattern = /"(?:[^"\\]|\\.)*"/g;
    const matches = jsonStr.match(stringPattern) || [];
    
    // Проверяем, все ли строки правильно экранированы
    return matches.every(str => {
      // Убираем внешние кавычки
      const content = str.slice(1, -1);
      // Проверяем экранирование
      return !/(^|[^\\])["]/.test(content) || // Не должно быть неэкранированных кавычек
             /\\["\\/bfnrt]/.test(content) || // Должно быть правильное экранирование
             /\\u[0-9a-fA-F]{4}/.test(content); // Или юникодные экранирования
    });
  }
}

// Примеры использования
const testJson = '{"name": "John", "age": 30, "active": true}';
console.log(JsonStructureValidator.isJsonObject(testJson)); // true
console.log(JsonStructureValidator.hasRequiredFields(testJson, ['name', 'age'])); // true
console.log(JsonStructureValidator.countKeyValuePairs(testJson)); // 3
console.log(JsonStructureValidator.validateJsonStrings(testJson)); // true
```

## Проверка валидности структуры XML с помощью регулярных выражений

```javascript
// Класс для проверки структуры XML
class XmlStructureValidator {
  // Проверка на корректное закрытие тегов
  static hasBalancedTags(xmlStr) {
    const tags = [];
    const tagPattern = /<\s*\/?([a-zA-Z][a-zA-Z0-9]*)[^>]*>/g;
    let match;
    
    while ((match = tagPattern.exec(xmlStr)) !== null) {
      const fullTag = match[0];
      const tagName = match[1];
      
      if (fullTag.startsWith('</')) {
        // Закрывающий тег
        if (tags.length === 0 || tags.pop() !== tagName) {
          return false;
        }
      } else if (!fullTag.endsWith('/>')) {
        // Открывающий тег (не самозакрывающийся)
        tags.push(tagName);
      }
    }
    
    return tags.length === 0;
  }
  
  // Проверка на наличие корневого элемента
  static hasRootElement(xmlStr) {
    const rootPattern = /<([a-zA-Z][a-zA-Z0-9]*)[^>]*>[\s\S]*<\/\1>/;
    return rootPattern.test(xmlStr);
  }
  
  // Проверка на корректное экранирование специальных символов
  static hasValidEscaping(xmlStr) {
    // Проверяем, что &, <, > правильно экранированы
    const unescapedPattern = /(^|[^&])<|(^|[^&])>|&(?!(?:amp|lt|gt|quot|apos|#x?[0-9a-fA-F]+);)/;
    return !unescapedPattern.test(xmlStr);
  }
  
  // Проверка на корректность атрибутов
  static hasValidAttributes(xmlStr) {
    const attrPattern = /<[^>]*\s+[^>]*=[^>]*["'][^"']*["'][^>]*>/g;
    const matches = xmlStr.match(attrPattern) || [];
    
    return matches.every(tag => {
      // Проверяем, что атрибуты имеют правильный формат
      const attrs = tag.match(/[^<]*\s+([^>]*=[^>]*["'][^"']*["'])/g);
      return !attrs || attrs.every(attr => /=["'][^"']*["']/.test(attr));
    });
  }
}

// Примеры использования
const validXmlExample = '<root><child attr="value">data &amp; more</child></root>';
const invalidXmlExample = '<root><child>data</root></child>'; // Несбалансированные теги

console.log(XmlStructureValidator.hasBalancedTags(validXmlExample)); // true
console.log(XmlStructureValidator.hasBalancedTags(invalidXmlExample)); // false
console.log(XmlStructureValidator.hasRootElement(validXmlExample)); // true
console.log(XmlStructureValidator.hasValidEscaping(validXmlExample)); // true
```

## Практический пример: валидатор API ответов

```javascript
// Комплексный класс для валидации API ответов (JSON и XML)
class ApiResponseValidator {
  constructor() {
    this.jsonValidator = JsonStructureValidator;
    this.xmlValidator = XmlStructureValidator;
  }
  
  validateResponse(response, expectedFormat, requiredFields = []) {
    const result = {
      valid: false,
      format: expectedFormat,
      errors: []
    };
    
    if (expectedFormat === 'json') {
      if (!this.jsonValidator.isJsonObject(response) && !this.jsonValidator.isJsonArray(response)) {
        result.errors.push('Ответ не является валидным JSON');
        return result;
      }
      
      if (requiredFields.length > 0 && !this.jsonValidator.hasRequiredFields(response, requiredFields)) {
        result.errors.push(`Отсутствуют обязательные поля: ${requiredFields.join(', ')}`);
      }
      
      if (!this.jsonValidator.validateJsonStrings(response)) {
        result.errors.push('Некорректное экранирование строк в JSON');
      }
      
      result.valid = result.errors.length === 0;
    } else if (expectedFormat === 'xml') {
      if (!isPossibleXml(response)) {
        result.errors.push('Ответ не является XML');
        return result;
      }
      
      if (!this.xmlValidator.hasRootElement(response)) {
        result.errors.push('XML не имеет корневого элемента');
      }
      
      if (!this.xmlValidator.hasBalancedTags(response)) {
        result.errors.push('Несбалансированные XML теги');
      }
      
      if (!this.xmlValidator.hasValidEscaping(response)) {
        result.errors.push('Некорректное экранирование специальных символов');
      }
      
      result.valid = result.errors.length === 0;
    } else {
      result.errors.push(`Неподдерживаемый формат: ${expectedFormat}`);
    }
    
    return result;
  }
  
  // Извлечение данных из ответа
  extractData(response, format, path) {
    if (format === 'json') {
      try {
        const parsed = JSON.parse(response);
        return this.getNestedValue(parsed, path);
      } catch (e) {
        // Альтернативный метод извлечения через регулярные выражения
        return extractJsonValue(response, path);
      }
    } else if (format === 'xml') {
      return extractXmlValue(response, path);
    }
    
    return undefined;
  }
  
  getNestedValue(obj, path) {
    return path.split('.').reduce((current, key) => current && current[key], obj);
  }
}

// Пример использования
const validator = new ApiResponseValidator();

const jsonResponse = '{"status": "success", "data": {"user": "john", "id": 123}}';
const xmlResponse = '<response><status>success</status><data><user>john</user><id>123</id></data></response>';

console.log(validator.validateResponse(jsonResponse, 'json', ['status', 'data'])); // { valid: true, ... }
console.log(validator.validateResponse(xmlResponse, 'xml')); // { valid: true, ... }

console.log(validator.extractData(jsonResponse, 'json', 'data.user')); // john
console.log(validator.extractData(xmlResponse, 'xml', 'data.user')); // john
```

## Заключение

Хотя регулярные выражения не заменяют полноценные парсеры JSON и XML, они могут быть полезны для базовой валидации, извлечения данных и проверки структуры. Для критически важных приложений рекомендуется использовать специализированные парсеры, но регулярные выражения остаются полезным инструментом для промежуточной проверки и обработки.

## См. также

- [[JSON Parsing Best Practices]]
- [[XML Processing Techniques]]
- [[API Response Validation]]