# Advanced Module Systems in TypeScript

Продвинутые системы модулей в TypeScript охватывают сложные паттерны импорта/экспорта, совместимость различных систем модулей и продвинутую работу с пространствами имен.

## Современные ES Modules

### Статические и динамические импорты
```typescript
// Статические импорты (компилируются статически)
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

// Именованные импорты с переименованием
import { Component as NgComponent, OnInit as NgOnInit } from '@angular/core';

// Импорт всего модуля как namespace
import * as Angular from '@angular/core';

// Импорт по умолчанию
import React from 'react';
import ReactDOM from 'react-dom/client';

// Комбинированный импорт
import React, { Component, useState } from 'react';
import { BehaviorSubject, Observable } from 'rxjs';

// Типы могут быть импортированы отдельно (TS 3.8+)
import type { OnInit } from '@angular/core';
import { Component } from '@angular/core'; // значение + тип
```

### Динамические импорты
```typescript
// Динамические импорты для ленивой загрузки
async function loadModule() {
  const { heavyFunction } = await import('./heavy-module');
  return heavyFunction();
}

// Использование динамических импортов для условной загрузки
async function loadByPlatform() {
  if (typeof window !== 'undefined') {
    // Клиентский код
    const { ClientService } = await import('./client-service');
    return new ClientService();
  } else {
    // Серверный код
    const { ServerService } = await import('./server-service');
    return new ServerService();
  }
}

// Динамическая загрузка компонентов
async function loadComponent(type: string) {
  switch (type) {
    case 'chart':
      const { ChartComponent } = await import('./charts/chart.component');
      return ChartComponent;
    case 'table':
      const { TableComponent } = await import('./tables/table.component');
      return TableComponent;
    default:
      const { DefaultComponent } = await import('./common/default.component');
      return DefaultComponent;
  }
}
```

### Re-exports (Переэкспорт)
```typescript
// utils.ts
export { Component } from './component';
export { Service } from './service';
export { Helper } from './helper';

// types.ts
export type { Config } from './config.types';
export type { Response } from './response.types';

// index.ts - главный файл переэкспорта
export * from './utils';
export * from './types';
export { default as MainService } from './main.service';

// Переэкспорт с переименованием
export { Service as ApiService } from './service';

// Условный переэкспорт
export * from './platform-specific';

// Переэкспорт с фильтрацией
export { Component } from './complex-module'; // экспорт только одного
// export * from './complex-module'; // экспорт всего
```

## Совместимость систем модулей

### ES Modules ↔ CommonJS
```javascript
// commonjs-module.js
function myFunction() {
  return "Hello from CommonJS";
}

const myValue = 42;

// CommonJS экспорт
module.exports = {
  myFunction,
  myValue
};

// Или по умолчанию
// module.exports = myFunction;
```

```typescript
// es-module.ts
// ES Module импорт из CommonJS
import { myFunction, myValue } from './commonjs-module';
import myModule from './commonjs-module'; // default импорт
import * as ns from './commonjs-module'; // namespace импорт

// Или с createRequire для динамического импорта
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const cjsModule = require('./commonjs-module');
```

### Require ↔ Import взаимодействие
```typescript
// В Node.js с ES Modules можно использовать require через createRequire
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

// Это позволяет использовать CommonJS модули:
const fs = require('fs');
const path = require('path');

// Совместимость в одном файле:
import { readFile } from 'fs/promises'; // ES Module импорт
const fsCommon = require('fs');         // CommonJS импорт

// Трансформация между системами:
function ensureFileSync(filePath: string, content: string) {
  if (typeof require !== 'undefined') {
    // CommonJS путь
    const fs = require('fs');
    if (!fs.existsSync(filePath)) {
      fs.writeFileSync(filePath, content);
    }
  } else {
    // ES Module путь
    import('fs').then(fs => {
      if (!fs.existsSync(filePath)) {
        fs.writeFileSync(filePath, content);
      }
    });
  }
}
```

## Продвинутые паттерны модулей

### Module Augmentation
```typescript
// types/base.d.ts
export interface User {
  id: number;
  name: string;
}

// types/extensions.d.ts
import { User } from './base';

declare module './base' {
  interface User {
    email: string;        // добавляем email к существующему User
    getFullName(): string; // добавляем метод
  }
}

// Реализация дополнения
declare global {
  interface User {
    email: string;
    getFullName(): string;
  }
}

// Реализация метода
declare module './base' {
  interface User {
    getFullName(): string;
  }
}

// Использование
import { User } from './base';

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  getFullName() { return this.name; }
};
```

### Ambient Modules
```typescript
// types/ambient.d.ts
declare module 'legacy-library' {
  export function legacyFunction(): void;
  export const VERSION: string;
  
  // Определение класса в модуле
  export class LegacyClass {
    constructor(options: any);
    doStuff(): void;
  }
}

declare module '*!text' {
  const content: string;
  export default content;
}

declare module '*.json' {
  const value: any;
  export default value;
}

// Модули с шаблонными именами
declare module 'module-prefix-*' {
  export function dynamicFunction(): void;
}

// Использование ambient модулей
import data from './config.json';
import text from './template.html!text';
import { legacyFunction, LegacyClass } from 'legacy-library';
```

### Module Declaration Merging
```typescript
// module-a.ts
export function utilityA() { return "A"; }

// module-b.ts
export function utilityB() { return "B"; }

// merged-module.ts
// TypeScript позволяет объединять пространства имен из разных файлов
import { utilityA } from './module-a';
import { utilityB } from './module-b';

// Объединение функциональности в одном пространстве имен
namespace Utilities {
  export const A = utilityA;
  export const B = utilityB;
  
  export function combined() {
    return A() + B();
  }
}

// Или через экспорты
export { utilityA, utilityB } from './module-a';
export { utilityB } from './module-b';

// Или в глобальной области
declare global {
  var __UTILITY_FUNCTIONS: typeof import('./module-a') & typeof import('./module-b');
}

if (typeof global !== 'undefined') {
  global.__UTILITY_FUNCTIONS = { ...require('./module-a'), ...require('./module-b') };
}
```

## Модульные паттерны

### Factory Pattern для модулей
```typescript
// factory.ts
interface ModuleConfig {
  name: string;
  dependencies: string[];
}

interface Module<T = any> {
  instance: T;
  init(): Promise<void>;
  destroy(): Promise<void>;
}

type ModuleFactory<T> = (config: ModuleConfig) => Promise<Module<T>>;

// Создание фабрики модулей
class ModuleRegistry {
  private modules = new Map<string, Module>();
  private factories = new Map<string, ModuleFactory<any>>();
  
  registerFactory<T>(name: string, factory: ModuleFactory<T>) {
    this.factories.set(name, factory);
  }
  
  async instantiate<T>(name: string, config: ModuleConfig): Promise<Module<T>> {
    const factory = this.factories.get(name);
    if (!factory) {
      throw new Error(`Factory for module '${name}' not found`);
    }
    
    const module = await factory(config);
    this.modules.set(name, module);
    return module as Module<T>;
  }
  
  get<T>(name: string): Module<T> | undefined {
    return this.modules.get(name) as Module<T>;
  }
}

// Использование
const registry = new ModuleRegistry();

registry.registerFactory('http-client', async (config) => {
  // динамический импорт модуля
  const { HttpClient } = await import('./http-client');
  
  const client = new HttpClient(/* config */);
  
  return {
    instance: client,
    async init() { await client.init(); },
    async destroy() { await client.destroy(); }
  };
});

// Пример использования
async function example() {
  const httpClient = await registry.instantiate('http-client', {
    name: 'api-client',
    dependencies: []
  });
  
  await httpClient.init();
  // использовать httpClient.instance
}
```

### Plugin System
```typescript
// plugin.ts
export interface Plugin {
  name: string;
  version: string;
  dependencies?: string[];
  init(context: PluginContext): Promise<void>;
  destroy?(): Promise<void>;
}

export interface PluginContext {
  registerService<T>(name: string, service: T): void;
  getService<T>(name: string): T | undefined;
  emit(event: string, data: any): void;
  on(event: string, listener: (data: any) => void): void;
}

// plugin-manager.ts
export class PluginManager {
  private plugins = new Map<string, Plugin>();
  private services = new Map<string, any>();
  private eventListeners = new Map<string, Set<(data: any) => void>>();
  
  async loadPlugin(plugin: Plugin): Promise<void> {
    // Проверить зависимости
    if (plugin.dependencies) {
      for (const dep of plugin.dependencies) {
        if (!this.plugins.has(dep)) {
          throw new Error(`Dependency '${dep}' not loaded for plugin '${plugin.name}'`);
        }
      }
    }
    
    // Инициализировать плагин
    await plugin.init({
      registerService: (name, service) => this.services.set(name, service),
      getService: (name) => this.services.get(name),
      emit: (event, data) => this.emitEvent(event, data),
      on: (event, listener) => this.addEventListener(event, listener)
    });
    
    this.plugins.set(plugin.name, plugin);
  }
  
  private emitEvent(event: string, data: any) {
    const listeners = this.eventListeners.get(event);
    if (listeners) {
      listeners.forEach(listener => listener(data));
    }
  }
  
  private addEventListener(event: string, listener: (data: any) => void) {
    if (!this.eventListeners.has(event)) {
      this.eventListeners.set(event, new Set());
    }
    this.eventListeners.get(event)!.add(listener);
  }
  
  async unloadPlugin(name: string): Promise<void> {
    const plugin = this.plugins.get(name);
    if (plugin?.destroy) {
      await plugin.destroy();
    }
    this.plugins.delete(name);
  }
}

// Использование
const manager = new PluginManager();

// Плагин для логирования
class LoggerPlugin implements Plugin {
  name = 'logger';
  version = '1.0.0';
  
  async init(context: PluginContext) {
    context.registerService('logger', {
      log: (message: string) => console.log(`[LOG] ${message}`)
    });
    
    context.on('error', (error) => {
      const logger = context.getService('logger');
      logger?.log(`Error caught: ${error}`);
    });
  }
}

manager.loadPlugin(new LoggerPlugin());
```

## Conditional Exports

### Modern package.json structure
```json
{
  "name": "my-advanced-package",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "import": {
        "types": "./dist/types/index.d.ts",
        "default": "./dist/esm/index.js"
      },
      "require": {
        "types": "./dist/types/index.d.cts",
        "default": "./dist/cjs/index.cjs"
      }
    },
    "./utils": {
      "import": {
        "types": "./dist/types/utils.d.ts",
        "default": "./dist/esm/utils.js"
      },
      "require": {
        "types": "./dist/types/utils.d.cts", 
        "default": "./dist/cjs/utils.cjs"
      }
    },
    "./browser": {
      "browser": "./dist/browser/index.js",
      "import": "./dist/esm/browser.js",
      "require": "./dist/cjs/browser.cjs"
    },
    "./node": {
      "node": "./dist/node/index.js",
      "import": "./dist/esm/node.js", 
      "require": "./dist/cjs/node.cjs"
    }
  },
  "main": "./dist/cjs/index.cjs",
  "module": "./dist/esm/index.js",
  "types": "./dist/types/index.d.ts",
  "files": [
    "dist/",
    "README.md"
  ]
}
```

### Runtime environment detection
```typescript
// environment-aware.ts
interface ModuleMetadata {
  env: 'browser' | 'node' | 'webworker';
  support: {
    es2020: boolean;
    es2022: boolean;
    topLevelAwait: boolean;
  };
}

// Определение среды выполнения
function getEnvironment(): ModuleMetadata {
  if (typeof window !== 'undefined') {
    return {
      env: 'browser',
      support: {
        es2020: true,
        es2022: typeof globalThis.TextEncoder !== 'undefined',
        topLevelAwait: true
      }
    };
  } else if (typeof importScripts !== 'undefined') {
    return {
      env: 'webworker',
      support: {
        es2020: true,
        es2022: true,
        topLevelAwait: true
      }
    };
  } else {
    return {
      env: 'node',
      support: {
        es2020: parseInt(process.version.slice(1)) >= 14,
        es2022: parseInt(process.version.slice(1)) >= 17,
        topLevelAwait: parseInt(process.version.slice(1)) >= 14
      }
    };
  }
}

// Загрузка модуля в зависимости от среды
async function loadPlatformSpecificModule() {
  const env = getEnvironment();
  
  if (env.env === 'browser') {
    // Для браузера - использовать нативные APIs
    return await import('./browser-specific');
  } else if (env.env === 'node') {
    // Для Node.js - использовать специфичные модули
    return await import('./node-specific');
  } else {
    // Для воркеров - использовать другую реализацию
    return await import('./webworker-specific');
  }
}
```

## Tree Shaking и Dead Code Elimination

### Эффективные экспорты для tree shaking
```typescript
// utilities.ts - правильно структурированный модуль для tree shaking
export const add = (a: number, b: number): number => a + b;
export const subtract = (a: number, b: number): number => a - b;
export const multiply = (a: number, b: number): number => a * b;
export const divide = (a: number, b: number): number => {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
};

// Комплексные функции
export const complexCalculation = (data: number[]): number => {
  return data.reduce((sum, num) => multiply(sum, add(num, 1)), 1);
};

export const anotherComplexFunction = (input: string[]): string => {
  return input.filter(s => s.length > 0).join(',');
};

// Использование в другом модуле (только одна функция будет включена в bundle)
import { add } from './utilities';

const result = add(5, 3); // только add будет включен, остальные будут "обрещены"
```

### Side Effect Free Modules
```typescript
// pure-helpers.ts - чистые функции без побочных эффектов
export const clamp = (value: number, min: number, max: number): number => {
  return Math.max(min, Math.min(max, value));
};

export const roundToDecimal = (value: number, decimals: number): number => {
  const factor = Math.pow(10, decimals);
  return Math.round(value * factor) / factor;
};

export const percentage = (part: number, whole: number): number => {
  if (whole === 0) return 0;
  return (part / whole) * 100;
};

// Для tree shaking важно избегать побочных эффектов
// НЕ ДЕЛАЙТЕ:
// console.log("Loading helpers"); // побочный эффект
// global.SOME_VAR = "value"; // побочный эффект

// Файл без побочных эффектов может быть безопасно удален, если не используется
```

## Module Federation (для микросервисов фронтенда)

### Advanced Module Federation concepts
```typescript
// remote-entry.ts (упрощенная реализация module federation)
interface RemoteEntry {
  get<T>(moduleName: string): Promise<() => T>;
  init(shared: Record<string, any>): void;
}

// Упрощенный загрузчик удаленных модулей
class RemoteModuleLoader {
  private container: any;
  private shared: Record<string, any> = {};
  
  async loadRemoteEntry(remoteUrl: string): Promise<void> {
    // Загрузка удаленного entry файла
    this.container = await __webpack_init_sharing__('default');
    const remoteEntry = await import(remoteUrl);
    remoteEntry.init(this.shared);
    this.container = remoteEntry;
  }
  
  async get<T>(remoteName: string, moduleName: string): Promise<T> {
    if (!this.container) {
      throw new Error('Remote container not loaded');
    }
    
    const factory = await this.container.get(moduleName);
    const Module = factory();
    return Module as T;
  }
  
  share(scope: string, shared: Record<string, any>): void {
    this.shared[scope] = shared;
  }
}

// Использование (в реальности через Webpack Module Federation)
/*
const loader = new RemoteModuleLoader();
loader.share('react', { react: React });
await loader.loadRemoteEntry('http://localhost:3001/remoteEntry.js');
const RemoteComponent = await loader.get<React.ComponentType>('ui', './Button');
*/
```

## Best Practices

### 1. Структурирование экспорта
```typescript
// features/index.ts - центральный экспорт
export { default as FeatureA } from './feature-a';
export { default as FeatureB } from './feature-b';
export { utilityFunction } from './utility-function';
export type { FeatureAType, FeatureBType } from './types';

// components/index.ts - переэкспорт компонентов
export { Button } from './button/button.component';
export { Input } from './input/input.component';
export { Modal } from './modal/modal.component';

// services/index.ts - переэкспорт сервисов
export { HttpService } from './http.service';
export { StorageService } from './storage.service';
export { CacheService } from './cache.service';
```

### 2. Ленивая загрузка
```typescript
// lazy-loader.ts
interface LazyModule<T> {
  get(): Promise<T>;
  preload(): void;
  isLoaded(): boolean;
}

class ModulePreloader {
  private loadedModules = new Map<string, any>();
  private loadingPromises = new Map<string, Promise<any>>();
  
  async load<T>(modulePath: string): Promise<T> {
    if (this.loadedModules.has(modulePath)) {
      return this.loadedModules.get(modulePath);
    }
    
    if (this.loadingPromises.has(modulePath)) {
      return this.loadingPromises.get(modulePath);
    }
    
    const loadPromise = import(modulePath);
    this.loadingPromises.set(modulePath, loadPromise);
    
    const module = await loadPromise;
    this.loadedModules.set(modulePath, module);
    this.loadingPromises.delete(modulePath);
    
    return module as T;
  }
  
  preload(modulePath: string) {
    if (!this.loadingPromises.has(modulePath)) {
      this.loadingPromises.set(modulePath, import(modulePath));
    }
  }
  
  isLoaded(modulePath: string): boolean {
    return this.loadedModules.has(modulePath);
  }
}

const preloader = new ModulePreloader();

// Использование
async function useHeavyModule() {
  // Загрузить только когда нужно
  const { HeavyClass } = await preloader.load('./heavy-module');
  return new HeavyClass();
}
```

### 3. Управление зависимостями
```typescript
// dependency-injection.ts
interface ModuleDependencies {
  [key: string]: any;
}

class DependencyContainer {
  private dependencies: ModuleDependencies = {};
  
  register<T>(token: string, dependency: T): void {
    this.dependencies[token] = dependency;
  }
  
  resolve<T>(token: string): T {
    if (!this.dependencies[token]) {
      throw new Error(`Dependency ${token} not found`);
    }
    return this.dependencies[token] as T;
  }
  
  async resolveAsync<T>(token: string, factory: () => Promise<T>): Promise<T> {
    if (!this.dependencies[token]) {
      this.dependencies[token] = await factory();
    }
    return this.dependencies[token] as T;
  }
}

// Использование
const container = new DependencyContainer();
container.register('httpService', new HttpService());
container.register('config', { apiUrl: 'https://api.example.com' });

// В модуле
class UserService {
  constructor(private httpService = container.resolve('httpService')) {}
  
  async getUser(id: number) {
    return await this.httpService.get(`/users/${id}`);
  }
}
```

## Заключение

Продвинутые системы модулей в TypeScript позволяют:

1. **Создавать гибкие архитектуры** с чистыми зависимостями
2. **Обеспечивать совместимость** между разными системами модулей
3. **Оптимизировать сборку** через tree shaking и ленивую загрузку
4. **Реализовывать плагинные архитектуры** для расширяемых приложений
5. **Управлять сложными зависимостями** через контейнеры инъекций

Правильное использование модульных систем позволяет создавать масштабируемые, тестируемые и поддерживаемые приложения.

## Связь с другими концепциями
- [[../compatibility/module-compatibility]] - совместимость модулей
- [[../compilation/bundling]] - сборка и оптимизация модулей  
- [[../ecosystem/tools]] - инструменты для работы с модулями
- [[../runtime/javascript-interop]] - взаимодействие с JavaScript модулями
- [[../deployment/modules]] - деплой модулей