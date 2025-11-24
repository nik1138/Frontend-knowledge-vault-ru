---
aliases: [Документирование API, API Documentation, Документация API]
tags: [typescript, documentation, api, rest, openapi, swagger]
---

# Документирование API в TypeScript проектах

Документирование API является критически важной частью разработки программного обеспечения, особенно в TypeScript проектах, где статическая типизация уже обеспечивает часть документации. Правильно документированный API позволяет разработчикам эффективно использовать ваше API, уменьшает количество ошибок и упрощает интеграцию.

## Важность документирования API

Документирование API имеет множество преимуществ:
- Упрощает понимание и использование API для других разработчиков
- Служит основой для автоматической генерации кода
- Помогает в тестировании и отладке
- Обеспечивает единообразие в работе с API
- Упрощает поддержку и развитие API

## Структура документации API

Хорошая документация API должна включать:
1. **Описание API** - общее описание назначения API
2. **Аутентификация** - как аутентифицироваться в API
3. **Базовый URL** - точка входа в API
4. **Описание эндпоинтов** - каждый эндпоинт с методами HTTP
5. **Параметры запроса** - заголовки, пути, тело запроса
6. **Описание ответов** - формат и структура ответов
7. **Коды ошибок** - возможные коды ошибок и их значения
8. **Примеры использования** - практические примеры запросов и ответов

## Использование TypeScript для документирования API

TypeScript позволяет создавать строго типизированные интерфейсы, которые могут служить частью документации:

```ts
// Типы для пользователя
export interface User {
  /**
   * Уникальный идентификатор пользователя
   */
  id: number;
  
  /**
   * Имя пользователя
   */
  name: string;
  
  /**
   * Email пользователя
   */
  email: string;
  
  /**
   * Дата регистрации
   */
  createdAt: Date;
}

// Типы для запроса и ответа
export interface CreateUserRequest {
  name: string;
  email: string;
}

export interface CreateUserResponse {
  success: boolean;
  user: User;
}
```

## Интеграция с OpenAPI/Swagger

### Установка и настройка

Для автоматической генерации документации API в TypeScript проектах часто используются инструменты, такие как `tsoa` или `@nestjs/swagger`:

```bash
# Для NestJS проектов
npm install @nestjs/swagger swagger-ui-express

# Для других проектов
npm install tsoa
```

### Использование NestJS Swagger

```ts
// user.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';
import { UserService } from './user.service';
import { User, CreateUserRequest } from './user.interface';

@ApiTags('users')
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  @ApiOperation({ summary: 'Получить пользователя по ID' })
  @ApiResponse({ 
    status: 200, 
    description: 'Возвращает информацию о пользователе',
    type: User 
  })
  @ApiResponse({ 
    status: 404, 
    description: 'Пользователь не найден' 
  })
  async findOne(@Param('id') id: string): Promise<User> {
    return this.userService.findOne(+id);
  }

  @Post()
  @ApiOperation({ summary: 'Создать нового пользователя' })
  @ApiResponse({ 
    status: 201, 
    description: 'Пользователь успешно создан',
    type: User 
  })
  async create(@Body() createUserDto: CreateUserRequest): Promise<User> {
    return this.userService.create(createUserDto);
  }
}
```

### Использование tsoa

```ts
// controllers/user.controller.ts
import { Controller, Get, Path, Post, Body, Route, SuccessResponse, GetOperationId } from 'tsoa';
import { User, CreateUserRequest } from '../models/user';

@Route('users')
export class UserController extends Controller {
  @Get('{id}')
  @SuccessResponse('200', 'Успешный ответ')
  public async getUser(
    @Path() id: number
  ): Promise<User> {
    // реализация
    return {} as User;
  }

  @Post()
  @SuccessResponse('201', 'Пользователь создан')
  public async createUser(
    @Body() requestBody: CreateUserRequest
  ): Promise<User> {
    // реализация
    return {} as User;
  }
}
```

## Документирование REST API

### Структура REST эндпоинтов

```ts
// api/user-api.ts
import { User, CreateUserRequest, UpdateUserRequest } from '../models/user';

/**
 * API для работы с пользователями
 */
export class UserApi {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  /**
   * Получить пользователя по ID
   * @param id - ID пользователя
   * @returns Пользователь
   * 
   * @example
   * ```ts
   * const user = await userApi.getUser(1);
   * console.log(user.name); // "John Doe"
   * ```
   */
  async getUser(id: number): Promise<User> {
    const response = await fetch(`${this.baseUrl}/users/${id}`);
    if (!response.ok) {
      throw new Error(`Ошибка получения пользователя: ${response.status}`);
    }
    return response.json();
  }

  /**
   * Создать нового пользователя
   * @param userData - Данные нового пользователя
   * @returns Созданный пользователь
   * 
   * @example
   * ```ts
   * const newUser = await userApi.createUser({
   *   name: "Jane Doe",
   *   email: "jane@example.com"
   * });
   * ```
   */
  async createUser(userData: CreateUserRequest): Promise<User> {
    const response = await fetch(`${this.baseUrl}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      throw new Error(`Ошибка создания пользователя: ${response.status}`);
    }
    
    return response.json();
  }

  /**
   * Обновить пользователя
   * @param id - ID пользователя
   * @param userData - Данные для обновления
   * @returns Обновленный пользователь
   */
  async updateUser(id: number, userData: UpdateUserRequest): Promise<User> {
    const response = await fetch(`${this.baseUrl}/users/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    
    if (!response.ok) {
      throw new Error(`Ошибка обновления пользователя: ${response.status}`);
    }
    
    return response.json();
  }

  /**
   * Удалить пользователя
   * @param id - ID пользователя
   * @returns Статус удаления
   */
  async deleteUser(id: number): Promise<void> {
    const response = await fetch(`${this.baseUrl}/users/${id}`, {
      method: 'DELETE',
    });
    
    if (!response.ok) {
      throw new Error(`Ошибка удаления пользователя: ${response.status}`);
    }
  }
}
```

## Документирование GraphQL API

### Схема GraphQL с документацией

```graphql
"""
Пользователь в системе
"""
type User {
  """
  Уникальный идентификатор пользователя
  """
  id: ID!
  
  """
  Имя пользователя
  """
  name: String!
  
  """
  Email пользователя
  """
  email: String!
  
  """
  Дата регистрации
  """
  createdAt: String!
}

"""
Входные данные для создания пользователя
"""
input CreateUserInput {
  """
  Имя пользователя
  """
  name: String!
  
  """
  Email пользователя
  """
  email: String!
}

type Query {
  """
  Получить пользователя по ID
  """
  user(id: ID!): User
}

type Mutation {
  """
  Создать нового пользователя
  """
  createUser(input: CreateUserInput!): User!
}
```

### Использование TypeScript с GraphQL

```ts
// generated/graphql.ts (сгенерировано graphql-codegen)
export type Maybe<T> = T | null;
export type InputMaybe<T> = Maybe<T>;
export type Exact<T extends { [key: string]: unknown }> = { [K in keyof T]: T[K] };
export type MakeOptional<T, K extends keyof T> = Omit<T, K> & { [SubKey in K]?: Maybe<T[SubKey]> };
export type MakeMaybe<T, K extends keyof T> = Omit<T, K> & { [SubKey in K]: Maybe<T[SubKey]> };

/** Пользователь в системе */
export type User = {
  __typename?: 'User';
  /** Уникальный идентификатор пользователя */
  id: Scalars['ID'];
  /** Имя пользователя */
  name: Scalars['String'];
  /** Email пользователя */
  email: Scalars['String'];
  /** Дата регистрации */
  createdAt: Scalars['String'];
};

export type CreateUserInput = {
  /** Имя пользователя */
  name: Scalars['String'];
  /** Email пользователя */
  email: Scalars['String'];
};
```

## Генерация SDK из документации

### Использование OpenAPI Generator

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: API для работы с пользователями
paths:
  /users/{id}:
    get:
      summary: Получить пользователя по ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        email:
          type: string
          format: email
```

### Генерация TypeScript SDK

```bash
# Установка OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Генерация TypeScript клиента
openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-fetch \
  -o ./generated/
```

## Документирование ошибок и исключений

```ts
/**
 * Ошибки API
 */
export interface ApiError {
  /**
   * Код ошибки
   */
  code: string;
  
  /**
   * Сообщение об ошибке
   */
  message: string;
  
  /**
   * Дополнительные данные об ошибке
   */
  details?: any;
}

/**
 * Коды ошибок
 */
export enum ErrorCode {
  USER_NOT_FOUND = 'USER_NOT_FOUND',
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  UNAUTHORIZED = 'UNAUTHORIZED',
  INTERNAL_ERROR = 'INTERNAL_ERROR'
}

/**
 * Обработчик ошибок API
 */
export class ApiErrorHandler {
  /**
   * Обрабатывает ошибку API и возвращает стандартизированную ошибку
   * @param error - исходная ошибка
   * @returns стандартизированная ошибка API
   */
  static handle(error: any): ApiError {
    if (error.status === 404) {
      return {
        code: ErrorCode.USER_NOT_FOUND,
        message: 'Пользователь не найден'
      };
    }
    
    if (error.status === 401) {
      return {
        code: ErrorCode.UNAUTHORIZED,
        message: 'Неавторизованный доступ'
      };
    }
    
    return {
      code: ErrorCode.INTERNAL_ERROR,
      message: 'Внутренняя ошибка сервера'
    };
  }
}
```

## Лучшие практики документирования API

> [!tip] Совет
> Используйте автоматическую генерацию документации из TypeScript типов и аннотаций для обеспечения согласованности документации и кода.

### Структура документации

1. **Используйте единый формат** для всех эндпоинтов
2. **Документируйте все поля** в запросах и ответах
3. **Указывайте коды состояния HTTP** и их значения
4. **Приводите примеры** запросов и ответов
5. **Описывайте аутентификацию** и авторизацию
6. **Обновляйте документацию** при изменении API

### Пример комплексной документации

```ts
/**
 * API для управления задачами
 * 
 * ## Аутентификация
 * Для доступа к API необходимо указать токен в заголовке Authorization:
 * `Authorization: Bearer <token>`
 * 
 * ## Ограничения скорости
 * Максимальное количество запросов: 1000 в минуту на одного пользователя
 * 
 * ## Статусы ответов
 * - 200: Успешный запрос
 * - 201: Ресурс успешно создан
 * - 400: Неверный запрос
 * - 401: Неавторизованный доступ
 * - 404: Ресурс не найден
 * - 500: Внутренняя ошибка сервера
 */
export class TaskApi {
  // реализация API
}
```

## Интеграция с инструментами документации

### Swagger UI

```ts
// main.ts (для NestJS)
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('User API')
    .setDescription('API для работы с пользователями')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

## Связанные темы

- [[TSDoc]] - для понимания стандартов документирования
- [[Storybook]] - для документирования компонентов пользовательского интерфейса
- [[Генерация-документации]] - для автоматической генерации документации
- [[Лучшие-практики-документирования]] - общие рекомендации по документированию

Документирование API в TypeScript проектах должно сочетать автоматическую генерацию из типов с ручной документацией сложных сценариев. Это обеспечивает точность, актуальность и полноту документации, что критически важно для успешного использования API другими разработчиками.