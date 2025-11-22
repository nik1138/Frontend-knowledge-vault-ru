---
tags: [typescript, frontend, definitely-typed, type-declarations]
aliases: [DefinitelyTyped, Типы объявлений]
---

# Работа с DefinitelyTyped

## Введение

DefinitelyTyped - это репозиторий с определениями типов для библиотек JavaScript, которые не содержат встроенной типизации. Это важный ресурс для разработчиков TypeScript, позволяющий использовать сторонние библиотеки с полной типобезопасностью.

## Что такое DefinitelyTyped

DefinitelyTyped (DT) - это крупнейший репозиторий с определениями типов для JavaScript библиотек. Он содержит типы для тысяч популярных библиотек, таких как Lodash, jQuery, Express и многих других.

### Структура репозитория

```
DefinitelyTyped/
├── types/
│   ├── lodash/
│   │   ├── index.d.ts
│   │   ├── lodash-tests.ts
│   │   └── tsconfig.json
│   ├── react/
│   │   ├── index.d.ts
│   │   ├── react-tests.ts
│   │   └── tsconfig.json
│   └── ...
```

Каждая библиотека имеет свою папку с файлами определений типов (.d.ts), тестами и конфигурацией TypeScript.

## Установка типов из DefinitelyTyped

### Установка через npm

```bash
# Установка типов для конкретной библиотеки
npm install --save-dev @types/lodash
npm install --save-dev @types/react
npm install --save-dev @types/node
```

### Пример использования типов

```ts
// После установки @types/lodash
import * as _ from 'lodash';

// Теперь у нас есть полная типизация
const numbers = [1, 2, 3, 4, 5];
const doubled = _.map(numbers, n => n * 2); // Тип: number[]
const sum = _.sum(numbers); // Тип: number

// Типизированные методы
const grouped = _.groupBy(['one', 'two', 'three'], 'length');
// Тип: { [key: number]: string[] }
```

## Создание собственных определений типов

### Простое определение типа (.d.ts)

```ts
// my-library.d.ts
declare module 'my-library' {
    // Определение типа для функции
    export function myFunction(input: string): number;
    
    // Определение типа для класса
    export class MyClass {
        constructor(options: MyOptions);
        doSomething(): void;
    }
    
    // Определение интерфейса
    export interface MyOptions {
        enabled?: boolean;
        count?: number;
    }
    
    // Экспорт по умолчанию
    const defaultExport: {
        version: string;
        init(): void;
    };
    
    export default defaultExport;
}
```

### Более сложное определение с пространством имен

```ts
// advanced-library.d.ts
declare namespace AdvancedLibrary {
    interface Config {
        apiKey: string;
        timeout: number;
    }
    
    class Client {
        constructor(config: Config);
        getData(id: number): Promise<any>;
        setData(id: number, data: any): Promise<void>;
    }
    
    function createClient(config: Config): Client;
}

// Глобальная переменная (если библиотека добавляет глобальную переменную)
declare const advancedLib: typeof AdvancedLibrary;

export = AdvancedLibrary;
export as namespace AdvancedLibrary;
```

## Практические примеры для frontend разработки

### Типизация для популярных библиотек

```ts
// Пример с @types/react
import React, { useState, useEffect } from 'react';

interface User {
    id: number;
    name: string;
    email: string;
}

const UserList: React.FC = () => {
    const [users, setUsers] = useState<User[]>([]);
    const [loading, setLoading] = useState<boolean>(true);
    
    useEffect(() => {
        fetchUsers()
            .then(setUsers)
            .finally(() => setLoading(false));
    }, []);
    
    if (loading) return <div>Loading...</div>;
    
    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
};

async function fetchUsers(): Promise<User[]> {
    const response = await fetch('/api/users');
    return response.json();
}
```

### Типизация для библиотек анимации

```ts
// Пример с @types/framer-motion
import { motion } from 'framer-motion';

const AnimatedBox = () => {
    const containerVariants = {
        hidden: { opacity: 0 },
        visible: {
            opacity: 1,
            transition: {
                staggerChildren: 0.1
            }
        }
    };
    
    const itemVariants = {
        hidden: { y: 20, opacity: 0 },
        visible: {
            y: 0,
            opacity: 1
        }
    };
    
    return (
        <motion.div
            variants={containerVariants}
            initial="hidden"
            animate="visible"
        >
            <motion.div variants={itemVariants}>Item 1</motion.div>
            <motion.div variants={itemVariants}>Item 2</motion.div>
            <motion.div variants={itemVariants}>Item 3</motion.div>
        </motion.div>
    );
};
```

### Типизация для библиотек стилей

```ts
// Пример с @types/styled-components
import styled from 'styled-components';

interface ButtonProps {
    variant?: 'primary' | 'secondary';
    size?: 'small' | 'medium' | 'large';
    disabled?: boolean;
}

const StyledButton = styled.button<ButtonProps>`
    background-color: ${props => 
        props.variant === 'primary' ? '#007bff' : '#6c757d'
    };
    color: white;
    border: none;
    padding: ${props => {
        switch(props.size) {
            case 'small': return '0.25rem 0.5rem';
            case 'large': return '0.75rem 1.5rem';
            default: return '0.5rem 1rem';
        }
    }};
    cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};
    opacity: ${props => props.disabled ? 0.6 : 1};
`;
```

## Создание типов для собственных библиотек

### Пример структуры определения типов

```ts
// my-ui-library/index.d.ts
// Type definitions for my-ui-library 1.0
// Project: https://github.com/username/my-ui-library
// Definitions by: Your Name <https://github.com/your-username>

export as namespace MyUILibrary;

export interface ButtonProps {
    /**
     * Текст кнопки
     */
    children: React.ReactNode;
    
    /**
     * Тип кнопки
     */
    variant?: 'primary' | 'secondary' | 'danger';
    
    /**
     * Размер кнопки
     */
    size?: 'small' | 'medium' | 'large';
    
    /**
     * Обработчик клика
     */
    onClick?: React.MouseEventHandler<HTMLButtonElement>;
    
    /**
     * Состояние загрузки
     */
    loading?: boolean;
    
    /**
     * Отключенное состояние
     */
    disabled?: boolean;
}

export const Button: React.FC<ButtonProps>;

export interface ModalProps {
    isOpen: boolean;
    onClose: () => void;
    children: React.ReactNode;
}

export const Modal: React.FC<ModalProps>;
```

### Тестирование определений типов

```ts
// my-ui-library/my-ui-library-tests.ts
// Файл для тестирования корректности определений типов

import { Button, Modal } from 'my-ui-library';

// Тестирование Button
const button1 = <Button variant="primary" size="large">Click me</Button>;
const button2 = <Button variant="danger" loading={true} disabled={false}>Delete</Button>;

// Эти строки должны вызвать ошибки типизации при неправильном использовании
// const button3 = <Button variant="invalid">Invalid variant</Button>; // Ошибка
// const button4 = <Button invalidProp="value">Invalid prop</Button>; // Ошибка

// Тестирование Modal
const modal = (
    <Modal isOpen={true} onClose={() => console.log('closed')}>
        <p>Modal content</p>
    </Modal>
);
```

## Лучшие практики работы с DefinitelyTyped

### Проверка наличия типов

```bash
# Проверить, существуют ли типы для библиотеки
npm view @types/react versions --json
```

### Обновление типов

```bash
# Обновить типы до последней версии
npm update @types/react
```

### Создание кастомных типов при отсутствии в DT

```ts
// types/missing-library.d.ts
declare module 'missing-library' {
    // Простое определение, если библиотека не имеет типов в DT
    export function someFunction(options: any): any;
    export class SomeClass {
        constructor(config: any);
        someMethod(): void;
    }
}
```

### Проверка совместимости версий

```json
// package.json
{
  "dependencies": {
    "some-library": "^2.5.0"
  },
  "devDependencies": {
    "@types/some-library": "^2.5.0"  // Версия должна соответствовать основной библиотеке
  }
}
```

## Стандарты и руководства по написанию типов

### Структура файла определений

```ts
// types/library-name/index.d.ts
// Type definitions for [LIBRARY NAME] [VERSION]
// Project: [URL TO PROJECT]
// Definitions by: [YOUR NAME] <[URL TO YOUR GITHUB PROFILE]>
// Definitions: https://github.com/DefinitelyTyped/DefinitelyTyped

// Объявление модуля
declare module 'library-name' {
    // Экспорт типов, интерфейсов и функций
    export interface Options {
        // ...
    }
    
    export function mainFunction(options: Options): void;
    
    // ...
}
```

### Использование правильных паттернов типизации

```ts
// Правильно: использование generic для функций
export function map<T, U>(array: T[], callback: (item: T) => U): U[];

// Правильно: использование перегрузок функций
export function parse(value: string): string | null;
export function parse(value: number): number;
export function parse(value: string | number): any {
    // реализация
}
```

## Заключение

Работа с DefinitelyTyped является важной частью разработки на TypeScript, особенно в frontend разработке, где часто используются сторонние библиотеки. Правильное использование и создание определений типов обеспечивает типобезопасность и улучшает опыт разработки.

> [!tip] Совет
> Всегда проверяйте наличие типов в DefinitelyTyped перед созданием собственных определений.

> [!warning] Важно
> Следите за совместимостью версий основной библиотеки и её типов из DefinitelyTyped.

## Связанные темы

- [[Утилиты типов TypeScript]]
- [[Интерфейсы и классы]]
- [[Модули и системы сборки]]