---
aliases: [CSS Color Adjust Specification, CSS Color Adjustment, CSS Color Correction, CSS Color Theme Adjustment]
tags: [css, color, adjustment, specification, css-color-adjust]
---

# CSS Color Adjust: Корректировка цвета для темной темы и т.д. - Полное руководство

## Введение

CSS Color Adjust Module определяет способы корректировки цветов веб-страниц для соответствия настройкам пользователя, таким как предпочтения темного режима, высокой контрастности или уменьшения интенсивности анимации. Модуль позволяет веб-сайтам адаптироваться к системным настройкам пользователя, улучшая доступность и пользовательский опыт.

## CSS Color Adjust Module Level 1

Level 1 определяет основные свойства для корректировки цветов.

### Свойство color-adjust

Свойство `color-adjust` (также известное как `-webkit-print-color-adjust`) позволяет контролировать, как браузер корректирует цвета для соответствия настройкам пользователя.

```css
.color-adjust-examples {
  /* Позволяет браузеру корректировать цвета (по умолчанию) */
  color-adjust: economy;            /* Браузер может корректировать цвета */
  -webkit-print-color-adjust: economy; /* Альтернативное имя для WebKit */
  
  /* Запрещает корректировку цветов */
  color-adjust: exact;              /* Браузер не должен корректировать цвета */
  -webkit-print-color-adjust: exact; /* Альтернативное имя для WebKit */
}

/* Примеры использования */
.normal-adjustment {
  color-adjust: economy;            /* Позволить корректировку цветов */
  color: #ff6b6b;
  background-color: #4ecdc4;
}

.exact-colors {
  color-adjust: exact;              /* Запретить корректировку цветов */
  color: #ff6b6b;
  background-color: #4ecdc4;
}
```

## CSS Color Adjust Module Level 2

Level 2 добавляет расширенные возможности для корректировки цветов и взаимодействия с системными настройками.

### Взаимодействие с prefers-color-scheme

```css
.color-scheme-adjustment {
  /* Основные цвета для светлой темы */
  color: #333;
  background-color: #ffffff;
  border-color: #ddd;
}

/* Темная тема */
@media (prefers-color-scheme: dark) {
  .color-scheme-adjustment {
    color: #e0e0e0;
    background-color: #1a1a1a;
    border-color: #444;
  }
}

/* Пример компонента с автоматической сменой темы */
.theme-aware-component {
  background-color: #ffffff;
  color: #333;
  border: 1px solid #ddd;
  padding: 1rem;
  border-radius: 8px;
  transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .theme-aware-component {
    background-color: #2d2d2d;
    color: #f0f0f0;
    border-color: #555;
  }
}
```

### Взаимодействие с prefers-contrast

```css
.contrast-adjustment {
  /* Нормальный контраст */
  color: #333;
  background-color: #fff;
  border: 1px solid #ccc;
}

@media (prefers-contrast: high) {
  .contrast-adjustment {
    /* Высокий контраст */
    color: #000;
    background-color: #fff;
    border: 2px solid #000;
  }
}

@media (prefers-contrast: low) {
  .contrast-adjustment {
    /* Низкий контраст */
    color: #666;
    background-color: #f5f5f5;
    border: 1px solid #ddd;
  }
}
```

### Взаимодействие с forced-colors

```css
.forced-colors-adjustment {
  /* Цвета по умолчанию */
  color: #333;
  background-color: #f0f0f0;
  border: 1px solid #ccc;
}

@media (forced-colors: active) {
  .forced-colors-adjustment {
    /* Сброс до системных цветов в режиме принудительных цветов */
    color: ButtonText;              /* Системный цвет текста кнопки */
    background-color: ButtonFace;   /* Системный цвет фона кнопки */
    border: 1px solid ButtonBorder; /* Системный цвет границы кнопки */
    
    /* В режиме принудительных цветов пользовательские цвета игнорируются */
    /* и используются системные цвета для лучшей доступности */
  }
}

/* Компонент, адаптирующийся к принудительным цветам */
.accessible-button {
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  text-align: center;
  transition: all 0.2s ease;
}

@media (forced-colors: active) {
  .accessible-button {
    /* Использование системных цветов */
    background-color: ButtonFace;
    color: ButtonText;
    border: 2px solid ButtonText;
  }
  
  .accessible-button:hover {
    background-color: Highlight;
    color: HighlightText;
  }
}
```

## Практические применения

### Темная тема с плавным переходом

```css
.smooth-theme-transition {
  /* Установка переходов для плавной смены темы */
  background-color: #ffffff;
  color: #333;
  border-color: #ddd;
  transition: 
    background-color 0.3s ease,
    color 0.3s ease,
    border-color 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .smooth-theme-transition {
    background-color: #1a1a1a;
    color: #f0f0f0;
    border-color: #444;
  }
}

/* Компонент с кастомными свойствами для темизации */
.themed-component {
  /* Определение переменных для светлой темы */
  --bg-color: #ffffff;
  --text-color: #333333;
  --border-color: #dddddd;
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  
  background-color: var(--bg-color);
  color: var(--text-color);
  border: 1px solid var(--border-color);
  padding: 1.5rem;
  border-radius: 8px;
  transition: all 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .themed-component {
    /* Переопределение переменных для темной темы */
    --bg-color: #1a1a1a;
    --text-color: #f0f0f0;
    --border-color: #444444;
    --primary-color: #007bff;
    --secondary-color: #6c757d;
  }
}
```

### Высококонтрастная тема

```css
.high-contrast-theme {
  /* Основные цвета */
  color: #000;
  background-color: #fff;
  border: 2px solid #000;
  
  /* Увеличенные размеры для лучшей видимости */
  font-size: 1.1em;
  line-height: 1.6;
  
  /* Улучшенная визуальная иерархия */
  box-shadow: 0 0 0 2px #000;
}

@media (prefers-contrast: high) {
  .high-contrast-theme {
    /* Усиленные цвета для высокого контраста */
    color: #000;
    background-color: #fff;
    border: 3px solid #000;
    box-shadow: 0 0 0 3px #000;
  }
  
  /* Усиленные стили для интерактивных элементов */
  .high-contrast-theme .interactive {
    outline: 3px solid #000;
    outline-offset: 2px;
  }
}
```

### Адаптация к системным настройкам

```css
.system-adaptive {
  /* Базовые стили */
  padding: 1rem;
  border-radius: 6px;
  transition: all 0.3s ease;
}

/* Адаптация к темной теме */
@media (prefers-color-scheme: dark) {
  .system-adaptive {
    background-color: #2a2a2a;
    color: #ffffff;
    border: 1px solid #555555;
  }
}

/* Адаптация к высокому контрасту */
@media (prefers-contrast: high) {
  .system-adaptive {
    background-color: #ffffff;
    color: #000000;
    border: 2px solid #000000;
  }
}

/* Адаптация к принудительным цветам */
@media (forced-colors: active) {
  .system-adaptive {
    background-color: Canvas;
    color: CanvasText;
    border: 1px solid CanvasText;
  }
}

/* Адаптация к уменьшению интенсивности анимации */
@media (prefers-reduced-motion: reduce) {
  .system-adaptive {
    /* Уменьшение или отключение анимаций */
    transition: none;
    animation: none;
  }
}
```

### Комплексный подход к доступности

```css
.accessible-component {
  /* Определение всех цветовых переменных */
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --text-primary: #212529;
  --text-secondary: #6c757d;
  --border: #dee2e6;
  --primary: #0d6efd;
  --success: #198754;
  --danger: #dc3545;
  
  background-color: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border);
  padding: 1.25rem;
  border-radius: 0.5rem;
  transition: 
    background-color 0.3s ease,
    color 0.3s ease,
    border-color 0.3s ease;
}

/* Темная тема */
@media (prefers-color-scheme: dark) {
  .accessible-component {
    --bg-primary: #212529;
    --bg-secondary: #343a40;
    --text-primary: #f8f9fa;
    --text-secondary: #dee2e6;
    --border: #495057;
  }
}

/* Высокий контраст */
@media (prefers-contrast: high) {
  .accessible-component {
    --bg-primary: #ffffff;
    --text-primary: #000000;
    --border: #000000;
  }
}

/* Принудительные цвета */
@media (forced-colors: active) {
  .accessible-component {
    background-color: Canvas;
    color: CanvasText;
    border-color: CanvasText;
    forced-color-adjust: none;      /* Отключение автоматической корректировки */
  }
}

/* Уменьшение анимации */
@media (prefers-reduced-motion: reduce) {
  .accessible-component {
    transition: none;
  }
}
```

## Современные возможности

### CSS Custom Properties и темизация

```css
.dynamic-color-adjustment {
  /* Использование кастомных свойств для гибкой темизации */
  --surface-color: #ffffff;
  --on-surface-color: #000000;
  --primary-color: #6200ee;
  --primary-variant-color: #3700b3;
  --secondary-color: #03dac6;
  
  background-color: var(--surface-color);
  color: var(--on-surface-color);
  transition: all 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .dynamic-color-adjustment {
    --surface-color: #121212;
    --on-surface-color: #ffffff;
  }
}

@media (prefers-contrast: high) {
  .dynamic-color-adjustment {
    --surface-color: #ffffff;
    --on-surface-color: #000000;
    --primary-color: #000000;
    --secondary-color: #000000;
  }
}
```

### Адаптивные цветовые палитры

```css
.adaptive-palette {
  /* Создание адаптивной цветовой палитры */
  --hue: 210;
  --saturation: 100%;
  --lightness-normal: 50%;
  --lightness-dark: 30%;
  --lightness-light: 80%;
  
  /* Цвета, которые адаптируются к теме */
  --primary: hsl(var(--hue), var(--saturation), var(--lightness-normal));
  --primary-dark: hsl(var(--hue), var(--saturation), var(--lightness-dark));
  --primary-light: hsl(var(--hue), var(--saturation), var(--lightness-light));
  
  background-color: var(--primary-light);
  color: var(--primary-dark);
  border: 1px solid var(--primary);
}

@media (prefers-color-scheme: dark) {
  .adaptive-palette {
    /* Адаптация светлости для темной темы */
    --lightness-normal: 60%;
    --lightness-dark: 80%;
    --lightness-light: 30%;
  }
}
```

### Система цветов с автоматической коррекцией

```css
.color-system {
  /* Базовая цветовая система */
  --color-surface: #ffffff;
  --color-on-surface: #000000;
  --color-primary: #6200ee;
  --color-primary-variant: #3700b3;
  --color-secondary: #03dac6;
  --color-secondary-variant: #018786;
  --color-error: #b00020;
  
  background-color: var(--color-surface);
  color: var(--color-on-surface);
  transition: all 0.3s ease;
}

/* Темная тема с автоматической коррекцией */
@media (prefers-color-scheme: dark) {
  .color-system {
    --color-surface: #121212;
    --color-on-surface: #ffffff;
    --color-primary: #bb86fc;
    --color-primary-variant: #3700b3;
    --color-secondary: #03dac6;
    --color-secondary-variant: #03dac6;
    --color-error: #cf6679;
  }
}

/* Высокий контраст для темной темы */
@media (prefers-color-scheme: dark) and (prefers-contrast: high) {
  .color-system {
    --color-surface: #000000;
    --color-on-surface: #ffffff;
    --color-primary: #ffffff;
    --color-secondary: #ffffff;
  }
}
```

## Поддержка браузерами

- `color-adjust`: Хорошая поддержка в современных браузерах
- `prefers-color-scheme`: Хорошая поддержка в современных браузерах
- `prefers-contrast`: Поддержка в процессе внедрения
- `forced-colors`: Поддержка ведущими браузерами для доступности
- `prefers-reduced-motion`: Хорошая поддержка в современных браузерах

## Лучшие практики

### Создание доступных тем

```css
.accessible-theme {
  /* Использование контрастных цветов */
  color: #000;
  background-color: #fff;
  border: 1px solid #ccc;
  
  /* Обеспечение достаточного контраста */
  /* Минимум 4.5:1 для нормального текста, 3:1 для крупного текста */
}

/* Адаптация к пользовательским настройкам */
@media (prefers-color-scheme: dark) {
  .accessible-theme {
    /* Убедитесь, что контраст сохраняется в темной теме */
    color: #fff;
    background-color: #000;
    border: 1px solid #555;
  }
}

/* Проверка контраста в принудительных цветах */
@media (forced-colors: active) {
  .accessible-theme {
    /* Использование системных цветов для гарантии контраста */
    background-color: Canvas;
    color: CanvasText;
    border-color: CanvasText;
  }
}
```

### Постепенная деградация

```css
.progressive-enhancement {
  /* Базовые стили без зависимости от настроек */
  background-color: #ffffff;
  color: #000000;
  border: 1px solid #cccccc;
  
  /* Дополнительные стили для поддерживаемых браузеров */
  transition: background-color 0.3s ease;
}

/* Улучшения для браузеров с поддержкой медиа-запросов */
@media (prefers-color-scheme: dark) {
  .progressive-enhancement {
    background-color: #1a1a1a;
    color: #ffffff;
    border-color: #444444;
  }
}

/* Дополнительные улучшения для браузеров с расширенной поддержкой */
@media (prefers-color-scheme: dark) and (prefers-contrast: no-preference) {
  .progressive-enhancement {
    /* Специфичные стили для темной темы без высокого контраста */
    background-color: #2d2d2d;
  }
}
```

## Связанные темы

- [[CSS Color Module]] - модуль цветов
- [[CSS Media Queries]] - медиа-запросы
- [[CSS Accessibility]] - доступность

## Заключение

CSS Color Adjust Module предоставляет важные инструменты для создания доступных и адаптивных интерфейсов, которые учитывают предпочтения пользователей. Понимание и правильное использование этих возможностей позволяет создавать более инклюзивные веб-сайты, которые автоматически адаптируются к настройкам устройства и потребностям пользователей.