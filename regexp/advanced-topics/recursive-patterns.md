---
aliases: [Recursive Patterns, Nested Structures, Complex Parsing]
tags: [regexp, advanced, recursion, parsing]
---

# Рекурсивные паттерны и балансировка скобок

## Обзор

Рекурсивные паттерны позволяют обрабатывать вложенные структуры, такие как скобки, теги или JSON-объекты. В этой статье рассмотрим, как использовать рекурсивные регулярные выражения и техники балансировки скобок.

## Базовая балансировка скобок

```javascript
// Простая проверка баланса круглых скобок
function isBalancedBrackets(str) {
  const stack = [];
  for (let i = 0; i < str.length; i++) {
    if (str[i] === '(') {
      stack.push(str[i]);
    } else if (str[i] === ')') {
      if (stack.length === 0) {
        return false;
      }
      stack.pop();
    }
  }
  return stack.length === 0;
}

console.log(isBalancedBrackets('(())')); // true
console.log(isBalancedBrackets('(()')); // false
console.log(isBalancedBrackets('())')); // false
```

## Использование регулярных выражений для извлечения сбалансированных скобок

```javascript
// Функция для извлечения содержимого в сбалансированных скобках
function extractBalancedParens(str) {
  // Регулярное выражение для извлечения внешней пары скобок и их содержимого
  const regex = /\((?:[^()]*|\([^()]*\))*\)/g;
  return str.match(regex) || [];
}

const text = 'Функция (внутри (еще одни) скобки) и (другая (вложенная) часть)';
console.log(extractBalancedParens(text)); // ['(внутри (еще одни) скобки)', '(другая (вложенная) часть)']
```

## Рекурсивное регулярное выражение для вложенных структур (JavaScript не поддерживает рекурсивные регулярные выражения нативно)

```javascript
// В JavaScript нет встроенной поддержки рекурсивных регулярных выражений,
// но мы можем использовать подход с итеративной обработкой
function findNestedParens(text) {
  const results = [];
  let depth = 0;
  let start = -1;
  
  for (let i = 0; i < text.length; i++) {
    if (text[i] === '(') {
      if (depth === 0) {
        start = i;
      }
      depth++;
    } else if (text[i] === ')') {
      depth--;
      if (depth === 0 && start !== -1) {
        results.push(text.substring(start, i + 1));
      }
    }
  }
  
  return results;
}

const nestedText = 'Математическое выражение: ((a + b) * (c - d)) и (x + y)';
console.log(findNestedParens(nestedText)); // ['((a + b) * (c - d))', '(x + y)']
```

## Балансировка разных типов скобок

```javascript
// Функция для проверки баланса разных типов скобок
function isBalancedMultipleBrackets(str) {
  const stack = [];
  const pairs = {
    ')': '(',
    ']': '[',
    '}': '{'
  };
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (['(', '[', '{'].includes(char)) {
      stack.push(char);
    } else if ([')', ']', '}'].includes(char)) {
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    }
  }
  
  return stack.length === 0;
}

console.log(isBalancedMultipleBrackets('{[()]}')); // true
console.log(isBalancedMultipleBrackets('{[(])}')); // false
console.log(isBalancedMultipleBrackets('{{[[(())]]}}')); // true
```

## Извлечение вложенных JSON-подобных структур

```javascript
// Функция для извлечения вложенных JSON-подобных объектов
function extractNestedObjects(text) {
  const results = [];
  let braceCount = 0;
  let start = -1;
  
  for (let i = 0; i < text.length; i++) {
    if (text[i] === '{') {
      if (braceCount === 0) {
        start = i;
      }
      braceCount++;
    } else if (text[i] === '}') {
      braceCount--;
      if (braceCount === 0 && start !== -1) {
        results.push(text.substring(start, i + 1));
      }
    }
  }
  
  return results;
}

const jsonText = 'Данные: {"user": {"name": "John", "details": {"age": 30}}} и {"config": {"debug": true}}';
console.log(extractNestedObjects(jsonText));
// ['{"user": {"name": "John", "details": {"age": 30}}}', '{"config": {"debug": true}}']
```

## Обработка вложенных HTML-тегов

```javascript
// Функция для обработки вложенных HTML-тегов
function extractNestedTags(html, tagName) {
  const tagPattern = new RegExp(`<${tagName}[^>]*>`, 'gi');
  const closingTagPattern = new RegExp(`</${tagName}>`, 'gi');
  
  const results = [];
  let tagMatches = [...html.matchAll(tagPattern)];
  let closingMatches = [...html.matchAll(closingTagPattern)];
  
  // Простой способ - нахождение пар тегов
  let depth = 0;
  let start = -1;
  
  for (let i = 0; i < html.length; i++) {
    if (html.substr(i, tagName.length + 2) === `<${tagName}`) {
      if (depth === 0) {
        start = i;
      }
      depth++;
      // Пропускаем до конца открывающего тега
      while (i < html.length && html[i] !== '>') i++;
    } else if (html.substr(i, tagName.length + 3) === `</${tagName}>`) {
      depth--;
      if (depth === 0 && start !== -1) {
        results.push(html.substring(start, i + tagName.length + 3));
      }
    }
  }
  
  return results;
}

const htmlWithNested = '<div>Внешний <div>вложенный <div>еще один</div> контент</div> конец</div>';
console.log(extractNestedTags(htmlWithNested, 'div'));
```

## Рекурсивный парсер выражений

```javascript
// Пример рекурсивного парсера математических выражений
class ExpressionParser {
  constructor(expression) {
    this.expression = expression;
    this.index = 0;
  }
  
  parse() {
    return this.parseExpression();
  }
  
  parseExpression() {
    let result = this.parseTerm();
    
    while (this.index < this.expression.length) {
      const op = this.expression[this.index];
      
      if (op === '+') {
        this.index++;
        result += this.parseTerm();
      } else if (op === '-') {
        this.index++;
        result -= this.parseTerm();
      } else {
        break;
      }
    }
    
    return result;
  }
  
  parseTerm() {
    let result = this.parseFactor();
    
    while (this.index < this.expression.length) {
      const op = this.expression[this.index];
      
      if (op === '*') {
        this.index++;
        result *= this.parseFactor();
      } else if (op === '/') {
        this.index++;
        result /= this.parseFactor();
      } else {
        break;
      }
    }
    
    return result;
  }
  
  parseFactor() {
    if (this.expression[this.index] === '(') {
      this.index++; // пропускаем '('
      const result = this.parseExpression();
      this.index++; // пропускаем ')'
      return result;
    } else {
      // парсим число
      let numStr = '';
      while (this.index < this.expression.length && 
             /\d|\./.test(this.expression[this.index])) {
        numStr += this.expression[this.index];
        this.index++;
      }
      return parseFloat(numStr);
    }
  }
}

// Пример использования
const parser = new ExpressionParser('(3 + 5) * (2 - 1)');
console.log(parser.parse()); // 8
```

## Использование библиотек для сложных рекурсивных задач

```javascript
// В реальных приложениях рекомендуется использовать специализированные библиотеки
// Ниже пример использования PEG.js стиля (псевдокод)
function parseWithGrammar(input) {
  // В реальном приложении вы бы использовали библиотеку типа PEG.js, nearley.js и т.д.
  // или написали бы рекурсивный спуск вручную
  
  // Пример: парсинг арифметических выражений с рекурсией
  function parseExpression(expr, pos = 0) {
    return parseAddition(expr, pos);
  }
  
  function parseAddition(expr, pos) {
    let [left, newPos] = parseMultiplication(expr, pos);
    
    while (newPos < expr.length) {
      if (expr[newPos] === '+') {
        const [right, finalPos] = parseMultiplication(expr, newPos + 1);
        left = { type: 'add', left, right };
        newPos = finalPos;
      } else if (expr[newPos] === '-') {
        const [right, finalPos] = parseMultiplication(expr, newPos + 1);
        left = { type: 'subtract', left, right };
        newPos = finalPos;
      } else {
        break;
      }
    }
    
    return [left, newPos];
  }
  
  function parseMultiplication(expr, pos) {
    let [left, newPos] = parsePrimary(expr, pos);
    
    while (newPos < expr.length) {
      if (expr[newPos] === '*') {
        const [right, finalPos] = parsePrimary(expr, newPos + 1);
        left = { type: 'multiply', left, right };
        newPos = finalPos;
      } else if (expr[newPos] === '/') {
        const [right, finalPos] = parsePrimary(expr, newPos + 1);
        left = { type: 'divide', left, right };
        newPos = finalPos;
      } else {
        break;
      }
    }
    
    return [left, newPos];
  }
  
  function parsePrimary(expr, pos) {
    if (expr[pos] === '(') {
      const [result, endPos] = parseExpression(expr, pos + 1);
      return [result, endPos + 1]; // +1 для ')'
    } else {
      // парсим число
      let numStr = '';
      let newPos = pos;
      while (newPos < expr.length && /\d|\./.test(expr[newPos])) {
        numStr += expr[newPos];
        newPos++;
      }
      return [{ type: 'number', value: parseFloat(numStr) }, newPos];
    }
  }
  
  return parseExpression(input)[0];
}

// Пример использования
const ast = parseWithGrammar('(1 + 2) * (3 + 4)');
console.log(JSON.stringify(ast, null, 2));
```

## Заключение

Хотя JavaScript не поддерживает рекурсивные регулярные выражения нативно, можно использовать комбинацию регулярных выражений и алгоритмов для обработки вложенных структур. Для сложных задач рекомендуется использовать специализированные парсеры.

## См. также

- [[Advanced Lookahead and Lookbehind]]
- [[RegExp Performance Tips]]
- [[Parser Generators]]