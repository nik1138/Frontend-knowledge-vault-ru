# TypeScript Variables and Constants

Переменные и константы в TypeScript обеспечивают хранение и управление значениями с учетом строгой типизации. TypeScript расширяет возможности JavaScript, добавляя аннотации типов и различные режимы объявления переменных.

## Объявление переменных

### var, let, const в TypeScript

```typescript
// var - функциональная область видимости (не рекомендуется в TypeScript)
var legacyVar: string = "Hello"; // Избегайте использования var в TypeScript

// let - блочная область видимости
let message: string = "Hello, TypeScript!";
let count: number = 42;
let isActive: boolean = true;

// const - неизменяемая переменная (значение не может быть переназначено)
const PI: number = 3.14159;
const API_URL: string = "https://api.example.com";
const FEATURES: string[] = ["feature1", "feature2"];
```

### Вывод типов при объявлении

```typescript
// TypeScript может выводить типы автоматически
let inferredString = "Hello";        // тип: string
let inferredNumber = 42;             // тип: number
let inferredBoolean = true;          // тип: boolean
let inferredArray = [1, 2, 3];       // тип: number[]
let inferredObject = { name: "John" }; // тип: { name: string }

// Явная аннотация типа
let explicitString: string = "Hello";
let explicitNumber: number = 42;
```

## Область видимости переменных

### Блочная область видимости
```typescript
function scopeExample() {
  if (true) {
    let blockScoped = "доступна только внутри этого блока";
    const constant = "тоже блочная константа";
  }
  
  // console.log(blockScoped); // ОШИБКА: переменная не видна за пределами блока
  // blockScoped = "новое значение"; // ОШИБКА: нельзя переназначить const
}

// Разница между let и var
function varVsLet() {
  console.log(tempLet);    // ReferenceError: Cannot access before initialization
  console.log(tempVar);    // undefined (из-за hoisting)
  
  let tempLet = "let";
  var tempVar = "var";
}
```

### Функциональная область видимости (var)
```typescript
// Пример проблемы с var
function varIssue() {
  var x = 1;
  
  if (true) {
    var x = 2;  // Перезаписывает предыдущее значение
  }
  
  console.log(x); // 2, а не 1
}

// Решение с let
function letSolution() {
  let x = 1;
  
  if (true) {
    let x = 2;  // Создает новую переменную в блоке
  }
  
  console.log(x); // 1, оригинальное значение
}
```

## Аннотации типов

### Базовые типы
```typescript
let userName: string = "Alice";
let userAge: number = 30;
let isActiveUser: boolean = true;
let emptyValue: null = null;
let notAssigned: undefined = undefined;
let uniqueId: symbol = Symbol("id");
let bigNumber: bigint = 123456789012345678901234567890n;
```

### Объектные типы
```typescript
// Объект с конкретной структурой
let user: { name: string; age: number; email?: string } = {
  name: "Bob",
  age: 25
};

// Массивы
let stringArray: string[] = ["hello", "world"];
let numberArray: Array<number> = [1, 2, 3, 4];
let mixedArray: (string | number)[] = ["hello", 42, "world", 123];

// Кортежи (массивы с фиксированным количеством элементов с известными типами)
let tuple: [string, number, boolean] = ["hello", 42, true];
```

### Функциональные типы
```typescript
// Переменные, содержащие функции
let greet: (name: string) => string;
greet = function(name: string): string {
  return `Hello, ${name}!`;
};

// Стрелочные функции
let add: (a: number, b: number) => number = (a, b) => a + b;
let multiply = (a: number, b: number): number => a * b; // Вывод возвращаемого типа
```

## Константы и неизменяемость

### const с примитивами
```typescript
const userName: string = "Alice";
const userAge: number = 30;
// userName = "Bob"; // ОШИБКА: не может быть переназначена
```

### const с объектами и массивами
```typescript
// const защищает от переназначения, но не от изменений содержимого
const user = { name: "Alice", age: 30 };
user.name = "Bob";        // OK: содержимое может изменяться
user.age = 31;            // OK: содержимое может изменяться
// user = { name: "Carol" }; // ОШИБКА: переменная не может быть переназначена

// Для полной неизменяемости используйте readonly
const readonlyUser: { readonly name: string; readonly age: number } = {
  name: "Alice",
  age: 30
};
// readonlyUser.name = "Bob"; // ОШИБКА: свойство только для чтения
```

### readonly с массивами
```typescript
const readonlyArray: readonly number[] = [1, 2, 3, 4];
// readonlyArray.push(5); // ОШИБКА: push не доступен для readonly массивов
// readonlyArray[0] = 10; // ОШИБКА: нельзя изменить элемент

// Или с типом ReadonlyArray
const readonlyArray2: ReadonlyArray<string> = ["hello", "world"];
```

## Специальные случаи

### any и типизация
```typescript
// Избегайте any, когда это возможно
let flexible: any = 4;
flexible = "теперь строка";
flexible = true; // Любой тип допустим

// Лучше использовать union типы
let stringOrNumber: string | number = 4;
stringOrNumber = "теперь строка"; // OK
// stringOrNumber = true; // ОШИБКА: boolean не входит в объединение
```

### unknown для безопасного динамического значения
```typescript
// unknown безопаснее any
let unknownValue: unknown = "может быть что угодно";
// console.log(unknownValue.toUpperCase()); // ОШИБКА: нужно сначала проверить тип

if (typeof unknownValue === "string") {
  console.log(unknownValue.toUpperCase()); // OK: теперь TypeScript знает, что это строка
}
```

## Практические примеры

### Конфигурация приложения
```typescript
// Пример использования различных типов переменных для конфигурации
const APP_CONFIG = {
  name: "MyApp",
  version: "1.0.0",
  production: false,
  features: {
    analytics: true,
    notifications: false
  },
  endpoints: {
    api: "https://api.example.com",
    auth: "https://auth.example.com"
  }
} as const; // as const делает всю структуру readonly

// Типы для конфигурации
type AppConfig = typeof APP_CONFIG;
type FeatureFlags = typeof APP_CONFIG.features;
```

### Параметры функций
```typescript
// Переменные как параметры функций с типизацией
function processUser(
  id: number,           // обязательный параметр
  name: string,         // обязательный параметр
  age?: number,         // необязательный параметр
  isActive: boolean = true  // параметр с значением по умолчанию
): { id: number; name: string; age?: number; isActive: boolean } {
  return { id, name, age, isActive };
}

// Использование
const user1 = processUser(1, "Alice");
const user2 = processUser(2, "Bob", 30);
const user3 = processUser(3, "Charlie", 25, false);
```

### Локальные переменные с уточнением типов
```typescript
// Сужение типов в локальных переменных
function handleValue(value: string | number | boolean) {
  if (typeof value === "string") {
    // В этой области value имеет тип string
    const upperValue = value.toUpperCase();
    return upperValue;
  } else if (typeof value === "number") {
    // Здесь value имеет тип number
    return value.toFixed(2);
  } else {
    // Здесь value имеет тип boolean
    return value ? "true" : "false";
  }
}
```

## Лучшие практики

1. **Предпочитайте `const` над `let`, а `let` над `var`**
2. **Используйте явные аннотации типов для публичных API**
3. **Пользуйтесь выводом типов для локальных переменных**
4. **Используйте `readonly` для неизменяемых структур данных**
5. **Избегайте `any`, используйте более точные типы**

## Связь с другими концепциями
- [[basics/type-annotations]] - основы аннотаций типов
- [[functions]] - переменные-функции
- [[objects]] - объектные типы для переменных
- [[type-system/type-inference]] - автоматическое определение типов