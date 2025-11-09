# Дженерики с ключами и значениями объектов

Дженерики с ключами и значениями объектов позволяют создавать типобезопасные функции и типы, которые работают с ключами и значениями объектов на уровне типов.

## Работа с keyof

### 1. Базовое использование keyof

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

// Тип всех ключей интерфейса
type UserKeys = keyof User; // "name" | "age" | "email"

// Дженерик для получения ключей
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

const userName = getProperty(user, "name"); // ✅ string
const userAge = getProperty(user, "age");   // ✅ number
// const invalid = getProperty(user, "invalid"); // ❌ Ошибка компиляции
```

### 2. Дженерик для установки значений

```typescript
function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
  obj[key] = value;
}

const user: User = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

setProperty(user, "name", "Jane"); // ✅
setProperty(user, "age", 25);      // ✅
// setProperty(user, "name", 123);    // ❌ Ошибка: тип не совпадает
```

## Работа с типами значений

### 1. Получение типа значений

```typescript
type ValueOf<T> = T[keyof T];

interface User {
  name: string;
  age: number;
  active: boolean;
}

type UserValues = ValueOf<User>; // string | number | boolean

// Дженерик для получения значений
function getValues<T>(obj: T): ValueOf<T>[] {
  return Object.values(obj) as ValueOf<T>[];
}

const user: User = {
  name: "John",
  age: 30,
  active: true
};

const values = getValues(user); // (string | number | boolean)[]
```

### 2. Дженерик для фильтрации по типу значений

```typescript
type KeysOfType<T, U> = { [K in keyof T]: T[K] extends U ? K : never }[keyof T];

interface User {
  name: string;
  age: number;
  email: string;
  active: boolean;
  score: number;
}

type StringKeys = KeysOfType<User, string>; // "name" | "email"
type NumberKeys = KeysOfType<User, number>; // "age" | "score"
type BooleanKeys = KeysOfType<User, boolean>; // "active"

// Дженерик для получения подмножества свойств по типу
function pickByType<T, U>(obj: T, type: U): Pick<T, KeysOfType<T, U>> {
  const result = {} as Pick<T, KeysOfType<T, U>>;
  
  for (const key in obj) {
    if (typeof (obj as any)[key] === typeof type) {
      (result as any)[key] = (obj as any)[key];
    }
  }
  
  return result;
}

const user: User = {
  name: "John",
  age: 30,
  email: "john@example.com",
  active: true,
  score: 95
};

const stringProps = pickByType(user, ""); 
// { name: string; email: string }

const numberProps = pickByType(user, 0); 
// { age: number; score: number }
```

## Продвинутые примеры

### 1. Дженерик для создания маппинга ключей

```typescript
type MapKeys<T, M extends Record<keyof T, string>> = {
  [K in keyof T as M[K]]: T[K];
};

interface User {
  firstName: string;
  lastName: string;
  userAge: number;
}

type UserMapping = {
  firstName: "first_name";
  lastName: "last_name";
  userAge: "age";
};

type MappedUser = MapKeys<User, UserMapping>;
// {
//   first_name: string;
//   last_name: string;
//   age: number;
// }

// Дженерик для переименования ключей
function renameKeys<T, M extends Record<keyof T, string>>(
  obj: T,
  mapping: M
): MapKeys<T, M> {
  const result = {} as MapKeys<T, M>;
  
  for (const key in obj) {
    const newKey = mapping[key as keyof T] as keyof MapKeys<T, M>;
    result[newKey] = obj[key];
  }
  
  return result;
}

const user: User = {
  firstName: "John",
  lastName: "Doe",
  userAge: 30
};

const mapping: UserMapping = {
  firstName: "first_name",
  lastName: "last_name",
  userAge: "age"
};

const renamedUser = renameKeys(user, mapping);
// {
//   first_name: "John",
//   last_name: "Doe",
//   age: 30
// }
```

### 2. Дженерик для создания обратного маппинга

```typescript
type ReverseMapping<T> = {
  [K in keyof T as T[K] extends PropertyKey ? T[K] : never]: K;
};

interface ApiMapping {
  userId: "id";
  userName: "name";
  userAge: "age";
}

type ReversedMapping = ReverseMapping<ApiMapping>;
// {
//   id: "userId";
//   name: "userName";
//   age: "userAge";
// }

// Дженерик для создания объекта с обратным маппингом
function createReverseMapping<T extends Record<keyof T, PropertyKey>>(
  mapping: T
): ReverseMapping<T> {
  const reversed = {} as ReverseMapping<T>;
  
  for (const key in mapping) {
    const value = mapping[key];
    (reversed as any)[value] = key;
  }
  
  return reversed;
}
```

### 3. Дженерик для работы с вложенными объектами

```typescript
type NestedKeyOf<T> = T extends object 
  ? { [K in keyof T]: K extends string 
      ? T[K] extends object 
        ? `${K}` | `${K}.${NestedKeyOf<T[K]>}` 
        : `${K}`
      : never 
    }[keyof T]
  : never;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
    coordinates: {
      lat: number;
      lng: number;
    };
  };
}

type UserNestedKeys = NestedKeyOf<User>;
// "name" | "address" | "address.street" | "address.city" | 
// "address.coordinates" | "address.coordinates.lat" | "address.coordinates.lng"

// Дженерик для получения значения по вложенному ключу
function getNestedProperty<T, K extends NestedKeyOf<T>>(
  obj: T,
  path: K
): any {
  return path.split('.').reduce((current, key) => 
    current?.[key as keyof typeof current], obj as any
  );
}

const user: User = {
  name: "John",
  address: {
    street: "123 Main St",
    city: "New York",
    coordinates: {
      lat: 40.7128,
      lng: -74.0060
    }
  }
};

const cityName = getNestedProperty(user, "address.city"); // ✅ string
const latitude = getNestedProperty(user, "address.coordinates.lat"); // ✅ number
```

## Практические примеры

### 1. Дженерик для создания конфигурационного объекта

```typescript
type ConfigSchema<T> = {
  [K in keyof T]: {
    type: "string" | "number" | "boolean";
    required: boolean;
    defaultValue?: T[K];
  };
};

interface AppConfig {
  apiUrl: string;
  timeout: number;
  debug: boolean;
  retries: number;
}

type AppConfigSchema = ConfigSchema<AppConfig>;

const configSchema: AppConfigSchema = {
  apiUrl: { type: "string", required: true },
  timeout: { type: "number", required: false, defaultValue: 5000 },
  debug: { type: "boolean", required: false, defaultValue: false },
  retries: { type: "number", required: false, defaultValue: 3 }
};

// Дженерик для валидации конфигурации
function validateConfig<T>(
  config: Record<string, any>,
  schema: ConfigSchema<T>
): config is T {
  for (const key in schema) {
    const field = schema[key];
    const value = config[key];
    
    if (field.required && (value === undefined || value === null)) {
      return false;
    }
    
    if (value !== undefined) {
      const valueType = typeof value;
      if (valueType !== field.type) {
        return false;
      }
    }
  }
  
  return true;
}
```

### 2. Дженерик для создания API клиента

```typescript
interface ApiEndpoints {
  getUser: { params: { id: string }; response: { id: string; name: string } };
  createUser: { params: { name: string }; response: { id: string; name: string } };
  deleteUser: { params: { id: string }; response: void };
}

type ApiClient<T extends Record<string, { params: any; response: any }>> = {
  [K in keyof T]: (params: T[K]["params"]) => Promise<T[K]["response"]>;
};

// Дженерик для создания типобезопасного API клиента
function createApiClient<T extends Record<string, { params: any; response: any }>>(
  endpoints: T
): ApiClient<T> {
  const client = {} as ApiClient<T>;
  
  for (const key in endpoints) {
    client[key] = async (params: any) => {
      // Здесь была бы реализация вызова API
      console.log(`Calling ${key as string} with params:`, params);
      return {} as any; // Имитация ответа
    };
  }
  
  return client;
}

const apiClient = createApiClient({
  getUser: { params: { id: "1" }, response: { id: "1", name: "John" } },
  createUser: { params: { name: "Jane" }, response: { id: "2", name: "Jane" } },
  deleteUser: { params: { id: "1" }, response: undefined }
});

// Типобезопасное использование
apiClient.getUser({ id: "1" }).then(user => {
  // TypeScript знает, что user имеет тип { id: string; name: string }
  console.log(user.name);
});
```

### 3. Дженерик для создания ORM-подобного API

```typescript
interface ModelSchema {
  [modelName: string]: {
    [fieldName: string]: "string" | "number" | "boolean" | "date";
  };
}

type ModelInstance<T extends ModelSchema, M extends keyof T> = {
  [K in keyof T[M]]: T[M][K] extends "string" ? string :
                    T[M][K] extends "number" ? number :
                    T[M][K] extends "boolean" ? boolean :
                    T[M][K] extends "date" ? Date :
                    never;
};

interface AppSchema extends ModelSchema {
  User: {
    id: "string";
    name: "string";
    age: "number";
    active: "boolean";
    createdAt: "date";
  };
  Product: {
    id: "string";
    title: "string";
    price: "number";
    inStock: "boolean";
  };
}

// Дженерик для создания модели
class Model<T extends ModelSchema> {
  constructor(private schema: T) {}
  
  create<M extends keyof T>(
    modelName: M,
    data: Partial<ModelInstance<T, M>>
  ): ModelInstance<T, M> {
    // Здесь была бы реализация создания записи
    return data as ModelInstance<T, M>;
  }
  
  find<M extends keyof T>(
    modelName: M,
    id: string
  ): ModelInstance<T, M> | null {
    // Здесь была бы реализация поиска записи
    return null;
  }
}

const appSchema: AppSchema = {
  User: {
    id: "string",
    name: "string",
    age: "number",
    active: "boolean",
    createdAt: "date"
  },
  Product: {
    id: "string",
    title: "string",
    price: "number",
    inStock: "boolean"
  }
};

const model = new Model(appSchema);

// Типобезопасное использование
const user = model.create("User", {
  name: "John",
  age: 30,
  active: true,
  createdAt: new Date()
});

// TypeScript знает типы всех полей
console.log(user.name);    // ✅ string
console.log(user.age);     // ✅ number
console.log(user.active);  // ✅ boolean
console.log(user.createdAt); // ✅ Date
```

## Ограничения и особенности

1. Сложные keyof дженерики могут замедлять компиляцию
2. При работе с большими объектами могут потребоваться дополнительные аннотации
3. Вложенные keyof дженерики могут быть трудны для понимания
4. Некоторые паттерны могут не работать с индексными сигнатурами

## Связанные концепции

- [[ts/type-system/keyof-operator|Оператор keyof]]
- [[ts/utility-types/built-in-property-selection|Утилиты для выбора свойств]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/type-system/indexed-access-types|Индексные типы]]