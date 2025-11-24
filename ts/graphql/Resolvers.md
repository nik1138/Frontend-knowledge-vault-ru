---
aliases: ["GraphQL Resolvers", "Резолверы GraphQL", "GraphQL Server Implementation"]
tags: [programming, graphql, typescript, resolvers, server]
---

# Resolvers в GraphQL

## Обзор

Резолверы (resolvers) - это функции, которые отвечают за получение данных для каждого поля в GraphQL-запросе. Они определяют, как значение для конкретного поля в схеме должно быть вычислено и возвращено клиенту.

## Структура резолверов

Каждый резолвер принимает 4 аргумента:

1. `parent` - результат родительского поля
2. `args` - аргументы, переданные в поле
3. `context` - объект контекста, доступный для всех резолверов
4. `info` - информация о поле и запросе

```typescript
type ResolverFn = (
  parent: any,
  args: { [argName: string]: any },
  context: Context,
  info: GraphQLResolveInfo
) => any;
```

## Примеры резолверов

### Базовый резолвер

```typescript
const resolvers = {
  Query: {
    user: (parent, args, context, info) => {
      return context.db.users.findById(args.id);
    },
    posts: (parent, args, context, info) => {
      return context.db.posts.findAll();
    }
  },
  
  Mutation: {
    createUser: (parent, args, context, info) => {
      return context.db.users.create(args.input);
    }
  },
  
  User: {
    posts: (user, args, context, info) => {
      return context.db.posts.findByUserId(user.id);
    }
  }
};
```

## Типизация резолверов

Для обеспечения типобезопасности резолверов можно использовать сгенерированные типы. При использовании [[Codegen]] можно получить строго типизированные резолверы:

```typescript
import { Resolvers, User, Post } from './generated/graphql';

const resolvers: Resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      const user = await context.db.users.findById(id);
      return user;
    },
    
    posts: async (parent, args, context) => {
      const posts = await context.db.posts.findAll();
      return posts;
    }
  },
  
  User: {
    posts: async (user, args, context) => {
      const posts = await context.db.posts.findByUserId(user.id);
      return posts;
    }
  },
  
  Mutation: {
    createUser: async (parent, { input }, context) => {
      const newUser = await context.db.users.create(input);
      return newUser;
    }
  }
};
```

## Контекст в резолверах

Контекст - это объект, который доступен для всех резолверов и обычно содержит:

- Подключения к базе данных
- Аутентификационные данные
- Информацию о текущем пользователе
- Утилиты и сервисы

```typescript
interface Context {
  db: Database;
  user?: User;
  loaders: DataLoaderMap;
}

const context = ({ req }: { req: Request }): Context => {
  return {
    db: database,
    user: req.user,
    loaders: createLoaders()
  };
};
```

## DataLoader для оптимизации

Для предотвращения проблемы N+1 запросов рекомендуется использовать DataLoader:

```typescript
import DataLoader from 'dataloader';

const createLoaders = () => ({
  users: new DataLoader<string, User>(async (userIds) => {
    const users = await database.users.findByIds(userIds);
    return userIds.map(id => users.find(user => user.id === id));
  })
});

// Использование в резолвере
const resolvers = {
  Post: {
    author: async (post, args, { loaders }) => {
      return loaders.users.load(post.authorId);
    }
  }
};
```

## Обработка ошибок

В резолверах важно правильно обрабатывать ошибки:

```typescript
import { UserInputError, AuthenticationError, ForbiddenError } from 'apollo-server-express';

const resolvers = {
  Mutation: {
    updateUser: async (parent, { id, input }, context) => {
      if (!context.user) {
        throw new AuthenticationError('Вы должны быть авторизованы');
      }
      
      if (context.user.id !== id) {
        throw new ForbiddenError('У вас нет доступа к этому ресурсу');
      }
      
      try {
        const updatedUser = await context.db.users.update(id, input);
        return updatedUser;
      } catch (error) {
        if (error.code === 'VALIDATION_ERROR') {
          throw new UserInputError('Неверные данные пользователя');
        }
        throw error;
      }
    }
  }
};
```

## Асинхронные резолверы

Большинство резолверов являются асинхронными, так как часто взаимодействуют с базой данных:

```typescript
const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      // Асинхронный вызов к базе данных
      return await context.db.users.findById(id);
    },
    
    posts: async (parent, { limit = 10, offset = 0 }, context) => {
      return await context.db.posts.findMany({ limit, offset });
    }
  }
};
```

## Продвинутые паттерны

### Middleware для резолверов

Можно создать middleware для резолверов:

```typescript
const withAuth = (resolver) => async (parent, args, context, info) => {
  if (!context.user) {
    throw new AuthenticationError('Авторизация обязательна');
  }
  return resolver(parent, args, context, info);
};

const resolvers = {
  Query: {
    profile: withAuth(async (parent, args, context) => {
      return context.user;
    })
  }
};
```

## Практические рекомендации

- Используйте строгую типизацию для резолверов
- Реализуйте правильную обработку ошибок
- Используйте DataLoader для оптимизации запросов
- Разделяйте резолверы по доменам/модулям
- Документируйте сложные резолверы
- Покрывайте резолверы тестами

## Связанные темы

- [[Схемы]] - определение схем
- [[Типизация-GraphQL]] - типизация GraphQL
- [[Codegen]] - генерация кода
- [[Клиенты]] - работа с клиентами GraphQL