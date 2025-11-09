# Less

Этот файл описывает CSS-препроцессор Less.

Less (Leaner Style Sheets) — это CSS-препроцессор, похожий на SCSS, но с некоторыми отличиями в синтаксисе. Less расширяет возможности CSS, добавляя переменные, вложенность, миксины, операции и функции.

## Основные возможности:

- Переменные: Для хранения значений, таких как цвета и размеры
- Вложенность: Возможность вкладывать CSS-правила
- Миксины: Переиспользуемые блоки кода
- Операции: Возможность выполнять математические операции
- Функции: Встроенные функции для манипуляции цветами и другими значениями
- Пространства имен и простые наследования

## Пример:

```less
@primary-color: #007bff;

.button {
  background-color: @primary-color;
  padding: 10px 20px;
  
  &.large {
    padding: 15px 30px;
  }
  
  .hover-state() {
    &:hover {
      background-color: darken(@primary-color, 10%);
    }
  }
  
  .hover-state();
}
```

## См. также:
- [[preprocessors]]
- [[preprocessors/less]]
- [[css-preprocessors]]

#less #preprocessors #css #web-development