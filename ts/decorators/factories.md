# Фабрики декораторов

Фабрики декораторов - это функции, которые возвращают декораторы. Они позволяют параметризовать декораторы и создавать более гибкие и переиспользуемые решения.

## Базовый синтаксис

Фабрика декоратора - это функция, которая возвращает функцию декоратора:

```typescript
function decoratorFactory(parameter: any) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Логика декоратора с использованием parameter
  };
}

// Использование
class MyClass {
  @decoratorFactory("some value")
  method() {
    // ...
  }
}
```

## Простые примеры

### 1. Базовая фабрика декораторов методов

```typescript
function log(level: "info" | "warn" | "error" = "info") {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      console[level](`[${level.toUpperCase()}] Calling method ${propertyKey}`);
      const result = originalMethod.apply(this, args);
      console[level](`[${level.toUpperCase()}] Method ${propertyKey} completed`);
      return result;
    };
  };
}

class UserService {
  @log("info")
  getUser(id: string) {
    return { id, name: `User ${id}` };
  }
  
  @log("warn")
  deleteUser(id: string) {
    console.log(`Deleting user ${id}`);
  }
  
  @log("error")
  riskyOperation() {
    throw new Error("Something went wrong");
  }
}

const service = new UserService();
service.getUser("123");
// Выведет: "[INFO] Calling method getUser"
// Выведет: "[INFO] Method getUser completed"

service.deleteUser("123");
// Выведет: "[WARN] Calling method deleteUser"
// Выведет: "[WARN] Method deleteUser completed"

// service.riskyOperation();
// Выведет: "[ERROR] Calling method riskyOperation"
// Выведет: "[ERROR] Method riskyOperation completed"
// Затем ошибка будет выброшена
```

### 2. Фабрика декораторов для валидации

```typescript
function validate(validationFn: (value: any) => boolean, errorMessage: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      args.forEach((arg, index) => {
        if (!validationFn(arg)) {
          throw new Error(`Parameter ${index} of method ${propertyKey}: ${errorMessage}`);
        }
      });
      
      return originalMethod.apply(this, args);
    };
  };
}

function isString(value: any): boolean {
  return typeof value === "string" && value.length > 0;
}

function isPositiveNumber(value: any): boolean {
  return typeof value === "number" && value > 0;
}

function isEmail(value: any): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return typeof value === "string" && emailRegex.test(value);
}

class OrderService {
  @validate(isString, "Product ID must be a non-empty string")
  @validate(isPositiveNumber, "Quantity must be a positive number")
  createOrder(productId: string, quantity: number) {
    return {
      id: `order_${Date.now()}`,
      productId,
      quantity
    };
  }
  
  @validate(isEmail, "Email must be valid")
  sendNotification(email: string, message: string) {
    console.log(`Sending notification to ${email}: ${message}`);
  }
}

const service = new OrderService();
service.createOrder("product123", 5); // ✅ Валидация пройдена

// service.createOrder("", 5); // ❌ Ошибка: Parameter 0 of method createOrder: Product ID must be a non-empty string
// service.createOrder("product123", -1); // ❌ Ошибка: Parameter 1 of method createOrder: Quantity must be a positive number
```

## Продвинутые примеры

### 1. Фабрика декораторов для кэширования с настройками

```typescript
interface CacheOptions {
  ttl?: number; // Время жизни кэша в миллисекундах
  keyGenerator?: (...args: any[]) => string; // Функция для генерации ключа кэша
  cacheInstance?: Map<string, { value: any; timestamp: number }>; // Экземпляр кэша
}

function cache(options: CacheOptions = {}) {
  const {
    ttl = 60000, // По умолчанию 1 минута
    keyGenerator = (args: any[]) => JSON.stringify(args),
    cacheInstance = new Map()
  } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      const key = keyGenerator(args);
      const cached = cacheInstance.get(key);
      const now = Date.now();
      
      if (cached && now - cached.timestamp < ttl) {
        console.log(`Cache hit for method ${propertyKey} with key: ${key}`);
        return cached.value;
      }
      
      console.log(`Cache miss for method ${propertyKey} with key: ${key}`);
      const result = originalMethod.apply(this, args);
      
      cacheInstance.set(key, {
        value: result,
        timestamp: now
      });
      
      return result;
    };
  };
}

class DataProvider {
  @cache({ ttl: 30000 }) // 30 секунд
  fetchData(id: string): any {
    console.log(`Fetching data for ID: ${id}`);
    return { id, data: `Data for ${id}`, timestamp: Date.now() };
  }
  
  @cache({
    keyGenerator: (userId: string, page: number) => `${userId}:${page}`,
    ttl: 60000 // 1 минута
  })
  getUserData(userId: string, page: number): any {
    console.log(`Fetching data for user ${userId}, page ${page}`);
    return { userId, page, data: `Page ${page} data`, timestamp: Date.now() };
  }
}

const provider = new DataProvider();
console.log(provider.fetchData("123")); // Запрос данных
console.log(provider.fetchData("123")); // Из кэша

setTimeout(() => {
  console.log(provider.fetchData("123")); // Запрос данных (кэш истек)
}, 35000);

console.log(provider.getUserData("user123", 1)); // Запрос данных
console.log(provider.getUserData("user123", 1)); // Из кэша
console.log(provider.getUserData("user123", 2)); // Запрос данных (другая страница)
```

### 2. Фабрика декораторов для повторных попыток с настройками

```typescript
interface RetryOptions {
  maxAttempts?: number;
  delay?: number;
  exponentialBackoff?: boolean;
  shouldRetry?: (error: Error) => boolean;
}

function retry(options: RetryOptions = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    exponentialBackoff = false,
    shouldRetry = () => true
  } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
      let lastError: Error | null = null;
      
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error as Error;
          
          if (!shouldRetry(lastError)) {
            throw lastError;
          }
          
          console.log(`Attempt ${attempt} failed for method ${propertyKey}:`, lastError.message);
          
          if (attempt < maxAttempts) {
            const currentDelay = exponentialBackoff ? delay * Math.pow(2, attempt - 1) : delay;
            console.log(`Retrying in ${currentDelay}ms...`);
            await new Promise(resolve => setTimeout(resolve, currentDelay));
          }
        }
      }
      
      throw new Error(`Method ${propertyKey} failed after ${maxAttempts} attempts: ${lastError?.message}`);
    };
  };
}

class NetworkService {
  @retry({
    maxAttempts: 5,
    delay: 1000,
    exponentialBackoff: true,
    shouldRetry: (error: Error) => error.message !== "Unauthorized"
  })
  async fetchData(url: string): Promise<any> {
    // Имитация сетевого запроса с возможными ошибками
    const random = Math.random();
    
    if (random < 0.3) {
      throw new Error("Network error");
    } else if (random < 0.4) {
      throw new Error("Unauthorized");
    }
    
    return { data: "Success!", url };
  }
}

const networkService = new NetworkService();
networkService.fetchData("https://api.example.com/data")
  .then(result => console.log("Success:", result))
  .catch(error => console.error("Final error:", error));
```

### 3. Фабрика декораторов для метрик и мониторинга

```typescript
interface MetricsOptions {
  namespace?: string;
  labels?: Record<string, string>;
  includeArguments?: boolean;
  includeResult?: boolean;
}

class MetricsCollector {
  private static metrics: Map<string, number[]> = new Map();
  
  static record(metricName: string, value: number) {
    if (!this.metrics.has(metricName)) {
      this.metrics.set(metricName, []);
    }
    this.metrics.get(metricName)!.push(value);
  }
  
  static getStats(metricName: string): { count: number; avg: number; min: number; max: number } | null {
    const values = this.metrics.get(metricName);
    if (!values || values.length === 0) {
      return null;
    }
    
    const sum = values.reduce((a, b) => a + b, 0);
    return {
      count: values.length,
      avg: sum / values.length,
      min: Math.min(...values),
      max: Math.max(...values)
    };
  }
}

function metrics(options: MetricsOptions = {}) {
  const {
    namespace = "default",
    labels = {},
    includeArguments = false,
    includeResult = false
  } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
      const startTime = performance.now();
      
      try {
        const result = await originalMethod.apply(this, args);
        const duration = performance.now() - startTime;
        
        const metricName = `${namespace}.${target.constructor.name}.${propertyKey}`;
        MetricsCollector.record(metricName, duration);
        
        // Дополнительные метрики
        if (includeArguments) {
          const argsMetricName = `${metricName}.args`;
          MetricsCollector.record(argsMetricName, args.length);
        }
        
        if (includeResult && result !== undefined) {
          const resultMetricName = `${metricName}.result_size`;
          const resultSize = typeof result === "string" ? result.length : 
                            Array.isArray(result) ? result.length : 
                            typeof result === "object" ? Object.keys(result).length : 0;
          MetricsCollector.record(resultMetricName, resultSize);
        }
        
        return result;
      } catch (error) {
        const duration = performance.now() - startTime;
        const errorMetricName = `${namespace}.${target.constructor.name}.${propertyKey}.errors`;
        MetricsCollector.record(errorMetricName, duration);
        throw error;
      }
    };
  };
}

class DatabaseService {
  @metrics({
    namespace: "database",
    labels: { service: "user" },
    includeArguments: true,
    includeResult: true
  })
  async findUsers(filter: any): Promise<any[]> {
    // Имитация запроса к базе данных
    await new Promise(resolve => setTimeout(resolve, Math.random() * 100));
    return Array.from({ length: Math.floor(Math.random() * 100) }, (_, i) => ({
      id: i,
      name: `User ${i}`
    }));
  }
  
  @metrics({
    namespace: "database",
    labels: { service: "order" }
  })
  async createOrder(orderData: any): Promise<any> {
    // Имитация создания заказа
    await new Promise(resolve => setTimeout(resolve, Math.random() * 50));
    return { id: `order_${Date.now()}`, ...orderData };
  }
}

async function runMetricsExample() {
  const db = new DatabaseService();
  
  // Выполняем несколько операций
  await db.findUsers({ active: true });
  await db.findUsers({ active: false });
  await db.createOrder({ userId: "123", amount: 100 });
  
  // Получаем статистику
  console.log("Database.User.findUsers stats:", 
    MetricsCollector.getStats("database.DatabaseService.findUsers"));
  console.log("Database.Order.createOrder stats:", 
    MetricsCollector.getStats("database.DatabaseService.createOrder"));
}

runMetricsExample();
```

## Практические примеры

### 1. Фабрика декораторов для авторизации с ролями

```typescript
interface AuthOptions {
  roles?: string[];
  permissions?: string[];
  allowAnonymous?: boolean;
}

function authorize(options: AuthOptions = {}) {
  const {
    roles = [],
    permissions = [],
    allowAnonymous = false
  } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      // Получаем текущего пользователя (в реальной реализации из контекста)
      const currentUser = getCurrentUser();
      
      if (!currentUser && !allowAnonymous) {
        throw new Error("Authentication required");
      }
      
      if (currentUser) {
        // Проверка ролей
        if (roles.length > 0 && !roles.some(role => currentUser.roles.includes(role))) {
          throw new Error(`Required roles: ${roles.join(", ")}`);
        }
        
        // Проверка разрешений
        if (permissions.length > 0 && !permissions.every(perm => currentUser.permissions.includes(perm))) {
          throw new Error(`Required permissions: ${permissions.join(", ")}`);
        }
      }
      
      return originalMethod.apply(this, args);
    };
  };
}

// Моковая функция для получения текущего пользователя
function getCurrentUser() {
  // В реальной реализации это будет извлекаться из контекста запроса
  return {
    id: "user123",
    roles: ["user", "admin"],
    permissions: ["read", "write"]
  };
}

class ContentService {
  @authorize({ roles: ["user", "admin"] })
  getContent(id: string) {
    return { id, content: `Content ${id}` };
  }
  
  @authorize({ permissions: ["write"] })
  updateContent(id: string, content: string) {
    return { id, content, updated: true };
  }
  
  @authorize({ allowAnonymous: true })
  getPublicContent(id: string) {
    return { id, content: `Public content ${id}` };
  }
}

const contentService = new ContentService();
console.log(contentService.getContent("123")); // ✅ Доступ есть
console.log(contentService.updateContent("123", "New content")); // ✅ Доступ есть
console.log(contentService.getPublicContent("123")); // ✅ Доступ есть (анонимный)
```

### 2. Фабрика декораторов для транзакций

```typescript
interface TransactionOptions {
  connectionName?: string;
  isolationLevel?: "READ_UNCOMMITTED" | "READ_COMMITTED" | "REPEATABLE_READ" | "SERIALIZABLE";
  rollbackOnError?: boolean;
}

class TransactionManager {
  private static transactions: Map<string, any> = new Map();
  private static connections: Map<string, any> = new Map();
  
  static async beginTransaction(options: TransactionOptions = {}): Promise<string> {
    const transactionId = `tx_${Date.now()}_${Math.random()}`;
    console.log(`Beginning transaction ${transactionId} with options:`, options);
    this.transactions.set(transactionId, { status: "active", options });
    return transactionId;
  }
  
  static async commitTransaction(transactionId: string) {
    const transaction = this.transactions.get(transactionId);
    if (transaction) {
      console.log(`Committing transaction ${transactionId}`);
      transaction.status = "committed";
    }
  }
  
  static async rollbackTransaction(transactionId: string) {
    const transaction = this.transactions.get(transactionId);
    if (transaction) {
      console.log(`Rolling back transaction ${transactionId}`);
      transaction.status = "rolledback";
    }
  }
}

function transaction(options: TransactionOptions = {}) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
      const transactionId = await TransactionManager.beginTransaction(options);
      
      try {
        const result = await originalMethod.apply(this, args);
        await TransactionManager.commitTransaction(transactionId);
        return result;
      } catch (error) {
        if (options.rollbackOnError !== false) {
          await TransactionManager.rollbackTransaction(transactionId);
        }
        throw error;
      }
    };
  };
}

class OrderService {
  @transaction({ 
    isolationLevel: "READ_COMMITTED",
    rollbackOnError: true 
  })
  async createOrder(userId: string, items: any[]): Promise<any> {
    console.log(`Creating order for user ${userId}`);
    
    // Имитация создания заказа
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // Имитация возможной ошибки
    if (Math.random() < 0.3) {
      throw new Error("Database error");
    }
    
    return { 
      id: `order_${Date.now()}`, 
      userId, 
      items,
      status: "created"
    };
  }
  
  @transaction({ 
    isolationLevel: "REPEATABLE_READ"
  })
  async updateOrderStatus(orderId: string, status: string): Promise<any> {
    console.log(`Updating order ${orderId} status to ${status}`);
    
    // Имитация обновления статуса
    await new Promise(resolve => setTimeout(resolve, 50));
    
    return { orderId, status, updated: Date.now() };
  }
}

async function runTransactionExample() {
  const orderService = new OrderService();
  
  try {
    const order = await orderService.createOrder("user123", [
      { productId: "product1", quantity: 2 },
      { productId: "product2", quantity: 1 }
    ]);
    console.log("Order created:", order);
  } catch (error) {
    console.error("Order creation failed:", error.message);
  }
  
  try {
    const updated = await orderService.updateOrderStatus("order123", "shipped");
    console.log("Order updated:", updated);
  } catch (error) {
    console.error("Order update failed:", error.message);
  }
}

runTransactionExample();
```

### 3. Фабрика декораторов для throttling и debouncing

```typescript
interface ThrottleOptions {
  delay: number;
  leading?: boolean; // Выполнять сразу при первом вызове
  trailing?: boolean; // Выполнять последний вызов после окончания задержки
}

interface DebounceOptions {
  delay: number;
  immediate?: boolean; // Выполнять сразу при первом вызове
}

function throttle(options: ThrottleOptions) {
  const { delay, leading = true, trailing = true } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    let timeoutId: NodeJS.Timeout | null = null;
    let lastExecTime = 0;
    let lastArgs: any[] | null = null;
    let lastThis: any = null;
    
    descriptor.value = function(...args: any[]) {
      const now = Date.now();
      const elapsed = now - lastExecTime;
      
      const execute = () => {
        lastExecTime = Date.now();
        originalMethod.apply(lastThis, lastArgs);
      };
      
      if (elapsed > delay) {
        // Достаточно времени прошло, выполняем немедленно
        if (leading) {
          lastExecTime = now;
          originalMethod.apply(this, args);
        } else {
          // Сохраняем для trailing вызова
          lastArgs = args;
          lastThis = this;
          if (trailing && !timeoutId) {
            timeoutId = setTimeout(() => {
              execute();
              timeoutId = null;
            }, delay);
          }
        }
      } else {
        // Слишком рано, планируем выполнение
        lastArgs = args;
        lastThis = this;
        if (trailing) {
          if (timeoutId) {
            clearTimeout(timeoutId);
          }
          timeoutId = setTimeout(() => {
            execute();
            timeoutId = null;
          }, delay - elapsed);
        }
      }
    };
  };
}

function debounce(options: DebounceOptions) {
  const { delay, immediate = false } = options;
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    let timeoutId: NodeJS.Timeout | null = null;
    
    descriptor.value = function(...args: any[]) {
      const callNow = immediate && !timeoutId;
      
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
      
      timeoutId = setTimeout(() => {
        timeoutId = null;
        if (!immediate) {
          originalMethod.apply(this, args);
        }
      }, delay);
      
      if (callNow) {
        originalMethod.apply(this, args);
      }
    };
  };
}

class SearchService {
  @debounce({ delay: 300 })
  search(query: string) {
    console.log(`Searching for: ${query}`);
    // Здесь был бы реальный поиск
    return `Results for ${query}`;
  }
  
  @throttle({ delay: 1000, trailing: true })
  trackEvent(eventName: string, data: any) {
    console.log(`Tracking event: ${eventName}`, data);
    // Здесь была бы отправка аналитики
  }
}

const searchService = new SearchService();

// Быстрые вызовы search будут debounced
searchService.search("a");
searchService.search("ap");
searchService.search("app");
searchService.search("appl");
searchService.search("apple");
// Только последний вызов будет выполнен через 300ms

// Быстрые вызовы trackEvent будут throttled
for (let i = 0; i < 10; i++) {
  setTimeout(() => {
    searchService.trackEvent("click", { button: i });
  }, i * 100);
}
// Будет выполнено ограниченное количество вызовов
```

## Ограничения и особенности

1. Фабрики декораторов позволяют параметризовать поведение декораторов
2. Параметры фабрики вычисляются во время определения класса, а не во время выполнения
3. Сложные фабрики могут замедлять компиляцию и выполнение
4. При использовании асинхронных фабрик необходимо корректно обрабатывать Promise

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/composition|Композиция декораторов]]
- [[ts/decorators/metadata|Метаданные и рефлексия]]