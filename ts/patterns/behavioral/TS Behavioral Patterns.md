# Behavioral Patterns

Поведенческие паттерны (Behavioral Patterns) определяют способы эффективного взаимодействия и распределения ответственности между объектами. Они фокусируются на коммуникации между объектами и на том, как они выполняют свою работу совместно.

## Основные поведенческие паттерны

### [Observer Pattern](observer.md)
Определяет зависимость "один-ко-многим" между объектами, чтобы при изменении состояния одного объекта все зависящие от него объекты уведомлялись автоматически.

### [Strategy Pattern](strategy.md)
Позволяет определить семейство алгоритмов, инкапсулировать каждый из них и сделать их взаимозаменяемыми.

### [Command Pattern](command.md)
Инкапсулирует запрос как объект, позволяя параметризовать клиентов различными запросами, ставить запросы в очередь и поддерживать отмену/повтор операций.

### [State Pattern](state.md)
Позволяет объекту изменять свое поведение в зависимости от внутреннего состояния.

### [Iterator Pattern](iterator.md)
Предоставляет способ последовательного доступа к элементам составного объекта без раскрытия его внутреннего представления.

### [Mediator Pattern](mediator.md)
Определяет объект, инкапсулирующий способ взаимодействия между объектами.

### [Memento Pattern](memento.md)
Позволяет зафиксировать и восстановить внутреннее состояние объекта без нарушения принципа инкапсуляции.

### [Visitor Pattern](visitor.md)
Позволяет определить новую операцию без изменения классов объектов, над которыми она выполняется.

### [Template Method Pattern](template-method.md)
Определяет скелет алгоритма в базовом классе, позволяя подклассам переопределять некоторые шаги алгоритма.

### [Chain of Responsibility Pattern](chain-of-responsibility.md)
Передает запросы последовательно по цепочке обработчиков, позволяя одному из них обработать запрос.

### [Interpreter Pattern](interpreter.md)
Определяет грамматику языка и интерпретатор для этой грамматики.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │       Behavioral Design Patterns                        │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Observer      │               │      Strategy            │                                    │    Command      │
  │  Pattern        │               │  Pattern               │                                    │  Pattern        │
  │                 │               │                          │                                    │                 │
  │ - Publisher/    │◄──────────────┤ - Algorithm Families     │─────────────────────────────────────►│ - Encapsulated  │
  │   Subscriber    │               │ - Replaceable Algorithms │                                    │   Actions       │
  │ - Notifications │               │ - Context Pattern        │                                    │ - Undo/Redo     │
  │ - Reactive      │               │ - Runtime Switching      │                                    │ - Transactional │
  │   Programming   │               │ - Open/Closed Principle  │                                    │   Operations    │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │           State Pattern         │
        │                           │                               │
        │                           │ - Internal State Management     │
        │                           │ - Behavior Changes by State     │
        │                           │ - Finite State Machine          │
        │                           │ - State Transitions             │
        │                           │ - Context Pattern               │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                        Iterator Pattern                          │
        │                           │                                                                 │
        │                           │ - Sequential Access                                               │
        │                           │ - Collection Traversal                                            │
        │                           │ - Internal vs External Iteration                                  │
        │                           │ - Iterator Protocol                                               │
        │                           │ - Generator Functions                                             │
        └───────────────────────────┤ - Lazy Evaluation                                               │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Observer Pattern

### Реализация Observer
```typescript
// Интерфейс наблюдателя
interface Observer<T> {
  update(data: T): void;
}

// Интерфейс субъекта
interface Subject<T> {
  subscribe(observer: Observer<T>): void;
  unsubscribe(observer: Observer<T>): void;
  notify(data: T): void;
}

// Конкретный субъект
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
  
  setNews(news: string): void {
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

agency.setNews("Breaking: TypeScript 5.0 released!");
// Output:
// CNN received news: Breaking: TypeScript 5.0 released!
// BBC received news: Breaking: TypeScript 5.0 released!
```

### Функциональная реализация Observer
```typescript
// Функциональная реализация системы наблюдения
type Observer<T> = (data: T) => void;
type Unsubscribe = () => void;

class Observable<T> {
  private observers: Observer<T>[] = [];
  
  subscribe(observer: Observer<T>): Unsubscribe {
    this.observers.push(observer);
    
    // Возвращаем функцию отписки
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  notify(data: T): void {
    // Создаем копию массива, чтобы избежать проблем с изменением во время вызова
    [...this.observers].forEach(observer => observer(data));
  }
  
  // Дополнительные операции
  map<U>(fn: (value: T) => U): Observable<U> {
    const newObservable = new Observable<U>();
    this.subscribe(value => newObservable.notify(fn(value)));
    return newObservable;
  }
  
  filter(predicate: (value: T) => boolean): Observable<T> {
    const filteredObservable = new Observable<T>();
    this.subscribe(value => {
      if (predicate(value)) {
        filteredObservable.notify(value);
      }
    });
    return filteredObservable;
  }
}

// Использование
const observable = new Observable<string>();

const unsubscribe1 = observable.subscribe(data => console.log("Observer 1:", data));
const unsubscribe2 = observable.subscribe(data => console.log("Observer 2:", data.toUpperCase()));

observable.notify("Hello"); // "Observer 1: Hello", "Observer 2: HELLO"

unsubscribe1();
observable.notify("World"); // "Observer 2: WORLD"
```

## Strategy Pattern

### Классическая реализация Strategy
```typescript
// Интерфейс стратегии
interface SortingStrategy {
  sort(numbers: number[]): number[];
}

// Конкретные стратегии
class BubbleSortStrategy implements SortingStrategy {
  sort(numbers: number[]): number[] {
    const result = [...numbers]; // создаем копию
    const n = result.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (result[j] > result[j + 1]) {
          [result[j], result[j + 1]] = [result[j + 1], result[j]];
        }
      }
    }
    
    return result;
  }
}

class QuickSortStrategy implements SortingStrategy {
  sort(numbers: number[]): number[] {
    if (numbers.length <= 1) {
      return [...numbers];
    }
    
    const pivot = numbers[Math.floor(numbers.length / 2)];
    const less: number[] = [];
    const equal: number[] = [];
    const greater: number[] = [];
    
    for (const num of numbers) {
      if (num < pivot) {
        less.push(num);
      } else if (num > pivot) {
        greater.push(num);
      } else {
        equal.push(num);
      }
    }
    
    return [
      ...this.sort(less),
      ...equal,
      ...this.sort(greater)
    ];
  }
}

class MergeSortStrategy implements SortingStrategy {
  sort(numbers: number[]): number[] {
    if (numbers.length <= 1) {
      return [...numbers];
    }
    
    const mid = Math.floor(numbers.length / 2);
    const left = this.sort(numbers.slice(0, mid));
    const right = this.sort(numbers.slice(mid));
    
    return this.merge(left, right);
  }
  
  private merge(left: number[], right: number[]): number[] {
    const result: number[] = [];
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

// Контекст стратегии
class Sorter {
  constructor(private strategy: SortingStrategy) {}
  
  setStrategy(strategy: SortingStrategy): void {
    this.strategy = strategy;
  }
  
  sort(numbers: number[]): number[] {
    return this.strategy.sort(numbers);
  }
}

// Использование
const data = [64, 34, 25, 12, 22, 11, 90];

const bubbleSorter = new Sorter(new BubbleSortStrategy());
const quickSorter = new Sorter(new QuickSortStrategy());
const mergeSorter = new Sorter(new MergeSortStrategy());

console.log("Bubble sort:", bubbleSorter.sort(data));
console.log("Quick sort:", quickSorter.sort(data));
console.log("Merge sort:", mergeSorter.sort(data));

// Динамическое изменение стратегии
const sorter = new Sorter(new BubbleSortStrategy());
if (data.length > 1000) {
  sorter.setStrategy(new QuickSortStrategy());
}
```

### Функциональная стратегия
```typescript
// Функциональный подход к стратегии
type SortingFunction = (numbers: number[]) => number[];

const bubbleSort: SortingFunction = (arr) => {
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

const insertionSort: SortingFunction = (arr) => {
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

// Контекст для функциональных стратегий
class FunctionalSorter {
  constructor(private strategy: SortingFunction) {}
  
  setStrategy(strategy: SortingFunction): void {
    this.strategy = strategy;
  }
  
  sort(numbers: number[]): number[] {
    return this.strategy(numbers);
  }
}

const functionalSorter = new FunctionalSorter(bubbleSort);
console.log(functionalSorter.sort([5, 2, 8, 1, 9]));

functionalSorter.setStrategy(insertionSort);
console.log(functionalSorter.sort([5, 2, 8, 1, 9]));
```

## Command Pattern

### Классическая реализация Command
```typescript
// Интерфейс команды
interface Command {
  execute(): void;
  undo?(): void;
  redo?(): void;
}

// Receiver (получатель команды)
class Calculator {
  private currentValue: number = 0;
  
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
    if (value === 0) {
      throw new Error("Division by zero");
    }
    this.currentValue /= value;
  }
  
  getValue(): number {
    return this.currentValue;
  }
  
  setValue(value: number): void {
    this.currentValue = value;
  }
}

// Конкретные команды
class AddCommand implements Command {
  private previousValue: number;
  
  constructor(private calculator: Calculator, private value: number) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.add(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

class SubtractCommand implements Command {
  private previousValue: number;
  
  constructor(private calculator: Calculator, private value: number) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.subtract(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

class MultiplyCommand implements Command {
  private previousValue: number;
  
  constructor(private calculator: Calculator, private value: number) {}
  
  execute(): void {
    this.previousValue = this.calculator.getValue();
    this.calculator.multiply(this.value);
  }
  
  undo(): void {
    this.calculator.setValue(this.previousValue);
  }
}

// Менеджер команд для отмены/повтора
class CommandManager {
  private history: Command[] = [];
  private currentCommandIndex: number = -1;
  
  execute(command: Command): void {
    // Удаляем команды после текущей для отмены
    this.history = this.history.slice(0, this.currentCommandIndex + 1);
    
    command.execute();
    this.history.push(command);
    this.currentCommandIndex++;
  }
  
  undo(): void {
    if (this.currentCommandIndex >= 0) {
      const command = this.history[this.currentCommandIndex];
      if (command.undo) {
        command.undo();
      }
      this.currentCommandIndex--;
    }
  }
  
  redo(): void {
    if (this.currentCommandIndex < this.history.length - 1) {
      this.currentCommandIndex++;
      const command = this.history[this.currentCommandIndex];
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

console.log(calculator.getValue()); // 0

commandManager.execute(addCommand);
console.log(calculator.getValue()); // 10

commandManager.execute(multiplyCommand);
console.log(calculator.getValue()); // 20

commandManager.execute(subtractCommand);
console.log(calculator.getValue()); // 15

// Отмена последней команды
commandManager.undo();
console.log(calculator.getValue()); // 20

// Повтор последней команды
commandManager.redo();
console.log(calculator.getValue()); // 15
```

### Макрос команд
```typescript
// Команда-макрос для группировки нескольких команд
class MacroCommand implements Command {
  private commands: Command[] = [];
  
  add(command: Command): void {
    this.commands.push(command);
  }
  
  remove(command: Command): void {
    const index = this.commands.indexOf(command);
    if (index > -1) {
      this.commands.splice(index, 1);
    }
  }
  
  execute(): void {
    for (const command of this.commands) {
      command.execute();
    }
  }
  
  undo(): void {
    // Отмена в обратном порядке
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

commandManager.execute(macro);
console.log(calculator.getValue()); // (0 + 5) * 3 - 2 = 13
```

## State Pattern

### Классическая реализация State
```typescript
// Интерфейс состояния
interface State {
  handle(context: Context): void;
  toString(): string;
}

// Контекст
class Context {
  private state: State;
  
  constructor(initialState: State) {
    this.state = initialState;
  }
  
  setState(state: State): void {
    this.state = state;
    console.log(`Context state changed to: ${state.toString()}`);
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

// Использование
const context = new Context(new ConcreteStateA());
console.log(context.getState().toString()); // "State A"

context.request(); // "Handling in State A" -> change to State B
console.log(context.getState().toString()); // "State B"

context.request(); // "Handling in State B" -> change to State A
console.log(context.getState().toString()); // "State A"
```

### Пример игрового персонажа
```typescript
// Интерфейс состояния персонажа
interface CharacterState {
  move(character: GameCharacter): void;
  attack(character: GameCharacter): void;
  takeDamage(character: GameCharacter, damage: number): void;
  toString(): string;
}

// Игровой персонаж
class GameCharacter {
  private state: CharacterState;
  public health: number = 100;
  public position: { x: number; y: number } = { x: 0, y: 0 };
  
  constructor(initialState: CharacterState) {
    this.state = initialState;
  }
  
  setState(state: CharacterState): void {
    this.state = state;
  }
  
  move(): void {
    this.state.move(this);
  }
  
  attack(): void {
    this.state.attack(this);
  }
  
  takeDamage(damage: number): void {
    this.state.takeDamage(this, damage);
  }
  
  getState(): string {
    return this.state.toString();
  }
}

// Конкретные состояния
class StandingState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Character starts walking...");
    character.setState(new WalkingState());
  }
  
  attack(character: GameCharacter): void {
    console.log("Character swings sword while standing");
  }
  
  takeDamage(character: GameCharacter, damage: number): void {
    console.log(`Character takes ${damage} damage while standing`);
    character.health -= damage;
    if (character.health <= 0) {
      character.setState(new DeadState());
    }
  }
  
  toString(): string {
    return "Standing";
  }
}

class WalkingState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Character continues walking...");
    // Меняет позицию
    character.position.x += 1;
  }
  
  attack(character: GameCharacter): void {
    console.log("Character cannot attack while walking");
    character.setState(new StandingState());
  }
  
  takeDamage(character: GameCharacter, damage: number): void {
    console.log(`Character takes ${damage} damage while walking`);
    character.health -= damage * 1.5; // Бонус к урону при движении
    if (character.health <= 0) {
      character.setState(new DeadState());
    } else {
      character.setState(new StunnedState());
    }
  }
  
  toString(): string {
    return "Walking";
  }
}

class DeadState implements CharacterState {
  move(character: GameCharacter): void {
    console.log("Dead characters cannot move");
  }
  
  attack(character: GameCharacter): void {
    console.log("Dead characters cannot attack");
  }
  
  takeDamage(character: GameCharacter, damage: number): void {
    console.log("Dead characters take no additional damage");
  }
  
  toString(): string {
    return "Dead";
  }
}

class StunnedState implements CharacterState {
  private stunDuration: number = 2;
  
  move(character: GameCharacter): void {
    console.log("Stunned character cannot move");
    this.stunDuration--;
    if (this.stunDuration <= 0) {
      character.setState(new StandingState());
    }
  }
  
  attack(character: GameCharacter): void {
    console.log("Stunned character cannot attack");
    this.stunDuration--;
    if (this.stunDuration <= 0) {
      character.setState(new StandingState());
    }
  }
  
  takeDamage(character: GameCharacter, damage: number): void {
    console.log(`Stunned character takes ${damage} damage`);
    character.health -= damage;
    if (character.health <= 0) {
      character.setState(new DeadState());
    } else {
      character.setState(new WalkingState());
    }
  }
  
  toString(): string {
    return `Stunned (${this.stunDuration} turns left)`;
  }
}

// Использование
const character = new GameCharacter(new StandingState());

console.log(`State: ${character.getState()}`); // "Standing"
character.attack(); // "Character swings sword while standing"
character.move();   // "Character starts walking..." -> State: "Walking"
character.attack(); // "Character cannot attack while walking" -> State: "Standing"
```

## Iterator Pattern

### Классическая реализация Iterator
```typescript
// Интерфейс итератора
interface Iterator<T> {
  next(): { value: T; done: boolean };
  return?(value?: any): { value: any; done: boolean };
  throw?(e?: any): { value: any; done: boolean };
}

// Интерфейс итерируемого объекта
interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>;
}

// Коллекция с итератором
class NumberRange implements Iterable<number> {
  constructor(private start: number, private end: number, private step: number = 1) {}
  
  *[Symbol.iterator](): Iterator<number> {
    let current = this.start;
    while (current < this.end) {
      yield current;
      current += this.step;
    }
  }
  
  // Альтернативная реализация без генераторов
  createIterator(): Iterator<number> {
    let current = this.start;
    const end = this.end;
    const step = this.step;
    
    return {
      next(): { value: number; done: boolean } {
        if (current < end) {
          const value = current;
          current += step;
          return { value, done: false };
        } else {
          return { value: undefined!, done: true };
        }
      }
    };
  }
}

// Использование
const range = new NumberRange(1, 10, 2);

// С использованием цикла for...of
for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}

// С использованием итератора
const iterator = range[Symbol.iterator]();
let result = iterator.next();
while (!result.done) {
  console.log(result.value); // 1, 3, 5, 7, 9
  result = iterator.next();
}
```

### Композитный итератор
```typescript
// Итерируемый композитный элемент
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
  [Symbol.iterator](): Iterator<TreeNode<T>>;
}

class TreeIterator<T> implements Iterator<TreeNode<T>> {
  private stack: TreeNode<T>[] = [];
  
  constructor(root: TreeNode<T>) {
    this.stack.push(root);
  }
  
  next(): { value: TreeNode<T>; done: boolean } {
    if (this.stack.length === 0) {
      return { value: undefined!, done: true };
    }
    
    const node = this.stack.shift()!;
    // Добавляем детей в начало стека для обхода в ширину
    node.children.forEach(child => this.stack.push(child));
    
    return { value: node, done: false };
  }
}

class TreeNodeImpl<T> implements TreeNode<T> {
  children: TreeNodeImpl<T>[] = [];
  
  constructor(public value: T) {}
  
  addChild(child: TreeNodeImpl<T>): void {
    this.children.push(child);
  }
  
  [Symbol.iterator](): Iterator<TreeNode<T>> {
    return new TreeIterator(this) as Iterator<TreeNode<T>>;
  }
}

// Использование
const root = new TreeNodeImpl("root");
const child1 = new TreeNodeImpl("child1");
const child2 = new TreeNodeImpl("child2");
const grandchild = new TreeNodeImpl("grandchild");

root.addChild(child1);
root.addChild(child2);
child1.addChild(grandchild);

// Обход дерева
for (const node of root) {
  console.log(node.value);
  // "root", "child1", "child2", "grandchild"
}
```

## Mediator Pattern

### Классическая реализация Mediator
```typescript
// Интерфейс коллеги
interface Colleague {
  send(message: string, colleague?: Colleague): void;
  receive(message: string): void;
}

// Интерфейс медиатора
interface Mediator {
  register(colleague: Colleague): void;
  send(message: string, sender: Colleague): void;
}

// Конкретный медиатор
class ChatRoom implements Mediator {
  private colleagues: Colleague[] = [];
  
  register(colleague: Colleague): void {
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

// Конкретные коллеги
class Participant implements Colleague {
  constructor(private name: string, private mediator: Mediator) {
    this.mediator.register(this);
  }
  
  send(message: string, target?: Colleague): void {
    if (target) {
      // Прямое сообщение (если поддерживается)
      target.receive(`${this.name} to you: ${message}`);
    } else {
      // Сообщение всем через медиатор
      this.mediator.send(`${this.name} says: ${message}`, this);
    }
  }
  
  receive(message: string): void {
    console.log(`${this.name} received: ${message}`);
  }
}

// Использование
const chatRoom = new ChatRoom();

const alice = new Participant("Alice", chatRoom);
const bob = new Participant("Bob", chatRoom);
const charlie = new Participant("Charlie", chatRoom);

alice.send("Hello everyone!"); 
// Bob received: Alice says: Hello everyone!
// Charlie received: Alice says: Hello everyone!

bob.send("Hi Alice!");
// Alice received: Bob says: Hi Alice!
// Charlie received: Bob says: Hi Alice!
```

### Медиатор с типизацией событий
```typescript
// Медиатор с типизацией событий
interface Message {
  type: string;
  payload: any;
  senderId: string;
}

interface MessageHandler<T extends Message> {
  handle(message: T): void;
}

class TypedMediator {
  private handlers: Map<string, Array<MessageHandler<any>>> = new Map();
  private participants: Map<string, (message: Message) => void> = new Map();
  
  registerHandler<T extends Message>(type: string, handler: MessageHandler<T>): void {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, []);
    }
    this.handlers.get(type)!.push(handler as MessageHandler<any>);
  }
  
  registerParticipant(id: string, handler: (message: Message) => void): void {
    this.participants.set(id, handler);
  }
  
  broadcast(message: Message): void {
    const handlers = this.handlers.get(message.type);
    if (handlers) {
      handlers.forEach(handler => handler.handle(message));
    }
  }
  
  sendTo(participantId: string, message: Message): boolean {
    const participant = this.participants.get(participantId);
    if (participant) {
      participant(message);
      return true;
    }
    return false;
  }
}

// Использование с конкретными типами сообщений
interface UserJoinedMessage extends Message {
  type: 'user-joined';
  payload: {
    userId: string;
    userName: string;
  };
}

interface ChatMessage extends Message {
  type: 'chat-message';
  payload: {
    userId: string;
    text: string;
    timestamp: Date;
  };
}

class UserJoinedHandler implements MessageHandler<UserJoinedMessage> {
  handle(message: UserJoinedMessage): void {
    console.log(`User ${message.payload.userName} joined the chat`);
  }
}

class ChatMessageHandler implements MessageHandler<ChatMessage> {
  handle(message: ChatMessage): void {
    console.log(`[${message.payload.timestamp}] ${message.payload.userId}: ${message.payload.text}`);
  }
}

// Регистрация обработчиков
const mediator = new TypedMediator();
mediator.registerHandler('user-joined', new UserJoinedHandler());
mediator.registerHandler('chat-message', new ChatMessageHandler());

// Отправка сообщений
const joinMessage: UserJoinedMessage = {
  type: 'user-joined',
  payload: { userId: '1', userName: 'Alice' },
  senderId: 'system'
};

const chatMessage: ChatMessage = {
  type: 'chat-message',
  payload: { 
    userId: '1', 
    text: 'Hello everyone!', 
    timestamp: new Date() 
  },
  senderId: '1'
};

mediator.broadcast(joinMessage);
mediator.broadcast(chatMessage);
```

## Memento Pattern

### Классическая реализация Memento
```typescript
// Memento (хранитель состояния)
interface Memento {
  getState(): any;
  getName(): string;
  getDate(): Date;
}

class ConcreteMemento implements Memento {
  private date: Date;
  
  constructor(private state: any, private name: string = "") {
    this.date = new Date();
  }
  
  getState(): any {
    return this.state;
  }
  
  getName(): string {
    return `${this.name || "Memento"}_${this.date.toISOString()}`;
  }
  
  getDate(): Date {
    return this.date;
  }
}

// Originator (создатель состояния)
class Editor {
  private state: any;
  
  constructor(state: any) {
    this.state = state;
    console.log(`Editor: Initial state set to: ${this.state}`);
  }
  
  setState(state: any): void {
    console.log(`Editor: Setting state to: ${state}`);
    this.state = state;
  }
  
  getState(): any {
    return this.state;
  }
  
  save(): Memento {
    console.log(`Editor: Saving state to Memento`);
    return new ConcreteMemento(this.state);
  }
  
  restore(memento: Memento): void {
    this.state = memento.getState();
    console.log(`Editor: State after restoring: ${this.state}`);
  }
}

// Caretaker (хранитель)
class HistoryCaretaker {
  private mementos: Memento[] = [];
  
  addMemento(memento: Memento): void {
    this.mementos.push(memento);
  }
  
  getMemento(index: number): Memento {
    return this.mementos[index];
  }
  
  removeMemento(index: number): void {
    this.mementos.splice(index, 1);
  }
  
  getHistory(): string[] {
    return this.mementos.map(memento => memento.getName());
  }
  
  getLength(): number {
    return this.mementos.length;
  }
  
  clear(): void {
    this.mementos = [];
  }
}

// Использование
const editor = new Editor("Initial text");
const caretaker = new HistoryCaretaker();

// Сохраняем состояние
caretaker.addMemento(editor.save());

editor.setState("First changes");
caretaker.addMemento(editor.save());

editor.setState("Second changes");
caretaker.addMemento(editor.save());

console.log("Available mementos:");
console.log(caretaker.getHistory());

// Восстановление до первого состояния
editor.restore(caretaker.getMemento(0));
console.log("Current state:", editor.getState()); // "Initial text"

// Восстановление до второго состояния
editor.restore(caretaker.getMemento(1));
console.log("Current state:", editor.getState()); // "First changes"
```

### Состояние с обобщениями
```typescript
// Обобщенный Memento
interface GenericMemento<T> {
  getState(): T;
}

class GenericOriginator<T> {
  private state: T;
  
  constructor(state: T) {
    this.state = state;
  }
  
  setState(state: T): void {
    this.state = state;
  }
  
  getState(): T {
    return this.state;
  }
  
  save(): GenericMemento<T> {
    return new GenericMementoImpl<T>(JSON.parse(JSON.stringify(this.state)));
  }
  
  restore(memento: GenericMemento<T>): void {
    this.state = memento.getState();
  }
}

class GenericMementoImpl<T> implements GenericMemento<T> {
  constructor(private state: T) {}
  
  getState(): T {
    return this.state;
  }
  
  setName(name: string): void {
    // можно добавить метаданные
  }
}

// Использование
interface AppState {
  user: { id: number; name: string };
  theme: string;
  preferences: Record<string, boolean>;
}

const initialState: AppState = {
  user: { id: 1, name: "Alice" },
  theme: "light",
  preferences: { notifications: true, darkMode: false }
};

const originator = new GenericOriginator<AppState>(initialState);

const snapshot1 = originator.save(); // сохраняем начальное состояние

originator.setState({
  ...initialState,
  theme: "dark",
  preferences: { ...initialState.preferences, darkMode: true }
});

const snapshot2 = originator.save(); // сохраняем темное состояние

// возврат к светлой теме
originator.restore(snapshot1);
console.log(originator.getState().theme); // "light"
```

## Visitor Pattern

### Классическая реализация Visitor
```typescript
// Интерфейс элемента
interface Element {
  accept(visitor: Visitor): void;
}

// Интерфейс посетителя
interface Visitor {
  visitConcreteElementA(element: ConcreteElementA): void;
  visitConcreteElementB(element: ConcreteElementB): void;
}

// Конкретные элементы
class ConcreteElementA implements Element {
  public name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  accept(visitor: Visitor): void {
    visitor.visitConcreteElementA(this);
  }
  
  operationA(): string {
    return `Element A: ${this.name}`;
  }
}

class ConcreteElementB implements Element {
  public name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  accept(visitor: Visitor): void {
    visitor.visitConcreteElementB(this);
  }
  
  operationB(): string {
    return `Element B: ${this.name}`;
  }
}

// Конкретный посетитель
class ConcreteVisitor implements Visitor {
  visitConcreteElementA(element: ConcreteElementA): void {
    console.log(`ConcreteVisitor: ${element.operationA()}`);
  }
  
  visitConcreteElementB(element: ConcreteElementB): void {
    console.log(`ConcreteVisitor: ${element.operationB()}`);
  }
}

// Еще один посетитель
class AnotherVisitor implements Visitor {
  visitConcreteElementA(element: ConcreteElementA): void {
    console.log(`AnotherVisitor: Processing ${element.operationA()}`);
  }
  
  visitConcreteElementB(element: ConcreteElementB): void {
    console.log(`AnotherVisitor: Processing ${element.operationB()}`);
  }
}

// Использование
const elements: Element[] = [new ConcreteElementA("Alpha"), new ConcreteElementB("Beta")];
const concreteVisitor = new ConcreteVisitor();
const anotherVisitor = new AnotherVisitor();

console.log("ConcreteVisitor:");
elements.forEach(element => element.accept(concreteVisitor));

console.log("\nAnotherVisitor:");
elements.forEach(element => element.accept(anotherVisitor));
```

### Паттерн для анализа AST
```typescript
// Пример использования Visitor для анализа выражений
interface Expression {
  accept(visitor: ExpressionVisitor): any;
}

interface ExpressionVisitor {
  visitBinary(expr: BinaryExpression): any;
  visitUnary(expr: UnaryExpression): any;
  visitNumber(expr: NumberExpression): any;
  visitVariable(expr: VariableExpression): any;
}

class BinaryExpression implements Expression {
  constructor(
    public left: Expression,
    public operator: string,
    public right: Expression
  ) {}
  
  accept(visitor: ExpressionVisitor): any {
    return visitor.visitBinary(this);
  }
}

class UnaryExpression implements Expression {
  constructor(
    public operator: string,
    public operand: Expression
  ) {}
  
  accept(visitor: ExpressionVisitor): any {
    return visitor.visitUnary(this);
  }
}

class NumberExpression implements Expression {
  constructor(public value: number) {}
  
  accept(visitor: ExpressionVisitor): any {
    return visitor.visitNumber(this);
  }
}

class VariableExpression implements Expression {
  constructor(public name: string) {}
  
  accept(visitor: ExpressionVisitor): any {
    return visitor.visitVariable(this);
  }
}

// Постетитель для вычисления выражений
class ExpressionEvaluator implements ExpressionVisitor {
  visitBinary(expr: BinaryExpression): number {
    const left = expr.left.accept(this) as number;
    const right = expr.right.accept(this) as number;
    
    switch (expr.operator) {
      case '+': return left + right;
      case '-': return left - right;
      case '*': return left * right;
      case '/': return left / right;
      default: throw new Error(`Unknown operator: ${expr.operator}`);
    }
  }
  
  visitUnary(expr: UnaryExpression): number {
    const operand = expr.operand.accept(this) as number;
    
    switch (expr.operator) {
      case '+': return operand;
      case '-': return -operand;
      default: throw new Error(`Unknown unary operator: ${expr.operator}`);
    }
  }
  
  visitNumber(expr: NumberExpression): number {
    return expr.value;
  }
  
  visitVariable(_expr: VariableExpression): number {
    throw new Error("Variables not supported in evaluation");
  }
}

// Постетитель для печати выражений
class ExpressionPrinter implements ExpressionVisitor {
  visitBinary(expr: BinaryExpression): string {
    return `(${expr.left.accept(this)} ${expr.operator} ${expr.right.accept(this)})`;
  }
  
  visitUnary(expr: UnaryExpression): string {
    return `(${expr.operator}${expr.operand.accept(this)})`;
  }
  
  visitNumber(expr: NumberExpression): string {
    return expr.value.toString();
  }
  
  visitVariable(expr: VariableVariableExpression): string {
    return expr.name;
  }
}

// Использование
const expression = new BinaryExpression(
  new NumberExpression(5),
  '+',
  new BinaryExpression(
    new NumberExpression(3),
    '*',
    new NumberExpression(2)
  )
);

const evaluator = new ExpressionEvaluator();
const printer = new ExpressionPrinter();

console.log(printer.visitBinary(expression)); // "(5 + (3 * 2))"
console.log(evaluator.visitBinary(expression)); // 11
```

## Chain of Responsibility Pattern

### Классическая реализация Chain of Responsibility
```typescript
// Обработчик запроса
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

// Конкретные обработчики
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
// null (никто не может обработать запрос)
```

### Цепочка обработки ошибок
```typescript
// Цепочка обработки ошибок
interface ErrorHandler {
  setNext(handler: ErrorHandler): ErrorHandler;
  handle(error: Error): boolean; // возвращает true, если ошибка обработана
}

abstract class BaseErrorHandler implements ErrorHandler {
  protected nextHandler: ErrorHandler | null = null;
  
  setNext(handler: ErrorHandler): ErrorHandler {
    this.nextHandler = handler;
    return handler;
  }
  
  handle(error: Error): boolean {
    if (this.canHandle(error)) {
      this.processError(error);
      return true;
    }
    
    if (this.nextHandler) {
      return this.nextHandler.handle(error);
    }
    
    return false; // ошибка не обработана
  }
  
  protected abstract canHandle(error: Error): boolean;
  protected abstract processError(error: Error): void;
}

class ValidationErrorHandler extends BaseErrorHandler {
  protected canHandle(error: Error): boolean {
    return error.message.startsWith('Validation Error:');
  }
  
  protected processError(error: Error): void {
    console.warn('Validation error:', error.message);
    // отправка ошибки валидации в соответствующую систему
  }
}

class AuthenticationErrorHandler extends BaseErrorHandler {
  protected canHandle(error: Error): boolean {
    return error.message.includes('Authentication') || error.message.includes('Unauthorized');
  }
  
  protected processError(error: Error): void {
    console.error('Authentication error:', error.message);
    // перенаправление на страницу входа
  }
}

class NetworkErrorHandler extends BaseErrorHandler {
  protected canHandle(error: Error): boolean {
    return error.message.includes('Network') || error.message.includes('fetch failed');
  }
  
  protected processError(error: Error): void {
    console.error('Network error:', error.message);
    // повторная попытка или уведомление пользователя о проблеме
  }
}

class DefaultErrorHandler extends BaseErrorHandler {
  protected canHandle(_error: Error): boolean {
    return true; // всегда может обработать - последний в цепочке
  }
  
  protected processError(error: Error): void {
    console.error('Unhandled error:', error.message, error.stack);
    // отправка в систему мониторинга
  }
}

// Создание цепочки
const validationHandler = new ValidationErrorHandler();
const authHandler = new AuthenticationErrorHandler();
const networkHandler = new NetworkErrorHandler();
const defaultHandler = new DefaultErrorHandler();

validationHandler.setNext(authHandler).setNext(networkHandler).setNext(defaultHandler);

// Использование
const errors = [
  new Error('Validation Error: Invalid email format'),
  new Error('Unauthorized: Token expired'),
  new Error('Network error: Failed to fetch'),
  new Error('Unknown error: Something went wrong')
];

errors.forEach(error => {
  const handled = validationHandler.handle(error);
  if (!handled) {
    console.error('Error not handled:', error.message);
  }
});
```

## Template Method Pattern

### Классическая реализация Template Method
```typescript
// Абстрактный класс с шаблонным методом
abstract class DataProcessor {
  // Шаблонный метод (не должен быть переопределен)
  process(dataSource: string): void {
    this.loadData(dataSource);
    this.transformData();
    this.saveData();
    this.cleanup();
  }
  
  // Абстрактные шаги (должны быть реализованы подклассами)
  protected abstract loadData(source: string): void;
  protected abstract transformData(): void;
  protected abstract saveData(): void;
  
  // Шаг с реализацией по умолчанию
  protected cleanup(): void {
    // очистка временных данных
    console.log("Cleanup completed");
  }
  
  // Защита от переопределения шаблонного метода
  protected isDataValid(): boolean {
    // базовая проверка
    return true;
  }
}

// Конкретные реализации
class JSONDataProcessor extends DataProcessor {
  private data: any = null;
  
  protected loadData(source: string): void {
    console.log(`Loading JSON data from: ${source}`);
    this.data = { source, timestamp: Date.now() };
  }
  
  protected transformData(): void {
    console.log("Transforming JSON data");
    if (this.data) {
      this.data.processed = true;
      this.data.transformedAt = Date.now();
    }
  }
  
  protected saveData(): void {
    console.log("Saving transformed JSON data");
    // сохранение данных
  }
}

class CSVDataProcessor extends DataProcessor {
  private data: string[][] = [];
  
  protected loadData(source: string): void {
    console.log(`Loading CSV data from: ${source}`);
    this.data = [["header1", "header2"], ["data1", "data2"]];
  }
  
  protected transformData(): void {
    console.log("Transforming CSV data");
    // трансформация CSV данных
    this.data.forEach(row => {
      row.push("processed");
    });
  }
  
  protected saveData(): void {
    console.log("Saving transformed CSV data");
    // сохранение CSV данных
  }
  
  protected cleanup(): void {
    console.log("Cleaning up CSV processor resources");
    this.data = [];
    super.cleanup();
  }
}

// Использование
const jsonProcessor = new JSONDataProcessor();
const csvProcessor = new CSVDataProcessor();

jsonProcessor.process("./data.json");
console.log("---");
csvProcessor.process("./data.csv");
```

### Функциональный подход к Template Method
```typescript
// Функциональный подход к шаблонному методу
type ProcessingStep<T> = (data: T) => T;

interface ProcessingPipeline<T> {
  loadData(source: string): T;
  steps: ProcessingStep<T>[];
  execute(source: string): T;
}

class FunctionalProcessor<T> implements ProcessingPipeline<T> {
  constructor(
    public loadData: (source: string) => T,
    public steps: ProcessingStep<T>[] = []
  ) {}
  
  execute(source: string): T {
    let data = this.loadData(source);
    
    for (const step of this.steps) {
      data = step(data);
    }
    
    return data;
  }
  
  addStep(step: ProcessingStep<T>): this {
    this.steps.push(step);
    return this;
  }
  
  setSteps(steps: ProcessingStep<T>[]): this {
    this.steps = steps;
    return this;
  }
}

// Использование
const jsonProcessor = new FunctionalProcessor<any>(
  (source: string) => ({ source, loaded: true })
)
.addStep(data => ({ ...data, transformed: true }))
.addStep(data => ({ ...data, saved: true }));

const result = jsonProcessor.execute("./data.json");
console.log(result); // { source: "./data.json", loaded: true, transformed: true, saved: true }
```

## Лучшие практики

### 1. Используйте Observer для реактивности
```typescript
// Обсервер с обобщениями
class TypedObserver<T> {
  private observers: Array<(data: T) => void> = [];
  
  subscribe(callback: (data: T) => void): () => void {
    this.observers.push(callback);
    
    return () => {
      const index = this.observers.indexOf(callback);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  notify(data: T): void {
    this.observers.forEach(observer => observer(data));
  }
}

// Использование
const userObserver = new TypedObserver<{ id: number; name: string }>();

const unsubscribe = userObserver.subscribe(user => {
  console.log(`User updated: ${user.name} (ID: ${user.id})`);
});

userObserver.notify({ id: 1, name: "Alice" });
```

### 2. Используйте Command для отмены/повтора
```typescript
// Простая система команд для отмены
interface UndoableCommand {
  execute(): void;
  undo(): void;
  canUndo(): boolean;
}

class TextEditor {
  private content: string = "";
  private history: UndoableCommand[] = [];
  private historyIndex: number = -1;
  
  insertText(text: string): void {
    const command = new InsertTextCommand(this, text);
    command.execute();
    this.addToHistory(command);
  }
  
  private addToHistory(command: UndoableCommand): void {
    this.history = this.history.slice(0, this.historyIndex + 1);
    this.history.push(command);
    this.historyIndex++;
  }
  
  undo(): void {
    if (this.historyIndex >= 0) {
      const command = this.history[this.historyIndex];
      if (command.canUndo()) {
        command.undo();
        this.historyIndex--;
      }
    }
  }
  
  getContent(): string {
    return this.content;
  }
  
  setContent(content: string): void {
    this.content = content;
  }
}

class InsertTextCommand implements UndoableCommand {
  private previousContent: string;
  
  constructor(private editor: TextEditor, private text: string) {}
  
  execute(): void {
    this.previousContent = this.editor.getContent();
    const newContent = this.previousContent + this.text;
    this.editor.setContent(newContent);
  }
  
  undo(): void {
    this.editor.setContent(this.previousContent);
  }
  
  canUndo(): boolean {
    return true;
  }
}
```

## Современные применения

### Использование с RxJS
```typescript
// Совмещение паттернов с реактивным программированием
import { Observable, Subject, BehaviorSubject } from 'rxjs';
import { map, filter, tap } from 'rxjs/operators';

class ReactiveMediator {
  private subjects = new Map<string, Subject<any>>();
  
  getChannel<T>(name: string): Observable<T> {
    if (!this.subjects.has(name)) {
      this.subjects.set(name, new Subject<T>());
    }
    return this.subjects.get(name)!.asObservable() as Observable<T>;
  }
  
  getChannelSender<T>(name: string) {
    if (!this.subjects.has(name)) {
      this.subjects.set(name, new Subject<T>());
    }
    return (data: T) => this.subjects.get(name)!.next(data);
  }
}

// Использование
const mediator = new ReactiveMediator();

const userUpdates$ = mediator.getChannel<{ id: number; name: string }>('user-updates');
const sendUserUpdate = mediator.getChannelSender<{ id: number; name: string }>('user-updates');

userUpdates$.pipe(
  filter(user => user.id > 0),
  map(user => ({ ...user, processed: true })),
  tap(user => console.log('Processed user:', user))
).subscribe();

sendUserUpdate({ id: 1, name: 'Alice' });
// Выводит: "Processed user: { id: 1, name: 'Alice', processed: true }"
```

### Функциональное состояние с паттернами
```typescript
// Состояние с использованием функционального подхода
type StateReducer<T, A> = (state: T, action: A) => T;

class FunctionalStateMachine<T, A> {
  private state: T;
  private subscribers: Array<(state: T) => void> = [];
  
  constructor(
    initialState: T,
    private reducer: StateReducer<T, A>
  ) {
    this.state = initialState;
  }
  
  dispatch(action: A): void {
    this.state = this.reducer(this.state, action);
    this.notifySubscribers();
  }
  
  getState(): T {
    return this.state;
  }
  
  subscribe(subscriber: (state: T) => void): () => void {
    this.subscribers.push(subscriber);
    
    return () => {
      const index = this.subscribers.indexOf(subscriber);
      if (index > -1) {
        this.subscribers.splice(index, 1);
      }
    };
  }
  
  private notifySubscribers(): void {
    this.subscribers.forEach(subscriber => subscriber(this.state));
  }
}

// Использование
interface CounterState {
  count: number;
  lastUpdated: Date | null;
}

type CounterAction = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' } 
  | { type: 'RESET' };

const counterReducer: StateReducer<CounterState, CounterAction> = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1, lastUpdated: new Date() };
    case 'DECREMENT':
      return { ...state, count: state.count - 1, lastUpdated: new Date() };
    case 'RESET':
      return { count: 0, lastUpdated: null };
    default:
      return state;
  }
};

const counterMachine = new FunctionalStateMachine<CounterState, CounterAction>(
  { count: 0, lastUpdated: null },
  counterReducer
);

counterMachine.subscribe(state => {
  console.log(`Count: ${state.count}, Last updated: ${state.lastUpdated}`);
});

counterMachine.dispatch({ type: 'INCREMENT' }); // Count: 1, Last updated: [date]
counterMachine.dispatch({ type: 'INCREMENT' }); // Count: 2, Last updated: [date]
counterMachine.dispatch({ type: 'RESET' });     // Count: 0, Last updated: null
```

## Сравнение паттернов

| Паттерн | Назначение | Когда использовать |
|---------|-----------|------------------|
| Observer | Уведомление о изменениях | При изменении состояния нескольких объектов |
| Strategy | Выбор алгоритма во время выполнения | Когда нужно менять алгоритмы динамически |
| Command | Инкапсуляция запроса как объекта | Для отмены/повтора действий |
| State | Изменение поведения в зависимости от состояния | Когда поведение зависит от внутреннего состояния |
| Iterator | Последовательный доступ к элементам | Для унификации обхода коллекций |
| Mediator | Централизованное взаимодействие | Когда объекты слишком связаны |
| Memento | Сохранение и восстановление состояния | Для отмены/истории изменений |
| Visitor | Выполнение операции над элементами структуры | Когда нужно добавить функциональность без модификации классов |
| Template Method | Каркас алгоритма с возможностью переопределения | Для определения скелета алгоритма |
| Chain of Responsibility | Передача запросов по цепочке | Для обработки запросов несколькими способами |

## Ограничения и подводные камни

1. **Observer**: может привести к утечке памяти, если не отписываться
2. **Strategy**: увеличивает количество классов/интерфейсов
3. **Command**: может увеличить количество классов для простых операций
4. **State**: может усложнить код при большом количестве состояний
5. **Visitor**: нарушает инкапсуляцию, добавление новых элементов требует обновления всех посетителей

## Связь с другими концепциями
- [[../functional-programming/index]] - функциональные альтернативы поведенческим паттернам
- [[../design-patterns/structural-patterns]] - структурные паттерны для организации поведения
- [[../type-system/advanced-types]] - сложные типы для паттернов
- [[../modules/system-design]] - организационные паттерны для модулей
- [[../ecosystem/reactive-programming]] - реактивные альтернативы Observer и Mediator