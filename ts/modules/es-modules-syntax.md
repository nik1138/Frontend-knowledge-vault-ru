# ES Modules Syntax in TypeScript

ES Modules (ECMAScript Modules) - это стандартный способ определения модулей в JavaScript и TypeScript. Он обеспечивает статические импорты и экспорты, что позволяет инструментам оптимизации предварительно обработать и уменьшить размер конечного кода.

## Основной синтаксис

### Именованные экспорты
```typescript
// math-operations.ts
export const PI = 3.14159;
export const E = 2.71828;

export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export class Calculator {
  static version = '1.0.0';
  
  static multiply(a: number, b: number): number {
    return a * b;
  }
  
  static add(a: number, b: number): number {
    return a + b;
  }
}
```

### Использование именованных импортов
```typescript
// main.ts
import { add, multiply, PI, Calculator } from './math-operations';

console.log(add(2, 3)); // 5
console.log(multiply(4, 5)); // 20
console.log(PI); // 3.14159

const calc = new Calculator();
console.log(calc.constructor.name); // Calculator
```

### Дефолтные экспорты
```typescript
// calculator.ts
class Calculator {
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
    if (b === 0) {
      throw new Error("Division by zero");
    }
    return a / b;
  }
}

export default Calculator;
```

### Использование дефолтных импортов
```typescript
// app.ts
import Calculator from './calculator';

const calc = new Calculator();
console.log(Calculator.add(10, 5)); // 15
```

## Комбинированный синтаксис

### Импорт всего модуля как namespace
```typescript
// utils.ts
export function formatDate(date: Date): string {
  return date.toISOString();
}

export function formatNumber(num: number): string {
  return num.toLocaleString();
}

export const constants = {
  MAX_FILE_SIZE: 1024 * 1024,
  DEFAULT_TIMEOUT: 5000
};
```

```typescript
// main.ts
import * as Utils from './utils';

console.log(Utils.formatDate(new Date()));
console.log(Utils.formatNumber(12345));
console.log(Utils.constants.MAX_FILE_SIZE);
```

### Комбинированный импорт
```typescript
// math.ts
export const ZERO = 0;
export const ONE = 1;

export default function calculate(operation: 'add' | 'multiply', a: number, b: number): number {
  switch (operation) {
    case 'add': return a + b;
    case 'multiply': return a * b;
  }
}

export class MathHelper {
  static isPositive(num: number): boolean {
    return num > 0;
  }
}
```

```typescript
// usage.ts
import Calculate, { ZERO, ONE, MathHelper } from './math';

console.log(Calculate('add', 5, 3)); // 8
console.log(ZERO); // 0
console.log(ONE); // 1
console.log(MathHelper.isPositive(-5)); // false
```

## Динамические импорты

### Асинхронные импорты
```typescript
// dynamic-loading.ts
async function loadCalculator() {
  // Динамический импорт позволяет лениво загружать модули
  const { default: Calculator } = await import('./calculator');
  return new Calculator();
}

async function performCalculation() {
  const calculator = await loadCalculator();
  return calculator.constructor.name;
}

// Условная загрузка
async function loadByPlatform() {
  if (typeof window !== 'undefined') {
    // Загрузка клиентского кода
    const { ClientService } = await import('./client-service');
    return new ClientService();
  } else {
    // Загрузка серверного кода
    const { ServerService } = await import('./server-service');
    return new ServerService();
  }
}
```

### Ленивая загрузка компонентов
```typescript
// component-loader.ts
interface ComponentConstructor {
  new (...args: any[]): any;
}

async function loadComponent(type: string): Promise<ComponentConstructor> {
  switch (type) {
    case 'chart':
      const { ChartComponent } = await import('./components/chart');
      return ChartComponent;
    case 'table':
      const { TableComponent } = await import('./components/table');
      return TableComponent;
    case 'form':
      const { FormComponent } = await import('./components/form');
      return FormComponent;
    default:
      const { DefaultComponent } = await import('./components/default');
      return DefaultComponent;
  }
}

// Использование
async function renderComponent(componentType: string) {
  const ComponentClass = await loadComponent(componentType);
  return new ComponentClass();
}
```

## Переэкспорт (Re-exports)

### Частичный переэкспорт
```typescript
// math-restricted.ts - ограничение экспорта
export { add, multiply } from './math-operations';
export { PI } from './math-operations';
```

### Полный переэкспорт
```typescript
// math-all.ts - переэкспорт всего
export * from './math-operations';
```

### Переэкспорт с переименованием
```typescript
// renamed-exports.ts
export { add as sum, multiply as product, Calculator as MathCalculator } from './math-operations';
```

## Импорт типов

### Синтаксис import type
```typescript
// types-only.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export type UserRole = 'admin' | 'user' | 'guest';

export class UserService {
  static validateUser(user: User): boolean {
    return user.name.length > 0 && user.email.includes('@');
  }
}
```

```typescript
// using-types.ts
import type { User, UserRole } from './types-only';  // Только типы, не импортует значения
import { UserService } from './types-only';         // Только значения

// User и UserRole доступны только для типизации
const user: User = { id: 1, name: "Alice", email: "alice@example.com" };
const role: UserRole = 'admin';

// UserService - доступно значение
UserService.validateUser(user); // OK
```

### Условные импорты типов
```typescript
// conditional-imports.ts
import type { Platform } from './platform-types';

// Платформо-специфичный импорт
let PlatformService: any;

if (typeof window !== 'undefined') {
  // Клиентская версия
  const { BrowserService } = await import('./browser-service');
  PlatformService = BrowserService;
} else {
  // Серверная версия
  const { NodeService } = await import('./node-service');
  PlatformService = NodeService;
}
```

## Мета-данные модуля

### Meta-импорт
```typescript
// meta-info.ts
async function getModuleInfo() {
  // Получение информации о текущем модуле
  const meta = import.meta;
  
  console.log(meta.url); // URL текущего модуля
  
  // Получение информации о родительском модуле
  // (поддерживается не во всех средах)
  try {
    console.log(meta.scriptElement); // В браузерах
  } catch {
    console.log("Script element not available");
  }
}

// Использование в сборке
const isDevelopment = import.meta.env?.MODE === 'development';
if (isDevelopment) {
  console.log("Running in development mode");
}
```

## Практические паттерны

### Модульный API
```typescript
// api.ts
import { HttpClient } from './http-client';
import { AuthService } from './auth-service';

export interface ApiOptions {
  baseUrl: string;
  timeout?: number;
  headers?: Record<string, string>;
}

class ApiClient {
  private http: HttpClient;
  private auth: AuthService;
  
  constructor(private options: ApiOptions) {
    this.http = new HttpClient(options);
    this.auth = new AuthService();
  }
  
  async getUser(id: number) {
    const token = await this.auth.getToken();
    return this.http.get(`/users/${id}`, {
      headers: { 'Authorization': `Bearer ${token}` }
    });
  }
  
  async updateUser(id: number, data: Partial<User>) {
    const token = await this.auth.getToken();
    return this.http.put(`/users/${id}`, data, {
      headers: { 'Authorization': `Bearer ${token}` }
    });
  }
}

export default ApiClient;
export { User, UserRole } from './user-types';
```

### Модульный плагин
```typescript
// plugin-system.ts
export interface Plugin {
  name: string;
  version: string;
  init(api: PluginAPI): void;
  destroy?(): void;
}

export interface PluginAPI {
  registerCommand(name: string, handler: (...args: any[]) => any): void;
  registerHook(name: string, handler: (data: any) => any): void;
}

// plugin.ts
import type { Plugin, PluginAPI } from './plugin-system';

class LoggerPlugin implements Plugin {
  name = 'logger';
  version = '1.0.0';
  
  init(api: PluginAPI) {
    api.registerHook('log', (message) => {
      console.log(`[${new Date().toISOString()}] ${message}`);
      return message;
    });
  }
}

export default LoggerPlugin;
```

## Совместимость с разными системами сборки

### Webpack
```typescript
// webpack-friendly.ts
// Условная загрузка для разных целей
async function loadPlatformSpecific() {
  if (process.env.BUILD_TARGET === 'web') {
    return await import('./web-platform');
  } else if (process.env.BUILD_TARGET === 'node') {
    return await import('./node-platform');
  } else {
    return await import('./universal-platform');
  }
}
```

### Conditional exports (package.json)
```json
{
  "name": "my-module",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.cjs"
    },
    "./utils": {
      "import": "./dist/esm/utils.js", 
      "require": "./dist/cjs/utils.cjs"
    }
  }
}
```

## Лучшие практики

### 1. Используйте именованные экспорты для нескольких значений
```typescript
// Хорошо: понятные именованные экспорты
export function validateEmail(email: string): boolean { /* */ }
export function validatePassword(password: string): boolean { /* */ }
export const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
```

### 2. Используйте default экспорт для основного экспорта
```typescript
// Хорошо: один основной класс/функция
export default class MainService { /* */ }
export { SecondaryUtil } from './secondary';
```

### 3. Используйте type-only импорты когда возможно
```typescript
// Для типов используйте import type
import type { SomeType } from './types';
import { SomeValue } from './values'; // для значений
```

### 4. Избегайте циклических зависимостей
```typescript
// ПЛОХО: циклическая зависимость
// a.ts -> b.ts -> c.ts -> a.ts

// ЛУЧШЕ: извлеките общие зависимости
// common.ts <- a.ts
// common.ts <- b.ts  
// common.ts <- c.ts
```

## Совместимость

ES Modules в TypeScript имеют хорошую совместимость:

- **Node.js** (12.17+ с флагом, 14+ по умолчанию)
- **Browser** (современные браузеры)
- **Webpack/Rollup** (отличная поддержка)
- **TypeScript** (нативная поддержка с ES5+)

## Связь с другими концепциями
- [[exports]] - подробнее об экспортах
- [[module-resolution]] - как TypeScript разрешает модули
- [[declaration-files]] - типизация для JavaScript модулей
- [[advanced]] - продвинутые паттерны модулей