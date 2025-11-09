# Creational Design Patterns

Порождающие паттерны отвечают за создание экземпляров объектов. Они абстрагируют процесс инстанцирования, делая систему независимой от способа создания, композиции и представления объектов.

## Overview схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Creational Design Patterns                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Factory       │               │      Abstract Factory    │                                    │   Singleton     │
  │  Pattern        │               │  Pattern               │                                    │  Pattern        │
  │                 │               │                          │                                    │                 │
  │ - Object Creation │◄──────────────┤ - Family of Products     │─────────────────────────────────────►│ - Single Instance │
  │ - Flexible      │               │ - Related Factories      │                                    │ - Global Access │
  │   Instantiation │               │ - Common Interface       │                                    │ - Controlled    │
  │ - Type Safety   │               │ - Cross-platform         │                                    │   Instantiation │
  │ - Configurable  │               │ - Consistent Products    │                                    │ - Thread Safety │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │         Builder Pattern         │
        │                           │                               │
        │                           │ - Step-by-step Construction     │
        │                           │ - Complex Object Building       │
        │                           │ - Fluent Interface              │
        │                           │ - Director Pattern              │
        │                           │ - Product Variants              │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │      Prototype Pattern        │
        │                           │                               │
        │                           │ - Clone Instead of New        │
        │                           │ - Avoid Costly Initialization   │
        │                           │ - Custom Clone Logic          │
        │                           │ - Deep vs Shallow Copy        │
        │                           │ - Registry Pattern            │
        └───────────────────────────┤ - Prototype-Based Inheritance │
                                    └─────────────────────────────────┘
```

## Основные паттерны

### [Factory Pattern](factory.md)
Определяет интерфейс для создания объекта, но позволяет субклассам выбрать, какой класс инстанцировать.

```typescript
// Простой пример фабрики
interface Product {
  operation(): string;
}

class ConcreteProductA implements Product {
  operation(): string {
    return "Result of Product A";
  }
}

class ConcreteProductB implements Product {
  operation(): string {
    return "Result of Product B";
  }
}

class ProductFactory {
  static create(type: "A" | "B"): Product {
    switch (type) {
      case "A": return new ConcreteProductA();
      case "B": return new ConcreteProductB();
      default: throw new Error("Unknown product type");
    }
  }
}

const productA = ProductFactory.create("A");
console.log(productA.operation()); // "Result of Product A"
```

### [Abstract Factory Pattern](abstract-factory.md)
Предоставляет интерфейс для создания семейств взаимосвязанных или взаимозависимых объектов без спецификации их конкретных классов.

```typescript
interface Button {
  render(): void;
}

interface Checkbox {
  render(): void;
}

// Семейство продукции для Windows
class WindowsButton implements Button {
  render(): void { console.log("Windows button"); }
}

class WindowsCheckbox implements Checkbox {
  render(): void { console.log("Windows checkbox"); }
}

// Семейство продукции для macOS
class MacOSButton implements Button {
  render(): void { console.log("macOS button"); }
}

class MacOSCheckbox implements Checkbox {
  render(): void { console.log("macOS checkbox"); }
}

// Абстрактная фабрика
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

class WindowsFactory implements GUIFactory {
  createButton(): Button { return new WindowsButton(); }
  createCheckbox(): Checkbox { return new WindowsCheckbox(); }
}

class MacOSFactory implements GUIFactory {
  createButton(): Button { return new MacOSButton(); }
  createCheckbox(): Checkbox { return new MacOSCheckbox(); }
}
```

### [Builder Pattern](builder.md)
Разделяет конструирование сложного объекта от его представления, чтобы один и тот же процесс конструирования мог создавать разные представления.

```typescript
class House {
  walls: number = 0;
  windows: number = 0;
  doors: number = 0;
  roof: string = '';
  garage: boolean = false;
}

class HouseBuilder {
  private house = new House();
  
  setWalls(count: number): this {
    this.house.walls = count;
    return this;
  }
  
  setWindows(count: number): this {
    this.house.windows = count;
    return this;
  }
  
  setDoors(count: number): this {
    this.house.doors = count;
    return this;
  }
  
  setRoof(type: string): this {
    this.house.roof = type;
    return this;
  }
  
  addGarage(): this {
    this.house.garage = true;
    return this;
  }
  
  build(): House {
    return this.house;
  }
}

// Использование (fluent interface)
const house = new HouseBuilder()
  .setWalls(4)
  .setWindows(8)
  .setDoors(2)
  .setRoof("tile")
  .addGarage()
  .build();
```

### [Prototype Pattern](prototype.md)
Определяет виды создаваемых объектов с помощью экземпляра-прототипа и создает новые объекты путем копирования этого прототипа.

```typescript
interface Prototype<T> {
  clone(): T;
}

class Document implements Prototype<Document> {
  constructor(
    public title: string,
    public content: string,
    public metadata: Record<string, any> = {}
  ) {}
  
  clone(): Document {
    return new Document(
      this.title,
      this.content,
      { ...this.metadata } // поверхностное копирование метаданных
    );
  }
  
  deepClone(): Document {
    return new Document(
      this.title,
      this.content,
      JSON.parse(JSON.stringify(this.metadata)) // глубокое копирование
    );
  }
}

const originalDoc = new Document("Report", "Content here", { author: "Alice", date: new Date() });
const clonedDoc = originalDoc.clone();
clonedDoc.title = "Cloned Report";
console.log(originalDoc.title); // "Report"
console.log(clonedDoc.title);   // "Cloned Report"
```

### [Singleton Pattern](singleton.md)
Гарантирует, что класс имеет только один экземпляр, и предоставляет глобальную точку доступа к нему.

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  
  private constructor() {
    // Инициализация соединения
  }
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
  
  query(sql: string) {
    console.log(`Executing: ${sql}`);
  }
}

const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true - один и тот же экземпляр
```

### [[../../dependency-injection/Внедрение-зависимостей|Dependency Injection]]
Шаблон, при котором один объект (инжектор) вводит зависимые объекты в другой объект (клиент). Это позволяет достичь слабой связанности.

```typescript
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(message);
  }
}

class UserService {
  constructor(private logger: Logger) {} // Внедрение зависимости
  
  createUser(name: string): void {
    this.logger.log(`Creating user: ${name}`);
    // логика создания пользователя
  }
}

// Использование
const logger = new ConsoleLogger();
const userService = new UserService(logger);
userService.createUser("Alice");
```

## Сравнение паттернов

| Pattern | Когда использовать | Преимущества | Недостатки |
|---------|-------------------|--------------|------------|
| Factory | Когда создание объекта сложно или зависит от условий | Простота использования, инкапсуляция создания | Может усложнить код при простых случаях |
| Abstract Factory | Когда нужно создавать семейства связанных объектов | Гарантия совместимости продуктов | Сложность добавления новых типов продуктов |
| Builder | Когда нужно создавать сложные объекты пошагово | Контроль над процессом конструирования | Больше кода, чем фабрика |
| Prototype | Когда создание экземпляра дорогое, но копирование дешевое | Эффективность при сложной инициализации | Сложности с глубоким копированием |
| Singleton | Когда нужен ровно один экземпляр | Глобальный доступ, контролируемое создание | Сложности с тестированием, скрытая зависимость |
| DI | Когда нужно уменьшить зацепление | Тестируемость, гибкость, инверсия зависимости | Более сложная конфигурация |

## Современные тенденции

### Использование с TypeScript
- Обобщения для типобезопасного создания
- Декораторы для автоматического внедрения
- Conditional types для сложных фабрик

### Интеграция с фреймворками
- Angular: встроенный DI контейнер
- NestJS: декораторы для DI
- React: dependency injection через hooks и контекст

### Функциональные альтернативы
- Dependency injection через функции высшего порядка
- Service locator через замыкания
- Composition roots для функциональных систем

## Практические применения

### В веб-приложениях
- Создание сервисов данных
- Управление состоянием
- Работа с API
- Логирование и мониторинг

### В enterprise-приложениях
- Управление ресурсами (соединения с БД)
- Конфигурация приложения
- Аутентификация и авторизация
- Кеширование

## Лучшие практики

1. **Предпочитайте фабрики синглтонам** - когда не нужна уникальность
2. **Используйте DI вместо жестких зависимостей** - для тестируемости
3. **Применяйте builder для сложных объектов** - с множеством опциональных параметров
4. **Используйте prototype для дорогой инициализации** - когда копирование эффективнее создания
5. **Избегайте синглтонов при возможных альтернативах** - из-за проблем с тестированием

## Связь с другими концепциями
- [[../structural-patterns/index]] - как созданные объекты соединяются структурно
- [[../behavioral-patterns/index]] - как созданные объекты взаимодействуют
- [[../functional-programming/di-patterns]] - функциональные альтернативы порождающим паттернам
- [[../modules/system-design]] - организация созданных компонентов в модули
- [[../type-system/generics-patterns]] - обобщенные паттерны создания