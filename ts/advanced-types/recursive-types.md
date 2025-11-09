# Рекурсивные типы (Recursive Types)

Рекурсивные типы в TypeScript позволяют определять типы, которые ссылаются на самих себя. Это мощный механизм для создания сложных структур данных, таких как деревья, связанные списки и другие иерархические структуры.

## Основная концепция

Рекурсивный тип - это тип, который ссылается на себя в своем определении:

```typescript
type RecursiveType = {
  value: string;
  next?: RecursiveType;
};
```

## Простые примеры

### 1. Связанный список

```typescript
type LinkedList<T> = {
  value: T;
  next?: LinkedList<T>;
};

const list: LinkedList<number> = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: undefined
    }
  }
};
```

### 2. Дерево

```typescript
interface TreeNode<T> {
  value: T;
  children?: TreeNode<T>[];
}

const tree: TreeNode<string> = {
  value: "root",
  children: [
    {
      value: "child1",
      children: [
        { value: "grandchild1" },
        { value: "grandchild2" }
      ]
    },
    {
      value: "child2"
    }
  ]
};
```

## Продвинутые примеры

### 1. JSON-подобная структура

```typescript
type JSONValue = 
  | string
  | number
  | boolean
  | null
  | JSONObject
  | JSONArray;

interface JSONObject {
  [key: string]: JSONValue;
}

interface JSONArray extends Array<JSONValue> {}

// Пример использования
const jsonData: JSONValue = {
  name: "John",
  age: 30,
  hobbies: ["reading", "swimming"],
  address: {
    street: "123 Main St",
    city: "New York"
  }
};
```

### 2. AST (Abstract Syntax Tree)

```typescript
type Expression = 
  | NumberLiteral
  | StringLiteral
  | BinaryExpression
  | CallExpression;

interface NumberLiteral {
  type: "NumberLiteral";
  value: number;
}

interface StringLiteral {
  type: "StringLiteral";
  value: string;
}

interface BinaryExpression {
  type: "BinaryExpression";
  operator: "+" | "-" | "*" | "/";
  left: Expression;
  right: Expression;
}

interface CallExpression {
  type: "CallExpression";
  callee: string;
  arguments: Expression[];
}
```

### 3. Вложенные меню

```typescript
interface MenuItem {
  id: string;
  label: string;
  icon?: string;
  children?: MenuItem[];
  action?: () => void;
}

const menu: MenuItem[] = [
  {
    id: "file",
    label: "File",
    children: [
      {
        id: "new",
        label: "New",
        children: [
          { id: "new-file", label: "File", action: () => {} },
          { id: "new-folder", label: "Folder", action: () => {} }
        ]
      },
      { id: "open", label: "Open", action: () => {} },
      { id: "save", label: "Save", action: () => {} }
    ]
  }
];
```

## Рекурсивные условные типы

Можно комбинировать рекурсивные типы с условными типами:

```typescript
type DeepReadonly<T> = T extends object 
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> } 
  : T;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
  };
}

type ReadonlyUser = DeepReadonly<User>;
// Все вложенные свойства становятся readonly
```

## Рекурсивные сопоставленные типы

```typescript
type DeepPartial<T> = T extends object 
  ? { [P in keyof T]?: DeepPartial<T[P]> } 
  : T;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
  };
}

type PartialUser = DeepPartial<User>;
// {
//   name?: string;
//   address?: {
//     street?: string;
//     city?: string;
//   };
// }
```

## Практические примеры

### 1. Конфигурация с вложенными секциями

```typescript
interface ConfigSection {
  name: string;
  enabled: boolean;
  settings?: Record<string, any>;
  subsections?: ConfigSection[];
}

const config: ConfigSection = {
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
      }
    }
  ]
};
```

### 2. Фильтры для поиска

```typescript
type FilterOperator = "eq" | "ne" | "gt" | "lt" | "in" | "contains";

interface FilterCondition {
  field: string;
  operator: FilterOperator;
  value: any;
}

type Filter = 
  | FilterCondition
  | { and: Filter[] }
  | { or: Filter[] }
  | { not: Filter };

const filter: Filter = {
  and: [
    { field: "age", operator: "gt", value: 18 },
    { or: [
      { field: "status", operator: "eq", value: "active" },
      { field: "role", operator: "in", value: ["admin", "moderator"] }
    ]}
  ]
};
```

### 3. Схема валидации

```typescript
type ValidationRule = 
  | { type: "required" }
  | { type: "minLength", value: number }
  | { type: "maxLength", value: number }
  | { type: "pattern", value: RegExp }
  | { type: "nested", schema: ValidationSchema };

interface ValidationSchema {
  [field: string]: ValidationRule[];
}

const userSchema: ValidationSchema = {
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
```

## Ограничения и особенности

### 1. Глубина рекурсии

TypeScript имеет ограничения на глубину рекурсии типов для предотвращения бесконечных вычислений:

```typescript
// Может вызвать ошибку компиляции при глубокой вложенности
type Deep<T> = { value: T } | { nested: Deep<T> };
```

### 2. Производительность

Сложные рекурсивные типы могут замедлять компиляцию:

```typescript
// Следует избегать чрезмерно сложных рекурсивных типов
type ComplexRecursive = 
  | { type: "a", value: ComplexRecursive }
  | { type: "b", left: ComplexRecursive, right: ComplexRecursive }
  | { type: "c", items: ComplexRecursive[] }
  // ... и так далее
```

### 3. Типобезопасность

Рекурсивные типы могут быть сложны для правильного использования:

```typescript
// Важно правильно определять базовые случаи
type LinkedList<T> = {
  value: T;
  next?: LinkedList<T>; // ? делает тип конечным
};
```

## Паттерны использования

### 1. Базовый случай + рекурсивный случай

```typescript
// Правильное определение рекурсивного типа
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[]; // Необязательно для листьев
} | {
  value: T;
  children: []; // Базовый случай для листьев
};
```

### 2. Использование объединений для вариативности

```typescript
type FileSystemItem = 
  | { type: "file", name: string, size: number }
  | { type: "directory", name: string, contents: FileSystemItem[] };
```

## Связанные концепции

- [[ts/advanced-types/mapped-types|Сопоставленные типы]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/generics/TS Generics|Дженерики]]
- [[ts/type-system/structural-typing|Структурная типизация]]