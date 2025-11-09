# Утиная типизация в TypeScript

Утиная типизация (duck typing) - это концепция, основанная на поговорке: "Если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка". В контексте программирования это означает, что объект определяется по своим методам и свойствам, а не по его классу или интерфейсу. Утиная типизация является частным случаем структурной типизации, реализованной в TypeScript.

## Основная концепция

В языке с утиной типизацией объекты считаются совместимыми, если они имеют одинаковую структуру, независимо от их наследования или явных объявлений типов.

```typescript
interface Duck {
  quack(): void;
  swim(): void;
}

interface Dog {
  bark(): void;
  swim(): void;
}

// Утиная типизация позволяет использовать объекты с общими методами
function makeItSwim(creature: { swim(): void }) {
  creature.swim();
}

// Оба объекта могут быть использованы, так как имеют метод swim()
const duck: Duck = {
  quack: () => console.log("Quack!"),
  swim: () => console.log("Duck swimming")
};

const dog: Dog = {
  bark: () => console.log("Woof!"),
  swim: () => console.log("Dog swimming")
};

makeItSwim(duck);  // OK: Duck имеет метод swim()
makeItSwim(dog);   // OK: Dog также имеет метод swim()
```

## Примеры утиной типизации

### 1. Простые объекты

```typescript
// Определение интерфейса для объекта с координатами
interface Point {
  x: number;
  y: number;
}

// Объект, который не объявлен как Point, но имеет нужную структуру
const myPoint = {
  x: 10,
  y: 20,
  z: 30  // дополнительное свойство
};

// Это работает благодаря утиной типизации
function calculateDistance(point: Point): number {
  return Math.sqrt(point.x ** 2 + point.y ** 2);
}

// myPoint может быть использован как Point, так как имеет x и y
const distance = calculateDistance(myPoint);  // OK
```

### 2. Функции и методы

```typescript
interface Drawable {
  draw(): void;
}

class Circle {
  radius: number;
  constructor(radius: number) {
    this.radius = radius;
  }
  
  draw(): void {
    console.log(`Drawing circle with radius ${this.radius}`);
  }
}

class Square {
  side: number;
  constructor(side: number) {
    this.side = side;
  }
  
  draw(): void {
    console.log(`Drawing square with side ${this.side}`);
  }
  
  // Дополнительный метод
  calculateArea(): number {
    return this.side * this.side;
  }
}

// Функция принимает любой объект с методом draw()
function render(shape: Drawable) {
  shape.draw();
}

// Оба класса могут быть использованы, несмотря на разное наследование
render(new Circle(5));  // OK
render(new Square(10)); // OK
```

## Утиная типизация в функциях

### 1. Совместимость функций

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// Функция, которая соответствует интерфейсу по структуре
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  return src.search(sub) !== -1;
};

// Даже если параметры имеют другие имена, структура совпадает
let mySearch2: SearchFunc;
mySearch2 = function(source: string, subString: string): boolean {
  return source.includes(subString);
};
```

### 2. Контравариантность параметров

```typescript
interface Input {
  data: string;
}

interface ExtendedInput extends Input {
  extra: number;
}

// Функция, принимающая более общий тип
let generalProcessor: (input: Input) => void;
// Функция, принимающая более конкретный тип
let specificProcessor: (input: ExtendedInput) => void;

// Благодаря утиной типизации и контравариантности параметров:
generalProcessor = specificProcessor;  // OK: функция, принимающая конкретный тип, может обработать общий
// specificProcessor = generalProcessor;  // Ошибка: функция, ожидающая конкретный тип, не может принять общий
```

## Практические применения

### 1. Работа с API и внешними библиотеками

```typescript
// Определение интерфейса для API клиента
interface ApiClient {
  get(url: string): Promise<any>;
  post(url: string, data: any): Promise<any>;
}

// Наша реализация, которая не наследуется от интерфейса явно
const fetchClient = {
  get: async (url: string) => {
    const response = await fetch(url);
    return response.json();
  },
  post: async (url: string, data: any) => {
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' }
    });
    return response.json();
  },
  // Дополнительный метод
  put: async (url: string, data: any) => {
    const response = await fetch(url, {
      method: 'PUT',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' }
    });
    return response.json();
  }
};

// Благодаря утиной типизации, fetchClient может быть использован как ApiClient
function useApiClient(client: ApiClient) {
  client.get('/api/users');
  client.post('/api/users', { name: 'John' });
}

useApiClient(fetchClient);  // OK: структура совпадает
```

### 2. Создание плагинов и модулей

```typescript
// Интерфейс для плагина
interface Plugin {
  name: string;
  activate(): void;
  deactivate(): void;
}

// Разные реализации плагинов
const authPlugin = {
  name: "Authentication",
  activate: () => console.log("Auth plugin activated"),
  deactivate: () => console.log("Auth plugin deactivated"),
  // Дополнительные методы
  login: (username: string, password: string) => console.log("Logging in...")
};

const loggerPlugin = {
  name: "Logger",
  activate: () => console.log("Logger plugin activated"),
  deactivate: () => console.log("Logger plugin deactivated"),
  // Дополнительные методы
  logLevel: "info"
};

// Менеджер плагинов
class PluginManager {
  private plugins: Plugin[] = [];
  
  addPlugin(plugin: Plugin) {
    this.plugins.push(plugin);
    plugin.activate();
  }
  
  removePlugin(plugin: Plugin) {
    plugin.deactivate();
    this.plugins = this.plugins.filter(p => p !== plugin);
  }
}

const manager = new PluginManager();
manager.addPlugin(authPlugin);   // OK
manager.addPlugin(loggerPlugin); // OK
```

### 3. Работа с DOM элементами

```typescript
interface Clickable {
  click(): void;
}

// Разные DOM элементы могут быть Clickable
const button = document.getElementById('myButton');
const link = document.getElementById('myLink');

// Функция, работающая с любыми кликабельными элементами
function handleMultipleClicks(elements: Clickable[]) {
  elements.forEach(element => element.click());
}

// Эти элементы могут быть использованы, так как у них есть метод click()
if (button && link) {
  handleMultipleClicks([button, link]);  // OK: HTMLElement имеет метод click()
}
```

## Продвинутые паттерны

### 1. Совместимость с частичными объектами

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

// Функция для обновления пользователя
function updateUser(id: number, updates: Partial<User>) {
  console.log(`Updating user ${id} with:`, updates);
}

// Благодаря утиной типизации, можно передать объект с любыми подмножествами свойств
updateUser(1, { name: "John" });           // OK
updateUser(2, { email: "john@example.com" }); // OK
updateUser(3, { name: "Jane", isActive: true }); // OK

// Даже объект с дополнительными свойствами может работать в некоторых контекстах
const updateData = {
  name: "Bob",
  tempId: 123  // дополнительное свойство
};
updateUser(4, updateData);  // OK в большинстве случаев
```

### 2. Совместимость с функциями обратного вызова

```typescript
interface EventHandler<T> {
  (data: T): void;
}

// Разные функции с одинаковой сигнатурой
const userHandler: EventHandler<{ id: number; name: string }> = (user) => {
  console.log(`User: ${user.name} (ID: ${user.id})`);
};

const productHandler: EventHandler<{ id: number; name: string; price: number }> = (product) => {
  console.log(`Product: ${product.name} ($${product.price})`);
};

// Благодаря утиной типизации и ковариантности возвращаемого значения
// более конкретный обработчик может быть использован как более общий
const genericHandler: EventHandler<{ id: number; name: string }> = productHandler;
// Это работает, потому что Product имеет все свойства User
```

## Ограничения и особенности

### 1. Совместимость при прямом присвоении

```typescript
interface Point {
  x: number;
  y: number;
}

// Прямое присвоение объектного литерала проверяется строже
// const point: Point = { x: 1, y: 2, z: 3 };  // Ошибка: лишнее свойство z

// Но через переменную работает:
const tempObj = { x: 1, y: 2, z: 3 };
const point: Point = tempObj;  // OK: z игнорируется
```

### 2. Совместимость перечислений

```typescript
enum Status { Success, Error }
enum Color { Red, Blue }

// Даже если значения одинаковы, перечисления несовместимы
const status = Status.Success;
// const color: Color = status;  // Ошибка: разные перечисления
```

### 3. Совместимость функций с разным количеством параметров

```typescript
function processCallback(cb: (x: number, y: number) => void) {
  cb(1, 2);
}

// Функция с меньшим количеством параметров может быть использована
processCallback((x: number) => console.log(x));  // OK: лишний параметр игнорируется

// Функция с большим количеством параметров может быть использована
processCallback((x: number, y: number, z: number) => console.log(x, y));  // OK: лишний параметр просто не используется
```

## Практические примеры из реальной разработки

### 1. Работа с формами и валидацией

```typescript
interface Validator<T> {
  validate(value: T): boolean;
  errorMessage: string;
}

// Разные валидаторы
const emailValidator = {
  validate: (email: string) => email.includes('@'),
  errorMessage: 'Invalid email format'
};

const minLengthValidator = {
  validate: (str: string) => str.length >= 6,
  errorMessage: 'Minimum length is 6 characters'
};

const numberValidator = {
  validate: (num: number) => num > 0,
  errorMessage: 'Number must be positive'
};

// Универсальная функция валидации
function validate<T>(value: T, validator: Validator<T>): boolean {
  if (!validator.validate(value)) {
    console.error(validator.errorMessage);
    return false;
  }
  return true;
}

// Использование разных валидаторов
validate("test@example.com", emailValidator);  // OK
validate("hello", minLengthValidator);         // OK
validate(-5, numberValidator);                 // OK
```

### 2. Создание middleware системы

```typescript
interface Middleware<T> {
  process(data: T): T;
}

// Разные типы middleware
const loggingMiddleware = {
  process: (req: { url: string; method: string }) => {
    console.log(`Processing request: ${req.method} ${req.url}`);
    return req;
  }
};

const authMiddleware = {
  process: (req: { url: string; method: string; user?: any }) => {
    // Добавление информации о пользователе
    return { ...req, user: { id: 1, name: "John" } };
  }
};

// Универсальная цепочка middleware
function runMiddleware<T>(data: T, middlewares: Middleware<T>[]): T {
  return middlewares.reduce((current, middleware) => middleware.process(current), data);
}

// Использование
const request = { url: "/api/users", method: "GET" };
const processedRequest = runMiddleware(
  request,
  [loggingMiddleware, authMiddleware]
);
```

## Лучшие практики

1. **Используйте интерфейсы для определения контрактов** - даже с утиной типизацией явные интерфейсы улучшают читаемость
2. **Будьте осторожны с лишними свойствами** - они могут привести к непредвиденному поведению
3. **Понимайте различия между ковариантностью и контравариантностью** - особенно в функциях
4. **Используйте утиную типизацию для гибкости API** - позволяйте пользователям передавать объекты с нужной структурой
5. **Тестируйте граничные случаи** - особенно при работе с необязательными свойствами
6. **Документируйте ожидаемую структуру** - особенно когда принимаете анонимные объекты

## Связи с другими концепциями

- [[ts/type-system/structural-typing-comprehensive|Структурная типизация]] - основа утиной типизации
- [[ts/type-system/type-compatibility-comprehensive|Совместимость типов]] - как определяется совместимость
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - для безопасной работы с разными структурами
- [[ts/type-system/type-guards|Защитники типов]] - для безопасного сужения типов
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - как они взаимодействуют с утиной типизацией