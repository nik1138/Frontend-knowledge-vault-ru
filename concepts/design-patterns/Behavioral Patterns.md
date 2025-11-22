---
aliases: [Поведенческие паттерны проектирования, Поведенческие паттерны]
tags: [design-patterns, programming, architecture]
---

# Поведенческие паттерны проектирования

## Обзор

Поведенческие паттерны проектирования сосредоточены на взаимодействии между объектами и распределении обязанностей. Они определяют, как объекты общаются друг с другом, и помогают создавать гибкие и повторно используемые архитектурные решения. Эти паттерны особенно важны при разработке сложных систем, где взаимодействие между компонентами играет ключевую роль.

## Observer (Наблюдатель)

Паттерн Observer определяет зависимость "один ко многим" между объектами, так что при изменении состояния одного объекта все зависящие от него объекты уведомляются и обновляются автоматически.

### Структура

- **Subject (Издатель)**: знает о своих наблюдателях, предоставляет интерфейс для добавления и удаления наблюдателей
- **Observer (Наблюдатель)**: определяет интерфейс для объектов, получающих уведомления
- **ConcreteSubject (Конкретный издатель)**: хранит состояние, которое может заинтересовать наблюдателей
- **ConcreteObserver (Конкретный наблюдатель)**: поддерживает согласованность со своим субъектом

### Пример реализации

```typescript
// Интерфейс наблюдателя
interface Observer {
  update(subject: Subject): void;
}

// Интерфейс издателя
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Конкретный издатель
class NewsAgency implements Subject {
  private observers: Observer[] = [];
  private news: string = '';

  attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      return console.log('Observer has been attached already.');
    }
    
    this.observers.push(observer);
    console.log('Observer attached.');
  }

  detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex === -1) {
      return console.log('Observer was not attached.');
    }

    this.observers.splice(observerIndex, 1);
    console.log('Observer detached.');
  }

  notify(): void {
    console.log('NewsAgency: Notifying observers...');
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  // Метод для установки новостей
  setNews(news: string): void {
    console.log(`NewsAgency: Setting news to "${news}"`);
    this.news = news;
    this.notify();
  }

  getNews(): string {
    return this.news;
  }
}

// Конкретные наблюдатели
class NewsChannel implements Observer {
  private name: string;

  constructor(name: string) {
    this.name = name;
  }

  update(subject: Subject): void {
    if (subject instanceof NewsAgency) {
      console.log(`${this.name}: News received - "${subject.getNews()}"`);
    }
  }
}

// Использование
const newsAgency = new NewsAgency();

const cnn = new NewsChannel('CNN');
const bbc = new NewsChannel('BBC');
const fox = new NewsChannel('Fox News');

newsAgency.attach(cnn);
newsAgency.attach(bbc);

newsAgency.setNews('Breaking: Major event occurred!');

newsAgency.detach(bbc);

newsAgency.setNews('Update: More details revealed.');
```

### Применение

- Реализация систем уведомлений
- Поддержка интерфейса MVC (Model-View-Controller)
- Реализация событийной архитектуры
- Создание подписок на изменения данных

## Strategy (Стратегия)

Паттерн Strategy позволяет определять семейство алгоритмов, инкапсулировать каждый из них и делать их взаимозаменяемыми. Strategy позволяет изменять алгоритмы независимо от клиентов, которые их используют.

### Структура

- **Strategy**: объявляет общий интерфейс для всех поддерживаемых алгоритмов
- **ConcreteStrategy**: реализует конкретный алгоритм
- **Context**: использует стратегию для выполнения определенной задачи

### Пример реализации

```typescript
// Интерфейс стратегии
interface PaymentStrategy {
  pay(amount: number): void;
}

// Конкретные стратегии
class CreditCardPayment implements PaymentStrategy {
  private cardNumber: string;
  private cvv: string;

  constructor(cardNumber: string, cvv: string) {
    this.cardNumber = cardNumber;
    this.cvv = cvv;
  }

  pay(amount: number): void {
    console.log(`Paid $${amount} using Credit Card ending in ${this.cardNumber.slice(-4)}`);
  }
}

class PayPalPayment implements PaymentStrategy {
  private email: string;

  constructor(email: string) {
    this.email = email;
  }

  pay(amount: number): void {
    console.log(`Paid $${amount} using PayPal account ${this.email}`);
  }
}

class BankTransferPayment implements PaymentStrategy {
  private accountNumber: string;

  constructor(accountNumber: string) {
    this.accountNumber = accountNumber;
  }

  pay(amount: number): void {
    console.log(`Paid $${amount} via Bank Transfer to account ${this.accountNumber}`);
  }
}

// Контекст
class PaymentProcessor {
  private strategy: PaymentStrategy;

  constructor(strategy: PaymentStrategy) {
    this.strategy = strategy;
  }

  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy;
  }

  processPayment(amount: number): void {
    this.strategy.pay(amount);
  }
}

// Использование
const orderAmount = 99.99;

const creditCard = new CreditCardPayment('1234567890123456', '123');
const paymentProcessor = new PaymentProcessor(creditCard);

paymentProcessor.processPayment(orderAmount);

// Изменение стратегии в рантайме
const paypal = new PayPalPayment('user@example.com');
paymentProcessor.setPaymentStrategy(paypal);
paymentProcessor.processPayment(orderAmount);
```

### Применение

- Выбор алгоритмов в зависимости от входных данных
- Реализация различных способов сортировки
- Поддержка различных методов аутентификации
- Создание гибких систем оплаты

## Command (Команда)

Паттерн Command инкапсулирует запрос в виде объекта, позволяя параметризовать клиентов с различными запросами, организовывать очередь запросов или поддерживать отмену операций.

### Структура

- **Command**: объявляет интерфейс для выполнения операции
- **ConcreteCommand**: связывает получателя и действие
- **Client**: создает объект конкретной команды и устанавливает его получателя
- **Invoker**: вызывает команду
- **Receiver**: знает, как выполнить операцию

### Пример реализации

```typescript
// Интерфейс команды
interface Command {
  execute(): void;
  undo?(): void;
}

// Получатель
class TextEditor {
  private content: string = '';

  write(text: string): void {
    this.content += text;
    console.log(`Content: ${this.content}`);
  }

  deleteLast(): void {
    this.content = this.content.slice(0, -1);
    console.log(`Content: ${this.content}`);
  }

  getContent(): string {
    return this.content;
  }
}

// Конкретные команды
class WriteCommand implements Command {
  private editor: TextEditor;
  private text: string;

  constructor(editor: TextEditor, text: string) {
    this.editor = editor;
    this.text = text;
  }

  execute(): void {
    this.editor.write(this.text);
  }

  undo(): void {
    for (let i = 0; i < this.text.length; i++) {
      this.editor.deleteLast();
    }
  }
}

class DeleteCommand implements Command {
  private editor: TextEditor;
  private deletedChar: string;

  constructor(editor: TextEditor) {
    this.editor = editor;
    this.deletedChar = this.editor.getContent().slice(-1);
  }

  execute(): void {
    this.editor.deleteLast();
  }

  undo(): void {
    this.editor.write(this.deletedChar);
  }
}

// Инвокер
class TextEditorInvoker {
  private history: Command[] = [];
  private undoHistory: Command[] = [];

  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
  }

  undo(): void {
    if (this.history.length === 0) {
      console.log('No commands to undo');
      return;
    }

    const command = this.history.pop();
    if (command && command.undo) {
      command.undo();
      this.undoHistory.push(command);
    }
  }

  redo(): void {
    if (this.undoHistory.length === 0) {
      console.log('No commands to redo');
      return;
    }

    const command = this.undoHistory.pop();
    if (command) {
      command.execute();
      this.history.push(command);
    }
  }
}

// Использование
const editor = new TextEditor();
const invoker = new TextEditorInvoker();

invoker.executeCommand(new WriteCommand(editor, 'Hello'));
invoker.executeCommand(new WriteCommand(editor, ' World'));
invoker.undo();
invoker.redo();
```

### Применение

- Поддержка операций отмены/повтора
- Создание очередей выполнения команд
- Реализация логов транзакций
- Создание макросов (сериализованных команд)

## Iterator (Итератор)

Паттерн Iterator предоставляет способ последовательного доступа к элементам объекта-агрегата, не раскрывая его внутреннего представления.

### Структура

- **Iterator**: объявляет интерфейс для доступа к элементам
- **ConcreteIterator**: реализует интерфейс и отслеживает текущую позицию
- **Aggregate**: объявляет интерфейс для создания итератора
- **ConcreteAggregate**: возвращает конкретный итератор

### Пример реализации

```typescript
// Интерфейс итератора
interface Iterator<T> {
  next(): T | null;
  hasNext(): boolean;
}

// Интерфейс агрегата
interface Aggregate<T> {
  createIterator(): Iterator<T>;
}

// Конкретный итератор для списка задач
class TaskListIterator implements Iterator<string> {
  private tasks: string[];
  private position: number = 0;

  constructor(tasks: string[]) {
    this.tasks = tasks;
  }

  next(): string | null {
    if (this.hasNext()) {
      return this.tasks[this.position++];
    }
    return null;
  }

  hasNext(): boolean {
    return this.position < this.tasks.length;
  }
}

// Конкретный агрегат
class TaskList implements Aggregate<string> {
  private tasks: string[] = [];

  addTask(task: string): void {
    this.tasks.push(task);
  }

  createIterator(): Iterator<string> {
    return new TaskListIterator(this.tasks);
  }

  getTasks(): string[] {
    return this.tasks;
  }
}

// Использование
const taskList = new TaskList();
taskList.addTask('Задача 1');
taskList.addTask('Задача 2');
taskList.addTask('Задача 3');

const iterator = taskList.createIterator();

console.log('Задачи:');
while (iterator.hasNext()) {
  const task = iterator.next();
  if (task) {
    console.log(`- ${task}`);
  }
}
```

### Применение

- Обход коллекций без раскрытия внутренней структуры
- Поддержка различных способов обхода коллекции
- Единообразный интерфейс для обхода различных типов коллекций

## Mediator (Посредник)

Паттерн Mediator определяет объект, инкапсулирующий способ взаимодействия множества объектов. Посредник обеспечивает слабую связанность системы, избегая прямых ссылок между объектами и позволяя независимо изменять их взаимодействие.

### Структура

- **Mediator**: объявляет интерфейс для общения с компонентами
- **ConcreteMediator**: реализует взаимодействие между компонентами
- **Component**: базовый класс компонентов, которые взаимодействуют через посредника

### Пример реализации

```typescript
// Интерфейс посредника
interface ChatRoomMediator {
  showMessage(user: User, message: string): void;
}

// Конкретный посредник
class ChatRoom implements ChatRoomMediator {
  showMessage(user: User, message: string): void {
    const time = new Date().toLocaleTimeString();
    const sender = user.getName();
    console.log(`[${time}] ${sender}: ${message}`);
  }
}

// Компонент
class User {
  private name: string;
  private mediator: ChatRoomMediator;

  constructor(name: string, mediator: ChatRoomMediator) {
    this.name = name;
    this.mediator = mediator;
  }

  getName(): string {
    return this.name;
  }

  send(message: string): void {
    this.mediator.showMessage(this, message);
  }
}

// Использование
const mediator = new ChatRoom();

const john = new User('John', mediator);
const jane = new User('Jane', mediator);
const bob = new User('Bob', mediator);

john.send('Привет всем!');
jane.send('Привет, Джон!');
bob.send('Привет, ребята!');
```

### Применение

- Реализация чат-систем
- Управление взаимодействием между GUI-компонентами
- Создание систем обмена сообщениями
- Упрощение сложных зависимостей между объектами

## Memento (Хранитель)

Паттерн Memento позволяет фиксировать и восстанавливать внутреннее состояние объекта без нарушения инкапсуляции.

### Структура

- **Memento**: хранит внутреннее состояние объекта Originator
- **Originator**: создает снимок своего состояния и может восстановиться из него
- **Caretaker**: отвечает за сохранение снимков, но не изменяет их

### Пример реализации

```typescript
// Хранитель
class EditorMemento {
  private content: string;
  private timestamp: Date;

  constructor(content: string) {
    this.content = content;
    this.timestamp = new Date();
  }

  getContent(): string {
    return this.content;
  }

  getTimestamp(): Date {
    return this.timestamp;
  }
}

// Создатель
class Editor {
  private content: string = '';

  type(words: string): void {
    this.content += words;
  }

  getContent(): string {
    return this.content;
  }

  save(): EditorMemento {
    return new EditorMemento(this.content);
  }

  restore(memento: EditorMemento): void {
    this.content = memento.getContent();
  }
}

// Опекун
class History {
  private mementos: EditorMemento[] = [];

  push(memento: EditorMemento): void {
    this.mementos.push(memento);
  }

  pop(): EditorMemento | undefined {
    return this.mementos.pop();
  }

  isEmpty(): boolean {
    return this.mementos.length === 0;
  }
}

// Использование
const editor = new Editor();
const history = new History();

editor.type('Первая строка.\n');
history.push(editor.save());

editor.type('Вторая строка.\n');
history.push(editor.save());

editor.type('Третья строка.\n');

console.log('Текущий контент:');
console.log(editor.getContent());

// Отмена последнего действия
if (!history.isEmpty()) {
  const lastState = history.pop();
  if (lastState) {
    editor.restore(lastState);
  }
}

console.log('После отмены:');
console.log(editor.getContent());
```

### Применение

- Поддержка операций отмены/повтора
- Создание контрольных точек в приложениях
- Реализация транзакций
- Сохранение состояния игры

## State (Состояние)

Паттерн State позволяет объекту изменять свое поведение в зависимости от внутреннего состояния. Он выглядит так, как будто объект изменил свой класс.

### Структура

- **State**: определяет интерфейс для конкретных состояний
- **ConcreteState**: реализует поведение, связанное с конкретным состоянием
- **Context**: определяет интерфейс для клиентов и хранит ссылку на текущее состояние

### Пример реализации

```typescript
// Интерфейс состояния
interface PlayerState {
  play(player: MediaPlayer): void;
  pause(player: MediaPlayer): void;
  stop(player: MediaPlayer): void;
}

// Конкретные состояния
class PlayingState implements PlayerState {
  play(player: MediaPlayer): void {
    console.log('Плеер уже воспроизводит');
  }

  pause(player: MediaPlayer): void {
    console.log('Пауза воспроизведения');
    player.setState(new PausedState());
  }

  stop(player: MediaPlayer): void {
    console.log('Остановка воспроизведения');
    player.setState(new StoppedState());
  }
}

class PausedState implements PlayerState {
  play(player: MediaPlayer): void {
    console.log('Возобновление воспроизведения');
    player.setState(new PlayingState());
  }

  pause(player: MediaPlayer): void {
    console.log('Плеер уже на паузе');
  }

  stop(player: MediaPlayer): void {
    console.log('Остановка воспроизведения');
    player.setState(new StoppedState());
  }
}

class StoppedState implements PlayerState {
  play(player: MediaPlayer): void {
    console.log('Начало воспроизведения');
    player.setState(new PlayingState());
  }

  pause(player: MediaPlayer): void {
    console.log('Невозможно поставить на паузу - плеер остановлен');
  }

  stop(player: MediaPlayer): void {
    console.log('Плеер уже остановлен');
  }
}

// Контекст
class MediaPlayer {
  private state: PlayerState;

  constructor() {
    this.state = new StoppedState();
  }

  setState(state: PlayerState): void {
    this.state = state;
  }

  play(): void {
    this.state.play(this);
  }

  pause(): void {
    this.state.pause(this);
  }

  stop(): void {
    this.state.stop(this);
  }
}

// Использование
const player = new MediaPlayer();

player.play();   // Начало воспроизведения
player.pause();  // Пауза воспроизведения
player.play();   // Возобновление воспроизведения
player.stop();   // Остановка воспроизведения
player.pause();  // Невозможно поставить на паузу - плеер остановлен
```

### Применение

- Реализация конечных автоматов
- Управление жизненным циклом объектов
- Создание сложных рабочих процессов
- Реализация игр с различными состояниями

## Visitor (Посетитель)

Паттерн Visitor позволяет определять новую операцию без изменения классов объектов, над которыми она выполняется. Он позволяет отделить алгоритм от структуры объекта.

### Структура

- **Visitor**: объявляет интерфейс для каждой операции, принимающей определенный тип объекта
- **ConcreteVisitor**: реализует алгоритмы для конкретных типов объектов
- **Element**: определяет метод accept, принимающий посетителя
- **ConcreteElement**: реализует метод accept
- **ObjectStructure**: может перебирать элементы структуры

### Пример реализации

```typescript
// Интерфейс посетителя
interface Visitor {
  visitShape(shape: Shape): void;
  visitCircle(circle: Circle): void;
  visitRectangle(rectangle: Rectangle): void;
}

// Конкретные посетители
class AreaCalculator implements Visitor {
  private totalArea: number = 0;

  visitShape(shape: Shape): void {
    // Обработка общего случая
  }

  visitCircle(circle: Circle): void {
    this.totalArea += Math.PI * Math.pow(circle.getRadius(), 2);
  }

  visitRectangle(rectangle: Rectangle): void {
    this.totalArea += rectangle.getWidth() * rectangle.getHeight();
  }

  getTotalArea(): number {
    return this.totalArea;
  }
}

class ShapeInfoVisitor implements Visitor {
  private info: string[] = [];

  visitShape(shape: Shape): void {
    // Обработка общего случая
  }

  visitCircle(circle: Circle): void {
    this.info.push(`Круг с радиусом ${circle.getRadius()}`);
  }

  visitRectangle(rectangle: Rectangle): void {
    this.info.push(`Прямоугольник ${rectangle.getWidth()}x${rectangle.getHeight()}`);
  }

  getInfo(): string[] {
    return this.info;
  }
}

// Интерфейс элемента
interface Shape {
  accept(visitor: Visitor): void;
}

// Конкретные элементы
class Circle implements Shape {
  private radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }

  getRadius(): number {
    return this.radius;
  }

  accept(visitor: Visitor): void {
    visitor.visitCircle(this);
  }
}

class Rectangle implements Shape {
  private width: number;
  private height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  getWidth(): number {
    return this.width;
  }

  getHeight(): number {
    return this.height;
  }

  accept(visitor: Visitor): void {
    visitor.visitRectangle(this);
  }
}

// Структура объекта
class ShapeCollection {
  private shapes: Shape[] = [];

  addShape(shape: Shape): void {
    this.shapes.push(shape);
  }

  accept(visitor: Visitor): void {
    for (const shape of this.shapes) {
      shape.accept(visitor);
    }
  }
}

// Использование
const collection = new ShapeCollection();
collection.addShape(new Circle(5));
collection.addShape(new Rectangle(10, 8));
collection.addShape(new Circle(3));

const areaCalculator = new AreaCalculator();
collection.accept(areaCalculator);
console.log(`Общая площадь: ${areaCalculator.getTotalArea().toFixed(2)}`);

const infoVisitor = new ShapeInfoVisitor();
collection.accept(infoVisitor);
console.log('Информация о фигурах:', infoVisitor.getInfo());
```

### Применение

- Выполнение операций над структурой объектов
- Экспорт данных в различные форматы
- Создание отчетов
- Реализация сложных алгоритмов обхода

## Interpreter (Интерпретатор)

Паттерн Interpreter определяет представление грамматики для заданного языка и интерпретатор для этого языка. Он особенно полезен для создания языков программирования, скриптовых языков и других DSL (Domain Specific Languages).

### Структура

- **AbstractExpression**: объявляет интерфейс для интерпретации
- **TerminalExpression**: реализует интерпретацию терминальных символов грамматики
- **NonTerminalExpression**: реализует интерпретацию нетерминальных символов
- **Context**: содержит информацию, общую для интерпретатора
- **Client**: строит синтаксическое дерево, состоящее из Expression экземпляров

### Пример реализации

```typescript
// Контекст
class Context {
  private variables: Map<string, number> = new Map();

  setVariable(name: string, value: number): void {
    this.variables.set(name, value);
  }

  getVariable(name: string): number {
    const value = this.variables.get(name);
    if (value === undefined) {
      throw new Error(`Переменная ${name} не найдена`);
    }
    return value;
  }
}

// Абстрактное выражение
abstract class Expression {
  abstract interpret(context: Context): number;
}

// Терминальное выражение
class NumberExpression extends Expression {
  private number: number;

  constructor(number: number) {
    super();
    this.number = number;
  }

  interpret(context: Context): number {
    return this.number;
  }
}

class VariableExpression extends Expression {
  private name: string;

  constructor(name: string) {
    super();
    this.name = name;
  }

  interpret(context: Context): number {
    return context.getVariable(this.name);
  }
}

// Нетерминальные выражения
class AddExpression extends Expression {
  private left: Expression;
  private right: Expression;

  constructor(left: Expression, right: Expression) {
    super();
    this.left = left;
    this.right = right;
  }

  interpret(context: Context): number {
    return this.left.interpret(context) + this.right.interpret(context);
  }
}

class SubtractExpression extends Expression {
  private left: Expression;
  private right: Expression;

  constructor(left: Expression, right: Expression) {
    super();
    this.left = left;
    this.right = right;
  }

  interpret(context: Context): number {
    return this.left.interpret(context) - this.right.interpret(context);
  }
}

class MultiplyExpression extends Expression {
  private left: Expression;
  private right: Expression;

  constructor(left: Expression, right: Expression) {
    super();
    this.left = left;
    this.right = right;
  }

  interpret(context: Context): number {
    return this.left.interpret(context) * this.right.interpret(context);
  }
}

// Парсер выражений (упрощенная версия)
class ExpressionParser {
  static parse(expression: string): Expression {
    // Упрощенный парсер для выражений вида "a + b * c"
    // В реальном приложении потребуется более сложный парсер
    const tokens = expression.split(' ');
    
    if (tokens.length === 1) {
      const token = tokens[0];
      if (!isNaN(Number(token))) {
        return new NumberExpression(Number(token));
      } else {
        return new VariableExpression(token);
      }
    }
    
    // Для упрощения, предположим, что выражение всегда вида "a + b"
    if (tokens.length === 3) {
      const left = this.parse(tokens[0]);
      const operator = tokens[1];
      const right = this.parse(tokens[2]);
      
      switch (operator) {
        case '+':
          return new AddExpression(left, right);
        case '-':
          return new SubtractExpression(left, right);
        case '*':
          return new MultiplyExpression(left, right);
        default:
          throw new Error(`Неизвестный оператор: ${operator}`);
      }
    }
    
    throw new Error(`Неподдерживаемое выражение: ${expression}`);
  }
}

// Использование
const context = new Context();
context.setVariable('x', 10);
context.setVariable('y', 5);
context.setVariable('z', 3);

// Создание выражения: x + y * z
const expr1 = new AddExpression(
  new VariableExpression('x'),
  new MultiplyExpression(
    new VariableExpression('y'),
    new VariableExpression('z')
  )
);

console.log(`x + y * z = ${expr1.interpret(context)}`); // 10 + 5 * 3 = 25

// Использование парсера
const expr2 = ExpressionParser.parse('x + y');
console.log(`x + y = ${expr2.interpret(context)}`); // 10 + 5 = 15

const expr3 = ExpressionParser.parse('x * y');
console.log(`x * y = ${expr3.interpret(context)}`); // 10 * 5 = 50
```

### Применение

- Создание DSL (Domain Specific Languages)
- Реализация калькуляторов
- Парсинг конфигурационных файлов
- Создание систем шаблонов

## Template Method (Шаблонный метод)

Паттерн Template Method определяет skeleton алгоритма в операции, откладывая некоторые шаги до подклассов. Он позволяет подклассам переопределять части алгоритма без изменения его структуры.

### Структура

- **AbstractClass**: определяет абстрактный интерфейс для алгоритма
- **ConcreteClass**: реализует шаги алгоритма, определенные в абстрактном классе

### Пример реализации

```typescript
// Абстрактный класс
abstract class DataProcessor {
  // Шаблонный метод
  public process(): void {
    this.loadData();
    this.validateData();
    this.transformData();
    this.saveData();
    this.notifyComplete();
  }

  // Общие шаги
  protected loadData(): void {
    console.log('Загрузка данных...');
  }

  protected validateData(): void {
    console.log('Валидация данных...');
  }

  protected saveData(): void {
    console.log('Сохранение данных...');
  }

  protected notifyComplete(): void {
    console.log('Обработка завершена!');
  }

  // Абстрактные шаги
  protected abstract transformData(): void;
}

// Конкретные классы
class CSVDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log('Преобразование CSV данных...');
    // Логика преобразования CSV данных
  }
}

class JSONDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log('Преобразование JSON данных...');
    // Логика преобразования JSON данных
  }
}

class XMLDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log('Преобразование XML данных...');
    // Логика преобразования XML данных
  }
}

// Использование
const processors = [
  new CSVDataProcessor(),
  new JSONDataProcessor(),
  new XMLDataProcessor()
];

for (const processor of processors) {
  console.log(`\nОбработка ${processor.constructor.name}:`);
  processor.process();
}
```

### Применение

- Создание фреймворков
- Реализация алгоритмов с общими шагами
- Создание систем обработки данных
- Реализация жизненных циклов компонентов

## Chain of Responsibility (Цепочка обязанностей)

Паттерн Chain of Responsibility передает запрос по цепочке потенциальных обработчиков, пока один из них не обработает запрос. Каждый обработчик может либо обработать запрос, либо передать его следующему в цепочке.

### Структура

- **Handler**: объявляет интерфейс для обработки запросов
- **ConcreteHandler**: обрабатывает запросы, которые он может обработать
- **Client**: инициирует запрос к первому обработчику в цепочке

### Пример реализации

```typescript
// Интерфейс обработчика
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: Request): string | null;
}

// Класс запроса
class Request {
  constructor(
    public type: string,
    public priority: number,
    public content: string
  ) {}
}

// Базовый обработчик
abstract class AbstractHandler implements Handler {
  private nextHandler: Handler | null = null;

  setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler;
  }

  handle(request: Request): string | null {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    return null;
  }
}

// Конкретные обработчики
class AuthenticationHandler extends AbstractHandler {
  handle(request: Request): string | null {
    console.log(`Проверка аутентификации для запроса: ${request.type}`);
    
    // Имитация проверки аутентификации
    if (request.content.includes('auth')) {
      console.log('Аутентификация прошла успешно');
      return super.handle(request);
    } else {
      return 'Ошибка аутентификации';
    }
  }
}

class AuthorizationHandler extends AbstractHandler {
  handle(request: Request): string | null {
    console.log(`Проверка авторизации для запроса: ${request.type}`);
    
    // Имитация проверки авторизации
    if (request.priority >= 1) {
      console.log('Авторизация прошла успешно');
      return super.handle(request);
    } else {
      return 'Недостаточно прав для выполнения запроса';
    }
  }
}

class RequestValidationHandler extends AbstractHandler {
  handle(request: Request): string | null {
    console.log(`Валидация запроса: ${request.type}`);
    
    // Имитация валидации
    if (request.content && request.content.length > 0) {
      console.log('Валидация прошла успешно');
      return super.handle(request);
    } else {
      return 'Некорректный запрос';
    }
  }
}

class BusinessLogicHandler extends AbstractHandler {
  handle(request: Request): string | null {
    console.log(`Выполнение бизнес-логики для запроса: ${request.type}`);
    return `Запрос ${request.type} успешно обработан`;
  }
}

// Использование
const authHandler = new AuthenticationHandler();
const authzHandler = new AuthorizationHandler();
const validationHandler = new RequestValidationHandler();
const businessHandler = new BusinessLogicHandler();

// Создание цепочки
authHandler
  .setNext(authzHandler)
  .setNext(validationHandler)
  .setNext(businessHandler);

// Тестирование
const validRequest = new Request('GET', 2, 'auth:user:read');
console.log('\nОбработка валидного запроса:');
console.log(authHandler.handle(validRequest));

console.log('\nОбработка невалидного запроса:');
const invalidRequest = new Request('POST', 0, 'invalid');
console.log(authHandler.handle(invalidRequest));
```

### Применение

- Обработка HTTP-запросов (middleware)
- Системы обработки заказов
- Цепочки фильтров
- Обработка исключений

## Заключение

Поведенческие паттерны проектирования предоставляют мощные инструменты для организации взаимодействия между объектами и распределения обязанностей. Каждый паттерн решает определенную проблему взаимодействия и может быть использован в соответствующем контексте:

- **Observer** - для уведомлений и событийной архитектуры
- **Strategy** - для выбора алгоритмов
- **Command** - для поддержки отмены/повтора
- **Iterator** - для обхода коллекций
- **Mediator** - для сложных взаимодействий
- **Memento** - для сохранения состояния
- **State** - для изменения поведения
- **Visitor** - для операций над структурами
- **Interpreter** - для языков и DSL
- **Template Method** - для общих алгоритмов
- **Chain of Responsibility** - для обработки запросов

Выбор подходящего паттерна зависит от конкретной проблемы, которую нужно решить, и архитектуры приложения. Правильное применение поведенческих паттернов делает код более гибким, расширяемым и понятным.

Ссылки на связанные концепции: [[Creational Patterns]], [[Structural Patterns]], [[Design Patterns]], [[Software Architecture]], [[SOLID Principles]], [[Clean Architecture]], [[Architecture Patterns]].