# Оптимизация производительности в TypeScript

Оптимизация производительности в TypeScript включает в себя как компиляционные аспекты, так и runtime оптимизации. TypeScript предоставляет мощные возможности для создания производительного кода, которые можно использовать на этапе разработки и в продакшене.

## Компиляционные оптимизации

### 1. Настройки компилятора для производительности

```json
{
  "compilerOptions": {
    // Основные настройки для оптимизации
    "target": "ES2020",
    "module": "ESNext",
    
    // Производительность компиляции
    "incremental": true,
    "composite": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo",
    
    // Оптимизация вывода
    "removeComments": true,
    "importHelpers": true,
    
    // Проверки для оптимизации
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    
    // Оптимизация для продакшена
    "declaration": true,
    "declarationMap": false,
    "sourceMap": false,
    "inlineSources": false
  },
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

### 2. Использование tslib для оптимизации

```typescript
// В tsconfig.json
// {
//   "compilerOptions": {
//     "importHelpers": true,
//     "noEmitHelpers": true
//   }
// }

// TypeScript будет использовать tslib для общих вспомогательных функций
import { __assign, __extends, __awaiter } from "tslib";

// Вместо генерации вспомогательных функций inline
// TypeScript будет использовать импорты из tslib
```

## Runtime оптимизации

### 1. Оптимизация структур данных

```typescript
// Использование Map вместо объектов для частых операций
class OptimizedCache<T> {
  private cache: Map<string, T> = new Map();
  
  get(key: string): T | undefined {
    return this.cache.get(key);
  }
  
  set(key: string, value: T): void {
    this.cache.set(key, value);
  }
  
  has(key: string): boolean {
    return this.cache.has(key);
  }
  
  delete(key: string): boolean {
    return this.cache.delete(key);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Оптимизированный Set для проверки принадлежности
class FastLookup {
  private lookup: Set<string> = new Set();
  
  add(item: string): this {
    this.lookup.add(item);
    return this;
  }
  
  has(item: string): boolean {
    return this.lookup.has(item);
  }
  
  // Быстрая проверка нескольких элементов
  hasAll(items: string[]): boolean {
    return items.every(item => this.lookup.has(item));
  }
  
  hasAny(items: string[]): boolean {
    return items.some(item => this.lookup.has(item));
  }
}

// Использование ArrayBuffer для бинарных данных
function createOptimizedBuffer(size: number): ArrayBuffer {
  return new ArrayBuffer(size);
}

function processBinaryData(data: Uint8Array): Uint8Array {
  // Оптимизированная обработка бинарных данных
  const result = new Uint8Array(data.length);
  
  for (let i = 0; i < data.length; i++) {
    result[i] = data[i] ^ 0xFF; // XOR операция
  }
  
  return result;
}
```

### 2. Оптимизация алгоритмов и структур

```typescript
// Оптимизированный поиск с бинарным поиском
class SortedArray<T> {
  private items: T[] = [];
  
  constructor(private compare: (a: T, b: T) => number) {}
  
  add(item: T): void {
    const index = this.binarySearch(item);
    this.items.splice(index, 0, item);
  }
  
  has(item: T): boolean {
    const index = this.binarySearch(item);
    return index < this.items.length && this.compare(this.items[index], item) === 0;
  }
  
  private binarySearch(item: T): number {
    let left = 0;
    let right = this.items.length;
    
    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      const comparison = this.compare(this.items[mid], item);
      
      if (comparison < 0) {
        left = mid + 1;
      } else {
        right = mid;
      }
    }
    
    return left;
  }
  
  getItems(): readonly T[] {
    return this.items as readonly T[];
  }
}

// Использование
const sortedNumbers = new SortedArray<number>((a, b) => a - b);
[5, 2, 8, 1, 9].forEach(num => sortedNumbers.add(num));
console.log(sortedNumbers.getItems()); // [1, 2, 5, 8, 9]

// Оптимизированный пул объектов
class ObjectPool<T> {
  private available: T[] = [];
  private allObjects: T[] = [];
  
  constructor(
    private factory: () => T,
    private reset: (obj: T) => void,
    private initialSize: number = 10
  ) {
    this.initializePool();
  }
  
  private initializePool(): void {
    for (let i = 0; i < this.initialSize; i++) {
      const obj = this.factory();
      this.available.push(obj);
      this.allObjects.push(obj);
    }
  }
  
  acquire(): T {
    if (this.available.length === 0) {
      // Создаем новый объект если пул пуст
      const obj = this.factory();
      this.allObjects.push(obj);
      return obj;
    }
    return this.available.pop()!;
  }
  
  release(obj: T): void {
    this.reset(obj);
    this.available.push(obj);
  }
  
  size(): number {
    return this.allObjects.length;
  }
  
  availableCount(): number {
    return this.available.length;
  }
}

// Использование пула
interface Particle {
  x: number;
  y: number;
  active: boolean;
}

const particlePool = new ObjectPool<Particle>(
  () => ({ x: 0, y: 0, active: false }),
  (particle) => {
    particle.x = 0;
    particle.y = 0;
    particle.active = false;
  }
);

// Быстрое создание и повторное использование частиц
const particle1 = particlePool.acquire();
particle1.x = 10;
particle1.y = 20;
particle1.active = true;

// Возврат в пул
particlePool.release(particle1);
```

### 3. Оптимизация функций и вызовов

```typescript
// Оптимизированная мемоизация с кэшем LRU
class LRUCache<K, V> {
  private cache: Map<K, V> = new Map();
  private capacity: number;
  
  constructor(capacity: number) {
    this.capacity = capacity;
  }
  
  get(key: K): V | undefined {
    const value = this.cache.get(key);
    if (value === undefined) {
      return undefined;
    }
    
    // Перемещаем элемент в конец (самый недавний)
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  set(key: K, value: V): void {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // Удаляем самый старый элемент
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, value);
  }
  
  has(key: K): boolean {
    return this.cache.has(key);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Мемоизированная функция
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new LRUCache<string, any>(100);
  
  return function (...args: Parameters<T>): ReturnType<T> {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  } as T;
}

// Оптимизированная версия для функций с одним аргументом
function memoizeSingle<T, R>(fn: (arg: T) => R): (arg: T) => R {
  const cache = new Map<T, R>();
  
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
const expensiveCalculation = memoizeSingle((n: number) => {
  console.log(`Вычисляем для ${n}...`);
  return n * n;
});

console.log(expensiveCalculation(5)); // "Вычисляем для 5...", 25
console.log(expensiveCalculation(5)); // 25 (из кэша)
```

## Оптимизация асинхронных операций

### 1. Управление конкурентностью

```typescript
// Оптимизированный пул задач
class TaskPool {
  private running: number = 0;
  private queue: Array<() => Promise<any>> = [];
  private concurrency: number;
  
  constructor(concurrency: number = 4) {
    this.concurrency = concurrency;
  }
  
  async add<T>(task: () => Promise<T>): Promise<T> {
    return new Promise<T>((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await task();
          resolve(result);
        } catch (error) {
          reject(error);
        } finally {
          this.running--;
          this.processNext();
        }
      });
      
      this.processNext();
    });
  }
  
  private processNext(): void {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const task = this.queue.shift()!;
    task();
  }
  
  async waitForAll(): Promise<void> {
    while (this.running > 0 || this.queue.length > 0) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
  }
}

// Использование
const taskPool = new TaskPool(3); // Максимум 3 параллельные задачи

async function example() {
  const promises = Array.from({ length: 10 }, (_, i) => 
    taskPool.add(async () => {
      await new Promise(resolve => setTimeout(resolve, 1000));
      return `Результат ${i}`;
    })
  );
  
  const results = await Promise.all(promises);
  console.log(results);
}
```

### 2. Оптимизация сетевых запросов

```typescript
// Кэширование HTTP запросов
class HttpCache {
  private cache: Map<string, { data: any; timestamp: number; ttl: number }> = new Map();
  
  async get<T>(
    url: string, 
    ttl: number = 5 * 60 * 1000 // 5 минут по умолчанию
  ): Promise<T> {
    const cached = this.cache.get(url);
    
    if (cached && Date.now() - cached.timestamp < cached.ttl) {
      return cached.data;
    }
    
    const response = await fetch(url);
    const data = await response.json();
    
    this.cache.set(url, { data, timestamp: Date.now(), ttl });
    return data;
  }
  
  invalidate(url: string): void {
    this.cache.delete(url);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Батчинг запросов
class RequestBatcher {
  private pendingRequests: Map<string, Array<(data: any) => void>> = new Map();
  private batchTimeout: NodeJS.Timeout | null = null;
  private batchUrl: string;
  
  constructor(batchUrl: string) {
    this.batchUrl = batchUrl;
  }
  
  async request<T>(endpoint: string): Promise<T> {
    return new Promise<T>((resolve) => {
      if (!this.pendingRequests.has(endpoint)) {
        this.pendingRequests.set(endpoint, []);
      }
      
      this.pendingRequests.get(endpoint)!.push(resolve);
      
      if (this.batchTimeout) {
        clearTimeout(this.batchTimeout);
      }
      
      this.batchTimeout = setTimeout(() => this.sendBatch(), 10); // 10ms delay
    });
  }
  
  private async sendBatch(): Promise<void> {
    const endpoints = Array.from(this.pendingRequests.keys());
    if (endpoints.length === 0) return;
    
    try {
      // Отправляем батч запросов
      const response = await fetch(this.batchUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ endpoints })
      });
      
      const results = await response.json();
      
      // Раздаем результаты
      endpoints.forEach(endpoint => {
        const resolvers = this.pendingRequests.get(endpoint) || [];
        const result = results[endpoint];
        
        resolvers.forEach(resolve => resolve(result));
      });
    } catch (error) {
      // Обработка ошибок
      endpoints.forEach(endpoint => {
        const resolvers = this.pendingRequests.get(endpoint) || [];
        resolvers.forEach(resolve => resolve(null));
      });
    } finally {
      this.pendingRequests.clear();
    }
  }
}
```

## Оптимизация веб-приложений

### 1. Оптимизация React компонентов

```typescript
import React, { memo, useMemo, useCallback, useState } from 'react';

// Мемоизированный компонент
interface ExpensiveComponentProps {
  data: any[];
  filter: string;
  onItemSelect: (item: any) => void;
}

const ExpensiveComponent: React.FC<ExpensiveComponentProps> = memo(
  ({ data, filter, onItemSelect }) => {
    // Мемоизация отфильтрованных данных
    const filteredData = useMemo(() => {
      return data.filter(item => 
        item.name.toLowerCase().includes(filter.toLowerCase())
      );
    }, [data, filter]);
    
    // Мемоизация обработчика
    const handleSelect = useCallback((item: any) => {
      onItemSelect(item);
    }, [onItemSelect]);
    
    return (
      <div>
        {filteredData.map(item => (
          <ItemComponent 
            key={item.id} 
            item={item} 
            onSelect={handleSelect} 
          />
        ))}
      </div>
    );
  }
);

// Чистый компонент для отдельного элемента
interface ItemComponentProps {
  item: { id: number; name: string; value: number };
  onSelect: (item: any) => void;
}

const ItemComponent: React.FC<ItemComponentProps> = memo(({ item, onSelect }) => {
  const handleClick = useCallback(() => {
    onSelect(item);
  }, [item, onSelect]);
  
  return (
    <div onClick={handleClick}>
      <span>{item.name}</span>
      <span>{item.value}</span>
    </div>
  );
});

// Оптимизация состояния с использованием функций обновления
const OptimizedCounter: React.FC = () => {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState<any[]>([]);
  
  // Используем функциональное обновление для избежания захвата устаревших значений
  const increment = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);
  
  const addItem = useCallback((newItem: any) => {
    setItems(prevItems => [...prevItems, newItem]);
  }, []);
  
  // Используем useMemo для вычислений
  const processedItems = useMemo(() => {
    return items.map(item => ({
      ...item,
      processed: true
    }));
  }, [items]);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={increment}>Увеличить</button>
      <div>
        {processedItems.map((item, index) => (
          <div key={index}>{item.name}</div>
        ))}
      </div>
    </div>
  );
};
```

### 2. Оптимизация алгоритмов поиска

```typescript
// Оптимизированный индекс для поиска
class SearchIndex<T> {
  private index: Map<string, T[]> = new Map();
  private items: T[] = [];
  private idGetter: (item: T) => string;
  private searchableFields: ((item: T) => string)[];
  
  constructor(
    idGetter: (item: T) => string,
    searchableFields: ((item: T) => string)[]
  ) {
    this.idGetter = idGetter;
    this.searchableFields = searchableFields;
  }
  
  addItem(item: T): void {
    this.items.push(item);
    this.updateIndexForItem(item);
  }
  
  addItems(items: T[]): void {
    items.forEach(item => this.addItem(item));
  }
  
  search(query: string): T[] {
    if (!query) return this.items;
    
    const queryLower = query.toLowerCase();
    const results = new Set<T>();
    
    // Поиск по индексу
    for (const [term, indexedItems] of this.index.entries()) {
      if (term.includes(queryLower)) {
        indexedItems.forEach(item => results.add(item));
      }
    }
    
    return Array.from(results);
  }
  
  private updateIndexForItem(item: T): void {
    const id = this.idGetter(item);
    
    // Удаляем старые индексные записи для этого элемента
    for (const [term, items] of this.index.entries()) {
      const index = items.findIndex(i => this.idGetter(i) === id);
      if (index !== -1) {
        items.splice(index, 1);
        if (items.length === 0) {
          this.index.delete(term);
        }
      }
    }
    
    // Добавляем новые индексные записи
    this.searchableFields.forEach(fieldExtractor => {
      const fieldValue = fieldExtractor(item);
      const terms = this.tokenize(fieldValue.toLowerCase());
      
      terms.forEach(term => {
        if (!this.index.has(term)) {
          this.index.set(term, []);
        }
        this.index.get(term)!.push(item);
      });
    });
  }
  
  private tokenize(text: string): string[] {
    // Простой токенизатор
    return text
      .split(/\s+/)
      .filter(token => token.length > 0);
  }
}

// Использование
interface Product {
  id: string;
  name: string;
  description: string;
  category: string;
}

const productSearch = new SearchIndex<Product>(
  (product) => product.id,
  [
    (product) => product.name,
    (product) => product.description,
    (product) => product.category
  ]
);

const products: Product[] = [
  { id: '1', name: 'iPhone', description: 'Смартфон Apple', category: 'Electronics' },
  { id: '2', name: 'MacBook', description: 'Ноутбук Apple', category: 'Electronics' }
];

productSearch.addItems(products);
const results = productSearch.search('iPhone');
```

## Профилирование и измерение производительности

### 1. Инструменты профилирования

```typescript
// Простой профилировщик функций
class PerformanceProfiler {
  private timings: Map<string, { total: number; count: number; min: number; max: number }> = new Map();
  
  async measure<T>(name: string, fn: () => Promise<T>): Promise<T> {
    const start = performance.now();
    try {
      const result = await fn();
      const end = performance.now();
      this.recordTiming(name, end - start);
      return result;
    } catch (error) {
      const end = performance.now();
      this.recordTiming(name, end - start);
      throw error;
    }
  }
  
  syncMeasure<T>(name: string, fn: () => T): T {
    const start = performance.now();
    try {
      const result = fn();
      const end = performance.now();
      this.recordTiming(name, end - start);
      return result;
    } catch (error) {
      const end = performance.now();
      this.recordTiming(name, end - start);
      throw error;
    }
  }
  
  private recordTiming(name: string, duration: number): void {
    if (!this.timings.has(name)) {
      this.timings.set(name, { total: 0, count: 0, min: Infinity, max: -Infinity });
    }
    
    const timing = this.timings.get(name)!;
    timing.total += duration;
    timing.count += 1;
    timing.min = Math.min(timing.min, duration);
    timing.max = Math.max(timing.max, duration);
  }
  
  getReport(): string {
    let report = "Отчет о производительности:\n";
    for (const [name, timing] of this.timings.entries()) {
      const avg = timing.total / timing.count;
      report += `${name}: avg=${avg.toFixed(2)}ms, min=${timing.min.toFixed(2)}ms, max=${timing.max.toFixed(2)}ms, count=${timing.count}\n`;
    }
    return report;
  }
  
  reset(): void {
    this.timings.clear();
  }
}

// Использование
const profiler = new PerformanceProfiler();

async function example() {
  await profiler.measure('api-call', async () => {
    // Симуляция API вызова
    await new Promise(resolve => setTimeout(resolve, 100));
  });
  
  profiler.syncMeasure('heavy-calculation', () => {
    // Тяжелое вычисление
    let sum = 0;
    for (let i = 0; i < 1000000; i++) {
      sum += i;
    }
    return sum;
  });
  
  console.log(profiler.getReport());
}
```

### 2. Оптимизация памяти

```typescript
// Класс для отслеживания утечек памяти
class MemoryOptimizer {
  private trackedObjects: WeakSet<object> = new WeakSet();
  private objectCounts: Map<string, number> = new Map();
  
  trackObject<T extends object>(obj: T, type: string): T {
    this.trackedObjects.add(obj);
    
    const count = this.objectCounts.get(type) || 0;
    this.objectCounts.set(type, count + 1);
    
    return obj;
  }
  
  createDisposableObject<T extends object>(
    factory: () => T, 
    cleanup: (obj: T) => void,
    type: string
  ): T & { dispose: () => void } {
    const obj = this.trackObject(factory(), type);
    
    const disposableObj = Object.assign(obj, {
      dispose: () => {
        cleanup(obj);
        const count = this.objectCounts.get(type) || 0;
        this.objectCounts.set(type, Math.max(0, count - 1));
      }
    });
    
    return disposableObj;
  }
  
  getMemoryReport(): string {
    let report = "Отчет об использовании памяти:\n";
    for (const [type, count] of this.objectCounts.entries()) {
      report += `${type}: ${count} объектов\n`;
    }
    return report;
  }
}

// Использование
const memoryOptimizer = new MemoryOptimizer();

class ResourceManager {
  private resources: Array<{ id: string; data: any }> = [];
  
  createResource(id: string, data: any) {
    const resource = { id, data };
    this.resources.push(resource);
    
    return memoryOptimizer.createDisposableObject(
      () => resource,
      (res) => {
        // Очистка ресурса
        const index = this.resources.findIndex(r => r.id === res.id);
        if (index !== -1) {
          this.resources.splice(index, 1);
        }
      },
      'Resource'
    );
  }
}
```

## Практические примеры из реальной разработки

### 1. Оптимизация обработки данных

```typescript
// Оптимизированный обработчик данных с батчингом
class DataProcessor<T> {
  private buffer: T[] = [];
  private batchSize: number;
  private processCallback: (batch: T[]) => Promise<void>;
  private timer: NodeJS.Timeout | null = null;
  
  constructor(
    batchSize: number,
    processCallback: (batch: T[]) => Promise<void>
  ) {
    this.batchSize = batchSize;
    this.processCallback = processCallback;
  }
  
  async add(item: T): Promise<void> {
    this.buffer.push(item);
    
    if (this.buffer.length >= this.batchSize) {
      await this.processBatch();
    } else if (this.timer === null) {
      // Устанавливаем таймер для обработки оставшихся элементов
      this.timer = setTimeout(() => this.processBatch(), 1000);
    }
  }
  
  private async processBatch(): Promise<void> {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
    
    if (this.buffer.length === 0) return;
    
    const batch = this.buffer.splice(0, this.batchSize);
    await this.processCallback(batch);
  }
  
  async flush(): Promise<void> {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
    await this.processBatch();
  }
}

// Использование
const processor = new DataProcessor<{ id: number; data: string }>(
  100, // размер батча
  async (batch) => {
    // Обработка батча данных
    console.log(`Обработано ${batch.length} элементов`);
    // Здесь может быть отправка в базу данных, API и т.д.
  }
);

// Добавление данных
for (let i = 0; i < 250; i++) {
  await processor.add({ id: i, data: `data-${i}` });
}

await processor.flush(); // Обработка оставшихся элементов
```

### 2. Оптимизация валидации

```typescript
// Оптимизированный валидатор с кэшированием схем
class OptimizedValidator {
  private schemaCache: Map<string, any> = new Map();
  private compiledValidators: Map<string, (data: any) => boolean> = new Map();
  
  validate<T>(data: T, schema: any): { valid: boolean; errors?: string[] } {
    const schemaKey = this.getSchemaKey(schema);
    
    // Кэшируем скомпилированный валидатор
    if (!this.compiledValidators.has(schemaKey)) {
      const validator = this.compileValidator(schema);
      this.compiledValidators.set(schemaKey, validator);
    }
    
    const validator = this.compiledValidators.get(schemaKey)!;
    const valid = validator(data);
    
    if (valid) {
      return { valid: true };
    } else {
      // В реальном приложении здесь была бы генерация подробных ошибок
      return { valid: false, errors: ['Validation failed'] };
    }
  }
  
  private compileValidator(schema: any): (data: any) => boolean {
    // В реальном приложении использовалась бы библиотека вроде Ajv
    // или кастомный компилятор схем
    
    // Простой пример компиляции
    return (data: any): boolean => {
      if (schema.type === 'object') {
        for (const [key, valueSchema] of Object.entries(schema.properties || {})) {
          if (data[key] === undefined && schema.required?.includes(key)) {
            return false;
          }
          if (data[key] !== undefined) {
            // Рекурсивная валидация вложенных свойств
            if (typeof valueSchema === 'object' && valueSchema.type) {
              const nestedValidator = this.compileValidator(valueSchema);
              if (!nestedValidator(data[key])) {
                return false;
              }
            }
          }
        }
      }
      return true;
    };
  }
  
  private getSchemaKey(schema: any): string {
    // Создаем уникальный ключ для схемы
    return JSON.stringify(schema);
  }
}

// Использование
const validator = new OptimizedValidator();

const schema = {
  type: 'object',
  required: ['name', 'email'],
  properties: {
    name: { type: 'string', minLength: 2 },
    email: { type: 'string', format: 'email' },
    age: { type: 'number', minimum: 0, maximum: 120 }
  }
};

const result = validator.validate(
  { name: 'John', email: 'john@example.com', age: 30 },
  schema
);
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки оптимизации

```typescript
// ОШИБКА: Premature optimization
// const badExample = () => {
//   // Слишком сложная оптимизация для простой задачи
//   let result = 0;
//   for (let i = 0; i < 10; i++) {
//     result += i;
//   }
//   return result;
// };

// ЛУЧШЕ: Простое и понятное решение
const goodExample = () => {
  return [0, 1, 2, 3, 4, 5, 6, 7, 8, 9].reduce((sum, num) => sum + num, 0);
};

// ОШИБКА: Неправильное использование мемоизации
// const badMemoization = memoize((obj) => {
//   // Объекты каждый раз разные, кэш бесполезен
//   return obj.value * 2;
// });

// ЛУЧШЕ: Использование мемоизации с примитивными ключами
const goodMemoization = memoizeSingle((value: number) => {
  return value * 2;
});
```

### 2. Лучшие практики

```typescript
// 1. Измеряйте перед оптимизацией
const measurePerformance = async <T>(fn: () => Promise<T>): Promise<[T, number]> => {
  const start = performance.now();
  const result = await fn();
  const end = performance.now();
  return [result, end - start];
};

// 2. Используйте структурное сравнение для мемоизации
const shallowEqual = (objA: any, objB: any): boolean => {
  if (objA === objB) return true;
  
  if (!objA || !objB || typeof objA !== 'object' || typeof objB !== 'object') {
    return false;
  }
  
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);
  
  if (keysA.length !== keysB.length) return false;
  
  for (let i = 0; i < keysA.length; i++) {
    const key = keysA[i];
    if (objA[key] !== objB[key]) return false;
  }
  
  return true;
};

// 3. Оптимизируйте "горячие" пути кода
const hotPathFunction = (data: any[]) => {
  // Этот код вызывается часто, поэтому оптимизируем его
  let sum = 0;
  for (let i = 0; i < data.length; i++) {
    sum += data[i].value || 0;
  }
  return sum;
};

// 4. Используйте подходящие структуры данных
// Для частых вставок/удалений: LinkedList
// Для частых поисков: Set/Map
// Для сортировки: SortedArray
```

## Лучшие практики

1. **Измеряйте производительность** - используйте профилировщики для определения узких мест
2. **Оптимизируйте "горячие" пути** - места, которые вызываются часто
3. **Используйте подходящие структуры данных** - для конкретных операций
4. **Избегайте преждевременной оптимизации** - сначала напишите читаемый код
5. **Используйте мемоизацию осознанно** - только для дорогостоящих вычислений
6. **Рассмотрите батчинг операций** - для уменьшения накладных расходов

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как типизация может помочь в оптимизации
- [[ts/async-await/async-typescript-comprehensive|Async/Await]] - оптимизация асинхронных операций
- [[react/performance]] - оптимизация React приложений
- [[build-tools/performance]] - оптимизация сборки
- [[architecture/performance-patterns]] - архитектурные паттерны производительности