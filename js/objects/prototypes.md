# Прототипное наследование в JavaScript

## Обзор

Прототипное наследование - это механизм в JavaScript, который позволяет объектам наследовать свойства и методы от других объектов. Каждый объект в JavaScript имеет внутреннюю ссылку на прототип (объект), от которого он наследует свойства и методы.

## Прототипы и цепочка прототипов

```javascript
// Создание объекта
const person = {
    name: "Иван",
    greet() {
        return `Привет, я ${this.name}`;
    }
};

// Проверка прототипа
console.log(person.__proto__ === Object.prototype); // true
console.log(Object.getPrototypeOf(person) === Object.prototype); // true

// Пример цепочки прототипов
const student = Object.create(person);
student.major = "Информатика";

console.log(student.name); // "Иван" (наследуется от person)
console.log(student.greet()); // "Привет, я Иван" (наследуется от person)
console.log(student.major); // "Информатика" (собственное свойство)
```

## Конструкторы и прототипы

```javascript
// Функция-конструктор
function Animal(name) {
    this.name = name;
}

// Добавление метода в прототип
Animal.prototype.speak = function() {
    return `${this.name} издает звук`;
};

// Создание экземпляров
const dog = new Animal("Рекс");
const cat = new Animal("Мурка");

console.log(dog.speak()); // "Рекс издает звук"
console.log(cat.speak()); // "Мурка издает звук"

// Проверка принадлежности
console.log(dog instanceof Animal); // true
console.log(cat instanceof Animal); // true
```

## Прототипное наследование с использованием классов (ES6+)

```javascript
class Vehicle {
    constructor(brand, model) {
        this.brand = brand;
        this.model = model;
    }
    
    start() {
        return `${this.brand} ${this.model} запускается`;
    }
}

class Car extends Vehicle {
    constructor(brand, model, doors) {
        super(brand, model); // Вызов конструктора родительского класса
        this.doors = doors;
    }
    
    honk() {
        return "Бип-бип!";
    }
    
    // Переопределение метода родителя
    start() {
        return `${super.start()} и готов к поездке`; // Вызов метода родителя
    }
}

const myCar = new Car("Toyota", "Camry", 4);
console.log(myCar.start()); // "Toyota Camry запускается и готов к поездке"
console.log(myCar.honk()); // "Бип-бип!"
console.log(myCar instanceof Car); // true
console.log(myCar instanceof Vehicle); // true
```

## Работа с прототипами

### Object.create()

```javascript
// Создание объекта с указанным прототипом
const animal = {
    eats: true,
    walk() {
        return "Ходит";
    }
};

const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (наследуется)
console.log(rabbit.walk()); // "Ходит" (наследуется)
console.log(rabbit.jumps); // true (собственное свойство)

// Получение прототипа
console.log(Object.getPrototypeOf(rabbit) === animal); // true

// Установка прототипа
const horse = {};
Object.setPrototypeOf(horse, animal);
console.log(horse.eats); // true
```

### Прототипы встроенных объектов

```javascript
// Все функции наследуются от Function.prototype
function myFunction() {
    return "Привет";
}

console.log(myFunction.__proto__ === Function.prototype); // true
console.log(Function.prototype.__proto__ === Object.prototype); // true

// Все массивы наследуются от Array.prototype
const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true
console.log(Array.prototype.__proto__ === Object.prototype); // true

// Методы массивов наследуются от Array.prototype
console.log(arr.map); // [Function: map] (наследуется от Array.prototype)
```

## Продвинутые примеры прототипного наследования

### Создание иерархии классов

```javascript
class Shape {
    constructor(color) {
        this.color = color;
    }
    
    move(x, y) {
        this.x = x;
        this.y = y;
    }
    
    getColor() {
        return this.color;
    }
}

class Rectangle extends Shape {
    constructor(color, width, height) {
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
}

class Square extends Rectangle {
    constructor(color, size) {
        super(color, size, size); // Квадрат - это прямоугольник с равными сторонами
    }
    
    // Квадрат наследует все методы Rectangle и Shape
}

const mySquare = new Square("красный", 5);
console.log(mySquare.getArea()); // 25
console.log(mySquare.getPerimeter()); // 20
console.log(mySquare.getColor()); // "красный"
```

### Манипуляции с прототипом

```javascript
// Добавление метода в прототип существующего класса
String.prototype.reverse = function() {
    return this.split('').reverse().join('');
};

console.log("привет".reverse()); // "тевирп"

// Осторожно: изменение встроенных прототипов может повлиять на всю программу

// Создание "защищенного" метода
Object.defineProperty(Array.prototype, 'first', {
    value: function() {
        return this[0];
    },
    enumerable: false, // Не будет отображаться в цикле for...in
    writable: true,
    configurable: true
});

const nums = [10, 20, 30];
console.log(nums.first()); // 10
```

## Проверка прототипов и свойств

```javascript
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

// hasOwnProperty() проверяет только собственные свойства
console.log(rabbit.hasOwnProperty('jumps')); // true
console.log(rabbit.hasOwnProperty('eats')); // false

// Проверка цепочки прототипов
console.log('eats' in rabbit); // true (наследуется)
console.log('jumps' in rabbit); // true (собственное)

// Object.keys() возвращает только собственные перечисляемые свойства
console.log(Object.keys(rabbit)); // ['jumps']

// Object.getOwnPropertyNames() возвращает все собственные свойства
console.log(Object.getOwnPropertyNames(rabbit)); // ['jumps']
```

## Современные альтернативы прототипному наследованию

```javascript
// Использование Object.create() для создания объектов без наследования от Object.prototype
const pureObject = Object.create(null);
// pureObject не наследуется от Object.prototype, поэтому у него нет методов вроде hasOwnProperty

// Использование ES6 классов (предпочтительный современный подход)
class ModernAnimal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} издает звук`;
    }
}

class ModernDog extends ModernAnimal {
    constructor(name, breed) {
        super(name);
        this.breed = breed;
    }
    
    bark() {
        return "Гав!";
    }
}
```