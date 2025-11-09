# Поведенческие паттерны проектирования

Поведенческие паттерны проектирования определяют способы эффективного взаимодействия между объектами, определяя, как они обмениваются информацией и какие роли они выполняют. Эти паттерны сосредоточены на поведении объектов и алгоритмах, обеспечивая гибкость и переиспользуемость в сложных системах.

## Определение поведенческих паттернов

Поведенческие паттерны решают задачи распределения обязанностей между объектами, определяя, как они взаимодействуют друг с другом. В отличие от структурных паттернов, которые заботятся о композиции классов, поведенческие паттерны определяют динамику между объектами.

> [!info] Связь с другими паттернами
> Поведенческие паттерны часто используются в сочетании с [[structural]] и [[creational]] паттернами для создания полной архитектуры приложения.

## Командный паттерн (Command Pattern)

Командный паттерн инкапсулирует запрос как объект, позволяя параметризовать клиентов с различными запросами, ставить запросы в очередь или поддерживать отмену операций.

### Пример реализации

```javascript
class Command {
  execute() {}
  undo() {}
}

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
```

### Когда использовать
- Когда необходима поддержка отмены/повтора операций
- Когда нужно параметризовать объекты выполнением операций
- Когда важна очередь команд или логирование операций

### Преимущества и недостатки
- **Преимущества**: Упрощает добавление новых команд, поддержка отмены/повтора, параметризация объектов
- **Недостатки**: Увеличение количества классов, сложность для простых операций

## Паттерн итератор (Iterator Pattern)

Итератор предоставляет способ последовательного доступа к элементам коллекции без раскрытия внутреннего представления.

### Пример реализации

```javascript
class Iterator {
  hasNext() {}
  next() {}
}

class ArrayIterator extends Iterator {
  constructor(array) {
    super();
    this.array = array;
    this.index = 0;
  }
  
  hasNext() {
    return this.index < this.array.length;
  }
  
  next() {
    return this.array[this.index++];
  }
}
```

### Когда использовать
- Когда нужно унифицировать обход различных структур данных
- Когда нужен контроль над процессом обхода

## Паттерн посредник (Mediator Pattern)

Посредник определяет объект, который инкапсулирует способ взаимодействия между группой объектов, делая их слабосвязанными.

### Пример реализации

```javascript
class ChatRoomMediator {
  showMessage(user, message) {
    const time = new Date().toISOString();
    console.log(`${time} [${user.getName()}]: ${message}`);
  }
}

class User {
  constructor(name, chatMediator) {
    this.name = name;
    this.chatMediator = chatMediator;
  }
  
  send(message) {
    this.chatMediator.showMessage(this, message);
  }
  
  getName() {
    return this.name;
  }
}
```

### Когда использовать
- Когда сложные зависимости между объектами приводят к спагетти-коду
- Когда нужно централизовать управление взаимодействиями

## Паттерн хранитель (Memento Pattern)

Хранитель позволяет сохранять и восстанавливать предыдущее состояние объекта без раскрытия его внутреннего представления.

### Пример реализации

```javascript
class Memento {
  constructor(state) {
    this.state = state;
  }
  
  getState() {
    return this.state;
  }
}

class Originator {
  setState(state) {
    this.state = state;
  }
  
  saveStateToMemento() {
    return new Memento(this.state);
  }
  
  restoreStateFromMemento(memento) {
    this.state = memento.getState();
  }
}
```

### Когда использовать
- Когда нужно реализовать механизм отката состояния
- Когда прямой доступ к состоянию нарушает инкапсуляцию

## Паттерн наблюдатель (Observer Pattern)

Наблюдатель определяет зависимость "один ко многим" между объектами, чтобы при изменении состояния одного объекта все зависящие от него автоматически обновлялись.

### Пример реализации

```javascript
class Subject {
  constructor() {
    this.observers = [];
  }
  
  attach(observer) {
    this.observers.push(observer);
  }
  
  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  update(data) {
    console.log('Received:', data);
  }
}
```

### Когда использовать
- Когда изменения одного объекта требуют изменений других
- Когда нужно уведомлять объекты без жесткой зависимости

## Паттерн состояние (State Pattern)

Состояние позволяет объекту изменять свое поведение при изменении внутреннего состояния.

### Пример реализации

```javascript
class State {
  handle(context) {}
}

class ConcreteStateA extends State {
  handle(context) {
    console.log("State A behavior");
    context.setState(new ConcreteStateB());
  }
}

class ConcreteStateB extends State {
  handle(context) {
    console.log("State B behavior");
    context.setState(new ConcreteStateA());
  }
}

class Context {
  constructor() {
    this.state = new ConcreteStateA();
  }
  
  setState(state) {
    this.state = state;
  }
  
  request() {
    this.state.handle(this);
  }
}
```

### Когда использовать
- Когда поведение объекта должно меняться в зависимости от его состояния
- Когда код содержит условные операторы, зависящие от состояния

## Паттерн стратегия (Strategy Pattern)

Стратегия определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми.

### Пример реализации

```javascript
class Strategy {
  execute(a, b) {}
}

class ConcreteStrategyAdd extends Strategy {
  execute(a, b) {
    return a + b;
  }
}

class ConcreteStrategySubtract extends Strategy {
  execute(a, b) {
    return a - b;
  }
}

class Context {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  executeStrategy(a, b) {
    return this.strategy.execute(a, b);
  }
}
```

### Когда использовать
- Когда нужно выбирать алгоритм во время выполнения
- Когда есть несколько связанных классов, отличающихся поведением

## Паттерн шаблонный метод (Template Method Pattern)

Шаблонный метод определяет скелет алгоритма в методе, откладывая определение некоторых шагов на подклассы.

### Пример реализации

```javascript
class AbstractClass {
  templateMethod() {
    this.baseOperation1();
    this.requiredOperation();
    this.baseOperation2();
    if (this.hook()) {
      this.optionalOperation();
    }
  }
  
  baseOperation1() {
    console.log("Base operation 1");
  }
  
  baseOperation2() {
    console.log("Base operation 2");
  }
  
  requiredOperation() {}
  
  hook() {
    return true;
  }
  
  optionalOperation() {}
}

class ConcreteClass extends AbstractClass {
  requiredOperation() {
    console.log("Required operation implementation");
  }
  
  hook() {
    return false;
  }
}
```

### Когда использовать
- Когда подклассы должны переопределять только часть алгоритма
- Когда нужно избежать дублирования кода в нескольких классах

## Паттерн посетитель (Visitor Pattern)

Посетитель позволяет определить новую операцию без изменения классов объектов, над которыми она выполняется.

### Пример реализации

```javascript
class Visitor {
  visit(element) {}
}

class ConcreteVisitor extends Visitor {
  visit(element) {
    console.log(`Visiting ${element.constructor.name}`);
  }
}

class Element {
  accept(visitor) {
    visitor.visit(this);
  }
}

class ConcreteElement extends Element {
  accept(visitor) {
    visitor.visit(this);
  }
}
```

### Когда использовать
- Когда нужно выполнить операцию над объектами сложной структуры
- Когда операции часто изменяются, а структура объектов стабильна

## Рекомендации по использованию

Каждый поведенческий паттерн решает конкретную проблему взаимодействия объектов. При выборе паттерна учитывайте:

- **Сложность взаимодействия** между объектами
- **Изменчивость** поведения в зависимости от условий
- **Необходимость** отката операций или поддержки истории
- **Частоту изменения** алгоритмов или операций

## Заключение

Поведенческие паттерны обеспечивают гибкость в проектировании взаимодействий между объектами. Правильное применение этих паттернов улучшает поддерживаемость кода и упрощает его расширение.

> [!tip] См. также
> - [[creational]] - порождающие паттерны
> - [[structural]] - структурные паттерны
> - [[architecture/component-architecture]] - архитектура компонентов

#programming #design-patterns #behavioral-patterns #architecture