# Слияние пространств имен в TypeScript

Слияние пространств имен (Namespace Merging) - это мощная возможность TypeScript, которая позволяет объединять несколько объявлений с одинаковым именем в одно пространство имен. Это позволяет организовывать код в логические группы и расширять функциональность без изменения исходного кода.

## Содержание

1. [Основы слияния пространств имен](#основы-слияния-пространств-имен)
2. [Типы слияний](#типы-слияний)
3. [Практические примеры](#практические-примеры)
4. [Слияние с классами и функциями](#слияние-с-классами-и-функциями)
5. [Глобальные расширения](#глобальные-расширения)
6. [Ограничения и лучшие практики](#ограничения-и-лучшие-практики)
7. [Совместимость и проблемы](#совместимость-и-проблемы)

## Основы слияния пространств имен

Слияние пространств имен происходит автоматически, когда TypeScript встречает несколько объявлений с одинаковым именем. Все объявления объединяются в одно пространство имен, содержащее все свойства и методы из всех объявлений.

### Базовый пример

```typescript
// math.ts - Первое объявление
namespace MathUtils {
  export function add(a: number, b: number): number {
    return a + b;
  }
  
  export const PI = 3.14159;
}

// math.ts - Второе объявление (в том же файле или другом)
namespace MathUtils {
  export function multiply(a: number, b: number): number {
    return a * b;
  }
  
  export function circleArea(radius: number): number {
    return PI * radius * radius; // Доступ к PI из первого объявления
  }
}

// Использование
const sum = MathUtils.add(2, 3);
const product = MathUtils.multiply(4, 5);
const area = MathUtils.circleArea(10);
```

### Как работает слияние

TypeScript объединяет все экспортируемые члены в одно пространство имен:

```typescript
// Объявление 1
namespace MyNamespace {
  export interface User {
    name: string;
  }
  
  export function createUser(name: string): User {
    return { name };
  }
}

// Объявление 2
namespace MyNamespace {
  export interface Product {
    title: string;
    price: number;
  }
  
  export function createProduct(title: string, price: number): Product {
    return { title, price };
  }
}

// Результат - одно пространство имен с обоими интерфейсами и функциями
const user: MyNamespace.User = MyNamespace.createUser('John');
const product: MyNamespace.Product = MyNamespace.createProduct('Book', 29.99);
```

## Типы слияний

### 1. Слияние интерфейсов

```typescript
// interfaces.ts
namespace API {
  export interface User {
    id: string;
    name: string;
  }
}

namespace API {
  export interface User {
    email: string;
    createdAt: Date;
  }
}

// Результат: User содержит все свойства
const user: API.User = {
  id: '1',
  name: 'John',
  email: 'john@example.com',
  createdAt: new Date()
};
```

### 2. Слияние типов и значений

```typescript
// types-and-values.ts
namespace Config {
  export type Environment = 'development' | 'production' | 'test';
}

namespace Config {
  export const environment: Config.Environment = 'development';
  
  export function isDevelopment(): boolean {
    return environment === 'development';
  }
}

// Использование
const env: Config.Environment = Config.environment;
const isDev = Config.isDevelopment();
```

### 3. Слияние функций

```typescript
// function-overloads.ts
namespace MathLib {
  export function calculate(a: number, b: number): number;
  export function calculate(a: string, b: string): string;
  export function calculate(a: any, b: any): any {
    if (typeof a === 'number' && typeof b === 'number') {
      return a + b;
    }
    if (typeof a === 'string' && typeof b === 'string') {
      return a + b;
    }
    throw new Error('Unsupported types');
  }
}

namespace MathLib {
  export function calculate(a: number[]): number;
  export function calculate(a: number[]): number {
    return a.reduce((sum, val) => sum + val, 0);
  }
}

// Использование
const result1 = MathLib.calculate(2, 3); // 5
const result2 = MathLib.calculate('Hello', ' World'); // Hello World
const result3 = MathLib.calculate([1, 2, 3, 4]); // 10
```

## Практические примеры

### 1. Организация библиотеки

```typescript
// library-core.ts
namespace MyLibrary {
  export interface Options {
    debug?: boolean;
    timeout?: number;
  }
  
  export class Core {
    constructor(private options: Options = {}) {}
    
    getOptions(): Options {
      return { ...this.options };
    }
  }
}

// library-utils.ts
namespace MyLibrary {
  export namespace Utils {
    export function formatDate(date: Date): string {
      return date.toISOString();
    }
    
    export function generateId(): string {
      return Math.random().toString(36).substr(2, 9);
    }
  }
}

// library-http.ts
namespace MyLibrary {
  export class HttpClient {
    constructor(private baseUrl: string) {}
    
    async get<T>(url: string): Promise<T> {
      const response = await fetch(`${this.baseUrl}${url}`);
      return response.json();
    }
  }
}

// library-extensions.ts
namespace MyLibrary {
  export interface Core {
    // Расширение функциональности Core
    log(message: string): void;
  }
}

// Реализация расширения
MyLibrary.Core.prototype.log = function(message: string): void {
  if (this.getOptions().debug) {
    console.log(`[MyLibrary] ${message}`);
  }
};

// Использование
const core = new MyLibrary.Core({ debug: true });
core.log('Library initialized');

const utils = MyLibrary.Utils;
const id = utils.generateId();
const date = utils.formatDate(new Date());

const http = new MyLibrary.HttpClient('https://api.example.com');
```

### 2. Модульная архитектура приложения

```typescript
// app-core.ts
namespace App {
  export interface Config {
    appName: string;
    version: string;
  }
  
  let config: Config;
  
  export function init(appConfig: Config): void {
    config = appConfig;
    console.log(`App ${config.appName} v${config.version} initialized`);
  }
  
  export function getConfig(): Config {
    return { ...config };
  }
}

// app-services.ts
namespace App {
  export namespace Services {
    export class UserService {
      async getCurrentUser(): Promise<any> {
        // Логика получения текущего пользователя
        return { id: '1', name: 'John Doe' };
      }
    }
    
    export class ProductService {
      async getProducts(): Promise<any[]> {
        // Логика получения продуктов
        return [
          { id: '1', name: 'Product 1' },
          { id: '2', name: 'Product 2' }
        ];
      }
    }
  }
}

// app-components.ts
namespace App {
  export namespace Components {
    export class Header {
      render(): string {
        return `<header>${App.getConfig().appName}</header>`;
      }
    }
    
    export class Footer {
      render(): string {
        return `<footer>Version ${App.getConfig().version}</footer>`;
      }
    }
  }
}

// app-extensions.ts
namespace App {
  export interface Config {
    // Расширение конфигурации
    theme?: 'light' | 'dark';
    language?: string;
  }
  
  export namespace Extensions {
    export function enableDarkMode(): void {
      document.body.classList.add('dark-mode');
    }
    
    export function setLanguage(lang: string): void {
      // Логика установки языка
      console.log(`Language set to ${lang}`);
    }
  }
}

// Использование
App.init({
  appName: 'My App',
  version: '1.0.0',
  theme: 'dark',
  language: 'en'
});

const userService = new App.Services.UserService();
const productService = new App.Services.ProductService();

const header = new App.Components.Header();
const footer = new App.Components.Footer();

if (App.getConfig().theme === 'dark') {
  App.Extensions.enableDarkMode();
}
```

### 3. Плагинная система

```typescript
// plugin-system.ts
namespace PluginSystem {
  export interface Plugin {
    name: string;
    version: string;
    initialize(): Promise<void>;
    execute(data: any): any;
  }
  
  const plugins: Map<string, Plugin> = new Map();
  
  export async function registerPlugin(plugin: Plugin): Promise<void> {
    await plugin.initialize();
    plugins.set(plugin.name, plugin);
    console.log(`Plugin ${plugin.name} v${plugin.version} registered`);
  }
  
  export function executePlugin(name: string, data: any): any {
    const plugin = plugins.get(name);
    if (plugin) {
      return plugin.execute(data);
    }
    throw new Error(`Plugin ${name} not found`);
  }
}

// plugin-registry.ts
namespace PluginSystem {
  export namespace Registry {
    const pluginFactories: Map<string, () => Plugin> = new Map();
    
    export function registerFactory(name: string, factory: () => Plugin): void {
      pluginFactories.set(name, factory);
    }
    
    export async function loadPlugin(name: string): Promise<void> {
      const factory = pluginFactories.get(name);
      if (factory) {
        const plugin = factory();
        await registerPlugin(plugin);
      } else {
        throw new Error(`Plugin factory ${name} not found`);
      }
    }
  }
}

// plugin-loader.ts
namespace PluginSystem {
  export namespace Loader {
    export async function loadFromModule(modulePath: string): Promise<void> {
      try {
        const module = await import(modulePath);
        const plugin: Plugin = new module.default();
        await registerPlugin(plugin);
      } catch (error) {
        console.error(`Failed to load plugin from ${modulePath}:`, error);
        throw error;
      }
    }
    
    export async function loadAllPlugins(pluginPaths: string[]): Promise<void> {
      const promises = pluginPaths.map(path => loadFromModule(path));
      await Promise.all(promises);
    }
  }
}

// example-plugin.ts
class ExamplePlugin implements PluginSystem.Plugin {
  name = 'example-plugin';
  version = '1.0.0';
  
  async initialize(): Promise<void> {
    console.log('Example plugin initialized');
  }
  
  execute(data: any): any {
    return {
      ...data,
      processed: true,
      plugin: this.name
    };
  }
}

// Регистрация плагина
PluginSystem.Registry.registerFactory('example', () => new ExamplePlugin());

// Использование
async function main() {
  await PluginSystem.Registry.loadPlugin('example');
  const result = PluginSystem.executePlugin('example', { data: 'test' });
  console.log(result);
}
```

## Слияние с классами и функциями

TypeScript позволяет сливать пространства имен с классами и функциями, создавая мощные API.

### 1. Слияние с классом

```typescript
// class-with-namespace.ts
class Database {
  private connectionString: string;
  
  constructor(connectionString: string) {
    this.connectionString = connectionString;
  }
  
  connect(): void {
    console.log(`Connecting to ${this.connectionString}`);
  }
  
  query(sql: string): any[] {
    // Логика выполнения запроса
    return [];
  }
}

namespace Database {
  export interface ConnectionOptions {
    host: string;
    port: number;
    database: string;
    username: string;
    password: string;
  }
  
  export function createConnection(options: ConnectionOptions): Database {
    const connectionString = `postgresql://${options.username}:${options.password}@${options.host}:${options.port}/${options.database}`;
    return new Database(connectionString);
  }
  
  export namespace Utils {
    export function escapeString(str: string): string {
      return str.replace(/'/g, "''");
    }
    
    export function formatDate(date: Date): string {
      return date.toISOString().slice(0, 19).replace('T', ' ');
    }
  }
}

// Использование
const db = Database.createConnection({
  host: 'localhost',
  port: 5432,
  database: 'myapp',
  username: 'user',
  password: 'pass'
});

db.connect();
const escaped = Database.Utils.escapeString("O'Reilly");
```

### 2. Слияние с функцией

```typescript
// function-with-namespace.ts
function math(a: number, b: number): number {
  return a + b;
}

namespace math {
  export function add(a: number, b: number): number {
    return a + b;
  }
  
  export function subtract(a: number, b: number): number {
    return a - b;
  }
  
  export function multiply(a: number, b: number): number {
    return a * b;
  }
  
  export function divide(a: number, b: number): number {
    if (b === 0) throw new Error('Division by zero');
    return a / b;
  }
  
  export namespace Constants {
    export const PI = 3.14159;
    export const E = 2.71828;
  }
  
  export namespace Trigonometry {
    export function sin(angle: number): number {
      return Math.sin(angle);
    }
    
    export function cos(angle: number): number {
      return Math.cos(angle);
    }
    
    export function tan(angle: number): number {
      return Math.tan(angle);
    }
  }
}

// Использование
const result1 = math(2, 3); // 5 (основная функция)
const result2 = math.add(2, 3); // 5
const result3 = math.multiply(4, 5); // 20
const pi = math.Constants.PI; // 3.14159
const sine = math.Trigonometry.sin(Math.PI / 2); // 1
```

### 3. Слияние с перечислением

```typescript
// enum-with-namespace.ts
enum HttpStatus {
  OK = 200,
  NotFound = 404,
  InternalServerError = 500
}

namespace HttpStatus {
  export function getMessage(code: HttpStatus): string {
    switch (code) {
      case HttpStatus.OK:
        return 'OK';
      case HttpStatus.NotFound:
        return 'Not Found';
      case HttpStatus.InternalServerError:
        return 'Internal Server Error';
      default:
        return 'Unknown';
    }
  }
  
  export function isSuccessful(code: HttpStatus): boolean {
    return code >= 200 && code < 300;
  }
  
  export function isClientError(code: HttpStatus): boolean {
    return code >= 400 && code < 500;
  }
  
  export function isServerError(code: HttpStatus): boolean {
    return code >= 500 && code < 600;
  }
}

// Использование
const status = HttpStatus.OK;
const message = HttpStatus.getMessage(status);
const isOk = HttpStatus.isSuccessful(status);
const isClientErr = HttpStatus.isClientError(HttpStatus.NotFound);
```

## Глобальные расширения

Глобальные расширения позволяют добавлять функциональность к глобальному пространству имен.

### 1. Расширение глобального объекта

```typescript
// global-extensions.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      // Добавление пользовательских переменных окружения
      DATABASE_URL: string;
      API_KEY: string;
      NODE_ENV: 'development' | 'production' | 'test';
    }
  }
  
  interface Console {
    // Добавление пользовательских методов
    success(message: string): void;
    warn(message: string): void;
    debug(message: string): void;
  }
}

// Реализация методов
console.success = function(message: string): void {
  console.log(`✅ ${message}`);
};

console.warn = function(message: string): void {
  console.log(`⚠️  ${message}`);
};

console.debug = function(message: string): void {
  if (process.env.NODE_ENV === 'development') {
    console.log(`🐛 ${message}`);
  }
};

// Использование
console.success('Operation completed successfully');
console.warn('This is a warning');
console.debug('Debug information');
```

### 2. Расширение встроенных типов

```typescript
// built-in-extensions.ts
declare global {
  interface Array<T> {
    // Добавление пользовательских методов к Array
    groupBy<K extends keyof T>(key: K): Record<string, T[]>;
    distinct(): T[];
    shuffle(): T[];
  }
  
  interface String {
    // Добавление пользовательских методов к String
    toCamelCase(): string;
    toKebabCase(): string;
    capitalize(): string;
  }
  
  interface Number {
    // Добавление пользовательских методов к Number
    toCurrency(currency?: string): string;
    format(decimals?: number): string;
  }
}

// Реализация методов
Array.prototype.groupBy = function<T, K extends keyof T>(
  this: T[], 
  key: K
): Record<string, T[]> {
  return this.reduce((groups, item) => {
    const groupKey = String(item[key]);
    if (!groups[groupKey]) {
      groups[groupKey] = [];
    }
    groups[groupKey].push(item);
    return groups;
  }, {} as Record<string, T[]>);
};

Array.prototype.distinct = function<T>(this: T[]): T[] {
  return [...new Set(this)];
};

Array.prototype.shuffle = function<T>(this: T[]): T[] {
  const array = [...this];
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
};

String.prototype.toCamelCase = function(this: string): string {
  return this.replace(/[-_\s]+(.)?/g, (_, c) => c ? c.toUpperCase() : '');
};

String.prototype.toKebabCase = function(this: string): string {
  return this.replace(/[A-Z]/g, match => `-${match.toLowerCase()}`)
    .replace(/[^a-z0-9-]/g, '-')
    .replace(/^-+|-+$/g, '')
    .replace(/-+/g, '-');
};

String.prototype.capitalize = function(this: string): string {
  return this.charAt(0).toUpperCase() + this.slice(1).toLowerCase();
};

Number.prototype.toCurrency = function(this: number, currency: string = 'USD'): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(this);
};

Number.prototype.format = function(this: number, decimals: number = 2): string {
  return this.toFixed(decimals);
};

// Использование
const users = [
  { name: 'John', department: 'IT' },
  { name: 'Jane', department: 'HR' },
  { name: 'Bob', department: 'IT' }
];

const grouped = users.groupBy('department');
const distinctNames = ['a', 'b', 'a', 'c'].distinct();
const shuffled = [1, 2, 3, 4, 5].shuffle();

const camelCase = 'hello-world'.toCamelCase();
const kebabCase = 'helloWorld'.toKebabCase();
const capitalized = 'hello world'.capitalize();

const price = 29.99;
const currency = price.toCurrency('EUR');
const formatted = price.format(2);
```

## Ограничения и лучшие практики

### Ограничения

1. **Время компиляции** - Все объявления должны быть видны компилятору
2. **Конфликты имен** - Возможны конфликты при слиянии
3. **Порядок объявления** - Порядок может влиять на результат слияния
4. **Поддержка типов** - Не все библиотеки поддерживают слияния

### Лучшие практики

#### 1. Организация кода

```typescript
// Плохо - все в одном файле
// math.ts
namespace MathUtils {
  export function add() { /* ... */ }
}
namespace MathUtils {
  export function multiply() { /* ... */ }
}

// Хорошо - модульная организация
// math/
//   core.ts
//   utils.ts
//   advanced.ts
//   index.ts

// math/core.ts
namespace MathUtils {
  export function add(a: number, b: number): number {
    return a + b;
  }
}

// math/utils.ts
namespace MathUtils {
  export function multiply(a: number, b: number): number {
    return a * b;
  }
}

// math/index.ts
import './core';
import './utils';
```

#### 2. Явное импортирование

```typescript
// Плохо - слияние не импортируется
// main.ts
// MathUtils.divide(); // Ошибка, если divide определен в другом файле

// Хорошо - явное импортирование
// main.ts
import 'math/divide-extension';
// MathUtils.divide(); // Работает корректно
```

#### 3. Безопасные расширения

```typescript
// Проверка существования перед расширением
namespace SafeExtensions {
  export function extendArray() {
    if (!Array.prototype.groupBy) {
      Array.prototype.groupBy = function<T, K extends keyof T>(
        this: T[], 
        key: K
      ): Record<string, T[]> {
        return this.reduce((groups, item) => {
          const groupKey = String(item[key]);
          if (!groups[groupKey]) {
            groups[groupKey] = [];
          }
          groups[groupKey].push(item);
          return groups;
        }, {} as Record<string, T[]>);
      };
    }
  }
}

// Вызов расширения
SafeExtensions.extendArray();
```

#### 4. Документирование слияний

```typescript
/**
 * Расширение пространства имен MathUtils
 * 
 * @namespace MathUtils.Advanced
 * @description Добавляет продвинутые математические функции
 */
namespace MathUtils {
  /**
   * Продвинутые математические функции
   */
  export namespace Advanced {
    /**
     * Вычисляет факториал числа
     * @param n - Число для вычисления факториала
     * @returns Факториал числа n
     * @throws {Error} Если n отрицательное
     */
    export function factorial(n: number): number {
      if (n < 0) throw new Error('Factorial of negative number');
      if (n === 0 || n === 1) return 1;
      return n * factorial(n - 1);
    }
    
    /**
     * Вычисляет наибольший общий делитель двух чисел
     * @param a - Первое число
     * @param b - Второе число
     * @returns Наибольший общий делитель
     */
    export function gcd(a: number, b: number): number {
      a = Math.abs(a);
      b = Math.abs(b);
      while (b) {
        [a, b] = [b, a % b];
      }
      return a;
    }
  }
}
```

## Совместимость и проблемы

### 1. Проблемы с модулями ES6

```typescript
// При использовании ES6 модулей слияния могут не работать как ожидается
// math.ts
export namespace MathUtils {
  export function add(a: number, b: number): number {
    return a + b;
  }
}

// extensions.ts
// Это не сольется с предыдущим объявлением!
export namespace MathUtils {
  export function multiply(a: number, b: number): number {
    return a * b;
  }
}

// Лучше использовать declare module
// extensions.ts
declare module './math' {
  namespace MathUtils {
    function multiply(a: number, b: number): number;
  }
}
```

### 2. Конфликты с другими библиотеками

```typescript
// Библиотека 1
namespace Utils {
  export function formatDate(date: Date): string {
    return date.toISOString();
  }
}

// Библиотека 2
namespace Utils {
  export function formatDate(date: Date): string {
    return date.toLocaleDateString(); // Разный формат!
  }
}

// Решение - использовать модули
// utils/date-formatter.ts
export function formatDate(date: Date): string {
  return date.toISOString();
}

// utils/localized-date-formatter.ts
export function formatDate(date: Date): string {
  return date.toLocaleDateString();
}
```

### 3. Проблемы с tree-shaking

```typescript
// Слияния могут предотвратить tree-shaking
namespace LargeLibrary {
  export function function1() { /* ... */ }
  export function function2() { /* ... */ }
  // ... много функций
}

namespace LargeLibrary {
  export function function3() { /* ... */ }
  export function function4() { /* ... */ }
  // ... еще функции
}

// Лучше - использовать отдельные модули
// module1.ts
export function function1() { /* ... */ }
export function function2() { /* ... */ }

// module2.ts
export function function3() { /* ... */ }
export function function4() { /* ... */ }
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/module-augmentation|Расширение модулей]] - Похожий механизм расширения функциональности
- [[ts/type-system/interfaces|Интерфейсы]] - Основа для слияния типов
- [[ts/type-system/declaration-merging|Слияние объявлений]] - Концепция, лежащая в основе слияния пространств имен
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Инструменты для создания расширений

## Рекомендации по изучению

1. Практикуйтесь в базовых слияниях пространств имен
2. Освойте слияние с классами и функциями
3. Изучите глобальные расширения и их ограничения
4. Практикуйтесь в создании безопасных расширений
5. Освойте организацию слияний в проекте
6. Изучите проблемы совместимости с ES6 модулями
7. Практикуйтесь в документировании слияний

## Следующие шаги

- [[ts/advanced-modules/ambient-modules|Внешние модули]] - Работа с внешними библиотеками
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей
- [[ts/type-system/declaration-merging|Слияние объявлений]] - Концепция, лежащая в основе слияния пространств имен
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Оптимизация для сборки