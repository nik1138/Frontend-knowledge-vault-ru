---
aliases: ["GraphQL Codegen", "Генерация кода GraphQL", "GraphQL Types Generator"]
tags: [programming, graphql, typescript, codegen, automation]
---

# GraphQL Codegen

## Обзор

GraphQL Codegen - это мощный инструмент для автоматической генерации TypeScript типов, хуков, компонентов и других артефактов на основе GraphQL схемы и операций. Он значительно упрощает разработку и обеспечивает согласованность между клиентом и сервером.

## Установка и настройка

Для начала работы с GraphQL Codegen установите необходимые пакеты:

```bash
npm install @graphql-codegen/cli @graphql-codegen/typescript --save-dev
```

Создайте конфигурационный файл `codegen.yml`:

```yaml
overwrite: true
schema: "http://localhost:4000/graphql"
documents: "src/**/*.graphql"
generates:
  src/generated/graphql.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
    config:
      withHooks: true
```

## Основные возможности

### Генерация типов

GraphQL Codegen может генерировать:

- Типы для всех типов в схеме
- Типы для запросов, мутаций и сабскрипшенов
- Типы переменных операций
- Интерфейсы для результатов операций

Пример сгенерированного кода:

```typescript
export type GetUserQuery = {
  __typename?: 'Query';
  user?: {
    __typename?: 'User';
    id: string;
    name: string;
    email: string;
  } | null;
};

export type GetUserQueryVariables = {
  id: string;
};
```

### Генерация хуков

При использовании Apollo Client, Codegen может генерировать React-хуки:

```typescript
export function useGetUserQuery(baseOptions?: Apollo.QueryHookOptions<GetUserQuery, GetUserQueryVariables>) {
  return Apollo.useQuery<GetUserQuery, GetUserQueryVariables>(GetUserDocument, baseOptions);
}
```

## Плагины

GraphQL Codegen поддерживает множество плагинов:

- `typescript` - генерация базовых TypeScript типов
- `typescript-operations` - типы для операций
- `typescript-react-apollo` - React хуки для Apollo Client
- `typescript-graphql-request` - для использования с graphql-request
- `typescript-resolvers` - типы для резолверов сервера

## Практические примеры

### Пример схемы и операции

Допустим, у нас есть следующая схема:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User!
}

type Query {
  user(id: ID!): User
  posts: [Post!]!
}
```

И GraphQL операция:

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    posts {
      id
      title
    }
  }
}
```

GraphQL Codegen сгенерирует типы, которые обеспечат строгую типизацию при использовании этой операции.

## Интеграция с различными клиентами

### Apollo Client

Для интеграции с Apollo Client используйте плагин `typescript-react-apollo`:

```yaml
generates:
  src/generated/apollo.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
    config:
      withHooks: true
```

### URQL

Для URQL используйте соответствующий плагин:

```yaml
generates:
  src/generated/urql.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-urql"
```

## Автоматизация процесса

Добавьте скрипт в `package.json`:

```json
{
  "scripts": {
    "generate": "graphql-codegen",
    "watch": "graphql-codegen --watch"
  }
}
```

## Практические рекомендации

- Регулярно обновляйте сгенерированные файлы при изменении схемы
- Используйте `--watch` режим при разработке
- Проверяйте сгенерированные типы на соответствие ожидаемому поведению
- Используйте настройки конфигурации для кастомизации генерации
- Интегрируйте генерацию в процесс CI/CD

## Связанные темы

- [[Типизация-GraphQL]] - основы типизации
- [[Схемы]] - определение схем
- [[Клиенты]] - работа с клиентами GraphQL
- [[Resolvers]] - реализация резолверов