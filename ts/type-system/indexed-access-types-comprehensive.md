# Индексные типы доступа в TypeScript

Индексные типы доступа (indexed access types), также известные как операторы поиска (lookup types), позволяют получать тип значения по ключу из другого типа. Они используют синтаксис `T[K]`, где `T` - тип объекта, а `K` - тип ключа. Этот механизм тесно связан с оператором `keyof` и позволяет создавать мощные и выразительные типы.

## Основной синтаксис

Индексные типы используют синтаксис `SomeType[Key]`, аналогично доступу к свойствам объекта во время выполнения:

```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonName = Person["name"];    // string
type PersonAge = Person["age"];      // number
type PersonEmail = Person["email"];  // string

// Использование объединения ключей
type PersonInfo = Person["name" | "email"];  // string (объединение типов значений)
```

## Простые примеры

### 1. Базовое использование

```typescript
const person: Person = {
  name: "Alice",
  age: 30,
  email: "alice@example.com"
};

// Тип значения, возвращаемого функцией getProperty
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const name = getProperty(person, "name");  // Тип: string
const age = getProperty(person, "age");    // Тип: number
```

### 2. Использование с объединениями ключей

```typescript
type NameOrEmail = Person["name" | "email"];  // string
type AgeOrEmail = Person["age" | "email"];    // number | string
```

## Продвинутые паттерны

### 1. Использование с keyof

```typescript
// Получение типа значения для любого ключа типа
type ValueOf<T> = T[keyof T];

type PersonValue = ValueOf<Person>;  // string | number

// Это эквивалентно: Person["name"] | Person["age"] | Person["email"]
```

### 2. Работа с вложенными структурами

```typescript
interface User {
  id: number;
  profile: {
    name: string;
    contact: {
      email: string;
      phone: string;
    };
  };
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
  };
}

// Доступ к вложенным свойствам
type ProfileType = User["profile"];                    // { name: string; contact: { email: string; phone: string; }; }
type ContactType = User["profile"]["contact"];         // { email: string; phone: string; }
type EmailType = User["profile"]["contact"]["email"];  // string
type ThemeType = User["preferences"]["theme"];         // "light" | "dark"
```

### 3. Использование с массивами и кортежами

```typescript
type StringArray = string[];
type ArrayElementType = StringArray[number];  // string

type NumberArray = number[];
type MixedArray = (string | number)[];
type MixedElementType = MixedArray[number];  // string | number

// С кортежами
type Tuple = [string, number, boolean];
type FirstElement = Tuple[0];   // string
type SecondElement = Tuple[1];  // number
type ThirdElement = Tuple[2];   // boolean
type AnyElement = Tuple[number]; // string | number | boolean (псевдоним для union всех элементов)
```

## Практические применения

### 1. Создание типобезопасных функций доступа

```typescript
// Функция для получения вложенного свойства по пути
type NestedValue<T, K> = K extends keyof T
  ? T[K]
  : K extends `${infer First}.${infer Rest}`
    ? First extends keyof T
      ? Rest extends keyof T[First]
        ? T[First][Rest]
        : NestedValue<T[First], Rest>
      : never
    : never;

interface Config {
  server: {
    host: string;
    port: number;
  };
  database: {
    url: string;
    options: {
      timeout: number;
      retries: number;
    };
  };
}

// Типизированное получение значений конфигурации
type ServerHost = NestedValue<Config, "server.host">;        // string
type DbTimeout = NestedValue<Config, "database.options.timeout">; // number
```

### 2. Работа с API ответами

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Извлечение типа данных из ответа API
type GetData<T> = T extends ApiResponse<infer U> ? U : never;

interface User {
  id: number;
  name: string;
  email: string;
}

type UserApiResponse = ApiResponse<User>;
type ExtractedUser = GetData<UserApiResponse>;  // User

// Или более простой способ получить тип поля data
type ResponseDataType = UserApiResponse["data"];  // User
```

### 3. Создание типов для обновления объектов

```typescript
// Тип для обновления отдельных полей объекта
type UpdateType<T> = {
  [K in keyof T]?: T[K]
};

type PartialUser = UpdateType<User>;
// { id?: number; profile?: { name: string; contact: { email: string; phone: string; }; }; preferences?: { theme: "light" | "dark"; notifications: boolean; }; }

// Или обновление конкретного вложенного поля
type UpdateContact = {
  profile: {
    contact: Partial<User["profile"]["contact"]>
  }
};
// { profile: { contact: { email?: string; phone?: string; }; }; }
```

## Комбинация с другими операторами

### 1. С keyof для получения всех значений

```typescript
// Получение объединения всех возможных значений свойств объекта
type AllValues<T> = T[keyof T];

interface Options {
  mode: "development" | "production";
  timeout: number;
  enabled: boolean;
}

type OptionValues = AllValues<Options>;  // "development" | "production" | number | boolean
```

### 2. С условными типами

```typescript
// Извлечение только строковых свойств
type StringValueProperties<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T];

type StringProps = StringValueProperties<User>;
// "profile" | "preferences" (ключи свойств, значения которых могут быть строками)

// Точнее - только ключи со строковыми значениями
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T];

// На самом деле это вернет все ключи, где значения могут быть строками
// Для точной фильтрации нужно более сложное условие
type ExactStringKeys<T> = {
  [K in keyof T]: string extends T[K] ? never : T[K] extends string ? K : never
}[keyof T];
```

### 3. С сопоставленными типами

```typescript
// Создание типа с измененными свойствами
type WithOptional<T> = {
  [K in keyof T]?: T[K]
};

type OptionalUser = WithOptional<User>;
// Все свойства User становятся опциональными
```

## Практические примеры из реальной разработки

### 1. Работа с настройками и конфигурацией

```typescript
interface AppConfig {
  api: {
    baseUrl: string;
    timeout: number;
    retries: number;
  };
  ui: {
    theme: "light" | "dark";
    language: string;
  };
  features: {
    notifications: boolean;
    analytics: boolean;
  };
}

// Тип для обновления конкретной секции конфигурации
type ConfigSection = keyof AppConfig;  // "api" | "ui" | "features"

// Тип для обновления полей в секции
type SectionUpdate<T extends ConfigSection> = Partial<AppConfig[T]>;

type ApiUpdate = SectionUpdate<"api">;
// { baseUrl?: string; timeout?: number; retries?: number; }

type UiUpdate = SectionUpdate<"ui">;
// { theme?: "light" | "dark"; language?: string; }
```

### 2. Создание типобезопасных мапперов

```typescript
interface DatabaseRecord {
  id: number;
  createdAt: Date;
  updatedAt: Date;
}

interface User extends DatabaseRecord {
  name: string;
  email: string;
}

interface Product extends DatabaseRecord {
  title: string;
  price: number;
}

// Тип для преобразования одного типа в другой
type Mapper<Input, Output> = {
  [K in keyof Output]: K extends keyof Input ? Input[K] : Output[K]
};

// Пример использования для маппинга данных из БД в DTO
interface UserDto {
  id: number;
  name: string;
  email: string;
  fullName: string;  // вычисляемое поле
}

type UserMapper = Mapper<User, UserDto>;
// {
//   id: number;
//   name: string;
//   email: string;
//   fullName: string;  // должен быть предоставлен отдельно
// }
```

### 3. Работа с событиями и коллбэками

```typescript
interface EventPayloads {
  user_created: { userId: number; name: string };
  user_updated: { userId: number; updates: Partial<User> };
  user_deleted: { userId: number; deletedAt: Date };
}

// Тип для обработчика события
type EventHandler<T extends keyof EventPayloads> = (payload: EventPayloads[T]) => void;

// Регистрация обработчиков
const handlers: {
  [K in keyof EventPayloads]: EventHandler<K>[];
} = {
  user_created: [],
  user_updated: [],
  user_deleted: []
};

// Типобезопасная функция для добавления обработчиков
function subscribe<T extends keyof EventPayloads>(
  event: T,
  handler: EventHandler<T>
) {
  handlers[event].push(handler);
}

// Использование
subscribe("user_created", (payload) => {
  // payload имеет тип { userId: number; name: string; }
  console.log(`Создан пользователь ${payload.name} с ID ${payload.userId}`);
});
```

## Ограничения и особенности

### 1. Ограничения на сложность

Слишком сложные индексные типы могут привести к ошибкам компиляции:

```typescript
// Сложные рекурсивные структуры могут вызвать ошибки
// type TooComplex = SomeComplexType[SomeComplexKey];
```

### 2. Работа с индексными сигнатурами

```typescript
interface Dictionary {
  [key: string]: number;
  special: string;  // конкретное свойство
}

type DictValue = Dictionary[string];  // number | undefined (из-за индексной сигнатуры)
type SpecialValue = Dictionary["special"];  // string
```

### 3. Доступ к несуществующим свойствам

```typescript
// Попытка доступа к несуществующему свойству вызывает ошибку
// type Invalid = Person["invalid"];  // Ошибка компиляции
```

## Лучшие практики

1. **Используйте для создания типобезопасных функций доступа** - предотвращает опечатки
2. **Комбинируйте с keyof для максимальной безопасности** - вместе они обеспечивают полную типобезопасность
3. **Документируйте сложные индексные типы** - особенно когда они используются в утилитах
4. **Тестируйте с разными структурами данных** - убедитесь, что типы работают корректно
5. **Избегайте излишней сложности** - сложные индексные типы могут затруднить понимание кода
6. **Используйте для создания переиспользуемых утилит** - многие утилиты типов основаны на индексных типах

## Связи с другими концепциями

- [[ts/type-system/keyof-operator|Оператор keyof]] - тесно связан с индексными типами
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - часто используют индексные типы
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использовать индексные типы с infer
- [[ts/generics/TS Generics|Обобщения]] - индексные типы часто параметризованы
- [[Утилиты типов TypeScript|Утилиты типов]] - многие утилиты используют индексные типы