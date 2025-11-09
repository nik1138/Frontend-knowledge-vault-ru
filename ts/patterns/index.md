# Паттерны проектирования в TypeScript

## Введение в паттерны проектирования

Паттерны проектирования - это проверенные решения типичных проблем проектирования в объектно-ориентированном программировании. TypeScript, благодаря своей системе типов, позволяет реализовывать паттерны проектирования более безопасным и выразительным способом.

## Категории паттернов

### Порождающие паттерны
Паттерны, которые связаны с созданием объектов, позволяя создавать объекты гибким образом без указания конкретных классов.

- [[./creational|Порождающие паттерны]]
- [[../creational/dependency-injection|Внедрение зависимости]]

### Структурные паттерны
Паттерны, которые помогают составлять более крупные структуры из отдельных объектов, обеспечивая гибкость и эффективность.

- [[./structural|Структурные паттерны]]

### Поведенческие паттерны
Паттерны, которые определяют способы взаимодействия объектов, позволяя установить эффективное взаимодействие между ними.

- [[./behavioral|Поведенческие паттерны]]

### Архитектурные паттерны
Паттерны, которые определяют архитектурную структуру приложения в целом.

- [[./architectural|Архитектурные паттерны]]

### Функциональные паттерны
Паттерны, основанные на функциональном программировании.

- [[./functional|Функциональные паттерны]]

## Примеры паттернов в TypeScript

### Паттерн "Стратегия" (Strategy)

```ts
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string, private cvv: string) {}
  
  pay(amount: number): void {
    console.log(`Paid $${amount} using credit card ending in ${this.cardNumber.slice(-4)}`);
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}
  
  pay(amount: number): void {
    console.log(`Paid $${amount} using PayPal account ${this.email}`);
  }
}

class PaymentProcessor {
  constructor(private strategy: PaymentStrategy) {}
  
  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy;
  }
  
  processPayment(amount: number): void {
    this.strategy.pay(amount);
  }
}

// Использование
const processor = new PaymentProcessor(new CreditCardPayment("1234567890123456", "123"));
processor.processPayment(100); // Paid $100 using credit card ending in 3456

processor.setPaymentStrategy(new PayPalPayment("user@example.com"));
processor.processPayment(50); // Paid $50 using PayPal account user@example.com
```

### Паттерн "Наблюдатель" (Observer)

```ts
interface Observer<T> {
  update(data: T): void;
}

interface Subject<T> {
  subscribe(observer: Observer<T>): void;
  unsubscribe(observer: Observer<T>): void;
  notify(data: T): void;
}

class NewsAgency implements Subject<string> {
  private observers: Observer<string>[] = [];
  
  subscribe(observer: Observer<string>): void {
    this.observers.push(observer);
  }
  
  unsubscribe(observer: Observer<string>): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  
  notify(news: string): void {
    this.observers.forEach(observer => observer.update(news));
  }
  
  addNews(news: string): void {
    this.notify(news);
  }
}

class NewsChannel implements Observer<string> {
  constructor(private name: string) {}
  
  update(news: string): void {
    console.log(`${this.name} received news: ${news}`);
  }
}

// Использование
const agency = new NewsAgency();
const channel1 = new NewsChannel("Channel 1");
const channel2 = new NewsChannel("Channel 2");

agency.subscribe(channel1);
agency.subscribe(channel2);

agency.addNews("Breaking news: TypeScript 5.0 released!");
// Channel 1 received news: Breaking news: TypeScript 5.0 released!
// Channel 2 received news: Breaking news: TypeScript 5.0 released!
```

### Паттерн "Фабричный метод" (Factory Method)

```ts
interface Animal {
  makeSound(): string;
}

class Dog implements Animal {
  makeSound(): string {
    return "Woof!";
  }
}

class Cat implements Animal {
  makeSound(): string {
    return "Meow!";
  }
}

abstract class AnimalFactory {
  abstract createAnimal(): Animal;
  
  getAnimalInfo(): string {
    const animal = this.createAnimal();
    return `Animal sound: ${animal.makeSound()}`;
  }
}

class DogFactory extends AnimalFactory {
  createAnimal(): Animal {
    return new Dog();
  }
}

class CatFactory extends AnimalFactory {
  createAnimal(): Animal {
    return new Cat();
  }
}

// Использование
const dogFactory = new DogFactory();
console.log(dogFactory.getAnimalInfo()); // Animal sound: Woof!

const catFactory = new CatFactory();
console.log(catFactory.getAnimalInfo()); // Animal sound: Meow!
```

## Ключевые концепции

- **Интерфейсы и абстрактные классы**: основа для реализации паттернов
- **Полиморфизм**: возможность использовать разные реализации одного интерфейса
- **Инкапсуляция**: сокрытие внутренней реализации
- **Принципы SOLID**: основа для хорошего объектно-ориентированного дизайна

## Дальнейшее изучение

- [[../type-system/type-compatibility|Совместимость типов]]
- [[../generics/index|Обобщения]]
- [[../interfaces-classes/index|Интерфейсы и классы]]
- [[../functional-programming/index|Функциональное программирование]]
- [[./relations|Связи между паттернами]]