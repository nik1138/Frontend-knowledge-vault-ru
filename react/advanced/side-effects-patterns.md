---
aliases: ["React Side Effects", "Side Effects Patterns"]
tags: 
  - #react
  - #side-effects
  - #useEffect
  - #patterns
  - #async
---

# Паттерны обработки побочных эффектов в React

## Введение

Побочные эффекты (side effects) в React — это операции, которые происходят вне "чистого" потока рендеринга компонента, такие как сетевые запросы, подписки, манипуляции с DOM, таймеры и т.д. Правильное управление побочными эффектами критично для производительности и надежности приложения.

## Что такое побочные эффекты в React

Побочные эффекты — это любые операции, которые влияют на внешнее состояние или имеют побочные результаты. В контексте React это:

- API вызовы
- Подписки на события
- Таймеры
- Манипуляции с DOM
- Логирование
- Локальное хранилище

> [!note] Важно
> Побочные эффекты не должны происходить во время рендеринга компонента, чтобы сохранить предсказуемость и детерминированность рендеринга.

## Побочные эффекты в функциональных vs классовых компонентах

### Классовые компоненты

В классовых компонентах побочные эффекты обрабатываются с помощью методов жизненного цикла:

```jsx
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = { user: null, loading: true };
  }

  componentDidMount() {
    this.fetchUser();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  componentWillUnmount() {
    // Очистка
    if (this.abortController) {
      this.abortController.abort();
    }
  }

  fetchUser = async () => {
    this.setState({ loading: true });
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const user = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ loading: false, error });
    }
  };

  render() {
    // ...
  }
}
```

### Функциональные компоненты

Функциональные компоненты используют хук `useEffect` для обработки побочных эффектов:

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isMounted = true;
    
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const userData = await response.json();
        
        if (isMounted) {
          setUser(userData);
          setLoading(false);
        }
      } catch (error) {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchUser();

    return () => {
      isMounted = false;
    };
  }, [userId]);

  // ...
}
```

## Паттерны useEffect

### Базовый useEffect

```jsx
useEffect(() => {
  document.title = `Пользователь: ${name}`;
}, [name]);
```

### Подписка на события

```jsx
useEffect(() => {
  const handleResize = () => {
    setWindowWidth(window.innerWidth);
  };

  window.addEventListener('resize', handleResize);

  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

### Один запуск (mount)

```jsx
useEffect(() => {
  // Выполняется только при монтировании
  analytics.track('page_view');
}, []);
```

## Пользовательские хуки для побочных эффектов

Создание пользовательских хуков позволяет инкапсулировать логику побочных эффектов:

```jsx
// hooks/useApi.js
import { useState, useEffect } from 'react';

function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, {
          signal: abortController.signal
        });
        
        if (!response.ok) throw new Error('Network response was not ok');
        
        const result = await response.json();
        if (!abortController.signal.aborted) {
          setData(result);
        }
      } catch (err) {
        if (!abortController.signal.aborted) {
          setError(err.message);
        }
      } finally {
        if (!abortController.signal.aborted) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => abortController.abort();
  }, [url]);

  return { data, loading, error };
}

export default useApi;
```

## Паттерны получения данных

### Простое получение данных

```jsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(response => response.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      })
      .catch(error => {
        console.error('Error fetching users:', error);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Загрузка...</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Получение данных с AbortController

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) return;

    const abortController = new AbortController();

    const search = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/search?q=${query}`, {
          signal: abortController.signal
        });
        const data = await response.json();
        
        if (!abortController.signal.aborted) {
          setResults(data);
          setLoading(false);
        }
      } catch (error) {
        if (error.name !== 'AbortError' && !abortController.signal.aborted) {
          console.error('Search error:', error);
          setLoading(false);
        }
      }
    };

    search();

    return () => abortController.abort();
  }, [query]);

  // ...
}
```

## Шаблоны очистки

Каждый побочный эффект должен иметь соответствующую логику очистки:

```jsx
useEffect(() => {
  const timer = setTimeout(() => {
    console.log('Timer completed');
  }, 1000);

  // Очистка таймера
  return () => clearTimeout(timer);
}, []);

useEffect(() => {
  const subscription = someExternalLibrary.subscribe(callback);
  
  // Отписка
  return () => subscription.unsubscribe();
}, []);
```

## Избегание распространенных ошибок

### Бесконечный ререндер

```jsx
// НЕПРАВИЛЬНО
useEffect(() => {
  setState(prev => prev + 1); // Бесконечный цикл!
}, [state]);

// ПРАВИЛЬНО
useEffect(() => {
  setState(prev => prev + 1);
}, []); // Убран state из зависимостей
```

### Утечки памяти

```jsx
// НЕПРАВИЛЬНО
useEffect(() => {
  fetch('/api/data').then(response => response.json()).then(setData);
}, []);

// ПРАВИЛЬНО
useEffect(() => {
  const abortController = new AbortController();
  
  fetch('/api/data', { signal: abortController.signal })
    .then(response => response.json())
    .then(setData);
    
  return () => abortController.abort();
}, []);
```

## Асинхронные операции в useEffect

```jsx
useEffect(() => {
  const fetchData = async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      setData(data);
    } catch (error) {
      setError(error);
    }
  };

  fetchData();
}, []);
```

## Обработка зависимостей в эффектах

```jsx
// Правильная обработка функций в зависимостях
const fetchData = useCallback(async () => {
  const response = await fetch(`/api/users/${userId}`);
  const data = await response.json();
  setData(data);
}, [userId]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

## Сложные обновления состояния

```jsx
useEffect(() => {
  const updateComplexState = async () => {
    const newState = await complexCalculation();
    setState(prevState => ({
      ...prevState,
      ...newState
    }));
  };

  updateComplexState();
}, [dependency]);
```

## Оптимизация побочных эффектов

- Использование мемоизации для предотвращения ненужных запусков
- Оптимизация зависимостей useEffect
- Использование пользовательских хуков для повторного использования логики

## Обработка ошибок в побочных эффектах

```jsx
useEffect(() => {
  const performOperation = async () => {
    try {
      setLoading(true);
      const result = await apiCall();
      setResult(result);
    } catch (error) {
      setError(error);
    } finally {
      setLoading(false);
    }
  };

  performOperation();
}, [dependency]);
```

## Паттерны отмены

```jsx
useEffect(() => {
  const abortController = new AbortController();

  const fetchData = async () => {
    try {
      const response = await fetch('/api/data', {
        signal: abortController.signal
      });
      const data = await response.json();
      setData(data);
    } catch (error) {
      if (error.name !== 'AbortError') {
        setError(error);
      }
    }
  };

  fetchData();

  return () => abortController.abort();
}, []);
```

## AbortController

AbortController позволяет отменять fetch-запросы:

```jsx
const [data, setData] = useState(null);
const abortControllerRef = useRef(null);

useEffect(() => {
  abortControllerRef.current = new AbortController();
  
  const fetchData = async () => {
    try {
      const response = await fetch('/api/data', {
        signal: abortControllerRef.current.signal
      });
      const result = await response.json();
      setData(result);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Fetch error:', error);
      }
    }
  };

  fetchData();

  return () => {
    abortControllerRef.current.abort();
  };
}, []);
```

## Побочные эффекты в Concurrent React

В Concurrent React важно учитывать, что эффекты могут выполняться несколько раз:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    // Это может выполняться несколько раз в Concurrent Mode
    console.log('Tick');
  }, 1000);

  return () => clearInterval(id);
}, []); // Убедитесь, что зависимости корректны
```

## Рассмотрение производительности

- Минимизация количества эффектов
- Оптимизация зависимостей
- Использование memo для предотвращения ненужных перезапусков
- Избегание тяжелых операций в эффектах

## Отладка побочных эффектов

- Использование React DevTools
- Логирование в useEffect
- Проверка зависимостей
- Использование ESLint-правил для useEffect

## Тестирование побочных эффектов

```jsx
// Пример теста с React Testing Library
import { render, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'Test User' }]));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('fetches and displays users', async () => {
  const { getByText } = render(<UserList />);
  
  await waitFor(() => {
    expect(getByText('Test User')).toBeInTheDocument();
  });
});
```

## Связи с другими файлами

- [[react/hooks/useEffect]]
- [[react/state-management]]
- [[react/error-boundaries]]
- [[react/performance]]
- [[react/testing]]
- [[js/async-await]]
- [[ts/generics]]
- [[architecture/api-integration]]

## Заключение

Правильное управление побочными эффектами критично для надежности и производительности React-приложений. Понимание паттернов и лучших практик позволяет создавать более предсказуемые и эффективные приложения.