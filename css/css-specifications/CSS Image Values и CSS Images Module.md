---
aliases: [CSS Images Specification, CSS Image Values, CSS Image Functions, CSS Images Module]
tags: [css, images, gradients, specification, css-images]
---

# CSS Image Values и CSS Images Module: Градиенты, функции изображений - Полное руководство

## Введение

CSS Image Values и CSS Images Module определяют способы работы с изображениями в CSS, включая градиенты, функции изображений и другие изображения как значения свойств. Модуль предоставляет расширенные возможности для создания визуальных эффектов без использования внешних изображений.

## CSS Images Module Level 3

Level 3 вводит основные функции для работы с изображениями.

### Функция image()

Функция `image()` позволяет указать несколько альтернативных изображений для использования в зависимости от возможностей браузера или условий отображения.

```css
.image-element {
  /* Базовое использование */
  background-image: image("image1.png", "image2.jpg");
  
  /* С указанием цвета резерва */
  background-image: image("image1.png", "image2.jpg", #ff0000);
  
  /* С указанием направления для градиента */
  background-image: image("fallback.png", linear-gradient(to right, red, blue));
  
  /* Использование с различными форматами */
  background-image: image(
    "image.avif", 
    "image.webp", 
    "image.jpg"
  );
}
```

### Градиенты

CSS поддерживает различные типы градиентов:

#### Линейные градиенты

```css
.linear-gradient-examples {
  /* Базовый линейный градиент */
  background: linear-gradient(to right, red, blue);
  
  /* Угол в градусах */
  background: linear-gradient(45deg, red, yellow, blue);
  
  /* Несколько цветовых стопов */
  background: linear-gradient(
    to bottom,
    red 0%,
    yellow 25%,
    green 50%,
    blue 75%,
    purple 100%
  );
  
  /* Повторяющийся линейный градиент */
  background: repeating-linear-gradient(
    45deg,
    red,
    red 10px,
    blue 10px,
    blue 20px
  );
}
```

#### Радиальные градиенты

```css
.radial-gradient-examples {
  /* Базовый радиальный градиент */
  background: radial-gradient(circle, red, blue);
  
  /* Эллиптический градиент */
  background: radial-gradient(ellipse, red, blue);
  
  /* С указанием размера и позиции */
  background: radial-gradient(
    circle at center,
    red 0%,
    yellow 50%,
    blue 100%
  );
  
  /* Повторяющийся радиальный градиент */
  background: repeating-radial-gradient(
    circle at center,
    red,
    red 10px,
    blue 10px,
    blue 20px
  );
}
```

#### Конические градиенты

```css
.conic-gradient-examples {
  /* Базовый конический градиент */
  background: conic-gradient(red, blue, red);
  
  /* С указанием центра */
  background: conic-gradient(at 50% 25%, red, blue, red);
  
  /* Цветовой спектр */
  background: conic-gradient(
    red, orange, yellow, green, blue, indigo, violet, red
  );
  
  /* Повторяющийся конический градиент */
  background: repeating-conic-gradient(
    red 0deg 15deg,
    blue 15deg 30deg
  );
}
```

### Функция cross-fade()

Функция `cross-fade()` позволяет создавать плавные переходы между двумя изображениями.

```css
.crossfade-element {
  /* Простое смешивание 50/50 */
  background-image: cross-fade(url(image1.jpg), url(image2.jpg));
  
  /* С указанием соотношения */
  background-image: cross-fade(url(image1.jpg) 70%, url(image2.jpg) 30%);
  
  /* С fallback изображением */
  background-image: cross-fade(url(image1.jpg), url(image2.jpg), #ff0000);
}
```

## CSS Images Module Level 4

Level 4 расширяет возможности работы с изображениями.

### Функция image-set()

Функция `image-set()` позволяет указать несколько версий одного изображения для разных плотностей пикселей и разрешений.

```css
.image-set-element {
  /* Базовое использование */
  background-image: image-set(
    "icon.png" 1x,
    "icon@2x.png" 2x,
    "icon@3x.png" 3x
  );
  
  /* С различными разрешениями */
  background-image: image-set(
    "bg-low.jpg" 1dppx,
    "bg-high.jpg" 2dppx
  );
  
  /* Внутри image() для fallback */
  background-image: image(
    image-set(
      "icon.svg" 16w,
      "icon@2x.png" 32w
    ),
    "fallback.png"
  );
}
```

### Функция element()

Функция `element()` позволяет использовать содержимое другого элемента как изображение.

```css
.element-as-image {
  /* Использование содержимого другого элемента */
  background-image: element(#other-element);
  
  /* С маской */
  mask-image: element(#other-element);
}
```

### Функция image()

Улучшенная функция `image()` с дополнительными возможностями:

```css
.advanced-image-function {
  /* С указанием ориентации */
  background-image: image("image.jpg", auto 90deg);  /* Поворот на 90 градусов */
  
  /* С указанием размера */
  background-image: image("image.jpg", auto 200px 100px);  /* Размер 200x100 */
}
```

## Практические применения

### Креативные градиенты

```css
/* Градиентный текст */
.gradient-text {
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4, #ffeaa7);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 8s ease infinite;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* Анимированные градиенты */
.animated-gradient {
  background: linear-gradient(0.25turn, #3f87a6, #ebf8e1, #f69d3c);
  background-size: 400% 400%;
  animation: gradientAnimation 10s ease infinite;
}

@keyframes gradientAnimation {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* Геометрические узоры с градиентами */
.pattern-background {
  background: 
    repeating-linear-gradient(
      45deg,
      #606dbc,
      #606dbc 10px,
      #465298 10px,
      #465298 20px
    );
}
```

### Адаптивные изображения

```css
.adaptive-image {
  /* Использование разных изображений для разных плотностей */
  background-image: image-set(
    "bg-small.jpg" 1x,
    "bg-small@2x.jpg" 2x
  );
  
  /* С fallback для старых браузеров */
  background-image: image(
    image-set(
      "bg.avif" 1x,
      "bg.webp" 1x,
      "bg.jpg" 1x
    ),
    url("bg-fallback.jpg")
  );
}

/* Адаптивный фон с градиентом */
.adaptive-gradient-bg {
  background: 
    linear-gradient(
      rgba(0, 0, 0, 0.5),
      rgba(0, 0, 0, 0.5)
    ),
    image-set(
      "hero-small.jpg" 1x,
      "hero-large.jpg" 2x
    );
  background-size: cover;
  background-position: center;
}
```

### Эффекты и визуальные иллюзии

```css
/* Эффект стеклянной морфологии */
.glass-effect {
  background: 
    linear-gradient(
      135deg, 
      rgba(255, 255, 255, 0.1) 0%, 
      rgba(255, 255, 255, 0) 100%
    ),
    linear-gradient(
      45deg, 
      rgba(255, 255, 255, 0.05) 0%, 
      rgba(255, 255, 255, 0) 100%
    );
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.18);
}

/* Радужные эффекты */
.rainbow-border {
  position: relative;
}

.rainbow-border::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: conic-gradient(
    #ff0000, #ff8000, #ffff00, 
    #00ff00, #0080ff, #0000ff, 
    #8000ff, #ff0000
  );
  border-radius: inherit;
  z-index: -1;
  animation: rotate 3s linear infinite;
}

@keyframes rotate {
  100% {
    transform: rotate(360deg);
  }
}
```

### Оптимизация производительности

```css
.performance-optimized {
  /* Использование простых градиентов вместо изображений */
  background: 
    linear-gradient(rgba(0,0,0,0.1), rgba(0,0,0,0.1)),
    url(texture.png);
  
  /* Избегать сложных градиентов в анимациях */
  background-size: 200% 200%;
  transition: background-position 0.3s ease;
}

.performance-optimized:hover {
  background-position: 100% 100%;
}

/* Использование image-set для оптимизации загрузки */
.optimized-image {
  background-image: image-set(
    "image-avif.avif" type("image/avif"),
    "image-webp.webp" type("image/webp"),
    "image-jpg.jpg" type("image/jpeg")
  );
}
```

## Современные возможности

### CSS Color Mix и градиенты

```css
.modern-gradient {
  /* Использование color-mix в градиентах */
  background: linear-gradient(
    to right,
    color-mix(in srgb, red 70%, blue 30%),
    color-mix(in srgb, green 50%, yellow 50%)
  );
}
```

### Генерация узоров

```css
.pattern-generator {
  /* Сложные узоры с несколькими градиентами */
  background: 
    repeating-linear-gradient(
      0deg,
      transparent,
      transparent 10px,
      rgba(0,0,0,0.1) 10px,
      rgba(0,0,0,0.1) 20px
    ),
    repeating-linear-gradient(
      90deg,
      transparent,
      transparent 10px,
      rgba(0,0,0,0.1) 10px,
      rgba(0,0,0,0.1) 20px
    );
}
```

## Поддержка браузерами

- `linear-gradient()`, `radial-gradient()`: Поддерживается во всех современных браузерах
- `conic-gradient()`: Хорошая поддержка в современных браузерах
- `repeating-*` градиенты: Поддержка в современных браузерах
- `image-set()`: Хорошая поддержка, особенно в Chrome, Safari и Edge
- `image()`: Ограниченная поддержка, лучше использовать с fallback

## Связанные темы

- [[CSS Color Module]] - модуль цветов
- [[CSS Transform Module]] - трансформации
- [[CSS Filter Effects Module]] - фильтры изображений

## Заключение

CSS Images Module предоставляет мощные инструменты для создания визуальных эффектов без использования внешних изображений. Понимание различных функций изображений и градиентов позволяет создавать адаптивные, производительные и визуально привлекательные интерфейсы.