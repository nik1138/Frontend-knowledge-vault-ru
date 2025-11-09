# Функторы

Функторы - это фундаментальная концепция функционального программирования, которая позволяет применять функции к значениям внутри контейнеров, сохраняя структуру контейнера.

## Содержание

- [Функторы](#функторы)
  - [Содержание](#содержание)
  - [Определение функтора](#определение-функтора)
  - [Законы функторов](#законы-функторов)
    - [1. Закон идентичности (Identity)](#1-закон-идентичности-identity)
    - [2. Закон композиции (Composition)](#2-закон-композиции-composition)
  - [Реализация функторов](#реализация-функторов)
    - [1. Maybe функтор](#1-maybe-функтор)
    - [2. Either функтор](#2-either-функтор)
    - [3. Array функтор](#3-array-функтор)
  - [Распространенные функторы](#распространенные-функторы)
    - [1. Promise как функтор](#1-promise-как-функтор)
    - [2. Function как функтор](#2-function-как-функтор)
  - [Практические примеры](#практические-примеры)
  - [Преимущества использования функторов](#преимущества-использования-функторов)
  - [Лучшие практики](#лучшие-практики)
    - [1. Соблюдение законов функторов](#1-соблюдение-законов-функторов)
    - [2. Явная обработка ошибок](#2-явная-обработка-ошибок)
    - [3. Иммутабельность](#3-иммутабельность)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Определение функтора

Функтор - это структура данных, которая реализует метод `map` и подчиняется двум законам (Identity и Composition). Метод `map` позволяет применять функции к значениям внутри контейнера, не извлекая их.

```typescript
// Базовый интерфейс функтора
interface Functor<T> {
  map: <U>(fn: (value: T) => U) => Functor<U>;
}

// Простой пример функтора - Box
class Box<T> implements Functor<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): Box<U> {
    return new Box(fn(this.value));
  }
  
  getValue(): T {
    return this.value;
  }
}

// Использование
const box = new Box(5);
const doubledBox = box.map(x => x * 2);
console.log(doubledBox.getValue()); // 10

// Композиция map
const result = box
  .map(x => x + 1)
  .map(x => x * 2)
  .map(x => x.toString());
  
console.log(result.getValue()); // "12"
```

## Законы функторов

Функторы должны подчиняться двум важным законам:

### 1. Закон идентичности (Identity)
Применение функции идентичности к функтору должно вернуть тот же функтор.

```typescript
// Закон идентичности: functor.map(x => x) === functor
const testIdentityLaw = () => {
  const box = new Box(5);
  const identity = <T>(x: T): T => x;
  
  const result1 = box.map(identity);
  const result2 = box;
  
  console.log(result1.getValue() === result2.getValue()); // true
};
```

### 2. Закон композиции (Composition)
Композиция функций внутри map должна быть эквивалентна композиции map'ов.

```typescript
// Закон композиции: functor.map(f).map(g) === functor.map(x => g(f(x)))
const testCompositionLaw = () => {
  const box = new Box(5);
  
  const f = (x: number): number => x + 1;
  const g = (x: number): number => x * 2;
  
  const result1 = box.map(f).map(g);
  const result2 = box.map(x => g(f(x)));
  
  console.log(result1.getValue() === result2.getValue()); // true
};
```

## Реализация функторов

Рассмотрим различные реализации функторов в TypeScript.

### 1. Maybe функтор

```typescript
// Maybe функтор для работы с потенциально отсутствующими значениями
type Maybe<T> = T | null | undefined;

const Maybe = {
  of<T>(value: T | null | undefined): MaybeFunctor<T> {
    return new MaybeFunctor(value);
  },
  
  map<T, U>(fn: (value: T) => U): (maybe: Maybe<T>) => Maybe<U> {
    return (maybe: Maybe<T>): Maybe<U> => {
      if (maybe === null || maybe === undefined) {
        return maybe as Maybe<U>;
      }
      return fn(maybe);
    };
  }
};

class MaybeFunctor<T> {
  constructor(private value: T | null | undefined) {}
  
  map<U>(fn: (value: T) => U): MaybeFunctor<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeFunctor<U>(null);
    }
    return new MaybeFunctor<U>(fn(this.value));
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

// Использование Maybe функтора
const safeDivide = (a: number, b: number): Maybe<number> => {
  if (b === 0) return null;
  return a / b;
};

const result = Maybe.of(10)
  .map(x => x * 2)
  .map(x => safeDivide(x, 4))
  .map(x => x ? x + 1 : null);

console.log(result.getValue()); // 6
```

### 2. Either функтор

```typescript
// Either функтор для обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherFunctor<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherFunctor<E, A> {
    return new EitherFunctor(value);
  }
  
  static right<E, A>(a: A): EitherFunctor<E, A> {
    return new EitherFunctor(right(a));
  }
  
  static left<E, A>(e: E): EitherFunctor<E, A> {
    return new EitherFunctor(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherFunctor<E, B> {
    if (this.value._tag === "Left") {
      return new EitherFunctor(left(this.value.left));
    }
    return new EitherFunctor(right(fn(this.value.right)));
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
}

// Использование Either функтора
const parseNumber = (str: string): Either<string, number> => {
  const num = Number(str);
  return isNaN(num) ? left(`Невозможно преобразовать "${str}" в число`) : right(num);
};

const sqrt = (num: number): Either<string, number> => {
  return num < 0 ? left(`Невозможно вычислить квадратный корень из ${num}`) : right(Math.sqrt(num));
};

const result2 = EitherFunctor.right("16")
  .map(parseNumber)
  .map(either => either._tag === "Right" ? sqrt(either.right) : either)
  .getValue();

console.log(result2); // { _tag: "Right", right: 4 }
```

### 3. Array функтор

```typescript
// Array как функтор
const numbers = [1, 2, 3, 4, 5];

// map как метод функтора
const doubled = numbers.map(x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Композиция операций
const result3 = numbers
  .map(x => x * 2)
  .map(x => x + 1)
  .map(x => x.toString());

console.log(result3); // ["3", "5", "7", "9", "11"]
```

## Распространенные функторы

### 1. Promise как функтор

```typescript
// Promise реализует интерфейс функтора через метод then
const fetchData = (): Promise<string> => {
  return new Promise(resolve => {
    setTimeout(() => resolve("Данные загружены"), 1000);
  });
};

// Использование Promise как функтора
const result4 = fetchData()
  .then(data => data.toUpperCase())
  .then(data => `Обработанные: ${data}`);

result4.then(console.log); // "Обработанные: ДАННЫЕ ЗАГРУЖЕНЫ"
```

### 2. Function как функтор

```typescript
// Функции можно рассматривать как функторы
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): ((a: A) => C) => 
  (a: A) => f(g(a));

// map для функций - это композиция
const add = (x: number): number => x + 1;
const multiply = (x: number): number => x * 2;

// map как композиция
const addAndMultiply = compose(multiply, add);
console.log(addAndMultiply(5)); // 12
```

## Практические примеры

Рассмотрим практические примеры использования функторов в реальных приложениях.

```typescript
// Пример 1: Обработка данных формы с использованием Maybe
interface FormField<T> {
  value: T;
  isValid: boolean;
  error?: string;
}

class FormFieldFunctor<T> {
  constructor(private field: FormField<T>) {}
  
  map<U>(fn: (value: T) => U): FormFieldFunctor<U> {
    if (!this.field.isValid) {
      return new FormFieldFunctor({
        value: undefined as any,
        isValid: false,
        error: this.field.error
      });
    }
    
    try {
      return new FormFieldFunctor({
        value: fn(this.field.value),
        isValid: true
      });
    } catch (error) {
      return new FormFieldFunctor({
        value: undefined as any,
        isValid: false,
        error: error instanceof Error ? error.message : "Ошибка обработки"
      });
    }
  }
  
  getValue(): FormField<T> {
    return this.field;
  }
}

// Использование
const emailField: FormField<string> = {
  value: "user@example.com",
  isValid: true
};

const processedField = new FormFieldFunctor(emailField)
  .map(email => email.toLowerCase())
  .map(email => email.trim())
  .map(email => ({ email, domain: email.split("@")[1] }));

console.log(processedField.getValue());

// Пример 2: Обработка API ответов с Either
interface ApiResponse<T> {
  status: number;
  data?: T;
  error?: string;
}

class ApiResponseFunctor<T> {
  private constructor(private response: ApiResponse<T>) {}
  
  static success<T>(data: T): ApiResponseFunctor<T> {
    return new ApiResponseFunctor({ status: 200, data });
  }
  
  static error<T>(error: string, status: number = 500): ApiResponseFunctor<T> {
    return new ApiResponseFunctor({ status, error });
  }
  
  map<U>(fn: (data: T) => U): ApiResponseFunctor<U> {
    if (this.response.error) {
      return new ApiResponseFunctor({
        status: this.response.status,
        error: this.response.error
      });
    }
    
    try {
      return new ApiResponseFunctor({
        status: this.response.status,
        data: fn(this.response.data!)
      });
    } catch (error) {
      return new ApiResponseFunctor({
        status: 500,
        error: error instanceof Error ? error.message : "Ошибка обработки"
      });
    }
  }
  
  getValue(): ApiResponse<T> {
    return this.response;
  }
}

// Использование
const apiResponse = ApiResponseFunctor.success({ id: 1, name: "Иван" })
  .map(user => ({ ...user, displayName: user.name.toUpperCase() }))
  .map(user => ({ ...user, isAdmin: user.id === 1 }));

console.log(apiResponse.getValue());
```

## Преимущества использования функторов

1. **Безопасность** - Функторы позволяют безопасно работать с потенциально опасными значениями
2. **Композиция** - Функторы поддерживают цепочки операций
3. **Предсказуемость** - Функторы подчиняются строгим законам
4. **Повторное использование** - Один и тот же интерфейс для разных типов данных
5. **Тестируемость** - Функторы легко тестировать изолированно

## Лучшие практики

### 1. Соблюдение законов функторов

```typescript
// Хорошо: соблюдение законов
class GoodFunctor<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): GoodFunctor<U> {
    return new GoodFunctor(fn(this.value));
  }
}

// Плохо: нарушение законов
class BadFunctor<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): BadFunctor<U> {
    // Нарушение закона идентичности
    return new BadFunctor(fn(this.value) + " modified");
  }
}
```

### 2. Явная обработка ошибок

```typescript
// Хорошо: использование Maybe для явной обработки null/undefined
const safeHead = <T>(arr: T[]): Maybe<T> => {
  return arr.length > 0 ? arr[0] : null;
};

const result5 = Maybe.of([1, 2, 3])
  .map(safeHead)
  .map(head => head ? head * 2 : null);

// Плохо: игнорирование потенциальных ошибок
const unsafeHead = <T>(arr: T[]): T => arr[0]; // Может бросить ошибку
```

### 3. Иммутабельность

```typescript
// Хорошо: иммутабельные функторы
class ImmutableFunctor<T> {
  constructor(private readonly value: T) {}
  
  map<U>(fn: (value: T) => U): ImmutableFunctor<U> {
    return new ImmutableFunctor(fn(this.value));
  }
  
  getValue(): T {
    return this.value;
  }
}

// Плохо: мутация внутреннего состояния
class MutableFunctor<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): MutableFunctor<U> {
    this.value = fn(this.value as any); // Мутация!
    return this as any;
  }
}
```

## Связи с другими концепциями

- [[ts/functional-programming/functors-monads/Функторы_и_монады|Функторы и монады]] - Основная страница раздела
- [[ts/functional-programming/functors-monads/monads|Монады]] - Расширение концепции функторов
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/error-handling/index|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование функторов

## Следующие шаги

- [[ts/functional-programming/functors-monads/monads|Монады]] - Подробное изучение монад
- [[ts/functional-programming/functors-monads/common-monads|Распространенные монады]] - Изучение популярных монад
- [[ts/functional-programming/functors-monads/practical-examples|Практические примеры]] - Реальные примеры использования
- [[ts/error-handling/index|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование функторов и монад