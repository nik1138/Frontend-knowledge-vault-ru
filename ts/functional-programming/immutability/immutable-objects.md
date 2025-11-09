# Неизменяемые объекты

Неизменяемые объекты - это объекты, состояние которых не может быть изменено после создания. Вместо модификации существующих объектов создаются новые версии с обновленными значениями.

## Создание неизменяемых объектов

### Использование readonly модификатора

```typescript
// Интерфейс с readonly свойствами
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly preferences: {
    readonly theme: 'light' | 'dark';
    readonly notifications: boolean;
  };
}

// Создание неизменяемого объекта
const user: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    notifications: true
  }
};

// Попытка изменения вызовет ошибку компиляции
// user.name = 'Jane Doe'; // Ошибка: Cannot assign to 'name' because it is a read-only property

// Вместо изменения создаем новый объект
const updatedUser: User = {
  ...user,
  name: 'Jane Doe',
  preferences: {
    ...user.preferences,
    theme: 'dark'
  }
};
```

### Глубокая неизменяемость

```typescript
// DeepReadonly тип для глубокой неизменяемости
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface ComplexObject {
  user: User;
  settings: {
    theme: string;
    permissions: string[];
  };
}

type ImmutableComplexObject = DeepReadonly<ComplexObject>;

// Пример использования
const complexObject: ImmutableComplexObject = {
  user: {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    preferences: {
      theme: 'light',
      notifications: true
    }
  },
  settings: {
    theme: 'dark',
    permissions: ['read', 'write']
  }
};

// Все свойства доступны только для чтения
// complexObject.user.name = 'Jane'; // Ошибка компиляции
// complexObject.settings.permissions.push('admin'); // Ошибка компиляции
```

### Использование Object.freeze

```typescript
// Создание полностью неизменяемого объекта
const createUser = (id: number, name: string, email: string) => 
  Object.freeze({
    id,
    name,
    email,
    createdAt: new Date()
  });

const user = createUser(1, 'John Doe', 'john@example.com');

// Попытка изменения будет проигнорирована в строгом режиме
// или вызовет ошибку в нестрогом режиме
// user.name = 'Jane Doe'; // Игнорируется в строгом режиме

// Создание нового объекта вместо изменения
const updateUser = (user: typeof user, newName: string) => 
  Object.freeze({
    ...user,
    name: newName
  });
```

## Работа с неизменяемыми объектами

### Обновление свойств

```typescript
// Функция для обновления свойств объекта
const updateObject = <T extends Record<string, any>>(
  obj: T, 
  updates: Partial<T>
): T => ({
  ...obj,
  ...updates
});

// Пример использования
interface Product {
  readonly id: number;
  readonly name: string;
  readonly price: number;
  readonly category: string;
}

const product: Product = {
  id: 1,
  name: 'Laptop',
  price: 1000,
  category: 'Electronics'
};

const updatedProduct = updateObject(product, {
  price: 1200,
  category: 'Computers'
});
```

### Удаление свойств

```typescript
// Функция для удаления свойств из объекта
const omit = <T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> => {
  const result = { ...obj };
  keys.forEach(key => delete result[key]);
  return result;
};

// Функция для выбора определенных свойств
const pick = <T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> => {
  const result = {} as Pick<T, K>;
  keys.forEach(key => result[key] = obj[key]);
  return result;
};

// Пример использования
interface UserWithExtra {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly password: string;
  readonly createdAt: Date;
}

const userWithExtra: UserWithExtra = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  password: 'secret123',
  createdAt: new Date()
};

// Удаление чувствительных данных
const publicUser = omit(userWithExtra, ['password']);
// Тип: Pick<UserWithExtra, "id" | "name" | "email" | "createdAt">

// Выбор только необходимых свойств
const userSummary = pick(userWithExtra, ['id', 'name']);
// Тип: Pick<UserWithExtra, "id" | "name">
```

### Работа с вложенными объектами

```typescript
// Функция для обновления вложенных свойств
const updateNestedProperty = <T, K1 extends keyof T, K2 extends keyof T[K1]>(
  obj: T,
  key1: K1,
  key2: K2,
  value: T[K1][K2]
): T => ({
  ...obj,
  [key1]: {
    ...(obj[key1] as object),
    [key2]: value
  } as T[K1]
});

// Пример использования
interface UserProfile {
  readonly id: number;
  readonly personalInfo: {
    readonly name: string;
    readonly age: number;
  };
  readonly preferences: {
    readonly theme: 'light' | 'dark';
    readonly language: string;
  };
}

const userProfile: UserProfile = {
  id: 1,
  personalInfo: {
    name: 'John Doe',
    age: 30
  },
  preferences: {
    theme: 'light',
    language: 'en'
  }
};

const updatedProfile = updateNestedProperty(
  userProfile,
  'preferences',
  'theme',
  'dark'
);
```

## Паттерны работы с неизменяемыми объектами

### Lens паттерн

```typescript
// Lens паттерн для работы с вложенными структурами
interface Lens<S, A> {
  get: (s: S) => A;
  set: (a: A, s: S) => S;
}

// Создание линз
const lens = <S, A>(get: (s: S) => A, set: (a: A, s: S) => S): Lens<S, A> => ({
  get,
  set
});

// Пример использования
interface Address {
  readonly street: string;
  readonly city: string;
  readonly zipCode: string;
}

interface Person {
  readonly name: string;
  readonly age: number;
  readonly address: Address;
}

const addressLens: Lens<Person, Address> = lens(
  (person) => person.address,
  (address, person) => ({ ...person, address })
);

const streetLens: Lens<Address, string> = lens(
  (address) => address.street,
  (street, address) => ({ ...address, street })
);

// Композиция линз
const composeLens = <A, B, C>(lens1: Lens<A, B>, lens2: Lens<B, C>): Lens<A, C> => ({
  get: (a) => lens2.get(lens1.get(a)),
  set: (c, a) => lens1.set(lens2.set(c, lens1.get(a)), a)
});

const personStreetLens = composeLens(addressLens, streetLens);

// Использование
const person: Person = {
  name: 'John Doe',
  age: 30,
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001'
  }
};

const street = personStreetLens.get(person); // '123 Main St'
const updatedPerson = personStreetLens.set('456 Oak Ave', person);
```

### Builder паттерн

```typescript
// Функциональный Builder паттерн
class UserBuilder {
  private user: Partial<User> = {};
  
  static create(): UserBuilder {
    return new UserBuilder();
  }
  
  withId(id: number): this {
    this.user.id = id;
    return this;
  }
  
  withName(name: string): this {
    this.user.name = name;
    return this;
  }
  
  withEmail(email: string): this {
    this.user.email = email;
    return this;
  }
  
  build(): User {
    if (!this.user.id || !this.user.name || !this.user.email) {
      throw new Error('Missing required fields');
    }
    
    return Object.freeze({
      id: this.user.id,
      name: this.user.name,
      email: this.user.email,
      preferences: {
        theme: 'light',
        notifications: true
      }
    });
  }
}

// Использование
const user = UserBuilder.create()
  .withId(1)
  .withName('John Doe')
  .withEmail('john@example.com')
  .build();
```

### Record паттерн

```typescript
// Работа с коллекциями неизменяемых объектов
type UserRecord = Record<number, User>;

// Функции для работы с коллекциями
const addUser = (users: UserRecord, user: User): UserRecord => ({
  ...users,
  [user.id]: user
});

const updateUserInRecord = (users: UserRecord, user: User): UserRecord => ({
  ...users,
  [user.id]: user
});

const removeUser = (users: UserRecord, userId: number): UserRecord => {
  const newUsers = { ...users };
  delete newUsers[userId];
  return newUsers;
};

const getUser = (users: UserRecord, userId: number): User | undefined => 
  users[userId];

// Пример использования
let users: UserRecord = {};

users = addUser(users, {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    notifications: true
  }
});

users = addUser(users, {
  id: 2,
  name: 'Jane Smith',
  email: 'jane@example.com',
  preferences: {
    theme: 'dark',
    notifications: false
  }
});

const updatedUser = {
  ...users[1]!,
  name: 'John Smith'
};

users = updateUserInRecord(users, updatedUser);
```

## Преимущества неизменяемых объектов

### 1. Предсказуемость

```typescript
// Предсказуемое поведение
const processUser = (user: User): string => {
  // Функция не может изменить входной объект
  return `${user.name} (${user.email})`;
};

const user: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    notifications: true
  }
};

const result = processUser(user);
// user остается неизменным
console.log(user.name); // 'John Doe'
```

### 2. Потокобезопасность

```typescript
// Безопасное использование в многопоточных средах
const sharedUsers: UserRecord = {
  1: {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    preferences: {
      theme: 'light',
      notifications: true
    }
  }
};

// Функции могут безопасно читать данные без блокировок
const getUserInfo = (users: UserRecord, userId: number): string => {
  const user = users[userId];
  return user ? `${user.name} (${user.email})` : 'User not found';
};

// Обновление создает новую версию, не влияя на существующие ссылки
const newUsers = addUser(sharedUsers, {
  id: 2,
  name: 'Jane Smith',
  email: 'jane@example.com',
  preferences: {
    theme: 'dark',
    notifications: false
  }
});
```

### 3. Упрощенное тестирование

```typescript
// Легко тестируемые функции
const calculateUserStats = (users: UserRecord): { total: number; active: number } => {
  const total = Object.keys(users).length;
  const active = Object.values(users).filter(user => 
    user.preferences.notifications
  ).length;
  
  return { total, active };
};

// Тестирование
const testUsers: UserRecord = {
  1: {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    preferences: {
      theme: 'light',
      notifications: true
    }
  },
  2: {
    id: 2,
    name: 'Jane Smith',
    email: 'jane@example.com',
    preferences: {
      theme: 'dark',
      notifications: false
    }
  }
};

const stats = calculateUserStats(testUsers);
console.assert(stats.total === 2);
console.assert(stats.active === 1);
```

## Распространенные ошибки

### 1. Поверхностная копия вместо глубокой

```typescript
// Ошибка: поверхностная копия
const updateUserPreferencesBad = (user: User, theme: 'light' | 'dark'): User => ({
  ...user,
  preferences: {
    ...user.preferences,
    theme // Это правильно
  }
});

// Но если preferences содержит вложенные объекты, это может быть проблемой
interface UserWithNestedPreferences {
  readonly id: number;
  readonly name: string;
  readonly preferences: {
    readonly theme: 'light' | 'dark';
    readonly notifications: {
      readonly email: boolean;
      readonly push: boolean;
    };
  };
}

// Ошибка: неполная копия вложенных объектов
const updateUserNotificationsBad = (
  user: UserWithNestedPreferences, 
  email: boolean
): UserWithNestedPreferences => ({
  ...user,
  preferences: {
    ...user.preferences,
    notifications: {
      email, // Забыли push
      push: user.preferences.notifications.push
    }
  }
});

// Правильно: полная копия
const updateUserNotificationsGood = (
  user: UserWithNestedPreferences, 
  email: boolean
): UserWithNestedPreferences => ({
  ...user,
  preferences: {
    ...user.preferences,
    notifications: {
      ...user.preferences.notifications,
      email
    }
  }
});
```

### 2. Попытка изменения неизменяемых объектов

```typescript
// Ошибка: попытка изменения неизменяемого объекта
const processUserBad = (user: User): void => {
  // user.name = 'New Name'; // Ошибка компиляции
  // Но в runtime это может быть проигнорировано
};

// Правильно: создание нового объекта
const processUserGood = (user: User): User => ({
  ...user,
  name: 'New Name'
});
```

### 3. Неправильная работа с массивами внутри объектов

```typescript
// Ошибка: изменение массива внутри неизменяемого объекта
interface UserWithTags {
  readonly id: number;
  readonly name: string;
  readonly tags: readonly string[];
}

const addUserTagBad = (user: UserWithTags, tag: string): UserWithTags => {
  // user.tags.push(tag); // Ошибка: push изменяет массив
  return user;
};

// Правильно: создание нового массива
const addUserTagGood = (user: UserWithTags, tag: string): UserWithTags => ({
  ...user,
  tags: [...user.tags, tag]
});
```

## Лучшие практики

### 1. Использование readonly модификаторов

```typescript
// Хорошо: явное указание неизменяемости
interface ImmutableUser {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly preferences: {
    readonly theme: 'light' | 'dark';
    readonly notifications: boolean;
  };
}

// Плохо: отсутствие readonly модификаторов
interface MutableUser {
  id: number;
  name: string;
  email: string;
  preferences: {
    theme: 'light' | 'dark';
    notifications: boolean;
  };
}
```

### 2. Использование Object.freeze для дополнительной защиты

```typescript
// Дополнительная защита с Object.freeze
const createImmutableUser = (
  id: number, 
  name: string, 
  email: string
): ImmutableUser => 
  Object.freeze({
    id,
    name,
    email,
    preferences: Object.freeze({
      theme: 'light',
      notifications: true
    })
  });
```

### 3. Создание утилит для работы с неизменяемыми структурами

```typescript
// Утилиты для работы с неизменяемыми объектами
const ImmutableUtils = {
  // Глубокое обновление
  deepUpdate<T>(obj: T, path: string[], value: any): T {
    if (path.length === 0) return value;
    
    const [head, ...tail] = path;
    return {
      ...obj,
      [head]: tail.length === 0 
        ? value 
        : this.deepUpdate((obj as any)[head], tail, value)
    } as T;
  },
  
  // Глубокое слияние
  deepMerge<T>(obj1: T, obj2: Partial<T>): T {
    const result = { ...obj1 } as any;
    
    for (const key in obj2) {
      if (obj2.hasOwnProperty(key)) {
        if (
          obj2[key] !== null && 
          typeof obj2[key] === 'object' && 
          !Array.isArray(obj2[key]) &&
          result[key] !== null &&
          typeof result[key] === 'object'
        ) {
          result[key] = this.deepMerge(result[key], obj2[key] as any);
        } else {
          result[key] = obj2[key];
        }
      }
    }
    
    return result as T;
  }
};

// Пример использования
const userToUpdate: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    notifications: true
  }
};

const updatedUserWithUtils = ImmutableUtils.deepUpdate(
  userToUpdate,
  ['preferences', 'theme'],
  'dark'
);
```

## Связи с другими концепциями

- [[ts/functional-programming/immutability/Неизменяемость|Неизменяемость]] - Основная страница раздела
- [[ts/functional-programming/immutability/immutable-arrays|Неизменяемые массивы]] - Работа с неизменяемыми массивами
- [[ts/functional-programming/immutability/immutable-classes|Неизменяемые классы]] - Создание неизменяемых классов
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функциями
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции

## Следующие шаги

- [[ts/functional-programming/immutability/immutable-arrays|Неизменяемые массивы]] - Работа с неизменяемыми массивами
- [[ts/functional-programming/immutability/immutable-classes|Неизменяемые классы]] - Создание неизменяемых классов
- [[ts/functional-programming/immutability/patterns|Паттерны неизменяемости]] - Распространенные паттерны работы с неизменяемыми структурами
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции