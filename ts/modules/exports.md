# Default and Named Exports

Экспорты в TypeScript позволяют определять, какие компоненты модуля будут доступны для импорта в других файлах. Существуют два типа экспортов: именованные (named) и дефолтные (default), каждый из которых имеет свои особенности и способы использования.

## Именованные экспорты (Named Exports)

### Прямые именованные экспорты
```typescript
// math-utils.ts
// Прямые экспорты переменных
export const PI = 3.14159;
export const E = 2.71828;
export const GOLDEN_RATIO = 1.61803;

// Прямые экспорты функций
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Division by zero");
  }
  return a / b;
}

// Прямые экспорты классов
export class Calculator {
  static version = '1.0.0';
  
  static add(a: number, b: number): number {
    return a + b;
  }
  
  static multiply(a: number, b: number): number {
    return a * b;
  }
}

// Прямые экспорты интерфейсов и типов
export interface Operation {
  execute(a: number, b: number): number;
}

export type BinaryOperation = (a: number, b: number) => number;
```

### Именованные экспорты через export statement
```typescript
// exports-section.ts
const PI = 3.14159;
const E = 2.71828;

function add(a: number, b: number): number {
  return a + b;
}

function multiply(a: number, b: number): number {
  return a * b;
}

class Calculator {
  static add = add;
  static multiply = multiply;
}

// Экспортируем конкретные переменные по именам
export { PI, E };
export { add, multiply };
export { Calculator };

// Экспортируем с переименованием
export { Calculator as MathCalculator, add as sum, multiply as product };
```

### Импорт именованных экспортов
```typescript
// main.ts
// Импорт конкретных именованных экспортов
import { add, multiply, PI, Calculator } from './math-utils';

// Импорт с переименованием
import { Calculator as MathCalc, add as addNumbers } from './math-utils';

// Импорт всего как namespace
import * as MathUtils from './math-utils';

// Использование
console.log(add(2, 3)); // 5
console.log(multiply(4, 5)); // 20
console.log(PI); // 3.14159

const calc = new MathCalc();
console.log(calc.constructor.name); // Calculator

// Использование namespace
console.log(MathUtils.subtract(10, 5)); // 5
console.log(MathUtils.GOLDEN_RATIO); // 1.61803
console.log(MathUtils.MathCalculator); // Calculator class
```

## Дефолтные экспорты (Default Exports)

### Дефолтные экспорты классов
```typescript
// calculator.ts
class Calculator {
  static version = '1.0.0';
  
  static add(a: number, b: number): number {
    return a + b;
  }
  
  static subtract(a: number, b: number): number {
    return a - b;
  }
  
  static multiply(a: number, b: number): number {
    return a * b;
  }
  
  static divide(a: number, b: number): number {
    if (b === 0) throw new Error("Division by zero");
    return a / b;
  }
  
  calculate(operation: 'add' | 'subtract' | 'multiply' | 'divide', a: number, b: number): number {
    switch (operation) {
      case 'add': return Calculator.add(a, b);
      case 'subtract': return Calculator.subtract(a, b);
      case 'multiply': return Calculator.multiply(a, b);
      case 'divide': return Calculator.divide(a, b);
    }
  }
}

export default Calculator;
```

### Дефолтные экспорты функций
```typescript
// string-processor.ts
export default function processString(str: string, operation: 'upper' | 'lower' | 'reverse' = 'upper'): string {
  switch (operation) {
    case 'upper': return str.toUpperCase();
    case 'lower': return str.toLowerCase();
    case 'reverse': return str.split('').reverse().join('');
    default: return str;
  }
}

// Также можно экспортировать функцию отдельно
function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// И потом сделать дефолтным
export { validateEmail as default };
```

### Дефолтные экспорты объектов и констант
```typescript
// config.ts
const AppConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
  debug: false as boolean
};

export default AppConfig;

// utils.ts
const StringHelpers = {
  capitalize: (str: string) => str.charAt(0).toUpperCase() + str.slice(1),
  truncate: (str: string, length: number) => str.length > length ? str.substring(0, length) + '...' : str,
  slugify: (str: string) => str.toLowerCase().replace(/[^a-z0-9]+/g, '-')
};

export default StringHelpers;
```

### Импорт дефолтных экспортов
```typescript
// usage.ts
import Calculator from './calculator';  // Дефолтный импорт (любое имя)
import processString from './string-processor';  // Дефолтный импорт
import config from './config';  // Дефолтный импорт
import stringHelpers from './utils';  // Дефолтный импорт

const calc = new Calculator();
console.log(calc.calculate('add', 5, 3)); // 8

console.log(processString("hello", "upper")); // "HELLO"
console.log(config.apiUrl); // "https://api.example.com"
console.log(stringHelpers.capitalize("hello")); // "Hello"
```

## Комбинированные экспорты

### Микс дефолтного и именованных экспортов
```typescript
// mixed-exports.ts
export const DEFAULT_TIMEOUT = 5000;
export const MAX_RETRIES = 3;

export class ApiService {
  constructor(private timeout: number = DEFAULT_TIMEOUT) {}
  
  static createDefault(): ApiService {
    return new ApiService();
  }
}

function createService(): ApiService {
  return new ApiService();
}

// Дефолтный экспорт
export default createService;

// Также можно экспортировать как именованный
export { createService as createApiService };
```

### Импорт комбинированных экспортов
```typescript
// main.ts
// Импорт дефолтного и именованных экспортов одновременно
import createService, { DEFAULT_TIMEOUT, MAX_RETRIES, ApiService, createApiService } from './mixed-exports';

// Использование
const service1 = createService(); // через дефолтный экспорт
const service2 = createApiService(); // через именованный экспорт
const defaultService = ApiService.createDefault(); // через именованный класс

console.log(DEFAULT_TIMEOUT); // 5000
console.log(MAX_RETRIES); // 3
```

## Переэкспорт (Re-exports)

### Частичный переэкспорт
```typescript
// math-restricted.ts
// Переэкспортируем только часть из другого модуля
export { add, multiply, PI } from './math-utils';

// Переэкспортируем с переименованием
export { Calculator as BasicCalculator } from './math-utils';

// Переэкспортируем с условиями
export { subtract as difference, divide as quotient } from './math-utils';
```

### Полный переэкспорт
```typescript
// math-full.ts
// Переэкспортируем всё
export * from './math-utils';

// Или с фильтрацией
export { add, multiply } from './math-utils';
export { subtract, divide } from './math-utils';
export { PI } from './math-utils';
```

### Условный переэкспорт
```typescript
// platform-specific.ts - пример условного переэкспорта
if (typeof window !== 'undefined') {
  // В браузере
  export { BrowserService } from './browser-service';
  export { LocalStorage } from './browser-storage';
} else {
  // На сервере
  export { NodeService } from './node-service';
  export { FsStorage } from './filesystem-storage';
}

// В реальности так не делают, но показывает идею
// Настоящий условный экспорт делается через package.json
```

## Практические примеры

### API модуль с различными типами экспортов
```typescript
// api.ts
// Именованные экспорты для вспомогательных типов
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

export interface RequestOptions {
  timeout?: number;
  retries?: number;
  headers?: Record<string, string>;
}

// Именованные экспорты для вспомогательных функций
export function createDefaultHeaders(): Record<string, string> {
  return {
    'Content-Type': 'application/json',
    'Accept': 'application/json'
  };
}

// Дефолтный экспорт основного класса
class ApiService {
  constructor(private baseUrl: string, private options: RequestOptions = {}) {}
  
  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    try {
      const response = await fetch(`${this.baseUrl}${endpoint}`, {
        headers: { ...createDefaultHeaders(), ...this.options.headers }
      });
      
      const data = await response.json();
      return { success: true, data };
    } catch (error) {
      return { success: false, error: (error as Error).message };
    }
  }
  
  async post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    try {
      const response = await fetch(`${this.baseUrl}${endpoint}`, {
        method: 'POST',
        headers: { ...createDefaultHeaders(), ...this.options.headers },
        body: JSON.stringify(data)
      });
      
      const result = await response.json();
      return { success: true, data: result };
    } catch (error) {
      return { success: false, error: (error as Error).message };
    }
  }
}

export default ApiService;

// Также экспортируем фабричный метод как именованный
export function createApiService(baseUrl: string, options: RequestOptions = {}): ApiService {
  return new ApiService(baseUrl, options);
}
```

### Импорт в приложении
```typescript
// app.ts
import ApiService, { 
  type ApiResponse, 
  type RequestOptions, 
  createDefaultHeaders,
  createApiService 
} from './api';

// Использование
const api = new ApiService('https://api.example.com');
const altApi = createApiService('https://api.example.com');

const headers = createDefaultHeaders();
type ResponseType = ApiResponse<{ id: number; name: string }>;
```

## Специфичные TypeScript особенности

### Экспорт типов
```typescript
// types.ts
export type UserID = string & { readonly brand: unique symbol };
export type Timestamp = number & { readonly __timestamp: unique symbol };
export type Percentage = number & { readonly __percentage: unique symbol };

export interface User {
  id: UserID;
  name: string;
  createdAt: Timestamp;
  satisfaction: Percentage;
}

// Функция-типа
export type Validator<T> = (value: T) => boolean;

// Обобщённый тип
export type Response<T> = {
  data: T;
  success: boolean;
  errors?: string[];
};
```

### Type-only экспорты
```typescript
// type-exports.ts
export type { User, Validator, Response } from './types';

// Это позволяет экспортировать только типы, 
// не создавая JavaScript кода для импорта
```

## Лучшие практики

### 1. Используйте именованные экспорты для библиотек
```typescript
// utils.ts - хорошая практика для библиотек
export function formatDate(date: Date): string { /* */ }
export function formatNumber(num: number): string { /* */ }
export function validateEmail(email: string): boolean { /* */ }
export const FORMATTER_CONFIG = { /* */ };
```

### 2. Используйте дефолтные экспорты для основных компонентов
```typescript
// main-component.ts
class MainComponent { /* */ }
export default MainComponent; // Один основной экспорт
```

### 3. Будьте последовательны в проекте
```typescript
// Для функций часто используют именованные экспорты
export function utilityFunction() { }
export function helperFunction() { }

// Для классов может быть как именованный, так и дефолтный экспорт
export class ServiceClass { } // если нужно экспортировать другие вспомогательные классы
// или
export default class MainServiceClass { } // если это основной класс модуля
```

### 4. Используйте переэкспорт для удобства API
```typescript
// index.ts - точка входа в библиотеку
export { default as Calculator } from './calculator';
export { add, multiply, subtract, divide } from './math-operations';
export { PI, E } from './constants';
export type { Operation, BinaryOperation } from './types';
```

## Ошибки и ловушки

### Частые ошибки
```typescript
// 1. Попытка экспорта по умолчанию более одного значения
// ПЛОХО:
// export default function func1() {}
// export default function func2() {} // ОШИБКА: только один default на модуль

// 2. Неправильный синтаксис
// ПЛОХО:
// export default const value = 42; // ОШИБКА
// ХОРОШО:
export default 42;

// ПЛОХО:
// export default let value = 42; // ОШИБКА
// ХОРОШО:
let value = 42;
export default value;

// 3. Импорт default с фигурными скобками (CommonJS в ES Modules)
// ПЛОХО:
// import { Calculator } from './calculator'; // если Calculator - default экспорт
// ХОРОШО:
// import Calculator from './calculator'; // для default экспортов
```

### Совместимость с CommonJS
```typescript
// Для совместимости с CommonJS могут быть особенности:
// my-module.ts
export default class MyClass { }
export const CONSTANT = 42;

// В CommonJS:
// const MyClass = require('./my-module').default; // для default экспорт
// const { CONSTANT } = require('./my-module'); // для именованных

// Это нужно учитывать при публикации библиотек
```

## Сравнение именованных и дефолтных экспортов

| Feature | Named Exports | Default Exports |
|---------|---------------|-----------------|
| Количество | Много на модуль | Только один на модуль |
| Имя при импорте | Должно совпадать | Любое имя при импорте |
| Импорт | `import { name }` | `import name` |
| Перемещение | Труднее переименовать | Легко переименовать |
| Tree Shaking | Лучше | Зависит от реализации |
| Читаемость | Ясные имена | Гибкие имена |
| Ошибки импорта | Компилятор поймает ошибки имен | Меньше защиты от ошибок |

## Совместимость с инструментами

### Webpack и tree shaking
```typescript
// Для лучшего tree shaking используйте именованные экспорты
export function heavyFunction1() { }
export function heavyFunction2() { }
export function heavyFunction3() { }
// Можно импортировать только нужные:
// import { heavyFunction1 } from './utils'; // heavyFunction2 и 3 будут "обрещены"
```

## Связь с другими концепциями
- [[es-modules-syntax]] - синтаксис ES Modules
- [[module-resolution]] - как TypeScript разрешает импорты
- [[declaration-files]] - экспорт типов в declaration файлов
- [[advanced]] - продвинутые паттерны экспорта