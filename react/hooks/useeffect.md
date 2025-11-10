---
aliases: ["useEffect Hook", "React useEffect"]
tags: 
  - react
  - hooks
  - side-effects
  - component-lifecycle
---

# React useEffect Hook: Полное Руководство

## Что такое useEffect

`useEffect` - это хук в React, который позволяет выполнять побочные эффекты в функциональных компонентах. Он объединяет в себе функциональность методов жизненного цикла классовых компонентов, таких как `componentDidMount`, `componentDidUpdate` и `componentWillUnmount`.

```jsx
import React, { useEffect, useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Вы нажали ${count} раз`;
  });

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
}
```

## Базовое использование

`useEffect` принимает функцию, которая содержит код побочного эффекта. По умолчанию эффект запускается после каждого рендера компонента.

```jsx
useEffect(() => {
  // Код эффекта
  console.log('Эффект запущен');
});
```

## Массив зависимостей

Массив зависимостей определяет, когда эффект должен перезапускаться. Если массив пуст, эффект запускается только один раз (при монтировании).

```jsx
// Запускается при каждом рендере
useEffect(() => {
  console.log('Каждый рендер');
});

// Запускается только при монтировании
useEffect(() => {
  console.log('Только при монтировании');
}, []);

// Запускается при изменении count
useEffect(() => {
  console.log('Count изменился:', count);
}, [count]);
```

## Функции очистки

Функция очистки возвращается из эффекта и вызывается перед тем, как эффект выполнится снова или компонент размонтируется.

```jsx
useEffect(() => {
  const subscription = subscribeToSomething();
  
  return () => {
    // Очистка при размонтировании или перед повторным запуском
    subscription.unsubscribe();
  };
}, []);
```

## Распространенные паттерны

### Подписка на события

```jsx
useEffect(() => {
  const handleResize = () => {
    console.log(window.innerWidth);
  };

  window.addEventListener('resize', handleResize);
  
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

### Загрузка данных

```jsx
useEffect(() => {
  const fetchData = async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      setData(data);
    } catch (error) {
      setError(error);
    }
  };

  fetchData();
}, []);
```

## Избегание бесконечных циклов

Используйте массив зависимостей для предотвращения бесконечных циклов:

```jsx
// ПЛОХО - бесконечный цикл
useEffect(() => {
  setCount(count + 1); // count изменяется, эффект перезапускается
});

// ХОРОШО - с зависимостями
useEffect(() => {
  if (count < 10) {
    setCount(count + 1);
  }
}, [count]);
```

## Условные эффекты

Вы можете использовать условные операторы внутри эффектов:

```jsx
useEffect(() => {
  if (shouldUpdate) {
    document.title = `Новое значение: ${value}`;
  }
}, [value, shouldUpdate]);
```

## Несколько эффектов

Разделяйте разные побочные эффекты на несколько `useEffect`:

```jsx
// Отдельные эффекты для разных задач
useEffect(() => {
  document.title = title;
}, [title]);

useEffect(() => {
  const subscription = subscribeToChat();
  return () => subscription.unsubscribe();
}, []);
```

## Сравнение с методами жизненного цикла

| Классовый компонент | Хук |
|---------------------|-----|
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [dep])` |
| `componentWillUnmount` | `return () => {}` в `useEffect` |

## Соображения производительности

- Избегайте создания функций в теле компонента при каждом рендере
- Используйте `useCallback` для функций, передаваемых в зависимости
- Оптимизируйте массив зависимостей

```jsx
const handleCallback = useCallback(() => {
  // функция, которая используется в useEffect
}, [dependency]);

useEffect(() => {
  someAPI.subscribe(handleCallback);
  return () => someAPI.unsubscribe(handleCallback);
}, [handleCallback]);
```

## Пропуск эффектов

Пустой массив зависимостей означает, что эффект запустится только при монтировании:

```jsx
useEffect(() => {
  // Запускается только один раз
  initializeSomething();
}, []); // Пустой массив - эффект не перезапускается
```

## Зависимости эффектов

Важно правильно указывать все зависимости:

```jsx
// ПЛОХО - отсутствует зависимость
useEffect(() => {
  document.title = user.name;
}, []); // user.name не в зависимостях

// ХОРОШО - все зависимости указаны
useEffect(() => {
  document.title = user.name;
}, [user.name]);
```

## Асинхронные операции в эффектах

Асинхронные функции не могут быть напрямую переданы в `useEffect`:

```jsx
// ПЛОХО
useEffect(async () => {
  const data = await fetch('/api/data');
  setData(data);
}, []);

// ХОРОШО
useEffect(() => {
  const fetchData = async () => {
    const data = await fetch('/api/data');
    setData(data);
  };
  
  fetchData();
}, []);
```

## Пользовательские эффекты на основе useEffect

Создание пользовательских хуков с использованием `useEffect`:

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    document.title = title;
  }, [title]);
}

// Использование
function MyComponent({ name }) {
  useDocumentTitle(`Привет, ${name}!`);
  return <div>Привет, {name}!</div>;
}
```

## Распространенные ловушки

1. **Забытые зависимости**: Приводит к устаревшим значениям в замыкании
2. **Неправильные зависимости**: Вызывает ненужные перезапуски
3. **Асинхронные эффекты**: Не используйте async напрямую
4. **Утечки памяти**: Не забывайте про функции очистки

## Тестирование useEffect

Для тестирования используйте React Testing Library:

```jsx
// В компоненте
useEffect(() => {
  if (count > 10) {
    setShowMessage(true);
  }
}, [count]);

// В тесте
await act(async () => {
  fireEvent.click(button);
});
expect(getByText(/сообщение/i)).toBeInTheDocument();
```

## Связь с другими файлами React

- [[react/hooks/state]] - для понимания состояния компонентов
- [[react/components]] - для контекста использования эффектов
- [[react/context]] - для передачи данных между компонентами
- [[react/performance]] - для оптимизации эффектов

## Заключение

`useEffect` - мощный инструмент для управления побочными эффектами в React. Правильное понимание массива зависимостей, функций очистки и производительности поможет избежать распространенных ошибок и создавать эффективные приложения.

> [!tip] 
> Всегда проверяйте массив зависимостей с помощью ESLint-правила `react-hooks/exhaustive-deps` для предотвращения багов.

> [!warning]
> Неправильное использование useEffect может привести к утечкам памяти и бесконечным циклам обновлений.
