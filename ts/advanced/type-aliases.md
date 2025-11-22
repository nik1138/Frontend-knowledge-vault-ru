# Псевдонимы типов в TypeScript

## Введение

Псевдонимы типов (type aliases) - это способ создания нового имени для существующего типа. Они позволяют давать описательные имена типам, упрощая понимание кода и делая его более читаемым. Псевдонимы могут представлять как простые типы, так и сложные структуры, включая объединения, пересечения и другие составные типы.

## Основы псевдонимов типов

### Создание простых псевдонимов

```typescript
// Псевдоним для примитивного типа
type ID = string | number;

// Псевдоним для объединения типов
type Status = "active" | "inactive" | "pending";

// Псевдоним для сложного типа
type Callback = (error: Error | null, result?: any) => void;
```

### Псевдонимы объектных типов

```typescript
// Псевдоним для объектного типа
type User = {
  id: ID;
  name: string;
  email: string;
  status: Status;
  createdAt: Date;
};

// Использование псевдонима
const user: User = {
  id: 123,
  name: "Алексей",
  email: "alex@example.com",
  status: "active",
  createdAt: new Date()
};
```

## Псевдонимы с обобщениями

Псевдонимы типов могут быть обобщенными, принимая параметры типа:

```typescript
// Обобщенный псевдоним для контейнера данных
type Container<T> = {
  value: T;
  isValid: boolean;
  timestamp: Date;
};

// Использование обобщенного псевдонима
const stringContainer: Container<string> = {
  value: "Тестовое значение",
  isValid: true,
  timestamp: new Date()
};

const numberContainer: Container<number> = {
  value: 42,
  isValid: false,
  timestamp: new Date()
};
```

### Ограничения обобщенных псевдонимов

```typescript
// Псевдоним с ограничением типа
type FilteredArray<T extends { id: number }> = T[];

// Это будет работать
type ValidItem = { id: number; name: string };
type ItemList = FilteredArray<ValidItem>;

// Это вызовет ошибку, потому что MyObject не имеет свойства id
// type MyObject = { name: string };
// type MyList = FilteredArray<MyObject>; // ОШИБКА
```

## Продвинутые паттерны использования

### Псевдонимы для функций

```typescript
// Псевдоним для функционального типа
type Predicate<T> = (value: T) => boolean;

// Использование псевдонима
const isPositive: Predicate<number> = (num) => num > 0;
const isValidEmail: Predicate<string> = (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

// Псевдоним для функции с несколькими параметрами
type Comparator<T> = (a: T, b: T) => number;

const stringComparator: Comparator<string> = (a, b) => a.localeCompare(b);
const numberComparator: Comparator<number> = (a, b) => a - b;
```

### Псевдонимы для объединений и пересечений

```typescript
// Псевдоним для сложного объединения
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string; errorCode: number };

// Псевдоним для пересечения типов
type Timestamped<T> = T & { createdAt: Date; updatedAt: Date };

interface Product {
  id: number;
  name: string;
  price: number;
}

type TimestampedProduct = Timestamped<Product>;
// Результат: {
//   id: number;
//   name: string;
//   price: number;
//   createdAt: Date;
//   updatedAt: Date;
// }
```

## Практические примеры

### Использование в API клиентах

```typescript
// Определение типов для API ответов
type ApiSuccess<T> = {
  status: 'success';
  data: T;
  message?: string;
};

type ApiError = {
  status: 'error';
  message: string;
  code: number;
};

type ApiResponse<T> = ApiSuccess<T> | ApiError;

// Использование в функции обработки API
function handleApiResponse<T>(response: ApiResponse<T>): void {
  if (response.status === 'success') {
    console.log('Данные:', response.data);
  } else {
    console.error(`Ошибка ${response.code}: ${response.message}`);
  }
}
```

### Псевдонимы для конфигурационных объектов

```typescript
// Определение различных частей конфигурации
type ServerConfig = {
  host: string;
  port: number;
};

type DatabaseConfig = {
  url: string;
  name: string;
};

type AuthConfig = {
  secret: string;
  expiresIn: string;
};

// Комбинирование в один псевдоним
type AppConfig = ServerConfig & DatabaseConfig & AuthConfig;

const config: AppConfig = {
  host: 'localhost',
  port: 3000,
  url: 'mongodb://localhost:27017',
  name: 'myapp',
  secret: 'my-secret-key',
  expiresIn: '1h'
};
```

### Псевдонимы для обработчиков событий

```typescript
// Псевдонимы для различных типов обработчиков
type EventHandler<T = void> = (data?: T) => void;
type AsyncEventHandler<T = void> = (data?: T) => Promise<void>;

interface UserEvent {
  user: User;
}

type UserEventHandler = EventHandler<UserEvent>;
type AsyncUserEventHandler = AsyncEventHandler<UserEvent>;

// Использование
const handleUserLogin: UserEventHandler = (event) => {
  console.log(`Пользователь ${event?.user.name} вошел в систему`);
};

const handleAsyncUserAction: AsyncUserEventHandler = async (event) => {
  await saveUserAction(event?.user);
  console.log('Действие пользователя сохранено');
};
```

## Отличия от интерфейсов

Хотя псевдонимы типов и интерфейсы могут использоваться для определения объектных типов, у них есть важные различия:

```typescript
// Псевдоним типа
type PersonAlias = {
  name: string;
};

// Интерфейс
interface PersonInterface {
  name: string;
}

// Интерфейсы могут быть расширены
interface PersonInterface {
  age: number; // OK - дополнение интерфейса
}

// Псевдонимы не могут быть повторно объявлены
// type PersonAlias = {
//   age: number; // ОШИБКА - нельзя повторно объявить
// }

// Но псевдонимы могут быть объединены
type ExtendedPerson = PersonAlias & { age: number };
```

## Рекурсивные псевдонимы

Псевдонимы могут быть рекурсивными, что полезно для представления древовидных структур:

```typescript
// Рекурсивный псевдоним для дерева
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

const tree: TreeNode<string> = {
  value: 'корень',
  children: [
    {
      value: 'левый потомок',
      children: []
    },
    {
      value: 'правый потомок',
      children: [
        {
          value: 'внук',
          children: []
        }
      ]
    }
  ]
};

// Рекурсивный псевдоним с объединением
type NestedContent = 
  | string 
  | number 
  | { [key: string]: NestedContent }
  | NestedContent[];
```

## Связь с другими концепциями

- [[types/union-intersection-types-comprehensive|Объединения и пересечения типов]] - псевдонимы часто используют эти концепции
- [[Продвинутые дженерики в TypeScript|Обобщения]] - псевдонимы могут быть обобщенными
- [[interfaces-classes/interfaces|Интерфейсы]] - альтернатива псевдонимам для объектных типов
- [[advanced-types/mapped-types|Сопоставляющие типы]] - могут использоваться вместе с псевдонимами

## Лучшие практики

1. **Используйте описательные имена** - псевдонимы должны ясно отражать назначение типа
2. **Предпочитайте интерфейсы для объектных типов** - если тип будет расширяться, используйте интерфейс
3. **Используйте псевдонимы для объединений и сложных типов** - особенно когда тип используется в нескольких местах
4. **Документируйте сложные псевдонимы** - особенно если они содержат обобщения или сложные структуры

## Заключение

Псевдонимы типов - мощный инструмент для создания читаемого и поддерживаемого кода в TypeScript. Они позволяют создавать понятные имена для сложных типов, упрощая понимание структуры данных и улучшая автодополнение в редакторах кода.