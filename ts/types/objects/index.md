# Объектные типы в TypeScript

## Основы объектных типов

Объектные типы в TypeScript позволяют определять структуру объектов, включая их свойства и методы.

### Определение объектного типа через интерфейс

```ts
interface User {
  name: string;
  age: number;
  email: string;
  isActive: boolean;
}

const user: User = {
  name: "Alice",
  age: 30,
  email: "alice@example.com",
  isActive: true
};
```

### Определение объектного типа через type alias

```ts
type Product = {
  id: number;
  title: string;
  price: number;
  description?: string; // необязательное свойство
};

const product: Product = {
  id: 1,
  title: "Laptop",
  price: 999.99
  // description необязательно
};
```

## Необязательные и readonly свойства

```ts
interface Config {
  apiUrl: string;
  timeout?: number;        // необязательное свойство
  readonly version: string; // только для чтения
}

let config: Config = {
  apiUrl: "https://api.example.com",
  version: "1.0.0"
};
// config.version = "2.0.0"; // Ошибка - нельзя изменить readonly свойство
```

## Индексные сигнатуры

```ts
// Объект с произвольными строковыми ключами
interface StringDictionary {
  [key: string]: string;
}

const dict: StringDictionary = {
  key1: "value1",
  key2: "value2"
};

// Объект с числовыми ключами
interface NumberDictionary {
  [index: number]: string;
}

const arr: NumberDictionary = ["first", "second", "third"];
```

## Вложенные объекты

```ts
interface Address {
  street: string;
  city: string;
  country: string;
}

interface Person {
  name: string;
  age: number;
  address: Address;
  contacts: {
    email: string;
    phone?: string;
  };
}

const person: Person = {
  name: "Bob",
  age: 25,
  address: {
    street: "123 Main St",
    city: "New York",
    country: "USA"
  },
  contacts: {
    email: "bob@example.com"
  }
};
```

## Методы в объектных типах

```ts
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
  multiply: (a: number, b: number) => number;
}

const calc: Calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  },
  multiply: (a, b) => a * b
};
```

## Функции как свойства объекта

```ts
interface ApiClient {
  baseUrl: string;
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
}

const client: ApiClient = {
  baseUrl: "https://api.example.com",
  get: async <T>(url: string): Promise<T> => {
    const response = await fetch(`${client.baseUrl}${url}`);
    return response.json();
  },
  post: async <T>(url: string, data: any): Promise<T> => {
    const response = await fetch(`${client.baseUrl}${url}`, {
      method: 'POST',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json' }
    });
    return response.json();
  }
};
```

## Объединения и пересечения объектных типов

```ts
// Пересечение типов
interface Identifiable {
  id: number;
}

interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

type Entity = Identifiable & Timestamped;

const entity: Entity = {
  id: 1,
  createdAt: new Date(),
  updatedAt: new Date()
};

// Объединение объектных типов
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

## Ключевые концепции

- **Интерфейсы** vs **Type aliases**: оба подхода позволяют определять объектные типы
- **Необязательные свойства**: использование `?` после имени свойства
- **Только для чтения**: использование `readonly` для недоступных для изменения свойств
- **Индексы**: определение типов для объектов с динамическими ключами
- **Вложенные структуры**: объекты могут содержать другие объекты как свойства

## Дальнейшее изучение

- [[../object-types|Объектные типы подробнее]]
- [[../union-types|Объединения типов]]
- [[../intersection-types|Пересечения типов]]
- [[../../interfaces-classes/interfaces|Интерфейсы]]
- [[../../interfaces-classes/classes|Классы]]