# TypeScript Arrays

Массивы в TypeScript расширяют возможности массивов JavaScript, предоставляя строгую типизацию элементов и богатые возможности для манипуляции данными. TypeScript поддерживает различные способы объявления типов массивов и предоставляет безопасные методы для работы с ними.

## Объявление типов массивов

### Основные способы объявления
```typescript
// Синтаксис с квадратными скобками
let stringArray: string[] = ["hello", "world", "typescript"];
let numberArray: number[] = [1, 2, 3, 4, 5];
let booleanArray: boolean[] = [true, false, true];

// Обобщенный синтаксис Array<T>
let genericStringArray: Array<string> = ["hello", "world"];
let genericNumberArray: Array<number> = [10, 20, 30];

// Оба способа эквивалентны
let arr1: string[] = ["a", "b", "c"];
let arr2: Array<string> = ["a", "b", "c"];
```

### Массивы с объединением типов
```typescript
// Массив с несколькими возможными типами элементов
let mixedArray: (string | number)[] = ["hello", 42, "world", 123];
let unionArray: Array<string | number | boolean> = ["test", 123, true];

// Использование в функциях
function processMixed(items: (string | number)[]): string[] {
  return items.map(item => 
    typeof item === "string" ? item.toUpperCase() : item.toString()
  );
}

const result = processMixed(["hello", 42, "world"]); // ["HELLO", "42", "WORLD"]
```

### Readonly массивы
```typescript
// Массив только для чтения
let readonlyArray: readonly string[] = ["hello", "world"];
// readonlyArray.push("!"); // ОШИБКА: push не доступен для readonly массивов
// readonlyArray[0] = "changed"; // ОШИБКА: нельзя изменить элемент

// Использование типа ReadonlyArray
let readonlyArray2: ReadonlyArray<number> = [1, 2, 3, 4];
// readonlyArray2.push(5); // ОШИБКА: методы изменения недоступны

// Конвертация обычного массива в readonly
let mutableArray = [1, 2, 3];
let immutableArray: readonly number[] = mutableArray as const; // или просто использовать как readonly
```

## Создание и инициализация массивов

### Различные способы создания
```typescript
// Создание пустого массива
let emptyArray: string[] = [];
let emptyArray2: Array<number> = new Array();
let emptyArray3: Array<boolean> = new Array<boolean>();

// Создание массива с начальными значениями
let initializedArray: string[] = ["first", "second", "third"];
let numbers: number[] = [1, 1, 2, 3, 5, 8, 13];

// Использование конструктора с размером
let sizedArray: number[] = new Array(5); // [empty × 5] - 5 элементов, undefined

// Создание массива с заполнением
let filledArray: string[] = new Array(3).fill("default"); // ["default", "default", "default"]
```

### Создание из других коллекций
```typescript
// Создание из строки
let stringToArray: string[] = "hello".split(""); // ["h", "e", "l", "l", "o"]

// Создание из других массивов
let sourceArray: number[] = [1, 2, 3];
let newArray: number[] = [...sourceArray]; // копирование через spread

// Создание из arguments или подобных коллекций
function createArrayFromArguments(...items: string[]) {
  let argsArray: string[] = Array.from(arguments); // Не рекомендуется в TypeScript
  let properArray: string[] = items; // Лучше использовать rest параметры
  return properArray;
}

// Использование Array.from для создания с преобразованием
let transformedArray: number[] = Array.from("12345", char => parseInt(char));
// [1, 2, 3, 4, 5]
```

## Основные методы массивов

### Методы изменения (мутаторы)
```typescript
let fruits: string[] = ["apple", "banana"];

// push - добавить в конец
fruits.push("orange"); // ["apple", "banana", "orange"]

// pop - удалить с конца
let lastFruit = fruits.pop(); // "orange", fruits = ["apple", "banana"]

// unshift - добавить в начало
fruits.unshift("mango"); // ["mango", "apple", "banana"]

// shift - удалить с начала
let firstFruit = fruits.shift(); // "mango", fruits = ["apple", "banana"]

// splice - вставить/удалить элементы в произвольной позиции
fruits.splice(1, 1, "grape", "kiwi"); // ["mango", "grape", "kiwi", "banana"]
// начиная с индекса 1, удалить 1 элемент и вставить "grape", "kiwi"

// sort - сортировка
let numbers: number[] = [3, 1, 4, 1, 5];
numbers.sort(); // [1, 1, 3, 4, 5] - мутирует оригинальный массив

// reverse - реверс
numbers.reverse(); // [5, 4, 3, 1, 1]
```

### Методы доступа (не мутирующие)
```typescript
let colors: string[] = ["red", "green", "blue", "yellow"];

// concat - объединение массивов
let moreColors: string[] = colors.concat(["purple", "orange"]);
// ["red", "green", "blue", "yellow", "purple", "orange"]

// slice - получение подмассива
let someColors: string[] = colors.slice(1, 3); // ["green", "blue"]

// join - объединение элементов в строку
let colorString: string = colors.join(", "); // "red, green, blue, yellow"

// indexOf/lastIndexOf - поиск индекса
let redIndex: number = colors.indexOf("red"); // 0
let notFoundIndex: number = colors.indexOf("pink"); // -1

// includes - проверка на вхождение
let hasBlue: boolean = colors.includes("blue"); // true

// Методы доступа к элементам
let firstColor: string | undefined = colors[0];
let lastColor: string | undefined = colors[colors.length - 1];
let safeAccess: string | undefined = colors[10]; // undefined, если индекс не существует
```

## Итерация по массивам

### Циклы и перебор
```typescript
let items: string[] = ["apple", "banana", "orange"];

// for цикл
for (let i = 0; i < items.length; i++) {
  console.log(items[i]);
}

// for...of - итерация по значениям
for (const item of items) {
  console.log(item); // "apple", "banana", "orange"
}

// for...in - итерация по индексам
for (const index in items) {
  console.log(`${index}: ${items[index]}`);
}

// forEach - функциональный подход
items.forEach((item, index) => {
  console.log(`${index}: ${item}`);
});
```

### Функциональные методы перебора
```typescript
let numbers: number[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map - преобразование элементов
let doubled: number[] = numbers.map(n => n * 2); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

// filter - фильтрация элементов
let evens: number[] = numbers.filter(n => n % 2 === 0); // [2, 4, 6, 8, 10]

// find - найти первый соответствующий элемент
let firstEven: number | undefined = numbers.find(n => n % 2 === 0); // 2

// findIndex - найти индекс первого соответствующего элемента
let evenIndex: number = numbers.findIndex(n => n % 2 === 0); // 1

// every - проверка, что все элементы соответствуют условию
let allPositive: boolean = numbers.every(n => n > 0); // true

// some - проверка, что хотя бы один элемент соответствует условию
let hasEven: boolean = numbers.some(n => n % 2 === 0); // true
```

## Продвинутые методы массивов

### Сводящие методы
```typescript
let scores: number[] = [85, 92, 78, 96, 88];

// reduce - свертка массива в одно значение
let totalScore: number = scores.reduce((sum, score) => sum + score, 0); // 439
let averageScore: number = scores.reduce((sum, score) => sum + score, 0) / scores.length;

// reduce с объектом аккумулятором
interface ScoreStats {
  count: number;
  sum: number;
  max: number;
  min: number;
}

let stats: ScoreStats = scores.reduce((acc: ScoreStats, score) => ({
  count: acc.count + 1,
  sum: acc.sum + score,
  max: Math.max(acc.max, score),
  min: Math.min(acc.min, score)
}), { count: 0, sum: 0, max: -Infinity, min: Infinity });

// reduceRight - свертка справа налево
let concatenated: string = ["a", "b", "c"].reduceRight((acc, val) => acc + val, ""); // "cba"
```

### Цепочки методов
```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

let products: Product[] = [
  { id: 1, name: "Laptop", price: 1000, category: "Electronics", inStock: true },
  { id: 2, name: "Book", price: 20, category: "Education", inStock: true },
  { id: 3, name: "Phone", price: 800, category: "Electronics", inStock: false },
  { id: 4, name: "Desk", price: 300, category: "Furniture", inStock: true }
];

// Цепочка методов для сложных преобразований
let expensiveElectronicsNames: string[] = products
  .filter(p => p.category === "Electronics")      // Отфильтровать электронику
  .filter(p => p.inStock)                         // Только в наличии
  .filter(p => p.price > 500)                     // Только дороже 500
  .map(p => p.name)                               // Выбрать только имена
  .map(name => name.toUpperCase());               // Преобразовать к верхнему регистру

// Результат: ["LAPTOP"]
```

## Двумерные и многомерные массивы

### Работа с матрицами
```typescript
// Двумерный массив
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Создание двумерного массива
function createMatrix(rows: number, cols: number, initialValue: number = 0): number[][] {
  return Array(rows).fill(null).map(() => Array(cols).fill(initialValue));
}

let newMatrix: number[][] = createMatrix(3, 4, 1);
// [ [1, 1, 1, 1], [1, 1, 1, 1], [1, 1, 1, 1] ]

// Перебор двумерного массива
matrix.forEach((row, rowIndex) => {
  row.forEach((cell, colIndex) => {
    console.log(`matrix[${rowIndex}][${colIndex}] = ${cell}`);
  });
});

// Трехмерный массив
let cube: number[][][] = Array(2).fill(null).map(() => 
  Array(3).fill(null).map(() => Array(4).fill(0))
);
// 2x3x4 трехмерный массив, заполненный нулями
```

## Использование с обобщениями

### Обобщенные функции для массивов
```typescript
// Обобщенная функция для получения последнего элемента
function lastElement<T>(arr: T[]): T | undefined {
  return arr.length > 0 ? arr[arr.length - 1] : undefined;
}

const lastNum = lastElement([1, 2, 3]); // number | undefined
const lastStr = lastElement(["a", "b", "c"]); // string | undefined

// Обобщенная функция для перестановки элементов
function shuffle<T>(array: T[]): T[] {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

// Обобщенная функция для разделения массива
function chunk<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

const chunked = chunk([1, 2, 3, 4, 5, 6, 7], 3); 
// [[1, 2, 3], [4, 5, 6], [7]]
```

## Практические примеры

### Работа с данными пользователей
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
  role: "admin" | "user" | "moderator";
}

let users: User[] = [
  { id: 1, name: "Alice", email: "alice@example.com", isActive: true, role: "user" },
  { id: 2, name: "Bob", email: "bob@example.com", isActive: false, role: "user" },
  { id: 3, name: "Charlie", email: "charlie@example.com", isActive: true, role: "admin" },
  { id: 4, name: "Diana", email: "diana@example.com", isActive: true, role: "moderator" }
];

// Найти активных администраторов
let activeAdmins: User[] = users.filter(u => u.isActive && u.role === "admin");

// Получить только имена пользователей в верхнем регистре
let upperNames: string[] = users
  .filter(u => u.isActive)
  .map(u => u.name.toUpperCase());

// Группировка по ролям
interface GroupedUsers {
  [role: string]: User[];
}

let grouped: GroupedUsers = users.reduce((acc, user) => {
  if (!acc[user.role]) {
    acc[user.role] = [];
  }
  acc[user.role].push(user);
  return acc;
}, {} as GroupedUsers);
```

### Работа с асинхронными массивами
```typescript
// Параллельная обработка элементов массива
async function processItemsAsync<T, R>(items: T[], processor: (item: T) => Promise<R>): Promise<R[]> {
  return Promise.all(items.map(processor));
}

// Последовательная обработка
async function processItemsSequential<T, R>(items: T[], processor: (item: T) => Promise<R>): Promise<R[]> {
  const results: R[] = [];
  for (const item of items) {
    results.push(await processor(item));
  }
  return results;
}

// Пример использования
async function fetchUserData(id: number): Promise<string> {
  // симуляция асинхронной операции
  return `User ${id}`;
}

// const users = await processItemsAsync([1, 2, 3], fetchUserData);
```

## Частые ошибки и ловушки

### Мутирующие методы
```typescript
// Осторожность с мутирующими методами
let originalArray: number[] = [1, 2, 3, 4, 5];

// Эти методы мутируют оригинальный массив:
// originalArray.push(6)    // изменяет originalArray
// originalArray.pop()      // изменяет originalArray
// originalArray.sort()     // изменяет originalArray

// Для иммутабельности используйте spread или копирование:
let newArrayAfterPush = [...originalArray, 6]; // не изменяет originalArray
let sortedCopy = [...originalArray].sort();    // не изменяет originalArray

// Ошибка: изменение элементов
let objects: { value: number }[] = [{ value: 1 }, { value: 2 }];
objects.forEach(obj => obj.value++); // изменяет оригинальные объекты!
```

### Работа с undefined
```typescript
// Осторожность с доступом к элементам
let arr: string[] = ["hello", "world"];
// let element: string = arr[5]; // тип: string, значение: undefined - потенциальная ошибка!

// Безопасный доступ
function safeGet<T>(array: T[], index: number): T | undefined {
  return index >= 0 && index < array.length ? array[index] : undefined;
}

let safeElement = safeGet(arr, 5); // безопасно, возвращает undefined
```

### Выход за границы
```typescript
// Осторожность с индексами
function getNthElement<T>(arr: T[], n: number): T | undefined {
  // Проверка на отрицательные индексы и выход за границы
  if (n < 0 || n >= arr.length) {
    return undefined;
  }
  return arr[n];
}
```

## Лучшие практики

### Использование функциональных методов
```typescript
// Предпочтение функциональным методам над циклами
// Вместо:
let numbers: number[] = [1, 2, 3, 4, 5];
let doubled: number[] = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// Лучше:
let doubledBetter: number[] = numbers.map(n => n * 2);
```

### Создание безопасных оберток
```typescript
// Создание оберток для безопасной работы с массивами
class SafeArray<T> {
  private data: T[] = [];

  constructor(initialData?: T[]) {
    this.data = initialData ? [...initialData] : [];
  }

  push(item: T): void {
    this.data.push(item);
  }

  get(index: number): T | undefined {
    return index >= 0 && index < this.data.length ? this.data[index] : undefined;
  }

  length(): number {
    return this.data.length;
  }

  slice(start?: number, end?: number): T[] {
    return this.data.slice(start, end);
  }

  map<U>(callback: (value: T, index: number, array: T[]) => U): U[] {
    return this.data.map(callback);
  }

  filter(predicate: (value: T, index: number, array: T[]) => boolean): T[] {
    return this.data.filter(predicate);
  }
}
```

## Связь с другими концепциями
- [[objects]] - работа с массивами объектов
- [[functions]] - передача функций в методы массивов
- [[generics]] - обобщенные типы для массивов
- [[basics/control-flow]] - сужение типов в массивах
- [[advanced-types]] - продвинутые типы для массивов