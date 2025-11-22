---
tags: [react, error-handling, best-practices, javascript]
aliases: [React Error Handling, Обработка ошибок в React]
---

# Лучшие практики обработки ошибок в React

## Введение

Обработка ошибок в React - это критический аспект разработки стабильных и надежных приложений. Эффективная система обработки ошибок позволяет избежать сбоев приложения и обеспечивает пользователя понятными сообщениями об ошибках.

## Error Boundaries (Граничные компоненты ошибок)

Error boundaries - это компоненты React, которые перехватывают JavaScript ошибки в дереве компонентов потомков, регистрируют эти ошибки и отображают резервный UI вместо падающего дерева компонентов.

### Реализация Error Boundary

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    // Обновляем состояние, чтобы следующий рендер показал запасной UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Записываем ошибку в систему логирования
    console.error('Error caught by boundary:', error, errorInfo);
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
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

## Методы Error Boundary

### getDerivedStateFromError

Этот метод вызывается после возникновения ошибки и позволяет обновить состояние компонента для отображения запасного UI.

```jsx
static getDerivedStateFromError(error) {
  // Возвращаем новое состояние для отображения запасного UI
  return { hasError: true };
}
```

### componentDidCatch

Этот метод вызывается после возникновения ошибки и позволяет записать информацию об ошибке в систему логирования.

```jsx
componentDidCatch(error, errorInfo) {
  // Логируем ошибку в систему отчетности
  logErrorToService(error, errorInfo);
}
```

## Стратегии обработки ошибок

### 1. Иерархическая обработка

Размещайте error boundaries на разных уровнях иерархии компонентов в зависимости от важности и критичности:

- **Корневой уровень**: Обрабатывает критические ошибки приложения
- **Роутинг**: Защищает отдельные страницы
- **Функциональные блоки**: Защищает отдельные виджеты или компоненты

### 2. Локальная обработка

Для менее критичных ошибок используйте локальные обработчики в компонентах:

```jsx
const MyComponent = () => {
  const [error, setError] = useState(null);

  const handleAsyncOperation = async () => {
    try {
      const data = await fetchData();
      // обработка данных
    } catch (error) {
      setError(error.message);
    }
  };

  if (error) {
    return <div>Произошла ошибка: {error}</div>;
  }

  return <div>Контент компонента</div>;
};
```

## Пользовательские компоненты ошибок

Создавайте переиспользуемые компоненты для отображения ошибок:

```jsx
const ErrorDisplay = ({ error, onRetry, children }) => {
  return (
    <div className="error-display">
      <h3>Произошла ошибка</h3>
      <p>{error?.message || 'Неизвестная ошибка'}</p>
      {children}
      {onRetry && (
        <button onClick={onRetry}>
          Повторить попытку
        </button>
      )}
    </div>
  );
};
```

## Глобальная обработка ошибок

Для обработки ошибок на уровне всего приложения:

```jsx
// ErrorBoundary.js
const AppErrorBoundary = ({ children }) => {
  return (
    <ErrorBoundary fallback={<GlobalErrorFallback />}>
      {children}
    </ErrorBoundary>
  );
};

// GlobalErrorFallback.js
const GlobalErrorFallback = ({ error, resetError }) => {
  return (
    <div role="alert">
      <h2>Ой! Что-то пошло не так.</h2>
      <pre>{error.message}</pre>
      <button onClick={resetError}>Попробовать снова</button>
    </div>
  );
};
```

## Обработка асинхронных ошибок

### В hooks

```jsx
const useAsyncError = () => {
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (asyncFunction) => {
    try {
      setError(null);
      return await asyncFunction();
    } catch (err) {
      setError(err);
      throw err;
    }
  }, []);

  return [error, execute];
};
```

### В асинхронных операциях

```jsx
const AsyncComponent = () => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch('/api/data');
        if (!response.ok) throw new Error('Network response was not ok');
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []);

  if (loading) return <div>Загрузка...</div>;
  if (error) return <ErrorDisplay error={error} />;
  return <div>{/* рендер данных */}</div>;
};
```

## Логирование и отчет об ошибках

Интеграция с системами отчетности об ошибках:

```jsx
// error-logger.js
const logError = (error, errorInfo = null) => {
  // Отправляем ошибку в систему мониторинга
  console.error('Logging error:', error, errorInfo);
  
  // Пример интеграции с Sentry
  if (window.Sentry) {
    window.Sentry.captureException(error, {
      contexts: errorInfo
    });
  }
};
```

## Паттерны восстановления после ошибок

### 1. Перезапуск компонента

```jsx
const ErrorRecoveryComponent = () => {
  const [error, setError] = useState(null);
  const [retryCount, setRetryCount] = useState(0);

  const resetComponent = () => {
    setError(null);
    setRetryCount(prev => prev + 1);
  };

  if (error) {
    return (
      <ErrorDisplay error={error} onRetry={resetComponent}>
        <p>Попытка восстановления: {retryCount}</p>
      </ErrorDisplay>
    );
  }

  return <ChildComponent key={retryCount} onError={setError} />;
};
```

### 2. Резервный контент

Предоставление запасного контента вместо полного отображения ошибки:

```jsx
const ComponentWithFallback = ({ data }) => {
  const [fallback, setFallback] = useState(false);

  if (fallback) {
    return <div>Показываем резервный контент...</div>;
  }

  try {
    return <ComplexComponent data={data} />;
  } catch (error) {
    setFallback(true);
    return <div>Показываем резервный контент...</div>;
  }
};
```

## Отображение дружелюбных сообщений об ошибках

Создавайте понятные сообщения для пользователей:

```jsx
const UserFriendlyError = ({ errorType }) => {
  const getErrorMessage = (type) => {
    switch (type) {
      case 'NETWORK_ERROR':
        return 'Не удалось подключиться к серверу. Проверьте подключение к интернету.';
      case 'DATA_ERROR':
        return 'Данные временно недоступны. Попробуйте позже.';
      case 'PERMISSION_ERROR':
        return 'У вас нет доступа к этой информации.';
      default:
        return 'Произошла ошибка. Пожалуйста, попробуйте снова.';
    }
  };

  return (
    <div className="user-friendly-error">
      <h3>Упс! Что-то пошло не так</h3>
      <p>{getErrorMessage(errorType)}</p>
      <button>Повторить попытку</button>
    </div>
  );
};
```

## Отладка ошибок в React

### Инструменты разработчика React

Используйте React DevTools для отладки ошибок:

- Проверка дерева компонентов
- Мониторинг состояния и пропсов
- Отслеживание жизненного цикла компонентов

### Логирование ошибок

```jsx
// utils/errorHandler.js
export const handleComponentError = (componentName, error, context = {}) => {
  console.group(`Ошибка в компоненте: ${componentName}`);
  console.error('Ошибка:', error);
  console.log('Контекст:', context);
  console.groupEnd();
};
```

## Обработка ошибок в хуках

### Custom Hook для обработки ошибок

```jsx
// hooks/useError.js
const useError = () => {
  const [error, setError] = useState(null);

  const executeWithErrorHandling = useCallback(async (asyncFunction) => {
    try {
      setError(null);
      return await asyncFunction();
    } catch (err) {
      setError(err);
      console.error('Ошибка в хуке:', err);
      return null;
    }
  }, []);

  const clearError = useCallback(() => setError(null), []);

  return { error, setError, executeWithErrorHandling, clearError };
};
```

## Тестирование состояний ошибок

Тестирование компонентов с ошибками:

```jsx
// tests/ErrorBoundary.test.js
import { render, screen } from '@testing-library/react';
import ErrorBoundary from '../ErrorBoundary';

const BrokenComponent = () => {
  throw new Error('Test error');
};

test('renders error boundary fallback when child throws error', () => {
  render(
    <ErrorBoundary>
      <BrokenComponent />
    </ErrorBoundary>
  );

  expect(screen.getByText(/что-то пошло не так/i)).toBeInTheDocument();
});
```

## Обработка ошибок в продакшене

### Минимизация влияния ошибок

- Используйте error boundaries для изоляции ошибок
- Реализуйте резервные пути для критических функций
- Настройте мониторинг и оповещения об ошибках

### Оптимизация для производительности

- Не создавайте лишние error boundaries
- Используйте lazy loading для изоляции потенциально проблемных компонентов

## Связи с другими файлами React

- [[react/hooks/best-practices]] - для понимания обработки ошибок в хуках
- [[react/state-management]] - для управления состоянием ошибок
- [[react/testing]] - для тестирования обработки ошибок
- [[react/performance]] - для оптимизации обработки ошибок
- [[Типизация контекста в React с TypeScript]] - для глобального управления ошибками через контекст

## Заключение

Эффективная обработка ошибок в React требует комбинации техник на разных уровнях приложения. Правильное использование error boundaries, пользовательских компонентов ошибок и стратегий восстановления позволяет создавать устойчивые и дружелюбные к пользователю приложения.

> [!important] 
> Всегда тестируйте состояния ошибок в своих компонентах, чтобы обеспечить надежную работу приложения в нештатных ситуациях.

> [!tip]
> Используйте систему логирования ошибок (например, Sentry, LogRocket) для мониторинга и анализа ошибок в продакшене.