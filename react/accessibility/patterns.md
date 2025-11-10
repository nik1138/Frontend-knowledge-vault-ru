---
aliases: ["Реакт доступность", "Паттерны доступности"]
tags: 
  - #react
  - #accessibility
  - #web-development
  - #a11y
  - #patterns
---

# Паттерны доступности в React

## Введение

Доступность (a11y) в React приложениях критически важна для обеспечения равного доступа к информации и функциональности для пользователей с ограниченными возможностями. Эта статья охватывает ключевые паттерны доступности, специфичные для React разработки.

## Семантическая разметка HTML в React

Семантическая разметка обеспечивает правильную структуру контента, что помогает вспомогательным технологиям понимать и представлять информацию пользователям.

```jsx
// Плохо
<div onClick={handleClick}>Кнопка</div>

// Хорошо
<button onClick={handleClick}>Кнопка</button>

// Использование заголовков в правильной иерархии
function ContentStructure() {
  return (
    <article>
      <header>
        <h1>Заголовок статьи</h1>
      </header>
      <main>
        <section>
          <h2>Подраздел</h2>
          <p>Содержимое...</p>
        </section>
      </main>
    </article>
  );
}
```

## Реализация ARIA атрибутов

ARIA (Accessible Rich Internet Applications) предоставляет возможности для улучшения доступности сложных компонентов интерфейса.

```jsx
// Пример использования ARIA для вспомогательных технологий
function AccessibleForm() {
  const [value, setValue] = useState('');
  
  return (
    <div>
      <label htmlFor="search-input">Поиск</label>
      <input
        id="search-input"
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        aria-describedby="search-help"
        aria-autocomplete="list"
        aria-controls="search-results"
      />
      <div id="search-help" className="sr-only">
        Введите поисковый запрос
      </div>
      <ul id="search-results" role="listbox">
        {/* Результаты поиска */}
      </ul>
    </div>
  );
}
```

## Паттерны навигации с клавиатуры

Обеспечение навигации только с клавиатуры критично для пользователей, не использующих мышь.

```jsx
function KeyboardAccessibleMenu({ items, isOpen, onClose }) {
  const [activeIndex, setActiveIndex] = useState(0);
  
  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex(prev => (prev + 1) % items.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(prev => (prev - 1 + items.length) % items.length);
        break;
      case 'Escape':
        onClose();
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        items[activeIndex].onClick();
        break;
    }
  };
  
  return (
    <ul 
      role="menu" 
      onKeyDown={handleKeyDown}
      tabIndex={0}
    >
      {items.map((item, index) => (
        <li 
          key={item.id}
          role="menuitem"
          className={index === activeIndex ? 'active' : ''}
          onClick={item.onClick}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

## Стратегии управления фокусом

Управление фокусом особенно важно при динамическом изменении содержимого.

```jsx
// Хук для управления фокусом
function useFocusReturn(triggerElement) {
  useEffect(() => {
    const previouslyFocusedElement = document.activeElement;
    
    return () => {
      if (previouslyFocusedElement && previouslyFocusedElement.focus) {
        previouslyFocusedElement.focus();
      }
    };
  }, [triggerElement]);
}

// Пример модального окна с управлением фокусом
function Modal({ isOpen, onClose, children }) {
  useFocusReturn(isOpen);
  
  useEffect(() => {
    if (isOpen) {
      // Установка фокуса на модальное окно
      const modalElement = document.getElementById('modal');
      if (modalElement) {
        modalElement.focus();
      }
    }
  }, [isOpen]);
  
  // Ограничение фокуса внутри модального окна
  useEffect(() => {
    if (isOpen) {
      const focusableElements = document.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      const handleTab = (e) => {
        if (e.key === 'Tab') {
          if (e.shiftKey) {
            if (document.activeElement === firstElement) {
              e.preventDefault();
              lastElement.focus();
            }
          } else {
            if (document.activeElement === lastElement) {
              e.preventDefault();
              firstElement.focus();
            }
          }
        }
      };
      
      document.addEventListener('keydown', handleTab);
      return () => document.removeEventListener('keydown', handleTab);
    }
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div 
        id="modal" 
        className="modal-content" 
        role="dialog" 
        aria-modal="true"
        tabIndex={-1}
      >
        {children}
      </div>
    </div>
  );
}
```

## Совместимость со скринридерами

Обеспечение корректной работы с программами чтения с экрана требует правильного использования семантических элементов и ARIA атрибутов.

```jsx
// Пример компонента с описанием для скринридеров
function VisuallyHidden({ children, ...props }) {
  return (
    <span
      style={{
        position: 'absolute',
        width: '1px',
        height: '1px',
        padding: 0,
        margin: '-1px',
        overflow: 'hidden',
        clip: 'rect(0, 0, 0, 0)',
        whiteSpace: 'nowrap',
        borderWidth: 0,
      }}
      {...props}
    >
      {children}
    </span>
  );
}

function ProgressBar({ value, max = 100 }) {
  return (
    <div>
      <VisuallyHidden id="progress-label">Прогресс выполнения</VisuallyHidden>
      <progress
        value={value}
        max={max}
        aria-labelledby="progress-label"
        aria-valuenow={value}
        aria-valuemin={0}
        aria-valuemax={max}
      />
      <span>{value}%</span>
    </div>
  );
}
```

## Паттерны доступных форм

Формы должны быть доступны для всех пользователей, включая тех, кто использует вспомогательные технологии.

```jsx
function AccessibleForm() {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');
  
  const validateEmail = (email) => {
    // Валидация email
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    return isValid ? '' : 'Пожалуйста, введите действительный email адрес';
  };
  
  const handleEmailChange = (e) => {
    const newEmail = e.target.value;
    setEmail(newEmail);
    
    // Временная задержка для обновления ошибки
    setTimeout(() => {
      const error = validateEmail(newEmail);
      setEmailError(error);
    }, 0);
  };
  
  return (
    <form>
      <div>
        <label htmlFor="email-input">Email</label>
        <input
          id="email-input"
          type="email"
          value={email}
          onChange={handleEmailChange}
          aria-invalid={!!emailError}
          aria-describedby={emailError ? "email-error" : undefined}
        />
        {emailError && (
          <div id="email-error" role="alert">
            {emailError}
          </div>
        )}
      </div>
    </form>
  );
}
```

## Доступные модальные окна

Модальные окна требуют особого внимания к доступности.

```jsx
// Пример доступного модального окна
function AccessibleModal({ isOpen, onClose, title, children }) {
  if (!isOpen) return null;
  
  return (
    <div 
      className="modal-backdrop"
      onClick={onClose}
      aria-hidden={!isOpen}
    >
      <div 
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onClick={(e) => e.stopPropagation()}
      >
        <header>
          <h2 id="modal-title">{title}</h2>
          <button 
            onClick={onClose}
            aria-label="Закрыть модальное окно"
          >
            &times;
          </button>
        </header>
        <main>{children}</main>
      </div>
    </div>
  );
}
```

## Доступные выпадающие списки и меню

Выпадающие компоненты требуют сложной логики для обеспечения доступности.

```jsx
function AccessibleDropdown({ options, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [highlightedIndex, setHighlightedIndex] = useState(0);
  const dropdownRef = useRef(null);
  
  const toggleDropdown = () => setIsOpen(!isOpen);
  
  const selectOption = (option) => {
    onSelect(option);
    setIsOpen(false);
  };
  
  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'Enter':
      case ' ':
        e.preventDefault();
        if (isOpen) {
          selectOption(options[highlightedIndex]);
        } else {
          setIsOpen(true);
        }
        break;
      case 'ArrowDown':
        e.preventDefault();
        setHighlightedIndex(prev => 
          prev < options.length - 1 ? prev + 1 : 0
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setHighlightedIndex(prev => 
          prev > 0 ? prev - 1 : options.length - 1
        );
        break;
      case 'Escape':
        setIsOpen(false);
        break;
    }
  };
  
  return (
    <div className="dropdown" ref={dropdownRef}>
      <button
        onClick={toggleDropdown}
        onKeyDown={handleKeyDown}
        aria-haspopup="listbox"
        aria-expanded={isOpen}
      >
        Выберите опцию
      </button>
      {isOpen && (
        <ul role="listbox">
          {options.map((option, index) => (
            <li
              key={option.id}
              role="option"
              aria-selected={index === highlightedIndex}
              className={index === highlightedIndex ? 'highlighted' : ''}
              onClick={() => selectOption(option)}
              onMouseEnter={() => setHighlightedIndex(index)}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Доступные таблицы

Таблицы должны быть структурированы так, чтобы вспомогательные технологии могли правильно интерпретировать данные.

```jsx
function AccessibleTable({ data, columns }) {
  return (
    <table>
      <caption>Описание таблицы данных</caption>
      <thead>
        <tr>
          {columns.map(column => (
            <th key={column.key} scope="col">
              {column.header}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row, rowIndex) => (
          <tr key={row.id}>
            {columns.map(column => (
              <td key={`${row.id}-${column.key}`}>
                {row[column.key]}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Доступные панели вкладок

Панели вкладок требуют сложной логики для обеспечения доступности.

```jsx
function AccessibleTabs({ tabs }) {
  const [activeTab, setActiveTab] = useState(0);
  
  const handleKeyDown = (index, e) => {
    switch (e.key) {
      case 'ArrowRight':
        e.preventDefault();
        setActiveTab(prev => (prev + 1) % tabs.length);
        break;
      case 'ArrowLeft':
        e.preventDefault();
        setActiveTab(prev => (prev - 1 + tabs.length) % tabs.length);
        break;
      case 'Home':
        e.preventDefault();
        setActiveTab(0);
        break;
      case 'End':
        e.preventDefault();
        setActiveTab(tabs.length - 1);
        break;
    }
  };
  
  return (
    <div>
      <div role="tablist">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            role="tab"
            aria-selected={index === activeTab}
            aria-controls={`panel-${tab.id}`}
            id={`tab-${tab.id}`}
            onKeyDown={(e) => handleKeyDown(index, e)}
            onClick={() => setActiveTab(index)}
          >
            {tab.title}
          </button>
        ))}
      </div>
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={index !== activeTab}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

## Паттерны соответствия WCAG

Соответствие стандартам WCAG (Web Content Accessibility Guidelines) обеспечивает универсальный доступ к веб-контенту.

```jsx
// Пример компонента, соответствующего WCAG
function HighContrastToggle() {
  const [highContrast, setHighContrast] = useState(false);
  
  useEffect(() => {
    if (highContrast) {
      document.body.classList.add('high-contrast');
    } else {
      document.body.classList.remove('high-contrast');
    }
  }, [highContrast]);
  
  return (
    <button
      onClick={() => setHighContrast(!highContrast)}
      aria-pressed={highContrast}
    >
      {highContrast ? 'Отключить' : 'Включить'} режим высокой контрастности
    </button>
  );
}
```

## Тестирование доступности в React

Для тестирования доступности в React приложениях можно использовать различные инструменты.

```jsx
// Пример использования react-axe для тестирования доступности
import { useEffect } from 'react';
import axe from 'react-axe';

if (process.env.NODE_ENV !== 'production') {
  axe(React);
}

// Компонент для тестирования
function TestComponent() {
  useEffect(() => {
    // Автоматическое тестирование доступности
  }, []);
  
  return (
    <div>
      <h1>Заголовок</h1>
      <button>Кнопка</button>
    </div>
  );
}
```

## Распространенные ловушки доступности

- Использование div вместо семантических элементов
- Отсутствие альтернативного текста для изображений
- Недостаточная контрастность цветов
- Неправильное использование ARIA атрибутов
- Отсутствие управления фокусом

## Библиотеки доступности

- [[react-axe]] - инструмент для тестирования доступности
- react-aria - библиотека от Adobe для создания доступных компонентов
- react-stately - библиотека для управления состоянием компонентов

## Автоматизированное тестирование доступности

```jsx
// Пример автоматизированного тестирования
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

it('должен быть доступным', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Заключение

Реализация паттернов доступности в React приложениях требует понимания как основных принципов доступности, так и особенностей работы с React. Следование этим паттернам помогает создавать более инклюзивные интерфейсы.

## См. также

- [[react-axe]] - библиотека для тестирования доступности
- [[React State Management]] - управление состоянием в React
- [[Component Architecture]] - архитектура компонентов
- [[WCAG Guidelines]] - рекомендации по доступности веб-контента