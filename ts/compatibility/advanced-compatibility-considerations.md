# Совместимость в TypeScript: Версии, платформы и интеграции

Совместимость в TypeScript включает в себя несколько важных аспектов: совместимость между версиями TypeScript, совместимость с различными версиями ECMAScript, совместимость с JavaScript и различными платформами выполнения, а также совместимость между различными библиотеками и фреймворками.

## Совместимость версий TypeScript

### 1. Совместимость между версиями компилятора

```typescript
// Примеры изменений, которые могут повлиять на совместимость

// TypeScript 3.4+ добавил поддержку для const assertions
// Следующий код работает в TypeScript 3.4+, но не в более ранних версиях:
const coordinates = [10, 20] as const; // readonly [10, 20]

// TypeScript 4.0+ добавил поддержку для тюплей с остаточными элементами
// Следующий код работает в TypeScript 4.0+, но не в более ранних версиях:
type Foo = [number, ...string[]];

// TypeScript 4.1+ добавил строковые шаблонные типы
// Следующий код работает в TypeScript 4.1+, но не в более ранних версиях:
type EmailLocaleIDs = "welcome_email" | "email_heading";
type AllLocaleIDs = `${EmailLocaleIDs}_id`;
```

### 2. Повышение версии TypeScript

```json
// tsconfig.json - пример настройки для поддержки различных версий
{
  "compilerOptions": {
    // Для поддержки старых версий
    "target": "ES2018",
    "lib": ["ES2020"],
    
    // Проверка совместимости
    "strict": true,
    "skipLibCheck": true, // Полезно при работе с библиотеками, скомпилированными старыми версиями
    "forceConsistentCasingInFileNames": true
  }
}
```

### 3. Работа с устаревшими возможностями

```typescript
// Пример работы с устаревшими возможностями
class LegacyClass {
  // Старый синтаксис объявления поля
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  // Использование устаревшего декоратора (если используется)
  @deprecated
  oldMethod(): void {
    console.log("Этот метод устарел");
  }
}

// Функция для пометки устаревших API
function deprecated(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.warn(`Метод ${propertyKey} устарел и будет удален в следующей версии`);
  return descriptor;
}
```

## Совместимость с ECMAScript

### 1. Совместимость по целевой версии

```typescript
// target: ES3 (максимальная совместимость)
// Поддерживает только базовые возможности
function legacyFunction() {
  var result = [];
  for (var i = 0; i < 10; i++) {
    result.push(i);
  }
  return result;
}

// target: ES2015 (ES6) - поддержка классов, стрелочных функций и т.д.
class ModernClass {
  items: string[] = [];
  
  addItem(item: string): void {
    this.items.push(item);
  }
  
  getItems(): string[] {
    return this.items;
  }
}

// target: ES2020 - поддержка optional chaining, nullish coalescing и т.д.
function modernFeatures(obj?: { data?: { value?: string } }) {
  // Optional chaining
  const value = obj?.data?.value;
  
  // Nullish coalescing
  const result = value ?? "default";
  
  return result;
}

// target: ESNext - последние возможности
async function* asyncGenerator() {
  yield "hello";
  yield "world";
}
```

### 2. Совместимость с полифилами

```typescript
// Использование полифилов для поддержки старых сред
// Для ES2015+ возможностей в старых браузерах
import "core-js/stable";
import "regenerator-runtime/runtime";

// Проверка поддержки возможностей во время выполнения
function supportsBigInt(): boolean {
  return typeof BigInt !== "undefined";
}

function safeBigInt(value: number): bigint | number {
  if (supportsBigInt()) {
    return BigInt(value);
  }
  return value; // Возврат обычного числа для совместимости
}
```

## Совместимость с JavaScript

### 1. Совместимость с существующим JavaScript кодом

```typescript
// Пример объявления типов для существующей JavaScript библиотеки
declare module "legacy-js-library" {
  export function processData(data: any): any;
  export class DataProcessor {
    constructor(options: any);
    process(input: any): any;
  }
}

// Использование объявления типов
import { processData, DataProcessor } from "legacy-js-library";

// TypeScript теперь знает типы для этих элементов
const processor = new DataProcessor({ option: "value" });
const result = processData({ input: "data" });
```

### 2. allowJs и checkJs опции

```json
{
  "compilerOptions": {
    // Позволяет импортировать JS файлы в TS
    "allowJs": true,
    
    // Проверяет JS файлы как TS при включении allowJs
    "checkJs": true,
    
    // Игнорирует определенные JS файлы
    "skipLibCheck": true
  }
}
```

### 3. Работа с .d.ts файлами

```typescript
// example.d.ts - файл объявлений для JS библиотеки
export interface User {
  id: number;
  name: string;
  email: string;
}

export function getUser(id: number): Promise<User>;
export function createUser(userData: Omit<User, 'id'>): Promise<User>;
export function updateUser(id: number, updates: Partial<User>): Promise<User>;

// Модульные объявления
declare module "*.json" {
  const content: any;
  export default content;
}

// Глобальные объявления
declare global {
  interface Window {
    myCustomProperty: string;
  }
  
  var globalConfig: {
    apiUrl: string;
    timeout: number;
  };
}
```

## Совместимость с различными платформами

### 1. Совместимость с Node.js

```typescript
// Для Node.js приложений
import * as fs from "fs";
import * as path from "path";
import { createServer, ServerResponse, IncomingMessage } from "http";

// Использование Node.js специфичных API
async function readFileAsync(filePath: string): Promise<string> {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, "utf8", (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// Совместимость с CommonJS и ES Modules
// package.json:
// {
//   "type": "module",  // для ES modules
//   "exports": "./dist/index.js"
// }
```

### 2. Совместимость с веб-браузерами

```typescript
// Использование Web API
function useWebAPIs() {
  // Совместимость с DOM API
  const element = document.getElementById("myElement");
  if (element) {
    element.addEventListener("click", (e: Event) => {
      console.log("Clicked!");
    });
  }
  
  // Совместимость с Web Storage API
  if (typeof Storage !== "undefined") {
    localStorage.setItem("key", "value");
    const value = localStorage.getItem("key");
  }
  
  // Совместимость с Fetch API
  fetch("/api/data")
    .then(response => response.json())
    .then(data => console.log(data));
}

// Проверка поддержки возможностей
function supportsFeature(): boolean {
  // Проверка поддержки Web Workers
  if (typeof Worker === "undefined") {
    console.log("Web Workers не поддерживаются");
    return false;
  }
  
  // Проверка поддержки Canvas
  const canvas = document.createElement("canvas");
  if (!canvas.getContext("2d")) {
    console.log("Canvas не поддерживается");
    return false;
  }
  
  return true;
}
```

### 3. Совместимость с мобильными платформами

```typescript
// Для мобильных приложений (React Native, Cordova и т.д.)
// Использование платформо-специфичных API
interface MobileAPI {
  getDeviceInfo(): Promise<{
    platform: string;
    version: string;
    model: string;
  }>;
  
  vibrate(duration: number): void;
  openSettings(): void;
}

// Условная логика для разных платформ
function getPlatformAPI(): MobileAPI | null {
  if (typeof navigator !== "undefined" && /android/i.test(navigator.userAgent)) {
    // Android специфичная логика
    return getAndroidAPI();
  } else if (typeof navigator !== "undefined" && /iPad|iPhone|iPod/.test(navigator.userAgent)) {
    // iOS специфичная логика
    return getIOSAPI();
  }
  return null;
}

function getAndroidAPI(): MobileAPI {
  return {
    getDeviceInfo: () => Promise.resolve({ platform: "android", version: "10", model: "Generic" }),
    vibrate: (duration) => {
      if ((navigator as any).vibrate) {
        (navigator as any).vibrate(duration);
      }
    },
    openSettings: () => {
      // Android специфичная реализация
    }
  };
}

function getIOSAPI(): MobileAPI {
  return {
    getDeviceInfo: () => Promise.resolve({ platform: "ios", version: "14", model: "iPhone" }),
    vibrate: () => {
      // iOS не поддерживает вибрацию через Web API
    },
    openSettings: () => {
      // iOS специфичная реализация
    }
  };
}
```

## Совместимость библиотек и фреймворков

### 1. Совместимость с популярными фреймворками

```typescript
// Совместимость с React
import React, { useState, useEffect } from "react";

interface Props {
  name: string;
  age?: number;
}

const MyComponent: React.FC<Props> = ({ name, age = 0 }) => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Вы нажали ${count} раз`;
  }, [count]);
  
  return (
    <div>
      <p>Привет, {name}! Возраст: {age}</p>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
};

// Совместимость с Vue.js
import { defineComponent, ref, computed } from "vue";

const VueComponent = defineComponent({
  props: {
    name: { type: String, required: true },
    age: { type: Number, default: 0 }
  },
  setup(props) {
    const count = ref(0);
    const doubleCount = computed(() => count.value * 2);
    
    const increment = () => {
      count.value++;
    };
    
    return {
      count,
      doubleCount,
      increment,
      // props доступны в возвращаемом объекте
    };
  },
  template: `
    <div>
      <p>Привет, {{ name }}! Возраст: {{ age }}</p>
      <p>Счетчик: {{ count }}, Удвоенный: {{ doubleCount }}</p>
      <button @click="increment">Нажми меня</button>
    </div>
  `
});
```

### 2. Совместимость с различными библиотеками типов

```typescript
// Работа с разными версиями @types
// package.json
// {
//   "devDependencies": {
//     "@types/node": "^14.0.0",  // Устаревшая версия
//     "@types/react": "^17.0.0", // Современная версия
//   }
// }

// Управление версиями типов
interface CompatibleAPI {
  // Используем возможности, совместимые с разными версиями
  data: string | number; // Совместимо с большинством версий
  callback: (result: any) => void; // Совместимо с разными подходами к типизации
}

// Совместимость с разными версиями lodash
import * as _ from "lodash";

// Используем стабильные, хорошо поддерживаемые функции
function processData(items: any[]): any[] {
  return _.chain(items)
    .filter(item => item.active)
    .sortBy("name")
    .value();
}
```

## Проблемы совместимости и их решения

### 1. Проблемы с типами зависимостей

```typescript
// Проблема: устаревшие типы для библиотеки
// Решение: создание собственных объявлений типов

// types/legacy-lib.d.ts
declare module "some-legacy-lib" {
  // Определение типов для устаревшей библиотеки
  export function legacyFunction(param: any): any;
  export class LegacyClass {
    method(): any;
  }
}

// Решение: создание обертки для лучшей типизации
class TypedLegacyLibWrapper {
  static legacyFunction<T>(param: T): T {
    // Проверки типов и преобразования
    return require("some-legacy-lib").legacyFunction(param);
  }
}
```

### 2. Проблемы с различными системами модулей

```typescript
// Проблема: совместимость между CommonJS и ES Modules
// Решение: использование правильных импортов/экспортов

// commonjs-module.js (старый модуль)
// exports.myFunction = function() { return "hello"; };

// В TypeScript файле:
// Совместимый импорт CommonJS модуля
import * as commonJsModule from "./commonjs-module";
// или
import { myFunction } from "./commonjs-module";

// Для ES Modules:
// export function myFunction() { return "hello"; }
// import { myFunction } from "./es-module";
```

### 3. Проблемы с версиями зависимостей

```typescript
// Решение: использование conditional types для поддержки разных версий
type CompatibleMap<T> = 
  // Проверка версии TypeScript или библиотеки
  [T] extends [string] 
    ? StringMap 
    : [T] extends [number] 
      ? NumberMap 
      : GenericMap<T>;

interface StringMap {
  [key: string]: string;
}

interface NumberMap {
  [key: number]: number;
}

interface GenericMap<T> {
  [key: string]: T;
}

// Решение: создание абстракции над разными версиями API
interface CompatibleStorage {
  setItem(key: string, value: string): void;
  getItem(key: string): string | null;
  removeItem(key: string): void;
}

class StorageAdapter implements CompatibleStorage {
  private storage: Storage;
  
  constructor(storage: Storage = localStorage) {
    this.storage = storage;
  }
  
  setItem(key: string, value: string): void {
    try {
      this.storage.setItem(key, value);
    } catch (e) {
      // Обработка ошибок, специфичных для разных браузеров
      console.error("Не удалось сохранить в хранилище:", e);
    }
  }
  
  getItem(key: string): string | null {
    try {
      return this.storage.getItem(key);
    } catch (e) {
      // Обработка ошибок
      console.error("Не удалось получить из хранилища:", e);
      return null;
    }
  }
  
  removeItem(key: string): void {
    try {
      this.storage.removeItem(key);
    } catch (e) {
      console.error("Не удалось удалить из хранилища:", e);
    }
  }
}
```

## Практические примеры совместимости

### 1. Кросс-платформенная библиотека

```typescript
// Пример создания библиотеки, совместимой с разными платформами
interface PlatformAPI {
  readFile(path: string): Promise<string>;
  writeFile(path: string, content: string): Promise<void>;
  exists(path: string): Promise<boolean>;
}

class NodePlatformAPI implements PlatformAPI {
  private fs = require("fs").promises;
  
  async readFile(path: string): Promise<string> {
    return this.fs.readFile(path, "utf8");
  }
  
  async writeFile(path: string, content: string): Promise<void> {
    await this.fs.writeFile(path, content, "utf8");
  }
  
  async exists(path: string): Promise<boolean> {
    try {
      await this.fs.access(path);
      return true;
    } catch {
      return false;
    }
  }
}

class BrowserPlatformAPI implements PlatformAPI {
  // Имитация файловой системы в браузере
  private storage: Map<string, string> = new Map();
  
  async readFile(path: string): Promise<string> {
    const content = this.storage.get(path);
    if (content === undefined) {
      throw new Error(`Файл не найден: ${path}`);
    }
    return content;
  }
  
  async writeFile(path: string, content: string): Promise<void> {
    this.storage.set(path, content);
  }
  
  async exists(path: string): Promise<boolean> {
    return this.storage.has(path);
  }
}

// Фабрика для выбора подходящей реализации
class PlatformAPIFactory {
  static create(): PlatformAPI {
    if (typeof window !== "undefined" && typeof document !== "undefined") {
      // Браузер
      return new BrowserPlatformAPI();
    } else if (typeof require !== "undefined") {
      // Node.js
      return new NodePlatformAPI();
    } else {
      throw new Error("Неизвестная платформа");
    }
  }
}

// Использование
async function usePlatformAPI() {
  const api = PlatformAPIFactory.create();
  
  await api.writeFile("test.txt", "Hello, World!");
  const content = await api.readFile("test.txt");
  console.log(content); // "Hello, World!"
  
  const exists = await api.exists("test.txt");
  console.log("Файл существует:", exists); // true
}
```

### 2. Совместимость с разными версиями API

```typescript
// Пример работы с разными версиями внешнего API
interface UserV1 {
  id: number;
  name: string;
  email: string;
}

interface UserV2 {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  createdAt: string;
}

interface UserV3 {
  id: number;
  profile: {
    firstName: string;
    lastName: string;
  };
  contact: {
    email: string;
  };
  metadata: {
    createdAt: string;
    lastLogin: string;
  };
}

// Адаптер для совместимости
class UserAdapter {
  static toLatest<T>(user: T): UserV3 {
    if (this.isV1(user)) {
      return this.fromV1(user);
    } else if (this.isV2(user)) {
      return this.fromV2(user);
    } else if (this.isV3(user)) {
      return user;
    }
    throw new Error("Неизвестный формат пользователя");
  }
  
  private static isV1(user: any): user is UserV1 {
    return 'name' in user && 'email' in user && !('profile' in user);
  }
  
  private static isV2(user: any): user is UserV2 {
    return 'firstName' in user && 'createdAt' in user && !('profile' in user);
  }
  
  private static isV3(user: any): user is UserV3 {
    return 'profile' in user && 'contact' in user;
  }
  
  private static fromV1(user: UserV1): UserV3 {
    const [firstName, lastName] = user.name.split(' ');
    return {
      id: user.id,
      profile: {
        firstName: firstName || user.name,
        lastName: lastName || ''
      },
      contact: {
        email: user.email
      },
      metadata: {
        createdAt: new Date().toISOString(),
        lastLogin: new Date().toISOString()
      }
    };
  }
  
  private static fromV2(user: UserV2): UserV3 {
    return {
      id: user.id,
      profile: {
        firstName: user.firstName,
        lastName: user.lastName
      },
      contact: {
        email: user.email
      },
      metadata: {
        createdAt: user.createdAt,
        lastLogin: user.createdAt
      }
    };
  }
}

// Использование
const legacyUser: UserV1 = { id: 1, name: "John Doe", email: "john@example.com" };
const currentVersionUser = UserAdapter.toLatest(legacyUser);
console.log(currentVersionUser);
```

## Тестирование совместимости

### 1. Тесты для различных версий

```typescript
// Пример тестов совместимости
import { expect } from "chai";

describe("Совместимость API", () => {
  it("должна поддерживать V1 формат", () => {
    const v1User = { id: 1, name: "John", email: "john@example.com" };
    const result = UserAdapter.toLatest(v1User);
    
    expect(result.id).to.equal(1);
    expect(result.profile.firstName).to.equal("John");
    expect(result.contact.email).to.equal("john@example.com");
  });
  
  it("должна поддерживать V2 формат", () => {
    const v2User = { 
      id: 1, 
      firstName: "John", 
      lastName: "Doe", 
      email: "john@example.com", 
      createdAt: "2023-01-01T00:00:00Z" 
    };
    const result = UserAdapter.toLatest(v2User);
    
    expect(result.id).to.equal(1);
    expect(result.profile.firstName).to.equal("John");
    expect(result.profile.lastName).to.equal("Doe");
    expect(result.contact.email).to.equal("john@example.com");
  });
  
  it("должна возвращать V3 без изменений", () => {
    const v3User = {
      id: 1,
      profile: { firstName: "John", lastName: "Doe" },
      contact: { email: "john@example.com" },
      metadata: { createdAt: "2023-01-01T00:00:00Z", lastLogin: "2023-01-01T00:00:00Z" }
    };
    const result = UserAdapter.toLatest(v3User);
    
    expect(result).to.deep.equal(v3User);
  });
});
```

### 2. Проверка поддержки возможностей во время выполнения

```typescript
// Утилита для проверки поддержки возможностей
class FeatureDetection {
  static hasBigInt(): boolean {
    return typeof BigInt !== "undefined";
  }
  
  static hasAsyncAwait(): boolean {
    try {
      // Проверка синтаксиса async/await
      new Function("return (async function() {})()");
      return true;
    } catch (e) {
      return false;
    }
  }
  
  static hasProxy(): boolean {
    return typeof Proxy !== "undefined";
  }
  
  static hasWebAssembly(): boolean {
    return typeof WebAssembly !== "undefined";
  }
  
  static hasES2019Features(): boolean {
    return (
      typeof Promise.prototype.finally !== "undefined" &&
      typeof Array.prototype.flat !== "undefined" &&
      typeof Array.prototype.flatMap !== "undefined"
    );
  }
  
  static getEnvironmentInfo(): {
    platform: string;
    hasBigInt: boolean;
    hasAsyncAwait: boolean;
    hasProxy: boolean;
  } {
    return {
      platform: typeof window !== "undefined" ? "browser" : "node",
      hasBigInt: this.hasBigInt(),
      hasAsyncAwait: this.hasAsyncAwait(),
      hasProxy: this.hasProxy()
    };
  }
}

// Использование
const envInfo = FeatureDetection.getEnvironmentInfo();
console.log("Информация о среде выполнения:", envInfo);

// Условное выполнение в зависимости от поддержки возможностей
if (FeatureDetection.hasBigInt()) {
  const bigNumber = BigInt("123456789012345678901234567890");
  console.log("Работа с большими числами:", bigNumber);
} else {
  console.log("BigInt не поддерживается, используем Number");
}
```

## Лучшие практики совместимости

### 1. Миграция между версиями

```typescript
// Паттерн для постепенной миграции между версиями API
interface MigrationStrategy<T, U> {
  migrate(data: T): U;
  supports(data: any): boolean;
}

class V1ToV2Migration implements MigrationStrategy<UserV1, UserV2> {
  migrate(data: UserV1): UserV2 {
    const [firstName, lastName] = data.name.split(' ');
    return {
      id: data.id,
      firstName: firstName,
      lastName: lastName || '',
      email: data.email,
      createdAt: new Date().toISOString()
    };
  }
  
  supports(data: any): boolean {
    return 'name' in data && 'email' in data && !('firstName' in data);
  }
}

class MigrationManager {
  private strategies: MigrationStrategy<any, any>[] = [];
  
  addStrategy<T, U>(strategy: MigrationStrategy<T, U>): void {
    this.strategies.push(strategy);
  }
  
  migrate<T, U>(data: T): U {
    for (const strategy of this.strategies) {
      if (strategy.supports(data)) {
        return strategy.migrate(data);
      }
    }
    throw new Error("Не найдена подходящая стратегия миграции");
  }
}

// Использование
const migrationManager = new MigrationManager();
migrationManager.addStrategy(new V1ToV2Migration());

const v1User: UserV1 = { id: 1, name: "John Doe", email: "john@example.com" };
const v2User = migrationManager.migrate<UserV1, UserV2>(v1User);
console.log(v2User);
```

### 2. Управление зависимостями

```json
{
  "name": "my-cross-platform-lib",
  "version": "1.0.0",
  "engines": {
    "node": ">=12.0.0"
  },
  "peerDependencies": {
    "typescript": ">=3.8.0"
  },
  "devDependencies": {
    "@types/node": "^14.0.0",
    "@types/jest": "^26.0.0"
  },
  "dependencies": {
    "tslib": "^2.0.0"
  }
}
```

## Лучшие практики

1. **Тестируйте совместимость на разных платформах** - используйте CI/CD для проверки на разных средах
2. **Документируйте требования к совместимости** - укажите поддерживаемые версии и платформы
3. **Используйте фича-детектинг** - проверяйте возможности среды во время выполнения
4. **Поддерживайте обратную совместимость** - когда это возможно, избегайте критических изменений
5. **Используйте промежуточные версии для миграции** - постепенно обновляйте API
6. **Создавайте адаптеры для разных версий** - для плавного перехода между версиями

## Связи с другими концепциями

- [[ts/type-system/type-compatibility|Совместимость типов]] - основы системы типов
- [[ts/configuration/advanced-config-comprehensive|Конфигурация TypeScript]] - настройки для обеспечения совместимости
- [[build-tools/typescript-compiler]] - интеграция с инструментами сборки
- [[architecture/component-architecture]] - архитектурные аспекты совместимости
- [[js/es6-modules]] - совместимость с модульными системами