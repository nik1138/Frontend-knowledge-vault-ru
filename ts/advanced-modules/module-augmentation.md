# Расширение модулей в TypeScript

Расширение модулей (Module Augmentation) - это мощная возможность TypeScript, которая позволяет добавлять новую функциональность к существующим модулям без изменения их исходного кода. Это особенно полезно при работе с библиотеками сторонних разработчиков или при необходимости добавления типов к глобальным объектам.

## Содержание

1. [Основы расширения модулей](#основы-расширения-модулей)
2. [Глобальное расширение](#глобальное-расширение)
3. [Расширение внешних модулей](#расширение-внешних-модулей)
4. [Расширение встроенных типов](#расширение-встроенных-типов)
5. [Практические примеры](#практические-примеры)
6. [Ограничения и лучшие практики](#ограничения-и-лучшие-практики)
7. [Совместимость и проблемы](#совместимость-и-проблемы)

## Основы расширения модулей

Расширение модулей позволяет добавлять новые свойства, методы и типы к существующим модулям. Это достигается с помощью специальных синтаксических конструкций, которые сообщают TypeScript о дополнительной функциональности.

### Базовый синтаксис

```typescript
// Расширение существующего модуля
import { SomeModule } from 'some-library';

declare module 'some-library' {
  // Добавление новых свойств к модулю
  interface SomeModule {
    newMethod(): void;
  }
  
  // Добавление новых типов
  interface NewInterface {
    property: string;
  }
}

// Реализация новых методов
SomeModule.prototype.newMethod = function() {
  console.log('New method implementation');
};
```

### Когда использовать расширение модулей

1. **Добавление типов к библиотекам** - когда библиотека не имеет встроенных типов
2. **Расширение функциональности** - добавление методов к существующим классам
3. **Глобальные расширения** - добавление свойств к глобальным объектам
4. **Полифиллы** - добавление поддержки новых API в старые среды выполнения

## Глобальное расширение

Глобальное расширение позволяет добавлять новые типы и свойства к глобальному пространству имен.

### Расширение Window

```typescript
// global-augmentation.ts
declare global {
  interface Window {
    myCustomProperty: string;
    myCustomFunction(): void;
  }
  
  // Расширение Console
  interface Console {
    customLog(message: string): void;
  }
}

// Реализация
window.myCustomProperty = 'Hello World';

window.myCustomFunction = function() {
  console.log('Custom function called');
};

console.customLog = function(message: string) {
  console.log(`[CUSTOM] ${message}`);
};

// Использование
console.customLog(window.myCustomProperty);
window.myCustomFunction();
```

### Расширение глобальных типов

```typescript
// global-types.ts
declare global {
  // Расширение Array
  interface Array<T> {
    groupBy(key: keyof T): Record<string, T[]>;
    distinct(): T[];
  }
  
  // Добавление новых глобальных типов
  type UUID = string & { readonly brand: unique symbol };
  
  // Расширение String
  interface String {
    toCamelCase(): string;
    toKebabCase(): string;
  }
}

// Реализация методов
Array.prototype.groupBy = function<T>(this: T[], key: keyof T): Record<string, T[]> {
  return this.reduce((groups, item) => {
    const group = String(item[key]);
    if (!groups[group]) {
      groups[group] = [];
    }
    groups[group].push(item);
    return groups;
  }, {} as Record<string, T[]>);
};

Array.prototype.distinct = function<T>(this: T[]): T[] {
  return [...new Set(this)];
};

String.prototype.toCamelCase = function(this: string): string {
  return this.replace(/-([a-z])/g, (g) => g[1].toUpperCase());
};

String.prototype.toKebabCase = function(this: string): string {
  return this.replace(/[A-Z]/g, (match) => `-${match.toLowerCase()}`);
};

// Использование
const users = [
  { name: 'John', department: 'IT' },
  { name: 'Jane', department: 'HR' },
  { name: 'Bob', department: 'IT' }
];

const grouped = users.groupBy('department');
const distinctNames = users.map(u => u.name).distinct();
const camelCase = 'hello-world'.toCamelCase();
```

## Расширение внешних модулей

Расширение внешних модулей позволяет добавлять типы и функциональность к сторонним библиотекам.

### Расширение библиотеки Express

```typescript
// express-augmentation.ts
import { Request, Response, NextFunction } from 'express';

// Расширение модуля Express
declare module 'express' {
  interface Request {
    userId?: string;
    userRole?: string;
  }
  
  interface Response {
    success(data: any): this;
    error(message: string, code?: number): this;
  }
}

// Расширение Response
declare module 'express-serve-static-core' {
  interface Response {
    success(data: any): this;
    error(message: string, code?: number): this;
  }
}

// Реализация методов
export function extendExpressResponse() {
  // @ts-ignore
  Response.prototype.success = function(data: any) {
    return this.status(200).json({
      success: true,
      data
    });
  };
  
  // @ts-ignore
  Response.prototype.error = function(message: string, code: number = 400) {
    return this.status(code).json({
      success: false,
      error: message
    });
  };
}

// middleware.ts
import { Request, Response, NextFunction } from 'express';

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  // Теперь TypeScript знает о новых свойствах
  req.userId = 'user123';
  req.userRole = 'admin';
  next();
}

// controller.ts
import { Request, Response } from 'express';

export function getUsers(req: Request, res: Response) {
  // TypeScript знает о расширенных свойствах
  if (req.userRole !== 'admin') {
    return res.error('Access denied', 403);
  }
  
  // TypeScript знает о новых методах
  res.success([
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
  ]);
}
```

### Расширение библиотеки Lodash

```typescript
// lodash-augmentation.ts
import * as _ from 'lodash';

// Расширение модуля Lodash
declare module 'lodash' {
  interface LoDashStatic {
    // Добавление нового метода
    deepClone<T>(value: T): T;
    
    // Расширение существующего метода
    map<T, TResult>(
      collection: T[] | null | undefined,
      iteratee: (value: T, index: number, collection: T[]) => TResult
    ): TResult[];
    
    // Добавление нового типа
    isUUID(value: any): value is string;
  }
}

// Реализация новых методов
_.mixin({
  deepClone<T>(value: T): T {
    if (value === null || typeof value !== 'object') {
      return value;
    }
    
    if (Array.isArray(value)) {
      return value.map(item => _.deepClone(item)) as any;
    }
    
    const cloned: any = {};
    for (const key in value) {
      if (value.hasOwnProperty(key)) {
        cloned[key] = _.deepClone(value[key]);
      }
    }
    return cloned;
  },
  
  isUUID(value: any): value is string {
    if (typeof value !== 'string') return false;
    const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
    return uuidRegex.test(value);
  }
});

// Использование
const original = { 
  name: 'John', 
  details: { 
    age: 30, 
    hobbies: ['reading', 'coding'] 
  } 
};

const cloned = _.deepClone(original);
const isUuid = _.isUUID('550e8400-e29b-41d4-a716-446655440000');
```

## Расширение встроенных типов

TypeScript позволяет расширять встроенные типы JavaScript, добавляя к ним новые методы и свойства.

### Расширение Array

```typescript
// array-augmentation.ts
declare global {
  interface Array<T> {
    // Статистические методы
    sum(this: number[]): number;
    average(this: number[]): number;
    
    // Функциональные методы
    tap(callback: (item: T, index: number, array: T[]) => void): T[];
    
    // Утилиты
    chunk(size: number): T[][];
    groupBy<K extends keyof T>(key: K): Record<T[K] extends string | number | symbol ? T[K] : string, T[]>;
    
    // Асинхронные методы
    asyncMap<U>(callback: (item: T, index: number, array: T[]) => Promise<U>): Promise<U[]>;
  }
}

// Реализация методов
Array.prototype.sum = function(this: number[]): number {
  return this.reduce((acc, val) => acc + val, 0);
};

Array.prototype.average = function(this: number[]): number {
  return this.length > 0 ? this.sum() / this.length : 0;
};

Array.prototype.tap = function<T>(
  this: T[], 
  callback: (item: T, index: number, array: T[]) => void
): T[] {
  this.forEach((item, index) => callback(item, index, this));
  return this;
};

Array.prototype.chunk = function<T>(this: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < this.length; i += size) {
    chunks.push(this.slice(i, i + size));
  }
  return chunks;
};

Array.prototype.groupBy = function<T, K extends keyof T>(
  this: T[], 
  key: K
): Record<T[K] extends string | number | symbol ? T[K] : string, T[]> {
  return this.reduce((groups, item) => {
    const groupKey = String(item[key]) as any;
    if (!groups[groupKey]) {
      groups[groupKey] = [];
    }
    groups[groupKey].push(item);
    return groups;
  }, {} as Record<any, T[]>);
};

Array.prototype.asyncMap = async function<T, U>(
  this: T[],
  callback: (item: T, index: number, array: T[]) => Promise<U>
): Promise<U[]> {
  const results: U[] = [];
  for (let i = 0; i < this.length; i++) {
    results.push(await callback(this[i], i, this));
  }
  return results;
};

// Использование
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.sum();
const avg = numbers.average();

const users = [
  { name: 'John', department: 'IT' },
  { name: 'Jane', department: 'HR' },
  { name: 'Bob', department: 'IT' }
];

const grouped = users.groupBy('department');
const chunks = users.chunk(2);

users.tap(user => console.log(user.name));

// Асинхронное использование
const asyncResults = await [1, 2, 3].asyncMap(async (num) => {
  await new Promise(resolve => setTimeout(resolve, 100));
  return num * 2;
});
```

### Расширение String

```typescript
// string-augmentation.ts
declare global {
  interface String {
    // Форматирование
    capitalize(): string;
    titleCase(): string;
    
    // Валидация
    isEmail(): boolean;
    isUrl(): boolean;
    isUUID(): boolean;
    
    // Преобразования
    toCamelCase(): string;
    toKebabCase(): string;
    toSnakeCase(): string;
    
    // Утилиты
    truncate(length: number, suffix?: string): string;
    words(): string[];
  }
}

// Реализация методов
String.prototype.capitalize = function(this: string): string {
  return this.charAt(0).toUpperCase() + this.slice(1).toLowerCase();
};

String.prototype.titleCase = function(this: string): string {
  return this.split(' ')
    .map(word => word.capitalize())
    .join(' ');
};

String.prototype.isEmail = function(this: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(this);
};

String.prototype.isUrl = function(this: string): boolean {
  try {
    new URL(this);
    return true;
  } catch {
    return false;
  }
};

String.prototype.isUUID = function(this: string): boolean {
  const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
  return uuidRegex.test(this);
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

String.prototype.toSnakeCase = function(this: string): string {
  return this.replace(/[A-Z]/g, match => `_${match.toLowerCase()}`)
    .replace(/[^a-z0-9_]/g, '_')
    .replace(/^_+|_+$/g, '')
    .replace(/_+/g, '_');
};

String.prototype.truncate = function(this: string, length: number, suffix: string = '...'): string {
  return this.length <= length ? this : this.substring(0, length) + suffix;
};

String.prototype.words = function(this: string): string[] {
  return this.match(/\b\w+\b/g) || [];
};

// Использование
const email = 'user@example.com';
console.log(email.isEmail()); // true

const text = 'hello world';
console.log(text.titleCase()); // Hello World

const snake = 'hello_world';
console.log(snake.toCamelCase()); // helloWorld

const longText = 'This is a very long text that needs to be truncated';
console.log(longText.truncate(20)); // This is a very long...

const sentence = 'The quick brown fox';
console.log(sentence.words()); // ['The', 'quick', 'brown', 'fox']
```

## Практические примеры

### 1. Расширение библиотеки Moment.js

```typescript
// moment-augmentation.ts
import moment from 'moment';

declare module 'moment' {
  interface Moment {
    // Добавление пользовательских форматов
    toISOStringWithTimezone(): string;
    
    // Бизнес-логика
    isWeekday(): boolean;
    isWeekend(): boolean;
    
    // Утилиты
    nextWeekday(): Moment;
    previousWeekday(): Moment;
  }
}

// Расширение прототипа Moment
moment.fn.toISOStringWithTimezone = function(this: moment.Moment): string {
  return this.format('YYYY-MM-DDTHH:mm:ssZ');
};

moment.fn.isWeekday = function(this: moment.Moment): boolean {
  const day = this.day();
  return day > 0 && day < 6; // Понедельник-пятница
};

moment.fn.isWeekend = function(this: moment.Moment): boolean {
  return !this.isWeekday();
};

moment.fn.nextWeekday = function(this: moment.Moment): moment.Moment {
  let next = this.clone().add(1, 'day');
  while (!next.isWeekday()) {
    next = next.add(1, 'day');
  }
  return next;
};

moment.fn.previousWeekday = function(this: moment.Moment): moment.Moment {
  let prev = this.clone().subtract(1, 'day');
  while (!prev.isWeekday()) {
    prev = prev.subtract(1, 'day');
  }
  return prev;
};

// Использование
const now = moment();
console.log(now.toISOStringWithTimezone());
console.log(now.isWeekday());
console.log(now.nextWeekday().format('YYYY-MM-DD'));
```

### 2. Расширение Node.js модулей

```typescript
// node-augmentation.ts
declare module 'fs' {
  interface ReadFileOptions {
    // Добавление новых опций
    encoding?: BufferEncoding | 'auto';
    autoDetectEncoding?: boolean;
  }
}

declare module 'http' {
  interface IncomingMessage {
    // Добавление пользовательских свойств
    clientIP: string;
    userAgent: string;
    isMobile: boolean;
  }
  
  interface ServerResponse {
    // Добавление пользовательских методов
    json(data: any, statusCode?: number): void;
    redirect(url: string, statusCode?: number): void;
  }
}

// Реализация методов
import { ServerResponse } from 'http';

// @ts-ignore
ServerResponse.prototype.json = function(data: any, statusCode: number = 200) {
  this.statusCode = statusCode;
  this.setHeader('Content-Type', 'application/json');
  this.end(JSON.stringify(data));
};

// @ts-ignore
ServerResponse.prototype.redirect = function(url: string, statusCode: number = 302) {
  this.statusCode = statusCode;
  this.setHeader('Location', url);
  this.end();
};

// middleware.ts
import { IncomingMessage, ServerResponse } from 'http';

export function enhanceRequest(req: IncomingMessage) {
  // @ts-ignore
  req.clientIP = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
  // @ts-ignore
  req.userAgent = req.headers['user-agent'] || '';
  // @ts-ignore
  req.isMobile = /mobile/i.test(req.userAgent);
}
```

### 3. Расширение пользовательских модулей

```typescript
// calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  multiply(a: number, b: number): number {
    return a * b;
  }
}

// calculator-augmentation.ts
import { Calculator } from './calculator';

declare module './calculator' {
  interface Calculator {
    // Расширение функциональности
    subtract(a: number, b: number): number;
    divide(a: number, b: number): number;
    
    // Комплексные операции
    power(base: number, exponent: number): number;
    factorial(n: number): number;
    
    // Утилиты
    chain(initialValue: number): CalculationChain;
  }
  
  interface CalculationChain {
    add(value: number): CalculationChain;
    subtract(value: number): CalculationChain;
    multiply(value: number): CalculationChain;
    divide(value: number): CalculationChain;
    result(): number;
  }
}

// Реализация новых методов
Calculator.prototype.subtract = function(a: number, b: number): number {
  return a - b;
};

Calculator.prototype.divide = function(a: number, b: number): number {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
};

Calculator.prototype.power = function(base: number, exponent: number): number {
  return Math.pow(base, exponent);
};

Calculator.prototype.factorial = function(n: number): number {
  if (n < 0) {
    throw new Error('Factorial of negative number is not defined');
  }
  if (n === 0 || n === 1) {
    return 1;
  }
  return n * this.factorial(n - 1);
};

Calculator.prototype.chain = function(initialValue: number): any {
  let result = initialValue;
  const calculator = this;
  
  return {
    add(value: number) {
      result = calculator.add(result, value);
      return this;
    },
    
    subtract(value: number) {
      result = calculator.subtract(result, value);
      return this;
    },
    
    multiply(value: number) {
      result = calculator.multiply(result, value);
      return this;
    },
    
    divide(value: number) {
      result = calculator.divide(result, value);
      return this;
    },
    
    result() {
      return result;
    }
  };
};

// Использование
const calc = new Calculator();
const result = calc.chain(10)
  .add(5)
  .multiply(2)
  .subtract(3)
  .result(); // 27

console.log(calc.factorial(5)); // 120
console.log(calc.power(2, 3)); // 8
```

## Ограничения и лучшие практики

### Ограничения

1. **Время компиляции** - Расширения модулей должны быть импортированы для вступления в силу
2. **Конфликты имен** - Возможны конфликты с существующими свойствами
3. **Поддержка типов** - Не все библиотеки поддерживают расширения типов
4. **Совместимость** - Расширения могут сломаться при обновлении библиотек

### Лучшие практики

#### 1. Явное импортирование

```typescript
// Плохо - расширение не импортируется
// main.ts
import { SomeClass } from 'some-library';
// someClass.newMethod(); // Ошибка типов

// Хорошо - явное импортирование расширения
// main.ts
import { SomeClass } from 'some-library';
import 'some-library/augmentation'; // Импорт расширения
// someClass.newMethod(); // Работает корректно
```

#### 2. Организация расширений

```typescript
// Плохо - все расширения в одном файле
// extensions.ts - содержит все расширения

// Хорошо - модульная организация
// extensions/
//   array.ts
//   string.ts
//   express.ts
//   lodash.ts
//   index.ts - реэкспорт всех расширений

// extensions/index.ts
import './array';
import './string';
import './express';
import './lodash';

// main.ts
import 'extensions'; // Импорт всех расширений
```

#### 3. Безопасные расширения

```typescript
// Проверка существования метода перед расширением
declare global {
  interface Array<T> {
    safeMap?: <U>(callback: (item: T, index: number, array: T[]) => U) => U[];
  }
}

// Безопасная реализация
if (!Array.prototype.safeMap) {
  Array.prototype.safeMap = function<T, U>(
    this: T[], 
    callback: (item: T, index: number, array: T[]) => U
  ): U[] {
    try {
      return this.map(callback);
    } catch (error) {
      console.error('Error in safeMap:', error);
      return [];
    }
  };
}
```

#### 4. Документирование расширений

```typescript
// math-extensions.ts
/**
 * Расширение встроенных математических функций
 * 
 * @module MathExtensions
 * @description Добавляет полезные математические методы к глобальному объекту Math
 */

declare global {
  interface Math {
    /**
     * Вычисляет факториал числа
     * @param n - Число для вычисления факториала
     * @returns Факториал числа n
     * @throws {Error} Если n отрицательное
     */
    factorial(n: number): number;
    
    /**
     * Вычисляет наибольший общий делитель двух чисел
     * @param a - Первое число
     * @param b - Второе число
     * @returns Наибольший общий делитель
     */
    gcd(a: number, b: number): number;
  }
}

Math.factorial = function(n: number): number {
  if (n < 0) throw new Error('Factorial of negative number');
  if (n === 0 || n === 1) return 1;
  return n * Math.factorial(n - 1);
};

Math.gcd = function(a: number, b: number): number {
  a = Math.abs(a);
  b = Math.abs(b);
  while (b) {
    [a, b] = [b, a % b];
  }
  return a;
};
```

## Совместимость и проблемы

### 1. Проблемы с обновлениями

```typescript
// Версия 1.0 библиотеки
declare module 'some-library' {
  interface Config {
    apiUrl: string;
  }
}

// Версия 2.0 библиотеки - изменилась структура
// Нужно обновить расширение
declare module 'some-library' {
  interface Config {
    api: {
      url: string;
      timeout: number;
    };
  }
}
```

### 2. Конфликты с другими расширениями

```typescript
// Расширение 1
declare module 'express' {
  interface Request {
    user: { id: string };
  }
}

// Расширение 2
declare module 'express' {
  interface Request {
    user: { name: string }; // Конфликт типов!
  }
}

// Решение - использовать объединение типов
declare module 'express' {
  interface Request {
    user: { id: string } | { name: string } | { id: string; name: string };
  }
}
```

### 3. Проблемы с tree-shaking

```typescript
// Расширение может предотвратить tree-shaking
import 'lodash/augmentation'; // Импортирует все расширения

// Лучше - импортировать только нужные расширения
import 'lodash/extensions/array';
import 'lodash/extensions/object';
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/namespace-merging|Слияние пространств имен]] - Похожий механизм объединения объявлений
- [[ts/type-system/interfaces|Интерфейсы]] - Основа для расширения типов
- [[ts/type-system/declaration-merging|Слияние объявлений]] - Концепция, лежащая в основе расширения модулей
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Инструменты для создания расширений

## Рекомендации по изучению

1. Практикуйтесь в расширении встроенных типов (Array, String)
2. Освойте расширение внешних библиотек
3. Изучите глобальные расширения и их ограничения
4. Практикуйтесь в создании безопасных расширений
5. Освойте организацию расширений в проекте
6. Изучите инструменты для управления расширениями
7. Практикуйтесь в документировании расширений

## Следующие шаги

- [[ts/advanced-modules/namespace-merging|Слияние пространств имен]] - Объединение объявлений
- [[ts/advanced-modules/ambient-modules|Внешние модули]] - Работа с внешними библиотеками
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей
- [[ts/type-system/declaration-merging|Слияние объявлений]] - Концепция, лежащая в основе расширения модулей