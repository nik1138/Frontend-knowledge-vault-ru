# Неизменяемые массивы

Неизменяемые массивы - это массивы, которые не могут быть изменены после создания. Вместо модификации существующих массивов создаются новые версии с обновленными значениями.

## Создание неизменяемых массивов

### Использование readonly модификатора

```typescript
// Тип для неизменяемого массива
type ImmutableList<T> = readonly T[];

// Создание неизменяемого массива
const numbers: ImmutableList<number> = [1, 2, 3, 4, 5] as const;
const strings: ImmutableList<string> = ['a', 'b', 'c'] as const;

// Попытка изменения вызовет ошибку компиляции
// numbers.push(6); // Ошибка: Property 'push' does not exist on type 'readonly number[]'
// numbers[0] = 10; // Ошибка: Index signature in type 'readonly number[]' only permits reading
```

### Использование Object.freeze

```typescript
// Создание полностью неизменяемого массива
const createImmutableArray = <T>(items: T[]): ImmutableList<T> => 
  Object.freeze([...items]);

const immutableNumbers = createImmutableArray([1, 2, 3, 4, 5]);

// Попытка изменения будет проигнорирована в строгом режиме
// или вызовет ошибку в нестрогом режиме
// immutableNumbers.push(6); // Ошибка в строгом режиме

// Создание нового массива вместо изменения
const addNumber = <T>(array: ImmutableList<T>, item: T): ImmutableList<T> => 
  Object.freeze([...array, item]);
```

## Работа с неизменяемыми массивами

### Операции добавления элементов

```typescript
// Добавление в начало массива
const prepend = <T>(array: ImmutableList<T>, item: T): ImmutableList<T> => 
  Object.freeze([item, ...array]);

// Добавление в конец массива
const append = <T>(array: ImmutableList<T>, item: T): ImmutableList<T> => 
  Object.freeze([...array, item]);

// Добавление нескольких элементов
const prependMany = <T>(array: ImmutableList<T>, items: T[]): ImmutableList<T> => 
  Object.freeze([...items, ...array]);

const appendMany = <T>(array: ImmutableList<T>, items: T[]): ImmutableList<T> => 
  Object.freeze([...array, ...items]);

// Пример использования
const originalArray: ImmutableList<number> = Object.freeze([2, 3, 4]);

const withFirst = prepend(originalArray, 1); // [1, 2, 3, 4]
const withLast = append(originalArray, 5); // [2, 3, 4, 5]
const withMany = appendMany(originalArray, [5, 6, 7]); // [2, 3, 4, 5, 6, 7]
```

### Операции удаления элементов

```typescript
// Удаление первого элемента
const tail = <T>(array: ImmutableList<T>): ImmutableList<T> => 
  Object.freeze(array.slice(1));

// Удаление последнего элемента
const init = <T>(array: ImmutableList<T>): ImmutableList<T> => 
  Object.freeze(array.slice(0, -1));

// Удаление элемента по индексу
const removeAt = <T>(array: ImmutableList<T>, index: number): ImmutableList<T> => {
  if (index < 0 || index >= array.length) {
    return array;
  }
  
  return Object.freeze([
    ...array.slice(0, index),
    ...array.slice(index + 1)
  ]);
};

// Удаление элементов по предикату
const remove = <T>(
  array: ImmutableList<T>, 
  predicate: (item: T) => boolean
): ImmutableList<T> => 
  Object.freeze(array.filter(item => !predicate(item)));

// Пример использования
const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

const withoutFirst = tail(numbers); // [2, 3, 4, 5]
const withoutLast = init(numbers); // [1, 2, 3, 4]
const withoutThird = removeAt(numbers, 2); // [1, 2, 4, 5]
const evensOnly = remove(numbers, n => n % 2 !== 0); // [2, 4]
```

### Операции обновления элементов

```typescript
// Обновление элемента по индексу
const updateAt = <T>(
  array: ImmutableList<T>, 
  index: number, 
  newValue: T
): ImmutableList<T> => {
  if (index < 0 || index >= array.length) {
    return array;
  }
  
  return Object.freeze([
    ...array.slice(0, index),
    newValue,
    ...array.slice(index + 1)
  ]);
};

// Обновление элементов по предикату
const update = <T>(
  array: ImmutableList<T>, 
  predicate: (item: T) => boolean,
  updater: (item: T) => T
): ImmutableList<T> => 
  Object.freeze(array.map(item => 
    predicate(item) ? updater(item) : item
  ));

// Пример использования
const users: ImmutableList<{ id: number; name: string; active: boolean }> = 
  Object.freeze([
    { id: 1, name: 'John', active: true },
    { id: 2, name: 'Jane', active: false },
    { id: 3, name: 'Bob', active: true }
  ]);

const updatedUser = updateAt(users, 1, { id: 2, name: 'Jane Smith', active: false });
const activatedUsers = update(users, u => !u.active, u => ({ ...u, active: true }));
```

## Функции высшего порядка для массивов

### Map и Filter

```typescript
// Неизменяемый map
const map = <T, U>(
  array: ImmutableList<T>, 
  fn: (item: T) => U
): ImmutableList<U> => 
  Object.freeze(array.map(fn));

// Неизменяемый filter
const filter = <T>(
  array: ImmutableList<T>, 
  predicate: (item: T) => boolean
): ImmutableList<T> => 
  Object.freeze(array.filter(predicate));

// Пример использования
const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

const doubled = map(numbers, n => n * 2); // [2, 4, 6, 8, 10]
const evens = filter(numbers, n => n % 2 === 0); // [2, 4]
```

### Reduce

```typescript
// Неизменяемый reduce
const reduce = <T, U>(
  array: ImmutableList<T>, 
  reducer: (accumulator: U, currentValue: T) => U,
  initialValue: U
): U => 
  array.reduce(reducer, initialValue);

// Reduce для создания новых структур
const groupBy = <T, K extends string | number>(
  array: ImmutableList<T>, 
  keyFn: (item: T) => K
): Record<K, ImmutableList<T>> => 
  array.reduce((groups, item) => {
    const key = keyFn(item);
    return {
      ...groups,
      [key]: [...(groups[key] || []), item]
    };
  }, {} as Record<K, ImmutableList<T>>);

// Пример использования
const users: ImmutableList<{ id: number; name: string; department: string }> = 
  Object.freeze([
    { id: 1, name: 'John', department: 'Engineering' },
    { id: 2, name: 'Jane', department: 'Marketing' },
    { id: 3, name: 'Bob', department: 'Engineering' }
  ]);

const departmentGroups = groupBy(users, u => u.department);
// {
//   Engineering: [{ id: 1, name: 'John', department: 'Engineering' }, { id: 3, name: 'Bob', department: 'Engineering' }],
//   Marketing: [{ id: 2, name: 'Jane', department: 'Marketing' }]
// }
```

### Sort

```typescript
// Неизменяемый sort
const sort = <T>(
  array: ImmutableList<T>, 
  compareFn?: (a: T, b: T) => number
): ImmutableList<T> => 
  Object.freeze([...array].sort(compareFn));

// Пример использования
const numbers: ImmutableList<number> = Object.freeze([3, 1, 4, 1, 5, 9, 2, 6]);

const sortedNumbers = sort(numbers); // [1, 1, 2, 3, 4, 5, 6, 9]
const reverseSorted = sort(numbers, (a, b) => b - a); // [9, 6, 5, 4, 3, 2, 1, 1]
```

## Паттерны работы с неизменяемыми массивами

### Pipeline паттерн

```typescript
// Pipeline для цепочки операций над массивами
const pipe = <T>(value: T, ...fns: Array<(arg: T) => T>): T =>
  fns.reduce((acc, fn) => fn(acc), value);

// Утилиты для работы с массивами в стиле pipeline
const ArrayUtils = {
  map: <T, U>(fn: (item: T) => U) => 
    (array: ImmutableList<T>): ImmutableList<U> => 
      Object.freeze(array.map(fn)),
  
  filter: <T>(predicate: (item: T) => boolean) => 
    (array: ImmutableList<T>): ImmutableList<T> => 
      Object.freeze(array.filter(predicate)),
  
  sort: <T>(compareFn?: (a: T, b: T) => number) => 
    (array: ImmutableList<T>): ImmutableList<T> => 
      Object.freeze([...array].sort(compareFn)),
  
  take: <T>(count: number) => 
    (array: ImmutableList<T>): ImmutableList<T> => 
      Object.freeze(array.slice(0, count)),
  
  skip: <T>(count: number) => 
    (array: ImmutableList<T>): ImmutableList<T> => 
      Object.freeze(array.slice(count))
};

// Пример использования pipeline
const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const result = pipe(
  numbers,
  ArrayUtils.filter(n => n % 2 === 0), // [2, 4, 6, 8, 10]
  ArrayUtils.map(n => n * 2),          // [4, 8, 12, 16, 20]
  ArrayUtils.sort((a, b) => b - a),    // [20, 16, 12, 8, 4]
  ArrayUtils.take(3)                   // [20, 16, 12]
);

console.log(result); // [20, 16, 12]
```

### Batch операции

```typescript
// Batch операции для множественных изменений
const ArrayBatch = {
  // Множественное обновление
  updateMany: <T>(
    array: ImmutableList<T>,
    updates: { index: number; value: T }[]
  ): ImmutableList<T> => {
    const updateMap = new Map(updates.map(u => [u.index, u.value]));
    
    return Object.freeze(array.map((item, index) => 
      updateMap.has(index) ? updateMap.get(index)! : item
    ));
  },
  
  // Множественное удаление
  removeMany: <T>(
    array: ImmutableList<T>,
    indices: number[]
  ): ImmutableList<T> => {
    const indicesSet = new Set(indices);
    
    return Object.freeze(array.filter((_, index) => !indicesSet.has(index)));
  },
  
  // Множественное добавление
  insertMany: <T>(
    array: ImmutableList<T>,
    inserts: { index: number; items: T[] }[]
  ): ImmutableList<T> => {
    // Сортируем вставки по индексу в обратном порядке
    // чтобы индексы не сдвигались при вставке
    const sortedInserts = [...inserts].sort((a, b) => b.index - a.index);
    
    let result = [...array];
    
    for (const { index, items } of sortedInserts) {
      result.splice(index, 0, ...items);
    }
    
    return Object.freeze(result);
  }
};

// Пример использования
const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

const updated = ArrayBatch.updateMany(numbers, [
  { index: 1, value: 20 },
  { index: 3, value: 40 }
]); // [1, 20, 3, 40, 5]

const filtered = ArrayBatch.removeMany(numbers, [1, 3]); // [1, 3, 5]

const withInserts = ArrayBatch.insertMany(numbers, [
  { index: 2, items: [25, 26] },
  { index: 0, items: [0] }
]); // [0, 1, 2, 25, 26, 3, 4, 5]
```

## Иммутабельные структуры данных

### Иммутабельный список

```typescript
// Интерфейс иммутабельного списка
interface ImmutableList<T> {
  readonly head: T | undefined;
  readonly tail: ImmutableList<T> | undefined;
  isEmpty(): boolean;
  prepend(value: T): ImmutableList<T>;
  append(value: T): ImmutableList<T>;
  map<U>(fn: (value: T) => U): ImmutableList<U>;
  filter(predicate: (value: T) => boolean): ImmutableList<T>;
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U;
}

// Пустой список
class EmptyList<T> implements ImmutableList<T> {
  readonly head: undefined = undefined;
  readonly tail: undefined = undefined;
  
  isEmpty(): boolean {
    return true;
  }
  
  prepend(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  append(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  map<U>(fn: (value: T) => U): ImmutableList<U> {
    return new EmptyList<U>();
  }
  
  filter(predicate: (value: T) => boolean): ImmutableList<T> {
    return new EmptyList<T>();
  }
  
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U {
    return initial;
  }
}

// Узел списка
class ListNode<T> implements ImmutableList<T> {
  constructor(
    readonly head: T,
    readonly tail: ImmutableList<T>
  ) {}
  
  isEmpty(): boolean {
    return false;
  }
  
  prepend(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  append(value: T): ImmutableList<T> {
    return new ListNode(value, new EmptyList<T>()).prependList(this);
  }
  
  private prependList(other: ImmutableList<T>): ImmutableList<T> {
    if (other.isEmpty()) {
      return this;
    } else {
      return this.prependList(other.tail!).prepend(other.head!);
    }
  }
  
  map<U>(fn: (value: T) => U): ImmutableList<U> {
    return new ListNode(
      fn(this.head),
      this.tail.map(fn)
    );
  }
  
  filter(predicate: (value: T) => boolean): ImmutableList<T> {
    const filteredTail = this.tail.filter(predicate);
    
    if (predicate(this.head)) {
      return new ListNode(this.head, filteredTail);
    } else {
      return filteredTail;
    }
  }
  
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U {
    return this.tail.reduce(fn, fn(initial, this.head));
  }
}

// Использование
const list = new EmptyList<number>()
  .prepend(3)
  .prepend(2)
  .prepend(1); // [1, 2, 3]

const doubled = list.map(x => x * 2); // [2, 4, 6]
const evens = list.filter(x => x % 2 === 0); // [2]
const sum = list.reduce((acc, x) => acc + x, 0); // 6
```

## Преимущества неизменяемых массивов

### 1. Предсказуемость

```typescript
// Предсказуемое поведение
const processArray = (array: ImmutableList<number>): number => {
  // Функция не может изменить входной массив
  return array.reduce((sum, n) => sum + n, 0);
};

const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

const result = processArray(numbers);
// numbers остается неизменным
console.log(numbers); // [1, 2, 3, 4, 5]
```

### 2. Потокобезопасность

```typescript
// Безопасное использование в многопоточных средах
const sharedNumbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

// Функции могут безопасно читать данные без блокировок
const calculateAverage = (numbers: ImmutableList<number>): number => {
  if (numbers.length === 0) return 0;
  return numbers.reduce((sum, n) => sum + n, 0) / numbers.length;
};

// Обновление создает новую версию, не влияя на существующие ссылки
const newNumbers = append(sharedNumbers, 6);
```

### 3. Упрощенное тестирование

```typescript
// Легко тестируемые функции
const findMax = (numbers: ImmutableList<number>): number => 
  numbers.length > 0 ? Math.max(...numbers) : 0;

// Тестирование
const testNumbers: ImmutableList<number> = Object.freeze([1, 5, 3, 9, 2]);

const max = findMax(testNumbers);
console.assert(max === 9);
// testNumbers остается неизменным
```

## Распространенные ошибки

### 1. Попытка изменения неизменяемого массива

```typescript
// Ошибка: попытка изменения неизменяемого массива
const processArrayBad = (array: ImmutableList<number>): void => {
  // array.push(6); // Ошибка компиляции
  // Но в runtime это может быть проигнорировано
};

// Правильно: создание нового массива
const processArrayGood = (array: ImmutableList<number>): ImmutableList<number> => 
  Object.freeze([...array, 6]);
```

### 2. Мутация вложенных объектов

```typescript
// Ошибка: изменение вложенных объектов
interface User {
  readonly id: number;
  readonly name: string;
  readonly tags: readonly string[];
}

const addUserTagBad = (user: User, tag: string): User => {
  // user.tags.push(tag); // Ошибка: push изменяет массив
  return user;
};

// Правильно: создание нового массива
const addUserTagGood = (user: User, tag: string): User => ({
  ...user,
  tags: Object.freeze([...user.tags, tag])
});
```

### 3. Неправильное использование slice

```typescript
// Ошибка: slice возвращает изменяемый массив
const getSliceBad = (array: ImmutableList<number>, start: number, end: number) => 
  array.slice(start, end); // Возвращает number[], а не ImmutableList<number>

// Правильно: заморозка результата
const getSliceGood = (array: ImmutableList<number>, start: number, end: number) => 
  Object.freeze(array.slice(start, end));
```

## Лучшие практики

### 1. Использование readonly модификаторов

```typescript
// Хорошо: явное указание неизменяемости
type ImmutableList<T> = readonly T[];

// Плохо: отсутствие readonly модификаторов
type MutableArray<T> = T[];
```

### 2. Использование Object.freeze для дополнительной защиты

```typescript
// Дополнительная защита с Object.freeze
const createImmutableArray = <T>(items: T[]): ImmutableList<T> => 
  Object.freeze([...items]);
```

### 3. Создание утилит для работы с неизменяемыми массивами

```typescript
// Утилиты для работы с неизменяемыми массивами
const ArrayImmutable = {
  // Безопасный slice
  slice: <T>(
    array: ImmutableList<T>, 
    start?: number, 
    end?: number
  ): ImmutableList<T> => 
    Object.freeze(array.slice(start, end)),
  
  // Безопасный concat
  concat: <T>(
    array: ImmutableList<T>, 
    ...items: (T | ImmutableList<T>)[]
  ): ImmutableList<T> => 
    Object.freeze(array.concat(...items)),
  
  // Безопасный reverse
  reverse: <T>(array: ImmutableList<T>): ImmutableList<T> => 
    Object.freeze([...array].reverse())
};

// Пример использования
const numbers: ImmutableList<number> = Object.freeze([1, 2, 3, 4, 5]);

const reversed = ArrayImmutable.reverse(numbers); // [5, 4, 3, 2, 1]
const sliced = ArrayImmutable.slice(numbers, 1, 4); // [2, 3, 4]
const concatenated = ArrayImmutable.concat(numbers, 6, 7); // [1, 2, 3, 4, 5, 6, 7]
```

## Связи с другими концепциями

- [[ts/functional-programming/immutability/Неизменяемость|Неизменяемость]] - Основная страница раздела
- [[ts/functional-programming/immutability/immutable-objects|Неизменяемые объекты]] - Работа с неизменяемыми объектами
- [[ts/functional-programming/immutability/immutable-classes|Неизменяемые классы]] - Создание неизменяемых классов
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции

## Следующие шаги

- [[ts/functional-programming/immutability/immutable-classes|Неизменяемые классы]] - Создание неизменяемых классов
- [[ts/functional-programming/immutability/patterns|Паттерны неизменяемости]] - Распространенные паттерны работы с неизменяемыми структурами
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции