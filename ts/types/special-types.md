# Специальные типы в TypeScript

## Введение

Специальные типы в TypeScript предоставляют дополнительные возможности для моделирования различных ситуаций в системе типов. Эти типы позволяют более точно описывать поведение кода и обеспечивают большую гибкость при типизации.

## any

Тип `any` отключает проверку типов для переменной. Использование этого типа позволяет присваивать переменной значения любого типа.

```typescript
// Переменная типа any может принимать любые значения
let flexibleValue: any = 42;
flexibleValue = "Теперь строка";
flexibleValue = false;
flexibleValue = { name: "Объект" };
flexibleValue = [1, 2, 3];

// any также позволяет вызывать любые методы и обращаться к любым свойствам
flexibleValue.someMethod(); // Компилятор не проверяет существование метода
flexibleValue.nonExistentProperty; // Компилятор не проверяет существование свойства
```

> [!warning] Предупреждение
> Использование `any` отключает проверку типов и может привести к ошибкам во время выполнения. Рекомендуется избегать использования `any`, когда это возможно.

## unknown

Тип `unknown` является безопасной альтернативой `any`. Переменная типа `unknown` может принимать значения любого типа, но перед использованием требует проверку типа.

```typescript
let unknownValue: unknown = 42;
unknownValue = "Строка";
unknownValue = true;

// Нельзя использовать unknown значение напрямую
// console.log(unknownValue.length); // ОШИБКА: Тип "unknown" не может быть использован здесь

// Необходима проверка типа
if (typeof unknownValue === "string") {
  console.log(unknownValue.length); // OK: длина строки
}

// Или утверждение типа
const strLength = (unknownValue as string).length;
```

## never

Тип `never` представляет значения, которые никогда не происходят. Используется для функций, которые всегда выбрасывают ошибки или никогда не завершаются.

```typescript
// Функция, которая всегда выбрасывает ошибку
function throwError(message: string): never {
  throw new Error(message);
}

// Функция, которая никогда не завершается
function infiniteLoop(): never {
  while (true) {
    // Бесконечный цикл
  }
}

// В узлах ветвления, которые никогда не будут достигнуты
function checkValue(value: string | number): string {
  if (typeof value === "string") {
    return `Строка: ${value}`;
  } else if (typeof value === "number") {
    return `Число: ${value}`;
  } else {
    // Этот блок никогда не выполнится, так как value может быть только string или number
    const exhaustiveCheck: never = value;
    return exhaustiveCheck;
  }
}
```

## void

Тип `void` представляет отсутствие значения. Используется для функций, которые ничего не возвращают.

```typescript
// Функция, возвращающая void
function logMessage(message: string): void {
  console.log(message);
  // Неявно возвращает undefined
}

// Переменная типа void может содержать только undefined
let emptyValue: void;
emptyValue = undefined; // OK
// emptyValue = 123; // ОШИБКА: Тип "number" не может быть присвоен типу "void"

// Пример функции с возвращаемым типом void
function processItems(items: string[]): void {
  items.forEach(item => console.log(item));
}
```

## null и undefined

В TypeScript `null` и `undefined` имеют собственные типы, которые содержат только одно значение - `null` и `undefined` соответственно.

```typescript
// Типы null и undefined
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// В strict mode (strictNullChecks: true) эти типы не могут быть присвоены другим типам
let stringValue: string;
// stringValue = null; // ОШИБКА в strict mode
// stringValue = undefined; // ОШИБКА в strict mode

// Но можно использовать объединения типов
let nullableString: string | null = "значение";
nullableString = null; // OK

let optionalNumber: number | undefined;
optionalNumber = 42; // OK
optionalNumber = undefined; // OK
```

## Сравнение any, unknown и never

| Тип | Назначение | Использование |
|-----|------------|---------------|
| `any` | Отключение проверки типов | Когда тип неизвестен или не важен |
| `unknown` | Безопасная альтернатива any | Когда тип может быть любым, но требуется проверка |
| `never` | Значения, которые никогда не происходят | Функции, которые не завершаются или всегда выбрасывают ошибки |

## Практические примеры

### Использование unknown для безопасной типизации

```typescript
// Функция для безопасной обработки неизвестных данных
function safeParseJSON(json: string): unknown {
  try {
    return JSON.parse(json);
  } catch {
    return undefined;
  }
}

// Использование результата
const data = safeParseJSON('{"name": "Алексей"}');

// Проверка типа перед использованием
if (typeof data === "object" && data !== null && "name" in data) {
  const name = (data as { name: string }).name;
  console.log(`Имя: ${name}`);
}
```

### Использование never для обеспечения исчерпывающего сопоставления

```typescript
type Status = "success" | "error" | "loading";

function getStatusMessage(status: Status): string {
  switch (status) {
    case "success":
      return "Операция выполнена успешно";
    case "error":
      return "Произошла ошибка";
    case "loading":
      return "Загрузка...";
    default:
      // Если добавить новый статус в тип Status, 
      // это приведет к ошибке компиляции
      const exhaustiveCheck: never = status;
      return exhaustiveCheck;
  }
}
```

### Практическое применение void

```typescript
// Функция, которая только выполняет побочные эффекты
function updateUI(newState: { loading: boolean; data?: any }): void {
  if (newState.loading) {
    showLoadingSpinner();
  } else {
    hideLoadingSpinner();
    if (newState.data) {
      renderData(newState.data);
    }
  }
}

// Использование void для отбрасывания возвращаемого значения
function handleResult(result: Promise<any>): void {
  result
    .then(data => console.log(data))
    .catch(error => console.error(error));
  // Нам не нужен результат Promise, только побочные эффекты
}
```

## Связь с другими концепциями

- [[type-system/type-guards|Тип-гарды]] - проверка типов для безопасной работы с unknown
- [[types/union-intersection-types-comprehensive|Объединения и пересечения типов]] - комбинирование специальных типов с другими
- [[../type-system/type-compatibility|Совместимость типов]] - как специальные типы взаимодействуют с другими типами

## Заключение

Специальные типы в TypeScript предоставляют мощные возможности для моделирования различных ситуаций. Правильное использование этих типов помогает создавать более безопасный и надежный код, а также улучшает автодополнение и проверку типов.