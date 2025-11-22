---
aliases: [Прототипы в JavaScript, Прототипы в JS, JavaScript прототипы]
tags: [javascript, прототипы, объекты, oop]
---

# Прототипы в JavaScript

Прототипы - это один из фундаментальных механизмов JavaScript, который позволяет объектам наследовать свойства и методы от других объектов. Понимание прототипов критически важно для глубокого понимания работы JavaScript и эффективной разработки.

## Что такое прототип?

В JavaScript каждый объект имеет внутреннее свойство `[[Prototype]]`, которое может быть равно `null` или ссылаться на другой объект - прототип. Этот прототип может также иметь свой прототип и так далее, образуя цепочку прототипов. При обращении к свойству объекта, если оно не найдено в самом объекте, JavaScript ищет его по цепочке прототипов.

> [!note] Важно
> В современном JavaScript `[[Prototype]]` можно получить/установить с помощью `Object.getPrototypeOf()` и `Object.setPrototypeOf()`, а также через свойство `__proto__` (не рекомендуется использовать в производстве).

## Создание объектов и их прототипы

Когда вы создаете объект в JavaScript, он автоматически получает прототип:

```javascript
// Объект, созданный с помощью литерала
const obj = {};
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true

// Массив
const arr = [];
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true

// Функция
function myFunc() {}
console.log(Object.getPrototypeOf(myFunc) === Function.prototype); // true
```

## Прототип функции-конструктора

Каждая функция в JavaScript имеет свойство `prototype`, которое используется при создании новых объектов с помощью оператора `new`. Это свойство становится прототипом создаваемых экземпляров.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Добавляем метод в прототип
Person.prototype.sayHello = function() {
  console.log(`Привет, я ${this.name}`);
};

Person.prototype.getAge = function() {
  return this.age;
};

// Создаем экземпляр
const person1 = new Person("Иван", 30);
const person2 = new Person("Мария", 25);

person1.sayHello(); // "Привет, я Иван"
person2.sayHello(); // "Привет, я Мария"

// Методы находятся в прототипе, а не в самих объектах
console.log(person1.hasOwnProperty('sayHello')); // false
console.log('sayHello' in person1); // true
```

## Object.create()

Метод `Object.create()` позволяет создать новый объект с указанным прототипом:

```javascript
const animal = {
  eats: true,
  walk() {
    console.log("Животное идет");
  }
};

// Создаем объект с прототипом animal
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (наследуется от animal)
console.log(rabbit.jumps); // true (собственное свойство)
rabbit.walk(); // "Животное идет"

// Проверяем прототип
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

## Цепочка прототипов

Когда свойство не находится в текущем объекте, JavaScript ищет его в прототипе, затем в прототипе прототипа и так далее:

```javascript
const animal = {
  eats: true,
  walk() {
    console.log("Животное идет");
  }
};

const rabbit = Object.create(animal);
rabbit.jumps = true;

const whiteRabbit = Object.create(rabbit);
whiteRabbit.color = "white";

// whiteRabbit наследует от rabbit, который наследует от animal
console.log(whiteRabbit.color); // "white" (собственное свойство)
console.log(whiteRabbit.jumps); // true (наследуется от rabbit)
console.log(whiteRabbit.eats); // true (наследуется от animal через rabbit)
whiteRabbit.walk(); // "Животное идет"
```

## Прототипы встроенных объектов

Все объекты в JavaScript наследуют от встроенных прототипов:

```javascript
// Прототипы встроенных объектов
console.log(Object.getPrototypeOf({}) === Object.prototype); // true
console.log(Object.getPrototypeOf([]) === Array.prototype); // true
console.log(Object.getPrototypeOf(function(){}) === Function.prototype); // true
console.log(Object.getPrototypeOf(/regex/) === RegExp.prototype); // true

// Добавление метода ко всем массивам
Array.prototype.last = function() {
  return this[this.length - 1];
};

const nums = [1, 2, 3, 4, 5];
console.log(nums.last()); // 5
```

## Работа с прототипами

### Проверка принадлежности свойства

```javascript
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

// hasOwnProperty проверяет только собственные свойства
console.log(rabbit.hasOwnProperty('jumps')); // true
console.log(rabbit.hasOwnProperty('eats')); // false

// in проверяет все свойства (собственные и унаследованные)
console.log('jumps' in rabbit); // true
console.log('eats' in rabbit); // true

// instanceof проверяет цепочку прототипов
console.log(rabbit instanceof Object); // true
console.log(rabbit instanceof Object.constructor); // false
```

### Итерация по свойствам

```javascript
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

// for...in перебирает все перечисляемые свойства (включая унаследованные)
for (let key in rabbit) {
  console.log(key); // "jumps", затем "eats"
}

// Чтобы перебрать только собственные свойства:
for (let key in rabbit) {
  if (rabbit.hasOwnProperty(key)) {
    console.log(key); // "jumps"
  }
}
```

## Современные альтернативы

С появлением ES6 и классов, прототипное наследование стало более абстрактным, но основа осталась той же:

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
    super(name);
    this.breed = breed;
  }

  bark() {
    console.log(`${this.name} лает`);
  }
}

const dog = new Dog("Бобик", "Дворняга");
dog.speak(); // "Бобик издает звук"
dog.bark(); // "Бобик лает"

// Проверка цепочки прототипов
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
console.log(dog instanceof Object); // true
```

## Практические рекомендации

- Используйте `Object.create()` вместо `__proto__` для установки прототипов
- Не изменяйте прототипы встроенных объектов в производственном коде
- Используйте `hasOwnProperty()` для проверки принадлежности свойства объекту
- Помните, что свойства прототипа разделяются между всеми экземплярами
- Избегайте чрезмерного усложнения цепочек прототипов

## Заключение

Прототипы - это мощный механизм JavaScript, который лежит в основе наследования и позволяет создавать гибкие и переиспользуемые структуры данных. Понимание прототипов критически важно для понимания JavaScript на продвинутом уровне.

## См. также

- [[Прототипное-наследование]]
- [[Прототипы-и-классы]]
- [[Прототипы-и-производительность]]
- [[Прототипы-в-современном-JavaScript]]
- [[Объекты]]
- [[Объектно-ориентированное-программирование]]