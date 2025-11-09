# Types API

## Введение

Системы типов в JavaScript, особенно TypeScript, предоставляют мощные инструменты для создания надежных и поддерживаемых приложений. В этой главе мы рассмотрим все аспекты типизации: базовые типы, интерфейсы, классы, дженерики, продвинутые типы и лучшие практики использования систем типов.

## Содержание

- [[Базовые типы]]
- [[Интерфейсы и типы]]
- [[Классы и наследование с типами]]
- [[Дженерики]]
- [[Продвинутые типы]]
- [[Типизация функций]]
- [[Совместимость типов]]
- [[Создание типов в рантайме]]
- [[Библиотеки типизации]]

## Базовые типы

TypeScript расширяет JavaScript, добавляя статическую систему типов. Это позволяет обнаруживать ошибки на этапе компиляции.

### Примитивные типы

```typescript
// Базовые типы
let isDone: boolean = false;
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
let big: bigint = 100n;
let color: string = "blue";

// Массивы
let list: number[] = [1, 2, 3];
let listGeneric: Array<number> = [1, 2, 3];
let mixed: (number | string)[] = [1, "hello", 2];

// Кортежи
let x: [string, number];
x = ["hello", 10]; // OK
// x = [10, "hello"]; // Ошибка!

// Перечисления
enum Color {
    Red = 1,
    Green = 2,
    Blue = 4
}
let c: Color = Color.Green;

// Неизвестные типы
let notSure: unknown = 4;
notSure = "maybe a string instead";
notSure = false; // ОК, может быть любым значением

// Любой тип (лучше избегать)
let anyType: any = 4;
anyType = "could be anything";
anyType = false;

// Объекты
let obj: object = {
    name: "John",
    age: 30
};

// Null и Undefined
let u: undefined = undefined;
let n: null = null;

// Void - отсутствие значения
function warnUser(): void {
    console.log("This is my warning message");
}

// Never - тип для функций, которые никогда не возвращаются
function error(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {
    }
}
```

### Работа с типами

```typescript
// Типизация переменных
let userName: string = "Alice";
let userAge: number = 25;
let isActive: boolean = true;

// Выведение типов
let inferredString = "This is a string"; // string
let inferredNumber = 42; // number
let inferredBoolean = true; // boolean

// Типизация объектов
let user: {
    name: string;
    age: number;
    email?: string; // опциональное поле
    address: {
        street: string;
        city: string;
    };
} = {
    name: "John",
    age: 30,
    address: {
        street: "123 Main St",
        city: "New York"
    }
};

// Типизация функций
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };

// Стрелочные функции с типизацией
const add: (a: number, b: number) => number = (a, b) => a + b;

// Функция с опциональными параметрами
function buildName(firstName: string, lastName?: string) {
    if (lastName) {
        return firstName + " " + lastName;
    } else {
        return firstName;
    }
}

// Функция с параметрами по умолчанию
function buildNameWithDefault(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

// Функция с rest параметрами
function buildNameWithRest(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}
```

## Интерфейсы и типы

Интерфейсы и типы позволяют определять структуру объектов и создавать переиспользуемые определения типов.

### Интерфейсы

```typescript
// Основы интерфейсов
interface User {
    readonly id: number; // только для чтения
    name: string;
    email: string;
    age?: number; // опциональное поле
    phoneNumbers?: string[];
}

interface Employee extends User {
    employeeId: number;
    department: string;
    salary: number;
}

// Интерфейс для функций
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
};

// Интерфейс с индексными сигнатурами
interface StringArray {
    [index: number]: string;
}

interface NumberDictionary {
    [index: string]: number;
    length: number;    // тип свойства length должен быть подтипом индексного типа
    name: string;      // OK, string является подтипом string
}

// Интерфейс с вызываемыми свойствами
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = (function (start: number) { return "Hello"; }) as Counter;
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

// Гибридные типы
interface LinkedList<T> {
    value: T;
    next?: LinkedList<T>;
    [index: number]: T;
}

// Интерфейсы для классов
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

### Типы (Type Aliases)

```typescript
// Типы vs Интерфейсы
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;

function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    } else {
        return n();
    }
}

// Объединения типов
type Easing = "ease-in" | "ease-out" | "ease-in-out";

class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        } else if (easing === "ease-out") {
            // ...
        } else if (easing === "ease-in-out") {
            // ...
        }
    }
}

// Типы с дженериками
type Container<T> = { value: T };

// Типы для функций
type Lazy<T> = T | (() => T);

function getValue<T>(wrapper: Lazy<T>): T {
    if (typeof wrapper === "function") {
        return wrapper();
    } else {
        return wrapper;
    }
}

// Типы с пересечениями
type Identity<T, U> = T & U;

let identity: Identity<{ name: string }, { age: number }> = {
    name: "John",
    age: 30
};

// Дополнительные примеры типов
type StringOrNumber = string | number;
type Text = string | { text: string };
type NameLookup = Dictionary<string, Person>;
type Callback<T> = (data: T) => void;
type Dictionary<T, U> = { [K: string]: T | U };
```

## Классы и наследование с типами

### Классы с типами

```typescript
// Классы с модификаторами доступа
class Animal {
    private name: string;
    protected age: number;
    public species: string;
    
    constructor(name: string, age: number, species: string) {
        this.name = name;
        this.age = age;
        this.species = species;
    }
    
    public move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
    
    protected makeSound(): string {
        return "Some generic animal sound";
    }
    
    // Геттеры и сеттеры
    get Name(): string {
        return this.name;
    }
    
    set Name(newName: string) {
        if (newName.length < 3) {
            throw new Error("Name too short");
        }
        this.name = newName;
    }
}

// Класс-наследник
class Dog extends Animal {
    private breed: string;
    
    constructor(name: string, age: number, breed: string) {
        super(name, age, "Canis lupus");
        this.breed = breed;
    }
    
    public bark() {
        console.log("Woof! Woof!");
    }
    
    // Переопределение метода
    public move(distanceInMeters = 5) {
        console.log("Running...");
        super.move(distanceInMeters);
    }
    
    // Доступ к защищенному методу родителя
    public animalSound() {
        return this.makeSound();
    }
}

// Абстрактные классы
abstract class Department {
    protected name: string;
    
    constructor(name: string) {
        this.name = name;
    }
    
    printName(): void {
        console.log("Department name: " + this.name);
    }
    
    // Абстрактный метод - должен быть реализован в наследниках
    abstract printMeeting(): void;
}

class AccountingDepartment extends Department {
    constructor() {
        super("Accounting and Auditing"); // Конструкторы в производных классах должны вызывать super()
    }
    
    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }
    
    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}
```

### Продвинутая типизация классов

```typescript
// Классы с дженериками
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

// Классы с статическими свойствами
class Grid {
    static origin = { x: 0, y: 0 };
    
    calculateDistanceFromOrigin(point: { x: number; y: number; }) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    
    constructor(public scale: number) { }
}

// Приватные поля ES2022 (также поддерживаются в TypeScript)
class Person {
    #name: string;
    
    constructor(name: string) {
        this.#name = name;
    }
    
    greet() {
        return `Hello, my name is ${this.#name}!`;
    }
    
    get name() {
        return this.#name;
    }
    
    set name(value: string) {
        if (value.length < 3) {
            throw new Error("Name too short");
        }
        this.#name = value;
    }
}

// Классы как интерфейсы
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = { x: 1, y: 2, z: 3 };
```

## Дженерики

Дженерики позволяют создавать компоненты, которые работают с множеством типов, а не с одним конкретным.

### Основы дженериков

```typescript
// Дженерик функции
function identity<T>(arg: T): T {
    return arg;
}

// Использование дженерик функции
let output1 = identity<string>("myString");
let output2 = identity<number>(100);
let output3 = identity("myString"); // Выведение типа

// Дженерик интерфейсы
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identityFn<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identityFn;

// Дженерик классы
class GenericNumberClass<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let stringNumeric = new GenericNumberClass<string>();
stringNumeric.zeroValue = "abc";
stringNumeric.add = function(x, y) { return x + y; };

// Ограничения дженериков
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length); // Теперь мы знаем, что у arg есть свойство length
    return arg;
}

// Использование ограничений
loggingIdentity({ length: 10, value: 3 }); // OK
// loggingIdentity(3); // Ошибка!

// Определение ключей объекта
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let xObj = { a: 1, b: 2, c: 3 };
getProperty(xObj, "a"); // Okay
// getProperty(xObj, "m"); // Error
```

### Продвинутые дженерики

```typescript
// Дженерики с несколькими типами
class KeyValuePair<K, V> {
    constructor(public key: K, public value: V) {}
}

let pair = new KeyValuePair<string, number>("age", 25);

// Условные типы
type TypeName<T> = 
    T extends string ? "string" :
    T extends number ? "number" :
    T extends boolean ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<() => void>;  // "function"

// Распределительные условные типы
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T2 = Boxed<string>;  // BoxedValue<string>
type T3 = Boxed<string[]>;  // BoxedArray<string>

// Индексные типы
interface PersonIndex {
    name: string;
    age: number;
    location: string;
}

type K1 = keyof PersonIndex; // "name" | "age" | "location"
type K2 = keyof PersonIndex[]; // "length" | "push" | "pop" | ...
type K3 = keyof { [x: number]: PersonIndex }; // number | string (числа автоматически преобразуются в строки)

// Типы поиска
type P1 = PersonIndex["name"]; // string
type P2 = PersonIndex["name" | "age"]; // string | number
type P3 = PersonIndex[keyof PersonIndex]; // string | number
type P4 = string["charAt"]; // (pos: number) => string
type P5 = string[]["push"]; // (...items: string[]) => number
type P6 = string[][number]; // string

// Сопоставленные типы
type Partial<T> = {
    [P in keyof T]?: T[P];
};

type Required<T> = {
    [P in keyof T]-?: T[P];
};

type ReadOnly<T> = {
    readonly [P in keyof T]: T[P];
};

// Сопоставленные типы с преобразованием
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface PersonGetters {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<PersonGetters>;
// LazyPerson теперь:
// {
//     getName: () => string;
//     getAge: () => number;
//     getLocation: () => string;
// }
```

## Продвинутые типы

### Утилитарные типы

```typescript
// Встроенные утилитарные типы TypeScript

// Partial<T> - делает все свойства опциональными
interface Todo {
    title: string;
    description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate };
}

// Readonly<T> - делает все свойства только для чтения
type PointReadonly = {
    readonly x: number;
    readonly y: number;
};

let p1: PointReadonly = { x: 10, y: 20 };
// p1.x = 5; // ошибка!

// Record<K,T> - объект с ключами типа K и значениями типа T
interface PageInfo {
    title: string;
}

type Page = "home" | "about" | "contact";

const xRecord: Record<Page, PageInfo> = {
    home: { title: "Home" },
    about: { title: "About" },
    contact: { title: "Contact" }
};

// Pick<T,K> - выбирает подмножество свойств
interface TodoPick {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Pick<TodoPick, "title" | "completed">;

const todoPreview: TodoPreview = {
    title: "Clean room",
    completed: false,
};

// Omit<T,K> - исключает подмножество свойств
type TodoInfo = Omit<TodoPick, "completed" | "description">;

const todoInfo: TodoInfo = {
    title: "Pick up kids",
};

// Exclude<T,U> - исключает типы из T, которые присутствуют в U
type T0Exclude = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1Exclude = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2Exclude = Exclude<string | number | (() => void), Function>;  // string | number

// Extract<T,U> - извлекает типы из T, которые присутствуют в U
type T0Extract = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1Extract = Extract<string | number | (() => void), Function>;  // () => void

// NonNullable<T> - исключает null и undefined
type T0NonNullable = NonNullable<string | number | undefined>; // string | number
type T1NonNullable = NonNullable<string[] | null | undefined>; // string[]

// Parameters<T> - получает типы параметров функции
declare function f1(arg: { a: number; b: string }): void;
type T0Params = Parameters<() => string>;  // []
type T1Params = Parameters<(s: string) => void>;  // [s: string]
type T2Params = Parameters<typeof f1>;  // [{ a: number, b: string }]

// ConstructorParameters<T> - получает типы параметров конструктора
type T0Constructor = ConstructorParameters<ErrorConstructor>;  // [message?: string]
type T1Constructor = ConstructorParameters<FunctionConstructor>;  // string[]
type T2Constructor = ConstructorParameters<DateConstructor>;  // [value: any]

// ReturnType<T> - получает возвращаемый тип функции
type T0Return = ReturnType<() => string>;  // string
type T1Return = ReturnType<(s: string) => void>;  // void
type T2Return = ReturnType<<T>() => T>;  // {}
type T3Return = ReturnType<<T extends U, U extends number[]>() => T>;  // number[]

// InstanceType<T> - получает тип экземпляра класса
class C {
    x = 0;
    y = 0;
}

type T0Instance = InstanceType<typeof C>;  // C
type T1Instance = InstanceType<any>;  // any
type T2Instance = InstanceType<Function>;  // {}
```

### Создание собственных утилитарных типов

```typescript
// Собственные утилитарные типы

// DeepPartial - глубокое превращение всех свойств в опциональные
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface Address {
    street: string;
    city: string;
    country: string;
}

interface UserDeep {
    name: string;
    age: number;
    address: Address;
}

type PartialUser = DeepPartial<UserDeep>;
// Теперь все вложенные свойства также опциональны

// DeepReadonly - глубокое превращение всех свойств в readonly
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Mutable - делает все свойства изменяемыми (обратное Readonly)
type Mutable<T> = {
    -readonly [P in keyof T]: T[P];
};

// RequiredBut<T, K> - делает определенные свойства обязательными
type RequiredBut<T, K extends keyof T> = Required<Pick<T, K>> & Partial<Omit<T, K>>;

interface UserRequired {
    name?: string;
    email?: string;
    id: string;
}

type MandatoryEmailUser = RequiredBut<UserRequired, 'email'>;
// email теперь обязательное поле, остальные опциональны

// PromiseValue - извлекает тип значения из Promise
type PromiseValue<T> = T extends Promise<infer U> ? U : T;

type T1Promise = PromiseValue<Promise<string>>; // string
type T2Promise = PromiseValue<number>; // number

// Flatten - уплощает типы массивов
type Flatten<T> = T extends Array<infer U> ? U : T;

type T1Flatten = Flatten<number[]>; // number
type T2Flatten = Flatten<string>; // string

// Get - получение вложенного свойства по пути
type Get<T, K> = K extends keyof T
    ? T[K]
    : K extends `${infer P}.${infer S}`
        ? P extends keyof T
            ? Get<T[P], S>
            : never
        : never;

interface UserNested {
    name: string;
    address: {
        street: string;
        city: string;
        location: {
            lat: number;
            lng: number;
        };
    };
}

type CityType = Get<UserNested, 'address.city'>; // string
type LatType = Get<UserNested, 'address.location.lat'>; // number
```

## Типизация функций

### Продвинутая типизация функций

```typescript
// Перегрузки функций
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
    if (d !== undefined && y !== undefined) {
        return new Date(y, mOrTimestamp, d);
    } else {
        return new Date(mOrTimestamp);
    }
}

const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);

// Типизация коллбэков
interface EventCallback {
    <T>(data: T): void;
}

function subscribe(callback: EventCallback) {
    // реализация
}

// Типизация функций с this
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}

class Handler {
    info: string;
    constructor() {
        this.info = "message";
    }
    onClickBad(this: void, e: Event) {
        // this.info = e.message // ошибка
    }
    onClickGood(this: Handler, e: Event) {
        this.info = e.message; // OK
    }
}

// Типизация функций высшего порядка
type HOC<TProps, TEnhancedProps = {}> = 
    (component: React.ComponentType<TProps>) => 
    React.ComponentType<Omit<TEnhancedProps, keyof TProps> & TProps>;

// Каррированные функции с типами
function curry<T1, T2, T3, R>(
    fn: (a: T1, b: T2, c: T3) => R
): (a: T1) => (b: T2) => (c: T3) => R {
    return (a) => (b) => (c) => fn(a, b, c);
}

function addThreeNumbers(a: number, b: number, c: number): number {
    return a + b + c;
}

const curriedAdd = curry(addThreeNumbers);
const add5 = curriedAdd(5);
const add5and3 = add5(3);
const result = add5and3(2); // 10
```

## Примеры из промышленной разработки

### Система типов для API клиента

```typescript
// Типы для HTTP клиента
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';

interface ApiRequestConfig {
    url: string;
    method: HttpMethod;
    headers?: Record<string, string>;
    params?: Record<string, any>;
    data?: any;
}

interface ApiResponse<T = any> {
    data: T;
    status: number;
    statusText: string;
    headers: Record<string, string>;
}

interface ApiError {
    message: string;
    code: number;
    response?: ApiResponse;
}

// Абстрактный класс API клиента
abstract class ApiClient {
    protected abstract request<T>(config: ApiRequestConfig): Promise<ApiResponse<T>>;
    
    async get<T>(url: string, config?: Partial<ApiRequestConfig>): Promise<ApiResponse<T>> {
        return this.request<T>({ method: 'GET', url, ...config });
    }
    
    async post<T>(url: string, data?: any, config?: Partial<ApiRequestConfig>): Promise<ApiResponse<T>> {
        return this.request<T>({ method: 'POST', url, data, ...config });
    }
    
    async put<T>(url: string, data?: any, config?: Partial<ApiRequestConfig>): Promise<ApiResponse<T>> {
        return this.request<T>({ method: 'PUT', url, data, ...config });
    }
    
    async delete<T>(url: string, config?: Partial<ApiRequestConfig>): Promise<ApiResponse<T>> {
        return this.request<T>({ method: 'DELETE', url, ...config });
    }
}

// Конкретная реализация
class FetchApiClient extends ApiClient {
    async request<T>(config: ApiRequestConfig): Promise<ApiResponse<T>> {
        try {
            const { url, method, headers = {}, params = {}, data } = config;
            
            // Формирование URL с параметрами
            const urlWithParams = new URL(url, window.location.origin);
            Object.entries(params).forEach(([key, value]) => {
                if (value !== undefined) {
                    urlWithParams.searchParams.append(key, String(value));
                }
            });
            
            const response = await fetch(urlWithParams.toString(), {
                method,
                headers: {
                    'Content-Type': 'application/json',
                    ...headers
                },
                body: data ? JSON.stringify(data) : undefined
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            const responseData = await response.json();
            
            return {
                data: responseData,
                status: response.status,
                statusText: response.statusText,
                headers: Object.fromEntries(response.headers.entries())
            };
        } catch (error) {
            throw {
                message: error instanceof Error ? error.message : 'Network error',
                code: 0,
                response: undefined
            } as ApiError;
        }
    }
}

// Типы для конкретного API
interface User {
    id: number;
    name: string;
    email: string;
    createdAt: string;
}

interface CreateUserRequest {
    name: string;
    email: string;
    password: string;
}

interface UpdateUserRequest {
    name?: string;
    email?: string;
}

// Сервис для работы с пользователями
class UserService {
    constructor(private apiClient: ApiClient) {}
    
    async getUsers(): Promise<User[]> {
        const response = await this.apiClient.get<User[]>('/api/users');
        return response.data;
    }
    
    async getUser(id: number): Promise<User> {
        const response = await this.apiClient.get<User>(`/api/users/${id}`);
        return response.data;
    }
    
    async createUser(userData: CreateUserRequest): Promise<User> {
        const response = await this.apiClient.post<User>('/api/users', userData);
        return response.data;
    }
    
    async updateUser(id: number, userData: UpdateUserRequest): Promise<User> {
        const response = await this.apiClient.put<User>(`/api/users/${id}`, userData);
        return response.data;
    }
    
    async deleteUser(id: number): Promise<void> {
        await this.apiClient.delete<void>(`/api/users/${id}`);
    }
}

// Использование
const apiClient = new FetchApiClient();
const userService = new UserService(apiClient);

// userService.getUsers().then(users => console.log(users));
```

### Система типов для валидации данных

```typescript
// Система валидации с типами
interface Validator<T> {
    validate(value: unknown): value is T;
    errors: string[];
}

// Абстрактный класс валидатора
abstract class BaseValidator<T> implements Validator<T> {
    errors: string[] = [];
    
    abstract validate(value: unknown): value is T;
    
    protected addError(message: string): void {
        this.errors.push(message);
    }
    
    protected clearErrors(): void {
        this.errors = [];
    }
    
    isValid(value: unknown): value is T {
        this.clearErrors();
        return this.validate(value);
    }
}

// Валидаторы для примитивных типов
class StringValidator extends BaseValidator<string> {
    constructor(private minLength: number = 0, private maxLength?: number) {
        super();
    }
    
    validate(value: unknown): value is string {
        if (typeof value !== 'string') {
            this.addError('Value must be a string');
            return false;
        }
        
        if (value.length < this.minLength) {
            this.addError(`String must be at least ${this.minLength} characters long`);
            return false;
        }
        
        if (this.maxLength && value.length > this.maxLength) {
            this.addError(`String must be no more than ${this.maxLength} characters long`);
            return false;
        }
        
        return true;
    }
}

class NumberValidator extends BaseValidator<number> {
    constructor(private min?: number, private max?: number) {
        super();
    }
    
    validate(value: unknown): value is number {
        if (typeof value !== 'number' || isNaN(value)) {
            this.addError('Value must be a number');
            return false;
        }
        
        if (this.min !== undefined && value < this.min) {
            this.addError(`Number must be at least ${this.min}`);
            return false;
        }
        
        if (this.max !== undefined && value > this.max) {
            this.addError(`Number must be no more than ${this.max}`);
            return false;
        }
        
        return true;
    }
}

// Композитный валидатор для объектов
class ObjectValidator<T extends Record<string, any>> extends BaseValidator<T> {
    private validators: { [K in keyof T]: Validator<T[K]> } = {} as any;
    
    addField<K extends keyof T>(fieldName: K, validator: Validator<T[K]>): this {
        this.validators[fieldName] = validator;
        return this;
    }
    
    validate(value: unknown): value is T {
        if (typeof value !== 'object' || value === null) {
            this.addError('Value must be an object');
            return false;
        }
        
        let isValid = true;
        
        for (const [fieldName, validator] of Object.entries(this.validators)) {
            const fieldValue = (value as Record<string, unknown>)[fieldName];
            
            if (!validator.isValid(fieldValue)) {
                isValid = false;
                this.errors.push(...validator.errors.map(error => `${fieldName}: ${error}`));
            }
        }
        
        return isValid;
    }
}

// Использование системы валидации
const userValidator = new ObjectValidator<Record<string, any>>()
    .addField('name', new StringValidator(1, 50))
    .addField('age', new NumberValidator(0, 150))
    .addField('email', new StringValidator(5, 255));

const userData = {
    name: 'John Doe',
    age: 30,
    email: 'john@example.com'
};

if (userValidator.isValid(userData)) {
    console.log('Данные пользователя валидны');
} else {
    console.log('Ошибки валидации:', userValidator.errors);
}

// Валидаторы с дженериками для конкретных типов
class TypedObjectValidator<T> extends BaseValidator<T> {
    constructor(private schema: { [K in keyof T]: Validator<T[K]> }) {
        super();
    }
    
    validate(value: unknown): value is T {
        if (typeof value !== 'object' || value === null) {
            this.addError('Value must be an object');
            return false;
        }
        
        let isValid = true;
        
        for (const [key, validator] of Object.entries(this.schema)) {
            const fieldValue = (value as Record<string, unknown>)[key];
            
            if (!validator.isValid(fieldValue)) {
                isValid = false;
                this.errors.push(...validator.errors.map(error => `${key}: ${error}`));
            }
        }
        
        return isValid;
    }
}

// Типизированный валидатор для конкретного интерфейса
interface Product {
    id: number;
    name: string;
    price: number;
    inStock: boolean;
}

const productValidator = new TypedObjectValidator<Product>({
    id: new NumberValidator(1),
    name: new StringValidator(1, 100),
    price: new NumberValidator(0),
    inStock: {
        validate: (value: unknown): value is boolean => typeof value === 'boolean',
        errors: []
    } as Validator<boolean>
});

const product: unknown = {
    id: 1,
    name: 'Product Name',
    price: 29.99,
    inStock: true
};

if (productValidator.isValid(product)) {
    console.log('Продукт валиден:', product);
} else {
    console.log('Ошибки валидации продукта:', productValidator.errors);
}
```

## Лучшие практики

### 1. Использование readonly для иммутабельности

```typescript
// Хорошо: явное указание иммутабельности
interface User {
    readonly id: number;
    readonly name: string;
    readonly email: string;
}

function processUser(user: Readonly<User>) {
    // user.id = 5; // Ошибка компиляции
    return user;
}
```

### 2. Использование дженериков для переиспользуемости

```typescript
// Хорошо: дженерик функция
function identity<T>(arg: T): T {
    return arg;
}

// Плохо: конкретный тип
function stringIdentity(arg: string): string {
    return arg;
}
```

### 3. Использование пересечений типов для композиции

```typescript
// Хорошо: композиция через пересечение
interface Timestamped {
    createdAt: Date;
    updatedAt: Date;
}

interface SoftDelete {
    deletedAt?: Date;
}

type VersionedEntity<T> = T & Timestamped & SoftDelete;
```

## Безопасность

При работе с типами важно учитывать безопасность:

```typescript
// Безопасная обработка пользовательского ввода с типами
interface SafeUserInput {
    readonly name: string;
    readonly email: string;
    readonly age: number;
}

function validateAndSanitizeInput(input: unknown): SafeUserInput | null {
    if (typeof input !== 'object' || input === null) {
        return null;
    }
    
    const obj = input as Record<string, unknown>;
    
    // Валидация и санитизация
    const name = typeof obj.name === 'string' ? obj.name.trim() : '';
    const email = typeof obj.email === 'string' ? obj.email.trim().toLowerCase() : '';
    const age = typeof obj.age === 'number' ? obj.age : NaN;
    
    // Проверка на безопасность
    if (name.includes('<') || name.includes('>') || 
        email.includes('<') || email.includes('>')) {
        return null; // Потенциальная XSS-атака
    }
    
    // Валидация
    if (!name || name.length < 2 || name.length > 50) {
        return null;
    }
    
    if (!email || !email.includes('@') || email.length > 255) {
        return null;
    }
    
    if (isNaN(age) || age < 0 || age > 150) {
        return null;
    }
    
    return { name, email, age } as SafeUserInput;
}

// Использование
const userInput = { name: 'John', email: 'john@example.com', age: 30 };
const safeInput = validateAndSanitizeInput(userInput);

if (safeInput) {
    console.log('Безопасные данные:', safeInput);
} else {
    console.log('Небезопасный ввод!');
}
```

## Теги

#typescript #typing #types #interfaces #generics #type-safety #static-typing #javascript