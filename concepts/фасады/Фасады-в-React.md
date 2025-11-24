---
aliases: [Фасады в React, React Facades, Component Facades]
tags: [react, programming, design-patterns, facade, frontend, architecture]
---

# Фасады в React

Паттерн **Фасад** в React-приложениях используется для упрощения сложных компонентов, управления состоянием, интеграции с внешними API и других задач. Фасады в React помогают скрывать сложность внутренней реализации и предоставляют простой интерфейс для взаимодействия с компонентами и логикой приложения.

## Общее описание

В контексте React, фасады могут быть реализованы в различных формах:
- Компоненты, упрощающие сложные интерфейсы
- Хуки, инкапсулирующие сложную логику
- Сервисы, упрощающие работу с API
- Контексты, централизующие сложную логику управления состоянием

## Примеры использования фасадов в React

### 1. Компонент-фасад для сложных UI-библиотек

Многие UI-библиотеки предоставляют базовые компоненты, которые требуют сложной настройки. Фасад позволяет создать более простой интерфейс:

```jsx
// Сложный компонент DatePicker с множеством настроек
import { DatePicker as UIDatePicker } from 'ui-library';

// Фасад для упрощения использования
function SimpleDatePicker({ onChange, value, ...props }) {
  return (
    <UIDatePicker
      value={value}
      onChange={onChange}
      format="DD.MM.YYYY"
      allowClear
      showToday
      locale="ru-RU"
      style={{ width: '100%' }}
      {...props}
    />
  );
}

// Использование
function MyForm() {
  const [date, setDate] = useState(null);
  
  return (
    <SimpleDatePicker
      value={date}
      onChange={setDate}
      placeholder="Выберите дату"
    />
  );
}
```

### 2. Хук-фасад для сложной логики

```jsx
// Сложная логика управления формой
function useForm(initialValues, validationRules) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Валидация при изменении
    if (validationRules[name]) {
      const error = validationRules[name](value);
      setErrors(prev => ({
        ...prev,
        [name]: error
      }));
    }
  };
  
  const handleSubmit = async (onSubmit) => {
    // Валидация перед отправкой
    const newErrors = {};
    for (const [field, validator] of Object.entries(validationRules)) {
      const error = validator(values[field]);
      if (error) newErrors[field] = error;
    }
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } catch (error) {
      console.error('Ошибка при отправке:', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return {
    values,
    errors,
    isSubmitting,
    handleChange,
    handleSubmit
  };
}

// Использование хука-фасада
function UserForm({ onSubmit }) {
  const { values, errors, isSubmitting, handleChange, handleSubmit } = useForm(
    { name: '', email: '' },
    {
      name: (value) => !value ? 'Имя обязательно' : null,
      email: (value) => !value.includes('@') ? 'Некорректный email' : null
    }
  );
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleSubmit(onSubmit);
    }}>
      <input
        value={values.name}
        onChange={(e) => handleChange('name', e.target.value)}
        placeholder="Имя"
      />
      {errors.name && <span className="error">{errors.name}</span>}
      
      <input
        value={values.email}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="Email"
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Отправка...' : 'Отправить'}
      </button>
    </form>
  );
}
```

### 3. Фасад для управления состоянием

```jsx
// Фасад для управления глобальным состоянием
class StateFacade {
  constructor() {
    this.state = {};
    this.listeners = [];
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.notifyListeners();
  }
  
  getState() {
    return this.state;
  }
  
  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
  
  notifyListeners() {
    this.listeners.forEach(listener => listener(this.state));
  }
}

// Хук для использования фасада состояния
function useStateFacade(selector) {
  const [state, setState] = useState(facade.getState());
  
  useEffect(() => {
    const unsubscribe = facade.subscribe(setState);
    return unsubscribe;
  }, []);
  
  return selector ? selector(state) : state;
}

// Использование
const user = useStateFacade(state => state.user);
const theme = useStateFacade(state => state.theme);
```

## Примеры фасадов для работы с API в React

### 1. Кастомный хук-фасад для API

```jsx
// Хук-фасад для работы с API
function useAPI() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const apiCall = useCallback(async (apiFunction, ...args) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiFunction(...args);
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);
  
  return { loading, error, apiCall };
}

// Использование в компоненте
function UserList() {
  const [users, setUsers] = useState([]);
  const { loading, error, apiCall } = useAPI();
  
  useEffect(() => {
    apiCall(userAPI.getAll)
      .then(setUsers)
      .catch(console.error);
  }, []);
  
  if (loading) return <div>Загрузка пользователей...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 2. Фасад для управления асинхронными операциями

```jsx
// Фасад для управления асинхронными операциями
class AsyncOperationFacade {
  constructor() {
    this.operations = new Map();
  }
  
  async execute(key, asyncFunction, options = {}) {
    // Отмена предыдущей операции с тем же ключом
    if (this.operations.has(key)) {
      this.operations.get(key).abort();
    }
    
    const controller = new AbortController();
    this.operations.set(key, controller);
    
    try {
      const result = await asyncFunction(controller.signal);
      this.operations.delete(key);
      return result;
    } catch (error) {
      if (error.name === 'AbortError') {
        // Операция была отменена
        this.operations.delete(key);
        return null;
      }
      this.operations.delete(key);
      throw error;
    }
  }
  
  cancel(key) {
    if (this.operations.has(key)) {
      this.operations.get(key).abort();
      this.operations.delete(key);
    }
  }
  
  cancelAll() {
    for (const [key, controller] of this.operations) {
      controller.abort();
    }
    this.operations.clear();
  }
}

// Хук для использования фасада асинхронных операций
function useAsyncOperation() {
  const facadeRef = useRef(new AsyncOperationFacade());
  
  useEffect(() => {
    return () => {
      facadeRef.current.cancelAll();
    };
  }, []);
  
  return facadeRef.current;
}

// Использование в компоненте
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const asyncFacade = useAsyncOperation();
  
  const handleSearch = useCallback(async () => {
    if (!query) {
      setResults([]);
      return;
    }
    
    try {
      const data = await asyncFacade.execute(
        'search',
        async (signal) => {
          const response = await fetch(`/api/search?q=${query}`, { signal });
          return response.json();
        }
      );
      
      if (data) {
        setResults(data.results);
      }
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Ошибка поиска:', error);
      }
    }
  }, [query]);
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Поиск..."
      />
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Фасады для управления сложными формами

```jsx
// Фасад для управления сложными формами
class FormFacade {
  constructor(initialValues = {}) {
    this.state = {
      values: { ...initialValues },
      errors: {},
      touched: {},
      isValid: true
    };
    this.validators = {};
    this.subscribers = [];
  }
  
  setFieldValue(name, value) {
    this.state.values = { ...this.state.values, [name]: value };
    this.validateField(name, value);
    this.notifySubscribers();
  }
  
  setFieldError(name, error) {
    this.state.errors = { ...this.state.errors, [name]: error };
    this.state.isValid = Object.keys(this.state.errors).length === 0;
    this.notifySubscribers();
  }
  
  setFieldTouched(name) {
    this.state.touched = { ...this.state.touched, [name]: true };
    this.notifySubscribers();
  }
  
  addValidator(fieldName, validator) {
    this.validators[fieldName] = validator;
  }
  
  validateField(name, value) {
    const validator = this.validators[name];
    if (validator) {
      const error = validator(value, this.state.values);
      this.setFieldError(name, error);
    }
  }
  
  validateAll() {
    for (const [fieldName, value] of Object.entries(this.state.values)) {
      this.validateField(fieldName, value);
    }
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers() {
    this.subscribers.forEach(callback => callback(this.state));
  }
  
  getValues() {
    return this.state.values;
  }
  
  getErrors() {
    return this.state.errors;
  }
  
  isValid() {
    return this.state.isValid;
  }
}

// Хук для использования фасада форм
function useFormFacade(initialValues) {
  const facadeRef = useRef(new FormFacade(initialValues));
  const [state, setState] = useState(facadeRef.current.state);
  
  useEffect(() => {
    const unsubscribe = facadeRef.current.subscribe(setState);
    return unsubscribe;
  }, []);
  
  return facadeRef.current;
}

// Использование в компоненте
function ComplexForm() {
  const formFacade = useFormFacade({ name: '', email: '', phone: '' });
  
  useEffect(() => {
    formFacade.addValidator('email', (value) => {
      return !value.includes('@') ? 'Некорректный email' : null;
    });
    
    formFacade.addValidator('phone', (value) => {
      return value.length < 10 ? 'Номер слишком короткий' : null;
    });
  }, []);
  
  const handleSubmit = () => {
    formFacade.validateAll();
    if (formFacade.isValid()) {
      console.log('Форма валидна:', formFacade.getValues());
    }
  };
  
  return (
    <div>
      <input
        value={formFacade.state.values.name}
        onChange={(e) => formFacade.setFieldValue('name', e.target.value)}
        onBlur={() => formFacade.setFieldTouched('name')}
        placeholder="Имя"
      />
      
      <input
        value={formFacade.state.values.email}
        onChange={(e) => formFacade.setFieldValue('email', e.target.value)}
        onBlur={() => formFacade.setFieldTouched('email')}
        placeholder="Email"
      />
      {formFacade.state.errors.email && (
        <span className="error">{formFacade.state.errors.email}</span>
      )}
      
      <button onClick={handleSubmit}>Отправить</button>
    </div>
  );
}
```

## Фасады для управления анимациями

```jsx
// Фасад для управления анимациями
class AnimationFacade {
  constructor(elementRef) {
    this.elementRef = elementRef;
    this.currentAnimation = null;
  }
  
  async fadeIn(duration = 300) {
    if (this.currentAnimation) {
      this.currentAnimation.cancel();
    }
    
    this.elementRef.current.style.opacity = '0';
    this.elementRef.current.style.transition = `opacity ${duration}ms ease-in-out`;
    
    // Ждем следующего кадра перед началом анимации
    await new Promise(resolve => requestAnimationFrame(resolve));
    this.elementRef.current.style.opacity = '1';
    
    return new Promise(resolve => {
      setTimeout(resolve, duration);
    });
  }
  
  async fadeOut(duration = 300) {
    if (this.currentAnimation) {
      this.currentAnimation.cancel();
    }
    
    this.elementRef.current.style.transition = `opacity ${duration}ms ease-in-out`;
    this.elementRef.current.style.opacity = '0';
    
    return new Promise(resolve => {
      setTimeout(resolve, duration);
    });
  }
  
  async slideIn(direction = 'down', duration = 300) {
    const element = this.elementRef.current;
    
    switch (direction) {
      case 'up':
        element.style.transform = 'translateY(100%)';
        break;
      case 'down':
        element.style.transform = 'translateY(-100%)';
        break;
      case 'left':
        element.style.transform = 'translateX(100%)';
        break;
      case 'right':
        element.style.transform = 'translateX(-100%)';
        break;
    }
    
    element.style.transition = `transform ${duration}ms ease-in-out`;
    await new Promise(resolve => requestAnimationFrame(resolve));
    
    element.style.transform = 'translateY(0)';
    
    return new Promise(resolve => {
      setTimeout(resolve, duration);
    });
  }
}

// Хук для использования фасада анимаций
function useAnimation(elementRef) {
  const [animationFacade, setAnimationFacade] = useState(null);
  
  useEffect(() => {
    if (elementRef.current) {
      setAnimationFacade(new AnimationFacade(elementRef));
    }
  }, [elementRef]);
  
  return animationFacade;
}

// Использование в компоненте
function AnimatedComponent() {
  const elementRef = useRef(null);
  const animations = useAnimation(elementRef);
  
  const handleFadeIn = async () => {
    await animations?.fadeIn();
  };
  
  const handleSlideIn = async () => {
    await animations?.slideIn('up');
  };
  
  return (
    <div>
      <div ref={elementRef}>
        Анимируемый элемент
      </div>
      <button onClick={handleFadeIn}>Появиться</button>
      <button onClick={handleSlideIn}>Сдвинуться</button>
    </div>
  );
}
```

## Лучшие практики при создании фасадов в React

### 1. Используйте мемоизацию для производительности

```jsx
// Мемоизированный компонент-фасад
const MemoizedChart = memo(({ data, options }) => {
  // Сложная логика рендеринга диаграммы
  return <Chart data={data} options={options} />;
});

// Мемоизированный хук
const useMemoizedAPI = (endpoint) => {
  return useMemo(() => {
    return {
      get: () => fetchAPI.get(endpoint),
      post: (data) => fetchAPI.post(endpoint, data),
      put: (id, data) => fetchAPI.put(`${endpoint}/${id}`, data)
    };
  }, [endpoint]);
};
```

### 2. Обеспечьте гибкость через пропсы

```jsx
// Фасад с возможностью настройки
function FlexibleModal({ 
  children, 
  isOpen, 
  onClose, 
  overlay = true, 
  closeOnEscape = true,
  closeOnOutsideClick = true,
  ...props 
}) {
  // Реализация с учетом настроек
  useEffect(() => {
    if (!closeOnEscape) return;
    
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };
    
    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [closeOnEscape, onClose]);
  
  // Рендер с учетом всех настроек
  return (
    <div 
      style={{ display: isOpen ? 'block' : 'none' }}
      onClick={closeOnOutsideClick ? onClose : undefined}
    >
      {overlay && <div className="modal-overlay" />}
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
}
```

### 3. Обрабатывайте ошибки корректно

```jsx
// Фасад с обработкой ошибок
function useSafeAPI() {
  const [error, setError] = useState(null);
  
  const safeCall = useCallback(async (apiFunction, ...args) => {
    try {
      setError(null);
      return await apiFunction(...args);
    } catch (err) {
      setError(err);
      throw err; // Перебрасываем ошибку для обработки в вызывающем коде
    }
  }, []);
  
  return { error, safeCall };
}
```

## Заключение

Фасады в React являются мощным инструментом для упрощения сложных интерфейсов и управления внутренней логикой приложений. Они позволяют создавать более чистый, понятный и поддерживаемый код, скрывая сложность реализации за простыми интерфейсами. При правильном применении фасады улучшают архитектуру React-приложений и упрощают разработку компонентов.

## Связанные темы

- [[Паттерн-фасад]]
- [[Фасады-в-библиотеках]]
- [[Фасады-для-API]]
- [[React-хуки]]
- [[Управление-состоянием-в-React]]