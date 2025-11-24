---
aliases: ["Транспиляция TypeScript", "Компиляция TypeScript", "TS в HTML"]
tags: [programming, typescript, transpilation, html, frontend, javascript]
---

# TypeScript

## Общее описание

**TypeScript** — это надмножество JavaScript, которое добавляет статическую типизацию в язык. TypeScript требует транспиляции (компиляции) в JavaScript перед выполнением в браузере. Это делает его важным инструментом в разработке HTML-приложений, особенно в крупных проектах, где важна надежность и поддержка кода.

## Основные особенности

TypeScript предоставляет следующие возможности:

- Статическая типизация
- Интерфейсы и типы
- Обобщения (generics)
- Модули ES6+
- Декораторы (опционально)

## Установка и настройка

Установка TypeScript в проект:

```bash
npm install --save-dev typescript @types/node
```

Создание конфигурационного файла `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "sourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

## Примеры транспиляции

### Статическая типизация

**TypeScript код:**
```typescript
function greetUser(name: string, age: number): string {
  return `Привет, ${name}! Тебе ${age} лет.`;
}

const userName: string = "Алексей";
const userAge: number = 30;

console.log(greetUser(userName, userAge));
```

**Транспилированный JavaScript:**
```javascript
function greetUser(name, age) {
  return "Привет, " + name + "! Тебе " + age + " лет.";
}

var userName = "Алексей";
var userAge = 30;

console.log(greetUser(userName, userAge));
```

### Интерфейсы

**TypeScript код:**
```typescript
interface User {
  name: string;
  email: string;
  age: number;
}

function createUser(userData: User): User {
  return {
    name: userData.name,
    email: userData.email,
    age: userData.age
  };
}
```

**Транспилированный JavaScript:**
```javascript
function createUser(userData) {
  return {
    name: userData.name,
    email: userData.email,
    age: userData.age
  };
}
```

### Классы с типами

**TypeScript код:**
```typescript
class BankAccount {
  private balance: number = 0;

  constructor(initialBalance: number) {
    this.balance = initialBalance;
  }

  deposit(amount: number): void {
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

**Транспилированный JavaScript:**
```javascript
var BankAccount = /** @class */ (function () {
  function BankAccount(initialBalance) {
    this.balance = 0;
    this.balance = initialBalance;
  }

  BankAccount.prototype.deposit = function (amount) {
    this.balance += amount;
  };

  BankAccount.prototype.getBalance = function () {
    return this.balance;
  };

  return BankAccount;
}());
```

## Интеграция с HTML

Для использования TypeScript в HTML-проекте:

1. Создайте файлы с расширением `.ts`
2. Настройте компиляцию в JavaScript
3. Подключите результат в HTML

Пример HTML-файла:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Проект с TypeScript</title>
</head>
<body>
  <div id="app">
    <h1>Добро пожаловать в TypeScript!</h1>
  </div>
  <script src="dist/app.js"></script>
</body>
</html>
```

## Сборка и транспиляция

### Компиляция вручную

```bash
npx tsc
```

### Наблюдение за изменениями

```bash
npx tsc --watch
```

### Использование с Webpack

Установка необходимых зависимостей:

```bash
npm install --save-dev ts-loader webpack webpack-cli
```

Конфигурация Webpack (`webpack.config.js`):

```javascript
module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: __dirname + '/dist',
  },
};
```

## Преимущества использования TypeScript в HTML-проектах

> [!tip] Преимущества TypeScript
> - Снижение количества ошибок в runtime
> - Улучшенная автодополнение в редакторах кода
> - Лучшая документация кода через типы
> - Повышенная надежность крупных приложений
> - Упрощение рефакторинга

## Совместимость с браузерами

TypeScript позволяет настраивать целевую версию JavaScript:

```json
{
  "compilerOptions": {
    "target": "ES5", // Для поддержки старых браузеров
    "lib": ["ES2020", "DOM", "DOM.Iterable"]
  }
}
```

В российских условиях часто требуется поддержка IE11, что требует настройки `target: "ES5"`.

## Работа с DOM в TypeScript

TypeScript предоставляет строгую типизацию для работы с DOM:

```typescript
const button = document.getElementById('myButton') as HTMLButtonElement;
const input = document.querySelector('input[name="username"]') as HTMLInputElement;

button.addEventListener('click', () => {
  console.log(`Значение поля: ${input.value}`);
});
```

## Ошибки типизации

TypeScript позволяет обнаруживать ошибки на этапе компиляции:

```typescript
// Ошибка: Argument of type 'string' is not assignable to parameter of type 'number'
function addNumbers(a: number, b: number): number {
  return a + b;
}

addNumbers("5", "10"); // Ошибка будет обнаружена до выполнения
```

## Лучшие практики

1. **Используйте строгую типизацию** (`strict: true` в `tsconfig.json`)
2. **Создавайте интерфейсы для объектов** данных
3. **Используйте типы для функций** и их параметров
4. **Настройте правильные цели компиляции** в зависимости от поддерживаемых браузеров
5. **Используйте определения типов** (`@types`) для сторонних библиотек

## См. также

- [[Babel]] - инструмент транспиляции JavaScript
- [[Преобразование-кода]] - общие концепции транспиляции
- [[Полифилы]] - дополнительные инструменты для совместимости
- [[HTML]] - основы веб-разработки