# Рекурсивные дженерики

Рекурсивные дженерики - это дженерики, которые ссылаются на самих себя в своем определении. Это мощная возможность для создания сложных типов данных и алгоритмов на уровне типов.

## Базовая концепция

Рекурсивный дженерик - это дженерик, который использует себя в своем определении:

```typescript
type RecursiveGeneric<T> = T | RecursiveGeneric<T>[];
```

## Простые примеры

### 1. Рекурсивный массив

```typescript
type NestedArray<T> = T | NestedArray<T>[];

const nested: NestedArray<number> = [1, [2, 3], [[4, 5], 6]];
```

### 2. Рекурсивный объект

```typescript
type JsonValue = 
  | string
  | number
  | boolean
  | null
  | JsonObject
  | JsonArray;

interface JsonObject {
  [key: string]: JsonValue;
}

interface JsonArray extends Array<JsonValue> {}

// Использование
const jsonData: JsonValue = {
  name: "John",
  age: 30,
  hobbies: ["reading", "swimming"],
  address: {
    street: "123 Main St",
    city: "New York"
  }
};
```

## Продвинутые примеры

### 1. Дженерик для создания дерева

```typescript
interface TreeNode<T> {
  value: T;
  children: RecursiveTreeNode<T>[];
}

type RecursiveTreeNode<T> = TreeNode<T> | null;

// Альтернативное определение
type Tree<T> = {
  value: T;
  children: Tree<T>[];
};

// Создание дерева
const tree: Tree<string> = {
  value: "root",
  children: [
    {
      value: "child1",
      children: [
        { value: "grandchild1", children: [] },
        { value: "grandchild2", children: [] }
      ]
    },
    {
      value: "child2",
      children: []
    }
  ]
};
```

### 2. Дженерик для создания связанных списков

```typescript
interface LinkedListNode<T> {
  value: T;
  next: RecursiveLinkedListNode<T>;
}

type RecursiveLinkedListNode<T> = LinkedListNode<T> | null;

// Создание связанного списка
const list: RecursiveLinkedListNode<number> = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: null
    }
  }
};

// Функции для работы со списком
function traverseList<T>(node: RecursiveLinkedListNode<T>): T[] {
  const result: T[] = [];
  let current = node;
  
  while (current !== null) {
    result.push(current.value);
    current = current.next;
  }
  
  return result;
}

function findInList<T>(node: RecursiveLinkedListNode<T>, value: T): boolean {
  let current = node;
  
  while (current !== null) {
    if (current.value === value) {
      return true;
    }
    current = current.next;
  }
  
  return false;
}
```

### 3. Дженерик для создания AST (Abstract Syntax Tree)

```typescript
type Expression<T> = 
  | Literal<T>
  | BinaryExpression<T>
  | CallExpression<T>;

interface Literal<T> {
  type: "Literal";
  value: T;
}

interface BinaryExpression<T> {
  type: "BinaryExpression";
  operator: "+" | "-" | "*" | "/";
  left: Expression<T>;
  right: Expression<T>;
}

interface CallExpression<T> {
  type: "CallExpression";
  callee: string;
  arguments: Expression<T>[];
}

// Создание AST
const ast: Expression<number> = {
  type: "BinaryExpression",
  operator: "+",
  left: {
    type: "Literal",
    value: 5
  },
  right: {
    type: "BinaryExpression",
    operator: "*",
    left: {
      type: "Literal",
      value: 3
    },
    right: {
      type: "Literal",
      value: 4
    }
  }
};
```

## Практические примеры

### 1. Дженерик для создания конфигурационных объектов

```typescript
interface ConfigSection<T> {
  name: string;
  enabled: boolean;
  settings: T;
  subsections: RecursiveConfigSection<T>[];
}

type RecursiveConfigSection<T> = ConfigSection<T> | null;

// Создание конфигурации
const config: RecursiveConfigSection<{ theme: string; language: string }> = {
  name: "main",
  enabled: true,
  settings: {
    theme: "dark",
    language: "ru"
  },
  subsections: [
    {
      name: "database",
      enabled: true,
      settings: {
        host: "localhost",
        port: 5432
      },
      subsections: []
    }
  ]
};

// Функция для поиска секции
function findSection<T>(
  section: RecursiveConfigSection<T>, 
  name: string
): ConfigSection<T> | null {
  if (section === null) {
    return null;
  }
  
  if (section.name === name) {
    return section;
  }
  
  for (const subsection of section.subsections) {
    const found = findSection(subsection, name);
    if (found) {
      return found;
    }
  }
  
  return null;
}
```

### 2. Дженерик для создания фильтров

```typescript
type FilterOperator = "eq" | "ne" | "gt" | "lt" | "in" | "contains";

interface FilterCondition<T> {
  field: keyof T;
  operator: FilterOperator;
  value: any;
}

type Filter<T> = 
  | FilterCondition<T>
  | { and: RecursiveFilter<T>[] }
  | { or: RecursiveFilter<T>[] }
  | { not: RecursiveFilter<T> };

type RecursiveFilter<T> = Filter<T>;

// Создание фильтра
interface User {
  name: string;
  age: number;
  status: "active" | "inactive";
  role: string;
}

const filter: RecursiveFilter<User> = {
  and: [
    { field: "age", operator: "gt", value: 18 },
    { or: [
      { field: "status", operator: "eq", value: "active" },
      { field: "role", operator: "in", value: ["admin", "moderator"] }
    ]}
  ]
};

// Функция для применения фильтра
function applyFilter<T>(data: T[], filter: RecursiveFilter<T>): T[] {
  // Реализация применения фильтра
  return data.filter(item => evaluateFilter(item, filter));
}

function evaluateFilter<T>(item: T, filter: RecursiveFilter<T>): boolean {
  if ("and" in filter) {
    return filter.and.every(subFilter => evaluateFilter(item, subFilter));
  }
  
  if ("or" in filter) {
    return filter.or.some(subFilter => evaluateFilter(item, subFilter));
  }
  
  if ("not" in filter) {
    return !evaluateFilter(item, filter.not);
  }
  
  // Обработка простого условия
  const condition = filter as FilterCondition<T>;
  const value = item[condition.field];
  
  switch (condition.operator) {
    case "eq": return value === condition.value;
    case "ne": return value !== condition.value;
    case "gt": return value > condition.value;
    case "lt": return value < condition.value;
    case "in": return Array.isArray(condition.value) && condition.value.includes(value);
    case "contains": return typeof value === "string" && value.includes(condition.value);
    default: return false;
  }
}
```

### 3. Дженерик для создания схем валидации

```typescript
type ValidationRule<T> = 
  | { type: "required" }
  | { type: "minLength", value: number }
  | { type: "maxLength", value: number }
  | { type: "pattern", value: RegExp }
  | { type: "nested", schema: ValidationSchema<T> };

interface ValidationSchema<T> {
  [K in keyof T]: ValidationRule<T>[];
}

type RecursiveValidationSchema<T> = ValidationSchema<T>;

// Создание схемы валидации
interface User {
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
  };
}

const userSchema: RecursiveValidationSchema<User> = {
  name: [
    { type: "required" },
    { type: "minLength", value: 2 },
    { type: "maxLength", value: 50 }
  ],
  email: [
    { type: "required" },
    { type: "pattern", value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ }
  ],
  address: [
    { type: "nested", schema: {
      street: [{ type: "required" }],
      city: [{ type: "required" }]
    }}
  ]
};

// Функция для валидации
function validate<T>(data: T, schema: RecursiveValidationSchema<T>): boolean {
  for (const key in schema) {
    const rules = schema[key];
    const value = data[key];
    
    for (const rule of rules) {
      if (!validateRule(value, rule, schema)) {
        return false;
      }
    }
  }
  
  return true;
}

function validateRule<T>(
  value: any, 
  rule: ValidationRule<T>, 
  schema: RecursiveValidationSchema<T>
): boolean {
  switch (rule.type) {
    case "required":
      return value !== undefined && value !== null;
    case "minLength":
      return typeof value === "string" && value.length >= rule.value;
    case "maxLength":
      return typeof value === "string" && value.length <= rule.value;
    case "pattern":
      return typeof value === "string" && rule.value.test(value);
    case "nested":
      return typeof value === "object" && value !== null && 
             validate(value, rule.schema);
    default:
      return false;
  }
}
```

## Ограничения и особенности

### 1. Глубина рекурсии

TypeScript имеет ограничения на глубину рекурсии типов:

```typescript
// Может вызвать ошибку компиляции при глубокой вложенности
type Deep<T> = { value: T } | { nested: Deep<T> };
```

### 2. Производительность

Сложные рекурсивные дженерики могут замедлять компиляцию:

```typescript
// Следует избегать чрезмерно сложных рекурсивных дженериков
type ComplexRecursive<T> = 
  | { type: "a", value: ComplexRecursive<T> }
  | { type: "b", left: ComplexRecursive<T>, right: ComplexRecursive<T> }
  | { type: "c", items: ComplexRecursive<T>[] };
```

### 3. Типобезопасность

Рекурсивные дженерики могут быть сложны для правильного использования:

```typescript
// Важно правильно определять базовые случаи
type LinkedList<T> = {
  value: T;
  next: LinkedList<T>; // Может привести к бесконечной рекурсии
} | null; // Базовый случай для завершения рекурсии
```

## Паттерны использования

### 1. Базовый случай + рекурсивный случай

```typescript
// Правильное определение рекурсивного дженерика
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[]; // Рекурсивный случай
} | null; // Базовый случай
```

### 2. Использование объединений для вариативности

```typescript
type FileSystemItem<T> = 
  | { type: "file", name: string, content: T }
  | { type: "directory", name: string, contents: FileSystemItem<T>[] };
```

## Связанные концепции

- [[ts/advanced-types/recursive-types|Рекурсивные типы]]
- [[ts/advanced-generics/generic-constraints|Ограничения дженериков]]
- [[ts/generics/TS Generics|Основы дженериков]]
- [[ts/type-system/structural-typing|Структурная типизация]]