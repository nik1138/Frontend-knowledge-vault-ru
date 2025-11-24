---
aliases: ["CSS Модули", "CSS Modules", "Стилевые модули"]
tags: [css, модульность, фронтенд, стили, компоненты]
---

# Модули CSS (CSS Modules)

**CSS Modules** - это техника инкапсуляции стилей на уровне компонента, которая автоматически создает уникальные имена классов для стилей, предотвращая конфликты имен и побочные эффекты от стилей. Это позволяет создавать изолированные стили для компонентов, как если бы использовался Shadow DOM, но без необходимости в специальных API.

## Основные понятия

CSS Modules позволяют использовать обычные CSS-файлы, но с возможностью изоляции. Каждый CSS-файл обрабатывается так, что имена классов становятся уникальными в глобальной области видимости, но остаются понятными и предсказуемыми в рамках одного компонента.

## Пример использования

```css
/* styles.module.css */
.container {
  padding: 20px;
  margin: 10px;
}

.title {
  font-size: 24px;
  color: #333;
}

.button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
}
```

```jsx
// Component.jsx
import React from 'react';
import styles from './styles.module.css';

const MyComponent = () => {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Заголовок компонента</h1>
      <button className={styles.button}>Нажми меня</button>
    </div>
  );
};

export default MyComponent;
```

## Преимущества CSS Modules

- **Изоляция стилей**: Предотвращает конфликты имен классов между компонентами
- **Предсказуемость**: Легко понять, какие стили принадлежат какому компоненту
- **Поддерживаемость**: Упрощает рефакторинг и удаление неиспользуемых стилей
- **Интеграция**: Хорошо работает с популярными фреймворками и сборщиками

## Варианты реализации

### 1. С использованием Webpack

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
            },
          },
        ],
      },
    ],
  },
};
```

### 2. С использованием Create React App

В Create React App CSS Modules включены по умолчанию для файлов с именами вида `*.module.css`.

## Продвинутые возможности

### Локальные и глобальные имена классов

```css
/* По умолчанию все имена классов локальные */
.localClass {
  color: red;
}

/* Глобальное имя класса */
:global(.globalClass) {
  color: blue;
}

/* Смешанный подход */
.localClass :global(.globalChild) {
  margin: 10px;
}
```

### Композиция классов

```css
.className {
  color: green;
  background: red;
}

.otherClassName {
  composes: className;
  color: yellow;
}
```

## Сравнение с другими подходами

| Подход | Изоляция | Удобство | Производительность |
|--------|----------|----------|-------------------|
| CSS Modules | Высокая | Высокая | Высокая |
| Inline стили | Полная | Средняя | Средняя |
| BEM | Низкая | Низкая | Высокая |
| Styled Components | Высокая | Высокая | Средняя |

## Практические рекомендации

- Используйте осмысленные имена классов, даже в модульной системе
- Объединяйте CSS Modules с препроцессорами (Sass, Less) для более мощных возможностей
- Рассмотрите использование CSS-in-JS решений для более сложных случаев
- Следите за размером итогового CSS-бандла

## Связанные концепции

- [[Компонентная-архитектура]]
- [[CSS]]
- [[Стилизация компонентов]]
- [[Изоляция стилей]]

## Внешние ресурсы

- [Официальная документация CSS Modules](https://github.com/css-modules/css-modules)
- [CSS Modules в Create React App](https://create-react-app.dev/docs/adding-css-modules)