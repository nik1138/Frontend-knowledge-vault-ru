# TypeScript во фреймворках

TypeScript интегрируется с различными фреймворками, обеспечивая типобезопасность и лучший опыт разработки. В этом документе рассмотрим основные фреймворки и особенности их использования с TypeScript.

## TypeScript в веб-фреймворках

### Angular

Angular изначально разработан на TypeScript и тесно интегрирован с ним:

```typescript
// app.component.ts
import { Component } from '@angular/core';

export interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-root',
  template: `
    <div *ngFor="let user of users">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `
})
export class AppComponent {
  users: User[] = [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' }
  ];
}
```

Использование сервисов с типами:

```typescript
// user.service.ts
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
  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }

  createUser(userData: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>('/api/users', userData);
  }
}
```

### React

React поддерживает TypeScript с помощью определений типов:

```tsx
// UserCard.tsx
import React from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  onDelete?: (id: number) => void;
}

export const UserCard: React.FC<UserCardProps> = ({ 
  user, onEdit, onDelete 
}) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit?.(user)}>Edit</button>
      <button onClick={() => onDelete?.(user.id)}>Delete</button>
    </div>
  );
};
```

React Hooks с типами:

```tsx
// useUsers.ts
import { useState, useEffect } from 'react';

export interface User {
  id: number;
  name: string;
  email: string;
}

export const useUsers = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        const data: User[] = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  return { users, loading, error };
};
```

### Vue.js

Vue 3 предоставляет отличную поддержку TypeScript:

```typescript
// UserComponent.vue
<script setup lang="ts">
import { ref, onMounted } from 'vue';

export interface User {
  id: number;
  name: string;
  email: string;
}

const users = ref<User[]>([]);
const loading = ref<boolean>(true);

const fetchUsers = async (): Promise<void> => {
  try {
    const response = await fetch('/api/users');
    users.value = await response.json() as User[];
  } catch (error) {
    console.error('Failed to fetch users:', error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchUsers();
});
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else>
    <div v-for="user in users" :key="user.id">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  </div>
</template>
```

Composition API с типами:

```typescript
// userStore.ts (используя Pinia)
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export interface User {
  id: number;
  name: string;
  email: string;
}

export const useUserStore = defineStore('user', () => {
  const users = ref<User[]>([]);
  const loading = ref<boolean>(false);
  
  const getUsers = computed(() => users.value);
  const getUserById = computed(() => (id: number) => 
    users.value.find(user => user.id === id)
  );

  const fetchUsers = async (): Promise<void> => {
    loading.value = true;
    try {
      const response = await fetch('/api/users');
      users.value = await response.json() as User[];
    } finally {
      loading.value = false;
    }
  };

  const addUser = (user: Omit<User, 'id'>): void => {
    const newUser: User = { ...user, id: Date.now() };
    users.value.push(newUser);
  };

  return {
    users,
    loading,
    getUsers,
    getUserById,
    fetchUsers,
    addUser
  };
});
```

## Фреймворки для бэкенда

### Node.js: Express с TypeScript

```typescript
// server.ts
import express, { Request, Response } from 'express';
import { User } from './models/User';

const app = express();
app.use(express.json());

// Типизированный маршрут
app.get('/users/:id', async (req: Request, res: Response) => {
  try {
    const id = parseInt(req.params.id);
    if (isNaN(id)) {
      return res.status(400).json({ error: 'Invalid ID' });
    }
    
    const user: User | null = await findUserById(id);
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

### NestJS

Фреймворк, созданный с учетом TypeScript:

```typescript
// user.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User, UserDocument } from './schemas/user.schema';

export type CreateUserInput = Omit<User, 'id'>;

@Injectable()
export class UserService {
  constructor(
    @InjectModel(User.name) private userModel: Model<UserDocument>,
  ) {}

  async create(createUserDto: CreateUserInput): Promise<User> {
    const createdUser = new this.userModel(createUserDto);
    return createdUser.save();
  }

  async findAll(): Promise<User[]> {
    return this.userModel.find();
  }

  async findOne(id: string): Promise<User> {
    const user = await this.userModel.findById(id);
    if (!user) {
      throw new NotFoundException(`User #${id} not found`);
    }
    return user;
  }
}
```

### Fastify с TypeScript

```typescript
// server.ts
import fastify, { FastifyInstance } from 'fastify';
import { User } from './types/User';

const server: FastifyInstance = fastify({ logger: true });

// Определение схемы для валидации и типизации
const userSchema = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'number' },
          name: { type: 'string' },
          email: { type: 'string' }
        }
      }
    }
  }
};

server.get<{ Params: { id: number } }>('/users/:id', userSchema, async (request, reply) => {
  const id = request.params.id;
  const user: User = await getUserById(id);
  
  return user;
});

server.listen({ port: 3000 }, (err, address) => {
  if (err) {
    server.log.error(err);
    process.exit(1);
  }
  console.log(`Server listening at ${address}`);
});
```

## Типобезопасные фреймворки

### SvelteKit с TypeScript

```typescript
// src/routes/users/[id]/+page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ params }) => {
  const response = await fetch(`/api/users/${params.id}`);
  const user: { id: number; name: string; email: string } = await response.json();
  
  return {
    user
  };
};
```

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
  
  const { user } = data;
</script>

<h1>{user.name}</h1>
<p>{user.email}</p>
```

## Фреймворки для мобильных приложений

### React Native

```tsx
// UserList.tsx
import React from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';

export interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  users: User[];
  onUserPress: (user: User) => void;
}

const UserList: React.FC<UserListProps> = ({ users, onUserPress }) => {
  const renderItem = ({ item }: { item: User }) => (
    <View style={styles.userContainer}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userEmail}>{item.email}</Text>
    </View>
  );

  return (
    <FlatList
      data={users}
      renderItem={renderItem}
      keyExtractor={item => item.id.toString()}
    />
  );
};

const styles = StyleSheet.create({
  userContainer: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
  },
  userName: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  userEmail: {
    fontSize: 14,
    color: '#666',
  },
});

export default UserList;
```

## Интеграция TypeScript с фреймворками

### Специфические особенности

#### React
- `React.FC` и `React.FunctionComponent` для компонентов
- Типы пропсов и состояния
- Использование `useContext`, `useReducer` с типами

#### Angular
- Декораторы с типами (`@Input`, `@Output`)
- Инъекция зависимостей
- Типизация маршрутов

#### Vue
- Типизация в Composition API
- Типы для props, emits и refs
- Использование defineComponent

### Плагины и расширения

```typescript
// Конфигурация TypeScript для использования с фреймворками
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"  // для React
  },
  "include": [
    "src"
  ]
}
```

## Лучшие практики

### Повторное использование типов

```typescript
// shared/types.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  limit: number;
}

// Использование в разных фреймворках
// Angular
interface UserService {
  getUsers(): Observable<ApiResponse<User[]>>;
}

// React
const useUsers = (): UseQueryResult<ApiResponse<User[]>> => {
  // реализация
};
```

### Обработка ошибок

```typescript
// error-handler.ts
export interface ApiError {
  message: string;
  code: string;
  details?: Record<string, any>;
}

export const handleApiError = (error: unknown): ApiError => {
  if (error instanceof Error) {
    return {
      message: error.message,
      code: 'CLIENT_ERROR'
    };
  }
  
  return {
    message: 'An unknown error occurred',
    code: 'UNKNOWN_ERROR'
  };
};
```

## Современные тенденции

### Meta Frameworks

#### Next.js
```typescript
// pages/api/users/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { User } from '../../../types/User';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<User | { error: string }>
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { id } = req.query;
    const user = await getUserById(Number(id));
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.status(200).json(user);
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

#### Nuxt.js
```typescript
// server/api/users/[id].ts
import { User } from '~/types/User';

export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id');
  const userId = parseInt(id as string);

  if (isNaN(userId)) {
    throw createError({
      statusCode: 400,
      statusMessage: 'Invalid user ID'
    });
  }

  const user: User | null = await findUserById(userId);

  if (!user) {
    throw createError({
      statusCode: 404,
      statusMessage: 'User not found'
    });
  }

  return user;
});
```

## Связь с другими концепциями

- [[../react/index]] - специфика использования TypeScript с React
- [[../vue/vue]] - специфика использования TypeScript с Vue
- [[error-handling/framework-errors]] - обработка ошибок во фреймворках
- [[tooling-workflows/framework-workflows]] - рабочие процессы с фреймворками
- [[api-integration/typed-apis]] - типизированная работа с API