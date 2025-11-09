# Async/Await в TypeScript

Async/await - это синтаксический сахар для работы с промисами, который позволяет писать асинхронный код в синхронном стиле. В TypeScript async/await также обеспечивает строгую типизацию асинхронных операций, что делает код более безопасным и предсказуемым.

## Основы Async/Await

### 1. Простые примеры

```typescript
// Простая асинхронная функция
async function fetchUserData(): Promise<string> {
  // Симуляция асинхронной операции
  return new Promise(resolve => {
    setTimeout(() => {
      resolve("Данные пользователя");
    }, 1000);
  });
}

// Использование async/await
async function processUser() {
  try {
    const userData = await fetchUserData();
    console.log(userData); // "Данные пользователя"
    return userData;
  } catch (error) {
    console.error("Ошибка:", error);
  }
}
```

### 2. Возвращаемые типы

```typescript
// Асинхронная функция всегда возвращает Promise
async function getNumber(): Promise<number> {
  return 42;
}

// Даже если не указывать Promise возвращаемого типа
async function getString() {
  return "hello"; // Тип возвращаемого значения: Promise<string>
}

// Функция без возвращаемого значения
async function doSomething(): Promise<void> {
  console.log("Выполняю асинхронную операцию");
}

// Использование
const numPromise: Promise<number> = getNumber();
const strPromise: Promise<string> = getString();
const voidPromise: Promise<void> = doSomething();
```

## Обработка ошибок

### 1. Try/catch блоки

```typescript
interface User {
  id: number;
  name: string;
}

async function fetchUser(id: number): Promise<User> {
  if (id <= 0) {
    throw new Error("ID должен быть положительным числом");
  }
  
  // Симуляция запроса к API
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) {
        resolve({ id, name: `User ${id}` });
      } else {
        reject(new Error("Не удалось загрузить пользователя"));
      }
    }, 1000);
  });
}

async function handleUser(id: number) {
  try {
    const user = await fetchUser(id);
    console.log(`Пользователь: ${user.name}`);
    return user;
  } catch (error) {
    if (error instanceof Error) {
      console.error(`Ошибка при загрузке пользователя: ${error.message}`);
    } else {
      console.error("Неизвестная ошибка:", error);
    }
    // Можно вернуть значение по умолчанию или повторно выбросить ошибку
    throw error;
  }
}
```

### 2. Обработка ошибок в цепочках

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

async function safeFetchUser(id: number): Promise<ApiResponse<User>> {
  try {
    const user = await fetchUser(id);
    return { success: true, data: user };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : "Неизвестная ошибка"
    };
  }
}

async function processUserSafely(id: number) {
  const response = await safeFetchUser(id);
  
  if (response.success) {
    console.log("Пользователь:", response.data);
    return response.data;
  } else {
    console.error("Ошибка:", response.error);
    return null;
  }
}
```

## Параллельное выполнение

### 1. Promise.all

```typescript
async function fetchMultipleUsers(ids: number[]): Promise<User[]> {
  // Выполнение всех запросов параллельно
  const promises = ids.map(id => fetchUser(id));
  return Promise.all(promises);
}

async function processMultipleUsers() {
  try {
    const users = await fetchMultipleUsers([1, 2, 3, 4, 5]);
    console.log("Все пользователи:", users);
    return users;
  } catch (error) {
    console.error("Ошибка при загрузке одного из пользователей:", error);
    // Promise.all отклоняет весь промис, если хотя бы один из промисов отклонен
  }
}
```

### 2. Promise.allSettled

```typescript
async function fetchUsersWithStatus(ids: number[]): Promise<PromiseSettledResult<User>[]> {
  const promises = ids.map(id => fetchUser(id));
  return Promise.allSettled(promises);
}

async function processUsersWithStatus() {
  const results = await fetchUsersWithStatus([1, 2, 3, 4, 5]);
  
  const successful = results
    .filter(result => result.status === 'fulfilled')
    .map(result => (result as PromiseFulfilledResult<User>).value);
    
  const failed = results
    .filter(result => result.status === 'rejected')
    .map(result => (result as PromiseRejectedResult).reason);
  
  console.log(`Успешно загружено: ${successful.length} пользователей`);
  console.log(`Ошибка в: ${failed.length} запросах`);
  
  return { successful, failed };
}
```

### 3. Promise.race

```typescript
async function fetchWithTimeout<T>(promise: Promise<T>, timeoutMs: number): Promise<T> {
  const timeoutPromise = new Promise<never>((_, reject) => {
    setTimeout(() => reject(new Error('Таймаут')), timeoutMs);
  });
  
  return Promise.race([promise, timeoutPromise]);
}

async function fetchUserWithTimeout(id: number) {
  try {
    const user = await fetchWithTimeout(fetchUser(id), 2000); // 2 секунды таймаут
    console.log("Пользователь загружен:", user);
    return user;
  } catch (error) {
    console.error("Ошибка или таймаут:", error);
    throw error;
  }
}
```

## Продвинутые паттерны

### 1. Асинхронные генераторы

```typescript
// Асинхронный генератор
async function* asyncSequence(start: number, end: number): AsyncGenerator<number, void, unknown> {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, 100)); // Задержка 100ms
    yield i;
  }
}

async function useAsyncGenerator() {
  for await (const num of asyncSequence(1, 5)) {
    console.log(`Число: ${num}`);
  }
  // Выведет: 1, 2, 3, 4, 5 с задержками
}
```

### 2. Асинхронные итераторы

```typescript
interface AsyncIterableQueue<T> {
  [Symbol.asyncIterator](): AsyncIterator<T>;
  enqueue(item: T): void;
  dequeue(): Promise<T>;
}

class AsyncQueue<T> implements AsyncIterableQueue<T> {
  private items: T[] = [];
  private resolvers: ((value: T) => void)[] = [];
  
  async *[Symbol.asyncIterator](): AsyncIterator<T> {
    while (true) {
      if (this.items.length > 0) {
        yield this.items.shift()!;
      } else {
        const item = await this.dequeue();
        yield item;
      }
    }
  }
  
  enqueue(item: T) {
    if (this.resolvers.length > 0) {
      const resolve = this.resolvers.shift()!;
      resolve(item);
    } else {
      this.items.push(item);
    }
  }
  
  dequeue(): Promise<T> {
    return new Promise<T>(resolve => {
      if (this.items.length > 0) {
        resolve(this.items.shift()!);
      } else {
        this.resolvers.push(resolve);
      }
    });
  }
}
```

### 3. Повторение операций с экспоненциальной задержкой

```typescript
interface RetryOptions {
  maxRetries: number;
  baseDelay: number;
  backoffMultiplier: number;
}

async function retryAsync<T>(
  operation: () => Promise<T>,
  options: RetryOptions
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt <= options.maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt === options.maxRetries) {
        break;
      }
      
      // Экспоненциальная задержка
      const delay = options.baseDelay * Math.pow(options.backoffMultiplier, attempt);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError!;
}

// Использование
async function fetchWithRetry(url: string) {
  return retryAsync(
    () => fetch(url).then(res => res.json()),
    { maxRetries: 3, baseDelay: 1000, backoffMultiplier: 2 }
  );
}
```

## Практические применения

### 1. Работа с API

```typescript
interface ApiClientOptions {
  baseUrl: string;
  timeout?: number;
  retries?: number;
}

class ApiClient {
  private baseUrl: string;
  private timeout: number;
  private retries: number;
  
  constructor(options: ApiClientOptions) {
    this.baseUrl = options.baseUrl;
    this.timeout = options.timeout || 5000;
    this.retries = options.retries || 3;
  }
  
  async get<T>(endpoint: string): Promise<T> {
    return this.request<T>('GET', endpoint);
  }
  
  async post<T>(endpoint: string, data: any): Promise<T> {
    return this.request<T>('POST', endpoint, data);
  }
  
  async put<T>(endpoint: string, data: any): Promise<T> {
    return this.request<T>('PUT', endpoint, data);
  }
  
  async delete<T>(endpoint: string): Promise<T> {
    return this.request<T>('DELETE', endpoint);
  }
  
  private async request<T>(method: string, endpoint: string, data?: any): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    
    for (let attempt = 0; attempt <= this.retries; attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.timeout);
        
        const response = await fetch(url, {
          method,
          headers: {
            'Content-Type': 'application/json',
          },
          body: data ? JSON.stringify(data) : undefined,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return await response.json();
      } catch (error) {
        if (attempt === this.retries) {
          throw error;
        }
        
        // Задержка перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, 1000 * (attempt + 1)));
      }
    }
    
    throw new Error('Должно быть невозможно достичь этой точки');
  }
}

// Использование
async function useApiClient() {
  const api = new ApiClient({ 
    baseUrl: 'https://api.example.com',
    timeout: 10000,
    retries: 3
  });
  
  try {
    const users = await api.get<User[]>('/users');
    console.log('Пользователи:', users);
    
    const newUser = await api.post<User>('/users', {
      name: 'John Doe',
      email: 'john@example.com'
    });
    console.log('Новый пользователь:', newUser);
  } catch (error) {
    console.error('Ошибка API:', error);
  }
}
```

### 2. Управление состоянием с асинхронными операциями

```typescript
interface LoadingState<T> {
  isLoading: boolean;
  data: T | null;
  error: string | null;
}

class AsyncStateManager<T> {
  private state: LoadingState<T> = {
    isLoading: false,
    data: null,
    error: null
  };
  
  private listeners: Array<(state: LoadingState<T>) => void> = [];
  
  subscribe(listener: (state: LoadingState<T>) => void) {
    this.listeners.push(listener);
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  private notify() {
    this.listeners.forEach(listener => listener(this.state));
  }
  
  async execute(asyncOperation: () => Promise<T>): Promise<T> {
    this.state = { ...this.state, isLoading: true, error: null };
    this.notify();
    
    try {
      const data = await asyncOperation();
      this.state = { isLoading: false, data, error: null };
      this.notify();
      return data;
    } catch (error) {
      this.state = { 
        isLoading: false, 
        data: null, 
        error: error instanceof Error ? error.message : 'Неизвестная ошибка' 
      };
      this.notify();
      throw error;
    }
  }
  
  getState(): LoadingState<T> {
    return { ...this.state };
  }
}

// Использование
async function useAsyncStateManager() {
  const manager = new AsyncStateManager<User>();
  
  manager.subscribe(state => {
    console.log('Состояние изменилось:', state);
  });
  
  try {
    const user = await manager.execute(() => fetchUser(1));
    console.log('Пользователь загружен:', user);
  } catch (error) {
    console.error('Ошибка загрузки:', error);
  }
}
```

### 3. Пул соединений с базой данных

```typescript
interface DbConnection {
  id: number;
  connected: boolean;
  lastUsed: number;
}

class ConnectionPool {
  private connections: DbConnection[] = [];
  private available: number[] = [];
  private nextId: number = 1;
  
  constructor(private maxConnections: number = 10) {
    this.initializeConnections();
  }
  
  private initializeConnections() {
    for (let i = 0; i < this.maxConnections; i++) {
      this.connections.push({
        id: this.nextId++,
        connected: false,
        lastUsed: Date.now()
      });
      this.available.push(i);
    }
  }
  
  async acquire(): Promise<number> {
    while (this.available.length === 0) {
      // Ждем освобождения соединения
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    const index = this.available.shift()!;
    this.connections[index].connected = true;
    this.connections[index].lastUsed = Date.now();
    
    return this.connections[index].id;
  }
  
  release(connectionId: number) {
    const index = this.connections.findIndex(conn => conn.id === connectionId);
    if (index !== -1) {
      this.connections[index].connected = false;
      this.available.push(index);
    }
  }
  
  async executeQuery(query: string): Promise<any> {
    const connectionId = await this.acquire();
    try {
      // Симуляция выполнения запроса
      await new Promise(resolve => setTimeout(resolve, 100));
      return `Результат запроса: ${query}`;
    } finally {
      this.release(connectionId);
    }
  }
}

// Использование
async function useConnectionPool() {
  const pool = new ConnectionPool(5);
  
  const promises = Array.from({ length: 10 }, (_, i) => 
    pool.executeQuery(`SELECT * FROM users WHERE id = ${i}`)
  );
  
  const results = await Promise.all(promises);
  console.log('Результаты:', results);
}
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// Ошибка 1: Необработанные исключения в асинхронных функциях
async function badExample() {
  throw new Error("Это исключение не будет обработано");
}

// Решение: всегда обрабатывать асинхронные функции
async function goodExample() {
  try {
    await badExample();
  } catch (error) {
    console.error("Обработанное исключение:", error);
  }
}

// Ошибка 2: await внутри циклов без необходимости параллельности
async function sequentialFetch(ids: number[]) {
  const results = [];
  for (const id of ids) {
    // Это выполняется последовательно - медленно!
    const result = await fetchUser(id);
    results.push(result);
  }
  return results;
}

// Лучше: параллельное выполнение
async function parallelFetch(ids: number[]) {
  const promises = ids.map(id => fetchUser(id));
  return Promise.all(promises);
}
```

### 2. Лучшие практики

```typescript
// 1. Всегда обрабатывайте ошибки
async function safeAsyncOperation() {
  try {
    const result = await someAsyncFunction();
    return result;
  } catch (error) {
    // Логирование ошибки
    console.error('Ошибка асинхронной операции:', error);
    // Возврат значения по умолчанию или повторный выброс ошибки
    throw error;
  }
}

// 2. Используйте типизацию для асинхронных функций
async function wellTypedFunction(userId: number): Promise<User | null> {
  try {
    return await fetchUser(userId);
  } catch {
    return null;
  }
}

// 3. Избегайте лишних await
async function avoidUnnecessaryAwait() {
  // Плохо
  // const user = await fetchUser(1);
  // return user;
  
  // Хорошо - напрямую возвращаем промис
  return fetchUser(1);
}

// 4. Используйте Promise.all для параллельных операций
async function parallelOperations() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(1),
    fetchUserPosts(1),
    fetchUserComments(1)
  ]);
  
  return { user, posts, comments };
}
```

## Практические примеры из реальной разработки

### 1. Стратегия кеширования

```typescript
class AsyncCache<T> {
  private cache = new Map<string, { data: T; timestamp: number; promise?: Promise<T> }>();
  
  constructor(private ttl: number = 5 * 60 * 1000) {} // 5 минут по умолчанию
  
  async get(key: string, fetcher: () => Promise<T>): Promise<T> {
    const cached = this.cache.get(key);
    
    // Проверяем, есть ли валидный кэш
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }
    
    // Проверяем, есть ли текущий запрос
    if (cached && cached.promise) {
      return cached.promise;
    }
    
    // Выполняем новый запрос
    const promise = fetcher()
      .then(data => {
        this.cache.set(key, { data, timestamp: Date.now() });
        return data;
      })
      .catch(error => {
        this.cache.delete(key); // Удаляем ошибочный кэш
        throw error;
      });
    
    // Сохраняем промис для предотвращения дублирующихся запросов
    if (cached) {
      this.cache.set(key, { ...cached, promise });
    } else {
      this.cache.set(key, { data: undefined as any, timestamp: Date.now(), promise });
    }
    
    return promise;
  }
  
  clear() {
    this.cache.clear();
  }
}

// Использование
const userCache = new AsyncCache<User>();

async function getCachedUser(id: number) {
  return userCache.get(`user-${id}`, () => fetchUser(id));
}
```

### 2. Очередь асинхронных операций

```typescript
interface QueueItem<T, R> {
  id: string;
  operation: () => Promise<R>;
  resolve: (value: R) => void;
  reject: (reason: any) => void;
  timestamp: number;
}

class AsyncQueue {
  private queue: QueueItem<any, any>[] = [];
  private isProcessing = false;
  private concurrency: number;
  
  constructor(concurrency: number = 1) {
    this.concurrency = concurrency;
  }
  
  async add<T>(operation: () => Promise<T>): Promise<T> {
    return new Promise<T>((resolve, reject) => {
      this.queue.push({
        id: Math.random().toString(36),
        operation,
        resolve,
        reject,
        timestamp: Date.now()
      });
      
      if (!this.isProcessing) {
        this.processQueue();
      }
    });
  }
  
  private async processQueue() {
    if (this.queue.length === 0) {
      this.isProcessing = false;
      return;
    }
    
    this.isProcessing = true;
    
    // Запускаем ограниченное количество операций параллельно
    const batch = this.queue.splice(0, this.concurrency);
    const promises = batch.map(item => this.executeItem(item));
    
    await Promise.allSettled(promises);
    
    // Продолжаем обработку оставшихся элементов
    this.processQueue();
  }
  
  private async executeItem<T>(item: QueueItem<T, T>) {
    try {
      const result = await item.operation();
      item.resolve(result);
    } catch (error) {
      item.reject(error);
    }
  }
}

// Использование
const queue = new AsyncQueue(3); // Максимум 3 параллельные операции

async function useAsyncQueue() {
  const promises = Array.from({ length: 10 }, (_, i) => 
    queue.add(() => fetchUser(i + 1))
  );
  
  const results = await Promise.all(promises);
  console.log('Результаты:', results);
}
```

## Лучшие практики

1. **Всегда обрабатывайте ошибки** - используйте try/catch для асинхронных операций
2. **Используйте параллелизм когда это возможно** - Promise.all для независимых операций
3. **Избегайте лишних await** - возвращайте промисы напрямую когда возможно
4. **Управляйте конкурентностью** - особенно при работе с внешними API
5. **Используйте таймауты** - для предотвращения зависания операций
6. **Документируйте асинхронные функции** - особенно их поведение при ошибках

## Связи с другими концепциями

- [[js/promises]] - основы промисов, на которых основан async/await
- [[ts/type-system/type-compatibility|Совместимость типов]] - как TypeScript проверяет асинхронные типы
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использоваться для извлечения типов из промисов
- [[ts/utility-types/index|Утилиты типов]] - такие как Awaited для работы с типами промисов
- [[architecture/api-integration]] - архитектурные аспекты асинхронных операций