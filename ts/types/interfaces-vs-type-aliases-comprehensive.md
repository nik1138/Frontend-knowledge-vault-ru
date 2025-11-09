# Интерфейсы и псевдонимы типов: Различия и лучшие практики

Интерфейсы (interfaces) и псевдонимы типов (type aliases) - это два способа в TypeScript для определения форм объектов и других типов. Хотя они часто могут использоваться взаимозаменяемо, у них есть важные различия, которые влияют на выбор в зависимости от конкретной ситуации.

## Основные синтаксисы

### Интерфейс

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
};
```

### Псевдоним типа

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
};
```

## Основные различия

### 1. Расширение (Declaration Merging)

**Интерфейсы поддерживают объявление с объединением (declaration merging):**

```typescript
interface User {
  id: number;
  name: string;
}

interface User {
  email: string;
}

// Результат: объединение обоих интерфейсов
const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
};
```

**Псевдонимы типов этого не поддерживают:**

```typescript
type User = {
  id: number;
  name: string;
};

// Следующая строка вызовет ошибку:
// type User = { email: string; }; // Error: Duplicate identifier 'User'
```

### 2. Возможности определения

**Интерфейсы:**
- Могут расширять другие интерфейсы
- Могут реализовываться классами
- Ограничены объектными типами

```typescript
interface Person {
  name: string;
}

interface Employee extends Person {
  employeeId: number;
}

class StaffMember implements Employee {
  name: string;
  employeeId: number;
  
  constructor(name: string, employeeId: number) {
    this.name = name;
    this.employeeId = employeeId;
  }
}
```

**Псевдонимы типов:**
- Могут представлять любые типы (не только объектные)
- Могут использовать объединения, кортежи, примитивы и т.д.

```typescript
type Name = string;
type NameOrId = string | number;
type Coordinates = [number, number];
type UserOrAdmin = User & { isAdmin: boolean };
type StringDictionary = { [key: string]: string };
```

## Подробное сравнение возможностей

### 1. Наследование/Расширение

**Интерфейсы:**

```typescript
interface Animal {
  name: string;
}

interface Mammal extends Animal {
  warmBlooded: true;
}

interface Dog extends Mammal {
  breed: string;
}

const dog: Dog = {
  name: "Buddy",
  warmBlooded: true,
  breed: "Golden Retriever"
};
```

**Псевдонимы типов:**

```typescript
type Animal = {
  name: string;
};

type Mammal = Animal & {
  warmBlooded: true;
};

type Dog = Mammal & {
  breed: string;
};

const dog: Dog = {
  name: "Buddy",
  warmBlooded: true,
  breed: "Golden Retriever"
};
```

### 2. Реализация классами

**Только интерфейсы могут быть реализованы классами:**

```typescript
interface Flyable {
  altitude: number;
  fly(): void;
}

class Bird implements Flyable {
  altitude: number = 0;
  
  fly(): void {
    this.altitude += 100;
  }
}

// Псевдоним типа не может быть реализован:
// type FlyableType = { altitude: number; fly(): void; };
// class Plane implements FlyableType { ... } // Ошибка
```

### 3. Объявление с объединением (Declaration Merging)

```typescript
// Множественные объявления интерфейса объединяются
interface Config {
  apiUrl: string;
}

interface Config {
  timeout: number;
}

// Результат: Config с обоими полями
const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};

// Псевдонимы типов не поддерживают это:
// type Settings = { theme: string; };
// type Settings = { language: string; }; // Ошибка: дублирующий идентификатор
```

## Практические применения

### Когда использовать интерфейсы

1. **Для определения форм объектов, которые будут расширяться:**

```typescript
// Хороший кандидат для интерфейса
interface APIResponse {
  status: number;
  message: string;
}

// Другой файл/модуль может добавить дополнительные поля
interface APIResponse {
  timestamp: Date; // Добавлено в другом месте
}
```

2. **Когда классы будут реализовывать этот тип:**

```typescript
interface Drawable {
  x: number;
  y: number;
  draw(): void;
}

class Circle implements Drawable {
  x: number;
  y: number;
  
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
  
  draw(): void {
    console.log(`Drawing circle at (${this.x}, ${this.y})`);
  }
}
```

3. **Для публичных API, где может потребоваться расширение:**

```typescript
// Библиотечный интерфейс
export interface ComponentConfig {
  id: string;
  visible: boolean;
}

// Пользователь может расширить его в своем коде
interface ComponentConfig {
  theme: 'light' | 'dark';
}
```

### Когда использовать псевдонимы типов

1. **Для объединений типов:**

```typescript
type Status = "success" | "error" | "loading";
type ID = string | number;
type Callback = () => void;
```

2. **Для примитивных типов с дополнительной семантикой:**

```typescript
type Email = string;
type Password = string;
type HexColor = string;
```

3. **Для кортежей:**

```typescript
type Coordinates = [number, number];
type KeyValuePair<K, V> = [K, V];
```

4. **Для сложных комбинаций типов:**

```typescript
type RequestHandler<T> = (req: T) => Promise<{ data: any; status: number }>;
type FilterFunction<T> = (item: T) => boolean;
```

## Продвинутые паттерны

### 1. Гибридное использование

```typescript
// Использовать интерфейс для основной структуры
interface UserBase {
  id: number;
  name: string;
}

// И псевдонимы для производных типов
type AdminUser = UserBase & { permissions: string[] };
type GuestUser = UserBase & { visitCount: number };
type User = AdminUser | GuestUser;
```

### 2. Условные типы с псевдонимами

```typescript
// Псевдонимы типов лучше подходят для сложных условных типов
type ElementType<T> = T extends (infer U)[] ? U : T;

type ArrayElement = ElementType<string[]>;  // string
type NonArrayElement = ElementType<number>; // number
```

### 3. Сопоставленные типы

```typescript
// Псевдонимы часто используются для сопоставленных типов
type Partial<T> = { [P in keyof T]?: T[P] };
type ReadOnly<T> = { readonly [P in keyof T]: T[P] };
type Nullable<T> = { [P in keyof T]: T[P] | null };
```

## Практические примеры из реальной разработки

### 1. Система событий

```typescript
// Использование интерфейса для основного типа
interface EventHandler {
  (data: any): void;
}

// И псевдонима для конкретных обработчиков
type UserEventHandler = (data: { userId: number; action: string }) => void;
type ErrorEventHandler = (data: { message: string; code: number }) => void;

// Или более сложный пример с объединением
interface BaseEvent {
  timestamp: Date;
}

interface UserEvent extends BaseEvent {
  type: "user_action";
  userId: number;
  action: string;
}

interface SystemEvent extends BaseEvent {
  type: "system";
  code: number;
  message: string;
}

type AppEvent = UserEvent | SystemEvent;

function handleEvent(event: AppEvent) {
  switch (event.type) {
    case "user_action":
      console.log(`User ${event.userId} performed ${event.action}`);
      break;
    case "system":
      console.log(`System event: ${event.code} - ${event.message}`);
      break;
  }
}
```

### 2. Работа с API и конфигурацией

```typescript
// Использование интерфейсов для основных структур
interface ApiConfig {
  baseUrl: string;
  timeout: number;
}

// Псевдонимы для вариантов конфигурации
type DevelopmentConfig = ApiConfig & { debug: true; logLevel: "verbose" };
type ProductionConfig = ApiConfig & { debug: false; logLevel: "error" };

// Объединение для полной системы типов
type AppConfig = DevelopmentConfig | ProductionConfig;

// Использование declaration merging для расширения конфигурации
interface ApiConfig {
  retries: number;
}

// Теперь ApiConfig включает baseUrl, timeout и retries
```

### 3. Создание типобезопасных утилит

```typescript
// Использование псевдонимов для сложных утилит
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
type Extract<T, U> = T extends U ? T : never;
type Diff<T, U> = T extends U ? never : T;

// И интерфейсов для основных форм
interface Entity {
  id: string;
}

interface Timestamped extends Entity {
  createdAt: Date;
  updatedAt: Date;
}

type Versioned<T> = T & { version: number };
type SoftDelete<T> = T & { deletedAt?: Date };
```

## Ограничения и особенности

### 1. Производительность компиляции

```typescript
// Интерфейсы могут быть немного быстрее в больших проектах
// из-за более простой обработки компилятором
interface LargeObject {
  // много полей...
}

// Псевдонимы с объединениями и условными типами могут быть медленнее
type ComplexType = SomeCondition extends true ? TypeA : TypeB;
```

### 2. Отладка типов

```typescript
// Интерфейсы обычно показывают более читаемые сообщения об ошибках
interface User {
  name: string;
}

// При ошибке TypeScript покажет "User", а не развернутую структуру

// Псевдонимы могут показывать развернутую структуру
type UserAlias = { name: string; };
// При ошибке может показать всю структуру
```

## Лучшие практики

1. **По умолчанию используйте интерфейсы** - для определения форм объектов
2. **Используйте псевдонимы для объединений и других типов** - когда нужно больше гибкости
3. **Используйте интерфейсы для публичных API** - когда расширяемость важна
4. **Предпочитайте интерфейсы для форм, которые могут быть реализованы классами**
5. **Используйте псевдонимы для вспомогательных типов и утилит**
6. **Будьте последовательны в проекте** - выберите один подход и придерживайтесь его

## Рекомендации от Microsoft

Официальная рекомендация TypeScript:

> "Если вы не уверены, начните с `interface`. Если вам нужны возможности, которые предоставляет `type`, используйте `type`."

Это связано с тем, что:
- Интерфейсы более ограничены, но проще понимаются
- Интерфейсы поддерживают declaration merging
- Интерфейсы лучше работают с объектно-ориентированным кодом

## Связи с другими концепциями

- [[ts/types/type-aliases|Псевдонимы типов]] - подробное рассмотрение псевдонимов
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - использование интерфейсов с классами
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - часто используют псевдонимы
- [[ts/advanced-types/conditional-types|Условные типы]] - реализуются через псевдонимы
- [[ts/type-system/type-compatibility|Совместимость типов]] - как интерфейсы и псевдонимы влияют на совместимость