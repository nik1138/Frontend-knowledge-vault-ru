# Ленивые вычисления

Ленивые вычисления (lazy evaluation) - это стратегия вычислений, при которой выражения вычисляются только тогда, когда их значение действительно необходимо. В функциональном программировании ленивые вычисления являются мощным инструментом для оптимизации производительности и работы с потенциально бесконечными структурами данных.

## Содержание

- [Ленивые вычисления](#ленивые-вычисления)
  - [Содержание](#содержание)
  - [Основы ленивых вычислений](#основы-ленивых-вычислений)
    - [Преимущества ленивых вычислений:](#преимущества-ленивых-вычислений)
    - [Недостатки ленивых вычислений:](#недостатки-ленивых-вычислений)
    - [Ленивые вычисления с мемоизацией](#ленивые-вычисления-с-мемоизацией)
  - [Ленивые последовательности](#ленивые-последовательности)
    - [Бесконечные последовательности](#бесконечные-последовательности)
  - [Ленивые структуры данных](#ленивые-структуры-данных)
    - [Ленивые списки (Streams)](#ленивые-списки-streams)
  - [Потоки данных](#потоки-данных)
  - [Ленивые итераторы](#ленивые-итераторы)
  - [Оптимизация ленивых вычислений](#оптимизация-ленивых-вычислений)
  - [Практические примеры](#практические-примеры)
    - [Обработка больших файлов](#обработка-больших-файлов)
    - [Ленивые вычисления в веб-приложениях](#ленивые-вычисления-в-веб-приложениях)

## Основы ленивых вычислений

Ленивые вычисления противопоставляются энергичным (eager) вычислениям, при которых выражения вычисляются немедленно. В ленивом подходе вычисления откладываются до момента, когда результат действительно нужен.

### Преимущества ленивых вычислений:

1. **Экономия ресурсов** - Вычисления выполняются только при необходимости
2. **Работа с бесконечными структурами** - Можно создавать потенциально бесконечные последовательности
3. **Композиция** - Ленивые операции можно эффективно комбинировать
4. **Оптимизация** - Возможность оптимизировать выполнение цепочек операций

### Недостатки ленивых вычислений:

1. **Накладные расходы** - Дополнительная сложность для отслеживания отложенных вычислений
2. **Память** - Накопление отложенных операций может привести к утечкам памяти
3. **Сложность отладки** - Труднее понять, когда и в каком порядке выполняются вычисления

```typescript
// Простая реализация ленивого значения
class LazyValue<T> {
  private value: T | undefined;
  private computed = false;
  private readonly computation: () => T;
  
  constructor(computation: () => T) {
    this.computation = computation;
  }
  
  get(): T {
    if (!this.computed) {
      this.value = this.computation();
      this.computed = true;
    }
    return this.value!;
  }
  
  isComputed(): boolean {
    return this.computed;
  }
}

// Пример использования
const expensiveComputation = new LazyValue(() => {
  console.log("Выполняется дорогостоящая операция...");
  // Симуляция сложных вычислений
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
});

console.log("Ленивое значение создано");
// Вычисления еще не выполнены

const value1 = expensiveComputation.get(); // Вычисления выполняются здесь
console.log("Значение 1:", value1);

const value2 = expensiveComputation.get(); // Значение берется из кэша
console.log("Значение 2:", value2);
console.log("Вычислено:", expensiveComputation.isComputed());
```

### Ленивые вычисления с мемоизацией

```typescript
// Ленивое значение с мемоизацией и отслеживанием ошибок
class MemoizedLazy<T> {
  private value: T | undefined;
  private error: any;
  private computed = false;
  private readonly computation: () => T;
  
  constructor(computation: () => T) {
    this.computation = computation;
  }
  
  get(): T {
    if (this.computed) {
      if (this.error) {
        throw this.error;
      }
      return this.value!;
    }
    
    try {
      this.value = this.computation();
      this.error = undefined;
    } catch (error) {
      this.error = error;
      throw error;
    } finally {
      this.computed = true;
    }
    
    return this.value!;
  }
  
  isComputed(): boolean {
    return this.computed;
  }
  
  reset(): void {
    this.value = undefined;
    this.error = undefined;
    this.computed = false;
  }
}

// Пример использования с ошибками
const lazyWithError = new MemoizedLazy(() => {
  console.log("Попытка вычисления...");
  if (Math.random() > 0.5) {
    throw new Error("Случайная ошибка");
  }
  return "Успех";
});

try {
  console.log(lazyWithError.get());
} catch (error) {
  console.log("Ошибка:", (error as Error).message);
}

// Повторный вызов будет использовать кэшированную ошибку
try {
  console.log(lazyWithError.get());
} catch (error) {
  console.log("Кэшированная ошибка:", (error as Error).message);
}
```

## Ленивые последовательности

Ленивые последовательности позволяют работать с потенциально бесконечными наборами данных без их полной материализации в памяти.

```typescript
// Базовый интерфейс для ленивых последовательностей
interface LazySequence<T> {
  // Преобразование элементов
  map<U>(fn: (item: T) => U): LazySequence<U>;
  
  // Фильтрация элементов
  filter(predicate: (item: T) => boolean): LazySequence<T>;
  
  // Ограничение количества элементов
  take(count: number): LazySequence<T>;
  
  // Пропуск элементов
  skip(count: number): LazySequence<T>;
  
  // Преобразование в массив
  toArray(): T[];
  
  // Выполнение операции для каждого элемента
  forEach(fn: (item: T) => void): void;
  
  // Получение первого элемента
  first(): T | undefined;
  
  // Проверка наличия элементов
  isEmpty(): boolean;
}

// Реализация ленивой последовательности
class LazySequenceImpl<T> implements LazySequence<T> {
  constructor(private readonly generator: () => Iterator<T>) {}
  
  map<U>(fn: (item: T) => U): LazySequence<U> {
    const generator = this.generator;
    return new LazySequenceImpl(function* () {
      for (const item of generator()) {
        yield fn(item);
      }
    });
  }
  
  filter(predicate: (item: T) => boolean): LazySequence<T> {
    const generator = this.generator;
    return new LazySequenceImpl(function* () {
      for (const item of generator()) {
        if (predicate(item)) {
          yield item;
        }
      }
    });
  }
  
  take(count: number): LazySequence<T> {
    const generator = this.generator;
    return new LazySequenceImpl(function* () {
      let taken = 0;
      for (const item of generator()) {
        if (taken >= count) break;
        yield item;
        taken++;
      }
    });
  }
  
  skip(count: number): LazySequence<T> {
    const generator = this.generator;
    return new LazySequenceImpl(function* () {
      let skipped = 0;
      for (const item of generator()) {
        if (skipped < count) {
          skipped++;
          continue;
        }
        yield item;
      }
    });
  }
  
  toArray(): T[] {
    const result: T[] = [];
    for (const item of this.generator()) {
      result.push(item);
    }
    return result;
  }
  
  forEach(fn: (item: T) => void): void {
    for (const item of this.generator()) {
      fn(item);
    }
  }
  
  first(): T | undefined {
    for (const item of this.generator()) {
      return item; // Возвращаем первый элемент
    }
    return undefined;
  }
  
  isEmpty(): boolean {
    for (const _ of this.generator()) {
      return false; // Последовательность не пуста
    }
    return true;
  }
}

// Создание ленивой последовательности из генератора
function fromGenerator<T>(generator: () => Iterator<T>): LazySequence<T> {
  return new LazySequenceImpl(generator);
}

// Создание ленивой последовательности чисел
function* numberGenerator(start = 0, step = 1) {
  let current = start;
  while (true) {
    yield current;
    current += step;
  }
}

const numbers = fromGenerator(() => numberGenerator(1, 1));

// Композиция операций (выполняется лениво)
const result = numbers
  .filter(x => x % 2 === 0)  // Только четные числа
  .map(x => x * x)           // Квадраты
  .take(5)                   // Первые 5 элементов
  .toArray();                // Материализация

console.log(result); // [4, 16, 36, 64, 100]
```

### Бесконечные последовательности

```typescript
// Бесконечная последовательность Фибоначчи
function* fibonacciGenerator() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fibonacci = fromGenerator(fibonacciGenerator);

// Получение первых 10 чисел Фибоначчи
console.log(fibonacci.take(10).toArray());

// Получение четных чисел Фибоначчи
const evenFibonacci = fibonacci
  .filter(x => x % 2 === 0)
  .take(5)
  .toArray();

console.log("Четные числа Фибоначчи:", evenFibonacci);

// Бесконечная последовательность случайных чисел
function* randomGenerator() {
  while (true) {
    yield Math.random();
  }
}

const randomNumbers = fromGenerator(randomGenerator);

// Получение первых 5 случайных чисел больше 0.5
const highRandoms = randomNumbers
  .filter(x => x > 0.5)
  .take(5)
  .toArray();

console.log("Высокие случайные числа:", highRandoms);
```

## Ленивые структуры данных

Ленивые структуры данных позволяют создавать сложные структуры, которые вычисляются по мере необходимости.

```typescript
// Ленивое дерево
class LazyTree<T> {
  private readonly value: LazyValue<T>;
  private readonly leftChild: LazyValue<LazyTree<T> | null>;
  private readonly rightChild: LazyValue<LazyTree<T> | null>;
  
  constructor(
    value: () => T,
    leftChild: () => LazyTree<T> | null = () => null,
    rightChild: () => LazyTree<T> | null = () => null
  ) {
    this.value = new LazyValue(value);
    this.leftChild = new LazyValue(leftChild);
    this.rightChild = new LazyValue(rightChild);
  }
  
  getValue(): T {
    return this.value.get();
  }
  
  getLeftChild(): LazyTree<T> | null {
    return this.leftChild.get();
  }
  
  getRightChild(): LazyTree<T> | null {
    return this.rightChild.get();
  }
  
  // Ленивый обход дерева в глубину
  *inOrderTraversal(): Generator<T, void, undefined> {
    const left = this.getLeftChild();
    if (left) {
      yield* left.inOrderTraversal();
    }
    
    yield this.getValue();
    
    const right = this.getRightChild();
    if (right) {
      yield* right.inOrderTraversal();
    }
  }
  
  // Ленивый обход дерева в ширину
  *breadthFirstTraversal(): Generator<T, void, undefined> {
    const queue: LazyTree<T>[] = [this];
    
    while (queue.length > 0) {
      const node = queue.shift()!;
      yield node.getValue();
      
      const left = node.getLeftChild();
      if (left) {
        queue.push(left);
      }
      
      const right = node.getRightChild();
      if (right) {
        queue.push(right);
      }
    }
  }
}

// Создание ленивого дерева с вычисляемыми значениями
const lazyTree = new LazyTree(
  () => {
    console.log("Вычисление корневого значения");
    return 1;
  },
  () => {
    console.log("Вычисление левого поддерева");
    return new LazyTree(() => 2);
  },
  () => {
    console.log("Вычисление правого поддерева");
    return new LazyTree(() => 3);
  }
);

console.log("Дерево создано");

// Обход дерева (вычисления выполняются по мере обхода)
console.log("Обход в глубину:");
for (const value of lazyTree.inOrderTraversal()) {
  console.log(value);
}

console.log("Обход в ширину:");
for (const value of lazyTree.breadthFirstTraversal()) {
  console.log(value);
}
```

### Ленивые списки (Streams)

```typescript
// Ленивый список (Stream)
class Stream<T> {
  private readonly head: LazyValue<T>;
  private readonly tail: LazyValue<Stream<T> | null>;
  
  constructor(head: () => T, tail: () => Stream<T> | null = () => null) {
    this.head = new LazyValue(head);
    this.tail = new LazyValue(tail);
  }
  
  static empty<T>(): Stream<T> {
    return new Stream(() => { throw new Error("Пустой поток"); }, () => null);
  }
  
  static of<T>(...values: T[]): Stream<T> {
    if (values.length === 0) {
      return Stream.empty();
    }
    
    const [head, ...tail] = values;
    return new Stream(() => head, () => Stream.of(...tail));
  }
  
  static iterate<T>(seed: T, fn: (value: T) => T): Stream<T> {
    return new Stream(() => seed, () => Stream.iterate(fn(seed), fn));
  }
  
  getHead(): T {
    return this.head.get();
  }
  
  getTail(): Stream<T> | null {
    return this.tail.get();
  }
  
  isEmpty(): boolean {
    return this === Stream.empty();
  }
  
  map<U>(fn: (item: T) => U): Stream<U> {
    if (this.isEmpty()) {
      return Stream.empty();
    }
    
    return new Stream(
      () => fn(this.getHead()),
      () => {
        const tail = this.getTail();
        return tail ? tail.map(fn) : null;
      }
    );
  }
  
  filter(predicate: (item: T) => boolean): Stream<T> {
    if (this.isEmpty()) {
      return Stream.empty();
    }
    
    const head = this.getHead();
    const tail = this.getTail();
    
    if (predicate(head)) {
      return new Stream(
        () => head,
        () => tail ? tail.filter(predicate) : null
      );
    } else {
      return tail ? tail.filter(predicate) : Stream.empty();
    }
  }
  
  take(count: number): Stream<T> {
    if (count <= 0 || this.isEmpty()) {
      return Stream.empty();
    }
    
    return new Stream(
      () => this.getHead(),
      () => {
        const tail = this.getTail();
        return tail ? tail.take(count - 1) : null;
      }
    );
  }
  
  toArray(): T[] {
    if (this.isEmpty()) {
      return [];
    }
    
    const result: T[] = [];
    let current: Stream<T> | null = this;
    
    while (current && !current.isEmpty()) {
      result.push(current.getHead());
      current = current.getTail();
    }
    
    return result;
  }
  
  forEach(fn: (item: T) => void): void {
    let current: Stream<T> | null = this;
    
    while (current && !current.isEmpty()) {
      fn(current.getHead());
      current = current.getTail();
    }
  }
}

// Примеры использования Stream
console.log("Создание потока из массива:");
const stream1 = Stream.of(1, 2, 3, 4, 5);
console.log(stream1.toArray());

console.log("Поток квадратов чисел:");
const squares = Stream.iterate(1, x => x + 1)
  .map(x => x * x)
  .take(5);
console.log(squares.toArray());

console.log("Поток четных чисел:");
const evenNumbers = Stream.iterate(2, x => x + 2)
  .take(10);
console.log(evenNumbers.toArray());

console.log("Фильтрация потока:");
const filtered = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  .filter(x => x % 2 === 0)
  .take(3);
console.log(filtered.toArray());
```

## Потоки данных

Потоки данных представляют собой последовательности значений, которые могут быть обработаны лениво.

```typescript
// Поток данных с поддержкой событий
class DataStream<T> {
  private observers: ((data: T) => void)[] = [];
  private readonly generator: () => AsyncIterator<T>;
  
  constructor(generator: () => AsyncIterator<T>) {
    this.generator = generator;
  }
  
  // Подписка на поток данных
  subscribe(observer: (data: T) => void): () => void {
    this.observers.push(observer);
    
    // Возвращаем функцию для отписки
    return () => {
      const index = this.observers.indexOf(observer);
      if (index >= 0) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  // Запуск потока данных
  async start(): Promise<void> {
    const iterator = this.generator();
    
    try {
      while (true) {
        const result = await iterator.next();
        if (result.done) break;
        
        // Уведомляем всех наблюдателей
        for (const observer of this.observers) {
          observer(result.value);
        }
      }
    } catch (error) {
      console.error("Ошибка в потоке данных:", error);
    }
  }
  
  // Преобразование потока данных
  map<U>(fn: (data: T) => U): DataStream<U> {
    const self = this;
    return new DataStream(async function* () {
      const iterator = self.generator();
      while (true) {
        const result = await iterator.next();
        if (result.done) break;
        yield fn(result.value);
      }
    });
  }
  
  // Фильтрация потока данных
  filter(predicate: (data: T) => boolean): DataStream<T> {
    const self = this;
    return new DataStream(async function* () {
      const iterator = self.generator();
      while (true) {
        const result = await iterator.next();
        if (result.done) break;
        if (predicate(result.value)) {
          yield result.value;
        }
      }
    });
  }
  
  // Ограничение количества элементов
  take(count: number): DataStream<T> {
    const self = this;
    return new DataStream(async function* () {
      const iterator = self.generator();
      let taken = 0;
      
      while (taken < count) {
        const result = await iterator.next();
        if (result.done) break;
        
        yield result.value;
        taken++;
      }
    });
  }
}

// Создание потока данных из асинхронного генератора
async function* sensorDataGenerator() {
  let id = 1;
  while (true) {
    yield {
      id: id++,
      timestamp: Date.now(),
      value: Math.random() * 100,
      sensor: `sensor-${Math.floor(Math.random() * 3) + 1}`
    };
    
    // Задержка между измерениями
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}

const sensorStream = new DataStream(sensorDataGenerator);

// Подписка на поток данных
const unsubscribe = sensorStream
  .filter(data => data.value > 50) // Только высокие значения
  .map(data => ({ ...data, normalized: data.value / 100 })) // Нормализация
  .take(5) // Только первые 5 элементов
  .subscribe(data => {
    console.log("Получены данные:", data);
  });

// Запуск потока данных
sensorStream.start().then(() => {
  console.log("Поток данных завершен");
});

// Отписка через 10 секунд
setTimeout(() => {
  unsubscribe();
  console.log("Отписка от потока данных");
}, 10000);
```

## Ленивые итераторы

Ленивые итераторы предоставляют гибкий способ работы с последовательностями данных.

```typescript
// Ленивый итератор с расширенными возможностями
class LazyIterator<T> implements Iterator<T> {
  private readonly source: Iterator<T>;
  private readonly transformers: ((iterator: Iterator<T>) => Iterator<T>)[] = [];
  
  constructor(source: Iterator<T>) {
    this.source = source;
  }
  
  // Добавление преобразования
  transform<U>(transformer: (iterator: Iterator<T>) => Iterator<U>): LazyIterator<U> {
    const newIterator = new LazyIterator(this.source) as any;
    newIterator.transformers = [...this.transformers, transformer];
    return newIterator;
  }
  
  // Преобразование элементов
  map<U>(fn: (item: T) => U): LazyIterator<U> {
    return this.transform(function* (iterator: Iterator<T>) {
      for (const item of iterator) {
        yield fn(item);
      }
    });
  }
  
  // Фильтрация элементов
  filter(predicate: (item: T) => boolean): LazyIterator<T> {
    return this.transform(function* (iterator: Iterator<T>) {
      for (const item of iterator) {
        if (predicate(item)) {
          yield item;
        }
      }
    });
  }
  
  // Ограничение количества элементов
  take(count: number): LazyIterator<T> {
    return this.transform(function* (iterator: Iterator<T>) {
      let taken = 0;
      for (const item of iterator) {
        if (taken >= count) break;
        yield item;
        taken++;
      }
    });
  }
  
  // Пропуск элементов
  skip(count: number): LazyIterator<T> {
    return this.transform(function* (iterator: Iterator<T>) {
      let skipped = 0;
      for (const item of iterator) {
        if (skipped < count) {
          skipped++;
          continue;
        }
        yield item;
      }
    });
  }
  
  // Получение следующего элемента
  next(): IteratorResult<T> {
    // Применяем все преобразования
    let iterator: Iterator<any> = this.source;
    for (const transformer of this.transformers) {
      iterator = transformer(iterator);
    }
    
    return iterator.next();
  }
  
  // Реализация Symbol.iterator для использования в for...of
  [Symbol.iterator](): Iterator<T> {
    return this;
  }
}

// Создание ленивого итератора из массива
function fromArray<T>(array: T[]): LazyIterator<T> {
  return new LazyIterator(array[Symbol.iterator]());
}

// Создание ленивого итератора из генератора
function fromGeneratorFunction<T>(generator: () => Generator<T>): LazyIterator<T> {
  return new LazyIterator(generator());
}

// Примеры использования
console.log("Ленивый итератор из массива:");
const arrayIterator = fromArray([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  .filter(x => x % 2 === 0)
  .map(x => x * x)
  .take(3);

for (const value of arrayIterator) {
  console.log(value);
}

// Ленивый итератор из генератора
function* fibonacciSequence() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

console.log("Числа Фибоначчи:");
const fibIterator = fromGeneratorFunction(fibonacciSequence)
  .filter(x => x % 2 === 0)
  .take(5);

for (const value of fibIterator) {
  console.log(value);
}
```

## Оптимизация ленивых вычислений

Оптимизация ленивых вычислений важна для предотвращения накопления отложенных операций и утечек памяти.

```typescript
// Оптимизированный ленивый итератор с кэшированием
class OptimizedLazyIterator<T> implements Iterator<T> {
  private readonly source: Iterator<T>;
  private cache: T[] = [];
  private cachePosition = 0;
  private sourceExhausted = false;
  
  constructor(source: Iterator<T>) {
    this.source = source;
  }
  
  // Получение следующего элемента с кэшированием
  next(): IteratorResult<T> {
    // Проверяем кэш
    if (this.cachePosition < this.cache.length) {
      const value = this.cache[this.cachePosition];
      this.cachePosition++;
      return { value, done: false };
    }
    
    // Если источник исчерпан, возвращаем done
    if (this.sourceExhausted) {
      return { value: undefined, done: true } as any;
    }
    
    // Получаем следующий элемент из источника
    const result = this.source.next();
    
    if (result.done) {
      this.sourceExhausted = true;
      return result;
    }
    
    // Кэшируем элемент
    this.cache.push(result.value);
    this.cachePosition++;
    
    return result;
  }
  
  // Сброс позиции итератора
  reset(): void {
    this.cachePosition = 0;
  }
  
  // Получение всех элементов в массив
  toArray(): T[] {
    const result: T[] = [];
    
    while (true) {
      const nextResult = this.next();
      if (nextResult.done) break;
      result.push(nextResult.value);
    }
    
    return result;
  }
  
  [Symbol.iterator](): Iterator<T> {
    return this;
  }
}

// Ленивый итератор с ограничением размера кэша (LRU)
class LRULazyIterator<T> implements Iterator<T> {
  private readonly source: Iterator<T>;
  private readonly maxSize: number;
  private cache: T[] = [];
  private cacheStartIndex = 0;
  private currentIndex = 0;
  private sourceIndex = 0;
  private sourceExhausted = false;
  
  constructor(source: Iterator<T>, maxSize: number = 1000) {
    this.source = source;
    this.maxSize = maxSize;
  }
  
  next(): IteratorResult<T> {
    // Проверяем, есть ли элемент в кэше
    const cacheIndex = this.currentIndex - this.cacheStartIndex;
    
    if (cacheIndex >= 0 && cacheIndex < this.cache.length) {
      const value = this.cache[cacheIndex];
      this.currentIndex++;
      return { value, done: false };
    }
    
    // Если источник исчерпан
    if (this.sourceExhausted) {
      return { value: undefined, done: true } as any;
    }
    
    // Получаем следующий элемент из источника
    const result = this.source.next();
    
    if (result.done) {
      this.sourceExhausted = true;
      return result;
    }
    
    // Добавляем элемент в кэш
    this.cache.push(result.value);
    this.sourceIndex++;
    this.currentIndex++;
    
    // Если кэш переполнен, удаляем старые элементы
    if (this.cache.length > this.maxSize) {
      const removeCount = Math.floor(this.maxSize / 2);
      this.cache.splice(0, removeCount);
      this.cacheStartIndex += removeCount;
    }
    
    return result;
  }
  
  [Symbol.iterator](): Iterator<T> {
    return this;
  }
}

// Пример использования оптимизированных итераторов
function* largeNumberSequence() {
  for (let i = 0; i < 1000000; i++) {
    yield i;
  }
}

console.log("Оптимизированный итератор:");
const optimizedIterator = new OptimizedLazyIterator(largeNumberSequence());
const first10 = [];
for (let i = 0; i < 10; i++) {
  const result = optimizedIterator.next();
  if (!result.done) {
    first10.push(result.value);
  }
}
console.log("Первые 10 элементов:", first10);

// Сброс и повторное использование
optimizedIterator.reset();
const next10 = [];
for (let i = 0; i < 10; i++) {
  const result = optimizedIterator.next();
  if (!result.done) {
    next10.push(result.value);
  }
}
console.log("Следующие 10 элементов (повтор):", next10);

console.log("LRU итератор:");
const lruIterator = new LRULazyIterator(largeNumberSequence(), 100);
const lruFirst150 = [];
for (let i = 0; i < 150; i++) {
  const result = lruIterator.next();
  if (!result.done) {
    lruFirst150.push(result.value);
  }
}
console.log("Первые 150 элементов:", lruFirst150.slice(0, 5), "...", lruFirst150.slice(-5));
```

## Практические примеры

### Обработка больших файлов

```typescript
// Ленивый парсер CSV файлов
class CSVParser {
  static async* parseLines(stream: ReadableStream): AsyncGenerator<string[], void, undefined> {
    const reader = stream.getReader();
    const decoder = new TextDecoder();
    let buffer = '';
    
    try {
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        buffer += decoder.decode(value, { stream: true });
        
        // Разбиваем на строки
        const lines = buffer.split('\n');
        buffer = lines.pop() || ''; // Последняя неполная строка сохраняется
        
        for (const line of lines) {
          if (line.trim()) {
            yield line.split(',').map(field => field.trim());
          }
        }
      }
      
      // Обрабатываем последнюю строку
      if (buffer.trim()) {
        yield buffer.split(',').map(field => field.trim());
      }
    } finally {
      reader.releaseLock();
    }
  }
  
  static async* parseWithTransform(
    stream: ReadableStream,
    transform: (row: string[]) => any
  ): AsyncGenerator<any, void, undefined> {
    for await (const row of this.parseLines(stream)) {
      yield transform(row);
    }
  }
}

// Пример использования для обработки больших CSV файлов
async function processLargeCSV() {
  // Симуляция потока данных
  const mockStream = new ReadableStream({
    start(controller) {
      let count = 0;
      const interval = setInterval(() => {
        if (count >= 1000) {
          controller.close();
          clearInterval(interval);
          return;
        }
        
        const row = `${count},${Math.random()},${Date.now()}\n`;
        controller.enqueue(new TextEncoder().encode(row));
        count++;
      }, 1);
    }
  });
  
  // Ленивая обработка данных
  const startTime = Date.now();
  let processedCount = 0;
  
  const parser = CSVParser.parseWithTransform(mockStream, (row) => ({
    id: parseInt(row[0]),
    value: parseFloat(row[1]),
    timestamp: parseInt(row[2])
  }));
  
  // Обрабатываем только первые 100 записей
  for await (const record of parser) {
    processedCount++;
    if (processedCount > 100) break;
    
    // Обработка записи
    if (record.value > 0.5) {
      console.log(`Запись ${record.id}: высокое значение ${record.value}`);
    }
  }
  
  const endTime = Date.now();
  console.log(`Обработано ${processedCount} записей за ${endTime - startTime}ms`);
}

// processLargeCSV();
```

### Ленивые вычисления в веб-приложениях

```typescript
// Ленивые компоненты для веб-приложений
class LazyComponent<T> {
  private readonly factory: () => Promise<T>;
  private instance: T | null = null;
  private loadingPromise: Promise<T> | null = null;
  
  constructor(factory: () => Promise<T>) {
    this.factory = factory;
  }
  
  async getInstance(): Promise<T> {
    if (this.instance) {
      return this.instance;
    }
    
    if (this.loadingPromise) {
      return this.loadingPromise;
    }
    
    this.loadingPromise = this.factory().then(instance => {
      this.instance = instance;
      this.loadingPromise = null;
      return instance;
    });
    
    return this.loadingPromise;
  }
  
  isLoaded(): boolean {
    return this.instance !== null;
  }
}

// Пример использования для динамической загрузки модулей
const heavyChartComponent = new LazyComponent(async () => {
  console.log("Загрузка тяжелого компонента графиков...");
  // Симуляция загрузки большого модуля
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  return {
    render: (data: any[]) => {
      console.log("Отрисовка графика с", data.length, "точками");
      return `<div>График с ${data.length} точками</div>`;
    }
  };
});

// Использование ленивого компонента
async function renderDashboard() {
  console.log("Начало отрисовки дашборда");
  
  // Другие быстрые компоненты
  console.log("Отрисовка заголовка");
  console.log("Отрисовка меню");
  
  // Ленивая загрузка тяжелого компонента
  const chart = await heavyChartComponent.getInstance();
  const chartHtml = chart.render([1, 2, 3, 4, 5]);
  console.log("График загружен и отрисован:", chartHtml);
  
  console.log("Дашборд полностью отрисован");
}

// renderDashboard();
```

Ленивые вычисления являются мощным инструментом в функциональном программировании, позволяющим создавать эффективные и масштабируемые приложения. Правильное использование ленивых структур данных и последовательностей помогает оптимизировать использование памяти и улучшить производительность, особенно при работе с большими объемами данных.