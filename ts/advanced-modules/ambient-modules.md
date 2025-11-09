# Внешние модули в TypeScript

Внешние модули (Ambient Modules) - это объявления типов для JavaScript-библиотек, которые не имеют встроенных типов TypeScript. Они позволяют использовать сторонние библиотеки с полной поддержкой типизации, автодополнения и проверки ошибок.

## Содержание

1. [Основы внешних модулей](#основы-внешних-модулей)
2. [Создание объявлений типов](#создание-объявлений-типов)
3. [Структура файлов типов](#структура-файлов-типов)
4. [Практические примеры](#практические-примеры)
5. [Расширенные возможности](#расширенные-возможности)
6. [Публикация типов](#публикация-типов)
7. [Лучшие практики](#лучшие-практики)

## Основы внешних модулей

Внешние модули позволяют TypeScript понимать структуру и типы JavaScript-библиотек, которые не были написаны на TypeScript. Это достигается с помощью специальных файлов объявлений (`.d.ts`), которые описывают форму библиотеки.

### Базовый синтаксис

```typescript
// Объявление внешнего модуля
declare module 'library-name' {
  export function someFunction(param: string): number;
  export const someConstant: string;
  
  export interface SomeInterface {
    property: string;
  }
  
  export default class SomeClass {
    constructor(options: SomeInterface);
    method(): void;
  }
}

// Использование
import SomeClass, { someFunction, someConstant, SomeInterface } from 'library-name';

const result = someFunction('test');
const instance = new SomeClass({ property: 'value' });
```

### Когда использовать внешние модули

1. **Сторонние библиотеки без типов** - когда библиотека не предоставляет встроенные типы
2. **Глобальные переменные** - для типизации глобальных объектов
3. **Полифиллы** - для добавления типов к новым API в старых средах
4. **Пользовательские глобальные модули** - для типизации собственных глобальных модулей

## Создание объявлений типов

### 1. Базовое объявление

```typescript
// types/my-library.d.ts
declare module 'my-library' {
  // Экспорт функций
  export function add(a: number, b: number): number;
  export function multiply(a: number, b: number): number;
  
  // Экспорт констант
  export const VERSION: string;
  export const PI: number;
  
  // Экспорт интерфейсов
  export interface CalculatorOptions {
    precision?: number;
    rounding?: 'up' | 'down' | 'nearest';
  }
  
  // Экспорт классов
  export class Calculator {
    constructor(options?: CalculatorOptions);
    add(a: number, b: number): number;
    multiply(a: number, b: number): number;
    setPrecision(precision: number): void;
  }
  
  // Экспорт по умолчанию
  export default Calculator;
}

// Использование
import Calculator, { add, multiply, VERSION, CalculatorOptions } from 'my-library';

const calc = new Calculator({ precision: 2 });
const sum = add(2, 3);
const product = multiply(4, 5);
```

### 2. Объявление с шаблонными строками

```typescript
// types/assets.d.ts
declare module '*.css' {
  const content: { [className: string]: string };
  export default content;
}

declare module '*.scss' {
  const content: { [className: string]: string };
  export default content;
}

declare module '*.sass' {
  const content: { [className: string]: string };
  export default content;
}

declare module '*.module.css' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.png' {
  const value: string;
  export default value;
}

declare module '*.jpg' {
  const value: string;
  export default value;
}

declare module '*.jpeg' {
  const value: string;
  export default value;
}

declare module '*.gif' {
  const value: string;
  export default value;
}

declare module '*.svg' {
  const content: any;
  export default content;
}

// Использование
import styles from './component.module.css';
import logo from './logo.png';

const className = styles.container;
const imageUrl = logo;
```

### 3. Объявление глобальных переменных

```typescript
// types/global.d.ts
declare global {
  // Глобальные переменные
  var __DEV__: boolean;
  var __VERSION__: string;
  
  // Расширение существующих интерфейсов
  interface Window {
    myCustomLibrary: any;
    analytics: {
      track(event: string, data?: any): void;
      page(pageName: string): void;
    };
  }
  
  interface Process {
    env: ProcessEnv;
  }
  
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
    API_URL: string;
    API_KEY: string;
  }
}

// Эти переменные теперь доступны глобально
if (__DEV__) {
  console.log('Development mode');
}

window.analytics.track('page_view', { page: 'home' });
```

## Структура файлов типов

### 1. Организация типов в проекте

```typescript
// types/
//   index.d.ts - основной файл типов
//   libraries/ - типы для сторонних библиотек
//     lodash.d.ts
//     express.d.ts
//     react.d.ts
//   custom/ - типы для собственных модулей
//     api.d.ts
//     models.d.ts
//     utils.d.ts
//   globals.d.ts - глобальные типы
//   assets.d.ts - типы для ресурсов

// types/index.d.ts
/// <reference path="./globals.d.ts" />
/// <reference path="./assets.d.ts" />
/// <reference path="./libraries/lodash.d.ts" />
/// <reference path="./libraries/express.d.ts" />
/// <reference path="./custom/api.d.ts" />
/// <reference path="./custom/models.d.ts" />
```

### 2. Файл типов для библиотеки

```typescript
// types/libraries/axios.d.ts
declare module 'axios' {
  export interface AxiosRequestConfig {
    url?: string;
    method?: Method;
    baseURL?: string;
    headers?: any;
    params?: any;
    data?: any;
    timeout?: number;
    withCredentials?: boolean;
  }
  
  export interface AxiosResponse<T = any> {
    data: T;
    status: number;
    statusText: string;
    headers: any;
    config: AxiosRequestConfig;
    request?: any;
  }
  
  export interface AxiosInstance {
    <T = any>(config: AxiosRequestConfig): Promise<AxiosResponse<T>>;
    get<T = any>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>;
    post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>;
    put<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>;
    delete<T = any>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>>;
  }
  
  export interface AxiosStatic extends AxiosInstance {
    create(config?: AxiosRequestConfig): AxiosInstance;
    CancelToken: CancelTokenStatic;
    isCancel(value: any): boolean;
  }
  
  const axios: AxiosStatic;
  export default axios;
}
```

### 3. Файл типов для собственного API

```typescript
// types/custom/api.d.ts
declare module 'api' {
  // Модели данных
  export interface User {
    id: string;
    name: string;
    email: string;
    createdAt: string;
  }
  
  export interface Product {
    id: string;
    name: string;
    price: number;
    description: string;
    category: string;
  }
  
  export interface Order {
    id: string;
    userId: string;
    products: Product[];
    total: number;
    status: 'pending' | 'processing' | 'shipped' | 'delivered';
    createdAt: string;
  }
  
  // Параметры запросов
  export interface PaginationParams {
    page?: number;
    limit?: number;
    sortBy?: string;
    sortOrder?: 'asc' | 'desc';
  }
  
  export interface UserFilter {
    name?: string;
    email?: string;
  }
  
  // Ответы API
  export interface ApiResponse<T> {
    data: T;
    success: boolean;
    message?: string;
  }
  
  export interface PaginatedResponse<T> {
    data: T[];
    pagination: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
  }
  
  // Клиент API
  export class ApiClient {
    constructor(baseURL: string, apiKey: string);
    
    // Пользователи
    getUsers(params?: PaginationParams & UserFilter): Promise<ApiResponse<PaginatedResponse<User>>>;
    getUser(id: string): Promise<ApiResponse<User>>;
    createUser(userData: Omit<User, 'id' | 'createdAt'>): Promise<ApiResponse<User>>;
    updateUser(id: string, userData: Partial<User>): Promise<ApiResponse<User>>;
    deleteUser(id: string): Promise<ApiResponse<void>>;
    
    // Продукты
    getProducts(params?: PaginationParams): Promise<ApiResponse<PaginatedResponse<Product>>>;
    getProduct(id: string): Promise<ApiResponse<Product>>;
    createProduct(productData: Omit<Product, 'id'>): Promise<ApiResponse<Product>>;
    updateProduct(id: string, productData: Partial<Product>): Promise<ApiResponse<Product>>;
    deleteProduct(id: string): Promise<ApiResponse<void>>;
    
    // Заказы
    getOrders(params?: PaginationParams): Promise<ApiResponse<PaginatedResponse<Order>>>;
    getOrder(id: string): Promise<ApiResponse<Order>>;
    createOrder(orderData: Omit<Order, 'id' | 'createdAt' | 'status'>): Promise<ApiResponse<Order>>;
    updateOrderStatus(id: string, status: Order['status']): Promise<ApiResponse<Order>>;
  }
  
  // Фабрика клиента
  export function createApiClient(baseURL: string, apiKey: string): ApiClient;
  
  // Экспорт по умолчанию
  export default ApiClient;
}
```

## Практические примеры

### 1. Типы для библиотеки jQuery

```typescript
// types/libraries/jquery.d.ts
declare module 'jquery' {
  // Глобальный $
  export = $;
  export as namespace $;
  
  interface JQueryStatic {
    (selector: string, context?: Element | JQuery): JQuery;
    (element: Element): JQuery;
    (elementArray: Element[]): JQuery;
    (object: {}): JQuery;
    (func: Function): JQuery;
    (html: string, ownerDocument?: Document): JQuery;
    (): JQuery;
    
    // Свойства
    fn: any;
    version: string;
    
    // Методы
    each<T>(collection: T[], callback: (indexInArray: number, valueOfElement: T) => any): any;
    each(collection: any, callback: (indexInArray: string, valueOfElement: any) => any): any;
    extend(target: any, ...objs: any[]): any;
    extend(deep: boolean, target: any, ...objs: any[]): any;
    grep<T>(array: T[], func: (elementOfArray: T, indexInArray: number) => boolean, invert?: boolean): T[];
    inArray<T>(value: T, array: T[], fromIndex?: number): number;
    isArray(obj: any): boolean;
    isEmptyObject(obj: any): boolean;
    isFunction(obj: any): boolean;
    isNumeric(value: any): boolean;
    isPlainObject(obj: any): boolean;
    isWindow(obj: any): boolean;
    isXMLDoc(node: Node): boolean;
    makeArray(obj: any): any[];
    map<T, U>(array: T[], callback: (elementOfArray: T, indexInArray: number) => U): U[];
    map(array: any, callback: (elementOfArray: any, indexInArray: any) => any): any;
    merge<T>(first: T[], second: T[]): T[];
    noop(): any;
    now(): number;
    parseHTML(data: string, context?: HTMLElement, keepScripts?: boolean): any[];
    parseJSON(json: string): any;
    parseXML(data: string): any;
    trim(str: string): string;
    type(obj: any): string;
    unique<T extends Element>(array: T[]): T[];
  }
  
  interface JQuery {
    // Атрибуты
    attr(attributeName: string): string;
    attr(attributeName: string, value: string | number): JQuery;
    attr(attributes: any): JQuery;
    attr(attributeName: string, func: (index: number, attr: string) => string | number): JQuery;
    
    // CSS
    css(propertyName: string): string;
    css(propertyNames: string[]): any;
    css(properties: any): JQuery;
    css(propertyName: string, value: string | number): JQuery;
    css(propertyName: string, value: (index: number, value: string) => string | number): JQuery;
    
    // Размеры
    height(): number;
    height(value: number | string): JQuery;
    height(func: (index: number, height: number) => number | string): JQuery;
    
    width(): number;
    width(value: number | string): JQuery;
    width(func: (index: number, width: number) => number | string): JQuery;
    
    // События
    on(events: string, handler: (eventObject: JQueryEventObject) => any): JQuery;
    on(events: string, data: any, handler: (eventObject: JQueryEventObject) => any): JQuery;
    on(events: string, selector: string, handler: (eventObject: JQueryEventObject) => any): JQuery;
    on(events: string, selector: string, data: any, handler: (eventObject: JQueryEventObject) => any): JQuery;
    
    off(): JQuery;
    off(events: string): JQuery;
    off(events: string, selector: string): JQuery;
    off(events: string, handler: (eventObject: JQueryEventObject) => any): JQuery;
    off(events: string, selector: string, handler: (eventObject: JQueryEventObject) => any): JQuery;
    
    // AJAX
    ajax(url: string, settings?: JQueryAjaxSettings): JQueryXHR;
    get(url: string, data?: any, success?: (data: any, textStatus: string, jqXHR: JQueryXHR) => any, dataType?: string): JQueryXHR;
    post(url: string, data?: any, success?: (data: any, textStatus: string, jqXHR: JQueryXHR) => any, dataType?: string): JQueryXHR;
  }
  
  interface JQueryEventObject extends Event {
    data: any;
    delegateTarget: Element;
    isDefaultPrevented(): boolean;
    isImmediatePropagationStopped(): boolean;
    isPropagationStopped(): boolean;
    namespace: string;
    originalEvent: Event;
    preventDefault(): any;
    relatedTarget: Element;
    result: any;
    stopImmediatePropagation(): void;
    stopPropagation(): void;
    pageX: number;
    pageY: number;
    which: number;
    metaKey: boolean;
  }
  
  interface JQueryXHR extends XMLHttpRequest, JQueryPromise<any> {
    overrideMimeType(mimeType: string): any;
    abort(statusText?: string): void;
  }
  
  interface JQueryAjaxSettings {
    accepts?: any;
    async?: boolean;
    beforeSend?(jqXHR: JQueryXHR, settings: JQueryAjaxSettings): any;
    cache?: boolean;
    complete?(jqXHR: JQueryXHR, textStatus: string): any;
    contents?: { [key: string]: any; };
    contentType?: any;
    context?: any;
    converters?: { [key: string]: any; };
    crossDomain?: boolean;
    data?: any;
    dataFilter?(data: any, type: string): any;
    dataType?: string;
    error?(jqXHR: JQueryXHR, textStatus: string, errorThrown: string): any;
    global?: boolean;
    headers?: { [key: string]: any; };
    ifModified?: boolean;
    isLocal?: boolean;
    jsonp?: any;
    jsonpCallback?: any;
    method?: string;
    mimeType?: string;
    password?: string;
    processData?: boolean;
    scriptCharset?: string;
    statusCode?: { [key: string]: any; };
    success?(data: any, textStatus: string, jqXHR: JQueryXHR): any;
    timeout?: number;
    traditional?: boolean;
    type?: string;
    url?: string;
    username?: string;
    xhr?: any;
    xhrFields?: { [key: string]: any; };
  }
  
  const $: JQueryStatic;
}

// Глобальный $
declare const $: JQueryStatic;
```

### 2. Типы для пользовательской библиотеки

```typescript
// types/libraries/my-custom-library.d.ts
declare module 'my-custom-library' {
  // Основные типы
  export type LogLevel = 'debug' | 'info' | 'warn' | 'error';
  
  export interface LoggerOptions {
    level?: LogLevel;
    format?: 'json' | 'text';
    transports?: Transport[];
  }
  
  export interface Transport {
    log(level: LogLevel, message: string, meta?: any): void;
  }
  
  export interface FileTransportOptions {
    filename: string;
    maxSize?: number;
    maxFiles?: number;
    level?: LogLevel;
  }
  
  // Классы
  export class Logger {
    constructor(options?: LoggerOptions);
    
    debug(message: string, meta?: any): void;
    info(message: string, meta?: any): void;
    warn(message: string, meta?: any): void;
    error(message: string, meta?: any): void;
    
    addTransport(transport: Transport): void;
    removeTransport(transport: Transport): void;
    
    setLevel(level: LogLevel): void;
    getLevel(): LogLevel;
  }
  
  export class ConsoleTransport implements Transport {
    constructor(options?: { level?: LogLevel });
    log(level: LogLevel, message: string, meta?: any): void;
  }
  
  export class FileTransport implements Transport {
    constructor(options: FileTransportOptions);
    log(level: LogLevel, message: string, meta?: any): void;
  }
  
  // Функции
  export function createLogger(options?: LoggerOptions): Logger;
  export function formatMessage(level: LogLevel, message: string, meta?: any): string;
  
  // Константы
  export const DEFAULT_LEVEL: LogLevel;
  export const VERSION: string;
  
  // Экспорт по умолчанию
  export default Logger;
}

// Использование
import Logger, { createLogger, ConsoleTransport, FileTransport } from 'my-custom-library';

const logger = createLogger({
  level: 'debug',
  transports: [
    new ConsoleTransport(),
    new FileTransport({ filename: 'app.log' })
  ]
});

logger.info('Application started');
logger.error('Something went wrong', { error: new Error('Test error') });
```

### 3. Типы для CSS-модулей

```typescript
// types/assets/css-modules.d.ts
declare module '*.module.css' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.scss' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.sass' {
  const classes: { [key: string]: string };
  export default classes;
}

// types/assets/images.d.ts
declare module '*.png' {
  const value: string;
  export default value;
}

declare module '*.jpg' {
  const value: string;
  export default value;
}

declare module '*.jpeg' {
  const value: string;
  export default value;
}

declare module '*.gif' {
  const value: string;
  export default value;
}

declare module '*.svg' {
  const content: React.FunctionComponent<React.SVGAttributes<SVGElement>>;
  export default content;
}

declare module '*.svg?url' {
  const value: string;
  export default value;
}

// Использование
import styles from './Button.module.css';
import logo from './logo.png';
import Icon from './icon.svg';

const className = styles.button;
const imageUrl = logo;
const IconComponent = Icon;
```

## Расширенные возможности

### 1. Условные типы в объявлениях

```typescript
// types/advanced/conditional-types.d.ts
declare module 'advanced-library' {
  // Условные типы
  export type NonNullable<T> = T extends null | undefined ? never : T;
  
  export type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
  
  export type Parameters<T> = T extends (...args: infer P) => any ? P : never;
  
  // Распределительные условные типы
  export type Diff<T, U> = T extends U ? never : T;
  
  export type Filter<T, U> = T extends U ? T : never;
  
  export type NonUndefined<T> = T extends undefined ? never : T;
  
  // Практическое применение
  export interface ApiResponse<T> {
    data: T;
    error: null | string;
    loading: boolean;
  }
  
  export type ExtractData<T> = T extends ApiResponse<infer U> ? U : never;
  
  export type ExtractError<T> = T extends ApiResponse<any> ? T['error'] : never;
  
  // Пример использования
  export function fetchData<T>(url: string): Promise<ApiResponse<T>>;
}
```

### 2. Объявления с дженериками

```typescript
// types/advanced/generics.d.ts
declare module 'generic-library' {
  // Дженерики в интерфейсах
  export interface Repository<T> {
    findById(id: string): Promise<T | null>;
    findAll(): Promise<T[]>;
    create(data: Omit<T, 'id'>): Promise<T>;
    update(id: string, data: Partial<T>): Promise<T | null>;
    delete(id: string): Promise<boolean>;
  }
  
  // Дженерики в классах
  export class BaseRepository<T> implements Repository<T> {
    constructor(protected tableName: string);
    
    findById(id: string): Promise<T | null>;
    findAll(): Promise<T[]>;
    create(data: Omit<T, 'id'>): Promise<T>;
    update(id: string, data: Partial<T>): Promise<T | null>;
    delete(id: string): Promise<boolean>;
  }
  
  // Дженерики в функциях
  export function createRepository<T>(tableName: string): Repository<T>;
  
  export function map<T, U>(array: T[], callback: (item: T, index: number) => U): U[];
  
  export function filter<T>(array: T[], predicate: (item: T, index: number) => boolean): T[];
  
  export function reduce<T, U>(array: T[], callback: (accumulator: U, item: T, index: number) => U, initialValue: U): U;
}
```

### 3. Объявления с mapped types

```typescript
// types/advanced/mapped-types.d.ts
declare module 'mapped-types-library' {
  // Mapped types
  export type Partial<T> = {
    [P in keyof T]?: T[P];
  };
  
  export type Required<T> = {
    [P in keyof T]-?: T[P];
  };
  
  export type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
  };
  
  export type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
  
  export type Record<K extends keyof any, T> = {
    [P in K]: T;
  };
  
  // Пользовательские mapped types
  export type Nullable<T> = {
    [P in keyof T]: T[P] | null;
  };
  
  export type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
  };
  
  export type ReadonlyDeep<T> = {
    readonly [P in keyof T]: T[P] extends object ? ReadonlyDeep<T[P]> : T[P];
  };
  
  export type KeysOfType<T, U> = { [K in keyof T]: T[K] extends U ? K : never }[keyof T];
  
  export type RequiredKeys<T> = { [K in keyof T]-?: {} extends Pick<T, K> ? never : K }[keyof T];
  
  export type OptionalKeys<T> = { [K in keyof T]-?: {} extends Pick<T, K> ? K : never }[keyof T];
}
```

## Публикация типов

### 1. Публикация в DefinitelyTyped

```typescript
// Для публикации в DefinitelyTyped:
// 1. Создать репозиторий DefinitelyTyped/DefinitelyTyped
// 2. Добавить типы в папку types/library-name/
// 3. Создать файлы:
//    - index.d.ts - основные типы
//    - library-name-tests.ts - тесты типов
//    - tsconfig.json - конфигурация TypeScript
//    - package.json - метаданные пакета

// types/library-name/index.d.ts
// Объявления типов библиотеки

// types/library-name/library-name-tests.ts
import library from 'library-name';

// Тесты использования типов
const result = library.someFunction('test');
```

### 2. Встроенные типы в пакет

```json
// package.json библиотеки
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist/",
    "src/"
  ]
}
```

```typescript
// src/index.ts - исходный код с типами
export interface CalculatorOptions {
  precision?: number;
}

export class Calculator {
  constructor(private options: CalculatorOptions = {}) {}
  
  add(a: number, b: number): number {
    return a + b;
  }
}

export default Calculator;

// При сборке tsc создаст dist/index.d.ts с объявлениями типов
```

### 3. Отдельный пакет типов

```json
// @types/my-library/package.json
{
  "name": "@types/my-library",
  "version": "1.0.0",
  "description": "TypeScript definitions for my-library",
  "license": "MIT",
  "types": "index.d.ts",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/my-library-types.git"
  }
}
```

## Лучшие практики

### 1. Точность типов

```typescript
// Плохо - слишком общие типы
declare module 'library' {
  export function processData(data: any): any; // Неинформативно
}

// Хорошо - точные типы
declare module 'library' {
  export interface ProcessOptions {
    encoding?: 'utf8' | 'base64';
    validate?: boolean;
  }
  
  export interface ProcessResult {
    success: boolean;
    data: string;
    errors: string[];
  }
  
  export function processData(data: string, options?: ProcessOptions): ProcessResult;
}
```

### 2. Документирование типов

```typescript
// Хорошо - документированные типы
declare module 'documented-library' {
  /**
   * HTTP клиент для выполнения запросов
   * 
   * @example
   * ```typescript
   * const client = new HttpClient('https://api.example.com');
   * const response = await client.get('/users');
   * ```
   */
  export class HttpClient {
    /**
     * Создает новый экземпляр HTTP клиента
     * @param baseURL Базовый URL для всех запросов
     * @param options Опции клиента
     */
    constructor(baseURL: string, options?: HttpClientOptions);
    
    /**
     * Выполняет GET запрос
     * @param url URL запроса (относительно baseURL)
     * @param options Опции запроса
     * @returns Promise с результатом запроса
     */
    get<T = any>(url: string, options?: RequestOptions): Promise<HttpResponse<T>>;
  }
  
  /**
   * Опции HTTP клиента
   */
  export interface HttpClientOptions {
    /** Таймаут запроса в миллисекундах */
    timeout?: number;
    
    /** Заголовки по умолчанию */
    headers?: Record<string, string>;
    
    /** Включить логирование запросов */
    debug?: boolean;
  }
}
```

### 3. Поддержка разных версий

```typescript
// types/library/v1/index.d.ts
declare module 'library/v1' {
  export interface Config {
    apiUrl: string;
  }
}

// types/library/v2/index.d.ts
declare module 'library/v2' {
  export interface Config {
    api: {
      url: string;
      version: string;
    };
  }
}

// package.json
{
  "typesVersions": {
    ">=4.0": {
      "*": [
        "types/library/v2/*"
      ]
    },
    ">=3.0 <4.0": {
      "*": [
        "types/library/v1/*"
      ]
    }
  }
}
```

### 4. Тестирование типов

```typescript
// types/tests/library-tests.ts
import library, { 
  Calculator, 
  CalculatorOptions, 
  add, 
  multiply 
} from 'library';

// Тесты типов
const options: CalculatorOptions = {
  precision: 2
};

const calc = new Calculator(options);
const sum = add(2, 3); // number
const product = multiply(4, 5); // number

// Тесты ошибок типов (должны вызывать ошибки компиляции)
// @ts-expect-error
const invalidSum = add('2', 3);

// @ts-expect-error
const calc2 = new Calculator({ invalidOption: true });
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/module-augmentation|Расширение модулей]] - Похожий механизм расширения функциональности
- [[ts/type-system/interfaces|Интерфейсы]] - Основа для объявления типов
- [[ts/advanced-types/conditional-types|Условные типы]] - Продвинутые возможности типизации
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Встроенные типы для упрощения объявления

## Рекомендации по изучению

1. Практикуйтесь в создании объявлений типов для простых библиотек
2. Освойте структуру файлов типов в проекте
3. Изучите создание типов для CSS-модулей и ресурсов
4. Практикуйтесь в документировании типов
5. Освойте тестирование объявлений типов
6. Изучите публикацию типов в DefinitelyTyped
7. Практикуйтесь в создании сложных типов с дженериками и условными типами

## Следующие шаги

- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Оптимизация для сборки
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/advanced-types/conditional-types|Условные типы]] - Продвинутые возможности типизации