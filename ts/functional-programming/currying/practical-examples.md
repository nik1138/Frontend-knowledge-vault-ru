# Практические примеры каррирования

В этом разделе мы рассмотрим реальные примеры использования каррирования в TypeScript приложениях, включая работу с API, обработку данных и создание утилит.

## Содержание

- [Практические примеры каррирования](#практические-примеры-каррирования)
  - [Содержание](#содержание)
  - [Работа с API](#работа-с-api)
  - [Обработка данных](#обработка-данных)
  - [Создание утилит](#создание-утилит)
  - [Работа с DOM](#работа-с-dom)
  - [Функциональные компоненты React](#функциональные-компоненты-react)
  - [Валидация данных](#валидация-данных)
  - [Работа с событиями](#работа-с-событиями)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Работа с API

Каррирование полезно при создании API клиентов с предустановленной конфигурацией.

```typescript
// Базовый API клиент
interface ApiClientConfig {
  baseUrl: string;
  defaultHeaders: Record<string, string>;
}

interface ApiRequest {
  method: string;
  endpoint: string;
  headers?: Record<string, string>;
  body?: any;
}

const createApiClient = (config: ApiClientConfig) => (request: ApiRequest) => {
  const url = `${config.baseUrl}${request.endpoint}`;
  const headers = { ...config.defaultHeaders, ...request.headers };
  
  console.log(`Запрос: ${request.method} ${url}`, { headers, body: request.body });
  
  // Здесь был бы реальный fetch запрос
  return Promise.resolve({
    status: 200,
    data: request.body || {}
  });
};

// Создание специализированных клиентов
const githubApiClient = createApiClient({
  baseUrl: "https://api.github.com",
  defaultHeaders: {
    "Accept": "application/vnd.github.v3+json",
    "User-Agent": "MyApp/1.0"
  }
});

const jsonPlaceholderApiClient = createApiClient({
  baseUrl: "https://jsonplaceholder.typicode.com",
  defaultHeaders: {
    "Content-Type": "application/json"
  }
});

// Использование
githubApiClient({
  method: "GET",
  endpoint: "/users"
});

jsonPlaceholderApiClient({
  method: "POST",
  endpoint: "/posts",
  body: {
    title: "Новый пост",
    body: "Содержание поста",
    userId: 1
  }
});
```

## Обработка данных

Каррирование упрощает создание функций для обработки данных с различными параметрами.

```typescript
// Функции для обработки массивов данных
const filterBy = <T>(predicate: (item: T) => boolean) => (array: T[]): T[] => 
  array.filter(predicate);

const mapWith = <T, R>(fn: (item: T) => R) => (array: T[]): R[] => 
  array.map(fn);

const sortBy = <T>(keyFn: (item: T) => any) => (array: T[]): T[] => 
  [...array].sort((a, b) => {
    const aKey = keyFn(a);
    const bKey = keyFn(b);
    if (aKey < bKey) return -1;
    if (aKey > bKey) return 1;
    return 0;
  });

// Создание специализированных функций
interface User {
  id: number;
  name: string;
  age: number;
  department: string;
  salary: number;
}

const users: User[] = [
  { id: 1, name: "Иван", age: 30, department: "IT", salary: 50000 },
  { id: 2, name: "Мария", age: 25, department: "HR", salary: 45000 },
  { id: 3, name: "Алексей", age: 35, department: "IT", salary: 60000 },
  { id: 4, name: "Елена", age: 28, department: "Finance", salary: 55000 }
];

// Фильтрация пользователей
const filterByDepartment = (department: string) => 
  filterBy((user: User) => user.department === department);

const filterByMinAge = (minAge: number) => 
  filterBy((user: User) => user.age >= minAge);

// Сортировка пользователей
const sortByAge = sortBy((user: User) => user.age);
const sortByName = sortBy((user: User) => user.name);
const sortBySalary = sortBy((user: User) => user.salary);

// Преобразование пользователей
const mapToNames = mapWith((user: User) => user.name);
const mapToSalaries = mapWith((user: User) => user.salary);

// Композиция операций
const getItEmployees = filterByDepartment("IT");
const getAdults = filterByMinAge(18);
const getItEmployeeNames = (users: User[]) => 
  mapToNames(sortByName(getItEmployees(users)));

console.log(getItEmployeeNames(users)); // ["Алексей", "Иван"]
```

## Создание утилит

Каррирование полезно при создании универсальных утилит с настраиваемым поведением.

```typescript
// Утилита для форматирования значений
const formatWith = (formatter: Intl.NumberFormat) => (value: number): string => 
  formatter.format(value);

// Создание специализированных форматтеров
const formatCurrencyRu = formatWith(
  new Intl.NumberFormat("ru-RU", {
    style: "currency",
    currency: "RUB"
  })
);

const formatCurrencyUs = formatWith(
  new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD"
  })
);

const formatPercent = formatWith(
  new Intl.NumberFormat("ru-RU", {
    style: "percent",
    minimumFractionDigits: 2
  })
);

// Использование
console.log(formatCurrencyRu(1234.56)); // "1 234,56 ₽"
console.log(formatCurrencyUs(1234.56)); // "$1,234.56"
console.log(formatPercent(0.1234)); // "12,34 %"

// Утилита для проверки условий
const checkWith = <T>(predicate: (value: T) => boolean) => (value: T): boolean => 
  predicate(value);

// Создание специализированных проверок
const isPositive = checkWith((n: number) => n > 0);
const isEven = checkWith((n: number) => n % 2 === 0);
const isEmpty = checkWith((str: string) => str.length === 0);

// Использование
console.log(isPositive(5)); // true
console.log(isEven(4)); // true
console.log(isEmpty("")); // true
```

## Работа с DOM

Каррирование упрощает создание функций для работы с DOM элементами.

```typescript
// Утилиты для работы с DOM
const setAttribute = (name: string) => (value: string) => (element: HTMLElement): HTMLElement => {
  element.setAttribute(name, value);
  return element;
};

const addClass = (className: string) => (element: HTMLElement): HTMLElement => {
  element.classList.add(className);
  return element;
};

const setText = (text: string) => (element: HTMLElement): HTMLElement => {
  element.textContent = text;
  return element;
};

const setStyle = (property: string) => (value: string) => (element: HTMLElement): HTMLElement => {
  element.style.setProperty(property, value);
  return element;
};

// Создание специализированных функций
const setId = setAttribute("id");
const setClass = setAttribute("class");
const setColor = setStyle("color");
const setBackgroundColor = setStyle("background-color");

// Пример использования
const createStyledElement = (tag: string) => {
  const element = document.createElement(tag);
  
  // Композиция стилей
  return setColor("blue")(
    setBackgroundColor("lightgray")(
      addClass("styled-element")(
        setId("my-element")(element)
      )
    )
  );
};

// Альтернативный подход с композицией
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): ((a: A) => C) => 
  (a: A) => f(g(a));

const createStyledElement2 = (tag: string) => {
  const element = document.createElement(tag);
  
  const applyStyles = compose(
    setColor("blue"),
    compose(
      setBackgroundColor("lightgray"),
      compose(
        addClass("styled-element"),
        setId("my-element")
      )
    )
  );
  
  return applyStyles(element);
};
```

## Функциональные компоненты React

Каррирование полезно при создании переиспользуемых компонентов в React.

```typescript
// Пример с функциональными компонентами React
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: "primary" | "secondary" | "danger";
  size?: "small" | "medium" | "large";
}

// Каррированная функция для создания кнопок с предустановленными стилями
const createButton = (variant: ButtonProps["variant"]) => 
  (size: ButtonProps["size"]) => 
  (props: Omit<ButtonProps, "variant" | "size">) => ({
    ...props,
    variant,
    size
  });

// Создание специализированных кнопок
const createPrimaryButton = createButton("primary");
const createSecondaryButton = createButton("secondary");
const createDangerButton = createButton("danger");

const createLargeButton = (createBtn: ReturnType<typeof createButton>) => 
  createBtn("large");

const createMediumButton = (createBtn: ReturnType<typeof createButton>) => 
  createBtn("medium");

const createSmallButton = (createBtn: ReturnType<typeof createButton>) => 
  createBtn("small");

// Создание конкретных кнопок
const createLargePrimaryButton = createLargeButton(createPrimaryButton);
const createMediumSecondaryButton = createMediumButton(createSecondaryButton);
const createSmallDangerButton = createSmallButton(createDangerButton);

// Использование
const primaryButtonProps = createLargePrimaryButton({
  text: "Сохранить",
  onClick: () => console.log("Сохранено")
});

const secondaryButtonProps = createMediumSecondaryButton({
  text: "Отмена",
  onClick: () => console.log("Отменено")
});

const dangerButtonProps = createSmallDangerButton({
  text: "Удалить",
  onClick: () => console.log("Удалено")
});
```

## Валидация данных

Каррирование упрощает создание валидаторов с различными правилами.

```typescript
// Система валидации данных
interface ValidationError {
  field: string;
  message: string;
}

type Validator<T> = (value: T) => ValidationError | null;

// Каррированные функции для создания валидаторов
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

// Создание специализированных валидаторов
const validateName = (value: string): ValidationError | null => {
  const requiredValidator = required("name");
  const minLengthValidator = minLength("name")(2);
  const maxLengthValidator = maxLength("name")(50);
  
  return requiredValidator(value) || minLengthValidator(value) || maxLengthValidator(value);
};

const validateEmail = (value: string): ValidationError | null => {
  const requiredValidator = required("email");
  const patternValidator = pattern("email")(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
  
  return requiredValidator(value) || patternValidator(value);
};

const validateAge = (value: number): ValidationError | null => {
  const requiredValidator = required("age");
  const minValidator = min("age")(18);
  const maxValidator = max("age")(120);
  
  return requiredValidator(value) || minValidator(value) || maxValidator(value);
};

// Валидация объекта
interface UserForm {
  name: string;
  email: string;
  age: number;
}

const validateUserForm = (form: UserForm): ValidationError[] => {
  const errors: (ValidationError | null)[] = [
    validateName(form.name),
    validateEmail(form.email),
    validateAge(form.age)
  ];
  
  return errors.filter((error): error is ValidationError => error !== null);
};

// Использование
const formData: UserForm = {
  name: "И",
  email: "invalid-email",
  age: 15
};

const errors = validateUserForm(formData);
console.log(errors);
// [
//   { field: "name", message: "Минимальная длина 2 символов" },
//   { field: "email", message: "Неверный формат" },
//   { field: "age", message: "Минимальное значение 18" }
// ]
```

## Работа с событиями

Каррирование полезно при обработке событий с различными параметрами.

```typescript
// Система обработки событий
interface EventData {
  type: string;
  payload: any;
  timestamp: number;
}

type EventHandler = (event: EventData) => void;

// Каррированные функции для создания обработчиков событий
const logEvent = (prefix: string) => (event: EventData): void => {
  console.log(`[${prefix}] ${event.type}:`, event.payload);
};

const filterByType = (type: string) => (handler: EventHandler): EventHandler => 
  (event: EventData) => {
    if (event.type === type) {
      handler(event);
    }
  };

const throttle = (delay: number) => (handler: EventHandler): EventHandler => {
  let lastCall = 0;
  return (event: EventData) => {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      handler(event);
    }
  };
};

const debounce = (delay: number) => (handler: EventHandler): EventHandler => {
  let timeoutId: NodeJS.Timeout;
  return (event: EventData) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => handler(event), delay);
  };
};

// Создание специализированных обработчиков
const logUserEvent = logEvent("USER");
const logSystemEvent = logEvent("SYSTEM");

const handleUserLogin = filterByType("USER_LOGIN")(logUserEvent);
const handleUserLogout = filterByType("USER_LOGOUT")(logUserEvent);

const throttledClickHandler = throttle(1000)(logEvent("CLICK"));
const debouncedInputHandler = debounce(300)(logEvent("INPUT"));

// Использование
const events: EventData[] = [
  { type: "USER_LOGIN", payload: { userId: 1 }, timestamp: Date.now() },
  { type: "USER_LOGOUT", payload: { userId: 1 }, timestamp: Date.now() },
  { type: "CLICK", payload: { x: 100, y: 200 }, timestamp: Date.now() },
  { type: "INPUT", payload: { value: "test" }, timestamp: Date.now() }
];

events.forEach(event => {
  handleUserLogin(event);
  handleUserLogout(event);
  throttledClickHandler(event);
  debouncedInputHandler(event);
});
```

## Связи с другими концепциями

- [[ts/functional-programming/currying/Каррирование|Каррирование]] - Основная страница раздела
- [[ts/functional-programming/currying/implementation|Реализация каррирования]] - Различные способы реализации каррирования
- [[ts/functional-programming/currying/partial-application|Частичное применение]] - Использование частично примененных функций
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[Типизация компонентов React в TypeScript|React компоненты]] - Использование каррирования в React
- [[Тестирование в TypeScript|Тестирование]] - Тестирование каррированных функций

## Следующие шаги

- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация каррированных функций
- [[Тестирование в TypeScript|Тестирование]] - Тестирование каррированных функций
- [[TypeScript с React|TypeScript с React]] - Использование функциональных концепций в React