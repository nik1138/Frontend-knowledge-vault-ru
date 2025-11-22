---
aliases: ["Интеграция CSS-in-JS с фреймворками", "React CSS-in-JS", "Vue CSS-in-JS", "Angular CSS-in-JS", "CSS-in-JS Framework Integration"]
tags: ["css", "css-in-js", "react", "vue", "angular", "frameworks", "integration"]
---

# Интеграция CSS-in-JS с различными фреймворками

CSS-in-JS библиотеки могут быть интегрированы с различными JavaScript-фреймворками. Рассмотрим особенности интеграции с наиболее популярными фреймворками: React, Vue и Angular.

## Интеграция с React

React является наиболее подходящей платформой для CSS-in-JS благодаря своей архитектуре компонентов.

### Styled Components в React

```jsx
import React from 'react';
import styled from 'styled-components';

// Создание стилизованного компонента
const Container = styled.div`
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  background-color: #f8f9fa;
`;

const Title = styled.h1`
  color: ${props => props.color || '#007bff'};
  margin-bottom: 1rem;
`;

const Button = styled.button`
  background-color: ${props => props.variant === 'primary' ? '#007bff' : '#6c757d'};
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
  
  ${props => props.disabled && `
    opacity: 0.6;
    cursor: not-allowed;
  `}
`;

// Компонент с использованием styled-components
const Card = ({ title, content, primary = false }) => (
  <Container>
    <Title color={primary ? '#28a745' : '#007bff'}>
      {title}
    </Title>
    <p>{content}</p>
    <Button variant={primary ? 'primary' : 'secondary'}>
      Нажми меня
    </Button>
  </Container>
);

export default Card;
```

### Emotion в React

```jsx
/** @jsxImportSource @emotion/react */
import React from 'react';
import { css } from '@emotion/react';
import styled from '@emotion/styled';

// Использование css пропа
const StyledDiv = ({ children, primary }) => (
  <div
    css={css`
      padding: 1rem;
      background-color: ${primary ? '#d4edda' : '#f8f9fa'};
      border: 1px solid ${primary ? '#c3e6cb' : '#dee2e6'};
      border-radius: 0.25rem;
    `}
  >
    {children}
  </div>
);

// Использование styled компонентов
const StyledButton = styled.button`
  background-color: ${props => props.primary ? '#28a745' : '#007bff'};
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
`;

const EmotionExample = () => (
  <div>
    <StyledDiv primary>
      <h3>Заголовок</h3>
      <p>Содержимое</p>
    </StyledDiv>
    <StyledButton primary>Главная кнопка</StyledButton>
    <StyledButton>Обычная кнопка</StyledButton>
  </div>
);

export default EmotionExample;
```

### Stitches в React

```jsx
import React from 'react';
import { createStitches } from '@stitches/react';

const { styled, css, theme } = createStitches({
  theme: {
    colors: {
      primary: '#007bff',
      secondary: '#6c757d',
      success: '#28a745',
      background: '#ffffff',
      text: '#212529',
    },
    space: {
      1: '0.25rem',
      2: '0.5rem',
      3: '1rem',
      4: '2rem',
    },
  },
  utils: {
    marginX: (value) => ({ marginLeft: value, marginRight: value }),
    marginY: (value) => ({ marginTop: value, marginBottom: value }),
  },
});

const Card = styled('div', {
  backgroundColor: '$background',
  color: '$text',
  padding: '$3',
  borderRadius: '0.25rem',
  boxShadow: '0 0.125rem 0.25rem rgba(0, 0, 0, 0.075)',
  
  variants: {
    size: {
      small: {
        maxWidth: '20rem',
      },
      medium: {
        maxWidth: '30rem',
      },
      large: {
        maxWidth: '40rem',
      },
    },
    variant: {
      primary: {
        border: '2px solid $primary',
      },
      success: {
        border: '2px solid $success',
      },
    },
  },
  
  defaultVariants: {
    size: 'medium',
    variant: 'primary',
  },
});

const Button = styled('button', {
  backgroundColor: '$primary',
  color: 'white',
  border: 'none',
  padding: '$2',
  borderRadius: '0.25rem',
  cursor: 'pointer',
  
  '&:hover': {
    opacity: 0.8,
  },
  
  variants: {
    variant: {
      primary: {
        backgroundColor: '$primary',
      },
      success: {
        backgroundColor: '$success',
      },
      secondary: {
        backgroundColor: '$secondary',
      },
    },
    size: {
      small: {
        fontSize: '0.75rem',
        padding: '$1 $2',
      },
      large: {
        fontSize: '1.25rem',
        padding: '$3 $4',
      },
    },
  },
  
  defaultVariants: {
    variant: 'primary',
    size: 'small',
  },
});

const StitchesExample = () => (
  <Card size="large" variant="success">
    <h3>Заголовок карточки</h3>
    <p>Содержимое карточки с использованием Stitches</p>
    <Button variant="success" size="large">
      Успешная кнопка
    </Button>
  </Card>
);

export default StitchesExample;
```

## Интеграция с Vue.js

Хотя Vue.js не так тесно интегрирован с CSS-in-JS как React, существуют решения для его использования.

### Использование Emotion в Vue

```vue
<template>
  <div :css="containerStyle">
    <h1 :css="titleStyle">Привет из Vue с Emotion!</h1>
    <button 
      :css="buttonStyle" 
      @click="handleClick"
      :disabled="disabled"
    >
      {{ buttonText }}
    </button>
  </div>
</template>

<script>
import { css } from '@emotion/css';

export default {
  name: 'VueEmotionExample',
  data() {
    return {
      disabled: false,
      buttonText: 'Нажми меня'
    };
  },
  computed: {
    containerStyle() {
      return css`
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        background-color: #f8f9fa;
        border-radius: 0.25rem;
      `;
    },
    titleStyle() {
      return css`
        color: #007bff;
        margin-bottom: 1rem;
      `;
    },
    buttonStyle() {
      return css`
        background-color: ${this.disabled ? '#6c757d' : '#007bff'};
        color: white;
        border: none;
        padding: 0.5rem 1rem;
        border-radius: 0.25rem;
        cursor: ${this.disabled ? 'not-allowed' : 'pointer'};
        
        &:hover {
          opacity: 0.8;
        }
      `;
    }
  },
  methods: {
    handleClick() {
      this.disabled = true;
      this.buttonText = 'Отправлено!';
    }
  }
};
</script>
```

### Альтернатива: Vue с scoped стилями и CSS-переменными

```vue
<template>
  <div class="card" :class="{ 'card--primary': isPrimary }">
    <h3 class="card__title">{{ title }}</h3>
    <p class="card__content">{{ content }}</p>
    <button 
      class="card__button"
      :style="{ '--button-color': buttonColor }"
      @click="onClick"
    >
      {{ buttonText }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'VueCard',
  props: {
    title: String,
    content: String,
    isPrimary: Boolean,
    buttonColor: {
      type: String,
      default: '#007bff'
    }
  },
  data() {
    return {
      buttonText: 'Нажми меня'
    };
  },
  methods: {
    onClick() {
      this.buttonText = 'Нажато!';
    }
  }
};
</script>

<style scoped>
.card {
  padding: 1rem;
  border-radius: 0.25rem;
  background-color: #f8f9fa;
  max-width: 300px;
}

.card--primary {
  border: 2px solid #007bff;
}

.card__title {
  color: #007bff;
  margin-bottom: 0.5rem;
}

.card__button {
  background-color: var(--button-color);
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
}
</style>
```

## Интеграция с Angular

Angular имеет встроенную поддержку стилизации компонентов, но CSS-in-JS также может быть интегрирован.

### Использование Emotion в Angular

```typescript
// component.ts
import { Component, OnInit } from '@angular/core';
import { css, injectGlobal } from '@emotion/css';

@Component({
  selector: 'app-styled-component',
  template: `
    <div [ngClass]="containerClass">
      <h1 [ngClass]="titleClass">Angular с Emotion</h1>
      <p [ngClass]="contentClass">{{ message }}</p>
      <button [ngClass]="buttonClass" (click)="onClick()">Нажми меня</button>
    </div>
  `,
})
export class StyledComponent implements OnInit {
  message = 'Пример использования Emotion в Angular';
  containerClass = '';
  titleClass = '';
  contentClass = '';
  buttonClass = '';

  ngOnInit() {
    // Создание CSS-классов
    this.containerClass = css`
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 1rem;
      background-color: #f8f9fa;
      border-radius: 0.25rem;
      max-width: 400px;
      margin: 0 auto;
    `;

    this.titleClass = css`
      color: #007bff;
      margin-bottom: 1rem;
      text-align: center;
    `;

    this.contentClass = css`
      margin-bottom: 1rem;
      text-align: center;
    `;

    this.buttonClass = css`
      background-color: #007bff;
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      border-radius: 0.25rem;
      cursor: pointer;
      
      &:hover {
        opacity: 0.8;
      }
    `;
  }

  onClick() {
    this.message = 'Кнопка была нажата!';
  }
}
```

### Angular с ViewEncapsulation

```typescript
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <h3 class="card__title">{{ title }}</h3>
      <p class="card__content">{{ content }}</p>
      <button class="card__button" (click)="onButtonClick()">
        {{ buttonLabel }}
      </button>
    </div>
  `,
  styles: [`
    .card {
      padding: 1rem;
      border-radius: 0.25rem;
      background-color: #f8f9fa;
      max-width: 300px;
      margin: 1rem;
    }
    
    .card__title {
      color: #007bff;
      margin-bottom: 0.5rem;
    }
    
    .card__button {
      background-color: var(--button-color, #007bff);
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      border-radius: 0.25rem;
      cursor: pointer;
    }
    
    .card__button:hover {
      opacity: 0.8;
    }
  `],
  encapsulation: ViewEncapsulation.Emulated // Это изолирует стили компонента
})
export class CardComponent {
  title = 'Angular Card';
  content = 'Пример компонента с изолированными стилями';
  buttonLabel = 'Нажми меня';
  
  onButtonClick() {
    this.buttonLabel = 'Нажато!';
  }
}
```

## Сравнение интеграции с фреймворками

| Фреймворк | Уровень интеграции | Производительность | Простота использования |
|-----------|-------------------|-------------------|------------------------|
| React | Высокий | Хорошая | Высокая |
| Vue | Средний | Хорошая | Средняя |
| Angular | Средний | Удовлетворительная | Низкая-Средняя |

### Особенности интеграции:

1. **React**: Наиболее естественная интеграция благодаря схожести парадигм компонентного подхода
2. **Vue**: Требует дополнительных настроек, но возможно использование
3. **Angular**: Наименее естественная интеграция, так как Angular имеет встроенную систему стилизации

## Практические рекомендации

### Для React:
- Используйте styled-components или Emotion для максимальной интеграции
- Рассмотрите Stitches для создания дизайн-систем
- Обратите внимание на производительность при большом количестве компонентов

### Для Vue:
- Рассмотрите использование scoped CSS и CSS-переменных как альтернативу
- Если нужен CSS-in-JS, Emotion может быть хорошим выбором
- Используйте CSS-модули как промежуточный вариант

### Для Angular:
- Встроенные возможности Angular часто достаточны
- CSS-in-JS добавляет сложность без значительной пользы
- Рассмотрите CSS-переменные для динамических стилей

## SSR (Server-Side Rendering) со всеми фреймворками

Все фреймворки требуют специальной настройки для корректного рендеринга CSS-in-JS на сервере:

### React SSR с Styled Components:
```jsx
import { ServerStyleSheet } from 'styled-components';

const sheet = new ServerStyleSheet();
try {
  const html = ReactDOMServer.renderToString(
    sheet.collectStyles(<App />)
  );
  const css = sheet.getStyleTags();
  // Включить css в HTML ответ
} finally {
  sheet.seal();
}
```

### Vue SSR:
```javascript
// В файле entry-server.js
import { createApp } from './app';

export default (context) => {
  const { app } = createApp();
  return app;
};
```

### Angular Universal:
```typescript
// app.server.module.ts
@NgModule({
  imports: [
    AppModule,
    ServerModule,
  ],
  bootstrap: [AppComponent],
})
export class AppServerModule {}
```

> [!tip] Совет
> Для React проектов CSS-in-JS интегрируется наиболее естественным образом, в то время как для Vue и Angular могут потребоваться дополнительные настройки.

> [!warning] Важно
> При использовании CSS-in-JS в проектах с SSR обязательно настройте серверный рендеринг стилей, иначе пользователи увидят "мелькание нестилизованного контента" (FOUC).

[[CSS-in-JS библиотеки: styled-components, emotion, stitches]]
[[Преимущества и недостатки CSS-in-JS подхода]]
[[CSS-архитектура]]