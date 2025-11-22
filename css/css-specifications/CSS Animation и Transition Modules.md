---
aliases: [CSS Animation Specification, CSS Transition Specification, CSS Animations and Transitions, CSS Motion]
tags: [css, animation, transition, specification, css-animations]
---

# CSS Animation и Transition Modules: Полное руководство

## Введение

CSS Animation и Transition Modules определяют способы создания анимированных эффектов в CSS. Transition позволяет плавно изменять значения свойств, а Animation позволяет создавать более сложные последовательности анимаций с помощью keyframes.

## CSS Transitions Module

Transitions позволяют плавно изменять значения CSS свойств от одного состояния к другому.

### Основные свойства переходов

```css
.transition-element {
  /* Основные свойства */
  transition-property: width;        /* Свойство для анимации */
  transition-duration: 0.3s;         /* Продолжительность */
  transition-timing-function: ease;  /* Функция времени */
  transition-delay: 0.1s;            /* Задержка начала */
  
  /* Краткая запись */
  transition: width 0.3s ease 0.1s;
  
  /* Анимация нескольких свойств */
  transition: width 0.3s ease, height 0.5s ease-in-out, background-color 0.2s;
  
  /* Анимация всех изменяемых свойств */
  transition: all 0.3s ease;
  
  /* Анимация с разными параметрами */
  transition: 
    width 0.3s ease,
    height 0.5s ease-in-out 0.1s,
    background-color 0.2s linear;
}
```

### Функции времени (timing functions)

```css
.timing-examples {
  /* Встроенные функции времени */
  transition-timing-function: ease;           /* ease-in-out с акцентом на начало */
  transition-timing-function: linear;         /* Равномерная скорость */
  transition-timing-function: ease-in;        /* Медленное начало */
  transition-timing-function: ease-out;       /* Медленный конец */
  transition-timing-function: ease-in-out;    /* Медленное начало и конец */
  
  /* Кастомные кривые Безье */
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
  
  /* Ступенчатые функции */
  transition-timing-function: step-start;     /* Начальное значение сразу */
  transition-timing-function: step-end;       /* Конечное значение сразу */
  transition-timing-function: steps(4, end);  /* 4 ступени, конец интервала */
}
```

### Практические примеры переходов

```css
/* Простой hover эффект */
.button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.button:hover {
  background-color: #0056b3;
}

/* Перемещение элемента */
.slide-element {
  position: relative;
  left: 0;
  transition: left 0.4s ease-in-out;
}

.slide-element:hover {
  left: 100px;
}

/* Изменение размеров */
.scale-element {
  width: 100px;
  height: 100px;
  background-color: #007bff;
  transition: transform 0.3s ease, width 0.3s ease, height 0.3s ease;
}

.scale-element:hover {
  transform: scale(1.1);
  width: 120px;
  height: 120px;
}

/* Цветовые переходы */
.color-change {
  background-color: #ff0000;
  transition: background-color 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.color-change:hover {
  background-color: #00ff00;
}
```

## CSS Animations Module

Animations позволяют создавать сложные анимации с помощью keyframes, без необходимости в JavaScript.

### Основные свойства анимации

```css
.animation-element {
  /* Основные свойства */
  animation-name: slideIn;                    /* Имя анимации */
  animation-duration: 2s;                     /* Продолжительность */
  animation-timing-function: ease-in-out;     /* Функция времени */
  animation-delay: 0.5s;                      /* Задержка начала */
  animation-iteration-count: infinite;        /* Количество повторений */
  animation-direction: normal;                /* Направление */
  animation-fill-mode: forwards;              /* Поведение до/после анимации */
  animation-play-state: running;              /* Состояние воспроизведения */
  
  /* Краткая запись */
  animation: slideIn 2s ease-in-out 0.5s infinite normal forwards;
  
  /* Несколько анимаций */
  animation: 
    slideIn 2s ease-in-out,
    pulse 1s ease-in-out infinite;
}
```

### Направления анимации

```css
.direction-examples {
  /* Направления */
  animation-direction: normal;        /* Нормальное воспроизведение */
  animation-direction: reverse;       /* Обратное воспроизведение */
  animation-direction: alternate;     /* Нормальное, затем обратное */
  animation-direction: alternate-reverse; /* Обратное, затем нормальное */
}
```

### Режимы заполнения (fill-mode)

```css
.fill-mode-examples {
  /* Поведение до и после анимации */
  animation-fill-mode: none;          /* Без влияния до/после */
  animation-fill-mode: forwards;      /* Сохраняет конечное состояние */
  animation-fill-mode: backwards;     /* Применяет начальное состояние */
  animation-fill-mode: both;          /* forwards + backwards */
}
```

### Создание keyframes

```css
/* Простая анимация */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Анимация с промежуточными состояниями */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  50% {
    transform: translateX(10px);
    opacity: 0.5;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

/* Комплексная анимация */
@keyframes complexAnimation {
  0% {
    transform: translateX(0) rotate(0deg);
    background-color: #ff0000;
    border-radius: 0;
  }
  25% {
    transform: translateX(100px) rotate(90deg);
    background-color: #00ff00;
    border-radius: 50%;
  }
  50% {
    transform: translateX(200px) rotate(180deg);
    background-color: #0000ff;
    border-radius: 0;
  }
  75% {
    transform: translateX(100px) rotate(270deg);
    background-color: #ffff00;
    border-radius: 50%;
  }
  100% {
    transform: translateX(0) rotate(360deg);
    background-color: #ff0000;
    border-radius: 0;
  }
}
```

## Практические применения анимаций

### Загрузочные индикаторы

```css
.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.pulse-loader {
  width: 20px;
  height: 20px;
  background-color: #007bff;
  border-radius: 50%;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
}
```

### Интерактивные элементы

```css
/* Плавающая кнопка */
.floating-button {
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

/* Эффект набора текста */
.typewriter {
  overflow: hidden;
  border-right: 2px solid;
  white-space: nowrap;
  animation: typing 3.5s steps(40, end), blink-caret 0.75s step-end infinite;
}

@keyframes typing {
  from {
    width: 0;
  }
  to {
    width: 100%;
  }
}

@keyframes blink-caret {
  from, to {
    border-color: transparent;
  }
  50% {
    border-color: black;
  }
}
```

### Микроанимации

```css
/* Появление элементов */
.fade-in-up {
  opacity: 0;
  transform: translateY(30px);
  animation: fadeInUp 0.6s ease-out forwards;
}

@keyframes fadeInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Эффекты при наведении */
.hover-bounce {
  transition: transform 0.2s ease;
}

.hover-bounce:hover {
  transform: scale(1.05);
  animation: bounce 0.6s ease-in-out;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-10px);
  }
  60% {
    transform: translateY(-5px);
  }
}
```

### Анимации для пользовательских интерфейсов

```css
/* Открытие/закрытие модального окна */
.modal {
  opacity: 0;
  transform: scale(0.8);
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.modal.open {
  opacity: 1;
  transform: scale(1);
}

/* Анимация меню гамбургера */
.hamburger {
  transition: transform 0.3s ease;
}

.hamburger.active {
  transform: rotate(45deg);
}

/* Сворачивание панелей */
.collapse {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease-out;
}

.collapse.open {
  max-height: 500px; /* или другое подходящее значение */
}
```

## Современные возможности

### Animation Worklet API (через CSS)

```css
/* Использование CSS Custom Properties для анимаций */
.animated-element {
  --progress: 0;
  transform: translateX(calc(var(--progress) * 100px));
  animation: updateProgress 2s infinite linear;
}

@keyframes updateProgress {
  0% {
    --progress: 0;
  }
  100% {
    --progress: 1;
  }
}
```

### Animation Range API

```css
.range-animation {
  animation-timeline: view();         /* Анимация по прокрутке */
  animation-name: slideIn;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

## Производительность и оптимизация

### Выбор оптимизированных свойств для анимации

```css
/* Хорошие свойства для анимации (GPU ускорены) */
.optimized-properties {
  animation: transform 0.3s ease;
  animation: opacity 0.3s ease;
}

/* Свойства, вызывающие перерисовку (избегать) */
.expensive-properties {
  animation: width 0.3s ease;         /* Вызывает перерасчет макета */
  animation: height 0.3s ease;
  animation: background-color 0.3s ease; /* Вызывает перерисовку */
  animation: border 0.3s ease;
}
```

### Использование will-change

```css
.performance-optimized {
  will-change: transform, opacity;    /* Подсказка браузеру */
  /* Только для элементов, которые действительно будут анимироваться */
}

/* Использование в JavaScript для управления */
.js-controlled {
  /* Устанавливается/удаляется через JS при необходимости */
}
```

## Поддержка браузерами

- Transitions: Поддерживается во всех современных браузерах
- Animations: Хорошая поддержка в современных браузерах (ограничения в старых IE)
- Необходимы префиксы для старых браузеров: `-webkit-transition`, `-moz-animation`

## Связанные темы

- [[CSS Transform Module]] - трансформации элементов
- [[CSS Custom Properties Module]] - кастомные свойства (переменные)
- [[CSS Containment Module]] - производительность и изоляция

## Заключение

CSS Animation и Transition Modules предоставляют мощные инструменты для создания плавных и интерактивных интерфейсов без использования JavaScript. Понимание различий между переходами и анимациями, а также оптимизации производительности, позволяет создавать эффективные и визуально привлекательные веб-приложения.