# Интеграционное тестирование в JavaScript

## Обзор

Интеграционное тестирование проверяет взаимодействие между различными модулями или компонентами системы. В отличие от модульного тестирования, которое тестирует отдельные части изолированно, интеграционное тестирование проверяет, как компоненты работают вместе.

## Основные концепции интеграционного тестирования

### Отличие от модульного тестирования

```javascript
// user-service.js
export class UserService {
    constructor(apiClient, validator) {
        this.apiClient = apiClient;
        this.validator = validator;
    }
    
    async createUser(userData) {
        // Проверка данных
        const validation = this.validator.validate(userData);
        if (!validation.isValid) {
            throw new Error(`Ошибка валидации: ${validation.errors.join(', ')}`);
        }
        
        // Сохранение в API
        const response = await this.apiClient.post('/users', userData);
        return response.data;
    }
    
    async getUser(id) {
        if (!id) {
            throw new Error('ID пользователя обязателен');
        }
        
        const response = await this.apiClient.get(`/users/${id}`);
        return response.data;
    }
}

// validator.js
export class UserValidator {
    validate(userData) {
        const errors = [];
        
        if (!userData.name || userData.name.length < 2) {
            errors.push('Имя должно содержать хотя бы 2 символа');
        }
        
        if (!userData.email || !userData.email.includes('@')) {
            errors.push('Email должен быть валидным');
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
}

// api-client.js
export class ApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    async get(endpoint) {
        const response = await fetch(`${this.baseURL}${endpoint}`);
        if (!response.ok) {
            throw new Error(`HTTP ошибка: ${response.status}`);
        }
        return await response.json();
    }
    
    async post(endpoint, data) {
        const response = await fetch(`${this.baseURL}${endpoint}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка: ${response.status}`);
        }
        return await response.json();
    }
}
```

```javascript
// integration-test.js - Пример интеграционного теста
import { UserService } from './user-service.js';
import { UserValidator } from './validator.js';
import { ApiClient } from './api-client.js';

describe('Интеграционные тесты UserService', () => {
    let userService;
    let mockApiClient;
    let validator;
    
    beforeEach(() => {
        validator = new UserValidator();
        
        // Создаем mock API клиента для тестирования
        mockApiClient = {
            get: jest.fn(),
            post: jest.fn()
        };
        
        userService = new UserService(mockApiClient, validator);
    });
    
    test('createUser: успешно создает пользователя при валидных данных', async () => {
        // Arrange
        const userData = { name: 'Иван Иванов', email: 'ivan@example.com' };
        const mockResponse = { data: { id: 1, ...userData } };
        
        mockApiClient.post.mockResolvedValue(mockResponse);
        
        // Act
        const result = await userService.createUser(userData);
        
        // Assert
        expect(mockApiClient.post).toHaveBeenCalledWith('/users', userData);
        expect(result).toEqual(mockResponse.data);
    });
    
    test('createUser: выбрасывает ошибку при невалидных данных', async () => {
        // Arrange
        const invalidData = { name: 'I', email: 'invalid-email' };
        
        // Act & Assert
        await expect(userService.createUser(invalidData))
            .rejects
            .toThrow('Ошибка валидации: Имя должно содержать хотя бы 2 символа, Email должен быть валидным');
    });
    
    test('getUser: успешно получает пользователя', async () => {
        // Arrange
        const userId = 123;
        const mockUser = { id: userId, name: 'Иван', email: 'ivan@example.com' };
        
        mockApiClient.get.mockResolvedValue({ data: mockUser });
        
        // Act
        const result = await userService.getUser(userId);
        
        // Assert
        expect(mockApiClient.get).toHaveBeenCalledWith(`/users/${userId}`);
        expect(result).toEqual(mockUser);
    });
});
```

## Тестирование взаимодействия с внешними API

### Использование mock-сервера

```javascript
// api-integration.test.js
import { UserService } from './user-service.js';
import { UserValidator } from './validator.js';
import { ApiClient } from './api-client.js';

describe('Интеграционные тесты с реальным API', () => {
    let userService;
    let apiClient;
    
    beforeAll(() => {
        // Используем реальный API клиент с тестовым URL
        apiClient = new ApiClient('https://jsonplaceholder.typicode.com');
        const validator = new UserValidator();
        userService = new UserService(apiClient, validator);
    });
    
    test('getUser: получает пользователя от внешнего API', async () => {
        // Act
        const user = await userService.getUser(1);
        
        // Assert
        expect(user).toHaveProperty('id', 1);
        expect(user).toHaveProperty('name');
        expect(user).toHaveProperty('email');
    }, 10000); // Увеличиваем таймаут для сетевых запросов
    
    test('createUser: создает пользователя через внешний API', async () => {
        // Arrange
        const userData = { 
            name: 'Test User', 
            email: 'test@example.com',
            username: 'testuser'
        };
        
        // Act
        const result = await userService.createUser(userData);
        
        // Assert
        expect(result).toHaveProperty('id');
        expect(result.name).toBe(userData.name);
        expect(result.email).toBe(userData.email);
    }, 10000);
});
```

## Тестирование цепочек вызовов

### Комплексные сценарии

```javascript
// order-service.js
export class OrderService {
    constructor(userService, paymentService, notificationService) {
        this.userService = userService;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
    
    async processOrder(orderData) {
        // 1. Получаем информацию о пользователе
        const user = await this.userService.getUser(orderData.userId);
        
        // 2. Проверяем платежную информацию
        const paymentResult = await this.paymentService.processPayment(
            orderData.paymentInfo, 
            orderData.amount
        );
        
        // 3. Если оплата прошла успешно, создаем заказ
        if (paymentResult.success) {
            // 4. Отправляем уведомление пользователю
            await this.notificationService.sendNotification(
                user.email,
                'Заказ успешно оформлен',
                `Ваш заказ на сумму ${orderData.amount} обработан`
            );
            
            return {
                orderId: this.generateOrderId(),
                userId: orderData.userId,
                amount: orderData.amount,
                paymentId: paymentResult.paymentId,
                status: 'completed'
            };
        } else {
            throw new Error('Платеж не прошел');
        }
    }
    
    generateOrderId() {
        return `ORD-${Date.now()}-${Math.floor(Math.random() * 1000)}`;
    }
}

// integration-chain.test.js
describe('Цепочка вызовов OrderService', () => {
    let orderService;
    let mockUserService, mockPaymentService, mockNotificationService;
    
    beforeEach(() => {
        mockUserService = {
            getUser: jest.fn()
        };
        
        mockPaymentService = {
            processPayment: jest.fn()
        };
        
        mockNotificationService = {
            sendNotification: jest.fn()
        };
        
        orderService = new OrderService(
            mockUserService,
            mockPaymentService,
            mockNotificationService
        );
    });
    
    test('processOrder: успешно обрабатывает заказ через всю цепочку', async () => {
        // Arrange
        const orderData = {
            userId: 123,
            amount: 100,
            paymentInfo: { cardNumber: '1234', cvv: '123' }
        };
        
        const mockUser = { id: 123, name: 'Иван', email: 'ivan@example.com' };
        const mockPaymentResult = { success: true, paymentId: 'PAY-12345' };
        
        mockUserService.getUser.mockResolvedValue(mockUser);
        mockPaymentService.processPayment.mockResolvedValue(mockPaymentResult);
        
        // Act
        const result = await orderService.processOrder(orderData);
        
        // Assert
        expect(mockUserService.getUser).toHaveBeenCalledWith(123);
        expect(mockPaymentService.processPayment).toHaveBeenCalledWith(
            orderData.paymentInfo,
            orderData.amount
        );
        expect(mockNotificationService.sendNotification).toHaveBeenCalledWith(
            mockUser.email,
            'Заказ успешно оформлен',
            'Ваш заказ на сумму 100 обработан'
        );
        
        expect(result).toHaveProperty('orderId');
        expect(result).toHaveProperty('status', 'completed');
        expect(result.userId).toBe(123);
    });
    
    test('processOrder: выбрасывает ошибку при неудачной оплате', async () => {
        // Arrange
        const orderData = {
            userId: 123,
            amount: 100,
            paymentInfo: { cardNumber: '1234', cvv: '123' }
        };
        
        const mockUser = { id: 123, name: 'Иван', email: 'ivan@example.com' };
        const mockPaymentResult = { success: false, error: 'Недостаточно средств' };
        
        mockUserService.getUser.mockResolvedValue(mockUser);
        mockPaymentService.processPayment.mockResolvedValue(mockPaymentResult);
        
        // Act & Assert
        await expect(orderService.processOrder(orderData))
            .rejects
            .toThrow('Платеж не прошел');
            
        // Проверяем, что уведомление не отправляется при неудачной оплате
        expect(mockNotificationService.sendNotification).not.toHaveBeenCalled();
    });
});
```

## Тестирование с базами данных

### Интеграция с реальной базой данных

```javascript
// user-repository.js
export class UserRepository {
    constructor(dbConnection) {
        this.db = dbConnection;
    }
    
    async create(user) {
        const query = 'INSERT INTO users (name, email) VALUES (?, ?)';
        const result = await this.db.query(query, [user.name, user.email]);
        return { id: result.insertId, ...user };
    }
    
    async findById(id) {
        const query = 'SELECT * FROM users WHERE id = ?';
        const results = await this.db.query(query, [id]);
        return results[0] || null;
    }
    
    async findByEmail(email) {
        const query = 'SELECT * FROM users WHERE email = ?';
        const results = await this.db.query(query, [email]);
        return results[0] || null;
    }
    
    async update(id, updates) {
        const fields = Object.keys(updates);
        const values = Object.values(updates);
        const setClause = fields.map(field => `${field} = ?`).join(', ');
        
        const query = `UPDATE users SET ${setClause} WHERE id = ?`;
        await this.db.query(query, [...values, id]);
        
        return this.findById(id);
    }
}

// database-integration.test.js
describe('Интеграционные тесты UserRepository', () => {
    let userRepository;
    let dbConnection;
    
    beforeAll(async () => {
        // Создаем подключение к тестовой базе данных
        dbConnection = await createTestDatabaseConnection();
        userRepository = new UserRepository(dbConnection);
        
        // Создаем тестовую таблицу
        await dbConnection.query(`
            CREATE TABLE IF NOT EXISTS users (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(255) NOT NULL,
                email VARCHAR(255) UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        `);
    });
    
    afterAll(async () => {
        // Очищаем тестовые данные
        await dbConnection.query('DELETE FROM users');
        await dbConnection.end();
    });
    
    beforeEach(async () => {
        // Очищаем данные перед каждым тестом
        await dbConnection.query('DELETE FROM users');
    });
    
    test('create: успешно создает пользователя в базе данных', async () => {
        // Arrange
        const userData = { name: 'Иван Иванов', email: 'ivan@example.com' };
        
        // Act
        const createdUser = await userRepository.create(userData);
        
        // Assert
        expect(createdUser).toHaveProperty('id');
        expect(createdUser.name).toBe(userData.name);
        expect(createdUser.email).toBe(userData.email);
        
        // Проверяем, что пользователь действительно в базе
        const foundUser = await userRepository.findById(createdUser.id);
        expect(foundUser).toEqual(createdUser);
    });
    
    test('findById: находит пользователя по ID', async () => {
        // Arrange
        const userData = { name: 'Мария Сидорова', email: 'maria@example.com' };
        const createdUser = await userRepository.create(userData);
        
        // Act
        const foundUser = await userRepository.findById(createdUser.id);
        
        // Assert
        expect(foundUser).toEqual(createdUser);
    });
    
    test('update: обновляет данные пользователя', async () => {
        // Arrange
        const userData = { name: 'Алексей Козлов', email: 'alex@example.com' };
        const createdUser = await userRepository.create(userData);
        
        const updates = { name: 'Алексей Петров', email: 'alex.petrov@example.com' };
        
        // Act
        const updatedUser = await userRepository.update(createdUser.id, updates);
        
        // Assert
        expect(updatedUser.name).toBe(updates.name);
        expect(updatedUser.email).toBe(updates.email);
        
        // Проверяем, что изменения сохранились в базе
        const verifiedUser = await userRepository.findById(createdUser.id);
        expect(verifiedUser).toEqual(updatedUser);
    });
});
```

## Тестирование с внешними сервисами

### Использованиеnock для мокирования HTTP-запросов

```javascript
// payment-service.js
export class PaymentService {
    constructor(apiKey, baseUrl = 'https://api.payment-service.com') {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
    }
    
    async processPayment(paymentInfo, amount) {
        const response = await fetch(`${this.baseUrl}/payments`, {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${this.apiKey}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                amount,
                payment_method: {
                    card_number: paymentInfo.cardNumber,
                    cvv: paymentInfo.cvv,
                    expiry_month: paymentInfo.expiryMonth,
                    expiry_year: paymentInfo.expiryYear
                }
            })
        });
        
        if (!response.ok) {
            throw new Error(`Платежный шлюз вернул ошибку: ${response.status}`);
        }
        
        return await response.json();
    }
    
    async getPaymentStatus(paymentId) {
        const response = await fetch(`${this.baseUrl}/payments/${paymentId}`, {
            headers: {
                'Authorization': `Bearer ${this.apiKey}`
            }
        });
        
        if (!response.ok) {
            throw new Error(`Не удалось получить статус платежа: ${response.status}`);
        }
        
        return await response.json();
    }
}

// external-service-integration.test.js
import nock from 'nock';

describe('Интеграционные тесты PaymentService с внешним API', () => {
    let paymentService;
    
    beforeAll(() => {
        paymentService = new PaymentService('test-api-key');
    });
    
    afterEach(() => {
        // Убедиться, что все моки были вызваны
        nock.cleanAll();
    });
    
    test('processPayment: успешно обрабатывает платеж через внешний API', async () => {
        // Arrange
        const paymentInfo = {
            cardNumber: '4111111111111111',
            cvv: '123',
            expiryMonth: '12',
            expiryYear: '2025'
        };
        
        const amount = 100;
        
        // Мокаем внешний API
        nock('https://api.payment-service.com')
            .post('/payments')
            .reply(200, {
                id: 'PAY-12345',
                status: 'success',
                amount: 100,
                transaction_id: 'TXN-67890'
            });
        
        // Act
        const result = await paymentService.processPayment(paymentInfo, amount);
        
        // Assert
        expect(result).toEqual({
            id: 'PAY-12345',
            status: 'success',
            amount: 100,
            transaction_id: 'TXN-67890'
        });
    });
    
    test('getPaymentStatus: получает статус платежа от внешнего API', async () => {
        // Arrange
        const paymentId = 'PAY-12345';
        
        // Мокаем внешний API
        nock('https://api.payment-service.com')
            .get(`/payments/${paymentId}`)
            .reply(200, {
                id: 'PAY-12345',
                status: 'completed',
                amount: 100
            });
        
        // Act
        const result = await paymentService.getPaymentStatus(paymentId);
        
        // Assert
        expect(result).toEqual({
            id: 'PAY-12345',
            status: 'completed',
            amount: 100
        });
    });
    
    test('processPayment: выбрасывает ошибку при неудачном ответе от API', async () => {
        // Arrange
        const paymentInfo = {
            cardNumber: '4111111111111111',
            cvv: '123',
            expiryMonth: '12',
            expiryYear: '2025'
        };
        
        const amount = 100;
        
        // Мокаем ошибку от внешнего API
        nock('https://api.payment-service.com')
            .post('/payments')
            .reply(400, {
                error: 'Invalid card number'
            });
        
        // Act & Assert
        await expect(paymentService.processPayment(paymentInfo, amount))
            .rejects
            .toThrow('Платежный шлюз вернул ошибку: 400');
    });
});
```

## Практические советы по интеграционному тестированию

### Организация тестов

```javascript
// integration-test-helpers.js
export const setupIntegrationTest = async () => {
    // Инициализация тестовой среды
    const testDatabase = await createTestDatabase();
    const testServer = await startTestServer();
    const testServices = initializeTestServices(testDatabase);
    
    return {
        db: testDatabase,
        server: testServer,
        services: testServices,
        cleanup: async () => {
            await cleanupTestDatabase(testDatabase);
            await stopTestServer(testServer);
        }
    };
};

// example-integration-suite.test.js
describe('Полный интеграционный тест приложения', () => {
    let testEnv;
    
    beforeAll(async () => {
        testEnv = await setupIntegrationTest();
    }, 30000); // Увеличенный таймаут для настройки
    
    afterAll(async () => {
        await testEnv.cleanup();
    }, 10000); // Увеличенный таймаут для очистки
    
    test('Полный сценарий регистрации и аутентификации пользователя', async () => {
        // Тестирование полного сценария работы приложения
        const { services } = testEnv;
        
        // 1. Регистрация пользователя
        const newUser = await services.userService.register({
            name: 'Test User',
            email: 'test@example.com',
            password: 'securePassword'
        });
        
        // 2. Аутентификация
        const authResult = await services.authService.authenticate({
            email: 'test@example.com',
            password: 'securePassword'
        });
        
        // 3. Создание профиля
        const profile = await services.profileService.create({
            userId: newUser.id,
            bio: 'Тестовый пользователь'
        });
        
        // Проверки
        expect(newUser).toHaveProperty('id');
        expect(authResult).toHaveProperty('token');
        expect(profile).toHaveProperty('id');
        expect(profile.userId).toBe(newUser.id);
    });
});
```

## Преимущества и недостатки интеграционного тестирования

### Преимущества:
- Обнаруживает проблемы интеграции между компонентами
- Проверяет реальное взаимодействие с внешними сервисами
- Более близко к реальному использованию приложения
- Проверяет сложные сценарии использования

### Недостатки:
- Более медленные, чем модульные тесты
- Сложнее в отладке (ошибка может быть в любом из компонентов)
- Зависимость от внешних систем и сервисов
- Требуют больше ресурсов для запуска