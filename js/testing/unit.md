# Модульное тестирование в JavaScript

## Обзор

Модульное тестирование (unit testing) - это практика тестирования отдельных компонентов или "модулей" программного обеспечения изолированно. В JavaScript модульные тесты проверяют отдельные функции, классы или модули, чтобы убедиться, что они работают правильно.

## Основные концепции модульного тестирования

### Три A (AAA) подход
- **Arrange (Организация)**: Подготовка тестовых данных и настройка
- **Act (Действие)**: Вызов тестируемой функции/метода
- **Assert (Проверка)**: Проверка результата

```javascript
// Пример функции для тестирования
function add(a, b) {
    return a + b;
}

// Тест по паттерну AAA
function testAddFunction() {
    // Arrange (Организация)
    const num1 = 5;
    const num2 = 3;
    const expected = 8;
    
    // Act (Действие)
    const result = add(num1, num2);
    
    // Assert (Проверка)
    if (result === expected) {
        console.log('✓ Тест пройден: add(5, 3) = 8');
    } else {
        console.log(`✗ Тест не пройден: ожидается ${expected}, получено ${result}`);
    }
}

testAddFunction();
```

## Популярные фреймворки для тестирования

### Jest

```javascript
// math.js
export function add(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw new Error('Оба аргумента должны быть числами');
    }
    return a + b;
}

export function isEven(num) {
    return num % 2 === 0;
}

// math.test.js
import { add, isEven } from './math.js';

describe('Функция сложения', () => {
    test('должна корректно складывать два числа', () => {
        // Arrange
        const num1 = 5;
        const num2 = 3;
        const expected = 8;
        
        // Act
        const result = add(num1, num2);
        
        // Assert
        expect(result).toBe(expected);
    });
    
    test('должна выбрасывать ошибку при передаче нечисловых аргументов', () => {
        expect(() => add('5', 3)).toThrow('Оба аргумента должны быть числами');
        expect(() => add(5, '3')).toThrow('Оба аргумента должны быть числами');
        expect(() => add('5', '3')).toThrow('Оба аргумента должны быть числами');
    });
    
    test('должна корректно обрабатывать отрицательные числа', () => {
        expect(add(-1, 1)).toBe(0);
        expect(add(-5, -3)).toBe(-8);
        expect(add(-5, 3)).toBe(-2);
    });
});

describe('Функция проверки четности', () => {
    test('должна возвращать true для четных чисел', () => {
        expect(isEven(2)).toBe(true);
        expect(isEven(0)).toBe(true);
        expect(isEven(-4)).toBe(true);
    });
    
    test('должна возвращать false для нечетных чисел', () => {
        expect(isEven(1)).toBe(false);
        expect(isEven(3)).toBe(false);
        expect(isEven(-5)).toBe(false);
    });
});
```

### Встроенные тесты (без фреймворка)

```javascript
// simple-test-framework.js
class SimpleTestFramework {
    constructor() {
        this.tests = [];
        this.passed = 0;
        this.failed = 0;
    }
    
    test(description, testFunction) {
        this.tests.push({ description, testFunction });
    }
    
    async run() {
        console.log(`Запуск ${this.tests.length} тестов...\n`);
        
        for (const { description, testFunction } of this.tests) {
            try {
                await testFunction();
                console.log(`✓ ${description}`);
                this.passed++;
            } catch (error) {
                console.log(`✗ ${description}`);
                console.log(`  Ошибка: ${error.message}`);
                this.failed++;
            }
        }
        
        console.log(`\nРезультаты: ${this.passed} пройдено, ${this.failed} не пройдено`);
    }
    
    expect(actual) {
        return new Assertion(actual);
    }
}

class Assertion {
    constructor(actual) {
        this.actual = actual;
    }
    
    toBe(expected) {
        if (this.actual !== expected) {
            throw new Error(`Ожидается ${expected}, получено ${this.actual}`);
        }
    }
    
    toEqual(expected) {
        if (JSON.stringify(this.actual) !== JSON.stringify(expected)) {
            throw new Error(`Ожидается ${JSON.stringify(expected)}, получено ${JSON.stringify(this.actual)}`);
        }
    }
    
    toThrow(expectedError) {
        throw new Error('toThrow не реализован для простого фреймворка');
    }
}

// Использование простого фреймворка
const testFramework = new SimpleTestFramework();

function multiply(a, b) {
    return a * b;
}

testFramework.test('multiply: умножение двух положительных чисел', () => {
    const result = multiply(3, 4);
    testFramework.expect(result).toBe(12);
});

testFramework.test('multiply: умножение на ноль', () => {
    testFramework.expect(multiply(5, 0)).toBe(0);
    testFramework.expect(multiply(0, 7)).toBe(0);
});

testFramework.test('multiply: умножение отрицательных чисел', () => {
    testFramework.expect(multiply(-3, 4)).toBe(-12);
    testFramework.expect(multiply(-3, -4)).toBe(12);
});

// Запуск тестов
// testFramework.run();
```

## Тестирование асинхронных функций

```javascript
// async-utils.js
export async function fetchData(url) {
    if (!url) {
        throw new Error('URL обязателен');
    }
    
    // Имитация асинхронного запроса
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve({ data: `Данные с ${url}`, timestamp: Date.now() });
        }, 100);
    });
}

export async function processItems(items) {
    if (!Array.isArray(items)) {
        throw new Error('items должен быть массивом');
    }
    
    return items.map(item => ({
        ...item,
        processed: true,
        id: item.id || Math.random()
    }));
}
```

```javascript
// async-utils.test.js
import { fetchData, processItems } from './async-utils.js';

describe('Асинхронные утилиты', () => {
    test('fetchData: успешно получает данные', async () => {
        const url = 'https://api.example.com/data';
        const result = await fetchData(url);
        
        expect(result).toHaveProperty('data');
        expect(result.data).toContain(url);
        expect(result).toHaveProperty('timestamp');
        expect(typeof result.timestamp).toBe('number');
    });
    
    test('fetchData: выбрасывает ошибку при отсутствии URL', async () => {
        await expect(fetchData()).rejects.toThrow('URL обязателен');
        await expect(fetchData('')).rejects.toThrow('URL обязателен');
    });
    
    test('processItems: обрабатывает массив объектов', async () => {
        const items = [
            { name: 'Item 1' },
            { name: 'Item 2', id: 123 }
        ];
        
        const result = await processItems(items);
        
        expect(result).toHaveLength(2);
        expect(result[0]).toEqual({
            name: 'Item 1',
            processed: true,
            id: expect.any(Number) // ID генерируется случайно
        });
        
        expect(result[1]).toEqual({
            name: 'Item 2',
            processed: true,
            id: 123
        });
    });
    
    test('processItems: выбрасывает ошибку при передаче не массива', async () => {
        const nonArrays = [null, undefined, {}, 'string', 123, true];
        
        for (const nonArray of nonArrays) {
            await expect(processItems(nonArray)).rejects.toThrow('items должен быть массивом');
        }
    });
});
```

## Mock-объекты и шпионы

```javascript
// user-service.js
export class UserService {
    constructor(apiClient) {
        this.apiClient = apiClient;
    }
    
    async getUser(id) {
        if (!id) {
            throw new Error('ID пользователя обязателен');
        }
        
        return await this.apiClient.get(`/users/${id}`);
    }
    
    async createUser(userData) {
        if (!userData || !userData.name) {
            throw new Error('Имя пользователя обязательно');
        }
        
        return await this.apiClient.post('/users', userData);
    }
    
    async updateUser(id, updates) {
        if (!id || !updates) {
            throw new Error('ID и обновления обязательны');
        }
        
        const user = await this.getUser(id);
        const updatedUser = { ...user, ...updates };
        
        return await this.apiClient.put(`/users/${id}`, updatedUser);
    }
}
```

```javascript
// user-service.test.js
import { UserService } from './user-service.js';

describe('UserService', () => {
    let mockApiClient;
    let userService;
    
    beforeEach(() => {
        // Создание mock-объекта для API клиента
        mockApiClient = {
            get: jest.fn(),
            post: jest.fn(),
            put: jest.fn()
        };
        
        userService = new UserService(mockApiClient);
    });
    
    test('getUser: вызывает API с правильным URL', async () => {
        const userId = 123;
        const mockUser = { id: userId, name: 'Иван' };
        
        mockApiClient.get.mockResolvedValue(mockUser);
        
        const result = await userService.getUser(userId);
        
        expect(mockApiClient.get).toHaveBeenCalledWith(`/users/${userId}`);
        expect(result).toEqual(mockUser);
    });
    
    test('createUser: проверяет наличие имени и вызывает POST', async () => {
        const userData = { name: 'Мария', email: 'maria@example.com' };
        const mockResponse = { id: 456, ...userData };
        
        mockApiClient.post.mockResolvedValue(mockResponse);
        
        const result = await userService.createUser(userData);
        
        expect(mockApiClient.post).toHaveBeenCalledWith('/users', userData);
        expect(result).toEqual(mockResponse);
    });
    
    test('createUser: выбрасывает ошибку при отсутствии имени', async () => {
        await expect(userService.createUser({})).rejects.toThrow('Имя пользователя обязательно');
        await expect(userService.createUser({ email: 'test@example.com' })).rejects.toThrow('Имя пользователя обязательно');
    });
    
    test('updateUser: получает пользователя и обновляет его', async () => {
        const userId = 123;
        const existingUser = { id: userId, name: 'Иван', email: 'ivan@example.com' };
        const updates = { name: 'Иван Петров', age: 30 };
        const expectedUser = { ...existingUser, ...updates };
        
        mockApiClient.get.mockResolvedValue(existingUser);
        mockApiClient.put.mockResolvedValue(expectedUser);
        
        const result = await userService.updateUser(userId, updates);
        
        expect(mockApiClient.get).toHaveBeenCalledWith(`/users/${userId}`);
        expect(mockApiClient.put).toHaveBeenCalledWith(`/users/${userId}`, expectedUser);
        expect(result).toEqual(expectedUser);
    });
});
```

## Практические советы по модульному тестированию

1. **Тестируйте поведение, а не реализацию**
2. **Создавайте независимые тесты**
3. **Используйте понятные описания тестов**
4. **Тестируйте граничные условия**
5. **Изолируйте тестируемый код от внешних зависимостей**
6. **Покрывайте как успешные, так и неуспешные сценарии**

```javascript
// Пример хорошего теста
describe('Калькулятор', () => {
    test('деление: положительных чисел', () => {
        expect(divide(10, 2)).toBe(5);
    });
    
    test('деление: на отрицательное число', () => {
        expect(divide(10, -2)).toBe(-5);
    });
    
    test('деление: на ноль должно выбрасывать ошибку', () => {
        expect(() => divide(10, 0)).toThrow('Деление на ноль');
    });
    
    test('деление: граничные значения', () => {
        expect(divide(0, 5)).toBe(0);
        expect(divide(Number.MAX_VALUE, 1)).toBe(Number.MAX_VALUE);
    });
});
```