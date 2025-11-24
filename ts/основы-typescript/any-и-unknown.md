---
aliases: ["Типы any и unknown", "any vs unknown", "Специальные типы TypeScript"]
tags: [typescript, programming, types, any, unknown]
---

# any и unknown

В TypeScript есть два специальных типа, которые позволяют работать с ситуациями, когда точный тип неизвестен: `any` и `unknown`. Несмотря на некоторое сходство, эти типы имеют важные различия в использовании и безопасности типов.

## Тип any

Тип `any` отключает проверку типов для переменной. Это может быть полезно в ситуациях, когда:

- Работа с динамическим контентом
- Интеграция с JavaScript-библиотеками без определений типов
- Постепенная миграция JavaScript-кода на TypeScript

### Примеры использования any

```ts
let notSure: any = 4;
notSure = "может быть строкой";
notSure = false; // и логическим значением

// any позволяет любые операции
notSure.ifItExists(); // OK, ifItExists может существовать
notSure.toFixed(); // OK, toFixed существует (но может не существовать при выполнении)

// any в массивах
let list: any[] = [1, true, "free"];
list[1] = 100;
```

### Проблемы с any

Использование `any` отключает проверку типов, что может привести к ошибкам:

```ts
function getLength(obj: any) {
  // Компилятор не предупредит о потенциальной ошибке
  return obj.length; // Ошибка во время выполнения, если obj не имеет свойства length
}

getLength(5); // Ошибка во время выполнения: у числа нет свойства length
```

## Тип unknown

Тип `unknown` является более безопасной альтернативой `any`. Переменная типа `unknown` может содержать значение любого типа, но перед использованием требует проверку типа.

### Примеры использования unknown

```ts
let value: unknown;

value = "hello";
value = 42;
value = true;
value = [];
value = {};
```

### Ограничения unknown

В отличие от `any`, с переменной типа `unknown` нельзя выполнять произвольные операции:

```ts
let value: unknown = "hello";

// Следующие операции вызовут ошибки компиляции:
// value.substr(1); // Ошибка: неизвестный тип
// value.toFixed(); // Ошибка: неизвестный тип
// value(); // Ошибка: неизвестный тип
```

## Проверка типов с unknown

Для использования значения типа `unknown` необходимо выполнить проверку типа:

### Проверка типа с typeof

```ts
function processValue(value: unknown) {
  if (typeof value === "string") {
    // Здесь TypeScript знает, что value - строка
    return value.toUpperCase();
  }
  
  if (typeof value === "number") {
    // Здесь TypeScript знает, что value - число
    return value.toFixed(2);
  }
  
  // Если тип не определен, возвращаем undefined
  return undefined;
}
```

### Проверка с instanceof

```ts
function handleValue(value: unknown) {
  if (value instanceof Date) {
    // Здесь TypeScript знает, что value - объект Date
    return value.toISOString();
  }
  
  if (value instanceof Array) {
    // Здесь TypeScript знает, что value - массив
    return value.length;
  }
  
  return null;
}
```

### Проверка с in operator

```ts
interface User {
  name: string;
  age: number;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    "age" in value
  );
}

function processUser(userData: unknown) {
  if (isUser(userData)) {
    // Здесь TypeScript знает, что userData - User
    console.log(userData.name);
    console.log(userData.age);
  }
}
```

## Утверждение типа

В некоторых случаях можно использовать утверждение типа (type assertion) с `unknown`:

```ts
function getValue(): unknown {
  // Предположим, что функция возвращает неизвестный тип
  return { name: "Иван", age: 30 };
}

// Утверждение типа
const user = getValue() as { name: string; age: number };
console.log(user.name); // OK

// Альтернативный синтаксис
const user2 = <{ name: string; age: number }>getValue();
```

## Type predicates

Для более сложных проверок типов можно использовать предикаты типов:

```ts
// Функция-предикат типа
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function formatValue(value: unknown) {
  if (isString(value)) {
    // Здесь TypeScript знает, что value - строка
    return value.toUpperCase();
  }
  
  if (isNumber(value)) {
    // Здесь TypeScript знает, что value - число
    return value.toFixed(2);
  }
  
  return String(value);
}
```

## Сравнение any и unknown

| Характеристика | any | unknown |
|---|---|---|
| Проверка типов | Отключена | Включена |
| Безопасность | Низкая | Высокая |
| Использование | Без проверки | Требует проверки |
| Рекомендация | Ограничено | Предпочтительно |

### Пример сравнения

```ts
let anyValue: any = 5;
let unknownValue: unknown = 5;

// С any можно делать что угодно
anyValue.toFixed(); // OK
anyValue(); // OK (хотя это ошибка во время выполнения)
anyValue.field = "test"; // OK

// С unknown нужно проверить тип
if (typeof unknownValue === "number") {
  unknownValue.toFixed(); // OK
}

// unknownValue.toFixed(); // Ошибка: неизвестный тип
```

## Практические применения

### any в реальных сценариях

```ts
// Работа с JSON-данными
function parseJSON(json: string): any {
  return JSON.parse(json);
}

const data = parseJSON('{"name": "Иван", "age": 30}');
console.log(data.name); // Работает, но нет проверки типов
```

### unknown в реальных сценариях

```ts
// Более безопасная альтернатива для JSON
function safeParseJSON(json: string): unknown {
  try {
    return JSON.parse(json);
  } catch {
    return undefined;
  }
}

const data = safeParseJSON('{"name": "Иван", "age": 30}');

if (data && typeof data === "object" && data !== null && "name" in data) {
  // Здесь мы можем быть уверены в структуре данных
  console.log((data as { name: string }).name);
}
```

## Правила использования

### Когда использовать any

- При интеграции с JavaScript-библиотеками без определений типов
- При постепенной миграции JavaScript-кода на TypeScript
- В крайних случаях, когда точный тип не может быть определен

### Когда использовать unknown

- Когда тип значения действительно неизвестен
- В API, которые могут принимать значения любого типа
- В местах, где безопасность типов важнее удобства

## Рекомендации

1. **Предпочитайте `unknown` типу `any`** - это более безопасный выбор
2. **Используйте проверки типов** при работе с `unknown`
3. **Избегайте `any`** в новых проектах - стремитесь к строгой типизации
4. **Рассмотрите использование утилит типов** для более сложных сценариев

## Заключение

Типы `any` и `unknown` предоставляют разные уровни безопасности при работе с неизвестными типами. Тип `unknown` является более безопасной альтернативой `any`, требуя явной проверки типов перед использованием значений. Понимание различий между этими типами помогает писать более надежный и безопасный код.

Для продолжения изучения рекомендуется ознакомиться с [[never-и-void]] и [[Контроль-типов]].

## Дополнительные ресурсы

- [[Типы-данных]] - основные типы данных в TypeScript
- [[never-и-void]] - типы never и void
- [[Контроль-типов]] - управление типами в TypeScript
- [[Утилиты-типов]] - встроенные утилиты для работы с типами