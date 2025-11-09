# Sass

Этот файл описывает CSS-препроцессор Sass.

Sass (Syntactically Awesome Style Sheets) — один из самых популярных CSS-препроцессоров, который добавляет мощные возможности к стандартному CSS. Он имеет два синтаксиса: SCSS (Sassy CSS), который является надмножеством CSS, и оригинальный отступный синтаксис Sass.

## Основные возможности:

- Переменные: Для хранения значений, таких как цвета и размеры
- Вложенность: Возможность вкладывать CSS-правила
- Миксины: Переиспользуемые блоки кода с возможностью параметров
- Наследование: Возможность наследовать стили от других селекторов
- Функции: Возможность создания функций для вычислений
- Условия и циклы: Условные операторы и циклы для динамической генерации CSS

## Пример:

```scss
$primary-color: #007bff;

.button {
  background-color: $primary-color;
  padding: 10px 20px;
  
  &.large {
    padding: 15px 30px;
  }
  
  @mixin hover-state($color) {
    &:hover {
      background-color: darken($color, 10%);
    }
  }
  
  @include hover-state($primary-color);
}
```

## См. также:
- [[preprocessors]]
- [[preprocessors/sass]]
- [[css-preprocessors]]

#sass #scss #preprocessors #css #web-development