# Создание объектов в JavaScript

## Обзор

Объекты в JavaScript - это коллекции пар "ключ-значение", которые позволяют хранить и организовывать данные. Существует несколько способов создания объектов в JavaScript.

## Способы создания объектов

### 1. Литерал объекта

Самый распространенный и простой способ создания объекта:

```javascript
const person = {
    name: "Иван",
    age: 30,
    city: "Москва",
    isStudent: false,
    
    // Метод объекта
    greet: function() {
        return `Привет, меня зовут ${this.name}`;
    },
    
    // Сокращенный синтаксис метода (ES6+)
    introduce() {
        return `Меня зовут ${this.name}, мне ${this.age} лет`;
    }
};

console.log(person.name); // "Иван"
console.log(person.greet()); // "Привет, меня зовут Иван"
```

### 2. Конструктор Object()

```javascript
const car = new Object();
car.brand = "Toyota";
car.model = "Camry";
car.year = 2020;

// Или с передачей начальных свойств
const bike = new Object({
    brand: "Giant",
    type: "Mountain",
    gears: 21
});
```

### 3. Функция-конструктор

```javascript
function Person(name, age, city) {
    this.name = name;
    this.age = age;
    this.city = city;
    
    this.greet = function() {
        return `Привет, меня зовут ${this.name}`;
    };
}

const person1 = new Person("Мария", 25, "Санкт-Петербург");
const person2 = new Person("Алексей", 35, "Новосибирск");

console.log(person1.greet()); // "Привет, меня зовут Мария"
```

### 4. Классы (ES6+)

```javascript
class Animal {
    constructor(species, name, age) {
        this.species = species;
        this.name = name;
        this.age = age;
    }
    
    getInfo() {
        return `${this.name} - это ${this.species}, возраст: ${this.age}`;
    }
    
    static getKingdom() {
        return "Животные";
    }
}

const dog = new Animal("собака", "Рекс", 3);
console.log(dog.getInfo()); // "Рекс - это собака, возраст: 3"
console.log(Animal.getKingdom()); // "Животные"
```

### 5. Object.create()

```javascript
const animalPrototype = {
    speak() {
        return `${this.name} издает звук`;
    },
    
    getInfo() {
        return `Это ${this.species} по имени ${this.name}`;
    }
};

const cat = Object.create(animalPrototype);
cat.species = "кошка";
cat.name = "Мурка";

console.log(cat.speak()); // "Мурка издает звук"
console.log(cat.getInfo()); // "Это кошка по имени Мурка"
```

## Доступ к свойствам объекта

```javascript
const book = {
    title: "Война и мир",
    author: "Лев Толстой",
    year: 1869,
    "pages-count": 1225 // Свойство с дефисом в названии
};

// Точка (.)
console.log(book.title); // "Война и мир"

// Квадратные скобки
console.log(book["author"]); // "Лев Толстой"
console.log(book["pages-count"]); // 1225

// Динамический доступ
const property = "year";
console.log(book[property]); // 1869
```

## Вычисляемые имена свойств (ES6+)

```javascript
const nameKey = "full_name";
const ageKey = "user_age";

const user = {
    [nameKey]: "Анна Иванова",
    [ageKey]: 28,
    [`${nameKey}_length`]: "Анна Иванова".length
};

console.log(user.full_name); // "Анна Иванова"
console.log(user.user_age); // 28
console.log(user["full_name_length"]); // 12
```

## Деструктуризация объекта

```javascript
const student = {
    firstName: "Елена",
    lastName: "Петрова",
    grade: 95,
    subject: "Математика"
};

// Деструктуризация
const { firstName, lastName, grade } = student;
console.log(`${firstName} ${lastName}: ${grade}`); // "Елена Петрова: 95"

// Переименование переменных
const { firstName: name, lastName: surname } = student;
console.log(`${name} ${surname}`); // "Елена Петрова"

// Значения по умолчанию
const { subject, level = "beginner" } = student;
console.log(level); // "Математика" (не "beginner", потому что свойство существует)
```

## Специальные методы объектов

```javascript
const obj = {
    name: "Тест",
    value: 42
};

// Object.keys() - получить массив ключей
console.log(Object.keys(obj)); // ["name", "value"]

// Object.values() - получить массив значений
console.log(Object.values(obj)); // ["Тест", 42]

// Object.entries() - получить массив пар [ключ, значение]
console.log(Object.entries(obj)); // [["name", "Тест"], ["value", 42]]

// Object.assign() - объединение объектов
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = Object.assign({}, obj1, obj2);
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Spread оператор (ES6+) для объединения объектов
const spreadMerged = { ...obj1, ...obj2 };
console.log(spreadMerged); // { a: 1, b: 2, c: 3, d: 4 }
```