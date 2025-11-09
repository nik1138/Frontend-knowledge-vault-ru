# Прототипы и наследование в JavaScript

## Что такое прототипы

Прототипы - это фундаментальный механизм наследования в JavaScript. Каждый объект в JavaScript имеет внутреннее свойство, называемое прототипом, которое является ссылкой на другой объект. Когда вы пытаетесь получить доступ к свойству объекта, и это свойство не существует в самом объекте, JavaScript автоматически ищет его в прототипе объекта.

## Цепочка прототипов

### Базовое наследование

```javascript
// Создание объекта
const animal = {
  type: "животное",
  speak() {
    console.log(`${this.name} издает звук`);
  }
};

// Создание объекта с прототипом
const dog = Object.create(animal);
dog.name = "Бобик";
dog.breed = "Овчарка";

dog.speak(); // "Бобик издает звук"

// Проверка прототипа
console.log(Object.getPrototypeOf(dog) === animal); // true
console.log(dog.hasOwnProperty('name')); // true
console.log(dog.hasOwnProperty('speak')); // false
```

### Доступ к прототипу

```javascript
const obj = {};
const proto = { x: 1 };

// Установка прототипа
Object.setPrototypeOf(obj, proto);
console.log(obj.x); // 1

// Получение прототипа
console.log(Object.getPrototypeOf(obj) === proto); // true

// Альтернативный способ создания объекта с прототипом
const obj2 = Object.create(proto);
console.log(obj2.x); // 1
```

## Прототипы функций

### Свойство prototype

```javascript
function Person(name) {
  this.name = name;
}

// Добавление методов в прототип
Person.prototype.greet = function() {
  return `Привет, меня зовут ${this.name}`;
};

Person.prototype.age = 0;

const person1 = new Person("Иван");
const person2 = new Person("Мария");

console.log(person1.greet()); // "Привет, меня зовут Иван"
console.log(person2.greet()); // "Привет, меня зовут Мария"

// Все экземпляры разделяют один прототип
console.log(Person.prototype.isPrototypeOf(person1)); // true
console.log(Person.prototype.isPrototypeOf(person2)); // true
```

### Наследование через прототипы

```javascript
// Родительская функция
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} издает звук`);
};

// Дочерняя функция
function Dog(name, breed) {
  Animal.call(this, name); // Вызов родительского конструктора
  this.breed = breed;
}

// Наследование прототипа
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

// Добавление специфичных методов
Dog.prototype.bark = function() {
  console.log(`${this.name} лает`);
};

const dog = new Dog("Бобик", "Овчарка");
dog.speak(); // "Бобик издает звук"
dog.bark(); // "Бобик лает"

console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

## Современный синтаксис: Классы

### Базовые классы

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  greet() {
    return `Привет, меня зовут ${this.name}`;
  }
  
  static getSpecies() {
    return "Homo sapiens";
  }
}

const person = new Person("Иван", 30);
console.log(person.greet()); // "Привет, меня зовут Иван"
console.log(Person.getSpecies()); // "Homo sapiens"
```

### Наследование классов

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} издает звук`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Вызов родительского конструктора
    this.breed = breed;
  }
  
  speak() {
    super.speak(); // Вызов родительского метода
    console.log(`${this.name} лает`);
  }
  
  get info() {
    return `${this.name} - ${this.breed}`;
  }
  
  set info(value) {
    const [name, breed] = value.split(' - ');
    this.name = name;
    this.breed = breed;
  }
}

const dog = new Dog("Бобик", "Овчарка");
dog.speak(); 
// "Бобик издает звук"
// "Бобик лает"

console.log(dog.info); // "Бобик - Овчарка"
```

## Продвинутые концепции прототипов

### Object.prototype

```javascript
// Object.prototype - корень цепочки прототипов
const obj = {};
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true

// Методы Object.prototype
console.log(obj.toString()); // "[object Object]"
console.log(obj.valueOf() === obj); // true
console.log(obj.hasOwnProperty('toString')); // false

// toString с переопределением
const user = {
  name: "Иван",
  toString() {
    return this.name;
  }
};

console.log(user.toString()); // "Иван"
```

### Прототипы встроенных объектов

```javascript
// Расширение встроенных прототипов (не рекомендуется в production)
Array.prototype.sum = function() {
  return this.reduce((acc, val) => acc + val, 0);
};

console.log([1, 2, 3, 4, 5].sum()); // 15

// Лучше использовать утилиты
const ArrayUtils = {
  sum(arr) {
    return arr.reduce((acc, val) => acc + val, 0);
  }
};

console.log(ArrayUtils.sum([1, 2, 3, 4, 5])); // 15
```

## Практические примеры

### Создание системы наследования

```javascript
// Базовый класс для всех сущностей
class Entity {
  constructor(id) {
    this.id = id || Math.random().toString(36).substr(2, 9);
    this.createdAt = new Date();
  }
  
  getInfo() {
    return `ID: ${this.id}, Создан: ${this.createdAt}`;
  }
}

// Класс пользователя
class User extends Entity {
  constructor(id, name, email) {
    super(id);
    this.name = name;
    this.email = email;
    this.isActive = true;
  }
  
  getInfo() {
    return `${super.getInfo()}, Имя: ${this.name}, Email: ${this.email}`;
  }
  
  deactivate() {
    this.isActive = false;
    return this;
  }
}

// Класс продукта
class Product extends Entity {
  constructor(id, name, price) {
    super(id);
    this.name = name;
    this.price = price;
    this.inStock = true;
  }
  
  getInfo() {
    return `${super.getInfo()}, Название: ${this.name}, Цена: ${this.price}`;
  }
  
  applyDiscount(percent) {
    this.price = this.price * (1 - percent / 100);
    return this;
  }
}

const user = new User("user1", "Иван", "ivan@example.com");
const product = new Product("prod1", "Ноутбук", 50000);

console.log(user.getInfo());
console.log(product.getInfo());
console.log(product.applyDiscount(10).getInfo()); // Цена со скидкой 10%
```

### Миксины с использованием прототипов

```javascript
// Миксин для сериализации
const Serializable = {
  serialize() {
    return JSON.stringify(this);
  },
  
  deserialize(json) {
    const data = JSON.parse(json);
    Object.assign(this, data);
    return this;
  }
};

// Миксин для валидации
const Validatable = {
  validate() {
    const errors = [];
    
    if (!this.name || this.name.trim().length === 0) {
      errors.push("Имя обязательно");
    }
    
    if (this.age < 0) {
      errors.push("Возраст не может быть отрицательным");
    }
    
    return {
      isValid: errors.length === 0,
      errors: errors
    };
  }
};

// Класс, использующий миксины
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

// Применение миксинов
Object.assign(Person.prototype, Serializable, Validatable);

const person = new Person("Иван", 30);
console.log(person.serialize()); // Сериализованный объект

const person2 = new Person("", -5);
console.log(person2.validate()); 
// { isValid: false, errors: ["Имя обязательно", "Возраст не может быть отрицательным"] }
```

### Фабрика объектов с прототипами

```javascript
// Фабрика для создания различных типов объектов
function createObjectFactory() {
  const prototypes = new Map();
  
  return {
    // Регистрация прототипа
    register(type, prototype) {
      prototypes.set(type, prototype);
      return this;
    },
    
    // Создание объекта
    create(type, properties = {}) {
      const prototype = prototypes.get(type);
      if (!prototype) {
        throw new Error(`Неизвестный тип: ${type}`);
      }
      
      const obj = Object.create(prototype);
      Object.assign(obj, properties);
      return obj;
    }
  };
}

// Использование фабрики
const factory = createObjectFactory();

// Регистрация прототипов
factory.register('user', {
  greet() {
    return `Привет, я ${this.name}`;
  },
  
  getRole() {
    return this.role || 'user';
  }
});

factory.register('product', {
  getInfo() {
    return `${this.name} - ${this.price} руб.`;
  },
  
  applyDiscount(percent) {
    this.price = this.price * (1 - percent / 100);
  }
});

// Создание объектов
const user = factory.create('user', { name: "Иван", role: "admin" });
const product = factory.create('product', { name: "Ноутбук", price: 50000 });

console.log(user.greet()); // "Привет, я Иван"
console.log(user.getRole()); // "admin"
console.log(product.getInfo()); // "Ноутбук - 50000 руб."
```

## Различия между прототипами и классами

### Прототипы

```javascript
// Функция-конструктор
function Car(brand, model) {
  this.brand = brand;
  this.model = model;
}

Car.prototype.getInfo = function() {
  return `${this.brand} ${this.model}`;
};

Car.prototype.year = 2020; // Общее свойство для всех экземпляров

const car1 = new Car("Toyota", "Camry");
const car2 = new Car("Honda", "Civic");

console.log(car1.getInfo()); // "Toyota Camry"
console.log(car2.year); // 2020

// Изменение прототипа влияет на все экземпляры
Car.prototype.year = 2021;
console.log(car1.year); // 2021
console.log(car2.year); // 2021
```

### Классы

```javascript
class Car {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
    this.year = 2020; // Собственное свойство каждого экземпляра
  }
  
  getInfo() {
    return `${this.brand} ${this.model}`;
  }
}

const car1 = new Car("Toyota", "Camry");
const car2 = new Car("Honda", "Civic");

console.log(car1.getInfo()); // "Toyota Camry"
console.log(car1.year); // 2020

// Изменение свойства одного экземпляра не влияет на другие
car1.year = 2021;
console.log(car1.year); // 2021
console.log(car2.year); // 2020
```

## Лучшие практики

### 1. Используйте классы для нового кода

```javascript
// Предпочтительный способ
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  
  getArea() {
    return this.width * this.height;
  }
}

// Вместо функций-конструкторов
function Rectangle(width, height) {
  this.width = width;
  this.height = height;
}

Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
```

### 2. Правильное наследование

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error("Невозможно создать экземпляр абстрактного класса");
    }
  }
  
  getArea() {
    throw new Error("Метод getArea должен быть реализован");
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }
  
  getArea() {
    return Math.PI * this.radius ** 2;
  }
}

// const shape = new Shape(); // Ошибка!
const circle = new Circle(5);
console.log(circle.getArea()); // 78.53981633974483
```

## Заключение

Прототипы и наследование являются фундаментальными концепциями JavaScript, которые лежат в основе объектно-ориентированного программирования в этом языке. Понимание этих механизмов критически важно для создания эффективного и поддерживаемого кода. Современный синтаксис классов упрощает работу с наследованием, но понимание прототипов помогает глубже понять, как работает JavaScript под капотом.

#javascript #programming #prototypes #inheritance #classes #oop #web-development