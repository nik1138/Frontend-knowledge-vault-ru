---
aliases: ["Продвинутые ограничения дженериков", "TS Advanced Generic Constraints"]
tags: [typescript, generics, constraints, advanced]
---

# Продвинутые ограничения дженериков

## Введение

Ограничения дженериков в TypeScript позволяют создавать более гибкие и безопасные типы. Продвинутые ограничения позволяют налагать сложные условия на типы-параметры, что особенно полезно при создании библиотек и сложных типов.

## Базовые ограничения дженериков

### Простые ограничения с `extends`

```typescript
// Простое ограничение - T должен быть строкой или числом
function identity<T extends string | number>(arg: T): T {
  return arg;
}

// Пример использования
const str = identity('hello'); // string
const num = identity(42); // number
// const bool = identity(true); // Ошибка: boolean не соответствует ограничению

// Ограничение объектами с определенными свойствами
interface Lengthwise {
  length: number;
}

function logIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // Теперь мы можем безопасно получить доступ к свойству length
  return arg;
}

// Работает с типами, имеющими свойство length
logIdentity('hello'); // string
logIdentity([1, 2, 3]); // number[]
logIdentity({ length: 10, value: 'test' }); // объект с length
```

### Ограничения с ключами объекта

```typescript
// Ограничение ключей объекта
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key]; // Тип возвращаемого значения - T[K]
}

const user = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

const userName = getProperty(user, 'name'); // string
const userId = getProperty(user, 'id'); // number
// const userAge = getProperty(user, 'age'); // Ошибка: 'age' не существует в типе User
```

## Продвинутые ограничения

### Ограничения с условными типами

```typescript
// Ограничение с условными типами
type NonNullableProperty<T, K extends keyof T> = T[K] extends null | undefined ? never : K;

function safeGet<T, K extends keyof T>(
  obj: T, 
  key: NonNullableProperty<T, K>
): NonNullable<T[K]> {
  const value = obj[key];
  if (value == null) {
    throw new Error(`Property ${String(key)} is null or undefined`);
  }
  return value as NonNullable<T[K]>;
}

// Пример использования
interface UserWithOptional {
  id: number;
  name: string;
  email?: string;
  avatar?: string | null;
}

const userWithOptional: UserWithOptional = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

// Можно безопасно получить name (обязательное поле)
const name = safeGet(userWithOptional, 'name'); // string

// Но нельзя безопасно получить email, если он может быть undefined
// const email = safeGet(userWithOptional, 'email'); // Ошибка
```

### Ограничения с кортежами и массивами

```typescript
// Ограничение для кортежей
type Head<T extends readonly [any, ...any[]]> = T[0];
type Tail<T extends readonly [any, ...any[]]> = T extends [any, ...infer U] ? U : [];

// Пример использования
type FirstElement = Head<['a', 'b', 'c']>; // 'a'
type RemainingElements = Tail<['a', 'b', 'c']>; // ['b', 'c']

// Ограничение для массивов с определенными элементами
function firstElement<T extends [any, ...any[]]>(arr: T): Head<T> {
  return arr[0];
}

const result = firstElement(['hello', 42, true]); // string
```

### Ограничения с функциями

```typescript
// Ограничение для функций с определенной сигнатурой
type FunctionWithArgs<T extends (...args: any[]) => any> = T;

function callWithDefaults<T extends (...args: any[]) => any>(
  fn: T,
  ...defaults: Parameters<T>
): ReturnType<T> {
  return fn(...defaults);
}

// Пример использования
const add = (a: number, b: number, c: number = 0): number => a + b + c;

const result1 = callWithDefaults(add, 1, 2, 3); // 6
const result2 = callWithDefaults(add, 5, 10); // 15
```

## Практические примеры

### Ограничения для валидации форм

```typescript
// Ограничения для типов полей формы
interface FormField<T> {
  value: T;
  validators: ((value: T) => boolean)[];
  error?: string;
}

// Ограничение для объектов формы
type FormValues = Record<string, any>;

function createForm<T extends FormValues>(initialValues: T) {
  const fields = Object.keys(initialValues).reduce((acc, key) => {
    acc[key as keyof T] = {
      value: initialValues[key],
      validators: [],
      error: undefined
    } as FormField<T[keyof T]>;
    return acc;
  }, {} as { [K in keyof T]: FormField<T[K]> });
  
  return {
    fields,
    validate: (): boolean => {
      let isValid = true;
      Object.entries(fields).forEach(([key, field]) => {
        const fieldValid = field.validators.every(validator => validator(field.value));
        if (!fieldValid) {
          field.error = 'Validation failed';
          isValid = false;
        } else {
          field.error = undefined;
        }
      });
      return isValid;
    }
  };
}

// Пример использования
interface LoginForm {
  email: string;
  password: string;
}

const loginForm = createForm<LoginForm>({
  email: '',
  password: ''
});

// Добавляем валидаторы
loginForm.fields.email.validators.push(
  (value: string) => value.includes('@')
);
loginForm.fields.password.validators.push(
  (value: string) => value.length >= 8
);
```

### Ограничения для API клиентов

```typescript
// Ограничения для типов запросов и ответов
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// Ограничение для типов, которые могут быть сериализованы в JSON
type JsonSerializable = 
  | string 
  | number 
  | boolean 
  | null 
  | { [key: string]: JsonSerializable } 
  | JsonSerializable[];

function makeApiCall<T extends JsonSerializable>(
  url: string,
  method: 'GET' | 'POST' | 'PUT' | 'DELETE',
  body?: T
): Promise<ApiResponse<T>> {
  return fetch(url, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: body ? JSON.stringify(body) : undefined
  }).then(response => response.json());
}

// Пример использования
interface User {
  id: number;
  name: string;
  email: string;
}

const user: User = { id: 1, name: 'John', email: 'john@example.com' };

// Работает, так как User - это JsonSerializable
makeApiCall<User>('/api/users', 'POST', user);
```

### Ограничения для конфигурации

```typescript
// Ограничения для конфигурационных объектов
interface ConfigSchema {
  [key: string]: {
    type: 'string' | 'number' | 'boolean' | 'object';
    required?: boolean;
    default?: any;
    validator?: (value: any) => boolean;
  };
}

function createConfig<T extends ConfigSchema>(schema: T) {
  type ConfigType = {
    [K in keyof T]: T[K]['type'] extends 'string' ? string :
                    T[K]['type'] extends 'number' ? number :
                    T[K]['type'] extends 'boolean' ? boolean :
                    T[K]['type'] extends 'object' ? object : any;
  };
  
  return function(config: Partial<ConfigType>): ConfigType {
    const result = {} as ConfigType;
    
    for (const key in schema) {
      if (schema.hasOwnProperty(key)) {
        const configValue = (config as any)[key];
        const schemaItem = schema[key];
        
        if (configValue !== undefined) {
          // Проверяем валидность значения
          if (schemaItem.validator && !schemaItem.validator(configValue)) {
            throw new Error(`Invalid value for config property: ${key}`);
          }
          (result as any)[key] = configValue;
        } else if (schemaItem.default !== undefined) {
          (result as any)[key] = schemaItem.default;
        } else if (schemaItem.required) {
          throw new Error(`Required config property missing: ${key}`);
        }
      }
    }
    
    return result;
  };
}

// Пример использования
const appConfigSchema = {
  apiUrl: { type: 'string', required: true },
  timeout: { type: 'number', default: 5000 },
  debug: { type: 'boolean', default: false },
  features: { type: 'object', default: {} }
} as const;

const createAppConfig = createConfig(appConfigSchema);

const config = createAppConfig({
  apiUrl: 'https://api.example.com',
  debug: true
});

// Тип config: {
//   apiUrl: string;
//   timeout: number;
//   debug: boolean;
//   features: object;
// }
```

## Сложные примеры

### Ограничения для типов с вычисляемыми свойствами

```typescript
// Ограничения для объектов с вычисляемыми свойствами
type ComputeProperty<T, K extends string, R> = T & { [P in K]: R };

function addComputedProperty<
  T extends Record<string, any>, 
  K extends string, 
  R
>(
  obj: T,
  key: K,
  compute: (obj: T) => R
): ComputeProperty<T, K, R> {
  return {
    ...obj,
    [key]: compute(obj)
  } as ComputeProperty<T, K, R>;
}

// Пример использования
interface Product {
  id: number;
  name: string;
  price: number;
  currency: string;
}

const product: Product = {
  id: 1,
  name: 'Laptop',
  price: 999.99,
  currency: 'USD'
};

const productWithTotal = addComputedProperty(
  product,
  'totalPrice',
  (p) => p.price * 1.1 // добавляем 10% налога
);

// Тип productWithTotal: Product & { totalPrice: number }
```

### Ограничения для типов с зависимыми свойствами

```typescript
// Ограничения для объектов с зависимыми свойствами
interface ConditionalField<T> {
  type: T;
  value: T extends 'string' ? string : 
         T extends 'number' ? number : 
         T extends 'boolean' ? boolean : any;
}

function createConditionalField<T extends 'string' | 'number' | 'boolean'>(
  type: T,
  value: T extends 'string' ? string : 
         T extends 'number' ? number : 
         T extends 'boolean' ? boolean : any
): ConditionalField<T> {
  return { type, value } as ConditionalField<T>;
}

// Примеры использования
const stringField = createConditionalField('string', 'hello'); // { type: 'string'; value: string; }
const numberField = createConditionalField('number', 42); // { type: 'number'; value: number; }
const booleanField = createConditionalField('boolean', true); // { type: 'boolean'; value: boolean; }
```

### Ограничения для типов с динамическими ключами

```typescript
// Ограничения для объектов с динамическими ключами
type DynamicKeys<T extends string, V> = {
  [K in T]: V;
};

function createDynamicObject<T extends string, V>(
  keys: T[],
  valueFactory: (key: T) => V
): DynamicKeys<T, V> {
  const obj = {} as DynamicKeys<T, V>;
  
  for (const key of keys) {
    obj[key] = valueFactory(key);
  }
  
  return obj;
}

// Пример использования
const permissions = createDynamicObject(
  ['read', 'write', 'delete'] as const,
  (key) => key === 'read' // все права по умолчанию false, кроме read
);

// Тип permissions: { read: boolean; write: boolean; delete: boolean; }
```

## Заключение

Продвинутые ограничения дженериков позволяют создавать более гибкие и безопасные типы в TypeScript. Они особенно полезны при создании библиотек, фреймворков и сложных типов, где необходимо налагать сложные условия на типы-параметры.

См. также: [[type-guards-advanced]] | [[discriminated-unions]] | [[conditional-types-tricks]]