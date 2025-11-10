# Patterns API

## Введение

Паттерны проектирования - это проверенные решения типичных проблем в разработке программного обеспечения. В этой главе мы рассмотрим основные паттерны проектирования и их реализацию в JavaScript: порождающие, структурные, поведенческие паттерны и современные подходы к проектированию.

## Содержание

- [[Порождающие паттерны]]
- [[Структурные паттерны]]
- [[Поведенческие паттерны]]
- [[Функциональные паттерны]]
- [[Архитектурные паттерны]]
- [[Паттерны для асинхронного кода]]
- [[Паттерны безопасности]]
- [[Современные паттерны]]

## Порождающие паттерны

Порождающие паттерны отвечают за удобное и безопасное создание новых объектов или даже целых семейств объектов.

### Фабричный метод (Factory Method)

```javascript
// Базовый класс продукта
class Animal {
    speak() {
        throw new Error('Method speak() must be implemented');
    }
}

// Конкретные продукты
class Dog extends Animal {
    speak() {
        return 'Woof!';
    }
}

class Cat extends Animal {
    speak() {
        return 'Meow!';
    }
}

class Bird extends Animal {
    speak() {
        return 'Tweet!';
    }
}

// Фабрика животных
class AnimalFactory {
    static createAnimal(type) {
        switch (type.toLowerCase()) {
            case 'dog':
                return new Dog();
            case 'cat':
                return new Cat();
            case 'bird':
                return new Bird();
            default:
                throw new Error(`Unknown animal type: ${type}`);
        }
    }
}

// Использование
const dog = AnimalFactory.createAnimal('dog');
console.log(dog.speak()); // "Woof!"

const cat = AnimalFactory.createAnimal('cat');
console.log(cat.speak()); // "Meow!"
```

### Абстрактная фабрика (Abstract Factory)

```javascript
// Абстрактная фабрика
class UIAbstractFactory {
    createButton() {
        throw new Error('createButton() must be implemented');
    }
    
    createCheckbox() {
        throw new Error('createCheckbox() must be implemented');
    }
}

// Конкретные фабрики
class WindowsFactory extends UIAbstractFactory {
    createButton() {
        return new WindowsButton();
    }
    
    createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory extends UIAbstractFactory {
    createButton() {
        return new MacButton();
    }
    
    createCheckbox() {
        return new MacCheckbox();
    }
}

// Продукты
class Button {
    render() {
        throw new Error('render() must be implemented');
    }
}

class Checkbox {
    render() {
        throw new Error('render() must be implemented');
    }
}

class WindowsButton extends Button {
    render() {
        return '<button class="windows-btn">Windows Button</button>';
    }
}

class WindowsCheckbox extends Checkbox {
    render() {
        return '<input type="checkbox" class="windows-checkbox">';
    }
}

class MacButton extends Button {
    render() {
        return '<button class="mac-btn">Mac Button</button>';
    }
}

class MacCheckbox extends Checkbox {
    render() {
        return '<input type="checkbox" class="mac-checkbox">';
    }
}

// Клиентский код
class Application {
    constructor(factory) {
        this.factory = factory;
        this.button = null;
        this.checkbox = null;
    }
    
    createUI() {
        this.button = this.factory.createButton();
        this.checkbox = this.factory.createCheckbox();
    }
    
    render() {
        return this.button.render() + this.checkbox.render();
    }
}

// Использование
const windowsApp = new Application(new WindowsFactory());
windowsApp.createUI();
console.log(windowsApp.render());

const macApp = new Application(new MacFactory());
macApp.createUI();
console.log(macApp.render());
```

### Строитель (Builder)

```javascript
// Продукт
class Car {
    constructor() {
        this.type = '';
        this.engine = '';
        this.wheels = 4;
        this.color = 'white';
        this.options = [];
    }
    
    toString() {
        return `Car: ${this.type}, Engine: ${this.engine}, Wheels: ${this.wheels}, Color: ${this.color}, Options: [${this.options.join(', ')}]`;
    }
}

// Строитель
class CarBuilder {
    constructor() {
        this.car = new Car();
    }
    
    setType(type) {
        this.car.type = type;
        return this;
    }
    
    setEngine(engine) {
        this.car.engine = engine;
        return this;
    }
    
    setWheels(wheels) {
        this.car.wheels = wheels;
        return this;
    }
    
    setColor(color) {
        this.car.color = color;
        return this;
    }
    
    addOption(option) {
        this.car.options.push(option);
        return this;
    }
    
    build() {
        return this.car;
    }
}

// Директор (опционально)
class CarDirector {
    static constructSportsCar(builder) {
        return builder
            .setType('Sports')
            .setEngine('V8')
            .setColor('Red')
            .addOption('Leather Seats')
            .addOption('GPS Navigation')
            .addOption('Premium Sound System')
            .build();
    }
    
    static constructFamilyCar(builder) {
        return builder
            .setType('Family')
            .setEngine('V4')
            .setColor('Blue')
            .addOption('Child Safety Locks')
            .addOption('Rear Entertainment')
            .build();
    }
}

// Использование
const sportsCar = CarDirector.constructSportsCar(new CarBuilder());
console.log(sportsCar.toString());

const familyCar = CarDirector.constructFamilyCar(new CarBuilder());
console.log(familyCar.toString());

// Использование без директора
const customCar = new CarBuilder()
    .setType('Custom')
    .setEngine('Electric')
    .setColor('Green')
    .setWheels(3)
    .addOption('Solar Panels')
    .build();

console.log(customCar.toString());
```

### Прототип (Prototype)

```javascript
// Прототип
class DocumentPrototype {
    constructor(type, content, style = {}) {
        this.type = type;
        this.content = content;
        this.style = style;
        this.metadata = {
            createdAt: new Date(),
            version: 1
        };
    }
    
    clone() {
        const cloned = Object.create(Object.getPrototypeOf(this));
        cloned.type = this.type;
        cloned.content = typeof this.content === 'object' 
            ? JSON.parse(JSON.stringify(this.content)) 
            : this.content;
        cloned.style = { ...this.style };
        cloned.metadata = { 
            ...this.metadata, 
            createdAt: new Date(),
            version: this.metadata.version + 1
        };
        return cloned;
    }
    
    updateContent(newContent) {
        this.content = newContent;
        this.metadata.updatedAt = new Date();
    }
    
    addStyle(key, value) {
        this.style[key] = value;
    }
}

// Использование
const reportTemplate = new DocumentPrototype(
    'report', 
    { title: 'Monthly Report', sections: [] },
    { fontSize: '12px', fontFamily: 'Arial' }
);

const januaryReport = reportTemplate.clone();
januaryReport.updateContent({
    ...reportTemplate.content,
    title: 'January Report',
    month: 'January'
});

const februaryReport = reportTemplate.clone();
februaryReport.updateContent({
    ...reportTemplate.content,
    title: 'February Report',
    month: 'February'
});

februaryReport.addStyle('color', 'blue');

console.log('January:', januaryReport);
console.log('February:', februaryReport);
```

### Одиночка (Singleton)

```javascript
// Классический Singleton
class DatabaseConnection {
    constructor(connectionString) {
        if (DatabaseConnection.instance) {
            return DatabaseConnection.instance;
        }
        
        this.connectionString = connectionString;
        this.isConnected = false;
        this.connectionPool = [];
        
        DatabaseConnection.instance = this;
        return this;
    }
    
    connect() {
        if (!this.isConnected) {
            console.log(`Connecting to database: ${this.connectionString}`);
            this.isConnected = true;
        }
        return this;
    }
    
    query(sql) {
        if (!this.isConnected) {
            throw new Error('Database not connected');
        }
        console.log(`Executing query: ${sql}`);
        return { result: 'query result', success: true };
    }
    
    disconnect() {
        if (this.isConnected) {
            console.log('Disconnecting from database');
            this.isConnected = false;
        }
    }
}

// Использование
const db1 = new DatabaseConnection('mongodb://localhost:27017');
const db2 = new DatabaseConnection('mongodb://localhost:27018'); // Вернет тот же экземпляр

console.log(db1 === db2); // true
db1.connect();
db2.query('SELECT * FROM users'); // Использует одно и то же подключение

// Современный Singleton с замыканием
const DatabaseManager = (function() {
    let instance;
    
    function createInstance(connectionString) {
        return {
            connectionString,
            isConnected: false,
            
            connect() {
                if (!this.isConnected) {
                    console.log(`Connecting to database: ${this.connectionString}`);
                    this.isConnected = true;
                }
                return this;
            },
            
            query(sql) {
                if (!this.isConnected) {
                    throw new Error('Database not connected');
                }
                console.log(`Executing query: ${sql}`);
                return { result: 'query result', success: true };
            }
        };
    }
    
    return {
        getInstance(connectionString) {
            if (!instance) {
                instance = createInstance(connectionString);
            }
            return instance;
        }
    };
})();

// Использование
const dbManager1 = DatabaseManager.getInstance('mysql://localhost:3306');
const dbManager2 = DatabaseManager.getInstance('postgres://localhost:5432');
console.log(dbManager1 === dbManager2); // true
```

## Структурные паттерны

Структурные паттерны показывают, как из классов и объектов составляются более крупные структуры.

### Адаптер (Adapter)

```javascript
// Целевой интерфейс
class Target {
    request() {
        return 'Target: The default target\'s behavior.';
    }
}

// Адаптируемый класс
class Adaptee {
    specificRequest() {
        return '.eetpadA eht fo roivaheb laicifitraP';
    }
}

// Адаптер
class Adapter extends Target {
    constructor(adaptee) {
        super();
        this.adaptee = adaptee;
    }
    
    request() {
        const result = this.adaptee.specificRequest();
        return `Adapter: (TRANSLATED) ${result.split('').reverse().join('')}`;
    }
}

// Использование
const target = new Target();
console.log(target.request());

const adaptee = new Adaptee();
console.log(`Adaptee: ${adaptee.specificRequest()}`);

const adapter = new Adapter(adaptee);
console.log(adapter.request());

// Практический пример: адаптер для разных API
class OldAPI {
    getDataOldFormat() {
        return {
            user_name: 'John Doe',
            user_email: 'john@example.com',
            user_age: 30
        };
    }
}

class NewAPI {
    getDataNewFormat() {
        return {
            name: 'John Doe',
            email: 'john@example.com',
            age: 30
        };
    }
}

class APIAdapter {
    constructor(oldAPI) {
        this.oldAPI = oldAPI;
    }
    
    getDataNewFormat() {
        const oldData = this.oldAPI.getDataOldFormat();
        return {
            name: oldData.user_name,
            email: oldData.user_email,
            age: oldData.user_age
        };
    }
}

// Использование
const oldAPI = new OldAPI();
const adaptedAPI = new APIAdapter(oldAPI);

console.log('Old API format:', oldAPI.getDataOldFormat());
console.log('New API format:', adaptedAPI.getDataNewFormat());
```

### Мост (Bridge)

```javascript
// Абстракция
class Abstraction {
    constructor(implementation) {
        this.implementation = implementation;
    }
    
    operation() {
        const result = this.implementation.operationImplementation();
        return `Abstraction: Base operation with:\n${result}`;
    }
}

// Уточненная абстракция
class ExtendedAbstraction extends Abstraction {
    operation() {
        const result = this.implementation.operationImplementation();
        return `ExtendedAbstraction: Extended operation with:\n${result}`;
    }
}

// Реализация
class Implementation {
    operationImplementation() {
        return 'ConcreteImplementationA: The platform A is ready.';
    }
}

class ConcreteImplementationA extends Implementation {
    operationImplementation() {
        return 'ConcreteImplementationA: Here\'s the result on platform A.';
    }
}

class ConcreteImplementationB extends Implementation {
    operationImplementation() {
        return 'ConcreteImplementationB: Here\'s the result on platform B.';
    }
}

// Использование
let implementation = new ConcreteImplementationA();
let abstraction = new Abstraction(implementation);
console.log(abstraction.operation());

implementation = new ConcreteImplementationB();
abstraction = new Abstraction(implementation);
console.log(abstraction.operation());

// Практический пример: мост для рендеринга UI
class UIWidget {
    constructor(renderer) {
        this.renderer = renderer;
    }
    
    render() {
        throw new Error('render() must be implemented');
    }
}

class Button extends UIWidget {
    constructor(renderer, text) {
        super(renderer);
        this.text = text;
    }
    
    render() {
        return this.renderer.renderButton(this.text);
    }
}

class Circle extends UIWidget {
    constructor(renderer, radius) {
        super(renderer);
        this.radius = radius;
    }
    
    render() {
        return this.renderer.renderCircle(this.radius);
    }
}

class VectorRenderer {
    renderButton(text) {
        return `Drawing a vector button with text: ${text}`;
    }
    
    renderCircle(radius) {
        return `Drawing a vector circle with radius: ${radius}`;
    }
}

class RasterRenderer {
    renderButton(text) {
        return `Drawing a raster button with text: ${text}`;
    }
    
    renderCircle(radius) {
        return `Drawing a raster circle with radius: ${radius}`;
    }
}

// Использование
const vectorButton = new Button(new VectorRenderer(), 'Click me');
const rasterButton = new Button(new RasterRenderer(), 'Click me too');
const vectorCircle = new Circle(new VectorRenderer(), 5);

console.log(vectorButton.render());
console.log(rasterButton.render());
console.log(vectorCircle.render());
```

### Компоновщик (Composite)

```javascript
// Компонент
class FileSystemItem {
    constructor(name) {
        this.name = name;
    }
    
    getName() {
        return this.name;
    }
    
    getSize() {
        throw new Error('getSize() must be implemented');
    }
    
    isFolder() {
        throw new Error('isFolder() must be implemented');
    }
}

// Лист
class File extends FileSystemItem {
    constructor(name, size) {
        super(name);
        this.size = size;
    }
    
    getSize() {
        return this.size;
    }
    
    isFolder() {
        return false;
    }
}

// Контейнер
class Folder extends FileSystemItem {
    constructor(name) {
        super(name);
        this.children = [];
    }
    
    add(item) {
        this.children.push(item);
    }
    
    remove(item) {
        const index = this.children.indexOf(item);
        if (index !== -1) {
            this.children.splice(index, 1);
        }
    }
    
    getSize() {
        return this.children.reduce((total, child) => total + child.getSize(), 0);
    }
    
    isFolder() {
        return true;
    }
    
    listItems() {
        return this.children.map(child => child.getName());
    }
}

// Использование
const root = new Folder('root');
const photos = new Folder('photos');
const documents = new Folder('documents');

const photo1 = new File('vacation.jpg', 2048);
const photo2 = new File('family.png', 1024);
const doc1 = new File('report.pdf', 512);
const doc2 = new File('notes.txt', 64);

photos.add(photo1);
photos.add(photo2);
documents.add(doc1);
documents.add(doc2);

root.add(photos);
root.add(documents);

console.log(`Root folder size: ${root.getSize()} KB`); // 3648 KB
console.log('Root items:', root.listItems()); // ['photos', 'documents']
console.log('Photos size:', photos.getSize()); // 3072 KB

// Практический пример: компоновщик для UI
class UIComponent {
    constructor(name) {
        this.name = name;
        this.children = [];
        this.parent = null;
    }
    
    add(component) {
        component.parent = this;
        this.children.push(component);
    }
    
    remove(component) {
        const index = this.children.indexOf(component);
        if (index !== -1) {
            component.parent = null;
            this.children.splice(index, 1);
        }
    }
    
    render() {
        let result = `<div class="component ${this.name}">`;
        result += `<span>${this.name}</span>`;
        
        for (const child of this.children) {
            result += child.render();
        }
        
        result += '</div>';
        return result;
    }
    
    findByName(name) {
        if (this.name === name) {
            return this;
        }
        
        for (const child of this.children) {
            const found = child.findByName(name);
            if (found) {
                return found;
            }
        }
        
        return null;
    }
}

class ButtonComponent extends UIComponent {
    constructor(name, text) {
        super(name);
        this.text = text;
    }
    
    render() {
        return `<button class="button ${this.name}">${this.text}</button>`;
    }
}

// Создание UI дерева
const app = new UIComponent('app');
const header = new UIComponent('header');
const nav = new UIComponent('navigation');
const content = new UIComponent('content');

const loginBtn = new ButtonComponent('login', 'Login');
const signupBtn = new ButtonComponent('signup', 'Sign Up');

nav.add(loginBtn);
nav.add(signupBtn);

header.add(nav);
content.add(new UIComponent('main-content'));

app.add(header);
app.add(content);

console.log(app.render());
```

### Декоратор (Decorator)

```javascript
// Базовый компонент
class Coffee {
    cost() {
        return 5; // Базовая стоимость
    }
    
    description() {
        return 'Coffee';
    }
}

// Декоратор
class CoffeeDecorator {
    constructor(coffee) {
        this.coffee = coffee;
    }
    
    cost() {
        return this.coffee.cost();
    }
    
    description() {
        return this.coffee.description();
    }
}

// Конкретные декораторы
class Milk extends CoffeeDecorator {
    cost() {
        return this.coffee.cost() + 1;
    }
    
    description() {
        return this.coffee.description() + ', Milk';
    }
}

class Sugar extends CoffeeDecorator {
    cost() {
        return this.coffee.cost() + 0.5;
    }
    
    description() {
        return this.coffee.description() + ', Sugar';
    }
}

class Chocolate extends CoffeeDecorator {
    cost() {
        return this.coffee.cost() + 2;
    }
    
    description() {
        return this.coffee.description() + ', Chocolate';
    }
}

// Использование
let coffee = new Coffee();
console.log(`${coffee.description()}: $${coffee.cost()}`); // Coffee: $5

coffee = new Milk(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`); // Coffee, Milk: $6

coffee = new Sugar(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`); // Coffee, Milk, Sugar: $6.5

coffee = new Chocolate(coffee);
console.log(`${coffee.description()}: $${coffee.cost()}`); // Coffee, Milk, Sugar, Chocolate: $8.5

// Практический пример: декоратор для HTTP запросов
class HttpClient {
    async request(url, options = {}) {
        const response = await fetch(url, {
            ...options,
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            }
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return response.json();
    }
}

class HttpClientDecorator {
    constructor(client) {
        this.client = client;
    }
    
    async request(url, options) {
        return this.client.request(url, options);
    }
}

class LoggingDecorator extends HttpClientDecorator {
    async request(url, options) {
        console.log(`Making request to: ${url}`);
        const result = await super.request(url, options);
        console.log(`Response from ${url}:`, result);
        return result;
    }
}

class RetryDecorator extends HttpClientDecorator {
    constructor(client, maxRetries = 3) {
        super(client);
        this.maxRetries = maxRetries;
    }
    
    async request(url, options) {
        let lastError;
        
        for (let i = 0; i <= this.maxRetries; i++) {
            try {
                return await super.request(url, options);
            } catch (error) {
                lastError = error;
                if (i < this.maxRetries) {
                    console.log(`Request failed, retrying... (${i + 1}/${this.maxRetries})`);
                    await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
                }
            }
        }
        
        throw lastError;
    }
}

class CachingDecorator extends HttpClientDecorator {
    constructor(client) {
        super(client);
        this.cache = new Map();
    }
    
    async request(url, options) {
        const cacheKey = `${url}_${JSON.stringify(options)}`;
        
        if (this.cache.has(cacheKey)) {
            console.log('Returning cached response');
            return this.cache.get(cacheKey);
        }
        
        const result = await super.request(url, options);
        this.cache.set(cacheKey, result);
        return result;
    }
}

// Комбинирование декораторов
let client = new HttpClient();
client = new LoggingDecorator(client);
client = new RetryDecorator(client, 2);
client = new CachingDecorator(client);

// client.request('/api/data'); // Будет использовать все декораторы
```

## Поведенческие паттерны

Поведенческие паттерны определяют способы взаимодействия между объектами.

### Наблюдатель (Observer)

```javascript
// Издатель
class Subject {
    constructor() {
        this.observers = [];
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
    
    notify(data) {
        this.observers.forEach(observer => observer.update(data));
    }
}

// Наблюдатель
class Observer {
    update(data) {
        throw new Error('update() must be implemented');
    }
}

// Конкретный наблюдатель
class ConcreteObserver extends Observer {
    constructor(name) {
        super();
        this.name = name;
    }
    
    update(data) {
        console.log(`${this.name} received update:`, data);
    }
}

// Использование
const subject = new Subject();

const observer1 = new ConcreteObserver('Observer 1');
const observer2 = new ConcreteObserver('Observer 2');
const observer3 = new ConcreteObserver('Observer 3');

subject.subscribe(observer1);
subject.subscribe(observer2);

subject.notify('Hello Observers!'); // Observer 1 и Observer 2 получат уведомление

subject.unsubscribe(observer2);
subject.notify('Second update'); // Только Observer 1 получит уведомление

// Практический пример: система событий
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }
    
    off(event, listener) {
        if (this.events[event]) {
            const index = this.events[event].indexOf(listener);
            if (index > -1) {
                this.events[event].splice(index, 1);
            }
        }
    }
    
    emit(event, ...args) {
        if (this.events[event]) {
            this.events[event].forEach(listener => {
                try {
                    listener(...args);
                } catch (error) {
                    console.error('Error in event listener:', error);
                }
            });
        }
    }
    
    once(event, listener) {
        const onceWrapper = (...args) => {
            listener(...args);
            this.off(event, onceWrapper);
        };
        this.on(event, onceWrapper);
    }
}

// Использование
const emitter = new EventEmitter();

emitter.on('user:login', (user) => {
    console.log(`User ${user.name} logged in`);
});

emitter.on('user:logout', (user) => {
    console.log(`User ${user.name} logged out`);
});

emitter.once('app:start', () => {
    console.log('Application started (this will only run once)');
});

emitter.emit('user:login', { name: 'John' });
emitter.emit('user:logout', { name: 'John' });
emitter.emit('app:start');
emitter.emit('app:start'); // Не выполнится снова
```

### Стратегия (Strategy)

```javascript
// Контекст
class ShoppingCart {
    constructor(strategy) {
        this.items = [];
        this.strategy = strategy;
    }
    
    setStrategy(strategy) {
        this.strategy = strategy;
    }
    
    addItem(item) {
        this.items.push(item);
    }
    
    calculateTotal() {
        return this.strategy.calculate(this.items);
    }
}

// Стратегии
class NormalStrategy {
    calculate(items) {
        return items.reduce((total, item) => total + item.price, 0);
    }
}

class DiscountStrategy {
    calculate(items) {
        const normalTotal = items.reduce((total, item) => total + item.price, 0);
        return normalTotal * 0.9; // 10% скидка
    }
}

class VipStrategy {
    calculate(items) {
        const normalTotal = items.reduce((total, item) => total + item.price, 0);
        return normalTotal * 0.8; // 20% скидка
    }
}

// Использование
const cart = new ShoppingCart(new NormalStrategy());

cart.addItem({ name: 'Laptop', price: 1000 });
cart.addItem({ name: 'Mouse', price: 25 });

console.log('Normal pricing:', cart.calculateTotal()); // 1025

cart.setStrategy(new DiscountStrategy());
console.log('Discount pricing:', cart.calculateTotal()); // 922.5

cart.setStrategy(new VipStrategy());
console.log('VIP pricing:', cart.calculateTotal()); // 820

// Практический пример: стратегия сортировки
class DataSorter {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    setStrategy(strategy) {
        this.strategy = strategy;
    }
    
    sort(data) {
        return this.strategy.sort(data);
    }
}

class BubbleSortStrategy {
    sort(data) {
        console.log('Using Bubble Sort');
        const arr = [...data]; // Создаем копию
        for (let i = 0; i < arr.length; i++) {
            for (let j = 0; j < arr.length - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                }
            }
        }
        return arr;
    }
}

class QuickSortStrategy {
    sort(data) {
        console.log('Using Quick Sort');
        if (data.length <= 1) return data;
        
        const pivot = data[Math.floor(data.length / 2)];
        const left = data.filter(x => x < pivot);
        const right = data.filter(x => x > pivot);
        const equal = data.filter(x => x === pivot);
        
        return [
            ...this.sort(left),
            ...equal,
            ...this.sort(right)
        ];
    }
}

class NativeSortStrategy {
    sort(data) {
        console.log('Using Native Sort');
        return [...data].sort((a, b) => a - b);
    }
}

// Использование
const sorter = new DataSorter(new BubbleSortStrategy());
const numbers = [64, 34, 25, 12, 22, 11, 90];

console.log('Bubble sorted:', sorter.sort(numbers));

sorter.setStrategy(new QuickSortStrategy());
console.log('Quick sorted:', sorter.sort(numbers));

sorter.setStrategy(new NativeSortStrategy());
console.log('Native sorted:', sorter.sort(numbers));
```

### Команда (Command)

```javascript
// Получатель
class TextEditor {
    constructor() {
        this.content = '';
        this.history = [];
    }
    
    write(text) {
        this.history.push(this.content);
        this.content += text;
        console.log(`Text written: ${text}`);
    }
    
    copy() {
        console.log(`Copied: ${this.content}`);
        return this.content;
    }
    
    paste(text) {
        this.history.push(this.content);
        this.content += text;
        console.log(`Pasted: ${text}`);
    }
    
    undo() {
        if (this.history.length > 0) {
            this.content = this.history.pop();
            console.log('Undo performed');
        }
    }
}

// Команда
class Command {
    execute() {
        throw new Error('execute() must be implemented');
    }
    
    undo() {
        throw new Error('undo() must be implemented');
    }
}

// Конкретные команды
class WriteCommand extends Command {
    constructor(editor, text) {
        super();
        this.editor = editor;
        this.text = text;
    }
    
    execute() {
        this.editor.write(this.text);
    }
    
    undo() {
        // Для упрощения, просто удаляем последний добавленный текст
        if (this.editor.content.endsWith(this.text)) {
            this.editor.content = this.editor.content.slice(0, -this.text.length);
        }
    }
}

class PasteCommand extends Command {
    constructor(editor, text) {
        super();
        this.editor = editor;
        this.text = text;
        this.previousContent = '';
    }
    
    execute() {
        this.previousContent = this.editor.content;
        this.editor.paste(this.text);
    }
    
    undo() {
        this.editor.content = this.previousContent;
    }
}

class UndoCommand extends Command {
    constructor(editor) {
        super();
        this.editor = editor;
    }
    
    execute() {
        this.editor.undo();
    }
    
    undo() {
        throw new Error('Undo command cannot be undone');
    }
}

// История команд для составной команды
class MacroCommand extends Command {
    constructor() {
        super();
        this.commands = [];
    }
    
    add(command) {
        this.commands.push(command);
    }
    
    execute() {
        this.commands.forEach(command => command.execute());
    }
    
    undo() {
        // Выполняем команды в обратном порядке
        for (let i = this.commands.length - 1; i >= 0; i--) {
            this.commands[i].undo();
        }
    }
}

// Использование
const editor = new TextEditor();
const invoker = {
    executeCommand(command) {
        command.execute();
    }
};

// Простые команды
const writeCmd = new WriteCommand(editor, 'Hello ');
invoker.executeCommand(writeCmd);

const pasteCmd = new PasteCommand(editor, 'World!');
invoker.executeCommand(pasteCmd);

console.log('Current content:', editor.content);

// Отмена
const undoCmd = new UndoCommand(editor);
invoker.executeCommand(undoCmd);
console.log('After undo:', editor.content);

// Макрокоманда
const macro = new MacroCommand();
macro.add(new WriteCommand(editor, 'Good '));
macro.add(new WriteCommand(editor, 'Morning!'));
macro.add(new WriteCommand(editor, ' How are you?'));
invoker.executeCommand(macro);
console.log('After macro:', editor.content);

// Отмена макрокоманды
macro.undo();
console.log('After macro undo:', editor.content);
```

## Примеры из промышленной разработки

### Архитектурный паттерн MVC

```javascript
// Модель
class UserModel {
    constructor() {
        this.users = [];
        this.observers = [];
    }
    
    addObserver(observer) {
        this.observers.push(observer);
    }
    
    notifyObservers(action, data) {
        this.observers.forEach(observer => observer.update(action, data));
    }
    
    addUser(userData) {
        const user = {
            id: Date.now(),
            ...userData,
            createdAt: new Date()
        };
        this.users.push(user);
        this.notifyObservers('userAdded', user);
        return user;
    }
    
    updateUser(id, userData) {
        const index = this.users.findIndex(user => user.id === id);
        if (index !== -1) {
            const oldUser = { ...this.users[index] };
            this.users[index] = { ...this.users[index], ...userData };
            this.notifyObservers('userUpdated', { oldUser, newUser: this.users[index] });
            return this.users[index];
        }
        return null;
    }
    
    deleteUser(id) {
        const index = this.users.findIndex(user => user.id === id);
        if (index !== -1) {
            const deletedUser = this.users.splice(index, 1)[0];
            this.notifyObservers('userDeleted', deletedUser);
            return deletedUser;
        }
        return null;
    }
    
    getAllUsers() {
        return [...this.users];
    }
    
    getUserById(id) {
        return this.users.find(user => user.id === id);
    }
}

// Представление
class UserView {
    constructor() {
        this.container = document.createElement('div');
        this.container.id = 'user-view';
    }
    
    render(users) {
        this.container.innerHTML = `
            <h2>Пользователи</h2>
            <div id="users-list">
                ${users.map(user => `
                    <div class="user-item" data-id="${user.id}">
                        <span>${user.name} (${user.email})</span>
                        <button class="edit-btn" data-id="${user.id}">Редактировать</button>
                        <button class="delete-btn" data-id="${user.id}">Удалить</button>
                    </div>
                `).join('')}
            </div>
            <button id="add-user-btn">Добавить пользователя</button>
        `;
        
        // Добавляем обработчики событий
        this.container.querySelectorAll('.delete-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const id = parseInt(e.target.dataset.id);
                this.onDeleteUser && this.onDeleteUser(id);
            });
        });
        
        this.container.querySelectorAll('.edit-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                const id = parseInt(e.target.dataset.id);
                this.onEditUser && this.onEditUser(id);
            });
        });
        
        document.getElementById('add-user-btn').addEventListener('click', () => {
            this.onAddUser && this.onAddUser();
        });
    }
    
    update(action, data) {
        console.log(`View updated: ${action}`, data);
        // Перерисовка при необходимости
        this.render(this.currentUsers || []);
    }
}

// Контроллер
class UserController {
    constructor(model, view) {
        this.model = model;
        this.view = view;
        
        // Подписка представления на изменения модели
        this.model.addObserver(this.view);
        
        // Настройка обработчиков представления
        this.view.onDeleteUser = this.handleDeleteUser.bind(this);
        this.view.onEditUser = this.handleEditUser.bind(this);
        this.view.onAddUser = this.handleAddUser.bind(this);
        
        // Инициальная отрисовка
        this.refreshView();
    }
    
    refreshView() {
        const users = this.model.getAllUsers();
        this.view.currentUsers = users;
        this.view.render(users);
    }
    
    handleAddUser() {
        const name = prompt('Имя пользователя:');
        const email = prompt('Email пользователя:');
        
        if (name && email) {
            this.model.addUser({ name, email });
            this.refreshView();
        }
    }
    
    handleEditUser(id) {
        const user = this.model.getUserById(id);
        if (user) {
            const name = prompt('Новое имя:', user.name);
            const email = prompt('Новый email:', user.email);
            
            if (name && email) {
                this.model.updateUser(id, { name, email });
                this.refreshView();
            }
        }
    }
    
    handleDeleteUser(id) {
        if (confirm('Удалить пользователя?')) {
            this.model.deleteUser(id);
            this.refreshView();
        }
    }
}

// Использование MVC
const userModel = new UserModel();
const userView = new UserView();
const userController = new UserController(userModel, userView);

// Добавление тестовых данных
userModel.addUser({ name: 'John Doe', email: 'john@example.com' });
userModel.addUser({ name: 'Jane Smith', email: 'jane@example.com' });
```

### Паттерн Цепочка обязанностей

```javascript
// Базовый обработчик
class Handler {
    setNext(handler) {
        this.nextHandler = handler;
        return handler;
    }
    
    handle(request) {
        if (this.nextHandler) {
            return this.nextHandler.handle(request);
        }
        
        return null;
    }
}

// Конкретные обработчики
class AuthenticationHandler extends Handler {
    handle(request) {
        if (!request.token) {
            return { success: false, error: 'Authentication required' };
        }
        
        // Проверка токена (упрощенно)
        if (request.token !== 'valid-token') {
            return { success: false, error: 'Invalid token' };
        }
        
        console.log('Authentication passed');
        return super.handle(request);
    }
}

class AuthorizationHandler extends Handler {
    handle(request) {
        if (!request.userRole) {
            return { success: false, error: 'Authorization required' };
        }
        
        // Проверка прав (упрощенно)
        if (request.action === 'delete' && request.userRole !== 'admin') {
            return { success: false, error: 'Insufficient permissions' };
        }
        
        console.log('Authorization passed');
        return super.handle(request);
    }
}

class ValidationHandler extends Handler {
    handle(request) {
        if (request.action === 'create' && !request.data) {
            return { success: false, error: 'Data is required for create action' };
        }
        
        if (request.action === 'update' && (!request.id || !request.data)) {
            return { success: false, error: 'ID and data are required for update action' };
        }
        
        console.log('Validation passed');
        return super.handle(request);
    }
}

class BusinessLogicHandler extends Handler {
    handle(request) {
        // Выполнение бизнес-логики
        console.log(`Executing business logic for action: ${request.action}`);
        
        // Имитация выполнения
        const result = {
            success: true,
            data: `Processed ${request.action} action`,
            timestamp: new Date()
        };
        
        return result;
    }
}

// Использование
const authHandler = new AuthenticationHandler();
const authzHandler = new AuthorizationHandler();
const validationHandler = new ValidationHandler();
const businessHandler = new BusinessLogicHandler();

// Создание цепочки
authHandler
    .setNext(authzHandler)
    .setNext(validationHandler)
    .setNext(businessHandler);

// Тестирование
const request1 = {
    token: 'valid-token',
    userRole: 'admin',
    action: 'create',
    data: { name: 'New Item' }
};

const result1 = authHandler.handle(request1);
console.log('Result 1:', result1);

const request2 = {
    token: 'invalid-token',
    action: 'create'
};

const result2 = authHandler.handle(request2);
console.log('Result 2:', result2);

// Практический пример: обработка HTTP запросов
class RequestProcessor {
    constructor() {
        this.middlewareChain = [];
    }
    
    use(middleware) {
        this.middlewareChain.push(middleware);
        return this;
    }
    
    async process(request) {
        let currentIndex = 0;
        
        const next = async () => {
            if (currentIndex < this.middlewareChain.length) {
                const middleware = this.middlewareChain[currentIndex++];
                return middleware(request, next);
            }
            return { status: 200, body: 'Request processed successfully' };
        };
        
        return next();
    }
}

// Примеры middleware
const loggingMiddleware = async (request, next) => {
    console.log(`Processing request: ${request.method} ${request.url}`);
    return next();
};

const corsMiddleware = async (request, next) => {
    request.headers = { ...request.headers, 'Access-Control-Allow-Origin': '*' };
    return next();
};

const rateLimitMiddleware = async (request, next) => {
    // Простая проверка ограничения частоты
    if (request.rateLimited) {
        return { status: 429, body: 'Rate limit exceeded' };
    }
    return next();
};

// Использование
const processor = new RequestProcessor();
processor
    .use(loggingMiddleware)
    .use(corsMiddleware)
    .use(rateLimitMiddleware);

const request = { method: 'GET', url: '/api/data', rateLimited: false };
processor.process(request).then(result => console.log('Final result:', result));
```

## Лучшие практики

### 1. Выбор правильного паттерна

```javascript
// Плохо: избыточное использование паттернов
class BadExample {
    // Использование паттерна "Состояние" для простого флага
    constructor() {
        this.state = new ActiveState(this);
    }
    
    toggle() {
        this.state.handle();
    }
}

// Хорошо: простое решение для простой задачи
class GoodExample {
    constructor() {
        this.isActive = true;
    }
    
    toggle() {
        this.isActive = !this.isActive;
    }
}
```

### 2. Комбинирование паттернов

```javascript
// Комбинация Observer + Strategy + Factory
class NotificationSystem {
    constructor() {
        this.observers = [];
        this.strategies = new Map();
        this.createStrategyFactory();
    }
    
    createStrategyFactory() {
        this.strategies.set('email', () => new EmailNotificationStrategy());
        this.strategies.set('sms', () => new SmsNotificationStrategy());
        this.strategies.set('push', () => new PushNotificationStrategy());
    }
    
    addObserver(observer) {
        this.observers.push(observer);
    }
    
    notify(message, type = 'email') {
        const strategy = this.strategies.get(type)?.();
        if (strategy) {
            const result = strategy.send(message);
            this.observers.forEach(observer => observer.update(result));
            return result;
        }
        throw new Error(`Unknown notification type: ${type}`);
    }
}

class EmailNotificationStrategy {
    send(message) {
        console.log(`Sending email: ${message}`);
        return { type: 'email', status: 'sent', message };
    }
}

class SmsNotificationStrategy {
    send(message) {
        console.log(`Sending SMS: ${message}`);
        return { type: 'sms', status: 'sent', message };
    }
}

class PushNotificationStrategy {
    send(message) {
        console.log(`Sending push notification: ${message}`);
        return { type: 'push', status: 'sent', message };
    }
}
```

## Безопасность

При реализации паттернов важно учитывать безопасность:

```javascript
// Безопасная реализация паттернов
class SecureCommand {
    constructor(action, data, user) {
        this.action = action;
        this.data = this.sanitizeData(data);
        this.user = user;
        this.timestamp = new Date();
    }
    
    sanitizeData(data) {
        // Очистка данных от потенциально опасного содержимого
        if (typeof data === 'string') {
            return data
                .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
                .replace(/javascript:/gi, '')
                .replace(/vbscript:/gi, '');
        }
        return data;
    }
    
    validate() {
        // Проверка прав пользователя на выполнение действия
        const allowedActions = {
            'user': ['read', 'update'],
            'admin': ['read', 'update', 'delete', 'create']
        };
        
        const userActions = allowedActions[this.user.role] || [];
        return userActions.includes(this.action);
    }
    
    execute() {
        if (!this.validate()) {
            throw new Error(`User ${this.user.id} is not authorized to perform ${this.action}`);
        }
        
        console.log(`Executing secure command: ${this.action} by user ${this.user.id}`);
        return { success: true, action: this.action, timestamp: this.timestamp };
    }
}

// Использование
const secureCommand = new SecureCommand('delete', '<script>alert("xss")</script>data', {
    id: 1,
    role: 'admin'
});

try {
    const result = secureCommand.execute();
    console.log('Secure command result:', result);
} catch (error) {
    console.error('Security error:', error.message);
}
```

## Теги

#javascript #design-patterns #factory #singleton #observer #strategy #command #adapter #decorator #mvc