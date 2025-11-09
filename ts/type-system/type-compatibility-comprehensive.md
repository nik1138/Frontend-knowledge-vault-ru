# Совместимость типов в TypeScript

Совместимость типов (type compatibility) - это система правил, по которым TypeScript определяет, можно ли использовать значение одного типа в контексте другого типа. Это ключевой аспект системы типов TypeScript, который определяет, когда один тип может быть использован вместо другого.

## Основные принципы совместимости

### Структурная типизация

TypeScript использует структурную (а не номинальную) типизацию, что означает, что совместимость определяется структурой типов, а не их именем:

```typescript
interface Point {
  x: number;
  y: number;
}

class Point2D {
  x: number;
  y: number;
  
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

// Совместимость: структура совпадает, несмотря на разные имена
const point: Point = new Point2D(1, 2);  // OK
```

## Совместимость примитивных типов

### 1. Совместимость строк и чисел с литералами

```typescript
// Строковые литералы совместимы со строками
let str: string = "hello";  // Строковый литерал совместим с string
const literal: "world" = "world";
let generalStr: string = literal;  // OK: строковый литерал совместим со string

// Но не наоборот без утверждения типа
// let specific: "world" = str;  // Ошибка: string несовместим с "world"

// То же самое с числами
let num: number = 42;  // Числовой литерал совместим с number
const literalNum: 42 = 42;
let generalNum: number = literalNum;  // OK
```

### 2. Совместимость null и undefined

```typescript
// По умолчанию null и undefined совместимы со всеми типами в нестрогом режиме
let str: string = "hello";
// str = null;  // Ошибка в strict mode
// str = undefined;  // Ошибка в strict mode

// Но можно использовать объединения
let nullableStr: string | null = "hello";
nullableStr = null;  // OK

let optionalStr: string | undefined = "hello";
optionalStr = undefined;  // OK
```

## Совместимость объектных типов

### 1. Совместимость по подмножеству свойств

```typescript
interface Named {
  name: string;
}

interface Person extends Named {
  age: number;
}

let person: Person = { name: "Alice", age: 30 };
let named: Named = person;  // OK: Person имеет все свойства Named

// Но не наоборот:
// let person2: Person = named;  // Ошибка: у Named нет свойства age
```

### 2. Дополнительные свойства

```typescript
interface Square {
  size: number;
}

// Можно присвоить объект с дополнительными свойствами
let square: Square = { size: 10, color: "red" };  // OK при присвоении переменной

// Но при прямом присвоении нужно быть осторожным
const obj = { size: 10, color: "red" };
let square2: Square = obj;  // OK: объявление через переменную

// Но при прямом присвоении в вызове функции:
function calculateArea(square: Square): number {
  return square.size * square.size;
}

// calculateArea({ size: 10, color: "red" });  // Ошибка: лишнее свойство
calculateArea(obj);  // OK: через переменную
```

## Совместимость функций

Функциональная совместимость более сложна и зависит от параметров и возвращаемого значения.

### 1. Контравариантность параметров

Параметры функции являются контравариантными - функция с более "узкими" параметрами может быть присвоена переменной с более "широкими" параметрами:

```typescript
interface Person {
  name: string;
  age: number;
}

interface Named {
  name: string;
}

let personFunc = (p: Person) => console.log(p.name);
let namedFunc = (n: Named) => console.log(n.name);

// namedFunc может принимать всё, что может принять Person
// Поэтому namedFunc может быть присвоена переменной типа PersonFunc
personFunc = namedFunc;  // OK: контравариантность параметров

// Но не наоборот
// namedFunc = personFunc;  // Ошибка: Person не может быть присвоен Named
```

### 2. Ковариантность возвращаемого значения

Возвращаемые значения функций ковариантны - функция с более "узким" возвращаемым типом может быть присвоена переменной с более "широким" возвращаемым типом:

```typescript
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

let getAnimal = (): Animal => ({ name: "some animal" });
let getDog = (): Dog => ({ name: "Buddy", breed: "Golden Retriever" });

// getDog возвращает Dog, который совместим с Animal
getAnimal = getDog;  // OK: ковариантность возвращаемого значения

// Но не наоборот
// getDog = getAnimal;  // Ошибка: Animal не может быть присвоен Dog
```

### 3. Количество параметров

```typescript
let funcA = (x: number, y: number) => x + y;
let funcB = (x: number, y: number, z: number) => x + y + z;

// funcB может быть присвоена funcA (лишний параметр игнорируется)
funcA = funcB;  // OK

// funcA может быть присвоена funcB (недостающий параметр получает undefined)
funcB = funcA;  // OK

// Вызов функций с разным количеством параметров
funcA(1, 2);      // OK
funcA(1, 2, 3);   // OK: третий параметр игнорируется
```

### 4. Совместимость функций и методов

```typescript
interface FuncInterface {
  (arg: string): number;
}

interface MethodInterface {
  method(arg: string): number;
}

let func: FuncInterface = (str: string) => str.length;
let obj: MethodInterface = {
  method: (str: string) => str.length
};

// Совместимость между функцией и методом
let methodFunc: FuncInterface = obj.method;  // OK
```

## Совместимость массивов и объектов

### 1. Совместимость массивов

```typescript
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

let animals: Animal[] = [];
let dogs: Dog[] = [{ name: "Buddy", breed: "Labrador" }];

// В TypeScript до версии 4.0 следующее было возможно из-за инвариантности массивов
// animals = dogs;  // Это может быть проблемой, так как можно добавить Animal в массив Dog

// В современном TypeScript:
// animals = dogs;  // Это может вызвать ошибку в зависимости от настроек
// dogs = animals;  // Ошибка: Animal не может быть присвоен Dog

// Но чтение работает:
let animal: Animal = dogs[0];  // OK: Dog совместим с Animal
```

### 2. Совместимость с индексными сигнатурами

```typescript
interface Dictionary {
  [key: string]: string;
}

interface NumberDictionary {
  [key: string]: number;
}

// let strDict: Dictionary = { a: "hello", b: "world" };
// let numDict: NumberDictionary = { a: 1, b: 2 };

// Совместимость индексных сигнатур:
// numDict = strDict;  // Ошибка: string несовместим с number
```

## Совместимость объединений и пересечений

### 1. Совместимость объединений

```typescript
type StringOrNumber = string | number;
type StringOrBoolean = string | boolean;

let sn: StringOrNumber = "hello";
let sb: StringOrBoolean = "world";

// Совместимость объединений основана на совместимости каждого члена
// sn = sb;  // Ошибка: boolean несовместим с number
// sb = sn;  // Ошибка: number несовместим с boolean

// Но строка совместима с обоими объединениями
let str: string = "test";
sn = str;  // OK
sb = str;  // OK
```

### 2. Совместимость пересечений

```typescript
interface A { a: string; }
interface B { b: number; }
interface C { c: boolean; }

type AB = A & B;
type ABC = A & B & C;

let ab: AB = { a: "hello", b: 42 };
let abc: ABC = { a: "world", b: 100, c: true };

// ABC может быть присвоен AB (пересечение имеет все свойства AB)
ab = abc;  // OK

// Но не наоборот
// abc = ab;  // Ошибка: у AB нет свойства c
```

## Совместимость перечислений (enums)

### 1. Совместимость числовых перечислений

```typescript
enum Status { Ready, Waiting }
enum Color { Red, Blue, Green }

let status = Status.Ready;
let color = Color.Red;

// status = color;  // Ошибка: разные перечисления
// color = status;  // Ошибка: разные перечисления

// Но числовые перечисления совместимы с number
let num: number = Status.Ready;  // OK
status = num;  // OK (но небезопасно)
```

### 2. Совместимость строковых перечислений

```typescript
enum HttpStatus { OK = "OK", NOT_FOUND = "NOT_FOUND" }
enum CustomStatus { OK = "OK", PENDING = "PENDING" }

let httpStatus = HttpStatus.OK;
let customStatus = CustomStatus.OK;

// httpStatus = customStatus;  // Ошибка: разные перечисления
// customStatus = httpStatus;  // Ошибка: разные перечисления

// Но строковые перечисления совместимы со строками
let str: string = HttpStatus.OK;  // OK
// HttpStatus.OK = str;  // Ошибка: нельзя присвоить строку перечислению
```

## Практические примеры

### 1. Совместимость в функциях обратного вызова

```typescript
interface Event {
  timestamp: number;
}

interface MouseEvent extends Event {
  x: number;
  y: number;
}

interface KeyEvent extends Event {
  keyCode: number;
}

function listenEvent(callback: (n: Event) => void) {
  // ...
}

// Эти вызовы допустимы благодаря контравариантности параметров
listenEvent((e: Event) => console.log(e.timestamp));  // OK
listenEvent((e: MouseEvent) => console.log(e.x + "," + e.y));  // OK
listenEvent((e: KeyEvent) => console.log(e.keyCode));  // OK

// Но следующее может быть небезопасно:
// listenEvent((e: MouseEvent) => console.log(e.x + "," + e.y));  
// Если функция попытается использовать e.x или e.y, 
// а будет передан KeyEvent, это вызовет ошибку выполнения
```

### 2. Совместимость при работе с DOM

```typescript
// HTMLElement более общий, чем HTMLInputElement
let element: HTMLElement = document.getElementById("myId");
let input: HTMLInputElement = document.getElementById("inputId") as HTMLInputElement;

// input может быть присвоен element (HTMLInputElement extends HTMLElement)
element = input;  // OK

// Но не наоборот без утверждения типа
// input = element;  // Ошибка: HTMLElement несовместим с HTMLInputElement
input = element as HTMLInputElement;  // OK: с утверждением типа
```

### 3. Совместимость при работе с классами

```typescript
class Animal {
  name: string;
  constructor(name: string) { this.name = name; }
  move() { console.log(`${this.name} moves`); }
}

class Dog extends Animal {
  breed: string;
  constructor(name: string, breed: string) {
    super(name);
    this.breed = breed;
  }
  bark() { console.log("Woof!"); }
}

let animal: Animal = new Dog("Buddy", "Golden Retriever");  // OK
// let dog: Dog = new Animal("Generic");  // Ошибка: Animal несовместим с Dog

// Вызов методов
animal.move();  // OK
// animal.bark();  // Ошибка: метод bark не доступен в Animal
```

## Продвинутые паттерны совместимости

### 1. Совместимость с обобщениями

```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;  // OK
// myIdentity = (arg: string) => arg;  // Ошибка: string несовместим с number
```

### 2. Совместимость с условными типами

```typescript
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  "object";

// Совместимость определяется результирующим типом условного типа
type StrType = TypeName<string>;  // "string"
type NumType = TypeName<number>;  // "number"

let strType: "string" = "string";
let numType: "number" = "number";

strType = StrType;  // OK
numType = NumType;  // OK
```

## Ошибки совместимости и как с ними работать

### 1. Использование утверждений типов

```typescript
interface ExpectedType {
  name: string;
  id: number;
}

// Когда нужно обойти проверку совместимости
const rawData: any = { name: "Alice", id: 123, extra: "data" };
const typedData = rawData as ExpectedType;  // OK: обход проверки
```

### 2. Создание промежуточных переменных

```typescript
interface Source {
  fullName: string;
  userId: number;
}

interface Target {
  name: string;
  id: number;
}

const source: Source = { fullName: "Alice Smith", userId: 123 };

// Прямое присвоение невозможно
// const target: Target = source;  // Ошибка: несовместимые свойства

// Решение: создать новый объект
const target: Target = {
  name: source.fullName,
  id: source.userId
};
```

## Лучшие практики

1. **Понимайте структурную типизацию** - TypeScript проверяет структуру, а не имена типов
2. **Будьте осторожны с контравариантностью параметров** - убедитесь, что функция может обработать все возможные типы параметров
3. **Используйте дискриминированные объединения** - для безопасной работы с разными типами
4. **Избегайте лишних свойств при прямом присвоении** - это может вызвать ошибки
5. **Понимайте различие между ковариантностью и контравариантностью** - особенно в функциях
6. **Используйте строгую проверку типов** - для обнаружения потенциальных проблем совместимости

## Связи с другими концепциями

- [[ts/type-system/structural-typing-comprehensive|Структурная типизация]] - основа системы совместимости
- [[ts/type-system/type-guards|Защитники типов]] - для безопасного сужения типов
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - для безопасной работы с объединениями
- [[ts/type-system/type-assertions-narrowing|Утверждения типов]] - когда нужно обойти проверку совместимости
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - влияние на совместимость