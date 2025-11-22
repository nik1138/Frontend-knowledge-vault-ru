# Продвинутые дженерики в TypeScript

Продвинутые дженерики - это сложные паттерны и техники использования обобщенных типов в TypeScript, которые позволяют создавать гибкие и переиспользуемые типы с мощными возможностями проверки типов.

## Содержание раздела

1. [[ts/advanced-generics/generic-constraints|Ограничения дженериков]] - Наложение ограничений на типы дженериков
2. [[ts/advanced-generics/conditional-generics|Условные дженерики]] - Дженерики, поведение которых зависит от условий
3. [[ts/advanced-generics/recursive-generics|Рекурсивные дженерики]] - Дженерики, ссылающиеся на самих себя
4. [[ts/advanced-generics/variance|Вариантность дженериков]] - Как дженерики ведут себя при наследовании
5. [[ts/advanced-generics/keyof-generics|Дженерики с ключами и значениями объектов]] - Работа с keyof и индексными типами

## Введение

Дженерики - это одна из самых мощных возможностей TypeScript, позволяющая создавать обобщенные типы и функции, которые работают с различными типами данных, сохраняя при этом строгую типизацию. Продвинутые дженерики расширяют эту возможность, позволяя создавать сложные типы и алгоритмы на уровне типов.

## Основные концепции

### 1. Метапрограммирование типов

Продвинутые дженерики позволяют выполнять "метапрограммирование" на уровне типов - создавать типы, которые генерируют другие типы:

```typescript
// Условный дженерик, который выбирает тип в зависимости от условия
type NonNullable<T> = T extends null | undefined ? never : T;

// Рекурсивный дженерик, который преобразует все свойства в опциональные
type DeepPartial<T> = T extends object 
  ? { [P in keyof T]?: DeepPartial<T[P]> } 
  : T;
```

### 2. Автоматическое выведение типов

Многие продвинутые дженерики позволяют TypeScript автоматически выводить сложные типы:

```typescript
// TypeScript может вывести тип возвращаемого значения функции
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

// Дженерик с выводом типа из массива
type ArrayElement<T> = T extends (infer U)[] ? U : T;
```

### 3. Типобезопасность на уровне компиляции

Продвинутые дженерики позволяют переносить больше логики проверок на этап компиляции:

```typescript
// Дженерик для создания типобезопасных API клиентов
interface ApiEndpoints {
  getUser: (id: string) => Promise<{ id: string; name: string }>;
  createUser: (data: { name: string }) => Promise<{ id: string; name: string }>;
}

type ApiClient<T extends Record<string, (...args: any[]) => any>> = {
  [K in keyof T]: T[K] extends (...args: infer Args) => Promise<infer Result>
    ? (...args: Args) => Promise<Result>
    : never;
};
```

## Практическое применение

### 1. Создание библиотек

Продвинутые дженерики особенно полезны при создании библиотек, где важна гибкость API:

```typescript
// Библиотека для работы с конфигурациями
type ConfigSchema<T> = {
  [K in keyof T]: {
    type: "string" | "number" | "boolean";
    required: boolean;
    defaultValue?: T[K];
  };
};

// Типобезопасная валидация конфигурации
function validateConfig<T>(
  config: Record<string, any>,
  schema: ConfigSchema<T>
): config is T {
  // Реализация валидации
  return true;
}
```

### 2. ORM и маппинг данных

Продвинутые дженерики полезны для создания типобезопасных ORM:

```typescript
// Схема модели
interface ModelSchema {
  [modelName: string]: {
    [fieldName: string]: "string" | "number" | "boolean" | "date";
  };
}

// Дженерик для создания экземпляра модели
type ModelInstance<T extends ModelSchema, M extends keyof T> = {
  [K in keyof T[M]]: T[M][K] extends "string" ? string :
                    T[M][K] extends "number" ? number :
                    T[M][K] extends "boolean" ? boolean :
                    T[M][K] extends "date" ? Date :
                    never;
};
```

### 3. Функциональное программирование

Продвинутые дженерики позволяют создавать мощные функциональные утилиты:

```typescript
// Дженерик для композиции функций
type Compose<F extends any[]> = 
  F extends [(...args: infer A) => infer B, (...args: infer C) => infer D]
    ? (...args: A) => D
    : never;

// Дженерик для каррирования функций
type Curry<F> = 
  F extends (...args: infer A) => infer R
    ? A extends [infer First, ...infer Rest]
      ? (arg: First) => Curry<(...args: Rest) => R>
      : () => R
    : never;
```

## Комбинирование концепций

Продвинутые дженерики часто комбинируют несколько концепций:

```typescript
// Дженерик, сочетающий условные типы, keyof и рекурсию
type DeepPick<T, K extends NestedKeyOf<T>> = 
  K extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
      ? { [P in Key]: DeepPick<T[Key], Rest> }
      : never
    : K extends keyof T
      ? { [P in K]: T[K] }
      : never;

type NestedKeyOf<T> = T extends object 
  ? { [K in keyof T]: K extends string 
      ? T[K] extends object 
        ? `${K}` | `${K}.${NestedKeyOf<T[K]>}` 
        : `${K}`
      : never 
    }[keyof T]
  : never;
```

## Ограничения и лучшие практики

### 1. Производительность

Сложные дженерики могут замедлять компиляцию:

```typescript
// Избегайте чрезмерно сложных вложенных дженериков
type ComplexType = DeepPartial<DeepReadonly<DeepRequired<SomeType>>>;

// Лучше разбить на несколько шагов
type Step1 = DeepPartial<SomeType>;
type Step2 = DeepReadonly<Step1>;
type Step3 = DeepRequired<Step2>;
```

### 2. Читаемость

Слишком сложные дженерики могут быть трудны для понимания:

```typescript
// Сложный для понимания дженерик
type Complex = ConditionalGeneric<
  RecursiveGeneric<KeyOfGeneric<SomeType, AnotherType>>,
  ThirdType
>;

// Более читаемый вариант
type Step1 = KeyOfGeneric<SomeType, AnotherType>;
type Step2 = RecursiveGeneric<Step1>;
type Step3 = ConditionalGeneric<Step2, ThirdType>;
```

## Связи с другими концепциями

Продвинутые дженерики тесно связаны с другими аспектами системы типов TypeScript:

- [[ts/generics/TS Generics|Основы дженериков]] - Базовые концепции дженериков
- [[ts/advanced-types/conditional-types|Условные типы]] - Многие дженерики используют условные типы
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - Основа для многих дженериков
- [[ts/type-system/type-inference|Вывод типов]] - Многие дженерики зависят от механизма вывода
- [[ts/utility-types/built-in-object-transformations|Утилиты для преобразования объектов]] - Многие встроенные утилиты используют продвинутые дженерики

## Рекомендации по изучению

1. Начните с простых ограничений дженериков
2. Освойте условные дженерики и использование `infer`
3. Практикуйтесь в создании рекурсивных дженериков
4. Изучите вариантность и ее влияние на типобезопасность
5. Освойте работу с `keyof` в дженериках
6. Практикуйтесь в комбинировании различных концепций
7. Изучите исходный код сложных библиотек для понимания паттернов

## Следующие шаги

- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Встроенные и пользовательские типы-помощники
- [[ts/advanced-types/conditional-types|Условные типы]] - Типы, зависящие от условий
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - Преобразование существующих типов
- [[ts/type-system/type-inference|Вывод типов]] - Как TypeScript выводит типы автоматически