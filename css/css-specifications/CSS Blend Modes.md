---
aliases: [CSS Blend Modes Specification, CSS Compositing and Blending, CSS Blend Functions, CSS Blend Properties]
tags: [css, blend-modes, compositing, specification, css-blending]
---

# CSS Blend Modes: Режимы наложения - Полное руководство

## Введение

CSS Compositing and Blending Module определяет способы наложения и смешивания цветов между элементами и их фонами. Режимы наложения позволяют создавать сложные визуальные эффекты, аналогичные тем, что используются в графических редакторах, но в веб-среде.

## CSS Compositing and Blending Module Level 1

Level 1 определяет основные режимы наложения и свойства компоновки.

### Свойство background-blend-mode

Свойство `background-blend-mode` определяет, как смешиваются слои фона внутри одного элемента.

```css
.background-blend-examples {
  background: 
    linear-gradient(45deg, red, blue),
    url('texture.jpg'),
    radial-gradient(circle, yellow, green);
  
  /* Режимы наложения для слоев фона */
  background-blend-mode: normal;        /* Нормальный режим */
  background-blend-mode: multiply;      /* Умножение */
  background-blend-mode: screen;        /* Экранирование */
  background-blend-mode: overlay;       /* Перекрытие */
  background-blend-mode: darken;        /* Затемнение */
  background-blend-mode: lighten;       /* Осветление */
  background-blend-mode: color-dodge;   /* Цветное выносливание */
  background-blend-mode: color-burn;    /* Цветное сжигание */
  background-blend-mode: hard-light;    /* Жесткий свет */
  background-blend-mode: soft-light;    /* Мягкий свет */
  background-blend-mode: difference;    /* Разница */
  background-blend-mode: exclusion;     /* Исключение */
  background-blend-mode: hue;           /* Оттенок */
  background-blend-mode: saturation;    /* Насыщенность */
  background-blend-mode: color;         /* Цвет */
  background-blend-mode: luminosity;    /* Яркость */
  
  /* Несколько режимов для разных слоев */
  background-blend-mode: multiply, screen, overlay;
}
```

### Свойство mix-blend-mode

Свойство `mix-blend-mode` определяет, как элемент смешивается со своими родительскими элементами и фоном.

```css
.mix-blend-examples {
  /* Режимы смешивания элемента с фоном */
  mix-blend-mode: normal;           /* Нормальный режим */
  mix-blend-mode: multiply;         /* Умножение */
  mix-blend-mode: screen;           /* Экранирование */
  mix-blend-mode: overlay;          /* Перекрытие */
  mix-blend-mode: darken;           /* Затемнение */
  mix-blend-mode: lighten;          /* Осветление */
  mix-blend-mode: color-dodge;      /* Цветное выносливание */
  mix-blend-mode: color-burn;       /* Цветное сжигание */
  mix-blend-mode: hard-light;       /* Жесткий свет */
  mix-blend-mode: soft-light;       /* Мягкий свет */
  mix-blend-mode: difference;       /* Разница */
  mix-blend-mode: exclusion;        /* Исключение */
  mix-blend-mode: hue;              /* Оттенок */
  mix-blend-mode: saturation;       /* Насыщенность */
  mix-blend-mode: color;            /* Цвет */
  mix-blend-mode: luminosity;       /* Яркость */
}
```

### Свойство isolation

Свойство `isolation` создает новый контекст наложения, изолируя элемент от влияния родительских элементов.

```css
.isolation-examples {
  isolation: auto;                  /* Автоматическое определение контекста */
  isolation: isolate;               /* Создание изолированного контекста */
}
```

## Практические примеры режимов наложения

### background-blend-mode примеры

```css
.blend-container {
  width: 300px;
  height: 200px;
  background: 
    linear-gradient(135deg, #667eea 0%, #764ba2 100%),
    url('texture.jpg');
  background-size: cover, auto;
  background-blend-mode: overlay;   /* Смешивание градиента и текстуры */
}

/* Множественные слои фона с различными режимами */
.multiple-blends {
  width: 400px;
  height: 300px;
  background: 
    linear-gradient(45deg, rgba(255,0,0,0.5), transparent),
    linear-gradient(135deg, rgba(0,255,0,0.5), transparent),
    linear-gradient(225deg, rgba(0,0,255,0.5), transparent),
    url('background.jpg');
  background-blend-mode: multiply, screen, overlay;
  background-size: cover;
}

/* Текстурный эффект */
.textured-gradient {
  width: 300px;
  height: 200px;
  background: 
    linear-gradient(45deg, #ff6b6b, #4ecdc4),
    url('noise-texture.png');
  background-blend-mode: multiply;
  background-size: cover, 100px 100px;
}
```

### mix-blend-mode примеры

```css
.mix-blend-container {
  background: linear-gradient(45deg, #ff9a9e, #fecfef);
  padding: 20px;
  isolation: isolate;               /* Создание изолированного контекста */
}

.blended-element {
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, #a8edea, #fed6e3);
  mix-blend-mode: multiply;         /* Смешивание с фоном */
  border-radius: 50%;
}

/* Текст с режимом наложения */
.blended-text {
  font-size: 2rem;
  font-weight: bold;
  color: white;
  mix-blend-mode: difference;       /* Инверсия цветов относительно фона */
}
```

## Описание основных режимов наложения

### Нормальные режимы

```css
.normal-blends {
  /* Normal - без смешивания, обычное наложение */
  background-blend-mode: normal;
  /* Каждый слой просто накладывается поверх предыдущего */
}
```

### Темные режимы

```css
.darken-blends {
  /* Multiply - умножение цветов */
  background-blend-mode: multiply;
  /* Результат всегда темнее исходных цветов */
  
  /* Darken - берется темнейший пиксель */
  background-blend-mode: darken;
  /* Сравниваются цвета и выбирается темнейший */
  
  /* Color Burn - цветное сжигание */
  background-blend-mode: color-burn;
  /* Темнеет верхний цвет, усиливая контраст с нижним */
}
```

### Светлые режимы

```css
.lighten-blends {
  /* Screen - экранирование */
  background-blend-mode: screen;
  /* Результат всегда светлее исходных цветов */
  
  /* Lighten - берется светлейший пиксель */
  background-blend-mode: lighten;
  /* Сравниваются цвета и выбирается светлейший */
  
  /* Color Dodge - цветное выносливание */
  background-blend-mode: color-dodge;
  /* Осветляет нижний цвет в соответствии с верхним */
}
```

### Контрастные режимы

```css
.contrast-blends {
  /* Overlay - перекрытие */
  background-blend-mode: overlay;
  /* Комбинация multiply и screen, сохраняя контрасты */
  
  /* Soft Light - мягкий свет */
  background-blend-mode: soft-light;
  /* Мягкое освещение, как через матовое стекло */
  
  /* Hard Light - жесткий свет */
  background-blend-mode: hard-light;
  /* Жесткое освещение, как прямой свет */
}
```

### Инверсионные режимы

```css
.inversion-blends {
  /* Difference - разница */
  background-blend-mode: difference;
  /* Вычитает один цвет из другого */
  
  /* Exclusion - исключение */
  background-blend-mode: exclusion;
  /* Похоже на difference, но с меньшим контрастом */
}
```

### Цветовые режимы

```css
.color-blends {
  /* Hue - оттенок */
  background-blend-mode: hue;
  /* Использует оттенок верхнего слоя и насыщенность/яркость нижнего */
  
  /* Saturation - насыщенность */
  background-blend-mode: saturation;
  /* Использует насыщенность верхнего слоя и другие свойства нижнего */
  
  /* Color - цвет */
  background-blend-mode: color;
  /* Использует оттенок и насыщенность верхнего, яркость нижнего */
  
  /* Luminosity - яркость */
  background-blend-mode: luminosity;
  /* Использует яркость верхнего слоя и цвет нижнего */
}
```

## Практические применения

### Текстурные эффекты

```css
.texture-overlay {
  width: 400px;
  height: 300px;
  background: 
    linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)),
    url('background.jpg'),
    radial-gradient(circle, #ff9a9e, #fecfef);
  background-blend-mode: overlay, multiply;
  background-size: cover;
  background-position: center;
}

/* Эффект пленки */
.film-effect {
  background: 
    linear-gradient(rgba(0,0,0,0.1), rgba(0,0,0,0.1)),
    url('image.jpg'),
    repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(255,255,255,0.03) 2px,
      rgba(255,255,255,0.03) 4px
    );
  background-blend-mode: overlay, normal;
}
```

### Цветовые коррекции

```css
.color-correction {
  background: 
    linear-gradient(45deg, 
      rgba(255, 105, 180, 0.2), 
      rgba(0, 191, 255, 0.2)),
    url('original-image.jpg');
  background-blend-mode: overlay;
  background-size: cover;
}

/* Винтажный эффект */
.vintage-effect {
  background: 
    radial-gradient(ellipse at center, 
      rgba(0,0,0,0) 70%, 
      rgba(0,0,0,0.7) 100%),
    linear-gradient(to bottom, 
      rgba(255,220,140,0.3), 
      rgba(25,25,25,0.3)),
    url('photo.jpg');
  background-blend-mode: multiply, overlay;
  background-size: cover;
}
```

### Интерактивные эффекты

```css
.interactive-blend {
  width: 300px;
  height: 200px;
  background: 
    linear-gradient(45deg, #ff6b6b, #4ecdc4),
    url('texture.jpg');
  background-blend-mode: overlay;
  background-size: cover;
  transition: background-blend-mode 0.3s ease;
}

.interactive-blend:hover {
  background-blend-mode: multiply;
}

/* Анимированные смешивания */
.animated-blend {
  background: 
    linear-gradient(45deg, #ff9a9e, #fecfef),
    linear-gradient(135deg, #a8edea, #fed6e3);
  background-blend-mode: overlay;
  animation: blendAnimation 4s infinite alternate;
}

@keyframes blendAnimation {
  0% { background-blend-mode: overlay; }
  25% { background-blend-mode: multiply; }
  50% { background-blend-mode: screen; }
  75% { background-blend-mode: hard-light; }
  100% { background-blend-mode: soft-light; }
}
```

### Типографские эффекты

```css
.blended-text-container {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px;
  isolation: isolate;               /* Изолируем контекст */
}

.blended-text {
  font-size: 3rem;
  font-weight: bold;
  color: white;
  mix-blend-mode: difference;       /* Инвертирует цвета фона */
  text-align: center;
}
```

## Современные возможности

### CSS Custom Properties с режимами наложения

```css
.dynamic-blend {
  --blend-mode: overlay;
  
  background: 
    linear-gradient(45deg, #ff6b6b, #4ecdc4),
    url('texture.jpg');
  background-blend-mode: var(--blend-mode);
  background-size: cover;
  
  transition: background-blend-mode 0.3s ease;
}

.dynamic-blend[data-mode="multiply"] {
  --blend-mode: multiply;
}

.dynamic-blend[data-mode="screen"] {
  --blend-mode: screen;
}
```

### Адаптивные режимы наложения

```css
.responsive-blend {
  background: 
    linear-gradient(45deg, #6a11cb 0%, #2575fc 100%),
    url('pattern.svg');
  background-blend-mode: overlay;
  background-size: cover;
}

@media (prefers-contrast: high) {
  .responsive-blend {
    background-blend-mode: multiply; /* Более контрастный режим */
  }
}

@media (prefers-reduced-motion: reduce) {
  .responsive-blend {
    background-blend-mode: normal;   /* Отключение анимированных эффектов */
  }
}
```

## Производительность и оптимизация

```css
.optimized-blends {
  /* Использование GPU ускорения */
  transform: translateZ(0);
  
  /* Избегать чрезмерного использования сложных режимов на мобильных */
  background-blend-mode: multiply;   /* Простой, но эффективный режим */
  
  /* Оптимизация для производительности */
  will-change: background-blend-mode;
}

/* Упрощенные режимы для слабых устройств */
@media (prefers-reduced-data: reduce) {
  .optimized-blends {
    background-blend-mode: normal;   /* Отключение сложных эффектов */
  }
}
```

## Поддержка браузерами

- `background-blend-mode`: Хорошая поддержка в современных браузерах
- `mix-blend-mode`: Хорошая поддержка, с префиксами для старых WebKit браузеров
- `isolation`: Хорошая поддержка в современных браузерах
- Некоторые режимы наложения могут требовать префиксов в старых браузерах

## Связанные темы

- [[CSS Filters Module]] - фильтры изображений
- [[CSS Masking Module]] - маскировка элементов
- [[CSS Color Module]] - модуль цветов

## Заключение

CSS Blend Modes предоставляют мощные инструменты для создания сложных визуальных эффектов и цветовых взаимодействий. Понимание различных режимов наложения позволяет создавать профессионально выглядящие интерфейсы с богатыми визуальными эффектами.