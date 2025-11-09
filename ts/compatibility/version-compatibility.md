# Version Compatibility

Совместимость версий в TypeScript охватывает совместимость между разными версиями TypeScript, а также совместимость с различными версиями ECMAScript и сред выполнения JavaScript. Это критически важно для поддержки существующего кода и обеспечения плавных обновлений.

## Совместимость версий TypeScript

### Semantic Versioning в TypeScript
TypeScript следует семантическому версионированию (SemVer):
- **Major (X.0.0)**: Возможные обратно несовместимые изменения
- **Minor (Y.0)**: Новые возможности, обратно совместимы  
- **Patch (.Z)**: Исправления ошибок, обратно совместимы

### Breaking Changes между версиями
```typescript
// Пример: изменения в строгой проверке в разных версиях

// TypeScript 3.0 - нестрогая проверка (по умолчанию)
let value; // тип: any
value = "hello";
value = 42; // OK

// TypeScript 3.7+ - добавлена опция useUnknownInCatchVariables
// try {
//   // ...
// } catch (err) {
//   // err раньше был any, теперь unknown (если включен флаг)
//   console.log(err.message); // ОШИБКА без проверки типа
// }

// TypeScript 4.0 - изменения в кортежах
type OldTuple = [string, number?]; // до 4.0
type NewTuple = [string, number?]; // 4.0+ - более строгие правила

// В TypeScript 4.0+ кортежи имеют фиксированную длину с необязательными элементами
const tuple: NewTuple = ["hello"]; // OK
tuple[1] = 42; // OK в обоих версиях
// Но доступ к элементам может отличаться
```

### Совместимость синтаксиса
```typescript
// Optional Chaining (TypeScript 3.7+)
const obj = { a: { b: { c: 42 } } };
const value = obj?.a?.b?.c; // Работает в 3.7+, ошибка в ранних версиях

// Nullish Coalescing (TypeScript 3.7+)
const defaultValue = value ?? "default"; // 3.7+, ошибка в ранних версиях

// Logical Assignment Operators (TypeScript 4.0+)
a ||= b; // 4.0+, ошибка в ранних версиях
a &&= b;
a ??= b;

// Bigint (TypeScript 3.2+)
const bigNumber: bigint = 123n; // 3.2+, ошибка в ранних версиях

// Static Blocks in Classes (TypeScript 3.8+)
class MyClass {
  static { // 3.8+, ошибка в ранних версиях
    console.log("Static block executed");
  }
}
```

## Совместимость с ECMAScript версиями

### Target и Module компиляции
```json
// tsconfig.json - пример совместимости с разными версиями ES
{
  "compilerOptions": {
    // ECMAScript 2015 (ES6) - поддержка классов, модулей, стрелочных функций
    "target": "ES2015",
    
    // Для старых браузеров
    "target": "ES5",  // Поддержка IE11
    
    // Для современных браузеров
    "target": "ES2020", // Поддержка Optional Chaining, Nullish Coalescing
    
    // Для Node.js
    "target": "ES2018", // Async/Await support
    
    // Для самых современных возможностей
    "target": "ESNext"  // Latest features
  }
}
```

### Совместимость с ESNext возможностями
```typescript
// Top-level await (ES2022, TypeScript 3.8+)
// const response = await fetch('https://api.example.com'); // Только в ES2022+ target

// RegExp Match Indices (ES2022, TypeScript 4.5+)
const regex = /a+(?<Z>z)?/d;
const match = "xaa".match(regex);
// match.indices доступен только с правильной версией ES и TypeScript

// Private Fields (ES2022, TypeScript 3.8+)
class MyClass {
  #privateField = 42; // Работает с ES2022+
  
  getPrivate() {
    return this.#privateField; // Требует поддержку приватных полей
  }
}

// Using Declarations (ES Proposal, TypeScript 5.2+)
// await using fs = new FileSystem(); // Нужна экспериментальная поддержка
```

## Совместимость сред выполнения

### Node.js версии
```typescript
// Совместимость с Node.js API
import fs from 'fs/promises'; // Требует Node.js 14.0.0+
import { createRequire } from 'module'; // Требует Node.js 12.2.0+

// Проверка версии Node.js во время выполнения
const nodeVersion = process.version;
const [major] = nodeVersion.slice(1).split('.').map(Number);

if (major < 14) {
  // Использовать совместимый API
  console.log("Node.js version is too old for modern features");
}

// Использование полифилов для старых версий
import { Buffer } from 'buffer'; // В старых версиях Node.js может потребоваться полифил
```

### Совместимость браузеров
```typescript
// Проверка поддержки возможностей
function supportsES2020Features(): boolean {
  try {
    // Проверка Optional Chaining
    const obj = { a: { b: { c: 42 } } };
    const value = obj?.a?.b?.c;
    return value === 42;
  } catch {
    return false;
  }
}

function supportsES2022Features(): boolean {
  try {
    // Проверка Top-level await (в модуле)
    // Это сложно проверить в run-time, но можно проверить синтаксис
    return true;
  } catch {
    return false;
  }
}

// Условный импорт в зависимости от совместимости
async function importBasedOnSupport() {
  if (supportsES2020Features()) {
    return await import('./modern-module');
  } else {
    return await import('./legacy-module');
  }
}
```

## Совместимость с инструментами и зависимостями

### Совместимость с build tools
```json
// package.json - пример совместимости
{
  "engines": {
    "node": ">=14.0.0",           // Совместимость Node.js
    "npm": ">=6.0.0"              // Совместимость npm
  },
  "devDependencies": {
    "typescript": "^4.8.0",       // Совместимость с TS
    "ts-node": "^10.0.0",         // Совместимость с ts-node
    "@types/node": "^14.0.0"      // Совместимость типов Node.js
  },
  "peerDependencies": {
    "typescript": ">=4.0.0 <5.0.0"  // Диапазон совместимых версий
  }
}
```

### Совместимость с фреймворками
```typescript
// Совместимость с разными версиями React
// React 16 vs React 17 vs React 18 differences

// React 18 - автоматическое объединение обновлений
import { useState, useEffect } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  
  // Совместимо с React 16+
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  // flushSync (React 18+) - новая возможность
  // import { flushSync } from 'react-dom';
  // flushSync(() => setCount(c => c + 1)); // Только в 18+
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// Совместимость с Vue 3
// Vue 3.0 vs 3.2+ - изменения в реактивности
import { ref, reactive } from 'vue';

// Совместимо с Vue 3.0+
const count = ref(0);

// Новые возможности в Vue 3.2+
// const Comp = defineComponent({}); // Поведение изменилось в 3.2+
```

## Управление совместимостью

### Конфигурация tsconfig.json для совместимости
```json
{
  "compilerOptions": {
    // Базовая версия ECMAScript для совместимости
    "target": "ES2018",
    
    // Совместимость модулей
    "module": "ES2020",
    "moduleResolution": "node",
    
    // Строгие проверки
    "strict": true,
    
    // Совместимость с null/undefined
    "strictNullChecks": true,
    
    // Совместимость с неявным any
    "noImplicitAny": true,
    
    // Совместимость функций
    "strictFunctionTypes": true,
    
    // Совместимость свойств
    "strictPropertyInitialization": true,
    
    // Совместимость с this
    "noImplicitThis": true,
    
    // Совместимость с bind/call/apply
    "strictBindCallApply": true,
    
    // Совместимость с catch переменными
    "useUnknownInCatchVariables": true,
    
    // Совместимость с неиспользуемыми переменными
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    
    // Совместимость с лишними свойствами
    "exactOptionalPropertyTypes": false,
    
    // Совместимость имён файлов
    "forceConsistentCasingInFileNames": true
  },
  
  "files": ["src/index.ts"],
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

### Проверка совместимости
```typescript
// runtime-compatibility-check.ts
interface CompatibilityReport {
  typescriptVersion: string;
  targetEsVersion: string;
  nodeVersion?: string;
  browserSupport?: string;
  features: {
    [feature: string]: boolean;
  };
}

function checkCompatibility(): CompatibilityReport {
  return {
    typescriptVersion: process.env.TYPESCRIPT_VERSION || 'unknown',
    targetEsVersion: process.env.TARGET_ES_VERSION || 'unknown',
    nodeVersion: process.version,
    features: {
      optionalChaining: checkOptionalChaining(),
      nullishCoalescing: checkNullishCoalescing(),
      topLavelAwait: checkTopLevelAwait(),
      privateFields: checkPrivateFields(),
      decorators: checkDecorators()
    }
  };
}

function checkOptionalChaining(): boolean {
  try {
    const obj = { a: { b: 42 } };
    // @ts-ignore - проверяем синтаксис
    return obj?.a?.b === 42;
  } catch {
    return false;
  }
}

function checkNullishCoalescing(): boolean {
  try {
    // @ts-ignore - проверяем синтаксис
    const value = undefined ?? 'default';
    return value === 'default';
  } catch {
    return false;
  }
}

// Использование результатов проверки
const compatReport = checkCompatibility();
console.log('Compatibility Report:', compatReport);

if (!compatReport.features.optionalChaining) {
  console.warn('Optional chaining not supported, falling back to manual checks');
}
```

## Совместимость библиотек

### Совместимость с определениями типов
```typescript
// Совместимость с @types пакетами
// Проверка версий типов
import * as express from 'express';

// Совместимость типов Express
interface ExpressCompatibility {
  version: string;
  supportedMethods: string[];
  breakingChanges: string[];
}

// Проверка совместимости версий
function checkExpressVersionCompatibility(app: express.Application): ExpressCompatibility {
  const expressVersion = (express as any).version;
  
  return {
    version: expressVersion,
    supportedMethods: ['get', 'post', 'put', 'delete', 'patch'],
    breakingChanges: []
  };
}
```

### Совместимость API
```typescript
// API Compatibility Layer
abstract class ApiService {
  abstract getData<T>(id: string): Promise<T>;
  abstract postData<T>(data: T): Promise<T>;
}

// Concrete implementation that maintains compatibility
class CompatibleApiService implements ApiService {
  async getData<T>(id: string): Promise<T> {
    // Implementation that works across TypeScript versions
    const response = await fetch(`/api/data/${id}`);
    return response.json();
  }
  
  async postData<T>(data: T): Promise<T> {
    const response = await fetch('/api/data', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
    
    return response.json();
  }
}

// Wrapper for backward compatibility
class LegacyCompatibleApiService extends CompatibleApiService {
  // Add methods that were removed or changed in newer versions
  legacyGetData(id: string, callback: (data: any) => void) {
    this.getData(id).then(callback).catch(err => callback(err));
  }
}
```

## Практические стратегии совместимости

### Gradual Migration
```typescript
// migration-strategy.ts
interface MigrationStage {
  step: number;
  description: string;
  tsVersion: string;
  targetEs: string;
  featuresEnabled: string[];
}

class MigrationManager {
  private stages: MigrationStage[] = [
    {
      step: 1,
      description: "Enable strictNullChecks",
      tsVersion: "3.0+",
      targetEs: "ES2017",
      featuresEnabled: ["strictNullChecks"]
    },
    {
      step: 2,
      description: "Enable noImplicitAny",
      tsVersion: "2.0+",
      targetEs: "ES2017", 
      featuresEnabled: ["noImplicitAny"]
    },
    {
      step: 3,
      description: "Upgrade to ES2020, enable new syntax",
      tsVersion: "4.0+",
      targetEs: "ES2020",
      featuresEnabled: ["optionalChaining", "nullishCoalescing"]
    }
  ];
  
  async migrate(stage: number): Promise<void> {
    if (stage < 1 || stage > this.stages.length) {
      throw new Error(`Invalid migration stage: ${stage}`);
    }
    
    const config = this.stages[stage - 1];
    console.log(`Migrating to stage ${stage}: ${config.description}`);
    
    // Apply configuration changes
    await this.applyTsConfigChanges(config);
  }
  
  private async applyTsConfigChanges(config: MigrationStage): Promise<void> {
    // Apply configuration changes for gradual migration
    console.log(`Applied changes for TypeScript ${config.tsVersion}`);
  }
}

// Usage
const migrator = new MigrationManager();
// migrator.migrate(1); // Enable strict null checks first
```

### Feature Detection
```typescript
// feature-detection.ts
class FeatureDetector {
  static hasOptionalChaining(): boolean {
    try {
      // Try to evaluate optional chaining syntax
      // This would normally be done during build time
      return true;
    } catch {
      return false;
    }
  }
  
  static hasTopLevelAwait(): boolean {
    // Check based on Node.js version or other indicators
    if (typeof process !== 'undefined') {
      const [major, minor] = process.version.slice(1).split('.').map(Number);
      return major >= 14 || (major === 13 && minor >= 10);
    }
    return false;
  }
  
  static hasPrivateFields(): boolean {
    try {
      eval('(class { #field; })');
      return true;
    } catch {
      return false;
    }
  }
  
  static getSupportedFeatures(): string[] {
    const features = [];
    if (this.hasOptionalChaining()) features.push('optionalChaining');
    if (this.hasTopLevelAwait()) features.push('topLevelAwait');
    if (this.hasPrivateFields()) features.push('privateFields');
    return features;
  }
}

// Usage
const supportedFeatures = FeatureDetector.getSupportedFeatures();
console.log('Supported features:', supportedFeatures);
```

## Совместимость при деплое

### Совместимость production сред
```typescript
// production-compatibility.ts
interface ProductionEnvironment {
  nodeVersion: string;
  os: string;
  memory: number;
  supportedFeatures: string[];
}

class ProductionChecker {
  static async checkEnvironment(): Promise<ProductionEnvironment> {
    const os = await import('os');
    
    return {
      nodeVersion: process.version,
      os: `${os.platform()} ${os.release()}`,
      memory: os.totalmem(),
      supportedFeatures: [
        'asyncAwait',
        'es6Classes',
        'promises',
        'modules'
      ].filter(feature => this.supportsFeature(feature))
    };
  }
  
  private static supportsFeature(feature: string): boolean {
    // Check if the current environment supports the feature
    switch (feature) {
      case 'asyncAwait':
        return parseFloat(process.version.slice(1)) >= 7.6;
      case 'es6Classes':
        return parseFloat(process.version.slice(1)) >= 4.0;
      default:
        return true;
    }
  }
}

// Ensure compatibility before starting the application
async function ensureCompatibility() {
  const env = await ProductionChecker.checkEnvironment();
  
  if (!env.supportedFeatures.includes('asyncAwait')) {
    throw new Error('Environment does not support async/await');
  }
  
  console.log('Environment compatibility check passed');
  return env;
}
```

## Лучшие практики совместимости

### 1. Использование проверенных версий
```typescript
// Используйте точные версии в package-lock.json
// Это обеспечивает воспроизводимость сборки

// package.json
{
  "devDependencies": {
    "typescript": "~4.8.0",  // Патчи разрешены, минорные нет
    "ts-node": "^10.9.0"     // Минорные и патчи разрешены
  }
}
```

### 2. Тестирование на разных версиях
```typescript
// .github/workflows/compatibility.yml
interface CiMatrix {
  nodeVersions: string[];
  tsVersions: string[];
  targets: string[];
}

const testMatrix: CiMatrix = {
  nodeVersions: ['14', '16', '18', '19'],
  tsVersions: ['4.7', '4.8', '4.9', '5.0', '5.1'],
  targets: ['ES2018', 'ES2020', 'ESNext']
};

// Запуск тестов на разных комбинациях
function runCompatibilityTests(matrix: CiMatrix) {
  for (const nodeVer of matrix.nodeVersions) {
    for (const tsVer of matrix.tsVersions) {
      for (const target of matrix.targets) {
        console.log(`Testing Node ${nodeVer}, TS ${tsVer}, Target ${target}`);
        // Запуск тестов
      }
    }
  }
}
```

### 3. Постепенные обновления
```typescript
// Миграционный план
interface MigrationPlan {
  currentVersion: string;
  targetVersion: string;
  steps: {
    description: string;
    requiredChanges: string[];
    estimatedTime: string;
  }[];
}

const migrationPlan: MigrationPlan = {
  currentVersion: "4.5.0",
  targetVersion: "5.0.0",
  steps: [
    {
      description: "Enable new strict checks",
      requiredChanges: ["Update tsconfig", "Fix compilation errors"],
      estimatedTime: "2-3 days"
    },
    {
      description: "Migrate deprecated APIs",
      requiredChanges: ["Replace old patterns", "Update dependencies"],
      estimatedTime: "1 week"
    }
  ]
};
```

## Ошибки и подводные камни

### Частые проблемы совместимости
```typescript
// 1. Несовместимость версий зависимостей
// package.json
// {
//   "dependencies": {
//     "typescript": "^4.0.0",
//     "my-library": "1.0.0"  // требует typescript: ^3.8.0
//   }
// }
// Решение: синхронизация версий

// 2. Использование новых синтаксических возможностей
// const value = obj?.prop?.method?.(); // Требует TS 3.7+
// const result = value ?? "default";   // Требует TS 3.7+

// 3. Несовместимость target версии
// async function main() { await doSomething(); } // Требует ES2017+ для компиляции
// Если target ES5, то нужен полифил для Promises
```

## Заключение

Совместимость версий в TypeScript - это многоуровневая задача, включающая:

1. **Версии TypeScript** - различия между major/minor версиями
2. **ECMAScript версии** - совместимость с целевой средой выполнения
3. **Среды выполнения** - Node.js, браузеры, мобильные платформы
4. **Зависимости** - совместимость с библиотеками и фреймворками
5. **Инструменты сборки** - совместимость с транспайлерами и bundlers

Ключ к успешной совместимости - постепенные обновления, тестирование на разных версиях и использование подходящих флагов компиляции.

## Связь с другими концепциями
- [[../compilation/configuration]] - конфигурация для совместимости
- [[../ecosystem/tools]] - инструменты для управления совместимостью  
- [[../runtime/javascript-interop]] - совместимость с JavaScript
- [[../modules/compatibility]] - совместимость модулей
- [[../migration/guides]] - руководства по миграции между версиями