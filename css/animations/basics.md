# CSS Анимации: основы, ключевые кадры и продвинутые техники

## Введение

CSS-анимации позволяют создавать плавные, интерактивные и визуально привлекательные эффекты без использования JavaScript. Современные возможности CSS предоставляют мощные инструменты для анимации элементов с высокой производительностью.

## Основы CSS-анимаций

### Свойства анимации

CSS-анимации управляются несколькими свойствами:

- `animation-name` — имя ключевых кадров
- `animation-duration` — продолжительность анимации
- `animation-timing-function` — функция временной шкалы
- `animation-delay` — задержка перед началом
- `animation-iteration-count` — количество повторений
- `animation-direction` — направление анимации
- `animation-fill-mode` — поведение до/после анимации
- `animation-play-state` — состояние воспроизведения

### Простая анимация

```css
/* Определение ключевых кадров */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Применение анимации */
.fade-in-element {
  animation-name: fadeIn;
  animation-duration: 1s;
}

/* Сокращенная запись */
.fade-in-element {
  animation: fadeIn 1s;
}
```

### Функции временной шкалы

```css
/* Предопределенные функции */
.ease-animation { animation-timing-function: ease; }
.linear-animation { animation-timing-function: linear; }
.ease-in-animation { animation-timing-function: ease-in; }
.ease-out-animation { animation-timing-function: ease-out; }
.ease-in-out-animation { animation-timing-function: ease-in-out; }

/* Кастомные функции */
.custom-cubic { animation-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); }
.custom-steps { animation-timing-function: steps(4, end); }

/* Практические примеры функций */
.bounce { animation-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); }
.elastic { animation-timing-function: cubic-bezier(0.5, 1.25, 0.75, 1.25); }
.back { animation-timing-function: cubic-bezier(0.68, -0.6, 0.32, 1.6); }
```

## Ключевые кадры (Keyframes)

### Базовое использование

```css
/* Простая анимация перемещения */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in {
  animation: slideIn 0.5s ease-out;
}

/* Анимация с промежуточными кадрами */
@keyframes bounce {
  0%, 20%, 53%, 80%, 100% {
    transition-timing-function: cubic-bezier(0.215, 0.610, 0.355, 1.000);
    transform: translate3d(0,0,0);
  }
  40%, 43% {
    transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
    transform: translate3d(0, -30px, 0);
  }
  70% {
    transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
    transform: translate3d(0, -15px, 0);
  }
  90% {
    transform: translate3d(0,-4px,0);
  }
}

.bounce-element {
  animation: bounce 1s ease-in-out;
}
```

### Анимация нескольких свойств

```css
@keyframes complexAnimation {
  0% {
    transform: translateX(0) rotate(0deg) scale(1);
    opacity: 1;
    background-color: #3498db;
  }
  25% {
    transform: translateX(100px) rotate(90deg) scale(1.2);
    background-color: #e74c3c;
  }
  50% {
    transform: translateX(200px) rotate(180deg) scale(0.8);
    opacity: 0.7;
    background-color: #2ecc71;
  }
  75% {
    transform: translateX(100px) rotate(270deg) scale(1.1);
    background-color: #f39c12;
  }
  100% {
    transform: translateX(0) rotate(360deg) scale(1);
    opacity: 1;
    background-color: #3498db;
  }
}

.complex-animated {
  animation: complexAnimation 3s ease-in-out infinite;
}
```

## Направления и повторения анимаций

### Направления

```css
/* Анимация вперед */
.forward { animation-direction: normal; }

/* Анимация назад */
.backward { animation-direction: reverse; }

/* Анимация туда-обратно */
.alternate { animation-direction: alternate; }

/* Анимация туда-обратно в обратном порядке */
.alternate-reverse { animation-direction: alternate-reverse; }

/* Практический пример */
@keyframes slide {
  0% { transform: translateX(0); }
  100% { transform: translateX(100px); }
}

.slide-normal { animation: slide 2s infinite; }
.slide-reverse { animation: slide 2s infinite reverse; }
.slide-alternate { animation: slide 2s infinite alternate; }
.slide-alt-reverse { animation: slide 2s infinite alternate-reverse; }
```

### Повторения

```css
/* Один раз */
.once { animation-iteration-count: 1; }

/* Бесконечно */
.infinite { animation-iteration-count: infinite; }

/* Конкретное количество раз */
.three-times { animation-iteration-count: 3; }

/* С дробным значением */
.one-and-half { animation-iteration-count: 1.5; }
```

### Режим заполнения

```css
/* Поведение до начала и после окончания анимации */
.none-fill { animation-fill-mode: none; }
.forwards { animation-fill-mode: forwards; }
.backwards { animation-fill-mode: backwards; }
.both { animation-fill-mode: both; }

/* Практический пример */
@keyframes slideOut {
  0% { transform: translateX(0); opacity: 1; }
  100% { transform: translateX(100%); opacity: 0; }
}

.slide-out {
  animation: slideOut 1s forwards; /* Сохраняет конечное состояние */
}
```

## Продвинутые техники анимации

### Анимации с использованием CSS-переменных

```css
/* Параметризованные анимации */
.parametric-animation {
  --animation-duration: 2s;
  --animation-delay: 0s;
  --animation-easing: ease-in-out;
  --animation-iterations: infinite;
  
  animation-duration: var(--animation-duration);
  animation-delay: var(--animation-delay);
  animation-timing-function: var(--animation-easing);
  animation-iteration-count: var(--animation-iterations);
  animation-name: pulse;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

/* Анимации с динамическими параметрами */
.dynamic-animation {
  --start-color: #3498db;
  --end-color: #e74c3c;
  --animation-steps: 4;
  
  background: linear-gradient(45deg, var(--start-color), var(--end-color));
  animation: colorShift calc(var(--animation-steps) * 0.5s) infinite;
}

@keyframes colorShift {
  0% { background: linear-gradient(45deg, var(--start-color), var(--end-color)); }
  25% { background: linear-gradient(45deg, var(--end-color), var(--start-color)); }
  50% { background: linear-gradient(-45deg, var(--start-color), var(--end-color)); }
  75% { background: linear-gradient(-45deg, var(--end-color), var(--start-color)); }
  100% { background: linear-gradient(45deg, var(--start-color), var(--end-color)); }
}
```

### Множественные анимации

```css
/* Применение нескольких анимаций */
.multiple-animations {
  animation: 
    fadeIn 1s ease-out,
    slideIn 0.8s ease-out 0.2s,
    pulse 2s ease-in-out infinite 1s;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  from { transform: translateX(-50px); }
  to { transform: translateX(0); }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}
```

### Анимации с изменением формы

```css
/* Анимация border-radius для создания эффекта изменения формы */
@keyframes morph {
  0% {
    border-radius: 20% 40% 60% 80%;
    transform: rotate(0deg);
  }
  25% {
    border-radius: 80% 20% 40% 60%;
    transform: rotate(90deg);
  }
  50% {
    border-radius: 60% 80% 20% 40%;
    transform: rotate(180deg);
  }
  75% {
    border-radius: 40% 60% 80% 20%;
    transform: rotate(270deg);
  }
  100% {
    border-radius: 20% 40% 60% 80%;
    transform: rotate(360deg);
  }
}

.morph-element {
  width: 100px;
  height: 100px;
  background: linear-gradient(45deg, #3498db, #2ecc71);
  animation: morph 4s ease-in-out infinite;
}
```

## Практические примеры анимаций

### Анимации загрузки

```css
/* Простой спиннер */
.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Более сложный спиннер */
.spinner-dots {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5px;
}

.spinner-dots::before,
.spinner-dots::after,
.spinner-dots span {
  content: '';
  width: 10px;
  height: 10px;
  background: #3498db;
  border-radius: 50%;
  animation: bounce 1.3s ease-in-out infinite;
}

.spinner-dots span {
  animation-delay: 0.1s;
}

.spinner-dots::after {
  animation-delay: 0.2s;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}
```

### Анимации кнопок

```css
/* Анимированная кнопка с эффектом волны */
.wave-button {
  position: relative;
  overflow: hidden;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  background: #3498db;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.wave-button::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.wave-button:active::before {
  width: 300px;
  height: 300px;
}

/* Анимированная кнопка с градиентным эффектом */
.gradient-button {
  position: relative;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  background: #3498db;
  color: white;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s ease;
}

.gradient-button::before {
  content: '';
  position: absolute;
  top: -100%;
  left: -100%;
  width: 200%;
  height: 200%;
  background: linear-gradient(
    45deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  transition: all 0.5s ease;
  z-index: 0;
}

.gradient-button:hover::before {
  top: -50%;
  left: -50%;
}

.gradient-button span {
  position: relative;
  z-index: 1;
}
```

### Анимации карточек

```css
/* Анимированная карточка с эффектом при наведении */
.card {
  width: 300px;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
}

.card:hover {
  transform: translateY(-8px) scale(1.02);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

/* Анимация с эффектом параллакса */
.parallax-card {
  position: relative;
  overflow: hidden;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.parallax-card::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, transparent 20%, rgba(255,255,255,0.1) 80%);
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.parallax-card:hover::before {
  opacity: 1;
  animation: rotate 10s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

## Производительность анимаций

### Оптимизация для производительности

```css
/* Использование transform и opacity для высокой производительности */
.performant-animation {
  /* Эти свойства анимируются на GPU */
  transform: translateX(0);
  opacity: 1;
  will-change: transform, opacity;
  animation: slideFade 1s ease-in-out infinite;
}

@keyframes slideFade {
  0% {
    transform: translateX(0) scale(1);
    opacity: 1;
  }
  50% {
    transform: translateX(100px) scale(1.1);
    opacity: 0.7;
  }
  100% {
    transform: translateX(0) scale(1);
    opacity: 1;
  }
}

/* Использование contain для изоляции анимаций */
.contain-animation {
  contain: layout style paint;
  animation: complexAnimation 2s infinite;
}

/* Избегаем анимации свойств, вызывающих relayout */
/* ПЛОХО: */
.bad-animation {
  animation: changeWidth 1s infinite; /* Вызывает relayout */
}

@keyframes changeWidth {
  0% { width: 100px; }
  100% { width: 200px; }
}

/* ХОРОШО: */
.good-animation {
  animation: changeTransform 1s infinite; /* Анимирует только GPU-свойства */
}

@keyframes changeTransform {
  0% { transform: scaleX(0.5); }
  100% { transform: scaleX(1); }
}
```

### Анимации с учетом производительности устройства

```css
/* Условные анимации в зависимости от предпочтений пользователя */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Адаптивные анимации */
.adaptive-animation {
  animation: complexMove 2s ease-in-out infinite;
}

/* Медленные устройства */
@media (prefers-reduced-motion: no-preference) and (scripting: enabled) {
  .adaptive-animation {
    animation-duration: 1.5s; /* Более быстрая анимация для производительных устройств */
  }
}

/* Высокочастотные дисплеи */
@media (min-resolution: 2dppx) {
  .adaptive-animation {
    animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1); /* Более плавная функция для высокого DPI */
  }
}
```

## Современные возможности CSS-анимаций

### CSS Motion Path (экспериментально)

```css
/* Анимация по сложному пути */
.motion-path-element {
  offset-path: path('M 0 100 Q 200 0 400 100 T 800 100');
  offset-distance: 0%;
  offset-rotate: auto;
  animation: followPath 4s linear infinite;
  width: 40px;
  height: 40px;
  background: #e74c3c;
  border-radius: 50%;
}

@keyframes followPath {
  to {
    offset-distance: 100%;
  }
}

/* Анимация по круговому пути */
.circular-motion {
  offset-path: circle(100px at center);
  offset-distance: 0%;
  animation: circular 3s linear infinite;
  width: 30px;
  height: 30px;
  background: #2ecc71;
  border-radius: 50%;
}

@keyframes circular {
  to {
    offset-distance: 100%;
  }
}
```

### Container Queries с анимациями

```css
/* Анимации, адаптирующиеся к размеру контейнера */
.animation-container {
  container-type: inline-size;
  container-name: animation-container;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.animated-item {
  width: 100px;
  height: 100px;
  background: #3498db;
  animation: slide 2s infinite alternate;
}

@container animation-container (min-width: 400px) {
  .animated-item {
    animation: slide-wide 2s infinite alternate;
  }
}

@container animation-container (min-width: 600px) {
  .animated-item {
    animation: slide-extra-wide 2s infinite alternate;
  }
}

@keyframes slide {
  from { transform: translateX(0); }
  to { transform: translateX(50px); }
}

@keyframes slide-wide {
  from { transform: translateX(0); }
  to { transform: translateX(150px); }
}

@keyframes slide-extra-wide {
  from { transform: translateX(0); }
  to { transform: translateX(250px); }
}
```

## Лучшие практики

### 1. Семантические имена анимаций

```css
/* Хорошие имена анимаций */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideInFromLeft {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

@keyframes bounceIn {
  0% { transform: scale(0.3); opacity: 0; }
  50% { transform: scale(1.05); }
  70% { transform: scale(0.9); }
  100% { transform: scale(1); opacity: 1; }
}

/* Плохие имена анимаций */
@keyframes animation1 { /* Слишком общее */ }
@keyframes weirdMove { /* Неописательное */ }
```

### 2. Контроль анимаций через CSS-переменные

```css
/* Управляемые анимации */
.controlled-animation {
  --animation-duration: 1s;
  --animation-delay: 0s;
  --animation-iteration: infinite;
  --animation-direction: normal;
  
  animation-duration: var(--animation-duration);
  animation-delay: var(--animation-delay);
  animation-iteration-count: var(--animation-iteration);
  animation-direction: var(--animation-direction);
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Пример использования разных настроек */
.animation-fast { --animation-duration: 0.5s; }
.animation-slow { --animation-duration: 2s; }
.animation-delayed { --animation-delay: 1s; }
.animation-once { --animation-iteration: 1; }
.animation-reverse { --animation-direction: reverse; }
```

### 3. Оптимизация для доступности

```css
/* Анимации с учетом доступности */
.accessible-animation {
  animation: slideIn 0.3s ease-out;
}

/* Отключение анимаций при предпочтении пользователя */
@media (prefers-reduced-motion: reduce) {
  .accessible-animation {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
  
  /* Альтернативные стили без анимации */
  .accessible-animation {
    opacity: 1;
    transform: none;
  }
}

/* Плавное изменение свойств для пользователей */
.smooth-transition {
  transition: all 0.2s ease;
}

.smooth-transition.reduce-motion {
  transition: none;
}
```

## Примеры из промышленной разработки

### Анимированная система уведомлений

```css
/* Анимированные уведомления */
.notifications-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.notification {
  --notification-bg: #3498db;
  --notification-text: white;
  
  background: var(--notification-bg);
  color: var(--notification-text);
  padding: 15px 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  min-width: 300px;
  max-width: 400px;
  
  /* Анимация появления */
  transform: translateX(100%);
  opacity: 0;
  animation: slideInRight 0.3s ease-out forwards;
}

.notification.show {
  animation: slideInRight 0.3s ease-out forwards;
}

.notification.hide {
  animation: slideOutRight 0.3s ease-out forwards;
}

.notification.success { --notification-bg: #2ecc71; }
.notification.error { --notification-bg: #e74c3c; }
.notification.warning { --notification-bg: #f39c12; --notification-text: #2c3e50; }
.notification.info { --notification-bg: #3498db; }

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOutRight {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

/* Анимация прогресса */
.notification-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 3px;
  background: rgba(255,255,255,0.3);
  width: 100%;
  transform-origin: left;
  transform: scaleX(1);
  animation: progress linear;
}

.notification.success .notification-progress { background: rgba(255,255,255,0.5); }
.notification.error .notification-progress { background: rgba(255,255,255,0.5); }

@keyframes progress {
  from {
    transform: scaleX(1);
  }
  to {
    transform: scaleX(0);
  }
}
```

### Анимированный прогресс-бар

```css
/* Сложный анимированный прогресс-бар */
.progress-container {
  width: 100%;
  height: 8px;
  background: #ecf0f1;
  border-radius: 4px;
  overflow: hidden;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #3498db, #2ecc71);
  border-radius: 4px;
  width: 0%;
  transition: width 0.4s ease;
  position: relative;
}

/* Анимация загрузки */
.progress-bar.loading {
  animation: loadingPulse 2s ease-in-out infinite;
}

@keyframes loadingPulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

/* Анимированные полосы загрузки */
.progress-bar.striped::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    -45deg,
    rgba(255, 255, 255, 0.2) 25%,
    transparent 25%,
    transparent 50%,
    rgba(255, 255, 255, 0.2) 50%,
    rgba(255, 255, 255, 0.2) 75%,
    transparent 75%
  );
  background-size: 30px 30px;
  animation: progressStripes 1s linear infinite;
}

@keyframes progressStripes {
  0% { background-position: 0 0; }
  100% { background-position: 30px 0; }
}

/* Анимация при достижении цели */
.progress-bar.complete {
  animation: completePulse 0.6s ease-in-out 3;
}

@keyframes completePulse {
  0%, 100% { transform: scaleX(1); }
  50% { transform: scaleX(1.02); }
}
```

## Ссылки на связанные темы
- [[advanced]] - Продвинутые техники анимаций
- [[performance]] - Производительность CSS
- [[js/dom/animations]] - Анимации через JavaScript
- [[html/accessibility]] - Доступность

#programming #css #animations #keyframes #performance #best-practices