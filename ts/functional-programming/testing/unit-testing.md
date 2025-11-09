# Модульное тестирование функционального кода

Модульное тестирование в функциональном программировании фокусируется на тестировании отдельных функций и их свойств. Благодаря чистоте функций и отсутствию побочных эффектов, модульное тестирование становится более предсказуемым и надежным.

## Содержание

- [Модульное тестирование функционального кода](#модульное-тестирование-функционального-кода)
  - [Содержание](#содержание)
  - [Основы модульного тестирования](#основы-модульного-тестирования)
  - [Тестирование чистых функций](#тестирование-чистых-функций)
  - [Тестирование функций высшего порядка](#тестирование-функций-высшего-порядка)
  - [Тестирование каррированных функций](#тестирование-каррированных-функций)
  - [Тестирование композиции](#тестирование-композиции)
  - [Тестирование функторов](#тестирование-функторов)
  - [Тестирование монад](#тестирование-монад)
  - [Тестирование с использованием фикстур](#тестирование-с-использованием-фикстур)
  - [Лучшие практики](#лучшие-практики)
    - [1. Тестирование пограничных случаев](#1-тестирование-пограничных-случаев)
    - [2. Использование property-based тестирования](#2-использование-property-based-тестирования)
    - [3. Тестирование законов структур](#3-тестирование-законов-структур)
    - [4. Изоляция побочных эффектов](#4-изоляция-побочных-эффектов)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Основы модульного тестирования

Модульное тестирование в функциональном программировании имеет несколько ключевых принципов:

1. **Тестируемость по умолчанию** - Чистые функции легко тестировать
2. **Предсказуемость** - Одинаковые входные данные всегда дают одинаковые результаты
3. **Изоляция** - Функции не зависят от внешнего состояния
4. **Композиция тестируемости** - Композиция тестируемых функций остается тестируемой

```typescript
// Пример чистой функции для тестирования
const add = (a: number, b: number): number => a + b;

// Пример функции с побочными эффектами (труднее тестировать)
const addWithSideEffect = (a: number, b: number): number => {
  console.log(`Сложение ${a} и ${b}`); // Побочный эффект
  return a + b;
};

// Функциональный подход к побочным эффектам (тестируемо)
const addWithLogger = (a: number, b: number, logger: (msg: string) => void): number => {
  logger(`Сложение ${a} и ${b}`);
  return a + b;
};

// Тестирование
describe("Основы модульного тестирования", () => {
  test("add должен корректно складывать числа", () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
    expect(add(0, 0)).toBe(0);
  });

  test("addWithLogger должен складывать числа и логгировать", () => {
    const mockLogger = jest.fn();
    const result = addWithLogger(2, 3, mockLogger);
    
    expect(result).toBe(5);
    expect(mockLogger).toHaveBeenCalledWith("Сложение 2 и 3");
  });
});
```

## Тестирование чистых функций

Чистые функции - основа функционального программирования и самый простой объект для тестирования.

```typescript
// Различные типы чистых функций
const multiply = (a: number, b: number): number => a * b;

const isEven = (n: number): boolean => n % 2 === 0;

const filterEven = <T extends number>(arr: T[]): T[] => arr.filter(isEven);

const sum = (arr: number[]): number => arr.reduce((acc, curr) => acc + curr, 0);

const max = (arr: number[]): number => Math.max(...arr);

const min = (arr: number[]): number => Math.min(...arr);

const average = (arr: number[]): number => {
  if (arr.length === 0) return 0;
  return sum(arr) / arr.length;
};

const capitalize = (str: string): string => 
  str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();

const reverse = <T>(arr: T[]): T[] => [...arr].reverse();

const unique = <T>(arr: T[]): T[] => [...new Set(arr)];

// Тесты для чистых функций
describe("Тестирование чистых функций", () => {
  test("multiply должен корректно умножать числа", () => {
    expect(multiply(2, 3)).toBe(6);
    expect(multiply(-2, 3)).toBe(-6);
    expect(multiply(0, 5)).toBe(0);
    expect(multiply(1, 7)).toBe(7);
  });

  test("isEven должен корректно определять четные числа", () => {
    expect(isEven(2)).toBe(true);
    expect(isEven(3)).toBe(false);
    expect(isEven(0)).toBe(true);
    expect(isEven(-2)).toBe(true);
    expect(isEven(-3)).toBe(false);
  });

  test("filterEven должен фильтровать четные числа", () => {
    expect(filterEven([1, 2, 3, 4, 5, 6])).toEqual([2, 4, 6]);
    expect(filterEven([1, 3, 5])).toEqual([]);
    expect(filterEven([2, 4, 6])).toEqual([2, 4, 6]);
    expect(filterEven([])).toEqual([]);
  });

  test("sum должен корректно суммировать массив чисел", () => {
    expect(sum([1, 2, 3, 4, 5])).toBe(15);
    expect(sum([])).toBe(0);
    expect(sum([-1, 1])).toBe(0);
    expect(sum([10, -5, 3])).toBe(8);
  });

  test("max должен находить максимальное значение", () => {
    expect(max([1, 2, 3, 4, 5])).toBe(5);
    expect(max([-1, -2, -3])).toBe(-1);
    expect(max([0])).toBe(0);
  });

  test("min должен находить минимальное значение", () => {
    expect(min([1, 2, 3, 4, 5])).toBe(1);
    expect(min([-1, -2, -3])).toBe(-3);
    expect(min([0])).toBe(0);
  });

  test("average должен корректно рассчитывать среднее значение", () => {
    expect(average([1, 2, 3, 4, 5])).toBe(3);
    expect(average([10, 20, 30])).toBe(20);
    expect(average([])).toBe(0);
    expect(average([5])).toBe(5);
  });

  test("capitalize должен корректно капитализировать строку", () => {
    expect(capitalize("hello")).toBe("Hello");
    expect(capitalize("WORLD")).toBe("World");
    expect(capitalize("tYpEsCrIpT")).toBe("Typescript");
    expect(capitalize("")).toBe("");
  });

  test("reverse должен корректно реверсировать массив", () => {
    expect(reverse([1, 2, 3, 4, 5])).toEqual([5, 4, 3, 2, 1]);
    expect(reverse(["a", "b", "c"])).toEqual(["c", "b", "a"]);
    expect(reverse([])).toEqual([]);
  });

  test("unique должен удалять дубликаты", () => {
    expect(unique([1, 2, 2, 3, 3, 3])).toEqual([1, 2, 3]);
    expect(unique(["a", "b", "a", "c", "b"])).toEqual(["a", "b", "c"]);
    expect(unique([])).toEqual([]);
  });
});
```

## Тестирование функций высшего порядка

Функции высшего порядка принимают или возвращают другие функции, что требует специального подхода к тестированию.

```typescript
// Функции высшего порядка
const map = <T, U>(fn: (item: T) => U) => (arr: T[]): U[] => 
  arr.map(fn);

const filter = <T>(predicate: (item: T) => boolean) => (arr: T[]): T[] => 
  arr.filter(predicate);

const reduce = <T, U>(fn: (acc: U, item: T) => U, initial: U) => (arr: T[]): U => 
  arr.reduce(fn, initial);

const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): ((a: A) => C) => 
  (a: A) => f(g(a));

const pipe = <A, B, C>(f: (a: A) => B, g: (b: B) => C): ((a: A) => C) => 
  (a: A) => g(f(a));

// Каррированные функции высшего порядка
const mapWith = <T, U>(fn: (item: T) => U) => (arr: T[]): U[] => 
  arr.map(fn);

const filterBy = <T>(predicate: (item: T) => boolean) => (arr: T[]): T[] => 
  arr.filter(predicate);

// Тестирование функций высшего порядка
describe("Тестирование функций высшего порядка", () => {
  test("map должен применять функцию ко всем элементам", () => {
    const double = (x: number): number => x * 2;
    const mapDouble = map(double);
    
    expect(mapDouble([1, 2, 3, 4])).toEqual([2, 4, 6, 8]);
    expect(mapDouble([])).toEqual([]);
  });

  test("filter должен фильтровать элементы по предикату", () => {
    const isPositive = (x: number): boolean => x > 0;
    const filterPositive = filter(isPositive);
    
    expect(filterPositive([-2, -1, 0, 1, 2])).toEqual([1, 2]);
    expect(filterPositive([-2, -1, 0])).toEqual([]);
  });

  test("reduce должен аккумулировать значения", () => {
    const sumReducer = (acc: number, item: number): number => acc + item;
    const reduceSum = reduce(sumReducer, 0);
    
    expect(reduceSum([1, 2, 3, 4])).toBe(10);
    expect(reduceSum([])).toBe(0);
  });

  test("compose должен компоновать функции справа налево", () => {
    const addOne = (x: number): number => x + 1;
    const multiplyByTwo = (x: number): number => x * 2;
    
    const composed = compose(multiplyByTwo, addOne);
    expect(composed(5)).toBe(12); // (5 + 1) * 2 = 12
  });

  test("pipe должен компоновать функции слева направо", () => {
    const addOne = (x: number): number => x + 1;
    const multiplyByTwo = (x: number): number => x * 2;
    
    const piped = pipe(addOne, multiplyByTwo);
    expect(piped(5)).toBe(12); // (5 + 1) * 2 = 12
  });

  test("mapWith должен работать как каррированная версия map", () => {
    const square = (x: number): number => x * x;
    const mapSquare = mapWith(square);
    
    expect(mapSquare([1, 2, 3])).toEqual([1, 4, 9]);
  });

  test("filterBy должен работать как каррированная версия filter", () => {
    const isEven = (x: number): boolean => x % 2 === 0;
    const filterEven = filterBy(isEven);
    
    expect(filterEven([1, 2, 3, 4, 5, 6])).toEqual([2, 4, 6]);
  });
});
```

## Тестирование каррированных функций

Каррированные функции требуют тестирования всех уровней каррирования.

```typescript
// Каррированные функции
const add = (a: number) => (b: number): number => a + b;

const multiply = (a: number) => (b: number) => (c: number): number => a * b * c;

const filterBy = <T>(predicate: (item: T) => boolean) => (array: T[]): T[] => 
  array.filter(predicate);

const mapWith = <T, U>(fn: (item: T) => U) => (array: T[]): U[] => 
  array.map(fn);

// Универсальная функция каррирования
function curry<T extends any[], R>(fn: (...args: T) => R): Curried<T, R> {
  return function curried(...args: any[]): any {
    if (args.length >= fn.length) {
      return fn.apply(null, args);
    }
    
    return (...nextArgs: any[]) => curried(...args, ...nextArgs);
  } as Curried<T, R>;
}

type Curried<T extends any[], R> = 
  T extends [infer First, ...infer Rest] 
    ? (arg: First) => Curried<Rest, R>
    : R;

// Тестирование каррированных функций
describe("Тестирование каррированных функций", () => {
  test("add должен работать с частичным применением", () => {
    const addFive = add(5);
    expect(addFive(3)).toBe(8);
    expect(addFive(10)).toBe(15);
    
    // Полное применение
    expect(add(2)(3)).toBe(5);
  });

  test("multiply должен работать с многоуровневым каррированием", () => {
    const multiplyByTwo = multiply(2);
    const multiplyByTwoAndThree = multiplyByTwo(3);
    
    expect(multiplyByTwoAndThree(4)).toBe(24);
    
    // Частичное применение
    expect(multiply(2)(3)(4)).toBe(24);
    expect(multiply(1)(2)(3)).toBe(6);
  });

  test("filterBy должен корректно фильтровать массивы", () => {
    const isEven = (x: number): boolean => x % 2 === 0;
    const filterEven = filterBy(isEven);
    
    expect(filterEven([1, 2, 3, 4, 5, 6])).toEqual([2, 4, 6]);
    expect(filterEven([1, 3, 5])).toEqual([]);
  });

  test("mapWith должен корректно преобразовывать массивы", () => {
    const square = (x: number): number => x * x;
    const mapSquare = mapWith(square);
    
    expect(mapSquare([1, 2, 3, 4])).toEqual([1, 4, 9, 16]);
    expect(mapSquare([])).toEqual([]);
  });

  test("curry должен корректно каррировать функции", () => {
    const addThree = (a: number, b: number, c: number): number => a + b + c;
    const curriedAddThree = curry(addThree);
    
    expect(curriedAddThree(1)(2)(3)).toBe(6);
    expect(curriedAddThree(1, 2)(3)).toBe(6);
    expect(curriedAddThree(1)(2, 3)).toBe(6);
    expect(curriedAddThree(1, 2, 3)).toBe(6);
  });
});
```

## Тестирование композиции

Композиция функций требует тестирования как отдельных функций, так и их комбинаций.

```typescript
// Утилиты для композиции
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): ((a: A) => C) => 
  (a: A) => f(g(a));

const pipe = <A, B, C>(f: (a: A) => B, g: (b: B) => C): ((a: A) => C) => 
  (a: A) => g(f(a));

// Функции для композиции
const add = (x: number): number => x + 1;
const multiply = (x: number): number => x * 2;
const square = (x: number): number => x * x;
const toString = (x: number): string => x.toString();

// Сложные композиции
const complexOperation1 = compose(square, multiply);
const complexOperation2 = pipe(add, multiply, square);
const complexOperation3 = compose(toString, square, multiply, add);

// Тестирование композиции
describe("Тестирование композиции", () => {
  test("compose должен компоновать функции справа налево", () => {
    const composed = compose(multiply, add);
    expect(composed(5)).toBe(12); // (5 + 1) * 2 = 12
  });

  test("pipe должен компоновать функции слева направо", () => {
    const piped = pipe(add, multiply);
    expect(piped(5)).toBe(12); // (5 + 1) * 2 = 12
  });

  test("сложная композиция 1", () => {
    expect(complexOperation1(3)).toBe(36); // (3 * 2) ^ 2 = 36
  });

  test("сложная композиция 2", () => {
    expect(complexOperation2(3)).toBe(64); // ((3 + 1) * 2) ^ 2 = 64
  });

  test("сложная композиция 3", () => {
    expect(complexOperation3(3)).toBe("49"); // ((3 + 1) * 2) ^ 2 = 49 -> "49"
  });

  test("compose и pipe должны давать одинаковый результат при обратном порядке", () => {
    const piped = pipe(add, multiply);
    const composed = compose(multiply, add);
    
    const input = 5;
    expect(piped(input)).toBe(composed(input));
  });

  test("ассоциативность композиции", () => {
    const f = (x: number): number => x + 1;
    const g = (x: number): number => x * 2;
    const h = (x: number): number => x - 3;
    
    const leftSide = compose(compose(h, g), f);
    const rightSide = compose(h, compose(g, f));
    
    const input = 5;
    expect(leftSide(input)).toBe(rightSide(input));
  });
});
```

## Тестирование функторов

Функторы должны подчиняться законам, которые также нужно тестировать.

```typescript
// Maybe функтор
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
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
}

// Either функтор
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
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
}

// Тестирование законов функторов
describe("Тестирование законов функторов", () => {
  // Закон идентичности: functor.map(x => x) === functor
  test("Закон идентичности для Maybe", () => {
    const identity = <T>(x: T): T => x;
    
    // Тест с Just значением
    const justValue = MaybeMonad.just(5);
    expect(justValue.map(identity).getValue()).toBe(justValue.getValue());
    
    // Тест с Nothing
    const nothingValue = MaybeMonad.nothing<number>();
    expect(nothingValue.map(identity).isNothing()).toBe(true);
  });

  test("Закон идентичности для Either", () => {
    const identity = <T>(x: T): T => x;
    
    // Тест с Right значением
    const rightValue = EitherMonad.right(5);
    expect(rightValue.map(identity).getValue()).toEqual(rightValue.getValue());
    
    // Тест с Left значением
    const leftValue = EitherMonad.left("error");
    expect(leftValue.map(identity).getValue()).toEqual(leftValue.getValue());
  });

  // Закон композиции: functor.map(f).map(g) === functor.map(x => g(f(x)))
  test("Закон композиции для Maybe", () => {
    const f = (x: number): number => x + 1;
    const g = (x: number): number => x * 2;
    
    const justValue = MaybeMonad.just(5);
    
    const leftSide = justValue.map(f).map(g);
    const rightSide = justValue.map(x => g(f(x)));
    
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });

  test("Закон композиции для Either", () => {
    const f = (x: number): number => x + 1;
    const g = (x: number): number => x * 2;
    
    const rightValue = EitherMonad.right(5);
    
    const leftSide = rightValue.map(f).map(g);
    const rightSide = rightValue.map(x => g(f(x)));
    
    expect(leftSide.getValue()).toEqual(rightSide.getValue());
  });
});
```

## Тестирование монад

Монады должны подчиняться дополнительным законам по сравнению с функторами.

```typescript
// Maybe монада с flatMap
class MaybeMonadWithFlatMap<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonadWithFlatMap<T> {
    return new MaybeMonadWithFlatMap(value);
  }
  
  static just<T>(value: T): MaybeMonadWithFlatMap<T> {
    return new MaybeMonadWithFlatMap(value);
  }
  
  static nothing<T>(): MaybeMonadWithFlatMap<T> {
    return new MaybeMonadWithFlatMap<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonadWithFlatMap<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonadWithFlatMap<U>(null);
    }
    return new MaybeMonadWithFlatMap<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonadWithFlatMap<U>): MaybeMonadWithFlatMap<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonadWithFlatMap<U>(null);
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
}

// Either монада с flatMap
class EitherMonadWithFlatMap<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonadWithFlatMap<E, A> {
    return new EitherMonadWithFlatMap(value);
  }
  
  static right<E, A>(a: A): EitherMonadWithFlatMap<E, A> {
    return new EitherMonadWithFlatMap(right(a));
  }
  
  static left<E, A>(e: E): EitherMonadWithFlatMap<E, A> {
    return new EitherMonadWithFlatMap(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonadWithFlatMap<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonadWithFlatMap(left(this.value.left));
    }
    return new EitherMonadWithFlatMap(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonadWithFlatMap<E, B>): EitherMonadWithFlatMap<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonadWithFlatMap(left(this.value.left));
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
}

// Тестирование законов монад
describe("Тестирование законов монад", () => {
  // Левый закон единицы: Monad.of(a).flatMap(f) === f(a)
  test("Левый закон единицы для Maybe", () => {
    const a = 5;
    const f = (x: number) => MaybeMonadWithFlatMap.just(x * 2);
    
    const leftSide = MaybeMonadWithFlatMap.of(a).flatMap(f);
    const rightSide = f(a);
    
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });

  test("Левый закон единицы для Either", () => {
    const a = 5;
    const f = (x: number) => EitherMonadWithFlatMap.right(x * 2);
    
    const leftSide = EitherMonadWithFlatMap.right(a).flatMap(f);
    const rightSide = f(a);
    
    expect(leftSide.getValue()).toEqual(rightSide.getValue());
  });

  // Правый закон единицы: m.flatMap(Monad.of) === m
  test("Правый закон единицы для Maybe", () => {
    const m = MaybeMonadWithFlatMap.just(5);
    
    const leftSide = m.flatMap(x => MaybeMonadWithFlatMap.of(x));
    const rightSide = m;
    
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });

  test("Правый закон единицы для Either", () => {
    const m = EitherMonadWithFlatMap.right(5);
    
    const leftSide = m.flatMap(x => EitherMonadWithFlatMap.of(x));
    const rightSide = m;
    
    expect(leftSide.getValue()).toEqual(rightSide.getValue());
  });

  // Закон ассоциативности: m.flatMap(f).flatMap(g) === m.flatMap(x => f(x).flatMap(g))
  test("Закон ассоциативности для Maybe", () => {
    const m = MaybeMonadWithFlatMap.just(5);
    
    const f = (x: number) => MaybeMonadWithFlatMap.just(x * 2);
    const g = (x: number) => MaybeMonadWithFlatMap.just(x + 1);
    
    const leftSide = m.flatMap(f).flatMap(g);
    const rightSide = m.flatMap(x => f(x).flatMap(g));
    
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });

  test("Закон ассоциативности для Either", () => {
    const m = EitherMonadWithFlatMap.right(5);
    
    const f = (x: number) => EitherMonadWithFlatMap.right(x * 2);
    const g = (x: number) => EitherMonadWithFlatMap.right(x + 1);
    
    const leftSide = m.flatMap(f).flatMap(g);
    const rightSide = m.flatMap(x => f(x).flatMap(g));
    
    expect(leftSide.getValue()).toEqual(rightSide.getValue());
  });
});
```

## Тестирование с использованием фикстур

Фикстуры помогают создавать повторно используемые тестовые данные.

```typescript
// Фикстуры для тестирования
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

interface Order {
  id: number;
  userId: number;
  items: OrderItem[];
  total: number;
  status: string;
}

interface OrderItem {
  productId: number;
  quantity: number;
  price: number;
}

const createUserFixture = (overrides: Partial<User> = {}): User => ({
  id: Math.floor(Math.random() * 1000000),
  name: "Тестовый пользователь",
  email: "test@example.com",
  createdAt: new Date(),
  ...overrides
});

const createProductFixture = (overrides: Partial<Product> = {}): Product => ({
  id: Math.floor(Math.random() * 1000000),
  name: "Тестовый продукт",
  price: 100,
  inStock: true,
  ...overrides
});

const createOrderItemFixture = (overrides: Partial<OrderItem> = {}): OrderItem => ({
  productId: Math.floor(Math.random() * 1000000),
  quantity: 1,
  price: 100,
  ...overrides
});

const createOrderFixture = (overrides: Partial<Order> = {}): Order => ({
  id: Math.floor(Math.random() * 1000000),
  userId: Math.floor(Math.random() * 1000000),
  items: [createOrderItemFixture()],
  total: 100,
  status: "pending",
  ...overrides
});

// Функции для тестирования
const calculateOrderTotal = (order: Order): number => 
  order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

const isOrderValid = (order: Order): boolean => 
  order.items.length > 0 && order.total >= 0;

const formatUser = (user: User): string => 
  `${user.name} (${user.email})`;

const findUserByEmail = (users: User[], email: string): User | undefined => 
  users.find(user => user.email === email);

// Тесты с фикстурами
describe("Тестирование с фикстурами", () => {
  test("calculateOrderTotal должен корректно рассчитывать общую сумму", () => {
    const order = createOrderFixture({
      items: [
        createOrderItemFixture({ price: 100, quantity: 2 }),
        createOrderItemFixture({ price: 50, quantity: 1 })
      ]
    });
    
    expect(calculateOrderTotal(order)).toBe(250);
  });

  test("isOrderValid должен корректно валидировать заказы", () => {
    const validOrder = createOrderFixture();
    const invalidOrder = createOrderFixture({ items: [], total: -100 });
    
    expect(isOrderValid(validOrder)).toBe(true);
    expect(isOrderValid(invalidOrder)).toBe(false);
  });

  test("formatUser должен корректно форматировать пользователя", () => {
    const user = createUserFixture({ name: "Иван Иванов", email: "ivan@example.com" });
    expect(formatUser(user)).toBe("Иван Иванов (ivan@example.com)");
  });

  test("findUserByEmail должен находить пользователя по email", () => {
    const users = [
      createUserFixture({ email: "user1@example.com" }),
      createUserFixture({ email: "user2@example.com" }),
      createUserFixture({ email: "user3@example.com" })
    ];
    
    const foundUser = findUserByEmail(users, "user2@example.com");
    expect(foundUser).toBeDefined();
    expect(foundUser?.email).toBe("user2@example.com");
  });

  test("findUserByEmail должен возвращать undefined для несуществующего email", () => {
    const users = [
      createUserFixture({ email: "user1@example.com" }),
      createUserFixture({ email: "user2@example.com" })
    ];
    
    const foundUser = findUserByEmail(users, "nonexistent@example.com");
    expect(foundUser).toBeUndefined();
  });
});
```

## Лучшие практики

### 1. Тестирование пограничных случаев

```typescript
// Хорошо: тщательное тестирование пограничных случаев
const divide = (a: number, b: number): number => {
  if (b === 0) {
    throw new Error("Деление на ноль");
  }
  return a / b;
};

describe("Тестирование пограничных случаев", () => {
  test("divide должен корректно делить числа", () => {
    expect(divide(10, 2)).toBe(5);
    expect(divide(7, 3)).toBeCloseTo(2.333, 3);
  });
  
  test("divide должен бросать ошибку при делении на ноль", () => {
    expect(() => divide(10, 0)).toThrow("Деление на ноль");
    expect(() => divide(0, 0)).toThrow("Деление на ноль");
  });
  
  test("divide должен корректно работать с отрицательными числами", () => {
    expect(divide(-10, 2)).toBe(-5);
    expect(divide(10, -2)).toBe(-5);
    expect(divide(-10, -2)).toBe(5);
  });
  
  test("divide должен корректно работать с дробными числами", () => {
    expect(divide(0.1, 0.2)).toBeCloseTo(0.5, 10);
    expect(divide(1, 3)).toBeCloseTo(0.333, 3);
  });
});
```

### 2. Использование property-based тестирования

```typescript
// Хорошо: использование property-based тестирования
const sortArray = <T>(arr: T[]): T[] => [...arr].sort();

describe("Property-based тестирование", () => {
  test("результат всегда отсортирован", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sortArray(arr);
        for (let i = 0; i < sorted.length - 1; i++) {
          expect(sorted[i] <= sorted[i + 1]).toBe(true);
        }
      })
    );
  });
  
  test("результат содержит те же элементы", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sortArray(arr);
        const originalElements = [...arr].sort((a, b) => a - b);
        const sortedElements = [...sorted].sort((a, b) => a - b);
        expect(sortedElements).toEqual(originalElements);
      })
    );
  });
  
  test("длина результата равна длине исходного массива", () => {
    fc.assert(
      fc.property(fc.array(fc.integer()), (arr) => {
        const sorted = sortArray(arr);
        expect(sorted.length).toBe(arr.length);
      })
    );
  });
});
```

### 3. Тестирование законов структур

```typescript
// Хорошо: тестирование законов функторов и монад
describe("Тестирование законов структур", () => {
  // Законы функторов
  test("закон идентичности для функторов", () => {
    const identity = <T>(x: T): T => x;
    const functor = MaybeMonad.just(5);
    expect(functor.map(identity).getValue()).toBe(functor.getValue());
  });
  
  test("закон композиции для функторов", () => {
    const f = (x: number): number => x + 1;
    const g = (x: number): number => x * 2;
    const functor = MaybeMonad.just(5);
    
    const leftSide = functor.map(f).map(g);
    const rightSide = functor.map(x => g(f(x)));
    
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });
  
  // Законы монад
  test("законы монад", () => {
    const m = MaybeMonad.just(5);
    const f = (x: number) => MaybeMonad.just(x * 2);
    const g = (x: number) => MaybeMonad.just(x + 1);
    
    // Левый закон единицы
    expect(MaybeMonad.of(5).flatMap(f).getValue()).toBe(f(5).getValue());
    
    // Правый закон единицы
    expect(m.flatMap(x => MaybeMonad.of(x)).getValue()).toBe(m.getValue());
    
    // Закон ассоциативности
    const leftSide = m.flatMap(f).flatMap(g);
    const rightSide = m.flatMap(x => f(x).flatMap(g));
    expect(leftSide.getValue()).toBe(rightSide.getValue());
  });
});
```

### 4. Изоляция побочных эффектов

```typescript
// Хорошо: изоляция побочных эффектов
interface Dependencies {
  fetchUser: (id: number) => Promise<User | null>;
  sendEmail: (email: string, message: string) => Promise<boolean>;
  log: (message: string) => void;
}

const processUser = async (
  userId: number, 
  deps: Dependencies
): Promise<boolean> => {
  deps.log(`Начало обработки пользователя ${userId}`);
  
  const user = await deps.fetchUser(userId);
  if (!user) {
    deps.log(`Пользователь ${userId} не найден`);
    return false;
  }
  
  const success = await deps.sendEmail(
    user.email, 
    `Привет, ${user.name}!`
  );
  
  deps.log(`Отправка email ${success ? 'успешна' : 'не удалась'}`);
  return success;
};

// Тестирование с моками
describe("Тестирование с моками", () => {
  test("processUser должен возвращать false для несуществующего пользователя", async () => {
    const mockDeps: Dependencies = {
      fetchUser: async (id: number) => Promise.resolve(null),
      sendEmail: async (email: string, message: string) => Promise.resolve(true),
      log: jest.fn()
    };
    
    const result = await processUser(1, mockDeps);
    expect(result).toBe(false);
  });
  
  test("processUser должен возвращать true для существующего пользователя", async () => {
    const user = createUserFixture({ id: 1, name: "Иван", email: "ivan@example.com" });
    const mockDeps: Dependencies = {
      fetchUser: async (id: number) => Promise.resolve(user),
      sendEmail: async (email: string, message: string) => Promise.resolve(true),
      log: jest.fn()
    };
    
    const result = await processUser(1, mockDeps);
    expect(result).toBe(true);
  });
});
```

## Связи с другими концепциями

- [[ts/functional-programming/testing/Тестирование_функционального_кода|Тестирование функционального кода]] - Основная страница раздела
- [[ts/functional-programming/pure-functions|Чистые функции]] - Основа для тестируемости
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Тестирование функций, принимающих или возвращающих другие функции
- [[ts/functional-programming/currying|Каррирование]] - Тестирование каррированных функций
- [[ts/functional-programming/composition|Композиция функций]] - Тестирование композиции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Тестирование законов структур
- [[ts/testing/integration-testing|Интеграционное тестирование]] - Тестирование взаимодействия компонентов

## Следующие шаги

- [[ts/testing/integration-testing|Интеграционное тестирование]] - Тестирование взаимодействия компонентов
- [[ts/testing/e2e-testing|End-to-end тестирование]] - Тестирование полных сценариев
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация функционального кода
- [[ts/error-handling/index|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[ts/functional-programming/Функциональное_программирование|Функциональное программирование]] - Обзор всех концепций функционального программирования