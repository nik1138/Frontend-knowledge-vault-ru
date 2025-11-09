# Практические примеры композиции функций

В этом разделе мы рассмотрим реальные примеры использования композиции функций в TypeScript приложениях, включая обработку данных, работу с API, валидацию и другие практические сценарии.

## Содержание

- [Практические примеры композиции функций](#практические-примеры-композиции-функций)
  - [Содержание](#содержание)
  - [Обработка данных пользователей](#обработка-данных-пользователей)
  - [Работа с API](#работа-с-api)
  - [Валидация форм](#валидация-форм)
  - [Обработка событий](#обработка-событий)
  - [Работа с конфигурацией](#работа-с-конфигурацией)
  - [Обработка файлов](#обработка-файлов)
  - [Функциональные компоненты React](#функциональные-компоненты-react)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Обработка данных пользователей

Композиция особенно полезна при обработке сложных структур данных, таких как пользовательская информация.

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

// Функции обработки
const parseId = (user: RawUser): RawUser & { id: number } => ({
  ...user,
  id: parseInt(user.id, 10)
});

const formatName = (user: RawUser & { id: number }): RawUser & { id: number, fullName: string } => ({
  ...user,
  fullName: `${user.firstName.trim()} ${user.lastName.trim()}`
});

const calculateAge = (user: RawUser & { id: number, fullName: string }): RawUser & { id: number, fullName: string, age: number } => ({
  ...user,
  age: new Date().getFullYear() - new Date(user.birthDate).getFullYear()
});

const parseSalary = (user: RawUser & { id: number, fullName: string, age: number }): RawUser & { id: number, fullName: string, age: number, salary: number } => ({
  ...user,
  salary: parseFloat(user.salary)
});

const parseActiveStatus = (user: RawUser & { id: number, fullName: string, age: number, salary: number }): RawUser & { id: number, fullName: string, age: number, salary: number, isActive: boolean } => ({
  ...user,
  isActive: user.isActive === "true"
});

const addRegistrationDate = (user: RawUser & { id: number, fullName: string, age: number, salary: number, isActive: boolean }): ProcessedUser => ({
  ...user,
  registrationDate: new Date()
});

// Композиция обработки данных пользователя
const processUser = pipe(
  parseId,
  formatName,
  calculateAge,
  parseSalary,
  parseActiveStatus,
  addRegistrationDate
);

// Обработка массива пользователей
const processUsers = (users: RawUser[]): ProcessedUser[] => 
  users.map(processUser);

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
  }
];

const processedUsers = processUsers(rawUsers);
console.log(processedUsers);
```

## Работа с API

Композиция полезна при создании клиентов API с предустановленной конфигурацией и обработкой ответов.

```typescript
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

// Утилиты для работы с API
const createApiRequest = (config: ApiConfig) => (request: ApiRequest): string => {
  const url = new URL(`${config.baseUrl}${request.endpoint}`);
  
  // Добавление query параметров
  if (request.queryParams) {
    Object.entries(request.queryParams).forEach(([key, value]) => {
      url.searchParams.append(key, value);
    });
  }
  
  // Формирование заголовков
  const headers = { ...config.defaultHeaders, ...request.headers };
  
  return `Запрос: ${request.method} ${url.toString()}\nЗаголовки: ${JSON.stringify(headers)}\nТело: ${JSON.stringify(request.body)}`;
};

const parseApiResponse = <T>(response: string): ApiResponse<T> => {
  // Симуляция парсинга ответа
  return {
    status: 200,
    data: {} as T,
    headers: {}
  };
};

const handleApiError = <T>(response: ApiResponse<T>): Either<string, T> => {
  if (response.status >= 400) {
    return left(`API ошибка: ${response.status}`);
  }
  return right(response.data);
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

// Композиция API вызовов
const fetchGithubUser = pipe(
  (username: string) => ({
    method: "GET",
    endpoint: `/users/${username}`
  }),
  createGithubApiClient,
  parseApiResponse,
  handleApiError
);

const fetchPosts = pipe(
  (userId: number) => ({
    method: "GET",
    endpoint: "/posts",
    queryParams: { userId: userId.toString() }
  }),
  createJsonPlaceholderApiClient,
  parseApiResponse,
  handleApiError
);

// Использование
console.log(fetchGithubUser("octocat"));
console.log(fetchPosts(1));
```

## Валидация форм

Композиция упрощает создание сложных валидаторов форм с множественными правилами.

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
  min("age")(18)
);

// Валидация формы
interface UserForm {
  name: string;
  email: string;
  age: number;
}

const validateUserForm = (form: UserForm): ValidationResult<UserForm> => {
  const errors: ValidationError[] = [
    ...validateName(form.name),
    ...validateEmail(form.email),
    ...validateAge(form.age)
  ];
  
  return errors.length > 0 ? left(errors) : right(form);
};

// Композиция валидации с обработкой результата
const processUserForm = pipe(
  validateUserForm,
  (result: ValidationResult<UserForm>) => 
    result._tag === "Left" 
      ? { success: false, errors: result.left } 
      : { success: true, data: result.right }
);

// Использование
const validForm: UserForm = {
  name: "Иван Иванов",
  email: "ivan@example.com",
  age: 25
};

const invalidForm: UserForm = {
  name: "И",
  email: "invalid-email",
  age: 15
};

console.log(processUserForm(validForm));
console.log(processUserForm(invalidForm));
```

## Обработка событий

Композиция полезна при создании систем обработки событий с различными фильтрами и трансформациями.

```typescript
// Модели событий
interface EventData {
  type: string;
  payload: any;
  timestamp: number;
  source: string;
}

interface ProcessedEvent {
  type: string;
  payload: any;
  timestamp: number;
  source: string;
  processedAt: number;
  priority: "low" | "medium" | "high";
}

// Обработчики событий
const filterByType = (type: string) => (event: EventData): EventData | null => 
  event.type === type ? event : null;

const addTimestamp = (event: EventData): EventData & { processedAt: number } => ({
  ...event,
  processedAt: Date.now()
});

const addPriority = (event: EventData & { processedAt: number }): ProcessedEvent => {
  let priority: "low" | "medium" | "high" = "low";
  
  if (event.type.startsWith("ERROR")) {
    priority = "high";
  } else if (event.type.startsWith("WARNING")) {
    priority = "medium";
  }
  
  return {
    ...event,
    priority
  };
};

const logEvent = (prefix: string) => (event: ProcessedEvent): ProcessedEvent => {
  console.log(`[${prefix}] ${event.type}:`, event.payload);
  return event;
};

// Создание специализированных обработчиков
const createEventHandler = (eventType: string, logPrefix: string) => 
  pipe(
    filterByType(eventType),
    (event: EventData | null) => event ? right(event) : left(`Событие типа ${eventType} не найдено`),
    (result: Either<string, EventData>) => 
      result._tag === "Right" 
        ? right(pipe(addTimestamp, addPriority, logEvent(logPrefix))(result.right))
        : left(result.left)
  );

// Специализированные обработчики
const handleErrorEvent = createEventHandler("ERROR", "ОШИБКА");
const handleWarningEvent = createEventHandler("WARNING", "ПРЕДУПРЕЖДЕНИЕ");
const handleInfoEvent = createEventHandler("INFO", "ИНФОРМАЦИЯ");

// Обработка массива событий
const processEvents = (events: EventData[]): ProcessedEvent[] => {
  return events
    .map(event => {
      // Пробуем обработать событие разными обработчиками
      const errorResult = handleErrorEvent(event);
      if (errorResult._tag === "Right") {
        return errorResult.right;
      }
      
      const warningResult = handleWarningEvent(event);
      if (warningResult._tag === "Right") {
        return warningResult.right;
      }
      
      const infoResult = handleInfoEvent(event);
      if (infoResult._tag === "Right") {
        return infoResult.right;
      }
      
      return null;
    })
    .filter((event): event is ProcessedEvent => event !== null);
};

// Использование
const events: EventData[] = [
  { type: "ERROR", payload: { message: "Ошибка подключения" }, timestamp: Date.now(), source: "database" },
  { type: "WARNING", payload: { message: "Низкий уровень памяти" }, timestamp: Date.now(), source: "system" },
  { type: "INFO", payload: { message: "Пользователь вошел в систему" }, timestamp: Date.now(), source: "auth" }
];

const processedEvents = processEvents(events);
console.log(processedEvents);
```

## Работа с конфигурацией

Композиция полезна при обработке и валидации конфигурационных данных.

```typescript
// Модели конфигурации
interface RawConfig {
  serverPort: string;
  databaseUrl: string;
  maxConnections: string;
  enableLogging: string;
  logLevel: string;
  apiKeys: string; // comma-separated
}

interface ProcessedConfig {
  serverPort: number;
  databaseUrl: string;
  maxConnections: number;
  enableLogging: boolean;
  logLevel: "debug" | "info" | "warn" | "error";
  apiKeys: string[];
}

// Функции обработки конфигурации
const parsePort = (config: RawConfig): RawConfig & { serverPort: number } => ({
  ...config,
  serverPort: parseInt(config.serverPort, 10)
});

const parseMaxConnections = (config: RawConfig & { serverPort: number }): RawConfig & { serverPort: number, maxConnections: number } => ({
  ...config,
  maxConnections: parseInt(config.maxConnections, 10)
});

const parseEnableLogging = (config: RawConfig & { serverPort: number, maxConnections: number }): RawConfig & { serverPort: number, maxConnections: number, enableLogging: boolean } => ({
  ...config,
  enableLogging: config.enableLogging.toLowerCase() === "true"
});

const parseApiKeys = (config: RawConfig & { serverPort: number, maxConnections: number, enableLogging: boolean }): RawConfig & { serverPort: number, maxConnections: number, enableLogging: boolean, apiKeys: string[] } => ({
  ...config,
  apiKeys: config.apiKeys.split(",").map(key => key.trim()).filter(key => key.length > 0)
});

const validateLogLevel = (config: RawConfig & { serverPort: number, maxConnections: number, enableLogging: boolean, apiKeys: string[] }): Either<string, ProcessedConfig> => {
  const validLevels = ["debug", "info", "warn", "error"] as const;
  const logLevel = config.logLevel as "debug" | "info" | "warn" | "error";
  
  if (!validLevels.includes(logLevel)) {
    return left(`Неверный уровень логирования: ${config.logLevel}`);
  }
  
  return right({
    ...config,
    logLevel
  });
};

// Композиция обработки конфигурации
const processConfig = (config: RawConfig): Either<string, ProcessedConfig> => {
  const parsedConfig = pipe(
    parsePort,
    parseMaxConnections,
    parseEnableLogging,
    parseApiKeys
  )(config);
  
  return validateLogLevel(parsedConfig);
};

// Использование
const rawConfig: RawConfig = {
  serverPort: "3000",
  databaseUrl: "postgresql://localhost:5432/mydb",
  maxConnections: "10",
  enableLogging: "true",
  logLevel: "info",
  apiKeys: "key1, key2, key3"
};

const result = processConfig(rawConfig);
if (result._tag === "Right") {
  console.log("Обработанная конфигурация:", result.right);
} else {
  console.log("Ошибка конфигурации:", result.left);
}
```

## Обработка файлов

Композиция полезна при обработке файлов с различными трансформациями.

```typescript
// Модели файлов
interface FileInfo {
  name: string;
  size: number;
  type: string;
  lastModified: number;
}

interface ProcessedFile {
  name: string;
  size: number;
  type: string;
  lastModified: number;
  extension: string;
  isLarge: boolean;
  category: string;
}

// Функции обработки файлов
const addExtension = (file: FileInfo): FileInfo & { extension: string } => ({
  ...file,
  extension: file.name.split(".").pop() || ""
});

const addSizeCategory = (file: FileInfo & { extension: string }): FileInfo & { extension: string, isLarge: boolean } => ({
  ...file,
  isLarge: file.size > 1024 * 1024 // больше 1MB
});

const addFileTypeCategory = (file: FileInfo & { extension: string, isLarge: boolean }): ProcessedFile => {
  let category = "other";
  
  const imageExtensions = ["jpg", "jpeg", "png", "gif", "svg"];
  const documentExtensions = ["pdf", "doc", "docx", "txt", "md"];
  const videoExtensions = ["mp4", "avi", "mov", "mkv"];
  
  if (imageExtensions.includes(file.extension.toLowerCase())) {
    category = "image";
  } else if (documentExtensions.includes(file.extension.toLowerCase())) {
    category = "document";
  } else if (videoExtensions.includes(file.extension.toLowerCase())) {
    category = "video";
  }
  
  return {
    ...file,
    category
  };
};

// Композиция обработки файлов
const processFile = pipe(
  addExtension,
  addSizeCategory,
  addFileTypeCategory
);

// Обработка массива файлов
const processFiles = (files: FileInfo[]): ProcessedFile[] => 
  files.map(processFile);

// Фильтрация и сортировка файлов
const filterByCategory = (category: string) => (files: ProcessedFile[]): ProcessedFile[] => 
  files.filter(file => file.category === category);

const filterLargeFiles = (files: ProcessedFile[]): ProcessedFile[] => 
  files.filter(file => file.isLarge);

const sortBySize = (files: ProcessedFile[]): ProcessedFile[] => 
  [...files].sort((a, b) => b.size - a.size);

const sortByName = (files: ProcessedFile[]): ProcessedFile[] => 
  [...files].sort((a, b) => a.name.localeCompare(b.name));

// Композиция операций над файлами
const getLargeImages = pipe(
  processFiles,
  filterByCategory("image"),
  filterLargeFiles,
  sortBySize
);

const getAllDocumentsSorted = pipe(
  processFiles,
  filterByCategory("document"),
  sortByName
);

// Использование
const files: FileInfo[] = [
  { name: "photo.jpg", size: 2048 * 1024, type: "image/jpeg", lastModified: Date.now() },
  { name: "document.pdf", size: 512 * 1024, type: "application/pdf", lastModified: Date.now() },
  { name: "video.mp4", size: 10 * 1024 * 1024, type: "video/mp4", lastModified: Date.now() },
  { name: "large-image.png", size: 5 * 1024 * 1024, type: "image/png", lastModified: Date.now() }
];

console.log("Большие изображения:", getLargeImages(files));
console.log("Все документы по имени:", getAllDocumentsSorted(files));
```

## Функциональные компоненты React

Композиция полезна при создании переиспользуемых компонентов в React.

```typescript
// Пример с функциональными компонентами React
interface ComponentProps {
  className?: string;
  style?: React.CSSProperties;
  children?: React.ReactNode;
}

// HOC (Higher-Order Components) с использованием композиции
const withClassName = (className: string) => 
  <T extends ComponentProps>(Component: React.ComponentType<T>) => 
  (props: Omit<T, "className">) => 
    <Component {...props as T} className={className} />;

const withStyles = (style: React.CSSProperties) => 
  <T extends ComponentProps>(Component: React.ComponentType<T>) => 
  (props: Omit<T, "style">) => 
    <Component {...props as T} style={{ ...style, ...props.style }} />;

const withLogging = (componentName: string) => 
  <T extends ComponentProps>(Component: React.ComponentType<T>) => 
  (props: T) => {
    console.log(`Рендер компонента ${componentName}:`, props);
    return <Component {...props} />;
  };

// Базовые компоненты
const Button: React.FC<ComponentProps & { onClick: () => void; text: string }> = 
({ className, style, onClick, text }) => (
  <button className={className} style={style} onClick={onClick}>
    {text}
  </button>
);

const Card: React.FC<ComponentProps> = 
({ className, style, children }) => (
  <div className={className} style={style}>
    {children}
  </div>
);

// Создание специализированных компонентов
const createStyledButton = pipe(
  withClassName("btn"),
  withStyles({ padding: "10px 20px", border: "none", borderRadius: "4px" }),
  withLogging("StyledButton")
);

const createPrimaryButton = pipe(
  createStyledButton,
  withClassName("btn btn-primary"),
  withStyles({ backgroundColor: "#007bff", color: "white" })
);

const createCard = pipe(
  withClassName("card"),
  withStyles({ padding: "20px", borderRadius: "8px", boxShadow: "0 2px 4px rgba(0,0,0,0.1)" }),
  withLogging("Card")
);

// Специализированные компоненты
const StyledButton = createStyledButton(Button);
const PrimaryButton = createPrimaryButton(Button);
const StyledCard = createCard(Card);

// Использование
const App: React.FC = () => (
  <StyledCard>
    <h2>Пример композиции компонентов</h2>
    <StyledButton onClick={() => console.log("Нажата кнопка")} text="Обычная кнопка" />
    <PrimaryButton onClick={() => console.log("Нажата первичная кнопка")} text="Первичная кнопка" />
  </StyledCard>
);
```

## Связи с другими концепциями

- [[ts/functional-programming/composition/Композиция_функций|Композиция функций]] - Основная страница раздела
- [[ts/functional-programming/composition/implementation|Реализация композиции]] - Различные способы реализации композиции функций
- [[ts/functional-programming/composition/data-flow|Поток данных]] - Управление потоком данных через композицию
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Техника создания функций с одним аргументом
- [[ts/react/components|React компоненты]] - Использование композиции в React
- [[ts/testing/index|Тестирование]] - Тестирование композиции функций

## Следующие шаги

- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация композиции функций
- [[ts/testing/index|Тестирование]] - Тестирование композиции функций
- [[ts/error-handling/index|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
- [[ts/react/React_с_TypeScript|TypeScript с React]] - Использование функциональных концепций в React