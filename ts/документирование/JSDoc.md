---
aliases: [JSDoc для TypeScript, Документирование функций JSDoc]
tags: [typescript, documentation, jsdoc]
---

# JSDoc в TypeScript проектах

## Обзор

JSDoc - это стандарт документирования JavaScript кода, который также отлично работает с TypeScript. Хотя TypeScript предоставляет собственную систему типов, JSDoc остается важным инструментом для подробного описания поведения функций, классов и других элементов кода.

## Основные теги JSDoc

### @param
Описание параметров функции:

```javascript
/**
 * Сложение двух чисел
 * @param {number} a - Первое слагаемое
 * @param {number} b - Второе слагаемое
 * @returns {number} Результат сложения
 */
function add(a, b) {
    return a + b;
}
```

### @returns / @return
Описание возвращаемого значения:

```javascript
/**
 * Получить длину строки
 * @param {string} str - Входная строка
 * @returns {number} Длина строки
 */
function getStringLength(str) {
    return str.length;
}
```

### @throws
Описание возможных исключений:

```javascript
/**
 * Деление двух чисел
 * @param {number} dividend - Делимое
 * @param {number} divisor - Делитель
 * @returns {number} Результат деления
 * @throws {Error} Если делитель равен нулю
 */
function divide(dividend, divisor) {
    if (divisor === 0) {
        throw new Error('Деление на ноль невозможно');
    }
    return dividend / divisor;
}
```

### @example
Примеры использования:

```javascript
/**
 * Найти максимальное значение в массиве
 * @param {number[]} numbers - Массив чисел
 * @returns {number|null} Максимальное значение или null для пустого массива
 * @example
 * const arr = [1, 5, 3, 9, 2];
 * const max = findMax(arr);
 * console.log(max); // 9
 */
function findMax(numbers) {
    if (numbers.length === 0) return null;
    return Math.max(...numbers);
}
```

## Использование JSDoc с TypeScript

### Совместимость с типами TypeScript

JSDoc можно использовать вместе с TypeScript типами:

```typescript
/**
 * Интерфейс пользователя
 * @typedef {Object} User
 * @property {string} id - Уникальный идентификатор пользователя
 * @property {string} name - Имя пользователя
 * @property {number} age - Возраст пользователя
 */

/**
 * Получить информацию о пользователе
 * @param {string} userId - Идентификатор пользователя
 * @returns {Promise<User>} Обещание с информацией о пользователе
 */
async function getUserInfo(userId: string): Promise<User> {
    // реализация
    return { id: userId, name: 'John Doe', age: 30 };
}
```

### Использование тега @template для дженериков

```typescript
/**
 * Обобщенная функция для создания пары значений
 * @template T
 * @template U
 * @param {T} first - Первое значение
 * @param {U} second - Второе значение
 * @returns {{first: T, second: U}} Объект с двумя значениями
 */
function createPair<T, U>(first: T, second: U): { first: T, second: U } {
    return { first, second };
}
```

## Расширенные возможности JSDoc

### @typedef для определения типов

```javascript
/**
 * @typedef {Object} Point
 * @property {number} x - Координата X
 * @property {number} y - Координата Y
 */

/**
 * @typedef {Object} Rectangle
 * @property {Point} topLeft - Верхняя левая точка
 * @property {Point} bottomRight - Нижняя правая точка
 */

/**
 * Проверить, находится ли точка внутри прямоугольника
 * @param {Point} point - Проверяемая точка
 * @param {Rectangle} rect - Прямоугольник
 * @returns {boolean} true, если точка внутри прямоугольника
 */
function isPointInRectangle(point, rect) {
    return point.x >= rect.topLeft.x && 
           point.x <= rect.bottomRight.x &&
           point.y >= rect.topLeft.y && 
           point.y <= rect.bottomRight.y;
}
```

### @callback для описания функций обратного вызова

```javascript
/**
 * Функция обратного вызова для обработки результата
 * @callback ProcessCallback
 * @param {string} error - Ошибка, если произошла
 * @param {any} result - Результат обработки
 */

/**
 * Обработать данные асинхронно
 * @param {any} data - Входные данные
 * @param {ProcessCallback} callback - Функция обратного вызова
 */
function processData(data, callback) {
    try {
        // обработка данных
        const result = data.toString().toUpperCase();
        callback(null, result);
    } catch (error) {
        callback(error, null);
    }
}
```

## Настройка JSDoc в TypeScript

### Конфигурация в tsconfig.json

```json
{
  "compilerOptions": {
    "jsDocParsingMode": "parseForTypeErrors",
    "allowJs": true,
    "checkJs": true
  }
}
```

### Автодополнение и подсказки

TypeScript будет использовать JSDoc для предоставления информации в IDE:

```typescript
/**
 * Функция для форматирования даты
 * @param {Date} date - Дата для форматирования
 * @param {string} format - Формат (например, 'YYYY-MM-DD')
 * @returns {string} Отформатированная строка даты
 */
function formatDate(date: Date, format: string = 'YYYY-MM-DD'): string {
    // реализация
    return date.toISOString().split('T')[0];
}

// При вызове функции TypeScript покажет документацию
const today = formatDate(new Date()); // Подсказка будет отображаться в IDE
```

## Лучшие практики JSDoc

1. **Документируйте публичные API**: Все функции, классы и интерфейсы, предназначенные для использования внешними пользователями, должны быть задокументированы.

2. **Используйте примеры**: Включайте примеры использования в документацию с тегом `@example`.

3. **Описывайте сложные алгоритмы**: Если функция реализует сложную логику, добавьте подробные комментарии.

4. **Указывайте возможные исключения**: Используйте тег `@throws` для описания ситуаций, в которых функция может выбросить исключение.

5. **Поддерживайте актуальность**: Регулярно обновляйте документацию при изменении кода.

## Интеграция с инструментами генерации документации

JSDoc может использоваться вместе с инструментами для генерации HTML документации:

```bash
npm install -g jsdoc
jsdoc src/**/*.ts -d docs
```

## Связанные темы

- [[TSDoc]] - Современный стандарт документирования TypeScript
- [[Комментарии-в-коде]] - Принципы написания качественных комментариев
- [[Документация-API]] - Подробное руководство по документированию API
- [[Документирование-библиотек]] - Рекомендации по документированию библиотек