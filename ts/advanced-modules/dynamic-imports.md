# Динамические импорты в TypeScript

Динамический импорт - это механизм ECMAScript, который позволяет загружать модули по требованию во время выполнения. В TypeScript этот механизм поддерживается полностью с сохранением типобезопасности.

## Содержание

1. [Основы динамических импортов](#основы-динамических-импортов)
2. [Синтаксис и использование](#синтаксис-и-использование)
3. [Типизация динамических импортов](#типизация-динамических-импортов)
4. [Практические примеры](#практические-примеры)
5. [Оптимизация и производительность](#оптимизация-и-производительность)
6. [Обработка ошибок](#обработка-ошибок)
7. [Совместимость и ограничения](#совместимость-и-ограничения)

## Основы динамических импортов

Динамический импорт возвращает Promise, который разрешается в объект модуля. Это позволяет загружать модули асинхронно, что особенно полезно для:

- Ленивой загрузки (lazy loading)
- Условной загрузки модулей
- Оптимизации размера бандла
- Загрузки модулей по требованию

```typescript
// Статический импорт
import { add } from './math';

// Динамический импорт
const mathModule = await import('./math');
const result = mathModule.add(2, 3);
```

## Синтаксис и использование

### Базовый синтаксис

```typescript
// Динамический импорт возвращает Promise
const module = await import('./module-path');

// Использование с then/catch
import('./module-path')
  .then(module => {
    // Работа с модулем
  })
  .catch(error => {
    // Обработка ошибок
  });
```

### В асинхронных функциях

```typescript
async function loadCalculator() {
  try {
    const calculator = await import('./calculator');
    return calculator.default;
  } catch (error) {
    console.error('Failed to load calculator:', error);
    return null;
  }
}
```

### В синхронных функциях

```typescript
function loadModule() {
  import('./module')
    .then(module => {
      // Обработка модуля
    })
    .catch(error => {
      // Обработка ошибок
    });
}
```

## Типизация динамических импортов

### Автоматическая типизация

TypeScript автоматически выводит типы для динамических импортов:

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14159;

// main.ts
const mathModule = await import('./math');
// mathModule имеет тип:
// {
//   add: (a: number, b: number) => number;
//   PI: number;
//   default: any; // если есть экспорт по умолчанию
// }
```

### Явная типизация

```typescript
// calculator.ts
export interface Calculator {
  add(a: number, b: number): number;
  multiply(a: number, b: number): number;
}

export class BasicCalculator implements Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  multiply(a: number, b: number): number {
    return a * b;
  }
}

// main.ts
import type { Calculator } from './calculator';

async function loadCalculator(): Promise<Calculator | null> {
  try {
    const module = await import('./calculator');
    return new module.BasicCalculator();
  } catch (error) {
    console.error('Failed to load calculator:', error);
    return null;
  }
}
```

### Типизация экспортов по умолчанию

```typescript
// default-export.ts
export default class DefaultClass {
  method(): string {
    return 'Hello';
  }
}

// main.ts
async function loadDefaultClass() {
  const module = await import('./default-export');
  const instance = new module.default();
  return instance;
}
```

## Практические примеры

### 1. Ленивая загрузка компонентов

```typescript
// component-loader.ts
interface Component {
  render(): string;
}

async function loadComponent(name: string): Promise<Component | null> {
  try {
    switch (name) {
      case 'button':
        const buttonModule = await import('./components/button');
        return new buttonModule.Button();
      case 'input':
        const inputModule = await import('./components/input');
        return new inputModule.Input();
      case 'modal':
        const modalModule = await import('./components/modal');
        return new modalModule.Modal();
      default:
        throw new Error(`Unknown component: ${name}`);
    }
  } catch (error) {
    console.error(`Failed to load component ${name}:`, error);
    return null;
  }
}

// Использование
const button = await loadComponent('button');
if (button) {
  document.body.innerHTML = button.render();
}
```

### 2. Условная загрузка модулей

```typescript
// feature-loader.ts
interface Feature {
  execute(): void;
}

async function loadFeature(
  featureName: string, 
  condition: boolean
): Promise<Feature | null> {
  if (!condition) {
    return null;
  }
  
  try {
    const module = await import(`./features/${featureName}`);
    return new module.default();
  } catch (error) {
    console.error(`Failed to load feature ${featureName}:`, error);
    return null;
  }
}

// Использование
const debugFeature = await loadFeature('debug', process.env.NODE_ENV === 'development');
if (debugFeature) {
  debugFeature.execute();
}
```

### 3. Плагинная система

```typescript
// plugin-manager.ts
interface Plugin {
  name: string;
  version: string;
  initialize(): Promise<void>;
  execute(data: any): any;
}

class PluginManager {
  private plugins: Map<string, Plugin> = new Map();
  
  async loadPlugin(pluginPath: string): Promise<boolean> {
    try {
      const module = await import(pluginPath);
      const pluginClass = module.default;
      const plugin: Plugin = new pluginClass();
      
      await plugin.initialize();
      this.plugins.set(plugin.name, plugin);
      
      console.log(`Plugin ${plugin.name} v${plugin.version} loaded`);
      return true;
    } catch (error) {
      console.error(`Failed to load plugin from ${pluginPath}:`, error);
      return false;
    }
  }
  
  executePlugin(name: string, data: any): any {
    const plugin = this.plugins.get(name);
    if (plugin) {
      return plugin.execute(data);
    }
    throw new Error(`Plugin ${name} not found`);
  }
}

// example-plugin.ts
export default class ExamplePlugin implements Plugin {
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
```

### 4. Роутинг с ленивой загрузкой

```typescript
// router.ts
interface Route {
  path: string;
  component: () => Promise<any>;
  title: string;
}

const routes: Route[] = [
  {
    path: '/',
    component: () => import('./pages/home'),
    title: 'Home'
  },
  {
    path: '/about',
    component: () => import('./pages/about'),
    title: 'About'
  },
  {
    path: '/profile',
    component: () => import('./pages/profile'),
    title: 'Profile'
  },
  {
    path: '/admin',
    component: () => import('./pages/admin'),
    title: 'Admin Panel'
  }
];

class Router {
  private currentRoute: Route | null = null;
  
  async navigate(path: string): Promise<void> {
    const route = routes.find(r => r.path === path);
    if (!route) {
      throw new Error(`Route ${path} not found`);
    }
    
    try {
      // Динамическая загрузка компонента
      const module = await route.component();
      const component = module.default;
      
      // Рендеринг компонента
      document.title = route.title;
      this.currentRoute = route;
      
      console.log(`Navigated to ${path}`);
      console.log('Component loaded:', component);
    } catch (error) {
      console.error(`Failed to load component for route ${path}:`, error);
      throw error;
    }
  }
}
```

## Оптимизация и производительность

### 1. Предзагрузка модулей

```typescript
// preload.ts
class ModulePreloader {
  private preloadQueue: string[] = [];
  private preloaded: Set<string> = new Set();
  
  // Добавление модуля в очередь предзагрузки
  preload(modulePath: string): void {
    if (!this.preloaded.has(modulePath)) {
      this.preloadQueue.push(modulePath);
    }
  }
  
  // Асинхронная предзагрузка всех модулей
  async preloadAll(): Promise<void> {
    const promises = this.preloadQueue.map(async (path) => {
      try {
        await import(path);
        this.preloaded.add(path);
        console.log(`Preloaded module: ${path}`);
      } catch (error) {
        console.error(`Failed to preload module ${path}:`, error);
      }
    });
    
    await Promise.all(promises);
    this.preloadQueue = [];
  }
  
  // Проверка, загружен ли модуль
  isPreloaded(modulePath: string): boolean {
    return this.preloaded.has(modulePath);
  }
}

// Использование
const preloader = new ModulePreloader();
preloader.preload('./heavy-module');
preloader.preload('./another-module');

// Предзагрузка во время простоя
setTimeout(() => {
  preloader.preloadAll();
}, 1000);
```

### 2. Кэширование импортов

```typescript
// import-cache.ts
class ImportCache {
  private cache: Map<string, Promise<any>> = new Map();
  
  async import<T>(modulePath: string): Promise<T> {
    // Проверка кэша
    if (this.cache.has(modulePath)) {
      return this.cache.get(modulePath)!;
    }
    
    // Создание нового импорта
    const importPromise = import(modulePath);
    this.cache.set(modulePath, importPromise);
    
    try {
      const module = await importPromise;
      return module;
    } catch (error) {
      // Удаление из кэша при ошибке
      this.cache.delete(modulePath);
      throw error;
    }
  }
  
  // Очистка кэша
  clear(): void {
    this.cache.clear();
  }
  
  // Удаление конкретного модуля из кэша
  remove(modulePath: string): boolean {
    return this.cache.delete(modulePath);
  }
}

// Глобальный кэш импортов
const importCache = new ImportCache();

// Использование
async function loadModule(path: string) {
  try {
    const module = await importCache.import(path);
    return module;
  } catch (error) {
    console.error(`Failed to load module ${path}:`, error);
    throw error;
  }
}
```

### 3. Приоритезированная загрузка

```typescript
// priority-loader.ts
type Priority = 'high' | 'medium' | 'low';

interface PriorityImport {
  path: string;
  priority: Priority;
  promise: Promise<any> | null;
}

class PriorityLoader {
  private imports: PriorityImport[] = [];
  
  // Добавление импорта с приоритетом
  addImport(path: string, priority: Priority = 'medium'): void {
    this.imports.push({
      path,
      priority,
      promise: null
    });
  }
  
  // Загрузка всех импортов по приоритету
  async loadAll(): Promise<void> {
    // Сортировка по приоритету
    const sortedImports = [...this.imports].sort((a, b) => {
      const priorityMap: Record<Priority, number> = {
        high: 3,
        medium: 2,
        low: 1
      };
      return priorityMap[b.priority] - priorityMap[a.priority];
    });
    
    // Последовательная загрузка по приоритету
    for (const importItem of sortedImports) {
      try {
        importItem.promise = import(importItem.path);
        await importItem.promise;
        console.log(`Loaded ${importItem.path} (${importItem.priority} priority)`);
      } catch (error) {
        console.error(`Failed to load ${importItem.path}:`, error);
      }
    }
  }
  
  // Загрузка импортов с определенным приоритетом
  async loadByPriority(priority: Priority): Promise<void> {
    const imports = this.imports.filter(i => i.priority === priority);
    const promises = imports.map(i => {
      i.promise = import(i.path);
      return i.promise;
    });
    
    await Promise.all(promises);
  }
}

// Использование
const loader = new PriorityLoader();
loader.addImport('./critical-module', 'high');
loader.addImport('./feature-module', 'medium');
loader.addImport('./analytics-module', 'low');

// Загрузка критических модулей первыми
await loader.loadByPriority('high');
```

## Обработка ошибок

### 1. Базовая обработка ошибок

```typescript
// error-handling.ts
async function safeImport<T>(modulePath: string): Promise<T | null> {
  try {
    const module = await import(modulePath);
    return module;
  } catch (error) {
    if (error instanceof Error) {
      console.error(`Import error for ${modulePath}:`, {
        message: error.message,
        stack: error.stack
      });
    } else {
      console.error(`Unknown error importing ${modulePath}:`, error);
    }
    return null;
  }
}
```

### 2. Повторные попытки

```typescript
// retry-import.ts
interface RetryOptions {
  maxAttempts?: number;
  delay?: number;
  exponentialBackoff?: boolean;
}

async function importWithRetry<T>(
  modulePath: string,
  options: RetryOptions = {}
): Promise<T> {
  const {
    maxAttempts = 3,
    delay = 1000,
    exponentialBackoff = true
  } = options;
  
  let lastError: unknown;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const module = await import(modulePath);
      if (attempt > 1) {
        console.log(`Module ${modulePath} loaded on attempt ${attempt}`);
      }
      return module;
    } catch (error) {
      lastError = error;
      console.warn(`Attempt ${attempt} failed for ${modulePath}:`, error);
      
      if (attempt < maxAttempts) {
        const currentDelay = exponentialBackoff 
          ? delay * Math.pow(2, attempt - 1)
          : delay;
        
        console.log(`Retrying in ${currentDelay}ms...`);
        await new Promise(resolve => setTimeout(resolve, currentDelay));
      }
    }
  }
  
  throw new Error(`Failed to import ${modulePath} after ${maxAttempts} attempts: ${lastError}`);
}

// Использование
try {
  const module = await importWithRetry('./unstable-module', {
    maxAttempts: 5,
    delay: 500,
    exponentialBackoff: true
  });
  // Работа с модулем
} catch (error) {
  console.error('All retry attempts failed:', error);
}
```

### 3. Фолбэки

```typescript
// fallback-import.ts
async function importWithFallback<T>(
  primaryPath: string,
  fallbackPaths: string[]
): Promise<T> {
  // Попытка загрузить основной модуль
  try {
    console.log(`Trying to load primary module: ${primaryPath}`);
    const module = await import(primaryPath);
    console.log(`Primary module loaded successfully`);
    return module;
  } catch (primaryError) {
    console.warn(`Failed to load primary module ${primaryPath}:`, primaryError);
    
    // Попытка загрузить фолбэки
    for (const fallbackPath of fallbackPaths) {
      try {
        console.log(`Trying fallback module: ${fallbackPath}`);
        const module = await import(fallbackPath);
        console.log(`Fallback module loaded successfully`);
        return module;
      } catch (fallbackError) {
        console.warn(`Failed to load fallback module ${fallbackPath}:`, fallbackError);
      }
    }
    
    // Все попытки неудачны
    throw new Error(`Failed to load module from any path. Primary: ${primaryPath}, Fallbacks: ${fallbackPaths.join(', ')}`);
  }
}

// Использование
try {
  const module = await importWithFallback(
    './modern-module',
    ['./legacy-module', './fallback-module']
  );
  // Работа с модулем
} catch (error) {
  console.error('All import attempts failed:', error);
}
```

## Совместимость и ограниения

### 1. Поддержка браузеров

Динамические импорты поддерживаются большинством современных браузеров:

```typescript
// Проверка поддержки
function supportsDynamicImport(): boolean {
  try {
    // @ts-ignore
    new Function('import("")');
    return true;
  } catch (error) {
    return false;
  }
}

// Полифилл для старых браузеров
async function importModule(path: string): Promise<any> {
  if (supportsDynamicImport()) {
    return import(path);
  } else {
    // Использование SystemJS или другого полифилла
    // @ts-ignore
    return System.import(path);
  }
}
```

### 2. Ограничения TypeScript

Некоторые ограничения при использовании динамических импортов:

```typescript
// Ограничение 1: Нельзя использовать переменные в путях напрямую
const modulePath = './math';
// const module = await import(modulePath); // Ошибка типов

// Решение: Использовать switch или маппинг
async function loadModule(name: string) {
  switch (name) {
    case 'math':
      return await import('./math');
    case 'utils':
      return await import('./utils');
    default:
      throw new Error(`Unknown module: ${name}`);
  }
}

// Ограничение 2: Нет автодополнения для динамических путей
// Решение: Использовать явную типизацию
interface MathModule {
  add: (a: number, b: number) => number;
  multiply: (a: number, b: number) => number;
}

async function loadMathModule(): Promise<MathModule> {
  return await import('./math');
}
```

### 3. Сборщики и бандлеры

Различные сборщики по-разному обрабатывают динамические импорты:

```typescript
// Webpack - создает отдельные чанки
const module = await import('./feature');

// Webpack с магическими комментариями
const module = await import(
  /* webpackChunkName: "feature-module" */ 
  './feature'
);

// Webpack с предзагрузкой
const module = await import(
  /* webpackPreload: true */ 
  './critical-feature'
);

// Webpack с предзагрузкой в режиме ожидания
const module = await import(
  /* webpackPrefetch: true */ 
  './lazy-feature'
);
```

## Лучшие практики

### 1. Явная типизация

Всегда явно типизируйте результаты динамических импортов:

```typescript
// Хорошо
interface Calculator {
  add(a: number, b: number): number;
}

async function loadCalculator(): Promise<Calculator> {
  const module = await import('./calculator');
  return new module.Calculator();
}

// Плохо
async function loadCalculator() {
  const module = await import('./calculator');
  return new module.Calculator(); // Нет типизации
}
```

### 2. Обработка ошибок

Всегда обрабатывайте ошибки при динамических импортах:

```typescript
// Хорошо
async function loadModule(path: string) {
  try {
    const module = await import(path);
    return module;
  } catch (error) {
    console.error(`Failed to load module ${path}:`, error);
    return null;
  }
}

// Плохо
const module = await import('./module'); // Нет обработки ошибок
```

### 3. Избегайте дублирования

Используйте кэширование для избежания повторных загрузок:

```typescript
// Хорошо
class ModuleCache {
  private cache = new Map<string, Promise<any>>();
  
  async load(path: string) {
    if (!this.cache.has(path)) {
      this.cache.set(path, import(path));
    }
    return this.cache.get(path);
  }
}

// Плохо
// Каждый вызов создает новый импорт
const module1 = await import('./module');
const module2 = await import('./module');
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/code-splitting|Разделение кода]] - Оптимизация загрузки модулей
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Сборка приложений с динамическими импортами
- [[ts/functional-programming/async-programming|Асинхронное программирование]] - Работа с Promise и async/await
- [[ts/type-system/type-inference|Вывод типов]] - Автоматическая типизация импортов

## Рекомендации по изучению

1. Практикуйтесь в базовых динамических импортах
2. Освойте типизацию результатов импортов
3. Создавайте системы с ленивой загрузкой компонентов
4. Реализуйте обработку ошибок и повторные попытки
5. Изучите оптимизации для сборщиков (Webpack, Rollup)
6. Практикуйтесь в создании плагинных систем
7. Освойте предзагрузку и кэширование модулей

## Следующие шаги

- [[ts/advanced-modules/circular-dependencies|Циклические зависимости]] - Решение проблем циклических зависимостей
- [[ts/advanced-modules/module-augmentation|Расширение модулей]] - Добавление функциональности к существующим модулям
- [[ts/advanced-modules/code-splitting|Разделение кода]] - Разделение приложения на части
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора для модулей