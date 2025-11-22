---
aliases: [JS-Objects, JavaScript-Objects, Объекты-JS]
tags: [javascript, frontend, programming, objects]
---

# Объекты в JavaScript

Объекты являются одним из фундаментальных типов данных в JavaScript. Они позволяют хранить коллекции различных значений и представляют собой неупорядоченные коллекции свойств, состоящих из пар "ключ-значение".

## Создание объектов

### Литерал объекта

```javascript
const person = {
  name: 'Иван',
  age: 30,
  city: 'Москва',
  'is-developer': true
};
```

### Конструктор объекта

```javascript
const person = new Object();
person.name = 'Иван';
person.age = 30;
person.city = 'Москва';
```

### С помощью функции-конструктора

```javascript
function Person(name, age, city) {
  this.name = name;
  this.age = age;
  this.city = city;
}

const person = new Person('Иван', 30, 'Москва');
```

### С помощью Object.create()

```javascript
const personPrototype = {
  greet() {
    return `Привет, я ${this.name}`;
  }
};

const person = Object.create(personPrototype);
person.name = 'Иван';
person.age = 30;
```

## Доступ к свойствам объекта

```javascript
const person = {
  name: 'Иван',
  age: 30,
  'is-developer': true
};

// Точечная нотация
console.log(person.name); // 'Иван'

// Квадратные скобки
console.log(person['age']); // 30
console.log(person['is-developer']); // true

// Динамический доступ
const property = 'name';
console.log(person[property]); // 'Иван'
```

## Методы объекта

```javascript
const calculator = {
  value: 0,
  
  add(num) {
    this.value += num;
    return this; // для цепочки вызовов
  },
  
  subtract(num) {
    this.value -= num;
    return this;
  },
  
  multiply(num) {
    this.value *= num;
    return this;
  },
  
  getResult() {
    return this.value;
  }
};

calculator.add(5).multiply(2).subtract(3);
console.log(calculator.getResult()); // 7
```

## Деструктуризация объектов

```javascript
const person = {
  name: 'Иван',
  age: 30,
  city: 'Москва',
  job: 'разработчик'
};

// Базовая деструктуризация
const { name, age } = person;
console.log(name, age); // 'Иван', 30

// Переименование переменных
const { name: personName, age: personAge } = person;
console.log(personName, personAge); // 'Иван', 30

// Значения по умолчанию
const { name: n = 'Аноним', country = 'Россия' } = person;
console.log(n, country); // 'Иван', 'Россия'

// Вложенные объекты
const employee = {
  name: 'Иван',
  details: {
    position: 'разработчик',
    experience: 5
  }
};

const { details: { position, experience } } = employee;
console.log(position, experience); // 'разработчик', 5
```

## Методы объекта Object

### Object.keys(), Object.values(), Object.entries()

```javascript
const person = {
  name: 'Иван',
  age: 30,
  city: 'Москва'
};

console.log(Object.keys(person)); // ['name', 'age', 'city']
console.log(Object.values(person)); // ['Иван', 30, 'Москва']
console.log(Object.entries(person)); // [['name', 'Иван'], ['age', 30], ['city', 'Москва']]
```

### Object.assign()

```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target); // { a: 1, b: 4, c: 5 }
console.log(returnedTarget === target); // true
```

### Object.freeze() и Object.seal()

```javascript
// Object.freeze() - полная защита от изменений
const frozenObj = Object.freeze({
  name: 'Иван',
  age: 30
});

frozenObj.name = 'Петр'; // игнорируется в strict mode будет ошибка
console.log(frozenObj.name); // 'Иван'

// Object.seal() - можно изменять существующие свойства, но нельзя добавлять/удалять
const sealedObj = Object.seal({
  name: 'Иван',
  age: 30
});

sealedObj.name = 'Петр'; // разрешено
sealedObj.city = 'Москва'; // игнорируется в strict mode будет ошибка
delete sealedObj.age; // игнорируется в strict mode будет ошибка
```

## Прототипы и наследование

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} издает звук`);
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(`${this.name} лает`);
};

const dog = new Dog('Рекс', 'Лабрадор');
dog.speak(); // 'Рекс издает звук'
dog.bark(); // 'Рекс лает'
```

## Современные возможности ES6+

### Вычисляемые имена свойств

```javascript
const name = 'Иван';
const age = 30;
const key = 'job';

const person = {
  name, // сокращение от name: name
  age,  // сокращение от age: age
  [key]: 'разработчик', // вычисляемое имя свойства
  [`${name}_info`]: `Имя: ${name}, Возраст: ${age}`
};

console.log(person);
// {
//   name: 'Иван',
//   age: 30,
//   job: 'разработчик',
//   'Иван_info': 'Имя: Иван, Возраст: 30'
// }
```

### Spread-оператор с объектами

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// Объединение объектов
const combined = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Копирование объекта (поверхностное)
const copy = { ...obj1 };

// Переопределение свойств
const modified = { ...obj1, b: 20, e: 5 };
// { a: 1, b: 20, e: 5 }
```

## Практические рекомендации

### Использование объектов для конфигурации

```javascript
function createUser(config) {
  const defaultConfig = {
    isActive: true,
    role: 'user',
    permissions: []
  };
  
  return {
    ...defaultConfig,
    ...config
  };
}

const user = createUser({ name: 'Иван', role: 'admin' });
```

### Паттерн модуль

```javascript
const Calculator = (function() {
  let history = [];
  
  return {
    add(a, b) {
      const result = a + b;
      history.push(`${a} + ${b} = ${result}`);
      return result;
    },
    
    getHistory() {
      return [...history];
    },
    
    clearHistory() {
      history = [];
    }
  };
})();

Calculator.add(2, 3); // 5
console.log(Calculator.getHistory()); // ['2 + 3 = 5']
```

## Заключение

Объекты в JavaScript являются мощным инструментом для структурирования данных и реализации различных паттернов программирования. Понимание их работы, особенностей прототипного наследования и современных возможностей языка критически важно для эффективной разработки.

См. также:
- [[Объекты-в-TypeScript]]
- [[Объекты-в-React-компонентах]]
- [[Объекты-и-их-свойства]]
- [[Прототипы-в-JavaScript]]