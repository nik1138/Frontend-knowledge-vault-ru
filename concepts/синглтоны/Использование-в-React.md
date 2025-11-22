---
aliases: ["Singleton в React", "Синглтон React", "React Singleton"]
tags: [react, frontend, design-patterns]
---

# Использование паттерна Синглтон в React

## Обзор

Паттерн Синглтон в React может быть полезен для управления глобальным состоянием, кэширования данных, управления сессией и других задач, где необходим уникальный экземпляр. Однако его использование в React требует особого внимания к архитектуре и жизненному циклу компонентов.

## Основные области применения

### 1. Управление глобальным состоянием

```javascript
// stateManager.js
class StateManager {
    constructor() {
        if (StateManager.instance) {
            return StateManager.instance;
        }
        
        this.state = {};
        this.listeners = [];
        
        StateManager.instance = this;
        return this;
    }
    
    setState(newState) {
        this.state = { ...this.state, ...newState };
        this.notifyListeners();
    }
    
    getState() {
        return { ...this.state };
    }
    
    subscribe(callback) {
        this.listeners.push(callback);
        return () => {
            this.listeners = this.listeners.filter(listener => listener !== callback);
        };
    }
    
    notifyListeners() {
        this.listeners.forEach(callback => callback(this.state));
    }
}

export default new StateManager();
```

```jsx
// App.js
import React, { useState, useEffect } from 'react';
import stateManager from './stateManager';

function App() {
    const [appState, setAppState] = useState(stateManager.getState());
    
    useEffect(() => {
        const unsubscribe = stateManager.subscribe(setAppState);
        return unsubscribe;
    }, []);
    
    const updateUser = (user) => {
        stateManager.setState({ user });
    };
    
    return (
        <div>
            <h1>Текущий пользователь: {appState.user?.name || 'Гость'}</h1>
            <button onClick={() => updateUser({ name: 'John', id: 1 })}>
                Установить пользователя
            </button>
        </div>
    );
}

export default App;
```

### 2. Кэширование данных

```javascript
// cacheManager.js
class CacheManager {
    constructor() {
        if (CacheManager.instance) {
            return CacheManager.instance;
        }
        
        this.cache = new Map();
        this.timeouts = new Map();
        
        CacheManager.instance = this;
        return this;
    }
    
    set(key, value, ttl = 300000) { // 5 минут по умолчанию
        this.cache.set(key, value);
        
        // Автоматическое удаление по истечении времени
        if (this.timeouts.has(key)) {
            clearTimeout(this.timeouts.get(key));
        }
        
        const timeout = setTimeout(() => {
            this.cache.delete(key);
            this.timeouts.delete(key);
        }, ttl);
        
        this.timeouts.set(key, timeout);
    }
    
    get(key) {
        return this.cache.get(key);
    }
    
    has(key) {
        return this.cache.has(key);
    }
    
    clear() {
        this.cache.clear();
        this.timeouts.forEach(timeout => clearTimeout(timeout));
        this.timeouts.clear();
    }
}

export default new CacheManager();
```

```jsx
// UserProfile.js
import React, { useState, useEffect } from 'react';
import cacheManager from './cacheManager';

function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        const fetchUser = async () => {
            // Проверка кэша
            const cachedUser = cacheManager.get(`user_${userId}`);
            
            if (cachedUser) {
                setUser(cachedUser);
                setLoading(false);
                return;
            }
            
            try {
                // Загрузка данных
                const response = await fetch(`/api/users/${userId}`);
                const userData = await response.json();
                
                // Кэширование
                cacheManager.set(`user_${userId}`, userData);
                setUser(userData);
            } catch (error) {
                console.error('Ошибка загрузки пользователя:', error);
            } finally {
                setLoading(false);
            }
        };
        
        fetchUser();
    }, [userId]);
    
    if (loading) return <div>Загрузка...</div>;
    
    return (
        <div>
            <h2>{user?.name}</h2>
            <p>{user?.email}</p>
        </div>
    );
}

export default UserProfile;
```

### 3. Управление API клиентом

```javascript
// apiClient.js
class ApiClient {
    constructor() {
        if (ApiClient.instance) {
            return ApiClient.instance;
        }
        
        this.baseURL = process.env.REACT_APP_API_URL || 'http://localhost:3000';
        this.token = null;
        this.interceptors = [];
        
        ApiClient.instance = this;
        return this;
    }
    
    setToken(token) {
        this.token = token;
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        
        const config = {
            headers: {
                'Content-Type': 'application/json',
                ...(this.token && { 'Authorization': `Bearer ${this.token}` }),
                ...options.headers
            },
            ...options
        };
        
        try {
            const response = await fetch(url, config);
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.error('API ошибка:', error);
            throw error;
        }
    }
    
    get(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'GET' });
    }
    
    post(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    put(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    delete(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'DELETE' });
    }
}

export default new ApiClient();
```

## Использование с React Context

Синглтон может быть интегрирован с React Context для более удобного управления состоянием:

```jsx
// singletonContext.js
import React, { createContext, useContext } from 'react';
import stateManager from './stateManager';

const SingletonContext = createContext();

export const SingletonProvider = ({ children }) => {
    return (
        <SingletonContext.Provider value={stateManager}>
            {children}
        </SingletonContext.Provider>
    );
};

export const useSingleton = () => {
    const context = useContext(SingletonContext);
    if (!context) {
        throw new Error('useSingleton must be used within a SingletonProvider');
    }
    return context;
};
```

```jsx
// ComponentWithContext.js
import React from 'react';
import { useSingleton } from './singletonContext';

function ComponentWithContext() {
    const stateManager = useSingleton();
    const [state, setState] = React.useState(stateManager.getState());
    
    React.useEffect(() => {
        const unsubscribe = stateManager.subscribe(setState);
        return unsubscribe;
    }, [stateManager]);
    
    return (
        <div>
            <p>Состояние из синглтона: {JSON.stringify(state)}</p>
        </div>
    );
}

export default ComponentWithContext;
```

## Практические рекомендации

### 1. Использование в SSR (Server-Side Rendering)

При использовании синглтонов в SSR приложениях (например, с Next.js) важно учитывать, что синглтон будет общим для всех запросов на сервере:

```javascript
// serverSafeSingleton.js
let instance = null;

class ServerSafeSingleton {
    constructor() {
        // В браузере используем синглтон
        if (typeof window !== 'undefined') {
            if (instance) {
                return instance;
            }
            instance = this;
        }
        
        // На сервере создаем новый экземпляр для каждого запроса
        this.data = {};
    }
    
    setData(key, value) {
        this.data[key] = value;
    }
    
    getData(key) {
        return this.data[key];
    }
}

export default ServerSafeSingleton;
```

### 2. Тестирование синглтонов в React

Для тестирования компонентов, использующих синглтоны, рекомендуется использовать моки:

```javascript
// UserProfile.test.js
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import MockAdapter from 'axios-mock-adapter';
import UserProfile from './UserProfile';
import apiClient from './apiClient';

describe('UserProfile', () => {
    let mockApiClient;
    
    beforeEach(() => {
        // Мок синглтона
        mockApiClient = new MockAdapter(apiClient.axiosInstance);
    });
    
    afterEach(() => {
        mockApiClient.reset();
    });
    
    test('отображает данные пользователя', async () => {
        const userData = { id: 1, name: 'John', email: 'john@example.com' };
        mockApiClient.onGet('/api/users/1').reply(200, userData);
        
        render(<UserProfile userId={1} />);
        
        await waitFor(() => {
            expect(screen.getByText('John')).toBeInTheDocument();
        });
    });
});
```

## Альтернативы синглтону в React

### 1. Использование React Context

React Context может быть хорошей альтернативой для управления глобальным состоянием:

```jsx
// AppContext.js
import React, { createContext, useContext, useReducer } from 'react';

const AppContext = createContext();

const initialState = {
    user: null,
    theme: 'light'
};

function appReducer(state, action) {
    switch (action.type) {
        case 'SET_USER':
            return { ...state, user: action.payload };
        case 'SET_THEME':
            return { ...state, theme: action.payload };
        default:
            return state;
    }
}

export const AppProvider = ({ children }) => {
    const [state, dispatch] = useReducer(appReducer, initialState);
    
    return (
        <AppContext.Provider value={{ state, dispatch }}>
            {children}
        </AppContext.Provider>
    );
};

export const useAppContext = () => {
    const context = useContext(AppContext);
    if (!context) {
        throw new Error('useAppContext must be used within AppProvider');
    }
    return context;
};
```

### 2. Использование внешних библиотек

- [[Redux]] - полнофункциональное решение для управления состоянием
- [[MobX]] - реактивное управление состоянием
- [[Zustand]] - легковесное решение для управления состоянием

## Заключение

Паттерн Синглтон в React может быть полезным инструментом для управления глобальным состоянием, кэширования и других задач, требующих уникального экземпляра. Однако его использование требует осторожности из-за потенциальных проблем с тестируемостью и архитектурой.

В современных React-приложениях часто используются альтернативные решения, такие как React Context, Redux или Zustand, которые лучше интегрируются с экосистемой React и обеспечивают лучшую тестируемость.

## См. также

- [[Паттерн-синглтон]]
- [[Реализация-в-JavaScript]]
- [[Использование-в-Vue]]
- [[Проблемы-и-альтернативы]]
- [[React Context]]
- [[Redux]]
- [[Zustand]]