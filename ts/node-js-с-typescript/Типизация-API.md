---
aliases: [Типизация API, API с TypeScript, REST API TypeScript]
tags: [typescript, nodejs, api, rest, типизация]
---

# Типизация API в Node.js с TypeScript

## Введение

Типизация API - один из ключевых аспектов разработки надежных и поддерживаемых серверных приложений. TypeScript позволяет строго типизировать как входящие запросы, так и исходящие ответы, что значительно улучшает качество кода и уменьшает количество ошибок. В этом руководстве мы рассмотрим различные подходы к типизации API в Node.js.

## Базовая типизация API

### Определение типов для запросов и ответов

Создадим базовые интерфейсы для типизации API:

```typescript
// src/types/api.ts
export interface ApiResponse<T = void> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
  timestamp: string;
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

// Типы для HTTP методов
export type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH' | 'HEAD' | 'OPTIONS';

// Интерфейсы для параметров запроса
export interface QueryParams {
  [key: string]: string | string[] | undefined;
}

export interface PathParams {
  [key: string]: string;
}

// Типизированный запрос
export interface TypedRequest<TBody = any, TQuery = QueryParams, TParams = PathParams> {
  body: TBody;
  query: TQuery;
  params: TParams;
  headers: Record<string, string | undefined>;
}
```

### Типизация для пользователей

```typescript
// src/types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
  isActive: boolean;
}

export interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

export interface UpdateUserRequest {
  name?: string;
  email?: string;
  isActive?: boolean;
}

export interface UserResponse extends Omit<User, 'password'> {}
```

## Типизация Express маршрутов

### Базовая типизация контроллеров

```typescript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import { 
  ApiResponse, 
  PaginatedResponse, 
  User, 
  CreateUserRequest, 
  UpdateUserRequest, 
  UserResponse 
} from '../types';

// Типизированные контроллеры
export const getAllUsers = async (
  req: Request<{}, PaginatedResponse<UserResponse>, {}, { page?: string; limit?: string }>,
  res: Response<PaginatedResponse<UserResponse>>
): Promise<void> => {
  try {
    const page = parseInt(req.query.page || '1', 10);
    const limit = parseInt(req.query.limit || '10', 10);
    
    // Логика получения пользователей
    const users: User[] = []; // Заглушка для примера
    const total = 0; // Заглушка для примера
    
    const response: PaginatedResponse<UserResponse> = {
      success: true,
      data: users.map(user => ({
        id: user.id,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt,
        updatedAt: user.updatedAt,
        isActive: user.isActive
      })),
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      },
      timestamp: new Date().toISOString()
    };
    
    res.json(response);
  } catch (error) {
    const response: ApiResponse = {
      success: false,
      error: 'Ошибка при получении пользователей',
      message: (error as Error).message,
      timestamp: new Date().toISOString()
    };
    
    res.status(500).json(response);
  }
};

export const getUserById = async (
  req: Request<{ id: string }, ApiResponse<UserResponse>, {}, {}>,
  res: Response<ApiResponse<UserResponse>>
): Promise<void> => {
  try {
    const userId = req.params.id;
    
    // Логика получения пользователя по ID
    const user: User | null = null; // Заглушка для примера
    
    if (!user) {
      const response: ApiResponse = {
        success: false,
        error: 'Пользователь не найден',
        message: `Пользователь с ID ${userId} не найден`,
        timestamp: new Date().toISOString()
      };
      
      res.status(404).json(response);
      return;
    }
    
    const response: ApiResponse<UserResponse> = {
      success: true,
      data: {
        id: user.id,
        name: user.name,
        email: user.email,
        createdAt: user.createdAt,
        updatedAt: user.updatedAt,
        isActive: user.isActive
      },
      timestamp: new Date().toISOString()
    };
    
    res.json(response);
  } catch (error) {
    const response: ApiResponse = {
      success: false,
      error: 'Ошибка при получении пользователя',
      message: (error as Error).message,
      timestamp: new Date().toISOString()
    };
    
    res.status(500).json(response);
  }
};

export const createUser = async (
  req: Request<{}, ApiResponse<UserResponse>, CreateUserRequest, {}>,
  res: Response<ApiResponse<UserResponse>>
): Promise<void> => {
  try {
    const { name, email, password }: CreateUserRequest = req.body;
    
    // Валидация данных
    if (!name || !email || !password) {
      const response: ApiResponse = {
        success: false,
        error: 'Недостаточно данных',
        message: 'Имя, email и пароль обязательны',
        timestamp: new Date().toISOString()
      };
      
      res.status(400).json(response);
      return;
    }
    
    // Логика создания пользователя
    const newUser: User = { /* ... */ } as User; // Заглушка для примера
    
    const response: ApiResponse<UserResponse> = {
      success: true,
      data: {
        id: newUser.id,
        name: newUser.name,
        email: newUser.email,
        createdAt: newUser.createdAt,
        updatedAt: newUser.updatedAt,
        isActive: newUser.isActive
      },
      message: 'Пользователь успешно создан',
      timestamp: new Date().toISOString()
    };
    
    res.status(201).json(response);
  } catch (error) {
    const response: ApiResponse = {
      success: false,
      error: 'Ошибка при создании пользователя',
      message: (error as Error).message,
      timestamp: new Date().toISOString()
    };
    
    res.status(500).json(response);
  }
};

export const updateUser = async (
  req: Request<{ id: string }, ApiResponse<UserResponse>, UpdateUserRequest, {}>,
  res: Response<ApiResponse<UserResponse>>
): Promise<void> => {
  try {
    const userId = req.params.id;
    const updateData: UpdateUserRequest = req.body;
    
    // Логика обновления пользователя
    const updatedUser: User | null = null; // Заглушка для примера
    
    if (!updatedUser) {
      const response: ApiResponse = {
        success: false,
        error: 'Пользователь не найден',
        message: `Пользователь с ID ${userId} не найден`,
        timestamp: new Date().toISOString()
      };
      
      res.status(404).json(response);
      return;
    }
    
    const response: ApiResponse<UserResponse> = {
      success: true,
      data: {
        id: updatedUser.id,
        name: updatedUser.name,
        email: updatedUser.email,
        createdAt: updatedUser.createdAt,
        updatedAt: updatedUser.updatedAt,
        isActive: updatedUser.isActive
      },
      message: 'Пользователь успешно обновлен',
      timestamp: new Date().toISOString()
    };
    
    res.json(response);
  } catch (error) {
    const response: ApiResponse = {
      success: false,
      error: 'Ошибка при обновлении пользователя',
      message: (error as Error).message,
      timestamp: new Date().toISOString()
    };
    
    res.status(500).json(response);
  }
};
```

## Использование Zod для валидации

Zod - мощная библиотека для валидации данных и типизации:

```bash
npm install zod
npm install -D @types/zod
```

```typescript
// src/validation/userValidation.ts
import { z } from 'zod';

// Схема для создания пользователя
export const createUserSchema = z.object({
  name: z.string().min(2, 'Имя должно содержать не менее 2 символов').max(100),
  email: z.string().email('Неверный формат email'),
  password: z.string().min(8, 'Пароль должен содержать не менее 8 символов')
    .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 'Пароль должен содержать хотя бы одну заглавную букву, одну строчную и одну цифру')
});

// Схема для обновления пользователя
export const updateUserSchema = z.object({
  name: z.string().min(2).max(100).optional(),
  email: z.string().email().optional(),
  isActive: z.boolean().optional()
}).refine(obj => Object.keys(obj).length > 0, {
  message: 'Должно быть указано хотя бы одно поле для обновления'
});

// Схема для параметров запроса
export const userQuerySchema = z.object({
  page: z.string().regex(/^\d+$/).transform(Number).optional(),
  limit: z.string().regex(/^\d+$/).transform(Number).optional(),
  search: z.string().optional()
}).refine(data => {
  if (data.page !== undefined && data.page < 1) {
    return false;
  }
  if (data.limit !== undefined && (data.limit < 1 || data.limit > 100)) {
    return false;
  }
  return true;
}, {
  message: 'Некорректные параметры пагинации'
});

export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
export type UserQueryInput = z.infer<typeof userQuerySchema>;
```

### Использование схем валидации в контроллерах

```typescript
// src/middleware/validation.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema } from 'zod';

export const validate = (schema: ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse({
        body: req.body,
        query: req.query,
        params: req.params
      });
      next();
    } catch (error) {
      const response = {
        success: false,
        error: 'Ошибка валидации',
        message: error instanceof Error ? error.message : 'Неизвестная ошибка валидации',
        timestamp: new Date().toISOString()
      };
      
      res.status(400).json(response);
    }
  };
};
```

```typescript
// src/controllers/userControllerWithValidation.ts
import { Request, Response } from 'express';
import { validate } from '../middleware/validation';
import { createUserSchema, updateUserSchema, userQuerySchema } from '../validation/userValidation';
import { ApiResponse, PaginatedResponse, UserResponse } from '../types';

// Пример контроллера с использованием валидации
export const createUserWithValidation = [
  validate(createUserSchema),
  async (req: Request, res: Response<ApiResponse<UserResponse>>) => {
    try {
      // req.body теперь строго типизирован в соответствии со схемой
      const { name, email, password } = req.body;
      
      // Логика создания пользователя
      // ...
      
      const response: ApiResponse<UserResponse> = {
        success: true,
        data: { /* ... */ } as UserResponse,
        message: 'Пользователь успешно создан',
        timestamp: new Date().toISOString()
      };
      
      res.status(201).json(response);
    } catch (error) {
      // Обработка ошибок
    }
  }
];

export const getAllUsersWithValidation = [
  validate(userQuerySchema),
  async (req: Request, res: Response<PaginatedResponse<UserResponse>>) => {
    try {
      // req.query теперь строго типизирован
      const { page = 1, limit = 10, search } = req.query as any;
      
      // Логика получения пользователей
      // ...
      
      const response: PaginatedResponse<UserResponse> = {
        success: true,
        data: [], // Пользователи
        pagination: {
          page,
          limit,
          total: 0, // Общее количество
          totalPages: 0 // Общее количество страниц
        },
        timestamp: new Date().toISOString()
      };
      
      res.json(response);
    } catch (error) {
      // Обработка ошибок
    }
  }
];
```

## Генерация типов из OpenAPI

### Установка и настройка

Для генерации типов из OpenAPI спецификации можно использовать специальные инструменты:

```bash
npm install @openapi-generator-plus/generator
# или
npm install openapi-typescript
```

### Пример OpenAPI спецификации

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
  /users/{id}:
    get:
      summary: Get user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '404':
          description: User not found
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
        isActive:
          type: boolean
    CreateUserRequest:
      type: object
      required:
        - name
        - email
        - password
      properties:
        name:
          type: string
        email:
          type: string
        password:
          type: string
    UserResponse:
      allOf:
        - $ref: '#/components/schemas/User'
        - type: object
          properties:
            password:
              type: string
              writeOnly: true
    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
```

## Создание middleware для типизации

### Типизированный middleware аутентификации

```typescript
// src/middleware/authMiddleware.ts
import { Request, Response, NextFunction } from 'express';
import { User } from '../types/user';

// Интерфейс для запроса с аутентифицированным пользователем
export interface AuthenticatedRequest extends Request {
  user?: User;
}

export const authenticate = (req: AuthenticatedRequest, res: Response, next: NextFunction): void => {
  // В реальном приложении здесь будет проверка токена
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    const response = {
      success: false,
      error: 'Требуется аутентификация',
      message: 'Токен аутентификации отсутствует',
      timestamp: new Date().toISOString()
    };
    
    res.status(401).json(response);
    return;
  }
  
  // В реальном приложении тут будет проверка токена и извлечение пользователя
  req.user = {
    id: '1',
    name: 'Иван',
    email: 'ivan@example.com',
    createdAt: new Date(),
    updatedAt: new Date(),
    isActive: true
  };
  
  next();
};
```

### Типизированный middleware обработки ошибок

```typescript
// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';

export interface ApiErrorResponse {
  success: boolean;
  error: string;
  message?: string;
  stack?: string;
  timestamp: string;
}

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response<ApiErrorResponse>,
  next: NextFunction
): void => {
  console.error(err.stack);
  
  const response: ApiErrorResponse = {
    success: false,
    error: 'Внутренняя ошибка сервера',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined,
    stack: process.env.NODE_ENV === 'development' ? err.stack : undefined,
    timestamp: new Date().toISOString()
  };
  
  res.status(500).json(response);
};
```

## Пример полного API с типизацией

```typescript
// src/app.ts
import express, { Application } from 'express';
import { authenticate, AuthenticatedRequest } from './middleware/authMiddleware';
import { errorHandler, ApiErrorResponse } from './middleware/errorHandler';
import { ApiResponse, UserResponse } from './types';

const app: Application = express();

// Middleware для парсинга JSON
app.use(express.json());

// Защищенный маршрут с типизацией
app.get('/profile', authenticate, (req: AuthenticatedRequest, res: Response<ApiResponse<UserResponse>>) => {
  if (!req.user) {
    const response: ApiErrorResponse = {
      success: false,
      error: 'Пользователь не аутентифицирован',
      timestamp: new Date().toISOString()
    };
    
    res.status(401).json(response);
    return;
  }
  
  const response: ApiResponse<UserResponse> = {
    success: true,
    data: {
      id: req.user.id,
      name: req.user.name,
      email: req.user.email,
      createdAt: req.user.createdAt,
      updatedAt: req.user.updatedAt,
      isActive: req.user.isActive
    },
    timestamp: new Date().toISOString()
  };
  
  res.json(response);
});

// Middleware обработки ошибок
app.use(errorHandler);

export default app;
```

## Заключение

Типизация API в Node.js с TypeScript значительно улучшает качество кода, предотвращает ошибки и улучшает читаемость. Использование строгой типизации, валидации данных и соответствующих инструментов делает приложения более надежными и поддерживаемыми.

## См. также

- [[Настройка-окружения]]
- [[Express-с-TypeScript]]
- [[Работа-с-файлами]]
- [[Подключение-к-базам-данных]]
- [[Валидация данных в TypeScript]]
- [[OpenAPI и TypeScript]]