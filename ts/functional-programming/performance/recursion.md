# Оптимизация рекурсии

Рекурсия - важная часть функционального программирования, но без оптимизации она может привести к переполнению стека. В этой главе мы рассмотрим различные техники оптимизации рекурсивных функций.

## Содержание

1. [Проблемы рекурсии](#проблемы-рекурсии)
2. [Хвостовая рекурсия](#хвостовая-рекурсия)
3. [Trampoline](#trampoline)
4. [Continuation Passing Style](#continuation-passing-style)
5. [Мемоизация рекурсивных функций](#мемоизация-рекурсивных-функций)
6. [Разделяй и властвуй](#разделяй-и-властвуй)
7. [Практические примеры](#практические-примеры)

## Проблемы рекурсии

Рекурсивные функции могут сталкиваться с несколькими проблемами, особенно при работе с большими наборами данных:

### Переполнение стека

```typescript
// Пример рекурсивной функции, которая может вызвать переполнение стека
function factorial(n: number): number {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n - 1);
}

// Эта функция работает нормально для небольших значений
console.log(factorial(5)); // 120

// Но может вызвать переполнение стека для больших значений
try {
  console.log(factorial(100000)); // Скорее всего вызовет ошибку
} catch (error) {
  console.log("Ошибка переполнения стека:", error);
}
```

### Проблемы с производительностью

```typescript
// Неэффективная рекурсивная реализация чисел Фибоначчи
function fibonacci(n: number): number {
  if (n <= 1) {
    return n;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Эта функция имеет экспоненциальную сложность O(2^n)
console.log(fibonacci(10)); // 55

// Для больших значений она становится очень медленной
console.time("Фибоначчи 35");
console.log(fibonacci(35)); // 9227465
console.timeEnd("Фибоначчи 35");
```

### Проблемы с памятью

```typescript
// Рекурсивная функция, которая накапливает промежуточные результаты
function sumRecursive(numbers: number[]): number {
  if (numbers.length === 0) {
    return 0;
  }
  
  const [head, ...tail] = numbers;
  return head + sumRecursive(tail); // Создает новые массивы на каждом шаге
}

// Эта функция использует много памяти для больших массивов
const largeArray = Array.from({ length: 10000 }, (_, i) => i);
try {
  console.log(sumRecursive(largeArray));
} catch (error) {
  console.log("Ошибка из-за использования памяти:", error);
}
```

## Хвостовая рекурсия

Хвостовая рекурсия - это оптимизация, при которой рекурсивный вызов является последней операцией в функции. Это позволяет компилятору или интерпретатору оптимизировать использование стека.

### Преобразование в хвостовую рекурсию

```typescript
// Не хвостовая рекурсия
function factorial(n: number): number {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n - 1); // Рекурсивный вызов не последний
}

// Хвостовая рекурсия
function factorialTailRecursive(n: number, accumulator: number = 1): number {
  if (n <= 1) {
    return accumulator;
  }
  return factorialTailRecursive(n - 1, n * accumulator); // Рекурсивный вызов последний
}

// Тестирование
console.log(factorial(5)); // 120
console.log(factorialTailRecursive(5)); // 120

// Хвостовая рекурсия для суммы массива
function sumTailRecursive(numbers: number[], index: number = 0, accumulator: number = 0): number {
  if (index >= numbers.length) {
    return accumulator;
  }
  return sumTailRecursive(numbers, index + 1, accumulator + numbers[index]);
}

const testArray = [1, 2, 3, 4, 5];
console.log(sumTailRecursive(testArray)); // 15
```

### Хвостовая рекурсия для различных задач

```typescript
// Хвостовая рекурсия для вычисления степени
function powerTailRecursive(base: number, exponent: number, accumulator: number = 1): number {
  if (exponent === 0) {
    return accumulator;
  }
  if (exponent < 0) {
    return powerTailRecursive(1 / base, -exponent, accumulator);
  }
  return powerTailRecursive(base, exponent - 1, accumulator * base);
}

console.log(powerTailRecursive(2, 10)); // 1024
console.log(powerTailRecursive(3, 4));  // 81

// Хвостовая рекурсия для поиска в массиве
function findTailRecursive<T>(array: T[], predicate: (item: T) => boolean, index: number = 0): T | undefined {
  if (index >= array.length) {
    return undefined;
  }
  
  if (predicate(array[index])) {
    return array[index];
  }
  
  return findTailRecursive(array, predicate, index + 1);
}

const numbers = [1, 3, 5, 7, 9, 11, 13];
console.log(findTailRecursive(numbers, x => x > 6)); // 7
console.log(findTailRecursive(numbers, x => x > 20)); // undefined

// Хвостовая рекурсия для фильтрации массива
function filterTailRecursive<T>(array: T[], predicate: (item: T) => boolean): T[] {
  function filterHelper(index: number, result: T[]): T[] {
    if (index >= array.length) {
      return result;
    }
    
    const currentItem = array[index];
    const newResult = predicate(currentItem) ? [...result, currentItem] : result;
    
    return filterHelper(index + 1, newResult);
  }
  
  return filterHelper(0, []);
}

const evenNumbers = filterTailRecursive(numbers, x => x % 2 === 0);
console.log(evenNumbers); // [ ]

const oddNumbers = filterTailRecursive(numbers, x => x % 2 === 1);
console.log(oddNumbers); // [1, 3, 5, 7, 9, 11, 13]
```

### Оптимизированная хвостовая рекурсия

```typescript
// Оптимизированная хвостовая рекурсия с использованием аккумуляторов
class TailRecursionOptimizer {
  // Оптимизированная функция для вычисления НОД (алгоритм Евклида)
  static gcd(a: number, b: number): number {
    function gcdHelper(x: number, y: number): number {
      if (y === 0) {
        return x;
      }
      return gcdHelper(y, x % y);
    }
    
    return gcdHelper(Math.abs(a), Math.abs(b));
  }
  
  // Оптимизированная функция для вычисления НОК
  static lcm(a: number, b: number): number {
    return Math.abs(a * b) / this.gcd(a, b);
  }
  
  // Оптимизированная функция для проверки простоты числа
  static isPrime(n: number): boolean {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 === 0 || n % 3 === 0) return false;
    
    function isPrimeHelper(i: number): boolean {
      if (i * i > n) {
        return true;
      }
      if (n % i === 0 || n % (i + 2) === 0) {
        return false;
      }
      return isPrimeHelper(i + 6);
    }
    
    return isPrimeHelper(5);
  }
  
  // Оптимизированная функция для генерации простых чисел
  static generatePrimes(count: number): number[] {
    const primes: number[] = [];
    
    function generateHelper(current: number): void {
      if (primes.length >= count) {
        return;
      }
      
      if (this.isPrime(current)) {
        primes.push(current);
      }
      
      generateHelper(current + 1);
    }
    
    if (count > 0) {
      generateHelper(2);
    }
    
    return primes;
  }
}

// Тестирование оптимизированных функций
console.log("НОД(48, 18):", TailRecursionOptimizer.gcd(48, 18)); // 6
console.log("НОК(4, 6):", TailRecursionOptimizer.lcm(4, 6)); // 12
console.log("Простое число 17:", TailRecursionOptimizer.isPrime(17)); // true
console.log("Первые 10 простых чисел:", TailRecursionOptimizer.generatePrimes(10));
```

## Trampoline

Trampoline - это техника, которая позволяет выполнять рекурсивные функции без переполнения стека, преобразуя рекурсивные вызовы в итерации.

### Базовая реализация Trampoline

```typescript
// Типы для Trampoline
type Trampoline<T> = { value: T } | { continuation: () => Trampoline<T> };

function done<T>(value: T): Trampoline<T> {
  return { value };
}

function next<T>(continuation: () => Trampoline<T>): Trampoline<T> {
  return { continuation };
}

function trampoline<T>(trampoline: Trampoline<T>): T {
  let current = trampoline;
  
  while (!('value' in current)) {
    current = current.continuation();
  }
  
  return current.value;
}

// Рекурсивная функция с использованием trampoline
function sumTrampoline(numbers: number[], index: number = 0, accumulator: number = 0): Trampoline<number> {
  if (index >= numbers.length) {
    return done(accumulator);
  }
  
  return next(() => sumTrampoline(numbers, index + 1, accumulator + numbers[index]));
}

// Пример использования
const largeArray = Array.from({ length: 100000 }, (_, i) => i + 1);
const sum = trampoline(sumTrampoline(largeArray));
console.log(`Сумма: ${sum}`); // 5000050000
```

### Расширенная реализация Trampoline

```typescript
// Расширенная реализация Trampoline с поддержкой ошибок
type TrampolineResult<T> = 
  | { type: 'value'; value: T }
  | { type: 'continuation'; continuation: () => TrampolineResult<T> }
  | { type: 'error'; error: any };

function doneResult<T>(value: T): TrampolineResult<T> {
  return { type: 'value', value };
}

function nextResult<T>(continuation: () => TrampolineResult<T>): TrampolineResult<T> {
  return { type: 'continuation', continuation };
}

function errorResult<T>(error: any): TrampolineResult<T> {
  return { type: 'error', error };
}

function trampolineResult<T>(trampoline: TrampolineResult<T>): T {
  let current = trampoline;
  
  while (current.type === 'continuation') {
    try {
      current = current.continuation();
    } catch (error) {
      return trampolineResult(errorResult(error));
    }
  }
  
  if (current.type === 'error') {
    throw current.error;
  }
  
  return current.value;
}

// Рекурсивная функция с обработкой ошибок
function factorialTrampoline(n: number): TrampolineResult<number> {
  if (n < 0) {
    return errorResult(new Error("Факториал определен только для неотрицательных чисел"));
  }
  
  function factorialHelper(num: number, acc: number): TrampolineResult<number> {
    if (num <= 1) {
      return doneResult(acc);
    }
    
    if (acc > Number.MAX_SAFE_INTEGER / num) {
      return errorResult(new Error("Переполнение при вычислении факториала"));
    }
    
    return nextResult(() => factorialHelper(num - 1, acc * num));
  }
  
  return factorialHelper(n, 1);
}

// Пример использования
try {
  const result = trampolineResult(factorialTrampoline(10));
  console.log(`10! = ${result}`); // 3628800
} catch (error) {
  console.log("Ошибка:", error.message);
}

try {
  const result = trampolineResult(factorialTrampoline(-5));
  console.log(result);
} catch (error) {
  console.log("Ошибка:", error.message); // Факториал определен только для неотрицательных чисел
}
```

### Trampoline для сложных структур данных

```typescript
// Trampoline для работы с деревьями
interface TreeNode<T> {
  value: T;
  left?: TreeNode<T>;
  right?: TreeNode<T>;
}

// Рекурсивный обход дерева с использованием trampoline
function traverseTreeTrampoline<T>(node: TreeNode<T> | undefined): TrampolineResult<T[]> {
  if (!node) {
    return doneResult([]);
  }
  
  function traverseHelper(current: TreeNode<T> | undefined, result: T[]): TrampolineResult<T[]> {
    if (!current) {
      return doneResult(result);
    }
    
    // Добавляем значение текущего узла
    const newResult = [...result, current.value];
    
    // Рекурсивно обрабатываем левое поддерево
    return nextResult(() => {
      function handleLeft(leftResult: T[]): TrampolineResult<T[]> {
        // Затем обрабатываем правое поддерево
        return nextResult(() => traverseHelper(current.right, leftResult));
      }
      
      return traverseHelper(current.left, newResult).type === 'continuation'
        ? traverseHelper(current.left, newResult)
        : doneResult(handleLeft(trampolineResult(traverseHelper(current.left, newResult))));
    });
  }
  
  return traverseHelper(node, []);
}

// Создание тестового дерева
const testTree: TreeNode<number> = {
  value: 1,
  left: {
    value: 2,
    left: { value: 4 },
    right: { value: 5 }
  },
  right: {
    value: 3,
    left: { value: 6 },
    right: { value: 7 }
  }
};

// Обход дерева
try {
  const traversalResult = trampolineResult(traverseTreeTrampoline(testTree));
  console.log("Обход дерева:", traversalResult); // [1, 2, 4, 5, 3, 6, 7]
} catch (error) {
  console.log("Ошибка при обходе дерева:", error.message);
}
```

## Continuation Passing Style

Continuation Passing Style (CPS) - это стиль программирования, при котором функции принимают дополнительный параметр "продолжение", которое вызывается с результатом вычислений.

### Основы CPS

```typescript
// Обычная функция
function add(a: number, b: number): number {
  return a + b;
}

// Та же функция в CPS
function addCPS(a: number, b: number, continuation: (result: number) => void): void {
  continuation(a + b);
}

// Пример использования
addCPS(2, 3, (result) => {
  console.log(`Результат: ${result}`); // 5
});

// Рекурсивная функция в CPS
function factorialCPS(n: number, continuation: (result: number) => void): void {
  if (n <= 1) {
    continuation(1);
    return;
  }
  
  factorialCPS(n - 1, (result) => {
    continuation(n * result);
  });
}

// Пример использования
factorialCPS(5, (result) => {
  console.log(`Факториал: ${result}`); // 120
});
```

### CPS для массивов

```typescript
// CPS для map
function mapCPS<T, U>(
  array: T[],
  transform: (item: T, continuation: (result: U) => void) => void,
  continuation: (result: U[]) => void
): void {
  if (array.length === 0) {
    continuation([]);
    return;
  }
  
  const [head, ...tail] = array;
  
  transform(head, (transformedHead) => {
    mapCPS(tail, transform, (transformedTail) => {
      continuation([transformedHead, ...transformedTail]);
    });
  });
}

// Пример использования mapCPS
const numbers = [1, 2, 3, 4, 5];
mapCPS(
  numbers,
  (item, cont) => cont(item * item), // Возведение в квадрат
  (result) => console.log(`Квадраты: ${result}`) // [1, 4, 9, 16, 25]
);

// CPS для filter
function filterCPS<T>(
  array: T[],
  predicate: (item: T, continuation: (result: boolean) => void) => void,
  continuation: (result: T[]) => void
): void {
  if (array.length === 0) {
    continuation([]);
    return;
  }
  
  const [head, ...tail] = array;
  
  predicate(head, (shouldInclude) => {
    filterCPS(tail, predicate, (filteredTail) => {
      const result = shouldInclude ? [head, ...filteredTail] : filteredTail;
      continuation(result);
    });
  });
}

// Пример использования filterCPS
filterCPS(
  numbers,
  (item, cont) => cont(item % 2 === 0), // Только четные числа
  (result) => console.log(`Четные числа: ${result}`) // [2, 4]
);

// CPS для reduce
function reduceCPS<T, U>(
  array: T[],
  reducer: (accumulator: U, currentValue: T, continuation: (result: U) => void) => void,
  initialValue: U,
  continuation: (result: U) => void
): void {
  if (array.length === 0) {
    continuation(initialValue);
    return;
  }
  
  const [head, ...tail] = array;
  
  reducer(initialValue, head, (newAccumulator) => {
    reduceCPS(tail, reducer, newAccumulator, continuation);
  });
}

// Пример использования reduceCPS
reduceCPS(
  numbers,
  (acc, val, cont) => cont(acc + val), // Сумма
  0,
  (result) => console.log(`Сумма: ${result}`) // 15
);
```

### Расширенный CPS

```typescript
// CPS с поддержкой асинхронных операций
function asyncMapCPS<T, U>(
  array: T[],
  asyncTransform: (item: T, continuation: (result: U) => void) => void,
  continuation: (result: U[]) => void
): void {
  if (array.length === 0) {
    continuation([]);
    return;
  }
  
  const [head, ...tail] = array;
  
  asyncTransform(head, (transformedHead) => {
    asyncMapCPS(tail, asyncTransform, (transformedTail) => {
      continuation([transformedHead, ...transformedTail]);
    });
  });
}

// Пример асинхронной трансформации
function asyncSquare(item: number, continuation: (result: number) => void): void {
  setTimeout(() => {
    continuation(item * item);
  }, Math.random() * 100); // Случайная задержка
}

// Использование асинхронного CPS
const asyncNumbers = [1, 2, 3, 4, 5];
console.log("Начало асинхронного преобразования...");

asyncMapCPS(
  asyncNumbers,
  asyncSquare,
  (result) => console.log(`Асинхронные квадраты: ${result}`)
);

// CPS с обработкой ошибок
function safeMapCPS<T, U>(
  array: T[],
  safeTransform: (item: T, onSuccess: (result: U) => void, onError: (error: any) => void) => void,
  continuation: (result: { success: U[]; errors: any[] }) => void
): void {
  if (array.length === 0) {
    continuation({ success: [], errors: [] });
    return;
  }
  
  const [head, ...tail] = array;
  
  safeTransform(
    head,
    (transformedHead) => {
      safeMapCPS(tail, safeTransform, (tailResult) => {
        continuation({
          success: [transformedHead, ...tailResult.success],
          errors: tailResult.errors
        });
      });
    },
    (error) => {
      safeMapCPS(tail, safeTransform, (tailResult) => {
        continuation({
          success: tailResult.success,
          errors: [error, ...tailResult.errors]
        });
      });
    }
  );
}

// Пример безопасной трансформации
function safeParseInt(item: string, onSuccess: (result: number) => void, onError: (error: any) => void): void {
  const parsed = parseInt(item, 10);
  if (isNaN(parsed)) {
    onError(new Error(`Невозможно преобразовать "${item}" в число`));
  } else {
    onSuccess(parsed);
  }
}

// Использование безопасного CPS
const stringNumbers = ["1", "2", "invalid", "4", "5"];
safeMapCPS(
  stringNumbers,
  safeParseInt,
  (result) => {
    console.log("Успешные преобразования:", result.success); // [1, 2, 4, 5]
    console.log("Ошибки:", result.errors); // [Error: Невозможно преобразовать "invalid" в число]
  }
);
```

## Мемоизация рекурсивных функций

Мемоизация рекурсивных функций позволяет избежать повторных вычислений и значительно улучшить производительность.

### Базовая мемоизация рекурсии

```typescript
// Мемоизация рекурсивных функций
function memoizeRecursive<T extends any[], R>(
  fn: (...args: T) => R,
  keyFn: (...args: T) => string = (...args) => JSON.stringify(args)
): (...args: T) => R {
  const cache = new Map<string, R>();
  
  function memoizedFn(...args: T): R {
    const key = keyFn(...args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn.apply(null, args);
    cache.set(key, result);
    return result;
  }
  
  return memoizedFn;
}

// Мемоизированная рекурсивная функция Фибоначчи
const fibonacciMemoized = memoizeRecursive((n: number): number => {
  console.log(`Вычисление fibonacci(${n})`);
  if (n <= 1) return n;
  return fibonacciMemoized(n - 1) + fibonacciMemoized(n - 2);
});

console.time("Фибоначчи с мемоизацией 40");
console.log(fibonacciMemoized(40)); // 102334155
console.timeEnd("Фибоначчи с мемоизацией 40");

console.time("Фибоначчи с мемоизацией 40 (повтор)");
console.log(fibonacciMemoized(40)); // 102334155 (из кэша)
console.timeEnd("Фибоначчи с мемоизацией 40 (повтор)");
```

### Расширенная мемоизация с LRU

```typescript
// Расширенная мемоизация с LRU кэшем
interface CacheEntry<T> {
  value: T;
  timestamp: number;
  lastAccess: number;
}

class LRUCache<K, V> {
  private cache = new Map<K, CacheEntry<V>>();
  private readonly maxSize: number;
  private readonly ttl: number;
  
  constructor(maxSize: number, ttl: number) {
    this.maxSize = maxSize;
    this.ttl = ttl;
  }
  
  get(key: K): V | undefined {
    const now = Date.now();
    const entry = this.cache.get(key);
    
    if (!entry) {
      return undefined;
    }
    
    // Проверяем TTL
    if (now - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return undefined;
    }
    
    // Обновляем время последнего доступа
    entry.lastAccess = now;
    
    // Перемещаем элемент в конец (самый свежий)
    this.cache.delete(key);
    this.cache.set(key, entry);
    
    return entry.value;
  }
  
  set(key: K, value: V): void {
    const now = Date.now();
    
    // Если кэш переполнен, удаляем самый старый элемент
    if (this.cache.size >= this.maxSize) {
      this.evictOldest(now);
    }
    
    this.cache.set(key, {
      value,
      timestamp: now,
      lastAccess: now
    });
  }
  
  private evictOldest(now: number): void {
    let oldestKey: K | undefined;
    let oldestAccess = Infinity;
    
    // Находим наименее недавно использованный элемент
    for (const [key, entry] of this.cache.entries()) {
      // Проверяем TTL
      if (now - entry.timestamp > this.ttl) {
        this.cache.delete(key);
        return;
      }
      
      if (entry.lastAccess < oldestAccess) {
        oldestAccess = entry.lastAccess;
        oldestKey = key;
      }
    }
    
    if (oldestKey !== undefined) {
      this.cache.delete(oldestKey);
    }
  }
  
  has(key: K): boolean {
    return this.get(key) !== undefined;
  }
  
  size(): number {
    return this.cache.size;
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Мемоизация с расширенным LRU кэшем
function advancedMemoizeRecursive<T extends any[], R>(
  fn: (...args: T) => R,
  maxSize: number = 1000,
  ttl: number = 5 * 60 * 1000 // 5 минут
): (...args: T) => R {
  const cache = new LRUCache<string, R>(maxSize, ttl);
  
  function memoizedFn(...args: T): R {
    const key = JSON.stringify(args);
    const cached = cache.get(key);
    
    if (cached !== undefined) {
      return cached;
    }
    
    const result = fn.apply(null, args);
    cache.set(key, result);
    return result;
  }
  
  return memoizedFn;
}

// Пример использования расширенной мемоизации
const advancedFibonacci = advancedMemoizeRecursive((n: number): number => {
  if (n <= 1) return n;
  return advancedFibonacci(n - 1) + advancedFibonacci(n - 2);
}, 1000, 10000); // 1000 элементов, 10 секунд TTL

console.log(advancedFibonacci(50)); // Быстро вычисляется
console.log(advancedFibonacci(50)); // Из кэша
```

### Мемоизация взаимно рекурсивных функций

```typescript
// Мемоизация взаимно рекурсивных функций
function createMutualMemoization<T, U>() {
  const cache1 = new Map<string, T>();
  const cache2 = new Map<string, U>();
  
  return {
    memoize1: (fn: (arg: any) => T, keyFn: (arg: any) => string = (arg) => String(arg)) => {
      return (arg: any): T => {
        const key = keyFn(arg);
        if (cache1.has(key)) {
          return cache1.get(key)!;
        }
        const result = fn(arg);
        cache1.set(key, result);
        return result;
      };
    },
    
    memoize2: (fn: (arg: any) => U, keyFn: (arg: any) => string = (arg) => String(arg)) => {
      return (arg: any): U => {
        const key = keyFn(arg);
        if (cache2.has(key)) {
          return cache2.get(key)!;
        }
        const result = fn(arg);
        cache2.set(key, result);
        return result;
      };
    }
  };
}

// Пример взаимно рекурсивных функций (четное/нечетное)
const { memoize1, memoize2 } = createMutualMemoization<boolean, boolean>();

let isEven: (n: number) => boolean;
let isOdd: (n: number) => boolean;

isEven = memoize1((n: number): boolean => {
  console.log(`Проверка четности ${n}`);
  if (n === 0) return true;
  return isOdd(n - 1);
});

isOdd = memoize2((n: number): boolean => {
  console.log(`Проверка нечетности ${n}`);
  if (n === 0) return false;
  return isEven(n - 1);
});

console.log(isEven(1000)); // false
console.log(isOdd(1000));  // true
console.log(isEven(1000)); // Из кэша
```

## Разделяй и властвуй

Алгоритмы "разделяй и властвуй" эффективно используют рекурсию для решения сложных задач путем их разбиения на более простые подзадачи.

### Базовые алгоритмы "разделяй и властвуй"

```typescript
// Быстрая сортировка (QuickSort)
function quickSort<T>(array: T[]): T[] {
  if (array.length <= 1) {
    return array;
  }
  
  const pivot = array[Math.floor(array.length / 2)];
  const less = array.filter(x => x < pivot);
  const equal = array.filter(x => x === pivot);
  const greater = array.filter(x => x > pivot);
  
  return [...quickSort(less), ...equal, ...quickSort(greater)];
}

// Сортировка слиянием (MergeSort)
function mergeSort<T>(array: T[]): T[] {
  if (array.length <= 1) {
    return array;
  }
  
  const middle = Math.floor(array.length / 2);
  const left = array.slice(0, middle);
  const right = array.slice(middle);
  
  return merge(mergeSort(left), mergeSort(right));
}

function merge<T>(left: T[], right: T[]): T[] {
  const result: T[] = [];
  let leftIndex = 0;
  let rightIndex = 0;
  
  while (leftIndex < left.length && rightIndex < right.length) {
    if (left[leftIndex] < right[rightIndex]) {
      result.push(left[leftIndex]);
      leftIndex++;
    } else {
      result.push(right[rightIndex]);
      rightIndex++;
    }
  }
  
  return result
    .concat(left.slice(leftIndex))
    .concat(right.slice(rightIndex));
}

// Тестирование алгоритмов сортировки
const unsortedArray = [64, 34, 25, 12, 22, 11, 90, 5];
console.log("Исходный массив:", unsortedArray);
console.log("Быстрая сортировка:", quickSort(unsortedArray));
console.log("Сортировка слиянием:", mergeSort(unsortedArray));
```

### Оптимизированные алгоритмы "разделяй и властвуй"

```typescript
// Оптимизированная быстрая сортировка с хвостовой рекурсией
function optimizedQuickSort<T>(array: T[]): T[] {
  if (array.length <= 1) {
    return array;
  }
  
  return quickSortHelper(array, 0, array.length - 1);
}

function quickSortHelper<T>(array: T[], low: number, high: number): T[] {
  if (low < high) {
    const pivotIndex = partition(array, low, high);
    
    // Рекурсивно сортируем элементы до и после разделения
    quickSortHelper(array, low, pivotIndex - 1);
    quickSortHelper(array, pivotIndex + 1, high);
  }
  
  return array;
}

function partition<T>(array: T[], low: number, high: number): number {
  const pivot = array[high];
  let i = low - 1;
  
  for (let j = low; j < high; j++) {
    if (array[j] <= pivot) {
      i++;
      [array[i], array[j]] = [array[j], array[i]]; // Swap
    }
  }
  
  [array[i + 1], array[high]] = [array[high], array[i + 1]]; // Swap
  return i + 1;
}

// Бинарный поиск
function binarySearch<T>(array: T[], target: T): number {
  return binarySearchHelper(array, target, 0, array.length - 1);
}

function binarySearchHelper<T>(array: T[], target: T, low: number, high: number): number {
  if (low > high) {
    return -1; // Элемент не найден
  }
  
  const mid = Math.floor((low + high) / 2);
  
  if (array[mid] === target) {
    return mid;
  } else if (array[mid] > target) {
    return binarySearchHelper(array, target, low, mid - 1);
  } else {
    return binarySearchHelper(array, target, mid + 1, high);
  }
}

// Тестирование оптимизированных алгоритмов
const sortedArray = [1, 5, 11, 12, 22, 25, 34, 64, 90];
console.log("Оптимизированная быстрая сортировка:", optimizedQuickSort([...unsortedArray]));
console.log("Бинарный поиск 25:", binarySearch(sortedArray, 25)); // 5
console.log("Бинарный поиск 100:", binarySearch(sortedArray, 100)); // -1
```

### Параллельные алгоритмы "разделяй и властвуй"

```typescript
// Параллельная сортировка слиянием
async function parallelMergeSort<T>(array: T[]): Promise<T[]> {
  if (array.length <= 1) {
    return array;
  }
  
  const middle = Math.floor(array.length / 2);
  const left = array.slice(0, middle);
  const right = array.slice(middle);
  
  // Параллельно сортируем левую и правую части
  const [sortedLeft, sortedRight] = await Promise.all([
    parallelMergeSort(left),
    parallelMergeSort(right)
  ]);
  
  return merge(sortedLeft, sortedRight);
}

// Параллельный бинарный поиск
async function parallelBinarySearch<T>(array: T[], target: T): Promise<number> {
  return binarySearch(array, target);
}

// Тестирование параллельных алгоритмов
async function testParallelAlgorithms() {
  const largeArray = Array.from({ length: 10000 }, () => Math.floor(Math.random() * 10000));
  
  console.time("Параллельная сортировка");
  const sorted = await parallelMergeSort(largeArray);
  console.timeEnd("Параллельная сортировка");
  
  const target = sorted[Math.floor(sorted.length / 2)];
  console.time("Параллельный поиск");
  const index = await parallelBinarySearch(sorted, target);
  console.timeEnd("Параллельный поиск");
  
  console.log(`Найден элемент ${target} по индексу ${index}`);
}

// testParallelAlgorithms();
```

## Практические примеры

### Оптимизация обработки деревьев

```typescript
// Оптимизированная обработка деревьев с использованием различных техник
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}

class TreeProcessor {
  // Рекурсивная обработка дерева с хвостовой рекурсией
  static processTreeTailRecursive<T, R>(
    node: TreeNode<T>,
    processor: (value: T) => R,
    combiner: (results: R[]) => R
  ): R {
    function processHelper(nodes: TreeNode<T>[], results: R[]): R {
      if (nodes.length === 0) {
        return combiner(results);
      }
      
      const [currentNode, ...remainingNodes] = nodes;
      const processedValue = processor(currentNode.value);
      const childResults = currentNode.children.map(child => 
        processHelper([child], [])
      );
      
      const combinedChildren = childResults.length > 0 
        ? combiner(childResults) 
        : processedValue;
      
      const newResults = [...results, combinedChildren];
      
      return processHelper(remainingNodes, newResults);
    }
    
    return processHelper([node], []);
  }
  
  // Обработка дерева с использованием trampoline
  static processTreeTrampoline<T, R>(
    node: TreeNode<T>,
    processor: (value: T) => R,
    combiner: (results: R[]) => R
  ): R {
    type TreeTrampoline = 
      | { type: 'value'; value: R }
      | { type: 'continuation'; continuation: () => TreeTrampoline };
    
    function done(value: R): TreeTrampoline {
      return { type: 'value', value };
    }
    
    function next(continuation: () => TreeTrampoline): TreeTrampoline {
      return { type: 'continuation', continuation };
    }
    
    function trampoline(t: TreeTrampoline): R {
      let current = t;
      while (current.type === 'continuation') {
        current = current.continuation();
      }
      return current.value;
    }
    
    function processNode(currentNode: TreeNode<T>): TreeTrampoline {
      const processedValue = processor(currentNode.value);
      
      if (currentNode.children.length === 0) {
        return done(processedValue);
      }
      
      return next(() => {
        const childResults = currentNode.children.map(child => 
          trampoline(processNode(child))
        );
        const combined = combiner([processedValue, ...childResults]);
        return done(combined);
      });
    }
    
    return trampoline(processNode(node));
  }
  
  // Мемоизированная обработка дерева
  static createMemoizedTreeProcessor<T, R>() {
    const cache = new Map<string, R>();
    
    return {
      process: (
        node: TreeNode<T>,
        processor: (value: T) => R,
        combiner: (results: R[]) => R,
        keyFn: (node: TreeNode<T>) => string = (n) => JSON.stringify(n.value)
      ): R => {
        const key = keyFn(node);
        
        if (cache.has(key)) {
          return cache.get(key)!;
        }
        
        const processedValue = processor(node.value);
        
        if (node.children.length === 0) {
          cache.set(key, processedValue);
          return processedValue;
        }
        
        const childResults = node.children.map(child => 
          this.process(child, processor, combiner, keyFn)
        );
        
        const result = combiner([processedValue, ...childResults]);
        cache.set(key, result);
        return result;
      },
      
      clearCache: () => {
        cache.clear();
      }
    };
  }
}

// Создание тестового дерева
const testTree: TreeNode<number> = {
  value: 1,
  children: [
    {
      value: 2,
      children: [
        { value: 4, children: [] },
        { value: 5, children: [] }
      ]
    },
    {
      value: 3,
      children: [
        { value: 6, children: [] },
        { value: 7, children: [] }
      ]
    }
  ]
};

// Тестирование различных подходов
console.log("Обработка дерева с хвостовой рекурсией:");
const sumResult = TreeProcessor.processTreeTailRecursive(
  testTree,
  x => x,
  arr => arr.reduce((sum, val) => sum + val, 0)
);
console.log("Сумма всех значений:", sumResult); // 28

console.log("Обработка дерева с trampoline:");
const maxResult = TreeProcessor.processTreeTrampoline(
  testTree,
  x => x,
  arr => Math.max(...arr)
);
console.log("Максимальное значение:", maxResult); // 7

console.log("Мемоизированная обработка дерева:");
const memoizedProcessor = TreeProcessor.createMemoizedTreeProcessor<number, number>();
const memoizedSum = memoizedProcessor.process(
  testTree,
  x => x,
  arr => arr.reduce((sum, val) => sum + val, 0)
);
console.log("Сумма с мемоизацией:", memoizedSum); // 28
```

### Оптимизация математических вычислений

```typescript
// Оптимизированные математические функции с использованием рекурсии
class MathOptimizer {
  // Оптимизированное вычисление факториала
  static factorial(n: number): number {
    if (n < 0) {
      throw new Error("Факториал определен только для неотрицательных чисел");
    }
    
    function factorialHelper(num: number, acc: number): number {
      if (num <= 1) {
        return acc;
      }
      return factorialHelper(num - 1, acc * num);
    }
    
    return factorialHelper(n, 1);
  }
  
  // Оптимизированное вычисление чисел Фибоначчи с мемоизацией
  static createFibonacciCalculator() {
    const cache = new Map<number, number>();
    
    function fibonacci(n: number): number {
      if (n <= 1) return n;
      
      if (cache.has(n)) {
        return cache.get(n)!;
      }
      
      const result = fibonacci(n - 1) + fibonacci(n - 2);
      cache.set(n, result);
      return result;
    }
    
    return {
      calculate: fibonacci,
      clearCache: () => cache.clear(),
      getCacheSize: () => cache.size
    };
  }
  
  // Оптимизированное вычисление НОД
  static gcd(a: number, b: number): number {
    function gcdHelper(x: number, y: number): number {
      if (y === 0) {
        return x;
      }
      return gcdHelper(y, x % y);
    }
    
    return gcdHelper(Math.abs(a), Math.abs(b));
  }
  
  // Оптимизированное вычисление степени
  static power(base: number, exponent: number): number {
    if (exponent === 0) return 1;
    if (exponent === 1) return base;
    
    function powerHelper(exp: number, acc: number): number {
      if (exp === 0) {
        return acc;
      }
      if (exp % 2 === 0) {
        return powerHelper(exp / 2, acc * acc);
      }
      return powerHelper(exp - 1, acc * base);
    }
    
    if (exponent < 0) {
      return 1 / powerHelper(-exponent, 1);
    }
    
    return powerHelper(exponent, 1);
  }
  
  // Оптимизированное вычисление квадратного корня (метод Ньютона)
  static sqrt(n: number, precision: number = 1e-10): number {
    if (n < 0) {
      throw new Error("Квадратный корень определен только для неотрицательных чисел");
    }
    
    if (n === 0 || n === 1) {
      return n;
    }
    
    function sqrtHelper(guess: number): number {
      const newGuess = (guess + n / guess) / 2;
      if (Math.abs(newGuess - guess) < precision) {
        return newGuess;
      }
      return sqrtHelper(newGuess);
    }
    
    return sqrtHelper(n);
  }
}

// Тестирование оптимизированных математических функций
console.log("Факториал 10:", MathOptimizer.factorial(10)); // 3628800

const fibCalculator = MathOptimizer.createFibonacciCalculator();
console.time("Фибоначчи 50");
console.log("Фибоначчи 50:", fibCalculator.calculate(50)); // 12586269025
console.timeEnd("Фибоначчи 50");
console.log("Размер кэша:", fibCalculator.getCacheSize());

console.log("НОД(48, 18):", MathOptimizer.gcd(48, 18)); // 6
console.log("2^10:", MathOptimizer.power(2, 10)); // 1024
console.log("Квадратный корень 16:", MathOptimizer.sqrt(16)); // 4
```

### Обработка больших структур данных

```typescript
// Обработка больших структур данных с использованием оптимизированной рекурсии
class LargeDataProcessor {
  // Обработка больших массивов с использованием хвостовой рекурсии
  static processLargeArray<T, R>(
    array: T[],
    processor: (item: T, index: number) => R,
    combiner: (acc: R, current: R, index: number) => R,
    initialValue: R
  ): R {
    function processHelper(index: number, accumulator: R): R {
      if (index >= array.length) {
        return accumulator;
      }
      
      const processedItem = processor(array[index], index);
      const newAccumulator = combiner(accumulator, processedItem, index);
      
      return processHelper(index + 1, newAccumulator);
    }
    
    return processHelper(0, initialValue);
  }
  
  // Параллельная обработка больших массивов
  static async processLargeArrayParallel<T, R>(
    array: T[],
    processor: (item: T) => R | Promise<R>,
    chunkSize: number = 1000
  ): Promise<R[]> {
    const chunks: T[][] = [];
    
    // Разбиваем массив на чанки
    for (let i = 0; i < array.length; i += chunkSize) {
      chunks.push(array.slice(i, i + chunkSize));
    }
    
    // Обрабатываем чанки параллельно
    const chunkPromises = chunks.map(chunk => 
      Promise.all(chunk.map(item => Promise.resolve(processor(item))))
    );
    
    const results = await Promise.all(chunkPromises);
    return results.flat();
  }
  
  // Обработка с использованием trampoline для очень больших массивов
  static processHugeArray<T, R>(
    array: T[],
    processor: (item: T) => R,
    combiner: (acc: R, current: R) => R,
    initialValue: R
  ): R {
    type ArrayTrampoline = 
      | { type: 'value'; value: R }
      | { type: 'continuation'; continuation: () => ArrayTrampoline };
    
    function done(value: R): ArrayTrampoline {
      return { type: 'value', value };
    }
    
    function next(continuation: () => ArrayTrampoline): ArrayTrampoline {
      return { type: 'continuation', continuation };
    }
    
    function trampoline(t: ArrayTrampoline): R {
      let current = t;
      while (current.type === 'continuation') {
        current = current.continuation();
      }
      return current.value;
    }
    
    function processHelper(index: number, accumulator: R): ArrayTrampoline {
      if (index >= array.length) {
        return done(accumulator);
      }
      
      return next(() => {
        const processedItem = processor(array[index]);
        const newAccumulator = combiner(accumulator, processedItem);
        return processHelper(index + 1, newAccumulator);
      });
    }
    
    return trampoline(processHelper(0, initialValue));
  }
}

// Тестирование обработки больших структур данных
const largeArray = Array.from({ length: 100000 }, (_, i) => i);

console.time("Обработка большого массива (хвостовая рекурсия)");
const sumResult = LargeDataProcessor.processLargeArray(
  largeArray,
  x => x,
  (acc, current) => acc + current,
  0
);
console.timeEnd("Обработка большого массива (хвостовая рекурсия)");
console.log("Сумма:", sumResult);

console.time("Обработка большого массива (trampoline)");
const trampolineSum = LargeDataProcessor.processHugeArray(
  largeArray,
  x => x,
  (acc, current) => acc + current,
  0
);
console.timeEnd("Обработка большого массива (trampoline)");
console.log("Сумма (trampoline):", trampolineSum);

// Параллельная обработка
async function testParallelProcessing() {
  console.time("Параллельная обработка");
  const squared = await LargeDataProcessor.processLargeArrayParallel(
    largeArray.slice(0, 10000), // Меньший массив для демонстрации
    x => x * x
  );
  console.timeEnd("Параллельная обработка");
  console.log("Первые 10 квадратов:", squared.slice(0, 10));
}

// testParallelProcessing();
```

Оптимизация рекурсии является важным аспектом функционального программирования, позволяющим создавать эффективные и масштабируемые приложения. Использование таких техник, как хвостовая рекурсия, trampoline, CPS и мемоизация, помогает разработчикам избежать проблем с переполнением стека и улучшить производительность рекурсивных алгоритмов.