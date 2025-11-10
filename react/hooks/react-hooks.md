---
tags: [react, hooks, javascript, frontend]
aliases: [React Hooks, React.useEffect, React.useState]
---

# React Hooks: Полное руководство

React Hooks — это функции, которые позволяют использовать состояние и другие возможности React без написания классов. Появились в React 16.8 и кардинально изменили подход к написанию компонентов.

## Эволюция хуков

До появления хуков React-компоненты разделялись на:
- **Stateless functional components** — простые функции без состояния
- **Class components** — компоненты с состоянием и методами жизненного цикла

Проблемы классов:
- Сложность повторного использования логики между компонентами
- Запутанные методы жизненного цикла
- Необходимость понимания `this` в JavaScript

React Hooks решили эти проблемы, позволив использовать:
- Состояние в функциональных компонентах
- Повторное использование логики через кастомные хуки
- Более чистую архитектуру

## Основные хуки

### useState

Позволяет добавить состояние в функциональный компонент:

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счет: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}
```

### useEffect

Позволяет выполнять побочные эффекты в функциональных компонентах:

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Выполняется при монтировании и при изменении userId
    fetchUser(userId).then(setUser);
    
    // Функция очистки (аналог componentWillUnmount)
    return () => {
      console.log('Очистка эффекта');
    };
  }, [userId]); // Зависимости

  return <div>{user ? user.name : 'Загрузка...'}</div>;
}
```

### useContext

Позволяет использовать контекст без оборачивания в `<Context.Consumer>`:

```jsx
import React, { useContext } from 'react';

const ThemeContext = React.createContext();

function ThemedButton() {
  const theme = useContext(ThemeContext);
  
  return (
    <button style={{ background: theme.background, color: theme.color }}>
      Кнопка с темой
    </button>
  );
}
```

### useReducer

Альтернатива useState для сложного состояния:

```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      Счет: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Сброс</button>
    </div>
  );
}
```

## Продвинутые хуки

### useCallback

Возвращает мемоизированную версию функции:

```jsx
import React, { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Зависимости

  return <ChildComponent onClick={handleClick} />;
}

const ChildComponent = React.memo(({ onClick }) => {
  console.log('Child рендерится');
  return <button onClick={onClick}>Нажми меня</button>;
});
```

### useMemo

Мемоизирует вычисленное значение:

```jsx
import React, { useState, useMemo } from 'react';

function ExpensiveComponent({ items, filter }) {
  const [count, setCount] = useState(0);

  // Мемоизированный расчет
  const filteredItems = useMemo(() => {
    return items.filter(item => item.includes(filter));
  }, [items, filter]);

  return (
    <div>
      <p>Фильтрованных элементов: {filteredItems.length}</p>
      <button onClick={() => setCount(count + 1)}>Счет: {count}</button>
    </div>
  );
}
```

### useRef

Позволяет получить прямой доступ к DOM-элементу или хранить изменяемое значение:

```jsx
import React, { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
}
```

### useImperativeHandle

Позволяет настраивать экземпляр, передаваемый через ref:

```jsx
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    getValue: () => inputRef.current.value,
    setValue: (val) => inputRef.current.value = val
  }));

  return <input ref={inputRef} type="text" />;
});

// Использование
function Parent() {
  const fancyInputRef = useRef();

  const handleFocus = () => {
    fancyInputRef.current.focus();
  };

  return (
    <div>
      <FancyInput ref={fancyInputRef} />
      <button onClick={handleFocus}>Фокус</button>
    </div>
  );
}
```

### useLayoutEffect

Аналог useEffect, но синхронно после всех изменений DOM:

```jsx
import React, { useLayoutEffect, useRef } from 'react';

function MeasureExample() {
  const divRef = useRef();

  useLayoutEffect(() => {
    // Выполняется синхронно после обновления DOM
    const { offsetHeight } = divRef.current;
    console.log('Высота:', offsetHeight);
  });

  return <div ref={divRef}>Пример измерения</div>;
}
```

## Кастомные хуки

Кастомные хуки позволяют изолировать логику состояния в функции:

```jsx
import { useState, useEffect } from 'react';

// Кастомный хук для отслеживания размера окна
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', handleResize);
    handleResize(); // Установка начального размера

    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

// Использование кастомного хука
function WindowSizeDisplay() {
  const { width, height } = useWindowSize();
  
  return (
    <div>
      Ширина: {width}px, Высота: {height}px
    </div>
  );
}
```

## Правила хуков

1. **Вызывайте хуки только на верхнем уровне** — не внутри циклов, условий или вложенных функций
2. **Вызывайте хуки только из React-компонентов** — не из обычных функций

> [!tip] Совет
> Используйте ESLint-плагин `eslint-plugin-react-hooks` для автоматической проверки соблюдения правил.

## Оптимизация производительности

- Используйте `useMemo` для мемоизации вычислений
- Используйте `useCallback` для мемоизации функций
- Применяйте `React.memo` для предотвращения ненужных перерендеров дочерних компонентов
- Избегайте создания новых объектов/массивов в каждом рендере

## Распространенные паттерны

- **useToggle** — переключатель состояния
- **usePrevious** — получение предыдущего значения
- **useAsync** — управление асинхронными операциями
- **useForm** — управление формами

## Хуки vs Классы

| Хуки | Классы |
|------|--------|
| Более компактный код | Более многословный |
| Легче повторное использование логики | Сложнее повторное использование |
| Нет `this` | Требуется понимание `this` |
| Композиция вместо наследования | Возможность наследования |

## Ограничения хуков

- Не работают в классовых компонентах
- Требуют строгого соблюдения правил
- Могут усложнить отладку сложных эффектов
- Ограниченная поддержка в старых версиях React

## Тестирование хуков

Для тестирования хуков используйте `@testing-library/react-hooks`:

```jsx
import { renderHook, act } from '@testing-library/react-hooks';
import useCounter from './useCounter';

test('должен увеличивать значение счетчика', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Безопасность при использовании хуков

- Всегда очищайте подписки и таймеры в эффектах
- Не доверяйте пользовательским данным без валидации
- Избегайте утечек памяти при работе с асинхронными операциями
- Проверяйте зависимости в useEffect на предмет устаревших значений

## Связи с другими файлами

- [[react-components]] — компоненты React
- [[react-state-management]] — управление состоянием
- [[react-performance]] — производительность React
- [[javascript-closures]] — замыкания в JavaScript (важны для понимания работы хуков)
- [[react-context]] — контекст в React

## Заключение

React Hooks значительно упростили написание компонентов и повторное использование логики. Понимание их работы и правильное применение позволяет создавать более чистый и поддерживаемый код.