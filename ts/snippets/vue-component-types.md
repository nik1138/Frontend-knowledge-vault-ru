---
aliases: ["Типы для компонентов Vue", "TS Vue Component Types"]
tags: [typescript, vue, components, frontend]
---

# Типы для компонентов Vue

## Введение

Типизация компонентов Vue с использованием TypeScript обеспечивает безопасность типов, улучшает автодополнение в IDE и уменьшает количество ошибок. В этом разделе рассмотрим различные паттерны типизации компонентов Vue 3 с Composition API и Options API.

## Базовые типы для компонентов

### Composition API с defineComponent

```typescript
import { defineComponent, ref, computed, PropType } from 'vue';

// Простой компонент с Composition API
export default defineComponent({
  name: 'HelloWorld',
  props: {
    name: {
      type: String as PropType<string>,
      required: true
    },
    age: {
      type: Number as PropType<number>,
      default: 0
    }
  },
  setup(props) {
    // Реактивное состояние
    const message = ref<string>('Hello');
    const upperMessage = computed<string>(() => message.value.toUpperCase());
    
    // Методы
    const greet = (): void => {
      console.log(`${message.value}, ${props.name}!`);
    };
    
    return {
      message,
      upperMessage,
      greet
    };
  }
});
```

### Типизация Props

```typescript
import { defineComponent, PropType } from 'vue';

// Интерфейс для пропсов
interface User {
  id: number;
  name: string;
  email: string;
}

// Определяем типы пропсов
const userCardProps = {
  user: {
    type: Object as PropType<User>,
    required: true
  },
  showEmail: {
    type: Boolean as PropType<boolean>,
    default: false
  },
  tags: {
    type: Array as PropType<string[]>,
    default: () => []
  },
  userType: {
    type: String as PropType<'admin' | 'user' | 'moderator'>,
    default: 'user'
  }
} as const;

export default defineComponent({
  name: 'UserCard',
  props: userCardProps,
  setup(props) {
    // Теперь props имеет правильные типы
    const displayName = computed(() => props.user.name.toUpperCase());
    
    return {
      displayName
    };
  }
});
```

## Продвинутые паттерны

### Компоненты с дженериками

```typescript
import { defineComponent, PropType, computed } from 'vue';

// Дженерик-компонент для отображения списков
export default defineComponent({
  name: 'GenericList',
  props: {
    items: {
      type: Array as PropType<any[]>,
      required: true
    },
    renderItem: {
      type: Function as unknown as PropType<(item: any, index: number) => any>,
      required: true
    },
    keyField: {
      type: String,
      default: 'id'
    }
  },
  setup(props) {
    const keyedItems = computed(() => {
      return props.items.map((item, index) => ({
        ...item,
        __key: item[props.keyField as keyof typeof item] || index
      }));
    });
    
    return {
      keyedItems
    };
  }
});

// Более типобезопасный дженерик-компонент
export function createGenericList<T>() {
  return defineComponent({
    name: 'TypedGenericList',
    props: {
      items: {
        type: Array as PropType<T[]>,
        required: true
      },
      renderItem: {
        type: Function as PropType<(item: T, index: number) => any>,
        required: true
      },
      keyField: {
        type: String as PropType<keyof T>,
        default: 'id'
      }
    },
    setup(props) {
      const keyedItems = computed(() => {
        return props.items.map((item, index) => ({
          ...item,
          __key: item[props.keyField] || index
        }));
      });
      
      return {
        keyedItems
      };
    }
  });
}
```

### Типизация событий компонента

```typescript
import { defineComponent, PropType } from 'vue';

// Типы для событий
interface UserCardEmits {
  (e: 'update:user', user: User): void;
  (e: 'delete:user', userId: number): void;
  (e: 'click:avatar', event: MouseEvent): void;
}

export default defineComponent({
  name: 'UserCard',
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    }
  },
  emits: {
    'update:user': (user: User) => user !== undefined,
    'delete:user': (userId: number) => typeof userId === 'number',
    'click:avatar': (event: MouseEvent) => event instanceof MouseEvent
  },
  setup(props, { emit }) {
    const updateUser = (updatedUser: User) => {
      emit('update:user', updatedUser);
    };
    
    const deleteUser = () => {
      emit('delete:user', props.user.id);
    };
    
    const handleAvatarClick = (event: MouseEvent) => {
      emit('click:avatar', event);
    };
    
    return {
      updateUser,
      deleteUser,
      handleAvatarClick
    };
  }
});
```

### Типизация с Provide/Inject

```typescript
import { defineComponent, provide, inject, InjectionKey, Ref } from 'vue';

// Определяем ключи для инъекций
const USER_SERVICE_KEY: InjectionKey<UserService> = Symbol('userService');
const USER_DATA_KEY: InjectionKey<Ref<User | null>> = Symbol('userData');

// Сервис пользователя
interface UserService {
  getCurrentUser(): Promise<User>;
  updateUser(userData: Partial<User>): Promise<User>;
  logout(): Promise<void>;
}

// Компонент, предоставляющий сервис
export const UserProvider = defineComponent({
  name: 'UserProvider',
  props: {
    initialUser: {
      type: Object as PropType<User>,
      default: null
    }
  },
  setup(props) {
    const userData = ref<User | null>(props.initialUser);
    
    const userService: UserService = {
      getCurrentUser: async () => {
        // Реализация получения пользователя
        return userData.value!;
      },
      updateUser: async (updatedData) => {
        // Реализация обновления пользователя
        if (userData.value) {
          Object.assign(userData.value, updatedData);
        }
        return userData.value!;
      },
      logout: async () => {
        userData.value = null;
      }
    };
    
    // Предоставляем сервис и данные
    provide(USER_SERVICE_KEY, userService);
    provide(USER_DATA_KEY, userData);
    
    return () => this.$slots.default?.();
  }
});

// Компонент, использующий инъекции
export const UserProfile = defineComponent({
  name: 'UserProfile',
  setup() {
    // Инжектируем сервис и данные
    const userService = inject(USER_SERVICE_KEY);
    const userData = inject(USER_DATA_KEY);
    
    if (!userService || !userData) {
      throw new Error('UserProfile must be used within UserProvider');
    }
    
    const updateName = async (newName: string) => {
      if (userData.value) {
        await userService.updateUser({ name: newName });
      }
    };
    
    return {
      userData,
      updateName
    };
  }
});
```

## Практические примеры

### Типизация формы с валидацией

```typescript
import { defineComponent, ref, reactive, PropType, computed } from 'vue';

// Типы для формы
interface FormField<T> {
  value: T;
  error?: string;
  isValid: boolean;
}

interface FormErrors<T> {
  [K in keyof T]?: string;
}

interface FormConfig<T> {
  validate?: (values: T) => FormErrors<T>;
  onSubmit: (values: T) => void | Promise<void>;
}

export default defineComponent({
  name: 'TypedForm',
  props: {
    initialValues: {
      type: Object as PropType<Record<string, any>>,
      required: true
    },
    config: {
      type: Object as PropType<FormConfig<any>>,
      required: true
    }
  },
  setup(props) {
    // Создаем реактивное состояние формы
    const formState = reactive<Record<string, any>>({ ...props.initialValues });
    const errors = ref<FormErrors<Record<string, any>>>({});
    const isSubmitting = ref(false);
    
    // Проверяем валидность формы
    const isValid = computed(() => {
      return Object.keys(errors.value).length === 0;
    });
    
    // Обработчик изменения значения
    const handleChange = (name: string, value: any) => {
      formState[name] = value;
      
      // Очищаем ошибку при изменении поля
      if (errors.value[name]) {
        delete errors.value[name];
      }
    };
    
    // Обработчик отправки формы
    const handleSubmit = async () => {
      if (props.config.validate) {
        const validationErrors = props.config.validate(formState);
        errors.value = validationErrors;
        
        if (Object.keys(validationErrors).length > 0) {
          return;
        }
      }
      
      isSubmitting.value = true;
      
      try {
        await Promise.resolve(props.config.onSubmit(formState));
      } catch (error) {
        console.error('Form submission error:', error);
      } finally {
        isSubmitting.value = false;
      }
    };
    
    return {
      formState,
      errors,
      isSubmitting,
      isValid,
      handleChange,
      handleSubmit
    };
  }
});
```

### Кастомные хуки с типами

```typescript
import { ref, onMounted, onUnmounted, Ref } from 'vue';

// Тип для результатов хука
interface UseApiResult<T> {
  data: Ref<T | null>;
  loading: Ref<boolean>;
  error: Ref<string | null>;
  refetch: () => Promise<void>;
}

// Кастомный хук для работы с API
export function useApi<T>(url: string): UseApiResult<T> {
  const data = ref<T | null>(null);
  const loading = ref<boolean>(true);
  const error = ref<string | null>(null);
  
  const fetchData = async (): Promise<void> => {
    loading.value = true;
    error.value = null;
    
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      data.value = await response.json() as T;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'An error occurred';
      data.value = null;
    } finally {
      loading.value = false;
    }
  };
  
  onMounted(() => {
    fetchData();
  });
  
  return {
    data,
    loading,
    error,
    refetch: fetchData
  };
}

// Использование хука в компоненте
export default defineComponent({
  name: 'UserComponent',
  props: {
    userId: {
      type: Number as PropType<number>,
      required: true
    }
  },
  setup(props) {
    const { data: user, loading, error, refetch } = useApi<User>(`/api/users/${props.userId}`);
    
    return {
      user,
      loading,
      error,
      refetch
    };
  }
});
```

### Типизация с Vuex/Pinia

```typescript
import { defineComponent, computed } from 'vue';
import { useStore } from 'vuex'; // или useStore из pinia

// Типы для хранилища
interface RootState {
  user: User | null;
  theme: 'light' | 'dark';
  notifications: Notification[];
}

interface Notification {
  id: number;
  message: string;
  type: 'info' | 'success' | 'warning' | 'error';
  timestamp: Date;
}

export default defineComponent({
  name: 'UserProfileWithStore',
  setup() {
    const store = useStore<RootState>();
    
    // Типизированные вычисляемые свойства
    const currentUser = computed(() => store.state.user);
    const currentTheme = computed(() => store.state.theme);
    const notifications = computed(() => store.state.notifications);
    
    // Типизированные действия
    const updateUser = (userData: Partial<User>) => {
      store.dispatch('updateUser', userData);
    };
    
    const toggleTheme = () => {
      const newTheme = store.state.theme === 'light' ? 'dark' : 'light';
      store.dispatch('setTheme', newTheme);
    };
    
    return {
      currentUser,
      currentTheme,
      notifications,
      updateUser,
      toggleTheme
    };
  }
});
```

## Заключение

Типизация компонентов Vue с использованием TypeScript значительно улучшает качество кода, обеспечивает безопасность типов и упрощает поддержку приложений. Используйте эти паттерны как основу для создания типизированных компонентов в ваших Vue-приложениях.

См. также: [[react-component-types]] | [[api-type-definitions]] | [[validation-utilities]]