---
aliases: ["React Error Handling Patterns", "Обработка ошибок в React"]
tags: 
  - #react
  - #error-handling
  - #javascript
  - #frontend
  - #best-practices
---

# Паттерны обработки ошибок в React

## Введение

Обработка ошибок в React — это важная часть разработки надежных приложений. React предоставляет несколько механизмов для перехвата и обработки ошибок на разных уровнях приложения.

## Механизм Error Boundaries

Error boundaries — это React-компоненты, которые перехватывают JavaScript ошибки в дереве компонентов потомков, регистрируют эти ошибки и отображают резервный UI вместо падающего дерева компонентов.

Error boundaries работают только для ошибок, возникающих в методах рендеринга, lifecycle методах и конструкторах компонентов ниже по дереву.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Обновить состояние, чтобы следующий рендер показал резервный UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Также можно зарегистрировать ошибку в службе отчетности об ошибках
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Можно отрендерить любой резервный UI
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}
```

## componentDidCatch и getDerivedStateFromError

### getDerivedStateFromError

Метод `getDerivedStateFromError(error)` вызывается во время фазы "render" и позволяет обновить состояние компонента, чтобы следующий рендер показал резервный UI. Этот метод вызывается перед коммитом фазы рендеринга, поэтому здесь нельзя использовать побочные эффекты.

```jsx
static getDerivedStateFromError(error) {
  // Обновить состояние, чтобы следующий рендер показал резервный UI
  return { hasError: true };
}
```

### componentDidCatch

Метод `componentDidCatch(error, errorInfo)` вызывается во время фазы "commit", поэтому здесь можно использовать побочные эффекты. Используется для регистрации ошибок:

```jsx
componentDidCatch(error, errorInfo) {
  // Зарегистрировать ошибку в службе отчетности
  logErrorToService(error, errorInfo);
}
```

## Обработка ошибок в асинхронном коде

Error boundaries **не перехватывают** ошибки в асинхронном коде, таком как:
- Обработчики событий
- setTimeout/setInterval
- Promise
- async/await

Для этих случаев нужно использовать try/catch:

```jsx
function MyComponent() {
  const [error, setError] = useState(null);

  const handleClick = async () => {
    try {
      await fetchData();
    } catch (err) {
      setError(err.message);
    }
  };

  if (error) {
    return <div>Ошибка: {error}</div>;
  }

  return <button onClick={handleClick}>Загрузить данные</button>;
}
```

## Error Boundaries vs try-catch

| Error Boundaries | try-catch |
|------------------|-----------|
| Работают в render методе | Не работают в render методе |
| Перехватывают ошибки в дереве потомков | Перехватывают ошибки в синхронном коде |
| Компоненты React | Язык JavaScript |
| Побочные эффекты в componentDidCatch | Немедленная обработка |

## Логирование и отчет об ошибках

Для эффективного отслеживания ошибок в продакшене рекомендуется интегрировать сервисы отчетности:

```jsx
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Отправить ошибку в сервис отчетности
    Sentry.captureException(error, {
      contexts: { react: errorInfo }
    });
  }
}
```

## Отображение резервного UI

При создании резервного UI важно:
- Обеспечить информативность для пользователя
- Предоставить пути восстановления
- Сохранить визуальную целостность приложения

```jsx
function FallbackUI({ error, resetError }) {
  return (
    <div className="error-fallback">
      <h2>Произошла ошибка</h2>
      <p>{error.message}</p>
      <button onClick={resetError}>Попробовать снова</button>
    </div>
  );
}
```

## Глобальные обработчики ошибок

Для обработки ошибок на уровне всего приложения можно использовать глобальные обработчики:

```jsx
// Обработка ошибок на уровне окна
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
});

window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
});
```

## Стратегии восстановления ошибок

### Сброс состояния компонента

```jsx
function ErrorBoundary({ children }) {
  const [hasError, setHasError] = useState(false);

  const resetError = () => {
    setHasError(false);
  };

  if (hasError) {
    return <FallbackUI resetError={resetError} />;
  }

  return children;
}
```

### Повторная попытка операции

```jsx
function withRetry(fetchFn, maxRetries = 3) {
  return async (...args) => {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fetchFn(...args);
      } catch (error) {
        if (i === maxRetries - 1) throw error;
      }
    }
  };
}
```

## Обработка ошибок с хуками

### Try/catch в эффектах

```jsx
useEffect(() => {
  const fetchData = async () => {
    try {
      const response = await api.getData();
      setData(response.data);
    } catch (error) {
      setError(error);
    }
  };

  fetchData();
}, []);
```

### Пользовательский хук для обработки ошибок

```jsx
function useErrorBoundary() {
  const [error, setError] = useState(null);

  const executeWithErrorHandling = async (asyncFn) => {
    try {
      setError(null);
      return await asyncFn();
    } catch (err) {
      setError(err);
      throw err;
    }
  };

  return { error, executeWithErrorHandling };
}
```

## Обработка Promise и async/await

### Обработка Promise

```jsx
const handlePromise = (promise) => {
  return promise
    .then(data => {
      // Обработка успешного результата
      return data;
    })
    .catch(error => {
      // Обработка ошибки
      console.error('Promise error:', error);
      throw error;
    });
};
```

### Обработка async/await

```jsx
const handleAsyncOperation = async () => {
  try {
    const result = await asyncOperation();
    return result;
  } catch (error) {
    // Обработка ошибки
    console.error('Async operation error:', error);
    throw error;
  }
};
```

## Структура дерева Error Boundaries

Рекомендуется размещать Error Boundaries на разных уровнях:
- Корневой уровень (для критических ошибок)
- Уровень страницы
- Уровень компонента
- Уровень виджета

```jsx
<App>
  <ErrorBoundary> {/* Корневая граница ошибок */}
    <Header />
    <ErrorBoundary> {/* Граница ошибок для основного контента */}
      <MainContent />
    </ErrorBoundary>
    <Footer />
  </ErrorBoundary>
</App>
```

## Обработка сетевых ошибок

```jsx
const handleNetworkError = (error) => {
  if (error.response) {
    // Сервер ответил с кодом, не входящим в диапазон 2xx
    return `Сервер вернул ошибку: ${error.response.status}`;
  } else if (error.request) {
    // Запрос был сделан, но ответ не получен
    return 'Сетевая ошибка: не удается подключиться к серверу';
  } else {
    // Что-то произошло при настройке запроса
    return `Ошибка запроса: ${error.message}`;
  }
};
```

## Обработка ошибок валидации

```jsx
function validateForm(formData) {
  const errors = {};

  if (!formData.email) {
    errors.email = 'Email обязателен';
  } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
    errors.email = 'Email недействителен';
  }

  if (!formData.password) {
    errors.password = 'Пароль обязателен';
  } else if (formData.password.length < 6) {
    errors.password = 'Пароль должен содержать не менее 6 символов';
  }

  return errors;
}
```

## Плавная деградация

Важно обеспечить, чтобы приложение продолжало работать даже при ошибках:

```jsx
function FeatureComponent({ fallback = <div>Функция временно недоступна</div> }) {
  const [isFeatureEnabled, setIsFeatureEnabled] = useState(true);

  if (!isFeatureEnabled) {
    return fallback;
  }

  return <div>Расширенная функция</div>;
}
```

## Интеграция с инструментами отчетности об ошибках

### Sentry

```jsx
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'YOUR_DSN',
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay(),
  ],
  tracesSampleRate: 1.0,
});

function App() {
  return (
    <Sentry.ErrorBoundary fallback={<div>Что-то пошло не так</div>}>
      <YourApp />
    </Sentry.ErrorBoundary>
  );
}
```

## Мониторинг ошибок в продакшене

Для эффективного мониторинга ошибок в продакшене:
- Используйте сервисы отчетности (Sentry, LogRocket, Bugsnag)
- Логируйте контекстные данные (пользователь, сессия, действия)
- Установите лимиты на количество отчетов (rate limiting)

## Обработка ошибок в разных окружениях

```jsx
const handleError = (error, context) => {
  // В продакшене - отправить в сервис отчетности
  if (process.env.NODE_ENV === 'production') {
    reportErrorToService(error, context);
  } 
  // В разработке - вывести в консоль
  else {
    console.error('Development error:', error, context);
  }
};
```

## Связи с другими файлами React

- [[react/hooks]] - для понимания обработки ошибок с хуками
- [[react/lifecycle]] - для понимания жизненного цикла компонентов
- [[react/state-management]] - для управления состоянием ошибок
- [[react/testing]] - для тестирования обработки ошибок
- [[react/performance]] - для понимания влияния обработки ошибок на производительность

## Заключение

Эффективная обработка ошибок в React требует комбинации Error Boundaries для ошибок рендеринга и try/catch для асинхронных операций. Важно предусмотреть стратегии восстановления и интеграцию с инструментами мониторинга для обеспечения надежности приложения.

## Теги

#react #error-handling #javascript #frontend #best-practices #debugging