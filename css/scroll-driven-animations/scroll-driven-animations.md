---
aliases: ["Scroll-Driven Animations", "Анимации на основе прокрутки", "CSS Scroll-Driven Animations"]
tags: [css, animations, scroll, modern-css, web-development, frontend]
---

# Scroll-Driven Animations в CSS

## Введение

Scroll-Driven Animations — это современная функция CSS, которая позволяет синхронизировать анимации с прокруткой элементов. Эта технология позволяет создавать плавные анимации, которые активируются или изменяются в зависимости от позиции прокрутки, обеспечивая более интерактивный и плавный пользовательский опыт.

## Основные концепции

### Timeline и Range-based Animation

Scroll-driven animations работают с двумя основными концепциями:
- **Timeline** — временная шкала, привязанная к прокрутке
- **Range-based animation** — анимация, ограниченная определенным диапазоном прокрутки

### Свойства Scroll-Driven Animations

- `animation-timeline` — определяет, какая временная шкала используется для анимации
- `animation-range-start` — начальная точка анимации в прокрутке
- `animation-range-end` — конечная точка анимации в прокрутке

## Основы Scroll-Driven Animations

### Синтаксис

```css
.element {
  /* Основная анимация */
  animation-name: slideIn;
  animation-duration: 1s; /* Игнорируется при scroll-driven */
  
  /* Привязка к прокрутке */
  animation-timeline: scroll();
  
  /* Опционально: диапазон анимации */
  animation-range-start: normal;
  animation-range-end: normal;
}
```

### Простой пример

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.scroll-element {
  animation: fadeIn 1s linear;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

.slide-element {
  animation: slideIn 1s linear;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}
```

## Типы временных шкал

### view() timeline

Временная шкала, привязанная к видимости элемента в области просмотра:

```css
.view-driven {
  animation: reveal 1s linear;
  animation-timeline: view();
  /* Анимация запускается при прокрутке элемента в область просмотра */
}
```

### scroll() timeline

Временная шкала, привязанная к прокрутке конкретного элемента:

```css
.scroll-driven {
  animation: progress 1s linear;
  animation-timeline: scroll(root); /* Привязка к прокрутке корневого элемента */
  /* или */
  animation-timeline: scroll(.container); /* Привязка к прокрутке .container */
}
```

## Range-based Animation

### Определение диапазонов

```css
.range-example {
  animation: morph 1s linear;
  animation-timeline: view();
  
  /* Начало анимации: когда элемент появляется в области просмотра */
  animation-range-start: entry;
  /* Конец анимации: когда элемент полностью в области просмотра */
  animation-range-end: entry 100%;
}

/* Альтернативные значения диапазона */
.entry-to-cover {
  animation-timeline: view();
  animation-range: entry 0% cover 100%;
}

.cover-to-cover {
  animation-timeline: view();
  animation-range: cover 0% cover 100%;
}
```

### Единицы измерения диапазонов

- `percent` — процент от области просмотра или элемента прокрутки
- `px` — пиксельные значения
- `auto` — автоматическое определение диапазона

```css
.pixel-range {
  animation-timeline: scroll(root);
  animation-range: 100px 500px; /* Анимация от 100px до 500px прокрутки */
}

.percent-range {
  animation-timeline: view();
  animation-range: entry 20% entry 80%; /* Анимация от 20% до 80% видимости */
}
```

## Практические примеры

### 1. Анимация появления при прокрутке

```css
@keyframes fadeInUp {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.scroll-reveal {
  animation-name: fadeInUp;
  animation-duration: 1s;
  animation-timeline: view();
  animation-range: entry 20% entry 80%;
  opacity: 0;
}
```

### 2. Параллакс-эффект

```css
@keyframes parallax {
  0% { transform: translateY(0); }
  100% { transform: translateY(-50%); }
}

.parallax-element {
  animation: parallax linear;
  animation-timeline: scroll(root);
  animation-range: 0px 500px;
  position: sticky;
  top: 0;
}
```

### 3. Прогресс-индикатор

```css
@keyframes progressFill {
  0% { width: 0%; }
  100% { width: 100%; }
}

.progress-bar {
  animation: progressFill linear;
  animation-timeline: scroll(root);
  animation-range: 0px 100vh;
  height: 4px;
  background: #3498db;
  position: fixed;
  top: 0;
  left: 0;
  width: 0%;
}
```

### 4. Анимация появления списка

```css
@keyframes staggeredFade {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.staggered-list li {
  animation: staggeredFade 0.5s ease-out;
  animation-timeline: view();
  animation-range: entry 30% entry 70%;
}

.staggered-list li:nth-child(1) { animation-delay: 0s; }
.staggered-list li:nth-child(2) { animation-delay: 0.1s; }
.staggered-list li:nth-child(3) { animation-delay: 0.2s; }
.staggered-list li:nth-child(4) { animation-delay: 0.3s; }
```

## Продвинутые техники

### Множественные анимации с разными временными шкалами

```css
.complex-animation {
  /* Анимация поворота */
  animation: rotate 1s linear;
  animation-timeline: view();
  animation-range: entry 0% entry 50%;
}

.complex-animation .child-element {
  /* Дополнительная анимация масштабирования */
  animation: scale 1s ease;
  animation-timeline: view();
  animation-range: entry 25% entry 75%;
}
```

### Синхронизация нескольких элементов

```css
@keyframes syncFadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes syncSlide {
  from { transform: translateX(-50px); }
  to { transform: translateX(0); }
}

.sync-element-1 {
  animation: syncFadeIn 1s ease;
  animation-timeline: view();
  animation-range: entry 10% entry 40%;
}

.sync-element-2 {
  animation: syncSlide 1s ease;
  animation-timeline: view();
  animation-range: entry 10% entry 40%;
}
```

### Анимации с изменением свойств

```css
@keyframes colorTransition {
  0% { background-color: #3498db; }
  100% { background-color: #e74c3c; }
}

@keyframes sizeTransition {
  0% { width: 100px; height: 100px; }
  100% { width: 200px; height: 200px; }
}

.color-changing-box {
  animation: colorTransition linear;
  animation-timeline: scroll(root);
  animation-range: 0px 500px;
}

.size-changing-box {
  animation: sizeTransition linear;
  animation-timeline: scroll(root);
  animation-range: 0px 500px;
}
```

## Совместимость и резервные варианты

### Проверка поддержки

```css
@supports (animation-timeline: view()) {
  .supported-element {
    animation: fadeIn 1s linear;
    animation-timeline: view();
    animation-range: entry 20% entry 80%;
  }
}

@supports not (animation-timeline: view()) {
  .fallback-element {
    /* Резервный вариант без scroll-driven анимации */
    opacity: 1;
    transform: none;
  }
}
```

### Постепенное улучшение

```css
/* Базовые стили */
.scroll-element {
  opacity: 1;
  transform: none;
  transition: opacity 0.3s, transform 0.3s;
}

/* Современные стили */
@supports (animation-timeline: view()) {
  .scroll-element {
    opacity: 0;
    animation: fadeIn 1s linear;
    animation-timeline: view();
    animation-range: entry 20% entry 80%;
  }
}
```

## Производительность и оптимизация

### Оптимизация scroll-driven анимаций

```css
.optimized-scroll-element {
  /* Использование свойств, которые не вызывают перерисовку */
  animation: transform-opacity-animation linear;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
  
  /* Активация аппаратного ускорения */
  will-change: transform, opacity;
}

/* Избегайте анимации свойств, вызывающих layout */
.bad-example {
  animation: width-height-animation linear;
  animation-timeline: view();
  /* Избегайте анимации width, height, margin, padding и т.д. */
}
```

### Ограничение количества анимируемых элементов

```css
/* Оптимизированный подход */
.scroll-container {
  contain: layout style paint;
}

.scroll-element {
  /* Ограничение области влияния анимации */
  contain: layout style paint;
}
```

## Примеры из промышленной разработки

### Лендинг-страница с scroll-driven анимациями

```css
/* Заголовок */
.hero-title {
  animation: heroFadeIn 1.5s ease-out;
  animation-timeline: view();
  animation-range: entry 0% entry 50%;
}

/* Контентные блоки */
.content-section {
  animation: contentSlideIn 1s ease;
  animation-timeline: view();
  animation-range: entry 20% entry 80%;
}

/* Секция с цифрами */
.stat-item {
  animation: countUp 1s ease-out;
  animation-timeline: view();
  animation-range: entry 0% cover 100%;
}

/* Подвал */
.footer {
  animation: footerReveal 1s ease;
  animation-timeline: view();
  animation-range: entry 30% entry 100%;
}
```

### Портфолио с анимациями проектов

```css
.project-card {
  animation: cardReveal 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  animation-timeline: view();
  animation-range: entry 10% entry 90%;
  opacity: 0;
}

.project-card:nth-child(odd) {
  animation: slideFromLeft 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  animation-timeline: view();
  animation-range: entry 10% entry 90%;
}

.project-card:nth-child(even) {
  animation: slideFromRight 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  animation-timeline: view();
  animation-range: entry 10% entry 90%;
}
```

## Лучшие практики

### 1. Использование семантических имен анимаций

```css
/* Хорошо */
.animation-scroll-reveal { /* ... */ }
.animation-appear-on-view { /* ... */ }

/* Не рекомендуется */
.animation-1 { /* ... */ }
.complex-move { /* ... */ }
```

### 2. Учет доступности

```css
/* Уважение к предпочтениям пользователя по анимации */
@media (prefers-reduced-motion: reduce) {
  .scroll-element {
    animation: none;
    opacity: 1;
    transform: none;
  }
}
```

### 3. Плавные переходы

```css
.smooth-scroll-animation {
  animation: fadeInUp 1s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  animation-timeline: view();
  animation-range: entry 20% entry 80%;
}
```

## Заключение

Scroll-Driven Animations представляют собой мощный инструмент для создания интерактивных и плавных анимаций, синхронизированных с прокруткой. Эта технология позволяет создавать более вовлекающий пользовательский опыт, при этом важно учитывать производительность и доступность.

При правильном использовании scroll-driven анимации могут значительно улучшить восприятие контента и создать более плавные переходы между различными частями веб-страницы.

## См. также

- [[CSS View Transitions API]]
- [[Animation Composition]]
- [[Modern CSS Animations]]
- [[CSS Animation Performance]]
- [[Advanced CSS Animations]]
- [[Animations and Transitions]]
- [[CSS Custom Properties]]
- [[Container Queries]]
- [[CSS Grid]]
- [[CSS Flexbox]]
- [[CSS Containment]]
- [[CSS Transformations]]
- [[CSS Transitions]]
- [[CSS Keyframes]]
- [[CSS Timing Functions]]

## Дополнительные ресурсы

- [MDN Web Docs: Scroll-Driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)
- [CSS Working Group: Scroll-Driven Animations Specification](https://www.w3.org/TR/scroll-animations-1/)
- [Web.dev: Learn Scroll-Driven Animations](https://web.dev/scroll-driven-animations/)