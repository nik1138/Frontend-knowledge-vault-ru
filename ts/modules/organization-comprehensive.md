# Пространства имен и модули в TypeScript

Пространства имен (namespaces) и модули (modules) - это два способа организации кода в TypeScript. Они позволяют структурировать приложения, избегать конфликта имен и управлять зависимостями между различными частями кода.

## Пространства имен (Namespaces)

Пространства имен - это способ группировки связанного кода под одним именем. Они особенно полезны для организации кода в глобальной области видимости и для совместимости с более старыми JavaScript библиотеками.

### 1. Основы пространств имен

```typescript
// Определение пространства имен
namespace Validation {
  export const lettersRegexp = /^[A-Za-z]+$/;
  export const numberRegexp = /^[0-9]+$/;

  export class LettersOnlyValidator {
    isAcceptable(s: string): boolean {
      return lettersRegexp.test(s);
    }
  }

  export class ZipCodeValidator {
    isAcceptable(s: string): boolean {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}

// Использование
const validator = new Validation.LettersOnlyValidator();
console.log(validator.isAcceptable("Hello")); // true
console.log(validator.isAcceptable("Hello123")); // false
```

### 2. Вложенные пространства имен

```typescript
namespace Shapes {
  export namespace Polygons {
    export class Triangle { }
    export class Square { }
  }

  export class Circle { }
}

// Использование
const triangle = new Shapes.Polygons.Triangle();
const circle = new Shapes.Circle();
```

### 3. Разделение пространств имен по файлам

**validation.ts:**
```typescript
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }

  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;

  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return lettersRegexp.test(s);
    }
  }

  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```

**test.ts:**
```typescript
/// <reference path="validation.ts" />

// Создание массива тестов
const validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Тестирование строк
const strings = ["Hello", "98052", "101"];
const tests = [
  "Hello",
  "98052",
  "101"
];

for (const s of tests) {
  for (const name in validators) {
    console.log(`"${s}" - ${validators[name].isAcceptable(s)} для ${name}`);
  }
}
```

### 4. Псевдонимы для пространств имен

```typescript
namespace Shapes {
  export namespace Polygons {
    export class Triangle { }
    export class Square { }
  }
}

// Создание псевдонима
import polygons = Shapes.Polygons;
const triangle = new polygons.Triangle(); // эквивалентно Shapes.Polygons.Triangle()
```

## Модули (Modules)

Модули - это современный способ организации кода в TypeScript и JavaScript. Они позволяют экспортировать и импортировать код между файлами.

### 1. Основы модулей

**stringValidator.ts:**
```typescript
export interface StringValidator {
  isAcceptable(s: string): boolean;
}

export const lettersRegexp = /^[A-Za-z]+$/;
export const numberRegexp = /^[0-9]+$/;

export class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string): boolean {
    return lettersRegexp.test(s);
  }
}

// Экспорт по умолчанию
export default class NumberValidator implements StringValidator {
  isAcceptable(s: string): boolean {
    return numberRegexp.test(s);
  }
}
```

**validation.ts:**
```typescript
// Импорт всех экспортов
import * as validator from './stringValidator';

// Импорт конкретных элементов
import { StringValidator, LettersOnlyValidator } from './stringValidator';

// Импорт по умолчанию
import NumberValidator from './stringValidator';

// Импорт с переименованием
import { StringValidator as Validator } from './stringValidator';
```

### 2. Реэкспорт

**validation.ts:**
```typescript
// Реэкспорт из другого модуля
export { StringValidator } from './stringValidator';
export { LettersOnlyValidator } from './stringValidator';

// Реэкспорт всех экспортов
export * from './stringValidator';
```

### 3. Слияние модулей

```typescript
// В TypeScript можно расширять существующие модули
declare module "moment" {
  export function myNewFunction(): void;
}
```

## Сравнение пространств имен и модулей

### Пространства имен:
- Используются синтаксис `namespace`
- Работают в глобальной области видимости
- Подходят для крупных библиотек
- Требуют тегов `<reference>` для связи файлов
- Меньше подходят для модульных систем сборки

### Модули:
- Используются ключевые слова `import` и `export`
- Создают локальную область видимости
- Подходят для современных приложений
- Используются системами сборки (Webpack, Rollup и др.)
- Лучше подходят для управления зависимостями

## Практические применения

### 1. Организация библиотеки валидации

**validation/validators.ts:**
```typescript
export namespace Validators {
  export interface IValidator {
    validate(value: string): boolean;
    errorMessage: string;
  }

  export class EmailValidator implements IValidator {
    errorMessage = "Неверный формат email";
    
    validate(value: string): boolean {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailRegex.test(value);
    }
  }

  export class PhoneValidator implements IValidator {
    errorMessage = "Неверный формат телефона";
    
    validate(value: string): boolean {
      const phoneRegex = /^\+?[\d\s\-\(\)]{10,}$/;
      return phoneRegex.test(value);
    }
  }

  export class LengthValidator implements IValidator {
    constructor(private minLength: number, private maxLength: number) {}
    
    get errorMessage(): string {
      return `Длина должна быть от ${this.minLength} до ${this.maxLength} символов`;
    }
    
    validate(value: string): boolean {
      return value.length >= this.minLength && value.length <= this.maxLength;
    }
  }
}
```

**validation/validationService.ts:**
```typescript
import { Validators } from './validators';

export class ValidationService {
  static validate(value: string, ...validators: Validators.IValidator[]): { isValid: boolean; errors: string[] } {
    const errors: string[] = [];
    
    for (const validator of validators) {
      if (!validator.validate(value)) {
        errors.push(validator.errorMessage);
      }
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
}

// Использование
const emailValidator = new Validators.EmailValidator();
const result = ValidationService.validate("test@example.com", emailValidator);
console.log(result); // { isValid: true, errors: [] }
```

### 2. Модульная архитектура приложения

**models/user.ts:**
```typescript
export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

export class UserService {
  private users: User[] = [];
  
  addUser(user: Omit<User, 'id'>): User {
    const newUser: User = {
      ...user,
      id: this.generateId()
    };
    this.users.push(newUser);
    return newUser;
  }
  
  getUser(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }
  
  private generateId(): number {
    return this.users.length > 0 
      ? Math.max(...this.users.map(u => u.id)) + 1 
      : 1;
  }
}
```

**services/api.ts:**
```typescript
import { User, UserService } from '../models/user';

export class ApiService {
  private userService = new UserService();
  
  async getUser(id: number): Promise<User | null> {
    // Симуляция асинхронного запроса
    return new Promise(resolve => {
      setTimeout(() => {
        const user = this.userService.getUser(id);
        resolve(user || null);
      }, 100);
    });
  }
  
  async createUser(userData: Omit<User, 'id'>): Promise<User> {
    return new Promise(resolve => {
      setTimeout(() => {
        const user = this.userService.addUser(userData);
        resolve(user);
      }, 100);
    });
  }
}
```

### 3. Утилиты и хелперы

**utils/formatters.ts:**
```typescript
export namespace Formatters {
  export function formatDate(date: Date): string {
    return date.toLocaleDateString('ru-RU');
  }

  export function formatCurrency(amount: number, currency: string = 'RUB'): string {
    return new Intl.NumberFormat('ru-RU', {
      style: 'currency',
      currency: currency
    }).format(amount);
  }

  export function truncate(text: string, maxLength: number): string {
    if (text.length <= maxLength) {
      return text;
    }
    return text.substring(0, maxLength) + '...';
  }
}

// Использование
console.log(Formatters.formatDate(new Date())); // Например: 05.03.2023
console.log(Formatters.formatCurrency(1234.5)); // Например: 1 234,50 ₽
console.log(Formatters.truncate("Длинный текст", 10)); // "Длинный..."
```

## Современные паттерны использования модулей

### 1. Barrel (бочонок) экспорта

**components/index.ts:**
```typescript
// Barrel экcпорт - собирает все экспорты в одном месте
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export { Card } from './Card';
```

**Использование:**
```typescript
// Вместо импорта из нескольких файлов
// import { Button } from './components/Button';
// import { Input } from './components/Input';
// import { Modal } from './components/Modal';

// Можно импортировать из одного места
import { Button, Input, Modal } from './components';
```

### 2. Условные экспорты

**package.json:**
```json
{
  "name": "my-package",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js"
    },
    "./utils": {
      "import": "./dist/esm/utils/index.js",
      "require": "./dist/cjs/utils/index.js"
    }
  }
}
```

### 3. Динамические импорты

```typescript
// Динамический импорт для ленивой загрузки
async function loadSettings() {
  const { SettingsManager } = await import('./settings');
  return new SettingsManager();
}

// Условный импорт
async function loadFeature(flag: boolean) {
  if (flag) {
    const { AdvancedFeature } = await import('./advancedFeature');
    return new AdvancedFeature();
  } else {
    const { BasicFeature } = await import('./basicFeature');
    return new BasicFeature();
  }
}
```

## Практические примеры из реальной разработки

### 1. Структура библиотеки

**my-validation-library/src/index.ts:**
```typescript
// Основной файл библиотеки
export { Validator } from './Validator';
export { RequiredValidator } from './validators/RequiredValidator';
export { EmailValidator } from './validators/EmailValidator';
export { MinLengthValidator } from './validators/MinLengthValidator';
export { MaxLengthValidator } from './validators/MaxLengthValidator';

// Типы и интерфейсы
export type { IValidator } from './types';
```

**my-validation-library/src/types.ts:**
```typescript
export interface IValidator {
  validate(value: any): ValidationResult;
}

export interface ValidationResult {
  isValid: boolean;
  errorMessage?: string;
  errorCode?: string;
}
```

### 2. Архитектура приложения

**src/app.ts:**
```typescript
import { UserService } from './services/UserService';
import { AuthService } from './services/AuthService';
import { Router } from './routing/Router';
import { ComponentRegistry } from './ui/ComponentRegistry';

class App {
  private userService: UserService;
  private authService: AuthService;
  private router: Router;
  private componentRegistry: ComponentRegistry;

  constructor() {
    this.userService = new UserService();
    this.authService = new AuthService();
    this.router = new Router();
    this.componentRegistry = new ComponentRegistry();
  }

  async start() {
    // Инициализация приложения
    await this.authService.init();
    this.setupRoutes();
    this.registerComponents();
  }

  private setupRoutes() {
    // Настройка маршрутов
  }

  private registerComponents() {
    // Регистрация компонентов
  }
}

// Запуск приложения
new App().start();
```

## Ограничения и особенности

### 1. Пространства имен

```typescript
// Пространства имен не изолируют полностью глобальную область
namespace MyNamespace {
  // Это все равно будет в глобальной области при компиляции в JS
  export const globalVar = "I'm global";
}

// При использовании в браузере:
// window.MyNamespace.globalVar будет доступен
```

### 2. Модули

```typescript
// Модули создают свою область видимости
export const localVar = "I'm local to this module";

// В других файлах localVar не будет доступен без импорта
```

## Лучшие практики

1. **Предпочитайте модули пространствам имен** - в современном TypeScript
2. **Используйте Barrel экспорты** - для удобства импорта из библиотек
3. **Разделяйте код по функциональности** - не создавайте монолитные файлы
4. **Используйте динамические импорты для ленивой загрузки** - для оптимизации размера бандла
5. **Документируйте публичные API** - особенно в библиотеках
6. **Следите за циклическими зависимостями** - они могут вызвать проблемы

## Связи с другими концепциями

- [[Модули в TypeScript|Модули]] - подробное рассмотрение модульной системы
- [[ts/basics/variables|Переменные]] - как области видимости влияют на организацию кода
- [[ts/types/type-aliases|Псевдонимы типов]] - для определения типов в модулях
- [[js/modules]] - JavaScript модули, на которых основаны TypeScript модули
- [[architecture/component-architecture]] - архитектурные аспекты организации кода