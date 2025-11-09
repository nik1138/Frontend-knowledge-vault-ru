# Сложные анимации в CSS: техники и производительность

## Введение

CSS-анимации позволяют создавать плавные, интерактивные и визуально привлекательные эффекты без использования JavaScript. Современные возможности CSS предоставляют мощные инструменты для создания сложных анимаций с высокой производительностью.

## Продвинутые техники CSS-анимаций

### Ключевые кадры и сложные траектории

```css
/* Сложная анимация с несколькими свойствами */
.complex-animation {
    animation: complexMove 3s ease-in-out infinite;
}

@keyframes complexMove {
    0% {
        transform: translateX(0) translateY(0) rotate(0deg) scale(1);
        opacity: 1;
        background-color: #3498db;
    }
    
    25% {
        transform: translateX(100px) translateY(50px) rotate(90deg) scale(1.2);
        background-color: #e74c3c;
    }
    
    50% {
        transform: translateX(200px) translateY(0) rotate(180deg) scale(0.8);
        opacity: 0.7;
        background-color: #2ecc71;
    }
    
    75% {
        transform: translateX(100px) translateY(-50px) rotate(270deg) scale(1.1);
        background-color: #f39c12;
    }
    
    100% {
        transform: translateX(0) translateY(0) rotate(360deg) scale(1);
        opacity: 1;
        background-color: #3498db;
    }
}

/* Анимация по сложной траектории (бесконечная петля) */
.orbit-animation {
    position: relative;
    width: 100px;
    height: 100px;
    animation: orbit 4s linear infinite;
}

@keyframes orbit {
    0% {
        transform: translate(0, 0) rotate(0deg) translate(150px) rotate(0deg);
    }
    100% {
        transform: translate(0, 0) rotate(360deg) translate(150px) rotate(-360deg);
    }
}

/* Анимация с изменением нескольких CSS-свойств */
.multi-property-animation {
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

### Анимации с использованием CSS-переменных

```css
/* Анимации с параметризацией через CSS-переменные */
.parametric-animation {
    --duration: 2s;
    --delay: 0s;
    --easing: ease-in-out;
    --size: 50px;
    
    width: var(--size);
    height: var(--size);
    animation: bounce var(--duration) var(--easing) var(--delay) infinite;
}

@keyframes bounce {
    0%, 100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-30px);
    }
}

/* Анимации с динамическими параметрами */
.animated-element {
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

### Анимации с кастомными easing функциями

```css
/* Пользовательские easing функции */
.custom-easing-animation {
    animation: slide 2s cubic-bezier(0.68, -0.55, 0.265, 1.55) infinite;
    /* cubic-bezier для "bounce" эффекта */
}

@keyframes slide {
    0% { transform: translateX(0); }
    50% { transform: translateX(200px); }
    100% { transform: translateX(0); }
}

/* Сложные easing функции для разных эффектов */
.easing-examples {
    display: flex;
    gap: 2rem;
    padding: 2rem;
}

.easing-box {
    width: 50px;
    height: 50px;
    background: #3498db;
    border-radius: 4px;
}

/* Ease-in-out с задержкой */
.easing-1 {
    animation: moveRight 2s cubic-bezier(0.42, 0, 0.58, 1) infinite alternate;
}

/* "Bounce" easing */
.easing-2 {
    animation: moveRight 2s cubic-bezier(0.68, -0.55, 0.265, 1.55) infinite alternate;
}

/* "Elastic" easing */
.easing-3 {
    animation: moveRight 2s cubic-bezier(0.5, 1.25, 0.75, 1.25) infinite alternate;
}

/* "Back" easing */
.easing-4 {
    animation: moveRight 2s cubic-bezier(0.68, -0.6, 0.32, 1.6) infinite alternate;
}

@keyframes moveRight {
    from { transform: translateX(0); }
    to { transform: translateX(200px); }
}
```

## Анимации с использованием современных CSS-свойств

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

### View Transitions API (экспериментально)

```css
/* Пример view transitions (псевдокод) */
@keyframes slideFromRight {
    from {
        translate: 100% 0;
        opacity: 0;
    }
    to {
        translate: 0 0;
        opacity: 1;
    }
}

@keyframes slideToLeft {
    from {
        translate: 0 0;
        opacity: 1;
    }
    to {
        translate: -100% 0;
        opacity: 0;
    }
}

.page-transition {
    view-transition-name: page-slide;
}

::view-transition-old(slide) {
    animation: 90ms cubic-bezier(0.4, 0, 1, 1) both slideToLeft;
}

::view-transition-new(slide) {
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) both slideFromRight;
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

## Производительность анимаций

### Оптимизация CSS-анимаций

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

### Анимации с requestAnimationFrame

```css
/* CSS-анимации в сочетании с JavaScript для сложного контроля */
.js-controlled-animation {
    transition: transform 0.1s linear;
}
```

```javascript
// JavaScript для сложного контроля анимаций
class AnimationController {
    constructor(element) {
        this.element = element;
        this.isActive = false;
        this.currentFrame = 0;
        this.maxFrames = 60; // 60 кадров для 1 секунды при 60fps
    }
    
    start() {
        this.isActive = true;
        this.animate();
    }
    
    stop() {
        this.isActive = false;
    }
    
    animate() {
        if (!this.isActive) return;
        
        // Рассчитываем прогресс анимации (0-1)
        const progress = this.currentFrame / this.maxFrames;
        
        // Применяем CSS-переменные для анимации
        this.element.style.setProperty('--animation-progress', progress);
        
        // Пример сложной анимации
        const x = Math.sin(progress * Math.PI * 2) * 100;
        const y = Math.cos(progress * Math.PI * 2) * 50;
        const rotation = progress * 360;
        
        this.element.style.transform = `translate(${x}px, ${y}px) rotate(${rotation}deg)`;
        
        this.currentFrame = (this.currentFrame + 1) % this.maxFrames;
        
        requestAnimationFrame(() => this.animate());
    }
}

// Использование
const animatedElement = document.querySelector('.js-controlled-animation');
const controller = new AnimationController(animatedElement);
controller.start();
```

### Анимации с учетом производительности устройства

```css
/* Условные анимации в зависимости от производительности */
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

/* Динамическая адаптация анимаций */
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

## Сложные интерактивные анимации

### Анимации на основе пользовательского ввода

```css
/* Анимации, реагирующие на позицию мыши */
.interactive-hover {
    position: relative;
    width: 200px;
    height: 200px;
    background: linear-gradient(45deg, #3498db, #2ecc71);
    border-radius: 8px;
    overflow: hidden;
    transition: transform 0.3s ease;
}

.interactive-hover::before {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.2);
    transform: translate(-50%, -50%);
    transition: width 0.6s, height 0.6s;
}

.interactive-hover:hover {
    transform: scale(1.05);
}

.interactive-hover:hover::before {
    width: 400px;
    height: 400px;
}

/* Анимации с использованием CSS переменных и JavaScript */
.interactive-rotation {
    width: 100px;
    height: 100px;
    background: #e74c3c;
    border-radius: 8px;
    animation: rotate var(--rotation-duration, 2s) linear infinite;
    transform: rotate(var(--rotation-angle, 0deg));
}
```

```javascript
// JavaScript для интерактивных анимаций
document.addEventListener('mousemove', (e) => {
    const elements = document.querySelectorAll('.interactive-rotation');
    elements.forEach(element => {
        // Рассчитываем угол вращения на основе позиции мыши
        const rect = element.getBoundingClientRect();
        const centerX = rect.left + rect.width / 2;
        const centerY = rect.top + rect.height / 2;
        
        const angle = Math.atan2(e.clientY - centerY, e.clientX - centerX) * (180 / Math.PI);
        element.style.setProperty('--rotation-angle', `${angle}deg`);
        
        // Рассчитываем расстояние для изменения скорости
        const distance = Math.sqrt(
            Math.pow(e.clientX - centerX, 2) + 
            Math.pow(e.clientY - centerY, 2)
        );
        
        const duration = Math.max(0.5, 2 - (distance / 200));
        element.style.setProperty('--rotation-duration', `${duration}s`);
    });
});
```

### Анимации прокрутки (Scroll-Driven Animations)

```css
/* Scroll-driven animations (экспериментально) */
.scroll-animation {
    animation: slideIn linear;
    animation-timeline: view();
    animation-range: entry 0% entry 100%;
}

@keyframes slideIn {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}

/* Анимации с разными диапазонами */
.scroll-fade {
    animation: fade linear;
    animation-timeline: view();
    animation-range: contain 0% contain 50%;
}

@keyframes fade {
    from { opacity: 0; }
    to { opacity: 1; }
}

/* Параллакс-эффект с анимациями */
.parallax-container {
    height: 200vh;
    overflow-y: auto;
}

.parallax-element {
    position: sticky;
    top: 0;
    height: 100vh;
    animation: parallax linear;
    animation-timeline: scroll();
    animation-range: entry 0% entry 100%;
}

@keyframes parallax {
    from { transform: translateY(0) scale(1); }
    to { transform: translateY(-50%) scale(1.2); }
}
```

## Современные библиотеки и инструменты анимаций

### CSS-анимации в сочетании с современными фреймворками

```css
/* Пример для использования с Tailwind CSS */
.animate-complex {
    animation: 
        fade-in 0.5s ease-out,
        slide-up 0.6s ease-out 0.1s,
        scale-bounce 0.8s ease-in-out infinite 1s;
}

/* Анимации с переменными для темизации */
:root {
    --animation-speed-fast: 0.2s;
    --animation-speed-normal: 0.5s;
    --animation-speed-slow: 1s;
    --animation-easing: cubic-bezier(0.4, 0, 0.2, 1);
}

.themed-animation {
    animation-duration: var(--animation-speed-normal);
    animation-timing-function: var(--animation-easing);
}

.themed-animation.fast {
    animation-duration: var(--animation-speed-fast);
}

.themed-animation.slow {
    animation-duration: var(--animation-speed-slow);
}
```

### Анимации с использованием CSS Shadow Parts

```css
/* Анимации веб-компонентов с shadow parts */
custom-animated-element::part(container) {
    animation: glow 2s ease-in-out infinite alternate;
}

custom-animated-element::part(button) {
    transition: all 0.3s ease;
}

custom-animated-element::part(button):hover {
    animation: pulse 0.5s ease-in-out;
}

@keyframes glow {
    from { box-shadow: 0 0 5px rgba(52, 152, 219, 0.5); }
    to { box-shadow: 0 0 20px rgba(52, 152, 219, 0.8); }
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
}
```

## Лучшие практики анимаций

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

### Система уведомлений с анимациями

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

### Анимированные табы

```css
/* Анимированные табы с плавными переходами */
.tabs-container {
    display: flex;
    border-bottom: 2px solid #ecf0f1;
    margin-bottom: 20px;
}

.tab-button {
    padding: 12px 24px;
    border: none;
    background: none;
    cursor: pointer;
    position: relative;
    transition: color 0.3s ease;
    color: #7f8c8d;
}

.tab-button.active {
    color: #3498db;
}

.tab-button.active::after {
    content: '';
    position: absolute;
    bottom: -2px;
    left: 0;
    right: 0;
    height: 2px;
    background: #3498db;
    animation: tabIndicator 0.3s ease forwards;
}

@keyframes tabIndicator {
    from { width: 0; left: 50%; }
    to { width: 100%; left: 0; }
}

/* Анимация содержимого табов */
.tab-content {
    display: none;
    animation: fadeInScale 0.3s ease forwards;
}

.tab-content.active {
    display: block;
}

@keyframes fadeInScale {
    from {
        opacity: 0;
        transform: scale(0.95);
    }
    to {
        opacity: 1;
        transform: scale(1);
    }
}

/* Анимация при переключении табов */
.tab-content.fade-out {
    animation: fadeOutScale 0.2s ease forwards;
}

@keyframes fadeOutScale {
    from {
        opacity: 1;
        transform: scale(1);
    }
    to {
        opacity: 0;
        transform: scale(0.95);
    }
}
```

## Ссылки на связанные темы
- [[styling/advanced]] - Продвинутые техники стилизации
- [[performance]] - Производительность CSS
- [[js/dom/animations]] - Анимации через JavaScript
- [[html/accessibility]] - Доступность

#programming #css #animations #performance #best-practices