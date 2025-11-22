---
aliases: [CSS Masking Specification, CSS Mask Properties, CSS Clipping and Masking, CSS Mask Functions]
tags: [css, masking, clipping, specification, css-masking]
---

# CSS Masking Module: Маскировка и вырезание областей - Полное руководство

## Введение

CSS Masking Module определяет способы частичного или полного скрытия элементов, используя маски и альфа-маскирование. Модуль включает свойства для создания сложных визуальных эффектов, позволяя вырезать части элементов или изменять их прозрачность на основе изображений или градиентов.

## CSS Masking Module Level 1

Level 1 определяет основные свойства маскировки.

### Основные свойства маскировки

```css
.masking-examples {
  /* Основные свойства маски */
  mask-image: none;                 /* Без маски */
  mask-image: url(mask.png);        /* Изображение как маска */
  mask-image: linear-gradient(to bottom, black, transparent);
  mask-image: radial-gradient(circle, black 50%, transparent 50%);
  
  /* Повторение маски */
  mask-repeat: repeat;              /* Повторять маску */
  mask-repeat: repeat-x;            /* Повторять по горизонтали */
  mask-repeat: repeat-y;            /* Повторять по вертикали */
  mask-repeat: no-repeat;           /* Не повторять */
  
  /* Позиционирование маски */
  mask-position: center;            /* Позиция маски */
  mask-position: 10px 20px;
  mask-position: 25% 75%;
  
  /* Размер маски */
  mask-size: auto;                  /* Автоматический размер */
  mask-size: 100px 50px;           /* Фиксированный размер */
  mask-size: 50% 50%;              /* Процентный размер */
  mask-size: cover;                 /* Покрыть весь элемент */
  mask-size: contain;               /* Вместить в элемент */
  
  /* Краткая запись маски */
  mask: url(mask.png) center/cover repeat;
  mask: linear-gradient(to right, black, transparent) no-repeat;
}
```

### Свойства маскировки

```css
.mask-properties {
  /* Режим маскировки */
  mask-mode: match-source;          /* Соответствовать источнику */
  mask-mode: alpha;                 /* Использовать альфа-канал */
  mask-mode: luminance;             /* Использовать яркость */
  
  /* Клип-контент маски */
  mask-clip: border-box;            /* Обрезка по границе */
  mask-clip: padding-box;           /* Обрезка по внутреннему полю */
  mask-clip: content-box;           /* Обрезка по содержимому */
  mask-clip: text;                  /* Обрезка по тексту */
  
  /* Происхождение маски */
  mask-origin: border-box;          /* Происхождение от границы */
  mask-origin: padding-box;         /* Происхождение от внутреннего поля */
  mask-origin: content-box;         /* Происхождение от содержимого */
  
  /* Композиция масок */
  mask-composite: add;              /* Сложение масок */
  mask-composite: subtract;         /* Вычитание масок */
  mask-composite: intersect;        /* Пересечение масок */
  mask-composite: exclude;          /* Исключение масок */
}
```

### Свойства clip-path

```css
.clip-path-examples {
  /* Основные фигуры */
  clip-path: inset(10px);           /* Вписанный прямоугольник */
  clip-path: circle(50% at center); /* Круг */
  clip-path: ellipse(50% 30% at center); /* Эллипс */
  clip-path: polygon(0 0, 100% 0, 50% 100%); /* Полигон */
  
  /* Использование URL для SVG фигур */
  clip-path: url(#myClipPath);
  
  /* Использование path() для сложных фигур */
  clip-path: path('M 0,0 L 100,0 L 100,50 L 50,100 L 0,50 Z');
}
```

## CSS Masking Module Level 2

Level 2 добавляет дополнительные возможности и улучшения.

### Расширенные возможности

```css
.enhanced-masking {
  /* Множественные маски */
  mask-image: 
    url(mask1.png),
    url(mask2.png),
    linear-gradient(45deg, black, transparent);
  
  /* Множественные свойства масок */
  mask-repeat: no-repeat, repeat, no-repeat;
  mask-position: center, 10px 10px, 20px 20px;
  mask-size: contain, 50px 50px, auto;
  
  /* Комбинирование масок */
  mask-composite: add, intersect;
}
```

### Свойства маскировки для текста

```css
.text-masking {
  /* Маскировка текста */
  mask: url(text-mask.svg);
  -webkit-mask: url(text-mask.svg); /* Для браузеров WebKit */
  
  /* Анимация текстовой маски */
  animation: maskAnimation 2s infinite alternate;
}

@keyframes maskAnimation {
  0% { mask-size: 100% 100%; }
  100% { mask-size: 200% 200%; }
}
```

## Практические применения

### Креативные формы с помощью масок

```css
.masked-element {
  width: 300px;
  height: 200px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  
  /* Круглая маска */
  mask-image: radial-gradient(circle at center, black 70%, transparent 70%);
  -webkit-mask-image: radial-gradient(circle at center, black 70%, transparent 70%);
}

/* Анимированная маска */
.animated-mask {
  width: 300px;
  height: 200px;
  background: linear-gradient(90deg, #6a11cb 0%, #2575fc 100%);
  mask-image: url('mask-pattern.svg');
  -webkit-mask-image: url('mask-pattern.svg');
  mask-size: 200px 200px;
  -webkit-mask-size: 200px 200px;
  animation: maskMove 4s linear infinite;
}

@keyframes maskMove {
  0% { 
    mask-position: 0 0; 
    -webkit-mask-position: 0 0; 
  }
  100% { 
    mask-position: 200px 0; 
    -webkit-mask-position: 200px 0; 
  }
}
```

### Портретная маска для изображений

```css
.portrait-mask {
  width: 200px;
  height: 300px;
  background: url('portrait.jpg') center/cover;
  
  /* Маска в виде силуэта человека */
  mask-image: url('silhouette.svg');
  -webkit-mask-image: url('silhouette.svg');
  mask-size: contain;
  -webkit-mask-size: contain;
  mask-repeat: no-repeat;
  -webkit-mask-repeat: no-repeat;
  mask-position: center;
  -webkit-mask-position: center;
}
```

### Анимированные эффекты маскировки

```css
.mask-animation {
  width: 400px;
  height: 200px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  
  /* Маска для создания эффекта "открытия" */
  mask-image: linear-gradient(90deg, transparent 0%, black 50%, transparent 100%);
  -webkit-mask-image: linear-gradient(90deg, transparent 0%, black 50%, transparent 100%);
  animation: maskReveal 3s infinite alternate;
}

@keyframes maskReveal {
  0% { 
    mask-position: -100% 0; 
    -webkit-mask-position: -100% 0; 
  }
  100% { 
    mask-position: 100% 0; 
    -webkit-mask-position: 100% 0; 
  }
}
```

### Комбинирование масок

```css
.combined-masks {
  width: 300px;
  height: 300px;
  background: 
    linear-gradient(45deg, #ff9a9e, #fecfef),
    linear-gradient(135deg, #a8edea, #fed6e3);
  
  /* Комбинирование нескольких масок */
  mask-image: 
    radial-gradient(circle at 25% 25%, black 50%, transparent 50%),
    radial-gradient(circle at 75% 75%, black 50%, transparent 50%);
  -webkit-mask-image: 
    radial-gradient(circle at 25% 25%, black 50%, transparent 50%),
    radial-gradient(circle at 75% 75%, black 50%, transparent 50%);
  
  mask-composite: intersect;
  -webkit-mask-composite: source-in;
}
```

### Текстовая маскировка

```css
.text-mask {
  font-size: 4rem;
  font-weight: bold;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  
  /* Маска для текста */
  mask-image: url('text-pattern.svg');
  -webkit-mask-image: url('text-pattern.svg');
  mask-size: 200px 200px;
  -webkit-mask-size: 200px 200px;
  mask-repeat: repeat;
  -webkit-mask-repeat: repeat;
}
```

### Интерактивные маски

```css
.interactive-mask {
  width: 300px;
  height: 200px;
  background: linear-gradient(45deg, #6a11cb 0%, #2575fc 100%);
  
  /* Начальная маска */
  mask-image: radial-gradient(circle at center, transparent 30%, black 30%);
  -webkit-mask-image: radial-gradient(circle at center, transparent 30%, black 30%);
  
  /* Плавное изменение маски при наведении */
  transition: mask-size 0.5s ease, -webkit-mask-size 0.5s ease;
}

.interactive-mask:hover {
  mask-image: radial-gradient(circle at center, transparent 70%, black 70%);
  -webkit-mask-image: radial-gradient(circle at center, transparent 70%, black 70%);
}
```

## Современные возможности

### CSS Houdini и маскировка

```css
.houdini-mask {
  /* Использование Paint API для динамических масок */
  mask-image: paint(maskPainter);
}
```

### Маскировка с использованием CSS Custom Properties

```css
.dynamic-mask {
  --mask-size: 50px 50px;
  --mask-position: center;
  
  mask-image: url('pattern.svg');
  -webkit-mask-image: url('pattern.svg');
  mask-size: var(--mask-size);
  -webkit-mask-size: var(--mask-size);
  mask-position: var(--mask-position);
  -webkit-mask-position: var(--mask-position);
  
  transition: --mask-size 0.3s ease;
}
```

## Поддержка браузерами

- `clip-path`: Хорошая поддержка в современных браузерах (с префиксами для старых WebKit браузеров)
- `mask-image` и связанные свойства: Поддержка в процессе внедрения, особенно в WebKit браузерах
- `mask-composite`: Ограниченная поддержка, лучше использовать `-webkit-mask-composite`
- SVG маски: Хорошая поддержка в большинстве браузеров

## Связанные темы

- [[CSS Shapes Module]] - фигуры и обтекание
- [[CSS Filters Module]] - фильтры изображений
- [[CSS Blend Modes]] - режимы наложения

## Заключение

CSS Masking Module предоставляет мощные инструменты для создания сложных визуальных эффектов и нестандартных форм элементов. Понимание свойств маскировки позволяет создавать креативные и интерактивные интерфейсы с уникальными визуальными эффектами.