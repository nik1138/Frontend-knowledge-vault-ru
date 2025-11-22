# Документация и качество кода в TypeScript

Документация и качество кода являются критически важными аспектами разработки на TypeScript. Хорошо документированный и качественный код улучшает сопровождаемость, читаемость и надежность приложений.

## Стандарты документирования

### 1. JSDoc с TypeScript

```typescript
/**
 * Интерфейс для пользователя
 * @interface User
 * @property {number} id - Уникальный идентификатор пользователя
 * @property {string} name - Полное имя пользователя
 * @property {string} email - Адрес электронной почты
 * @property {boolean} isActive - Статус активности пользователя
 */
export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

/**
 * Сервис для управления пользователями
 * 
 * @class UserService
 * @description Этот класс предоставляет методы для работы с пользователями,
 * включая создание, обновление, удаление и поиск пользователей.
 */
export class UserService {
  private users: User[] = [];

  /**
   * Создает нового пользователя
   * 
   * @param {string} name - Имя пользователя
   * @param {string} email - Email пользователя
   * @returns {Promise<User>} Обещание с созданным пользователем
   * 
   * @throws {Error} Если email уже используется
   * @throws {Error} Если email имеет неверный формат
   * 
   * @example
   * ```typescript
   * const userService = new UserService();
   * const user = await userService.createUser('John Doe', 'john@example.com');
   * console.log(user); // { id: 1, name: 'John Doe', email: 'john@example.com', isActive: true }
   * ```
   */
  async createUser(name: string, email: string): Promise<User> {
    // Проверка уникальности email
    if (this.users.some(user => user.email === email)) {
      throw new Error(`Пользователь с email ${email} уже существует`);
    }

    // Валидация email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new Error(`Неверный формат email: ${email}`);
    }

    const newUser: User = {
      id: this.users.length + 1,
      name,
      email,
      isActive: true
    };

    this.users.push(newUser);
    return newUser;
  }

  /**
   * Находит пользователя по ID
   * 
   * @param {number} id - ID пользователя для поиска
   * @returns {Promise<User | null>} Обещание с пользователем или null если не найден
   * 
   * @example
   * ```typescript
   * const user = await userService.findById(1);
   * if (user) {
   *   console.log(`Найден пользователь: ${user.name}`);
   * } else {
   *   console.log('Пользователь не найден');
   * }
   * ```
   */
  async findById(id: number): Promise<User | null> {
    return this.users.find(user => user.id === id) || null;
  }

  /**
   * Обновляет информацию о пользователе
   * 
   * @param {number} id - ID пользователя для обновления
   * @param {Partial<User>} updates - Обновления для пользователя
   * @returns {Promise<User | null>} Обещание с обновленным пользователем или null
   * 
   * @example
   * ```typescript
   * const updatedUser = await userService.updateUser(1, { name: 'Jane Doe' });
   * if (updatedUser) {
   *   console.log('Пользователь обновлен:', updatedUser);
   * }
   * ```
   */
  async updateUser(id: number, updates: Partial<User>): Promise<User | null> {
    const index = this.users.findIndex(user => user.id === id);
    if (index === -1) {
      return null;
    }

    const updatedUser = { ...this.users[index], ...updates };
    
    // Проверка уникальности email при обновлении
    if (updates.email) {
      const existingUser = this.users.find(user => 
        user.id !== id && user.email === updates.email
      );
      if (existingUser) {
        throw new Error(`Пользователь с email ${updates.email} уже существует`);
      }
    }

    this.users[index] = updatedUser;
    return updatedUser;
  }

  /**
   * Возвращает всех пользователей с возможностью фильтрации
   * 
   * @param {Partial<User>} filter - Объект фильтрации
   * @returns {Promise<User[]>} Обещание с массивом пользователей
   * 
   * @example
   * ```typescript
   * // Получить всех активных пользователей
   * const activeUsers = await userService.findAll({ isActive: true });
   * 
   * // Получить пользователей с определенным email
   * const users = await userService.findAll({ email: 'john@example.com' });
   * ```
   */
  async findAll(filter: Partial<User> = {}): Promise<User[]> {
    return this.users.filter(user => {
      return Object.entries(filter).every(([key, value]) => 
        user[key as keyof User] === value
      );
    });
  }
}

/**
 * Тип для результатов поиска пользователей
 * 
 * @typedef {Object} SearchResults
 * @property {User[]} results - Массив найденных пользователей
 * @property {number} totalCount - Общее количество пользователей
 * @property {number} page - Номер текущей страницы
 * @property {number} pageSize - Размер страницы
 */
export interface SearchResults {
  results: User[];
  totalCount: number;
  page: number;
  pageSize: number;
}

/**
 * Интерфейс для параметров поиска пользователей
 * 
 * @interface SearchParams
 * @property {string} [query] - Поисковый запрос
 * @property {number} [page] - Номер страницы (по умолчанию 1)
 * @property {number} [pageSize] - Размер страницы (по умолчанию 10)
 * @property {'name' | 'email'} [sortBy] - Поле для сортировки
 * @property {'asc' | 'desc'} [sortOrder] - Направление сортировки
 */
export interface SearchParams {
  query?: string;
  page?: number;
  pageSize?: number;
  sortBy?: 'name' | 'email';
  sortOrder?: 'asc' | 'desc';
}
```

### 2. Документирование Generic типов

```typescript
/**
 * Класс для безопасной работы с асинхронными операциями
 * 
 * @template T - Тип успешного результата
 * @template E - Тип ошибки (по умолчанию Error)
 */
export class AsyncResult<T, E = Error> {
  /**
   * Создает успешный результат
   * 
   * @param {T} data - Данные успешного результата
   * @returns {AsyncResult<T, E>} Экземпляр AsyncResult с успешным результатом
   */
  static success<T, E = Error>(data: T): AsyncResult<T, E> {
    return new AsyncResult<T, E>({ success: true, data });
  }

  /**
   * Создает результат с ошибкой
   * 
   * @param {E} error - Ошибка
   * @returns {AsyncResult<T, E>} Экземпляр AsyncResult с ошибкой
   */
  static error<T, E = Error>(error: E): AsyncResult<T, E> {
    return new AsyncResult<T, E>({ success: false, error });
  }

  /**
   * @private
   */
  private constructor(
    private result: { success: true; data: T } | { success: false; error: E }
  ) {}

  /**
   * Проверяет, является ли результат успешным
   * 
   * @returns {boolean} true если результат успешный, false если содержит ошибку
   */
  isSuccess(): this is { result: { success: true; data: T } } {
    return this.result.success;
  }

  /**
   * Проверяет, содержит ли результат ошибку
   * 
   * @returns {boolean} true если результат содержит ошибку, false если успешный
   */
  isError(): this is { result: { success: false; error: E } } {
    return !this.result.success;
  }

  /**
   * Возвращает данные результата или значение по умолчанию
   * 
   * @param {T} defaultValue - Значение по умолчанию
   * @returns {T} Данные результата или значение по умолчанию
   */
  getOrElse(defaultValue: T): T {
    return this.result.success ? this.result.data : defaultValue;
  }

  /**
   * Преобразует успешный результат с помощью переданной функции
   * 
   * @template U - Новый тип результата
   * @param {(value: T) => U} fn - Функция преобразования
   * @returns {AsyncResult<U, E>} Новый AsyncResult с преобразованным результатом
   */
  map<U>(fn: (value: T) => U): AsyncResult<U, E> {
    if (this.result.success) {
      return AsyncResult.success<U, E>(fn(this.result.data));
    }
    return AsyncResult.error<U, E>(this.result.error);
  }

  /**
   * Преобразует результат с возможной ошибкой
   * 
   * @template U - Новый тип результата
   * @param {(value: T) => AsyncResult<U, E>} fn - Функция преобразования, возвращающая AsyncResult
   * @returns {AsyncResult<U, E>} Результат преобразования
   */
  flatMap<U>(fn: (value: T) => AsyncResult<U, E>): AsyncResult<U, E> {
    if (this.result.success) {
      return fn(this.result.data);
    }
    return AsyncResult.error<U, E>(this.result.error);
  }
}
```

## Статический анализ кода

### 1. Настройка ESLint для TypeScript

```json
// .eslintrc.json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module",
    "project": [
      "./tsconfig.json",
      "./tsconfig.test.json"
    ]
  },
  "plugins": [
    "@typescript-eslint",
    "import",
    "unused-imports",
    "simple-import-sort"
  ],
  "rules": {
    // Основные правила TypeScript
    "@typescript-eslint/consistent-type-definitions": ["error", "interface"],
    "@typescript-eslint/prefer-readonly": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/no-non-null-assertion": "error",
    "@typescript-eslint/prefer-optional-chain": "error",
    "@typescript-eslint/prefer-nullish-coalescing": "error",
    
    // Правила импортов
    "import/order": "off",
    "simple-import-sort/imports": "error",
    "simple-import-sort/exports": "error",
    "unused-imports/no-unused-imports": "error",
    "unused-imports/no-unused-vars": [
      "warn",
      {
        "vars": "all",
        "varsIgnorePattern": "^_",
        "args": "after-used",
        "argsIgnorePattern": "^_"
      }
    ],
    
    // Правила стиля
    "@typescript-eslint/semi": ["error"],
    "object-curly-spacing": ["error", "always"],
    "comma-dangle": ["error", "always-multiline"],
    
    // Правила безопасности
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/no-misused-promises": "error",
    "@typescript-eslint/prefer-promise-reject-errors": "error"
  },
  "overrides": [
    {
      "files": ["*.test.ts", "*.spec.ts"],
      "rules": {
        "@typescript-eslint/no-explicit-any": "off",
        "@typescript-eslint/no-non-null-assertion": "off"
      }
    }
  ]
}
```

### 2. Правила для сложных метрик

```typescript
// Пример кода, который будет проверяться на сложность
export class ComplexCalculator {
  /**
   * Сложный метод с высокой cyclomatic complexity
   * 
   * @param {number} a - Первое число
   * @param {number} b - Второе число
   * @param {string} operation - Операция
   * @param {boolean} validate - Нужно ли валидировать
   * @returns {number} Результат операции
   */
  calculate(a: number, b: number, operation: string, validate: boolean = true): number {
    // Много условий - высокая сложность
    if (validate) {
      if (typeof a !== 'number' || typeof b !== 'number') {
        throw new Error('Arguments must be numbers');
      }
      
      if (isNaN(a) || isNaN(b)) {
        throw new Error('Arguments must not be NaN');
      }
      
      if (!operation) {
        throw new Error('Operation is required');
      }
    }
    
    switch (operation) {
      case 'add':
        if (a > 0 && b > 0) {
          return a + b;
        } else if (a < 0 && b < 0) {
          return a + b;
        } else {
          return a + b;
        }
      case 'subtract':
        if (a > b) {
          return a - b;
        } else {
          return b - a;
        }
      case 'multiply':
        if (a === 0 || b === 0) {
          return 0;
        } else if (a === 1) {
          return b;
        } else if (b === 1) {
          return a;
        } else {
          return a * b;
        }
      case 'divide':
        if (b === 0) {
          throw new Error('Division by zero');
        } else if (a === 0) {
          return 0;
        } else {
          return a / b;
        }
      default:
        throw new Error(`Unknown operation: ${operation}`);
    }
  }

  /**
   * Метод с высокой cognitive complexity из-за вложенных условий
   */
  processComplexLogic(data: any[]): any[] {
    const results: any[] = [];
    
    for (const item of data) {
      if (item.type === 'user') {
        if (item.isActive) {
          if (item.profile) {
            if (item.profile.permissions) {
              if (item.profile.permissions.length > 0) {
                results.push(item);
              } else {
                if (item.settings) {
                  if (item.settings.defaultPermissions) {
                    results.push({ ...item, permissions: item.settings.defaultPermissions });
                  }
                }
              }
            }
          }
        }
      } else if (item.type === 'group') {
        if (item.members) {
          if (item.members.length > 0) {
            for (const member of item.members) {
              if (member.isActive) {
                results.push(member);
              }
            }
          }
        }
      }
    }
    
    return results;
  }
}
```

### 3. Правила для улучшения читаемости

```json
{
  "rules": {
    // Ограничения на длину строк
    "max-len": ["error", { "code": 120 }],
    
    // Ограничения на количество параметров
    "@typescript-eslint/max-params": ["error", 4],
    
    // Ограничения на количество строк в функции
    "max-lines-per-function": ["error", { "max": 50, "skipBlankLines": true, "skipComments": true }],
    
    // Ограничения на вложенность
    "max-depth": ["error", 4],
    "max-nested-callbacks": ["error", 3],
    
    // Проверка на дублирующийся код
    "max-statements": ["error", 20],
    
    // Проверка на цикломатическую сложность
    "complexity": ["error", 10]
  }
}
```

## Линтеры и форматирование

### 1. Настройка Prettier

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 120,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "overrides": [
    {
      "files": "*.ts",
      "options": {
        "parser": "typescript"
      }
    }
  ]
}
```

### 2. Интеграция ESLint и Prettier

```json
// .eslintrc.json (продолжение)
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"  // Должен быть последним
  ],
  "plugins": [
    "@typescript-eslint",
    "prettier"
  ],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

### 3. Pre-commit хуки с lint-staged

```json
// package.json
{
  "scripts": {
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "type-check": "tsc --noEmit",
    "pre-commit": "lint-staged"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{json,md}": [
      "prettier --write",
      "git add"
    ]
  },
  "devDependencies": {
    "husky": "^8.0.0",
    "lint-staged": "^13.0.0",
    "prettier": "^2.8.0",
    "@typescript-eslint/eslint-plugin": "^5.45.0",
    "@typescript-eslint/parser": "^5.45.0",
    "eslint": "^8.28.0",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-prettier": "^4.2.1"
  }
}
```

## Код-ревью практики

### 1. Проверка типов и интерфейсов

```typescript
// ХОРОШО: Четко определенные интерфейсы
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface UpdateUserDto {
  name?: string;
  email?: string;
}

// ПЛОХО: Использование any или слишком широкие типы
// interface User {
//   [key: string]: any;  // Слишком общий
// }

// ХОРОШО: Конкретные типы для функций
class UserService {
  async updateUser(id: number, updates: UpdateUserDto): Promise<User | null> {
    // реализация
    return null;
  }
  
  async getUsers(filters: { 
    isActive?: boolean; 
    sortBy?: 'name' | 'email'; 
  } = {}): Promise<User[]> {
    // реализация
    return [];
  }
}

// ПЛОХО: Нечеткие типы
// async function badFunction(data: any): Promise<any> {
//   // непонятно, что принимает и возвращает
// }
```

### 2. Проверка обработки ошибок

```typescript
// ХОРОШО: Правильная обработка ошибок
class ApiService {
  async fetchData<T>(url: string): Promise<T> {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
      }
      
      const data = await response.json();
      return data;
    } catch (error) {
      if (error instanceof TypeError) {
        // Ошибка сети
        throw new Error('Network error occurred');
      }
      if (error instanceof Error) {
        // Переподнимаем ошибку с дополнительной информацией
        throw new Error(`API request failed: ${error.message}`);
      }
      // Неизвестная ошибка
      throw new Error('Unknown error occurred');
    }
  }
}

// ПЛОХО: Неправильная обработка ошибок
// class BadApiService {
//   async badFetchData(url: string) {
//     const response = await fetch(url);  // Не проверяем статус
//     return response.json();  // Не обрабатываем ошибки парсинга
//   }
// }
```

### 3. Проверка производительности и оптимизации

```typescript
// ХОРОШО: Оптимизированные операции
class OptimizedDataService {
  private data: Map<string, any> = new Map();
  
  // Быстрый доступ к данным O(1)
  get(key: string): any | undefined {
    return this.data.get(key);
  }
  
  // Эффективное обновление
  updateMany(updates: Array<{ key: string; value: any }>): void {
    for (const { key, value } of updates) {
      this.data.set(key, value);
    }
  }
  
  // Оптимизированный поиск
  findMatching(predicate: (value: any) => boolean): Array<{ key: string; value: any }> {
    const results: Array<{ key: string; value: any }> = [];
    
    for (const [key, value] of this.data.entries()) {
      if (predicate(value)) {
        results.push({ key, value });
      }
    }
    
    return results;
  }
}

// ПЛОХО: Неоптимизированные операции
// class InefficientDataService {
//   private data: any[] = [];
//   
//   // Медленный поиск O(n)
//   get(key: string): any {
//     return this.data.find(item => item.key === key)?.value;  // O(n) поиск
//   }
//   
//   // Медленное обновление
//   updateMany(updates: Array<{ key: string; value: any }>): void {
//     for (const update of updates) {
//       const index = this.data.findIndex(item => item.key === update.key);  // O(n) поиск
//       if (index !== -1) {
//         this.data[index].value = update.value;
//       } else {
//         this.data.push(update);
//       }
//     }
//   }
// }
```

## Документация API

### 1. OpenAPI/Swagger с TypeScript

```typescript
// Использование декораторов для генерации документации
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({
    description: 'Полное имя пользователя',
    example: 'John Doe',
    minLength: 2,
    maxLength: 100
  })
  name: string;

  @ApiProperty({
    description: 'Адрес электронной почты',
    example: 'john@example.com',
    format: 'email'
  })
  email: string;

  @ApiPropertyOptional({
    description: 'Статус активности пользователя',
    example: true,
    default: true
  })
  isActive?: boolean;
}

export class UserResponseDto {
  @ApiProperty({
    description: 'Уникальный идентификатор пользователя',
    example: 1
  })
  id: number;

  @ApiProperty({
    description: 'Полное имя пользователя',
    example: 'John Doe'
  })
  name: string;

  @ApiProperty({
    description: 'Адрес электронной почты',
    example: 'john@example.com'
  })
  email: string;

  @ApiProperty({
    description: 'Статус активности пользователя',
    example: true
  })
  isActive: boolean;

  @ApiProperty({
    description: 'Дата создания пользователя',
    type: 'string',
    format: 'date-time'
  })
  createdAt: Date;
}
```

### 2. Генерация документации с TypeDoc

```json
// typedoc.json
{
  "entryPoints": ["./src/index.ts"],
  "out": "./docs/api",
  "name": "My TypeScript Project",
  "includeVersion": true,
  "readme": "./README.md",
  "excludeNotExported": true,
  "excludePrivate": true,
  "excludeProtected": true,
  "theme": "default",
  "gaID": "",
  "hideGenerator": true,
  "categorizeByGroup": true,
  "sort": ["static-first", "visibility", "alphabetical"]
}
```

## Практические примеры из реальной разработки

### 1. Система валидации с документацией

```typescript
/**
 * Интерфейс для валидационного правила
 * 
 * @interface ValidationRule
 * @template T - Тип значения для валидации
 */
export interface ValidationRule<T> {
  /**
   * Функция валидации
   * 
   * @param {T} value - Значение для валидации
   * @returns {ValidationResult} Результат валидации
   */
  validate(value: T): ValidationResult;
  
  /**
   * Сообщение об ошибке
   */
  message: string;
}

/**
 * Результат валидации
 * 
 * @interface ValidationResult
 * @property {boolean} isValid - Успешна ли валидация
 * @property {string} [errorMessage] - Сообщение об ошибке (если есть)
 */
export interface ValidationResult {
  isValid: boolean;
  errorMessage?: string;
}

/**
 * Базовый класс валидатора
 * 
 * @abstract
 * @class BaseValidator
 * @template T - Тип значения для валидации
 */
export abstract class BaseValidator<T> {
  /**
   * Правила валидации
   */
  protected rules: ValidationRule<T>[] = [];

  /**
   * Добавляет правило валидации
   * 
   * @param {ValidationRule<T>} rule - Правило валидации
   * @returns {this} Текущий экземпляр валидатора
   */
  addRule(rule: ValidationRule<T>): this {
    this.rules.push(rule);
    return this;
  }

  /**
   * Выполняет валидацию значения
   * 
   * @param {T} value - Значение для валидации
   * @returns {ValidationResult[]} Массив результатов валидации
   */
  validate(value: T): ValidationResult[] {
    return this.rules.map(rule => rule.validate(value));
  }

  /**
   * Проверяет, прошло ли значение валидацию
   * 
   * @param {T} value - Значение для проверки
   * @returns {boolean} true если значение прошло валидацию, false если нет
   */
  isValid(value: T): boolean {
    const results = this.validate(value);
    return results.every(result => result.isValid);
  }

  /**
   * Возвращает сообщения об ошибках
   * 
   * @param {T} value - Значение для проверки
   * @returns {string[]} Массив сообщений об ошибках
   */
  getErrors(value: T): string[] {
    const results = this.validate(value);
    return results
      .filter(result => !result.isValid && result.errorMessage)
      .map(result => result.errorMessage!)
      .filter(Boolean);
  }
}

/**
 * Валидатор строк
 * 
 * @class StringValidator
 * @extends {BaseValidator<string>}
 */
export class StringValidator extends BaseValidator<string> {
  /**
   * Добавляет правило проверки минимальной длины
   * 
   * @param {number} minLength - Минимальная длина
   * @returns {this} Текущий экземпляр валидатора
   */
  minLength(minLength: number): this {
    this.addRule({
      validate: (value: string) => ({
        isValid: value.length >= minLength,
        errorMessage: value.length < minLength ? 
          `Строка должна содержать не менее ${minLength} символов` : undefined
      }),
      message: `Минимальная длина: ${minLength}`
    });
    return this;
  }

  /**
   * Добавляет правило проверки максимальной длины
   * 
   * @param {number} maxLength - Максимальная длина
   * @returns {this} Текущий экземпляр валидатора
   */
  maxLength(maxLength: number): this {
    this.addRule({
      validate: (value: string) => ({
        isValid: value.length <= maxLength,
        errorMessage: value.length > maxLength ? 
          `Строка не должна превышать ${maxLength} символов` : undefined
      }),
      message: `Максимальная длина: ${maxLength}`
    });
    return this;
  }

  /**
   * Добавляет правило проверки регулярного выражения
   * 
   * @param {RegExp} regex - Регулярное выражение
   * @param {string} [customMessage] - Пользовательское сообщение об ошибке
   * @returns {this} Текущий экземпляр валидатора
   */
  pattern(regex: RegExp, customMessage?: string): this {
    this.addRule({
      validate: (value: string) => ({
        isValid: regex.test(value),
        errorMessage: !regex.test(value) ? 
          customMessage || `Значение не соответствует шаблону: ${regex}` : undefined
      }),
      message: `Шаблон: ${regex}`
    });
    return this;
  }

  /**
   * Добавляет правило проверки email
   * 
   * @returns {this} Текущий экземпляр валидатора
   */
  email(): this {
    return this.pattern(
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      'Неверный формат email'
    );
  }

  /**
   * Добавляет правило проверки не пустого значения
   * 
   * @returns {this} Текущий экземпляр валидатора
   */
  required(): this {
    this.addRule({
      validate: (value: string) => ({
        isValid: value.trim().length > 0,
        errorMessage: value.trim().length === 0 ? 'Значение обязательно' : undefined
      }),
      message: 'Обязательное поле'
    });
    return this;
  }
}

// Использование
const emailValidator = new StringValidator()
  .required()
  .email()
  .maxLength(254);

const result = emailValidator.validate('test@example.com');
console.log(result); // [{ isValid: true }, { isValid: true }, { isValid: true }]
```

### 2. Система логирования с типизацией

```typescript
/**
 * Уровни логирования
 */
export enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error'
}

/**
 * Интерфейс для записи лога
 * 
 * @interface LogEntry
 * @property {Date} timestamp - Время записи
 * @property {LogLevel} level - Уровень логирования
 * @property {string} message - Сообщение
 * @property {any} [metadata] - Дополнительные данные
 */
export interface LogEntry {
  timestamp: Date;
  level: LogLevel;
  message: string;
  metadata?: any;
}

/**
 * Интерфейс для логгера
 */
export interface Logger {
  debug(message: string, metadata?: any): void;
  info(message: string, metadata?: any): void;
  warn(message: string, metadata?: any): void;
  error(message: string, error?: Error, metadata?: any): void;
}

/**
 * Базовый класс логгера
 * 
 * @class BaseLogger
 * @implements {Logger}
 */
export abstract class BaseLogger implements Logger {
  /**
   * Фильтр уровней логирования
   */
  protected levelFilter: LogLevel = LogLevel.DEBUG;

  /**
   * Устанавливает минимальный уровень логирования
   * 
   * @param {LogLevel} level - Уровень логирования
   */
  setLevel(level: LogLevel): void {
    this.levelFilter = level;
  }

  /**
   * Проверяет, должен ли логироваться уровень
   * 
   * @param {LogLevel} level - Уровень для проверки
   * @returns {boolean} true если уровень должен логироваться
   */
  protected shouldLog(level: LogLevel): boolean {
    const levels: LogLevel[] = [LogLevel.DEBUG, LogLevel.INFO, LogLevel.WARN, LogLevel.ERROR];
    const filterIndex = levels.indexOf(this.levelFilter);
    const levelIndex = levels.indexOf(level);
    return levelIndex >= filterIndex;
  }

  /**
   * Создает запись лога
   * 
   * @param {LogLevel} level - Уровень логирования
   * @param {string} message - Сообщение
   * @param {Error} [error] - Ошибка (если есть)
   * @param {any} [metadata] - Дополнительные данные
   * @returns {LogEntry} Запись лога
   */
  protected createLogEntry(
    level: LogLevel,
    message: string,
    error?: Error,
    metadata?: any
  ): LogEntry {
    const logEntry: LogEntry = {
      timestamp: new Date(),
      level,
      message,
      metadata: metadata || {}
    };

    if (error) {
      logEntry.metadata.error = {
        name: error.name,
        message: error.message,
        stack: error.stack
      };
    }

    return logEntry;
  }

  abstract debug(message: string, metadata?: any): void;
  abstract info(message: string, metadata?: any): void;
  abstract warn(message: string, metadata?: any): void;
  abstract error(message: string, error?: Error, metadata?: any): void;

  /**
   * Логирует запись
   * 
   * @param {LogEntry} entry - Запись лога
   */
  protected abstract log(entry: LogEntry): void;
}

/**
 * Консольный логгер
 * 
 * @class ConsoleLogger
 * @extends {BaseLogger}
 */
export class ConsoleLogger extends BaseLogger {
  debug(message: string, metadata?: any): void {
    if (this.shouldLog(LogLevel.DEBUG)) {
      const entry = this.createLogEntry(LogLevel.DEBUG, message, undefined, metadata);
      this.log(entry);
    }
  }

  info(message: string, metadata?: any): void {
    if (this.shouldLog(LogLevel.INFO)) {
      const entry = this.createLogEntry(LogLevel.INFO, message, undefined, metadata);
      this.log(entry);
    }
  }

  warn(message: string, metadata?: any): void {
    if (this.shouldLog(LogLevel.WARN)) {
      const entry = this.createLogEntry(LogLevel.WARN, message, undefined, metadata);
      this.log(entry);
    }
  }

  error(message: string, error?: Error, metadata?: any): void {
    if (this.shouldLog(LogLevel.ERROR)) {
      const entry = this.createLogEntry(LogLevel.ERROR, message, error, metadata);
      this.log(entry);
    }
  }

  protected log(entry: LogEntry): void {
    const output = `[${entry.level.toUpperCase()}] ${entry.message}`;
    
    switch (entry.level) {
      case LogLevel.ERROR:
        if (entry.metadata?.error) {
          console.error(output, entry.metadata.error);
        } else {
          console.error(output, entry.metadata);
        }
        break;
      case LogLevel.WARN:
        console.warn(output, entry.metadata);
        break;
      case LogLevel.INFO:
        console.info(output, entry.metadata);
        break;
      case LogLevel.DEBUG:
        console.debug(output, entry.metadata);
        break;
    }
  }
}
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки в документировании

```typescript
// ПЛОХО: Недостаточная документация
/**
 * Функция
 */
function badFunction(a: any, b: any) {
  return a + b;
}

// ХОРОШО: Полная документация
/**
 * Складывает два числа
 * 
 * @param {number} a - Первое число
 * @param {number} b - Второе число
 * @returns {number} Сумма двух чисел
 * 
 * @example
 * ```typescript
 * const result = add(2, 3); // 5
 * ```
 */
function add(a: number, b: number): number {
  return a + b;
}
```

### 2. Лучшие практики документирования

```typescript
// 1. Используйте consistent formatting
/**
 * Описание функции
 * 
 * @param {Type} param - Описание параметра
 * @returns {ReturnType} Описание возвращаемого значения
 */

// 2. Документируйте не только public API
/**
 * @internal
 * Внутренняя утилита для преобразования данных
 * Не предназначен для использования вне модуля
 */
function internalUtility(data: any): any {
  // реализация
}

// 3. Используйте @deprecated для устаревшего кода
/**
 * @deprecated С версии 2.0.0 используйте {@link newFunction} вместо этого
 * 
 * @param {string} oldParam - Старый параметр
 * @returns {string} Результат
 */
function deprecatedFunction(oldParam: string): string {
  return oldParam;
}

// 4. Документируйте типы и интерфейсы
/**
 * Интерфейс конфигурации приложения
 * 
 * @property {string} apiUrl - Базовый URL API
 * @property {number} timeout - Таймаут запросов в миллисекундах
 * @property {boolean} debug - Режим отладки
 */
export interface AppConfig {
  apiUrl: string;
  timeout: number;
  debug: boolean;
}
```

## Лучшие практики

1. **Документируйте public API** - все экспортируемые функции, классы и интерфейсы
2. **Используйте типобезопасные комментарии** - JSDoc с указанием типов
3. **Пишите читаемый код** - код должен быть самодокументированным
4. **Используйте линтеры** - для обеспечения качества и согласованности
5. **Регулярно обновляйте документацию** - особенно при изменениях API
6. **Используйте автоматическую генерацию** - для создания документации из исходного кода
7. **Проводите code review** - для проверки качества документации
8. **Тестируйте документацию** - убедитесь, что примеры из документации работают

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как документация помогает с типизацией
- [[Утилиты типов TypeScript|Утилиты типов]] - для создания вспомогательных типов
- [[architecture/documentation]] - общие принципы документирования
- [[testing/documentation]] - документирование тестов
- [[code-review/process]] - процессы код-ревью