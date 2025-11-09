# Оптимизация структур данных

В функциональном программировании структуры данных часто неизменяемы, но существуют специализированные реализации, оптимизированные для производительности. Эти структуры данных сохраняют преимущества неизменяемости, одновременно обеспечивая эффективность операций.

## Содержание

1. [Persistent Data Structures](#persistent-data-structures)
2. [Ленивые структуры данных](#ленивые-структуры-данных)
3. [Оптимизированные массивы](#оптимизированные-массивы)
4. [Функциональные деревья](#функциональные-деревья)
5. [Хэш-таблицы и словари](#хэш-таблицы-и-словари)
6. [Графы и сети](#графы-и-сети)
7. [Практические примеры](#практические-примеры)

## Persistent Data Structures

Persistent data structures сохраняют предыдущие версии при изменении, что идеально подходит для функционального программирования. Они обеспечивают эффективность за счет структурного совместного использования.

### Persistent Array

```typescript
// Persistent массив с оптимизацией для обновлений
class PersistentArray<T> {
  private readonly data: T[];
  
  constructor(data: T[] = []) {
    this.data = [...data]; // Копируем данные для иммутабельности
  }
  
  get(index: number): T | undefined {
    return this.data[index];
  }
  
  set(index: number, value: T): PersistentArray<T> {
    const newData = [...this.data];
    newData[index] = value;
    return new PersistentArray(newData);
  }
  
  push(value: T): PersistentArray<T> {
    return new PersistentArray([...this.data, value]);
  }
  
  pop(): [T | undefined, PersistentArray<T>] {
    if (this.data.length === 0) {
      return [undefined, this];
    }
    
    const newData = this.data.slice(0, -1);
    const lastElement = this.data[this.data.length - 1];
    return [lastElement, new PersistentArray(newData)];
  }
  
  length(): number {
    return this.data.length;
  }
  
  toArray(): T[] {
    return [...this.data];
  }
  
  // Оптимизированная операция map
  map<U>(fn: (item: T, index: number) => U): PersistentArray<U> {
    const result = new Array<U>(this.data.length);
    for (let i = 0; i < this.data.length; i++) {
      result[i] = fn(this.data[i], i);
    }
    return new PersistentArray(result);
  }
  
  // Оптимизированная операция filter
  filter(predicate: (item: T, index: number) => boolean): PersistentArray<T> {
    const result: T[] = [];
    for (let i = 0; i < this.data.length; i++) {
      if (predicate(this.data[i], i)) {
        result.push(this.data[i]);
      }
    }
    return new PersistentArray(result);
  }
  
  // Оптимизированная операция reduce
  reduce<U>(fn: (accumulator: U, currentValue: T, index: number) => U, initialValue: U): U {
    let accumulator = initialValue;
    for (let i = 0; i < this.data.length; i++) {
      accumulator = fn(accumulator, this.data[i], i);
    }
    return accumulator;
  }
  
  // Поиск элемента
  find(predicate: (item: T, index: number) => boolean): T | undefined {
    for (let i = 0; i < this.data.length; i++) {
      if (predicate(this.data[i], i)) {
        return this.data[i];
      }
    }
    return undefined;
  }
  
  // Проверка наличия элемента
  includes(item: T): boolean {
    return this.data.includes(item);
  }
}

// Пример использования
const arr1 = new PersistentArray([1, 2, 3, 4, 5]);
const arr2 = arr1.set(2, 10); // Изменяем элемент по индексу 2
const arr3 = arr2.push(6);     // Добавляем элемент

console.log(arr1.toArray()); // [1, 2, 3, 4, 5] - оригинальный массив не изменился
console.log(arr2.toArray()); // [1, 2, 10, 4, 5] - новая версия
console.log(arr3.toArray()); // [1, 2, 10, 4, 5, 6] - еще одна версия

// Функциональные операции
const doubled = arr3.map(x => x * 2);
console.log(doubled.toArray()); // [2, 4, 20, 8, 10, 12]

const evens = arr3.filter(x => x % 2 === 0);
console.log(evens.toArray()); // [2, 10, 4, 6]

const sum = arr3.reduce((acc, val) => acc + val, 0);
console.log(sum); // 28
```

### Persistent Vector (Rope-like structure)

```typescript
// Persistent Vector с оптимизацией для больших массивов
class PersistentVector<T> {
  private readonly chunks: T[][];
  private readonly chunkSize: number;
  private readonly length: number;
  
  constructor(
    chunks: T[][] = [[]],
    chunkSize: number = 32,
    length: number = 0
  ) {
    this.chunks = chunks;
    this.chunkSize = chunkSize;
    this.length = length;
  }
  
  static fromArray<T>(array: T[], chunkSize: number = 32): PersistentVector<T> {
    if (array.length === 0) {
      return new PersistentVector();
    }
    
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += chunkSize) {
      chunks.push(array.slice(i, i + chunkSize));
    }
    
    return new PersistentVector(chunks, chunkSize, array.length);
  }
  
  get(index: number): T | undefined {
    if (index < 0 || index >= this.length) {
      return undefined;
    }
    
    const chunkIndex = Math.floor(index / this.chunkSize);
    const elementIndex = index % this.chunkSize;
    
    return this.chunks[chunkIndex]?.[elementIndex];
  }
  
  set(index: number, value: T): PersistentVector<T> {
    if (index < 0 || index >= this.length) {
      throw new Error("Индекс вне диапазона");
    }
    
    const chunkIndex = Math.floor(index / this.chunkSize);
    const elementIndex = index % this.chunkSize;
    
    const newChunks = [...this.chunks];
    newChunks[chunkIndex] = [...this.chunks[chunkIndex]];
    newChunks[chunkIndex][elementIndex] = value;
    
    return new PersistentVector(newChunks, this.chunkSize, this.length);
  }
  
  push(value: T): PersistentVector<T> {
    const newChunks = [...this.chunks];
    const lastChunk = newChunks[newChunks.length - 1];
    
    if (lastChunk.length < this.chunkSize) {
      // Добавляем в последний чанк
      newChunks[newChunks.length - 1] = [...lastChunk, value];
    } else {
      // Создаем новый чанк
      newChunks.push([value]);
    }
    
    return new PersistentVector(newChunks, this.chunkSize, this.length + 1);
  }
  
  pop(): [T | undefined, PersistentVector<T>] {
    if (this.length === 0) {
      return [undefined, this];
    }
    
    const newChunks = [...this.chunks];
    const lastChunk = newChunks[newChunks.length - 1];
    
    if (lastChunk.length === 1) {
      // Удаляем последний чанк
      newChunks.pop();
    } else {
      // Удаляем последний элемент из последнего чанка
      newChunks[newChunks.length - 1] = lastChunk.slice(0, -1);
    }
    
    const lastElement = lastChunk[lastChunk.length - 1];
    return [lastElement, new PersistentVector(newChunks, this.chunkSize, this.length - 1)];
  }
  
  length(): number {
    return this.length;
  }
  
  toArray(): T[] {
    return this.chunks.flat();
  }
  
  // Оптимизированная операция map
  map<U>(fn: (item: T, index: number) => U): PersistentVector<U> {
    const newChunks: U[][] = [];
    
    let globalIndex = 0;
    for (const chunk of this.chunks) {
      const newChunk: U[] = [];
      for (let i = 0; i < chunk.length; i++) {
        newChunk.push(fn(chunk[i], globalIndex + i));
      }
      newChunks.push(newChunk);
      globalIndex += chunk.length;
    }
    
    return new PersistentVector(newChunks, this.chunkSize, this.length);
  }
  
  // Оптимизированная операция filter
  filter(predicate: (item: T, index: number) => boolean): PersistentVector<T> {
    const result: T[] = [];
    
    let globalIndex = 0;
    for (const chunk of this.chunks) {
      for (let i = 0; i < chunk.length; i++) {
        if (predicate(chunk[i], globalIndex + i)) {
          result.push(chunk[i]);
        }
      }
      globalIndex += chunk.length;
    }
    
    return PersistentVector.fromArray(result, this.chunkSize);
  }
}

// Пример использования Persistent Vector
const vector1 = PersistentVector.fromArray([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 3);
const vector2 = vector1.set(5, 100); // Изменяем элемент по индексу 5
const vector3 = vector2.push(11);     // Добавляем элемент

console.log(vector1.toArray()); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log(vector2.toArray()); // [1, 2, 3, 4, 5, 100, 7, 8, 9, 10]
console.log(vector3.toArray()); // [1, 2, 3, 4, 5, 100, 7, 8, 9, 10, 11]

// Производительность с большими массивами
const largeArray = Array.from({ length: 10000 }, (_, i) => i);
const largeVector = PersistentVector.fromArray(largeArray, 64);

console.log("Длина большого вектора:", largeVector.length());
console.log("Элемент по индексу 5000:", largeVector.get(5000));
```

## Ленивые структуры данных

Ленивые структуры данных вычисляют значения только при необходимости, что позволяет эффективно работать с потенциально бесконечными структурами.

### Lazy List (Stream)

```typescript
// Ленивый список (Stream)
class Stream<T> {
  private readonly head: () => T;
  private readonly tail: () => Stream<T> | null;
  private headValue: T | undefined;
  private tailValue: Stream<T> | null | undefined;
  private headComputed = false;
  private tailComputed = false;
  
  constructor(head: () => T, tail: () => Stream<T> | null = () => null) {
    this.head = head;
    this.tail = tail;
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
  
  static generate<T>(generator: () => T): Stream<T> {
    return new Stream(generator, () => Stream.generate(generator));
  }
  
  getHead(): T {
    if (!this.headComputed) {
      this.headValue = this.head();
      this.headComputed = true;
    }
    return this.headValue!;
  }
  
  getTail(): Stream<T> | null {
    if (!this.tailComputed) {
      this.tailValue = this.tail();
      this.tailComputed = true;
    }
    return this.tailValue;
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
  
  skip(count: number): Stream<T> {
    if (count <= 0 || this.isEmpty()) {
      return this;
    }
    
    const tail = this.getTail();
    if (!tail) {
      return Stream.empty();
    }
    
    return tail.skip(count - 1);
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
  
  reduce<U>(fn: (accumulator: U, currentValue: T) => U, initialValue: U): U {
    let accumulator = initialValue;
    let current: Stream<T> | null = this;
    
    while (current && !current.isEmpty()) {
      accumulator = fn(accumulator, current.getHead());
      current = current.getTail();
    }
    
    return accumulator;
  }
  
  find(predicate: (item: T) => boolean): T | undefined {
    let current: Stream<T> | null = this;
    
    while (current && !current.isEmpty()) {
      const item = current.getHead();
      if (predicate(item)) {
        return item;
      }
      current = current.getTail();
    }
    
    return undefined;
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

// Бесконечный поток простых чисел
function isPrime(n: number): boolean {
  if (n < 2) return false;
  if (n === 2) return true;
  if (n % 2 === 0) return false;
  
  for (let i = 3; i <= Math.sqrt(n); i += 2) {
    if (n % i === 0) return false;
  }
  
  return true;
}

const primes = Stream.iterate(2, x => x + 1)
  .filter(isPrime)
  .take(10);

console.log("Первые 10 простых чисел:", primes.toArray());
```

## Оптимизированные массивы

Оптимизированные массивы используют различные техники для повышения производительности при работе с большими объемами данных.

### Immutable Array с Copy-on-Write

```typescript
// Immutable Array с Copy-on-Write оптимизацией
class ImmutableArray<T> {
  private readonly data: T[];
  private readonly isMutable: boolean;
  
  constructor(data: T[] = [], isMutable: boolean = false) {
    this.data = isMutable ? data : [...data];
    this.isMutable = isMutable;
  }
  
  static fromArray<T>(array: T[]): ImmutableArray<T> {
    return new ImmutableArray(array, true);
  }
  
  get(index: number): T | undefined {
    return this.data[index];
  }
  
  set(index: number, value: T): ImmutableArray<T> {
    if (this.isMutable) {
      // Copy-on-Write: создаем копию при первом изменении
      const newData = [...this.data];
      newData[index] = value;
      return new ImmutableArray(newData, false);
    } else {
      // Массив уже неизменяем
      const newData = [...this.data];
      newData[index] = value;
      return new ImmutableArray(newData, false);
    }
  }
  
  push(value: T): ImmutableArray<T> {
    if (this.isMutable) {
      // Copy-on-Write
      const newData = [...this.data, value];
      return new ImmutableArray(newData, false);
    } else {
      // Массив уже неизменяем
      const newData = [...this.data, value];
      return new ImmutableArray(newData, false);
    }
  }
  
  pop(): [T | undefined, ImmutableArray<T>] {
    if (this.data.length === 0) {
      return [undefined, this];
    }
    
    if (this.isMutable) {
      // Copy-on-Write
      const newData = this.data.slice(0, -1);
      const lastElement = this.data[this.data.length - 1];
      return [lastElement, new ImmutableArray(newData, false)];
    } else {
      // Массив уже неизменяем
      const newData = this.data.slice(0, -1);
      const lastElement = this.data[this.data.length - 1];
      return [lastElement, new ImmutableArray(newData, false)];
    }
  }
  
  length(): number {
    return this.data.length;
  }
  
  toArray(): T[] {
    return [...this.data];
  }
  
  // Быстрая операция slice без копирования данных
  slice(start?: number, end?: number): ImmutableArray<T> {
    return new ImmutableArray(this.data.slice(start, end), false);
  }
  
  // Оптимизированная операция map
  map<U>(fn: (item: T, index: number) => U): ImmutableArray<U> {
    const result = new Array<U>(this.data.length);
    for (let i = 0; i < this.data.length; i++) {
      result[i] = fn(this.data[i], i);
    }
    return new ImmutableArray(result, false);
  }
  
  // Оптимизированная операция filter
  filter(predicate: (item: T, index: number) => boolean): ImmutableArray<T> {
    const result: T[] = [];
    for (let i = 0; i < this.data.length; i++) {
      if (predicate(this.data[i], i)) {
        result.push(this.data[i]);
      }
    }
    return new ImmutableArray(result, false);
  }
  
  // Оптимизированная операция concat
  concat(...arrays: (T[] | ImmutableArray<T>)[]): ImmutableArray<T> {
    const result = [...this.data];
    
    for (const array of arrays) {
      if (array instanceof ImmutableArray) {
        result.push(...array.data);
      } else {
        result.push(...array);
      }
    }
    
    return new ImmutableArray(result, false);
  }
}

// Пример использования
const arr1 = ImmutableArray.fromArray([1, 2, 3, 4, 5]);
const arr2 = arr1.set(2, 10); // Изменяем элемент по индексу 2
const arr3 = arr2.push(6);     // Добавляем элемент

console.log(arr1.toArray()); // [1, 2, 3, 4, 5] - оригинальный массив не изменился
console.log(arr2.toArray()); // [1, 2, 10, 4, 5] - новая версия
console.log(arr3.toArray()); // [1, 2, 10, 4, 5, 6] - еще одна версия

// Операции с массивами
const doubled = arr3.map(x => x * 2);
console.log(doubled.toArray()); // [2, 4, 20, 8, 10, 12]

const evens = arr3.filter(x => x % 2 === 0);
console.log(evens.toArray()); // [2, 10, 4, 6]

const combined = arr3.concat([7, 8], ImmutableArray.fromArray([9, 10]));
console.log(combined.toArray()); // [1, 2, 10, 4, 5, 6, 7, 8, 9, 10]
```

## Функциональные деревья

Функциональные деревья обеспечивают эффективные операции вставки, удаления и поиска с сохранением неизменяемости.

### Persistent Binary Search Tree

```typescript
// Persistent Binary Search Tree
class PersistentBST<T> {
  private readonly value: T | undefined;
  private readonly left: PersistentBST<T> | undefined;
  private readonly right: PersistentBST<T> | undefined;
  private readonly size: number;
  
  constructor(
    value?: T,
    left?: PersistentBST<T>,
    right?: PersistentBST<T>
  ) {
    this.value = value;
    this.left = left;
    this.right = right;
    this.size = (left?.size || 0) + (right?.size || 0) + (value !== undefined ? 1 : 0);
  }
  
  static empty<T>(): PersistentBST<T> {
    return new PersistentBST();
  }
  
  isEmpty(): boolean {
    return this.value === undefined;
  }
  
  insert(value: T, compare: (a: T, b: T) => number = (a, b) => (a as any) - (b as any)): PersistentBST<T> {
    if (this.isEmpty()) {
      return new PersistentBST(value);
    }
    
    const cmp = compare(value, this.value!);
    
    if (cmp < 0) {
      return new PersistentBST(this.value, this.left!.insert(value, compare), this.right);
    } else if (cmp > 0) {
      return new PersistentBST(this.value, this.left, this.right!.insert(value, compare));
    } else {
      // Значение уже существует
      return this;
    }
  }
  
  contains(value: T, compare: (a: T, b: T) => number = (a, b) => (a as any) - (b as any)): boolean {
    if (this.isEmpty()) {
      return false;
    }
    
    const cmp = compare(value, this.value!);
    
    if (cmp < 0) {
      return this.left!.contains(value, compare);
    } else if (cmp > 0) {
      return this.right!.contains(value, compare);
    } else {
      return true;
    }
  }
  
  remove(value: T, compare: (a: T, b: T) => number = (a, b) => (a as any) - (b as any)): PersistentBST<T> {
    if (this.isEmpty()) {
      return this;
    }
    
    const cmp = compare(value, this.value!);
    
    if (cmp < 0) {
      return new PersistentBST(this.value, this.left!.remove(value, compare), this.right);
    } else if (cmp > 0) {
      return new PersistentBST(this.value, this.left, this.right!.remove(value, compare));
    } else {
      // Найден узел для удаления
      if (this.left!.isEmpty()) {
        return this.right!;
      } else if (this.right!.isEmpty()) {
        return this.left!;
      } else {
        // Узел с двумя детьми
        const successor = this.right!.findMin();
        return new PersistentBST(
          successor,
          this.left,
          this.right!.remove(successor, compare)
        );
      }
    }
  }
  
  private findMin(): T {
    if (this.left!.isEmpty()) {
      return this.value!;
    }
    return this.left!.findMin();
  }
  
  size(): number {
    return this.size;
  }
  
  // In-order traversal
  *inOrder(): Generator<T, void, undefined> {
    if (!this.isEmpty()) {
      if (this.left) {
        yield* this.left.inOrder();
      }
      yield this.value!;
      if (this.right) {
        yield* this.right.inOrder();
      }
    }
  }
  
  // Pre-order traversal
  *preOrder(): Generator<T, void, undefined> {
    if (!this.isEmpty()) {
      yield this.value!;
      if (this.left) {
        yield* this.left.preOrder();
      }
      if (this.right) {
        yield* this.right.preOrder();
      }
    }
  }
  
  // Post-order traversal
  *postOrder(): Generator<T, void, undefined> {
    if (!this.isEmpty()) {
      if (this.left) {
        yield* this.left.postOrder();
      }
      if (this.right) {
        yield* this.right.postOrder();
      }
      yield this.value!;
    }
  }
  
  toArray(): T[] {
    return Array.from(this.inOrder());
  }
  
  // Поиск k-го элемента
  get(k: number): T | undefined {
    if (k < 0 || k >= this.size) {
      return undefined;
    }
    
    if (this.left && k < this.left.size) {
      return this.left.get(k);
    } else if (this.left && k === this.left.size) {
      return this.value;
    } else if (this.left) {
      return this.right!.get(k - this.left.size - 1);
    } else if (k === 0) {
      return this.value;
    } else {
      return this.right!.get(k - 1);
    }
  }
}

// Пример использования Persistent BST
const compareNumbers = (a: number, b: number) => a - b;

let tree = PersistentBST.empty<number>();

// Вставка элементов
tree = tree.insert(5, compareNumbers);
tree = tree.insert(3, compareNumbers);
tree = tree.insert(7, compareNumbers);
tree = tree.insert(2, compareNumbers);
tree = tree.insert(4, compareNumbers);
tree = tree.insert(6, compareNumbers);
tree = tree.insert(8, compareNumbers);

console.log("Дерево в порядке обхода:", tree.toArray());
console.log("Размер дерева:", tree.size());

// Поиск элементов
console.log("Содержит 4:", tree.contains(4, compareNumbers));
console.log("Содержит 9:", tree.contains(9, compareNumbers));

// Получение элементов по индексу
console.log("Элемент по индексу 0:", tree.get(0));
console.log("Элемент по индексу 3:", tree.get(3));

// Удаление элементов
tree = tree.remove(3, compareNumbers);
console.log("После удаления 3:", tree.toArray());
```

## Хэш-таблицы и словари

Хэш-таблицы обеспечивают быстрый доступ к данным по ключу, а функциональные реализации сохраняют неизменяемость.

### Persistent Hash Map

```typescript
// Persistent Hash Map с использованием хэш-таблицы
class PersistentHashMap<K, V> {
  private readonly buckets: Array<Array<[K, V]>>;
  private readonly size: number;
  private readonly bucketCount: number;
  
  constructor(
    buckets: Array<Array<[K, V]>> = [],
    size: number = 0,
    bucketCount: number = 16
  ) {
    this.buckets = buckets;
    this.size = size;
    this.bucketCount = bucketCount;
  }
  
  static empty<K, V>(bucketCount: number = 16): PersistentHashMap<K, V> {
    return new PersistentHashMap([], 0, bucketCount);
  }
  
  private hash(key: K): number {
    // Простая хэш-функция (в реальной реализации лучше использовать более сложную)
    return Math.abs(JSON.stringify(key).split('').reduce((a, b) => ((a << 5) - a + b.charCodeAt(0)) | 0, 0)) % this.bucketCount;
  }
  
  set(key: K, value: V): PersistentHashMap<K, V> {
    const bucketIndex = this.hash(key);
    const bucket = this.buckets[bucketIndex] || [];
    
    // Проверяем, существует ли уже такой ключ
    const existingIndex = bucket.findIndex(([k]) => this.keysEqual(k, key));
    
    const newBuckets = [...this.buckets];
    
    if (existingIndex >= 0) {
      // Обновляем существующее значение
      const newBucket = [...bucket];
      newBucket[existingIndex] = [key, value];
      newBuckets[bucketIndex] = newBucket;
      return new PersistentHashMap(newBuckets, this.size, this.bucketCount);
    } else {
      // Добавляем новую пару ключ-значение
      const newBucket = [...bucket, [key, value]];
      newBuckets[bucketIndex] = newBucket;
      return new PersistentHashMap(newBuckets, this.size + 1, this.bucketCount);
    }
  }
  
  get(key: K): V | undefined {
    const bucketIndex = this.hash(key);
    const bucket = this.buckets[bucketIndex];
    
    if (!bucket) {
      return undefined;
    }
    
    const entry = bucket.find(([k]) => this.keysEqual(k, key));
    return entry ? entry[1] : undefined;
  }
  
  delete(key: K): PersistentHashMap<K, V> {
    const bucketIndex = this.hash(key);
    const bucket = this.buckets[bucketIndex];
    
    if (!bucket) {
      return this;
    }
    
    const existingIndex = bucket.findIndex(([k]) => this.keysEqual(k, key));
    
    if (existingIndex < 0) {
      return this;
    }
    
    const newBuckets = [...this.buckets];
    const newBucket = [...bucket];
    newBucket.splice(existingIndex, 1);
    newBuckets[bucketIndex] = newBucket;
    
    return new PersistentHashMap(newBuckets, this.size - 1, this.bucketCount);
  }
  
  has(key: K): boolean {
    return this.get(key) !== undefined;
  }
  
  size(): number {
    return this.size;
  }
  
  keys(): K[] {
    const result: K[] = [];
    for (const bucket of this.buckets) {
      if (bucket) {
        for (const [key] of bucket) {
          result.push(key);
        }
      }
    }
    return result;
  }
  
  values(): V[] {
    const result: V[] = [];
    for (const bucket of this.buckets) {
      if (bucket) {
        for (const [, value] of bucket) {
          result.push(value);
        }
      }
    }
    return result;
  }
  
  entries(): [K, V][] {
    const result: [K, V][] = [];
    for (const bucket of this.buckets) {
      if (bucket) {
        result.push(...bucket);
      }
    }
    return result;
  }
  
  forEach(callback: (value: V, key: K) => void): void {
    for (const bucket of this.buckets) {
      if (bucket) {
        for (const [key, value] of bucket) {
          callback(value, key);
        }
      }
    }
  }
  
  map<U>(fn: (value: V, key: K) => U): PersistentHashMap<K, U> {
    const newBuckets: Array<Array<[K, U]>> = [];
    let newSize = 0;
    
    for (let i = 0; i < this.buckets.length; i++) {
      const bucket = this.buckets[i];
      if (bucket) {
        const newBucket: Array<[K, U]> = [];
        for (const [key, value] of bucket) {
          newBucket.push([key, fn(value, key)]);
        }
        newBuckets[i] = newBucket;
        newSize += newBucket.length;
      }
    }
    
    return new PersistentHashMap(newBuckets, newSize, this.bucketCount);
  }
  
  filter(predicate: (value: V, key: K) => boolean): PersistentHashMap<K, V> {
    const newBuckets: Array<Array<[K, V]>> = [];
    let newSize = 0;
    
    for (const bucket of this.buckets) {
      if (bucket) {
        const newBucket: Array<[K, V]> = [];
        for (const [key, value] of bucket) {
          if (predicate(value, key)) {
            newBucket.push([key, value]);
          }
        }
        if (newBucket.length > 0) {
          newBuckets.push(newBucket);
          newSize += newBucket.length;
        }
      }
    }
    
    return new PersistentHashMap(newBuckets, newSize, this.bucketCount);
  }
  
  private keysEqual(key1: K, key2: K): boolean {
    // Для простых случаев используем строгое равенство
    if (key1 === key2) {
      return true;
    }
    
    // Для объектов используем JSON.stringify
    try {
      return JSON.stringify(key1) === JSON.stringify(key2);
    } catch {
      return false;
    }
  }
}

// Пример использования Persistent Hash Map
let map = PersistentHashMap.empty<string, number>();

// Добавление элементов
map = map.set("apple", 5);
map = map.set("banana", 3);
map = map.set("orange", 8);
map = map.set("apple", 10); // Обновление значения

console.log("Значение для 'apple':", map.get("apple"));
console.log("Значение для 'banana':", map.get("banana"));
console.log("Значение для 'grape':", map.get("grape")); // undefined

console.log("Размер:", map.size());
console.log("Ключи:", map.keys());
console.log("Значения:", map.values());

// Удаление элементов
map = map.delete("banana");
console.log("После удаления 'banana':", map.keys());

// Фильтрация
const evenValues = map.filter(value => value % 2 === 0);
console.log("Элементы с четными значениями:", evenValues.entries());

// Преобразование значений
const doubled = map.map(value => value * 2);
console.log("Удвоенные значения:", doubled.entries());
```

## Графы и сети

Функциональные графы обеспечивают эффективную работу с сетевыми структурами данных.

### Persistent Graph

```typescript
// Persistent Graph
class PersistentGraph<T> {
  private readonly adjacencyList: PersistentHashMap<T, Set<T>>;
  
  constructor(adjacencyList: PersistentHashMap<T, Set<T>> = PersistentHashMap.empty()) {
    this.adjacencyList = adjacencyList;
  }
  
  static empty<T>(): PersistentGraph<T> {
    return new PersistentGraph();
  }
  
  addVertex(vertex: T): PersistentGraph<T> {
    if (this.hasVertex(vertex)) {
      return this;
    }
    
    const newAdjacencyList = this.adjacencyList.set(vertex, new Set());
    return new PersistentGraph(newAdjacencyList);
  }
  
  addEdge(from: T, to: T): PersistentGraph<T> {
    let newGraph = this.addVertex(from).addVertex(to);
    
    const neighbors = newGraph.adjacencyList.get(from) || new Set();
    const newNeighbors = new Set(neighbors);
    newNeighbors.add(to);
    
    const newAdjacencyList = newGraph.adjacencyList.set(from, newNeighbors);
    return new PersistentGraph(newAdjacencyList);
  }
  
  removeVertex(vertex: T): PersistentGraph<T> {
    if (!this.hasVertex(vertex)) {
      return this;
    }
    
    // Удаляем вершину из списка смежности
    let newAdjacencyList = this.adjacencyList.delete(vertex);
    
    // Удаляем все ребра, ведущие к этой вершине
    for (const [v, neighbors] of newAdjacencyList.entries()) {
      if (neighbors.has(vertex)) {
        const newNeighbors = new Set(neighbors);
        newNeighbors.delete(vertex);
        newAdjacencyList = newAdjacencyList.set(v, newNeighbors);
      }
    }
    
    return new PersistentGraph(newAdjacencyList);
  }
  
  removeEdge(from: T, to: T): PersistentGraph<T> {
    if (!this.hasEdge(from, to)) {
      return this;
    }
    
    const neighbors = this.adjacencyList.get(from) || new Set();
    const newNeighbors = new Set(neighbors);
    newNeighbors.delete(to);
    
    const newAdjacencyList = this.adjacencyList.set(from, newNeighbors);
    return new PersistentGraph(newAdjacencyList);
  }
  
  hasVertex(vertex: T): boolean {
    return this.adjacencyList.has(vertex);
  }
  
  hasEdge(from: T, to: T): boolean {
    const neighbors = this.adjacencyList.get(from);
    return neighbors ? neighbors.has(to) : false;
  }
  
  getNeighbors(vertex: T): Set<T> {
    return this.adjacencyList.get(vertex) || new Set();
  }
  
  getVertices(): T[] {
    return this.adjacencyList.keys();
  }
  
  getEdges(): [T, T][] {
    const edges: [T, T][] = [];
    
    this.adjacencyList.forEach((neighbors, vertex) => {
      for (const neighbor of neighbors) {
        edges.push([vertex, neighbor]);
      }
    });
    
    return edges;
  }
  
  vertexCount(): number {
    return this.adjacencyList.size();
  }
  
  edgeCount(): number {
    let count = 0;
    this.adjacencyList.forEach(neighbors => {
      count += neighbors.size;
    });
    return count;
  }
  
  // Поиск в глубину (DFS)
  *dfs(start: T): Generator<T, void, undefined> {
    const visited = new Set<T>();
    const stack: T[] = [start];
    
    while (stack.length > 0) {
      const vertex = stack.pop()!;
      
      if (visited.has(vertex)) {
        continue;
      }
      
      visited.add(vertex);
      yield vertex;
      
      const neighbors = this.getNeighbors(vertex);
      for (const neighbor of neighbors) {
        if (!visited.has(neighbor)) {
          stack.push(neighbor);
        }
      }
    }
  }
  
  // Поиск в ширину (BFS)
  *bfs(start: T): Generator<T, void, undefined> {
    const visited = new Set<T>();
    const queue: T[] = [start];
    
    while (queue.length > 0) {
      const vertex = queue.shift()!;
      
      if (visited.has(vertex)) {
        continue;
      }
      
      visited.add(vertex);
      yield vertex;
      
      const neighbors = this.getNeighbors(vertex);
      for (const neighbor of neighbors) {
        if (!visited.has(neighbor)) {
          queue.push(neighbor);
        }
      }
    }
  }
  
  // Поиск кратчайшего пути (BFS)
  shortestPath(from: T, to: T): T[] | null {
    const visited = new Set<T>();
    const queue: { vertex: T; path: T[] }[] = [{ vertex: from, path: [from] }];
    
    while (queue.length > 0) {
      const { vertex, path } = queue.shift()!;
      
      if (vertex === to) {
        return path;
      }
      
      if (visited.has(vertex)) {
        continue;
      }
      
      visited.add(vertex);
      
      const neighbors = this.getNeighbors(vertex);
      for (const neighbor of neighbors) {
        if (!visited.has(neighbor)) {
          queue.push({ vertex: neighbor, path: [...path, neighbor] });
        }
      }
    }
    
    return null; // Путь не найден
  }
  
  // Проверка связности графа
  isConnected(): boolean {
    const vertices = this.getVertices();
    if (vertices.length === 0) {
      return true;
    }
    
    const startVertex = vertices[0];
    const visited = new Set(this.dfs(startVertex));
    
    return vertices.every(vertex => visited.has(vertex));
  }
}

// Пример использования Persistent Graph
let graph = PersistentGraph.empty<string>();

// Добавление вершин и ребер
graph = graph.addEdge("A", "B");
graph = graph.addEdge("B", "C");
graph = graph.addEdge("C", "D");
graph = graph.addEdge("A", "D");
graph = graph.addEdge("B", "D");

console.log("Вершины:", graph.getVertices());
console.log("Ребра:", graph.getEdges());
console.log("Количество вершин:", graph.vertexCount());
console.log("Количество ребер:", graph.edgeCount());

console.log("Соседи вершины B:", Array.from(graph.getNeighbors("B")));

// Обход графа
console.log("DFS из A:", Array.from(graph.dfs("A")));
console.log("BFS из A:", Array.from(graph.bfs("A")));

// Поиск кратчайшего пути
const path = graph.shortestPath("A", "C");
console.log("Кратчайший путь от A до C:", path);

console.log("Граф связный:", graph.isConnected());

// Удаление ребра
graph = graph.removeEdge("B", "C");
console.log("После удаления ребра B-C:", graph.getEdges());

// Удаление вершины
graph = graph.removeVertex("D");
console.log("После удаления вершины D:", graph.getVertices());
```

## Практические примеры

### Оптимизация обработки больших наборов данных

```typescript
// Оптимизация обработки больших наборов данных с использованием функциональных структур
class DataProcessor {
  // Обработка данных с использованием Persistent Vector
  static processLargeDataset(data: number[]): PersistentVector<number> {
    console.log("Начало обработки данных...");
    
    // Преобразуем массив в Persistent Vector
    const vector = PersistentVector.fromArray(data);
    
    // Выполняем серию преобразований
    const processed = vector
      .filter(x => x > 0)           // Фильтрация положительных чисел
      .map(x => x * x)              // Возведение в квадрат
      .filter(x => x > 100)         // Фильтрация больших значений
      .map(x => Math.sqrt(x));      // Извлечение квадратного корня
    
    console.log("Обработка завершена");
    return processed;
  }
  
  // Агрегация данных с использованием Immutable Array
  static aggregateData(data: number[]): { sum: number; average: number; max: number; min: number } {
    const immutableArray = new ImmutableArray(data);
    
    const sum = immutableArray.reduce((acc, val) => acc + val, 0);
    const average = sum / immutableArray.length();
    const max = Math.max(...data);
    const min = Math.min(...data);
    
    return { sum, average, max, min };
  }
  
  // Группировка данных с использованием Persistent Hash Map
  static groupData<T>(data: T[], keyFn: (item: T) => string): PersistentHashMap<string, T[]> {
    let map = PersistentHashMap.empty<string, T[]>();
    
    for (const item of data) {
      const key = keyFn(item);
      const group = map.get(key) || [];
      map = map.set(key, [...group, item]);
    }
    
    return map;
  }
}

// Пример использования
const largeDataset = Array.from({ length: 100000 }, () => Math.random() * 1000 - 500);

// Обработка данных
console.time("Обработка данных");
const processedData = DataProcessor.processLargeDataset(largeDataset);
console.timeEnd("Обработка данных");

console.log("Количество обработанных элементов:", processedData.length());

// Агрегация данных
console.time("Агрегация данных");
const aggregation = DataProcessor.aggregateData(largeDataset);
console.timeEnd("Агрегация данных");

console.log("Агрегация:", aggregation);

// Группировка данных
interface Person {
  name: string;
  age: number;
  department: string;
}

const people: Person[] = [
  { name: "Иван", age: 30, department: "IT" },
  { name: "Мария", age: 25, department: "HR" },
  { name: "Алексей", age: 35, department: "IT" },
  { name: "Елена", age: 28, department: "Finance" },
  { name: "Дмитрий", age: 32, department: "IT" },
  { name: "Анна", age: 27, department: "HR" }
];

console.time("Группировка данных");
const groupedByDepartment = DataProcessor.groupData(people, p => p.department);
console.timeEnd("Группировка данных");

console.log("Группировка по отделам:");
groupedByDepartment.forEach((employees, department) => {
  console.log(`${department}: ${employees.length} сотрудников`);
});
```

### Кэширование с использованием функциональных структур

```typescript
// Кэширование с использованием функциональных структур данных
class FunctionalCache<K, V> {
  private readonly cache: PersistentHashMap<K, { value: V; timestamp: number }>;
  private readonly maxSize: number;
  private readonly ttl: number;
  
  constructor(
    cache: PersistentHashMap<K, { value: V; timestamp: number }> = PersistentHashMap.empty(),
    maxSize: number = 1000,
    ttl: number = 5 * 60 * 1000 // 5 минут
  ) {
    this.cache = cache;
    this.maxSize = maxSize;
    this.ttl = ttl;
  }
  
  get(key: K): V | undefined {
    const entry = this.cache.get(key);
    
    if (!entry) {
      return undefined;
    }
    
    // Проверяем TTL
    if (Date.now() - entry.timestamp > this.ttl) {
      // Удаляем устаревшую запись
      const newCache = this.cache.delete(key);
      return undefined;
    }
    
    return entry.value;
  }
  
  set(key: K, value: V): FunctionalCache<K, V> {
    const newEntry = { value, timestamp: Date.now() };
    let newCache = this.cache.set(key, newEntry);
    
    // Проверяем размер кэша
    if (newCache.size() > this.maxSize) {
      // Удаляем самую старую запись
      const oldestKey = this.findOldestKey();
      if (oldestKey) {
        newCache = newCache.delete(oldestKey);
      }
    }
    
    return new FunctionalCache(newCache, this.maxSize, this.ttl);
  }
  
  private findOldestKey(): K | undefined {
    let oldestKey: K | undefined;
    let oldestTimestamp = Infinity;
    
    this.cache.forEach((entry, key) => {
      if (entry.timestamp < oldestTimestamp) {
        oldestTimestamp = entry.timestamp;
        oldestKey = key;
      }
    });
    
    return oldestKey;
  }
  
  size(): number {
    return this.cache.size();
  }
  
  clear(): FunctionalCache<K, V> {
    return new FunctionalCache(PersistentHashMap.empty(), this.maxSize, this.ttl);
  }
  
  // Получение статистики кэша
  getStats(): { size: number; maxSize: number; ttl: number } {
    return {
      size: this.cache.size(),
      maxSize: this.maxSize,
      ttl: this.ttl
    };
  }
}

// Пример использования функционального кэша
let cache = new FunctionalCache<string, number>();

// Добавление элементов в кэш
cache = cache.set("key1", 100);
cache = cache.set("key2", 200);
cache = cache.set("key3", 300);

console.log("Значение key1:", cache.get("key1"));
console.log("Значение key2:", cache.get("key2"));
console.log("Значение несуществующего ключа:", cache.get("key4"));

console.log("Статистика кэша:", cache.getStats());

// Имитация истечения TTL
setTimeout(() => {
  console.log("После истечения TTL:", cache.get("key1"));
}, 6000);
```

Оптимизация структур данных в функциональном программировании позволяет создавать эффективные и надежные приложения, сохраняя преимущества неизменяемости и чистоты функций. Использование специализированных структур данных, таких как Persistent Arrays, Hash Maps и Trees, помогает достичь высокой производительности при работе с большими объемами данных.