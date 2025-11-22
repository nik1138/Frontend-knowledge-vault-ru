---
aliases: [Mocking, Mock Strategies, Мокирование]
tags: [testing, frontend, javascript]
---

# Стратегии мокирования в тестировании фронтенда

## Обзор

Мокирование (mocking) - это техника в тестировании, при которой реальные зависимости заменяются имитациями для изоляции тестируемого компонента. В фронтенд-разработке мокирование особенно важно для изоляции компонентов, сервисов и API-вызовов, позволяя тестировать отдельные части приложения независимо друг от друга.

## Типы мокирования

### 1. Мокирование функций (Function Mocking)

Замена реальных функций на имитации, которые можно контролировать в тестах.

```javascript
// Пример мокирования функции
import { render, fireEvent, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import UserProfile from './UserProfile';

// Мокирование API сервиса
const mockFetchUser = jest.fn();

jest.mock('../services/userService', () => ({
  fetchUser: mockFetchUser
}));

describe('UserProfile Component', () => {
  beforeEach(() => {
    mockFetchUser.mockClear(); // Очистка моков перед каждым тестом
  });

  test('вызывает fetchUser при монтировании', async () => {
    mockFetchUser.mockResolvedValue({
      id: 1,
      name: 'Иван Иванов',
      email: 'ivan@example.com'
    });

    render(<UserProfile userId={1} />);

    expect(mockFetchUser).toHaveBeenCalledWith(1);
    expect(screen.getByText('Загрузка...')).toBeInTheDocument();

    // Ждем завершения асинхронной операции
    expect(await screen.findByText('Иван Иванов')).toBeInTheDocument();
  });
});
```

### 2. Мокирование модулей (Module Mocking)

Полная замена модуля имитацией.

```javascript
// __mocks__/apiClient.js
export const apiClient = {
  get: jest.fn(),
  post: jest.fn(),
  put: jest.fn(),
  delete: jest.fn()
};

// В тесте
import { apiClient } from '../api/apiClient';

jest.mock('../api/apiClient');

describe('DataService', () => {
  test('должен получить данные', async () => {
    apiClient.get.mockResolvedValue({ data: 'test data' });

    const result = await DataService.fetchData();
    
    expect(apiClient.get).toHaveBeenCalledWith('/api/data');
    expect(result).toEqual({ data: 'test data' });
  });
});
```

### 3. Мокирование компонентов

Замена компонентов другими компонентами или элементами для изоляции тестируемого компонента.

```jsx
// UserProfile.jsx
import React from 'react';
import Avatar from './Avatar';
import ContactForm from './ContactForm';

const UserProfile = ({ user }) => (
  <div className="user-profile">
    <Avatar src={user.avatar} alt={user.name} />
    <h1>{user.name}</h1>
    <ContactForm userId={user.id} />
  </div>
);

export default UserProfile;
```

```jsx
// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';

// Мокирование дочерних компонентов
jest.mock('./Avatar', () => () => <div data-testid="mock-avatar">Mock Avatar</div>);
jest.mock('./ContactForm', () => () => <div data-testid="mock-contact-form">Mock Contact Form</div>);

describe('UserProfile', () => {
  test('рендерит имя пользователя и мокированные компоненты', () => {
    const user = {
      id: 1,
      name: 'Иван Иванов',
      avatar: 'avatar.jpg'
    };

    render(<UserProfile user={user} />);

    expect(screen.getByText('Иван Иванов')).toBeInTheDocument();
    expect(screen.getByTestId('mock-avatar')).toBeInTheDocument();
    expect(screen.getByTestId('mock-contact-form')).toBeInTheDocument();
  });
});
```

## Мокирование API и HTTP-запросов

### 1. Axios мокирование

```javascript
// api/users.js
import axios from 'axios';

export const userAPI = {
  getUsers: () => axios.get('/api/users'),
  getUserById: (id) => axios.get(`/api/users/${id}`),
  createUser: (userData) => axios.post('/api/users', userData),
  updateUser: (id, userData) => axios.put(`/api/users/${id}`, userData),
  deleteUser: (id) => axios.delete(`/api/users/${id}`)
};
```

```javascript
// users.test.js
import axios from 'axios';
import { userAPI } from './api/users';

jest.mock('axios');

describe('userAPI', () => {
  test('должен получить пользователей', async () => {
    const mockUsers = [
      { id: 1, name: 'Иван' },
      { id: 2, name: 'Мария' }
    ];

    axios.get.mockResolvedValue({ data: mockUsers });

    const users = await userAPI.getUsers();

    expect(axios.get).toHaveBeenCalledWith('/api/users');
    expect(users.data).toEqual(mockUsers);
  });

  test('должен создать пользователя', async () => {
    const newUser = { name: 'Алексей' };
    const createdUser = { id: 3, ...newUser };

    axios.post.mockResolvedValue({ data: createdUser });

    const result = await userAPI.createUser(newUser);

    expect(axios.post).toHaveBeenCalledWith('/api/users', newUser);
    expect(result.data).toEqual(createdUser);
  });
});
```

### 2. MSW (Mock Service Worker)

MSW предоставляет более реалистичное мокирование, работая на уровне сети.

```javascript
// src/mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: 1, name: 'Иван Иванов', email: 'ivan@example.com' },
        { id: 2, name: 'Мария Смирнова', email: 'maria@example.com' }
      ])
    );
  }),
  
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    return res(
      ctx.json({ id: Number(id), name: 'Тестовый пользователь', email: 'test@example.com' })
    );
  }),
  
  rest.post('/api/users', (req, res, ctx) => {
    const { name, email } = req.body;
    return res(
      ctx.status(201),
      ctx.json({ id: 3, name, email })
    );
  })
];
```

```javascript
// src/mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```javascript
// tests/UserList.test.js
import { render, screen, waitFor } from '@testing-library/react';
import { server } from '../mocks/server';
import UserList from '../components/UserList';

// Установка сервера перед всеми тестами
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserList', () => {
  test('отображает список пользователей', async () => {
    render(<UserList />);

    expect(screen.getByText('Загрузка...')).toBeInTheDocument();

    await waitFor(() => {
      expect(screen.getByText('Иван Иванов')).toBeInTheDocument();
      expect(screen.getByText('Мария Смирнова')).toBeInTheDocument();
    });
  });
});
```

## Мокирование браузерных API

### 1. localStorage

```javascript
// Мокирование localStorage
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
  clear: jest.fn(),
};

Object.defineProperty(window, 'localStorage', {
  value: localStorageMock,
});

// В тесте
import { saveToLocalStorage, getFromLocalStorage } from './storage';

describe('Storage utilities', () => {
  beforeEach(() => {
    localStorageMock.getItem.mockClear();
    localStorageMock.setItem.mockClear();
  });

  test('должен сохранить данные в localStorage', () => {
    saveToLocalStorage('test-key', { name: 'test' });

    expect(localStorageMock.setItem).toHaveBeenCalledWith(
      'test-key',
      JSON.stringify({ name: 'test' })
    );
  });

  test('должен получить данные из localStorage', () => {
    localStorageMock.getItem.mockReturnValue(JSON.stringify({ name: 'test' }));

    const result = getFromLocalStorage('test-key');

    expect(localStorageMock.getItem).toHaveBeenCalledWith('test-key');
    expect(result).toEqual({ name: 'test' });
  });
});
```

### 2. Geolocation API

```javascript
// Мокирование геолокации
Object.defineProperty(navigator, 'geolocation', {
  value: {
    getCurrentPosition: jest.fn().mockImplementation((success, error) => {
      success({
        coords: {
          latitude: 55.7558,
          longitude: 37.6173,
          accuracy: 10
        }
      });
    })
  }
});

// В тесте
test('должен получить текущую геолокацию', async () => {
  const position = await getCurrentLocation();
  
  expect(position.coords.latitude).toBe(55.7558);
  expect(position.coords.longitude).toBe(37.6173);
});
```

## Мокирование событий и асинхронных операций

### 1. setTimeout и setInterval

```javascript
// Мокирование таймеров
jest.useFakeTimers();

test('должен выполнить функцию через заданное время', () => {
  const mockCallback = jest.fn();
  delayedExecution(mockCallback, 5000);

  expect(mockCallback).not.toBeCalled();

  jest.advanceTimersByTime(5000);

  expect(mockCallback).toBeCalled();
});

// Возвращение к реальным таймерам
jest.useRealTimers();
```

### 2. Promise и async/await

```javascript
// Мокирование асинхронных операций
test('должен обработать успешный результат', async () => {
  const mockAsyncFunction = jest.fn().mockResolvedValue('успешный результат');

  const result = await handleAsyncOperation(mockAsyncFunction);

  expect(mockAsyncFunction).toHaveBeenCalled();
  expect(result).toBe('успешный результат');
});

test('должен обработать ошибку', async () => {
  const mockAsyncFunction = jest
    .fn()
    .mockRejectedValue(new Error('ошибка асинхронной операции'));

  await expect(handleAsyncOperation(mockAsyncFunction)).rejects.toThrow('ошибка асинхронной операции');
});
```

## Практические примеры

### Пример 1: Мокирование Redux-стора

```javascript
// store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { userAPI } from '../api/userAPI';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId) => {
    const response = await userAPI.getUserById(userId);
    return response.data;
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export default userSlice.reducer;
```

```javascript
// tests/userSlice.test.js
import userReducer, { fetchUser } from '../store/userSlice';

// Мокирование API
jest.mock('../api/userAPI', () => ({
  userAPI: {
    getUserById: jest.fn()
  }
}));

import { userAPI } from '../api/userAPI';

describe('userSlice', () => {
  test('должен обработать начальное состояние', () => {
    expect(userReducer(undefined, { type: '' })).toEqual({
      data: null,
      loading: false,
      error: null
    });
  });

  test('должен обработать pending состояние', () => {
    const action = { type: fetchUser.pending };
    const state = userReducer({ data: null, loading: false, error: null }, action);

    expect(state.loading).toBe(true);
    expect(state.error).toBeNull();
  });

  test('должен обработать fulfilled состояние', () => {
    const action = {
      type: fetchUser.fulfilled,
      payload: { id: 1, name: 'Иван Иванов' }
    };
    const state = userReducer({ data: null, loading: true, error: null }, action);

    expect(state.loading).toBe(false);
    expect(state.data).toEqual({ id: 1, name: 'Иван Иванов' });
  });
});
```

### Пример 2: Мокирование контекста React

```jsx
// ThemeContext.jsx
import React, { createContext, useContext } from 'react';

const ThemeContext = createContext();

export const ThemeProvider = ({ children, theme: initialTheme = 'light' }) => {
  const [theme, setTheme] = React.useState(initialTheme);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

```jsx
// tests/ThemeContext.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import { useTheme, ThemeProvider } from '../ThemeContext';

// Компонент для тестирования
const ThemeDisplay = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <div>
      <span data-testid="theme">{theme}</span>
      <button onClick={toggleTheme} data-testid="toggle">Переключить</button>
    </div>
  );
};

test('должен отображать начальную тему и переключать тему', () => {
  render(
    <ThemeProvider theme="light">
      <ThemeDisplay />
    </ThemeProvider>
  );

  expect(screen.getByTestId('theme')).toHaveTextContent('light');

  fireEvent.click(screen.getByTestId('toggle'));
  
  expect(screen.getByTestId('theme')).toHaveTextContent('dark');
});
```

## Лучшие практики мокирования

### 1. Мокируйте только то, что нужно

- Не мокируйте всё подряд
- Мокируйте только зависимости, которые:
  - Трудно контролировать (API, базы данных)
  - Медленные
  - Нестабильные
  - Зависят от внешних факторов

### 2. Используйте конкретные моки

- Создавайте моки, которые точно соответствуют реальному API
- Поддерживайте моки в актуальном состоянии
- Используйте типизированные моки, если используете TypeScript

### 3. Очищайте моки

```javascript
// Правильная очистка моков
afterEach(() => {
  jest.clearAllMocks();
});
```

### 4. Используйте реальные данные для моков

```javascript
// Использование реальных структур данных
const mockUserData = {
  id: 1,
  name: 'Иван Иванов',
  email: 'ivan@example.com',
  createdAt: '2023-01-01T00:00:00Z',
  isActive: true
};
```

## Заключение

Мокирование - мощный инструмент в тестировании фронтенд-приложений. Правильное использование моков позволяет изолировать тестируемые компоненты, ускорить выполнение тестов и контролировать внешние зависимости. Важно использовать моки осознанно, только когда это действительно необходимо, и поддерживать их в актуальном состоянии по мере развития приложения.

Связанные концепции: [[Test Coverage]], [[Testing Strategies]], [[Component Testing]], [[API Testing]]