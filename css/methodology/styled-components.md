# Styled Components

Этот файл описывает библиотеку Styled Components.

Styled Components — это библиотека для React, позволяющая использовать компонентно-ориентированный подход к CSS. Она позволяет писать настоящий CSS-код в JavaScript, создавая компоненты со встроенными стилями.

## Основные особенности:

- Встроенные стили: CSS-код пишется непосредственно в JavaScript
- Темизация: Поддержка тем для всего приложения
- Адаптивность: Легкое создание адаптивных компонентов
- Автоматическая очистка: Неиспользуемые стили автоматически удаляются

## Пример:

```javascript
import styled from 'styled-components';

const Button = styled.button`
  padding: 10px 20px;
  background-color: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  border: none;
  border-radius: 4px;
  
  &:hover {
    background-color: ${props => props.primary ? '#0056b3' : '#545b62'};
  }
`;

function App() {
  return <Button primary>Primary Button</Button>;
}
```

## Преимущества:

- Компонентный подход: Стили связаны с компонентами
- Темизация: Легкое изменение темы всего приложения
- Динамические стили: Легкое изменение стилей на основе пропсов

## См. также:
- [[css-in-js]]
- [[emotion]]
- [[css-modules]]

#styled-components #css-in-js #react #web-development