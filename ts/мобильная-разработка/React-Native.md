---
aliases: [React Native, RN]
tags: [mobile-development, react-native, typescript]
---

# React Native

React Native - это фреймворк для разработки кроссплатформенных мобильных приложений с использованием JavaScript и TypeScript. Он позволяет разработчикам использовать знания React для создания приложений, которые работают на iOS и Android.

## Основы React Native

React Native позволяет использовать компоненты React для создания нативных интерфейсов. Основные компоненты включают:

- `View` - аналог div в вебе
- `Text` - для отображения текста
- `Image` - для отображения изображений
- `Button`, `TextInput` - для интерактивных элементов

```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const App = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Привет, React Native!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default App;
```

## Типизация в React Native

TypeScript особенно полезен в React Native для обеспечения безопасности типов. Вот пример типизации компонента:

```typescript
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

interface Props {
  title: string;
  onPress: () => void;
}

interface State {
  count: number;
}

const CounterButton: React.FC<Props> = ({ title, onPress }) => {
  const [state, setState] = useState<State>({ count: 0 });

  return (
    <View style={styles.container}>
      <TouchableOpacity onPress={() => setState({ count: state.count + 1 })}>
        <Text style={styles.button}>{title}</Text>
      </TouchableOpacity>
      <Text>Счетчик: {state.count}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
    alignItems: 'center',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
    color: 'white',
  },
});

export default CounterButton;
```

## Навигация

Для навигации в React Native часто используется библиотека `react-navigation`:

```bash
npm install @react-navigation/native @react-navigation/stack
```

```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { View, Text, Button } from 'react-native';

const Stack = createStackNavigator();

const HomeScreen = ({ navigation }) => {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Экран "Главная"</Text>
      <Button
        title="Перейти к профилю"
        onPress={() => navigation.navigate('Profile')}
      />
    </View>
  );
};

const ProfileScreen = () => {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Экран "Профиль"</Text>
    </View>
  );
};

const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

## Работа с API

Для работы с API рекомендуется использовать библиотеки вроде `axios` или встроенный `fetch`:

```typescript
import axios from 'axios';

interface User {
  id: number;
  name: string;
  email: string;
}

const fetchUsers = async (): Promise<User[]> => {
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/users');
    return response.data;
  } catch (error) {
    console.error('Ошибка при получении пользователей:', error);
    return [];
  }
};

// Использование в компоненте
const UsersList = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadUsers = async () => {
      const data = await fetchUsers();
      setUsers(data);
      setLoading(false);
    };

    loadUsers();
  }, []);

  if (loading) {
    return <Text>Загрузка...</Text>;
  }

  return (
    <View>
      {users.map(user => (
        <Text key={user.id}>{user.name} - {user.email}</Text>
      ))}
    </View>
  );
};
```

## Стилизация

React Native поддерживает стилизацию через StyleSheet:

```typescript
import { StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F5FCFF',
    padding: 16,
  },
  text: {
    fontSize: 16,
    marginVertical: 8,
  },
});
```

Также можно использовать сторонние библиотеки стилизации, такие как `styled-components`:

```typescript
import styled from 'styled-components/native';

const Container = styled.View`
  flex: 1;
  justify-content: center;
  align-items: center;
  background-color: #f5fcff;
`;

const Title = styled.Text`
  font-size: 24px;
  font-weight: bold;
  color: #333;
`;
```

## Состояние приложения

Для управления сложным состоянием приложения можно использовать Redux или Context API:

```typescript
// Context API
import React, { createContext, useContext, useReducer } from 'react';

interface State {
  user: User | null;
  isAuthenticated: boolean;
}

interface Action {
  type: 'LOGIN' | 'LOGOUT';
  payload?: User;
}

const initialState: State = {
  user: null,
  isAuthenticated: false,
};

const AuthContext = createContext<{
  state: State;
  dispatch: React.Dispatch<Action>;
}>({
  state: initialState,
  dispatch: () => null,
});

const authReducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'LOGIN':
      return {
        user: action.payload || null,
        isAuthenticated: true,
      };
    case 'LOGOUT':
      return {
        user: null,
        isAuthenticated: false,
      };
    default:
      return state;
  }
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  return (
    <AuthContext.Provider value={{ state, dispatch }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

## Практические советы

- Используйте `Platform` модуль для определения платформы и адаптации UI
- Всегда используйте `Dimensions` для адаптивного дизайна
- Оптимизируйте производительность с помощью `React.memo` и `useCallback`
- Используйте `FlatList` вместо `ScrollView` для списков с большим количеством элементов

## Связанные темы

- [[Мобильные-паттерны]]
- [[Оптимизация-для-мобильных-устройств]]
- [[TypeScript-в-мобильной-разработке]]
- [[Redux-Native]]