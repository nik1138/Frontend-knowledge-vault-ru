# Продвинутые техники тестирования в JavaScript

## Введение

Тестирование в JavaScript вышло за рамки простых юнит-тестов. Современные подходы включают mock-объекты, стратегии тестирования, тестирование производительности и интеграционные тесты. В этой статье мы рассмотрим продвинутые техники, используемые в промышленной разработке.

## Продвинутые техники mock-объектов

### Мокирование функций и модулей

```javascript
// Пример с Jest
describe('UserService', () => {
    let userService;
    let apiClient;
    
    beforeEach(() => {
        // Мокирование модуля
        apiClient = {
            getUser: jest.fn(),
            updateUser: jest.fn()
        };
        
        userService = new UserService(apiClient);
    });
    
    test('должен получить пользователя', async () => {
        // Подготовка мока
        const mockUser = { id: 1, name: 'Иван' };
        apiClient.getUser.mockResolvedValue(mockUser);
        
        // Выполнение
        const user = await userService.getUser(1);
        
        // Проверка
        expect(apiClient.getUser).toHaveBeenCalledWith(1);
        expect(user).toEqual(mockUser);
    });
    
    test('должен обработать ошибку получения пользователя', async () => {
        // Мокирование ошибки
        apiClient.getUser.mockRejectedValue(new Error('Пользователь не найден'));
        
        await expect(userService.getUser(999)).rejects.toThrow('Пользователь не найден');
        expect(apiClient.getUser).toHaveBeenCalledWith(999);
    });
});

// Пример с мокированием модуля целиком
// __mocks__/apiClient.js
export const getUser = jest.fn();
export const updateUser = jest.fn();

// В тесте
jest.mock('../apiClient'); // Автоматически подставит мок из __mocks__

import { getUser, updateUser } from '../apiClient';
import UserService from '../UserService';

test('мокирование всего модуля', () => {
    getUser.mockResolvedValue({ id: 1, name: 'Иван' });
    
    const userService = new UserService();
    // ...
});
```

### Мокирование асинхронных операций

```javascript
describe('Async Data Fetching', () => {
    test('должен корректно обрабатывать задержки', async () => {
        // Мокирование setTimeout
        jest.useFakeTimers();
        
        const asyncFunction = jest.fn().mockImplementation(() => {
            return new Promise(resolve => {
                setTimeout(() => resolve('результат'), 1000);
            });
        });
        
        const promise = asyncFunction();
        
        // Быстрое продвижение времени
        jest.advanceTimersByTime(1000);
        
        await expect(promise).resolves.toBe('результат');
        
        // Возврат к реальному времени
        jest.useRealTimers();
    });
    
    test('должен обрабатывать промисы с ошибками', async () => {
        const failingAsyncFunction = jest.fn().mockImplementation(() => {
            return new Promise((_, reject) => {
                setTimeout(() => reject(new Error('Ошибка')), 500);
            });
        });
        
        jest.useFakeTimers();
        
        const promise = failingAsyncFunction();
        jest.advanceTimersByTime(500);
        
        await expect(promise).rejects.toThrow('Ошибка');
        
        jest.useRealTimers();
    });
});
```

### Мокирование сетевых запросов

```javascript
// Мокирование fetch
describe('API Service', () => {
    beforeEach(() => {
        // Мокирование fetch
        global.fetch = jest.fn();
    });
    
    afterEach(() => {
        jest.restoreAllMocks();
    });
    
    test('должен выполнить GET запрос', async () => {
        const mockResponse = { id: 1, name: 'Иван' };
        
        global.fetch.mockResolvedValue({
            ok: true,
            json: async () => mockResponse
        });
        
        const response = await fetch('/api/users/1');
        const data = await response.json();
        
        expect(global.fetch).toHaveBeenCalledWith('/api/users/1', undefined);
        expect(data).toEqual(mockResponse);
    });
    
    test('должен обработать ошибку сети', async () => {
        global.fetch.mockResolvedValue({
            ok: false,
            status: 404,
            statusText: 'Not Found'
        });
        
        const response = await fetch('/api/users/999');
        
        expect(response.ok).toBe(false);
        expect(response.status).toBe(404);
    });
});

// Альтернатива: использование msw (Mock Service Worker)
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
    rest.get('/api/users/:userId', (req, res, ctx) => {
        const { userId } = req.params;
        
        if (userId === '1') {
            return res(
                ctx.json({ id: 1, name: 'Иван' }),
                ctx.status(200)
            );
        }
        
        return res(
            ctx.status(404),
            ctx.json({ error: 'Пользователь не найден' })
        );
    })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('мокирование с msw', async () => {
    const response = await fetch('/api/users/1');
    const data = await response.json();
    
    expect(data).toEqual({ id: 1, name: 'Иван' });
});
```

## Стратегии тестирования

### Тестирование по принципу AAA (Arrange, Act, Assert)

```javascript
describe('UserValidator', () => {
    test('валидирует корректного пользователя', () => {
        // Arrange - Подготовка
        const validator = new UserValidator();
        const validUser = {
            name: 'Иван',
            email: 'ivan@example.com',
            age: 25
        };
        
        // Act - Выполнение
        const result = validator.validate(validUser);
        
        // Assert - Проверка
        expect(result.isValid).toBe(true);
        expect(result.errors).toHaveLength(0);
    });
    
    test('находит ошибки валидации', () => {
        // Arrange
        const validator = new UserValidator();
        const invalidUser = {
            name: '', // Пустое имя
            email: 'invalid-email', // Невалидный email
            age: -5 // Отрицательный возраст
        };
        
        // Act
        const result = validator.validate(invalidUser);
        
        // Assert
        expect(result.isValid).toBe(false);
        expect(result.errors).toContain('Имя не может быть пустым');
        expect(result.errors).toContain('Email должен быть валидным');
        expect(result.errors).toContain('Возраст должен быть положительным');
    });
});
```

### Тестирование по принципу Given-When-Then

```javascript
describe('ShoppingCart', () => {
    test('увеличивает количество товара при добавлении существующего', () => {
        // Given - Дано
        const cart = new ShoppingCart();
        const item = { id: 1, name: 'Товар', price: 100 };
        
        cart.addItem(item, 2);
        
        // When - Когда
        cart.addItem(item, 3); // Добавляем еще 3 штуки
        
        // Then - Тогда
        const cartItem = cart.getItem(1);
        expect(cartItem.quantity).toBe(5); // 2 + 3 = 5
        expect(cart.getTotalPrice()).toBe(500); // 5 * 100
    });
});
```

### Пирамида тестирования

```javascript
// Unit tests (много)
describe('Calculator', () => {
    test('складывает два числа', () => {
        const calc = new Calculator();
        expect(calc.add(2, 3)).toBe(5);
    });
});

// Integration tests (среднее количество)
describe('UserService integration', () => {
    test('сохраняет пользователя в базу данных', async () => {
        const userService = new UserService(database);
        const user = { name: 'Иван', email: 'ivan@example.com' };
        
        const savedUser = await userService.createUser(user);
        
        expect(savedUser.id).toBeDefined();
        expect(savedUser.name).toBe('Иван');
        
        // Проверка, что пользователь действительно сохранен в БД
        const retrievedUser = await database.findUser(savedUser.id);
        expect(retrievedUser).toEqual(savedUser);
    });
});

// End-to-end tests (мало, но ключевые пути)
// Используется Cypress, Playwright или Selenium
describe('User registration flow', () => {
    test('новый пользователь может зарегистрироваться', () => {
        cy.visit('/register');
        cy.get('[data-testid="name-input"]').type('Иван');
        cy.get('[data-testid="email-input"]').type('ivan@example.com');
        cy.get('[data-testid="password-input"]').type('password123');
        cy.get('[data-testid="submit-button"]').click();
        
        cy.url().should('include', '/dashboard');
        cy.contains('Добро пожаловать, Иван!');
    });
});
```

## Тестирование асинхронного кода

### Тестирование Promises

```javascript
describe('Async operations', () => {
    test('должен разрешить промис с правильным значением', async () => {
        const asyncFunction = () => Promise.resolve('успех');
        
        const result = await asyncFunction();
        expect(result).toBe('успех');
    });
    
    test('должен отклонить промис с правильной ошибкой', async () => {
        const asyncFunction = () => Promise.reject(new Error('ошибка'));
        
        await expect(asyncFunction()).rejects.toThrow('ошибка');
    });
    
    test('должен корректно обрабатывать race conditions', async () => {
        const fastPromise = new Promise(resolve => 
            setTimeout(() => resolve('быстрый'), 100)
        );
        
        const slowPromise = new Promise(resolve => 
            setTimeout(() => resolve('медленный'), 200)
        );
        
        const winner = await Promise.race([fastPromise, slowPromise]);
        expect(winner).toBe('быстрый');
    });
});
```

### Тестирование с использованием async/await и done callback

```javascript
describe('Callback functions', () => {
    test('callback функция вызывается с правильными аргументами', (done) => {
        const callback = (error, result) => {
            try {
                expect(error).toBeNull();
                expect(result).toBe('успех');
                done(); // Завершаем тест
            } catch (err) {
                done(err); // Передаем ошибку в done
            }
        };
        
        someAsyncFunction(callback);
    });
    
    test('async/await с callback функцией', async () => {
        const result = await new Promise((resolve, reject) => {
            someAsyncFunction((error, data) => {
                if (error) {
                    reject(error);
                } else {
                    resolve(data);
                }
            });
        });
        
        expect(result).toBe('ожидаемый результат');
    });
});
```

## Тестирование производительности

### Измерение производительности

```javascript
describe('Performance tests', () => {
    test('функция сортировки работает быстро', () => {
        const largeArray = Array.from({ length: 10000 }, () => 
            Math.floor(Math.random() * 10000)
        );
        
        const startTime = performance.now();
        const sortedArray = sortFunction(largeArray);
        const endTime = performance.now();
        
        const executionTime = endTime - startTime;
        
        // Проверяем, что сортировка заняла менее 100ms
        expect(executionTime).toBeLessThan(100);
        
        // Проверяем, что массив отсортирован
        expect(isSorted(sortedArray)).toBe(true);
    });
    
    test('поиск элемента в массиве - O(log n)', () => {
        const sortedArray = Array.from({ length: 10000 }, (_, i) => i);
        const searchValue = 5000;
        
        const startTime = performance.now();
        const index = binarySearch(sortedArray, searchValue);
        const endTime = performance.now();
        
        const executionTime = endTime - startTime;
        
        expect(index).toBe(searchValue);
        expect(executionTime).toBeLessThan(1); // Должно быть быстро
    });
});
```

### Сравнение алгоритмов

```javascript
describe('Algorithm comparison', () => {
    const testData = Array.from({ length: 1000 }, () => 
        ({ id: Math.random(), value: Math.random() })
    );
    
    test('Map быстрее чем for loop для трансформации', () => {
        // Тест Map
        const mapStartTime = performance.now();
        const mapResult = testData.map(item => ({ ...item, doubled: item.value * 2 }));
        const mapTime = performance.now() - mapStartTime;
        
        // Тест for loop
        const forStartTime = performance.now();
        const forResult = [];
        for (let i = 0; i < testData.length; i++) {
            forResult.push({ ...testData[i], doubled: testData[i].value * 2 });
        }
        const forTime = performance.now() - forStartTime;
        
        // Вывод результатов (для анализа)
        console.log(`Map: ${mapTime}ms, For: ${forTime}ms`);
        
        // Проверка корректности
        expect(mapResult).toEqual(forResult);
    });
});
```

## Современные библиотеки тестирования

### Jest продвинутые возможности

```javascript
describe('Jest advanced features', () => {
    // Snapshot testing
    test('рендерит компонент корректно', () => {
        const component = renderUserCard({ name: 'Иван', email: 'ivan@example.com' });
        
        expect(component).toMatchSnapshot();
    });
    
    // Mock timers
    test('setTimeout работает корректно', () => {
        jest.useFakeTimers();
        
        const callback = jest.fn();
        const delay = 1000;
        
        setTimeout(callback, delay);
        
        // Проверяем, что callback еще не вызван
        expect(callback).not.toHaveBeenCalled();
        
        // "Прокручиваем" таймер
        jest.advanceTimersByTime(delay);
        
        // Проверяем, что callback вызван
        expect(callback).toHaveBeenCalledTimes(1);
        
        jest.useRealTimers();
    });
    
    // Mock modules
    test('мокирование модуля с возвращением разных значений', () => {
        const mockMath = Object.create(global.Math);
        mockMath.random = jest.fn().mockReturnValue(0.5);
        
        global.Math = mockMath;
        
        const randomValue = Math.random();
        expect(randomValue).toBe(0.5);
        
        // Восстанавливаем оригинальный Math
        global.Math = originalMath;
    });
    
    // Coverage-guided testing
    test('покрывает все ветки условий', () => {
        const processUser = (user) => {
            if (!user) return null;
            if (user.age < 18) return 'Несовершеннолетний';
            if (user.age >= 65) return 'Пенсионер';
            return 'Взрослый';
        };
        
        expect(processUser(null)).toBeNull();
        expect(processUser({ age: 15 })).toBe('Несовершеннолетний');
        expect(processUser({ age: 70 })).toBe('Пенсионер');
        expect(processUser({ age: 30 })).toBe('Взрослый');
    });
});
```

### Testing Library подход

```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom';

describe('React component testing', () => {
    test('пользователь может ввести данные и отправить форму', async () => {
        render(<UserForm />);
        
        // Находим элементы по роли или тексту
        const nameInput = screen.getByRole('textbox', { name: /имя/i });
        const submitButton = screen.getByRole('button', { name: /отправить/i });
        
        // Взаимодействуем с элементами
        fireEvent.change(nameInput, { target: { value: 'Иван' } });
        fireEvent.click(submitButton);
        
        // Ожидаем асинхронных изменений
        await waitFor(() => {
            expect(screen.getByText(/спасибо за регистрацию/i)).toBeInTheDocument();
        });
    });
    
    test('показывает ошибки валидации', async () => {
        render(<UserForm />);
        
        const submitButton = screen.getByRole('button', { name: /отправить/i });
        fireEvent.click(submitButton);
        
        // Проверяем, что ошибки отображаются
        expect(screen.getByText(/имя обязательно/i)).toBeInTheDocument();
        expect(screen.getByText(/email обязателен/i)).toBeInTheDocument();
    });
});
```

## Лучшие практики тестирования

### Организация тестов

```javascript
// Хорошая структура тестов
describe('User Service', () => {
    describe('getUser', () => {
        describe('when user exists', () => {
            test('returns user object', async () => {
                // тест
            });
        });
        
        describe('when user does not exist', () => {
            test('throws UserNotFound error', async () => {
                // тест
            });
        });
    });
    
    describe('createUser', () => {
        test('creates user with valid data', async () => {
            // тест
        });
        
        test('validates user data before creation', async () => {
            // тест
        });
    });
});
```

### Тестирование ошибок и граничных условий

```javascript
describe('Boundary testing', () => {
    test('обрабатывает минимальное значение', () => {
        const validator = new AgeValidator();
        expect(validator.validate(0)).toBe(false); // если 0 лет не разрешено
        expect(validator.validate(1)).toBe(true);  // если 1 год разрешен
    });
    
    test('обрабатывает максимальное значение', () => {
        const validator = new AgeValidator();
        expect(validator.validate(150)).toBe(false); // если 150 лет не разрешено
        expect(validator.validate(120)).toBe(true);  // если 120 лет разрешено
    });
    
    test('обрабатывает null и undefined', () => {
        const processor = new DataProcessor();
        
        expect(() => processor.process(null)).toThrow();
        expect(() => processor.process(undefined)).toThrow();
        expect(() => processor.process('')).not.toThrow();
    });
});
```

## Примеры из промышленной разработки

### Тестирование API клиента

```javascript
class ApiClient {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = options.headers || {};
        this.timeout = options.timeout || 5000;
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const config = {
            headers: { ...this.defaultHeaders, ...options.headers },
            ...options
        };
        
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.timeout);
        
        try {
            const response = await fetch(url, {
                ...config,
                signal: controller.signal
            });
            
            clearTimeout(timeoutId);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            return await response.json();
        } catch (error) {
            clearTimeout(timeoutId);
            
            if (error.name === 'AbortError') {
                throw new Error('Request timeout');
            }
            
            throw error;
        }
    }
    
    async get(endpoint) {
        return this.request(endpoint, { method: 'GET' });
    }
    
    async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data),
            headers: {
                'Content-Type': 'application/json',
                ...this.defaultHeaders
            }
        });
    }
}

// Тесты для API клиента
describe('ApiClient', () => {
    let apiClient;
    let originalFetch;
    
    beforeEach(() => {
        originalFetch = global.fetch;
        global.fetch = jest.fn();
        apiClient = new ApiClient('https://api.example.com', { timeout: 2000 });
    });
    
    afterEach(() => {
        global.fetch = originalFetch;
    });
    
    test('делает GET запрос с правильными параметрами', async () => {
        const mockResponse = { id: 1, name: 'Иван' };
        
        global.fetch.mockResolvedValue({
            ok: true,
            json: async () => mockResponse
        });
        
        const result = await apiClient.get('/users/1');
        
        expect(global.fetch).toHaveBeenCalledWith(
            'https://api.example.com/users/1',
            expect.objectContaining({
                method: 'GET',
                headers: {}
            })
        );
        
        expect(result).toEqual(mockResponse);
    });
    
    test('делает POST запрос с правильными заголовками', async () => {
        const postData = { name: 'Иван', email: 'ivan@example.com' };
        const mockResponse = { id: 2, ...postData };
        
        global.fetch.mockResolvedValue({
            ok: true,
            json: async () => mockResponse
        });
        
        const result = await apiClient.post('/users', postData);
        
        expect(global.fetch).toHaveBeenCalledWith(
            'https://api.example.com/users',
            expect.objectContaining({
                method: 'POST',
                body: JSON.stringify(postData),
                headers: expect.objectContaining({
                    'Content-Type': 'application/json'
                })
            })
        );
        
        expect(result).toEqual(mockResponse);
    });
    
    test('обрабатывает HTTP ошибки', async () => {
        global.fetch.mockResolvedValue({
            ok: false,
            status: 404,
            statusText: 'Not Found'
        });
        
        await expect(apiClient.get('/users/999')).rejects.toThrow('HTTP 404: Not Found');
    });
    
    test('обрабатывает timeout', async () => {
        global.fetch.mockImplementation(() => new Promise(() => {
            // fetch никогда не разрешается, симулируя timeout
        }));
        
        const promise = apiClient.get('/users/1');
        
        // Ждем, пока не произойдет timeout
        await expect(promise).rejects.toThrow('Request timeout');
    });
});
```

### Тестирование сложной бизнес-логики

```javascript
class OrderCalculator {
    constructor(taxRate = 0.2) {
        this.taxRate = taxRate;
    }
    
    calculateTotal(items, discountCode = null, shipping = 0) {
        if (!Array.isArray(items) || items.length === 0) {
            throw new Error('Items must be a non-empty array');
        }
        
        const subtotal = items.reduce((sum, item) => {
            if (typeof item.price !== 'number' || item.price < 0) {
                throw new Error('Invalid item price');
            }
            return sum + (item.price * (item.quantity || 1));
        }, 0);
        
        const discount = this.calculateDiscount(subtotal, discountCode);
        const tax = (subtotal - discount) * this.taxRate;
        
        return {
            subtotal,
            discount,
            tax,
            shipping,
            total: subtotal - discount + tax + shipping
        };
    }
    
    calculateDiscount(subtotal, discountCode) {
        if (!discountCode) return 0;
        
        const discountRules = {
            'SAVE10': subtotal * 0.1,
            'SAVE20': subtotal * 0.2,
            'FREESHIP': 0 // Скидка на доставку, не на товары
        };
        
        return discountRules[discountCode] || 0;
    }
}

// Комплексные тесты для OrderCalculator
describe('OrderCalculator', () => {
    let calculator;
    
    beforeEach(() => {
        calculator = new OrderCalculator(0.18); // 18% налог
    });
    
    test('рассчитывает простой заказ', () => {
        const items = [
            { price: 100, quantity: 2 },
            { price: 50, quantity: 1 }
        ];
        
        const result = calculator.calculateTotal(items);
        
        expect(result.subtotal).toBe(250); // 2*100 + 1*50
        expect(result.tax).toBe(45); // 250 * 0.18
        expect(result.total).toBe(295); // 250 + 45 + 0
    });
    
    test('применяет скидку', () => {
        const items = [{ price: 100, quantity: 1 }];
        const result = calculator.calculateTotal(items, 'SAVE10');
        
        expect(result.subtotal).toBe(100);
        expect(result.discount).toBe(10); // 10% от 100
        expect(result.tax).toBe(16.2); // (100 - 10) * 0.18
        expect(result.total).toBe(116.2);
    });
    
    test('добавляет доставку', () => {
        const items = [{ price: 100, quantity: 1 }];
        const result = calculator.calculateTotal(items, null, 10);
        
        expect(result.shipping).toBe(10);
        expect(result.total).toBe(128); // 100 + 18 (tax) + 10 (shipping)
    });
    
    test('обрабатывает невалидные данные', () => {
        expect(() => calculator.calculateTotal([])).toThrow('Items must be a non-empty array');
        expect(() => calculator.calculateTotal([{ price: -10 }])).toThrow('Invalid item price');
        expect(() => calculator.calculateTotal([{ price: 'invalid' }])).toThrow('Invalid item price');
    });
    
    test('тестируем различные комбинации', () => {
        const testCases = [
            {
                name: 'без скидки и доставки',
                items: [{ price: 100, quantity: 1 }],
                discount: null,
                shipping: 0,
                expectedTotal: 118 // 100 + 18% tax
            },
            {
                name: 'со скидкой и доставкой',
                items: [{ price: 200, quantity: 1 }],
                discount: 'SAVE20',
                shipping: 15,
                expectedTotal: 205.4 // (200 - 40) * 1.18 + 15
            }
        ];
        
        testCases.forEach(testCase => {
            test(testCase.name, () => {
                const result = calculator.calculateTotal(
                    testCase.items,
                    testCase.discount,
                    testCase.shipping
                );
                
                expect(result.total).toBeCloseTo(testCase.expectedTotal, 2);
            });
        });
    });
});
```

## Ссылки на связанные темы
- [[js/basics/types]] - Типы данных в JavaScript
- [[js/dom/testing]] - Тестирование DOM
- [[js/es6+/modules]] - Модули ES6
- [[testing/best-practices]] - Лучшие практики тестирования

#programming #javascript #testing #mocks #best-practices