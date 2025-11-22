---
aliases: ["Паттерны для SQL-запросов", "SQL Regex Patterns", "Анализ SQL-запросов"]
tags: [regexp, patterns, sql, database, validation, frontend, security]
---

# Паттерны для извлечения информации из SQL-запросов

## Общие принципы анализа SQL-запросов

Анализ SQL-запросов с помощью регулярных выражений - сложная, но полезная задача. Она может использоваться для логирования, безопасности, оптимизации запросов и аудита. В этой главе мы рассмотрим различные паттерны для извлечения информации из SQL-запросов.

## 1. Извлечение основных компонентов запроса

### Извлечение типа запроса

```javascript
// Извлечение типа SQL-запроса
function getQueryType(sql) {
  const typeRegex = /^\s*(SELECT|INSERT|UPDATE|DELETE|CREATE|ALTER|DROP|WITH)\b/i;
  const match = sql.match(typeRegex);
  return match ? match[1].toUpperCase() : 'UNKNOWN';
}

// Более детальный анализ типа запроса
function analyzeQueryType(sql) {
  sql = sql.trim();
  
  // Определение типа запроса
  if (/^\s*SELECT\b/i.test(sql)) {
    return { type: 'SELECT', subtype: getSelectType(sql) };
  } else if (/^\s*INSERT\b/i.test(sql)) {
    return { type: 'INSERT' };
  } else if (/^\s*UPDATE\b/i.test(sql)) {
    return { type: 'UPDATE' };
  } else if (/^\s*DELETE\b/i.test(sql)) {
    return { type: 'DELETE' };
  } else if (/^\s*(CREATE|ALTER|DROP)\b/i.test(sql)) {
    return { type: 'DDL', subtype: sql.match(/^\s*(CREATE|ALTER|DROP)\b/i)[1] };
  } else if (/^\s*WITH\b/i.test(sql)) {
    return { type: 'CTE' };
  }
  
  return { type: 'UNKNOWN' };
}

function getSelectType(sql) {
  if (/\bLIMIT\b/i.test(sql)) return 'SELECT_LIMITED';
  if (/\bJOIN\b/i.test(sql)) return 'SELECT_WITH_JOIN';
  if (/\bGROUP BY\b/i.test(sql)) return 'SELECT_WITH_GROUP';
  if (/\bUNION\b/i.test(sql)) return 'SELECT_WITH_UNION';
  return 'SELECT_BASIC';
}

// Примеры использования
console.log(getQueryType('SELECT * FROM users WHERE id = 1'));
console.log(getQueryType('INSERT INTO users (name, email) VALUES (?, ?)'));
console.log(analyzeQueryType('SELECT u.name, p.title FROM users u JOIN posts p ON u.id = p.user_id'));
```

### Извлечение таблиц из запроса

```javascript
// Извлечение таблиц из SQL-запроса
function extractTables(sql) {
  const tables = new Set();
  
  // Паттерн для извлечения таблиц из FROM
  const fromRegex = /FROM\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)/gi;
  let match;
  
  while ((match = fromRegex.exec(sql)) !== null) {
    tables.add(match[1]);
  }
  
  // Паттерн для извлечения таблиц из JOIN
  const joinRegex = /\b(JOIN|INNER JOIN|LEFT JOIN|RIGHT JOIN|FULL JOIN)\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)/gi;
  
  while ((match = joinRegex.exec(sql)) !== null) {
    tables.add(match[2]);
  }
  
  // Паттерн для извлечения таблиц из UPDATE
  const updateRegex = /^\s*UPDATE\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)/i;
  if ((match = sql.match(updateRegex)) !== null) {
    tables.add(match[1]);
  }
  
  // Паттерн для извлечения таблиц из DELETE
  const deleteRegex = /^\s*DELETE\s+FROM\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)/i;
  if ((match = sql.match(deleteRegex)) !== null) {
    tables.add(match[1]);
  }
  
  return Array.from(tables);
}

// Более сложный парсер таблиц с алиасами
function extractTablesWithAliases(sql) {
  const result = [];
  
  // Паттерн для извлечения таблиц с алиасами
  const tableRegex = /FROM\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)(?:\s+(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*))?|\b(JOIN|INNER JOIN|LEFT JOIN|RIGHT JOIN|FULL JOIN)\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z0-9_]*)?)(?:\s+(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*))?/gi;
  
  let match;
  while ((match = tableRegex.exec(sql)) !== null) {
    if (match[1]) { // FROM clause
      result.push({
        table: match[1],
        alias: match[2] || null,
        type: 'FROM'
      });
    } else if (match[4]) { // JOIN clause
      result.push({
        table: match[4],
        alias: match[5] || null,
        type: match[3]
      });
    }
  }
  
  return result;
}

// Примеры использования
console.log(extractTables('SELECT u.name, p.title FROM users u JOIN posts p ON u.id = p.user_id WHERE u.active = 1'));
console.log(extractTablesWithAliases('SELECT * FROM users AS u LEFT JOIN profiles AS p ON u.id = p.user_id'));
```

### Извлечение полей из SELECT

```javascript
// Извлечение полей из SELECT части
function extractSelectFields(sql) {
  // Ищем только SELECT часть запроса
  const selectMatch = sql.match(/^\s*SELECT\s+(.*?)\s+FROM\s+/i);
  if (!selectMatch) return [];
  
  const selectPart = selectMatch[1];
  
  // Разбиваем по запятым, но учитываем скобки и кавычки
  const fields = splitSqlFields(selectPart);
  
  return fields.map(field => {
    // Убираем лишние пробелы и обрабатываем алиасы
    field = field.trim();
    
    // Проверяем наличие алиаса (AS или пробел)
    const aliasMatch = field.match(/^(.+?)\s+(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*)$/i);
    if (aliasMatch) {
      return {
        expression: aliasMatch[1].trim(),
        alias: aliasMatch[2]
      };
    }
    
    return {
      expression: field,
      alias: null
    };
  });
}

// Вспомогательная функция для корректного разделения полей
function splitSqlFields(selectPart) {
  const fields = [];
  let current = '';
  let depth = 0;
  let inQuotes = false;
  let quoteChar = null;
  
  for (let i = 0; i < selectPart.length; i++) {
    const char = selectPart[i];
    
    if (!inQuotes && (char === '"' || char === "'" || char === '`')) {
      inQuotes = true;
      quoteChar = char;
    } else if (inQuotes && char === quoteChar) {
      inQuotes = false;
      quoteChar = null;
    } else if (!inQuotes) {
      if (char === '(') {
        depth++;
      } else if (char === ')') {
        depth--;
      } else if (char === ',' && depth === 0) {
        fields.push(current);
        current = '';
        continue;
      }
    }
    
    current += char;
  }
  
  if (current.trim()) {
    fields.push(current);
  }
  
  return fields;
}

// Примеры использования
console.log(extractSelectFields('SELECT id, name AS user_name, email, created_at FROM users'));
console.log(extractSelectFields('SELECT u.id, u.name, COUNT(p.id) AS post_count FROM users u LEFT JOIN posts p ON u.id = p.user_id'));
```

## 2. Анализ условий и параметров

### Извлечение WHERE условий

```javascript
// Извлечение условий из WHERE части
function extractWhereConditions(sql) {
  // Паттерн для извлечения WHERE части (до GROUP BY, ORDER BY, LIMIT)
  const whereRegex = /WHERE\s+(.+?)(?:\s+(GROUP BY|ORDER BY|LIMIT|HAVING|UNION|;|\)|$))/i;
  const match = sql.match(whereRegex);
  
  if (!match) return [];
  
  const wherePart = match[1];
  const conditions = splitSqlConditions(wherePart);
  
  return conditions.map(condition => {
    condition = condition.trim();
    
    // Пытаемся разобрать условие на составляющие
    const conditionMatch = condition.match(/^(\w+(?:\.\w+)?)\s*([<>=!]+|LIKE|IN|BETWEEN)\s*(.+)$/i);
    
    if (conditionMatch) {
      return {
        field: conditionMatch[1],
        operator: conditionMatch[2].toUpperCase(),
        value: conditionMatch[3].trim()
      };
    }
    
    return {
      raw: condition,
      field: null,
      operator: null,
      value: null
    };
  });
}

// Разделение условий с учетом логических операторов
function splitSqlConditions(wherePart) {
  const conditions = [];
  let current = '';
  let depth = 0;
  let inQuotes = false;
  let quoteChar = null;
  
  // Регулярное выражение для поиска логических операторов
  const logicOpRegex = /\b(AND|OR)\b/gi;
  
  let lastIndex = 0;
  let match;
  
  while ((match = logicOpRegex.exec(wherePart)) !== null) {
    const opIndex = match.index;
    const op = match[1].toUpperCase();
    
    // Проверяем, находится ли оператор внутри скобок или кавычек
    const beforeOp = wherePart.substring(0, opIndex);
    const depthBefore = (beforeOp.match(/\(/g) || []).length - (beforeOp.match(/\)/g) || []).length;
    
    if (depthBefore === 0) { // Оператор на верхнем уровне
      const condition = wherePart.substring(lastIndex, opIndex).trim();
      if (condition) conditions.push(condition);
      lastIndex = match.index + match[0].length;
    }
  }
  
  // Добавляем последнюю часть
  const lastCondition = wherePart.substring(lastIndex).trim();
  if (lastCondition) conditions.push(lastCondition);
  
  return conditions.filter(c => c.toUpperCase().trim() !== 'AND' && c.toUpperCase().trim() !== 'OR');
}

// Примеры использования
console.log(extractWhereConditions('SELECT * FROM users WHERE id = 1 AND name LIKE "John%" AND active = 1'));
console.log(extractWhereConditions('SELECT * FROM orders WHERE user_id = ? AND created_at BETWEEN "2023-01-01" AND "2023-12-31"'));
```

### Извлечение параметров запроса

```javascript
// Извлечение параметров (placeholders) из запроса
function extractParameters(sql) {
  // Паттерны для различных типов placeholder'ов
  const patterns = [
    { type: 'positional', regex: /\?/g },           // ?
    { type: 'named', regex: /:[a-zA-Z_][a-zA-Z0-9_]*/g },  // :name
    { type: 'dollar', regex: /\$[0-9]+/g },         // $1, $2, ...
    { type: 'at', regex: /@[a-zA-Z_][a-zA-Z0-9_]*/g }     // @name (SQL Server)
  ];
  
  const params = [];
  
  for (const pattern of patterns) {
    let match;
    while ((match = pattern.regex.exec(sql)) !== null) {
      params.push({
        value: match[0],
        type: pattern.type,
        index: match.index
      });
    }
  }
  
  return params;
}

// Классификация параметров по типам использования
function classifyParameters(sql) {
  const params = extractParameters(sql);
  
  const classification = {
    positional: [],
    named: [],
    dollar: [],
    at: []
  };
  
  params.forEach(param => {
    classification[param.type].push(param);
  });
  
  return classification;
}

// Примеры использования
console.log(extractParameters('SELECT * FROM users WHERE id = ? AND name = :name AND status = $1'));
console.log(classifyParameters('SELECT u.*, p.title FROM users u JOIN posts p ON u.id = @user_id WHERE u.created_at > $2'));
```

## 3. Анализ сложных запросов

### Извлечение подзапросов

```javascript
// Извлечение подзапросов
function extractSubqueries(sql) {
  const subqueries = [];
  const stack = [];
  let lastStart = -1;
  
  for (let i = 0; i < sql.length; i++) {
    if (sql[i] === '(') {
      if (stack.length === 0) {
        // Начало потенциального подзапроса
        const beforeParen = sql.substring(Math.max(0, i - 10), i).toUpperCase();
        if (/\b(FROM|WHERE|SELECT|HAVING|EXISTS|IN|JOIN)\s*$/.test(beforeParen)) {
          lastStart = i;
        }
      }
      stack.push(i);
    } else if (sql[i] === ')') {
      if (stack.length > 0) {
        const start = stack.pop();
        
        if (stack.length === 0 && lastStart === start) {
          // Это подзапрос
          const subquery = sql.substring(start + 1, i);
          subqueries.push({
            full: `(${subquery})`,
            content: subquery.trim(),
            start: start,
            end: i
          });
          lastStart = -1;
        }
      }
    }
  }
  
  return subqueries;
}

// Анализ вложенных запросов более высокого уровня
function analyzeNestedQueries(sql) {
  // Ищем паттерны вложенных запросов
  const patterns = [
    { type: 'subquery', regex: /\((\s*SELECT[^)]+)\)/gi, description: 'Subquery in parentheses' },
    { type: 'exists', regex: /\bEXISTS\s*\(([^)]+)\)/gi, description: 'EXISTS subquery' },
    { type: 'in_subquery', regex: /\bIN\s*\(([^)]+)\)/gi, description: 'IN subquery' }
  ];
  
  const results = [];
  
  for (const pattern of patterns) {
    let match;
    while ((match = pattern.regex.exec(sql)) !== null) {
      results.push({
        type: pattern.type,
        content: match[1],
        fullMatch: match[0],
        description: pattern.description,
        index: match.index
      });
    }
  }
  
  return results;
}

// Примеры использования
const complexQuery = `
  SELECT u.name, 
         (SELECT COUNT(*) FROM posts WHERE user_id = u.id) as post_count
  FROM users u 
  WHERE u.id IN (SELECT user_id FROM user_roles WHERE role = 'admin')
  AND EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.id AND p.status = 'published')
`;

console.log('Subqueries:', extractSubqueries(complexQuery));
console.log('Nested queries:', analyzeNestedQueries(complexQuery));
```

### Извлечение JOIN условий

```javascript
// Извлечение JOIN условий
function extractJoinConditions(sql) {
  const joinRegex = /\b(JOIN|INNER JOIN|LEFT JOIN|RIGHT JOIN|FULL JOIN)\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)\s*(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*)?\s+ON\s+(.+?)(?=\s+(?:JOIN|WHERE|GROUP BY|ORDER BY|LIMIT|;|$))/gi;
  
  const joins = [];
  let match;
  
  while ((match = joinRegex.exec(sql)) !== null) {
    joins.push({
      type: match[1].toUpperCase(),
      table: match[2],
      alias: match[3] || null,
      condition: match[4].trim()
    });
  }
  
  return joins;
}

// Разбор условий JOIN на составляющие
function parseJoinConditions(joinCondition) {
  // Разбираем условие JOIN на левую и правую части
  const andSplit = joinCondition.split(/\s+AND\s+/i);
  const conditions = [];
  
  for (const condition of andSplit) {
    const eqMatch = condition.trim().match(/^(\w+(?:\.\w+)?)\s*=\s*(\w+(?:\.\w+)?)$/);
    if (eqMatch) {
      conditions.push({
        left: eqMatch[1],
        operator: '=',
        right: eqMatch[2]
      });
    } else {
      conditions.push({
        raw: condition.trim(),
        left: null,
        operator: null,
        right: null
      });
    }
  }
  
  return conditions;
}

// Примеры использования
const joinQuery = `
  SELECT u.name, p.title, c.name as category_name
  FROM users u
  INNER JOIN posts p ON u.id = p.user_id AND p.status = 'published'
  LEFT JOIN categories c ON p.category_id = c.id
  WHERE u.active = 1
`;

console.log('Joins:', extractJoinConditions(joinQuery));
const joinConditions = extractJoinConditions(joinQuery);
joinConditions.forEach((join, index) => {
  console.log(`Join ${index + 1} conditions:`, parseJoinConditions(join.condition));
});
```

## 4. Практические примеры анализа

### Анализ безопасности запросов

```javascript
// Класс для анализа безопасности SQL-запросов
class SqlSecurityAnalyzer {
  constructor() {
    // Паттерны потенциально опасных операций
    this.dangerousPatterns = [
      { name: 'union_based', pattern: /\bUNION\b/i, severity: 'high' },
      { name: 'time_based', pattern: /\bSLEEP\b|\bWAITFOR\b|\bDELAY\b/i, severity: 'high' },
      { name: 'command_execution', pattern: /\bEXEC\b|\bEXECUTE\b|\bCMD\b/i, severity: 'critical' },
      { name: 'comment_based', pattern: /(?:--|#|\/\*|\*\/)/g, severity: 'medium' },
      { name: 'wildcard_select', pattern: /\bSELECT\s+\*/i, severity: 'low' },
      { name: 'drop_table', pattern: /\bDROP\s+TABLE\b/i, severity: 'critical' },
      { name: 'truncate_table', pattern: /\bTRUNCATE\s+TABLE\b/i, severity: 'critical' },
      { name: 'alter_table', pattern: /\bALTER\s+TABLE\b/i, severity: 'high' }
    ];
  }
  
  analyze(query) {
    const issues = [];
    
    for (const pattern of this.dangerousPatterns) {
      if (pattern.pattern.test(query)) {
        issues.push({
          name: pattern.name,
          severity: pattern.severity,
          pattern: pattern.pattern.toString()
        });
      }
    }
    
    // Проверка на отсутствие WHERE в UPDATE/DELETE
    const updateDeleteMatch = query.match(/^\s*(UPDATE|DELETE)\s+[a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?\s+((?!WHERE).)*$/i);
    if (updateDeleteMatch) {
      issues.push({
        name: 'missing_where_clause',
        severity: 'high',
        description: 'UPDATE or DELETE without WHERE clause'
      });
    }
    
    return {
      safe: issues.length === 0,
      issues,
      score: this.calculateSecurityScore(issues)
    };
  }
  
  calculateSecurityScore(issues) {
    const severityWeights = { critical: 10, high: 7, medium: 4, low: 1 };
    let score = 100;
    
    for (const issue of issues) {
      score -= severityWeights[issue.severity] || 0;
    }
    
    return Math.max(0, score);
  }
}

// Пример использования
const analyzer = new SqlSecurityAnalyzer();

console.log(analyzer.analyze('SELECT * FROM users WHERE id = ?'));
console.log(analyzer.analyze('SELECT * FROM users WHERE id = 1 OR 1=1')); // Подозрительный UNION
console.log(analyzer.analyze('UPDATE users SET admin = 1')); // Нет WHERE
```

### Статистика запроса

```javascript
// Класс для получения статистики SQL-запроса
class SqlQueryStats {
  static analyze(sql) {
    return {
      type: getQueryType(sql),
      tables: extractTables(sql),
      fields: extractSelectFields(sql).length,
      conditions: extractWhereConditions(sql).length,
      parameters: extractParameters(sql).length,
      joins: extractJoinConditions(sql).length,
      hasSubqueries: extractSubqueries(sql).length > 0,
      complexity: this.calculateComplexity(sql),
      length: sql.length,
      hasOrderBy: /\bORDER BY\b/i.test(sql),
      hasGroupBy: /\bGROUP BY\b/i.test(sql),
      hasHaving: /\bHAVING\b/i.test(sql),
      hasLimit: /\bLIMIT\b/i.test(sql)
    };
  }
  
  static calculateComplexity(sql) {
    let complexity = 1; // Базовая сложность
    
    // Каждый JOIN увеличивает сложность
    complexity += (sql.match(/\bJOIN\b/gi) || []).length;
    
    // Каждое условие WHERE увеличивает сложность
    complexity += (sql.match(/\bAND\b|\bOR\b/gi) || []).length;
    
    // Подзапросы значительно увеличивают сложность
    complexity += extractSubqueries(sql).length * 3;
    
    // GROUP BY и HAVING увеличивают сложность
    if (/\bGROUP BY\b/i.test(sql)) complexity += 2;
    if (/\bHAVING\b/i.test(sql)) complexity += 2;
    
    // ORDER BY и LIMIT немного увеличивают сложность
    if (/\bORDER BY\b/i.test(sql)) complexity += 1;
    if (/\bLIMIT\b/i.test(sql)) complexity += 1;
    
    return complexity;
  }
}

// Пример использования
const stats1 = SqlQueryStats.analyze('SELECT * FROM users WHERE id = ?');
const stats2 = SqlQueryStats.analyze(`
  SELECT u.name, COUNT(p.id) as post_count
  FROM users u
  LEFT JOIN posts p ON u.id = p.user_id
  WHERE u.active = 1
  GROUP BY u.id
  HAVING post_count > 5
  ORDER BY post_count DESC
  LIMIT 10
`);

console.log('Simple query stats:', stats1);
console.log('Complex query stats:', stats2);
```

## 5. Практические инструменты

### SQL форматирование

```javascript
// Простой SQL форматтер с использованием регулярных выражений
class SqlFormatter {
  static format(sql) {
    let formatted = sql;
    
    // Добавляем переносы строк перед ключевыми словами
    formatted = formatted.replace(/\b(SELECT|FROM|WHERE|GROUP BY|HAVING|ORDER BY|LIMIT|JOIN|INNER JOIN|LEFT JOIN|RIGHT JOIN|UNION|UNION ALL)\b/gi, '\n$1');
    
    // Убираем лишние пробелы
    formatted = formatted.replace(/\s+/g, ' ');
    
    // Добавляем отступы
    const lines = formatted.split('\n');
    let indent = 0;
    const indentedLines = [];
    
    for (let line of lines) {
      line = line.trim();
      if (!line) continue;
      
      // Уменьшаем отступ для закрывающих скобок
      const closingBrackets = (line.match(/\)/g) || []).length;
      indent = Math.max(0, indent - closingBrackets);
      
      indentedLines.push('  '.repeat(indent) + line);
      
      // Увеличиваем отступ для открывающих скобок
      const openingBrackets = (line.match(/\(/g) || []).length;
      indent += openingBrackets;
    }
    
    return indentedLines.join('\n');
  }
  
  // Форматирование с сохранением структуры
  static formatPreserving(sql) {
    // Более сложный форматтер, который учитывает скобки и кавычки
    let formatted = '';
    let indentLevel = 0;
    let inQuotes = false;
    let quoteChar = null;
    let currentLine = '';
    
    for (let i = 0; i < sql.length; i++) {
      const char = sql[i];
      
      if (!inQuotes && (char === '"' || char === "'" || char === '`')) {
        inQuotes = true;
        quoteChar = char;
      } else if (inQuotes && char === quoteChar) {
        inQuotes = false;
        quoteChar = null;
      }
      
      if (!inQuotes) {
        if (char === '(') {
          formatted += currentLine.trim() + char + '\n' + '  '.repeat(indentLevel + 1);
          currentLine = '';
          indentLevel++;
        } else if (char === ')') {
          indentLevel--;
          formatted += currentLine.trim() + '\n' + '  '.repeat(indentLevel) + char;
          currentLine = '';
        } else if (/\b(FROM|WHERE|GROUP BY|HAVING|ORDER BY|LIMIT)\b/i.test(sql.substring(i))) {
          // Если встречаем ключевое слово
          const wordMatch = sql.substring(i).match(/^\b(FROM|WHERE|GROUP BY|HAVING|ORDER BY|LIMIT)\b/i);
          if (wordMatch) {
            const word = wordMatch[1];
            formatted += currentLine.trim() + '\n' + '  '.repeat(indentLevel) + word;
            currentLine = '';
            i += word.length - 1; // Пропускаем обработанное слово
          } else {
            currentLine += char;
          }
        } else {
          currentLine += char;
        }
      } else {
        currentLine += char;
      }
    }
    
    if (currentLine.trim()) {
      formatted += currentLine.trim();
    }
    
    return formatted;
  }
}

// Пример использования
const rawSql = 'SELECT u.name, p.title FROM users u JOIN posts p ON u.id = p.user_id WHERE u.active = 1 AND p.status = "published" ORDER BY p.created_at DESC LIMIT 10';
console.log('Formatted SQL:');
console.log(SqlFormatter.format(rawSql));
```

## 6. Практические рекомендации

### Лучшие практики анализа SQL

```javascript
// Рекомендации по анализу SQL-запросов
const SqlAnalysisBestPractices = {
  // Порядок анализа
  analysisOrder: [
    'query_type',      // 1. Определить тип запроса
    'tables',          // 2. Извлечь таблицы
    'fields',          // 3. Извлечь поля
    'conditions',      // 4. Извлечь условия
    'parameters',      // 5. Извлечь параметры
    'security',        // 6. Проверить безопасность
    'complexity'       // 7. Оценить сложность
  ],
  
  // Рекомендуемые паттерны
  recommendedPatterns: {
    // Для извлечения таблиц
    tableExtraction: /FROM\s+([a-zA-Z_][a-zA-Z0-9_]*(?:\.[a-zA-Z_][a-zA-Z0-9_]*)?)(?:\s+(?:AS\s+)?([a-zA-Z_][a-zA-Z0-9_]*))?/gi,
    
    // Для извлечения параметров
    parameters: /\?|:[a-zA-Z_][a-zA-Z0-9_]*|\$[0-9]+|@[a-zA-Z_][a-zA-Z0-9_]*/g,
    
    // Для безопасного извлечения строковых значений
    stringValues: /'([^']*(?:''[^']*)*)'/g
  },
  
  // Часто используемые проверки
  commonValidations: {
    hasWhere: (sql) => /\bWHERE\b/i.test(sql),
    hasOrderBy: (sql) => /\bORDER BY\b/i.test(sql),
    isSelect: (sql) => /^\s*SELECT\b/i.test(sql),
    hasLimit: (sql) => /\bLIMIT\b/i.test(sql),
    isSafe: (sql) => !/\b(UNION|DROP|TRUNCATE|EXEC)\b/i.test(sql)
  }
};

// Практический пример комплексного анализа
function comprehensiveSqlAnalysis(sql) {
  return {
    basic: {
      type: getQueryType(sql),
      tables: extractTables(sql),
      parameters: extractParameters(sql).length
    },
    conditions: extractWhereConditions(sql),
    security: new SqlSecurityAnalyzer().analyze(sql),
    stats: SqlQueryStats.analyze(sql),
    recommendations: generateRecommendations(sql)
  };
}

function generateRecommendations(sql) {
  const recommendations = [];
  
  // Рекомендации по безопасному запросу
  if (!/\bWHERE\b/i.test(sql) && /^\s*(UPDATE|DELETE)\b/i.test(sql)) {
    recommendations.push('Add WHERE clause to prevent unintended data modification');
  }
  
  if (!/\bLIMIT\b/i.test(sql) && /^\s*SELECT\b/i.test(sql)) {
    recommendations.push('Consider adding LIMIT clause for large datasets');
  }
  
  if ((sql.match(/\bOR\b/gi) || []).length > 2) {
    recommendations.push('Multiple OR conditions might benefit from IN clause');
  }
  
  // Проверка на потенциальные проблемы производительности
  if (sql.includes('%LIKE%') && sql.indexOf('%LIKE%') === sql.indexOf('LIKE')) {
    recommendations.push('Leading wildcard in LIKE can cause performance issues');
  }
  
  return recommendations;
}

// Пример комплексного анализа
const exampleQuery = 'SELECT u.name, p.title FROM users u JOIN posts p ON u.id = p.user_id WHERE u.active = 1 AND p.status = "published" ORDER BY p.created_at DESC';
console.log('Comprehensive analysis:', comprehensiveSqlAnalysis(exampleQuery));
```

## Заключение

Анализ SQL-запросов с помощью регулярных выражений - мощный инструмент для безопасности, оптимизации и аудита. Хотя регулярные выражения не могут полностью заменить полноценный SQL-парсер, они эффективны для многих задач анализа и извлечения информации.

Ключевые моменты:
- Используйте специализированные паттерны для разных частей запроса
- Учитывайте безопасность при анализе пользовательского ввода
- Рассмотрите комбинацию регулярных выражений с полноценным парсингом
- Проверяйте запросы на потенциальные уязвимости
- Используйте анализ для оптимизации производительности