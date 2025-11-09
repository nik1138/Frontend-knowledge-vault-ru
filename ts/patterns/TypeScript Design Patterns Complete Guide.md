# TypeScript Design Patterns Complete Guide

Это полное руководство по паттернам проектирования в TypeScript, охватывающее все уровни абстракции: от базовых идиом языка до архитектурных решений enterprise-уровня.

## Структура паттернов

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        TypeScript Design Patterns                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │  Creational     │               │      Structural          │                                    │  Behavioral     │
  │  Patterns       │               │  Patterns                │                                    │  Patterns       │
  │                 │               │                          │                                    │                 │
  │ Factory         │◄──────────────┤ Adapter                  │─────────────────────────────────────►│ Observer        │
  │ Singleton       │               │ Decorator                │                                    │ Strategy        │
  │ Builder         │               │ Composite                │                                    │ Command         │
  │ Abstract Factory│               │ Facade                   │                                    │ State           │
  │ Prototype       │               │ Proxy                    │                                    │ Iterator        │
  │ Dependency      │               │ Bridge                   │                                    │ Mediator        │
  │   Injection     │               │ Flyweight                │                                    │ Memento         │
  │ Chain of        │               │                          │                                    │ Visitor         │
  │   Responsibilities│             │                          │                                    │ Interpreter     │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │        Functional Patterns      │
        │                           │                               │
        │                           │ - Pure Functions                │
        │                           │ - Immutability                  │
        │                           │ - Higher-Order Functions        │
        │                           │ - Currying & Partial Application│
        │                           │ - Composition                   │
        │                           │ - Functors & Monads             │
        │                           │ - Reactive Programming          │
        │                           │ - Functional State Management   │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                      Architectural Patterns                         │
        │                           │                                                                 │
        │                           │ - Clean Architecture                                              │
        │                           │ - MVC / MVP / MVVM                                                │
        │                           │ - Hexagonal Architecture                                          │
        │                           │ - Layered Architecture                                            │
        │                           │ - Microservices Architecture                                      │
        │                           │ - Event-Driven Architecture                                       │
        │                           │ - Component-Based Architecture                                    │
        │                           │ - Domain-Driven Design                                            │
        └───────────────────────────┤ - Service-Oriented Architecture                                   │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Классификация паттернов

### [[TS Creational Design Patterns]] - Порождающие паттерны
Паттерны, отвечающие за создание объектов:

- [Factory Pattern](creational/factory.md) - создание объектов через интерфейс
- [Abstract Factory](creational/abstract-factory.md) - создание семейств объектов
- [Builder Pattern](creational/builder.md) - пошаговое создание сложных объектов  
- [Prototype Pattern](creational/prototype.md) - создание объектов через клонирование
- [Singleton Pattern](creational/singleton.md) - гарантия единственности экземпляра
- [[creational/dependency-injection|Dependency Injection]] - внедрение зависимостей

### [[TS Structural Patterns]] - Структурные паттерны
Паттерны, описывающие структуру объектов:

- [Adapter Pattern](structural/adapter.md) - преобразование интерфейсов
- [Decorator Pattern](structural/decorator.md) - динамическое добавление функциональности
- [Composite Pattern](structural/composite.md) - древовидные структуры
- [Facade Pattern](structural/facade.md) - упрощенный интерфейс к сложной подсистеме
- [Proxy Pattern](structural/proxy.md) - контроль доступа к объекту
- [Bridge Pattern](structural/bridge.md) - отделение абстракции от реализации
- [Flyweight Pattern](structural/flyweight.md) - эффективное использование памяти

### [[TS Behavioral Patterns]] - Поведенческие паттерны
Паттерны, определяющие взаимодействие между объектами:

- [Observer Pattern](behavioral/observer.md) - уведомление об изменениях
- [Strategy Pattern](behavioral/strategy.md) - выбор алгоритма во время выполнения
- [Command Pattern](behavioral/command.md) - инкапсуляция запроса как объекта
- [State Pattern](behavioral/state.md) - изменение поведения в зависимости от состояния
- [Iterator Pattern](behavioral/iterator.md) - последовательный доступ к элементам
- [Mediator Pattern](behavioral/mediator.md) - координация между объектами
- [Memento Pattern](behavioral/memento.md) - сохранение и восстановление состояния
- [Visitor Pattern](behavioral/visitor.md) - выполнение операции над элементами структуры
- [Template Method](behavioral/template-method.md) - определение скелета алгоритма
- [Chain of Responsibility](behavioral/chain-of-responsibility.md) - передача запросов по цепочке
- [Interpreter Pattern](behavioral/interpreter.md) - интерпретация грамматики

### [[functional/index]] - Функциональные паттерны
Функциональные подходы к программированию:

- [Pure Functions](functional/pure-functions.md) - функции без побочных эффектов
- [Immutability](functional/immutability.md) - неизменяемость данных
- [Higher-Order Functions](functional/higher-order.md) - функции, принимающие или возвращающие другие функции
- [Currying and Composition](functional/composition.md) - преобразование и соединение функций
- [Functors and Monads](functional/functors-monads.md) - продвинутые функциональные концепции
- [Reactive Programming](functional/reactive.md) - функциональное реактивное программирование

### [[architectural/index]] - Архитектурные паттерны
Паттерны на уровне архитектуры приложения:

- [Clean Architecture](architectural/clean.md) - архитектура с чистыми слоями
- [MVC/MVVM/MVP](architectural/mvc.md) - разделение ответственности в UI
- [Hexagonal Architecture](architectural/hexagonal.md) - архитектура портов и адаптеров
- [Layered Architecture](architectural/layered.md) - многоуровневая архитектура
- [Microservices](architectural/microservices.md) - архитектура микросервисов
- [Event-Driven Architecture](architectural/eda.md) - архитектура, ориентированная на события

## Интеграция паттернов

Паттерны в TypeScript редко используются изолированно. В реальных приложениях они комбинируются для создания мощных архитектурных решений.

### Пример комплексного использования:

```typescript
// Использование нескольких паттернов одновременно
class UserService {
  // Creational: Dependency Injection
  constructor(
    private userRepository: UserRepository,  // инъекция зависимости
    private notificationService: NotificationService
  ) {}
  
  // Behavioral: Strategy Pattern (алгоритм валидации может варьироваться)
  private validationStrategy: ValidationStrategy = new UserValidationStrategy();
  
  // Behavioral: Command Pattern (для логирования операций)
  async createUser(userData: CreateUserRequest): Promise<User> {
    // Structural: Facade для комплексной операции
    const validation = this.validationStrategy.validate(userData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Creational: Builder для создания сложного объекта User
    const user = new UserBuilder()
      .withPersonalInfo(userData)
      .withDefaultPermissions()
      .withCreatedAt(new Date())
      .build();
    
    // Behavioral: Observer для уведомления о создании пользователя
    const result = await this.userRepository.save(user);
    
    // Behavioral: Observer - уведомляем подписчиков
    this.notificationService.notify('user.created', { userId: result.id });
    
    return result;
  }
  
  // Structural: Adapter для интеграции с внешними сервисами
  async syncWithExternalSystem(userId: number): Promise<boolean> {
    const userData = await this.userRepository.findById(userId);
    if (!userData) return false;
    
    // Адаптер преобразует внутренний формат к формату внешней системы
    const externalAdapter = new ExternalSystemUserAdapter(userData);
    return await externalAdapter.sync();
  }
}
```

## Руководства по выбору паттернов

### Когда использовать каждый тип:

#### Creational Patterns
- **Используйте при**: сложной логике создания объектов, необходимости выбора конкретного класса во время выполнения
- **Примеры**: фабрики для подключения к разным базам данных, билдеры для сложных объектов конфигурации

#### Structural Patterns  
- **Используйте при**: необходимости изменения интерфейса объекта, организации иерархии объектов, интеграции сторонних библиотек
- **Примеры**: адаптеры для интеграции с внешними API, фасады для сложных подсистем

#### Behavioral Patterns
- **Используйте при**: необходимости определения взаимодействия между объектами, реализации сложного поведения
- **Примеры**: обработчик событий, системы уведомлений, системы валидации

#### Functional Patterns
- **Используйте при**: обработке данных, трансформации данных, функциональном подходе к проектированию
- **Примеры**: обработка API данных, вычисления, преобразования конфигураций

## Современные тенденции

### 1. Комбинация парадигм
Современные TypeScript приложения часто используют комбинацию объектно-ориентированных паттернов и функциональных подходов:

```typescript
// ООП для структуры, функциональные паттерны для логики
class DataProcessor {
  // Состояние и поведение в классе
  private data: any[] = [];
  
  // Функциональные утилиты для обработки
  private filter: (item: any) => boolean = () => true;
  private transform: (item: any) => any = item => item;
  
  processData(): any[] {
    // Функциональная цепочка обработки
    return this.data
      .filter(this.filter)      // Behavioral (Strategy)
      .map(this.transform)      // Functional (Higher-order function)
      .reduce((acc, curr) => {  // Functional (Reducing)
        acc.push(curr);
        return acc;
      }, []);
  }
}
```

### 2. Типобезопасные паттерны
TypeScript позволяет создавать паттерны с сильной типизацией:

```typescript
// Типобезопасный паттерн "Посетитель"
interface Element {
  accept(visitor: Visitor): void;
}

interface Visitor {
  visitString(element: StringElement): void;
  visitNumber(element: NumberElement): void;
  visitObject(element: ObjectElement): void;
}

class StringElement implements Element {
  constructor(public value: string) {}
  accept(visitor: Visitor): void {
    visitor.visitString(this);  // Типобезопасный вызов
  }
}

// Использование обеспечивает гарантии типов на этапе компиляции
function processElements(elements: Element[], visitor: Visitor): void {
  elements.forEach(element => element.accept(visitor));
  // TypeScript гарантирует, что все элементы могут быть посещены
}
```

## Практические рекомендации

### 1. Не усложняйте без необходимости
```typescript
// ПЛОХО: лишняя сложность для простой задачи
class SimpleLogger {
  private strategy: LogStrategy;
  
  constructor(strategy: LogStrategy = new ConsoleLogStrategy()) {
    this.strategy = strategy;
  }
  
  log(message: string): void {
    this.strategy.log(message);
  }
}

// ЛУЧШЕ: простое решение для простой задачи
class SimpleLogger {
  log(message: string): void {
    console.log(message);  // если это всё, что нужно
  }
}
```

### 2. Используйте паттерны для решения реальных проблем
- Не применяйте паттерны ради паттернов
- Выбирайте паттерны на основе реальных потребностей архитектуры
- Используйте паттерны для улучшения сопровождаемости и тестируемости

### 3. Документируйте выбор паттернов
```typescript
/**
 * Используем паттерн Стратегия для:
 * 1. Легкого переключения между алгоритмами сортировки
 * 2. Возможности расширения новыми алгоритмами
 * 3. Упрощения юнит-тестирования каждого алгоритма отдельно
 */
class DataSorter {
  constructor(private strategy: SortingStrategy) {}
  
  sort(data: any[]): any[] {
    return this.strategy.sort(data);
  }
}
```

## Интеграция с экосистемой

### С популярными фреймворками:
- **Angular**: Dependency Injection, Observer (RxJS), Strategy (Pipes)
- **React**: State (useState), Effects (useEffect), Strategy (custom hooks)
- **Node.js**: Factory (dependency injection containers), Observer (EventEmitter), Decorator (middleware)

## Заключение

Паттерны проектирования в TypeScript:

1. **Решают определенные архитектурные задачи**
2. **Улучшают читаемость и сопровождаемость кода**
3. **Обеспечивают проверенные решения для распространенных проблем**
4. **Повышают качество и надежность приложений**

Ключ к успеху - понимание не только "как" использовать паттерны, но и "когда" и "почему". Правильный выбор паттерна в правильном контексте делает код более ясным, надежным и поддерживаемым.

## Связь с другими концепциями
- [[TypeScript Type System]] - система типов, обеспечивающая безопасность паттернов
- [[../generics/index]] - обобщения для создания типобезопасных паттернов
- [[../modules/index]] - организация паттернов в модульную структуру
- [[Advanced TypeScript Types]] - продвинутые типы для сложных паттернов
- [[../functional-programming/index]] - функциональные альтернативы ООП паттернам
- [[../ecosystem/architecture]] - архитектурные решения на базе паттернов