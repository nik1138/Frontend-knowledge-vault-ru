# Детерминированные функции

Детерминированность - одно из ключевых свойств чистых функций. Детерминированная функция всегда возвращает одинаковый результат для одинаковых входных данных.

## Что такое детерминированность?

Детерминированная функция - это функция, которая для одного и того же набора входных параметров всегда возвращает один и тот же результат. Это свойство делает функции предсказуемыми и надежными.

```typescript
// Детерминированная функция
const add = (a: number, b: number): number => a + b;

// Всегда возвращает одинаковый результат для одинаковых аргументов
console.log(add(2, 3)); // 5
console.log(add(2, 3)); // 5
console.log(add(2, 3)); // 5
```

## Примеры детерминированных функций

### Математические функции

```typescript
// Базовые математические операции
const multiply = (a: number, b: number): number => a * b;
const divide = (a: number, b: number): number => a / b;
const power = (base: number, exponent: number): number => Math.pow(base, exponent);

// Тригонометрические функции
const sin = (angle: number): number => Math.sin(angle);
const cos = (angle: number): number => Math.cos(angle);
const tan = (angle: number): number => Math.tan(angle);

// Логарифмические функции
const log = (n: number): number => Math.log(n);
const log10 = (n: number): number => Math.log10(n);
```

### Функции для работы со строками

```typescript
// Преобразование строк
const toUpperCase = (str: string): string => str.toUpperCase();
const toLowerCase = (str: string): string => str.toLowerCase();
const trim = (str: string): string => str.trim();

// Форматирование строк
const formatCurrency = (amount: number, currency: string = 'USD'): string => 
  new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);

// Хеширование (детерминированное)
const simpleHash = (str: string): number => {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash; // Преобразование в 32-bit integer
  }
  return hash;
};
```

### Функции для работы с массивами

```typescript
// Функции доступа к элементам
const head = <T>(array: readonly T[]): T | undefined => array[0];
const last = <T>(array: readonly T[]): T | undefined => 
  array[array.length - 1];

// Функции агрегации
const sum = (numbers: readonly number[]): number => 
  numbers.reduce((acc, n) => acc + n, 0);

const product = (numbers: readonly number[]): number => 
  numbers.reduce((acc, n) => acc * n, 1);

const average = (numbers: readonly number[]): number => {
  if (numbers.length === 0) return 0;
  return sum(numbers) / numbers.length;
};

// Функции поиска
const findMax = (numbers: readonly number[]): number => 
  Math.max(...numbers);

const findMin = (numbers: readonly number[]): number => 
  Math.min(...numbers);
```

### Функции для работы с объектами

```typescript
// Получение свойств
const getProperty = <T, K extends keyof T>(obj: T, key: K): T[K] => 
  obj[key];

// Проверка свойств
const hasProperty = <T, K extends keyof T>(obj: T, key: K): boolean => 
  key in obj;

// Подсчет свойств
const getPropertyCount = <T>(obj: T): number => 
  Object.keys(obj as object).length;
```

## Недетерминированные функции

Недетерминированные функции возвращают разные результаты для одинаковых входных данных. Такие функции не являются чистыми.

```typescript
// Недетерминированные функции
const getRandom = (): number => Math.random(); // Разные результаты
const getCurrentTime = (): Date => new Date(); // Разные результаты
const getUUID = (): string => crypto.randomUUID(); // Разные результаты

// Функции с побочными эффектами
let counter = 0;
const incrementCounter = (): number => ++counter; // Изменяет внешнее состояние

const readFromLocalStorage = (key: string): string | null => 
  localStorage.getItem(key); // Зависит от внешнего состояния

const getCurrentUser = (): User | null => 
  // Зависит от внешнего состояния (например, сессии)
  getCurrentSession()?.user || null;
```

## Преимущества детерминированности

### 1. Предсказуемость

Детерминированные функции всегда ведут себя одинаково, что делает их поведение предсказуемым:

```typescript
// Предсказуемая функция
const calculateArea = (width: number, height: number): number => 
  width * height;

// Всегда возвращает одинаковый результат
console.assert(calculateArea(5, 10) === 50);
console.assert(calculateArea(5, 10) === 50); // Всегда 50
```

### 2. Кэширование (мемоизация)

Детерминированные функции можно кэшировать, так как их результат зависит только от входных данных:

```typescript
// Мемоизация детерминированных функций
const memoize = <T extends (...args: any[]) => any>(fn: T): T => {
  const cache = new Map<string, ReturnType<T>>();
  
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
};

// Пример использования мемоизации
const expensiveCalculation = (n: number): number => {
  console.log(`Calculating for ${n}`);
  return n * n * n;
};

const memoizedCalculation = memoize(expensiveCalculation);

console.log(memoizedCalculation(5)); // Calculating for 5, 125
console.log(memoizedCalculation(5)); // 125 (без повторного вычисления)
console.log(memoizedCalculation(3)); // Calculating for 3, 27
```

### 3. Тестируемость

Детерминированные функции легко тестировать, так как они не зависят от внешнего состояния:

```typescript
// Детерминированная функция для тестирования
const multiply = (a: number, b: number): number => a * b;

// Легко тестировать
console.assert(multiply(2, 3) === 6, "2 * 3 should equal 6");
console.assert(multiply(0, 5) === 0, "0 * 5 should equal 0");
console.assert(multiply(-1, 4) === -4, "-1 * 4 should equal -4");
```

### 4. Параллелизм

Детерминированные функции безопасны для параллельного выполнения, так как они не изменяют общее состояние:

```typescript
// Детерминированные функции могут выполняться параллельно
const processData = (data: number[]): number[] => 
  data.map(x => x * 2).filter(x => x > 10);

// Безопасно для параллельного выполнения
const result1 = processData([1, 2, 3, 4, 5]);
const result2 = processData([6, 7, 8, 9, 10]);
```

## Преобразование недетерминированных функций

### Работа со случайными значениями

```typescript
// Недетерминированная функция
const getRandomNumber = (): number => Math.random();

// Детерминированная версия (псевдослучайные числа)
class PseudoRandom {
  private seed: number;
  
  constructor(seed: number) {
    this.seed = seed;
  }
  
  next(): number {
    this.seed = (this.seed * 9301 + 49297) % 233280;
    return this.seed / 233280;
  }
}

const getRandomWithSeed = (seed: number): number => {
  const prng = new PseudoRandom(seed);
  return prng.next();
};

// Теперь функция детерминирована
console.log(getRandomWithSeed(123)); // Всегда одинаковый результат
console.log(getRandomWithSeed(123)); // Всегда одинаковый результат
```

### Работа с временем

```typescript
// Недетерминированная функция
const getCurrentTimestamp = (): number => Date.now();

// Детерминированная версия
const getTimestamp = (date: Date): number => date.getTime();

// Использование
const specificDate = new Date('2023-01-01T00:00:00Z');
console.log(getTimestamp(specificDate)); // Всегда 1672531200000
```

### Работа с внешним состоянием

```typescript
// Недетерминированная функция
let globalCounter = 0;
const incrementImpure = (): number => ++globalCounter;

// Детерминированная версия
const incrementPure = (counter: number): number => counter + 1;

// Использование
const newCounter = incrementPure(globalCounter);
```

## Паттерны для обеспечения детерминированности

### 1. Передача зависимостей как параметров

```typescript
// Недетерминированная функция
const getCurrentUser = (): User | null => {
  return sessionStorage.getItem('user') 
    ? JSON.parse(sessionStorage.getItem('user')!) 
    : null;
};

// Детерминированная версия
const parseUser = (userData: string | null): User | null => {
  return userData ? JSON.parse(userData) : null;
};

// Использование
const userData = sessionStorage.getItem('user');
const user = parseUser(userData);
```

### 2. Использование замыканий

```typescript
// Создание детерминированной функции с фиксированными зависимостями
const createCalculator = (taxRate: number) => 
  (amount: number): number => amount * taxRate;

// Детерминированная функция с фиксированной ставкой налога
const calculateTax = createCalculator(0.2); // 20% налог

console.log(calculateTax(100)); // 20
console.log(calculateTax(100)); // 20 (всегда одинаково)
```

### 3. Использование конфигурации

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

// Недетерминированная функция
const fetchData = (endpoint: string): Promise<any> => 
  fetch(`${getApiUrl()}${endpoint}`).then(res => res.json());

// Детерминированная версия
const createApiClient = (config: Config) => 
  (endpoint: string): Promise<any> => 
    fetch(`${config.apiUrl}${endpoint}`).then(res => res.json());

// Создание детерминированного клиента
const apiClient = createApiClient({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
});

// Всегда использует одну и ту же конфигурацию
apiClient('/users').then(data => console.log(data));
```

## Лучшие практики

### 1. Предпочитайте детерминированные функции

```typescript
// Хорошо: детерминированная функция
const calculateTax = (amount: number, rate: number): number => 
  amount * rate;

// Плохо: функция с внешними зависимостями
let taxRate = 0.1;
const calculateTaxImpure = (amount: number): number => {
  return amount * taxRate; // Зависит от внешнего состояния
};

// Лучше: детерминированная функция с явными зависимостями
const calculateTaxPure = (amount: number, rate: number): number => {
  return amount * rate;
};
```

### 2. Избегайте глобального состояния

```typescript
// Плохо: использование глобального состояния
let globalMultiplier = 2;
const multiplyByGlobal = (n: number): number => n * globalMultiplier;

// Хорошо: явная передача зависимостей
const multiply = (n: number, multiplier: number): number => n * multiplier;
```

### 3. Используйте неизменяемые структуры данных

```typescript
// Хорошо: неизменяемость
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

const updateUser = (user: User, newName: string): User => ({
  ...user,
  name: newName
});

// Плохо: изменение оригинального объекта
const updateUserImpure = (user: User, newName: string): User => {
  user.name = newName; // Изменение оригинального объекта
  return user;
};
```

## Связи с другими концепциями

- [[ts/functional-programming/pure-functions/Чистые_функции|Чистые функции]] - Основная страница раздела
- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/testing/index|Тестирование]] - Тестирование детерминированных функций
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация детерминированных функций

## Следующие шаги

- [[ts/functional-programming/pure-functions/side-effects|Побочные эффекты]] - Изучение побочных эффектов и их минимизации
- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции