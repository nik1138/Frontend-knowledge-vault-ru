---
aliases: ["Паттерны проектирования", "Design Patterns", "JS Patterns"]
tags: [js, patterns, design, architecture]
---

# Паттерны проектирования в JavaScript

## Введение в паттерны проектирования

Паттерны проектирования - это проверенные решения типичных проблем проектирования программного обеспечения. В JavaScript они помогают создавать более гибкий, масштабируемый и поддерживаемый код.

## Порождающие паттерны

### Фабричный метод (Factory Method)

```javascript
// Простой фабричный метод
class AnimalFactory {
  static createAnimal(type, name) {
    switch (type) {
      case 'dog':
        return new Dog(name);
      case 'cat':
        return new Cat(name);
      case 'bird':
        return new Bird(name);
      default:
        throw new Error('Неизвестный тип животного');
    }
  }
}

class Dog {
  constructor(name) {
    this.name = name;
    this.type = 'dog';
  }
  
  makeSound() {
    return `${this.name} говорит: Гав!`;
  }
}

class Cat {
  constructor(name) {
    this.name = name;
    this.type = 'cat';
  }
  
  makeSound() {
    return `${this.name} говорит: Мяу!`;
  }
}

class Bird {
  constructor(name) {
    this.name = name;
    this.type = 'bird';
  }
  
  makeSound() {
    return `${this.name} поет: Чик-чирик!`;
  }
}

// Использование
const dog = AnimalFactory.createAnimal('dog', 'Бобик');
console.log(dog.makeSound()); // Бобик говорит: Гав!
```

### Абстрактная фабрика

```javascript
// Абстрактная фабрика для UI компонентов
class UIFactory {
  createButton() {
    throw new Error('Метод createButton должен быть реализован');
  }
  
  createInput() {
    throw new Error('Метод createInput должен быть реализован');
  }
}

class WindowsFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
  
  createInput() {
    return new WindowsInput();
  }
}

class MacOSFactory extends UIFactory {
  createButton() {
    return new MacOSButton();
  }
  
  createInput() {
    return new MacOSInput();
  }
}

class WindowsButton {
  render() {
    return '<button class="windows-btn">Windows Button</button>';
  }
}

class MacOSButton {
  render() {
    return '<button class="macos-btn">MacOS Button</button>';
  }
}

class WindowsInput {
  render() {
    return '<input class="windows-input" type="text">';
  }
}

class MacOSInput {
  render() {
    return '<input class="macos-input" type="text">';
  }
}

// Использование
function createUI(factory) {
  const button = factory.createButton();
  const input = factory.createInput();
  
  return {
    button: button.render(),
    input: input.render()
  };
}

const windowsUI = createUI(new WindowsFactory());
const macosUI = createUI(new MacOSFactory());
```

### Строитель (Builder)

```javascript
class UserBuilder {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.user = {
      firstName: '',
      lastName: '',
      email: '',
      age: 0,
      address: {},
      preferences: {}
    };
    return this;
  }
  
  setFirstName(firstName) {
    this.user.firstName = firstName;
    return this;
  }
  
  setLastName(lastName) {
    this.user.lastName = lastName;
    return this;
  }
  
  setEmail(email) {
    this.user.email = email;
    return this;
  }
  
  setAge(age) {
    this.user.age = age;
    return this;
  }
  
  setAddress(street, city, zipCode) {
    this.user.address = { street, city, zipCode };
    return this;
  }
  
  setPreferences(theme = 'light', language = 'ru') {
    this.user.preferences = { theme, language };
    return this;
  }
  
  build() {
    const user = { ...this.user };
    this.reset(); // Сбрасываем для следующего использования
    return user;
  }
}

// Использование
const user = new UserBuilder()
  .setFirstName('Иван')
  .setLastName('Иванов')
  .setEmail('ivan@example.com')
  .setAge(30)
  .setAddress('Ленина 123', 'Москва', '123456')
  .setPreferences('dark', 'en')
  .build();

console.log(user);
```

## Структурные паттерны

### Адаптер (Adapter)

```javascript
// Старый API
class OldPaymentSystem {
  makePayment(amount, currency) {
    return `Старый способ оплаты: ${amount} ${currency}`;
  }
}

// Новый API
class NewPaymentSystem {
  processPayment(paymentData) {
    return `Новый способ оплаты: ${paymentData.amount} ${paymentData.currency}`;
  }
}

// Адаптер
class PaymentAdapter {
  constructor(paymentSystem) {
    this.paymentSystem = paymentSystem;
  }
  
  makePayment(amount, currency) {
    if (this.paymentSystem instanceof OldPaymentSystem) {
      return this.paymentSystem.makePayment(amount, currency);
    } else if (this.paymentSystem instanceof NewPaymentSystem) {
      return this.paymentSystem.processPayment({ amount, currency });
    }
    throw new Error('Неизвестная платежная система');
  }
}

// Использование
const oldPayment = new OldPaymentSystem();
const newPayment = new NewPaymentSystem();

const oldAdapter = new PaymentAdapter(oldPayment);
const newAdapter = new PaymentAdapter(newPayment);

console.log(oldAdapter.makePayment(100, 'RUB')); // Старый способ оплаты: 100 RUB
console.log(newAdapter.makePayment(100, 'USD')); // Новый способ оплаты: 100 USD
```

### Декоратор (Decorator)

```javascript
// Базовый класс
class Coffee {
  cost() {
    return 100;
  }
  
  description() {
    return 'Кофе';
  }
}

// Декораторы
class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  
  cost() {
    return this.coffee.cost() + 30;
  }
  
  description() {
    return `${this.coffee.description()}, молоко`;
  }
}

class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  
  cost() {
    return this.coffee.cost() + 10;
  }
  
  description() {
    return `${this.coffee.description()}, сахар`;
  }
}

class ChocolateDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  
  cost() {
    return this.coffee.cost() + 50;
  }
  
  description() {
    return `${this.coffee.description()}, шоколад`;
  }
}

// Использование
let coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new ChocolateDecorator(coffee);

console.log(`${coffee.description()}: ${coffee.cost()} руб`); // Кофе, молоко, сахар, шоколад: 190 руб
```

### Фасад (Facade)

```javascript
// Сложная система
class CPU {
  freeze() {
    console.log('CPU заморожен');
  }
  
  jump(position) {
    console.log(`CPU прыгает на позицию ${position}`);
  }
  
  execute() {
    console.log('CPU выполняет команды');
  }
}

class Memory {
  load(position, data) {
    console.log(`Память загружает ${data} на позицию ${position}`);
    return data;
  }
}

class HardDrive {
  read(lba, size) {
    console.log(`Диск читает ${size} байт с позиции ${lba}`);
    return 'Boot Data';
  }
}

// Фасад
class ComputerFacade {
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
    this.bootSector = 0;
    this.sectorSize = 240;
  }
  
  start() {
    console.log('Компьютер запускается...');
    this.cpu.freeze();
    const bootData = this.hardDrive.read(this.bootSector, this.sectorSize);
    this.memory.load(0, bootData);
    this.cpu.jump(0);
    this.cpu.execute();
    console.log('Компьютер запущен!');
  }
  
  shutdown() {
    console.log('Компьютер выключается...');
    // Логика выключения
    console.log('Компьютер выключен!');
  }
}

// Использование
const computer = new ComputerFacade();
computer.start();
```

## Поведенческие паттерны

### Наблюдатель (Observer)

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }
  
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class NewsAgency extends Subject {
  constructor() {
    super();
    this.news = '';
  }
  
  setNews(news) {
    this.news = news;
    this.notify(news);
  }
}

class NewsChannel {
  constructor(name) {
    this.name = name;
  }
  
  update(news) {
    console.log(`${this.name} получает новости: ${news}`);
  }
}

// Использование
const newsAgency = new NewsAgency();
const cnn = new NewsChannel('CNN');
const bbc = new NewsChannel('BBC');

newsAgency.subscribe(cnn);
newsAgency.subscribe(bbc);

newsAgency.setNews('Новые открытия в JavaScript!'); // CNN получает новости: Новые открытия в JavaScript! и BBC получает новости: Новые открытия в JavaScript!
```

### Команда (Command)

```javascript
// Команда
class Command {
  execute() {
    throw new Error('Метод execute должен быть реализован');
  }
  
  undo() {
    throw new Error('Метод undo должен быть реализован');
  }
}

// Конкретные команды
class LightOnCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  
  execute() {
    this.light.turnOn();
  }
  
  undo() {
    this.light.turnOff();
  }
}

class LightOffCommand extends Command {
  constructor(light) {
    super();
    this.light = light;
  }
  
  execute() {
    this.light.turnOff();
  }
  
  undo() {
    this.light.turnOn();
  }
}

class Light {
  turnOn() {
    console.log('Свет включен');
  }
  
  turnOff() {
    console.log('Свет выключен');
  }
}

// Класс управления командами
class RemoteControl {
  constructor() {
    this.commands = [];
    this.current = 0;
  }
  
  executeCommand(command) {
    command.execute();
    this.commands.push(command);
    this.current++;
  }
  
  undo() {
    if (this.current > 0) {
      const command = this.commands[--this.current];
      command.undo();
    }
  }
}

// Использование
const light = new Light();
const remote = new RemoteControl();

remote.executeCommand(new LightOnCommand(light)); // Свет включен
remote.executeCommand(new LightOffCommand(light)); // Свет выключен
remote.undo(); // Свет включен (отмена последней команды)
```

### Стратегия (Strategy)

```javascript
// Интерфейс стратегии
class PaymentStrategy {
  pay(amount) {
    throw new Error('Метод pay должен быть реализован');
  }
}

// Конкретные стратегии
class CreditCardStrategy extends PaymentStrategy {
  constructor(cardNumber, cvv) {
    super();
    this.cardNumber = cardNumber;
    this.cvv = cvv;
  }
  
  pay(amount) {
    console.log(`Оплата ${amount} руб. с карты ${this.cardNumber.slice(0, 4)}...`);
  }
}

class PayPalStrategy extends PaymentStrategy {
  constructor(email, password) {
    super();
    this.email = email;
    this.password = password;
  }
  
  pay(amount) {
    console.log(`Оплата ${amount} руб. через PayPal на ${this.email}`);
  }
}

class BitcoinStrategy extends PaymentStrategy {
  constructor(walletAddress) {
    super();
    this.walletAddress = walletAddress;
  }
  
  pay(amount) {
    console.log(`Оплата ${amount} руб. через Bitcoin на ${this.walletAddress}`);
  }
}

// Контекст
class ShoppingCart {
  constructor() {
    this.items = [];
    this.paymentStrategy = null;
  }
  
  addItem(item) {
    this.items.push(item);
  }
  
  setPaymentStrategy(strategy) {
    this.paymentStrategy = strategy;
  }
  
  processPayment() {
    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    if (this.paymentStrategy) {
      this.paymentStrategy.pay(total);
    } else {
      throw new Error('Не установлена стратегия оплаты');
    }
  }
}

// Использование
const cart = new ShoppingCart();
cart.addItem({ name: 'Книга', price: 500 });
cart.addItem({ name: 'Мышь', price: 300 });

cart.setPaymentStrategy(new CreditCardStrategy('1234567890123456', '123'));
cart.processPayment(); // Оплата 800 руб. с карты 1234...
```

## Паттерны для работы с DOM

### Компонент (Component)

```javascript
class Component {
  constructor(props = {}) {
    this.props = props;
    this.state = {};
    this.element = null;
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.render();
  }
  
  render() {
    // Переопределяется в наследниках
    if (this.element) {
      this.element.innerHTML = this.template();
    }
  }
  
  template() {
    // Переопределяется в наследниках
    return '';
  }
  
  mount(container) {
    this.element = document.createElement('div');
    container.appendChild(this.element);
    this.render();
    this.componentDidMount();
  }
  
  componentDidMount() {
    // Переопределяется в наследниках
  }
  
  componentWillUnmount() {
    if (this.element && this.element.parentNode) {
      this.element.parentNode.removeChild(this.element);
    }
  }
}

// Пример компонента
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: props.initialCount || 0 };
  }
  
  template() {
    return `
      <div class="counter">
        <h3>Счетчик: ${this.state.count}</h3>
        <button onclick="counter.increment()">+</button>
        <button onclick="counter.decrement()">-</button>
      </div>
    `;
  }
  
  increment() {
    this.setState({ count: this.state.count + 1 });
  }
  
  decrement() {
    this.setState({ count: this.state.count - 1 });
  }
  
  componentDidMount() {
    console.log('Компонент счетчика смонтирован');
  }
}

// Использование
const counter = new Counter({ initialCount: 5 });
counter.mount(document.getElementById('app'));
```

### Посетитель (Visitor)

```javascript
// Интерфейс элемента
class Element {
  accept(visitor) {
    throw new Error('Метод accept должен быть реализован');
  }
}

// Конкретные элементы
class TextElement extends Element {
  constructor(text) {
    super();
    this.text = text;
  }
  
  accept(visitor) {
    visitor.visitTextElement(this);
  }
}

class ImageElement extends Element {
  constructor(src, alt) {
    super();
    this.src = src;
    this.alt = alt;
  }
  
  accept(visitor) {
    visitor.visitImageElement(this);
  }
}

class LinkElement extends Element {
  constructor(href, text) {
    super();
    this.href = href;
    this.text = text;
  }
  
  accept(visitor) {
    visitor.visitLinkElement(this);
  }
}

// Интерфейс посетителя
class Visitor {
  visitTextElement(element) {}
  visitImageElement(element) {}
  visitLinkElement(element) {}
}

// Конкретные посетители
class HTMLVisitor extends Visitor {
  constructor() {
    super();
    this.html = '';
  }
  
  visitTextElement(element) {
    this.html += `<p>${element.text}</p>`;
  }
  
  visitImageElement(element) {
    this.html += `<img src="${element.src}" alt="${element.alt}">`;
  }
  
  visitLinkElement(element) {
    this.html += `<a href="${element.href}">${element.text}</a>`;
  }
  
  getHTML() {
    return this.html;
  }
}

class PlainTextVisitor extends Visitor {
  constructor() {
    super();
    this.text = '';
  }
  
  visitTextElement(element) {
    this.text += element.text + '\n';
  }
  
  visitImageElement(element) {
    this.text += `[Изображение: ${element.alt}]\n`;
  }
  
  visitLinkElement(element) {
    this.text += `${element.text} (${element.href})\n`;
  }
  
  getText() {
    return this.text;
  }
}

// Использование
const elements = [
  new TextElement('Привет, мир!'),
  new ImageElement('image.jpg', 'Красивое изображение'),
  new LinkElement('https://example.com', 'Ссылка на пример')
];

const htmlVisitor = new HTMLVisitor();
const textVisitor = new PlainTextVisitor();

elements.forEach(element => {
  element.accept(htmlVisitor);
  element.accept(textVisitor);
});

console.log('HTML:', htmlVisitor.getHTML());
console.log('Текст:', textVisitor.getText());
```

## Современные паттерны

### Модуль (Module Pattern)

```javascript
// Классический модульный паттерн
const CalculatorModule = (function() {
  let history = [];
  
  function addToHistory(operation, result) {
    history.push({ operation, result, timestamp: new Date() });
  }
  
  return {
    add(a, b) {
      const result = a + b;
      addToHistory(`${a} + ${b}`, result);
      return result;
    },
    
    subtract(a, b) {
      const result = a - b;
      addToHistory(`${a} - ${b}`, result);
      return result;
    },
    
    multiply(a, b) {
      const result = a * b;
      addToHistory(`${a} * ${b}`, result);
      return result;
    },
    
    divide(a, b) {
      if (b === 0) {
        throw new Error('Деление на ноль');
      }
      const result = a / b;
      addToHistory(`${a} / ${b}`, result);
      return result;
    },
    
    getHistory() {
      return [...history]; // Возвращаем копию
    },
    
    clearHistory() {
      history = [];
    }
  };
})();

// Использование
console.log(CalculatorModule.add(5, 3)); // 8
console.log(CalculatorModule.multiply(4, 6)); // 24
console.log(CalculatorModule.getHistory());
```

### Прототип (Prototype)

```javascript
// Прототип
class Vehicle {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
    this.year = new Date().getFullYear();
  }
  
  clone() {
    const cloned = Object.create(Object.getPrototypeOf(this));
    Object.assign(cloned, this);
    return cloned;
  }
  
  getInfo() {
    return `${this.year} ${this.brand} ${this.model}`;
  }
}

// Использование
const car = new Vehicle('Toyota', 'Camry');
const carCopy = car.clone();

carCopy.year = 2023;
carCopy.brand = 'Honda';

console.log(car.getInfo()); // 2025 Toyota Camry
console.log(carCopy.getInfo()); // 2023 Honda Camry
```

### Итератор (Iterator)

```javascript
class TaskList {
  constructor() {
    this.tasks = [];
  }
  
  addTask(task) {
    this.tasks.push(task);
  }
  
  // Реализация протокола итератора
  [Symbol.iterator]() {
    let index = 0;
    const tasks = this.tasks;
    
    return {
      next() {
        if (index < tasks.length) {
          return { value: tasks[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
  
  // Дополнительные методы итерации
  forEach(callback) {
    for (const task of this) {
      callback(task);
    }
  }
  
  filter(predicate) {
    const result = new TaskList();
    for (const task of this) {
      if (predicate(task)) {
        result.addTask(task);
      }
    }
    return result;
  }
  
  map(transform) {
    const result = new TaskList();
    for (const task of this) {
      result.addTask(transform(task));
    }
    return result;
  }
}

// Использование
const tasks = new TaskList();
tasks.addTask({ id: 1, title: 'Задача 1', completed: false });
tasks.addTask({ id: 2, title: 'Задача 2', completed: true });
tasks.addTask({ id: 3, title: 'Задача 3', completed: false });

// Использование с for...of
for (const task of tasks) {
  console.log(task.title);
}

// Использование методов
const completedTasks = tasks.filter(task => task.completed);
const titles = tasks.map(task => task.title);
```

## Практические примеры использования

### Паттерн "Цепочка обязанностей" для обработки запросов

```javascript
class RequestHandler {
  constructor() {
    this.nextHandler = null;
  }
  
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

class AuthHandler extends RequestHandler {
  handle(request) {
    if (!request.token) {
      return { error: 'Требуется аутентификация' };
    }
    console.log('Пользователь аутентифицирован');
    return super.handle(request);
  }
}

class ValidationHandler extends RequestHandler {
  handle(request) {
    if (!request.data || !request.data.email) {
      return { error: 'Email обязателен' };
    }
    console.log('Данные валидны');
    return super.handle(request);
  }
}

class BusinessLogicHandler extends RequestHandler {
  handle(request) {
    console.log('Выполнение бизнес-логики');
    return { success: true, data: 'Результат обработки' };
  }
}

// Создание цепочки
const authHandler = new AuthHandler();
const validationHandler = new ValidationHandler();
const businessHandler = new BusinessLogicHandler();

authHandler.setNext(validationHandler).setNext(businessHandler);

// Использование
const request = { token: 'abc123', data: { email: 'user@example.com' } };
const result = authHandler.handle(request);
console.log(result); // { success: true, data: 'Результат обработки' }
```

## Лучшие практики

### Комбинация паттернов

```javascript
// Комбинация Observer + Factory + Strategy
class EventManager {
  constructor() {
    this.observers = [];
    this.strategies = new Map();
  }
  
  // Регистрация стратегии обработки события
  registerStrategy(eventType, strategy) {
    this.strategies.set(eventType, strategy);
  }
  
  // Подписка на события
  subscribe(observer) {
    this.observers.push(observer);
  }
  
  // Выпуск события
  emit(eventType, data) {
    // Уведомление наблюдателей
    this.observers.forEach(observer => observer.update(eventType, data));
    
    // Выполнение стратегии обработки
    const strategy = this.strategies.get(eventType);
    if (strategy) {
      strategy.execute(data);
    }
  }
}

// Фабрика стратегий
class StrategyFactory {
  static createStrategy(type, config) {
    switch (type) {
      case 'log':
        return new LoggingStrategy(config);
      case 'notify':
        return new NotificationStrategy(config);
      case 'process':
        return new ProcessingStrategy(config);
      default:
        throw new Error(`Неизвестный тип стратегии: ${type}`);
    }
  }
}

class LoggingStrategy {
  constructor(config) {
    this.level = config.level || 'info';
  }
  
  execute(data) {
    console.log(`[${this.level}]`, data);
  }
}

class NotificationStrategy {
  execute(data) {
    // Отправка уведомления
    console.log('Отправка уведомления:', data);
  }
}

class ProcessingStrategy {
  execute(data) {
    // Обработка данных
    console.log('Обработка данных:', data);
  }
}

// Использование
const eventManager = new EventManager();

// Регистрация стратегий
eventManager.registerStrategy('user_login', StrategyFactory.createStrategy('log', { level: 'info' }));
eventManager.registerStrategy('error', StrategyFactory.createStrategy('notify', {}));

// Подписка наблюдателей
eventManager.subscribe({
  update: (eventType, data) => {
    console.log(`Наблюдатель получил событие: ${eventType}`, data);
  }
});

// Выпуск событий
eventManager.emit('user_login', { userId: 123, timestamp: Date.now() });
eventManager.emit('error', { message: 'Ошибка сервера', code: 500 });
```

## Ссылки по теме

- [[js/patterns/creational]] - Порождающие паттерны
- [[js/patterns/structural]] - Структурные паттерны
- [[js/patterns/behavioral]] - Поведенческие паттерны
- [[js/patterns/module-pattern]] - Модульный паттерн
- [[js/patterns/observer]] - Паттерн Наблюдатель
- [[js/patterns/factory]] - Фабричный паттерн