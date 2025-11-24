---
aliases: [TSDoc стандарт, Документирование с TSDoc]
tags: [typescript, documentation, tsdoc]
---

# TSDoc в TypeScript проектах

## Обзор

TSDoc - это стандарт документирования TypeScript и JavaScript кода, разработанный Microsoft. Он предоставляет строгую и стандартизированную систему для документирования кода, которая поддерживается TypeScript компилятором и различными инструментами разработки.

## Основы TSDoc

TSDoc использует синтаксис, похожий на JSDoc, но с более строгими правилами и дополнительными возможностями, специально адаптированными для TypeScript.

```typescript
/**
 * Вычисляет сумму двух чисел
 * @param a - Первое число
 * @param b - Второе число
 * @returns Сумма a и b
 * @example
 * ```ts
 * const result = add(1, 2); // 3
 * ```
 */
function add(a: number, b: number): number {
    return a + b;
}
```

## Стандартные теги TSDoc

### @param
Документирование параметров функции:

```typescript
/**
 * Создает пользователя
 * @param name - Имя пользователя (обязательное)
 * @param age - Возраст пользователя (необязательный)
 * @param email - Email пользователя (необязательный)
 */
function createUser(
    name: string, 
    age?: number, 
    email?: string
): User {
    return { name, age, email };
}
```

### @returns / @returns
Описание возвращаемого значения:

```typescript
/**
 * Проверяет, является ли число четным
 * @param value - Число для проверки
 * @returns true, если число четное, иначе false
 */
function isEven(value: number): boolean {
    return value % 2 === 0;
}
```

### @example
Примеры использования с подсветкой синтаксиса:

```typescript
/**
 * Фильтрует массив чисел, оставляя только четные
 * @param numbers - Массив чисел для фильтрации
 * @returns Новый массив, содержащий только четные числа
 * @example
 * ```ts
 * const numbers = [1, 2, 3, 4, 5, 6];
 * const evens = filterEven(numbers);
 * console.log(evens); // [2, 4, 6]
 * ```
 */
function filterEven(numbers: number[]): number[] {
    return numbers.filter(isEven);
}
```

### @throws
Описание возможных исключений:

```typescript
/**
 * Делит одно число на другое
 * @param dividend - Делимое
 * @param divisor - Делитель
 * @returns Результат деления
 * @throws {Error} Если делитель равен нулю
 */
function divide(dividend: number, divisor: number): number {
    if (divisor === 0) {
        throw new Error('Деление на ноль невозможно');
    }
    return dividend / divisor;
}
```

## Расширенные теги TSDoc

### @defaultValue
Указывает значение по умолчанию для параметра:

```typescript
/**
 * Создает конфигурацию приложения
 * @param options - Опции конфигурации
 * @param options.timeout - Время ожидания запроса
 * @defaultValue 5000
 * @param options.retries - Количество попыток
 * @defaultValue 3
 */
function createConfig(options: {
    timeout?: number;
    retries?: number;
} = {}): AppConfig {
    return {
        timeout: options.timeout ?? 5000,
        retries: options.retries ?? 3
    };
}
```

### @deprecated
Помечает устаревшие API:

```typescript
/**
 * @deprecated Используйте вместо этого функцию `processDataV2`
 * Обрабатывает данные (устаревшая версия)
 * @param data - Входные данные
 */
function processData(data: string): string {
    // устаревшая реализация
    return data.toUpperCase();
}
```

### @beta @alpha @experimental
Помечает статус API:

```typescript
/**
 * @beta
 * Экспериментальная функция для обработки данных
 * Может измениться в будущих версиях
 * @param input - Входные данные
 * @returns Обработанные данные
 */
function experimentalProcess(input: DataInput): DataOutput {
    // реализация
    return process(input);
}
```

### @see
Ссылки на связанные элементы:

```typescript
/**
 * Выполняет асинхронную операцию
 * @see {@link syncOperation} для синхронной версии
 * @param input - Входные данные
 * @returns Promise с результатом
 */
async function asyncOperation(input: InputType): Promise<OutputType> {
    // реализация
    return await processAsync(input);
}
```

## Документирование сложных структур

### Интерфейсов и типов

```typescript
/**
 * Представляет информацию о пользователе
 * @public
 */
export interface User {
    /**
     * Уникальный идентификатор пользователя
     */
    id: string;
    
    /**
     * Имя пользователя
     */
    name: string;
    
    /**
     * Возраст пользователя
     * @remarks
     * Значение должно быть положительным числом
     */
    age: number;
    
    /**
     * Контактная информация
     */
    contactInfo?: {
        /**
         * Email пользователя
         */
        email: string;
        
        /**
         * Телефон пользователя
         */
        phone?: string;
    };
}
```

### Классов и методов

```typescript
/**
 * Менеджер пользователей
 * Обеспечивает операции CRUD для пользователей
 */
export class UserManager {
    private users: User[] = [];
    
    /**
     * Добавляет нового пользователя
     * @param user - Пользователь для добавления
     * @returns ID созданного пользователя
     * @throws {ValidationError} Если данные пользователя некорректны
     * @throws {DuplicateError} Если пользователь с таким ID уже существует
     */
    addUser(user: User): string {
        // реализация
        return user.id;
    }
    
    /**
     * Получает пользователя по ID
     * @param id - ID пользователя
     * @returns Пользователь или null, если не найден
     * @public
     */
    getUser(id: string): User | null {
        return this.users.find(u => u.id === id) || null;
    }
}
```

### Дженериков

```typescript
/**
 * Контейнер для хранения значений
 * @typeParam T - Тип хранимого значения
 */
export class Container<T> {
    private value: T;
    
    /**
     * @param initialValue - Начальное значение
     */
    constructor(initialValue: T) {
        this.value = initialValue;
    }
    
    /**
     * Получает текущее значение
     * @returns Текущее значение контейнера
     */
    getValue(): T {
        return this.value;
    }
    
    /**
     * Устанавливает новое значение
     * @param newValue - Новое значение для установки
     */
    setValue(newValue: T): void {
        this.value = newValue;
    }
}
```

## Интеграция с инструментами

### TypeScript компилятор

TSDoc комментарии обрабатываются TypeScript компилятором и используются для подсказок в IDE:

```typescript
/**
 * Выполняет сложную операцию с данными
 * @param data - Входные данные для обработки
 * @param options - Опции обработки
 * @returns Результат обработки
 * 
 * @remarks
 * Эта функция может занять продолжительное время выполнения
 * для больших объемов данных.
 * 
 * @example
 * ```ts
 * const result = await processData(
 *   largeDataset, 
 *   { timeout: 10000, compress: true }
 * );
 * ```
 */
async function processData(
    data: LargeDataStructure, 
    options: ProcessOptions
): Promise<ProcessResult> {
    // реализация
}
```

### Генерация документации

TSDoc поддерживается инструментами генерации документации, такими как TypeDoc:

```bash
npm install -g typedoc
typedoc src/index.ts --out docs/api
```

## Лучшие практики TSDoc

1. **Используйте стандартные теги**: Придерживайтесь официальных тегов TSDoc для консистентности.

2. **Документируйте публичные API**: Только те элементы, которые являются частью публичного API, должны быть задокументированы.

3. **Пишите читаемые примеры**: Примеры должны быть понятными и полезными для пользователей API.

4. **Используйте markdown форматирование**: В TSDoc можно использовать markdown для лучшего форматирования документации.

5. **Указывайте статус API**: Используйте теги @beta, @alpha, @experimental для обозначения статуса API.

6. **Избегайте избыточности**: Не дублируйте информацию, которая и так понятна из сигнатуры функции.

## Сравнение с JSDoc

| Особенность | JSDoc | TSDoc |
|-------------|-------|-------|
| Поддержка TypeScript | Ограниченная | Полная |
| Стандартизация | Различные реализации | Официальный стандарт |
| Интеграция с IDE | Базовая | Расширенная |
| Поддержка типов | Через @typedef | Нативная поддержка |

## Связанные темы

- [[JSDoc]] - Классический подход к документированию JavaScript
- [[Комментарии-в-коде]] - Принципы написания качественных комментариев
- [[Документация-API]] - Подробное руководство по документированию API
- [[Документирование-библиотек]] - Рекомендации по документированию библиотек