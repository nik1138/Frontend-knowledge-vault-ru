# Паттерн Фабрика в JavaScript

## Обзор

Паттерн Фабрика (Factory Pattern) - это порождающий паттерн проектирования, который предоставляет интерфейс для создания объектов в суперклассе, позволяя подклассам изменять тип создаваемых объектов. Фабрика инкапсулирует процесс создания объектов и позволяет избежать жесткой зависимости от конкретных классов.

## Простая реализация фабрики

### Базовая фабрика

```javascript
// Продукты
class Car {
    constructor(brand, model) {
        this.brand = brand;
        this.model = model;
    }
    
    start() {
        return `${this.brand} ${this.model} запускается`;
    }
    
    getInfo() {
        return `Автомобиль: ${this.brand} ${this.model}`;
    }
}

class Truck {
    constructor(brand, model, capacity) {
        this.brand = brand;
        this.model = model;
        this.capacity = capacity; // тонн
    }
    
    start() {
        return `${this.brand} ${this.model} грузовик запускается`;
    }
    
    getInfo() {
        return `Грузовик: ${this.brand} ${this.model}, грузоподъемность: ${this.capacity} т`;
    }
}

// Фабрика
class VehicleFactory {
    static createVehicle(type, ...args) {
        switch (type.toLowerCase()) {
            case 'car':
                return new Car(...args);
            case 'truck':
                return new Truck(...args);
            default:
                throw new Error(`Неизвестный тип транспортного средства: ${type}`);
        }
    }
}

// Использование
const car = VehicleFactory.createVehicle('car', 'Toyota', 'Camry');
const truck = VehicleFactory.createVehicle('truck', 'Volvo', 'FH16', 25);

console.log(car.getInfo()); // "Автомобиль: Toyota Camry"
console.log(truck.getInfo()); // "Грузовик: Volvo FH16, грузоподъемность: 25 т"
```

## Фабричный метод

### Реализация с использованием классов

```javascript
// Абстрактный класс для животных
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        throw new Error('Метод speak должен быть реализован');
    }
    
    getInfo() {
        return `Животное: ${this.name}`;
    }
}

// Конкретные классы животных
class Dog extends Animal {
    speak() {
        return `${this.name} лает: Гав!`;
    }
}

class Cat extends Animal {
    speak() {
        return `${this.name} мяукает: Мяу!`;
    }
}

class Bird extends Animal {
    speak() {
        return `${this.name} поет: Чик-чирик!`;
    }
}

// Фабричный метод в базовом классе
class AnimalFactory {
    static createAnimal(type, name) {
        switch (type.toLowerCase()) {
            case 'dog':
                return new Dog(name);
            case 'cat':
                return new Cat(name);
            case 'bird':
                return new Bird(name);
            default:
                throw new Error(`Неизвестный тип животного: ${type}`);
        }
    }
    
    // Альтернативный подход: фабрика с методами для каждого типа
    static createDog(name) {
        return new Dog(name);
    }
    
    static createCat(name) {
        return new Cat(name);
    }
    
    static createBird(name) {
        return new Bird(name);
    }
}

// Использование
const animals = [
    AnimalFactory.createAnimal('dog', 'Рекс'),
    AnimalFactory.createAnimal('cat', 'Мурка'),
    AnimalFactory.createAnimal('bird', 'Чижик')
];

animals.forEach(animal => {
    console.log(animal.getInfo());
    console.log(animal.speak());
});
```

## Продвинутая фабрика с конфигурацией

### Фабрика с параметрами и валидацией

```javascript
// Классы пользователей
class BasicUser {
    constructor(name, email) {
        this.name = name;
        this.email = email;
        this.role = 'basic';
        this.permissions = ['read'];
    }
    
    getInfo() {
        return `${this.name} (${this.email}), роль: ${this.role}`;
    }
    
    hasPermission(permission) {
        return this.permissions.includes(permission);
    }
}

class PremiumUser extends BasicUser {
    constructor(name, email, subscriptionType) {
        super(name, email);
        this.role = 'premium';
        this.subscriptionType = subscriptionType;
        this.permissions = ['read', 'write', 'comment'];
    }
    
    getSubscriptionInfo() {
        return `Тип подписки: ${this.subscriptionType}`;
    }
}

class AdminUser extends BasicUser {
    constructor(name, email, department) {
        super(name, email);
        this.role = 'admin';
        this.department = department;
        this.permissions = ['read', 'write', 'delete', 'manage_users'];
    }
    
    getDepartment() {
        return this.department;
    }
}

// Фабрика пользователей
class UserFactory {
    static createUser(type, userData) {
        // Валидация данных
        this.validateUserData(userData);
        
        switch (type.toLowerCase()) {
            case 'basic':
                return new BasicUser(userData.name, userData.email);
                
            case 'premium':
                if (!userData.subscriptionType) {
                    throw new Error('Для премиум пользователя требуется subscriptionType');
                }
                return new PremiumUser(userData.name, userData.email, userData.subscriptionType);
                
            case 'admin':
                if (!userData.department) {
                    throw new Error('Для администратора требуется department');
                }
                return new AdminUser(userData.name, userData.email, userData.department);
                
            default:
                throw new Error(`Неизвестный тип пользователя: ${type}`);
        }
    }
    
    static validateUserData(userData) {
        if (!userData || typeof userData !== 'object') {
            throw new Error('Данные пользователя должны быть объектом');
        }
        
        if (!userData.name || typeof userData.name !== 'string') {
            throw new Error('Имя пользователя обязательно и должно быть строкой');
        }
        
        if (!userData.email || typeof userData.email !== 'string' || !userData.email.includes('@')) {
            throw new Error('Email пользователя обязателен и должен быть валидным');
        }
    }
    
    // Метод для массового создания пользователей
    static createUsers(usersData) {
        return usersData.map(userData => {
            return this.createUser(userData.type, userData.data);
        });
    }
}

// Использование
try {
    const basicUser = UserFactory.createUser('basic', {
        name: 'Иван Петров',
        email: 'ivan@example.com'
    });
    
    const premiumUser = UserFactory.createUser('premium', {
        name: 'Мария Сидорова',
        email: 'maria@example.com',
        subscriptionType: 'yearly'
    });
    
    const adminUser = UserFactory.createUser('admin', {
        name: 'Алексей Козлов',
        email: 'alexey@example.com',
        department: 'IT'
    });
    
    console.log(basicUser.getInfo()); // "Иван Петров (ivan@example.com), роль: basic"
    console.log(premiumUser.getSubscriptionInfo()); // "Тип подписки: yearly"
    console.log(adminUser.getDepartment()); // "IT"
    
    // Массовое создание
    const users = UserFactory.createUsers([
        { type: 'basic', data: { name: 'Ольга', email: 'olga@example.com' } },
        { type: 'premium', data: { name: 'Сергей', email: 'sergey@example.com', subscriptionType: 'monthly' } }
    ]);
    
    console.log(`Создано ${users.length} пользователей`);
    
} catch (error) {
    console.error('Ошибка при создании пользователя:', error.message);
}
```

## Фабрика с регистрацией типов

### Динамическая фабрика

```javascript
// Базовый класс для компонентов UI
class UIComponent {
    constructor(props = {}) {
        this.props = props;
        this.id = props.id || this.generateId();
    }
    
    generateId() {
        return `component_${Math.random().toString(36).substr(2, 9)}`;
    }
    
    render() {
        throw new Error('Метод render должен быть реализован');
    }
}

// Конкретные компоненты
class Button extends UIComponent {
    constructor(props) {
        super(props);
        this.text = props.text || 'Кнопка';
        this.type = props.type || 'button';
        this.style = props.style || 'default';
    }
    
    render() {
        return `<button id="${this.id}" class="btn btn-${this.style}" type="${this.type}">${this.text}</button>`;
    }
}

class Input extends UIComponent {
    constructor(props) {
        super(props);
        this.type = props.type || 'text';
        this.placeholder = props.placeholder || '';
        this.value = props.value || '';
    }
    
    render() {
        return `<input id="${this.id}" type="${this.type}" placeholder="${this.placeholder}" value="${this.value}">`;
    }
}

class Modal extends UIComponent {
    constructor(props) {
        super(props);
        this.title = props.title || 'Модальное окно';
        this.content = props.content || '';
        this.isOpen = props.isOpen || false;
    }
    
    render() {
        const display = this.isOpen ? 'block' : 'none';
        return `
            <div id="${this.id}" class="modal" style="display: ${display}">
                <div class="modal-content">
                    <h3>${this.title}</h3>
                    <p>${this.content}</p>
                </div>
            </div>
        `;
    }
}

// Фабрика с регистрацией типов
class ComponentFactory {
    constructor() {
        this.components = new Map();
    }
    
    // Регистрация нового типа компонента
    register(type, componentClass) {
        this.components.set(type.toLowerCase(), componentClass);
    }
    
    // Создание компонента
    create(type, props = {}) {
        const ComponentClass = this.components.get(type.toLowerCase());
        
        if (!ComponentClass) {
            throw new Error(`Компонент типа "${type}" не зарегистрирован`);
        }
        
        return new ComponentClass(props);
    }
    
    // Получение списка зарегистрированных типов
    getRegisteredTypes() {
        return Array.from(this.components.keys());
    }
    
    // Проверка, зарегистрирован ли тип
    isRegistered(type) {
        return this.components.has(type.toLowerCase());
    }
    
    // Удаление зарегистрированного типа
    unregister(type) {
        return this.components.delete(type.toLowerCase());
    }
}

// Создание фабрики и регистрация компонентов
const factory = new ComponentFactory();
factory.register('button', Button);
factory.register('input', Input);
factory.register('modal', Modal);

// Использование фабрики
const button = factory.create('button', {
    text: 'Нажми меня',
    style: 'primary',
    id: 'my-button'
});

const input = factory.create('input', {
    type: 'email',
    placeholder: 'Введите email',
    value: 'user@example.com'
});

const modal = factory.create('modal', {
    title: 'Приветствие',
    content: 'Добро пожаловать!',
    isOpen: true
});

console.log(button.render());
console.log(input.render());
console.log(modal.render());

// Проверка зарегистрированных типов
console.log('Зарегистрированные типы:', factory.getRegisteredTypes());

// Регистрация нового компонента в рантайме
class Card extends UIComponent {
    constructor(props) {
        super(props);
        this.title = props.title || 'Заголовок карточки';
        this.content = props.content || 'Содержимое карточки';
    }
    
    render() {
        return `
            <div id="${this.id}" class="card">
                <h4>${this.title}</h4>
                <p>${this.content}</p>
            </div>
        `;
    }
}

factory.register('card', Card);
const card = factory.create('card', {
    title: 'Информационная карточка',
    content: 'Это пример карточки, созданной через фабрику'
});

console.log(card.render());
```

## Практические применения паттерна Фабрика

### Фабрика для API клиентов

```javascript
// Базовый класс API клиента
class ApiClient {
    constructor(baseUrl, apiKey) {
        this.baseUrl = baseUrl;
        this.apiKey = apiKey;
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseUrl}${endpoint}`;
        const headers = {
            ...options.headers,
            'Authorization': `Bearer ${this.apiKey}`,
            'Content-Type': 'application/json'
        };
        
        const response = await fetch(url, {
            ...options,
            headers
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка! Статус: ${response.status}`);
        }
        
        return await response.json();
    }
}

// Конкретные API клиенты
class UserApiClient extends ApiClient {
    async getUser(id) {
        return await this.request(`/users/${id}`);
    }
    
    async createUser(userData) {
        return await this.request('/users', {
            method: 'POST',
            body: JSON.stringify(userData)
        });
    }
}

class ProductApiClient extends ApiClient {
    async getProduct(id) {
        return await this.request(`/products/${id}`);
    }
    
    async searchProducts(query) {
        const encodedQuery = encodeURIComponent(query);
        return await this.request(`/products/search?q=${encodedQuery}`);
    }
}

// Фабрика API клиентов
class ApiClientFactory {
    static createClient(type, config) {
        const { baseUrl, apiKey } = config;
        
        switch (type.toLowerCase()) {
            case 'user':
                return new UserApiClient(baseUrl, apiKey);
            case 'product':
                return new ProductApiClient(baseUrl, apiKey);
            default:
                throw new Error(`Неизвестный тип API клиента: ${type}`);
        }
    }
    
    // Метод для создания нескольких клиентов
    static createMultipleClients(configs) {
        const clients = {};
        
        for (const [type, config] of Object.entries(configs)) {
            clients[`${type}Client`] = this.createClient(type, config);
        }
        
        return clients;
    }
}

// Использование
const userClient = ApiClientFactory.createClient('user', {
    baseUrl: 'https://api.example.com',
    apiKey: 'your-api-key'
});

const productClient = ApiClientFactory.createClient('product', {
    baseUrl: 'https://api.example.com',
    apiKey: 'your-api-key'
});

// Или создание нескольких клиентов за раз
const clients = ApiClientFactory.createMultipleClients({
    user: {
        baseUrl: 'https://api.example.com',
        apiKey: 'user-api-key'
    },
    product: {
        baseUrl: 'https://api.example.com',
        apiKey: 'product-api-key'
    }
});
```

## Преимущества и недостатки паттерна Фабрика

### Преимущества:
- Изолирует код от реализации создаваемых объектов
- Упрощает добавление новых типов объектов
- Позволяет легко изменять создаваемые объекты
- Поддерживает принцип открытости/закрытости (OCP)

### Недостатки:
- Может привести к созданию больших фабричных классов
- Усложняет код из-за введения дополнительных классов
- Требует создания новых классов для каждого нового типа продукта