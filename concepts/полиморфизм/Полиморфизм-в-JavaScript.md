---
aliases: ["Полиморфизм в JavaScript", "JS Полиморфизм"]
tags: ["#javascript", "#polymorphism", "#frontend", "#oop"]
---

# Полиморфизм-в-JavaScript

## Введение

Полиморфизм в JavaScript — это концепция объектно-ориентированного программирования, позволяющая объектам разных типов использовать один и тот же интерфейс. Слово "полиморфизм" происходит от греческих слов "poly" (много) и "morph" (форма), что буквально означает "много форм".

В JavaScript полиморфизм достигается через наследование, прототипы, интерфейсы и различные реализации методов. Он позволяет писать более гибкий и переиспользуемый код, который может работать с объектами различных типов, при этом не требуя знания их конкретного типа во время компиляции.

## Типы полиморфизма в JavaScript

### 1. Подтиповый полиморфизм (Subtype Polymorphism)

Это наиболее распространенная форма полиморфизма в JavaScript, где объекты подклассов могут использоваться везде, где ожидаются объекты суперкласса.

```javascript
class Animal {
  speak() {
    console.log("Животное издает звук");
  }
}

class Dog extends Animal {
  speak() {
    console.log("Собака лает");
  }
}

class Cat extends Animal {
  speak() {
    console.log("Кошка мяукает");
  }
}

// Полиморфное использование
function makeAnimalSpeak(animal) {
  animal.speak(); // Вызов метода, который реализован по-разному
}

const dog = new Dog();
const cat = new Cat();

makeAnimalSpeak(dog); // "Собака лает"
makeAnimalSpeak(cat); // "Кошка мяукает"
```

### 2. Параметрический полиморфизм

JavaScript не имеет строгой системы типов как TypeScript или Java, но параметрический полиморфизм можно наблюдать в функциях высшего порядка и универсальных функциях:

```javascript
function identity(arg) {
  return arg;
}

// Функция работает с любыми типами данных
const num = identity(42);
const str = identity("Hello");
const obj = identity({ name: "John" });
```

### 3. Ad-hoc полиморфизм

JavaScript реализует ad-hoc полиморфизм через перегрузку операторов и функций, хотя в чистом виде это ограничено. Однако мы можем наблюдать его в работе с встроенными методами:

```javascript
// Один и тот же оператор работает по-разному с разными типами
console.log(2 + 3);        // 5 (числа)
console.log("2" + "3");    // "23" (строки)
console.log([1,2] + [3,4]); // "1,23,4" (массивы)

// Использование одного метода с разными типами
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(x => x * 2); // [2, 4, 6, 8, 10]

const strings = ["hello", "world"];
const upper = strings.map(s => s.toUpperCase()); // ["HELLO", "WORLD"]
```

## Прототипный полиморфизм

JavaScript использует прототипную модель наследования, что делает полиморфизм особенно гибким:

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} издает звук`);
};

function Dog(name) {
  Animal.call(this, name);
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function() {
  console.log(`${this.name} лает`);
};

const myDog = new Dog("Бобик");
myDog.speak(); // "Бобик лает"
```

## Полиморфизм с использованием интерфейсов (виртуальных)

JavaScript не имеет встроенной поддержки интерфейсов, но мы можем имитировать их поведение:

```javascript
// Виртуальный "интерфейс" для фигуры
class Shape {
  area() {
    throw new Error("Метод area() должен быть реализован");
  }
  
  perimeter() {
    throw new Error("Метод perimeter() должен быть реализован");
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }
  
  area() {
    return this.width * this.height;
  }
  
  perimeter() {
    return 2 * (this.width + this.height);
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }
  
  area() {
    return Math.PI * this.radius ** 2;
  }
  
  perimeter() {
    return 2 * Math.PI * this.radius;
  }
}

// Полиморфное использование
function calculateShapeProperties(shape) {
  console.log(`Площадь: ${shape.area()}`);
  console.log(`Периметр: ${shape.perimeter()}`);
}

const rect = new Rectangle(5, 3);
const circle = new Circle(4);

calculateShapeProperties(rect);   // Площадь: 15, Периметр: 16
calculateShapeProperties(circle); // Площадь: 50.265..., Периметр: 25.132...
```

## Полиморфизм в функциональном программировании

JavaScript поддерживает функциональное программирование, где полиморфизм реализуется через функции высшего порядка:

```javascript
// Функция, которая принимает другую функцию
function processArray(arr, processor) {
  return arr.map(processor);
}

// Разные функции-обработчики
const double = x => x * 2;
const square = x => x * x;
const negate = x => -x;

const numbers = [1, 2, 3, 4, 5];

console.log(processArray(numbers, double)); // [2, 4, 6, 8, 10]
console.log(processArray(numbers, square)); // [1, 4, 9, 16, 25]
console.log(processArray(numbers, negate)); // [-1, -2, -3, -4, -5]
```

## Практические примеры использования полиморфизма

### 1. Работа с различными типами элементов UI

```javascript
class UIElement {
  render() {
    throw new Error("Метод render() должен быть реализован");
  }
}

class Button extends UIElement {
  constructor(text) {
    super();
    this.text = text;
  }
  
  render() {
    return `<button>${this.text}</button>`;
  }
}

class Input extends UIElement {
  constructor(placeholder) {
    super();
    this.placeholder = placeholder;
  }
  
  render() {
    return `<input placeholder="${this.placeholder}" />`;
  }
}

class Text extends UIElement {
  constructor(content) {
    super();
    this.content = content;
  }
  
  render() {
    return `<span>${this.content}</span>`;
  }
}

// Полиморфная функция рендеринга
function renderUIElements(elements) {
  return elements.map(element => element.render()).join('');
}

const elements = [
  new Button("Нажми меня"),
  new Input("Введите текст"),
  new Text("Простой текст")
];

console.log(renderUIElements(elements));
```

### 2. Обработка различных типов данных

```javascript
class DataProcessor {
  process(data) {
    throw new Error("Метод process() должен быть реализован");
  }
}

class JSONProcessor extends DataProcessor {
  process(data) {
    return JSON.parse(data);
  }
}

class CSVProcessor extends DataProcessor {
  process(data) {
    // Упрощенный парсер CSV
    const rows = data.split('\n');
    return rows.map(row => row.split(','));
  }
}

class XMLProcessor extends DataProcessor {
  process(data) {
    // В реальном приложении использовался бы XML парсер
    console.log("Парсинг XML данных:", data);
    return { xml: data };
  }
}

// Единая функция обработки
function handleData(data, processor) {
  return processor.process(data);
}

const jsonData = '{"name": "John", "age": 30}';
const csvData = "имя,возраст\nИван,25\nМария,30";

const jsonProcessor = new JSONProcessor();
const csvProcessor = new CSVProcessor();

console.log(handleData(jsonData, jsonProcessor)); // {name: "John", age: 30}
console.log(handleData(csvData, csvProcessor));   // [["имя", "возраст"], ["Иван", "25"], ["Мария", "30"]]
```

## Преимущества полиморфизма в JavaScript

1. **Повторное использование кода** - можно писать обобщенные функции, работающие с разными типами
2. **Гибкость** - легко добавлять новые типы, не изменяя существующий код
3. **Поддерживаемость** - изменения в конкретной реализации не влияют на код, использующий интерфейс
4. **Тестируемость** - легче создавать mock-объекты для тестирования

## Заключение

Полиморфизм в JavaScript позволяет создавать гибкие и расширяемые приложения. Благодаря динамической природе JavaScript, полиморфизм можно реализовать различными способами - от классического ООП с наследованием до функционального подхода с функциями высшего порядка.

Понимание и правильное применение полиморфизма помогает создавать более чистый, понятный и поддерживаемый код.

## См. также

- [[Наследование]]
- [[Инкапсуляция]]
- [[Объектно-ориентированное программирование]]
- [[Прототипы]]
- [[Функциональное программирование]]