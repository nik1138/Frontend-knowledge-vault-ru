# React useState Hook

## Введение

`useState` - это один из основных хуков в React, который позволяет добавлять состояние в функциональные компоненты. До появления хуков состояние было доступно только в классовых компонентах, что делало функциональные компоненты ограниченными по возможностям.

## Что такое useState

`useState` - это встроенный хук React, который позволяет функциональным компонентам хранить и управлять локальным состоянием. Он возвращает массив из двух элементов: текущего значения состояния и функции для его обновления.

```javascript
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}
```

## Объявление переменных состояния

При использовании `useState` мы деструктурируем возвращаемый массив, чтобы получить текущее значение состояния и функцию обновления:

```javascript
const [state, setState] = useState(initialValue);
```

Где:
- `state` - текущее значение состояния
- `setState` - функция для обновления состояния
- `initialValue` - начальное значение состояния

## Инициализация состояния

### Простые значения

```javascript
const [name, setName] = useState(''); // строка
const [age, setAge] = useState(0); // число
const [isVisible, setIsVisible] = useState(false); // булевое значение
```

### Ленивая инициализация

Для вычислительно затратных начальных значений:

```javascript
const [items, setItems] = useState(() => {
  // Вычисление начального значения происходит только один раз
  return expensiveCalculation();
});
```

## Обновление состояния

Функция обновления принимает новое значение состояния:

```javascript
// Прямое обновление
setCount(42);

// Функциональное обновление
setCount(prevCount => prevCount + 1);
```

## Пакетное обновление состояния

React объединяет несколько вызовов обновления состояния в один, чтобы оптимизировать рендеринг:

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const incrementTwice = () => {
    setCount(count + 1);
    setCount(count + 1); // Это не приведет к увеличению на 2
  };

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={incrementTwice}>Увеличить дважды</button>
    </div>
  );
}
```

## Функциональные обновления

Для обновления состояния на основе предыдущего значения используйте функциональное обновление:

```javascript
const [count, setCount] = useState(0);

// Правильно
setCount(prevCount => prevCount + 1);

// Неправильно при асинхронных вызовах
setCount(count + 1);
```

## useState с объектами и массивами

### Работа с объектами

```javascript
const [user, setUser] = useState({ name: '', email: '' });

// Правильное обновление объекта
setUser(prevUser => ({
  ...prevUser,
  name: 'Новое имя'
}));
```

### Работа с массивами

```javascript
const [items, setItems] = useState([]);

// Добавление элемента
setItems(prevItems => [...prevItems, newItem]);

// Удаление элемента
setItems(prevItems => prevItems.filter(item => item.id !== id));

// Обновление элемента
setItems(prevItems => 
  prevItems.map(item => 
    item.id === id ? { ...item, ...updates } : item
  )
);
```

## Распространенные ловушки

### 1. Мутация объектов и массивов

```javascript
// Неправильно - мутация
setUser({ ...user, name: 'новое имя' });

// Правильно - создание нового объекта
setUser(prevUser => ({ ...prevUser, name: 'новое имя' }));
```

### 2. Использование устаревших значений

```javascript
// Неправильно
function handleIncrement() {
  setTimeout(() => {
    setCount(count + 1); // Использует устаревшее значение
  }, 1000);
}

// Правильно
function handleIncrement() {
  setTimeout(() => {
    setCount(prevCount => prevCount + 1); // Использует актуальное значение
  }, 1000);
}
```

## Различия между useState и setState

| useState | setState |
|----------|----------|
| Используется в функциональных компонентах | Используется в классовых компонентах |
| Возвращает массив [state, setState] | Обновляет this.state |
| Пакетное обновление по умолчанию | Пакетное обновление в обработчиках событий |
| Требует функциональные обновления для асинхронности | Автоматически использует предыдущее состояние |

```javascript
// useState
const [count, setCount] = useState(0);
setCount(prevCount => prevCount + 1);

// setState
this.setState(prevState => ({
  count: prevState.count + 1
}));
```

## Лучшие практики с useState

### 1. Разделение состояния на независимые переменные

```javascript
// Вместо одного объекта
const [state, setState] = useState({ name: '', age: 0, email: '' });

// Лучше использовать отдельные состояния
const [name, setName] = useState('');
const [age, setAge] = useState(0);
const [email, setEmail] = useState('');
```

### 2. Использование функциональных обновлений при зависимости от предыдущего состояния

```javascript
// Правильно
setCount(prevCount => prevCount + incrementValue);
```

### 3. Ленивая инициализация для вычислительно затратных значений

```javascript
const [items, setItems] = useState(() => computeExpensiveValue());
```

## Соображения производительности

- `useState` не вызывает повторный рендер, если новое значение равно предыдущему (сравнение с помощью `Object.is`)
- Для сложных объектов может потребоваться оптимизация с помощью `useMemo` или `useCallback`
- Избегайте ненужных перерисовок при обновлении состояния

## Тестирование useState хуков

Для тестирования хуков используйте `@testing-library/react-hooks`:

```javascript
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

test('должен увеличивать счетчик', () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

## Расширенные паттерны useState

### 1. Состояние с логикой сброса

```javascript
function useResetState(initialValue) {
  const [state, setState] = useState(initialValue);
  
  const reset = useCallback(() => setState(initialValue), [initialValue]);
  
  return [state, setState, reset];
}
```

### 2. Множественные состояния с одним обновлением

```javascript
const [form, setForm] = useState({ name: '', email: '' });

const updateField = (field, value) => {
  setForm(prevForm => ({
    ...prevForm,
    [field]: value
  }));
};
```

## Связи с другими файлами React

- [[react/hooks/useeffect]] - для выполнения побочных эффектов при изменении состояния
- [[react/hooks/usecallback]] - для оптимизации функций обновления
- [[react/hooks/usememo]] - для оптимизации вычислений на основе состояния
- [[react/components]] - для понимания контекста использования хуков
- [[react/state-management]] - для сравнения с другими подходами управления состоянием

## Заключение

`useState` - мощный инструмент для управления локальным состоянием в функциональных компонентах React. Понимание его особенностей и правильное использование позволяет создавать эффективные и предсказуемые компоненты. Главное - помнить о неизменяемости данных и правильном обновлении состояния через функциональные обновления при необходимости.

## Теги

#react #hooks #useState #javascript #frontend #state-management #best-practices