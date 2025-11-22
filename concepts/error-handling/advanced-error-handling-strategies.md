---
aliases: [Обработка ошибок, Стратегии ошибок, Ошибки в приложениях]
tags: [error-handling, architecture, frontend, resilience, backend]
---

# Продвинутые стратегии обработки ошибок в современных приложениях

## Введение

Обработка ошибок - одна из самых критических аспектов разработки программного обеспечения. Современные приложения должны быть устойчивыми к сбоям, предоставлять пользователям понятную информацию о проблемах и обеспечивать возможность диагностики и восстановления. Эта статья охватывает продвинутые стратегии обработки ошибок, которые позволяют строить более надежные и отказоустойчивые системы.

## Архитектурные принципы обработки ошибок

### Централизованная обработка ошибок

Центральный обработчик ошибок позволяет унифицировать логику обработки исключений по всему приложению. Вместо того чтобы дублировать код обработки ошибок в каждом модуле, мы создаем единую точку, через которую проходят все исключения.

```javascript
class ErrorHandler {
  constructor(logger, notifier) {
    this.logger = logger;
    this.notifier = notifier;
  }

  handle(error, context = {}) {
    // Логирование ошибки с контекстом
    this.logger.error(error, {
      ...context,
      stack: error.stack,
      timestamp: new Date().toISOString()
    });

    // Отправка уведомления в систему мониторинга
    this.notifier.report(error, context);

    // Определение типа ошибки и соответствующей реакции
    if (error instanceof ValidationError) {
      return this.handleValidationError(error);
    } else if (error instanceof NetworkError) {
      return this.handleNetworkError(error);
    } else {
      return this.handleGenericError(error);
    }
  }

  handleValidationError(error) {
    return {
      status: 400,
      message: 'Неверные данные запроса',
      details: error.validationErrors
    };
  }

  handleNetworkError(error) {
    return {
      status: 503,
      message: 'Сервис временно недоступен',
      retryAfter: 30
    };
  }

  handleGenericError(error) {
    return {
      status: 500,
      message: 'Внутренняя ошибка сервера'
    };
  }
}
```

### Иерархия ошибок

Создание иерархии пользовательских ошибок позволяет более точно определять типы проблем и применять соответствующие стратегии обработки:

```javascript
class AppError extends Error {
  constructor(message, code, status = 500) {
    super(message);
    this.name = this.constructor.name;
    this.code = code;
    this.status = status;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, field, value) {
    super(message, 'VALIDATION_ERROR', 400);
    this.field = field;
    this.value = value;
  }
}

class AuthenticationError extends AppError {
  constructor(message = 'Требуется аутентификация') {
    super(message, 'AUTH_ERROR', 401);
  }
}

class AuthorizationError extends AppError {
  constructor(message = 'Недостаточно прав для выполнения операции') {
    super(message, 'AUTHZ_ERROR', 403);
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`Ресурс ${resource} с ID ${id} не найден`, 'NOT_FOUND', 404);
    this.resource = resource;
    this.id = id;
  }
}
```

## Паттерны обработки ошибок

### Паттерн Circuit Breaker

Паттерн Circuit Breaker предотвращает повторные попытки выполнения операций, которые, как известно, завершаются неудачей. Это особенно полезно при работе с внешними сервисами:

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.timeout = options.timeout || 60000; // 60 секунд
    this.resetTimeout = options.resetTimeout || 30000; // 30 секунд
    
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.lastFailureTime = null;
  }

  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
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
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

### Паттерн Retry с экспоненциальной задержкой

При временных сбоях полезно повторять попытки с увеличивающейся задержкой:

```javascript
class RetryHandler {
  constructor(maxRetries = 3, baseDelay = 1000, maxDelay = 30000) {
    this.maxRetries = maxRetries;
    this.baseDelay = baseDelay;
    this.maxDelay = maxDelay;
  }

  async execute(fn, shouldRetry = () => true) {
    let lastError;
    
    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        return await fn(attempt);
      } catch (error) {
        lastError = error;
        
        if (attempt === this.maxRetries || !shouldRetry(error)) {
          break;
        }
        
        const delay = Math.min(
          this.baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
          this.maxDelay
        );
        
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError;
  }
}
```

## Обработка ошибок в асинхронных операциях

### Глобальный обработчик необработанных промисов

```javascript
// Для Node.js
process.on('unhandledRejection', (reason, promise) => {
  console.error('Необработанный промис:', reason);
  // Логирование ошибки и уведомление системы мониторинга
});

process.on('uncaughtException', (error) => {
  console.error('Необработанное исключение:', error);
  // Логирование ошибки и безопасное завершение процесса
  process.exit(1);
});
```

### Обработка ошибок в асинхронных компонентах React

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
    
    // Отправка ошибки в систему мониторинга
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Что-то пошло не так.</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Логирование ошибок

### Структурированное логирование

Для эффективного анализа ошибок важно логировать их в структурированном формате:

```javascript
const createLogger = (serviceName) => {
  return {
    error: (error, context = {}) => {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'ERROR',
        service: serviceName,
        message: error.message,
        stack: error.stack,
        ...context
      };
      
      console.error(JSON.stringify(logEntry));
      // Отправка в систему логирования (например, ELK, Splunk)
    },
    
    warn: (message, context = {}) => {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'WARN',
        service: serviceName,
        message,
        ...context
      };
      
      console.warn(JSON.stringify(logEntry));
    }
  };
};
```

### Контекстная информация в ошибках

Включение контекстной информации помогает быстрее диагностировать проблемы:

```javascript
const createUser = async (userData, requestContext) => {
  try {
    // Проверка данных
    validateUserData(userData);
    
    // Создание пользователя
    const user = await userRepository.create(userData);
    
    return user;
  } catch (error) {
    // Добавление контекста к ошибке
    error.context = {
      userId: requestContext.userId,
      requestId: requestContext.requestId,
      endpoint: requestContext.endpoint,
      timestamp: new Date().toISOString(),
      userData: {
        email: userData.email, // только безопасные данные
        hasPassword: !!userData.password
      }
    };
    
    throw error;
  }
};
```

## Мониторинг и алертинг

### Системы мониторинга ошибок

Интеграция с системами мониторинга позволяет оперативно реагировать на проблемы:

```javascript
class ErrorReporter {
  constructor(services = []) {
    this.services = services;
  }

  async report(error, context = {}) {
    // Отправка в несколько систем одновременно
    const promises = this.services.map(service => 
      service.report(error, context).catch(e => {
        console.error(`Ошибка при отправке в ${service.name}:`, e);
      })
    );
    
    await Promise.allSettled(promises);
  }
}

// Пример интеграции с Sentry
const sentryService = {
  name: 'Sentry',
  report: (error, context) => {
    // Интеграция с Sentry API
    return fetch('https://sentry.io/api/...', {
      method: 'POST',
      body: JSON.stringify({
        error: error.message,
        stacktrace: error.stack,
        context
      })
    });
  }
};
```

## Обработка ошибок в микросервисах

### Распределенные трассировки

Для отладки ошибок в распределенных системах важна трассировка запросов:

```javascript
class TraceableError extends AppError {
  constructor(message, code, traceId, spanId) {
    super(message, code);
    this.traceId = traceId;
    this.spanId = spanId;
  }
}

// Пример использования в микросервисе
const processOrder = async (orderData, traceContext) => {
  try {
    // Обработка заказа
    const result = await orderService.process(orderData);
    return result;
  } catch (error) {
    // Добавление трассировочной информации к ошибке
    if (!(error instanceof TraceableError)) {
      const traceableError = new TraceableError(
        error.message,
        'ORDER_PROCESSING_ERROR',
        traceContext.traceId,
        traceContext.spanId
      );
      traceableError.originalError = error;
      throw traceableError;
    }
    throw error;
  }
};
```

## Тестирование обработки ошибок

### Модульное тестирование ошибок

```javascript
describe('UserService', () => {
  it('должен выбросить ValidationError при неверных данных пользователя', async () => {
    const invalidUserData = { email: 'invalid-email' };
    
    await expect(
      userService.createUser(invalidUserData)
    ).rejects.toThrow(ValidationError);
  });

  it('должен корректно обрабатывать ошибки базы данных', async () => {
    // Мокаем ошибку базы данных
    userRepository.create = jest.fn().mockRejectedValue(
      new DatabaseError('Connection failed')
    );
    
    await expect(
      userService.createUser(validUserData)
    ).rejects.toThrow(AppError);
  });
});
```

## Заключение

Эффективная обработка ошибок - это не просто отлов исключений, а комплексный подход к построению надежных систем. Использование продвинутых стратегий, таких как Circuit Breaker, централизованная обработка ошибок, структурированное логирование и распределенная трассировка, позволяет создавать более устойчивые приложения, которые лучше справляются с непредвиденными ситуациями.

Правильная обработка ошибок улучшает пользовательский опыт, упрощает диагностику проблем и способствует более быстрому восстановлению сервисов после сбоев.

## См. также

- [[Error boundaries в React]]
- [[Логирование в приложениях]]
- [[Мониторинг производительности]]
- [[Архитектура микросервисов]]
- [[Тестирование приложений]]

## Ключевые теги

#error-handling #architecture #frontend #resilience #backend #testing #logging #monitoring #circuit-breaker #retry-pattern