---
aliases: ["Пользовательские хуки", "Создание хуков", "Разработка хуков"]
tags: ["react", "hooks", "custom-hooks", "javascript", "frontend"]
---

# Пользовательские React хуки (Custom Hooks)

Пользовательские React хуки - это функции, которые используют встроенные React хуки для извлечения и повторного использования логики состояния между компонентами. Они позволяют абстрагировать сложную логику в переиспользуемые функции.

## Что такое пользовательские хуки

Пользовательский хук - это JavaScript-функция, название которой начинается с `use`, и которая может вызывать другие хуки. Это способ извлечения логики состояния из компонента в функцию, которую можно повторно использовать.

Пользовательские хуки позволяют:
- Разделять логику состояния между компонентами
- Упрощать тестирование логики
- Избегать дублирования кода
- Создавать более чистую архитектуру компонентов

## Создание пользовательских хуков

Для создания пользовательского хука:
1. Назовите функцию с префиксом `use`
2. Используйте встроенные хуки внутри функции
3. Возвращайте значения, которые будут использоваться в компоненте

```javascript
function useCounter(initialValue = 0) {
  const [count, setCount] = React.useState(initialValue);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}
```

## Правила пользовательских хуков

Пользовательские хуки должны соблюдать те же правила, что и встроенные хуки:

1. **Вызов только на верхнем уровне**: Не вызывайте хуки внутри циклов, условий или вложенных функций
2. **Вызов только из React-компонентов**: Хуки можно вызывать только из React-функциональных компонентов или из других пользовательских хуков

## Реальные примеры

### Хук для управления формой

```javascript
function useForm(initialValues) {
  const [values, setValues] = React.useState(initialValues);
  
  const handleChange = (name, value) => {
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  const reset = () => setValues(initialValues);
  
  return { values, handleChange, reset };
}
```

### Хук для работы с localStorage

```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = React.useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}
```

## Извлечение логики состояния

Пользовательские хуки позволяют извлекать логику состояния из компонентов. Это позволяет тестировать логику отдельно от компонентов и повторно использовать её в разных частях приложения.

```javascript
// Хук для отслеживания размера окна
function useWindowSize() {
  const [windowSize, setWindowSize] = React.useState({
    width: undefined,
    height: undefined,
  });
  
  React.useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    
    window.addEventListener("resize", handleResize);
    handleResize();
    
    return () => window.removeEventListener("resize", handleResize);
  }, []);
  
  return windowSize;
}
```

## Повторно используемые паттерны логики

### Паттерн "Состояние + Эффект"

```javascript
function useApi(url) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState(null);
  
  React.useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}
```

### Паттерн "Контроллер состояния"

```javascript
function useToggle(initialValue = false) {
  const [value, setValue] = React.useState(initialValue);
  
  const toggle = React.useCallback(() => setValue(v => !v), []);
  const setTrue = React.useCallback(() => setValue(true), []);
  const setFalse = React.useCallback(() => setValue(false), []);
  
  return { value, toggle, setTrue, setFalse };
}
```

## Распространенные пользовательские хуки

- [[react/hooks/useForm]] - для управления формами
- [[react/hooks/useDebounce]] - для дебаунса
- [[react/hooks/useThrottle]] - для троттлинга
- [[react/hooks/useClickOutside]] - для отслеживания кликов вне компонента
- [[react/hooks/useKeyPress]] - для отслеживания нажатий клавиш

## Тестирование пользовательских хуков

Для тестирования пользовательских хуков рекомендуется использовать React Hooks Testing Library:

```javascript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter(0));
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Совместное использование логики состояния между компонентами

Пользовательские хуки позволяют легко делиться логикой между компонентами:

```javascript
// Компонент 1
function ComponentA() {
  const { count, increment } = useCounter(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
    </div>
  );
}

// Компонент 2
function ComponentB() {
  const { count, increment } = useCounter(10);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

## Рассмотрение производительности

- Используйте `React.useCallback` и `React.useMemo` для оптимизации производительности
- Избегайте ненужных повторных вычислений в хуках
- Используйте мемоизацию для сложных вычислений

## Отладка пользовательских хуков

- Используйте React DevTools для отладки хуков
- Добавляйте логирование внутри хуков для отладки
- Разделяйте сложные хуки на более простые для лучшей отладки

## Продвинутые паттерны пользовательских хуков

### Комбинирование нескольких хуков

```javascript
function useFormData(initialValues) {
  const [values, setValues] = useLocalStorage('form-data', initialValues);
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  
  // Комбинируем логику из нескольких хуков
  return {
    values,
    errors,
    touched,
    handleChange: (name, value) => {
      setValues(prev => ({ ...prev, [name]: value }));
    },
    handleBlur: (name) => {
      setTouched(prev => ({ ...prev, [name]: true }));
    },
    validate: () => {
      // Логика валидации
    }
  };
}
```

## Передача параметров в хуки

Хуки могут принимать параметры, как и обычные функции:

```javascript
function useTimeout(callback, delay) {
  const savedCallback = React.useRef();
  
  React.useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  React.useEffect(() => {
    function tick() {
      savedCallback.current();
    }
    
    if (delay !== null) {
      const id = setTimeout(tick, delay);
      return () => clearTimeout(id);
    }
  }, [delay]);
  
  return [delay, (newDelay) => setDelay(newDelay)];
}
```

## Возврат значений из хуков

Хуки могут возвращать любые значения - примитивы, объекты, функции или массивы:

```javascript
function useToggle(initialValue = false) {
  const [value, setValue] = React.useState(initialValue);
  
  return [
    value,
    () => setValue(v => !v)
  ];
}

// Использование
const [isOn, toggle] = useToggle(true);
```

## Связь с другими файлами React

- [[react/hooks/basics]] - основы React хуков
- [[react/components/patterns]] - паттерны компонентов
- [[react/state-management]] - управление состоянием
- [[react/performance]] - оптимизация производительности
- [[react/testing]] - тестирование React приложений

> [!tip] Совет
> Пользовательские хуки особенно полезны при создании библиотек компонентов или при необходимости повторного использования сложной логики состояния в разных частях приложения.

> [!warning] Важно
> Пользовательские хуки должны следовать тем же правилам, что и встроенные хуки. Нарушение этих правил может привести к непредсказуемому поведению компонентов.

## Заключение

Пользовательские хуки - мощный инструмент для извлечения и повторного использования логики состояния в React-приложениях. Они позволяют создавать более чистый, тестируемый и поддерживаемый код, разделяя логику от представления.