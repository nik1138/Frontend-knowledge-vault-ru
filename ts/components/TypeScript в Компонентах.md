# TypeScript в Компонентах

TypeScript предоставляет мощные возможности для типизации компонентов в современных фронтенд-фреймворках, что значительно улучшает качество кода, предотвращает ошибки и упрощает рефакторинг.

## Типизация Props

При создании компонентов важно четко определять типы принимаемых параметров (props). Это позволяет IDE предоставлять автодополнение и проверять корректность использования компонентов.

```typescript
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
  };
  showEmail?: boolean;
  onEdit?: (userId: number) => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, showEmail = true, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      {showEmail && <p>{user.email}</p>}
      {onEdit && <button onClick={() => onEdit(user.id)}>Edit</button>}
    </div>
  );
};
```

## Дженерики в Компонентах

TypeScript позволяет создавать гибкие компоненты с использованием дженериков:

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>(props: ListProps<T>) {
  return (
    <ul>
      {props.items.map((item, index) => (
        <li key={index}>{props.renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

## Типизация событий

Важно правильно типизировать обработчики событий:

```typescript
interface ButtonProps {
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({ onClick, children }) => {
  return <button onClick={onClick}>{children}</button>;
};
```

## Типизация состояния

TypeScript также помогает типизировать состояние компонентов:

```typescript
interface UserFormState {
  name: string;
  email: string;
  errors: {
    name?: string;
    email?: string;
  };
}

const UserForm: React.FC = () => {
  const [state, setState] = useState<UserFormState>({
    name: '',
    email: '',
    errors: {}
  });

  // Обработка изменений
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setState(prev => ({
      ...prev,
      [name]: value
    }));
  };

  return (
    <form>
      <input 
        type="text" 
        name="name" 
        value={state.name} 
        onChange={handleChange} 
      />
      <input 
        type="email" 
        name="email" 
        value={state.email} 
        onChange={handleChange} 
      />
    </form>
  );
};
```

## Типизация с Context API

При использовании Context API важно типизировать значения контекста:

```typescript
interface AuthContextType {
  user: User | null;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);
```

## Полезные утилиты

TypeScript предоставляет несколько полезных утилит для работы с типами компонентов:

- `Partial<T>` - делает все свойства опциональными
- `Pick<T, K>` - выбирает подмножество свойств
- `Omit<T, K>` - исключает определенные свойства
- `Record<K, T>` - создает тип с ключами K и значениями T

## Заключение

Использование TypeScript в компонентах позволяет:
- Повысить надежность кода
- Улучшить опыт разработки
- Снизить количество багов
- Облегчить рефакторинг