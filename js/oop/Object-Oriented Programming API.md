# Object-Oriented Programming API

## Введение

Объектно-ориентированное программирование (ООП) - это парадигма программирования, основанная на понятии "объектов", которые могут содержать данные (атрибуты) и код (методы). В этой главе мы рассмотрим основные концепции ООП в JavaScript: классы, наследование, инкапсуляция, полиморфизм, а также современные подходы и паттерны.

## Содержание

- [[Классы и объекты]]
- [[Наследование]]
- [[Инкапсуляция]]
- [[Полиморфизм]]
- [[Абстракция]]
- [[Паттерны проектирования]]
- [[Прототипное наследование]]
- [[Современные классы ES6+]]
- [[Миксины и трейты]]

## Классы и объекты

Классы в JavaScript (начиная с ES6) предоставляют синтаксический сахар над прототипным наследованием и позволяют создавать объекты с определенными свойствами и методами.

### Основы классов

```javascript
// Простой класс
class Person {
    constructor(firstName, lastName, age) {
        // Приватные свойства (символы для эмуляции приватности)
        const _firstName = Symbol('firstName');
        const _lastName = Symbol('lastName');
        const _age = Symbol('age');
        
        // Публичные свойства
        this[_firstName] = firstName;
        this[_lastName] = lastName;
        this[_age] = age;
        
        // Геттеры и сеттеры для контролируемого доступа
        Object.defineProperties(this, {
            firstName: {
                get: () => this[_firstName],
                set: (value) => {
                    if (typeof value !== 'string' || value.trim() === '') {
                        throw new Error('Имя должно быть непустой строкой');
                    }
                    this[_firstName] = value.trim();
                },
                enumerable: true
            },
            lastName: {
                get: () => this[_lastName],
                set: (value) => {
                    if (typeof value !== 'string' || value.trim() === '') {
                        throw new Error('Фамилия должна быть непустой строкой');
                    }
                    this[_lastName] = value.trim();
                },
                enumerable: true
            },
            age: {
                get: () => this[_age],
                set: (value) => {
                    if (typeof value !== 'number' || value < 0 || value > 150) {
                        throw new Error('Возраст должен быть числом от 0 до 150');
                    }
                    this[_age] = value;
                },
                enumerable: true
            }
        });
    }
    
    // Публичный метод
    getFullName() {
        return `${this.firstName} ${this.lastName}`;
    }
    
    // Метод для приветствия
    greet() {
        return `Привет, меня зовут ${this.getFullName()} и мне ${this.age} лет.`;
    }
    
    // Статический метод
    static compareByAge(person1, person2) {
        return person1.age - person2.age;
    }
    
    // Метод для строкового представления
    toString() {
        return `Person(${this.getFullName()}, ${this.age})`;
    }
    
    // Метод для сериализации
    toJSON() {
        return {
            firstName: this.firstName,
            lastName: this.lastName,
            age: this.age
        };
    }
}

// Использование класса
const person1 = new Person('Иван', 'Иванов', 30);
console.log(person1.greet()); // "Привет, меня зовут Иван Иванов и мне 30 лет."
console.log(person1.getFullName()); // "Иван Иванов"

// Изменение свойств через сеттеры
person1.age = 31;
person1.firstName = 'Петр';
console.log(person1.getFullName()); // "Петр Иванов"
```

### Продвинутые возможности классов

```javascript
// Класс с приватными полями (ES2022+)
class BankAccount {
    // Приватные поля
    #balance = 0;
    #accountNumber;
    #owner;
    #transactionHistory = [];
    
    constructor(owner, initialBalance = 0) {
        this.#accountNumber = this.generateAccountNumber();
        this.#owner = owner;
        if (initialBalance > 0) {
            this.#balance = initialBalance;
            this.#addTransaction('DEPOSIT', initialBalance, 'Initial deposit');
        }
    }
    
    // Приватный метод
    #addTransaction(type, amount, description = '') {
        this.#transactionHistory.push({
            id: Date.now() + Math.random(),
            type,
            amount,
            description,
            timestamp: new Date(),
            balanceAfter: this.#balance
        });
    }
    
    // Приватный метод для генерации номера счета
    #generateAccountNumber() {
        return 'ACC' + Date.now().toString(36) + Math.random().toString(36).substr(2, 5).toUpperCase();
    }
    
    // Публичные методы
    deposit(amount, description = '') {
        if (amount <= 0) {
            throw new Error('Сумма депозита должна быть положительной');
        }
        this.#balance += amount;
        this.#addTransaction('DEPOSIT', amount, description);
        return this.#balance;
    }
    
    withdraw(amount, description = '') {
        if (amount <= 0) {
            throw new Error('Сумма снятия должна быть положительной');
        }
        if (amount > this.#balance) {
            throw new Error('Недостаточно средств на счете');
        }
        this.#balance -= amount;
        this.#addTransaction('WITHDRAW', amount, description);
        return this.#balance;
    }
    
    getBalance() {
        return this.#balance;
    }
    
    getAccountNumber() {
        return this.#accountNumber;
    }
    
    getOwner() {
        return this.#owner;
    }
    
    getTransactionHistory(limit = 10) {
        return this.#transactionHistory.slice(-limit);
    }
    
    // Статический метод для валидации номера счета
    static isValidAccountNumber(accountNumber) {
        return typeof accountNumber === 'string' && accountNumber.startsWith('ACC');
    }
    
    // Метод для получения информации о счете
    getAccountInfo() {
        return {
            accountNumber: this.#accountNumber,
            owner: this.#owner,
            balance: this.#balance,
            transactionCount: this.#transactionHistory.length
        };
    }
    
    // Строковое представление
    toString() {
        return `BankAccount(${this.#accountNumber}, ${this.#owner}, $${this.#balance})`;
    }
}

// Использование
const account = new BankAccount('Иван Иванов', 1000);
account.deposit(500, 'Зарплата');
account.withdraw(200, 'Покупка');
console.log(account.getBalance()); // 1300
console.log(account.getTransactionHistory());
```

## Наследование

Наследование позволяет создавать новые классы на основе существующих, наследуя их свойства и методы.

### Основы наследования

```javascript
// Базовый класс
class Animal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
        this.energy = 100;
    }
    
    eat(food) {
        this.energy += 10;
        return `${this.name} ест ${food}. Энергия: ${this.energy}`;
    }
    
    sleep() {
        this.energy = Math.min(100, this.energy + 30);
        return `${this.name} спит. Энергия восстановлена до ${this.energy}`;
    }
    
    move() {
        if (this.energy >= 5) {
            this.energy -= 5;
            return `${this.name} двигается. Энергия: ${this.energy}`;
        } else {
            return `${this.name} слишком устал для движения.`;
        }
    }
}

// Класс, наследующий от Animal
class Dog extends Animal {
    constructor(name, breed) {
        super(name, 'Canis lupus familiaris'); // Вызов конструктора родителя
        this.breed = breed;
        this.tailWagging = false;
    }
    
    bark() {
        return `${this.name} гавкает!`;
    }
    
    // Переопределение метода родителя
    move() {
        const result = super.move(); // Вызов метода родителя
        if (!result.includes('устал')) {
            this.tailWagging = true;
        }
        return result;
    }
    
    fetch(item) {
        if (this.energy >= 15) {
            this.energy -= 15;
            return `${this.name} приносит ${item}. Энергия: ${this.energy}`;
        } else {
            return `${this.name} слишком устал, чтобы принести ${item}.`;
        }
    }
    
    // Добавление нового метода
    showBreed() {
        return `${this.name} - это ${this.breed}`;
    }
}

// Ещё один класс-наследник
class Cat extends Animal {
    constructor(name, color) {
        super(name, 'Felis catus');
        this.color = color;
        this.purrPower = 5;
    }
    
    meow() {
        return `${this.name} мяукает!`;
    }
    
    purr() {
        return `${this.name} мурлычет с силой ${this.purrPower}`;
    }
    
    // Переопределение метода с дополнительной логикой
    sleep() {
        const result = super.sleep();
        this.purrPower = Math.min(10, this.purrPower + 1);
        return result;
    }
    
    hunt() {
        if (this.energy >= 20) {
            this.energy -= 20;
            this.purrPower = Math.max(1, this.purrPower - 1);
            return `${this.name} охотится. Энергия: ${this.energy}, Сила мурлыканья: ${this.purrPower}`;
        } else {
            return `${this.name} слишком устал для охоты.`;
        }
    }
}

// Использование наследования
const dog = new Dog('Бобик', 'Лабрадор');
const cat = new Cat('Мурка', 'Рыжий');

console.log(dog.bark()); // "Бобик гавкает!"
console.log(cat.meow()); // "Мурка мяукает!"
console.log(dog.fetch('палку')); // "Бобик приносит палку..."
console.log(cat.hunt()); // "Мурка охотится..."
```

### Продвинутое наследование

```javascript
// Абстрактный базовый класс
class AbstractShape {
    constructor(color = 'black') {
        if (this.constructor === AbstractShape) {
            throw new Error('Cannot instantiate abstract class AbstractShape');
        }
        this.color = color;
        this.id = Symbol(this.constructor.name);
    }
    
    // Абстрактный метод (должен быть реализован в наследниках)
    getArea() {
        throw new Error('Method getArea() must be implemented');
    }
    
    getPerimeter() {
        throw new Error('Method getPerimeter() must be implemented');
    }
    
    // Общий метод для всех фигур
    getColor() {
        return this.color;
    }
    
    // Метод для изменения цвета
    setColor(newColor) {
        this.color = newColor;
    }
    
    // Статический метод для сравнения фигур по площади
    static compareByArea(shape1, shape2) {
        return shape1.getArea() - shape2.getArea();
    }
}

// Конкретная реализация
class Rectangle extends AbstractShape {
    constructor(width, height, color = 'black') {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    getArea() {
        return this.width * this.height;
    }
    
    getPerimeter() {
        return 2 * (this.width + this.height);
    }
    
    // Дополнительный метод
    getDiagonal() {
        return Math.sqrt(this.width ** 2 + this.height ** 2);
    }
    
    // Переопределение toString
    toString() {
        return `Rectangle(${this.width}x${this.height}, color: ${this.color})`;
    }
}

class Circle extends AbstractShape {
    constructor(radius, color = 'black') {
        super(color);
        this.radius = radius;
    }
    
    getArea() {
        return Math.PI * this.radius ** 2;
    }
    
    getPerimeter() {
        return 2 * Math.PI * this.radius;
    }
    
    // Дополнительный метод
    getDiameter() {
        return 2 * this.radius;
    }
    
    toString() {
        return `Circle(radius: ${this.radius}, color: ${this.color})`;
    }
}

// Множественное наследование через цепочку
class ColoredShape extends AbstractShape {
    constructor(shape, color) {
        super(color);
        this.shape = shape;
    }
    
    getArea() {
        return this.shape.getArea();
    }
    
    getPerimeter() {
        return this.shape.getPerimeter();
    }
    
    getColor() {
        return this.color;
    }
    
    getOriginalShape() {
        return this.shape;
    }
}

// Использование
const rect = new Rectangle(5, 3, 'red');
const circle = new Circle(4, 'blue');

console.log('Площадь прямоугольника:', rect.getArea()); // 15
console.log('Площадь круга:', circle.getArea()); // ~50.27

// Сравнение фигур
const comparison = AbstractShape.compareByArea(rect, circle);
console.log('Сравнение по площади:', comparison); // отрицательное, т.к. rect < circle

// Цветные фигуры
const coloredRect = new ColoredShape(new Rectangle(2, 2), 'green');
console.log('Площадь цветного прямоугольника:', coloredRect.getArea()); // 4
```

## Инкапсуляция

Инкапсуляция - это принцип, согласно которому внутреннее состояние объекта защищено от прямого доступа извне.

### Реализация инкапсуляции

```javascript
// Класс с полной инкапсуляцией
class SecureDataContainer {
    #data = new Map();
    #accessKey;
    #creationTime;
    #modificationLog = [];
    
    constructor(initialData = {}, accessKey = null) {
        this.#creationTime = new Date();
        
        if (accessKey) {
            this.#accessKey = accessKey;
        } else {
            // Генерация безопасного ключа доступа
            this.#accessKey = this.#generateSecureKey();
        }
        
        // Инициализация данных
        Object.entries(initialData).forEach(([key, value]) => {
            this.#data.set(key, this.#deepClone(value));
        });
    }
    
    // Приватный метод генерации ключа
    #generateSecureKey() {
        return Array.from({length: 32}, () => 
            Math.floor(Math.random() * 256).toString(16).padStart(2, '0')
        ).join('');
    }
    
    // Приватный метод клонирования
    #deepClone(obj) {
        if (obj === null || typeof obj !== 'object') return obj;
        if (obj instanceof Date) return new Date(obj);
        if (obj instanceof Array) return obj.map(item => this.#deepClone(item));
        if (typeof obj === 'object') {
            const cloned = {};
            Object.keys(obj).forEach(key => {
                cloned[key] = this.#deepClone(obj[key]);
            });
            return cloned;
        }
    }
    
    // Приватный метод логирования изменений
    #logModification(operation, key, oldValue, newValue) {
        this.#modificationLog.push({
            timestamp: new Date(),
            operation,
            key,
            oldValue: this.#deepClone(oldValue),
            newValue: this.#deepClone(newValue)
        });
    }
    
    // Проверка ключа доступа
    #validateAccess(key) {
        if (key !== this.#accessKey) {
            throw new Error('Неверный ключ доступа');
        }
    }
    
    // Публичные методы с проверкой доступа
    get(key, accessKey) {
        this.#validateAccess(accessKey);
        const value = this.#data.get(key);
        return value !== undefined ? this.#deepClone(value) : undefined;
    }
    
    set(key, value, accessKey) {
        this.#validateAccess(accessKey);
        
        const oldValue = this.#data.get(key);
        this.#data.set(key, this.#deepClone(value));
        this.#logModification('SET', key, oldValue, value);
        
        return true;
    }
    
    delete(key, accessKey) {
        this.#validateAccess(accessKey);
        
        const oldValue = this.#data.get(key);
        const deleted = this.#data.delete(key);
        
        if (deleted) {
            this.#logModification('DELETE', key, oldValue, undefined);
        }
        
        return deleted;
    }
    
    has(key, accessKey) {
        this.#validateAccess(accessKey);
        return this.#data.has(key);
    }
    
    keys(accessKey) {
        this.#validateAccess(accessKey);
        return Array.from(this.#data.keys());
    }
    
    size(accessKey) {
        this.#validateAccess(accessKey);
        return this.#data.size;
    }
    
    // Метод для получения информации о контейнере (без доступа к данным)
    getInfo(accessKey) {
        this.#validateAccess(accessKey);
        return {
            creationTime: this.#creationTime,
            size: this.#data.size,
            modificationCount: this.#modificationLog.length
        };
    }
    
    // Метод для получения лога изменений
    getModificationLog(accessKey) {
        this.#validateAccess(accessKey);
        return this.#modificationLog.map(entry => ({
            ...entry,
            oldValue: undefined, // Не возвращаем старые значения для безопасности
            newValue: undefined  // Не возвращаем новые значения для безопасности
        }));
    }
    
    // Метод для смены ключа доступа
    changeAccessKey(oldKey, newKey) {
        this.#validateAccess(oldKey);
        
        if (typeof newKey !== 'string' || newKey.length < 8) {
            throw new Error('Новый ключ должен быть строкой длиной не менее 8 символов');
        }
        
        this.#accessKey = newKey;
        this.#logModification('CHANGE_KEY', 'accessKey', 'old', 'new');
    }
    
    // Метод для получения публичного хэша данных (для проверки целостности)
    getDataHash(accessKey) {
        this.#validateAccess(accessKey);
        const dataStr = JSON.stringify([...this.#data.entries()].sort());
        // Простой хэш (в реальном приложении используйте криптографический хэш)
        let hash = 0;
        for (let i = 0; i < dataStr.length; i++) {
            const char = dataStr.charCodeAt(i);
            hash = ((hash << 5) - hash) + char;
            hash = hash & hash; // Преобразование в 32-битное целое
        }
        return Math.abs(hash).toString(16);
    }
}

// Использование
const container = new SecureDataContainer({ name: 'John', age: 30 });
const accessKey = container.getDataHash('some_key'); // На самом деле ключ генерируется внутри

try {
    container.set('email', 'john@example.com', container['#accessKey']);
    console.log('Email установлен');
} catch (error) {
    console.error('Ошибка доступа:', error.message);
}
```

## Полиморфизм

Полиморфизм позволяет объектам разных классов отвечать на одни и те же сообщения (вызовы методов) по-своему.

### Основы полиморфизма

```javascript
// Интерфейс (в виде абстрактного класса)
class Drawable {
    constructor() {
        if (this.constructor === Drawable) {
            throw new Error('Cannot instantiate abstract class Drawable');
        }
    }
    
    draw(context) {
        throw new Error('Method draw() must be implemented');
    }
    
    // Общий метод для всех рисуемых объектов
    isVisible() {
        return this.opacity > 0;
    }
}

// Реализации интерфейса
class Circle extends Drawable {
    constructor(x, y, radius, color = 'black', opacity = 1) {
        super();
        this.x = x;
        this.y = y;
        this.radius = radius;
        this.color = color;
        this.opacity = opacity;
    }
    
    draw(context) {
        context.save();
        context.globalAlpha = this.opacity;
        context.beginPath();
        context.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
        context.fillStyle = this.color;
        context.fill();
        context.restore();
    }
    
    // Метод, специфичный для круга
    containsPoint(x, y) {
        const distance = Math.sqrt((x - this.x) ** 2 + (y - this.y) ** 2);
        return distance <= this.radius;
    }
}

class Rectangle extends Drawable {
    constructor(x, y, width, height, color = 'black', opacity = 1) {
        super();
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
        this.color = color;
        this.opacity = opacity;
    }
    
    draw(context) {
        context.save();
        context.globalAlpha = this.opacity;
        context.fillStyle = this.color;
        context.fillRect(this.x, this.y, this.width, this.height);
        context.restore();
    }
    
    // Метод, специфичный для прямоугольника
    containsPoint(x, y) {
        return x >= this.x && x <= this.x + this.width &&
               y >= this.y && y <= this.y + this.height;
    }
}

class Text extends Drawable {
    constructor(x, y, content, fontSize = 16, color = 'black', opacity = 1) {
        super();
        this.x = x;
        this.y = y;
        this.content = content;
        this.fontSize = fontSize;
        this.color = color;
        this.opacity = opacity;
    }
    
    draw(context) {
        context.save();
        context.globalAlpha = this.opacity;
        context.fillStyle = this.color;
        context.font = `${this.fontSize}px Arial`;
        context.fillText(this.content, this.x, this.y);
        context.restore();
    }
    
    // Метод, специфичный для текста
    measure(context) {
        context.font = `${this.fontSize}px Arial`;
        const metrics = context.measureText(this.content);
        return {
            width: metrics.width,
            height: this.fontSize
        };
    }
}

// Полиморфное использование
class CanvasRenderer {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.objects = [];
    }
    
    addObject(drawable) {
        if (!(drawable instanceof Drawable)) {
            throw new Error('Object must extend Drawable class');
        }
        this.objects.push(drawable);
    }
    
    removeObject(drawable) {
        const index = this.objects.indexOf(drawable);
        if (index > -1) {
            this.objects.splice(index, 1);
        }
    }
    
    // Полиморфный метод рендеринга
    render() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        this.objects.forEach(obj => {
            if (obj.isVisible()) {
                obj.draw(this.ctx); // Вызов нужной реализации draw()
            }
        });
    }
    
    // Поиск объектов в определенной точке
    getObjectsAt(x, y) {
        return this.objects.filter(obj => 
            obj.containsPoint && obj.containsPoint(x, y)
        );
    }
}

// Использование полиморфизма
const canvas = document.createElement('canvas');
canvas.width = 800;
canvas.height = 600;

const renderer = new CanvasRenderer(canvas);

// Добавление разных типов объектов
renderer.addObject(new Circle(100, 100, 50, 'red'));
renderer.addObject(new Rectangle(200, 100, 100, 80, 'blue'));
renderer.addObject(new Text(300, 100, 'Hello World', 20, 'green'));

// Рендеринг - каждый объект использует свою реализацию draw()
renderer.render();
```

## Абстракция

Абстракция позволяет скрывать сложность реализации и показывать только необходимые детали.

### Абстрактные классы и интерфейсы

```javascript
// Абстрактный класс для репозитория данных
class Repository {
    constructor() {
        if (this.constructor === Repository) {
            throw new Error('Cannot instantiate abstract class Repository');
        }
    }
    
    // Абстрактные методы
    async find(id) {
        throw new Error('Method find() must be implemented');
    }
    
    async findAll(options = {}) {
        throw new Error('Method findAll() must be implemented');
    }
    
    async create(data) {
        throw new Error('Method create() must be implemented');
    }
    
    async update(id, data) {
        throw new Error('Method update() must be implemented');
    }
    
    async delete(id) {
        throw new Error('Method delete() must be implemented');
    }
    
    // Общие вспомогательные методы
    validateId(id) {
        if (!id) {
            throw new Error('ID is required');
        }
        return true;
    }
    
    validateData(data) {
        if (!data || typeof data !== 'object') {
            throw new Error('Data must be a non-null object');
        }
        return true;
    }
}

// Конкретная реализация для пользователей
class UserRepository extends Repository {
    constructor(database) {
        super();
        this.db = database;
        this.collection = 'users';
    }
    
    async find(id) {
        this.validateId(id);
        const user = await this.db.collection(this.collection).findOne({ _id: id });
        return user ? this.#mapFromDb(user) : null;
    }
    
    async findAll(options = {}) {
        const { limit = 10, offset = 0, sort = {} } = options;
        
        const users = await this.db.collection(this.collection)
            .find({})
            .sort(sort)
            .skip(offset)
            .limit(limit)
            .toArray();
        
        return users.map(user => this.#mapFromDb(user));
    }
    
    async create(userData) {
        this.validateData(userData);
        
        const user = {
            ...userData,
            createdAt: new Date(),
            updatedAt: new Date(),
            isActive: true
        };
        
        const result = await this.db.collection(this.collection).insertOne(user);
        return this.#mapFromDb({ ...user, _id: result.insertedId });
    }
    
    async update(id, userData) {
        this.validateId(id);
        this.validateData(userData);
        
        const updateData = {
            ...userData,
            updatedAt: new Date()
        };
        
        const result = await this.db.collection(this.collection)
            .updateOne({ _id: id }, { $set: updateData });
        
        if (result.matchedCount === 0) {
            throw new Error(`User with id ${id} not found`);
        }
        
        return await this.find(id);
    }
    
    async delete(id) {
        this.validateId(id);
        
        const result = await this.db.collection(this.collection)
            .deleteOne({ _id: id });
        
        return result.deletedCount > 0;
    }
    
    // Приватный метод для преобразования данных из БД
    #mapFromDb(dbUser) {
        return {
            id: dbUser._id.toString(),
            name: dbUser.name,
            email: dbUser.email,
            createdAt: dbUser.createdAt,
            updatedAt: dbUser.updatedAt,
            isActive: dbUser.isActive
        };
    }
}

// Абстрактный класс для сервиса
class Service {
    constructor(repository) {
        if (this.constructor === Service) {
            throw new Error('Cannot instantiate abstract class Service');
        }
        this.repository = repository;
    }
    
    async getById(id) {
        return await this.repository.find(id);
    }
    
    async getAll(options = {}) {
        return await this.repository.findAll(options);
    }
    
    async create(data) {
        // Бизнес-логика перед созданием
        this.validateForCreate(data);
        return await this.repository.create(data);
    }
    
    async update(id, data) {
        // Бизнес-логика перед обновлением
        this.validateForUpdate(id, data);
        return await this.repository.update(id, data);
    }
    
    async remove(id) {
        // Бизнес-логика перед удалением
        this.validateForDelete(id);
        return await this.repository.delete(id);
    }
    
    // Абстрактные методы валидации
    validateForCreate(data) {
        throw new Error('Method validateForCreate() must be implemented');
    }
    
    validateForUpdate(id, data) {
        throw new Error('Method validateForUpdate() must be implemented');
    }
    
    validateForDelete(id) {
        throw new Error('Method validateForDelete() must be implemented');
    }
}

// Конкретная реализация сервиса
class UserService extends Service {
    validateForCreate(userData) {
        if (!userData.name) {
            throw new Error('Name is required');
        }
        if (!userData.email || !userData.email.includes('@')) {
            throw new Error('Valid email is required');
        }
    }
    
    validateForUpdate(id, userData) {
        this.validateForCreate(userData);
    }
    
    validateForDelete(id) {
        // Проверка, что пользователь может быть удален
        if (id === 'admin') {
            throw new Error('Cannot delete admin user');
        }
    }
    
    // Дополнительные бизнес-методы
    async findByEmail(email) {
        // Предполагается, что репозиторий имеет метод для поиска по email
        const users = await this.repository.findAll();
        return users.find(user => user.email === email);
    }
    
    async activateUser(id) {
        const user = await this.getById(id);
        if (!user) {
            throw new Error(`User with id ${id} not found`);
        }
        
        return await this.update(id, { isActive: true });
    }
    
    async deactivateUser(id) {
        const user = await this.getById(id);
        if (!user) {
            throw new Error(`User with id ${id} not found`);
        }
        
        return await this.update(id, { isActive: false });
    }
}

// Использование абстракций
// const db = new DatabaseConnection(); // предполагаем, что есть класс DatabaseConnection
// const userRepository = new UserRepository(db);
// const userService = new UserService(userRepository);

// const newUser = await userService.create({
//     name: 'John Doe',
//     email: 'john@example.com'
// });
```

## Примеры из промышленной разработки

### Система управления содержимым (CMS)

```javascript
// Иерархия классов для CMS
class Content {
    constructor(title, content, author, status = 'draft') {
        this.id = this.generateId();
        this.title = title;
        this.content = content;
        this.author = author;
        this.status = status;
        this.createdAt = new Date();
        this.updatedAt = new Date();
        this.version = 1;
        this.metadata = {};
    }
    
    generateId() {
        return Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
    
    update(content) {
        this.content = content;
        this.updatedAt = new Date();
        this.version++;
    }
    
    publish() {
        this.status = 'published';
        this.updatedAt = new Date();
    }
    
    unpublish() {
        this.status = 'draft';
        this.updatedAt = new Date();
    }
    
    toJSON() {
        return {
            id: this.id,
            title: this.title,
            content: this.content,
            author: this.author,
            status: this.status,
            createdAt: this.createdAt,
            updatedAt: this.updatedAt,
            version: this.version,
            metadata: this.metadata
        };
    }
}

class Article extends Content {
    constructor(title, content, author, tags = [], category = 'general') {
        super(title, content, author);
        this.tags = tags;
        this.category = category;
        this.readTime = this.calculateReadTime();
        this.comments = [];
    }
    
    calculateReadTime() {
        const wordsPerMinute = 200;
        const words = this.content.split(/\s+/).length;
        return Math.ceil(words / wordsPerMinute);
    }
    
    addTag(tag) {
        if (!this.tags.includes(tag)) {
            this.tags.push(tag);
        }
    }
    
    removeTag(tag) {
        this.tags = this.tags.filter(t => t !== tag);
    }
    
    addComment(comment) {
        this.comments.push(comment);
    }
    
    toJSON() {
        return {
            ...super.toJSON(),
            type: 'article',
            tags: this.tags,
            category: this.category,
            readTime: this.readTime,
            commentCount: this.comments.length
        };
    }
}

class Page extends Content {
    constructor(title, content, author, slug, template = 'default') {
        super(title, content, author);
        this.slug = slug;
        this.template = template;
        this.navigation = true;
    }
    
    setSlug(newSlug) {
        this.slug = newSlug.toLowerCase().replace(/\s+/g, '-');
    }
    
    hideFromNavigation() {
        this.navigation = false;
    }
    
    showInNavigation() {
        this.navigation = true;
    }
    
    toJSON() {
        return {
            ...super.toJSON(),
            type: 'page',
            slug: this.slug,
            template: this.template,
            navigation: this.navigation
        };
    }
}

// Менеджер содержимого
class ContentManager {
    constructor() {
        this.contents = new Map();
        this.observers = [];
    }
    
    addContent(content) {
        this.contents.set(content.id, content);
        this.notifyObservers('contentAdded', content);
        return content.id;
    }
    
    getContent(id) {
        return this.contents.get(id);
    }
    
    updateContent(id, updates) {
        const content = this.contents.get(id);
        if (!content) {
            throw new Error(`Content with id ${id} not found`);
        }
        
        Object.assign(content, updates, { updatedAt: new Date() });
        this.notifyObservers('contentUpdated', content);
        return content;
    }
    
    deleteContent(id) {
        const content = this.contents.get(id);
        if (content) {
            this.contents.delete(id);
            this.notifyObservers('contentDeleted', content);
            return true;
        }
        return false;
    }
    
    search(query) {
        return Array.from(this.contents.values()).filter(content =>
            content.title.toLowerCase().includes(query.toLowerCase()) ||
            content.content.toLowerCase().includes(query.toLowerCase())
        );
    }
    
    subscribe(observer) {
        this.observers.push(observer);
    }
    
    unsubscribe(observer) {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
            this.observers.splice(index, 1);
        }
    }
    
    notifyObservers(event, data) {
        this.observers.forEach(observer => {
            if (typeof observer.update === 'function') {
                observer.update(event, data);
            }
        });
    }
    
    getStatistics() {
        const stats = {
            total: this.contents.size,
            published: 0,
            draft: 0,
            types: {}
        };
        
        this.contents.forEach(content => {
            if (content.status === 'published') stats.published++;
            else stats.draft++;
            
            const type = content.constructor.name.toLowerCase();
            stats.types[type] = (stats.types[type] || 0) + 1;
        });
        
        return stats;
    }
}

// Наблюдатель для логирования
class LoggerObserver {
    update(event, data) {
        console.log(`[CMS Logger] Event: ${event}`, {
            id: data.id,
            title: data.title,
            timestamp: new Date().toISOString()
        });
    }
}

// Использование CMS
const cms = new ContentManager();
const logger = new LoggerObserver();

cms.subscribe(logger);

// Создание контента
const article = new Article(
    'Введение в ООП',
    'Объектно-ориентированное программирование - это парадигма...',
    'admin',
    ['javascript', 'oop', 'programming'],
    'education'
);

const page = new Page(
    'О нас',
    'Информация о нашей компании...',
    'admin',
    'about-us',
    'standard'
);

cms.addContent(article);
cms.addContent(page);

console.log('Статистика CMS:', cms.getStatistics());
```

### Система событий и наблюдателей

```javascript
// Система событий на основе ООП
class EventEmitter {
    constructor() {
        this.events = new Map();
    }
    
    on(event, listener) {
        if (!this.events.has(event)) {
            this.events.set(event, []);
        }
        this.events.get(event).push(listener);
    }
    
    once(event, listener) {
        const onceWrapper = (...args) => {
            listener(...args);
            this.off(event, onceWrapper);
        };
        this.on(event, onceWrapper);
    }
    
    off(event, listener) {
        if (this.events.has(event)) {
            const listeners = this.events.get(event);
            const index = listeners.indexOf(listener);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        }
    }
    
    emit(event, ...args) {
        if (this.events.has(event)) {
            const listeners = [...this.events.get(event)]; // Копия для безопасности
            listeners.forEach(listener => {
                try {
                    listener(...args);
                } catch (error) {
                    console.error(`Error in event listener for '${event}':`, error);
                }
            });
        }
    }
    
    removeAllListeners(event) {
        if (event) {
            this.events.delete(event);
        } else {
            this.events.clear();
        }
    }
    
    listenerCount(event) {
        return this.events.has(event) ? this.events.get(event).length : 0;
    }
}

// Базовый класс для сущностей, которые могут генерировать события
class EventEmittingEntity extends EventEmitter {
    constructor() {
        super();
        this.createdAt = new Date();
        this.isActive = true;
    }
    
    destroy() {
        this.isActive = false;
        this.emit('destroy');
        this.removeAllListeners();
    }
    
    activate() {
        if (!this.isActive) {
            this.isActive = true;
            this.emit('activate');
        }
    }
    
    deactivate() {
        if (this.isActive) {
            this.isActive = false;
            this.emit('deactivate');
        }
    }
}

// Конкретная реализация сущности
class User extends EventEmittingEntity {
    constructor(name, email) {
        super();
        this.name = name;
        this.email = email;
        this.loginCount = 0;
        this.lastLogin = null;
        
        // Подписка на внутренние события
        this.on('login', () => {
            this.loginCount++;
            this.lastLogin = new Date();
        });
    }
    
    login(credentials) {
        // Проверка учетных данных (упрощенно)
        if (credentials.email === this.email) {
            this.emit('login', { userId: this.id, timestamp: new Date() });
            return true;
        }
        return false;
    }
    
    updateProfile(updates) {
        const oldProfile = { ...this };
        Object.assign(this, updates);
        this.emit('profileUpdated', { oldProfile, newProfile: this });
    }
}

// Использование системы событий
const user = new User('John Doe', 'john@example.com');

user.on('login', (data) => {
    console.log(`Пользователь вошел: ${data.userId}`);
});

user.on('profileUpdated', (data) => {
    console.log('Профиль обновлен', data);
});

user.login({ email: 'john@example.com' });
user.updateProfile({ name: 'Johnny Doe' });
```

## Лучшие практики

### 1. Правильное использование наследования

```javascript
// Хорошо: IS-A relationship
class Vehicle {
    start() { console.log('Vehicle starting...'); }
}

class Car extends Vehicle {
    start() { 
        super.start();
        console.log('Car engine started');
    }
}

// Избегайте: наследования только для повторного использования кода
// Лучше использовать композицию
class Engine {
    start() { console.log('Engine started'); }
}

class Car {
    constructor() {
        this.engine = new Engine();
    }
    
    start() {
        this.engine.start();
    }
}
```

### 2. Инкапсуляция с использованием приватных полей

```javascript
class BankAccount {
    #balance = 0;
    #transactionHistory = [];
    
    constructor(initialBalance = 0) {
        if (initialBalance < 0) {
            throw new Error('Initial balance cannot be negative');
        }
        this.#balance = initialBalance;
    }
    
    getBalance() {
        return this.#balance;
    }
    
    deposit(amount) {
        if (amount <= 0) {
            throw new Error('Deposit amount must be positive');
        }
        this.#balance += amount;
        this.#addTransaction('DEPOSIT', amount);
    }
    
    #addTransaction(type, amount) {
        this.#transactionHistory.push({ type, amount, timestamp: new Date() });
    }
}
```

## Безопасность

При использовании ООП важно учитывать безопасность:

```javascript
// Безопасная обработка данных в классах
class SecureEntity {
    #data = new Map();
    
    constructor(initialData = {}) {
        // Валидация и очистка входных данных
        Object.entries(initialData).forEach(([key, value]) => {
            this.set(key, this.sanitize(value));
        });
    }
    
    sanitize(value) {
        // Очистка потенциально опасных данных
        if (typeof value === 'string') {
            return value
                .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
                .replace(/javascript:/gi, '')
                .replace(/vbscript:/gi, '');
        }
        return value;
    }
    
    set(key, value) {
        // Валидация ключа
        if (typeof key !== 'string' || !/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(key)) {
            throw new Error('Invalid key format');
        }
        
        this.#data.set(key, this.sanitize(value));
    }
    
    get(key) {
        return this.#data.get(key);
    }
    
    toJSON() {
        // Безопасное преобразование в JSON
        const obj = {};
        for (const [key, value] of this.#data) {
            obj[key] = value;
        }
        return obj;
    }
}
```

## Теги

#javascript #oop #object-oriented-programming #classes #inheritance #encapsulation #polymorphism #abstraction #design-patterns