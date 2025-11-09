# Паттерны проектирования в TypeScript

Паттерны проектирования - это проверенные временем решения типичных проблем проектирования программного обеспечения. TypeScript, благодаря своей системе типов и поддержке ООП, идеально подходит для реализации различных паттернов проектирования.

## Порождающие паттерны

### 1. Фабричный метод (Factory Method)

```typescript
// Продукт
interface Button {
  render(): void;
  onClick(): void;
}

// Конкретные продукты
class WindowsButton implements Button {
  render(): void {
    console.log("Отображение Windows кнопки");
  }
  
  onClick(): void {
    console.log("Windows кнопка нажата");
  }
}

class MacButton implements Button {
  render(): void {
    console.log("Отображение Mac кнопки");
  }
  
  onClick(): void {
    console.log("Mac кнопка нажата");
  }
}

// Создатель
abstract class Dialog {
  abstract createButton(): Button;
  
  render(): void {
    const button = this.createButton();
    button.render();
    button.onClick();
  }
}

class WindowsDialog extends Dialog {
  createButton(): Button {
    return new WindowsButton();
  }
}

class MacDialog extends Dialog {
  createButton(): Button {
    return new MacButton();
  }
}

// Использование
const windowsDialog = new WindowsDialog();
windowsDialog.render(); // Отображение Windows кнопки, Windows кнопка нажата

const macDialog = new MacDialog();
macDialog.render(); // Отображение Mac кнопки, Mac кнопка нажата
```

### 2. Абстрактная фабрика (Abstract Factory)

```typescript
// Абстрактные продукты
interface Button {
  paint(): void;
}

interface Checkbox {
  paint(): void;
}

// Конкретные продукты
class WindowsButton implements Button {
  paint(): void {
    console.log("Отрисовка Windows кнопки");
  }
}

class MacButton implements Button {
  paint(): void {
    console.log("Отрисовка Mac кнопки");
  }
}

class WindowsCheckbox implements Checkbox {
  paint(): void {
    console.log("Отрисовка Windows чекбокса");
  }
}

class MacCheckbox implements Checkbox {
  paint(): void {
    console.log("Отрисовка Mac чекбокса");
  }
}

// Абстрактная фабрика
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// Конкретные фабрики
class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return new WindowsButton();
  }
  
  createCheckbox(): Checkbox {
    return new WindowsCheckbox();
  }
}

class MacFactory implements GUIFactory {
  createButton(): Button {
    return new MacButton();
  }
  
  createCheckbox(): Checkbox {
    return new MacCheckbox();
  }
}

// Клиентский код
class Application {
  private button: Button;
  private checkbox: Checkbox;
  
  constructor(factory: GUIFactory) {
    this.button = factory.createButton();
    this.checkbox = factory.createCheckbox();
  }
  
  paint(): void {
    this.button.paint();
    this.checkbox.paint();
  }
}

// Использование
const platform = "Windows"; // или "Mac"
let factory: GUIFactory;

if (platform === "Windows") {
  factory = new WindowsFactory();
} else {
  factory = new MacFactory();
}

const app = new Application(factory);
app.paint();
```

### 3. Строитель (Builder)

```typescript
interface Car {
  brand: string;
  model: string;
  engine: string;
  transmission: string;
  color: string;
}

class CarBuilder {
  private car: Car = { brand: "", model: "", engine: "", transmission: "", color: "" };
  
  setBrand(brand: string): this {
    this.car.brand = brand;
    return this;
  }
  
  setModel(model: string): this {
    this.car.model = model;
    return this;
  }
  
  setEngine(engine: string): this {
    this.car.engine = engine;
    return this;
  }
  
  setTransmission(transmission: string): this {
    this.car.transmission = transmission;
    return this;
  }
  
  setColor(color: string): this {
    this.car.color = color;
    return this;
  }
  
  build(): Car {
    return { ...this.car };
  }
}

// Директор (опционально)
class CarDirector {
  constructSedan(builder: CarBuilder): Car {
    return builder
      .setBrand("Toyota")
      .setModel("Camry")
      .setEngine("2.5L 4-cylinder")
      .setTransmission("Automatic")
      .setColor("White")
      .build();
  }
  
  constructSUV(builder: CarBuilder): Car {
    return builder
      .setBrand("Honda")
      .setModel("CR-V")
      .setEngine("1.5L Turbo 4-cylinder")
      .setTransmission("CVT")
      .setColor("Black")
      .build();
  }
}

// Использование
const builder = new CarBuilder();
const director = new CarDirector();

const sedan = director.constructSedan(builder);
console.log(sedan);

const customCar = new CarBuilder()
  .setBrand("Tesla")
  .setModel("Model 3")
  .setEngine("Electric")
  .setTransmission("Automatic")
  .setColor("Red")
  .build();

console.log(customCar);
```

### 4. Прототип (Prototype)

```typescript
interface Prototype<T> {
  clone(): T;
}

class Address {
  constructor(
    public street: string,
    public city: string,
    public zipCode: string
  ) {}
  
  clone(): Address {
    return new Address(this.street, this.city, this.zipCode);
  }
}

class Employee implements Prototype<Employee> {
  constructor(
    public name: string,
    public role: string,
    public address: Address
  ) {}
  
  clone(): Employee {
    // Поверхностное копирование
    // return new Employee(this.name, this.role, this.address);
    
    // Глубокое копирование
    return new Employee(
      this.name,
      this.role,
      this.address.clone()
    );
  }
}

// Использование
const originalAddress = new Address("123 Main St", "New York", "10001");
const originalEmployee = new Employee("John Doe", "Developer", originalAddress);

const clonedEmployee = originalEmployee.clone();
clonedEmployee.name = "Jane Doe";
clonedEmployee.address.street = "456 Oak Ave"; // Это не повлияет на оригинальный адрес

console.log(originalEmployee); // Оригинальный адрес не изменился
console.log(clonedEmployee);   // Изменения в клоне
```

### 5. Одиночка (Singleton)

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connected: boolean = false;
  
  private constructor() {
    // Приватный конструктор
  }
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
  
  connect(): void {
    if (!this.connected) {
      console.log("Подключение к базе данных...");
      this.connected = true;
    } else {
      console.log("Уже подключено к базе данных");
    }
  }
  
  disconnect(): void {
    if (this.connected) {
      console.log("Отключение от базы данных...");
      this.connected = false;
    }
  }
  
  query(sql: string): any {
    if (!this.connected) {
      throw new Error("Не подключено к базе данных");
    }
    console.log(`Выполнение запроса: ${sql}`);
    return { result: "data" };
  }
}

// Использование
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();

console.log(db1 === db2); // true - это один и тот же экземпляр

db1.connect();
db2.query("SELECT * FROM users"); // Работает, так как это тот же экземпляр
```

## Структурные паттерны

### 1. Адаптер (Adapter)

```typescript
// Целевой интерфейс
interface MediaPlayer {
  play(audioType: string, fileName: string): void;
}

// Существующий класс
class AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    console.log(`Воспроизведение VLC файла: ${fileName}`);
  }
  
  playMp4(fileName: string): void {
    console.log(`Воспроизведение MP4 файла: ${fileName}`);
  }
}

// Адаптер
class MediaAdapter implements MediaPlayer {
  private advancedPlayer: AdvancedMediaPlayer;
  
  constructor(audioType: string) {
    this.advancedPlayer = new AdvancedMediaPlayer();
  }
  
  play(audioType: string, fileName: string): void {
    if (audioType.toLowerCase() === "vlc") {
      this.advancedPlayer.playVlc(fileName);
    } else if (audioType.toLowerCase() === "mp4") {
      this.advancedPlayer.playMp4(fileName);
    }
  }
}

// Клиентский класс
class AudioPlayer implements MediaPlayer {
  play(audioType: string, fileName: string): void {
    if (audioType.toLowerCase() === "mp3") {
      console.log(`Воспроизведение MP3 файла: ${fileName}`);
    } else if (audioType.toLowerCase() === "vlc" || audioType.toLowerCase() === "mp4") {
      const adapter = new MediaAdapter(audioType);
      adapter.play(audioType, fileName);
    } else {
      console.log(`Формат ${audioType} не поддерживается`);
    }
  }
}

// Использование
const audioPlayer = new AudioPlayer();
audioPlayer.play("mp3", "song.mp3");
audioPlayer.play("vlc", "movie.vlc");
audioPlayer.play("mp4", "video.mp4");
audioPlayer.play("avi", "film.avi");
```

### 2. Мост (Bridge)

```typescript
// Абстракция
interface Device {
  isEnabled(): boolean;
  enable(): void;
  disable(): void;
  getVolume(): number;
  setVolume(percent: number): void;
  getChannel(): number;
  setChannel(channel: number): void;
}

// Реализация
class TV implements Device {
  private enabled = false;
  private volume = 50;
  private channel = 1;
  
  isEnabled(): boolean {
    return this.enabled;
  }
  
  enable(): void {
    this.enabled = true;
  }
  
  disable(): void {
    this.enabled = false;
  }
  
  getVolume(): number {
    return this.volume;
  }
  
  setVolume(percent: number): void {
    this.volume = percent;
  }
  
  getChannel(): number {
    return this.channel;
  }
  
  setChannel(channel: number): void {
    this.channel = channel;
  }
}

class Radio implements Device {
  private enabled = false;
  private volume = 30;
  private channel = 101.5;
  
  isEnabled(): boolean {
    return this.enabled;
  }
  
  enable(): void {
    this.enabled = true;
  }
  
  disable(): void {
    this.enabled = false;
  }
  
  getVolume(): number {
    return this.volume;
  }
  
  setVolume(percent: number): void {
    this.volume = percent;
  }
  
  getChannel(): number {
    return this.channel;
  }
  
  setChannel(channel: number): void {
    this.channel = channel;
  }
}

// Абстракция с мостом
abstract class RemoteControl {
  protected device: Device;
  
  constructor(device: Device) {
    this.device = device;
  }
  
  togglePower(): void {
    if (this.device.isEnabled()) {
      this.device.disable();
    } else {
      this.device.enable();
    }
  }
  
  volumeUp(): void {
    const currentVolume = this.device.getVolume();
    this.device.setVolume(currentVolume + 10);
  }
  
  volumeDown(): void {
    const currentVolume = this.device.getVolume();
    this.device.setVolume(currentVolume - 10);
  }
  
  channelUp(): void {
    const currentChannel = this.device.getChannel();
    this.device.setChannel(currentChannel + 1);
  }
  
  channelDown(): void {
    const currentChannel = this.device.getChannel();
    this.device.setChannel(currentChannel - 1);
  }
}

class BasicRemote extends RemoteControl {
  constructor(device: Device) {
    super(device);
  }
}

class AdvancedRemote extends RemoteControl {
  constructor(device: Device) {
    super(device);
  }
  
  mute(): void {
    this.device.setVolume(0);
  }
}

// Использование
const tv = new TV();
const radio = new Radio();

const tvRemote = new BasicRemote(tv);
tvRemote.togglePower(); // Включить TV
tvRemote.volumeUp();   // Увеличить громкость
tvRemote.channelUp();  // Следующий канал

const radioRemote = new AdvancedRemote(radio);
radioRemote.togglePower(); // Включить Radio
radioRemote.mute();        // Выключить звук
```

### 3. Компоновщик (Composite)

```typescript
// Компонент
interface FileSystem {
  getName(): string;
  getSize(): number;
  ls(depth?: number): void;
}

// Лист
class File implements FileSystem {
  constructor(private name: string, private size: number) {}
  
  getName(): string {
    return this.name;
  }
  
  getSize(): number {
    return this.size;
  }
  
  ls(depth: number = 0): void {
    console.log(`${'  '.repeat(depth)}- ${this.name} (${this.size} bytes)`);
  }
}

// Контейнер
class Directory implements FileSystem {
  private children: FileSystem[] = [];
  
  constructor(private name: string) {}
  
  getName(): string {
    return this.name;
  }
  
  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }
  
  add(component: FileSystem): void {
    this.children.push(component);
  }
  
  remove(component: FileSystem): void {
    const index = this.children.indexOf(component);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  ls(depth: number = 0): void {
    console.log(`${'  '.repeat(depth)}+ ${this.name}/`);
    this.children.forEach(child => child.ls(depth + 1));
  }
}

// Использование
const root = new Directory("root");
const documents = new Directory("Documents");
const pictures = new Directory("Pictures");

const file1 = new File("readme.txt", 1024);
const file2 = new File("photo.jpg", 2048000);
const file3 = new File("image.png", 1024000);

documents.add(file1);
pictures.add(file2);
pictures.add(file3);

root.add(documents);
root.add(pictures);

root.ls(); // Вывести структуру файловой системы
console.log(`Общий размер: ${root.getSize()} bytes`);
```

### 4. Декоратор (Decorator)

```typescript
// Компонент
interface Coffee {
  cost(): number;
  description(): string;
}

// Конкретный компонент
class SimpleCoffee implements Coffee {
  cost(): number {
    return 5;
  }
  
  description(): string {
    return "Простая чашка кофе";
  }
}

// Абстрактный декоратор
abstract class CoffeeDecorator implements Coffee {
  protected coffee: Coffee;
  
  constructor(coffee: Coffee) {
    this.coffee = coffee;
  }
  
  abstract cost(): number;
  abstract description(): string;
}

// Конкретные декораторы
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1;
  }
  
  description(): string {
    return `${this.coffee.description()}, с молоком`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.5;
  }
  
  description(): string {
    return `${this.coffee.description()}, с сахаром`;
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1.5;
  }
  
  description(): string {
    return `${this.coffee.description()}, с взбитыми сливками`;
  }
}

// Использование
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
```

### 5. Фасад (Facade)

```typescript
// Подсистемы
class CPU {
  freeze(): void {
    console.log("CPU: Заморозка");
  }
  
  jump(position: number): void {
    console.log(`CPU: Перейти к ${position}`);
  }
  
  execute(): void {
    console.log("CPU: Выполнение");
  }
}

class Memory {
  load(position: number, data: string): string {
    console.log(`Memory: Загрузка ${data} в ${position}`);
    return data;
  }
}

class HardDrive {
  read(lba: number, size: number): string {
    console.log(`HardDrive: Чтение ${size} байт с ${lba}`);
    return "Boot data";
  }
}

// Фасад
class ComputerFacade {
  private cpu: CPU;
  private memory: Memory;
  private hardDrive: HardDrive;
  
  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }
  
  start(): void {
    console.log("Компьютер запускается...");
    this.cpu.freeze();
    this.hardDrive.read(0, 1024);
    this.memory.load(0, "Boot data");
    this.cpu.jump(0);
    this.cpu.execute();
    console.log("Компьютер успешно запущен!");
  }
  
  shutdown(): void {
    console.log("Компьютер выключается...");
    // Реализация выключения
    console.log("Компьютер выключен.");
  }
}

// Использование
const computer = new ComputerFacade();
computer.start();
computer.shutdown();
```

### 6. Заместитель (Proxy)

```typescript
interface Image {
  display(): void;
}

class RealImage implements Image {
  private fileName: string;
  
  constructor(fileName: string) {
    this.fileName = fileName;
    this.loadFromDisk();
  }
  
  private loadFromDisk(): void {
    console.log(`Загрузка изображения: ${this.fileName}`);
  }
  
  display(): void {
    console.log(`Отображение изображения: ${this.fileName}`);
  }
}

class ProxyImage implements Image {
  private realImage: RealImage | null = null;
  private fileName: string;
  
  constructor(fileName: string) {
    this.fileName = fileName;
  }
  
  display(): void {
    if (this.realImage === null) {
      this.realImage = new RealImage(this.fileName);
    }
    this.realImage.display();
  }
}

// Использование
const image1 = new ProxyImage("photo1.jpg");
const image2 = new ProxyImage("photo2.jpg");

// Изображения не загружены в память
image1.display(); // Теперь изображение загружается и отображается
image1.display(); // Изображение уже загружено, просто отображается

image2.display(); // Загрузка и отображение второго изображения
```

## Поведенческие паттерны

### 1. Цепочка обязанностей (Chain of Responsibility)

```typescript
abstract class Handler {
  protected nextHandler: Handler | null = null;
  
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

class MonkeyHandler extends Handler {
  handle(request: string): string | null {
    if (request === "Banana") {
      return `Monkey: I'll eat the ${request}.`;
    }
    return super.handle(request);
  }
}

class SquirrelHandler extends Handler {
  handle(request: string): string | null {
    if (request === "Nut") {
      return `Squirrel: I'll eat the ${request}.`;
    }
    return super.handle(request);
  }
}

class DogHandler extends Handler {
  handle(request: string): string | null {
    if (request === "MeatBall") {
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

console.log(monkey.handle("Nut"));
console.log(monkey.handle("MeatBall"));
console.log(monkey.handle("Banana"));
console.log(monkey.handle("Cup of coffee"));
```

### 2. Команда (Command)

```typescript
interface Command {
  execute(): void;
  undo(): void;
}

class Calculator {
  private currentValue = 0;
  
  add(value: number): void {
    this.currentValue += value;
    console.log(`Добавлено ${value}, текущее значение: ${this.currentValue}`);
  }
  
  subtract(value: number): void {
    this.currentValue -= value;
    console.log(`Вычтено ${value}, текущее значение: ${this.currentValue}`);
  }
  
  getCurrentValue(): number {
    return this.currentValue;
  }
  
  setCurrentValue(value: number): void {
    this.currentValue = value;
  }
}

class AddCommand implements Command {
  private previousValue: number;
  
  constructor(private calculator: Calculator, private value: number) {
    this.previousValue = calculator.getCurrentValue();
  }
  
  execute(): void {
    this.calculator.add(this.value);
  }
  
  undo(): void {
    this.calculator.setCurrentValue(this.previousValue);
  }
}

class SubtractCommand implements Command {
  private previousValue: number;
  
  constructor(private calculator: Calculator, private value: number) {
    this.previousValue = calculator.getCurrentValue();
  }
  
  execute(): void {
    this.calculator.subtract(this.value);
  }
  
  undo(): void {
    this.calculator.setCurrentValue(this.previousValue);
  }
}

class CalculatorInvoker {
  private history: Command[] = [];
  
  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
  }
  
  undoLastCommand(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// Использование
const calculator = new Calculator();
const invoker = new CalculatorInvoker();

invoker.executeCommand(new AddCommand(calculator, 5));
invoker.executeCommand(new AddCommand(calculator, 3));
invoker.executeCommand(new SubtractCommand(calculator, 2));

console.log(`Текущее значение: ${calculator.getCurrentValue()}`); // 6

invoker.undoLastCommand(); // Отменить вычитание 2
console.log(`После отмены: ${calculator.getCurrentValue()}`); // 8
```

### 3. Итератор (Iterator)

```typescript
interface Iterator<T> {
  next(): { value: T; done: boolean };
  hasNext(): boolean;
}

interface Iterable<T> {
  createIterator(): Iterator<T>;
}

class WordCollection implements Iterable<string> {
  private items: string[] = [];
  
  add(item: string): void {
    this.items.push(item);
  }
  
  createIterator(): Iterator<string> {
    return new AlphabeticalOrderIterator(this);
  }
  
  getItems(): string[] {
    return this.items;
  }
}

class AlphabeticalOrderIterator implements Iterator<string> {
  private position = 0;
  
  constructor(private collection: WordCollection) {}
  
  hasNext(): boolean {
    return this.position < this.collection.getItems().length;
  }
  
  next(): { value: string; done: boolean } {
    if (this.hasNext()) {
      const value = this.collection.getItems()[this.position];
      this.position++;
      return { value, done: false };
    } else {
      return { value: undefined as any, done: true };
    }
  }
}

// Использование
const collection = new WordCollection();
collection.add("First");
collection.add("Second");
collection.add("Third");

const iterator = collection.createIterator();
while (iterator.hasNext()) {
  console.log(iterator.next().value);
}
```

### 4. Посредник (Mediator)

```typescript
interface Component {
  setMediator(mediator: Mediator): void;
  getName(): string;
}

interface Mediator {
  notify(sender: Component, event: string): void;
}

class BaseComponent implements Component {
  protected mediator: Mediator | null = null;
  
  constructor(protected name: string) {}
  
  setMediator(mediator: Mediator): void {
    this.mediator = mediator;
  }
  
  getName(): string {
    return this.name;
  }
}

class Button extends BaseComponent {
  click(): void {
    console.log(`${this.name} clicked`);
    this.mediator?.notify(this, "click");
  }
}

class TextBox extends BaseComponent {
  private text: string = "";
  
  setText(text: string): void {
    this.text = text;
    console.log(`${this.name} text set to: ${text}`);
    this.mediator?.notify(this, "textChange");
  }
  
  getText(): string {
    return this.text;
  }
}

class FormMediator implements Mediator {
  private components: Component[] = [];
  
  addComponent(component: Component): void {
    component.setMediator(this);
    this.components.push(component);
  }
  
  notify(sender: Component, event: string): void {
    console.log(`Mediator: ${sender.getName()} triggered ${event}`);
    
    if (sender instanceof Button && event === "click") {
      // Когда кнопка нажата, очистить текстовое поле
      const textBox = this.components.find(c => c instanceof TextBox) as TextBox | undefined;
      textBox?.setText("");
    }
  }
}

// Использование
const mediator = new FormMediator();
const button = new Button("SubmitButton");
const textBox = new TextBox("InputField");

mediator.addComponent(button);
mediator.addComponent(textBox);

textBox.setText("Some text");
button.click(); // Это очистит текстовое поле
```

### 5. Хранитель (Memento)

```typescript
class EditorMemento {
  constructor(private content: string, private cursorPosition: number) {}
  
  getContent(): string {
    return this.content;
  }
  
  getCursorPosition(): number {
    return this.cursorPosition;
  }
}

class Editor {
  private content: string = "";
  private cursorPosition: number = 0;
  
  type(words: string): void {
    this.content += words;
    this.cursorPosition += words.length;
  }
  
  getContent(): string {
    return this.content;
  }
  
  getCursorPosition(): number {
    return this.cursorPosition;
  }
  
  save(): EditorMemento {
    return new EditorMemento(this.content, this.cursorPosition);
  }
  
  restore(memento: EditorMemento): void {
    this.content = memento.getContent();
    this.cursorPosition = memento.getCursorPosition();
  }
}

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

editor.type("Первая строка.\n");
history.push(editor.save());

editor.type("Вторая строка.\n");
history.push(editor.save());

editor.type("Третья строка.\n");

console.log("Текущий контент:");
console.log(editor.getContent());

// Отменить последнее действие
if (!history.isEmpty()) {
  editor.restore(history.pop()!);
}

console.log("\nПосле отмены:");
console.log(editor.getContent());
```

### 6. Наблюдатель (Observer)

```typescript
interface Observer {
  update(subject: Subject): void;
}

interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

class NewsAgency implements Subject {
  private observers: Observer[] = [];
  private news: string = "";
  
  attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (!isExist) {
      this.observers.push(observer);
    }
  }
  
  detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex !== -1) {
      this.observers.splice(observerIndex, 1);
    }
  }
  
  notify(): void {
    for (const observer of this.observers) {
      observer.update(this);
    }
  }
  
  setNews(news: string): void {
    this.news = news;
    this.notify();
  }
  
  getNews(): string {
    return this.news;
  }
}

class NewsChannel implements Observer {
  private news: string = "";
  
  update(subject: Subject): void {
    if (subject instanceof NewsAgency) {
      this.news = subject.getNews();
      console.log(`Канал получил новости: ${this.news}`);
    }
  }
}

// Использование
const newsAgency = new NewsAgency();
const newsChannel1 = new NewsChannel();
const newsChannel2 = new NewsChannel();

newsAgency.attach(newsChannel1);
newsAgency.attach(newsChannel2);

newsAgency.setNews("Новые технологии в TypeScript!");

newsAgency.detach(newsChannel2);
newsAgency.setNews("Вторая новость - только для первого канала");
```

### 7. Состояние (State)

```typescript
interface State {
  handle(context: AudioPlayer): void;
}

class AudioPlayer {
  private state: State;
  
  constructor() {
    this.state = new ReadyState(this);
  }
  
  changeState(state: State): void {
    this.state = state;
  }
  
  getState(): State {
    return this.state;
  }
  
  play(): void {
    this.state.handle(this);
  }
  
  lock(): void {
    console.log("Плеер заблокирован");
    this.changeState(new LockedState(this));
  }
  
  next(): void {
    console.log("Следующий трек");
  }
  
  prev(): void {
    console.log("Предыдущий трек");
  }
}

class ReadyState implements State {
  constructor(private player: AudioPlayer) {}
  
  handle(context: AudioPlayer): void {
    console.log("Готов к воспроизведению");
    context.changeState(new PlayingState(context));
  }
}

class PlayingState implements State {
  constructor(private player: AudioPlayer) {}
  
  handle(context: AudioPlayer): void {
    console.log("Пауза воспроизведения");
    context.changeState(new ReadyState(context));
  }
}

class LockedState implements State {
  constructor(private player: AudioPlayer) {}
  
  handle(context: AudioPlayer): void {
    console.log("Плеер заблокирован, сначала разблокируйте");
  }
}

// Использование
const player = new AudioPlayer();
player.play(); // Перейти в состояние воспроизведения
player.play(); // Поставить на паузу
player.play(); // Снова начать воспроизведение
player.lock(); // Заблокировать плеер
player.play(); // Попытка воспроизведения в заблокированном состоянии
```

### 8. Стратегия (Strategy)

```typescript
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardStrategy implements PaymentStrategy {
  constructor(private cardNumber: string, private cvv: string) {}
  
  pay(amount: number): void {
    console.log(`Оплата ${amount} руб. с кредитной карты ${this.cardNumber}`);
  }
}

class PayPalStrategy implements PaymentStrategy {
  constructor(private email: string) {}
  
  pay(amount: number): void {
    console.log(`Оплата ${amount} руб. через PayPal аккаунт ${this.email}`);
  }
}

class BitcoinStrategy implements PaymentStrategy {
  constructor(private wallet: string) {}
  
  pay(amount: number): void {
    console.log(`Оплата ${amount} руб. с Bitcoin кошелька ${this.wallet}`);
  }
}

class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  private paymentStrategy: PaymentStrategy;
  
  addItem(item: { name: string; price: number }): void {
    this.items.push(item);
  }
  
  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
  }
  
  processPayment(): void {
    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    this.paymentStrategy.pay(total);
  }
}

// Использование
const cart = new ShoppingCart();
cart.addItem({ name: "Книга", price: 500 });
cart.addItem({ name: "Кофе", price: 300 });

cart.setPaymentStrategy(new CreditCardStrategy("1234-5678-9012-3456", "123"));
cart.processPayment();

cart.setPaymentStrategy(new PayPalStrategy("user@example.com"));
cart.processPayment();
```

### 9. Шаблонный метод (Template Method)

```typescript
abstract class DataProcessor {
  // Шаблонный метод
  process(): void {
    this.loadData();
    this.validateData();
    this.transformData();
    this.saveData();
  }
  
  protected loadData(): void {
    console.log("Загрузка данных");
  }
  
  protected validateData(): void {
    console.log("Валидация данных");
  }
  
  protected abstract transformData(): void;
  
  protected saveData(): void {
    console.log("Сохранение данных");
  }
}

class XMLDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log("Трансформация XML данных");
  }
}

class JSONDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log("Трансформация JSON данных");
  }
}

class CSVDataProcessor extends DataProcessor {
  protected transformData(): void {
    console.log("Трансформация CSV данных");
  }
}

// Использование
const xmlProcessor = new XMLDataProcessor();
xmlProcessor.process();

const jsonProcessor = new JSONDataProcessor();
jsonProcessor.process();

const csvProcessor = new CSVDataProcessor();
csvProcessor.process();
```

### 10. Посетитель (Visitor)

```typescript
interface Visitor {
  visitElementA(element: ElementA): void;
  visitElementB(element: ElementB): void;
}

interface Element {
  accept(visitor: Visitor): void;
}

class ElementA implements Element {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  accept(visitor: Visitor): void {
    visitor.visitElementA(this);
  }
  
  operationA(): string {
    return `ElementA: ${this.name}`;
  }
}

class ElementB implements Element {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  accept(visitor: Visitor): void {
    visitor.visitElementB(this);
  }
  
  operationB(): string {
    return `ElementB: ${this.name}`;
  }
}

class ConcreteVisitor implements Visitor {
  visitElementA(element: ElementA): void {
    console.log(`Visitor: Обработка ${element.operationA()}`);
  }
  
  visitElementB(element: ElementB): void {
    console.log(`Visitor: Обработка ${element.operationB()}`);
  }
}

class ConcreteVisitor2 implements Visitor {
  visitElementA(element: ElementA): void {
    console.log(`Visitor2: Обработка ${element.operationA()} с дополнительной логикой`);
  }
  
  visitElementB(element: ElementB): void {
    console.log(`Visitor2: Обработка ${element.operationB()} с дополнительной логикой`);
  }
}

// Использование
const elements: Element[] = [new ElementA("A1"), new ElementB("B1"), new ElementA("A2")];
const visitor1 = new ConcreteVisitor();
const visitor2 = new ConcreteVisitor2();

elements.forEach(element => element.accept(visitor1));
console.log("---");
elements.forEach(element => element.accept(visitor2));
```

## Практические примеры из реальной разработки

### 1. Паттерн "Репозиторий" с использованием других паттернов

```typescript
// Интерфейс сущности
interface Entity {
  id: number;
}

// Интерфейс репозитория
interface Repository<T extends Entity> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  update(id: number, entity: Partial<T>): Promise<T | null>;
  delete(id: number): Promise<boolean>;
}

// Базовый класс репозитория с использованием стратегии хранения
abstract class BaseRepository<T extends Entity> implements Repository<T> {
  protected storageStrategy: StorageStrategy<T>;
  
  constructor(storageStrategy: StorageStrategy<T>) {
    this.storageStrategy = storageStrategy;
  }
  
  async findById(id: number): Promise<T | null> {
    return this.storageStrategy.findById(id);
  }
  
  async findAll(): Promise<T[]> {
    return this.storageStrategy.findAll();
  }
  
  async save(entity: T): Promise<T> {
    if (!entity.id) {
      entity.id = await this.generateId();
    }
    return this.storageStrategy.save(entity);
  }
  
  async update(id: number, entity: Partial<T>): Promise<T | null> {
    const existing = await this.findById(id);
    if (!existing) return null;
    
    const updatedEntity = { ...existing, ...entity } as T;
    return this.storageStrategy.update(id, updatedEntity);
  }
  
  async delete(id: number): Promise<boolean> {
    return this.storageStrategy.delete(id);
  }
  
  protected abstract generateId(): Promise<number>;
}

// Стратегии хранения
interface StorageStrategy<T extends Entity> {
  findById(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  update(id: number, entity: T): Promise<T | null>;
  delete(id: number): Promise<boolean>;
}

class InMemoryStorage<T extends Entity> implements StorageStrategy<T> {
  private data: Map<number, T> = new Map();
  private nextId = 1;
  
  async findById(id: number): Promise<T | null> {
    return this.data.get(id) || null;
  }
  
  async findAll(): Promise<T[]> {
    return Array.from(this.data.values());
  }
  
  async save(entity: T): Promise<T> {
    if (!entity.id) {
      entity.id = this.nextId++;
    }
    this.data.set(entity.id, { ...entity });
    return { ...entity };
  }
  
  async update(id: number, entity: T): Promise<T | null> {
    if (!this.data.has(id)) return null;
    this.data.set(id, { ...entity });
    return { ...entity };
  }
  
  async delete(id: number): Promise<boolean> {
    return this.data.delete(id);
  }
}

// Конкретный репозиторий
interface User extends Entity {
  name: string;
  email: string;
}

class UserRepository extends BaseRepository<User> {
  protected async generateId(): Promise<number> {
    // В реальном приложении это может быть вызовом API или запросом к БД
    return Date.now();
  }
}

// Использование
const userRepo = new UserRepository(new InMemoryStorage<User>());

async function example() {
  const newUser = await userRepo.save({ id: 0, name: "John", email: "john@example.com" });
  console.log("Созданный пользователь:", newUser);
  
  const foundUser = await userRepo.findById(newUser.id);
  console.log("Найденный пользователь:", foundUser);
  
  const updatedUser = await userRepo.update(newUser.id, { name: "John Doe" });
  console.log("Обновленный пользователь:", updatedUser);
}
```

### 2. Комбинирование паттернов для архитектуры приложения

```typescript
// Комбинирование Фабрики, Стратегии и Наблюдателя
interface NotificationService {
  send(message: string, recipient: string): Promise<void>;
}

class EmailNotificationService implements NotificationService {
  async send(message: string, recipient: string): Promise<void> {
    console.log(`Отправка email "${message}" на ${recipient}`);
  }
}

class SMSNotificationService implements NotificationService {
  async send(message: string, recipient: string): Promise<void> {
    console.log(`Отправка SMS "${message}" на ${recipient}`);
  }
}

// Стратегия уведомлений
class NotificationStrategy {
  constructor(private service: NotificationService) {}
  
  async notify(message: string, recipient: string): Promise<void> {
    await this.service.send(message, recipient);
  }
}

// Наблюдатель за событиями
interface EventObserver {
  update(event: string, data: any): void;
}

class NotificationObserver implements EventObserver {
  private strategies: Map<string, NotificationStrategy> = new Map();
  
  addStrategy(event: string, strategy: NotificationStrategy): void {
    this.strategies.set(event, strategy);
  }
  
  async update(event: string, data: any): Promise<void> {
    const strategy = this.strategies.get(event);
    if (strategy) {
      await strategy.notify(data.message, data.recipient);
    }
  }
}

// Субъект событий
class EventPublisher {
  private observers: EventObserver[] = [];
  
  addObserver(observer: EventObserver): void {
    this.observers.push(observer);
  }
  
  notifyObservers(event: string, data: any): void {
    this.observers.forEach(observer => observer.update(event, data));
  }
  
  triggerEvent(event: string, data: any): void {
    console.log(`Событие "${event}" с данными:`, data);
    this.notifyObservers(event, data);
  }
}

// Фабрика стратегий уведомлений
class NotificationStrategyFactory {
  static create(type: 'email' | 'sms'): NotificationStrategy {
    let service: NotificationService;
    
    switch (type) {
      case 'email':
        service = new EmailNotificationService();
        break;
      case 'sms':
        service = new SMSNotificationService();
        break;
      default:
        throw new Error(`Неизвестный тип уведомления: ${type}`);
    }
    
    return new NotificationStrategy(service);
  }
}

// Использование
const publisher = new EventPublisher();
const notificationObserver = new NotificationObserver();

// Настройка стратегий для разных событий
notificationObserver.addStrategy('user_registered', NotificationStrategyFactory.create('email'));
notificationObserver.addStrategy('password_reset', NotificationStrategyFactory.create('sms'));

publisher.addObserver(notificationObserver);

// Триггер событий
publisher.triggerEvent('user_registered', {
  message: 'Добро пожаловать!',
  recipient: 'user@example.com'
});

publisher.triggerEvent('password_reset', {
  message: 'Код сброса: 123456',
  recipient: '+1234567890'
});
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки при использовании паттернов

```typescript
// Ошибка 1: Избыточное использование паттернов
// ПЛОХО: Слишком сложная архитектура для простой задачи
class SimpleAdder {
  // Не нужно применять паттерн "Команда" для простого сложения
  add(a: number, b: number): number {
    return a + b; // Просто вернуть результат
  }
}

// Ошибка 2: Неправильное наследование
// ПЛОХО: Нарушение принципа подстановки Лисков
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  
  setWidth(width: number): void {
    this.width = width;
  }
  
  setHeight(height: number): void {
    this.height = height;
  }
  
  getArea(): number {
    return this.width * this.height;
  }
}

// Это нарушает принцип подстановки Лисков
class Square extends Rectangle {
  setWidth(width: number): void {
    super.setWidth(width);
    super.setHeight(width); // У квадрата ширина = высота
  }
  
  setHeight(height: number): void {
    super.setWidth(height); // У квадрата ширина = высота
    super.setHeight(height);
  }
}
```

### 2. Лучшие практики

```typescript
// 1. Используйте паттерны осознанно - только когда они решают реальную проблему
// 2. Следуйте принципам SOLID
// 3. Поддерживайте простоту там, где это возможно

// ХОРОШО: Применение паттерна только когда это оправдано
interface Logger {
  log(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`LOG: ${message}`);
  }
}

class FileLogger implements Logger {
  log(message: string): void {
    // Логика записи в файл
    console.log(`FILE LOG: ${message}`);
  }
}

// Паттерн Стратегия оправдан для выбора стратегии логирования
class UserService {
  constructor(private logger: Logger) {}
  
  createUser(name: string): void {
    // Логика создания пользователя
    this.logger.log(`Создан пользователь: ${name}`);
  }
}

// Зависимость инъектируется, что позволяет легко тестировать и изменять стратегию
const consoleUserService = new UserService(new ConsoleLogger());
const fileUserService = new UserService(new FileLogger());
```

## Лучшие практики

1. **Используйте паттерны только когда они решают реальную проблему** - не применяйте паттерны ради применения
2. **Следуйте принципам SOLID** - паттерны должны улучшать архитектуру, а не усложнять её
3. **Предпочитайте простоту** - если простое решение работает, не усложняйте его паттерном
4. **Тестируйте паттерны** - убедитесь, что паттерн работает как ожидается
5. **Документируйте использование паттернов** - особенно когда это неочевидно
6. **Избегайте избыточной абстракции** - паттерны не должны создавать ненужную сложность

## Связи с другими концепциями

- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - основа для многих паттернов
- [[ts/generics/TS Generics|Обобщения]] - для создания универсальных паттернов
- [[ts/decorators/advanced-decorators-comprehensive|Декораторы]] - альтернатива некоторым структурным паттернам
- [[architecture/component-architecture]] - архитектурные аспекты применения паттернов
- [[design-patterns/principles]] - основы проектирования, на которых строятся паттерны