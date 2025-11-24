---
aliases: [Express TypeScript, TypeScript с Express.js]
tags: [typescript, nodejs, express, web-framework]
---

# Express с TypeScript

## Введение

Express.js - один из самых популярных веб-фреймворков для Node.js. Использование Express с TypeScript позволяет получить преимущества строгой типизации, автодополнения и проверки ошибок на этапе разработки. В этом руководстве мы рассмотрим, как настроить и использовать Express с TypeScript.

## Установка зависимостей

Для начала установим Express и соответствующие типы:

```bash
npm install express
npm install -D @types/express
```

## Базовая настройка приложения

Создадим базовое приложение Express с использованием TypeScript:

```typescript
// src/app.ts
import express, { Application, Request, Response, NextFunction } from 'express';
import { Server } from 'http';

interface CustomRequest extends Request {
  user?: {
    id: string;
    name: string;
  };
}

const app: Application = express();
const PORT: number = parseInt(process.env.PORT || '3000', 10);

// Middleware для парсинга JSON
app.use(express.json());

// Простой маршрут
app.get('/', (req: Request, res: Response) => {
  res.json({ message: 'Добро пожаловать в TypeScript Express API!' });
});

// Пример маршрута с параметрами
app.get('/users/:id', (req: Request, res: Response) => {
  const userId: string = req.params.id;
  res.json({ userId, message: `Получен пользователь с ID: ${userId}` });
});

// Маршрут с использованием кастомного интерфейса
app.get('/profile', (req: CustomRequest, res: Response) => {
  const user = req.user;
  if (user) {
    res.json({ user });
  } else {
    res.status(401).json({ error: 'Пользователь не авторизован' });
  }
});

// Обработчик ошибок
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Что-то пошло не так!' });
});

const server = app.listen(PORT, () => {
  console.log(`Сервер запущен на порту ${PORT}`);
});

export { app, server };
```

## Определение типов для маршрутов

Для лучшей типизации маршрутов можно создать интерфейсы:

```typescript
// src/types.ts
export interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}

export interface PaginationOptions {
  page: number;
  limit: number;
  total: number;
  hasNext: boolean;
  hasPrev: boolean;
}
```

## Создание контроллеров

Создадим контроллеры с типизацией:

```typescript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import { User, ApiResponse } from '../types';

// Моковая база данных
let users: User[] = [
  { id: '1', name: 'Иван', email: 'ivan@example.com', createdAt: new Date() },
  { id: '2', name: 'Мария', email: 'maria@example.com', createdAt: new Date() }
];

export const getAllUsers = (req: Request, res: Response<ApiResponse<User[]>>) => {
  try {
    const response: ApiResponse<User[]> = {
      success: true,
      data: users
    };
    res.json(response);
  } catch (error) {
    const response: ApiResponse<null> = {
      success: false,
      error: 'Ошибка при получении пользователей',
      message: (error as Error).message
    };
    res.status(500).json(response);
  }
};

export const getUserById = (req: Request, res: Response<ApiResponse<User>>) => {
  try {
    const userId: string = req.params.id;
    const user = users.find(u => u.id === userId);
    
    if (!user) {
      const response: ApiResponse<null> = {
        success: false,
        error: 'Пользователь не найден'
      };
      return res.status(404).json(response);
    }

    const response: ApiResponse<User> = {
      success: true,
      data: user
    };
    res.json(response);
  } catch (error) {
    const response: ApiResponse<null> = {
      success: false,
      error: 'Ошибка при получении пользователя',
      message: (error as Error).message
    };
    res.status(500).json(response);
  }
};

export const createUser = (req: Request, res: Response<ApiResponse<User>>) => {
  try {
    const { name, email }: { name: string; email: string } = req.body;
    
    if (!name || !email) {
      const response: ApiResponse<null> = {
        success: false,
        error: 'Имя и email обязательны'
      };
      return res.status(400).json(response);
    }

    const newUser: User = {
      id: Date.now().toString(),
      name,
      email,
      createdAt: new Date()
    };

    users.push(newUser);

    const response: ApiResponse<User> = {
      success: true,
      data: newUser
    };
    res.status(201).json(response);
  } catch (error) {
    const response: ApiResponse<null> = {
      success: false,
      error: 'Ошибка при создании пользователя',
      message: (error as Error).message
    };
    res.status(500).json(response);
  }
};
```

## Роутинг с типизацией

Создадим файл маршрутов:

```typescript
// src/routes/userRoutes.ts
import { Router } from 'express';
import { 
  getAllUsers, 
  getUserById, 
  createUser 
} from '../controllers/userController';

const router = Router();

router.get('/', getAllUsers);
router.get('/:id', getUserById);
router.post('/', createUser);

export default router;
```

## Интеграция маршрутов в приложение

Обновим основное приложение:

```typescript
// src/app.ts
import express, { Application, Request, Response, NextFunction } from 'express';
import userRoutes from './routes/userRoutes';

const app: Application = express();
const PORT: number = parseInt(process.env.PORT || '3000', 10);

// Middleware для парсинга JSON
app.use(express.json());

// Подключение маршрутов
app.use('/api/users', userRoutes);

// Главный маршрут
app.get('/', (req: Request, res: Response) => {
  res.json({ message: 'Добро пожаловать в TypeScript Express API!' });
});

// Обработчик 404
app.use('*', (req: Request, res: Response) => {
  res.status(404).json({ error: 'Маршрут не найден' });
});

// Обработчик ошибок
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Что-то пошло не так!' });
});

export default app;
```

## Использование middleware с типизацией

Создадим типизированный middleware:

```typescript
// src/middleware/authMiddleware.ts
import { Request, Response, NextFunction } from 'express';
import { User } from '../types';

export interface AuthenticatedRequest extends Request {
  user?: User;
}

export const authenticate = (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
  // В реальном приложении здесь будет проверка токена
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Требуется аутентификация' });
  }
  
  // В реальном приложении тут будет проверка токена и извлечение пользователя
  req.user = { 
    id: '1', 
    name: 'Иван', 
    email: 'ivan@example.com', 
    createdAt: new Date() 
  };
  
  next();
};
```

## Пример использования middleware

```typescript
// src/routes/protectedRoutes.ts
import { Router } from 'express';
import { authenticate, AuthenticatedRequest } from '../middleware/authMiddleware';
import { User, ApiResponse } from '../types';

const router = Router();

router.get('/profile', authenticate, (req: AuthenticatedRequest, res: Response<ApiResponse<User>>) => {
  if (!req.user) {
    return res.status(401).json({ 
      success: false, 
      error: 'Пользователь не аутентифицирован' 
    });
  }
  
  const response: ApiResponse<User> = {
    success: true,
    data: req.user
  };
  
  res.json(response);
});

export default router;
```

## Использование enum для HTTP статусов

Для лучшей читаемости кода можно использовать enum:

```typescript
// src/enums/httpStatus.ts
export enum HttpStatus {
  OK = 200,
  CREATED = 201,
  BAD_REQUEST = 400,
  UNAUTHORIZED = 401,
  NOT_FOUND = 404,
  INTERNAL_SERVER_ERROR = 500
}
```

## Пример продвинутой типизации

Для более сложных сценариев можно создать универсальные типы:

```typescript
// src/types/expressTypes.ts
import { Request } from 'express';
import { User } from './types';

export type RequestHandler<T = any, U = any> = (
  req: Request<T>,
  res: Response<U>,
  next: NextFunction
) => void | Promise<void>;

export interface AuthenticatedRequest extends Request {
  user?: User;
  token?: string;
}

export interface PaginatedRequest extends Request {
  query: {
    page?: string;
    limit?: string;
    [key: string]: string | undefined;
  };
}
```

## Заключение

Использование Express с TypeScript значительно улучшает качество кода за счет строгой типизации. Это позволяет избежать многих ошибок на этапе разработки и улучшает читаемость кода.

## См. также

- [[Настройка-окружения]]
- [[Работа-с-файлами]]
- [[Подключение-к-базам-данных]]
- [[Типизация-API]]
- [[Middleware в Express с TypeScript]]