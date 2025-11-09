# Мемоизация

Мемоизация - это техника оптимизации, при которой результаты функций кэшируются для избежания повторных вычислений с теми же аргументами. В функциональном программировании мемоизация особенно эффективна, поскольку чистые функции всегда возвращают одинаковый результат для одинаковых входных данных.

## Содержание

1. [Основы мемоизации](#основы-мемоизации)
2. [Простая мемоизация](#простая-мемоизация)
3. [Мемоизация с различными типами аргументов](#мемоизация-с-различными-типами-аргументов)
4. [Продвинутая мемоизация](#продвинутая-мемоизация)
5. [LRU кэш](#lru-кэш)
6. [Мемоизация рекурсивных функций](#мемоизация-рекурсивных-функций)
7. [Практические примеры](#практические-примеры)

## Основы мемоизации

Мемоизация основана на принципе кэширования: если функция уже вызывалась с определенными аргументами, то результат можно взять из кэша вместо повторного вычисления.

### Преимущества мемоизации:

1. **Ускорение выполнения** - Избежание повторных вычислений
2. **Экономия ресурсов** - Снижение нагрузки на CPU и память
3. **Прозрачность** - Клиентский код не изменяется
4. **Предсказуемость** - Чистые функции идеально подходят для мемоизации

### Недостатки мемоизации:

1. **Память** - Кэш может занимать значительный объем памяти
2. **Накладные расходы** - Создание ключей и поиск в кэше
3. **Сложность управления** - Необходимость очистки кэша и управления его размером

```typescript
// Простая мемоизация для функций с одним аргументом
function memoize<T, R>(fn: (arg: T) => R): (arg: T) => R {
  const cache = new Map<T, R>();
  
  return function (arg: T): R {
    if (cache.has(arg)) {
      console.log(`Кэш: ${arg}`);
      return cache.get(arg)!;
    }
    
    console.log(`Вычисление: ${arg}`);
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

// Пример использования
const expensiveFunction = memoize((n: number): number => {
  // Симуляция сложных вычислений
  let result = 0;
  for (let i = 0; i < n * 1000000; i++) {
    result += Math.sin(i);
  }
  return result;
});

console.log(expensiveFunction(5)); // Вычисление выполняется
console.log(expensiveFunction(5)); // Результат берется из кэша
console.log(expensiveFunction(10)); // Вычисление выполняется
console.log(expensiveFunction(10)); // Результат берется из кэша
```

### Мемоизация с отслеживанием статистики

```typescript
// Мемоизация с отслеживанием статистики
interface MemoizationStats {
  hits: number;
  misses: number;
  cacheSize: number;
}

function memoizeWithStats<T, R>(fn: (arg: T) => R): [(arg: T) => R, () => MemoizationStats] {
  const cache = new Map<T, R>();
  let hits = 0;
  let misses = 0;
  
  const memoizedFn = function (arg: T): R {
    if (cache.has(arg)) {
      hits++;
      return cache.get(arg)!;
    }
    
    misses++;
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
  
  const getStats = (): MemoizationStats => ({
    hits,
    misses,
    cacheSize: cache.size
  });
  
  return [memoizedFn, getStats];
}

// Пример использования
const [memoizedFn, getStats] = memoizeWithStats((n: number): number => {
  return n * n;
});

// Вызовы функции
memoizedFn(5);
memoizedFn(3);
memoizedFn(5); // Кэш
memoizedFn(7);
memoizedFn(3); // Кэш

console.log("Статистика:", getStats());
// Статистика: { hits: 2, misses: 3, cacheSize: 3 }
```

## Простая мемоизация

Простая мемоизация подходит для функций с примитивными аргументами.

```typescript
// Мемоизация для функций с примитивными аргументами
function simpleMemoize<T extends string | number | boolean, R>(
  fn: (arg: T) => R
): (arg: T) => R {
  const cache: Record<string, R> = {};
  
  return function (arg: T): R {
    const key = String(arg);
    
    if (key in cache) {
      return cache[key];
    }
    
    const result = fn(arg);
    cache[key] = result;
    return result;
  };
}

// Пример использования
const square = simpleMemoize((n: number): number => {
  console.log(`Вычисление квадрата ${n}`);
  return n * n;
});

console.log(square(4)); // Вычисление квадрата 4
console.log(square(4)); // Из кэша
console.log(square(5)); // Вычисление квадрата 5
```

### Мемоизация для функций без аргументов

```typescript
// Мемоизация для функций без аргументов
function memoizeNoArgs<T>(fn: () => T): () => T {
  let cachedResult: T | undefined;
  let hasCached = false;
  
  return function (): T {
    if (hasCached) {
      return cachedResult!;
    }
    
    cachedResult = fn();
    hasCached = true;
    return cachedResult;
  };
}

// Пример использования
const getCurrentTime = memoizeNoArgs(() => {
  console.log("Получение текущего времени");
  return new Date().toISOString();
});

console.log(getCurrentTime()); // Получение текущего времени
console.log(getCurrentTime()); // Из кэша
console.log(getCurrentTime()); // Из кэша
```

## Мемоизация с различными типами аргументов

Для функций с несколькими аргументами или сложными типами данных требуется более сложная стратегия создания ключей.

```typescript
// Мемоизация для функций с несколькими аргументами
function memoizeMulti<T extends any[], R>(fn: (...args: T) => R): (...args: T) => R {
  const cache = new Map<string, R>();
  
  return function (...args: T): R {
    // Создаем ключ из аргументов
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Пример использования
const add = memoizeMulti((a: number, b: number): number => {
  console.log(`Сложение ${a} + ${b}`);
  return a + b;
});

console.log(add(2, 3)); // Сложение 2 + 3
console.log(add(2, 3)); // Из кэша
console.log(add(5, 7)); // Сложение 5 + 7

// Мемоизация для функций с объектными аргументами
const calculateArea = memoizeMulti((rect: { width: number; height: number }): number => {
  console.log(`Вычисление площади: ${rect.width} x ${rect.height}`);
  return rect.width * rect.height;
});

console.log(calculateArea({ width: 10, height: 5 })); // Вычисление площади
console.log(calculateArea({ width: 10, height: 5 })); // Из кэша
console.log(calculateArea({ width: 8, height: 6 }));  // Вычисление площади
```

### Мемоизация с пользовательской функцией хеширования

```typescript
// Мемоизация с пользовательской функцией хеширования
function memoizeWithHash<T extends any[], R>(
  fn: (...args: T) => R,
  hashFn: (...args: T) => string
): (...args: T) => R {
  const cache = new Map<string, R>();
  
  return function (...args: T): R {
    const key = hashFn(...args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Пример пользовательской функции хеширования
function hashObject(obj: any): string {
  return Object.entries(obj)
    .sort(([a], [b]) => a.localeCompare(b))
    .map(([key, value]) => `${key}:${value}`)
    .join('|');
}

// Пример использования
const processUser = memoizeWithHash(
  (user: { id: number; name: string; age: number }): string => {
    console.log(`Обработка пользователя ${user.name}`);
    return `${user.name} (${user.age} лет)`;
  },
  (user) => hashObject(user)
);

console.log(processUser({ id: 1, name: "Иван", age: 30 })); // Обработка пользователя Иван
console.log(processUser({ id: 1, name: "Иван", age: 30 })); // Из кэша
console.log(processUser({ name: "Иван", age: 30, id: 1 })); // Из кэша (порядок свойств не важен)
```

## Продвинутая мемоизация

Продвинутая мемоизация включает в себя управление размером кэша, автоматическую очистку и другие оптимизации.

```typescript
// Интерфейс для продвинутой мемоизации
interface AdvancedMemoizeOptions {
  maxSize?: number;
  ttl?: number; // Time to live в миллисекундах
  onCacheHit?: (key: string) => void;
  onCacheMiss?: (key: string) => void;
}

// Кэш с метаинформацией
interface CacheEntry<T> {
  value: T;
  timestamp: number;
}

// Продвинутая мемоизация
function advancedMemoize<T extends any[], R>(
  fn: (...args: T) => R,
  options: AdvancedMemoizeOptions = {}
): (...args: T) => R {
  const cache = new Map<string, CacheEntry<R>>();
  const { maxSize = Infinity, ttl, onCacheHit, onCacheMiss } = options;
  
  return function (...args: T): R {
    const key = JSON.stringify(args);
    const now = Date.now();
    
    // Проверяем TTL
    if (ttl && cache.has(key)) {
      const entry = cache.get(key)!;
      if (now - entry.timestamp > ttl) {
        cache.delete(key);
      }
    }
    
    // Проверяем кэш
    if (cache.has(key)) {
      onCacheHit?.(key);
      return cache.get(key)!.value;
    }
    
    onCacheMiss?.(key);
    
    // Выполняем функцию
    const result = fn(...args);
    
    // Управление размером кэша
    if (cache.size >= maxSize) {
      const firstKey = cache.keys().next().value;
      if (firstKey) {
        cache.delete(firstKey);
      }
    }
    
    // Сохраняем в кэш
    cache.set(key, { value: result, timestamp: now });
    
    return result;
  };
}

// Пример использования
const expensiveCalculation = advancedMemoize(
  (a: number, b: number): number => {
    console.log(`Вычисление для ${a}, ${b}`);
    // Симуляция сложных вычислений
    return a * b + Math.sqrt(a + b);
  },
  {
    maxSize: 100,
    ttl: 5000, // 5 секунд
    onCacheHit: (key) => console.log(`Кэш hit: ${key}`),
    onCacheMiss: (key) => console.log(`Кэш miss: ${key}`)
  }
);

console.log(expensiveCalculation(5, 3));
console.log(expensiveCalculation(5, 3)); // Из кэша
console.log(expensiveCalculation(10, 7));

// Ждем истечения TTL
setTimeout(() => {
  console.log(expensiveCalculation(5, 3)); // Повторное вычисление из-за TTL
}, 6000);
```

### Мемоизация с WeakMap для объектов

```typescript
// Мемоизация с WeakMap для объектных аргументов
function weakMemoize<T extends object, R>(fn: (arg: T) => R): (arg: T) => R {
  const cache = new WeakMap<T, R>();
  
  return function (arg: T): R {
    if (cache.has(arg)) {
      return cache.get(arg)!;
    }
    
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

// Пример использования
class DataProcessor {
  process(data: { values: number[] }): number {
    console.log("Обработка данных");
    return data.values.reduce((sum, val) => sum + val, 0);
  }
}

const processor = new DataProcessor();
const memoizedProcess = weakMemoize(processor.process.bind(processor));

const data1 = { values: [1, 2, 3, 4, 5] };
const data2 = { values: [10, 20, 30] };

console.log(memoizedProcess(data1)); // Обработка данных
console.log(memoizedProcess(data1)); // Из кэша
console.log(memoizedProcess(data2)); // Обработка данных

// Объекты могут быть собраны сборщиком мусора, если на них нет других ссылок
```

## LRU кэш

LRU (Least Recently Used) кэш автоматически удаляет наименее недавно использованные элементы при достижении максимального размера.

```typescript
// LRU (Least Recently Used) кэш
class LRUCache<K, V> {
  private cache = new Map<K, V>();
  private readonly maxSize: number;
  
  constructor(maxSize: number) {
    this.maxSize = maxSize;
  }
  
  get(key: K): V | undefined {
    const item = this.cache.get(key);
    if (item) {
      // Перемещаем элемент в конец (самый свежий)
      this.cache.delete(key);
      this.cache.set(key, item);
    }
    return item;
  }
  
  set(key: K, value: V): void {
    // Если ключ уже существует, удаляем его
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    // Если кэш переполнен, удаляем самый старый элемент
    else if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      if (firstKey !== undefined) {
        this.cache.delete(firstKey);
      }
    }
    
    this.cache.set(key, value);
  }
  
  has(key: K): boolean {
    return this.cache.has(key);
  }
  
  delete(key: K): boolean {
    return this.cache.delete(key);
  }
  
  size(): number {
    return this.cache.size;
  }
  
  clear(): void {
    this.cache.clear();
  }
  
  // Получение всех ключей в порядке использования
  keys(): K[] {
    return Array.from(this.cache.keys());
  }
}

// Мемоизация с LRU кэшем
function memoizeLRU<T extends any[], R>(fn: (...args: T) => R, maxSize: number): (...args: T) => R {
  const cache = new LRUCache<string, R>(maxSize);
  
  return function (...args: T): R {
    const key = JSON.stringify(args);
    const cached = cache.get(key);
    
    if (cached !== undefined) {
      return cached;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Пример использования
const fibonacci = memoizeLRU((n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}, 100); // Максимум 100 элементов в кэше

console.log(fibonacci(10)); // Быстро вычисляется благодаря мемоизации
console.log(fibonacci(15)); // Использует кэшированные значения для меньших чисел

// Проверка размера кэша
console.log("Размер кэша:", (fibonacci as any).cache?.size());
```

### Расширенный LRU кэш с TTL

```typescript
// Расширенный LRU кэш с TTL
interface ExtendedCacheEntry<V> {
  value: V;
  timestamp: number;
  lastAccess: number;
}

class ExtendedLRUCache<K, V> {
  private cache = new Map<K, ExtendedCacheEntry<V>>();
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
function advancedMemoizeLRU<T extends any[], R>(
  fn: (...args: T) => R,
  maxSize: number,
  ttl: number
): (...args: T) => R {
  const cache = new ExtendedLRUCache<string, R>(maxSize, ttl);
  
  return function (...args: T): R {
    const key = JSON.stringify(args);
    const cached = cache.get(key);
    
    if (cached !== undefined) {
      return cached;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

## Мемоизация рекурсивных функций

Мемоизация рекурсивных функций требует особого подхода, так как функция должна ссылаться на свою мемоизированную версию.

```typescript
// Мемоизация рекурсивных функций
function memoizeRecursive<T extends any[], R>(
  fn: (...args: T) => R,
  keyFn: (...args: T) => string = (...args) => JSON.stringify(args)
): (...args: T) => R {
  const cache = new Map<string, R>();
  let memoizedFn: ((...args: T) => R) | null = null;
  
  const wrapper = function (...args: T): R {
    const key = keyFn(...args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn.apply(null, args);
    cache.set(key, result);
    return result;
  };
  
  memoizedFn = wrapper;
  return wrapper;
}

// Пример: мемоизированная рекурсивная функция Фибоначчи
const fibonacci = memoizeRecursive((n: number): number => {
  console.log(`Вычисление fibonacci(${n})`);
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10));
console.log(fibonacci(12)); // Использует кэшированные значения

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

console.log(isEven(5)); // false
console.log(isOdd(5));  // true
console.log(isEven(5)); // Из кэша
```

## Практические примеры

### Мемоизация в веб-приложениях

```typescript
// Мемоизация API вызовов
class APIMemoizer {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private readonly ttl: number;
  
  constructor(ttl: number = 5 * 60 * 1000) { // 5 минут по умолчанию
    this.ttl = ttl;
  }
  
  async fetch<T>(url: string, options?: RequestInit): Promise<T> {
    const key = `${url}_${JSON.stringify(options)}`;
    const now = Date.now();
    
    // Проверяем кэш
    const cached = this.cache.get(key);
    if (cached && now - cached.timestamp < this.ttl) {
      console.log(`API кэш hit: ${url}`);
      return cached.data;
    }
    
    console.log(`API вызов: ${url}`);
    const response = await fetch(url, options);
    const data = await response.json();
    
    // Сохраняем в кэш
    this.cache.set(key, { data, timestamp: now });
    
    return data;
  }
  
  clear(): void {
    this.cache.clear();
  }
  
  clearExpired(): void {
    const now = Date.now();
    for (const [key, entry] of this.cache.entries()) {
      if (now - entry.timestamp >= this.ttl) {
        this.cache.delete(key);
      }
    }
  }
}

// Пример использования
const apiMemoizer = new APIMemoizer(10000); // 10 секунд TTL

async function fetchUserData(userId: number) {
  return apiMemoizer.fetch(`/api/users/${userId}`);
}

// Вызовы API
fetchUserData(1).then(data => console.log("Пользователь 1:", data));
fetchUserData(1).then(data => console.log("Пользователь 1 (кэш):", data));
fetchUserData(2).then(data => console.log("Пользователь 2:", data));
```

### Мемоизация вычислений в React-компонентах

```typescript
// Мемоизация для React-компонентов
interface ReactMemoizeOptions {
  equalityFn?: (a: any, b: any) => boolean;
}

// Мемоизация props для компонентов
function memoizeComponent<P extends object>(
  Component: React.ComponentType<P>,
  options: ReactMemoizeOptions = {}
): React.ComponentType<P> {
  const { equalityFn = (a, b) => a === b } = options;
  
  return React.memo(Component, equalityFn);
}

// Мемоизация сложных вычислений в компонентах
function useMemoizedValue<T>(factory: () => T, deps: React.DependencyList): T {
  const cache = React.useRef<Map<string, T>>();
  const lastDeps = React.useRef<React.DependencyList>();
  const lastResult = React.useRef<T>();
  
  if (!cache.current) {
    cache.current = new Map();
  }
  
  const depsKey = JSON.stringify(deps);
  
  if (!lastDeps.current || !deps.every((dep, i) => dep === lastDeps.current![i])) {
    lastDeps.current = deps;
    
    if (cache.current.has(depsKey)) {
      lastResult.current = cache.current.get(depsKey)!;
    } else {
      lastResult.current = factory();
      cache.current.set(depsKey, lastResult.current);
    }
  }
  
  return lastResult.current!;
}

// Пример использования в React-компоненте
const ExpensiveComponent: React.FC<{ data: number[] }> = ({ data }) => {
  // Мемоизированное вычисление
  const processedData = useMemoizedValue(() => {
    console.log("Выполнение сложных вычислений");
    return data.map(x => x * x).filter(x => x > 10);
  }, [data]);
  
  return (
    <div>
      <h3>Обработанные данные:</h3>
      <ul>
        {processedData.map((value, index) => (
          <li key={index}>{value}</li>
        ))}
      </ul>
    </div>
  );
};

// Мемоизированный компонент
const MemoizedExpensiveComponent = memoizeComponent(ExpensiveComponent);
```

### Мемоизация в функциональных пайплайнах

```typescript
// Мемоизация для функциональных пайплайнов
class PipelineMemoizer {
  private cache = new Map<string, any>();
  
  memoize<T, R>(fn: (input: T) => R): (input: T) => R {
    return (input: T): R => {
      const key = `${fn.name}_${JSON.stringify(input)}`;
      
      if (this.cache.has(key)) {
        return this.cache.get(key);
      }
      
      const result = fn(input);
      this.cache.set(key, result);
      return result;
    };
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Пример функционального пайплайна с мемоизацией
const pipelineMemoizer = new PipelineMemoizer();

// Функции для пайплайна
const parseData = pipelineMemoizer.memoize((input: string): any[] => {
  console.log("Парсинг данных");
  return JSON.parse(input);
});

const processData = pipelineMemoizer.memoize((data: any[]): any[] => {
  console.log("Обработка данных");
  return data.map(item => ({ ...item, processed: true }));
});

const validateData = pipelineMemoizer.memoize((data: any[]): any[] => {
  console.log("Валидация данных");
  return data.filter(item => item.id && item.name);
});

// Композиция функций
const processPipeline = (input: string) => {
  return validateData(processData(parseData(input)));
};

// Использование пайплайна
const jsonData = '[{"id": 1, "name": "Item 1"}, {"id": 2, "name": "Item 2"}]';

console.log("Первый вызов:");
const result1 = processPipeline(jsonData);

console.log("Второй вызов (с кэшем):");
const result2 = processPipeline(jsonData);

console.log("Результаты идентичны:", result1 === result2);
```

Мемоизация является мощной техникой оптимизации в функциональном программировании. Правильное использование мемоизации позволяет значительно улучшить производительность приложений, особенно при работе с чистыми функциями и повторяющимися вычислениями. Однако важно учитывать использование памяти и правильно управлять размером кэша для достижения оптимальных результатов.