# Утилиты типов TypeScript

Утилиты типов в TypeScript - это встроенные типы-помощники, которые позволяют производить трансформации типов без необходимости писать их с нуля. Эти утилиты особенно полезны при работе с композаблами и архитектурой больших приложений.

## Введение

Утилиты типов позволяют создавать более гибкие, переиспользуемые и типобезопасные интерфейсы. Они особенно полезны при создании библиотек, API-интерфейсов и композаблов в Vue.js приложениях.

## Основные утилиты типов

### 1. Partial<T>

Преобразует все свойства типа в необязательные:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type PartialUser = Partial<User>;
// Равнозначно:
// {
//   id?: number;
//   name?: string;
//   email?: string;
//   age?: number;
// }

// Полезно для обновления объектов
function updateUser(id: number, updates: Partial<User>) {
  // Обновляем только те поля, что переданы
}
```

### 2. Required<T>

Делает все свойства типа обязательными:

```typescript
interface User {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<User>;
// Равнозначно:
// {
//   id: number;
//   name: string;
//   email: string;
// }

// Полезно для валидации данных
function createValidUser(userData: Required<User>): User {
  if (!userData.name || !userData.email) {
    throw new Error('Пользователь должен иметь имя и email');
  }
  return userData;
}
```

### 3. Readonly<T>

Делает все свойства типа доступными только для чтения:

```typescript
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
// Поля объекта нельзя будет изменить

const user: ReadonlyUser = { id: 1, name: 'John' };
// user.name = 'Jane'; // Ошибка: не может быть изменено
```

### 4. Pick<T, K>

Создает тип, выбирая только указанные свойства K из типа T:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Используем только нужные поля для формы
type UserForm = Pick<User, 'name' | 'email'>;
// Равнозначно: { name: string; email: string; }

// Полезно для DTO
type UserDto = Pick<User, 'id' | 'name' | 'email'>;
```

### 5. Omit<T, K>

Создает тип, исключая указанные свойства K из типа T:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Исключаем чувствительные данные
type PublicUser = Omit<User, 'password'>;
// Равнозначно: { id: number; name: string; email: string; createdAt: Date; }

// Особенно полезно при обновлении данных
type UpdateUserDto = Omit<User, 'id' | 'createdAt'>;
```

### 6. Record<K, T>

Определяет объект, где ключи принадлежат типу K, а значения - типу T:

```typescript
// Создаем словарь пользователей по ID
type UsersMap = Record<number, User>;
const users: UsersMap = {
  1: { id: 1, name: 'John', email: 'john@example.com' },
  2: { id: 2, name: 'Jane', email: 'jane@example.com' }
};

// Создаем маппинг статусов
type Status = 'active' | 'inactive' | 'pending';
type StatusCount = Record<Status, number>;
```

### 7. Exclude<T, U>

Исключает из T все типы, которые являются подтипами U:

```typescript
type AvailableColors = 'red' | 'green' | 'blue' | 'yellow';
type PrimaryColors = 'red' | 'green' | 'blue';

type SecondaryColors = Exclude<AvailableColors, PrimaryColors>;
// Равнозначно: 'yellow'

// Полезно при работе с событиями:
type AllEvents = 'click' | 'hover' | 'focus' | 'blur';
type UserEvents = 'click' | 'hover';

type SystemEvents = Exclude<AllEvents, UserEvents>; // 'focus' | 'blur'
```

### 8. Extract<T, U>

Извлекает из T все типы, которые являются подтипами U:

```typescript
type Response = string | number | boolean | { error: string } | { data: any };
type ErrorResponses = Extract<Response, { error: string }>;
// Равнозначно: { error: string }

// Проверка результата ответа API
function handleResponse(response: Response) {
  if (isError(response)) {
    console.error(response.error);
  } else {
    console.log(response);
  }
}

function isError<T extends Response>(response: T): response is Extract<T, { error: string }> {
  return typeof response === 'object' && 'error' in response;
}
```

### 9. NonNullable<T>

Исключает из типа null и undefined:

```typescript
type PossibleValues = string | number | null | undefined;
type ValidValues = NonNullable<PossibleValues>;
// Равнозначно: string | number
```

### 10. Parameters<T>

Извлекает типы параметров функции:

```typescript
function createUser(name: string, email: string, age: number) {
  return { name, email, age };
}

type CreateUserParams = Parameters<typeof createUser>;
// Равнозначно: [string, string, number]

// Полезно для создания wrapper-функций
function loggedCreateUser(...args: CreateUserParams) {
  console.log('Создание пользователя с параметрами:', args);
  return createUser(...args);
}
```

### 11. ReturnType<T>

Извлекает возвращаемый тип функции:

```typescript
function getUser(id: number) {
  return { id, name: 'John', email: 'john@example.com' };
}

type UserResult = ReturnType<typeof getUser>;
// Равнозначно: { id: number; name: string; email: string; }

// Полезно при создании функций оберток
function withErrorHandling<T extends (...args: any[]) => any>(fn: T) {
  return (...args: Parameters<T>): ReturnType<T> | { error: string } => {
    try {
      return fn(...args);
    } catch (error) {
      return { error: (error as Error).message };
    }
  };
}
```

## Расширенные утилиты типов

### 1. Awaited<T>

Рекурсивно разворачивает тип Promise:

```typescript
type Example1 = Awaited<Promise<string>>;
// Равнозначно: string

type Example2 = Awaited<Promise<Promise<number>>>;
// Равнозначно: number

type Example3 = Awaited<boolean | Promise<number>>;
// Равнозначно: boolean | number

// Полезно при работе с асинхронными композаблами
async function fetchUserData(userId: number) {
  return { id: userId, name: 'John' };
}

type UserData = Awaited<ReturnType<typeof fetchUserData>>;
```

## Создание пользовательских утилит типов

### 1. DeepPartial<T>

Рекурсивно делает все свойства необязательными:

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface User {
  id: number;
  name: string;
  address: {
    street: string;
    city: string;
    country: string;
  };
}

type DeepPartialUser = DeepPartial<User>;
// Все поля, включая вложенные, становятся опциональными
```

### 2. Merge<T, U>

Объединяет два типа, где свойства из U переопределяют свойства из T:

```typescript
type Merge<T, U> = Omit<T, keyof U> & U;

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserUpdates {
  name: string;
  age: number;
}

type UpdatedUser = Merge<User, UserUpdates>;
// { id: number; email: string; name: string; age: number; }
```

### 3. DeepReadonly<T>

Рекурсивно делает все свойства и вложенные свойства доступными только для чтения:

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface User {
  id: number;
  name: string;
  address: {
    street: string;
    city: string;
  };
}

type ReadonlyUser = DeepReadonly<User>;
// Все свойства, включая вложенные, доступны только для чтения
```

## Практические примеры использования

### 1. В композаблах Vue

```typescript
// composables/useForm.ts
import { reactive, computed } from 'vue';

type ValidationResult = {
  isValid: boolean;
  errors: Record<string, string>;
};

// Используем утилиты типов для создания гибкого композабла формы
export function useForm<T extends Record<string, any>>(
  initialData: T,
  validationRules?: Partial<Record<keyof T, (value: any) => string | undefined>>
) {
  const state = reactive({
    data: { ...initialData } as T,
    errors: {} as Record<string, string>,
    isValid: true
  });

  const validate = (): ValidationResult => {
    const errors: Record<string, string> = {};
    
    if (validationRules) {
      for (const [field, rule] of Object.entries(validationRules)) {
        const value = state.data[field as keyof T];
        const error = rule?.(value);
        
        if (error) {
          errors[field] = error;
        }
      }
    }
    
    const isValid = Object.keys(errors).length === 0;
    state.errors = errors;
    state.isValid = isValid;
    
    return { isValid, errors };
  };

  const updateField = <K extends keyof T>(field: K, value: T[K]) => {
    state.data[field] = value;
    // Очищаем ошибку при изменении поля
    if (state.errors[field as string]) {
      delete state.errors[field as string];
      state.isValid = Object.keys(state.errors).length === 0;
    }
  };

  return {
    state: computed(() => ({ ...state })),
    validate,
    updateField,
    reset: () => {
      Object.assign(state.data, initialData);
      state.errors = {};
      state.isValid = true;
    }
  };
}
```

### 2. В API-сервисах

```typescript
// api/types.ts
interface ApiResult<T> {
  data: T;
  status: number;
  message: string;
}

// Используем Pick, чтобы создать тип для обновления данных
type UserUpdatePayload = Partial<Pick<User, 'name' | 'email' | 'age'>>;

// Тип для результата апи-запроса
type UpdateUserResult = ApiResult<User>;

async function updateUser(id: number, data: UserUpdatePayload): Promise<UpdateUserResult> {
  const response = await fetch(`/api/users/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  
  return response.json();
}
```

## Лучшие практики

### 1. Используйте утилиты типов для создания более гибких API

```typescript
// Вместо создания множества похожих типов
type CreateUserDto = { name: string; email: string; password: string; };
type UpdateUserDto = { name?: string; email?: string; password?: string; };

// Используйте утилиты типов
type CreateUserDto = Required<Pick<User, 'name' | 'email' | 'password'>>;
type UpdateUserDto = Partial<Omit<User, 'id'>>;
```

### 2. Создавайте собственные утилиты для часто используемых паттернов

```typescript
// shared/types.ts

// Удаляет указанные ключи из типа
type Without<T, K> = Pick<T, Exclude<keyof T, K>>;

// Делает указанные ключи обязательными
type RequiredKeys<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

// Делает указанные ключи необязательными
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
```

## Заключение

Утилиты типов - мощный инструмент для создания гибких и типобезопасных интерфейсов в TypeScript. Они особенно полезны при разработке композаблов, API-сервисов и библиотек, где необходимо обеспечить максимальную типобезопасность при минимальной дубликации кода.

Изучение и правильное использование этих утилит позволяет значительно улучшить качество кода, обеспечивая более строгую проверку типов и лучшую автодополнение в IDE.

## Связанные темы

- [[Расширенные типы TypeScript|Расширенные типы TypeScript]]
- [[../generics/index|Обобщения в TypeScript]]
- [[../../vue/composables/best-practices|Лучшие практики Vue 3 Composition API]]
- [[../type-system/type-inference|Вывод типов в TypeScript]]

## Теги

#typescript #utility-types #type-system #frontend #vuejs #composition-api