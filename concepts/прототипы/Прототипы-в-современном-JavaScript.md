---
aliases: [Прототипы в современном JavaScript, Современные прототипы, Прототипы ES6+]
tags: [javascript, прототипы, es6, классы, современный-javascript]
---

# Прототипы в современном JavaScript

Современный JavaScript (ES6 и более поздние версии) продолжает использовать прототипную систему, но предоставляет более удобные и понятные способы работы с прототипами. Понимание прототипов остается важным даже при использовании современных возможностей языка.

## Классы как синтаксический сахар

С ES6 классы в JavaScript являются синтаксическим сахаром над прототипным наследованием. Важно понимать, что под капотом всё ещё работает прототипная система.

```javascript
// Класс ES6
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} издает звук`);
  }

  static type = "Животное";

  get info() {
    return `${this.name} - ${Animal.type}`;
  }
}

// Это все еще использует прототипы под капотом
console.log(Animal.prototype.speak); // функция
console.log(Object.getPrototypeOf(Animal.prototype)); // Object.prototype
console.log(Object.getPrototypeOf(Animal)); // Function.prototype

const dog = new Animal("Собака");
console.log(dog instanceof Animal); // true
console.log(dog.__proto__ === Animal.prototype); // true
```

## Наследование в классах

Современные классы упрощают наследование, но оно все равно основывается на прототипах:

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
    super(name); // вызывает Animal.call(this, name)
    this.breed = breed;
  }

  bark() {
    console.log(`${this.name} лает`);
  }

  speak() {
    super.speak(); // вызывает Animal.prototype.speak.call(this)
    this.bark();
  }
}

// Проверяем прототипы
console.log(Object.getPrototypeOf(Dog.prototype)); // Animal.prototype
console.log(Object.getPrototypeOf(Dog)); // Animal (для статических методов)

const myDog = new Dog("Бобик", "Дворняга");
myDog.speak(); // "Бобик издает звук", затем "Бобик лает"
```

## Приватные поля и методы

ES2022 добавил поддержку приватных полей и методов, которые также реализованы через прототипы, но с дополнительными механизмами защиты:

```javascript
class BankAccount {
  #balance = 0;
  #transactionHistory = [];

  constructor(initialBalance = 0) {
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      this.#recordTransaction(`Депозит: +${amount}`);
    }
  }

  getBalance() {
    return this.#balance;
  }

  #recordTransaction(description) {
    this.#transactionHistory.push({
      description,
      timestamp: new Date()
    });
  }

  // Приватный метод
  #validateAmount(amount) {
    return typeof amount === 'number' && amount > 0;
  }
}

const account = new BankAccount(100);
console.log(account.getBalance()); // 100
// console.log(account.#balance); // Ошибка - приватное поле
```

## Статические методы и поля

Статические методы и поля также используют прототипную систему, но находятся на конструкторе, а не на прототипе экземпляра:

```javascript
class MathUtils {
  static PI = 3.14159;

  static calculateCircleArea(radius) {
    return this.PI * radius * radius;
  }

  static {
    console.log("Статический блок инициализации");
    // Может использоваться для сложной инициализации статических полей
    this.MAX_SAFE_INTEGER_SQRT = Math.floor(Math.sqrt(Number.MAX_SAFE_INTEGER));
  }
}

// Статические свойства находятся на самом классе, а не на прототипе
console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.calculateCircleArea(5)); // 78.53975
console.log(MathUtils.prototype.calculateCircleArea); // undefined
console.log(MathUtils.calculateCircleArea); // функция

// Это связано с прототипной системой
console.log(MathUtils.hasOwnProperty('PI')); // true
console.log(MathUtils.hasOwnProperty('calculateCircleArea')); // true
console.log(MathUtils.prototype.hasOwnProperty('calculateCircleArea')); // false
```

## Прототипы и модули

Современные JavaScript-приложения часто используют модули, но прототипы продолжают работать тем же образом:

```javascript
// shapes.js
export class Shape {
  constructor(color) {
    this.color = color;
  }

  describe() {
    return `Фигура цвета ${this.color}`;
  }
}

export class Circle extends Shape {
  constructor(color, radius) {
    super(color);
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius ** 2;
  }
}

// main.js
import { Circle } from './shapes.js';

const circle = new Circle("красный", 5);
console.log(circle.describe()); // "Фигура цвета красный"
console.log(circle.area()); // 78.53981633974483

// Прототипная цепочка сохраняется
console.log(circle instanceof Circle); // true
console.log(circle instanceof Shape); // true
console.log(Object.getPrototypeOf(circle) === Circle.prototype); // true
```

## Прототипы и функции высшего порядка

Современные возможности JavaScript, такие как стрелочные функции, генераторы и async/await, также взаимодействуют с прототипной системой:

```javascript
class AsyncProcessor {
  constructor(name) {
    this.name = name;
    this.processedCount = 0;
  }

  // Метод может возвращать промис
  async process(data) {
    await this.delay(100); // имитация асинхронной обработки
    this.processedCount++;
    return `Обработано: ${data} в ${this.name}`;
  }

  // Генератор в прототипе
  *getHistory(count) {
    for (let i = 0; i < count; i++) {
      yield `Запись истории ${i + 1}`;
    }
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

const processor = new AsyncProcessor("Главный процессор");
processor.process("данные").then(result => console.log(result));

// Использование генератора
for (const record of processor.getHistory(3)) {
  console.log(record);
}
```

## Прототипы и деструктуризация

Современные возможности деструктуризации также работают с прототипами:

```javascript
class Settings {
  constructor(options = {}) {
    this.theme = options.theme || 'light';
    this.language = options.language || 'ru';
    this.notifications = options.notifications ?? true;
  }

  update({ theme, language, notifications } = {}) {
    if (theme !== undefined) this.theme = theme;
    if (language !== undefined) this.language = language;
    if (notifications !== undefined) this.notifications = notifications;
  }

  toJSON() {
    return {
      theme: this.theme,
      language: this.language,
      notifications: this.notifications
    };
  }
}

const settings = new Settings({ theme: 'dark', language: 'en' });
const { theme, language } = settings; // деструктуризация свойств
console.log(theme, language); // 'dark' 'en'

// Прототип сохраняется при деструктуризации
console.log(settings instanceof Settings); // true
```

## Прототипы и Proxy

ES6 Proxy позволяет перехватывать и переопределять операции с объектами, включая работу с прототипами:

```javascript
class ValidatedUser {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  setName(name) {
    if (typeof name !== 'string' || name.length < 2) {
      throw new Error('Имя должно быть строкой длиной не менее 2 символов');
    }
    this.name = name;
  }
}

// Создаем Proxy для дополнительной валидации
const createValidatedProxy = (target) => {
  return new Proxy(target, {
    set(obj, prop, value) {
      // Проверяем возраст при его изменении
      if (prop === 'age' && (typeof value !== 'number' || value < 0 || value > 150)) {
        throw new Error('Возраст должен быть числом от 0 до 150');
      }
      obj[prop] = value;
      return true;
    }
  });
};

const user = new ValidatedUser("Иван", 30);
const validatedUser = createValidatedProxy(user);

validatedUser.age = 35; // OK
// validatedUser.age = -5; // Ошибка: возраст недействителен

console.log(validatedUser.name, validatedUser.age); // "Иван", 35
```

## Прототипы и миксины

Современные паттерны программирования, такие как миксины, также строятся на прототипной системе:

```javascript
// Определяем миксины как функции
const Flyable = (superclass) => class extends superclass {
  fly() {
    console.log(`${this.name} летит`);
  }
  
  land() {
    console.log(`${this.name} приземляется`);
  }
};

const Swimmable = (superclass) => class extends superclass {
  swim() {
    console.log(`${this.name} плывет`);
  }
  
  dive() {
    console.log(`${this.name} ныряет`);
  }
};

// Используем миксины с классами
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Duck extends Swimmable(Flyable(Animal)) {
  quack() {
    console.log(`${this.name} крякает`);
  }
}

const duck = new Duck("Дональд");
duck.fly(); // "Дональд летит"
duck.swim(); // "Дональд плывет"
duck.quack(); // "Дональд крякает"
duck.dive(); // "Дональд ныряет"
duck.land(); // "Дональд приземляется"

// Прототипная цепочка все еще работает
console.log(duck instanceof Duck); // true
console.log(duck instanceof Animal); // true
```

## Современные альтернативы прототипам

Хотя прототипы остаются основой, современные подходы могут использовать другие паттерны:

```javascript
// Использование Object.create для создания объектов без конструкторов
const animalPrototype = {
  speak() {
    console.log(`${this.name} издает звук`);
  },
  
  introduce() {
    return `Я ${this.name}, ${this.type}`;
  }
};

const createAnimal = (name, type) => {
  return Object.create(animalPrototype, {
    name: { value: name, writable: true, enumerable: true },
    type: { value: type, writable: true, enumerable: true }
  });
};

const cat = createAnimal("Мурзик", "кот");
cat.speak(); // "Мурзик издает звук"
console.log(cat.introduce()); // "Я Мурзик, кот"

// Использование фабричных функций с замыканиями
const createSecureCounter = (initialValue = 0) => {
  let count = initialValue;
  
  return {
    increment() { count++; },
    decrement() { count--; },
    getCount() { return count; }
  };
};

const counter = createSecureCounter(10);
console.log(counter.getCount()); // 10
counter.increment();
console.log(counter.getCount()); // 11
// count недоступен напрямую извне
```

## Практические рекомендации для современного JavaScript

### 1. Используйте классы для четкой иерархии

```javascript
// Хорошо подходит для иерархии с четкими отношениями наследования
class Component {
  constructor(props) {
    this.props = props;
    this.state = {};
  }

  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.render();
  }

  render() {
    throw new Error('Render method must be implemented');
  }
}

class Button extends Component {
  render() {
    return `<button>${this.props.text}</button>`;
  }
}
```

### 2. Используйте прототипы для общих методов

```javascript
// Общие методы лучше определять в прототипе
class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }

  // Методы в прототипе - разделяются между всеми экземплярами
  async get(endpoint) {
    const response = await fetch(`${this.baseUrl}/${endpoint}`);
    return response.json();
  }

  async post(endpoint, data) {
    const response = await fetch(`${this.baseUrl}/${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
}
```

### 3. Комбинируйте подходы по необходимости

```javascript
// Комбинирование классов с функциональными возможностями
class DataProcessor {
  constructor(data) {
    this.data = [...data]; // создаем копию
  }

  // Используем методы массивов, которые работают через прототипы
  filter(predicate) {
    return new DataProcessor(this.data.filter(predicate));
  }

  map(transformer) {
    return new DataProcessor(this.data.map(transformer));
  }

  reduce(reducer, initialValue) {
    return this.data.reduce(reducer, initialValue);
  }

  // Современные методы с использованием генераторов
  *getChunks(size) {
    for (let i = 0; i < this.data.length; i += size) {
      yield this.data.slice(i, i + size);
    }
  }
}

// Использование
const processor = new DataProcessor([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
const result = processor
  .filter(x => x % 2 === 0)
  .map(x => x * 2)
  .reduce((acc, val) => acc + val, 0);

console.log(result); // 40

// Использование генератора
for (const chunk of processor.getChunks(3)) {
  console.log(chunk); // [1, 2, 3], [4, 5, 6], [7, 8, 9], [10]
}
```

## Заключение

Несмотря на появление современных возможностей JavaScript, прототипная система остается фундаментом языка. Классы, приватные поля, статические методы и другие современные возможности по-прежнему используют прототипы под капотом. Понимание прототипов необходимо для:

1. Глубокого понимания работы JavaScript
2. Отладки сложных проблем с наследованием
3. Оптимизации производительности
4. Работы с библиотеками и фреймворками
5. Правильного использования современных возможностей языка

Современный JavaScript предоставляет более удобные и понятные способы работы с прототипами, но знание основной системы остается важным для профессионального развития.

## См. также

- [[Прототипное-наследование]]
- [[Прототипы-в-JavaScript]]
- [[Прототипы-и-классы]]
- [[Прототипы-и-производительность]]
- [[Классы-в-JavaScript]] [[Классы-в-JavaScript]] (предполагаемый файл)
- [[Современные-паттерны-JavaScript]]