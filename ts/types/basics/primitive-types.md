# Primitive Types

Примитивные типы в TypeScript - это базовые необъектные типы, которые не являются объектами и не имеют методов. Это основа всей системы типов TypeScript.

## JavaScript примитивы

### String
Тип `string` представляет строковые данные:

```typescript
// Литералы строк
let color: string = "blue";
let city: string = 'Paris';
let message: string = `Hello, ${city}!`;

// Шаблонные строки
let user = { name: "Alice", age: 30 };
let greeting: string = `Hello, ${user.name}. You are ${user.age} years old.`;

// Строки как объекты при выполнении (но не типизированы как объекты)
console.log("hello".toUpperCase()); // "HELLO" - работает во время выполнения
```

### Number
Тип `number` включает все числовые значения:

```typescript
// Целые числа
let decimal: number = 6;
let hex: number = 0xf00d;     // шестнадцатеричные
let binary: number = 0b1010;  // двоичные  
let octal: number = 0o744;    // восьмеричные

// Дробные числа
let float: number = 3.14159;
let scientific: number = 6e4; // 60000

// Специальные числовые значения
let infinity: number = Infinity;
let negativeInfinity: number = -Infinity;
let notANumber: number = NaN;

// Числовые методы
let num = 123.456;
console.log(num.toFixed(2)); // "123.46"
```

### Boolean
Тип `boolean` имеет только два значения: `true` и `false`:

```typescript
let isDone: boolean = false;
let isActive: boolean = true;

// Важно: не путать с new Boolean()!
let boolObject: Boolean = new Boolean(1); // Объект, а не примитив!
let boolPrimitive: boolean = Boolean(1);   // Примитив boolean

// Практические примеры
function isValid(value: string): boolean {
  return value.length > 0 && value.includes('@');
}

// Условия и булевы значения
if (isActive) {
  console.log("User is active");
}
```

## Новые примитивы ES2015+

### Symbol
Тип `symbol` для уникальных идентификаторов:

```typescript
// Создание символов
let sym1: symbol = Symbol();
let sym2: symbol = Symbol("description");
let sym3: symbol = Symbol("description"); // тот же описатель, но уникальный символ

// Символы уникальны
console.log(sym2 === sym3); // false - разные символы, несмотря на одинаковое описание

// Глобальный реестр символов
let globalSym1: symbol = Symbol.for("globalKey");
let globalSym2: symbol = Symbol.for("globalKey");  // тот же самый символ
console.log(globalSym1 === globalSym2); // true

// Использование символов
const id = Symbol("id");
const user = {
  name: "Alice",
  [id]: 123
};

console.log(user[id]); // 123 - безопасный приватный ключ
```

### BigInt
Тип `bigint` для целых чисел произвольной длины:

```typescript
// Создание BigInt
let bigNumber: bigint = 1234567890123456789012345678901234567890n;
let anotherBig: bigint = BigInt(123); // через конструктор
let fromString: bigint = BigInt("999999999999999999999999999999");

// Операции с BigInt
let a: bigint = 10n;
let b: bigint = 20n;
let sum: bigint = a + b;        // 30n
let product: bigint = a * b;    // 200n

// BigInt несовместим с number
// let mixed = a + 1;  // Ошибка: bigint и number несовместимы
let mixed = a + BigInt(1);      // OK: BigInt(1)

// Операции сравнения
console.log(a < b);             // true
console.log(a === 10n);         // true
```

## Специальные примитивы

### Undefined
Тип `undefined` для неинициализированных переменных:

```typescript
// Тип undefined
let notInitialized: undefined;
// notInitialized = 1;  // Ошибка: undefined не может содержать другие значения
notInitialized = undefined; // OK

// Переменная без инициализации
let uninitializedVar; // тип: any (если не strict)
// В strict mode тип будет: undefined | <other types>

// Проверка undefined
function checkUndefined(value: any) {
  if (value === undefined) {
    console.log("Value is undefined");
  }
}
```

### Null
Тип `null` для специально присвоенного "ничего":

```typescript
// Тип null
let nullValue: null = null;
// nullValue = undefined; // Ошибка: типы несовместимы
// nullValue = 0;        // Ошибка: null и 0 разные типы

// null vs undefined
console.log(null == undefined);  // true (при ==)
console.log(null === undefined); // false (при ===)

// Практическое использование
interface User {
  id: number;
  name: string;
  email?: string; // необязательное, может быть undefined
  profile?: Profile | null; // может быть объектом или null
}
```

## Совместимость примитивов

### Строгий и не строгий режимы
```typescript
// Не строгий режим (noImplicitAny: false)
let inferredString = "hello"; // string
let inferredNumber = 42;      // number
let inferredBoolean = true;   // boolean

// Строгий режим (strict: true)
// null и undefined несовместимы с другими типами
let name: string = "Alice";
// name = null;    // Ошибка в strict mode
// name = undefined; // Ошибка в strict mode
```

### Автоматические преобразования
```typescript
// Автоматическое преобразование строк
let str: string = "123";
let num: number = 456;
let result: string = str + num;  // "123456"
let result2: number = +str;      // 123 (унарный плюс)

// Сравнение разных типов
console.log("5" == 5);  // true (не строгое равенство)
console.log("5" === 5); // false (строгое равенство)
```

## Продвинутые паттерны с примитивами

### Литеральные типы из примитивов
```typescript
// Строковые литеральные типы
type Direction = "up" | "down" | "left" | "right";
type HttpStatus = 200 | 400 | 404 | 500;

function move(direction: Direction) {
  // direction теперь может быть только одним из перечисленных
}

// Числовые литеральные типы
type DiceValue = 1 | 2 | 3 | 4 | 5 | 6;
type Weekday = 1 | 2 | 3 | 4 | 5;

// Булевые литеральные типы
type YesNo = true | false;
type FeatureFlag = "enabled" | "disabled";

// Практический пример
interface ApiResponse<T> {
  status: 200 | 400 | 401 | 404 | 500;
  data?: T;
  success: true | false;
}
```

### Утилиты для примитивов
```typescript
// Преобразование типов
type StringToNumber<T extends string> = T extends `${infer Num extends number}` ? Num : never;

// Проверка на примитивы
type IsPrimitive<T> = 
  T extends string | number | boolean | bigint | symbol | null | undefined 
  ? true 
  : false;

type T1 = IsPrimitive<string>;     // true
type T2 = IsPrimitive<{a: string}>; // false
```

## Лучшие практики

1. **Предпочитайте конкретные литеральные типы** абстрактным примитивам:
   - `status: "pending" | "completed"` лучше чем `status: string`

2. **Используйте объединения** для расширения примитивов:
   - `userId: number | string` для гибкости

3. **Будьте осторожны с null и undefined** в strict режиме:
   - Используйте `string | null` явно, когда null допустим

4. **Используйте символы для приватных ключей** в объектах:
   - Обеспечивает настоящую приватность на уровне типов

5. **BigInt для больших чисел**:
   - Используйте BigInt для чисел, превышающих Number.MAX_SAFE_INTEGER

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - совместимость примитивов
- [[literal-types]] - продвинутые литеральные типы
- [[union-types]] - объединения примитивов
- [[type-system/type-inference]] - вывод примитивных типов