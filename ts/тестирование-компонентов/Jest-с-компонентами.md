---
aliases: [Jest с компонентами, Тестирование компонентов Jest, Jest React]
tags: [testing, javascript, typescript, frontend, react, jest]
---

# Jest с компонентами - комплексное тестирование

Jest - это мощный фреймворк для тестирования JavaScript и TypeScript приложений, который особенно популярен для тестирования компонентов React. Он предоставляет все необходимые инструменты для написания юнит-тестов, интеграционных тестов и тестов компонентов.

## Обзор Jest для тестирования компонентов

Jest изначально разработан Facebook как фреймворк для тестирования JavaScript. Он включает в себя все необходимое для тестирования: раннер тестов, утверждения (assertions), моки (mocks) и покрытие кода. При тестировании компонентов Jest часто используется совместно с Testing Library или Enzyme.

## Установка Jest в проекте с компонентами

Для использования Jest с компонентами React и TypeScript установите необходимые зависимости:

```bash
npm install --save-dev jest @types/jest ts-jest @testing-library/react @testing-library/jest-dom
```

## Базовая настройка Jest

Создайте конфигурационный файл `jest.config.js`:

```javascript
module.exports = {
  testEnvironment: 'jsdom',
  preset: 'ts-jest',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
  ],
};
```

## Основные возможности Jest для тестирования компонентов

### Тестирование с помощью Testing Library

```typescript
// Counter.tsx
import React, { useState } from 'react';

interface CounterProps {
  initialValue?: number;
}

const Counter: React.FC<CounterProps> = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <div>
      <span data-testid="count">Count: {count}</span>
      <button onClick={increment} data-testid="increment-btn">+</button>
      <button onClick={decrement} data-testid="decrement-btn">-</button>
    </div>
  );
};

export default Counter;
```

```typescript
// Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

describe('Counter Component', () => {
  test('renders initial count correctly', () => {
    render(<Counter initialValue={5} />);
    expect(screen.getByTestId('count')).toHaveTextContent('Count: 5');
  });

  test('increments count when increment button is clicked', () => {
    render(<Counter />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const countElement = screen.getByTestId('count');

    fireEvent.click(incrementBtn);
    expect(countElement).toHaveTextContent('Count: 1');

    fireEvent.click(incrementBtn);
    expect(countElement).toHaveTextContent('Count: 2');
  });

  test('decrements count when decrement button is clicked', () => {
    render(<Counter initialValue={5} />);
    const decrementBtn = screen.getByTestId('decrement-btn');
    const countElement = screen.getByTestId('count');

    fireEvent.click(decrementBtn);
    expect(countElement).toHaveTextContent('Count: 4');
  });
});
```

## Мокирование зависимостей

Jest предоставляет мощные возможности для мокирования зависимостей:

```typescript
// ApiService.ts
export const fetchUserData = async (userId: string): Promise<any> => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
};
```

```typescript
// UserProfile.tsx
import React, { useState, useEffect } from 'react';
import { fetchUserData } from './ApiService';

interface UserProfileProps {
  userId: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  const [userData, setUserData] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadUserData = async () => {
      try {
        const data = await fetchUserData(userId);
        setUserData(data);
      } catch (error) {
        console.error('Failed to fetch user data:', error);
      } finally {
        setLoading(false);
      }
    };

    loadUserData();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (!userData) return <div>No user data</div>;

  return (
    <div>
      <h1>{userData.name}</h1>
      <p>{userData.email}</p>
    </div>
  );
};

export default UserProfile;
```

```typescript
// UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from './UserProfile';
import { fetchUserData } from './ApiService';

// Мокируем API сервис
jest.mock('./ApiService');

const mockFetchUserData = fetchUserData as jest.MockedFunction<typeof fetchUserData>;

describe('UserProfile Component', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('displays user data when loaded successfully', async () => {
    const mockUserData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    mockFetchUserData.mockResolvedValue(mockUserData);

    render(<UserProfile userId="123" />);

    expect(screen.getByText(/loading/i)).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });

    expect(mockFetchUserData).toHaveBeenCalledWith('123');
  });

  test('displays error state when API fails', async () => {
    mockFetchUserData.mockRejectedValue(new Error('API Error'));

    render(<UserProfile userId="123" />);

    await waitFor(() => {
      expect(screen.getByText(/no user data/i)).toBeInTheDocument();
    });
  });
});
```

## Тестирование событий и пользовательских взаимодействий

```typescript
// SearchForm.tsx
import React, { useState } from 'react';

interface SearchFormProps {
  onSearch: (query: string) => void;
}

const SearchForm: React.FC<SearchFormProps> = ({ onSearch }) => {
  const [query, setQuery] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
        data-testid="search-input"
      />
      <button type="submit" data-testid="search-button">
        Search
      </button>
    </form>
  );
};

export default SearchForm;
```

```typescript
// SearchForm.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import SearchForm from './SearchForm';

describe('SearchForm Component', () => {
  test('updates input value when typing', () => {
    render(<SearchForm onSearch={jest.fn()} />);
    
    const input = screen.getByTestId('search-input');
    fireEvent.change(input, { target: { value: 'test query' } });
    
    expect(input).toHaveValue('test query');
  });

  test('calls onSearch with correct query when form is submitted', () => {
    const mockOnSearch = jest.fn();
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const input = screen.getByTestId('search-input');
    const button = screen.getByTestId('search-button');
    
    fireEvent.change(input, { target: { value: 'react testing' } });
    fireEvent.click(button);
    
    expect(mockOnSearch).toHaveBeenCalledWith('react testing');
  });

  test('prevents default form submission', () => {
    const mockOnSearch = jest.fn();
    const preventDefault = jest.fn();
    
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const form = screen.getByRole('form');
    fireEvent.submit(form, { preventDefault });
    
    expect(preventDefault).toHaveBeenCalled();
  });
});
```

## Специфичные для Jest возможности

### Snapshot тестирование

```typescript
// Header.test.tsx
import { render } from '@testing-library/react';
import Header from './Header';

test('matches snapshot', () => {
  const { asFragment } = render(<Header title="Test App" />);
  expect(asFragment()).toMatchSnapshot();
});
```

### Mock функции

```typescript
test('calls callback with correct arguments', () => {
  const mockCallback = jest.fn();
  const component = render(<Button onClick={mockCallback}>Click me</Button>);
  
  const button = screen.getByRole('button', { name: /click me/i });
  fireEvent.click(button);
  
  expect(mockCallback).toHaveBeenCalledTimes(1);
  expect(mockCallback).toHaveBeenCalledWith();
});
```

### Таймеры

```typescript
// TimerComponent.tsx
import React, { useState, useEffect } from 'react';

const TimerComponent: React.FC = () => {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <div>Timer: {seconds}s</div>;
};
```

```typescript
// TimerComponent.test.tsx
import { render, screen } from '@testing-library/react';
import TimerComponent from './TimerComponent';

test('updates timer every second', () => {
  jest.useFakeTimers();
  
  render(<TimerComponent />);
  
  expect(screen.getByText('Timer: 0s')).toBeInTheDocument();
  
  // Пропускаем 1 секунду
  jest.advanceTimersByTime(1000);
  expect(screen.getByText('Timer: 1s')).toBeInTheDocument();
  
  // Пропускаем еще 2 секунды
  jest.advanceTimersByTime(2000);
  expect(screen.getByText('Timer: 3s')).toBeInTheDocument();
  
  jest.useRealTimers();
});
```

## Покрытие кода

Jest предоставляет встроенную поддержку покрытия кода:

```bash
npm test -- --coverage
```

## Расширенные паттерны тестирования

### Тестирование с контекстом

```typescript
// ThemeContext.test.tsx
import { render, screen } from '@testing-library/react';
import { ThemeProvider, useTheme } from './ThemeContext';

const TestComponent = () => {
  const { theme } = useTheme();
  return <div data-testid="theme">{theme}</div>;
};

test('provides theme context to child components', () => {
  render(
    <ThemeProvider initialTheme="dark">
      <TestComponent />
    </ThemeProvider>
  );
  
  expect(screen.getByTestId('theme')).toHaveTextContent('dark');
});
```

### Тестирование пользовательских хуков

```typescript
// useCounter.test.tsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('initializes with default value', () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0);
});

test('increments count', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

test('decrements count', () => {
  const { result } = renderHook(() => useCounter(5));
  
  act(() => {
    result.current.decrement();
  });
  
  expect(result.current.count).toBe(4);
});
```

## Заключение

Jest предоставляет мощный и гибкий фреймворк для тестирования компонентов. Его интеграция с Testing Library и другими инструментами делает его идеальным выбором для создания надежных и эффективных тестов компонентов в TypeScript приложениях.

См. также:
- [[Testing-Library]]
- [[Cypress-для-компонентов]]
- [[Snapshot-тестирование]]
- [[E2E-тестирование-компонентов]]