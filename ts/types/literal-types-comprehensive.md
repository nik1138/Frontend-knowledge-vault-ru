# Литеральные типы в TypeScript

Литеральные типы (literal types) - это типы, которые представляют собой конкретные значения, а не общие категории значений. Они позволяют указывать точные значения, которые может принимать переменная, что значительно повышает точность типизации и помогает избежать ошибок.

## Основные виды литеральных типов

### 1. Строковые литеральные типы

```typescript
// Переменная может принимать только конкретное значение
let direction: "up" = "up";
// direction = "down";  // Ошибка: "down" не может быть присвоено "up"

// Объединение строковых литералов
type Direction = "up" | "down" | "left" | "right";
let currentDirection: Direction = "up";  // OK
// currentDirection = "forward";  // Ошибка: "forward" не входит в объединение
```

### 2. Числовые литеральные типы

```typescript
type HTTPStatus = 200 | 201 | 400 | 401 | 404 | 500;
let status: HTTPStatus = 200;  // OK
// status = 300;  // Ошибка: 300 не входит в объединение
```

### 3. Булевы литеральные типы

```typescript
type YesNo = true | false;
let confirmed: YesNo = true;  // OK
// confirmed = "yes";  // Ошибка: "yes" не может быть присвоено true | false
```

## Практические применения

### 1. Создание ограниченных наборов значений

```typescript
// Определение допустимых значений для настроек
type Theme = "light" | "dark";
type Language = "en" | "es" | "fr" | "de";

interface UserPreferences {
  theme: Theme;
  language: Language;
  notifications: true | false;
}

const userPrefs: UserPreferences = {
  theme: "dark",      // OK
  language: "en",     // OK
  notifications: true // OK
};

// Такие ошибки будут пойманы на этапе компиляции:
// userPrefs.theme = "blue";     // Ошибка
// userPrefs.language = "jp";    // Ошибка
```

### 2. Типобезопасные функции с ограниченными параметрами

```typescript
function move(direction: "up" | "down" | "left" | "right"): void {
  console.log(`Moving ${direction}`);
}

move("up");    // OK
// move("forward");  // Ошибка: "forward" не входит в объединение

// Более сложный пример с возвращаемыми значениями
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

function fetchData(): Result<string> {
  // Симуляция получения данных
  if (Math.random() > 0.5) {
    return { success: true, data: "Данные успешно получены" };
  } else {
    return { success: false, error: "Не удалось получить данные" };
  }
}
```

### 3. Работа с API и внешними библиотеками

```typescript
// Типобезопасная работа с fetch API
type RequestMethod = "GET" | "POST" | "PUT" | "DELETE" | "PATCH";

interface RequestOptions {
  method: RequestMethod;
  url: string;
  headers?: Record<string, string>;
}

function request(options: RequestOptions): Promise<any> {
  return fetch(options.url, {
    method: options.method,
    headers: options.headers
  }).then(response => response.json());
}

// Использование
request({ method: "GET", url: "/api/users" });     // OK
// request({ method: "HEAD", url: "/api/users" }); // Ошибка: "HEAD" не входит в RequestMethod
```

## Комбинация с другими типами

### 1. С интерфейсами и типами объектов

```typescript
interface ApiResponse<T> {
  status: "success" | "error" | "loading";
  data?: T;
  error?: string;
}

type SuccessResponse<T> = ApiResponse<T> & { status: "success"; data: T };
type ErrorResponse = ApiResponse<any> & { status: "error"; error: string };
type LoadingResponse = ApiResponse<any> & { status: "loading" };

function handleResponse<T>(response: ApiResponse<T>): void {
  switch (response.status) {
    case "success":
      // response сужается до SuccessResponse<T>
      console.log("Данные:", response.data);
      break;
    case "error":
      // response сужается до ErrorResponse
      console.error("Ошибка:", response.error);
      break;
    case "loading":
      // response сужается до LoadingResponse
      console.log("Загрузка...");
      break;
  }
}
```

### 2. С массивами и кортежами

```typescript
// Массив с определенными строковыми значениями
type AllowedColors = "red" | "green" | "blue";
const colors: AllowedColors[] = ["red", "green", "blue"];  // OK
// const invalidColors: AllowedColors[] = ["red", "yellow"];  // Ошибка: "yellow" не входит в объединение

// Кортеж с литеральными типами
type Version = [1 | 2, number, number];  // [major, minor, patch]
const version1: Version = [1, 2, 3];     // OK
const version2: Version = [2, 10, 5];    // OK
// const version3: Version = [3, 0, 0];   // Ошибка: 3 не входит в объединение 1 | 2
```

## Продвинутые паттерны

### 1. Извлечение литеральных типов из значений

```typescript
// Использование typeof для получения литерального типа
const directions = ["up", "down", "left", "right"] as const;
type Direction = typeof directions[number];  // "up" | "down" | "left" | "right"

// Или из объекта
const config = {
  theme: "dark",
  lang: "en",
  notifications: true
} as const;

type ConfigKey = keyof typeof config;        // "theme" | "lang" | "notifications"
type ConfigValue = typeof config[keyof typeof config];  // "dark" | "en" | true
```

### 2. Комбинация с дженериками

```typescript
// Функция, которая работает только с определенными полями объекта
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Более сложный пример с литералами
type ValidKeys<T> = keyof T;
type ValidValues<T> = T[keyof T];

interface User {
  id: number;
  name: "Alice" | "Bob" | "Charlie";
  role: "admin" | "user" | "guest";
}

type UserName = User["name"];  // "Alice" | "Bob" | "Charlie"
type UserRole = User["role"];  // "admin" | "user" | "guest"
```

### 3. Создание типов на основе литералов

```typescript
// Создание типа на основе строкового литерала
type CreateEventName<T extends string> = `${T}Changed`;

type UserNameChanged = CreateEventName<"userName">;  // "userNameChanged"
type UserEmailChanged = CreateEventName<"userEmail">;  // "userEmailChanged"

// Комбинирование с объединениями
type ModelEvents = CreateEventName<"user" | "post" | "comment">;
// "userChanged" | "postChanged" | "commentChanged"
```

## Практические примеры из реальной разработки

### 1. Создание типобезопасной системы событий

```typescript
// Определение всех возможных событий
type AppEvent = 
  | { type: "USER_LOGIN"; userId: number }
  | { type: "USER_LOGOUT"; userId: number }
  | { type: "DATA_FETCHED"; data: any }
  | { type: "ERROR_OCCURRED"; message: string };

function handleEvent(event: AppEvent): void {
  switch (event.type) {
    case "USER_LOGIN":
      console.log(`Пользователь ${event.userId} вошёл`);
      break;
    case "USER_LOGOUT":
      console.log(`Пользователь ${event.userId} вышел`);
      break;
    case "DATA_FETCHED":
      console.log("Данные получены:", event.data);
      break;
    case "ERROR_OCCURRED":
      console.error("Ошибка:", event.message);
      break;
  }
}
```

### 2. Работа с настройками приложения

```typescript
interface AppSettings {
  theme: "light" | "dark" | "auto";
  language: "en" | "ru" | "es" | "fr";
  notifications: "enabled" | "disabled";
  autoSave: "on" | "off";
}

// Функция обновления настроек с типобезопасностью
function updateSettings<K extends keyof AppSettings>(
  settings: AppSettings,
  key: K,
  value: AppSettings[K]
): void {
  settings[key] = value;
}

const settings: AppSettings = {
  theme: "auto",
  language: "en",
  notifications: "enabled",
  autoSave: "on"
};

updateSettings(settings, "theme", "dark");      // OK
// updateSettings(settings, "theme", "blue");   // Ошибка: "blue" не входит в объединение
```

### 3. Создание типобезопасных конфигураций

```typescript
type Environment = "development" | "staging" | "production";

interface Config {
  environment: Environment;
  apiUrl: string;
  debug: boolean;
}

// Конфигурации для разных сред
const devConfig: Config = {
  environment: "development",
  apiUrl: "http://localhost:3000/api",
  debug: true
};

const prodConfig: Config = {
  environment: "production",
  apiUrl: "https://api.example.com",
  debug: false
};

// Функция валидации конфигурации
function validateConfig(config: Config): boolean {
  return config.environment !== undefined && 
         config.apiUrl.startsWith("http");
}
```

## Ограничения и особенности

### 1. Автоматическое расширение литералов

```typescript
// При инициализации переменной без явного типа литерал может быть расширен
const str = "hello";      // тип: string (не "hello")
const num = 42;           // тип: number (не 42)

// Для сохранения литерального типа используйте as const или явную аннотацию
const literalStr: "hello" = "hello";    // тип: "hello"
const readonlyObj = { name: "Alice" } as const;  // тип: { readonly name: "Alice"; }
```

### 2. Ограничения на числовые литералы

```typescript
// Числовые литералы работают с любыми числами
type SmallPrime = 2 | 3 | 5 | 7 | 11;
let prime: SmallPrime = 5;  // OK

// Но могут быть ограничены для больших наборов
// Слишком большое объединение числовых литералов может замедлить компиляцию
// type LargeSet = 1 | 2 | 3 | /* ... */ | 1000;  // Не рекомендуется
```

### 3. Совместимость с родительскими типами

```typescript
function processValue(value: string | number): void {
  console.log(value);
}

const literalValue: "hello" = "hello";
processValue(literalValue);  // OK: "hello" совместим с string

// Литеральные типы совместимы с их родительскими типами
// Но не наоборот без явного преобразования
```

## Лучшие практики

1. **Используйте литеральные типы для ограничения значений** - особенно полезно для настроек и конфигураций
2. **Комбинируйте с объединениями для создания ограниченных наборов** - создавайте перечисления значений
3. **Используйте `as const` для сохранения литеральных типов** - при работе с объектами и массивами
4. **Создавайте переиспользуемые типы для часто используемых значений** - определяйте общие литеральные типы в одном месте
5. **Документируйте сложные литеральные типы** - особенно когда они используются в API
6. **Избегайте чрезмерно больших объединений литералов** - это может замедлить компиляцию

## Связи с другими концепциями

- [[ts/advanced-types/template-literal-types|Типы-шаблонные литералы]] - позволяют создавать строковые литералы на основе других литералов
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - часто используют строковые литералы как дискриминанты
- [[ts/type-system/keyof-operator|Оператор keyof]] - возвращает литеральные типы ключей
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использовать литеральные типы в условиях
- [[ts/utility-types/index|Утилиты типов]] - многие утилиты работают с литеральными типами