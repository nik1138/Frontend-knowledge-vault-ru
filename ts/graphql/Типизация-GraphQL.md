---
aliases: ["GraphQL TypeScript", "Типизация GraphQL", "GraphQL Types"]
tags: [programming, graphql, typescript, typing]
---

# Типизация GraphQL в TypeScript

## Обзор

Типизация GraphQL в TypeScript позволяет создавать безопасные типы для запросов, мутаций и схем. Это обеспечивает лучшую проверку типов, автодополнение и отладку в процессе разработки.

## Основы типизации GraphQL

GraphQL по своей сути предоставляет строгую систему типов для API, а TypeScript усиливает эту систему на этапе компиляции. Комбинация этих двух систем позволяет разработчикам создавать более надежные и предсказуемые приложения.

### Примеры базовых типов GraphQL

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  isActive: Boolean
  createdAt: String
}
```

Этот GraphQL-тип будет представлен в TypeScript как:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number | null;
  isActive?: boolean | null;
  createdAt?: string | null;
}
```

## Создание типов на основе схемы

Для генерации типов из схемы GraphQL можно использовать инструменты кодогенерации, такие как [[Codegen]].

### Запросы и мутации

При работе с запросами и мутациями важно типизировать как входные данные, так и ожидаемый результат:

```typescript
// Типизация для запроса
interface GetUserQueryVariables {
  id: string;
}

interface GetUserQueryResult {
  user: User | null;
}

// Типизация для мутации
interface CreateUserMutationVariables {
  input: {
    name: string;
    email: string;
  };
}

interface CreateUserMutationResult {
  createUser: User;
}
```

## Использование Generic-типов

Для более гибкой типизации можно использовать дженерики:

```typescript
interface GraphQLResponse<T> {
  data?: T;
  errors?: Array<{
    message: string;
    locations?: Array<{ line: number; column: number }>;
    path?: string[];
  }>;
}

interface GraphQLRequest<TVariables, TData> {
  query: string;
  variables?: TVariables;
  data?: TData;
}
```

## Работа с Union и Interface типами

GraphQL поддерживает сложные типы, такие как Union и Interface, которые требуют особого подхода к типизации:

```typescript
// Union тип
type SearchResult = User | Post | Comment;

// В TypeScript это может быть представлено как:
type SearchResult = User | Post | Comment;

// Interface тип
interface Node {
  id: string;
}

interface User extends Node {
  name: string;
}

interface Post extends Node {
  title: string;
}
```

## Практические рекомендации

- Используйте [[Схемы|GraphQL схемы]] для генерации типов
- Применяйте строгую типизацию для всех входных параметров
- Используйте необязательные поля (nullable) там, где это необходимо
- Создавайте отдельные интерфейсы для переменных запросов и результатов
- Регулярно обновляйте типы при изменении схемы

## Связанные темы

- [[Codegen]] - генерация типов
- [[Resolvers]] - реализация резолверов
- [[Схемы]] - определение схем
- [[Клиенты]] - работа с клиентами GraphQL