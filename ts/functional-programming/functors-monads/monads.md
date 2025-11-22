# Монады

Монады - это расширение концепции функторов, которое позволяет объединять вычисления, возвращающие монады, и управлять эффектами вычислений.

## Содержание

1. [Определение монады](#определение-монады)
2. [Законы монад](#законы-монад)
3. [Реализация монад](#реализация-монад)
4. [Различие между функторами и монадами](#различие-между-функторами-и-монадами)
5. [Практические примеры](#практические-примеры)
6. [Преимущества использования монад](#преимущества-использования-монад)
7. [Лучшие практики](#лучшие-практики)

## Определение монады

Монада - это функтор с дополнительными возможностями. Монада должна реализовывать метод `flatMap` (или `chain`), который позволяет объединять вычисления, возвращающие монады. Монада также должна иметь метод `of` (или `pure`) для создания монады из обычного значения.

```typescript
// Базовый интерфейс монады
interface Monad<T> extends Functor<T> {
  flatMap: <U>(fn: (value: T) => Monad<U>) => Monad<U>;
  of: (value: T) => Monad<T>;
}

// Простой пример монады - Identity
class Identity<T> implements Monad<T> {
  constructor(private value: T) {}
  
  static of<T>(value: T): Identity<T> {
    return new Identity(value);
  }
  
  map<U>(fn: (value: T) => U): Identity<U> {
    return new Identity(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => Identity<U>): Identity<U> {
    return fn(this.value);
  }
  
  getValue(): T {
    return this.value;
  }
}

// Использование
const result = Identity.of(5)
  .flatMap(x => Identity.of(x * 2))
  .flatMap(x => Identity.of(x + 1));

console.log(result.getValue()); // 11
```

## Законы монад

Монады должны подчиняться трем важным законам:

### 1. Левый закон единицы (Left Identity)
`Monad.of(a).flatMap(f) === f(a)`

```typescript
// Левый закон единицы
const testLeftIdentity = () => {
  const a = 5;
  const f = (x: number) => Identity.of(x * 2);
  
  const result1 = Identity.of(a).flatMap(f);
  const result2 = f(a);
  
  console.log(result1.getValue() === result2.getValue()); // true
};
```

### 2. Правый закон единицы (Right Identity)
`m.flatMap(Monad.of) === m`

```typescript
// Правый закон единицы
const testRightIdentity = () => {
  const m = Identity.of(5);
  
  const result1 = m.flatMap(Identity.of);
  const result2 = m;
  
  console.log(result1.getValue() === result2.getValue()); // true
};
```

### 3. Закон ассоциативности (Associativity)
`m.flatMap(f).flatMap(g) === m.flatMap(x => f(x).flatMap(g))`

```typescript
// Закон ассоциативности
const testAssociativity = () => {
  const m = Identity.of(5);
  
  const f = (x: number) => Identity.of(x * 2);
  const g = (x: number) => Identity.of(x + 1);
  
  const result1 = m.flatMap(f).flatMap(g);
  const result2 = m.flatMap(x => f(x).flatMap(g));
  
  console.log(result1.getValue() === result2.getValue()); // true
};
```

## Реализация монад

Рассмотрим различные реализации монад в TypeScript.

### 1. Maybe монада

```typescript
// Maybe монада для работы с потенциально отсутствующими значениями
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  // Дополнительные методы для удобства
  filter(predicate: (value: T) => boolean): MaybeMonad<T> {
    if (this.isNothing() || !predicate(this.value!)) {
      return MaybeMonad.nothing();
    }
    return this;
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
}

// Использование Maybe монады
const safeDivide = (a: number, b: number): MaybeMonad<number> => {
  if (b === 0) {
    return MaybeMonad.nothing();
  }
  return MaybeMonad.just(a / b);
};

const safeSqrt = (x: number): MaybeMonad<number> => {
  if (x < 0) {
    return MaybeMonad.nothing();
  }
  return MaybeMonad.just(Math.sqrt(x));
};

const result = MaybeMonad.of(16)
  .flatMap(x => safeDivide(x, 2))
  .flatMap(x => safeSqrt(x))
  .map(x => x * 2);

console.log(result.getValue()); // 4 (sqrt(16/2) * 2 = sqrt(8) * 2 ≈ 2.83 * 2 ≈ 5.66)
```

### 2. Either монада

```typescript
// Either монада для обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherMonad<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonad<E, A> {
    return new EitherMonad(value);
  }
  
  static right<E, A>(a: A): EitherMonad<E, A> {
    return new EitherMonad(right(a));
  }
  
  static left<E, A>(e: E): EitherMonad<E, A> {
    return new EitherMonad(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return new EitherMonad(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonad<E, B>): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return fn(this.value.right);
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
  
  // Дополнительные методы для удобства
  mapLeft<F>(fn: (e: E) => F): EitherMonad<F, A> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(fn(this.value.left)));
    }
    return new EitherMonad(right(this.value.right));
  }
  
  fold<B>(onLeft: (e: E) => B, onRight: (a: A) => B): B {
    return this.value._tag === "Left" 
      ? onLeft(this.value.left) 
      : onRight(this.value.right);
  }
}

// Использование Either монады
const parseNumber = (str: string): EitherMonad<string, number> => {
  const num = Number(str);
  return isNaN(num) 
    ? EitherMonad.left(`Невозможно преобразовать "${str}" в число`)
    : EitherMonad.right(num);
};

const validatePositive = (num: number): EitherMonad<string, number> => {
  return num > 0 
    ? EitherMonad.right(num)
    : EitherMonad.left(`Число должно быть положительным, получено: ${num}`);
};

const sqrt = (num: number): EitherMonad<string, number> => {
  return num >= 0
    ? EitherMonad.right(Math.sqrt(num))
    : EitherMonad.left(`Невозможно вычислить квадратный корень из ${num}`);
};

const result2 = EitherMonad.right("16")
  .flatMap(parseNumber)
  .flatMap(validatePositive)
  .flatMap(sqrt)
  .map(x => x * 2);

console.log(result2.getValue()); // { _tag: "Right", right: 8 }
```

### 3. IO монада

```typescript
// IO монада для работы с побочными эффектами
class IOMonad<A> {
  constructor(private computation: () => A) {}
  
  static of<A>(value: A): IOMonad<A> {
    return new IOMonad(() => value);
  }
  
  map<B>(fn: (a: A) => B): IOMonad<B> {
    return new IOMonad(() => fn(this.computation()));
  }
  
  flatMap<B>(fn: (a: A) => IOMonad<B>): IOMonad<B> {
    return new IOMonad(() => fn(this.computation()).computation());
  }
  
  run(): A {
    return this.computation();
  }
}

// Использование IO монады
const readLine = (): IOMonad<string> => {
  return new IOMonad(() => {
    // В реальном приложении здесь был бы реальный ввод
    return "Привет, мир!";
  });
};

const writeLine = (message: string): IOMonad<void> => {
  return new IOMonad(() => {
    console.log(message);
  });
};

const program = readLine()
  .map(str => str.toUpperCase())
  .flatMap(str => writeLine(`Обработанная строка: ${str}`));

// Выполнение программы
program.run();
```

## Различие между функторами и монадами

Основное различие между функторами и монадами заключается в том, что:

1. **Функтор** позволяет применять функции к значениям внутри контейнера с помощью `map`
2. **Монада** позволяет объединять вычисления, возвращающие контейнеры, с помощью `flatMap`

```typescript
// Пример, демонстрирующий разницу
const demonstrateDifference = () => {
  // Функция, возвращающая Maybe
  const safeDivide = (a: number, b: number): MaybeMonad<number> => {
    if (b === 0) {
      return MaybeMonad.nothing();
    }
    return MaybeMonad.just(a / b);
  };
  
  // С функтором мы получим вложенные контейнеры
  const functorResult = MaybeMonad.of(10)
    .map(x => safeDivide(x, 2)); // MaybeMonad<MaybeMonad<number>>
  
  // С монадой мы получаем плоский результат
  const monadResult = MaybeMonad.of(10)
    .flatMap(x => safeDivide(x, 2)); // MaybeMonad<number>
  
  console.log(functorResult.getValue()); // MaybeMonad<number> (вложенный)
  console.log(monadResult.getValue()); // number или null (плоский)
};
```

## Практические примеры

Рассмотрим практические примеры использования монад в реальных приложениях.

```typescript
// Пример 1: Обработка данных пользователя с валидацией
interface UserInput {
  name: string;
  email: string;
  age: string;
}

interface ValidatedUser {
  name: string;
  email: string;
  age: number;
}

// Валидаторы, возвращающие Either
const validateName = (input: UserInput): EitherMonad<string, UserInput & { name: string }> => {
  const name = input.name.trim();
  if (name.length < 2) {
    return EitherMonad.left("Имя должно содержать минимум 2 символа");
  }
  return EitherMonad.right({ ...input, name });
};

const validateEmail = (input: UserInput & { name: string }): EitherMonad<string, UserInput & { name: string, email: string }> => {
  const email = input.email.trim();
  if (!email.includes("@")) {
    return EitherMonad.left("Неверный формат email");
  }
  return EitherMonad.right({ ...input, email });
};

const validateAge = (input: UserInput & { name: string, email: string }): EitherMonad<string, ValidatedUser> => {
  const age = parseInt(input.age, 10);
  if (isNaN(age) || age < 18) {
    return EitherMonad.left("Возраст должен быть числом не менее 18");
  }
  return EitherMonad.right({
    name: input.name,
    email: input.email,
    age
  });
};

// Композиция валидации с использованием монады
const validateUser = (input: UserInput): EitherMonad<string, ValidatedUser> => {
  return EitherMonad.right(input)
    .flatMap(validateName)
    .flatMap(validateEmail)
    .flatMap(validateAge);
};

// Использование
const validInput: UserInput = {
  name: "Иван",
  email: "ivan@example.com",
  age: "25"
};

const invalidInput: UserInput = {
  name: "И",
  email: "invalid-email",
  age: "15"
};

const validResult = validateUser(validInput);
const invalidResult = validateUser(invalidInput);

console.log(validResult.getValue()); // { _tag: "Right", right: ValidatedUser }
console.log(invalidResult.getValue()); // { _tag: "Left", left: "Имя должно содержать минимум 2 символа" }

// Пример 2: Работа с опциональными значениями в объектах
interface Address {
  street: string;
  city?: string;
  country?: string;
}

interface User {
  id: number;
  name: string;
  address?: Address;
}

// Функции для безопасного доступа к вложенным свойствам
const getUserAddress = (user: User): MaybeMonad<Address> => {
  return MaybeMonad.of(user.address);
};

const getAddressCity = (address: Address): MaybeMonad<string> => {
  return MaybeMonad.of(address.city);
};

const getCountry = (address: Address): MaybeMonad<string> => {
  return MaybeMonad.of(address.country);
};

// Композиция для получения города пользователя
const getUserCity = (user: User): MaybeMonad<string> => {
  return MaybeMonad.of(user)
    .flatMap(getUserAddress)
    .flatMap(getAddressCity);
};

// Использование
const userWithAddress: User = {
  id: 1,
  name: "Иван",
  address: {
    street: "Ленина 1",
    city: "Москва",
    country: "Россия"
  }
};

const userWithoutAddress: User = {
  id: 2,
  name: "Мария"
};

const city1 = getUserCity(userWithAddress);
const city2 = getUserCity(userWithoutAddress);

console.log(city1.getValue()); // "Москва"
console.log(city2.getValue()); // null
```

## Преимущества использования монад

1. **Композиция эффектов** - Монады позволяют легко объединять вычисления с эффектами
2. **Безопасная обработка ошибок** - Either монада обеспечивает безопасную обработку ошибок
3. **Явное управление побочными эффектами** - IO монада делает побочные эффекты явными
4. **Предсказуемость** - Монады подчиняются строгим законам
5. **Повторное использование** - Один и тот же интерфейс для разных типов эффектов

## Лучшие практики

### 1. Соблюдение законов монад

```typescript
// Хорошо: соблюдение законов
class GoodMonad<T> {
  constructor(private value: T) {}
  
  static of<T>(value: T): GoodMonad<T> {
    return new GoodMonad(value);
  }
  
  map<U>(fn: (value: T) => U): GoodMonad<U> {
    return new GoodMonad(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => GoodMonad<U>): GoodMonad<U> {
    return fn(this.value);
  }
}

// Плохо: нарушение законов
class BadMonad<T> {
  constructor(private value: T) {}
  
  static of<T>(value: T): BadMonad<T> {
    return new BadMonad(value);
  }
  
  map<U>(fn: (value: T) => U): BadMonad<U> {
    return new BadMonad(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => BadMonad<U>): BadMonad<U> {
    // Нарушение правого закона единицы
    return new BadMonad(fn(this.value).getValue() + " modified");
  }
  
  getValue(): T {
    return this.value;
  }
}
```

### 2. Явная обработка эффектов

```typescript
// Хорошо: явная обработка эффектов с помощью монад
const safeComputation = (input: number): EitherMonad<string, number> => {
  if (input < 0) {
    return EitherMonad.left("Входное значение не может быть отрицательным");
  }
  
  return EitherMonad.right(Math.sqrt(input));
};

// Плохо: скрытая обработка ошибок
const unsafeComputation = (input: number): number => {
  if (input < 0) {
    throw new Error("Входное значение не может быть отрицательным");
  }
  
  return Math.sqrt(input);
};
```

### 3. Иммутабельность

```typescript
// Хорошо: иммутабельные монады
class ImmutableMonad<T> {
  constructor(private readonly value: T) {}
  
  static of<T>(value: T): ImmutableMonad<T> {
    return new ImmutableMonad(value);
  }
  
  map<U>(fn: (value: T) => U): ImmutableMonad<U> {
    return new ImmutableMonad(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => ImmutableMonad<U>): ImmutableMonad<U> {
    return fn(this.value);
  }
  
  getValue(): T {
    return this.value;
  }
}

// Плохо: мутация внутреннего состояния
class MutableMonad<T> {
  constructor(private value: T) {}
  
  static of<T>(value: T): MutableMonad<T> {
    return new MutableMonad(value);
  }
  
  map<U>(fn: (value: T) => U): MutableMonad<U> {
    this.value = fn(this.value as any); // Мутация!
    return this as any;
  }
  
  flatMap<U>(fn: (value: T) => MutableMonad<U>): MutableMonad<U> {
    this.value = fn(this.value).getValue() as any; // Мутация!
    return this as any;
  }
}
```

## Связи с другими концепциями

- [[ts/functional-programming/functors-monads/Функторы_и_монады|Функторы и монады]] - Основная страница раздела
- [[ts/functional-programming/functors-monads/functors|Функторы]] - Базовая концепция функторов
- [[ts/functional-programming/functors-monads/common-monads|Распространенные монады]] - Изучение популярных монад
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[Обработка ошибок в TypeScript|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[Тестирование в TypeScript|Тестирование]] - Тестирование монад

## Следующие шаги

- [[ts/functional-programming/functors-monads/common-monads|Распространенные монады]] - Изучение популярных монад
- [[ts/functional-programming/functors-monads/practical-examples|Практические примеры]] - Реальные примеры использования
- [[Обработка ошибок в TypeScript|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
- [[Тестирование в TypeScript|Тестирование]] - Тестирование монад
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация монад