# Structural Patterns

Структурные паттерны в TypeScript описывают способы компоновки объектов и классов для формирования крупных структур. Они сосредоточены на том, как части системы соединяются друг с другом, обеспечивая гибкость и эффективность структуры программы.

## Основные структурные паттерны

### [Adapter Pattern](adapter.md)
Преобразует интерфейс одного класса в интерфейс, который ожидают клиенты.

### [Bridge Pattern](bridge.md)
Разделяет абстракцию и реализацию, позволяя изменять их независимо.

### [Composite Pattern](composite.md)
Составляет объекты в древовидные структуры, позволяя работать с ними как с отдельными объектами.

### [Decorator Pattern](decorator.md)
Добавляет обязанности объекту динамически, без изменения кода других объектов того же класса.

### [Facade Pattern](facade.md)
Предоставляет унифицированный интерфейс к подсистеме, состоящей из множества интерфейсов.

### [Flyweight Pattern](flyweight.md)
Использует разделение для эффективной поддержки большого количества мелких объектов.

### [Proxy Pattern](proxy.md)
Предоставляет заменитель или заполнитель для другого объекта для контроля доступа к нему.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Structural Design Patterns                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Adapter       │               │      Bridge              │                                    │  Composite      │
  │  Pattern        │               │  Pattern               │                                    │  Pattern        │
  │                 │               │                          │                                    │                 │
  │ - Interface     │◄──────────────┤ - Abstraction &          │─────────────────────────────────────►│ - Component     │
  │   Translation   │               │   Implementation         │                                    │   Hierarchies   │
  │ - Legacy        │               │ - Decoupling             │                                    │ - Recursive     │
  │   Integration   │               │ - Runtime Selection      │                                    │   Composition   │
  │ - Type          │               │ - Strategy Integration   │                                    │ - Uniform       │
  │   Compatibility │               │                          │                                    │   Interface     │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │         Decorator               │
        │                           │                               │
        │                           │ - Dynamic Behavior Addition     │
        │                           │ - Wrapper Pattern               │
        │                           │ - AOP-like Functionality        │
        │                           │ - Middleware Pattern            │
        │                           │ - Cross-cutting Concerns        │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │          Proxy                  │
        │                           │                               │
        │                           │ - Access Control                │
        │                           │ - Lazy Loading                  │
        │                           │ - Virtual Proxy                 │
        │                           │ - Protection Proxy              │
        │                           │ - Smart Reference               │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                           Facade                                    │
        │                           │                                                                 │
        │                           │ - Simplified Interface                                                │
        │                           │ - Subsystem Abstraction                                               │
        │                           │ - High-level Operations                                               │
        │                           │ - Integration Layer                                                   │
        │                           │ - Service Orchestration                                               │
        │                           └─────────────────────────────────────────────────────────────────────────┘
```

## Adapter Pattern

### Базовый адаптер
```typescript
// Целевой интерфейс
interface MediaPlayer {
  play(audioType: string, fileName: string): void;
}

// Конкретная реализация
class AudioPlayer implements MediaPlayer {
  play(audioType: string, fileName: string): void {
    audioType = audioType.toLowerCase();
    
    if (audioType === "mp3") {
      console.log(`Playing MP3 file: ${fileName}`);
    } else {
      console.log(`Unsupported format: ${audioType}`);
    }
  }
}

// Адаптируемый класс
class AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    console.log(`Playing VLC file: ${fileName}`);
  }
  
  playMp4(fileName: string): void {
    console.log(`Playing MP4 file: ${fileName}`);
  }
}

// Адаптер
class MediaAdapter implements MediaPlayer {
  private advancedMusicPlayer: AdvancedMediaPlayer;
  
  constructor(audioType: string) {
    if (audioType.toLowerCase() === "vlc") {
      this.advancedMusicPlayer = new VlcPlayer();
    } else if (audioType.toLowerCase() === "mp4") {
      this.advancedMusicPlayer = new Mp4Player();
    }
  }
  
  play(audioType: string, fileName: string): void {
    audioType = audioType.toLowerCase();
    
    if (audioType === "vlc") {
      this.advancedMusicPlayer.playVlc(fileName);
    } else if (audioType === "mp4") {
      this.advancedMusicPlayer.playMp4(fileName);
    }
  }
}

class VlcPlayer extends AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    console.log(`Playing VLC file: ${fileName}`);
  }
  
  playMp4(fileName: string): void {
    // Не реализовано в VlcPlayer
  }
}

class Mp4Player extends AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    // Не реализовано в Mp4Player
  }
  
  playMp4(fileName: string): void {
    console.log(`Playing MP4 file: ${fileName}`);
  }
}

// Использование
class AudioPlayer implements MediaPlayer {
  private mediaAdapter: MediaAdapter | null = null;
  
  play(audioType: string, fileName: string): void {
    audioType = audioType.toLowerCase();
    
    if (audioType === "mp3") {
      console.log(`Playing MP3 file: ${fileName}`);
    } else if (audioType === "vlc" || audioType === "mp4") {
      this.mediaAdapter = new MediaAdapter(audioType);
      this.mediaAdapter.play(audioType, fileName);
    } else {
      console.log(`Invalid media: ${audioType} format not supported`);
    }
  }
}
```

### Адаптер с обобщениями
```typescript
// Адаптер для преобразования интерфейсов
interface OldAPI {
  legacyMethod(data: string[]): number;
}

interface NewAPI {
  modernMethod(data: string[]): Promise<number>;
}

class ModernAPIAdapter implements NewAPI {
  constructor(private legacyAPI: OldAPI) {}
  
  async modernMethod(data: string[]): Promise<number> {
    const result = this.legacyAPI.legacyMethod(data);
    return Promise.resolve(result);
  }
}

// Универсальный адаптер
interface Adapter<T, U> {
  adapt(input: T): U;
}

class StringToNumberAdapter implements Adapter<string, number> {
  adapt(input: string): number {
    return parseFloat(input) || 0;
  }
}

class NumberToStringAdapter implements Adapter<number, string> {
  adapt(input: number): string {
    return input.toString();
  }
}

// Использование с обобщениями
class UniversalConverter<T, U> {
  constructor(private adapter: Adapter<T, U>) {}
  
  convert(value: T): U {
    return this.adapter.adapt(value);
  }
}

const converter = new UniversalConverter(new StringToNumberAdapter());
const result = converter.convert("42"); // 42 (number)
```

## Bridge Pattern

### Классический bridge
```typescript
// Абстракция
abstract class Device {
  protected volume: number = 30;
  protected channel: number = 1;
  
  constructor(protected remote: RemoteControl) {}
  
  abstract turnOn(): void;
  abstract turnOff(): void;
  
  setVolume(volume: number): void {
    this.volume = volume;
    this.remote.setVolume(volume);
  }
  
  setChannel(channel: number): void {
    this.channel = channel;
    this.remote.setChannel(channel);
  }
}

// Реализация
interface RemoteControl {
  setVolume(volume: number): void;
  setChannel(channel: number): void;
  power(device: Device): void;
}

class BasicRemote implements RemoteControl {
  setVolume(volume: number): void {
    console.log(`Setting volume to ${volume}`);
  }
  
  setChannel(channel: number): void {
    console.log(`Setting channel to ${channel}`);
  }
  
  power(device: Device): void {
    console.log("Powering device...");
  }
}

class AdvancedRemote implements RemoteControl {
  setVolume(volume: number): void {
    console.log(`Setting volume to ${volume} with surround sound`);
  }
  
  setChannel(channel: number): void {
    console.log(`Setting channel to ${channel} with favorites`);
  }
  
  power(device: Device): void {
    console.log("Powering and muting other devices...");
  }
  
  mute(): void {
    console.log("Muting TV");
  }
  
  nextChannel(): void {
    console.log("Going to next channel");
  }
}

// Конкретные устройства
class TV extends Device {
  turnOn(): void {
    console.log("TV is ON");
    this.remote.power(this);
  }
  
  turnOff(): void {
    console.log("TV is OFF");
  }
}

class Radio extends Device {
  turnOn(): void {
    console.log("Radio is ON");
    this.remote.power(this);
  }
  
  turnOff(): void {
    console.log("Radio is OFF");
  }
}

// Использование
const tv = new TV(new BasicRemote());
tv.turnOn();
tv.setVolume(50);

const radio = new Radio(new AdvancedRemote());
radio.turnOn();
radio.setChannel(10);
```

### Bridge с обобщениями
```typescript
// Абстракция с обобщениями
interface Renderer<T> {
  render(element: T): string;
}

interface Component<T, R extends Renderer<T>> {
  renderer: R;
  data: T;
  
  render(): string;
}

// Конкретная реализация рендерера
class HtmlRenderer<T> implements Renderer<T> {
  render(element: T): string {
    return `<div>${JSON.stringify(element)}</div>`;
  }
}

class JsonRenderer<T> implements Renderer<T> {
  render(element: T): string {
    return JSON.stringify(element);
  }
}

// Конкретные компоненты
class UserCardComponent implements Component<{ name: string; email: string }, HtmlRenderer<{ name: string; email: string }>> {
  renderer: HtmlRenderer<{ name: string; email: string }>;
  
  constructor(public data: { name: string; email: string }) {
    this.renderer = new HtmlRenderer();
  }
  
  render(): string {
    return this.renderer.render(this.data);
  }
}

class UserJsonComponent implements Component<{ name: string; email: string }, JsonRenderer<{ name: string; email: string }>> {
  renderer: JsonRenderer<{ name: string; email: string }>;
  
  constructor(public data: { name: string; email: string }) {
    this.renderer = new JsonRenderer();
  }
  
  render(): string {
    return this.renderer.render(this.data);
  }
}
```

## Composite Pattern

### Иерархия компонентов
```typescript
// Базовый компонент
interface FileSystemItem {
  name: string;
  getSize(): number;
  render(): string;
}

// Листовой элемент
class File implements FileSystemItem {
  constructor(
    public name: string,
    private size: number
  ) {}
  
  getSize(): number {
    return this.size;
  }
  
  render(): string {
    return `📄 ${this.name} (${this.size} bytes)`;
  }
}

// Компонент-контейнер
class Directory implements FileSystemItem {
  private children: FileSystemItem[] = [];
  
  constructor(public name: string) {}
  
  add(item: FileSystemItem): void {
    this.children.push(item);
  }
  
  remove(item: FileSystemItem): void {
    const index = this.children.indexOf(item);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }
  
  render(): string {
    const childOutput = this.children.map(child => `  ${child.render()}`).join('\n');
    return `📁 ${this.name} (${this.getSize()} bytes)\n${childOutput}`;
  }
}

// Использование
const root = new Directory("root");
const docs = new Directory("documents");
const photos = new Directory("photos");

const resume = new File("resume.pdf", 1024);
const photo = new File("vacation.jpg", 2048);

docs.add(resume);
photos.add(photo);

root.add(docs);
root.add(photos);

console.log(root.render());
// Выводит:
// 📁 root (3072 bytes)
//   📁 documents (1024 bytes)
//     📄 resume.pdf (1024 bytes)
//   📁 photos (2048 bytes)
//     📄 vacation.jpg (2048 bytes)

console.log(`Total size: ${root.getSize()} bytes`); // Total size: 3072 bytes
```

### Универсальный композитный паттерн
```typescript
interface Component<T> {
  add(child: Component<T>): void;
  remove(child: Component<T>): void;
  getChildren(): Component<T>[];
  getValue(): T;
  isComposite(): boolean;
}

class Leaf<T> implements Component<T> {
  constructor(private value: T) {}
  
  add(child: Component<T>): void {
    throw new Error("Cannot add to leaf component");
  }
  
  remove(child: Component<T>): void {
    throw new Error("Cannot remove from leaf component");
  }
  
  getChildren(): Component<T>[] {
    return [];
  }
  
  getValue(): T {
    return this.value;
  }
  
  isComposite(): boolean {
    return false;
  }
}

class Composite<T> implements Component<T> {
  private children: Component<T>[] = [];
  private value: T;
  
  constructor(value: T) {
    this.value = value;
  }
  
  add(child: Component<T>): void {
    this.children.push(child);
  }
  
  remove(child: Component<T>): void {
    const index = this.children.indexOf(child);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  getChildren(): Component<T>[] {
    return [...this.children];
  }
  
  getValue(): T {
    return this.value;
  }
  
  isComposite(): boolean {
    return true;
  }
}

// Использование
const root = new Composite("root");
const branch = new Composite("branch");
const leaf1 = new Leaf("leaf1");
const leaf2 = new Leaf("leaf2");

root.add(branch);
branch.add(leaf1);
branch.add(leaf2);

console.log(`Root is composite: ${root.isComposite()}`); // true
console.log(`Leaf1 is composite: ${leaf1.isComposite()}`); // false
console.log(`Branch has ${branch.getChildren().length} children`); // Branch has 2 children
```

## Decorator Pattern

### Классический декоратор
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
    return "Coffee";
  }
}

// Абстрактный декоратор
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}
  
  abstract cost(): number;
  abstract description(): string;
}

// Конкретные декораторы
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 2;
  }
  
  description(): string {
    return `${this.coffee.description()}, milk`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.5;
  }
  
  description(): string {
    return `${this.coffee.description()}, sugar`;
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1;
  }
  
  description(): string {
    return `${this.coffee.description()}, whip`;
  }
}

// Использование
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} - $${coffee.cost()}`); // Coffee - $5

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`); // Coffee, milk - $7

coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`); // Coffee, milk, whip - $8

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`); // Coffee, milk, whip, sugar - $8.5
```

### Декораторы как утилиты
```typescript
// Утилиты декораторов
function timeDecorator<T extends (...args: any[]) => any>(fn: T): T {
  return function(...args: Parameters<T>): ReturnType<T> {
    const start = performance.now();
    const result = fn.apply(this, args);
    const end = performance.now();
    console.log(`${fn.name} took ${end - start}ms`);
    return result;
  } as T;
}

function memoizeDecorator<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, any>();
  
  return function(...args: Parameters<T>): ReturnType<T> {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  } as T;
}

function logDecorator<T extends (...args: any[]) => any>(fn: T): T {
  return function(...args: Parameters<T>): ReturnType<T> {
    console.log(`Calling ${fn.name} with arguments:`, args);
    const result = fn.apply(this, args);
    console.log(`Result of ${fn.name}:`, result);
    return result;
  } as T;
}

// Использование
const expensiveFunction = (n: number) => {
  // имитация дорогой операции
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
};

const timedFunction = timeDecorator(expensiveFunction);
const memoizedFunction = memoizeDecorator(expensiveFunction);

timedFunction(1000000); // выводит время выполнения
memoizedFunction(1000000); // первый вызов
memoizedFunction(1000000); // второй вызов из кэша
```

### TypeScript декораторы (experimental)
```typescript
// Property decorator
function readonly(target: any, propertyKey: string) {
  const descriptor = Object.getOwnPropertyDescriptor(target, propertyKey) || {};
  descriptor.writable = false;
  Object.defineProperty(target, propertyKey, descriptor);
}

// Method decorator
function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with arguments:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result of ${propertyKey}:`, result);
    return result;
  };
  
  return descriptor;
}

// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Calculator {
  @readonly
  name: string = "Calculator";
  
  @logMethod
  add(a: number, b: number): number {
    return a + b;
  }
  
  multiply(@readonly a: number, b: number): number {
    return a * b;
  }
}
```

## Facade Pattern

### Простая фасадная система
```typescript
// Подсистемы
class CPU {
  freeze(): void { console.log("CPU frozen"); }
  jump(position: number): void { console.log(`CPU jumped to ${position}`); }
  execute(): void { console.log("CPU executed instruction"); }
}

class Memory {
  load(position: number, data: string): void {
    console.log(`Memory loaded ${data} at ${position}`);
  }
}

class HardDrive {
  read(lba: number, size: number): string {
    console.log(`HardDrive reading ${size} bytes from ${lba}`);
    return "boot data";
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
  
  startComputer(): void {
    console.log("Starting computer...");
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    console.log("Computer started successfully");
  }
  
  shutdownComputer(): void {
    console.log("Shutting down computer...");
    // логика выключения
  }
  
  installOS(osName: string): void {
    console.log(`Installing ${osName}...`);
    // сложная логика установки
  }
}

// Использование
const computer = new ComputerFacade();
computer.startComputer();
// Вывод: "Starting computer...", "CPU frozen", "Memory loaded boot data at 0", "CPU jumped to 0", "CPU executed instruction", "Computer started successfully"
```

### API фасад
```typescript
// Фасад для работы с API
class APIClient {
  private baseURL: string;
  
  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }
  
  async get(path: string): Promise<any> {
    const response = await fetch(`${this.baseURL}${path}`);
    return response.json();
  }
  
  async post(path: string, data: any): Promise<any> {
    const response = await fetch(`${this.baseURL}${path}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

class CacheService {
  private cache = new Map<string, { data: any; timestamp: number }>();
  
  get(key: string): any | null {
    const cached = this.cache.get(key);
    if (cached && Date.now() - cached.timestamp < 60000) { // 1 минута
      return cached.data;
    }
    return null;
  }
  
  set(key: string, data: any): void {
    this.cache.set(key, { data, timestamp: Date.now() });
  }
}

// Фасад для работы с API
class APIFacade {
  private apiClient: APIClient;
  private cache: CacheService;
  
  constructor(baseURL: string) {
    this.apiClient = new APIClient(baseURL);
    this.cache = new CacheService();
  }
  
  async getUser(id: number): Promise<any> {
    const cacheKey = `user:${id}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached) {
      return cached;
    }
    
    const user = await this.apiClient.get(`/users/${id}`);
    this.cache.set(cacheKey, user);
    return user;
  }
  
  async createUser(userData: any): Promise<any> {
    const user = await this.apiClient.post('/users', userData);
    // очистить кэш пользователей
    return user;
  }
  
  async updateUser(id: number, userData: any): Promise<any> {
    const user = await this.apiClient.post(`/users/${id}`, userData);
    // обновить кэш
    const cacheKey = `user:${id}`;
    this.cache.set(cacheKey, user);
    return user;
  }
}
```

## Proxy Pattern

### Базовый прокси
```typescript
// Интерфейс для проксируемого объекта
interface Image {
  display(): void;
}

// Реальная реализация
class RealImage implements Image {
  private filename: string;
  
  constructor(filename: string) {
    this.filename = filename;
    this.loadFromDisk();
  }
  
  private loadFromDisk(): void {
    console.log(`Loading image from disk: ${this.filename}`);
  }
  
  display(): void {
    console.log(`Displaying image: ${this.filename}`);
  }
}

// Виртуальный прокси
class ProxyImage implements Image {
  private realImage: RealImage | null = null;
  private filename: string;
  
  constructor(filename: string) {
    this.filename = filename;
  }
  
  display(): void {
    if (this.realImage === null) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

// Использование
const image = new ProxyImage("photo.jpg");
// Загрузка изображения происходит только при вызове display()
image.display(); // "Loading image from disk: photo.jpg", "Displaying image: photo.jpg"
image.display(); // "Displaying image: photo.jpg" (загрузка второй раз не происходит)
```

### Защитный прокси
```typescript
interface Document {
  view(): void;
  edit(): void;
  delete(): void;
}

class RealDocument implements Document {
  private content: string;
  private owner: string;
  
  constructor(content: string, owner: string) {
    this.content = content;
    this.owner = owner;
  }
  
  view(): void {
    console.log(`Viewing document: ${this.content.substring(0, 50)}...`);
  }
  
  edit(): void {
    console.log("Editing document...");
    // логика редактирования
  }
  
  delete(): void {
    console.log("Deleting document...");
    // логика удаления
  }
}

class ProtectedDocument implements Document {
  private realDocument: RealDocument;
  private currentUser: string;
  
  constructor(content: string, owner: string, currentUser: string) {
    this.realDocument = new RealDocument(content, owner);
    this.currentUser = currentUser;
  }
  
  view(): void {
    this.realDocument.view();
  }
  
  edit(): void {
    if (this.canModify()) {
      this.realDocument.edit();
    } else {
      throw new Error("Access denied: insufficient permissions to edit");
    }
  }
  
  delete(): void {
    if (this.isOwner()) {
      this.realDocument.delete();
    } else {
      throw new Error("Access denied: only owner can delete");
    }
  }
  
  private canModify(): boolean {
    return this.currentUser === this.realDocument.owner || this.isAdmin();
  }
  
  private isOwner(): boolean {
    return this.currentUser === this.realDocument.owner;
  }
  
  private isAdmin(): boolean {
    // логика проверки администраторских прав
    return this.currentUser === "admin";
  }
}
```

### Прокси с обертыванием
```typescript
// Обобщенный прокси для логирования
class LoggingProxy<T extends object> {
  constructor(private target: T) {}
  
  create(): T {
    return new Proxy({}, {
      get: (target: any, prop: string) => {
        if (prop in this.target) {
          const value = this.target[prop as keyof T];
          
          if (typeof value === 'function') {
            // Обертываем методы для логирования
            return (...args: any[]) => {
              console.log(`Calling method: ${String(prop)} with args:`, args);
              const result = (value as Function).apply(this.target, args);
              console.log(`Method ${String(prop)} returned:`, result);
              return result;
            };
          }
          
          // Логируем доступ к свойствам
          console.log(`Accessing property: ${String(prop)}`);
          return value;
        }
        
        return undefined;
      },
      
      set: (target: any, prop: string, value: any) => {
        console.log(`Setting property ${String(prop)} to:`, value);
        this.target[prop as keyof T] = value;
        return true;
      }
    }) as T;
  }
}

// Использование
interface UserService {
  getUser(id: number): any;
  updateUser(id: number, data: any): void;
  deleteUser(id: number): void;
}

class ConcreteUserService implements UserService {
  getUser(id: number) {
    return { id, name: "User" + id };
  }
  
  updateUser(id: number, data: any) {
    console.log(`Updated user ${id} with`, data);
  }
  
  deleteUser(id: number) {
    console.log(`Deleted user ${id}`);
  }
}

const userService = new ConcreteUserService();
const loggingProxy = new LoggingProxy(userService);
const loggingUserService = loggingProxy.create();

loggingUserService.getUser(1);
// Вывод: "Calling method: getUser with args: [1]"
//        "Method getUser returned: { id: 1, name: 'User1' }"
```

## Специфические TypeScript паттерны

### Conditional Types + Structural Patterns
```typescript
// Утилита для создания прокси-типов
type ProxyType<T> = {
  [K in keyof T]: T[K] extends (...args: infer A) => infer R
    ? (...args: A) => R extends Promise<any> ? R : Promise<R>
    : T[K]
};

// Пример: прокси, делающий все методы асинхронными
class AsyncProxy<T extends object> {
  constructor(private target: T) {}
  
  create(): ProxyType<T> {
    return new Proxy(this.target, {
      get: (target, prop) => {
        const value = target[prop as keyof T];
        
        if (typeof value === 'function') {
          return async (...args: any[]) => {
            return Promise.resolve((value as Function).apply(target, args));
          };
        }
        
        return value;
      }
    }) as ProxyType<T>;
  }
}
```

### Модульный подход к структурным паттернам
```typescript
// Модуль для адаптеров
namespace Adapters {
  export interface IAdapter<T, U> {
    adapt(from: T): U;
  }
  
  export class StringNumberAdapter implements IAdapter<string, number> {
    adapt(from: string): number {
      return Number(from);
    }
  }
  
  export class NumberStringAdapter implements IAdapter<number, string> {
    adapt(from: number): string {
      return String(from);
    }
  }
}

// Модуль для композитов
namespace Composites {
  export interface IComponent<T> {
    getChild(index: number): IComponent<T> | null;
    add(component: IComponent<T>): void;
    remove(component: IComponent<T>): void;
    operation(): T;
  }
}

// Использование
const numberAdapter = new Adapters.StringNumberAdapter();
const result = numberAdapter.adapt("123"); // 123
```

## Практические примеры

### State Management с Composite
```typescript
// Структура для управления состоянием
interface StateNode<T> {
  id: string;
  getState(): T;
  setState(newState: T): void;
  addChild(node: StateNode<T>): void;
  removeChild(node: StateNode<T>): void;
  propagateState(state: T): void;
}

class StateLeaf<T> implements StateNode<T> {
  constructor(
    public id: string,
    private state: T
  ) {}
  
  getState(): T {
    return this.state;
  }
  
  setState(newState: T): void {
    this.state = newState;
  }
  
  addChild(node: StateNode<T>): void {
    throw new Error("Cannot add child to leaf node");
  }
  
  removeChild(node: StateNode<T>): void {
    throw new Error("Cannot remove child from leaf node");
  }
  
  propagateState(state: T): void {
    this.setState(state);
  }
}

class StateComposite<T> implements StateNode<T> {
  private children: StateNode<T>[] = [];
  
  constructor(public id: string) {}
  
  getState(): T {
    // Возвращаем состояние первого ребенка или пустое состояние
    return this.children[0]?.getState() || {} as T;
  }
  
  setState(newState: T): void {
    this.children.forEach(child => child.setState(newState));
  }
  
  addChild(node: StateNode<T>): void {
    this.children.push(node);
  }
  
  removeChild(node: StateNode<T>): void {
    const index = this.children.indexOf(node);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  propagateState(state: T): void {
    this.setState(state);
    this.children.forEach(child => child.propagateState(state));
  }
}
```

### Middleware с Decorator
```typescript
// Паттерн декоратора для middleware (упрощенно)
interface Handler {
  handle(request: any): any;
}

class RequestHandler implements Handler {
  handle(request: any): any {
    console.log("Handling request:", request);
    return { status: 200, data: "OK" };
  }
}

abstract class MiddlewareDecorator implements Handler {
  constructor(protected handler: Handler) {}
  
  abstract handle(request: any): any;
}

class LoggingMiddleware extends MiddlewareDecorator {
  handle(request: any): any {
    console.log("Before processing:", request);
    const result = this.handler.handle(request);
    console.log("After processing:", result);
    return result;
  }
}

class ValidationMiddleware extends MiddlewareDecorator {
  handle(request: any): any {
    if (!request.valid) {
      throw new Error("Invalid request");
    }
    return this.handler.handle(request);
  }
}

class AuthMiddleware extends MiddlewareDecorator {
  handle(request: any): any {
    if (!request.authenticated) {
      throw new Error("Unauthorized");
    }
    return this.handler.handle(request);
  }
}

// Использование
let handler: Handler = new RequestHandler();
handler = new LoggingMiddleware(new ValidationMiddleware(new AuthMiddleware(handler)));

try {
  const response = handler.handle({ authenticated: true, valid: true, data: "test" });
  console.log(response);
} catch (error) {
  console.error("Error:", error.message);
}
```

## Лучшие практики

### 1. Используйте интерфейсы для четкого определения контрактов
```typescript
// Хорошо: явно определенные интерфейсы
interface Component {
  operation(): void;
}

interface ComponentDecorator extends Component {
  component: Component;
}
```

### 2. Избегайте чрезмерной вложенности декораторов
```typescript
// Плохо: слишком много вложений
const decorated = new Decorator5(
  new Decorator4(
    new Decorator3(
      new Decorator2(
        new Decorator1(baseObject)
      )
    )
  )
);

// Лучше: использовать фабрику или цепочку
const decorated = createDecoratorChain([
  Decorator1,
  Decorator2,
  Decorator3,
  Decorator4,
  Decorator5
])(baseObject);
```

### 3. Используйте типы для безопасности при использовании структурных паттернов
```typescript
// Использование обобщений для безопасности типов
interface IComposite<T> {
  add(item: IComponent<T>): void;
  remove(item: IComponent<T>): void;
  getChildren(): IComponent<T>[];
  operate(): T;
}

interface IComponent<T> {
  operate(): T;
  isComposite(): this is IComposite<T>;
}
```

## Совместимость и миграция

### Совместимость с существующим кодом
```typescript
// Адаптер для совместимости с устаревшим API
interface LegacyAPI {
  processOld(item: any): any;
}

interface ModernAPI {
  process(item: unknown): Promise<unknown>;
}

class LegacyModernAdapter implements ModernAPI {
  constructor(private legacyAPI: LegacyAPI) {}
  
  async process(item: unknown): Promise<unknown> {
    return this.legacyAPI.processOld(item);
  }
}
```

## Ограничения и подводные камни

### 1. Производительность прокси
```typescript
// Прокси добавляют оверхед
const proxy = new Proxy(target, {
  get(target, prop) {
    // Эта функция вызывается при каждом доступе к свойству
    return target[prop];
  }
});
```

### 2. Сложность отладки композитных структур
```typescript
// Сложные композитные структуры могут быть трудны для отладки
// Используйте toString или специальные методы отладки
```

## Современные тенденции

### Использование Proxy API для реактивности
```typescript
// Пример реактивного прокси (упрощенный)
function createReactive<T extends object>(obj: T, onChange?: (prop: keyof T, value: any) => void): T {
  return new Proxy(obj, {
    get(target, prop: keyof T) {
      return target[prop];
    },
    set(target, prop: keyof T, value) {
      const oldValue = target[prop];
      target[prop] = value;
      if (onChange && oldValue !== value) {
        onChange(prop, value);
      }
      return true;
    }
  });
}

const reactiveObj = createReactive({ count: 0 }, (prop, value) => {
  console.log(`Property ${String(prop)} changed to ${value}`);
});
```

## Заключение

Структурные паттерны в TypeScript:

1. **Обеспечивают гибкость** - позволяют изменять структуру без изменения клиентского кода
2. **Повышают переиспользуемость** - одни и те же компоненты могут использоваться в разных структурах
3. **Упрощают сложность** - скрывают сложные структуры за простыми интерфейсами
4. **Улучшают сопровождаемость** - структура кода становится более понятной

Каждый шаблон решает конкретные архитектурные задачи и должен использоваться осознанно в зависимости от требований системы.

## Связь с другими концепциями
- [[../design-patterns/creational-patterns]] - порождающие паттерны для создания структур
- [[../design-patterns/behavioral-patterns]] - поведенческие паттерны в структурных контекстах
- [[TypeScript Modules]] - организация структурных паттернов в модули
- [[../generics/advanced]] - обобщенные структурные паттерны
- [[../advanced-types/mapped-types]] - сопоставляющие типы для структурных паттернов