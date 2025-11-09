# Объектные типы в TypeScript

## Введение

Объектные типы в TypeScript позволяют определять структуру объектов, включая их свойства и методы. Они обеспечивают строгую типизацию для сложных структур данных, что позволяет избежать ошибок и улучшить читаемость кода.

## Основные способы определения объектных типов

### Интерфейсы

Интерфейсы - это один из основных способов определения объектных типов в TypeScript:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

// Использование интерфейса
const user: User = {
  id: 1,
  name: "Алексей",
  email: "alexey@example.com",
  isActive: true
};
```

### Псевдонимы типов

Псевдонимы типов (type aliases) также могут использоваться для определения объектных типов:

```typescript
type Product = {
  id: number;
  title: string;
  price: number;
  description?: string; // необязательное свойство
};

// Использование псевдонима типа
const product: Product = {
  id: 101,
  title: "Ноутбук",
  price: 50000
};
```

## Опциональные и readonly свойства

### Опциональные свойства

Свойства могут быть помечены как необязательные с помощью оператора `?`:

```typescript
interface UserProfile {
  id: number;
  name: string;
  email: string;
  bio?: string;      // необязательное свойство
  avatar?: string;   // необязательное свойство
  lastLogin: Date | null; // может быть датой или null
}
```

### Свойства только для чтения

Свойства могут быть помечены как `readonly`, что предотвращает их изменение после инициализации:

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

const point: Point = { x: 10, y: 20 };
// point.x = 5; // ОШИБКА: Невозможно присвоить значение свойству "x", так как оно только для чтения
```

## Методы в объектных типах

Объектные типы могут включать определения методов:

```typescript
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
  multiply(a: number, b: number): number;
  divide(a: number, b: number): number;
}

const calc: Calculator = {
  add(a, b) { return a + b; },
  subtract(a, b) { return a - b; },
  multiply(a, b) { return a * b; },
  divide(a, b) { 
    if (b === 0) throw new Error("Деление на ноль");
    return a / b; 
  }
};
```

## Индексные сигнатуры

Индексные сигнатуры позволяют определять типы для объектов с динамическими свойствами:

```typescript
// Объект, где ключи - строки, а значения - числа
interface StringToNumber {
  [key: string]: number;
}

const scores: StringToNumber = {
  math: 95,
  physics: 87,
  chemistry: 92
};

// Объект с числовыми ключами и строковыми значениями
interface NumberToString {
  [key: number]: string;
}

const months: NumberToString = {
  1: "Январь",
  2: "Февраль",
  3: "Март"
};
```

## Наследование интерфейсов

Интерфейсы могут наследовать другие интерфейсы:

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: number;
  department: string;
  salary: number;
}

const employee: Employee = {
  name: "Елена",
  age: 30,
  employeeId: 12345,
  department: "Разработка",
  salary: 100000
};
```

## Гибридные типы

Объектные типы могут комбинировать различные возможности:

```typescript
interface Counter {
  // Свойства
  count: number;
  
  // Методы
  increment(): void;
  decrement(): void;
  reset(): void;
  
  // Индексная сигнатура
  [key: string]: any;
  
  // Функция (если объект может быть вызван как функция)
  (start: number): string;
}

// Пример реализации
const counter: Counter = {
  count: 0,
  increment() { this.count++; },
  decrement() { this.count--; },
  reset() { this.count = 0; },
  [Symbol.toStringTag]: 'Counter',
  
  // Позволяет вызывать объект как функцию
  (start: number): string {
    this.count = start;
    return `Счетчик установлен на ${start}`;
  }
};
```

## Расширенные примеры

### Сложная структура с вложенными объектами

```typescript
interface Address {
  street: string;
  city: string;
  country: string;
  zipCode: string;
}

interface ContactInfo {
  email: string;
  phone: string;
  address: Address;
}

interface Company {
  id: number;
  name: string;
  contact: ContactInfo;
  employees: Employee[];
}

const company: Company = {
  id: 1,
  name: "ТехноЛаб",
  contact: {
    email: "info@techlab.ru",
    phone: "+7 (495) 123-45-67",
    address: {
      street: "ул. Тверская, 1",
      city: "Москва",
      country: "Россия",
      zipCode: "125000"
    }
  },
  employees: [employee] // используем объект из предыдущего примера
};
```

### Использование пересечений типов

```typescript
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type SoftDelete = {
  deletedAt?: Date;
};

type VersionedEntity = {
  version: number;
};

// Комбинирование нескольких типов
type AuditableEntity = Timestamped & SoftDelete & VersionedEntity & {
  id: number;
  name: string;
};

const entity: AuditableEntity = {
  id: 1,
  name: "Тестовая сущность",
  createdAt: new Date(),
  updatedAt: new Date(),
  version: 1
};
```

## Связь с другими концепциями

- [[interfaces-classes/interfaces|Интерфейсы]] - подробное рассмотрение интерфейсов
- [[types/type-aliases-comprehensive|Псевдонимы типов]] - подробное рассмотрение псевдонимов типов
- [[types/union-intersection-types-comprehensive|Объединения и пересечения типов]] - комбинирование типов
- [[../type-system/type-compatibility|Совместимость типов]] - как объектные типы сравниваются

## Заключение

Объектные типы являются мощным инструментом для создания строго типизированных структур данных в TypeScript. Они обеспечивают безопасность типов, автодополнение в редакторах кода и лучшую документацию для разработчиков.