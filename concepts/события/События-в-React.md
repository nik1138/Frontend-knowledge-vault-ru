---
aliases: [События в React, React Events, Обработка событий в React]
tags: [react, javascript, events, frontend, programming]
---

# События-в-React

## Введение

Обработка событий в React имеет свои особенности, отличающиеся от традиционного подхода к обработке событий в DOM. React предоставляет унифицированный интерфейс для работы с событиями, который работает одинаково во всех браузерах и использует синтетические события.

## Основные отличия от DOM-событий

### 1. Именование обработчиков

В React имена событий используют стиль camelCase вместо lowercase:

```jsx
// В HTML
<button onclick="handleClick()">Кликни меня</button>

// В React
<button onClick={handleClick}>Кликни меня</button>
```

### 2. Передача функции, а не строки

В React мы передаем функцию в качестве обработчика, а не строку с кодом:

```jsx
// В HTML
<button onclick="alert('Привет')">Кликни меня</button>

// В React
<button onClick={() => alert('Привет')}>Кликни меня</button>
```

### 3. Синтетические события

React оборачивает нативные события в синтетические события (SyntheticEvent), которые обеспечивают кросс-браузерную совместимость:

```jsx
function handleClick(event) {
    // event - это синтетическое событие React
    console.log('Тип события:', event.type);
    console.log('Целевой элемент:', event.target);
    console.log('Координаты:', event.clientX, event.clientY);
}
```

## Обработка различных типов событий

### Mouse Events

```jsx
function MouseEventsExample() {
    const handleClick = (event) => {
        console.log('Клик!', event.clientX, event.clientY);
    };

    const handleDoubleClick = () => {
        console.log('Двойной клик!');
    };

    const handleMouseEnter = () => {
        console.log('Курсор вошел на элемент');
    };

    const handleMouseMove = (event) => {
        console.log('Движение мыши:', event.clientX, event.clientY);
    };

    return (
        <div 
            onClick={handleClick}
            onDoubleClick={handleDoubleClick}
            onMouseEnter={handleMouseEnter}
            onMouseMove={handleMouseMove}
            style={{ padding: '20px', border: '1px solid #ccc' }}
        >
            Наведи на меня курсор или кликни
        </div>
    );
}
```

### Keyboard Events

```jsx
function KeyboardEventsExample() {
    const handleKeyDown = (event) => {
        console.log('Клавиша нажата:', event.key, event.keyCode);
        
        if (event.key === 'Enter') {
            console.log('Нажата клавиша Enter');
        }
    };

    const handleKeyPress = (event) => {
        console.log('Клавиша нажата (keypress):', event.key);
    };

    const handleKeyUp = (event) => {
        console.log('Клавиша отпущена:', event.key);
    };

    return (
        <input 
            type="text" 
            placeholder="Начни вводить..."
            onKeyDown={handleKeyDown}
            onKeyPress={handleKeyPress}
            onKeyUp={handleKeyUp}
        />
    );
}
```

### Form Events

```jsx
import { useState } from 'react';

function FormEventsExample() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        message: ''
    });

    const handleInputChange = (event) => {
        const { name, value } = event.target;
        setFormData(prev => ({
            ...prev,
            [name]: value
        }));
    };

    const handleSubmit = (event) => {
        event.preventDefault(); // Предотвращаем стандартное поведение формы
        console.log('Данные формы:', formData);
    };

    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label>
                    Имя:
                    <input 
                        type="text" 
                        name="name" 
                        value={formData.name}
                        onChange={handleInputChange}
                    />
                </label>
            </div>
            
            <div>
                <label>
                    Email:
                    <input 
                        type="email" 
                        name="email" 
                        value={formData.email}
                        onChange={handleInputChange}
                    />
                </label>
            </div>
            
            <div>
                <label>
                    Сообщение:
                    <textarea 
                        name="message" 
                        value={formData.message}
                        onChange={handleInputChange}
                    />
                </label>
            </div>
            
            <button type="submit">Отправить</button>
        </form>
    );
}
```

## Передача аргументов в обработчики

### 1. Стрелочные функции

```jsx
function ListWithClickHandler() {
    const items = ['Яблоко', 'Банан', 'Апельсин'];

    const handleClick = (itemName) => {
        console.log(`Нажат элемент: ${itemName}`);
    };

    return (
        <ul>
            {items.map(item => (
                <li key={item} onClick={() => handleClick(item)}>
                    {item}
                </li>
            ))}
        </ul>
    );
}
```

### 2. bind-метод

```jsx
function ListWithClickHandler() {
    const items = ['Яблоко', 'Банан', 'Апельсин'];

    const handleClick = function(itemName, event) {
        console.log(`Нажат элемент: ${itemName}`, event);
    };

    return (
        <ul>
            {items.map(item => (
                <li key={item} onClick={handleClick.bind(this, item)}>
                    {item}
                </li>
            ))}
        </ul>
    );
}
```

## Работа с синтетическими событиями

### Синхронное и асинхронное поведение

```jsx
function SyntheticEventExample() {
    const [value, setValue] = useState('');

    const handleChange = (event) => {
        // Синтетическое событие доступно только синхронно
        console.log('Синхронно:', event.target.value); // Работает
        
        // Если нужно использовать событие асинхронно:
        const targetValue = event.target.value;
        
        setTimeout(() => {
            // event.target будет null, но targetValue - доступен
            console.log('Асинхронно:', targetValue);
        }, 0);
        
        // Или использовать persist() для сохранения события
        event.persist();
        setTimeout(() => {
            console.log('С асинхронным сохранением:', event.target.value);
        }, 0);
        
        setValue(event.target.value);
    };

    return <input type="text" value={value} onChange={handleChange} />;
}
```

## Делегирование событий

React автоматически использует делегирование событий, что делает его эффективным:

```jsx
function EventDelegationExample() {
    const handleClick = (event) => {
        if (event.target.tagName === 'BUTTON') {
            console.log('Нажата кнопка:', event.target.textContent);
        }
    };

    return (
        <div onClick={handleClick}>
            <button>Кнопка 1</button>
            <button>Кнопка 2</button>
            <button>Кнопка 3</button>
        </div>
    );
}
```

## Практические примеры

### Пример 1: Динамическое добавление обработчиков

```jsx
import { useState } from 'react';

function DynamicHandlers() {
    const [items, setItems] = useState([
        { id: 1, text: 'Элемент 1', active: false },
        { id: 2, text: 'Элемент 2', active: false },
        { id: 3, text: 'Элемент 3', active: false }
    ]);

    const toggleItem = (id) => {
        setItems(prevItems => 
            prevItems.map(item => 
                item.id === id 
                    ? { ...item, active: !item.active } 
                    : item
            )
        );
    };

    return (
        <div>
            {items.map(item => (
                <div 
                    key={item.id} 
                    onClick={() => toggleItem(item.id)}
                    style={{ 
                        padding: '10px', 
                        margin: '5px',
                        backgroundColor: item.active ? '#e3f2fd' : '#fff',
                        cursor: 'pointer'
                    }}
                >
                    {item.text} {item.active && '(активный)'}
                </div>
            ))}
        </div>
    );
}
```

### Пример 2: Работа с событиями вложенных компонентов

```jsx
function ParentComponent() {
    const handleParentClick = (event) => {
        console.log('Клик в родительском компоненте');
    };

    return (
        <div onClick={handleParentClick}>
            <h3>Родительский компонент</h3>
            <ChildComponent />
        </div>
    );
}

function ChildComponent() {
    const handleChildClick = (event) => {
        event.stopPropagation(); // Останавливает всплытие
        console.log('Клик в дочернем компоненте');
    };

    return (
        <div onClick={handleChildClick}>
            <p>Дочерний компонент</p>
        </div>
    );
}
```

## Hooks и события

### Использование useEffect для событий на window/document

```jsx
import { useEffect } from 'react';

function WindowEvents() {
    useEffect(() => {
        const handleResize = () => {
            console.log('Размер окна изменен:', window.innerWidth, window.innerHeight);
        };

        const handleScroll = () => {
            console.log('Прокрутка:', window.scrollY);
        };

        window.addEventListener('resize', handleResize);
        window.addEventListener('scroll', handleScroll);

        // Очистка - важна для предотвращения утечек памяти
        return () => {
            window.removeEventListener('resize', handleResize);
            window.removeEventListener('scroll', handleScroll);
        };
    }, []); // Пустой массив зависимостей - обработчики добавляются при монтировании

    return <div>Следит за событиями окна</div>;
}
```

### useCallback для оптимизации

```jsx
import { useCallback, useState } from 'react';

function OptimizedHandlers() {
    const [count, setCount] = useState(0);

    // useCallback предотвращает создание новой функции при каждом рендере
    const handleClick = useCallback(() => {
        setCount(prev => prev + 1);
    }, []); // Пустой массив зависимостей означает, что функция не изменяется

    return (
        <div>
            <p>Счетчик: {count}</p>
            <button onClick={handleClick}>Увеличить</button>
        </div>
    );
}
```

## Современные подходы

### Работа с пользовательскими хуками для событий

```jsx
import { useState, useEffect, useRef } from 'react';

// Пользовательский хук для отслеживания кликов вне компонента
function useClickOutside(handler) {
    const ref = useRef();

    useEffect(() => {
        const handleClickOutside = (event) => {
            if (ref.current && !ref.current.contains(event.target)) {
                handler();
            }
        };

        document.addEventListener('mousedown', handleClickOutside);
        return () => {
            document.removeEventListener('mousedown', handleClickOutside);
        };
    }, [handler]);

    return ref;
}

function Dropdown() {
    const [isOpen, setIsOpen] = useState(false);
    const dropdownRef = useClickOutside(() => setIsOpen(false));

    return (
        <div ref={dropdownRef} style={{ position: 'relative' }}>
            <button onClick={() => setIsOpen(!isOpen)}>
                {isOpen ? 'Закрыть' : 'Открыть'} меню
            </button>
            {isOpen && (
                <div style={{ 
                    position: 'absolute', 
                    top: '100%', 
                    left: 0, 
                    background: '#fff', 
                    border: '1px solid #ccc' 
                }}>
                    <div>Пункт 1</div>
                    <div>Пункт 2</div>
                    <div>Пункт 3</div>
                </div>
            )}
        </div>
    );
}
```

## Заключение

Обработка событий в React требует понимания особенностей синтетических событий и эффективного использования состояния компонентов. Правильное использование делегирования, оптимизированных обработчиков и хуков позволяет создавать интерактивные и производительные приложения.

## Смежные темы

- [[Обработка-событий]]
- [[Всплытие-и-перехват-событий]]
- [[События-в-Vue]]
- [[События-в-Svelte]]
- [[Хуки]]
- [[Компонентная-архитектура]]
- [[Фреймворки]]