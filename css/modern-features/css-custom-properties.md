---
aliases: [CSS Variables, CSS Custom Properties, Переменные CSS]
tags: [css, variables, custom-properties, frontend]
---

# CSS Custom Properties (Переменные CSS)

CSS Custom Properties, также известные как CSS Variables, представляют собой мощный инструмент в современном CSS, который позволяет определять переиспользуемые значения, которые можно изменять динамически в процессе выполнения. Это позволяет создавать более гибкие и поддерживаемые стили.

## Основы CSS Custom Properties

CSS Custom Properties - это специальные объявления в CSS, которые позволяют хранить значения (например, цвета, размеры, шрифты) в пользовательских свойствах и использовать их повторно по всему стилю. Они определяются с помощью префикса `--` и могут быть использованы с функцией `var()`.

```css
:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size: 16px;
}

.header {
  color: var(--primary-color);
  font-size: var(--font-size);
}
```

## Объявление и использование переменных

### Где объявлять переменные

CSS Custom Properties можно объявлять в любых селекторах, но наиболее распространенные места:

- `:root` - глобальные переменные, доступные по всему документу
- Специфические селекторы - локальные переменные для определенных компонентов
- Псевдоклассы, такие как `:hover`, `:focus` - для динамических изменений

```css
/* Глобальные переменные */
:root {
  --main-bg-color: #f0f0f0;
  --main-text-color: #333;
  --border-radius: 8px;
}

/* Локальные переменные для конкретного компонента */
.card {
  --card-padding: 20px;
  --card-shadow: 0 2px 10px rgba(0,0,0,0.1);
  padding: var(--card-padding);
  box-shadow: var(--card-shadow);
}
```

### Синтаксис объявления

Переменные CSS объявляются с использованием префикса `--` за которым следует имя переменной:

```css
селектор {
  --имя-переменной: значение;
}
```

### Использование переменных

Для использования переменной используется функция `var()`:

```css
селектор {
  свойство: var(--имя-переменной);
  /* или с fallback значением */
  свойство: var(--имя-переменной, fallback-значение);
}
```

## Область видимости и наследование

CSS Custom Properties подчиняются правилам наследования CSS. Переменные, объявленные в родительском элементе, доступны в дочерних элементах:

```css
.parent {
  --text-color: blue;
}

.child {
  color: var(--text-color); /* Наследует blue от .parent */
}
```

Если переменная не найдена в текущем контексте, браузер ищет её в родительских элементах до `:root`. Если переменная не найдена, используется fallback значение или свойство сбрасывается к значению по умолчанию.

## Практические примеры использования

### Цветовая палитра

Одним из наиболее распространенных применений переменных CSS является создание согласованной цветовой палитры:

```css
:root {
  /* Основные цвета */
  --primary: #3498db;
  --secondary: #2ecc71;
  --accent: #e74c3c;
  --neutral: #95a5a6;
  
  /* Цвета текста */
  --text-primary: #2c3e50;
  --text-secondary: #7f8c8d;
  --text-inverse: #ffffff;
  
  /* Цвета фона */
  --bg-primary: #ffffff;
  --bg-secondary: #ecf0f1;
  --bg-accent: #f8f9fa;
}
```

### Размеры и отступы

Переменные также полезны для создания согласованных размеров и отступов:

```css
:root {
  /* Размеры шрифта */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  
  /* Отступы */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Радиусы */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;
}
```

### Темы оформления

CSS Custom Properties идеально подходят для реализации тем оформления:

```css
/* Светлая тема (по умолчанию) */
:root {
  --bg-color: #ffffff;
  --text-color: #000000;
  --border-color: #e0e0e0;
}

/* Темная тема */
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
  --border-color: #444444;
}

/* Использование переменных */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
  border: 1px solid var(--border-color);
}
```

## Динамическое изменение переменных

CSS Custom Properties можно изменять динамически с помощью JavaScript:

```javascript
// Получение значения переменной
const element = document.documentElement;
const primaryColor = getComputedStyle(element).getPropertyValue('--primary-color');

// Установка нового значения переменной
element.style.setProperty('--primary-color', '#e74c3c');

// Изменение переменной при наведении
document.addEventListener('DOMContentLoaded', () => {
  const button = document.querySelector('.theme-toggle');
  button.addEventListener('click', () => {
    const currentTheme = document.body.getAttribute('data-theme');
    const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
    document.body.setAttribute('data-theme', newTheme);
  });
});
```

## Лучшие практики

### 1. Используйте осмысленные имена переменных

Имена переменных должны быть понятными и описывать их назначение:

```css
/* Хорошо */
:root {
  --primary-button-bg: #007bff;
  --header-height: 60px;
  --shadow-medium: 0 4px 8px rgba(0,0,0,0.1);
}

/* Плохо */
:root {
  --btn1: #007bff;
  --h: 60px;
  --s1: 0 4px 8px rgba(0,0,0,0.1);
}
```

### 2. Организуйте переменные в логические группы

Группируйте переменные по функциональности и используйте комментарии:

```css
:root {
  /* Цвета */
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  
  /* Размеры шрифта */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  
  /* Отступы */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  
  /* Радиусы */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
}
```

### 3. Используйте CSS-фреймворки и препроцессоры

CSS Custom Properties можно эффективно комбинировать с препроцессорами:

```scss
// Определение переменных в SCSS
$primary-color: #007bff;
$secondary-color: #6c757d;

:root {
  --primary-color: #{$primary-color};
  --secondary-color: #{$secondary-color};
  --hover-brightness: 110%;
}

// Использование в компонентах
.button {
  background-color: var(--primary-color);
  
  &:hover {
    filter: brightness(var(--hover-brightness));
  }
}
```

### 4. Обеспечьте совместимость с браузерами

Для поддержки старых браузеров используйте fallback значения:

```css
.element {
  /* Fallback без переменной */
  background-color: #007bff;
  /* Современное значение с переменной */
  background-color: var(--primary-color, #007bff);
}
```

## Продвинутые возможности

### Вычисления с переменными

CSS Custom Properties можно использовать с функцией `calc()` для выполнения вычислений:

```css
:root {
  --base-font-size: 16px;
  --font-scale: 1.2;
}

.text-large {
  font-size: calc(var(--base-font-size) * var(--font-scale));
}
```

### CSS Custom Properties в анимациях

Хотя CSS переменные не анимируются напрямую, их можно использовать в анимациях через JavaScript:

```css
.animated-element {
  width: var(--element-width, 100px);
  height: var(--element-height, 100px);
  background-color: var(--element-color, blue);
}
```

```javascript
const element = document.querySelector('.animated-element');
let width = 100;

function animate() {
  width += 1;
  if (width > 200) width = 100;
  
  element.style.setProperty('--element-width', width + 'px');
  element.style.setProperty('--element-color', `hsl(${width}, 70%, 50%)`);
  
  requestAnimationFrame(animate);
}

animate();
```

## Поддержка браузерами

CSS Custom Properties поддерживаются всеми современными браузерами:

- Chrome 49+
- Firefox 31+
- Safari 9.1+
- Edge 15+
- Opera 36+

Для поддержки более старых браузеров рекомендуется использовать fallback значения или полифиллы.

## Преимущества использования CSS Custom Properties

1. **Повторное использование значений** - одинаковые значения (цвета, размеры) можно определять один раз и использовать по всему проекту
2. **Простота изменения тем** - можно легко переключать между светлой и темной темой, изменяя значения переменных
3. **Динамическое изменение стилей** - переменные можно изменять с помощью JavaScript в реальном времени
4. **Улучшенная читаемость CSS** - именованные переменные делают код более понятным
5. **Легкость поддержки** - при изменении цвета или размера нужно изменить только одну переменную

## Потенциальные проблемы

1. **Не поддерживаются в IE** - Internet Explorer не поддерживает CSS Custom Properties
2. **Сложность отладки** - может быть сложнее отследить, откуда берется значение переменной
3. **Ограниченная анимация** - переменные не анимируются напрямую (требуется JavaScript или другие подходы)

## Сравнение с препроцессорами

CSS Custom Properties отличаются от переменных в препроцессорах (Sass, Less) следующими особенностями:

- **Время обработки**: Переменные препроцессоров обрабатываются на этапе компиляции, CSS Custom Properties - во время выполнения
- **Динамичность**: CSS Custom Properties можно изменять в реальном времени, переменные препроцессоров - нет
- **Наследование**: CSS Custom Properties наследуются как обычные CSS-свойства, переменные препроцессоров - нет
- **Область видимости**: CSS Custom Properties могут иметь разную область видимости в зависимости от селектора, переменные препроцессоров глобальные в рамках файла

## Заключение

CSS Custom Properties являются важным инструментом в современном веб-дизайне и разработке. Они обеспечивают гибкость, повторное использование и динамическое изменение стилей без необходимости перезаписи CSS. Использование переменных CSS значительно упрощает поддержку и масштабирование проектов, особенно при работе с дизайн-системами и темами оформления.

Для эффективного использования CSS Custom Properties рекомендуется:

- Организовать переменные в логические группы
- Использовать понятные имена переменных
- Обеспечить fallback значения для совместимости
- Рассмотреть использование в сочетании с препроцессорами и фреймворками
- Документировать переменные для командной разработки

CSS Custom Properties открывают возможности для создания более адаптивных и гибких пользовательских интерфейсов, позволяя легко адаптировать внешний вид приложения под различные условия и предпочтения пользователей.

## См. также

- [[CSS Architecture]]
- [[CSS Preprocessors]]
- [[Frontend Performance]]
- [[Design Systems]]
- [[CSS Grid]]
- [[CSS Flexbox]]
- [[CSS Animations]]
- [[CSS Specificity]]
- [[CSS Methodologies]]
- [[Web Accessibility]]
- [[Responsive Design]]
- [[CSS Units]]
- [[CSS Functions]]
- [[CSS Selectors]]
- [[CSS Cascade]]