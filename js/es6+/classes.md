# Классы в JavaScript

## Обзор

Классы в JavaScript (введенные в ES6) предоставляют более чистый и понятный синтаксис для создания объектов и реализации наследования. На самом деле, классы в JavaScript являются "синтаксическим сахаром" над прототипным наследованием.

## Основы классов

### Определение класса

```javascript
// Определение класса
class Person {
    // Конструктор класса
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    // Метод класса
    greet() {
        return `Привет, меня зовут ${this.name}, мне ${this.age} лет`;
    }
    
    // Метод для получения возраста в месяцах
    getAgeInMonths() {
        return this.age * 12;
    }
    
    // Сеттер
    set setName(newName) {
        if (typeof newName !== 'string' || newName.length === 0) {
            throw new Error('Имя должно быть непустой строкой');
        }
        this.name = newName;
    }
    
    // Геттер
    get getName() {
        return this.name;
    }
}

// Использование класса
const person1 = new Person("Иван", 30);
console.log(person1.greet()); // "Привет, меня зовут Иван, мне 30 лет"
console.log(person1.getAgeInMonths()); // 360
console.log(person1.getName); // "Иван"
person1.setName = "Петр";
console.log(person1.getName); // "Петр"
```

## Наследование классов

### Ключевое слово extends

```javascript
// Родительский класс
class Animal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
    }
    
    speak() {
        return `${this.name} издает звук`;
    }
    
    getInfo() {
        return `Это ${this.species} по имени ${this.name}`;
    }
}

// Дочерний класс
class Dog extends Animal {
    constructor(name, breed) {
        // Вызов конструктора родительского класса
        super(name, "Собака");
        this.breed = breed;
    }
    
    // Переопределение метода родителя
    speak() {
        return `${this.name} лает: Гав!`;
    }
    
    // Добавление нового метода
    fetch() {
        return `${this.name} принесает палку`;
    }
    
    // Метод, использующий метод родителя
    getInfoWithBreed() {
        return `${super.getInfo()}, порода: ${this.breed}`;
    }
}

const myDog = new Dog("Рекс", "Лабрадор");
console.log(myDog.speak()); // "Рекс лает: Гав!"
console.log(myDog.fetch()); // "Рекс принесает палку"
console.log(myDog.getInfoWithBreed()); // "Это Собака по имени Рекс, порода: Лабрадор"
```

## Статические методы и свойства

```javascript
class MathUtils {
    // Статическое свойство
    static PI = 3.14159;
    
    // Статический метод
    static calculateCircleArea(radius) {
        return this.PI * radius * radius;
    }
    
    static calculateCircleCircumference(radius) {
        return 2 * this.PI * radius;
    }
    
    // Нестатический метод (требует создания экземпляра)
    constructor() {
        this.instanceId = Math.random();
    }
    
    getInstanceInfo() {
        return `Экземпляр с ID: ${this.instanceId}`;
    }
}

// Использование статических методов и свойств
console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.calculateCircleArea(5)); // 78.53975
console.log(MathUtils.calculateCircleCircumference(5)); // 31.4159

// Создание экземпляра для использования нестатических методов
const mathInstance = new MathUtils();
console.log(mathInstance.getInstanceInfo()); // "Экземпляр с ID: [случайный ID]"
```

## Приватные поля и методы (ES2022+)

```javascript
class BankAccount {
    // Приватные поля
    #balance = 0;
    #accountNumber;
    #transactionHistory = [];
    
    constructor(initialBalance = 0, accountNumber) {
        if (initialBalance < 0) {
            throw new Error("Баланс не может быть отрицательным");
        }
        
        this.#balance = initialBalance;
        this.#accountNumber = accountNumber;
    }
    
    // Публичный метод для депозита
    deposit(amount) {
        if (amount <= 0) {
            throw new Error("Сумма депозита должна быть положительной");
        }
        
        this.#balance += amount;
        this.#addTransaction("deposit", amount);
        return this.#balance;
    }
    
    // Публичный метод для снятия
    withdraw(amount) {
        if (amount <= 0) {
            throw new Error("Сумма снятия должна быть положительной");
        }
        
        if (amount > this.#balance) {
            throw new Error("Недостаточно средств на счете");
        }
        
        this.#balance -= amount;
        this.#addTransaction("withdrawal", amount);
        return this.#balance;
    }
    
    // Публичный метод для получения баланса
    getBalance() {
        return this.#balance;
    }
    
    // Приватный метод (доступен только внутри класса)
    #addTransaction(type, amount) {
        this.#transactionHistory.push({
            type,
            amount,
            timestamp: new Date(),
            balanceAfter: this.#balance
        });
    }
    
    // Публичный метод для получения истории транзакций
    getTransactionHistory() {
        // Возвращаем копию истории для защиты
        return [...this.#transactionHistory];
    }
    
    // Приватное поле не может быть доступно извне
    // this.#balance; // Ошибка за пределами класса
}

// Использование
const account = new BankAccount(1000, "ACC001");
account.deposit(500);
account.withdraw(200);
console.log(account.getBalance()); // 1300

// account.#balance; // Ошибка: приватное поле недоступно извне
const history = account.getTransactionHistory();
console.log(history.length); // 2
```

## Примеры продвинутых классов

### Класс для управления списком задач

```javascript
class TaskManager {
    #tasks = [];
    #nextId = 1;
    
    constructor() {
        this.completedCount = 0;
        this.activeCount = 0;
    }
    
    addTask(description, priority = 'medium') {
        const task = {
            id: this.#nextId++,
            description,
            priority,
            completed: false,
            createdAt: new Date()
        };
        
        this.#tasks.push(task);
        this.#updateCounts();
        return task;
    }
    
    removeTask(id) {
        const initialLength = this.#tasks.length;
        this.#tasks = this.#tasks.filter(task => task.id !== id);
        
        if (this.#tasks.length < initialLength) {
            this.#updateCounts();
            return true;
        }
        return false;
    }
    
    completeTask(id) {
        const task = this.#tasks.find(task => task.id === id);
        if (task && !task.completed) {
            task.completed = true;
            task.completedAt = new Date();
            this.#updateCounts();
            return true;
        }
        return false;
    }
    
    getTasks(filter = 'all') {
        switch (filter) {
            case 'completed':
                return this.#tasks.filter(task => task.completed);
            case 'active':
                return this.#tasks.filter(task => !task.completed);
            case 'high-priority':
                return this.#tasks.filter(task => task.priority === 'high');
            default:
                return [...this.#tasks]; // Возвращаем копию
        }
    }
    
    getTaskStats() {
        return {
            total: this.#tasks.length,
            completed: this.completedCount,
            active: this.activeCount,
            highPriority: this.#tasks.filter(t => t.priority === 'high').length
        };
    }
    
    #updateCounts() {
        this.completedCount = this.#tasks.filter(t => t.completed).length;
        this.activeCount = this.#tasks.length - this.completedCount;
    }
}

// Использование
const taskManager = new TaskManager();
taskManager.addTask("Изучить JavaScript", "high");
taskManager.addTask("Написать код", "medium");
taskManager.completeTask(1);

console.log(taskManager.getTaskStats()); // { total: 2, completed: 1, active: 1, highPriority: 1 }
console.log(taskManager.getTasks('active')); // Задача "Написать код"
```

### Абстрактный класс (шаблон)

```javascript
class AbstractRepository {
    constructor() {
        if (this.constructor === AbstractRepository) {
            throw new Error("Нельзя создать экземпляр абстрактного класса");
        }
    }
    
    findAll() {
        throw new Error("Метод findAll должен быть реализован в дочернем классе");
    }
    
    findById(id) {
        throw new Error("Метод findById должен быть реализован в дочернем классе");
    }
    
    save(entity) {
        throw new Error("Метод save должен быть реализован в дочернем классе");
    }
    
    delete(id) {
        throw new Error("Метод delete должен быть реализован в дочернем классе");
    }
}

// Конкретная реализация
class UserRepository extends AbstractRepository {
    #users = [];
    #nextId = 1;
    
    findAll() {
        return [...this.#users];
    }
    
    findById(id) {
        return this.#users.find(user => user.id === id);
    }
    
    save(user) {
        if (!user.id) {
            user.id = this.#nextId++;
            this.#users.push(user);
        } else {
            const index = this.#users.findIndex(u => u.id === user.id);
            if (index !== -1) {
                this.#users[index] = { ...user };
            }
        }
        return user;
    }
    
    delete(id) {
        const initialLength = this.#users.length;
        this.#users = this.#users.filter(user => user.id !== id);
        return this.#users.length < initialLength;
    }
}

// Использование
const userRepository = new UserRepository();
userRepository.save({ name: "Иван", email: "ivan@example.com" });
console.log(userRepository.findAll()); // [{ id: 1, name: "Иван", email: "ivan@example.com" }]
```

## Особенности классов в JavaScript

1. Классы не подвержены поднятию (hoisting) - их нужно определять до использования
2. Тело класса выполняется в строгом режиме (strict mode)
3. Методы не перечисляемы (non-enumerable)
4. Классы всегда должны вызываться с `new`
5. Переопределять `constructor` не обязательно - есть конструктор по умолчанию