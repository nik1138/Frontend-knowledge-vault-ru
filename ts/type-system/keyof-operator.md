# Keyof Operator

Оператор `keyof` в TypeScript используется для получения объединения строковых литералов всех публичных свойств типа. Это мощный инструмент для создания типобезопасного кода, который работает с динамическими именами свойств.

## Основное использование

### Получение ключей объекта
```typescript
// Простой интерфейс
interface Person {
  name: string;
  age: number;
  email: string;
}

// Получение всех ключей типа
type PersonKeys = keyof Person; // "name" | "age" | "email"

// Использование в функции
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person: Person = {
  name: "Alice",
  age: 30,
  email: "alice@example.com"
};

// Использование
const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
const email = getProperty(person, "email");  // string

// Попытка использовать несуществующий ключ вызовет ошибку:
// const invalid = getProperty(person, "phone");  // ОШИБКА: "phone" не входит в "name" | "age" | "email"
```

## Практические примеры

### 1. Функция для получения значения свойства
```typescript
// Универсальная функция получения значения по ключу
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Пример использования
interface User {
  id: number;
  username: string;
  isActive: boolean;
}

const user: User = {
  id: 123,
  username: "john_doe",
  isActive: true
};

const userId = getValue(user, "id");        // number
const username = getValue(user, "username"); // string
const status = getValue(user, "isActive");   // boolean
```

### 2. Изменение значения свойства
```typescript
// Функция для установки значения свойства
function setValue<T, K extends keyof T>(obj: T, key: K, value: T[K]): T {
  obj[key] = value;
  return obj;
}

const updatedUser = setValue(user, "isActive", false);
// updatedUser: User с isActive = false
```

### 3. Создание проекции объекта
```typescript
// Функция для получения подмножества свойств
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) {
    result[key] = obj[key];
  }
  return result;
}

const userSubset = pick(user, ["id", "username"]);
// Тип: { id: number; username: string; }
```

## Продвинутое использование

### 1. Работа с вложенными объектами
```typescript
interface Address {
  street: string;
  city: string;
  country: string;
}

interface UserProfile {
  name: string;
  age: number;
  address: Address;
}

// Получение ключей вложенного объекта
type AddressKeys = keyof Address; // "street" | "city" | "country"

// Функция для работы с вложенными свойствами
function getNestedProperty<T, K1 extends keyof T, K2 extends keyof T[K1]>(
  obj: T,
  key1: K1,
  key2: K2
): T[K1][K2] {
  return obj[key1][key2];
}

const profile: UserProfile = {
  name: "Bob",
  age: 25,
  address: {
    street: "123 Main St",
    city: "New York",
    country: "USA"
  }
};

const city = getNestedProperty(profile, "address", "city"); // string
```

### 2. Ключи с индексными сигнатурами
```typescript
// Работа с объектами, имеющими индексные сигнатуры
interface Dictionary {
  [key: string]: number;
  special: boolean;
}

type DictKeys = keyof Dictionary; // string | "special"
// Объединение: строковые индексы + конкретные свойства

// Практический пример
function processDictionary(dict: Dictionary, key: keyof Dictionary) {
  const value = dict[key]; // number, если строковый ключ, boolean, если "special"
  return value;
}
```

### 3. Массивы и ключи
```typescript
// Ключи массивов
type ArrayKeys = keyof string[]; // "length" | "toString" | "pop" | "push" | ... (методы массива)
const arr = [1, 2, 3];
type ArrType = keyof typeof arr; // keyof Array<number> = индексы + методы массива

// Получение типа элемента массива
type ArrayElementType<T> = T extends (infer U)[] ? U : never;
type Element = ArrayElementType<number[]>; // number

// Работа с индексами массива
function getElement<T>(arr: T[], index: keyof T[]) {
  return arr[index as number]; // нужно приведение к number
}
```

## Сопоставляющие типы с keyof

### 1. Создание новых типов на основе ключей
```typescript
// Создание типа с теми же ключами, но разными типами значений
type StringifyValues<T> = {
  [K in keyof T]: string;
};

interface UserConfig {
  maxRetries: number;
  timeout: number;
  enabled: boolean;
}

type StringifiedConfig = StringifyValues<UserConfig>;
// Результат:
// {
//   maxRetries: string;
//   timeout: string;
//   enabled: string;
// }
```

### 2. Создание типа только с определенными свойствами
```typescript
// Создание типа с подмножеством свойств
type SubType<T, K extends keyof T> = {
  [P in K]: T[P];
};

type UserSubset = SubType<UserConfig, "maxRetries" | "timeout">;
// Результат:
// {
//   maxRetries: number;
//   timeout: number;
// }
```

### 3. Преобразование ключей
```typescript
// Создание типа с преобразованными ключами
type AddPrefix<T, P extends string> = {
  [K in keyof T as `${P}_${string & K}`]: T[K]
};

interface APIResponse {
  status: number;
  data: string;
  timestamp: number;
}

type PrefixedResponse = AddPrefix<APIResponse, "api">;
// Результат:
// {
//   api_status: number;
//   api_data: string;
//   api_timestamp: number;
// }
```

## Практические паттерны

### 1. Создание функции копирования
```typescript
// Безопасное копирование объекта
function clone<T>(obj: T): { [K in keyof T]: T[K] } {
  const result = {} as { [K in keyof T]: T[K] };
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      result[key] = obj[key];
    }
  }
  return result;
}
```

### 2. Проверка на изменение свойств
```typescript
// Создание типа для отслеживания изменений
type Changes<T> = {
  [K in keyof T]?: {
    previous: T[K];
    current: T[K];
  };
};

interface Settings {
  theme: "light" | "dark";
  fontSize: number;
  notifications: boolean;
}

function compareSettings(oldSettings: Settings, newSettings: Settings): Changes<Settings> {
  const changes: Partial<Changes<Settings>> = {};
  
  (Object.keys(oldSettings) as (keyof Settings)[]).forEach(key => {
    if (oldSettings[key] !== newSettings[key]) {
      changes[key] = {
        previous: oldSettings[key],
        current: newSettings[key]
      };
    }
  });
  
  return changes as Changes<Settings>;
}
```

### 3. Функция для обновления частичных данных
```typescript
// Функция для обновления только части объекта
function updatePartial<T>(obj: T, updates: Partial<{ [K in keyof T]: T[K] }>): T {
  const result = { ...obj };
  for (const key in updates) {
    if (Object.prototype.hasOwnProperty.call(updates, key)) {
      result[key] = updates[key];
    }
  }
  return result;
}

// Использование
const originalUser: User = { id: 1, username: "old_user", isActive: false };
const updatedUser = updatePartial(originalUser, { username: "new_user", isActive: true });
```

## Расширенные примеры

### 1. Создание безопасного getter'а и setter'а
```typescript
class TypedObject<T> {
  private data: T;
  
  constructor(initialData: T) {
    this.data = initialData;
  }
  
  // Getter с типобезопасностью
  get<K extends keyof T>(key: K): T[K] {
    return this.data[key];
  }
  
  // Setter с типобезопасностью
  set<K extends keyof T>(key: K, value: T[K]): void {
    this.data[key] = value;
  }
  
  // Получение всех ключей
  getKeys(): (keyof T)[] {
    return Object.keys(this.data) as (keyof T)[];
  }
}

const userTyped = new TypedObject(user);
const nameValue = userTyped.get("username"); // string
userTyped.set("isActive", true); // boolean (только правильный тип)
```

### 2. Создание системы событий с типизацией
```typescript
// Система событий с типобезопасной подпиской
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  data_update: { updatedFields: string[]; oldValue: any; newValue: any };
}

class TypedEventEmitter {
  private listeners: {
    [K in keyof EventMap]?: ((data: EventMap[K]) => void)[];
  } = {};
  
  on<K extends keyof EventMap>(event: K, listener: (data: EventMap[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }
  
  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }
}

// Использование
const emitter = new TypedEventEmitter();
emitter.on('user_login', (data) => {
  // data: { userId: number; timestamp: Date }
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.emit('user_login', {
  userId: 123,
  timestamp: new Date()
});
```

## Ограничения и подводные камни

### 1. Приватные и защищенные свойства
```typescript
class MyClass {
  public publicProp: string = "public";
  protected protectedProp: number = 42;
  private privateProp: boolean = true;
}

type MyClassKeys = keyof MyClass; // "publicProp" (только публичные свойства)
// protectedProp и privateProp не включаются в keyof
```

### 2. Статические свойства
```typescript
class MyClassWithStatic {
  static staticProp: string = "static";
  instanceProp: string = "instance";
}

type Keys = keyof MyClassWithStatic; // "instanceProp" (не включает статические)

// Для статических свойств:
type StaticKeys = keyof typeof MyClassWithStatic; // "staticProp"
```

### 3. Объединения типов
```typescript
interface A { a: string; }
interface B { b: number; }

type UnionType = A | B;
type UnionKeys = keyof UnionType; // "a" | "b" (объединение всех ключей)

// Однако при обращении к свойствам могут быть ограничения:
function processUnion(union: UnionType) {
  // union.a и union.b могут быть недоступны без проверки типа
  // потому что не все варианты объединения имеют все ключи
}
```

## Лучшие практики

### 1. Использование в утилитах
```typescript
// Создание собственных утилит с keyof
type OptionalProperty<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface RequiredUser {
  id: number;
  name: string;
  email: string;
  password: string;
}

type CreateUserInput = OptionalProperty<RequiredUser, "id">;
// То же, что и RequiredUser, но id становится необязательным
```

### 2. Обеспечение безопасности при рефакторинге
```typescript
// При изменении интерфейса все места, использующие keyof, автоматически обновляются
interface ApiUser {
  id: number;
  name: string;
  // email: string; // если добавить это свойство
}

// Функция, использующая keyof, будет автоматически обновлена
function validateUser(user: ApiUser, key: keyof ApiUser): boolean {
  // Теперь можно безопасно передавать "email" как ключ
  return user[key] !== undefined;
}
```

## Совместимость с другими возможностями TypeScript

### 1. С условными типами
```typescript
// Условные типы с keyof
type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];

interface ExampleObject {
  name: string;
  age: number;
  greet(): string;
  calculate(): number;
}

type PropsOnly = NonFunctionPropertyNames<ExampleObject>; // "name" | "age"
```

### 2. С сопоставляющими типами
```typescript
// Сопоставляющие типы с keyof
type ReadonlyPartial<T> = {
  readonly [K in keyof T]?: T[K];
};

type ReadOnlyOptionalUser = ReadonlyPartial<RequiredUser>;
// Все свойства Optional и ReadOnly
```

## Связь с другими концепциями
- [[../advanced-types/mapped-types]] - сопоставляющие типы, использующие keyof
- [[../advanced-types/conditional-types]] - условные типы, использующие keyof
- [[index-access-types]] - получение типов значений по ключам
- [[TS Generics]] - обобщения, использующие keyof