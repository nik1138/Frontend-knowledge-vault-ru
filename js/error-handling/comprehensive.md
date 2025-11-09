---
aliases: ["Обработка ошибок в JavaScript", "Ошибки JS", "Error Handling"]
tags: 
  - #javascript
  - #error-handling
  - #programming
  - #best-practices
---

# Комплексное руководство по обработке ошибок в JavaScript

## Введение

Обработка ошибок в JavaScript — это критически важный аспект разработки, который позволяет создавать стабильные и надежные приложения. Правильная обработка ошибок помогает отлавливать проблемы на ранних стадиях, улучшает пользовательский опыт и облегчает отладку кода.

## Типы ошибок в JavaScript

JavaScript различает несколько типов ошибок:

- **Синтаксические ошибки** — ошибки, возникающие при нарушении синтаксиса языка
- **Временные ошибки выполнения** — ошибки, происходящие во время выполнения кода
- **Логические ошибки** — ошибки в логике программы, которые не вызывают исключений, но приводят к неправильному результату

```javascript
// Пример синтаксической ошибки
const invalidSyntax = { name: "example" // отсутствует закрывающая скобка

// Пример временной ошибки выполнения
const obj = null;
console.log(obj.property); // TypeError: Cannot read property 'property' of null
```

## Блоки try/catch/finally

Конструкция `try/catch/finally` позволяет перехватывать и обрабатывать ошибки в синхронном коде:

```javascript
try {
  // Код, который может вызвать ошибку
  const result = riskyOperation();
  console.log(result);
} catch (error) {
  // Обработка ошибки
  console.error('Произошла ошибка:', error.message);
} finally {
  // Код, который выполнится в любом случае
  console.log('Блок finally выполнен');
}
```

- [[js/try-catch-usage]] — подробнее о использовании try/catch
- [[js/exception-handling-patterns]] — паттерны обработки исключений

## Объекты ошибок и их свойства

Когда возникает ошибка, создается объект ошибки с определенными свойствами:

- `name` — имя ошибки (например, "Error", "TypeError", "ReferenceError")
- `message` — сообщение об ошибке
- `stack` — стек вызовов, который может помочь в отладке

```javascript
try {
  throw new Error("Пример пользовательской ошибки");
} catch (error) {
  console.log(error.name);    // "Error"
  console.log(error.message); // "Пример пользовательской ошибки"
  console.log(error.stack);   // Стек вызовов
}
```

## Пользовательские ошибки

Для более точной обработки ошибок можно создавать собственные классы ошибок:

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

try {
  throw new ValidationError("Неверный формат данных");
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Ошибка валидации:', error.message);
  }
}
```

- [[js/custom-error-classes]] — расширенное руководство по пользовательским ошибкам

## Распространение ошибок

Ошибки распространяются вверх по стеку вызовов до тех пор, пока не будут перехвачены блоком `catch` или не достигнут глобального обработчика:

```javascript
function level1() {
  level2();
}

function level2() {
  throw new Error("Ошибка в level2");
}

try {
  level1(); // Ошибка будет распространяться через level1 к блоку catch
} catch (error) {
  console.log("Ошибка перехвачена:", error.message);
}
```

## Асинхронная обработка ошибок

В асинхронном коде обработка ошибок требует специального подхода:

```javascript
// Обработка ошибок в Promise
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Ошибка:', error));

// Обработка ошибок с async/await
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка при получении данных:', error);
    throw error; // Передаем ошибку дальше при необходимости
  }
}
```

- [[js/async-error-handling]] — детальное руководство по асинхронной обработке ошибок
- [[js/promises-best-practices]] — лучшие практики работы с промисами

## Обработка ошибок в промисах

Промисы имеют встроенную систему обработки ошибок:

```javascript
// Обработка ошибок с использованием Promise.catch()
const promise = new Promise((resolve, reject) => {
  // Некоторая асинхронная операция
  reject(new Error("Ошибка в промисе"));
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error.message))
  .finally(() => console.log("Промис завершен"));
```

## Границы ошибок

Границы ошибок — это концепция, заимствованная из React, но применимая и в чистом JavaScript для изоляции ошибок:

```javascript
// Пример границы ошибок для асинхронных операций
class ErrorBoundary {
  constructor() {
    this.onError = null;
  }

  async executeAsyncOperation(asyncFn) {
    try {
      return await asyncFn();
    } catch (error) {
      if (this.onError) {
        this.onError(error);
      } else {
        console.error('Необработанная ошибка в асинхронной операции:', error);
      }
      throw error;
    }
  }
}
```

## Логирование и мониторинг ошибок

Для эффективного отслеживания ошибок в продакшене рекомендуется использовать системы логирования:

```javascript
// Пример системы логирования ошибок
class ErrorLogger {
  static log(error, context = {}) {
    const errorInfo = {
      message: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
      context,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    // Отправка в систему мониторинга
    console.error('Логированная ошибка:', errorInfo);
    
    // Пример отправки в удаленную систему:
    // sendToMonitoringService(errorInfo);
  }
}

// Использование
try {
  riskyOperation();
} catch (error) {
  ErrorLogger.log(error, { operation: 'riskyOperation', userId: 123 });
}
```

- [[js/logging-strategies]] — стратегии логирования в JavaScript
- [[js/monitoring-tools]] — инструменты мониторинга ошибок

## Отладка ошибок

Для отладки ошибок можно использовать следующие техники:

- Использование `console.error()` для вывода информации об ошибках
- Применение точек останова в отладчике браузера
- Использование `debugger` для остановки выполнения кода
- Анализ стека вызовов в объекте ошибки

```javascript
function debugExample() {
  try {
    problematicFunction();
  } catch (error) {
    console.error('Ошибка в debugExample:', error);
    console.error('Стек вызовов:', error.stack);
    debugger; // Установит точку останова в отладчике
  }
}
```

## Стратегии обработки ошибок

### 1. Логирование и продолжение работы

```javascript
function handleGracefully() {
  try {
    riskyOperation();
  } catch (error) {
    console.error('Ошибка обработана, выполнение продолжается:', error.message);
    // Приложение продолжает работу
  }
}
```

### 2. Восстановление после ошибки

```javascript
async function retryOnError(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error; // Последняя попытка
      console.log(`Попытка ${i + 1} не удалась, повтор...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1))); // Экспоненциальная задержка
    }
  }
}
```

### 3. Отказ и уведомление пользователя

```javascript
function criticalOperation() {
  try {
    return performCriticalTask();
  } catch (error) {
    showUserErrorNotification('Критическая ошибка: операция не может быть завершена');
    throw error; // Передаем ошибку выше для дальнейшей обработки
  }
}
```

## Заключение

Эффективная обработка ошибок в JavaScript требует понимания различных типов ошибок, умения использовать встроенные механизмы и разработки стратегий для конкретных сценариев. Правильная обработка ошибок делает приложения более надежными и улучшает пользовательский опыт.

## См. также

- [[js/async-programming]] — асинхронное программирование в JavaScript
- [[js/debugging-techniques]] — техники отладки в JavaScript
- [[js/error-types-reference]] — справочник по типам ошибок
- [[js/exception-safety]] — безопасность исключений в JavaScript