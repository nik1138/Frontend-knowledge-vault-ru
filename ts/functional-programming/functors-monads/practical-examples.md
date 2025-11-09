# Практические примеры функторов и монад

В этом разделе мы рассмотрим реальные примеры использования функторов и монад в TypeScript приложениях, включая обработку данных, работу с API, валидацию, управление состоянием и другие практические сценарии.

## Содержание

- [Практические примеры функторов и монад](#практические-примеры-функторов-и-монад)
  - [Содержание](#содержание)
  - [Обработка данных пользователей](#обработка-данных-пользователей)
  - [Работа с API и асинхронными операциями](#работа-с-api-и-асинхронными-операциями)
  - [Валидация форм и данных](#валидация-форм-и-данных)
  - [Управление состоянием приложения](#управление-состоянием-приложения)
  - [Обработка конфигурации](#обработка-конфигурации)
  - [Работа с файлами и потоками данных](#работа-с-файлами-и-потоками-данных)
  - [Функциональные компоненты React](#функциональные-компоненты-react)
  - [Обработка событий и команд](#обработка-событий-и-команд)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Обработка данных пользователей

Рассмотрим комплексный пример обработки данных пользователей с использованием различных монад.

```typescript
// Модели данных
interface RawUser {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  birthDate: string;
  salary: string;
  department: string;
  isActive: string; // "true" или "false"
}

interface ProcessedUser {
  id: number;
  fullName: string;
  email: string;
  age: number;
  salary: number;
  department: string;
  isActive: boolean;
  registrationDate: Date;
}

// Maybe монада для безопасной обработки потенциально отсутствующих значений
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
}

// Either монада для обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherMonad<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonad<E, A> {
    return new EitherMonad(value);
  }
  
  static right<E, A>(a: A): EitherMonad<E, A> {
    return new EitherMonad(right(a));
  }
  
  static left<E, A>(e: E): EitherMonad<E, A> {
    return new EitherMonad(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return new EitherMonad(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonad<E, B>): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return fn(this.value.right);
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
  
  fold<B>(onLeft: (e: E) => B, onRight: (a: A) => B): B {
    return this.value._tag === "Left" 
      ? onLeft(this.value.left) 
      : onRight(this.value.right);
  }
}

// Функции обработки данных пользователей
const parseId = (user: RawUser): EitherMonad<string, RawUser & { id: number }> => {
  const id = parseInt(user.id, 10);
  if (isNaN(id)) {
    return EitherMonad.left(`Неверный формат ID: ${user.id}`);
  }
  return EitherMonad.right({ ...user, id });
};

const formatName = (user: RawUser & { id: number }): RawUser & { id: number, fullName: string } => ({
  ...user,
  fullName: `${user.firstName.trim()} ${user.lastName.trim()}`
});

const calculateAge = (user: RawUser & { id: number, fullName: string }): EitherMonad<string, RawUser & { id: number, fullName: string, age: number }> => {
  const birthDate = new Date(user.birthDate);
  if (isNaN(birthDate.getTime())) {
    return EitherMonad.left(`Неверный формат даты рождения: ${user.birthDate}`);
  }
  
  const age = new Date().getFullYear() - birthDate.getFullYear();
  return EitherMonad.right({ ...user, age });
};

const parseSalary = (user: RawUser & { id: number, fullName: string, age: number }): EitherMonad<string, RawUser & { id: number, fullName: string, age: number, salary: number }> => {
  const salary = parseFloat(user.salary);
  if (isNaN(salary)) {
    return EitherMonad.left(`Неверный формат зарплаты: ${user.salary}`);
  }
  return EitherMonad.right({ ...user, salary });
};

const parseActiveStatus = (user: RawUser & { id: number, fullName: string, age: number, salary: number }): EitherMonad<string, RawUser & { id: number, fullName: string, age: number, salary: number, isActive: boolean }> => {
  if (user.isActive !== "true" && user.isActive !== "false") {
    return EitherMonad.left(`Неверный формат статуса активности: ${user.isActive}`);
  }
  return EitherMonad.right({ ...user, isActive: user.isActive === "true" });
};

const addRegistrationDate = (user: RawUser & { id: number, fullName: string, age: number, salary: number, isActive: boolean }): RawUser & { id: number, fullName: string, age: number, salary: number, isActive: boolean, registrationDate: Date } => ({
  ...user,
  registrationDate: new Date()
});

// Композиция обработки данных пользователя
const processUser = (rawUser: RawUser): EitherMonad<string, ProcessedUser> => {
  return EitherMonad.right(rawUser)
    .flatMap(parseId)
    .map(formatName)
    .flatMap(calculateAge)
    .flatMap(parseSalary)
    .flatMap(parseActiveStatus)
    .map(addRegistrationDate);
};

// Обработка массива пользователей
const processUsers = (rawUsers: RawUser[]): EitherMonad<string, ProcessedUser[]> => {
  const processedUsers: ProcessedUser[] = [];
  
  for (const rawUser of rawUsers) {
    const result = processUser(rawUser);
    if (result.isLeft()) {
      return result; // Возвращаем первую ошибку
    }
    processedUsers.push(result.getValue().right);
  }
  
  return EitherMonad.right(processedUsers);
};

// Альтернативная реализация с накоплением ошибок
const processUsersWithAllErrors = (rawUsers: RawUser[]): EitherMonad<string[], ProcessedUser[]> => {
  const processedUsers: ProcessedUser[] = [];
  const errors: string[] = [];
  
  for (const rawUser of rawUsers) {
    const result = processUser(rawUser);
    if (result.isLeft()) {
      errors.push(result.getValue().left);
    } else {
      processedUsers.push(result.getValue().right);
    }
  }
  
  if (errors.length > 0) {
    return EitherMonad.left(errors);
  }
  
  return EitherMonad.right(processedUsers);
};

// Пример использования
const rawUsers: RawUser[] = [
  {
    id: "1",
    firstName: "Иван  ",
    lastName: "  Иванов",
    email: "ivan@example.com",
    birthDate: "1990-05-15",
    salary: "50000.50",
    department: "IT",
    isActive: "true"
  },
  {
    id: "2",
    firstName: "Мария",
    lastName: "Петрова",
    email: "maria@example.com",
    birthDate: "1985-12-03",
    salary: "45000.00",
    department: "HR",
    isActive: "false"
  },
  {
    id: "invalid",
    firstName: "Алексей",
    lastName: "Сидоров",
    email: "alex@example.com",
    birthDate: "1992-03-20",
    salary: "invalid",
    department: "Finance",
    isActive: "true"
  }
];

// Обработка с остановкой на первой ошибке
const result1 = processUsers(rawUsers);
console.log("Результат с остановкой на первой ошибке:", result1.getValue());

// Обработка с накоплением всех ошибок
const result2 = processUsersWithAllErrors(rawUsers);
console.log("Результат с накоплением всех ошибок:", result2.getValue());
```

## Работа с API и асинхронными операциями

Рассмотрим пример работы с API с использованием Future монады для асинхронных операций.

```typescript
// Future монада для асинхронных операций
class FutureMonad<A> {
  constructor(private computation: () => Promise<A>) {}
  
  static of<A>(value: A): FutureMonad<A> {
    return new FutureMonad(() => Promise.resolve(value));
  }
  
  static fromPromise<A>(promise: Promise<A>): FutureMonad<A> {
    return new FutureMonad(() => promise);
  }
  
  map<B>(fn: (a: A) => B): FutureMonad<B> {
    return new FutureMonad(() => this.computation().then(fn));
  }
  
  flatMap<B>(fn: (a: A) => FutureMonad<B>): FutureMonad<B> {
    return new FutureMonad(() => 
      this.computation().then(a => fn(a).computation())
    );
  }
  
  runFuture(): Promise<A> {
    return this.computation();
  }
  
  tap(fn: (a: A) => void): FutureMonad<A> {
    return new FutureMonad(() => 
      this.computation().then(a => {
        fn(a);
        return a;
      })
    );
  }
  
  catchError<B>(fn: (error: any) => B): FutureMonad<A | B> {
    return new FutureMonad(() => 
      this.computation().catch(fn)
    );
  }
}

// Модели API
interface ApiConfig {
  baseUrl: string;
  defaultHeaders: Record<string, string>;
  timeout?: number;
}

interface ApiRequest {
  method: string;
  endpoint: string;
  headers?: Record<string, string>;
  body?: any;
  queryParams?: Record<string, string>;
}

interface ApiResponse<T> {
  status: number;
  data: T;
  headers: Record<string, string>;
}

// Either монада для обработки ошибок API
type ApiError = 
  | { type: "network", message: string }
  | { type: "http", status: number, message: string }
  | { type: "parse", message: string };

type ApiResult<T> = Either<ApiError, T>;

// Утилиты для работы с API
const createApiRequest = (config: ApiConfig) => (request: ApiRequest): FutureMonad<ApiResult<ApiResponse<any>>> => {
  return FutureMonad.fromPromise(
    new Promise<ApiResult<ApiResponse<any>>>(resolve => {
      // Симуляция API вызова
      setTimeout(() => {
        // Симуляция случайных ошибок
        if (Math.random() < 0.1) {
          resolve(left({ type: "network", message: "Ошибка сети" }));
          return;
        }
        
        if (Math.random() < 0.1) {
          resolve(left({ type: "http", status: 500, message: "Внутренняя ошибка сервера" }));
          return;
        }
        
        // Симуляция успешного ответа
        const response: ApiResponse<any> = {
          status: 200,
          data: { message: "Данные успешно получены" },
          headers: {}
        };
        
        resolve(right(response));
      }, 100);
    })
  );
};

// Создание специализированных API клиентов
const createGithubApiClient = createApiRequest({
  baseUrl: "https://api.github.com",
  defaultHeaders: {
    "Accept": "application/vnd.github.v3+json",
    "User-Agent": "MyApp/1.0"
  }
});

const createJsonPlaceholderApiClient = createApiRequest({
  baseUrl: "https://jsonplaceholder.typicode.com",
  defaultHeaders: {
    "Content-Type": "application/json"
  }
});

// Функции для работы с пользователями GitHub
interface GithubUser {
  id: number;
  login: string;
  name: string;
  email: string;
  public_repos: number;
}

const fetchGithubUser = (username: string): FutureMonad<ApiResult<GithubUser>> => {
  return createGithubApiClient({
    method: "GET",
    endpoint: `/users/${username}`
  }).map(result => {
    if (result._tag === "Left") {
      return result;
    }
    
    // Симуляция парсинга данных
    const userData: GithubUser = {
      id: Math.floor(Math.random() * 1000000),
      login: username,
      name: `${username} Name`,
      email: `${username}@example.com`,
      public_repos: Math.floor(Math.random() * 100)
    };
    
    return right(userData);
  });
};

const fetchUserRepos = (username: string): FutureMonad<ApiResult<any[]>> => {
  return createGithubApiClient({
    method: "GET",
    endpoint: `/users/${username}/repos`
  }).map(result => {
    if (result._tag === "Left") {
      return result;
    }
    
    // Симуляция данных репозиториев
    const repos = Array.from({ length: 5 }, (_, i) => ({
      id: i + 1,
      name: `repo-${i + 1}`,
      description: `Description for repo ${i + 1}`
    }));
    
    return right(repos);
  });
};

// Композиция API вызовов
const fetchUserWithRepos = (username: string): FutureMonad<ApiResult<{ user: GithubUser; repos: any[] }>> => {
  return fetchGithubUser(username)
    .flatMap(userResult => {
      if (userResult._tag === "Left") {
        return FutureMonad.of(userResult);
      }
      
      return fetchUserRepos(username)
        .map(reposResult => {
          if (reposResult._tag === "Left") {
            return reposResult;
          }
          
          return right({
            user: userResult.right,
            repos: reposResult.right
          });
        });
    });
};

// Использование
fetchUserWithRepos("octocat")
  .tap(result => {
    if (result._tag === "Right") {
      console.log("Пользователь с репозиториями:", result.right);
    } else {
      console.log("Ошибка:", result.left);
    }
  })
  .catchError(error => {
    console.log("Неожиданная ошибка:", error);
    return { error: "Неожиданная ошибка" };
  })
  .runFuture();
```

## Валидация форм и данных

Рассмотрим пример комплексной валидации форм с использованием Either монады.

```typescript
// Модели валидации
interface ValidationError {
  field: string;
  message: string;
}

type ValidationResult<T> = Either<ValidationError[], T>;

// Базовые валидаторы
const required = (field: string) => (value: any): ValidationError | null => 
  value == null || value === "" ? { field, message: "Поле обязательно для заполнения" } : null;

const minLength = (field: string) => (min: number) => (value: string): ValidationError | null => 
  value.length < min ? { field, message: `Минимальная длина ${min} символов` } : null;

const maxLength = (field: string) => (max: number) => (value: string): ValidationError | null => 
  value.length > max ? { field, message: `Максимальная длина ${max} символов` } : null;

const pattern = (field: string) => (regex: RegExp) => (value: string): ValidationError | null => 
  !regex.test(value) ? { field, message: "Неверный формат" } : null;

const min = (field: string) => (minValue: number) => (value: number): ValidationError | null => 
  value < minValue ? { field, message: `Минимальное значение ${minValue}` } : null;

const max = (field: string) => (maxValue: number) => (value: number): ValidationError | null => 
  value > maxValue ? { field, message: `Максимальное значение ${maxValue}` } : null;

// Комбинирование валидаторов
const combineValidators = <T>(...validators: ((value: T) => ValidationError | null)[]) => 
  (value: T): ValidationError[] => 
    validators.map(validator => validator(value)).filter((error): error is ValidationError => error !== null);

// Специализированные валидаторы
const validateName = combineValidators(
  required("name"),
  minLength("name")(2),
  maxLength("name")(50)
);

const validateEmail = combineValidators(
  required("email"),
  pattern("email")(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)
);

const validateAge = combineValidators(
  required("age"),
  min("age")(18),
  max("age")(120)
);

const validatePassword = combineValidators(
  required("password"),
  minLength("password")(8),
  pattern("password")(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) // Хотя бы одна заглавная, строчная буква и цифра
);

// Валидация формы регистрации
interface RegistrationForm {
  name: string;
  email: string;
  age: number;
  password: string;
  confirmPassword: string;
}

interface ValidatedUser {
  name: string;
  email: string;
  age: number;
  password: string;
}

const validateRegistrationForm = (form: RegistrationForm): ValidationResult<ValidatedUser> => {
  const errors: ValidationError[] = [
    ...validateName(form.name),
    ...validateEmail(form.email),
    ...validateAge(form.age),
    ...validatePassword(form.password)
  ];
  
  // Проверка совпадения паролей
  if (form.password !== form.confirmPassword) {
    errors.push({ field: "confirmPassword", message: "Пароли не совпадают" });
  }
  
  return errors.length > 0 ? left(errors) : right({
    name: form.name,
    email: form.email,
    age: form.age,
    password: form.password
  });
};

// Валидация с преобразованием данных
interface RawUserData {
  firstName: string;
  lastName: string;
  email: string;
  birthYear: string;
  salary: string;
}

interface ProcessedUserData {
  fullName: string;
  email: string;
  age: number;
  salary: number;
}

const validateAndProcessUserData = (data: RawUserData): ValidationResult<ProcessedUserData> => {
  const errors: ValidationError[] = [];
  
  // Валидация имени
  if (!data.firstName.trim() || !data.lastName.trim()) {
    errors.push({ field: "name", message: "Имя и фамилия обязательны" });
  }
  
  // Валидация email
  if (!data.email.includes("@")) {
    errors.push({ field: "email", message: "Неверный формат email" });
  }
  
  // Валидация года рождения
  const birthYear = parseInt(data.birthYear, 10);
  const currentYear = new Date().getFullYear();
  if (isNaN(birthYear) || birthYear < 1900 || birthYear > currentYear) {
    errors.push({ field: "birthYear", message: "Неверный год рождения" });
  }
  
  // Валидация зарплаты
  const salary = parseFloat(data.salary);
  if (isNaN(salary) || salary < 0) {
    errors.push({ field: "salary", message: "Неверный формат зарплаты" });
  }
  
  if (errors.length > 0) {
    return left(errors);
  }
  
  // Преобразование данных
  return right({
    fullName: `${data.firstName.trim()} ${data.lastName.trim()}`,
    email: data.email.trim(),
    age: currentYear - birthYear,
    salary
  });
};

// Использование
const validForm: RegistrationForm = {
  name: "Иван Иванов",
  email: "ivan@example.com",
  age: 25,
  password: "Password123",
  confirmPassword: "Password123"
};

const invalidForm: RegistrationForm = {
  name: "И",
  email: "invalid-email",
  age: 15,
  password: "123",
  confirmPassword: "456"
};

const validData: RawUserData = {
  firstName: "Иван",
  lastName: "Иванов",
  email: "ivan@example.com",
  birthYear: "1990",
  salary: "50000"
};

const invalidData: RawUserData = {
  firstName: "",
  lastName: "",
  email: "invalid",
  birthYear: "invalid",
  salary: "-1000"
};

console.log("Валидная форма:", validateRegistrationForm(validForm).getValue());
console.log("Невалидная форма:", validateRegistrationForm(invalidForm).getValue());
console.log("Валидные данные:", validateAndProcessUserData(validData).getValue());
console.log("Невалидные данные:", validateAndProcessUserData(invalidData).getValue());
```

## Управление состоянием приложения

Рассмотрим пример управления состоянием приложения с использованием State монады.

```typescript
// State монада
class StateMonad<S, A> {
  constructor(private run: (state: S) => [A, S]) {}
  
  static of<S, A>(value: A): StateMonad<S, A> {
    return new StateMonad(state => [value, state]);
  }
  
  static get<S>(): StateMonad<S, S> {
    return new StateMonad(state => [state, state]);
  }
  
  static put<S>(state: S): StateMonad<S, void> {
    return new StateMonad(() => [undefined, state]);
  }
  
  static modify<S>(fn: (state: S) => S): StateMonad<S, void> {
    return new StateMonad(state => [undefined, fn(state)]);
  }
  
  map<B>(fn: (a: A) => B): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return [fn(value), newState];
    });
  }
  
  flatMap<B>(fn: (a: A) => StateMonad<S, B>): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return fn(value).run(newState);
    });
  }
  
  evaluate(initialState: S): A {
    const [result] = this.run(initialState);
    return result;
  }
  
  execute(initialState: S): S {
    const [, newState] = this.run(initialState);
    return newState;
  }
}

// Модели состояния приложения
interface User {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

interface CartItem {
  productId: number;
  quantity: number;
}

interface AppState {
  users: User[];
  products: Product[];
  currentUser: User | null;
  cart: CartItem[];
  orders: any[]; // Упрощенная модель заказов
}

// Операции со state
const getCurrentUser = (): StateMonad<AppState, User | null> => {
  return StateMonad.get<AppState>().map(state => state.currentUser);
};

const setCurrentUser = (user: User | null): StateMonad<AppState, void> => {
  return StateMonad.modify<AppState>(state => ({
    ...state,
    currentUser: user
  }));
};

const getProducts = (): StateMonad<AppState, Product[]> => {
  return StateMonad.get<AppState>().map(state => state.products);
};

const getProductById = (id: number): StateMonad<AppState, Product | null> => {
  return StateMonad.get<AppState>().map(state => 
    state.products.find(p => p.id === id) || null
  );
};

const getCart = (): StateMonad<AppState, CartItem[]> => {
  return StateMonad.get<AppState>().map(state => state.cart);
};

const addToCart = (productId: number, quantity: number): StateMonad<AppState, boolean> => {
  return StateMonad.get<AppState>().flatMap(state => {
    const product = state.products.find(p => p.id === productId);
    
    // Проверка наличия товара
    if (!product || !product.inStock) {
      return StateMonad.of(false);
    }
    
    // Обновление корзины
    const existingItemIndex = state.cart.findIndex(item => item.productId === productId);
    let newCart: CartItem[];
    
    if (existingItemIndex >= 0) {
      // Обновление существующего элемента
      newCart = [...state.cart];
      newCart[existingItemIndex] = {
        ...newCart[existingItemIndex],
        quantity: newCart[existingItemIndex].quantity + quantity
      };
    } else {
      // Добавление нового элемента
      newCart = [...state.cart, { productId, quantity }];
    }
    
    // Обновление состояния
    return StateMonad.put<AppState>({
      ...state,
      cart: newCart
    }).map(() => true);
  });
};

const removeFromCart = (productId: number): StateMonad<AppState, void> => {
  return StateMonad.modify<AppState>(state => ({
    ...state,
    cart: state.cart.filter(item => item.productId !== productId)
  }));
};

const clearCart = (): StateMonad<AppState, void> => {
  return StateMonad.modify<AppState>(state => ({
    ...state,
    cart: []
  }));
};

const getCartTotal = (): StateMonad<AppState, number> => {
  return StateMonad.get<AppState>().map(state => {
    return state.cart.reduce((total, item) => {
      const product = state.products.find(p => p.id === item.productId);
      return total + (product ? product.price * item.quantity : 0);
    }, 0);
  });
};

const checkout = (): StateMonad<AppState, boolean> => {
  return StateMonad.get<AppState>().flatMap(state => {
    // Проверка наличия текущего пользователя
    if (!state.currentUser) {
      return StateMonad.of(false);
    }
    
    // Проверка наличия товаров в корзине
    if (state.cart.length === 0) {
      return StateMonad.of(false);
    }
    
    // Проверка наличия всех товаров
    for (const item of state.cart) {
      const product = state.products.find(p => p.id === item.productId);
      if (!product || !product.inStock) {
        return StateMonad.of(false);
      }
    }
    
    // Создание заказа (упрощенная реализация)
    const order = {
      id: Date.now(),
      userId: state.currentUser.id,
      items: [...state.cart],
      total: state.cart.reduce((total, item) => {
        const product = state.products.find(p => p.id === item.productId);
        return total + (product ? product.price * item.quantity : 0);
      }, 0),
      date: new Date()
    };
    
    // Обновление состояния: очистка корзины и добавление заказа
    return StateMonad.put<AppState>({
      ...state,
      cart: [],
      orders: [...state.orders, order]
    }).map(() => true);
  });
};

// Композиция операций
const shoppingSession = (): StateMonad<AppState, { success: boolean; total: number }> => {
  return getCurrentUser()
    .flatMap(user => {
      if (!user) {
        return StateMonad.of({ success: false, total: 0 });
      }
      
      return addToCart(1, 2) // Добавить 2 ноутбука
        .flatMap(success => {
          if (!success) {
            return StateMonad.of({ success: false, total: 0 });
          }
          
          return addToCart(3, 1) // Добавить 1 книгу
            .flatMap(success => {
              if (!success) {
                return StateMonad.of({ success: false, total: 0 });
              }
              
              return getCartTotal()
                .flatMap(total => 
                  checkout().map(success => ({ success, total }))
                );
            });
        });
    });
};

// Пример использования
const initialState: AppState = {
  users: [
    { id: 1, name: "Иван", email: "ivan@example.com" },
    { id: 2, name: "Мария", email: "maria@example.com" }
  ],
  products: [
    { id: 1, name: "Ноутбук", price: 50000, inStock: true },
    { id: 2, name: "Смартфон", price: 30000, inStock: false },
    { id: 3, name: "Книга", price: 500, inStock: true }
  ],
  currentUser: { id: 1, name: "Иван", email: "ivan@example.com" },
  cart: [],
  orders: []
};

const result = shoppingSession().evaluate(initialState);
const finalState = shoppingSession().execute(initialState);

console.log("Результат сессии:", result);
console.log("Финальное состояние корзины:", finalState.cart);
console.log("Финальные заказы:", finalState.orders);
```

## Обработка конфигурации

Рассмотрим пример обработки конфигурации приложения с использованием Reader монады.

```typescript
// Reader монада
class ReaderMonad<R, A> {
  constructor(private run: (env: R) => A) {}
  
  static of<R, A>(value: A): ReaderMonad<R, A> {
    return new ReaderMonad(() => value);
  }
  
  static ask<R>(): ReaderMonad<R, R> {
    return new ReaderMonad(env => env);
  }
  
  static local<R1, R2, A>(fn: (r1: R1) => R2, reader: ReaderMonad<R2, A>): ReaderMonad<R1, A> {
    return new ReaderMonad(env => reader.run(fn(env)));
  }
  
  map<B>(fn: (a: A) => B): ReaderMonad<R, B> {
    return new ReaderMonad(env => fn(this.run(env)));
  }
  
  flatMap<B>(fn: (a: A) => ReaderMonad<R, B>): ReaderMonad<R, B> {
    return new ReaderMonad(env => fn(this.run(env)).run(env));
  }
  
  runReader(env: R): A {
    return this.run(env);
  }
}

// Модели конфигурации
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
}

interface ApiConfig {
  baseUrl: string;
  apiKey: string;
  timeout: number;
}

interface LoggingConfig {
  level: "debug" | "info" | "warn" | "error";
  format: "json" | "text";
}

interface AppConfig {
  database: DatabaseConfig;
  api: ApiConfig;
  logging: LoggingConfig;
  environment: "development" | "production" | "test";
}

// Функции для работы с конфигурацией
const getDatabaseConfig = (): ReaderMonad<AppConfig, DatabaseConfig> => {
  return ReaderMonad.ask<AppConfig>().map(config => config.database);
};

const getApiConfig = (): ReaderMonad<AppConfig, ApiConfig> => {
  return ReaderMonad.ask<AppConfig>().map(config => config.api);
};

const getLoggingConfig = (): ReaderMonad<AppConfig, LoggingConfig> => {
  return ReaderMonad.ask<AppConfig>().map(config => config.logging);
};

const getEnvironment = (): ReaderMonad<AppConfig, "development" | "production" | "test"> => {
  return ReaderMonad.ask<AppConfig>().map(config => config.environment);
};

const isDevelopment = (): ReaderMonad<AppConfig, boolean> => {
  return getEnvironment().map(env => env === "development");
};

const isProduction = (): ReaderMonad<AppConfig, boolean> => {
  return getEnvironment().map(env => env === "production");
};

// Валидация конфигурации
const validateDatabaseConfig = (): ReaderMonad<AppConfig, boolean> => {
  return getDatabaseConfig().map(db => 
    db.host.length > 0 && 
    db.port > 0 && 
    db.port < 65536 && 
    db.username.length > 0
  );
};

const validateApiConfig = (): ReaderMonad<AppConfig, boolean> => {
  return getApiConfig().map(api => 
    api.baseUrl.length > 0 && 
    api.apiKey.length > 0
  );
};

const validateLoggingConfig = (): ReaderMonad<AppConfig, boolean> => {
  return getLoggingConfig().map(logging => 
    ["debug", "info", "warn", "error"].includes(logging.level) &&
    ["json", "text"].includes(logging.format)
  );
};

const validateConfig = (): ReaderMonad<AppConfig, boolean> => {
  return validateDatabaseConfig()
    .flatMap(dbValid => 
      validateApiConfig().flatMap(apiValid => 
        validateLoggingConfig().map(loggingValid => 
          dbValid && apiValid && loggingValid
        )
      )
    );
};

// Получение конфигурации для конкретной среды
const getDevelopmentConfig = (): ReaderMonad<AppConfig, AppConfig> => {
  return ReaderMonad.ask<AppConfig>().map(config => ({
    ...config,
    database: {
      ...config.database,
      host: "localhost"
    },
    api: {
      ...config.api,
      baseUrl: "http://localhost:3000"
    },
    logging: {
      ...config.logging,
      level: "debug"
    }
  }));
};

const getProductionConfig = (): ReaderMonad<AppConfig, AppConfig> => {
  return ReaderMonad.ask<AppConfig>().map(config => ({
    ...config,
    database: {
      ...config.database,
      host: "prod-db.example.com"
    },
    api: {
      ...config.api,
      baseUrl: "https://api.example.com"
    },
    logging: {
      ...config.logging,
      level: "error"
    }
  }));
};

const getConfigForEnvironment = (): ReaderMonad<AppConfig, AppConfig> => {
  return getEnvironment().flatMap(env => {
    switch (env) {
      case "development":
        return getDevelopmentConfig();
      case "production":
        return getProductionConfig();
      default:
        return ReaderMonad.ask<AppConfig>();
    }
  });
};

// Пример использования
const configProgram = ReaderMonad.of<AppConfig, string>("Начало")
  .flatMap(() => validateConfig())
  .flatMap(isValid => {
    if (!isValid) {
      return ReaderMonad.of("Конфигурация невалидна");
    }
    
    return getConfigForEnvironment()
      .flatMap(config => 
        ReaderMonad.of(`Конфигурация валидна для среды ${config.environment}`)
      );
  });

const developmentConfig: AppConfig = {
  database: {
    host: "localhost",
    port: 5432,
    username: "dev_user",
    password: "dev_password",
    database: "dev_db"
  },
  api: {
    baseUrl: "http://localhost:3000",
    apiKey: "dev_api_key",
    timeout: 5000
  },
  logging: {
    level: "debug",
    format: "json"
  },
  environment: "development"
};

const productionConfig: AppConfig = {
  database: {
    host: "prod.example.com",
    port: 5432,
    username: "prod_user",
    password: "prod_password",
    database: "prod_db"
  },
  api: {
    baseUrl: "https://api.example.com",
    apiKey: "prod_api_key",
    timeout: 10000
  },
  logging: {
    level: "error",
    format: "json"
  },
  environment: "production"
};

const invalidConfig: AppConfig = {
  database: {
    host: "",
    port: 5432,
    username: "user",
    password: "password",
    database: "db"
  },
  api: {
    baseUrl: "",
    apiKey: "",
    timeout: 5000
  },
  logging: {
    level: "invalid" as any,
    format: "json"
  },
  environment: "development"
};

console.log("Development:", configProgram.runReader(developmentConfig));
console.log("Production:", configProgram.runReader(productionConfig));
console.log("Invalid:", configProgram.runReader(invalidConfig));
```

## Работа с файлами и потоками данных

Рассмотрим пример работы с файлами и потоками данных с использованием различных монад.

```typescript
// Maybe монада для безопасной работы с файлами
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
}

// Either монада для обработки ошибок файловых операций
type FileError = 
  | { type: "not_found", message: string }
  | { type: "permission", message: string }
  | { type: "corrupted", message: string }
  | { type: "other", message: string };

type FileResult<T> = Either<FileError, T>;

// Future монада для асинхронных файловых операций
class FutureMonad<A> {
  constructor(private computation: () => Promise<A>) {}
  
  static of<A>(value: A): FutureMonad<A> {
    return new FutureMonad(() => Promise.resolve(value));
  }
  
  static fromPromise<A>(promise: Promise<A>): FutureMonad<A> {
    return new FutureMonad(() => promise);
  }
  
  map<B>(fn: (a: A) => B): FutureMonad<B> {
    return new FutureMonad(() => this.computation().then(fn));
  }
  
  flatMap<B>(fn: (a: A) => FutureMonad<B>): FutureMonad<B> {
    return new FutureMonad(() => 
      this.computation().then(a => fn(a).computation())
    );
  }
  
  runFuture(): Promise<A> {
    return this.computation();
  }
  
  catchError<B>(fn: (error: any) => B): FutureMonad<A | B> {
    return new FutureMonad(() => 
      this.computation().catch(fn)
    );
  }
}

// Модели файлов
interface FileInfo {
  name: string;
  size: number;
  type: string;
  lastModified: number;
  path: string;
}

interface FileContent {
  content: string;
  encoding: string;
}

interface ProcessedFile {
  info: FileInfo;
  content: FileContent;
  wordCount: number;
  lineCount: number;
}

// Симуляция файловой системы
class FileSystem {
  private files: Record<string, { info: FileInfo; content: string }> = {};
  
  constructor() {
    // Инициализация с тестовыми файлами
    this.files["/documents/readme.txt"] = {
      info: {
        name: "readme.txt",
        size: 100,
        type: "text/plain",
        lastModified: Date.now(),
        path: "/documents/readme.txt"
      },
      content: "Это тестовый файл\nС несколькими строками\nИ разным содержимым"
    };
    
    this.files["/data/users.json"] = {
      info: {
        name: "users.json",
        size: 200,
        type: "application/json",
        lastModified: Date.now(),
        path: "/data/users.json"
      },
      content: '[{"id": 1, "name": "Иван"}, {"id": 2, "name": "Мария"}]'
    };
    
    this.files["/logs/error.log"] = {
      info: {
        name: "error.log",
        size: 500,
        type: "text/plain",
        lastModified: Date.now(),
        path: "/logs/error.log"
      },
      content: "ERROR: Ошибка подключения к базе данных\nERROR: Таймаут запроса"
    };
  }
  
  readFile(path: string): FutureMonad<FileResult<{ info: FileInfo; content: string }>> {
    return FutureMonad.fromPromise(
      new Promise<FileResult<{ info: FileInfo; content: string }>>(resolve => {
        setTimeout(() => {
          const file = this.files[path];
          if (file) {
            resolve(right(file));
          } else {
            resolve(left({ type: "not_found", message: `Файл не найден: ${path}` }));
          }
        }, 100);
      })
    );
  }
  
  writeFile(path: string, content: string): FutureMonad<FileResult<void>> {
    return FutureMonad.fromPromise(
      new Promise<FileResult<void>>(resolve => {
        setTimeout(() => {
          try {
            this.files[path] = {
              info: {
                name: path.split("/").pop() || "",
                size: content.length,
                type: "text/plain",
                lastModified: Date.now(),
                path
              },
              content
            };
            resolve(right(undefined));
          } catch (error) {
            resolve(left({ type: "other", message: `Ошибка записи файла: ${error}` }));
          }
        }, 100);
      })
    );
  }
  
  listFiles(directory: string): FutureMonad<FileResult<FileInfo[]>> {
    return FutureMonad.fromPromise(
      new Promise<FileResult<FileInfo[]>>(resolve => {
        setTimeout(() => {
          try {
            const files = Object.values(this.files)
              .filter(file => file.info.path.startsWith(directory))
              .map(file => file.info);
            resolve(right(files));
          } catch (error) {
            resolve(left({ type: "other", message: `Ошибка получения списка файлов: ${error}` }));
          }
        }, 100);
      })
    );
  }
}

// Функции обработки файлов
const countWords = (content: string): number => {
  return content.trim().split(/\s+/).filter(word => word.length > 0).length;
};

const countLines = (content: string): number => {
  return content.split("\n").length;
};

const processFileContent = (file: { info: FileInfo; content: string }): ProcessedFile => {
  return {
    info: file.info,
    content: {
      content: file.content,
      encoding: "utf-8"
    },
    wordCount: countWords(file.content),
    lineCount: countLines(file.content)
  };
};

const filterTextFiles = (files: FileInfo[]): FileInfo[] => {
  return files.filter(file => file.type === "text/plain");
};

const filterJsonFiles = (files: FileInfo[]): FileInfo[] => {
  return files.filter(file => file.type === "application/json");
};

const getFileExtension = (filename: string): MaybeMonad<string> => {
  const parts = filename.split(".");
  return parts.length > 1 
    ? MaybeMonad.just(parts[parts.length - 1].toLowerCase())
    : MaybeMonad.nothing();
};

const isImageFile = (filename: string): boolean => {
  const imageExtensions = ["jpg", "jpeg", "png", "gif", "svg", "bmp"];
  return getFileExtension(filename)
    .map(ext => imageExtensions.includes(ext))
    .orElse(false);
};

const isDocumentFile = (filename: string): boolean => {
  const documentExtensions = ["txt", "doc", "docx", "pdf", "md"];
  return getFileExtension(filename)
    .map(ext => documentExtensions.includes(ext))
    .orElse(false);
};

// Композиция операций с файлами
const processTextFile = (fs: FileSystem, path: string): FutureMonad<FileResult<ProcessedFile>> => {
  return fs.readFile(path)
    .map(result => {
      if (result._tag === "Left") {
        return result;
      }
      
      return right(processFileContent(result.right));
    });
};

const processDirectory = (fs: FileSystem, directory: string): FutureMonad<FileResult<ProcessedFile[]>> => {
  return fs.listFiles(directory)
    .flatMap(result => {
      if (result._tag === "Left") {
        return FutureMonad.of(result);
      }
      
      const textFiles = filterTextFiles(result.right);
      const processingPromises = textFiles.map(file => 
        processTextFile(fs, file.path).runFuture()
      );
      
      return FutureMonad.fromPromise(
        Promise.all(processingPromises).then(results => {
          const processedFiles: ProcessedFile[] = [];
          const errors: FileError[] = [];
          
          results.forEach(result => {
            if (result._tag === "Right") {
              processedFiles.push(result.right);
            } else {
              errors.push(result.left);
            }
          });
          
          if (errors.length > 0) {
            return left(errors[0]); // Возвращаем первую ошибку
          }
          
          return right(processedFiles);
        })
      );
    });
};

const analyzeDirectory = (fs: FileSystem, directory: string): FutureMonad<FileResult<{
  totalFiles: number;
  textFiles: number;
  totalWords: number;
  totalLines: number;
}>> => {
  return fs.listFiles(directory)
    .flatMap(result => {
      if (result._tag === "Left") {
        return FutureMonad.of(result);
      }
      
      const allFiles = result.right;
      const textFiles = filterTextFiles(allFiles);
      
      const processingPromises = textFiles.map(file => 
        fs.readFile(file.path).runFuture()
      );
      
      return FutureMonad.fromPromise(
        Promise.all(processingPromises).then(fileResults => {
          let totalWords = 0;
          let totalLines = 0;
          
          fileResults.forEach(result => {
            if (result._tag === "Right") {
              totalWords += countWords(result.right.content);
              totalLines += countLines(result.right.content);
            }
          });
          
          return right({
            totalFiles: allFiles.length,
            textFiles: textFiles.length,
            totalWords,
            totalLines
          });
        })
      );
    });
};

// Использование
const fs = new FileSystem();

// Обработка одного файла
processTextFile(fs, "/documents/readme.txt")
  .tap(result => {
    if (result._tag === "Right") {
      console.log("Обработанный файл:", result.right);
    } else {
      console.log("Ошибка обработки файла:", result.left);
    }
  })
  .runFuture();

// Обработка директории
processDirectory(fs, "/documents")
  .tap(result => {
    if (result._tag === "Right") {
      console.log("Обработанные файлы:", result.right);
    } else {
      console.log("Ошибка обработки директории:", result.left);
    }
  })
  .runFuture();

// Анализ директории
analyzeDirectory(fs, "/")
  .tap(result => {
    if (result._tag === "Right") {
      console.log("Анализ директории:", result.right);
    } else {
      console.log("Ошибка анализа директории:", result.left);
    }
  })
  .runFuture();

// Примеры использования вспомогательных функций
console.log("Расширение файла:", getFileExtension("document.pdf").getValue()); // "pdf"
console.log("Файл изображения:", isImageFile("photo.jpg")); // true
console.log("Файл документа:", isDocumentFile("report.docx")); // true
```

## Функциональные компоненты React

Рассмотрим пример использования функторов и монад в функциональных компонентах React.

```typescript
// Пример с функциональными компонентами React
// Note: This is a conceptual example showing how monads can be used in React
// In practice, you would need to import React and use proper JSX syntax

// Maybe монада для безопасной работы с props
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
}

// Either монада для обработки ошибок в компонентах
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherMonad<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonad<E, A> {
    return new EitherMonad(value);
  }
  
  static right<E, A>(a: A): EitherMonad<E, A> {
    return new EitherMonad(right(a));
  }
  
  static left<E, A>(e: E): EitherMonad<E, A> {
    return new EitherMonad(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return new EitherMonad(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonad<E, B>): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return fn(this.value.right);
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
}

// HOC (Higher-Order Components) с использованием монад
const withUserData = <P extends object>(Component: React.ComponentType<P>) => 
  (props: P & { userId?: number }) => {
    const userData = MaybeMonad.of(props.userId)
      .flatMap(id => {
        // Симуляция получения данных пользователя
        if (id > 0) {
          return MaybeMonad.just({
            id,
            name: `Пользователь ${id}`,
            email: `user${id}@example.com`
          });
        }
        return MaybeMonad.nothing();
      });
    
    return userData.isJust() 
      ? <Component {...props} user={userData.getValue()} />
      : <Component {...props} user={null} />;
  };

const withErrorBoundary = <P extends object>(Component: React.ComponentType<P>) => 
  (props: P) => {
    try {
      return <Component {...props} />;
    } catch (error) {
      return <div>Ошибка: {error instanceof Error ? error.message : "Неизвестная ошибка"}</div>;
    }
  };

// Компоненты с использованием монад
interface UserCardProps {
  userId?: number;
  user?: {
    id: number;
    name: string;
    email: string;
  } | null;
}

const UserCard: React.FC<UserCardProps> = ({ user }) => {
  return MaybeMonad.of(user)
    .map(u => (
      <div className="user-card">
        <h3>{u.name}</h3>
        <p>Email: {u.email}</p>
      </div>
    ))
    .orElse(<div>Пользователь не найден</div>);
};

const EnhancedUserCard = withUserData(UserCard);

// Компонент для обработки форм с валидацией
interface FormFieldProps<T> {
  value: T;
  onChange: (value: T) => void;
  validator: (value: T) => Either<string, T>;
  label: string;
}

const FormField = <T,>({ value, onChange, validator, label }: FormFieldProps<T>) => {
  const validation = validator(value);
  
  return (
    <div className="form-field">
      <label>{label}</label>
      <input 
        value={value as any} 
        onChange={(e) => onChange(e.target.value as any)} 
      />
      {validation._tag === "Left" && (
        <span className="error">{validation.left}</span>
      )}
    </div>
  );
};

// Валидаторы
const validateEmail = (email: string): Either<string, string> => {
  return email.includes("@") 
    ? right(email) 
    : left("Неверный формат email");
};

const validateRequired = (value: string): Either<string, string> => {
  return value.trim().length > 0 
    ? right(value) 
    : left("Поле обязательно для заполнения");
};

// Компонент формы
const UserForm: React.FC = () => {
  const [email, setEmail] = React.useState("");
  const [name, setName] = React.useState("");
  
  const handleSubmit = () => {
    const emailValidation = validateEmail(email);
    const nameValidation = validateRequired(name);
    
    if (emailValidation._tag === "Right" && nameValidation._tag === "Right") {
      console.log("Форма валидна:", { email: emailValidation.right, name: nameValidation.right });
    } else {
      console.log("Форма невалидна");
    }
  };
  
  return (
    <form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
      <FormField 
        value={name}
        onChange={setName}
        validator={validateRequired}
        label="Имя"
      />
      <FormField 
        value={email}
        onChange={setEmail}
        validator={validateEmail}
        label="Email"
      />
      <button type="submit">Отправить</button>
    </form>
  );
};

// Компонент для работы с асинхронными данными
interface AsyncDataProps<T> {
  data: Promise<T>;
  render: (data: T) => React.ReactNode;
  loading?: React.ReactNode;
  error?: (error: any) => React.ReactNode;
}

const AsyncData = <T,>({ data, render, loading, error }: AsyncDataProps<T>) => {
  const [state, setState] = React.useState<Either<any, T> | null>(null);
  
  React.useEffect(() => {
    data.then(
      result => setState(right(result)),
      error => setState(left(error))
    );
  }, [data]);
  
  if (state === null) {
    return loading || <div>Загрузка...</div>;
  }
  
  return state._tag === "Right" 
    ? render(state.right)
    : (error ? error(state.left) : <div>Ошибка: {state.left.message || state.left}</div>);
};

// Пример использования AsyncData
const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const userData = new Promise<any>(resolve => {
    setTimeout(() => {
      resolve({
        id: userId,
        name: `Пользователь ${userId}`,
        email: `user${userId}@example.com`
      });
    }, 1000);
  });
  
  return (
    <AsyncData
      data={userData}
      render={user => (
        <div>
          <h2>{user.name}</h2>
          <p>Email: {user.email}</p>
        </div>
      )}
      loading={<div>Загрузка профиля...</div>}
      error={error => <div>Ошибка загрузки профиля: {error.message}</div>}
    />
  );
};
```

## Обработка событий и команд

Рассмотрим пример обработки событий и команд с использованием монад.

```typescript
// Модели событий и команд
interface Event {
  type: string;
  payload: any;
  timestamp: number;
  source: string;
}

interface Command {
  type: string;
  payload: any;
  correlationId: string;
}

interface CommandResult<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// Either монада для обработки результатов команд
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherMonad<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonad<E, A> {
    return new EitherMonad(value);
  }
  
  static right<E, A>(a: A): EitherMonad<E, A> {
    return new EitherMonad(right(a));
  }
  
  static left<E, A>(e: E): EitherMonad<E, A> {
    return new EitherMonad(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return new EitherMonad(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonad<E, B>): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return fn(this.value.right);
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
}

// Maybe монада для безопасной обработки событий
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
}

// Обработчики событий
class EventHandler {
  private handlers: Record<string, ((event: Event) => void)[]> = {};
  
  register(type: string, handler: (event: Event) => void) {
    if (!this.handlers[type]) {
      this.handlers[type] = [];
    }
    this.handlers[type].push(handler);
  }
  
  dispatch(event: Event) {
    const handlers = this.handlers[event.type] || [];
    handlers.forEach(handler => handler(event));
  }
}

// Обработчики команд
class CommandHandler {
  private handlers: Record<string, ((command: Command) => Promise<CommandResult<any>>)[]> = {};
  
  register<T>(type: string, handler: (command: Command) => Promise<CommandResult<T>>) {
    if (!this.handlers[type]) {
      this.handlers[type] = [];
    }
    this.handlers[type].push(handler);
  }
  
  async execute<T>(command: Command): Promise<CommandResult<T>> {
    const handlers = this.handlers[command.type] || [];
    
    if (handlers.length === 0) {
      return {
        success: false,
        error: `Нет обработчика для команды типа: ${command.type}`
      };
    }
    
    try {
      // Выполняем первый обработчик (в реальном приложении может быть логика выбора)
      const result = await handlers[0](command);
      return result;
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : "Неизвестная ошибка"
      };
    }
  }
}

// Специализированные обработчики
interface UserCreatedEvent extends Event {
  type: "USER_CREATED";
  payload: {
    userId: number;
    name: string;
    email: string;
  };
}

interface CreateUserCommand extends Command {
  type: "CREATE_USER";
  payload: {
    name: string;
    email: string;
  };
}

// Валидация команд
const validateCreateUserCommand = (command: CreateUserCommand): EitherMonad<string, CreateUserCommand> => {
  if (!command.payload.name || command.payload.name.trim().length === 0) {
    return EitherMonad.left("Имя пользователя обязательно");
  }
  
  if (!command.payload.email || !command.payload.email.includes("@")) {
    return EitherMonad.left("Неверный формат email");
  }
  
  return EitherMonad.right(command);
};

// Обработчик создания пользователя
const createUserHandler = async (command: CreateUserCommand): Promise<CommandResult<{ userId: number }>> => {
  return validateCreateUserCommand(command)
    .flatMap(validCommand => {
      // Симуляция создания пользователя
      return EitherMonad.right({
        success: true,
        data: {
          userId: Math.floor(Math.random() * 1000000)
        }
      });
    })
    .getValue() as CommandResult<{ userId: number }>;
};

// Обработчик события создания пользователя
const userCreatedEventHandler = (event: UserCreatedEvent) => {
  console.log(`Пользователь создан: ${event.payload.name} (${event.payload.email})`);
  
  // Дополнительная обработка события
  MaybeMonad.of(event.payload.userId)
    .map(userId => {
      // Отправка приветственного email
      console.log(`Отправка приветственного email пользователю ${userId}`);
      return userId;
    })
    .orElse(() => {
      console.log("Невозможно отправить email - отсутствует ID пользователя");
      return undefined;
    });
};

// Система обработки событий и команд
class EventSourcingSystem {
  private eventHandler: EventHandler;
  private commandHandler: CommandHandler;
  
  constructor() {
    this.eventHandler = new EventHandler();
    this.commandHandler = new CommandHandler();
    
    // Регистрация обработчиков
    this.commandHandler.register("CREATE_USER", createUserHandler);
    this.eventHandler.register("USER_CREATED", userCreatedEventHandler);
  }
  
  async executeCommand<T>(command: Command): Promise<CommandResult<T>> {
    const result = await this.commandHandler.execute<T>(command);
    
    // Если команда выполнена успешно, генерируем событие
    if (result.success && command.type === "CREATE_USER") {
      const createUserCommand = command as CreateUserCommand;
      const userCreatedEvent: UserCreatedEvent = {
        type: "USER_CREATED",
        payload: {
          userId: (result.data as any).userId,
          name: createUserCommand.payload.name,
          email: createUserCommand.payload.email
        },
        timestamp: Date.now(),
        source: "UserAggregate"
      };
      
      this.eventHandler.dispatch(userCreatedEvent);
    }
    
    return result;
  }
  
  dispatchEvent(event: Event) {
    this.eventHandler.dispatch(event);
  }
}

// Пример использования
const eventSourcingSystem = new EventSourcingSystem();

// Выполнение команды создания пользователя
const createUserCommand: CreateUserCommand = {
  type: "CREATE_USER",
  payload: {
    name: "Иван Иванов",
    email: "ivan@example.com"
  },
  correlationId: "cmd-" + Date.now()
};

eventSourcingSystem.executeCommand<{ userId: number }>(createUserCommand)
  .then(result => {
    if (result.success) {
      console.log("Пользователь успешно создан:", result.data);
    } else {
      console.log("Ошибка создания пользователя:", result.error);
    }
  });

// Пример с невалидной командой
const invalidCreateUserCommand: CreateUserCommand = {
  type: "CREATE_USER",
  payload: {
    name: "",
    email: "invalid-email"
  },
  correlationId: "cmd-" + Date.now()
};

eventSourcingSystem.executeCommand<{ userId: number }>(invalidCreateUserCommand)
  .then(result => {
    console.log("Результат невалидной команды:", result);
  });

// Генерация события напрямую
const customEvent: Event = {
  type: "CUSTOM_EVENT",
  payload: { message: "Пользовательское событие" },
  timestamp: Date.now(),
  source: "CustomSource"
};

eventSourcingSystem.dispatchEvent(customEvent);
```

## Связи с другими концепциями

- [[ts/functional-programming/functors-monads/Функторы_и_монады|Функторы и монады]] - Основная страница раздела
- [[ts/functional-programming/functors-monads/functors|Функторы]] - Базовая концепция функторов
- [[ts/functional-programming/functors-monads/monads|Монады]] - Подробное изучение монад
- [[ts/functional-programming/functors-monads/common-monads|Распространенные монады]] - Изучение популярных монад
- [[ts/error-handling/index|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование функторов и монад
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация монад
- [[ts/react/components|React компоненты]] - Использование функциональных концепций в React

## Следующие шаги

- [[ts/error-handling/index|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование функторов и монад
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация монад
- [[ts/functional-programming/Функциональное_программирование|Функциональное программирование]] - Обзор всех концепций функционального программирования
- [[ts/react/React_с_TypeScript|TypeScript с React]] - Использование функциональных концепций в React