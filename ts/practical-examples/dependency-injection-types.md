---
aliases: ["Примеры внедрения зависимостей с типами", "TS Dependency Injection Types"]
tags: [typescript, dependency-injection, patterns, architecture]
---

# Примеры внедрения зависимостей с типами

## Введение

Внедрение зависимостей (DI) - важный паттерн проектирования, который позволяет создавать более гибкие и тестируемые приложения. TypeScript предоставляет мощные возможности для типизации DI-контейнеров и зависимостей, что делает систему более надежной.

## Базовая архитектура DI

### Интерфейсы сервисов

```typescript
// Базовый интерфейс для сервисов
interface Service {
  readonly name: string;
}

// Интерфейс для логгера
interface Logger extends Service {
  log(message: string): void;
  error(message: string, error?: Error): void;
  warn(message: string): void;
  info(message: string): void;
}

// Интерфейс для API сервиса
interface ApiService extends Service {
  get<T>(url: string): Promise<T>;
  post<T, R = T>(url: string, data?: R): Promise<T>;
  put<T, R = T>(url: string, data?: R): Promise<T>;
  delete<T>(url: string): Promise<T>;
}

// Интерфейс для репозитория пользователей
interface UserRepository extends Service {
  findById(id: number): Promise<User | null>;
  findAll(): Promise<User[]>;
  create(user: Omit<User, 'id'>): Promise<User>;
  update(id: number, user: Partial<User>): Promise<User>;
  delete(id: number): Promise<boolean>;
}
```

### Базовый DI-контейнер

```typescript
// Тип для определения зависимости
type DependencyDefinition<T> = {
  useClass?: new (...args: any[]) => T;
  useValue?: T;
  useFactory?: (...args: any[]) => T;
  dependencies?: string[];
  singleton?: boolean;
};

// Типизированный DI-контейнер
class DIContainer {
  private services = new Map<string, DependencyDefinition<any>>();
  private instances = new Map<string, any>();
  
  register<T>(token: string, definition: DependencyDefinition<T>): void {
    this.services.set(token, definition);
  }
  
  resolve<T>(token: string): T {
    // Проверяем, существует ли уже инстанс для singleton
    if (this.instances.has(token)) {
      return this.instances.get(token);
    }
    
    const definition = this.services.get(token);
    if (!definition) {
      throw new Error(`Service ${token} is not registered`);
    }
    
    let instance: T;
    
    if (definition.useValue) {
      instance = definition.useValue;
    } else if (definition.useFactory) {
      const dependencies = definition.dependencies?.map(dep => this.resolve(dep)) || [];
      instance = definition.useFactory(...dependencies);
    } else if (definition.useClass) {
      const dependencies = definition.dependencies?.map(dep => this.resolve(dep)) || [];
      instance = new definition.useClass(...dependencies);
    } else {
      throw new Error(`Service ${token} has no valid definition`);
    }
    
    // Сохраняем инстанс для singleton-сервисов
    if (definition.singleton !== false) {
      this.instances.set(token, instance);
    }
    
    return instance;
  }
  
  reset(): void {
    this.instances.clear();
  }
}
```

## Продвинутые примеры

### Типизированный DI-контейнер с дженериками

```typescript
// Интерфейс для типизированного контейнера
interface Token<T> {
  readonly token: string;
  readonly type: T;
}

// Создание токена для типа
function createToken<T>(tokenName: string): Token<T> {
  return { token: tokenName, type: undefined as any };
}

// Улучшенный DI-контейнер с типами
class TypedDIContainer {
  private services = new Map<string, DependencyDefinition<any>>();
  private instances = new Map<string, any>();
  
  register<T>(token: Token<T>, definition: Omit<DependencyDefinition<T>, 'singleton'> & { singleton?: boolean }): void {
    this.services.set(token.token, { ...definition, singleton: definition.singleton !== false });
  }
  
  resolve<T>(token: Token<T>): T {
    const tokenKey = token.token;
    
    // Проверяем, существует ли уже инстанс для singleton
    if (this.instances.has(tokenKey)) {
      return this.instances.get(tokenKey);
    }
    
    const definition = this.services.get(tokenKey);
    if (!definition) {
      throw new Error(`Service ${tokenKey} is not registered`);
    }
    
    let instance: T;
    
    if (definition.useValue) {
      instance = definition.useValue;
    } else if (definition.useFactory) {
      const dependencies = definition.dependencies?.map(dep => this.resolveByToken(dep)) || [];
      instance = definition.useFactory(...dependencies);
    } else if (definition.useClass) {
      const dependencies = definition.dependencies?.map(dep => this.resolveByToken(dep)) || [];
      instance = new definition.useClass(...dependencies);
    } else {
      throw new Error(`Service ${tokenKey} has no valid definition`);
    }
    
    // Сохраняем инстанс для singleton-сервисов
    if (definition.singleton !== false) {
      this.instances.set(tokenKey, instance);
    }
    
    return instance;
  }
  
  private resolveByToken(token: string): any {
    return this.instances.get(token) || this.resolve({ token, type: undefined as any });
  }
  
  reset(): void {
    this.instances.clear();
  }
}

// Токены для сервисов
const LOGGER_TOKEN = createToken<Logger>('logger');
const API_SERVICE_TOKEN = createToken<ApiService>('apiService');
const USER_REPOSITORY_TOKEN = createToken<UserRepository>('userRepository');
```

### Реализация сервисов

```typescript
// Реализация логгера
class ConsoleLogger implements Logger {
  readonly name = 'ConsoleLogger';
  
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
  
  error(message: string, error?: Error): void {
    console.error(`[ERROR] ${message}`, error);
  }
  
  warn(message: string): void {
    console.warn(`[WARN] ${message}`);
  }
  
  info(message: string): void {
    console.info(`[INFO] ${message}`);
  }
}

// Реализация API сервиса
class HttpApiService implements ApiService {
  readonly name = 'HttpApiService';
  
  constructor(private baseUrl: string, private logger: Logger) {}
  
  async get<T>(url: string): Promise<T> {
    this.logger.info(`GET request to ${this.baseUrl}${url}`);
    const response = await fetch(`${this.baseUrl}${url}`);
    return response.json();
  }
  
  async post<T, R = T>(url: string, data?: R): Promise<T> {
    this.logger.info(`POST request to ${this.baseUrl}${url}`);
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
  
  async put<T, R = T>(url: string, data?: R): Promise<T> {
    this.logger.info(`PUT request to ${this.baseUrl}${url}`);
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
  
  async delete<T>(url: string): Promise<T> {
    this.logger.info(`DELETE request to ${this.baseUrl}${url}`);
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'DELETE'
    });
    return response.json();
  }
}

// Реализация репозитория пользователей
class HttpUserRepository implements UserRepository {
  readonly name = 'HttpUserRepository';
  
  constructor(private apiService: ApiService) {}
  
  async findById(id: number): Promise<User | null> {
    try {
      return await this.apiService.get<User>(`/users/${id}`);
    } catch {
      return null;
    }
  }
  
  async findAll(): Promise<User[]> {
    return await this.apiService.get<User[]>('/users');
  }
  
  async create(user: Omit<User, 'id'>): Promise<User> {
    return await this.apiService.post<User>('/users', user);
  }
  
  async update(id: number, user: Partial<User>): Promise<User> {
    return await this.apiService.put<User>(`/users/${id}`, user);
  }
  
  async delete(id: number): Promise<boolean> {
    try {
      await this.apiService.delete<any>(`/users/${id}`);
      return true;
    } catch {
      return false;
    }
  }
}
```

## Практические примеры

### Настройка контейнера

```typescript
// Функция для настройки контейнера
function setupContainer(container: TypedDIContainer): void {
  // Регистрируем логгер
  container.register(LOGGER_TOKEN, {
    useClass: ConsoleLogger,
    singleton: true
  });
  
  // Регистрируем API сервис с зависимостями
  container.register(API_SERVICE_TOKEN, {
    useFactory: (logger: Logger) => new HttpApiService('https://api.example.com', logger),
    dependencies: [LOGGER_TOKEN.token],
    singleton: true
  });
  
  // Регистрируем репозиторий пользователей
  container.register(USER_REPOSITORY_TOKEN, {
    useFactory: (apiService: ApiService) => new HttpUserRepository(apiService),
    dependencies: [API_SERVICE_TOKEN.token],
    singleton: true
  });
}

// Использование контейнера
const container = new TypedDIContainer();
setupContainer(container);

const userRepository = container.resolve(USER_REPOSITORY_TOKEN);
```

### Сервис бизнес-логики

```typescript
// Сервис для управления пользователями
interface UserService extends Service {
  getUserProfile(userId: number): Promise<UserProfile | null>;
  updateUserProfile(userId: number, profile: Partial<UserProfile>): Promise<UserProfile>;
  deleteUser(userId: number): Promise<boolean>;
}

interface UserProfile {
  id: number;
  name: string;
  email: string;
  preferences: UserPreferences;
}

interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notificationsEnabled: boolean;
}

class DefaultUserService implements UserService {
  readonly name = 'DefaultUserService';
  
  constructor(
    private userRepository: UserRepository,
    private logger: Logger
  ) {}
  
  async getUserProfile(userId: number): Promise<UserProfile | null> {
    try {
      this.logger.info(`Fetching user profile for ID: ${userId}`);
      const user = await this.userRepository.findById(userId);
      
      if (!user) {
        return null;
      }
      
      // В реальном приложении здесь может быть дополнительная логика
      return {
        id: user.id,
        name: user.name,
        email: user.email,
        preferences: {
          theme: 'light',
          language: 'en',
          notificationsEnabled: true
        }
      };
    } catch (error) {
      this.logger.error(`Failed to fetch user profile for ID: ${userId}`, error as Error);
      throw error;
    }
  }
  
  async updateUserProfile(userId: number, profile: Partial<UserProfile>): Promise<UserProfile> {
    try {
      this.logger.info(`Updating user profile for ID: ${userId}`);
      
      // Получаем текущего пользователя
      const existingUser = await this.userRepository.findById(userId);
      if (!existingUser) {
        throw new Error(`User with ID ${userId} not found`);
      }
      
      // Обновляем пользователя
      const updatedUser = await this.userRepository.update(userId, {
        name: profile.name || existingUser.name,
        email: profile.email || existingUser.email
      });
      
      // Здесь может быть дополнительная логика обновления профиля
      
      return {
        id: updatedUser.id,
        name: updatedUser.name,
        email: updatedUser.email,
        preferences: profile.preferences || {
          theme: 'light',
          language: 'en',
          notificationsEnabled: true
        }
      };
    } catch (error) {
      this.logger.error(`Failed to update user profile for ID: ${userId}`, error as Error);
      throw error;
    }
  }
  
  async deleteUser(userId: number): Promise<boolean> {
    try {
      this.logger.info(`Deleting user with ID: ${userId}`);
      return await this.userRepository.delete(userId);
    } catch (error) {
      this.logger.error(`Failed to delete user with ID: ${userId}`, error as Error);
      throw error;
    }
  }
}

// Токен для сервиса пользователей
const USER_SERVICE_TOKEN = createToken<UserService>('userService');

// Регистрация сервиса пользователей
function registerUserService(container: TypedDIContainer): void {
  container.register(USER_SERVICE_TOKEN, {
    useFactory: (userRepository: UserRepository, logger: Logger) => 
      new DefaultUserService(userRepository, logger),
    dependencies: [USER_REPOSITORY_TOKEN.token, LOGGER_TOKEN.token],
    singleton: true
  });
}
```

### DI для компонентов фреймворка

```typescript
// Пример интеграции с React
import { createContext, useContext, ReactNode } from 'react';

// Контекст для DI в React
const DIContext = createContext<TypedDIContainer | null>(null);

interface DIProviderProps {
  container: TypedDIContainer;
  children: ReactNode;
}

export const DIProvider: React.FC<DIProviderProps> = ({ container, children }) => {
  return (
    <DIContext.Provider value={container}>
      {children}
    </DIContext.Provider>
  );
};

// Хук для получения сервиса
export function useService<T>(token: Token<T>): T {
  const container = useContext(DIContext);
  if (!container) {
    throw new Error('DI Container not found. Wrap your component with DIProvider.');
  }
  return container.resolve(token);
}

// Пример использования в компоненте
interface UserComponentProps {
  userId: number;
}

const UserComponent: React.FC<UserComponentProps> = ({ userId }) => {
  const userService = useService(USER_SERVICE_TOKEN);
  const [userProfile, setUserProfile] = useState<UserProfile | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const profile = await userService.getUserProfile(userId);
        setUserProfile(profile);
      } catch (error) {
        console.error('Failed to fetch user:', error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId, userService]);
  
  if (loading) return <div>Loading...</div>;
  if (!userProfile) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{userProfile.name}</h1>
      <p>{userProfile.email}</p>
    </div>
  );
};
```

## Заключение

Внедрение зависимостей с типами в TypeScript позволяет создавать гибкие, тестируемые и надежные приложения. Используйте эти паттерны для построения архитектурно правильных приложений с четким разделением ответственности.

См. также: [[state-management-types]] | [[middleware-types]] | [[testing-types]]