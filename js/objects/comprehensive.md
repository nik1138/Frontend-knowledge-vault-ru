---
aliases: ["Объекты в JavaScript", "JS объекты", "Объекты"]
tags: [javascript, objects, programming, js]
---

# Объекты в JavaScript: Полное руководство

Объекты в JavaScript являются основой языка и представляют собой коллекции пар "ключ-значение", где ключи - это строки (или символы), а значения могут быть любыми типами данных.

## Синтаксис литерала объекта

Самый распространенный способ создания объекта - это литеральный синтаксис:

```javascript
const person = {
  name: "Иван",
  age: 30,
  greet: function() {
    return `Привет, я ${this.name}`;
  }
};
```

Можно использовать сокращённый синтаксис методов:

```javascript
const person = {
  name: "Иван",
  age: 30,
  greet() {
    return `Привет, я ${this.name}`;
  }
};
```

## Конструктор объекта

Объекты также можно создавать с помощью конструктора:

```javascript
const person = new Object();
person.name = "Иван";
person.age = 30;
person.greet = function() {
  return `Привет, я ${this.name}`;
};
```

> [!tip] 
> Литеральный синтаксис предпочтительнее, так как он более краткий и читаемый.

## Свойства и методы объектов

Свойства объекта можно получать и устанавливать разными способами:

```javascript
const obj = { name: "Иван", age: 30 };

// Получение значения
console.log(obj.name);        // "Иван"
console.log(obj["name"]);     // "Иван"

// Установка значения
obj.name = "Петр";
obj["age"] = 25;
```

Методы - это свойства, содержащие функции:

```javascript
const calculator = {
  value: 0,
  add(x) {
    this.value += x;
    return this;
  },
  subtract(x) {
    this.value -= x;
    return this;
  }
};
```

## Дескрипторы свойств

Дескрипторы позволяют точно контролировать поведение свойств:

```javascript
const obj = {};
Object.defineProperty(obj, 'name', {
  value: 'Иван',
  writable: true,
  enumerable: true,
  configurable: true
});
```

Типы дескрипторов:
- `writable` - можно ли изменить значение
- `enumerable` - будет ли перечисляться в циклах
- `configurable` - можно ли изменить дескриптор или удалить свойство

## Геттеры и сеттеры

Геттеры и сеттеры позволяют контролировать доступ к свойствам:

```javascript
const person = {
  firstName: "Иван",
  lastName: "Иванов",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(' ');
  }
};

console.log(person.fullName);  // "Иван Иванов"
person.fullName = "Петр Петров";
console.log(person.firstName); // "Петр"
```

## Методы объекта Object

### Object.assign()

Копирует все перечисляемые собственные свойства из одного или нескольких исходных объектов в целевой объект:

```javascript
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4 };

const result = Object.assign(target, source);
console.log(result); // { a: 1, b: 3, c: 4 }
```

### Object.keys(), Object.values(), Object.entries()

Позволяют получить ключи, значения или пары ключ-значение:

```javascript
const obj = { a: 1, b: 2, c: 3 };

console.log(Object.keys(obj));    // ['a', 'b', 'c']
console.log(Object.values(obj));  // [1, 2, 3]
console.log(Object.entries(obj)); // [['a', 1], ['b', 2], ['c', 3]]
```

### Object.hasOwnProperty()

Проверяет, содержит ли объект указанное свойство:

```javascript
const obj = { name: "Иван" };
console.log(obj.hasOwnProperty('name')); // true
console.log(obj.hasOwnProperty('toString')); // false
```

## Прототипы и прототипное наследование

Каждый объект имеет ссылку на прототип:

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  return `Привет, я ${this.name}`;
};

const ivan = new Person("Иван");
console.log(ivan.greet()); // "Привет, я Иван"
```

Прототипное наследование позволяет объектам наследовать свойства от других объектов.

## ES6 классы vs прототипное наследование

ES6 классы - это синтаксический сахар над прототипным наследованием:

```javascript
// Прототипный подход
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} издает звук`;
};

// ES6 классы
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} издает звук`;
  }
}
```

## Ключевое слово this в объектах

Контекст `this` зависит от способа вызова функции:

```javascript
const obj = {
  name: "Объект",
  getName() {
    return this.name;
  }
};

console.log(obj.getName()); // "Объект"

const getName = obj.getName;
console.log(getName()); // undefined (this !== obj)
```

## Деструктуризация объектов

Позволяет извлекать свойства в переменные:

```javascript
const person = { name: "Иван", age: 30, city: "Москва" };

const { name, age } = person;
console.log(name, age); // "Иван" 30

// С переименованием
const { name: personName, age: personAge } = person;

// Со значениями по умолчанию
const { country = "Россия" } = person;
```

## Сравнение объектов

Объекты сравниваются по ссылке, а не по значению:

```javascript
const obj1 = { name: "Иван" };
const obj2 = { name: "Иван" };
const obj3 = obj1;

console.log(obj1 === obj2); // false
console.log(obj1 === obj3); // true
```

Для глубокого сравнения нужно использовать специальные методы.

## Символы в объектах

Символы позволяют создавать уникальные идентификаторы:

```javascript
const id = Symbol('id');
const user = {
  name: "Иван",
  [id]: 123
};

console.log(user[id]); // 123
```

## Неизменяемость объектов

### Object.freeze()

Предотвращает изменение свойств объекта:

```javascript
const obj = { name: "Иван" };
Object.freeze(obj);

obj.name = "Петр"; // Не сработает в строгом режиме
console.log(obj.name); // "Иван"
```

### Object.seal()

Позволяет изменять существующие свойства, но запрещает добавлять/удалять:

```javascript
const obj = { name: "Иван" };
Object.seal(obj);

obj.name = "Петр"; // Работает
obj.age = 30;      // Не сработает
```

## Proxy для объектов

Proxy позволяет перехватывать и переопределять операции с объектами:

```javascript
const handler = {
  get(target, prop) {
    return prop in target ? target[prop] : `Свойство ${prop} не существует`;
  }
};

const obj = { name: "Иван" };
const proxy = new Proxy(obj, handler);

console.log(proxy.name);    // "Иван"
console.log(proxy.age);     // "Свойство age не существует"
```

## Связи с другими файлами

- [[js/functions]] - Функции в JavaScript, включая конструкторы объектов
- [[js/prototypes]] - Подробное руководство по прототипам
- [[js/classes]] - Классы ES6 и их связь с прототипами
- [[js/this-context]] - Глубокое погружение в контекст this
- [[js/destructuring]] - Деструктуризация в JavaScript
- [[js/symbols]] - Символы и их применение
- [[js/closures]] - Замыкания и объекты
- [[js/modules]] - Модули и экспорт объектов

## Заключение

Объекты в JavaScript - это мощная концепция, обеспечивающая гибкость и функциональность языка. Понимание всех аспектов работы с объектами необходимо для эффективной разработки на JavaScript.

#javascript #objects #programming #js