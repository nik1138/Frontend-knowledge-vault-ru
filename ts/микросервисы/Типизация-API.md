---
aliases: [Типизация API, API типы, Типы API]
tags: [api, typescript, typing, interfaces, validation]
---

# Типизация API в микросервисной архитектуре

## Обзор

Правильная типизация API является критически важной для разработки надежных микросервисов. Она обеспечивает безопасность типов, улучшает читаемость кода и позволяет избежать многих ошибок во время выполнения.

## Базовые типы API

### DTO (Data Transfer Objects)

DTO используются для передачи данных между сервисами:

```typescript
// user.dto.ts
export interface CreateUserDto {
  readonly email: string;
  readonly firstName: string;
  readonly lastName: string;
  readonly age?: number;
  readonly preferences?: UserPreferences;
}

export interface UpdateUserDto {
  readonly firstName?: string;
  readonly lastName?: string;
  readonly age?: number;
  readonly preferences?: Partial<UserPreferences>;
}

export interface UserPreferences {
  readonly theme: 'light' | 'dark';
  readonly language: string;
  readonly notifications: {
    readonly email: boolean;
    readonly push: boolean;
  };
}

export interface UserResponseDto {
  readonly id: string;
  readonly email: string;
  readonly firstName: string;
  readonly lastName: string;
  readonly createdAt: Date;
  readonly updatedAt: Date;
}
```

### Типы запросов и ответов

```typescript
// api.types.ts
export interface ApiResponse<T> {
  readonly data: T;
  readonly success: boolean;
  readonly message?: string;
  readonly error?: ApiError;
}

export interface ApiError {
  readonly code: string;
  readonly message: string;
  readonly details?: Record<string, any>;
}

export interface PaginationParams {
  readonly page: number;
  readonly limit: number;
  readonly sortBy?: string;
  readonly sortOrder?: 'asc' | 'desc';
}

export interface PaginatedResponse<T> {
  readonly data: T[];
  readonly pagination: {
    readonly page: number;
    readonly limit: number;
    readonly total: number;
    readonly totalPages: number;
  };
}
```

## Валидация типов

### Использование Zod для валидации

```typescript
import { z } from 'zod';

// Схема для создания пользователя
export const CreateUserSchema = z.object({
  email: z.string().email('Некорректный email'),
  firstName: z.string().min(1, 'Имя обязательно').max(50),
  lastName: z.string().min(1, 'Фамилия обязательна').max(50),
  age: z.number().min(0).max(150).optional(),
  preferences: z.object({
    theme: z.enum(['light', 'dark']),
    language: z.string().length(2),
    notifications: z.object({
      email: z.boolean(),
      push: z.boolean()
    })
  }).optional()
});

export type CreateUserDto = z.infer<typeof CreateUserSchema>;

// Валидация входящих данных
export function validateCreateUser(data: unknown): CreateUserDto {
  return CreateUserSchema.parse(data);
}

// Валидация с обработкой ошибок
export function safeValidateCreateUser(data: unknown): { success: true; data: CreateUserDto } | { success: false; error: string } {
  try {
    const validatedData = CreateUserSchema.parse(data);
    return { success: true, data: validatedData };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { 
        success: false, 
        error: error.errors.map(e => `${e.path.join('.')}: ${e.message}`).join(', ') 
      };
    }
    return { success: false, error: 'Неизвестная ошибка валидации' };
  }
}
```

### Валидация с использованием class-validator

```typescript
import { IsEmail, IsString, IsOptional, IsNumber, Min, Max, ValidateNested, IsEnum } from 'class-validator';
import { Type } from 'class-transformer';

export class UserPreferencesDto {
  @IsEnum(['light', 'dark'])
  theme: 'light' | 'dark';

  @IsString()
  language: string;

  @ValidateNested()
  @Type(() => NotificationPreferencesDto)
  notifications: NotificationPreferencesDto;
}

export class NotificationPreferencesDto {
  @IsBoolean()
  email: boolean;

  @IsBoolean()
  push: boolean;
}

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(1)
  firstName: string;

  @IsString()
  @MinLength(1)
  lastName: string;

  @IsOptional()
  @IsNumber()
  @Min(0)
  @Max(150)
  age?: number;

  @IsOptional()
  @ValidateNested()
  @Type(() => UserPreferencesDto)
  preferences?: UserPreferencesDto;
}
```

## Типы для HTTP-запросов

### Типы для Express.js

```typescript
import { Request, Response, NextFunction } from 'express';

// Типизированные параметры запроса
export interface GetUserRequest extends Request {
  params: {
    id: string;
  };
  query: {
    includePreferences?: string;
  };
  body: UpdateUserDto;
}

// Типизированный ответ
export type GetUserResponse = Response<ApiResponse<UserResponseDto>>;

// Типизированный middleware
export type AuthenticatedRequest = Request & {
  user: UserResponseDto;
};

// Типизированный обработчик
export type TypedRequestHandler<T extends Request> = (
  req: T,
  res: Response,
  next: NextFunction
) => Promise<void> | void;

// Пример использования
export const getUserHandler: TypedRequestHandler<GetUserRequest> = async (req, res) => {
  try {
    const userId = req.params.id;
    const includePreferences = req.query.includePreferences === 'true';
    
    const user = await userService.findById(userId, includePreferences);
    
    const response: ApiResponse<UserResponseDto> = {
      data: user,
      success: true
    };
    
    res.json(response);
  } catch (error) {
    res.status(500).json({
      data: null,
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: error.message
      }
    });
  }
};
```

### Типы для Fastify

```typescript
import { FastifyPluginAsync, FastifyReply, FastifyRequest } from 'fastify';

// Определение схем для Fastify
export const getUserSchema = {
  params: {
    type: 'object',
    properties: {
      id: { type: 'string' }
    },
    required: ['id']
  },
  response: {
    200: {
      type: 'object',
      properties: {
        data: {
          type: 'object',
          properties: {
            id: { type: 'string' },
            email: { type: 'string' },
            firstName: { type: 'string' },
            lastName: { type: 'string' },
            createdAt: { type: 'string', format: 'date-time' },
            updatedAt: { type: 'string', format: 'date-time' }
          }
        },
        success: { type: 'boolean' }
      }
    }
  }
};

// Типизированный маршрут для Fastify
export const getUserRoute: FastifyPluginAsync = async (fastify, opts) => {
  fastify.get<{
    Params: { id: string };
    Reply: ApiResponse<UserResponseDto>;
  }>('/users/:id', {
    schema: getUserSchema
  }, async (request, reply) => {
    const { id } = request.params;
    const user = await userService.findById(id);
    
    reply.send({
      data: user,
      success: true
    });
  });
};
```

## Типы для взаимодействия между сервисами

### Интерфейсы сервисов

```typescript
// user-service.interface.ts
export interface UserServiceInterface {
  getUserById(id: string, includePreferences?: boolean): Promise<UserResponseDto | null>;
  createUser(userData: CreateUserDto): Promise<UserResponseDto>;
  updateUser(id: string, updateData: UpdateUserDto): Promise<UserResponseDto>;
  deleteUser(id: string): Promise<boolean>;
  searchUsers(query: string, pagination: PaginationParams): Promise<PaginatedResponse<UserResponseDto>>;
}

// order-service.interface.ts
export interface OrderServiceInterface {
  createOrder(orderData: CreateOrderDto): Promise<OrderResponseDto>;
  getOrderById(id: string): Promise<OrderResponseDto | null>;
  getUserOrders(userId: string, pagination: PaginationParams): Promise<PaginatedResponse<OrderResponseDto>>;
  updateOrderStatus(orderId: string, status: OrderStatus): Promise<OrderResponseDto>;
}
```

### Типы для событийной архитектуры

```typescript
// events.types.ts
export interface BaseEvent<T = any> {
  readonly id: string;
  readonly type: string;
  readonly timestamp: Date;
  readonly data: T;
  readonly source: string;
}

export interface UserCreatedEvent extends BaseEvent<CreateUserDto> {
  readonly type: 'USER_CREATED';
}

export interface OrderCreatedEvent extends BaseEvent<CreateOrderDto> {
  readonly type: 'ORDER_CREATED';
}

export interface OrderStatusUpdatedEvent extends BaseEvent<{
  orderId: string;
  oldStatus: OrderStatus;
  newStatus: OrderStatus;
}> {
  readonly type: 'ORDER_STATUS_UPDATED';
}

// Типизированный обработчик событий
export type EventHandler<T extends BaseEvent> = (event: T) => Promise<void> | void;

// Регистрация обработчиков
export interface EventRegistry {
  register<T extends BaseEvent>(eventType: T['type'], handler: EventHandler<T>): void;
  emit<T extends BaseEvent>(event: T): Promise<void>;
}
```

## Автогенерация типов

### Использование OpenAPI и Swagger

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: User Service API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    $ref: '#/components/schemas/UserResponse'
                  success:
                    type: boolean
                  message:
                    type: string
components:
  schemas:
    UserResponse:
      type: object
      properties:
        id:
          type: string
        email:
          type: string
        firstName:
          type: string
        lastName:
          type: string
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
```

### Генерация типов из OpenAPI

```typescript
// Сгенерированные типы с помощью openapi-typescript
export interface paths {
  "/users/{id}": {
    get: {
      parameters: {
        path: {
          id: string;
        };
      };
      responses: {
        /** User found */
        200: {
          content: {
            "application/json": {
              data: definitions["UserResponse"];
              success: boolean;
              message?: string;
            };
          };
        };
      };
    };
  };
}

export interface definitions {
  UserResponse: {
    id: string;
    email: string;
    firstName: string;
    lastName: string;
    createdAt: string; // date-time
    updatedAt: string; // date-time
  };
}
```

## Лучшие практики типизации API

1. **Используйте readonly для DTO** - предотвращает изменение данных
2. **Разделяйте DTO для входящих и исходящих данных** - разные требования к валидации
3. **Используйте Union Types для статусов** - строгая типизация состояний
4. **Валидируйте данные на границе сервиса** - защита от некорректных данных
5. **Используйте Generic типы для общих структур** - переиспользование кода

```typescript
// Пример Generic типа для ответа API
export interface ApiResponse<T> {
  readonly data: T;
  readonly success: boolean;
  readonly message?: string;
  readonly error?: ApiError;
}

// Использование
const userResponse: ApiResponse<UserResponseDto> = {
  data: user,
  success: true
};

const usersResponse: ApiResponse<UserResponseDto[]> = {
  data: users,
  success: true
};
```

## Связанные темы

- [[Архитектура-микросервисов]]
- [[Взаимодействие-между-сервисами]]
- [[Тестирование-микросервисов]]
- [[Валидация-данных]]
- [[Domain-Driven-Design]]
- [[TypeScript-паттерны]]