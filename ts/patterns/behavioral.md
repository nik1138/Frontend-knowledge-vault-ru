# Behavioral Patterns

Поведенческие паттерны в TypeScript описывают распространенные шаблоны взаимодействия между объектами, определяя, как они передают данные и управляют коммуникацией. Эти паттерны сосредоточены на распределении обязанностей между объектами.

## Основные поведенческие паттерны

### [Observer Pattern](observer.md)
Позволяет объекту наблюдать за изменениями в другом объекте и реагировать на них автоматически.

### [Strategy Pattern](strategy.md)
Определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми.

### [Command Pattern](command.md)
Инкапсулирует запрос как объект, позволяя параметризовать клиентов различными запросами.

### [State Pattern](state.md)
Позволяет объекту изменять свое поведение в зависимости от внутреннего состояния.

### [Iterator Pattern](iterator.md)
Предоставляет способ последовательного доступа к элементам коллекции без раскрытия ее внутреннего представления.

### [Mediator Pattern](mediator.md)
Определяет объект, инкапсулирующий способ взаимодействия между объектами.

### [Memento Pattern](memento.md)
Позволяет зафиксировать и восстановить внутреннее состояние объекта.

### [Visitor Pattern](visitor.md)
Позволяет определить новую операцию без изменения классов объектов, над которыми она выполняется.

### [Template Method Pattern](template-method.md)
Определяет каркас алгоритма в базовом классе, позволяя подклассам переопределять шаги алгоритма.

### [Chain of Responsibility Pattern](chain-of-responsibility.md)
Передает запросы последовательно по цепочке обработчиков.

### [Interpreter Pattern](interpreter.md)
Позволяет определить грамматику языка и интерпретатор для этой грамматики.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Behavioral Design Patterns                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │  Observer       │               │      Strategy            │                                    │    Command      │
  │  Pattern        │               │  Pattern               │                                    │  Pattern        │
  │                 │               │                          │                                    │                 │
  │ - Publisher/    │◄──────────────┤ - Algorithm Family       │─────────────────────────────────────►│ - Encapsulated  │
  │   Subscriber    │               │ - Replaceable Algorithms │                                    │   Requests      │
  │ - Event         │               │ - Open/Closed Principle  │                                    │ - Undo/Redo     │
  │   System        │               │ - Runtime Selection      │                                    │ - Transactional │
  │ - Reactive      │               │ - Context Pattern        │                                    │   Operations    │
  │   Programming   │               │                          │                                    │ - Queueing      │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │          State Pattern          │
        │                           │                               │
        │                           │ - Internal State Management     │
        │                           │ - Behavior Changes by State     │
        │                           │ - Finite State Machine          │
        │                           │ - State Transitions             │
        │                           │ - Context Pattern               │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                         Iterator Pattern                          │
        │                           │                                                                 │
        │                           │ - Sequential Access                                               │
        │                           │ - Traversal Without Exposure                                        │
        │                           │ - Collection Agnostic                                               │
        │                           │ - Lazy Evaluation                                                 │
        │                           │ - Generator Functions                                               │
        └───────────────────────────┤ - External vs Internal Iteration                                   │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Observer Pattern

### Классический Observer
```typescript
interface Observer<T> {
  update(data: T): void;
}

interface Subject<T> {
  subscribe(observer: Observer<T>): void;
  unsubscribe(observer: Observer<T>): void;
  notify(data: T): void;
}

// Конкретный субъект
class NewsAgency implements Subject<string> {
  private observers: Observer<string>[] = [];
  private news: string = '';
  
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
  
  setNews(news: string): void {
    this.news = news;
    this.notify(news);
  }
}

// Конкретные наблюдатели
class NewsChannel implements Observer<string> {
  constructor(private name: string) {}
  
  update(news: string): void {
    console.log(`${this.name} received news: ${news}`);
  }
}

// Использование
const agency = new NewsAgency();
const cnn = new NewsChannel("CNN");
const bbc = new NewsChannel("BBC");

agency.subscribe(cnn);
agency.subscribe(bbc);

agency.setNews("Breaking: New TypeScript release!");
// Output:
// CNN received news: Breaking: New TypeScript release!
// BBC received news: Breaking: New TypeScript release!
```

### RxJS стиль Observer
```typescript
// Простая реализация Observable
interface Subscription {
  unsubscribe(): void;
}

interface Observer<T> {
  next?(value: T): void;
  error?(err: any): void;
  complete?(): void;
}

interface Observable<T> {
  subscribe(observer: Observer<T>): Subscription;
  subscribe(
    next?: (value: T) => void,
    error?: (err: any) => void,
    complete?: () => void
  ): Subscription;
  
  map<U>(fn: (value: T) => U): Observable<U>;
  filter(predicate: (value: T) => boolean): Observable<T>;
  throttle(time: number): Observable<T>;
}

class EventEmitter<T> implements Observable<T> {
  private observers: Observer<T>[] = [];
  
  subscribe(
    nextOrObserver: ((value: T) => void) | Observer<T>,
    error?: (err: any) => void,
    complete?: () => void
  ): Subscription {
    const observer = typeof nextOrObserver === 'function' 
      ? { next: nextOrObserver, error, complete }
      : nextOrObserver;
    
    this.observers.push(observer);
    
    return {
      unsubscribe: () => {
        const index = this.observers.indexOf(observer);
        if (index > -1) {
          this.observers.splice(index, 1);
        }
      }
    };
  }
  
  next(value: T): void {
    this.observers.forEach(observer => {
      try {
        observer.next?.(value);
      } catch (err) {
        observer.error?.(err);
      }
    });
  }
  
  error(err: any): void {
    this.observers.forEach(observer => {
      observer.error?.(err);
    });
  }
  
  complete(): void {
    this.observers.forEach(observer => {
      observer.complete?.();
    });
  }
  
  map<U>(fn: (value: T) => U): Observable<U> {
    return new MapOperator(this, fn);
  }
  
  filter(predicate: (value: T) => boolean): Observable<T> {
    return new FilterOperator(this, predicate);
  }
}

class MapOperator<T, U> implements Observable<U> {
  constructor(
    private source: Observable<T>,
    private fn: (value: T) => U
  ) {}
  
  subscribe(observer: Observer<U>): Subscription {
    return this.source.subscribe({
      next: (value) => observer.next?.(this.fn(value)),
      error: (err) => observer.error?.(err),
      complete: () => observer.complete?.()
    });
  }
  
  map<V>(fn: (value: U) => V): Observable<V> {
    return new MapOperator(this, fn);
  }
  
  filter(predicate: (value: U) => boolean): Observable<U> {
    return new FilterOperator(this, predicate);
  }
}

class FilterOperator<T> implements Observable<T> {
  constructor(
    private source: Observable<T>,
    private predicate: (value: T) => boolean
  ) {}
  
  subscribe(observer: Observer<T>): Subscription {
    return this.source.subscribe({
      next: (value) => {
        if (this.predicate(value)) {
          observer.next?.(value);
        }
      },
      error: (err) => observer.error?.(err),
      complete: () => observer.complete?.()
    });
  }
  
  map<U>(fn: (value: T) => U): Observable<U> {
    return new MapOperator(this, fn);
  }
  
  filter(predicate: (value: T) => boolean): Observable<T> {
    return new FilterOperator(this, predicate);
  }
}
```

## Strategy Pattern

### Классический стратегический паттерн
```typescript
interface SortingStrategy {
  sort<T>(array: T[]): T[];
}

class BubbleSort implements SortingStrategy {
  sort<T>(array: T[]): T[] {
    const result = [...array]; // создаем копию для иммутабельности
    const n = result.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (result[j] > result[j + 1]) {
          [result[j], result[j + 1]] = [result[j + 1], result[j]]; // swap
        }
      }
    }
    
    return result;
  }
}

class QuickSort implements SortingStrategy {
  sort<T>(array: T[]): T[] {
    if (array.length <= 1) {
      return [...array];
    }
    
    const pivot = array[Math.floor(array.length / 2)];
    const less: T[] = [];
    const equal: T[] = [];
    const greater: T[] = [];
    
    for (const element of array) {
      if (element < pivot) {
        less.push(element);
      } else if (element > pivot) {
        greater.push(element);
      } else {
        equal.push(element);
      }
    }
    
    return [
      ...this.sort(less),
      ...equal,
      ...this.sort(greater)
    ];
  }
}

class MergeSort implements SortingStrategy {
  sort<T>(array: T[]): T[] {
    if (array.length <= 1) {
      return [...array];
    }
    
    const mid = Math.floor(array.length / 2);
    const left = this.sort(array.slice(0, mid));
    const right = this.sort(array.slice(mid));
    
    return this.merge(left, right);
  }
  
  private merge<T>(left: T[], right: T[]): T[] {
    const result: T[] = [];
    let leftIndex = 0;
    let rightIndex = 0;
    
    while (leftIndex < left.length && rightIndex < right.length) {
      if (left[leftIndex] < right[rightIndex]) {
        result.push(left[leftIndex]);
        leftIndex++;
      } else {
        result.push(right[rightIndex]);
        rightIndex++;
      }
    }
    
    return [
      ...result,
      ...left.slice(leftIndex),
      ...right.slice(rightIndex)
    ];
  }
}

// Контекст
class Sorter<T> {
  constructor(private strategy: SortingStrategy) {}
  
  setStrategy(strategy: SortingStrategy) {
    this.strategy = strategy;
  }
  
  sort(array: T[]): T[] {
    return this.strategy.sort(array);
  }
}

// Использование
const data = [64, 34, 25, 12, 22, 11, 90];

const bubbleSorter = new Sorter<number>(new BubbleSort());
const quickSorter = new Sorter<number>(new QuickSort());
const mergeSorter = new Sorter<number>(new MergeSort());

console.log("Bubble sort:", bubbleSorter.sort(data));
console.log("Quick sort:", quickSorter.sort(data));
console.log("Merge sort:", mergeSorter.sort(data));

// Динамическое изменение стратегии
bubbleSorter.setStrategy(new QuickSort());
console.log("Changed to quick sort:", bubbleSorter.sort(data));
```

### Функциональный подход к стратегии
```typescript
// Использование функций вместо классов
type Strategy<T> = (data: T[]) => T[];

const bubbleSort: Strategy<number> = (arr) => {
  const result = [...arr];
  const n = result.length;
  
  for (let i = 0; i < n - 1; i++) {
    for (let j = 0; j < n - i - 1; j++) {
      if (result[j] > result[j + 1]) {
        [result[j], result[j + 1]] = [result[j + 1], result[j]];
      }
    }
  }
  
  return result;
};

const insertionSort: Strategy<number> = (arr) => {
  const result = [...arr];
  
  for (let i = 1; i < result.length; i++) {
    const current = result[i];
    let j = i - 1;
    
    while (j >= 0 && result[j] > current) {
      result[j + 1] = result[j];
      j--;
    }
    
    result[j + 1] = current;
  }
  
  return result;
};

// Использование функции стратегии
class FunctionalSorter<T> {
  constructor(private strategy: Strategy<T>) {}
  
  setStrategy(strategy: Strategy<T>) {
    this.strategy = strategy;
  }
  
  sort(data: T[]): T[] {
    return this.strategy(data);
  }
}

const functionalSorter = new FunctionalSorter<number>(bubbleSort);
console.log(functionalSorter.sort([5, 2, 8, 1, 9])); // [1, 2, 5, 8, 9]

functionalSorter.setStrategy(insertionSort);
console.log(functionalSorter.sort([5, 2, 8, 1, 9])); // [1, 2, 5, 8, 9]
```

## Command Pattern

### Классический Command
```typescript
interface Command {
  execute(): void;
  undo?(): void;
}

class Calculator {
  private currentValue = 0;
  
  add(value: number): void {
    this.currentValue += value;
  }
  
  subtract(value: number): void {
    this.currentValue -= value;
  }
  
  multiply(value: number): void {
    this.currentValue *= value;
  }
  
  divide(value: number): void {
    if (value === 0) throw new Error("Division by zero");
    this.currentValue /= value;
  }
  
  getValue(): number {
    return this.currentValue;
  }
  
  setValue(value: number): void {
    this.currentValue = value;
  }
}

// Команды
class AddCommand implements Command {
  constructor(
    private calculator: Calculator,
    private value: number,
    private previousValue: number = 0
  ) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.add(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

class SubtractCommand implements Command {
  constructor(
    private calculator: Calculator,
    private value: number,
    private previousValue: number = 0
  ) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.subtract(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

class MultiplyCommand implements Command {
  constructor(
    private calculator: Calculator,
    private value: number,
    private previousValue: number = 0
  ) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.multiply(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

// Менеджер команд
class CommandManager {
  private history: Command[] = [];
  private currentIndex = -1;
  
  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
    this.currentIndex++;
  }
  
  undo(): void {
    if (this.currentIndex >= 0) {
      const command = this.history[this.currentIndex];
      if (command.undo) {
        command.undo();
      }
      this.currentIndex--;
    }
  }
  
  redo(): void {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      const command = this.history[this.currentIndex];
      command.execute();
    }
  }
}

// Использование
const calculator = new Calculator();
const commandManager = new CommandManager();

const addCommand = new AddCommand(calculator, 10);
const multiplyCommand = new MultiplyCommand(calculator, 2);
const subtractCommand = new SubtractCommand(calculator, 5);

commandManager.executeCommand(addCommand);
console.log(calculator.getValue()); // 10

commandManager.executeCommand(multiplyCommand);
console.log(calculator.getValue()); // 20

commandManager.executeCommand(subtractCommand);
console.log(calculator.getValue()); // 15

// Отмена последней операции
commandManager.undo();
console.log(calculator.getValue()); // 20

// Повтор последней операции
commandManager.redo();
console.log(calculator.getValue()); // 15
```

### Менеджер команд с макросом
```typescript
// Команда-макрос для группировки нескольких команд
class MacroCommand implements Command {
  private commands: Command[] = [];
  
  add(command: Command): void {
    this.commands.push(command);
  }
  
  execute(): void {
    for (const command of this.commands) {
      command.execute();
    }
  }
  
  undo(): void {
    // Выполняем отмену в обратном порядке
    for (let i = this.commands.length - 1; i >= 0; i--) {
      const command = this.commands[i];
      if (command.undo) {
        command.undo();
      }
    }
  }
}

// Использование макроса
const macro = new MacroCommand();
macro.add(new AddCommand(calculator, 5));
macro.add(new MultiplyCommand(calculator, 3));
macro.add(new SubtractCommand(calculator, 2));

commandManager.executeCommand(macro);
console.log(calculator.getValue()); // (0 + 5) * 3 - 2 = 13
```

## State Pattern

### Классический State
```typescript
interface State {
  handle(context: Context): void;
  toString(): string;
}

class Context {
  private state: State;
  
  constructor(state: State) {
    this.state = state;
  }
  
  setState(state: State): void {
    this.state = state;
  }
  
  request(): void {
    this.state.handle(this);
  }
  
  getState(): State {
    return this.state;
  }
}

// Конкретные состояния
class ConcreteStateA implements State {
  handle(context: Context): void {
    console.log("Handling in State A");
    // Может изменить состояние контекста
    context.setState(new ConcreteStateB());
  }
  
  toString(): string {
    return "State A";
  }
}

class ConcreteStateB implements State {
  handle(context: Context): void {
    console.log("Handling in State B");
    // Может изменить состояние контекста
    context.setState(new ConcreteStateA());
  }
  
  toString(): string {
    return "State B";
  }
}

// Пример игрового персонажа
interface CharacterState {
  move(character: GameCharacter): void;
  attack(character: GameCharacter): void;
  toString(): string;
}

class GameCharacter {
  private state: CharacterState;
  
  constructor(initialState: CharacterState) {
    this.state = initialState;
  }
  
  setState(state: CharacterState): void {
    console.log(`State changed from ${this.state} to ${state}`);
    this.state = state;
  }
  
  move(): void {
    this.state.move(this);
  }
  
  attack(): void {
    this.state.attack(this);
  }
}

class StandingState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Character starts walking...");
    character.setState(new WalkingState());
  }
  
  attack(character: GameCharacter): void {
    console.log("Character can't attack while standing");
  }
  
  toString(): string {
    return "Standing";
  }
}

class WalkingState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Character continues walking...");
  }
  
  attack(character: GameCharacter): void {
    console.log("Character stops to aim and shoot...");
    character.setState(new AttackingState());
  }
  
  toString(): string {
    return "Walking";
  }
}

class AttackingState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Can't move while attacking");
  }
  
  attack(character: GameCharacter): void {
    console.log("Character shooting...");
  }
  
  toString(): string {
    return "Attacking";
  }
}

// Использование
const character = new GameCharacter(new StandingState());

character.move();    // "Character starts walking..."
console.log(character.getState().toString()); // "Walking"

character.attack();  // "Character stops to aim and shoot..." 
console.log(character.getState().toString()); // "Attacking"

character.move();    // "Can't move while attacking"
```

### Состояние с обобщениями
```typescript
// Обобщенное состояние для более сложных систем
interface State<T> {
  handle(context: Context<T>): void;
  canTransitionTo(state: string): boolean;
}

class Context<T> {
  private state: State<T>;
  private stateMap: Map<string, State<T>> = new Map();
  
  constructor(initialState: string, states: [string, State<T>][]) {
    states.forEach(([name, state]) => this.stateMap.set(name, state));
    this.state = this.stateMap.get(initialState)!;
  }
  
  setStateByName(name: string): void {
    const newState = this.stateMap.get(name);
    if (newState && this.state.canTransitionTo(name)) {
      this.state = newState;
    }
  }
  
  handle(): void {
    this.state.handle(this);
  }
}
```

## Iterator Pattern

### Классический итератор
```typescript
interface Iterator<T> {
  next(): { value: T; done: boolean };
  return?(value?: any): { value: any; done: boolean };
  throw?(e?: any): { value: any; done: boolean };
}

interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>;
}

class ArrayCollection<T> implements Iterable<T> {
  private items: T[] = [];
  
  constructor(items?: T[]) {
    this.items = items ? [...items] : [];
  }
  
  add(item: T): void {
    this.items.push(item);
  }
  
  remove(item: T): boolean {
    const index = this.items.indexOf(item);
    if (index > -1) {
      this.items.splice(index, 1);
      return true;
    }
    return false;
  }
  
  [Symbol.iterator](): Iterator<T> {
    let index = 0;
    const items = this.items;
    
    return {
      next(): { value: T; done: boolean } {
        if (index < items.length) {
          return { value: items[index++], done: false };
        } else {
          return { value: undefined!, done: true };
        }
      }
    };
  }
  
  // Дополнительный метод для получения конкретного итератора
  reverseIterator(): Iterator<T> {
    let index = this.items.length - 1;
    const items = this.items;
    
    return {
      next(): { value: T; done: boolean } {
        if (index >= 0) {
          return { value: items[index--], done: false };
        } else {
          return { value: undefined!, done: true };
        }
      }
    };
  }
}

// Использование
const collection = new ArrayCollection<number>([1, 2, 3, 4, 5]);

// Прямой итератор
for (const item of collection) {
  console.log(item); // 1, 2, 3, 4, 5
}

// Обратный итератор
const reverseIter = collection.reverseIterator();
let result = reverseIter.next();
while (!result.done) {
  console.log(result.value); // 5, 4, 3, 2, 1
  result = reverseIter.next();
}
```

### Итераторы с генериками
```typescript
interface IIterator<T> {
  current(): T;
  next(): T | undefined;
  hasNext(): boolean;
  reset(): void;
}

class GenericList<T> {
  private items: T[] = [];
  
  add(item: T): void {
    this.items.push(item);
  }
  
  createIterator(): IIterator<T> {
    return new ListIterator<T>(this);
  }
  
  getItems(): T[] {
    return [...this.items];
  }
}

class ListIterator<T> implements IIterator<T> {
  private index = 0;
  
  constructor(private list: GenericList<T>) {}
  
  current(): T {
    return this.list.getItems()[this.index];
  }
  
  next(): T | undefined {
    if (this.hasNext()) {
      return this.list.getItems()[++this.index];
    }
    return undefined;
  }
  
  hasNext(): boolean {
    return this.index < this.list.getItems().length - 1;
  }
  
  reset(): void {
    this.index = 0;
  }
}

// Использование
const stringList = new GenericList<string>();
stringList.add("Hello");
stringList.add("World");
stringList.add("TypeScript");

const iter = stringList.createIterator();
while (iter.hasNext()) {
  console.log(iter.current()); // "Hello", "World", "TypeScript"
  iter.next();
}
```

## Mediator Pattern

### Классический посредник
```typescript
interface Colleague {
  send(message: string, colleague: Colleague): void;
  receive(message: string): void;
}

interface Mediator {
  register(colleague: Colleague): void;
  send(message: string, sender: Colleague): void;
}

class ConcreteColleague implements Colleague {
  private mediator: Mediator;
  private name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  setMediator(mediator: Mediator): void {
    this.mediator = mediator;
  }
  
  send(message: string, colleague?: Colleague): void {
    if (this.mediator) {
      this.mediator.send(message, this);
    }
  }
  
  receive(message: string): void {
    console.log(`${this.name} received: ${message}`);
  }
}

class ChatMediator implements Mediator {
  private colleagues: Colleague[] = [];
  
  register(colleague: Colleague): void {
    colleague.setMediator(this);
    this.colleagues.push(colleague);
  }
  
  send(message: string, sender: Colleague): void {
    for (const colleague of this.colleagues) {
      if (colleague !== sender) {
        colleague.receive(message);
      }
    }
  }
}

// Использование
const mediator = new ChatMediator();

const alice = new ConcreteColleague("Alice");
const bob = new ConcreteColleague("Bob");
const charlie = new ConcreteColleague("Charlie");

mediator.register(alice);
mediator.register(bob);
mediator.register(charlie);

alice.send("Hello everyone!");
// Bob received: Hello everyone!
// Charlie received: Hello everyone!
```

### Генерический посредник
```typescript
interface Message {
  type: string;
  payload?: any;
}

interface Component<T extends Message> {
  id: string;
  handle(message: T): void;
}

class ComponentMediator {
  private components: Map<string, Component<any>> = new Map();
  
  register<T extends Message>(component: Component<T>): void {
    this.components.set(component.id, component);
  }
  
  send<T extends Message>(message: T, senderId?: string): void {
    this.components.forEach((component, id) => {
      // Не отправляем отправителю же сообщение
      if (senderId !== id) {
        try {
          component.handle(message);
        } catch (error) {
          console.error(`Error handling message in component ${id}:`, error);
        }
      }
    });
  }
  
  sendTo<T extends Message>(message: T, targetId: string): boolean {
    const target = this.components.get(targetId);
    if (target) {
      try {
        target.handle(message);
        return true;
      } catch (error) {
        console.error(`Error sending message to component ${targetId}:`, error);
        return false;
      }
    }
    return false;
  }
}

// Пример: система уведомлений
interface NotificationMessage extends Message {
  type: 'notification';
  userId: string;
  message: string;
  priority: 'low' | 'medium' | 'high';
}

class NotificationComponent implements Component<NotificationMessage> {
  constructor(public id: string) {}
  
  handle(message: NotificationMessage): void {
    if (message.type === 'notification') {
      console.log(`[${this.id}] Sending notification to user ${message.userId}: ${message.message}`);
      if (message.priority === 'high') {
        // отправить push уведомление
      }
    }
  }
}

class EmailComponent implements Component<NotificationMessage> {
  constructor(public id: string) {}
  
  handle(message: NotificationMessage): void {
    if (message.type === 'notification' && message.priority === 'low') {
      console.log(`[${this.id}] Sending email: ${message.message}`);
    }
  }
}

// Использование
const mediator = new ComponentMediator();
const notificationComp = new NotificationComponent("notification-svc");
const emailComp = new EmailComponent("email-svc");

mediator.register(notificationComp);
mediator.register(emailComp);

const message: NotificationMessage = {
  type: 'notification',
  userId: 'user123',
  message: 'Welcome to our service!',
  priority: 'high'
};

mediator.send(message); // NotificationComponent получит сообщение, EmailComponent - нет
```

## Memento Pattern

### Классический снимок
```typescript
class Memento {
  constructor(private state: string) {}
  
  getState(): string {
    return this.state;
  }
}

class Originator {
  private state: string = '';
  
  setState(state: string): void {
    console.log(`Originator: Setting state to ${state}`);
    this.state = state;
  }
  
  getState(): string {
    return this.state;
  }
  
  save(): Memento {
    console.log(`Originator: Saving state to Memento`);
    return new Memento(this.state);
  }
  
  restore(memento: Memento): void {
    this.state = memento.getState();
    console.log(`Originator: State restored to: ${this.state}`);
  }
}

class Caretaker {
  private mementos: Memento[] = [];
  private originator: Originator;
  
  constructor(originator: Originator) {
    this.originator = originator;
  }
  
  backup(): void {
    console.log(`Caretaker: Saving Originator's state...`);
    this.mementos.push(this.originator.save());
  }
  
  undo(): void {
    if (!this.mementos.length) {
      return;
    }
    
    const memento = this.mementos.pop()!;
    console.log(`Caretaker: Restoring state to ${memento.getState()}`);
    try {
      this.originator.restore(memento);
    } catch (e) {
      this.undo(); // если восстановление не удалось, попробовать предыдущее состояние
    }
  }
  
  showHistory(): void {
    console.log(`Caretaker: Memento List:`);
    for (const memento of this.mementos) {
      console.log(memento.getState());
    }
  }
}

// Использование
const originator = new Originator();
const caretaker = new Caretaker(originator);

originator.setState("State #1");
originator.setState("State #2");

caretaker.backup();

originator.setState("State #3");
console.log("Current State:", originator.getState());

caretaker.backup();

originator.setState("State #4");
console.log("Current State:", originator.getState());

caretaker.showHistory();

caretaker.undo();
console.log("Current State:", originator.getState());

caretaker.undo();
console.log("Current State:", originator.getState());
```

### Обобщенный Memento
```typescript
interface IMemento<T> {
  getState(): T;
}

class GenericMemento<T> implements IMemento<T> {
  constructor(private state: T) {}
  
  getState(): T {
    return this.state;
  }
}

class GenericOriginator<T> {
  constructor(private state: T) {}
  
  getState(): T {
    return this.state;
  }
  
  setState(state: T): void {
    this.state = state;
  }
  
  save(): IMemento<T> {
    return new GenericMemento<T>(JSON.parse(JSON.stringify(this.state)));
  }
  
  restore(memento: IMemento<T>): void {
    this.state = memento.getState();
  }
}

// Пример с объектом состояния
interface AppState {
  user: { name: string; id: number };
  theme: string;
  preferences: Record<string, boolean>;
}

class AppStateOriginator extends GenericOriginator<AppState> {
  constructor(initialState: AppState) {
    super(initialState);
  }
  
  updateTheme(theme: string): void {
    const currentState = this.getState();
    this.setState({ ...currentState, theme });
  }
  
  updatePreferences(prefName: string, value: boolean): void {
    const currentState = this.getState();
    this.setState({
      ...currentState,
      preferences: { ...currentState.preferences, [prefName]: value }
    });
  }
}
```

## Visitor Pattern

### Классический посетитель
```typescript
interface Visitor {
  visitElementA(element: ElementA): void;
  visitElementB(element: ElementB): void;
}

interface Element {
  accept(visitor: Visitor): void;
}

class ElementA implements Element {
  public name: string = "Element A";
  
  accept(visitor: Visitor): void {
    visitor.visitElementA(this);
  }
  
  operationA(): string {
    return "Element A operation";
  }
}

class ElementB implements Element {
  public name: string = "Element B";
  
  accept(visitor: Visitor): void {
    visitor.visitElementB(this);
  }
  
  operationB(): string {
    return "Element B operation";
  }
}

class ConcreteVisitor implements Visitor {
  visitElementA(element: ElementA): void {
    console.log(`Visited ${element.name} with ${element.operationA()}`);
  }
  
  visitElementB(element: ElementB): void {
    console.log(`Visited ${element.name} with ${element.operationB()}`);
  }
}

// Использование
const elements: Element[] = [new ElementA(), new ElementB()];
const visitor = new ConcreteVisitor();

elements.forEach(element => element.accept(visitor));
// Output:
// Visited Element A with Element A operation
// Visited Element B with Element B operation
```

### Утилита для посетителей с обобщениями
```typescript
// Обобщенный паттерн посетителя для AST
interface ASTNode {
  accept<T>(visitor: ASTVisitor<T>): T;
}

interface ASTVisitor<T> {
  visitBinaryExpression(node: BinaryExpression): T;
  visitIdentifier(node: Identifier): T;
  visitLiteral(node: Literal): T;
  visitFunctionDeclaration(node: FunctionDeclaration): T;
}

class BinaryExpression implements ASTNode {
  constructor(
    public left: ASTNode,
    public operator: string,
    public right: ASTNode
  ) {}
  
  accept<T>(visitor: ASTVisitor<T>): T {
    return visitor.visitBinaryExpression(this);
  }
}

class Identifier implements ASTNode {
  constructor(public name: string) {}
  
  accept<T>(visitor: ASTVisitor<T>): T {
    return visitor.visitIdentifier(this);
  }
}

class Literal implements ASTNode {
  constructor(public value: number | string) {}
  
  accept<T>(visitor: ASTVisitor<T>): T {
    return visitor.visitLiteral(this);
  }
}

class FunctionDeclaration implements ASTNode {
  constructor(
    public name: Identifier,
    public params: Identifier[],
    public body: ASTNode
  ) {}
  
  accept<T>(visitor: ASTVisitor<T>): T {
    return visitor.visitFunctionDeclaration(this);
  }
}

// Посетитель для вычисления выражений
class ExpressionEvaluator implements ASTVisitor<number> {
  visitBinaryExpression(node: BinaryExpression): number {
    const left = node.left.accept(this);
    const right = node.right.accept(this);
    
    switch (node.operator) {
      case '+': return left + right;
      case '-': return left - right;
      case '*': return left * right;
      case '/': return left / right;
      default: throw new Error(`Unknown operator: ${node.operator}`);
    }
  }
  
  visitIdentifier(node: Identifier): number {
    throw new Error(`Cannot evaluate identifier ${node.name}`);
  }
  
  visitLiteral(node: Literal): number {
    if (typeof node.value === 'number') {
      return node.value;
    }
    throw new Error(`Cannot evaluate string literal: ${node.value}`);
  }
  
  visitFunctionDeclaration(node: FunctionDeclaration): number {
    throw new Error("Cannot evaluate function declarations");
  }
}

// Посетитель для печати выражений
class ExpressionPrinter implements ASTVisitor<string> {
  visitBinaryExpression(node: BinaryExpression): string {
    return `(${node.left.accept(this)} ${node.operator} ${node.right.accept(this)})`;
  }
  
  visitIdentifier(node: Identifier): string {
    return node.name;
  }
  
  visitLiteral(node: Literal): string {
    return String(node.value);
  }
  
  visitFunctionDeclaration(node: FunctionDeclaration): string {
    const params = node.params.map(p => p.accept(this)).join(', ');
    return `function ${node.name.accept(this)}(${params}) { ${node.body.accept(this as any)} }`;
  }
}

// Использование
const ast = new BinaryExpression(
  new Literal(10),
  '+',
  new BinaryExpression(
    new Literal(5),
    '*',
    new Literal(2)
  )
);

const evaluator = new ExpressionEvaluator();
const printer = new ExpressionPrinter();

console.log(printer.visitBinaryExpression(ast)); // (10 + (5 * 2))
console.log(evaluator.visitBinaryExpression(ast)); // 20
```

## Chain of Responsibility

### Классическая цепочка
```typescript
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: string): string | null;
}

abstract class AbstractHandler implements Handler {
  private nextHandler: Handler | null = null;
  
  setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler;
  }
  
  handle(request: string): string | null {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    
    return null;
  }
}

class MonkeyHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request === 'Banana') {
      return `Monkey: I'll eat the ${request}.`;
    }
    
    return super.handle(request);
  }
}

class SquirrelHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request === 'Nut') {
      return `Squirrel: I'll eat the ${request}.`;
    }
    
    return super.handle(request);
  }
}

class DogHandler extends AbstractHandler {
  handle(request: string): string | null {
    if (request === 'MeatBall') {
      return `Dog: I'll eat the ${request}.`;
    }
    
    return super.handle(request);
  }
}

// Использование
const monkey = new MonkeyHandler();
const squirrel = new SquirrelHandler();
const dog = new DogHandler();

monkey.setNext(squirrel).setNext(dog);

console.log(monkey.handle('Nut'));
// Squirrel: I'll eat the Nut.

console.log(monkey.handle('MeatBall'));
// Dog: I'll eat the MeatBall.

console.log(monkey.handle('Banana'));
// Monkey: I'll eat the Banana.

console.log(monkey.handle('Cup of coffee'));
// null (никто не может обработать)
```

### Цепочка обработки ошибок
```typescript
interface ErrorHandler {
  setNext(handler: ErrorHandler): ErrorHandler;
  handle(error: Error): boolean; // возвращает true, если обработка завершена
}

class BaseErrorHandler implements ErrorHandler {
  protected nextHandler: ErrorHandler | null = null;
  
  setNext(handler: ErrorHandler): ErrorHandler {
    this.nextHandler = handler;
    return handler;
  }
  
  handle(error: Error): boolean {
    if (this.nextHandler) {
      return this.nextHandler.handle(error);
    }
    
    console.error('Unhandled error:', error);
    return false;
  }
}

class ValidationErrorHandler extends BaseErrorHandler {
  handle(error: Error): boolean {
    if (error.message.startsWith('Validation:')) {
      console.warn('Validation error handled:', error.message);
      // возможно, логирование или специфичная обработка
      return true; // обработка завершена
    }
    
    return super.handle(error);
  }
}

class AuthenticationErrorHandler extends BaseErrorHandler {
  handle(error: Error): boolean {
    if (error.message.includes('Authentication') || error.message.includes('Unauthorized')) {
      console.log('Redirecting to login...');
      // перенаправление на страницу логина
      return true; // обработка завершена
    }
    
    return super.handle(error);
  }
}

class NetworkErrorHandler extends BaseErrorHandler {
  handle(error: Error): boolean {
    if (error.message.includes('Network') || error.message.includes('fetch failed')) {
      console.error('Network error - check connection');
      // повторная попытка или уведомление пользователя
      return true; // обработка завершена
    }
    
    return super.handle(error);
  }
}

// Использование
const validationHandler = new ValidationErrorHandler();
const authHandler = new AuthenticationErrorHandler();
const networkHandler = new NetworkErrorHandler();
const baseHandler = new BaseErrorHandler();

validationHandler.setNext(authHandler).setNext(networkHandler).setNext(baseHandler);

validationHandler.handle(new Error('Validation: invalid email format'));
// Validation error handled: Validation: invalid email format

validationHandler.handle(new Error('Authentication failed'));
// Redirecting to login...

validationHandler.handle(new Error('Network error'));
// Network error - check connection
```

## Interpreter Pattern

### Простой интерпретатор выражений
```typescript
interface Expression {
  interpret(context: Context): number;
}

class Context {
  private variables: Map<string, number> = new Map();
  
  setValue(variable: string, value: number): void {
    this.variables.set(variable, value);
  }
  
  getValue(variable: string): number {
    return this.variables.get(variable) || 0;
  }
}

class NumberExpression implements Expression {
  constructor(private value: number) {}
  
  interpret(context: Context): number {
    return this.value;
  }
}

class VariableExpression implements Expression {
  constructor(private name: string) {}
  
  interpret(context: Context): number {
    return context.getValue(this.name);
  }
}

class AddExpression implements Expression {
  constructor(private left: Expression, private right: Expression) {}
  
  interpret(context: Context): number {
    return this.left.interpret(context) + this.right.interpret(context);
  }
}

class SubtractExpression implements Expression {
  constructor(private left: Expression, private right: Expression) {}
  
  interpret(context: Context): number {
    return this.left.interpret(context) - this.right.interpret(context);
  }
}

class MultiplyExpression implements Expression {
  constructor(private left: Expression, private right: Expression) {}
  
  interpret(context: Context): number {
    return this.left.interpret(context) * this.right.interpret(context);
  }
}

// Использование
const context = new Context();
context.setValue("x", 10);
context.setValue("y", 5);

// Выражение: (x + y) * 2
const expression = new MultiplyExpression(
  new AddExpression(
    new VariableExpression("x"),
    new VariableExpression("y")
  ),
  new NumberExpression(2)
);

console.log(expression.interpret(context)); // (10 + 5) * 2 = 30
```

### Parser для DSL
```typescript
class ExpressionParser {
  private tokens: string[];
  private position: number = 0;
  
  constructor(expression: string) {
    this.tokens = expression.match(/\d+|[+*/()-]|\w+/g) || [];
  }
  
  parse(): Expression {
    return this.parseExpression();
  }
  
  private parseExpression(): Expression {
    let left = this.parseTerm();
    
    while (this.position < this.tokens.length) {
      const token = this.tokens[this.position];
      
      if (token === '+') {
        this.position++;
        left = new AddExpression(left, this.parseTerm());
      } else if (token === '-') {
        this.position++;
        left = new SubtractExpression(left, this.parseTerm());
      } else {
        break;
      }
    }
    
    return left;
  }
  
  private parseTerm(): Expression {
    let left = this.parseFactor();
    
    while (this.position < this.tokens.length) {
      const token = this.tokens[this.position];
      
      if (token === '*') {
        this.position++;
        left = new MultiplyExpression(left, this.parseFactor());
      } else if (token === '/') {
        this.position++;
        this.position++; // пропускаем деление
        // Для упрощения пропустим реализацию деления
      } else {
        break;
      }
    }
    
    return left;
  }
  
  private parseFactor(): Expression {
    const token = this.tokens[this.position];
    
    if (!isNaN(parseInt(token))) { // число
      this.position++;
      return new NumberExpression(parseInt(token));
    } else if (token === '(') {
      this.position++;
      const expression = this.parseExpression();
      this.position++; // пропускаем ')'
      return expression;
    } else { // переменная
      this.position++;
      return new VariableExpression(token);
    }
  }
}

// Использование парсера
const parser = new ExpressionParser("(x + y) * 2");
const expression = parser.parse();

const context2 = new Context();
context2.setValue("x", 10);
context2.setValue("y", 5);

console.log(expression.interpret(context2)); // 30
```

## Лучшие практики

### 1. Используйте функциональные альтернативы при возможности
```typescript
// Вместо сложной иерархии классов Observer
type Observer<T> = (value: T) => void;

class EventEmitter<T> {
  private observers: Observer<T>[] = [];
  
  subscribe(observer: Observer<T>): () => void {
    this.observers.push(observer);
    
    // Возвращаем функцию отписки
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  emit(value: T): void {
    this.observers.forEach(observer => observer(value));
  }
}
```

### 2. Используйте обобщения для универсальных паттернов
```typescript
// Универсальный менеджер команд
interface Command<T> {
  execute(payload: T): void;
  undo?(): void;
}

class CommandManager<T> {
  private history: Command<T>[] = [];
  private currentPosition = -1;
  
  execute(command: Command<T>, payload: T): void {
    command.execute(payload);
    this.history.push(command);
    this.currentPosition++;
  }
  
  undo(): void {
    if (this.currentPosition >= 0) {
      const command = this.history[this.currentPosition];
      if (command.undo) {
        command.undo();
      }
      this.currentPosition--;
    }
  }
}
```

### 3. Используйте типы для безопасности
```typescript
// Использование литеральных типов для ограничения состояний
type GameState = 'menu' | 'playing' | 'paused' | 'game-over';

interface GameStateHandler {
  handle(currentState: GameState, nextState: GameState): GameState;
}

class GameStateManager {
  private state: GameState = 'menu';
  private handlers: Map<GameState, GameStateHandler> = new Map();
  
  registerHandler(state: GameState, handler: GameStateHandler): void {
    this.handlers.set(state, handler);
  }
  
  transition(newState: GameState): void {
    const handler = this.handlers.get(this.state);
    if (handler) {
      this.state = handler.handle(this.state, newState);
    }
  }
}
```

## Современные тенденции

### Использование Proxy для поведенческих паттернов
```typescript
// Использование Proxy для реализации Command pattern
interface CommandObject {
  execute(): void;
  undo?(): void;
}

class CommandProxy {
  private executedCommands: CommandObject[] = [];
  
  createCommand<T extends Record<string, any>>(
    target: T,
    methodName: keyof T
  ): T[keyof T] extends (...args: any[]) => any ? Function : never {
    return new Proxy(target[methodName], {
      apply: (targetFn, thisArg, args) => {
        // Создаем команду
        const command: CommandObject = {
          execute: () => (thisArg[methodName] as Function).apply(thisArg, args),
          undo: () => {
            // реализация отмены, если доступна
            console.log("Undo not implemented for this command");
          }
        };
        
        command.execute();
        this.executedCommands.push(command);
        
        return (thisArg[methodName] as Function).apply(thisArg, args);
      }
    }) as any;
  }
  
  undoLast(): void {
    if (this.executedCommands.length > 0) {
      const lastCommand = this.executedCommands.pop();
      if (lastCommand && lastCommand.undo) {
        lastCommand.undo();
      }
    }
  }
}
```

## Ограничения и подводные камни

### 1. Сложность диагностики
```typescript
// Паттерны могут усложнить отладку
// Рекомендуется добавлять логирование и типобезопасность
```

### 2. Производительность
```typescript
// Некоторые паттерны могут добавлять оверхед
// Особенно цепочки ответственности и посетители
// Оптимизируйте при необходимости
```

## Заключение

Поведенческие паттерны в TypeScript:

1. **Улучшают организацию взаимодействия** между объектами
2. **Повышают гибкость архитектуры** за счет слабой связанности
3. **Обеспечивают повторное использование кода** через стандартные шаблоны
4. **Упрощают расширение функциональности** за счет четких интерфейсов

Каждый паттерн решает конкретную проблему взаимодействия и должен выбираться в зависимости от требований архитектуры системы.

## Связь с другими концепциями
- [[../design-patterns/structural-patterns]] - структурные паттерны для организации объектов
- [[../design-patterns/creational-patterns]] - порождающие паттерны для создания объектов
- [[Функциональное программирование в TypeScript]] - функциональные альтернативы поведенческим паттернам
- [[../advanced-types/conditional-types]] - усложненные типы для паттернов
- [[Модули в TypeScript]] - организация паттернов в модульные системы