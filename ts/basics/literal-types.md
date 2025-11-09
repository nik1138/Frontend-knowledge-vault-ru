# Литеральные типы в TypeScript

## Введение

Литеральные типы в TypeScript позволяют создавать типы с очень узкими, конкретными значениями. В отличие от общих типов, таких как `string` или `number`, литеральные типы представляют собой конкретные значения.

## Типы литеральных значений

### Строковые литеральные типы

```typescript
// Определение строкового литерального типа
type Direction = "left" | "right" | "up" | "down";

function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

// Корректное использование
move("left");  // OK
move("right"); // OK

// Ошибка: Аргумент типа '"north"' не может быть назначен параметру типа 'Direction'
// move("north");
```

### Числовые литеральные типы

```typescript
type StatusCode = 200 | 201 | 400 | 401 | 404 | 500;

function handleResponse(code: StatusCode): void {
  switch (code) {
    case 200:
      console.log("Success");
      break;
    case 404:
      console.log("Not Found");
      break;
    default:
      console.log(`Error: ${code}`);
  }
}
```

### Булевые литеральные типы

```typescript
type BooleanLiteral = true | false;

function configure(setting: BooleanLiteral): void {
  if (setting) {
    console.log("Setting enabled");
  } else {
    console.log("Setting disabled");
  }
}
```

## Практическое применение

### Ограничение параметров функции

```typescript
type Alignment = "left" | "center" | "right";
type Theme = "light" | "dark";

interface TextOptions {
  alignment: Alignment;
  theme: Theme;
  fontSize: 12 | 14 | 16 | 18 | 20;
}

function formatText(options: TextOptions): string {
  return `Text formatted with alignment: ${options.alignment}, theme: ${options.theme}, font size: ${options.fontSize}`;
}
```

### Использование с интерфейсами

```typescript
interface Config {
  env: "development" | "production" | "test";
  debug: true | false;
  version: "1.0" | "2.0" | "3.0";
}

const developmentConfig: Config = {
  env: "development",
  debug: true,
  version: "1.0"
};
```

## Расширенные примеры

### Комбинирование литеральных типов

```typescript
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type HttpStatus = 200 | 201 | 204 | 400 | 401 | 403 | 404 | 500;

interface Request {
  method: HttpMethod;
  url: string;
  status: HttpStatus;
}

function logRequest(request: Request): void {
  console.log(`${request.method} ${request.url}: ${request.status}`);
}
```

### Литеральные типы в перечислениях

```typescript
type Permission = "read" | "write" | "delete";
type UserRole = "admin" | "moderator" | "user";

interface UserPermissions {
  role: UserRole;
  permissions: Permission[];
}

const adminPermissions: UserPermissions = {
  role: "admin",
  permissions: ["read", "write", "delete"]
};
```

## Связь с другими концепциями

- [[types/union-intersection-types-comprehensive|Объединения и пересечения типов]] - литеральные типы часто используются в объединениях
- [[type-system/type-guards|Тип-гарды]] - помогают работать с литеральными типами во время выполнения
- [[basics/type-annotations|Аннотации типов]] - основа для определения литеральных типов

## Заключение

Литеральные типы позволяют создавать более строгие и выразительные типы, ограничивая возможные значения конкретными литералами. Это помогает избежать ошибок и делает код более понятным и самодокументированным.