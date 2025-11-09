# Обработка ошибок (Error Handling)

## Введение

Обработка ошибок — это критический аспект разработки программного обеспечения, который обеспечивает надежность, устойчивость и удобство использования приложений. Правильная обработка ошибок позволяет системе корректно реагировать на неожиданные ситуации и предотвращать сбои.

## Типы ошибок

### Синтаксические ошибки (Syntax Errors)
Ошибки, возникающие при нарушении правил языка программирования.

```javascript
// Неправильно
if (x = 5 { 
  console.log("Ошибка синтаксиса");
}

// Правильно
if (x === 5) {
  console.log("Правильный синтаксис");
}
```

### Ошибки выполнения (Runtime Errors)
Ошибки, возникающие во время выполнения программы.

```javascript
// Ошибка: деление на ноль
let result = 10 / 0; // Результат: Infinity, но может вызвать исключение в некоторых контекстах

// Ошибка: обращение к несуществующему свойству
let obj = null;
console.log(obj.property); // TypeError: Cannot read property 'property' of null
```

### Логические ошибки (Logic Errors)
Ошибки в логике программы, которые не вызывают исключений, но приводят к неправильным результатам.

```javascript
// Программа работает, но дает неправильный результат
function calculateAverage(numbers) {
  let sum = 0;
  for (let i = 0; i <= numbers.length; i++) { // Ошибка: i <= вместо i <
    sum += numbers[i];
  }
  return sum / numbers.length;
}
```

## Стратегии обработки ошибок

### Обнаружение и обработка
Системы должны быть способны обнаруживать ошибки и принимать соответствующие меры.

```javascript
function divide(a, b) {
  if (b === 0) {
    throw new Error("Деление на ноль невозможно");
  }
  return a / b;
}
```

### Использование обработчиков ошибок
Применение try-catch блоков для перехвата исключений.

```javascript
try {
  let result = divide(10, 0);
  console.log(result);
} catch (error) {
  console.error("Произошла ошибка:", error.message);
}
```

## Исключения против кодов возврата

### Исключения (Exceptions)
Механизм, при котором ошибка прерывает нормальный ход выполнения программы и передает управление обработчику ошибок.

```javascript
function parseJSON(str) {
  try {
    return JSON.parse(str);
  } catch (error) {
    throw new Error(`Невозможно разобрать JSON: ${error.message}`);
  }
}
```

### Коды возврата (Return Codes)
Функция возвращает специальное значение, указывающее на успех или ошибку.

```javascript
function divideWithCode(a, b) {
  if (b === 0) {
    return { success: false, error: "Деление на ноль" };
  }
  return { success: true, result: a / b };
}

const result = divideWithCode(10, 0);
if (!result.success) {
  console.error(result.error);
} else {
  console.log(result.result);
}
```

## Fail-Fast против плавного ухудшения (Graceful Degradation)

### Fail-Fast
Система немедленно реагирует на ошибку, прекращая выполнение.

```javascript
class Configuration {
  constructor(config) {
    if (!config.apiEndpoint) {
      throw new Error("Обязательное поле apiEndpoint отсутствует");
    }
    this.apiEndpoint = config.apiEndpoint;
  }
}
```

### Плавное ухудшение
Система продолжает работать, даже если часть функций недоступна.

```javascript
class ImageLoader {
  async loadImage(src) {
    try {
      const response = await fetch(src);
      if (!response.ok) {
        // Использовать запасное изображение
        return this.loadFallbackImage();
      }
      return response.blob();
    } catch (error) {
      return this.loadFallbackImage();
    }
  }
  
  loadFallbackImage() {
    // Загрузка запасного изображения
    return new Promise(resolve => {
      resolve('fallback-image.jpg');
    });
  }
}
```

## Распространение ошибок (Error Propagation)

Ошибки могут передаваться по цепочке вызовов функций.

```javascript
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP ошибка! Статус: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    // Пробрасываем ошибку выше
    throw error;
  }
}

async function displayUserProfile(userId) {
  try {
    const userData = await fetchUserData(userId);
    renderProfile(userData);
  } catch (error) {
    showErrorMessage(`Не удалось загрузить профиль пользователя: ${error.message}`);
  }
}
```

## Стратегии ведения логов (Logging Strategies)

### Уровни логирования
- **DEBUG** — подробная информация для диагностики
- **INFO** — общая информация о ходе выполнения
- **WARN** — потенциально проблемные ситуации
- **ERROR** — ошибки, не прерывающие работу приложения
- **FATAL** — критические ошибки, приводящие к остановке приложения

```javascript
class Logger {
  static log(level, message, data = null) {
    const timestamp = new Date().toISOString();
    const logEntry = {
      timestamp,
      level,
      message,
      ...(data && { data })
    };
    console.log(JSON.stringify(logEntry));
  }
  
  static error(message, error) {
    this.log('ERROR', message, {
      errorMessage: error.message,
      stack: error.stack
    });
  }
  
  static info(message) {
    this.log('INFO', message);
  }
}
```

## Техники отладки (Debugging Techniques)

### Использование точек останова (Breakpoints)
Пауза выполнения программы в определенных точках для анализа состояния.

### Логирование состояния
Вывод значений переменных и состояния приложения в определенные моменты.

```javascript
function calculateTotal(items) {
  Logger.debug('Начало расчета итоговой суммы', { itemCount: items.length });
  
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    Logger.debug(`Обработка элемента ${i}`, { item });
    total += item.price * item.quantity;
  }
  
  Logger.debug('Расчет завершен', { total });
  return total;
}
```

## Защитное программирование (Defensive Programming)

Практика написания кода, устойчивого к ошибкам и непредвиденным ситуациям.

```javascript
class BankAccount {
  constructor(initialBalance = 0) {
    if (typeof initialBalance !== 'number' || initialBalance < 0) {
      throw new Error('Начальный баланс должен быть неотрицательным числом');
    }
    this.balance = initialBalance;
  }
  
  withdraw(amount) {
    // Защитная проверка
    if (typeof amount !== 'number' || amount <= 0) {
      throw new Error('Сумма вывода должна быть положительным числом');
    }
    
    if (amount > this.balance) {
      throw new Error('Недостаточно средств на счете');
    }
    
    this.balance -= amount;
    return this.balance;
  }
  
  deposit(amount) {
    if (typeof amount !== 'number' || amount <= 0) {
      throw new Error('Сумма депозита должна быть положительным числом');
    }
    
    this.balance += amount;
    return this.balance;
  }
}
```

## Паттерны восстановления после ошибок

### Повторная попытка (Retry Pattern)
Повторное выполнение операции после ошибки.

```javascript
async function retryAsync(operation, maxRetries = 3, delay = 1000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (i === maxRetries - 1) {
        throw error; // Все попытки исчерпаны
      }
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Использование
const result = await retryAsync(async () => {
  const response = await fetch('/api/data');
  if (!response.ok) throw new Error('Network error');
  return response.json();
});
```

### Цепочка ответственности (Circuit Breaker)
Паттерн, предотвращающий повторные попытки выполнить операцию, которая, как известно, завершается неудачей.

```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.lastFailureTime = null;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }
  
  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Цепь разомкнута');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}
```

## Связь с другими концепциями

- [[logging]] — подробное ведение логов как часть стратегии обработки ошибок
- [[validation]] — проверка данных как форма защитного программирования
- [[testing]] — написание тестов для проверки обработки ошибок
- [[resilience]] — устойчивость системы к сбоям
- [[debugging]] — техники отладки при ошибках

## Заключение

Эффективная обработка ошибок требует комплексного подхода, включающего правильное обнаружение, логирование, восстановление и предотвращение ошибок. Хорошо спроектированная система обработки ошибок делает приложение более надежным и удобным для пользователей.

## Теги

#programming #error-handling #defensive-programming #debugging #resilience