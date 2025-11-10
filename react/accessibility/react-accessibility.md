---
aliases: ["React и доступность", "Доступность в React", "a11y в React"]
tags: 
  - #react
  - #accessibility
  - #a11y
  - #frontend
  - #best-practices
---

# Доступность в React (React Accessibility)

## Что такое доступность в React

Доступность в React (React Accessibility) - это практика создания приложений, которые могут быть использованы людьми с различными ограничениями, включая нарушения зрения, слуха, моторики или когнитивные особенности. React предоставляет мощные инструменты для создания доступных интерфейсов, но требует осознанного подхода при разработке.

## Семантический HTML в React

React позволяет использовать стандартные HTML-элементы с их семантическим значением:

```jsx
// Хорошо: использование семантических элементов
function Header() {
  return (
    <header>
      <h1>Заголовок сайта</h1>
      <nav aria-label="Основная навигация">
        <ul>
          <li><a href="/">Главная</a></li>
          <li><a href="/about">О нас</a></li>
        </ul>
      </nav>
    </header>
  );
}
```

## ARIA атрибуты и роли

React полностью поддерживает ARIA-атрибуты, которые помогают улучшить доступность:

```jsx
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      aria-describedby="modal-description"
    >
      <div id="modal-title" className="sr-only">Заголовок модального окна</div>
      <div id="modal-description" className="sr-only">Описание модального окна</div>
      {children}
      <button onClick={onClose} aria-label="Закрыть модальное окно">X</button>
    </div>
  );
}
```

## Клавиатурная навигация

Важно обеспечить возможность использования приложения исключительно с клавиатуры:

```jsx
function FocusableButton({ onClick, children }) {
  const handleKeyDown = (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick();
    }
  };

  return (
    <button 
      onClick={onClick} 
      onKeyDown={handleKeyDown}
      tabIndex="0"
    >
      {children}
    </button>
  );
}
```

## Управление фокусом в React

React позволяет управлять фокусом с помощью ref и useEffect:

```jsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder="Это поле получит фокус при загрузке" />;
}
```

## Инструменты для тестирования доступности

- axe-core - библиотека для автоматического тестирования
- React Testing Library с поддержкой a11y
- WAVE - браузерное расширение
- Lighthouse - встроенная проверка доступности

## Распространенные паттерны доступности в React

### Кнопка-переключатель (Toggle Button)

```jsx
function ToggleButton({ isToggled, onToggle, label }) {
  return (
    <button
      role="switch"
      aria-checked={isToggled}
      onClick={onToggle}
    >
      {label}: {isToggled ? 'Вкл' : 'Выкл'}
    </button>
  );
}
```

### Контролируемый список с клавиатурной навигацией

```jsx
function AccessibleList({ items }) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  
  const handleKeyDown = (e, index) => {
    switch(e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex(prev => Math.min(prev + 1, items.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex(prev => Math.max(prev - 1, 0));
        break;
    }
  };

  return (
    <ul role="listbox">
      {items.map((item, index) => (
        <li 
          key={item.id} 
          role="option"
          aria-selected={focusedIndex === index}
          onKeyDown={(e) => handleKeyDown(e, index)}
          tabIndex={focusedIndex === index ? 0 : -1}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

## Паттерны, которых следует избегать

- Использование div вместо семантических элементов
- Пропуск заголовков (например, h1 -> h3 без h2)
- Неправильное использование role
- Отсутствие альтернативного текста для изображений

## Совместимость с программами чтения с экрана

Программы чтения с экрана, такие как NVDA, JAWS и VoiceOver, полагаются на:
- Правильную семантику HTML
- ARIA-атрибуты для динамического контента
- Четкие имена для интерактивных элементов

## Практики использования ARIA в React

- Используйте `aria-label` для элементов без видимого текста
- Применяйте `aria-describedby` для дополнительного описания
- Используйте `aria-live` для динамического контента

## Обработка доступности динамического контента

```jsx
function Notification({ message, isVisible }) {
  return (
    <div
      aria-live="polite"
      aria-atomic="true"
      hidden={!isVisible}
    >
      {message}
    </div>
  );
}
```

## Доступность форм

```jsx
function AccessibleForm() {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');

  const validateEmail = () => {
    if (!email.includes('@')) {
      setEmailError('Пожалуйста, введите действительный адрес электронной почты');
    } else {
      setEmailError('');
    }
  };

  return (
    <form>
      <label htmlFor="email-input">Email:</label>
      <input
        id="email-input"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onBlur={validateEmail}
        aria-describedby={emailError ? "email-error" : undefined}
        aria-invalid={!!emailError}
      />
      {emailError && (
        <div id="email-error" role="alert" className="error">
          {emailError}
        </div>
      )}
    </form>
  );
}
```

## Доступность библиотек компонентов

При создании библиотек компонентов:
- Обеспечьте гибкость ARIA-атрибутов
- Поддерживайте клавиатурную навигацию
- Документируйте доступность каждого компонента

## Создание доступных UI-компонентов

Каждый компонент должен:
- Иметь подходящую семантику
- Поддерживать клавиатурную навигацию
- Работать с программами чтения с экрана
- Иметь контрастные цвета

## Линтинг доступности

Используйте eslint-plugin-jsx-a11y для автоматической проверки:

```json
{
  "extends": ["react-app", "react-app/jest"],
  "rules": {
    "jsx-a11y/alt-text": "error",
    "jsx-a11y/anchor-has-content": "error",
    "jsx-a11y/anchor-is-valid": "error"
  }
}
```

## Лучшие практики доступности в React

1. Используйте семантические HTML-элементы
2. Обеспечьте надлежащее управление фокусом
3. Проверяйте контраст цветов
4. Обеспечьте клавиатурную навигацию
5. Тестируйте с программами чтения с экрана
6. Используйте линтеры доступности

## Международальные соображения

- Используйте атрибут `lang` для указания языка
- Обеспечьте поддержку направления текста (ltr/rtl)
- Предоставляйте переводы ARIA-этикеток

## Связи с другими файлами React

Для более глубокого понимания доступности в React рекомендуется ознакомиться с:
- [[react/react-components]] - для понимания жизненного цикла компонентов
- [[react/react-hooks]] - для управления состоянием доступности
- [[react/react-props]] - для передачи ARIA-атрибутов
- [[html/semantic-html]] - для понимания семантической разметки
- [[css/accessibility-css]] - для стилизации с учетом доступности

## Заключение

Доступность - не опциональная функция, а основной аспект разработки интерфейсов. React предоставляет мощные инструменты для создания доступных приложений, но требует осознанного подхода при реализации. Правильная реализация доступности делает приложение доступным для всех пользователей, независимо от их ограничений.
