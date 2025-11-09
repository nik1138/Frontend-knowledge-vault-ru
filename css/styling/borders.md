# Визуальное оформление - Границы

Границы в CSS позволяют добавлять визуальные рамки вокруг элементов, создавая структуру и иерархию в веб-интерфейсах. CSS предоставляет широкие возможности для настройки внешнего вида границ.

## Свойства границ

### border-width

Свойство `border-width` определяет толщину границы:

```css
.width-examples {
  border-width: 1px;              /* Одинаковая толщина для всех сторон */
  border-width: 2px 4px;          /* 2px для верх/низ, 4px для лев/прав */
  border-width: 1px 2px 3px 4px;  /* верх прав низ лев */
  
  /* Ключевые слова */
  border-width: thin;             /* Тонкая граница */
  border-width: medium;           /* Средняя граница (по умолчанию) */
  border-width: thick;            /* Толстая граница */
}
```

### border-style

Свойство `border-style` определяет стиль границы:

```css
.style-examples {
  border-style: none;         /* Без границы (по умолчанию) */
  border-style: solid;        /* Сплошная линия */
  border-style: dashed;       /* Пунктирная линия */
  border-style: dotted;       /* Точечная линия */
  border-style: double;       /* Двойная линия */
  border-style: groove;       /* Объемная граница (вдавленная) */
  border-style: ridge;        /* Объемная граница (выпуклая) */
  border-style: inset;        /* Внутренняя граница */
  border-style: outset;       /* Внешняя граница */
  
  /* Разные стили для разных сторон */
  border-style: solid dashed dotted double;
}
```

### border-color

Свойство `border-color` определяет цвет границы:

```css
.color-examples {
  border-color: #000000;              /* Черный */
  border-color: rgb(255, 0, 0);       /* Красный */
  border-color: hsl(120, 100%, 50%);  /* Зеленый */
  border-color: transparent;          /* Прозрачная граница */
  border-color: currentColor;         /* Текущий цвет текста */
  
  /* Разные цвета для разных сторон */
  border-color: red green blue yellow; /* верх прав низ лев */
}
```

## Индивидуальные свойства границ

### border-top, border-right, border-bottom, border-left

Эти свойства позволяют задавать границы для отдельных сторон:

```css
.specific-borders {
  border-top: 2px solid #333;         /* Верхняя граница */
  border-right: 1px dashed #666;      /* Правая граница */
  border-bottom: 3px double #999;     /* Нижняя граница */
  border-left: 2px dotted #ccc;       /* Левая граница */
}
```

### Комбинации свойств для отдельных сторон

```css
.combined-specific {
  border-top-width: 2px;
  border-top-style: solid;
  border-top-color: #333;
  
  border-right-width: 1px;
  border-right-style: dashed;
  border-right-color: #666;
  
  /* Или сокращенно */
  border-top: 2px solid #333;
  border-right: 1px dashed #666;
}
```

## Сокращенное свойство border

Свойство `border` позволяет задать стиль, ширину и цвет для всех сторон одновременно:

```css
.border-shorthand {
  border: 2px solid #333;             /* Все стороны одинаково */
  border: 1px dotted red;             /* Ширина, стиль, цвет */
  border: solid #000;                 /* Только стиль и цвет */
  border: 2px solid;                  /* Только ширина и стиль */
}
```

## border-radius

Свойство `border-radius` позволяет скруглять углы элемента:

```css
.border-radius-examples {
  border-radius: 5px;                 /* Одинаковое скругление для всех углов */
  border-radius: 10px 5px;            /* 10px для верх/низ, 5px для лев/прав */
  border-radius: 10px 5px 15px 20px;  /* лев-верх прав-верх прав-низ лев-низ */
  
  /* Индивидуальные углы */
  border-top-left-radius: 10px;
  border-top-right-radius: 15px;
  border-bottom-right-radius: 20px;
  border-bottom-left-radius: 25px;
  
  /* Эллиптическое скругление */
  border-radius: 10px / 20px;         /* Горизонтальный / вертикальный радиус */
  border-radius: 10px 20px / 30px 40px; /* Разные радиусы для разных осей */
  
  /* Круглые элементы */
  circle: 50%;                        /* Для квадратных элементов создает круг */
}
```

## border-image

Свойство `border-image` позволяет использовать изображение в качестве границы:

```css
.border-image-examples {
  /* border-image: source slice width outset repeat */
  border: 10px solid transparent;
  border-image: url('border-image.png') 30 round;
  
  /* Более подробное определение */
  border-image-source: url('frame.png');
  border-image-slice: 30%;
  border-image-width: 10px;
  border-image-outset: 0;
  border-image-repeat: round;
  
  /* Повторение изображения границы */
  border-image-repeat: stretch;   /* Растянуть (по умолчанию) */
  border-image-repeat: repeat;    /* Повторять */
  border-image-repeat: round;     /* Повторять с растяжением */
  border-image-repeat: space;     /* Повторять с пробелами */
}
```

## border-collapse

Свойство `border-collapse` используется для управления границами в таблицах:

```css
.table-borders {
  border-collapse: separate;  /* Отдельные границы ячеек (по умолчанию) */
  border-collapse: collapse;  /* Объединенные границы ячеек */
  
  /* Для отдельных границ */
  border-spacing: 5px;        /* Расстояние между границами ячеек */
}
```

## Практические примеры

### Пример 1: Карточка с границей

```css
.card {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
}

.card.highlighted {
  border: 2px solid #3182ce;
  box-shadow: 0 4px 6px rgba(49, 130, 206, 0.1);
}
```

### Пример 2: Кнопки с различными стилями границ

```css
.btn {
  padding: 0.5rem 1rem;
  border: 2px solid transparent;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
}

.btn-primary {
  border: 2px solid #3182ce;
  background-color: #3182ce;
  color: white;
}

.btn-outline {
  border: 2px solid #3182ce;
  background-color: transparent;
  color: #3182ce;
}

.btn-dashed {
  border: 2px dashed #cbd5e0;
  background-color: transparent;
  color: #4a5568;
}
```

### Пример 3: Круглые аватары

```css
.avatar {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 2px solid #e2e8f0;
  object-fit: cover;
}

.avatar-large {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: 3px solid #cbd5e0;
}
```

### Пример 4: Стилизованные поля ввода

```css
.input-field {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e2e8f0;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.2s ease;
}

.input-field:focus {
  outline: none;
  border-color: #3182ce;
  box-shadow: 0 0 0 3px rgba(49, 130, 206, 0.1);
}

.input-field.error {
  border-color: #e53e3e;
}

.input-field.success {
  border-color: #38a169;
}
```

### Пример 5: Декоративные элементы

```css
.quote-box {
  border-left: 4px solid #3182ce;
  padding-left: 1.5rem;
  margin: 1rem 0;
  font-style: italic;
  color: #4a5568;
}

.code-block {
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 1rem;
  background-color: #f7fafc;
  font-family: 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;
  overflow-x: auto;
}

.feature-box {
  border: 2px dashed #cbd5e0;
  border-radius: 8px;
  padding: 2rem;
  text-align: center;
  background-color: #f8f9fa;
}
```

## Современные возможности

### CSS-переменные для границ

```css
:root {
  --border-width: 1px;
  --border-style: solid;
  --border-color: #e2e8f0;
  --border-radius: 4px;
  --border-focus-color: #3182ce;
}

.modern-borders {
  border: var(--border-width) var(--border-style) var(--border-color);
  border-radius: var(--border-radius);
}

.modern-borders:focus {
  border-color: var(--border-focus-color);
}
```

### Адаптивные границы

```css
.responsive-border {
  border-width: max(1px, 0.1em);  /* Адаптируется к размеру шрифта */
  border-radius: clamp(4px, 1vw, 8px);  /* Адаптивное скругление */
}
```

## Лучшие практики

1. **Используйте семантические значения** - толщина границы должна соответствовать важности элемента
2. **Сохраняйте согласованность** - используйте одинаковые стили границ по всему проекту
3. **Обеспечьте доступность** - убедитесь, что границы не мешают восприятию контента
4. **Тестируйте на разных устройствах** - убедитесь, что границы выглядят хорошо на всех экранах
5. **Используйте transition для состояний** - плавные переходы между состояниями границы
6. **Оптимизируйте производительность** - избегайте сложных border-image там, где это не нужно

## Распространенные ошибки

### 1. Забытый border-box

```css
/* Плохо: границы увеличивают размер элемента */
.element {
  width: 200px;
  border: 10px solid #333;
  /* Фактическая ширина: 220px */
}

/* Хорошо: границы включаются в размер */
.element {
  width: 200px;
  border: 10px solid #333;
  box-sizing: border-box;
  /* Фактическая ширина: 200px */
}
```

### 2. Неправильное использование border-radius с border-image

```css
/* border-radius может повлиять на отображение border-image */
.border-image-rounded {
  border: 10px solid transparent;
  border-image: url('image.png') 30 round;
  border-radius: 10px;  /* Может исказить изображение границы */
}
```

## Заключение

Границы в CSS - это мощный инструмент для визуального оформления элементов. От простых линий до сложных изображений границ - CSS предоставляет гибкие возможности для создания привлекательных и функциональных интерфейсов. Правильное использование границ помогает создать четкую структуру и улучшить восприятие веб-страницы.

#programming #css #borders #styling #web-development #frontend