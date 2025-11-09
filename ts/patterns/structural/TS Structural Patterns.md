# Structural Patterns

Структурные паттерны описывают способы компоновки объектов и классов для формирования крупных структур. Они помогают обеспечить гибкость и эффективность архитектуры программного обеспечения.

## Основные структурные паттерны

### [Adapter Pattern](adapter.md)
Преобразует интерфейс одного класса в интерфейс, который ожидают клиенты.

### [Decorator Pattern](decorator.md)
Добавляет обязанности объекту динамически, без изменения кода других объектов того же класса.

### [Composite Pattern](composite.md)
Компонует объекты в древовидные структуры для представления иерархии "часть-целое".

### [Facade Pattern](facade.md)
Предоставляет унифицированный интерфейс к подсистеме с множеством интерфейсов.

### [Proxy Pattern](proxy.md)
Предоставляет заменитель или заполнитель для другого объекта для контроля доступа к нему.

### [Bridge Pattern](bridge.md)
Разделяет абстракцию и реализацию, позволяя изменять их независимо.

### [Flyweight Pattern](flyweight.md)
Использует разделение для эффективной поддержки большого количества мелких объектов.

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
  │ - Interface     │◄──────────────┤ - Abstraction &          │─────────────────────────────────────►│ - Recursive     │
  │   Translation   │               │   Implementation         │                                    │   Composition   │
  │ - Legacy        │               │ - Decoupling             │                                    │ - Hierarchical  │
  │   Integration   │               │ - Runtime Selection      │                                    │   Structure     │
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
        │                           │ - Layered Enhancement           │
        │                           │ - Multiple Decorators           │
        │                           │ - Compositional Chain          │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │          Proxy                  │
        │                           │                               │
        │                           │ - Access Control                │
        │                           │ - Lazy Initialization           │
        │                           │ - Virtual Proxy                 │
        │                           │ - Protection Proxy              │
        │                           │ - Remote Proxy                  │
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

### Основной пример адаптера
```typescript
// Целевой интерфейс
interface MediaPlayer {
  play(audioType: string, fileName: string): void;
}

// Адаптируемый класс
class LegacyMediaPlayer {
  playAudio(audioFile: string, format: string): void {
    console.log(`Playing ${format} file: ${audioFile}`);
  }
}

// Адаптер для интеграции
class MediaAdapter implements MediaPlayer {
  private advancedMusicPlayer: LegacyMediaPlayer;
  
  constructor(audioType: string) {
    if (audioType.toLowerCase() === "vlc") {
      this.advancedMusicPlayer = new VlcPlayer();
    } else if (audioType.toLowerCase() === "mp4") {
      this.advancedMusicPlayer = new Mp4Player();
    }
  }
  
  play(audioType: string, fileName: string): void {
    if (audioType.toLowerCase() === "vlc") {
      (this.advancedMusicPlayer as VlcPlayer).playVlc(fileName);
    } else if (audioType.toLowerCase() === "mp4") {
      (this.advancedMusicPlayer as Mp4Player).playMp4(fileName);
    }
  }
}

class VlcPlayer extends AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    console.log(`Playing VLC file: ${fileName}`);
  }
  
  playMp4(fileName: string): void {
    // Не реализовано для VLC
  }
}

class Mp4Player extends AdvancedMediaPlayer {
  playVlc(fileName: string): void {
    // Не реализовано для MP4
  }
  
  playMp4(fileName: string): void {
    console.log(`Playing MP4 file: ${fileName}`);
  }
}

// Основной медиаплеер
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
      console.log(`Invalid media type: ${audioType} format not supported`);
    }
  }
}
```

### Адаптер для сторонних библиотек
```typescript
// Сторонняя библиотека с нежелательным интерфейсом
interface ExternalLogger {
  log(level: number, message: string, context: any): void;
}

// Наш желательный интерфейс
interface Logger {
  info(message: string, context?: any): void;
  warn(message: string, context?: any): void;
  error(message: string, context?: any): void;
}

// Адаптер
class ExternalLoggerAdapter implements Logger {
  constructor(private externalLogger: ExternalLogger) {}
  
  info(message: string, context?: any): void {
    this.externalLogger.log(1, message, context);
  }
  
  warn(message: string, context?: any): void {
    this.externalLogger.log(2, message, context);
  }
  
  error(message: string, context?: any): void {
    this.externalLogger.log(3, message, context);
  }
}

// Использование
const externalLogger = new ExternalLoggerImplementation();
const adaptedLogger = new ExternalLoggerAdapter(externalLogger);

adaptedLogger.info("Application started"); // преобразуется в вызов externalLogger.log(1, ...)
```

## Bridge Pattern

### Разделение абстракции и реализации
```typescript
// Абстракция
abstract class Device {
  protected volume: number = 50;
  
  constructor(protected remote: RemoteControl) {}
  
  setVolume(level: number): void {
    if (level < 0 || level > 100) return;
    this.volume = level;
    this.remote.setVolume(level);
  }
  
  abstract turnOn(): void;
  abstract turnOff(): void;
}

// Реализация
interface RemoteControl {
  setVolume(level: number): void;
  powerOn(device: Device): void;
  powerOff(device: Device): void;
}

class BasicRemote implements RemoteControl {
  setVolume(level: number): void {
    console.log(`Setting volume to ${level}`);
  }
  
  powerOn(device: Device): void {
    console.log("Power button pressed - ON");
    device.turnOn();
  }
  
  powerOff(device: Device): void {
    console.log("Power button pressed - OFF");
    device.turnOff();
  }
}

class AdvancedRemote implements RemoteControl {
  private muteState = false;
  
  setVolume(level: number): void {
    console.log(`Advanced remote: setting volume to ${level}`);
  }
  
  powerOn(device: Device): void {
    console.log("Advanced remote - Power ON with smart features");
    device.turnOn();
  }
  
  powerOff(device: Device): void {
    console.log("Advanced remote - Power OFF with sleep timer");
    device.turnOff();
  }
  
  mute(): void {
    this.muteState = !this.muteState;
    console.log(`Mute ${this.muteState ? "ON" : "OFF"}`);
  }
}

// Конкретные устройства
class TV extends Device {
  turnOn(): void {
    console.log("TV is ON");
  }
  
  turnOff(): void {
    console.log("TV is OFF");
  }
}

class Radio extends Device {
  turnOn(): void {
    console.log("Radio is ON");
  }
  
  turnOff(): void {
    console.log("Radio is OFF");
  }
}

// Использование
const tv = new TV(new BasicRemote());
tv.setVolume(75);
tv.turnOn();

const radio = new Radio(new AdvancedRemote());
radio.turnOn();
(radio.remote as AdvancedRemote).mute(); // Уровень абстракции позволяет получить доступ к конкретной реализации
```

## Composite Pattern

### Иерархическая структура
```typescript
// Компонент
interface FileSystemItem {
  name: string;
  getSize(): number;
  render(): string;
  isFolder(): boolean;
}

// Лист
class File implements FileSystemItem {
  constructor(public name: string, private size: number) {}
  
  getSize(): number {
    return this.size;
  }
  
  render(): string {
    return `📄 ${this.name} (${this.size} bytes)`;
  }
  
  isFolder(): boolean {
    return false;
  }
}

// Компонент (контейнер)
class Folder implements FileSystemItem {
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
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }
  
  render(): string {
    const childOutput = this.children.map(child => `  ${child.render()}`).join('\n');
    return `📁 ${this.name} (${this.getSize()} bytes)\n${childOutput}`;
  }
  
  isFolder(): boolean {
    return true;
  }
}

// Использование
const root = new Folder("Root");
const documents = new Folder("Documents");
const photos = new Folder("Photos");

documents.add(new File("report.docx", 102400));
documents.add(new File("presentation.pptx", 204800));

photos.add(new File("vacation.jpg", 2048000));
photos.add(new File("family.png", 1024000));

root.add(documents);
root.add(photos);

console.log(root.render());
// Выводит древовидную структуру с размерами
```

## Decorator Pattern

### Динамическое добавление функциональности
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
    return "Simple coffee";
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
    return this.coffee.cost() + 1;
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
    return this.coffee.cost() + 1.5;
  }
  
  description(): string {
    return `${this.coffee.description()}, whip`;
  }
}

// Использование
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// Simple coffee, milk, whip, sugar - $8
```

### Декораторы TypeScript (экспериментальные)
```typescript
// TypeScript экспериментальные декораторы
function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with arguments:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result of ${propertyKey}:`, result);
    return result;
  };
  
  return descriptor;
}

function logClass(constructor: Function) {
  console.log(`Creating instance of ${constructor.name}`);
}

@logClass
class Calculator {
  @logMethod
  add(a: number, b: number): number {
    return a + b;
  }
  
  @logMethod
  multiply(a: number, b: number): number {
    return a * b;
  }
}

const calc = new Calculator();
calc.add(2, 3); // логируется вызов метода
```

## Facade Pattern

### Упрощение сложной подсистемы
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
    // сложная логика выключения
  }
}

// Использование
const computer = new ComputerFacade();
computer.startComputer();
```

### Фасад для API
```typescript
// Фасад для сложного API
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

// Фасад для упрощения работы с API
class APIService {
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
    // очистить связанные кэши
    return user;
  }
}
```

## Proxy Pattern

### Виртуальный прокси
```typescript
// Реальный субъект
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
    console.log(`Loading image ${this.fileName} from disk`);
  }
  
  display(): void {
    console.log(`Displaying image ${this.fileName}`);
  }
}

// Прокси
class ProxyImage implements Image {
  private realImage: RealImage | null = null;
  private fileName: string;
  
  constructor(fileName: string) {
    this.fileName = fileName;
  }
  
  display(): void {
    if (!this.realImage) {
      this.realImage = new RealImage(this.fileName);
    }
    this.realImage.display();
  }
}

// Использование
const image = new ProxyImage("photo.jpg");
// Изображение не загружено до вызова display()
image.display(); // "Loading image photo.jpg from disk", "Displaying image photo.jpg"
image.display(); // "Displaying image photo.jpg" (без повторной загрузки)
```

### Защитный прокси
```typescript
// Защитный прокси для ограничения доступа
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
  }
  
  delete(): void {
    console.log("Deleting document...");
  }
}

class ProtectedDocument implements Document {
  private realDocument: RealDocument;
  private currentUser: string;
  private userRole: string;
  
  constructor(
    content: string, 
    owner: string, 
    currentUser: string, 
    userRole: string
  ) {
    this.realDocument = new RealDocument(content, owner);
    this.currentUser = currentUser;
    this.userRole = userRole;
  }
  
  view(): void {
    this.realDocument.view();
  }
  
  edit(): void {
    if (this.hasEditPermission()) {
      this.realDocument.edit();
    } else {
      throw new Error("Access denied: no edit permission");
    }
  }
  
  delete(): void {
    if (this.hasDeletePermission()) {
      this.realDocument.delete();
    } else {
      throw new Error("Access denied: no delete permission");
    }
  }
  
  private hasEditPermission(): boolean {
    return this.currentUser === this.realDocument.owner || 
           this.userRole === "admin" || 
           this.userRole === "editor";
  }
  
  private hasDeletePermission(): boolean {
    return this.currentUser === this.realDocument.owner || 
           this.userRole === "admin";
  }
}
```

## Функциональные структурные паттерны

### Прокси через Proxy API
```typescript
// Прокси для логирования доступа
function createLoggingProxy<T extends object>(target: T): T {
  return new Proxy(target, {
    get(obj: T, prop: string) {
      console.log(`Accessing property: ${prop}`);
      const value = obj[prop as keyof T];
      
      if (typeof value === 'function') {
        // Логирование вызовов методов
        return function(...args: any[]) {
          console.log(`Calling method: ${prop} with args:`, args);
          const result = value.apply(obj, args);
          console.log(`Method ${prop} returned:`, result);
          return result;
        };
      }
      
      console.log(`Property ${prop} =`, value);
      return value;
    },
    
    set(obj: T, prop: string, value: any) {
      console.log(`Setting ${prop} = ${value}`);
      obj[prop as keyof T] = value;
      return true;
    }
  });
}

// Использование
interface UserService {
  getUser(id: number): any;
  updateUser(id: number, data: any): void;
}

class ConcreteUserService implements UserService {
  getUser(id: number) {
    return { id, name: `User ${id}` };
  }
  
  updateUser(id: number, data: any) {
    console.log(`Updated user ${id} with:`, data);
  }
}

const userService = new ConcreteUserService();
const loggingUserService = createLoggingProxy(userService);

loggingUserService.getUser(1);
// Logs:
// "Accessing property: getUser"
// "Calling method: getUser with args: [1]"
// "Method getUser returned: { id: 1, name: 'User 1' }"
```

### Функциональный декоратор
```typescript
// Функциональный подход к декорированию
type Middleware<T> = (target: T) => T;

function withLogging<T extends Record<string, any>>(target: T): T {
  const proxy = {} as T;
  
  for (const key in target) {
    if (typeof target[key] === 'function') {
      proxy[key] = function(...args: any[]) {
        console.log(`Calling ${key} with:`, args);
        const result = target[key].apply(target, args);
        console.log(`Result of ${key}:`, result);
        return result;
      };
    } else {
      proxy[key] = target[key];
    }
  }
  
  return proxy;
}

function withTiming<T extends Record<string, any>>(target: T): T {
  const proxy = {} as T;
  
  for (const key in target) {
    if (typeof target[key] === 'function') {
      proxy[key] = function(...args: any[]) {
        const start = performance.now();
        const result = target[key].apply(target, args);
        const end = performance.now();
        console.log(`${key} took ${end - start} milliseconds`);
        return result;
      };
    } else {
      proxy[key] = target[key];
    }
  }
  
  return proxy;
}

// Композиция декораторов
const enhancedUserService = withLogging(withTiming(userService));
```

## Продвинутые примеры

### Сложный адаптер для API
```typescript
// Адаптер для работы с разными API
interface UserAPI {
  getUser(id: string): Promise<any>;
  createUser(userData: any): Promise<any>;
}

class LegacyAPI {
  getUserData(userId: number): Promise<any> {
    // Старый формат API
    return fetch(`/api/user/${userId}`).then(res => res.json());
  }
  
  addNewUser(userInfo: any): Promise<any> {
    // Старый формат API
    return fetch('/api/add-user', {
      method: 'POST',
      body: JSON.stringify(userInfo)
    }).then(res => res.json());
  }
}

class ModernAPI {
  async getUser(userId: string): Promise<any> {
    // Новый формат API
    const response = await fetch(`/api/v2/users/${userId}`);
    return response.json();
  }
  
  async createUser(userData: { email: string; name: string }): Promise<any> {
    // Новый формат API
    const response = await fetch('/api/v2/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    return response.json();
  }
}

// Адаптер для устаревшего API
class LegacyAPIAdapter implements UserAPI {
  constructor(private legacyAPI: LegacyAPI) {}
  
  async getUser(id: string): Promise<any> {
    // Преобразование нового формата к старому
    const numericId = parseInt(id, 10);
    const userData = await this.legacyAPI.getUserData(numericId);
    
    // Преобразование формата данных к новому формату
    return {
      id: userData.ID?.toString() || id,
      email: userData.Email || userData.email,
      name: userData.Name || userData.name
    };
  }
  
  async createUser(userData: any): Promise<any> {
    // Преобразование нового формата к старому
    const legacyFormat = {
      email: userData.email,
      name: userData.name,
      // преобразование других полей
    };
    
    const result = await this.legacyAPI.addNewUser(legacyFormat);
    
    // Преобразование результата к новому формату
    return {
      id: result.userId?.toString(),
      email: userData.email,
      name: userData.name
    };
  }
}

// Использование
const legacyAPI = new LegacyAPI();
const adapter = new LegacyAPIAdapter(legacyAPI);

// Код может использовать новый интерфейс, но работать со старым API
async function useAPI(api: UserAPI) {
  const user = await api.getUser("123");
  console.log(user);
}
```

### Композитный компонент с типами
```typescript
// Сложный композитный компонент с типизацией
interface UIComponent {
  render(): string;
  add(component: UIComponent): void;
  remove(component: UIComponent): void;
  getChildren(): UIComponent[];
}

class TextElement implements UIComponent {
  constructor(private content: string) {}
  
  render(): string {
    return this.content;
  }
  
  add(_component: UIComponent): void {
    throw new Error("Text elements cannot have children");
  }
  
  remove(_component: UIComponent): void {
    throw new Error("Text elements cannot have children");
  }
  
  getChildren(): UIComponent[] {
    return [];
  }
}

class Container implements UIComponent {
  private children: UIComponent[] = [];
  
  constructor(private tagName: string) {}
  
  render(): string {
    const childrenHTML = this.children.map(child => child.render()).join('');
    return `<${this.tagName}>${childrenHTML}</${this.tagName}>`;
  }
  
  add(component: UIComponent): void {
    this.children.push(component);
  }
  
  remove(component: UIComponent): void {
    const index = this.children.indexOf(component);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  getChildren(): UIComponent[] {
    return [...this.children]; // возвращаем копию
  }
}

// Использование
const body = new Container('body');
const header = new Container('header');
const title = new TextElement('My Website');
const nav = new Container('nav');

header.add(title);
header.add(nav);

const main = new Container('main');
main.add(new TextElement('Welcome to my site'));

body.add(header);
body.add(main);

console.log(body.render());
// <body><header><h1>My Website</h1><nav></nav></header><main><p>Welcome to my site</p></main></body>
```

## Современные применения

### Proxy для реактивности
```typescript
// Прокси для создания реактивных объектов
interface ReactiveEvent {
  target: object;
  property: string | symbol;
  value: any;
  oldValue: any;
}

interface ReactiveHandler<T> {
  (event: ReactiveEvent): void;
}

class ReactiveObject<T extends object> {
  private listeners: ReactiveHandler<T>[] = [];
  
  constructor(private obj: T) {
    return this.createProxy();
  }
  
  private createProxy(): T {
    return new Proxy(this.obj, {
      set: (target: T, property: string | symbol, value: any) => {
        const oldValue = target[property as keyof T];
        target[property as keyof T] = value;
        
        this.listeners.forEach(listener => 
          listener({
            target,
            property: property.toString(),
            value,
            oldValue
          })
        );
        
        return true;
      }
    }) as T;
  }
  
  observe(handler: ReactiveHandler<T>): () => void {
    this.listeners.push(handler);
    
    return () => {
      const index = this.listeners.indexOf(handler);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
}

// Использование
const person = new ReactiveObject({
  name: "Alice",
  age: 30
});

const unsubscribe = person.observe(event => {
  console.log(`Property ${event.property} changed from ${event.oldValue} to ${event.value}`);
});

person.name = "Bob"; // "Property name changed from Alice to Bob"
person.age = 31;     // "Property age changed from 30 to 31"

unsubscribe(); // отписка от наблюдения
```

## Лучшие практики

### 1. Используйте адаптеры для интеграции
```typescript
// Адаптеры помогают интегрировать разные интерфейсы
interface PaymentProcessor {
  process(amount: number): boolean;
}

interface ExternalPayProcessor {
  pay(amountInCents: number, currency: string): { success: boolean; transactionId: string };
}

class ExternalPayAdapter implements PaymentProcessor {
  constructor(private external: ExternalPayProcessor, private currency: string = "USD") {}
  
  process(amount: number): boolean {
    const result = this.external.pay(amount * 100, this.currency); // конвертация в центы
    return result.success;
  }
}
```

### 2. Используйте фасады для упрощения сложных API
```typescript
// Фасад упрощает работу со сложными подсистемами
class ComplexSystemFacade {
  private subsystem1: Subsystem1;
  private subsystem2: Subsystem2;
  private subsystem3: Subsystem3;
  
  constructor() {
    this.subsystem1 = new Subsystem1();
    this.subsystem2 = new Subsystem2();
    this.subsystem3 = new Subsystem3();
  }
  
  performComplexOperation(): void {
    // Сложная операция требует взаимодействия с несколькими подсистемами
    this.subsystem1.initialize();
    this.subsystem2.configure();
    this.subsystem3.process();
    this.subsystem1.finalize();
  }
}

// Клиентский код работает только с фасадом
const facade = new ComplexSystemFacade();
facade.performComplexOperation();
```

### 3. Используйте композит для древовидных структур
```typescript
// Композит идеален для древовидных структур
interface TreeNode {
  getName(): string;
  getSize(): number;
  isFolder(): boolean;
}

// Легко расширяем: добавляем новые типы узлов без изменений в клиентском коде
class SpecialFolder implements TreeNode {
  // Реализация специфичной логики
}
```

## Ограничения и подводные камни

### 1. Сложность прокси
```typescript
// Прокси добавляют оверхед
const expensiveOperation = () => {
  // тяжелая операция
};

// Без прокси: прямой вызов
expensiveOperation();

// С прокси: дополнительная обертка
const proxiedOperation = createLoggingProxy({ operation: expensiveOperation });
proxiedOperation.operation(); // немного медленнее из-за прокси
```

### 2. Изменение семантики с прокси
```typescript
// Прокси могут изменить ожидаемое поведение
class MyClass {
  method() { return "original"; }
}

const instance = new MyClass();
const proxyInstance = new Proxy(instance, { /* handlers */ });

// instanceof может не работать как ожидается
console.log(instance instanceof MyClass); // true
console.log(proxyInstance instanceof MyClass); // может быть false
```

### 3. Декораторы и производительность
```typescript
// Много слоев декорирования могут увеличить оверхед
let decorated = new Component();
decorated = new LoggingDecorator(decorated);
decorated = new CachingDecorator(decorated);
decorated = new ValidationDecorator(decorated);
// 3 уровня оберток могут замедлить выполнение
```

## Когда использовать каждый паттерн

| Паттерн | Когда использовать | Примеры использования |
|---------|-------------------|----------------------|
| Adapter | Интеграция с существующим кодом/библиотеками | Интеграция с внешними API, работа с legacy кодом |
| Bridge | Когда нужно раздельно менять абстракцию и реализацию | ОС-специфичные реализации, UI для разных платформ |
| Composite | Для древовидных структур | Файловые системы, UI компоненты, меню |
| Decorator | Динамическое добавление функциональности | Логирование, кэширование, валидация |
| Facade | Упрощение сложных подсистем | API библиотек, сложные фреймворки |
| Proxy | Контроль доступа к объекту | Отложенная загрузка, безопасность, кэширование |
| Flyweight | Для экономии памяти при большом количестве объектов | Графика, текстовые редакторы, игры |

## Связь с другими концепциями
- [[../behavioral-patterns/strategy-pattern]] - декораторы как альтернатива стратегиям
- [[../creational-patterns/factory-pattern]] - фабрики для создания компонентов
- [[../functional-programming/decorators]] - функциональные подходы к декорированию
- [[../type-system/structural-typing]] - структурная типизация и совместимость
- [[../modules/system-design]] - модульные паттерны и структурные шаблоны