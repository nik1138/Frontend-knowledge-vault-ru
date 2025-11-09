# Экосистема TypeScript: Инструменты, библиотеки и интеграции

Экосистема TypeScript включает в себя широкий спектр инструментов, библиотек, фреймворков и интеграций, которые делают TypeScript мощным и гибким языком программирования. Эти компоненты работают вместе, чтобы обеспечить разработчикам полный стек для создания современных приложений.

## Инструменты сборки и компиляции

### 1. TypeScript Compiler (tsc)

```json
// Пример продвинутой конфигурации компилятора
{
  "compilerOptions": {
    // Основные настройки
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    
    // Система модулей
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    },
    
    // Типизация
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    
    // Вывод
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    
    // Производительность
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

### 2. Системы сборки

#### Webpack с TypeScript

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  devtool: 'source-map',
};
```

```json
// package.json зависимости для webpack
{
  "devDependencies": {
    "typescript": "^4.9.0",
    "ts-loader": "^9.4.0",
    "webpack": "^5.75.0",
    "webpack-cli": "^4.10.0",
    "webpack-dev-server": "^4.11.1"
  }
}
```

#### Vite с TypeScript

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
    },
  },
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html'),
      },
    },
  },
  esbuild: {
    target: 'es2020',
  },
  optimizeDeps: {
    include: ['typescript'],
  },
});
```

#### Rollup с TypeScript

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'es',
    sourcemap: true,
  },
  plugins: [
    nodeResolve(),
    commonjs(),
    typescript({ tsconfig: './tsconfig.json' }),
  ],
};
```

### 3. Современные сборщики

#### esbuild

```javascript
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.ts'],
  bundle: true,
  minify: true,
  sourcemap: true,
  format: 'esm',
  target: 'es2020',
  outfile: 'dist/bundle.js',
}).catch(() => process.exit(1));
```

#### SWC (Speedy Web Compiler)

```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true
    },
    "transform": {
      "decoratorMetadata": true
    },
    "target": "es2020"
  },
  "module": {
    "type": "es6"
  }
}
```

## Средства разработки и IDE

### 1. Поддержка в редакторах

#### VS Code

```json
// .vscode/settings.json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.useAliasesForRenames": false,
  "typescript.surveys.enabled": false,
  "typescript.tsc.autoDetect": "on"
}
```

#### Настройка плагинов

```json
// .vscode/extensions.json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-typescript-tslint-plugin",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

### 2. Инструменты статического анализа

#### ESLint с TypeScript

```json
// .eslintrc.json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module",
    "project": ["./tsconfig.json"]
  },
  "plugins": [
    "@typescript-eslint"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

```json
// package.json
{
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^5.45.0",
    "@typescript-eslint/parser": "^5.45.0",
    "eslint": "^8.28.0"
  }
}
```

#### Prettier для форматирования

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

```json
// package.json
{
  "devDependencies": {
    "prettier": "^2.8.0",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-prettier": "^4.2.1"
  }
}
```

## Тестирование

### 1. Модульное тестирование

#### Jest с TypeScript

```json
// jest.config.json
{
  "preset": "ts-jest",
  "testEnvironment": "node",
  "roots": [
    "<rootDir>/src"
  ],
  "testMatch": [
    "**/__tests__/**/*.+(ts|tsx|js)",
    "**/?(*.)+(spec|test).+(ts|tsx|js)"
  ],
  "transform": {
    "^.+\\.(ts|tsx)$": "ts-jest"
  },
  "moduleNameMapper": {
    "^@/(.*)$": "<rootDir>/src/$1"
  }
}
```

```typescript
// Пример теста
import { Calculator } from '../src/Calculator';

describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  test('should add two numbers correctly', () => {
    const result = calculator.add(2, 3);
    expect(result).toBe(5);
  });

  test('should multiply two numbers correctly', () => {
    const result = calculator.multiply(4, 5);
    expect(result).toBe(20);
  });
});
```

#### Vitest (альтернатива Jest)

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./test/setup.ts'],
  },
});
```

### 2. Типизированные тесты

```typescript
// Использование dtslint для проверки типов
// tests/types.test.ts
import { expectTypeOf } from 'vitest';

interface User {
  id: number;
  name: string;
}

function getUser(id: number): User {
  return { id, name: `User ${id}` };
}

// Проверка типов во время тестирования
test('getUser should return User type', () => {
  const user = getUser(1);
  expectTypeOf(user).toMatchTypeOf<User>();
  expectTypeOf(user.id).toBeNumber();
  expectTypeOf(user.name).toBeString();
});
```

## Библиотеки и утилиты

### 1. Управление состоянием

#### Redux Toolkit с TypeScript

```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  loading: false,
  error: null,
};

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    fetchUsersStart: (state) => {
      state.loading = true;
      state.error = null;
    },
    fetchUsersSuccess: (state, action: PayloadAction<User[]>) => {
      state.loading = false;
      state.users = action.payload;
    },
    fetchUsersFailure: (state, action: PayloadAction<string>) => {
      state.loading = false;
      state.error = action.payload;
    },
  },
});

export const { fetchUsersStart, fetchUsersSuccess, fetchUsersFailure } = userSlice.actions;
export default userSlice.reducer;
```

#### Zustand (легковесная альтернатива)

```typescript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface User {
  id: number;
  name: string;
}

interface UserStore {
  users: User[];
  addUser: (user: User) => void;
  removeUser: (id: number) => void;
  fetchUsers: () => Promise<void>;
}

export const useUserStore = create<UserStore>()(
  devtools(
    persist(
      (set, get) => ({
        users: [],
        addUser: (user) => set((state) => ({ users: [...state.users, user] })),
        removeUser: (id) => 
          set((state) => ({ 
            users: state.users.filter(user => user.id !== id) 
          })),
        fetchUsers: async () => {
          // Симуляция API вызова
          const response = await fetch('/api/users');
          const users = await response.json();
          set({ users });
        },
      }),
      {
        name: 'user-storage', // имя для localStorage
      }
    )
  )
);
```

### 2. Работа с API и HTTP-запросами

#### Axios с TypeScript

```typescript
import axios, { AxiosResponse } from 'axios';

interface User {
  id: number;
  name: string;
  email: string;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  statusText: string;
}

class ApiService {
  private axiosInstance = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 10000,
    headers: {
      'Content-Type': 'application/json',
    },
  });

  async getUser(id: number): Promise<ApiResponse<User>> {
    try {
      const response: AxiosResponse<User> = await this.axiosInstance.get(`/users/${id}`);
      return {
        data: response.data,
        status: response.status,
        statusText: response.statusText,
      };
    } catch (error) {
      if (axios.isAxiosError(error)) {
        throw new Error(`API Error: ${error.response?.data?.message || error.message}`);
      }
      throw error;
    }
  }

  async createUser(userData: Omit<User, 'id'>): Promise<ApiResponse<User>> {
    const response: AxiosResponse<User> = await this.axiosInstance.post('/users', userData);
    return {
      data: response.data,
      status: response.status,
      statusText: response.statusText,
    };
  }
}

export default new ApiService();
```

#### Fetch с TypeScript (без внешних зависимостей)

```typescript
interface RequestOptions {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  headers?: Record<string, string>;
  body?: any;
}

class TypedFetch {
  constructor(private baseUrl: string) {}

  async request<T>(endpoint: string, options: RequestOptions = {}): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    const config: RequestInit = {
      method: options.method || 'GET',
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
    };

    if (options.body) {
      config.body = JSON.stringify(options.body);
    }

    const response = await fetch(url, config);

    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    }

    return response.json() as Promise<T>;
  }

  get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  post<T, U>(endpoint: string, data: U): Promise<T> {
    return this.request<T>(endpoint, { method: 'POST', body: data });
  }

  put<T, U>(endpoint: string, data: U): Promise<T> {
    return this.request<T>(endpoint, { method: 'PUT', body: data });
  }

  delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const api = new TypedFetch('https://api.example.com');

async function example() {
  const users: User[] = await api.get('/users');
  const newUser: User = await api.post('/users', { name: 'John', email: 'john@example.com' });
}
```

### 3. Работа с формами

#### React Hook Form с TypeScript

```typescript
import { useForm, SubmitHandler } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Определение схемы валидации
const userSchema = z.object({
  name: z.string().min(2, 'Имя должно содержать не менее 2 символов'),
  email: z.string().email('Неверный формат email'),
  age: z.number().min(18, 'Возраст должен быть не менее 18').max(120, 'Возраст должен быть не более 120'),
});

type UserFormData = z.infer<typeof userSchema>;

const UserForm: React.FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit: SubmitHandler<UserFormData> = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('name')} placeholder="Имя" />
        {errors.name && <p>{errors.name.message}</p>}
      </div>
      
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      
      <div>
        <input type="number" {...register('age', { valueAsNumber: true })} placeholder="Возраст" />
        {errors.age && <p>{errors.age.message}</p>}
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Фреймворки и платформы

### 1. React с TypeScript

```typescript
// Типизированный компонент
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
    avatar?: string;
  };
  onEdit: (userId: number) => void;
  onDelete: (userId: number) => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, onEdit, onDelete }) => {
  return (
    <div className="user-card">
      <img 
        src={user.avatar || '/default-avatar.png'} 
        alt={`${user.name}'s avatar`} 
        className="avatar"
      />
      <div className="user-info">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
      </div>
      <div className="actions">
        <button onClick={() => onEdit(user.id)}>Редактировать</button>
        <button onClick={() => onDelete(user.id)}>Удалить</button>
      </div>
    </div>
  );
};

// Использование компонента
const UserList: React.FC = () => {
  const [users, setUsers] = React.useState<UserCardProps['user'][]>([]);
  
  const handleEdit = (userId: number) => {
    console.log(`Редактировать пользователя ${userId}`);
  };
  
  const handleDelete = (userId: number) => {
    setUsers(users.filter(user => user.id !== userId));
  };
  
  return (
    <div>
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user} 
          onEdit={handleEdit} 
          onDelete={handleDelete} 
        />
      ))}
    </div>
  );
};
```

### 2. Angular с TypeScript

```typescript
// Типизированный сервис
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface User {
  id: number;
  name: string;
  email: string;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'https://api.example.com/users';

  constructor(private http: HttpClient) { }

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}

// Типизированный компонент
import { Component, OnInit } from '@angular/core';
import { UserService, User } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `
    <div *ngFor="let user of users" class="user-card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `
})
export class UserListComponent implements OnInit {
  users: User[] = [];

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}
```

### 3. Vue.js с TypeScript

```typescript
// Composition API с TypeScript
import { ref, reactive, computed, onMounted } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

const useUsers = () => {
  const state = reactive<UserState>({
    users: [],
    loading: false,
    error: null,
  });

  const activeUsers = computed(() => 
    state.users.filter(user => user.email.includes('@'))
  );

  const fetchUsers = async () => {
    state.loading = true;
    state.error = null;
    
    try {
      // Симуляция API вызова
      const response = await fetch('/api/users');
      state.users = await response.json();
    } catch (error) {
      state.error = (error as Error).message;
    } finally {
      state.loading = false;
    }
  };

  onMounted(fetchUsers);

  return {
    ...toRefs(state),
    activeUsers,
    fetchUsers,
  };
};

// Использование в компоненте
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const { users, loading, error, fetchUsers } = useUsers();
    
    return {
      users,
      loading,
      error,
      fetchUsers,
    };
  },
  template: `
    <div>
      <button @click="fetchUsers" :disabled="loading">
        {{ loading ? 'Загрузка...' : 'Загрузить пользователей' }}
      </button>
      
      <div v-if="error" class="error">
        {{ error }}
      </div>
      
      <div v-for="user in users" :key="user.id" class="user-card">
        <h3>{{ user.name }}</h3>
        <p>{{ user.email }}</p>
      </div>
    </div>
  `
});
```

## Серверные фреймворки

### 1. Node.js с Express

```typescript
import express, { Request, Response, NextFunction } from 'express';
import { body, validationResult } from 'express-validator';

interface User {
  id: number;
  name: string;
  email: string;
}

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

const app = express();
app.use(express.json());

// Моковая база данных
let users: User[] = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
];

// Валидация для создания пользователя
const createUserValidation = [
  body('name').isString().isLength({ min: 2 }).withMessage('Имя должно содержать не менее 2 символов'),
  body('email').isEmail().withMessage('Неверный формат email')
];

// Получить всех пользователей
app.get('/users', (req: Request, res: Response<ApiResponse<User[]>>) => {
  res.json({ success: true, data: users });
});

// Создать пользователя
app.post('/users', createUserValidation, (req: Request, res: Response<ApiResponse<User>>) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ 
      success: false, 
      error: errors.array()[0].msg 
    });
  }
  
  const { name, email } = req.body;
  const newUser: User = {
    id: users.length + 1,
    name,
    email
  };
  
  users.push(newUser);
  res.status(201).json({ success: true, data: newUser });
});

// Получить пользователя по ID
app.get('/users/:id', (req: Request<{ id: string }>, res: Response<ApiResponse<User>>) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  
  if (!user) {
    return res.status(404).json({ success: false, error: 'Пользователь не найден' });
  }
  
  res.json({ success: true, data: user });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Сервер запущен на порту ${PORT}`);
});
```

### 2. NestJS

```typescript
import { Controller, Get, Post, Body, Param, Put, Delete, ValidationPipe } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { User } from './entities/user.entity';

@Controller('users')
export class UsersController {
  private users: User[] = [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
  ];

  @Get()
  findAll(): User[] {
    return this.users;
  }

  @Get(':id')
  findOne(@Param('id') id: string): User {
    const userId = parseInt(id);
    const user = this.users.find(u => u.id === userId);
    
    if (!user) {
      throw new Error('Пользователь не найден');
    }
    
    return user;
  }

  @Post()
  create(@Body(ValidationPipe) createUserDto: CreateUserDto): User {
    const newUser: User = {
      id: this.users.length + 1,
      ...createUserDto
    };
    
    this.users.push(newUser);
    return newUser;
  }

  @Put(':id')
  update(
    @Param('id') id: string, 
    @Body(ValidationPipe) updateUserDto: UpdateUserDto
  ): User {
    const userId = parseInt(id);
    const index = this.users.findIndex(u => u.id === userId);
    
    if (index === -1) {
      throw new Error('Пользователь не найден');
    }
    
    this.users[index] = { ...this.users[index], ...updateUserDto };
    return this.users[index];
  }

  @Delete(':id')
  remove(@Param('id') id: string): void {
    const userId = parseInt(id);
    const index = this.users.findIndex(u => u.id === userId);
    
    if (index === -1) {
      throw new Error('Пользователь не найден');
    }
    
    this.users.splice(index, 1);
  }
}

// DTO (Data Transfer Object)
export class CreateUserDto {
  name: string;
  email: string;
}

export class UpdateUserDto {
  name?: string;
  email?: string;
}

// Entity
export class User {
  id: number;
  name: string;
  email: string;
}
```

## Базы данных

### 1. TypeORM

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, BaseEntity } from 'typeorm';

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 100 })
  name: string;

  @Column('varchar', { unique: true })
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Использование
import { createConnection, getConnection } from 'typeorm';

async function example() {
  const connection = await createConnection({
    type: 'postgres',
    host: 'localhost',
    port: 5432,
    username: 'test',
    password: 'test',
    database: 'test',
    entities: [User],
    synchronize: true,
  });

  // Создание пользователя
  const user = new User();
  user.name = 'John Doe';
  user.email = 'john@example.com';
  await user.save();

  // Получение пользователей
  const users = await User.find({ where: { isActive: true } });
  console.log(users);

  // Поиск одного пользователя
  const foundUser = await User.findOne({ where: { email: 'john@example.com' } });
  console.log(foundUser);
}
```

### 2. Prisma

```typescript
// schema.prisma
// generator client {
//   provider = "prisma-client-js"
// }
// 
// datasource db {
//   provider = "postgresql"
//   url      = env("DATABASE_URL")
// }
// 
// model User {
//   id        Int      @id @default(autoincrement())
//   name      String
//   email     String   @unique
//   createdAt DateTime @default(now())
//   updatedAt DateTime @updatedAt
// }

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function example() {
  // Создание пользователя
  const user = await prisma.user.create({
    data: {
      name: 'John Doe',
      email: 'john@example.com',
    },
  });

  // Получение пользователей
  const users = await prisma.user.findMany({
    where: {
      email: {
        contains: '@example.com',
      },
    },
  });

  // Обновление пользователя
  const updatedUser = await prisma.user.update({
    where: { id: user.id },
    data: { name: 'John Smith' },
  });

  console.log({ user, users, updatedUser });
}
```

## Документация и генерация

### 1. Swagger/OpenAPI с TypeScript

```typescript
import { INestApplication } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function setupSwagger(app: INestApplication) {
  const config = new DocumentBuilder()
    .setTitle('API Example')
    .setDescription('API documentation')
    .setVersion('1.0')
    .addTag('users')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);
}

// В контроллере
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';

@ApiTags('users')
@Controller('users')
export class UsersController {
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'Return all users.', type: [User] })
  @Get()
  findAll(): User[] {
    return [];
  }
}
```

### 2. JSDoc с TypeScript

```typescript
/**
 * Интерфейс пользователя
 * @interface User
 * @property {number} id - Уникальный идентификатор пользователя
 * @property {string} name - Имя пользователя
 * @property {string} email - Email пользователя
 */
interface User {
  id: number;
  name: string;
  email: string;
}

/**
 * Класс для управления пользователями
 * @class UserManager
 */
class UserManager {
  private users: User[] = [];

  /**
   * Добавляет нового пользователя
   * @param {User} user - Пользователь для добавления
   * @returns {number} ID добавленного пользователя
   */
  addUser(user: User): number {
    const newId = this.users.length + 1;
    this.users.push({ ...user, id: newId });
    return newId;
  }

  /**
   * Находит пользователя по ID
   * @param {number} id - ID пользователя для поиска
   * @returns {User | undefined} Найденный пользователь или undefined
   */
  findUser(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }
}
```

## Лучшие практики и шаблоны

### 1. Репозиторий паттерн

```typescript
// Интерфейс репозитория
interface Repository<T, ID> {
  findById(id: ID): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  update(id: ID, entity: Partial<T>): Promise<T | null>;
  delete(id: ID): Promise<boolean>;
}

// Базовый класс репозитория
abstract class BaseRepository<T, ID> implements Repository<T, ID> {
  protected abstract getEntityName(): string;
  protected abstract getDatabaseConnection(): any; // Заменить на реальную реализацию

  async findById(id: ID): Promise<T | null> {
    // Реализация поиска по ID
    const connection = this.getDatabaseConnection();
    return connection.get(this.getEntityName(), id);
  }

  async findAll(): Promise<T[]> {
    // Реализация получения всех сущностей
    const connection = this.getDatabaseConnection();
    return connection.getAll(this.getEntityName());
  }

  async save(entity: T): Promise<T> {
    // Реализация сохранения сущности
    const connection = this.getDatabaseConnection();
    return connection.save(this.getEntityName(), entity);
  }

  async update(id: ID, entity: Partial<T>): Promise<T | null> {
    // Реализация обновления сущности
    const connection = this.getDatabaseConnection();
    const existing = await this.findById(id);
    if (!existing) return null;
    
    const updated = { ...existing, ...entity };
    return connection.update(this.getEntityName(), id, updated);
  }

  async delete(id: ID): Promise<boolean> {
    // Реализация удаления сущности
    const connection = this.getDatabaseConnection();
    return connection.delete(this.getEntityName(), id);
  }
}

// Конкретная реализация
interface User {
  id: number;
  name: string;
  email: string;
}

class UserRepository extends BaseRepository<User, number> {
  protected getEntityName(): string {
    return 'User';
  }

  protected getDatabaseConnection(): any {
    // Возвращаем реальное соединение с базой данных
    return null; // Заглушка
  }
}
```

### 2. Dependency Injection

```typescript
// Интерфейсы
interface Logger {
  log(message: string): void;
  error(message: string): void;
}

interface Database {
  connect(): Promise<void>;
  query(sql: string): Promise<any>;
}

// Реализации
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
  
  error(message: string): void {
    console.error(`[ERROR] ${message}`);
  }
}

class PostgreSQLDatabase implements Database {
  async connect(): Promise<void> {
    console.log('Подключение к PostgreSQL...');
  }
  
  async query(sql: string): Promise<any> {
    console.log(`Выполнение запроса: ${sql}`);
    return []; // Заглушка
  }
}

// Контейнер внедрения зависимостей
class DIContainer {
  private services = new Map<string, any>();
  
  register<T>(token: string, service: T): void {
    this.services.set(token, service);
  }
  
  resolve<T>(token: string): T {
    const service = this.services.get(token);
    if (!service) {
      throw new Error(`Сервис ${token} не зарегистрирован`);
    }
    return service;
  }
}

// Использование
const container = new DIContainer();
container.register<Logger>('logger', new ConsoleLogger());
container.register<Database>('database', new PostgreSQLDatabase());

class UserService {
  constructor(
    private logger: Logger,
    private database: Database
  ) {}
  
  async createUser(userData: any): Promise<void> {
    this.logger.log('Создание пользователя...');
    await this.database.query('INSERT INTO users ...');
    this.logger.log('Пользователь создан');
  }
}

// Получение сервиса из контейнера
const userService = new UserService(
  container.resolve<Logger>('logger'),
  container.resolve<Database>('database')
);
```

## Лучшие практики

1. **Используйте современные инструменты сборки** - Vite, esbuild, SWC для улучшения производительности
2. **Настройте статический анализ** - ESLint, Prettier для поддержания качества кода
3. **Тестируйте типы** - используйте утилиты для проверки типов
4. **Документируйте API** - используйте Swagger, JSDoc или другие инструменты
5. **Используйте паттерны проектирования** - особенно полезные в TypeScript
6. **Следите за совместимостью** - особенно при работе с различными версиями и платформами

## Связи с другими концепциями

- [[ts/configuration/advanced-config-comprehensive|Конфигурация TypeScript]] - настройка инструментов
- [[build-tools/typescript-compiler]] - интеграция с системами сборки
- [[architecture/component-architecture]] - архитектурные аспекты
- [[testing/unit-testing]] - тестирование TypeScript приложений
- [[deployment/typescript-apps]] - развертывание TypeScript приложений