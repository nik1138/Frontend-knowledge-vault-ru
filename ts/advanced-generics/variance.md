# Вариантность дженериков

Вариантность - это концепция, описывающая как обобщенные типы ведут себя при наследовании. Понимание вариантности важно для создания типобезопасных и гибких дженериков.

## Основные виды вариантности

### 1. Ковариантность (Covariance)

Ковариантность означает, что если тип A является подтипом типа B, то Generic<A> также является подтипом Generic<B>.

```typescript
// Пример ковариантности с массивами
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Dog extends Animal {
  breed: string;
  constructor(name: string, breed: string) {
    super(name);
    this.breed = breed;
  }
}

// Массивы ковариантны в TypeScript
const dogs: Dog[] = [new Dog("Buddy", "Golden Retriever")];
const animals: Animal[] = dogs; // ✅ Ковариантность

// Однако это может привести к проблемам:
// animals.push(new Animal("Generic Animal")); // Это создаст Dog[] с Animal внутри!
```

### 2. Контравариантность (Contravariance)

Контравариантность означает, что если тип A является подтипом типа B, то Generic<B> является подтипом Generic<A>.

```typescript
// Пример контравариантности с функциями
type AnimalHandler = (animal: Animal) => void;
type DogHandler = (dog: Dog) => void;

// Функции контравариантны по параметрам
const handleAnimal: AnimalHandler = (animal: Animal) => {
  console.log(`Animal: ${animal.name}`);
};

const handleDog: DogHandler = (dog: Dog) => {
  console.log(`Dog: ${dog.name}, Breed: ${dog.breed}`);
};

// Контравариантность: AnimalHandler может быть присвоен DogHandler
// потому что Animal является супертипом Dog
let handler: DogHandler = handleAnimal; // ✅ Контравариантность
```

### 3. Инвариантность (Invariance)

Инвариантность означает, что Generic<A> и Generic<B> не имеют отношения подтипирования, даже если A и B связаны.

```typescript
// Пример инвариантности
interface Box<T> {
  value: T;
}

// Box<Dog> и Box<Animal> не связаны отношениями подтипирования
const dogBox: Box<Dog> = { value: new Dog("Buddy", "Golden Retriever") };
// const animalBox: Box<Animal> = dogBox; // ❌ Ошибка: инвариантность
```

## Вариантность в TypeScript

### 1. Массивы и кортежи (ковариантны)

```typescript
// Массивы ковариантны
const numbers: number[] = [1, 2, 3];
const unknowns: unknown[] = numbers; // ✅

// Кортежи тоже ковариантны
const tuple: [number, string] = [1, "hello"];
const unknownTuple: [unknown, unknown] = tuple; // ✅
```

### 2. Функции (контравариантны по параметрам, ковариантны по возвращаемому значению)

```typescript
// Контравариантность по параметрам
type Func1 = (a: Animal) => void;
type Func2 = (a: Dog) => void;

const f1: Func1 = (a: Animal) => console.log(a.name);
const f2: Func2 = (d: Dog) => console.log(d.name, d.breed);

// Func1 может быть присвоен Func2, потому что Animal супертип Dog
let func: Func2 = f1; // ✅

// Ковариантность по возвращаемому значению
type Func3 = () => Animal;
type Func4 = () => Dog;

const f3: Func3 = () => new Animal("Generic");
const f4: Func4 = () => new Dog("Buddy", "Golden Retriever");

// Func4 может быть присвоен Func3, потому что Dog подтип Animal
let func2: Func3 = f4; // ✅
```

### 3. Объединения и пересечения

```typescript
// Объединения ковариантны
type Union1 = string | number;
type Union2 = string | number | boolean;

// Union1 является подтипом Union2
const u1: Union1 = "hello";
const u2: Union2 = u1; // ✅

// Пересечения контравариантны
type Intersection1 = { a: string } & { b: number };
type Intersection2 = { a: string };

// Intersection1 является подтипом Intersection2
const i1: Intersection1 = { a: "hello", b: 42 };
const i2: Intersection2 = i1; // ✅
```

## Практические примеры

### 1. Создание типобезопасного контейнера

```typescript
// Инвариантный контейнер
interface InvariantBox<T> {
  value: T;
  get(): T;
  set(value: T): void;
}

// Ковариантный контейнер (только для чтения)
interface CovariantBox<out T> { // TypeScript 4.7+
  get(): T;
}

// Контравариантный контейнер (только для записи)
interface ContravariantBox<in T> { // TypeScript 4.7+
  set(value: T): void;
}

// Реализация
class ReadOnlyBox<T> implements CovariantBox<T> {
  constructor(private value: T) {}
  
  get(): T {
    return this.value;
  }
}

class WriteOnlyBox<T> implements ContravariantBox<T> {
  private value: T | undefined;
  
  set(value: T): void {
    this.value = value;
  }
}

// Использование
const dogBox = new ReadOnlyBox(new Dog("Buddy", "Golden Retriever"));
const animalBox: CovariantBox<Animal> = dogBox; // ✅ Ковариантность
```

### 2. Типобезопасные коллекции

```typescript
// Ковариантная коллекция (только для чтения)
interface ReadOnlyCollection<out T> {
  get(index: number): T;
  size(): number;
}

// Контравариантная коллекция (только для записи)
interface WriteOnlyCollection<in T> {
  add(item: T): void;
}

// Инвариантная коллекция (чтение и запись)
interface ReadWriteCollection<T> {
  get(index: number): T;
  add(item: T): void;
  size(): number;
}

// Реализация
class AnimalCollection implements ReadOnlyCollection<Animal> {
  private items: Animal[] = [];
  
  get(index: number): Animal {
    return this.items[index];
  }
  
  size(): number {
    return this.items.length;
  }
  
  add(animal: Animal): void {
    this.items.push(animal);
  }
}

// Использование
const dogCollection = new AnimalCollection();
const animalCollection: ReadOnlyCollection<Animal> = dogCollection; // ✅
```

### 3. Функциональные интерфейсы

```typescript
// Ковариантность по возвращаемому значению
interface Producer<out T> {
  produce(): T;
}

// Контравариантность по параметрам
interface Consumer<in T> {
  consume(item: T): void;
}

// Инвариантность при чтении и записи
interface Processor<T> {
  process(item: T): T;
}

// Реализации
class StringProducer implements Producer<string> {
  produce(): string {
    return "Hello";
  }
}

class UnknownProducer implements Producer<unknown> {
  produce(): unknown {
    return Math.random() > 0.5 ? "string" : 42;
  }
}

// Использование ковариантности
const stringProd: Producer<string> = new StringProducer();
const unknownProd: Producer<unknown> = stringProd; // ✅

class AnimalConsumer implements Consumer<Animal> {
  consume(item: Animal): void {
    console.log(`Consuming animal: ${item.name}`);
  }
}

class DogConsumer implements Consumer<Dog> {
  consume(item: Dog): void {
    console.log(`Consuming dog: ${item.name}, breed: ${item.breed}`);
  }
}

// Использование контравариантности
const animalCons: Consumer<Animal> = new AnimalConsumer();
const dogCons: Consumer<Dog> = animalCons; // ✅
```

## Проблемы и решения

### 1. Проблема с массивами

```typescript
// Проблема: массивы ковариантны, но это может быть небезопасно
const dogs: Dog[] = [new Dog("Buddy", "Golden Retriever")];
const animals: Animal[] = dogs; // ✅ Ковариантность

// Это может привести к ошибке во время выполнения
// animals.push(new Animal("Generic Animal")); // ❌ Ошибка!

// Решение: использовать readonly массивы для ковариантности
const readonlyDogs: readonly Dog[] = [new Dog("Buddy", "Golden Retriever")];
const readonlyAnimals: readonly Animal[] = readonlyDogs; // ✅ Безопасно

// readonlyAnimals.push(new Animal("Generic Animal")); // ❌ Ошибка компиляции
```

### 2. Проблема с функциями

```typescript
// Проблема: контравариантность может быть неочевидной
type EventHandler<T extends Event> = (event: T) => void;

// MouseEvent является подтипом Event
// EventHandler<MouseEvent> должен быть подтипом EventHandler<Event>
// Но это означает обратное отношение параметров

// Решение: явно указывать вариантность
type CovariantEventHandler<out T extends Event> = (event: T) => void;
type ContravariantEventHandler<in T extends Event> = (event: T) => void;
```

## Рекомендации

### 1. Для разработчиков библиотек

```typescript
// Используйте явную вариантность в TypeScript 4.7+
interface ReadOnlyList<out T> {
  get(index: number): T;
}

interface WriteOnlyList<in T> {
  set(index: number, value: T): void;
}

// Или документируйте ожидаемое поведение
/**
 * Интерфейс для только чтения.
 * Поддерживает ковариантность по типу T.
 */
interface IReadonlyCollection<T> {
  get(index: number): T;
}
```

### 2. Для потребителей библиотек

```typescript
// Понимайте вариантность используемых типов
const dogArray: Dog[] = [new Dog("Buddy", "Golden Retriever")];

// Если вам нужна ковариантность, используйте readonly
const animalArray: readonly Animal[] = dogArray;

// Если вам нужна контравариантность, используйте функции
type AnimalProcessor = (animal: Animal) => void;
type DogProcessor = (dog: Dog) => void;

const processAnimal: AnimalProcessor = (animal) => console.log(animal.name);
const processDog: DogProcessor = processAnimal; // Контравариантность
```

## Связанные концепции

- [[ts/advanced-generics/generic-constraints|Ограничения дженериков]]
- [[ts/type-system/type-compatibility|Совместимость типов]]
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]]
- [[ts/type-system/structural-typing|Структурная типизация]]