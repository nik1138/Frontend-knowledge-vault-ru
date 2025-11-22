---
aliases: ["Заготовки для определения типов API", "TS API Type Definitions"]
tags: [typescript, api, types]
---

# Заготовки для определения типов API

## Введение

Определение типов для API - важная часть разработки frontend приложений. Правильно спроектированные типы API обеспечивают типобезопасность, улучшают автодополнение в IDE и уменьшают количество ошибок при работе с внешними сервисами.

## Базовые типы для API

### Общий тип для API ответов

```typescript
// Универсальный тип для API ответов
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
  timestamp: string;
}

// Успешный ответ
interface SuccessResponse<T> extends ApiResponse<T> {
  success: true;
  data: T;
  error?: never;
}

// Ошибка ответа
interface ErrorResponse extends ApiResponse<never> {
  success: false;
  data?: never;
  error: string;
}
```

### Типы для запросов и ответов

```typescript
// Базовый тип для API клиента
interface ApiClient {
  get<T>(url: string, config?: RequestConfig): Promise<ApiResponse<T>>;
  post<T, R = T>(url: string, data?: R, config?: RequestConfig): Promise<ApiResponse<T>>;
  put<T, R = T>(url: string, data?: R, config?: RequestConfig): Promise<ApiResponse<T>>;
  delete<T>(url: string, config?: RequestConfig): Promise<ApiResponse<T>>;
}

interface RequestConfig {
  headers?: Record<string, string>;
  params?: Record<string, string | number>;
  timeout?: number;
}
```

## Продвинутые заготовки

### Типы для пагинации

```typescript
// Тип для пагинированного ответа
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// Пример использования
type UsersResponse = PaginatedResponse<User>;

// Тип для параметров пагинации
interface PaginationParams {
  page?: number;
  limit?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}
```

### Типы для фильтрации

```typescript
// Базовый тип для фильтров
interface BaseFilter {
  [key: string]: string | number | boolean | (string | number)[];
}

// Тип для фильтра с операторами
interface FilterWithOperators {
  field: string;
  operator: 'eq' | 'ne' | 'gt' | 'gte' | 'lt' | 'lte' | 'like' | 'in' | 'notIn';
  value: string | number | boolean | (string | number)[];
}

type AdvancedFilter = FilterWithOperators[];

// Пример использования в API
interface GetUsersParams extends PaginationParams {
  filters?: BaseFilter | AdvancedFilter;
  search?: string;
}
```

### Типы для сортировки

```typescript
// Тип для параметров сортировки
interface SortParam {
  field: string;
  order: 'asc' | 'desc';
}

type SortParams = SortParam | SortParam[];

// Расширенный тип для запроса с фильтрацией, сортировкой и пагинацией
interface QueryParams<T = BaseFilter> extends PaginationParams {
  filters?: T;
  sort?: SortParams;
  search?: string;
}
```

## Практические примеры

### Определение типов для конкретного API

```typescript
// Типы для пользовательского API
interface UserApi {
  // Получение списка пользователей
  getUsers(params?: GetUsersParams): Promise<ApiResponse<PaginatedResponse<User>>>;
  
  // Получение пользователя по ID
  getUser(id: number): Promise<ApiResponse<User>>;
  
  // Создание пользователя
  createUser(data: CreateUserRequest): Promise<ApiResponse<User>>;
  
  // Обновление пользователя
  updateUser(id: number, data: UpdateUserRequest): Promise<ApiResponse<User>>;
  
  // Удаление пользователя
  deleteUser(id: number): Promise<ApiResponse<{ id: number }>>;
}

interface CreateUserRequest {
  name: string;
  email: string;
  role?: string;
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
  role?: string;
}
```

### Типы для REST API ресурсов

```typescript
// Типы для продуктового API
interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  category: string;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

interface ProductApi {
  getProducts(params?: QueryParams): Promise<ApiResponse<PaginatedResponse<Product>>>;
  getProduct(id: number): Promise<ApiResponse<Product>>;
  createProduct(data: CreateProductRequest): Promise<ApiResponse<Product>>;
  updateProduct(id: number, data: UpdateProductRequest): Promise<ApiResponse<Product>>;
  deleteProduct(id: number): Promise<ApiResponse<{ id: number }>>;
}

interface CreateProductRequest {
  name: string;
  description: string;
  price: number;
  category: string;
  isActive?: boolean;
}

interface UpdateProductRequest {
  name?: string;
  description?: string;
  price?: number;
  category?: string;
  isActive?: boolean;
}
```

### Типы для аутентификации

```typescript
interface AuthApi {
  login(credentials: LoginCredentials): Promise<ApiResponse<AuthResponse>>;
  register(userData: RegisterData): Promise<ApiResponse<AuthResponse>>;
  logout(): Promise<ApiResponse<{}>>;
  refreshToken(): Promise<ApiResponse<AuthResponse>>;
  getCurrentUser(): Promise<ApiResponse<User>>;
}

interface LoginCredentials {
  email: string;
  password: string;
}

interface RegisterData {
  name: string;
  email: string;
  password: string;
}

interface AuthResponse {
  token: string;
  refreshToken: string;
  user: User;
  expiresIn: number;
}
```

### Типы для загрузки файлов

```typescript
interface FileUploadResponse {
  id: string;
  filename: string;
  originalName: string;
  size: number;
  mimeType: string;
  url: string;
  uploadedAt: string;
}

interface FileApi {
  uploadFile(file: File, options?: UploadOptions): Promise<ApiResponse<FileUploadResponse>>;
  deleteFile(fileId: string): Promise<ApiResponse<{ id: string }>>;
  getFileUrl(fileId: string): string;
}

interface UploadOptions {
  maxSize?: number; // в байтах
  allowedTypes?: string[]; // MIME типы
  folder?: string;
}
```

## Утилиты для работы с API типами

### Утилита для создания типов запросов

```typescript
// Утилита для создания типа запроса на основе типа ответа
type RequestType<T> = Omit<T, 'id' | 'createdAt' | 'updatedAt'>;

type CreateUserRequestFromUser = RequestType<User>;
// Создает тип без id, createdAt и updatedAt полей

// Утилита для создания типа обновления
type UpdateType<T> = Partial<Omit<T, 'id' | 'createdAt'>>;
```

### Утилита для обработки ошибок API

```typescript
class ApiError extends Error {
  constructor(
    public status: number,
    public message: string,
    public details?: any
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

// Тип для обработки ошибок
interface ApiErrorHandler {
  handle(error: unknown): ApiError;
  isApiError(error: unknown): error is ApiError;
}

const apiErrorHandler: ApiErrorHandler = {
  handle(error: unknown): ApiError {
    if (error instanceof ApiError) {
      return error;
    }
    
    if (error instanceof Error) {
      return new ApiError(500, error.message);
    }
    
    return new ApiError(500, 'Unknown error occurred');
  },
  
  isApiError(error: unknown): error is ApiError {
    return error instanceof ApiError;
  }
};
```

### Типы для WebSocket API

```typescript
// Типы для WebSocket сообщений
interface WebSocketMessage<T = any> {
  type: string;
  payload: T;
  timestamp: number;
  correlationId?: string;
}

interface WebSocketApi {
  connect(url: string): Promise<void>;
  disconnect(): void;
  send<T>(message: WebSocketMessage<T>): void;
  subscribe<T>(event: string, handler: (data: T) => void): void;
  unsubscribe(event: string): void;
}

// Типы для конкретных сообщений
interface UserUpdateMessage {
  userId: number;
  updates: Partial<User>;
}

interface NotificationMessage {
  id: string;
  type: 'info' | 'warning' | 'error';
  title: string;
  content: string;
  timestamp: string;
}
```

## Заключение

Правильно спроектированные типы API обеспечивают типобезопасность, улучшают разработку и уменьшают количество ошибок. Используйте эти заготовки как основу для создания типов, специфичных для вашего приложения.

См. также: [[validation-utilities]] | [[react-component-types]] | [[state-management-types]]