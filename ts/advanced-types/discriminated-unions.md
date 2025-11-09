# Дискриминированные объединения (Discriminated Unions)

Дискриминированные объединения (также известные как тэгированные объединения или алгебраические типы данных) - это паттерн в TypeScript, который позволяет создавать типобезопасные объединения типов с помощью общего поля-дискриминатора.

## Основная концепция

Дискриминированное объединение состоит из:
1. Объединения типов
2. Общего поля-дискриминатора в каждом типе
3. Защитников типа (type guards) для безопасной работы с объединением

## Простой пример

```typescript
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "rectangle":
      return shape.width * shape.height;
    case "circle":
      return Math.PI * shape.radius ** 2;
  }
}
```

## Защитники типа

Дискриминированные объединения работают с защитниками типа:

```typescript
function isSquare(shape: Shape): shape is Square {
  return shape.kind === "square";
}

function getShapeInfo(shape: Shape): string {
  if (isSquare(shape)) {
    return `Square with size ${shape.size}`;
  }
  
  // TypeScript знает, что shape не является Square
  switch (shape.kind) {
    case "rectangle":
      return `Rectangle ${shape.width}x${shape.height}`;
    case "circle":
      return `Circle with radius ${shape.radius}`;
  }
}
```

## Продвинутые примеры

### 1. Состояния приложения

```typescript
interface LoadingState {
  status: "loading";
}

interface SuccessState {
  status: "success";
  data: string[];
}

interface ErrorState {
  status: "error";
  error: string;
}

type AppState = LoadingState | SuccessState | ErrorState;

function renderApp(state: AppState): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return `Data: ${state.data.join(", ")}`;
    case "error":
      return `Error: ${state.error}`;
  }
}
```

### 2. Команды в системе обработки сообщений

```typescript
interface SendMessageCommand {
  type: "send_message";
  recipient: string;
  content: string;
}

interface DeleteMessageCommand {
  type: "delete_message";
  messageId: string;
}

interface UpdateUserCommand {
  type: "update_user";
  userId: string;
  updates: Partial<User>;
}

type Command = 
  | SendMessageCommand 
  | DeleteMessageCommand 
  | UpdateUserCommand;

function processCommand(command: Command): void {
  switch (command.type) {
    case "send_message":
      // TypeScript знает, что command - это SendMessageCommand
      sendMessage(command.recipient, command.content);
      break;
    case "delete_message":
      deleteMessage(command.messageId);
      break;
    case "update_user":
      updateUser(command.userId, command.updates);
      break;
  }
}
```

### 3. AST (Abstract Syntax Tree) узлы

```typescript
interface NumberLiteral {
  type: "NumberLiteral";
  value: number;
}

interface StringLiteral {
  type: "StringLiteral";
  value: string;
}

interface BinaryExpression {
  type: "BinaryExpression";
  operator: "+" | "-" | "*" | "/";
  left: Expression;
  right: Expression;
}

interface CallExpression {
  type: "CallExpression";
  callee: string;
  arguments: Expression[];
}

type Expression = 
  | NumberLiteral
  | StringLiteral
  | BinaryExpression
  | CallExpression;

function evaluateExpression(expr: Expression): any {
  switch (expr.type) {
    case "NumberLiteral":
      return expr.value;
    case "StringLiteral":
      return expr.value;
    case "BinaryExpression":
      const left = evaluateExpression(expr.left);
      const right = evaluateExpression(expr.right);
      switch (expr.operator) {
        case "+": return left + right;
        case "-": return left - right;
        case "*": return left * right;
        case "/": return left / right;
      }
    case "CallExpression":
      // Обработка вызова функции
      return callFunction(
        expr.callee, 
        expr.arguments.map(evaluateExpression)
      );
  }
}
```

## Исчерпывающая проверка

TypeScript может проверять, что все возможные случаи обработаны:

```typescript
// Если забыть обработать один из случаев, TypeScript выдаст ошибку
function area(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "rectangle":
      return shape.width * shape.height;
    // case "circle": // Забыли обработать
    //   return Math.PI * shape.radius ** 2;
    default:
      // TypeScript выдаст ошибку, если не все случаи обработаны
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

## Практические примеры

### 1. Сетевые запросы

```typescript
interface LoadingAction {
  type: "LOADING";
}

interface SuccessAction<T> {
  type: "SUCCESS";
  payload: T;
}

interface ErrorAction {
  type: "ERROR";
  error: string;
}

type Action<T> = LoadingAction | SuccessAction<T> | ErrorAction;

function reducer<T>(
  state: AppState<T>, 
  action: Action<T>
): AppState<T> {
  switch (action.type) {
    case "LOADING":
      return { ...state, loading: true };
    case "SUCCESS":
      return { 
        loading: false, 
        data: action.payload, 
        error: null 
      };
    case "ERROR":
      return { 
        loading: false, 
        data: null, 
        error: action.error 
      };
  }
}
```

### 2. Конфигурация компонентов

```typescript
interface ButtonConfig {
  type: "button";
  text: string;
  onClick: () => void;
}

interface InputConfig {
  type: "input";
  placeholder: string;
  onChange: (value: string) => void;
}

interface SelectConfig {
  type: "select";
  options: { value: string; label: string }[];
  onSelect: (value: string) => void;
}

type ComponentConfig = ButtonConfig | InputConfig | SelectConfig;

function renderComponent(config: ComponentConfig): HTMLElement {
  switch (config.type) {
    case "button":
      const button = document.createElement("button");
      button.textContent = config.text;
      button.addEventListener("click", config.onClick);
      return button;
    case "input":
      const input = document.createElement("input");
      input.placeholder = config.placeholder;
      input.addEventListener("input", (e) => {
        config.onChange((e.target as HTMLInputElement).value);
      });
      return input;
    case "select":
      const select = document.createElement("select");
      config.options.forEach(option => {
        const optionElement = document.createElement("option");
        optionElement.value = option.value;
        optionElement.textContent = option.label;
        select.appendChild(optionElement);
      });
      select.addEventListener("change", (e) => {
        config.onSelect((e.target as HTMLSelectElement).value);
      });
      return select;
  }
}
```

### 3. События в системе

```typescript
interface UserLoginEvent {
  type: "user_login";
  userId: string;
  timestamp: Date;
}

interface UserLogoutEvent {
  type: "user_logout";
  userId: string;
  timestamp: Date;
}

interface PurchaseEvent {
  type: "purchase";
  userId: string;
  productId: string;
  amount: number;
  timestamp: Date;
}

type SystemEvent = UserLoginEvent | UserLogoutEvent | PurchaseEvent;

function processEvent(event: SystemEvent): void {
  switch (event.type) {
    case "user_login":
      logUserActivity(event.userId, "login");
      break;
    case "user_logout":
      logUserActivity(event.userId, "logout");
      break;
    case "purchase":
      recordPurchase(
        event.userId, 
        event.productId, 
        event.amount
      );
      break;
  }
}
```

## Паттерны и лучшие практики

### 1. Использование never для исчерпывающей проверки

```typescript
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}

function processShape(shape: Shape): string {
  switch (shape.kind) {
    case "square": return `Square: ${shape.size}`;
    case "rectangle": return `Rectangle: ${shape.width}x${shape.height}`;
    case "circle": return `Circle: ${shape.radius}`;
    default: return assertNever(shape); // Проверка на исчерпаемость
  }
}
```

### 2. Создание фабричных функций

```typescript
function createSquare(size: number): Square {
  return { kind: "square", size };
}

function createRectangle(width: number, height: number): Rectangle {
  return { kind: "rectangle", width, height };
}

function createCircle(radius: number): Circle {
  return { kind: "circle", radius };
}
```

## Ограничения и особенности

1. Дискриминатор должен быть литеральным типом (string literal, number literal, boolean)
2. Все типы в объединении должны иметь поле с одинаковым именем
3. TypeScript использует контроль потока для сужения типов
4. Не работает с вычисляемыми свойствами

## Связанные концепции

- [[ts/type-system/type-guards|Защитники типа]]
- [[ts/type-system/union-types|Объединения типов]]
- [[ts/type-system/control-flow-analysis|Анализ потока управления]]
- [[ts/advanced-types/conditional-types|Условные типы]]