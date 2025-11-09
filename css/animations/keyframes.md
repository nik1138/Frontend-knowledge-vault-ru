# Анимации - Ключевые кадры

CSS-анимации с использованием ключевых кадров (@keyframes) позволяют создавать сложные анимации с множеством промежуточных состояний. В отличие от переходов, анимации могут запускаться автоматически и содержать несколько точек изменения свойств.

## Основы @keyframes

### Синтаксис

```css
/* Определение анимации */
@keyframes animation-name {
  from {
    /* Начальное состояние (0%) */
    transform: translateX(0);
    opacity: 1;
  }
  
  to {
    /* Конечное состояние (100%) */
    transform: translateX(100px);
    opacity: 0;
  }
}

/* Или с процентами */
@keyframes slide-and-fade {
  0% {
    transform: translateX(0);
    opacity: 1;
  }
  
  50% {
    transform: translateX(50px);
    opacity: 0.5;
  }
  
  100% {
    transform: translateX(100px);
    opacity: 0;
  }
}
```

### Применение анимации

```css
.animated-element {
  /* Применение анимации */
  animation-name: slide-and-fade;
  animation-duration: 2s;
  animation-timing-function: ease-in-out;
  animation-delay: 0.5s;
  animation-iteration-count: 3;  /* Повторить 3 раза */
  animation-direction: alternate; /* Направление */
  animation-fill-mode: forwards;  /* Поведение до/после анимации */
  animation-play-state: running;  /* Состояние воспроизведения */
}

/* Сокращенное свойство animation */
.animated-element {
  animation: slide-and-fade 2s ease-in-out 0.5s 3 alternate forwards running;
}
```

## Свойства анимации

### animation-name

Определяет имя анимации, определенной в @keyframes:

```css
.element {
  animation-name: bounce, rotate;  /* Несколько анимаций */
}
```

### animation-duration

Определяет продолжительность одной итерации анимации:

```css
.element {
  animation-duration: 2s;         /* 2 секунды */
  animation-duration: 2000ms;     /* 2000 миллисекунд */
  
  /* Разная продолжительность для нескольких анимаций */
  animation-name: slide, fade;
  animation-duration: 1s, 2s;
}
```

### animation-timing-function

Определяет, как вычисляются значения анимации:

```css
.element {
  animation-timing-function: ease;        /* По умолчанию */
  animation-timing-function: linear;      /* Равномерная скорость */
  animation-timing-function: ease-in;     /* Медленно в начале */
  animation-timing-function: ease-out;    /* Медленно в конце */
  animation-timing-function: ease-in-out; /* Медленно в начале и в конце */
  
  /* Кастомные кривые Безье */
  animation-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
  
  /* Разные функции для нескольких анимаций */
  animation-name: slide, fade;
  animation-timing-function: ease-in, linear;
}
```

### animation-delay

Определяет задержку перед началом анимации:

```css
.element {
  animation-delay: 1s;  /* Задержка 1 секунда */
  
  /* Разная задержка для нескольких анимаций */
  animation-name: slide, fade;
  animation-delay: 0s, 0.5s;
}
```

### animation-iteration-count

Определяет количество повторений анимации:

```css
.element {
  animation-iteration-count: 1;      /* По умолчанию - один раз */
  animation-iteration-count: 3;      /* Три раза */
  animation-iteration-count: infinite; /* Бесконечно */
  
  /* Разное количество итераций для нескольких анимаций */
  animation-name: slide, fade;
  animation-iteration-count: infinite, 2;
}
```

### animation-direction

Определяет направление воспроизведения анимации:

```css
.element {
  animation-direction: normal;        /* По умолчанию - от 0% к 100% */
  animation-direction: reverse;       /* От 100% к 0% */
  animation-direction: alternate;     /* Нормально, затем обратно */
  animation-direction: alternate-reverse; /* Обратно, затем нормально */
}
```

### animation-fill-mode

Определяет, как стили применяются до начала и после окончания анимации:

```css
.element {
  animation-fill-mode: none;      /* По умолчанию */
  animation-fill-mode: forwards;  /* Сохраняет конечное состояние */
  animation-fill-mode: backwards; /* Применяет начальное состояние до начала */
  animation-fill-mode: both;      /* И forwards, и backwards */
}
```

### animation-play-state

Позволяет воспроизводить или приостанавливать анимацию:

```css
.element {
  animation-play-state: running;  /* По умолчанию */
  animation-play-state: paused;   /* Приостановлено */
}

.element:hover {
  animation-play-state: paused;  /* Приостановить при наведении */
}
```

## Сокращенное свойство animation

```css
.element {
  /* name duration timing-function delay iteration-count direction fill-mode play-state */
  animation: slide-and-fade 2s ease-in-out 0.5s infinite alternate forwards running;
  
  /* Можно комбинировать несколько анимаций */
  animation: 
    bounce 1s ease infinite,
    rotate 2s linear infinite;
}
```

## Практические примеры

### Пример 1: Загрузка спиннера

```css
@keyframes spinner {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spinner 1s linear infinite;
}
```

### Пример 2: Пульсирующая анимация

```css
@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.pulse-button {
  padding: 12px 24px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 4px;
  animation: pulse 2s infinite;
}

.pulse-button:hover {
  animation: none;
}
```

### Пример 3: Появление с анимацией

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translate3d(0, 20px, 0);
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0, 0);
  }
}

.fade-in-element {
  opacity: 0;
  animation: fadeInUp 0.6s ease-out forwards;
}

/* Анимация с задержкой для последовательного появления */
.fade-in-element:nth-child(1) { animation-delay: 0.1s; }
.fade-in-element:nth-child(2) { animation-delay: 0.2s; }
.fade-in-element:nth-child(3) { animation-delay: 0.3s; }
```

### Пример 4: Анимация хлебных крошек

```css
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

.bounce-element {
  display: inline-block;
  animation: bounce 2s infinite;
}
```

### Пример 5: Сложная анимация элемента

```css
@keyframes complex-animation {
  0% {
    transform: translateX(0) scale(1);
    opacity: 1;
    background-color: #3498db;
  }
  25% {
    transform: translateX(100px) scale(1.1);
    opacity: 0.8;
    background-color: #9b59b6;
  }
  50% {
    transform: translateX(200px) scale(1);
    opacity: 0.6;
    background-color: #e74c3c;
  }
  75% {
    transform: translateX(100px) scale(0.9);
    opacity: 0.8;
    background-color: #f39c12;
  }
  100% {
    transform: translateX(0) scale(1);
    opacity: 1;
    background-color: #3498db;
  }
}

.complex-animated {
  width: 100px;
  height: 100px;
  background-color: #3498db;
  animation: complex-animation 4s ease-in-out infinite;
}
```

## Современные возможности

### CSS-переменные в анимациях

```css
:root {
  --animation-duration: 2s;
  --primary-color: #3498db;
  --secondary-color: #e74c3c;
}

@keyframes color-change {
  0% {
    background-color: var(--primary-color);
  }
  100% {
    background-color: var(--secondary-color);
  }
}

.modern-animated {
  animation: color-change var(--animation-duration) infinite alternate;
}
```

### Адаптивные анимации

```css
@keyframes adaptive-move {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(clamp(50px, 10vw, 100px));
  }
}

.adaptive-animated {
  animation: adaptive-move 3s ease-in-out infinite alternate;
}

/* Учет пользовательских предпочтений */
@media (prefers-reduced-motion: reduce) {
  .adaptive-animated {
    animation: none;
  }
}
```

## Продвинутые техники

### Ступенчатые анимации

```css
@keyframes step-animation {
  0%, 20% {
    background-color: red;
    transform: translateX(0);
  }
  21%, 40% {
    background-color: blue;
    transform: translateX(50px);
  }
  41%, 60% {
    background-color: green;
    transform: translateX(100px);
  }
  61%, 80% {
    background-color: yellow;
    transform: translateX(150px);
  }
  81%, 100% {
    background-color: purple;
    transform: translateX(200px);
  }
}

.step-animated {
  animation: step-animation 5s step-end infinite;
}
```

### Комбинирование анимаций

```css
@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.combined-animation {
  animation: 
    float 3s ease-in-out infinite,
    rotate 10s linear infinite;
}
```

## Лучшие практики

1. **Используйте hardware-accelerated свойства** - transform и opacity для лучшей производительности
2. **Ограничивайте количество одновременных анимаций** - избегайте перегрузки интерфейса
3. **Учитывайте пользовательские предпочтения** - уважайте `prefers-reduced-motion`
4. **Оптимизируйте длительность анимаций** - обычно от 200ms до 2s
5. **Используйте логичные timing функции** - соответствующие типу анимации
6. **Тестируйте на разных устройствах** - особенно на мобильных с ограниченными ресурсами

## Производительность

Для оптимальной производительности анимаций:

```css
.performant-animation {
  /* Используйте transform и opacity */
  animation: transform-opacity-animation 1s ease infinite;
  
  /* Включите GPU ускорение при необходимости */
  will-change: transform;
  
  /* Или используйте transform3d для активации GPU */
  transform: translateZ(0);
}
```

## Заключение

CSS-анимации с ключевыми кадрами предоставляют мощный инструмент для создания сложных и привлекательных анимаций на веб-страницах. Они позволяют создавать плавные переходы, интерактивные элементы и визуальные эффекты без использования JavaScript. Понимание всех свойств анимации и следование лучшим практикам позволяет создавать эффективные и приятные пользовательские интерфейсы.

#programming #css #keyframes #animations #web-development #frontend