# Условные типы (Conditional Types)

Условные типы в TypeScript позволяют создавать типы, которые зависят от условий, проверяющих отношения между типами. Это мощный механизм для создания гибких и адаптивных систем типов.

## Синтаксис

```typescript
T extends U ? X : Y
```

Если тип `T` может быть присвоен типу `U`, то тип принимает значение `X`, иначе `Y`.

## Простой пример

```typescript
type MessageOf<T> = T extends { message: unknown } ? T['message'] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessage = MessageOf<Email>; // string
type DogMessage = MessageOf<Dog>;     // never
```

## Распределительные условные типы

Когда условные типы применяются к объединению типов, они становятся распределительными:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]
```

Без распределения результат был бы `(string | number)[]`.

## Вывод типов в условных типах

Мы можем использовать ключевое слово `infer` для вывода типов внутри условных типов:

```typescript
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Str = GetReturnType<() => string>;         // string
type Num = GetReturnType<(x: number) => number>; // number
```

## Практические примеры

### 1. Flatten тип

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;

type Str = Flatten<string[]>;  // string
type Num = Flatten<number>;    // number
```

### 2. Проверка на null и undefined

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type Str = NonNullable<string | null>; // string
```

## Продвинутые примеры

### 1. Получение типа аргументов функции

```typescript
type GetFirstArgument<T> = T extends (first: infer First, ...args: any[]) => any 
  ? First 
  : never;

type FirstArg = GetFirstArgument<(a: string, b: number) => void>; // string
```

### 2. Условные типы с шаблонными литералами

```typescript
type RemoveKindField<T> = T extends { kind: string } 
  ? Omit<T, 'kind'> 
  : T;

interface Circle {
  kind: 'circle';
  radius: number;
}

type KindlessCircle = RemoveKindField<Circle>; 
// { radius: number }
```

## Ограничения и особенности

1. Условные типы вычисляются "лениво" - только когда это необходимо
2. Распределительные условные типы применяются только к простым типам (не объединениям)
3. `infer` может использоваться несколько раз в одном условном типе

## Когда использовать

Условные типы особенно полезны для:

- Создания гибких библиотек типов
- Преобразования существующих типов
- Создания типов-помощников
- Реализации сложных систем типов

## Связанные концепции

- [[ts/type-system/type-inference|Вывод типов]]
- [[ts/generics/TS Generics|Дженерики]]
- [[ts/advanced-types/mapped-types|Сопоставленные типы]]