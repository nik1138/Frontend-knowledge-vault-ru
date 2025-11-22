---
aliases: ["GraphQL и глобальное состояние", "GraphQL State Management", "Apollo Client"]
tags: ["#frontend", "#state-management", "#graphql", "#apollo", "#javascript", "#react"]
---

# GraphQL и глобальное состояние

GraphQL предоставляет мощный подход к управлению глобальным состоянием в современных веб-приложениях. В отличие от традиционных решений, GraphQL позволяет объединить управление данными с их получением, создавая единый источник истины для приложения.

## Основные концепции

### Клиентский кеш как глобальное состояние

GraphQL-клиенты, такие как Apollo Client, используют локальный кеш для хранения полученных данных. Этот кеш действует как глобальное состояние, которое может быть использовано любыми компонентами приложения.

### Интеграция данных и состояния

GraphQL позволяет объединить:
- Данные, полученные с сервера
- Локальное состояние приложения
- Кэшированные результаты запросов

## Установка и настройка

```bash
npm install @apollo/client graphql
```

### Базовая настройка Apollo Client

```javascript
import { ApolloClient, InMemoryCache, gql, ApolloProvider } from '@apollo/client'

const client = new ApolloClient({
  uri: 'https://your-graphql-endpoint.com/graphql',
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          // Конфигурация кэширования
          todos: {
            merge(existing = [], incoming) {
              return [...existing, ...incoming]
            }
          }
        }
      }
    }
  })
})

// Обертывание приложения в провайдер
const App = () => (
  <ApolloProvider client={client}>
    <YourComponents />
  </ApolloProvider>
)
```

## Управление локальным состоянием

### Локальные поля

```javascript
import { gql, useQuery, useMutation } from '@apollo/client'

const TODO_QUERY = gql`
  query GetTodos {
    todos {
      id
      title
      completed
      # Локальное поле, не требующее серверного запроса
      isEditing @client
    }
  }
`

const TOGGLE_TODO = gql`
  mutation ToggleTodo($id: ID!) {
    toggleTodo(id: $id) @client {
      id
      completed
    }
  }
`
```

### Локальные мутации

```javascript
import { gql, useMutation } from '@apollo/client'

// Определение локальной мутации
const UPDATE_LOCAL_STATE = gql`
  mutation UpdateLocalState($theme: String!) {
    updateTheme(theme: $writeToStore
  }
`

// Использование в компоненте
const ThemeToggle = () => {
  const [updateTheme] = useMutation(UPDATE_LOCAL_STATE, {
    update(cache, { data: { updateTheme } }) {
      cache.writeQuery({
        query: gql`
          query GetTheme {
            theme @client
          }
        `,
        data: { theme: updateTheme }
      })
    }
  })

  return (
    <button onClick={() => updateTheme({ variables: { theme: 'dark' } })}>
      Переключить тему
    </button>
  )
}
```

## Кэширование и нормализация

### Нормализация данных

Apollo Client автоматически нормализует данные в кэше:

```javascript
// Запрос
const USER_WITH_POSTS = gql`
  query GetUserWithPosts($id: ID!) {
    user(id: $id) {
      id
      name
      posts {
        id
        title
        author {
          id
          name
        }
      }
    }
  }
`

// Данные в кэше будут нормализованы как:
// {
//   "User:1": { id: "1", name: "John", posts: [{ __ref: "Post:1" }] },
//   "Post:1": { id: "1", title: "Post 1", author: { __ref: "User:1" } }
// }
```

### Настройка кэширования

```javascript
const cache = new InMemoryCache({
  typePolicies: {
    User: {
      // Ключ для идентификации объектов
      keyFields: ['id'],
      fields: {
        // Кастомная логика для поля
        friends: {
          merge(existing = [], incoming) {
            return [...new Set([...existing, ...incoming])]
          }
        }
      }
    },
    Query: {
      fields: {
        // Кастомная логика для запросов
        feed: {
          keyArgs: ['type'], // Определение уникальности запроса
          merge(existing = [], incoming, { args }) {
            if (args.offset === 0) {
              return incoming
            }
            return [...existing, ...incoming]
          }
        }
      }
    }
  }
})
```

## Работа с локальным состоянием

### Apollo Client для локального состояния

```javascript
import { gql, useQuery, useMutation } from '@apollo/client'

// Определение локальных данных в схеме
const LOCAL_STATE_SCHEMA = gql`
  extend type Query {
    theme: String!
    notifications: [Notification!]!
    currentUser: User
  }

  extend type Mutation {
    toggleTheme: String!
    addNotification(message: String!): Notification!
    setCurrentUser(user: UserInput!): User!
  }

  type Notification {
    id: ID!
    message: String!
    read: Boolean!
  }
`

// Запрос локальных данных
const GET_LOCAL_STATE = gql`
  query GetLocalState {
    theme @client
    notifications @client {
      id
      message
      read
    }
    currentUser @client {
      id
      name
      email
    }
  }
`

// Использование в компоненте
const AppHeader = () => {
  const { data } = useQuery(GET_LOCAL_STATE)
  
  return (
    <header className={data.theme}>
      <h1>Привет, {data.currentUser?.name}!</h1>
      {/* Отображение уведомлений */}
    </header>
  )
}
```

### Локальные мутации и резолверы

```javascript
import { gql, useMutation } from '@apollo/client'

// Настройка резолверов для локальных мутаций
const resolvers = {
  Mutation: {
    toggleTheme: (root, args, { cache }) => {
      const { theme } = cache.readQuery({
        query: gql`
          query GetCurrentTheme {
            theme @client
          }
        `
      })
      
      const newTheme = theme === 'light' ? 'dark' : 'light'
      
      cache.writeQuery({
        query: gql`
          query GetCurrentTheme {
            theme @client
          }
        `,
        data: { theme: newTheme }
      })
      
      return newTheme
    },
    
    addNotification: (root, { message }, { cache }) => {
      const newNotification = {
        id: Math.random().toString(),
        message,
        read: false,
        __typename: 'Notification'
      }
      
      const { notifications } = cache.readQuery({
        query: gql`
          query GetNotifications {
            notifications @client
          }
        `
      })
      
      cache.writeQuery({
        query: gql`
          query GetNotifications {
            notifications @client
          }
        `,
        data: { notifications: [...notifications, newNotification] }
      })
      
      return newNotification
    }
  }
}

// Применение резолверов к клиенту
client.addResolvers(resolvers)
```

## Подходы к управлению состоянием

### Гибридный подход

Сочетание серверных и локальных данных:

```javascript
const COMBINED_QUERY = gql`
  query GetDashboard {
    # Данные с сервера
    currentUser {
      id
      name
      preferences
    }
    
    # Локальные данные
    theme @client
    sidebarCollapsed @client
    notifications @client {
      id
      message
      read
    }
  }
`

const Dashboard = () => {
  const { data, loading, error } = useQuery(COMBINED_QUERY)
  
  if (loading) return <div>Загрузка...</div>
  if (error) return <div>Ошибка: {error.message}</div>
  
  return (
    <div className={data.theme}>
      <Sidebar collapsed={data.sidebarCollapsed} />
      <MainContent user={data.currentUser} />
      <Notifications items={data.notifications} />
    </div>
  )
}
```

### Управление формами

```javascript
import { gql, useQuery, useMutation } from '@apollo/client'

// Локальное состояние формы
const FORM_STATE = gql`
  query GetFormState {
    formValues @client
    formErrors @client
    formSubmitting @client
  }
`

const UPDATE_FORM_VALUE = gql`
  mutation UpdateFormValue($field: String!, $value: String!) {
    updateFormValue(field: $field, value: $value) @client
  }
`

const ContactForm = () => {
  const { data } = useQuery(FORM_STATE)
  const [updateFormValue] = useMutation(UPDATE_FORM_VALUE)
  
  const handleChange = (field, value) => {
    updateFormValue({
      variables: { field, value }
    })
  }
  
  return (
    <form>
      <input
        value={data.formValues?.name || ''}
        onChange={(e) => handleChange('name', e.target.value)}
        placeholder="Имя"
      />
      <input
        value={data.formValues?.email || ''}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="Email"
      />
      {/* Остальные поля формы */}
    </form>
  )
}
```

## Продвинутые паттерны

### Подписки для реального времени

```javascript
import { gql, useSubscription } from '@apollo/client'

const NEW_NOTIFICATION_SUBSCRIPTION = gql`
  subscription OnNewNotification {
    newNotification {
      id
      message
      timestamp
    }
  }
`

const NotificationListener = () => {
  const { data } = useSubscription(NEW_NOTIFICATION_SUBSCRIPTION, {
    onSubscriptionData: ({ client, subscriptionData }) => {
      // Обновление кэша при получении новых данных
      const { newNotification } = subscriptionData.data
      
      client.cache.modify({
        fields: {
          notifications(existingNotifications = []) {
            return [...existingNotifications, newNotification]
          }
        }
      })
    }
  })
  
  return null // Компонент используется только для подписки
}
```

### Кэширование сложных структур

```javascript
const cache = new InMemoryCache({
  possibleTypes: {
    // Полиморфизм
    Character: ['Human', 'Droid', 'Starship']
  },
  typePolicies: {
    Query: {
      fields: {
        search: {
          // Кастомная логика для поисковых запросов
          keyArgs: ['text'],
          merge(existing, incoming, { args }) {
            if (!existing) return incoming
            
            const merged = existing.slice(0)
            
            // Добавление новых результатов
            for (const result of incoming) {
              if (!merged.some(item => item.id === result.id)) {
                merged.push(result)
              }
            }
            
            return merged
          }
        }
      }
    }
  }
})
```

## Сравнение с традиционными решениями

| Характеристика | GraphQL + Apollo | Redux | Context API | Zustand |
|----------------|------------------|-------|-------------|---------|
| Бойлерплейт | Средний | Высокий | Средний | Минимальный |
| Кэширование | Встроенное | Нужны дополнения | Нет | Нужны дополнения |
| Синхронизация данных | Автоматическая | Ручная | Ручная | Ручная |
| Поддержка оффлайн | Частичная | Полная | Нет | Ограниченная |
| Производительность | Высокая | Высокая | Зависит от реализации | Высокая |

## Преимущества GraphQL для управления состоянием

- **Единая схема**: Все данные описаны в одной схеме
- **Автоматическое кэширование**: Эффективное хранение и обновление данных
- **Нормализация**: Предотвращение дублирования данных
- **Синхронизация**: Автоматическое обновление компонентов при изменении данных
- **Гибкость**: Возможность комбинировать локальные и серверные данные

## Недостатки и ограничения

- **Сложность**: Более сложная настройка по сравнению с простыми решениями
- **Кривая обучения**: Требуется понимание GraphQL и его концепций
- **Размер**: Дополнительные зависимости увеличивают размер приложения
- **Ограничения локального состояния**: Не все сценарии удобно реализовать через GraphQL

## Лучшие практики

1. **Используйте локальные данные осознанно**: Не все состояние должно быть в GraphQL
2. **Настройте кэширование**: Правильно настройте `typePolicies` для оптимальной работы
3. **Комбинируйте с другими решениями**: Используйте GraphQL для данных, Context API для UI-состояния
4. **Документируйте локальные поля**: Используйте комментарии для обозначения локальных данных

## Когда использовать GraphQL для управления состоянием

- Приложения, активно использующие GraphQL API
- Когда нужна синхронизация данных между компонентами
- Для сложных сценариев кэширования
- Когда важна нормализация данных
- При необходимости оффлайн-функциональности

## Связанные концепции

- [[Redux]] - традиционный подход к управлению состоянием
- [[Zustand]] - простая альтернатива
- [[Context API]] - встроенный механизм React
- [[MobX]] - реактивное управление состоянием

## Теги

#frontend #javascript #react #state-management #graphql #apollo #cache #architecture