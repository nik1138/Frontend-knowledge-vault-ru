# CSS Custom Properties

Этот файл описывает кастомные свойства CSS (CSS-переменные).

CSS Custom Properties, также известные как CSS-переменные, позволяют определять переиспользуемые значения в CSS. Они помогают создавать более поддерживаемый и гибкий CSS-код, позволяя легко изменять значения в одном месте, которые затем применяются ко всему проекту.

## Определение и использование:

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size: 16px;
}

.element {
  color: var(--primary-color);
  font-size: var(--font-size);
}
```

## Динамическое изменение:

CSS-переменные можно изменять через JavaScript, что позволяет создавать динамические темы и интерактивные эффекты.

## См. также:
- [[custom-properties]]
- [[custom-properties]]
- [[variables-comprehensive]]

#css #custom-properties #variables #web-development