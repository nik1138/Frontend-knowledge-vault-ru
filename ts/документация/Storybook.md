---
aliases: [Storybook, Документирование компонентов, Storybook для TypeScript]
tags: [typescript, documentation, storybook, components, testing]
---

# Storybook: Документирование и тестирование компонентов

Storybook — это мощный инструмент для разработки, документирования и тестирования компонентов пользовательского интерфейса изолированно. Он особенно полезен в TypeScript проектах, где статическая типизация помогает создавать более надежные и документированные компоненты.

## Введение в Storybook

Storybook позволяет разработчикам создавать компоненты изолированно без необходимости запускать всю систему. Это идеально подходит для:
- Разработки компонентов в изоляции
- Документирования API компонентов
- Тестирования визуальных изменений
- Создания каталога компонентов
- Совместной работы между разработчиками и дизайнерами

## Установка и настройка

Для TypeScript проекта установка Storybook включает в себя несколько шагов:

```bash
# Установка Storybook
npx sb init

# Для React проекта с TypeScript
npx sb init --type react
```

После установки Storybook автоматически настроит конфигурационные файлы и добавит необходимые зависимости.

### Основная структура

Типичная структура файлов Storybook:

```
.storybook/
  main.js          # Основная конфигурация
  preview.js       # Глобальные настройки
src/
  components/
    Button/
      Button.tsx
      Button.stories.tsx
      Button.test.tsx
```

## Создание историй (stories)

Истории (stories) в Storybook описывают различные состояния компонентов. Для TypeScript проектов важно правильно типизировать компоненты и истории.

### Пример истории для кнопки

```tsx
// Button.stories.tsx
import React from 'react';
import { StoryFn, Meta } from '@storybook/react';

import { Button, ButtonProps } from './Button';

export default {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    backgroundColor: { control: 'color' },
    size: {
      control: { type: 'select' },
      options: ['small', 'medium', 'large'],
    },
  },
} as Meta<typeof Button>;

const Template: StoryFn<typeof Button> = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
  primary: true,
  label: 'Button',
};

export const Secondary = Template.bind({});
Secondary.args = {
  label: 'Button',
};

export const Large = Template.bind({});
Large.args = {
  size: 'large',
  label: 'Button',
};

export const Small = Template.bind({});
Small.args = {
  size: 'small',
  label: 'Button',
};
```

### Типизация компонентов

В TypeScript важно правильно типизировать пропсы компонентов и истории:

```tsx
// Button.tsx
import React from 'react';

export interface ButtonProps {
  /**
   * Является ли кнопка основной
   */
  primary?: boolean;
  /**
   * Размер кнопки
   */
  size?: 'small' | 'medium' | 'large';
  /**
   * Текст кнопки
   */
  label: string;
  /**
   * Обработчик клика
   */
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({
  primary = false,
  size = 'medium',
  label,
  onClick,
}) => {
  const mode = primary ? 'storybook-button--primary' : 'storybook-button--secondary';
  
  return (
    <button
      type="button"
      className={['storybook-button', `storybook-button--${size}`, mode].join(' ')}
      onClick={onClick}
    >
      {label}
    </button>
  );
};
```

## Документирование API компонентов

Storybook автоматически извлекает информацию из JSDoc/TSDoc комментариев и TypeScript типов, создавая интерактивную документацию.

### Использование Controls

Controls позволяют интерактивно изменять пропсы компонентов:

```tsx
export default {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'], // Автоматическая генерация документации
  argTypes: {
    backgroundColor: {
      control: { type: 'color' },
      description: 'Цвет фона кнопки',
      table: {
        defaultValue: { summary: '#000000' },
      },
    },
    onClick: { action: 'clicked' },
  },
} as Meta<typeof Button>;
```

### Таблица свойств (Props Table)

Storybook автоматически генерирует таблицу свойств на основе TypeScript интерфейсов:

```tsx
/**
 * Универсальная кнопка с поддержкой различных стилей
 */
export interface ButtonProps {
  /**
   * Является ли кнопка первичной
   * @default false
   */
  primary?: boolean;
  
  /**
   * Размер кнопки
   * @default 'medium'
   */
  size?: 'small' | 'medium' | 'large';
  
  /**
   * Текст кнопки
   */
  label: string;
  
  /**
   * Функция, вызываемая при клике
   */
  onClick?: () => void;
}
```

## Документирование сложных компонентов

### Использование Decorators

Для компонентов, требующих контекста или оберток:

```tsx
import { ThemeProvider } from 'styled-components';

export default {
  title: 'Components/Modal',
  component: Modal,
  decorators: [
    (Story) => (
      <ThemeProvider theme={theme}>
        <Story />
      </ThemeProvider>
    ),
  ],
} as Meta<typeof Modal>;
```

### Использование Args

Args позволяют легко управлять состоянием компонентов:

```tsx
export const WithLongContent = Template.bind({});
WithLongContent.args = {
  title: 'Длинный заголовок',
  children: (
    <div>
      <p>Очень длинный контент...</p>
      <p>Еще больше контента...</p>
    </div>
  ),
  isOpen: true,
};
```

## Автоматическая документация

### Autodocs

Storybook 7+ предоставляет автоматическую генерацию документации:

```tsx
export default {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
} as Meta<typeof Button>;
```

Это автоматически генерирует страницу документации с:
- Примерами использования
- Таблицей свойств
- Интерактивными Controls
- Предварительным просмотром кода

### MDX документация

Для более сложной документации можно использовать MDX формат:

```mdx
{/* Button.mdx */}
import { Meta, Story, Canvas } from '@storybook/addon-docs';
import { Button } from './Button';
import { Primary } from './Button.stories';

<Meta title="Components/Button/Documentation" component={Button} />

# Кнопка

Компонент Button используется для создания интерактивных элементов управления.

## Основное использование

<Canvas>
  <Story of={Primary} />
</Canvas>

## Доступные свойства

<Button argTypes={{}} />
```

## Тестирование компонентов

Storybook интегрируется с различными инструментами тестирования:

### Visual Testing

```tsx
// Button.test.tsx
import { render } from '@testing-library/react';
import { composeStories } from '@storybook/testing-react';
import * as stories from './Button.stories';

const { Primary, Secondary } = composeStories(stories);

test('renders primary button with correct text', () => {
  const { getByText } = render(<Primary />);
  expect(getByText('Button')).toBeInTheDocument();
});
```

### Snapshot Testing

```tsx
// Использование Storyshots
import initStoryshots from '@storybook/addon-storyshots';

initStoryshots();
```

## Практические примеры

### Документирование формы

```tsx
// Form.stories.tsx
import React from 'react';
import { StoryFn, Meta } from '@storybook/react';
import { Form, FormProps } from './Form';

export default {
  title: 'Components/Form',
  component: Form,
  parameters: {
    layout: 'centered',
  },
} as Meta<typeof Form>;

const Template: StoryFn<typeof Form> = (args) => <Form {...args} />;

export const LoginForm = Template.bind({});
LoginForm.args = {
  fields: [
    { name: 'email', type: 'email', label: 'Email' },
    { name: 'password', type: 'password', label: 'Пароль' },
  ],
  onSubmit: (data) => console.log(data),
};
```

### Документирование сложного компонента с состоянием

```tsx
// Accordion.stories.tsx
import React from 'react';
import { StoryFn, Meta } from '@storybook/react';
import { Accordion, AccordionProps } from './Accordion';

export default {
  title: 'Components/Accordion',
  component: Accordion,
} as Meta<typeof Accordion>;

const Template: StoryFn<typeof Accordion> = (args) => (
  <Accordion {...args} />
);

export const Default = Template.bind({});
Default.args = {
  items: [
    { title: 'Заголовок 1', content: 'Содержимое 1' },
    { title: 'Заголовок 2', content: 'Содержимое 2' },
    { title: 'Заголовок 3', content: 'Содержимое 3' },
  ],
};
```

## Лучшие практики

> [!tip] Совет
> Используйте Storybook не только для визуального тестирования, но и как живую документацию для ваших компонентов.

1. **Создавайте истории для всех возможных состояний компонента**
2. **Используйте осмысленные названия историй**
3. **Документируйте сложные компоненты с помощью MDX**
4. **Добавляйте примеры с различными пропсами**
5. **Используйте Controls для интерактивного тестирования**
6. **Создавайте истории для граничных случаев**

## Интеграция с CI/CD

Storybook можно интегрировать в процесс CI/CD для автоматического деплоя документации:

```yaml
# .github/workflows/deploy-storybook.yml
name: Deploy Storybook
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: npx storybook build
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./storybook-static
```

## Связанные темы

- [[TSDoc]] - для понимания стандартов документирования
- [[Документирование-API]] - для документирования API компонентов
- [[Генерация-документации]] - для автоматической генерации документации
- [[Лучшие-практики-документирования]] - общие рекомендации по документированию

Storybook является мощным инструментом для документирования и тестирования компонентов в TypeScript проектах. Его правильное использование помогает создавать живую документацию, упрощает разработку и тестирование компонентов, а также улучшает совместную работу в команде.