# Анимации - Переходы

CSS-переходы (transitions) - это способ плавного изменения значений CSS-свойств за определенный период времени. Это один из самых простых способов добавить интерактивность к веб-странице без использования JavaScript.

## Основы CSS-переходов

### transition-property

Свойство `transition-property` определяет, какие CSS-свойства будут анимироваться:

```css
.basic-transition {
  /* Переход для одного свойства */
  transition-property: background-color;
  
  /* Переход для нескольких свойств */
  transition-property: background-color, color, transform;
  
  /* Все изменяемые свойства */
  transition-property: all;
}
```

### transition-duration

Свойство `transition-duration` определяет, как долго длится переход:

```css
.duration-examples {
  transition-property: width;
  transition-duration: 0.3s;    /* 300 миллисекунд */
  
  /* Разная продолжительность для разных свойств */
  transition-property: width, height;
  transition-duration: 0.5s, 1s;
}
```

### transition-timing-function

Свойство `transition-timing-function` определяет, как вычисляются промежуточные значения перехода:

```css
.timing-examples {
  /* Предопределенные функции */
  transition-timing-function: ease;        /* По умолчанию: медленно-быстро-медленно */
  transition-timing-function: linear;      /* Равномерная скорость */
  transition-timing-function: ease-in;     /* Медленно в начале */
  transition-timing-function: ease-out;    /* Медленно в конце */
  transition-timing-function: ease-in-out; /* Медленно в начале и в конце */
  
  /* Кастомные кривые Безье */
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
  
  /* Ступенчатые функции */
  transition-timing-function: step-start;  /* Начальное значение на всю длительность */
  transition-timing-function: step-end;    /* Конечное значение на всю длительность */
  transition-timing-function: steps(4, end); /* 4 ступени, конечное значение */
}
```

### transition-delay

Свойство `transition-delay` определяет задержку перед началом перехода:

```css
.delay-examples {
  transition-property: opacity;
  transition-duration: 0.5s;
  transition-delay: 0.2s;     /* Задержка 200мс перед началом */
  
  /* Разные задержки для разных свойств */
  transition-property: opacity, transform;
  transition-duration: 0.5s, 0.3s;
  transition-delay: 0.2s, 0.5s;
}
```

## Сокращенное свойство transition

Свойство `transition` позволяет задать все параметры перехода в одной декларации:

```css
.transition-shorthand {
  /* property duration timing-function delay */
  transition: background-color 0.3s ease 0s;
  
  /* Несколько переходов */
  transition: 
    background-color 0.3s ease,
    color 0.5s linear 0.1s,
    transform 0.2s ease-in-out;
  
  /* Переход для всех свойств */
  transition: all 0.3s ease;
}
```

## Практические примеры

### Пример 1: Анимированные кнопки

```css
.animated-button {
  padding: 12px 24px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.animated-button:hover {
  background-color: #0056b3;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 123, 255, 0.3);
}

.animated-button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 123, 255, 0.3);
}
```

### Пример 2: Плавное изменение размера

```css
.resizable-box {
  width: 100px;
  height: 100px;
  background-color: #3498db;
  transition: width 0.5s ease, height 0.5s ease 0.1s;
}

.resizable-box:hover {
  width: 150px;
  height: 150px;
}
```

### Пример 3: Анимация наведения на карточку

```css
.card {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.card-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.card:hover .card-image {
  transform: scale(1.05);
}
```

### Пример 4: Анимация изменения прозрачности

```css
.fade-element {
  opacity: 1;
  transition: opacity 0.3s ease;
}

.fade-element.fade-out {
  opacity: 0;
}

.fade-element.fade-in {
  opacity: 1;
}

/* Анимация появления элемента */
.appear-element {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.4s ease, transform 0.4s ease;
}

.appear-element.visible {
  opacity: 1;
  transform: translateY(0);
}
```

### Пример 5: Анимация боковой панели

```css
.sidebar {
  position: fixed;
  top: 0;
  left: -300px;  /* Скрыта за левой границей */
  width: 300px;
  height: 100vh;
  background-color: #34495e;
  transition: left 0.3s ease;
  z-index: 1000;
}

.sidebar.open {
  left: 0;  /* Появляется из левой границы */
}

.sidebar-toggle {
  position: fixed;
  top: 20px;
  left: 20px;
  z-index: 1001;
  padding: 10px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.sidebar-toggle:hover {
  background-color: #2980b9;
}
```

## Современные возможности

### CSS-переменные в переходах

```css
:root {
  --transition-speed: 0.3s;
  --transition-easing: ease;
  --primary-color: #3498db;
  --secondary-color: #e74c3c;
}

.modern-transition {
  background-color: var(--primary-color);
  transition: background-color var(--transition-speed) var(--transition-easing);
}

.modern-transition:hover {
  background-color: var(--secondary-color);
}
```

### Адаптивные переходы

```css
.adaptive-transition {
  /* Адаптивная длительность перехода */
  transition-duration: max(0.2s, 200ms);
  
  /* Адаптивная функция тайминга */
  transition-timing-function: 
    cubic-bezier(0.4, 0, 0.2, 1); /* Material Design easing */
}

@media (prefers-reduced-motion: reduce) {
  .adaptive-transition {
    transition: none;
  }
}
```

## Анимируемые свойства

Не все CSS-свойства могут быть анимированы. Вот список часто используемых анимируемых свойств:

```css
.animatable-properties {
  /* Цвета */
  color: #333;
  background-color: #fff;
  border-color: #ddd;
  
  /* Размеры */
  width: 100px;
  height: 100px;
  max-width: 200px;
  padding: 10px;
  margin: 5px;
  
  /* Трансформации */
  transform: translateX(10px) rotate(45deg) scale(1.2);
  
  /* Прозрачность */
  opacity: 0.8;
  
  /* Тени */
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
  
  /* Границы */
  border-width: 2px;
  border-radius: 5px;
  
  /* Позиционирование */
  top: 10px;
  left: 20px;
}
```

## Лучшие практики

1. **Используйте hardware-accelerated свойства** - transform и opacity для лучшей производительности
2. **Избегайте анимации layout-свойств** - width, height, margin, padding и т.д.
3. **Учитывайте пользовательские предпочтения** - уважайте `prefers-reduced-motion`
4. **Ограничивайте длительность переходов** - обычно от 100ms до 500ms
5. **Используйте подходящие timing функции** - ease-out для появления, ease-in для исчезновения
6. **Тестируйте на разных устройствах** - особенно на мобильных

## Распространенные ошибки

### 1. Анимация неанимируемых свойств

```css
/* Плохо: анимация display */
.bad-example {
  transition: display 0.3s ease;  /* display не анимируется */
}

/* Хорошо: анимация opacity и visibility */
.good-example {
  opacity: 1;
  visibility: visible;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.good-example.hidden {
  opacity: 0;
  visibility: hidden;
}
```

### 2. Игнорирование prefers-reduced-motion

```css
/* Плохо: без учета предпочтений пользователя */
.animated-element {
  transition: all 0.3s ease;
}

/* Хорошо: с учетом предпочтений */
.animated-element {
  transition: all 0.3s ease;
}

@media (prefers-reduced-motion: reduce) {
  .animated-element {
    transition: none;
  }
}
```

## Производительность

Для оптимальной производительности анимаций:

```css
.optimized-transition {
  /* Используйте transform вместо изменений позиции */
  transition: transform 0.3s ease, opacity 0.3s ease;
  
  /* Включите hardware acceleration при необходимости */
  will-change: transform;
}

.optimized-transition:hover {
  transform: translateZ(0); /* Активирует GPU ускорение */
}
```

## Заключение

CSS-переходы - мощный инструмент для создания плавных и приятных пользовательских интерфейсов. Они позволяют добавить интерактивность к веб-страницам без сложного JavaScript кода. Понимание принципов работы переходов и соблюдение лучших практик позволяет создавать эффективные и приятные в использовании интерфейсы.

#programming #css #transitions #animations #web-development #frontend