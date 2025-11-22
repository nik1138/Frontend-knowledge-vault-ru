---
aliases: ["CSS View Transitions API", "View Transitions", "CSS View Transitions", "CSS Transition API"]
tags: [css, animations, transitions, modern-css, web-development, frontend, view-transitions]
---

# CSS View Transitions API

## Введение

CSS View Transitions API — это новая функция CSS, которая позволяет создавать плавные переходы между разными состояниями DOM без необходимости в сложных анимациях или JavaScript-библиотеках. Эта технология предоставляет способ анимировать переходы между разными "представлениями" (views) приложения, обеспечивая более плавный и вовлекающий пользовательский опыт.

## Основные концепции

### Что такое View Transitions

View Transitions API позволяет создавать анимации перехода между двумя разными DOM-состояниями. Это особенно полезно для:
- Навигации между страницами/секциями
- Переключения между режимами просмотра
- Анимации при добавлении/удалении элементов
- Сложных UI-переходов

### Основные возможности

- Автоматическое создание анимаций перехода
- Поддержка сложных преобразований
- Совместимость с существующими CSS-анимациями
- Контроль над продолжительностью и таймингом

## Основы View Transitions API

### JavaScript API

```javascript
// Простой пример использования View Transitions API
function navigateToNewView() {
  document.startViewTransition(() => {
    // Изменяем DOM
    document.getElementById('current-view').hidden = true;
    document.getElementById('new-view').hidden = false;
  });
}

// Более сложный пример с обновлением состояния
async function updateView(newContent) {
  const transition = document.startViewTransition(() => {
    // Обновляем содержимое
    document.querySelector('.content').innerHTML = newContent;
  });
  
  // Дожидаемся завершения анимации
  await transition.finished;
  console.log('Transition completed');
}
```

### CSS-свойства для View Transitions

```css
/* Основные селекторы для View Transitions */
::view-transition {
  /* Общий стиль для всех view transition элементов */
}

::view-transition-old(root) {
  /* Стили для старого представления корневого элемента */
}

::view-transition-new(root) {
  /* Стили для нового представления корневого элемента */
}

/* Анимации для конкретных элементов */
::view-transition-group(my-element) {
  /* Стили для группы перехода элемента */
}

::view-transition-old(my-element) {
  /* Стили для старого состояния элемента */
}

::view-transition-new(my-element) {
  /* Стили для нового состояния элемента */
}
```

## Практические примеры

### 1. Простой переход между представлениями

```css
/* Стили для view transition */
::view-transition-old(root) {
  animation: fade-out 0.25s ease-in both;
}

::view-transition-new(root) {
  animation: fade-in 0.25s ease-out 0.25s both;
}

@keyframes fade-out {
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
}

/* Альтернативный пример с перемещением */
::view-transition-old(my-card) {
  animation: slide-out-right 0.3s ease-in both;
}

::view-transition-new(my-card) {
  animation: slide-in-left 0.3s ease-out 0.1s both;
}

@keyframes slide-out-right {
  to { transform: translateX(100%); }
}

@keyframes slide-in-left {
  from { transform: translateX(-100%); }
}
```

```javascript
// JavaScript для управления переходами
function switchViews(viewId) {
  document.startViewTransition(() => {
    document.querySelectorAll('.view').forEach(view => {
      view.style.display = 'none';
    });
    document.getElementById(viewId).style.display = 'block';
  });
}
```

### 2. Анимация элемента при изменении

```css
.card {
  view-transition-name: card-transition;
  /* Другие стили */
}

::view-transition-group(card-transition) {
  isolation: isolate;
}

::view-transition-old(card-transition) {
  animation: scale-down 0.3s ease-out both;
}

::view-transition-new(card-transition) {
  animation: scale-up 0.3s ease-out 0.1s both;
}

@keyframes scale-down {
  to { transform: scale(0.8); opacity: 0; }
}

@keyframes scale-up {
  from { transform: scale(0.8); opacity: 0; }
}
```

```javascript
// Функция для изменения карточки с анимацией
function updateCard(cardId, newContent) {
  document.startViewTransition(() => {
    const card = document.getElementById(cardId);
    card.innerHTML = newContent;
  });
}
```

### 3. Сложный переход с несколькими элементами

```css
/* Определение нескольких view transition имен */
.header {
  view-transition-name: header;
}

.sidebar {
  view-transition-name: sidebar;
}

.main-content {
  view-transition-name: main-content;
}

/* Анимации для каждого элемента */
::view-transition-old(header) {
  animation: slide-up 0.4s ease-in both;
}

::view-transition-new(header) {
  animation: slide-down 0.4s ease-out 0.2s both;
}

::view-transition-old(sidebar) {
  animation: fade-out 0.3s ease both;
}

::view-transition-new(sidebar) {
  animation: fade-in 0.3s ease 0.3s both;
}

::view-transition-old(main-content) {
  animation: scale-down 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

::view-transition-new(main-content) {
  animation: scale-up 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) 0.2s both;
}

@keyframes slide-up {
  to { transform: translateY(-100%); opacity: 0; }
}

@keyframes slide-down {
  from { transform: translateY(-100%); opacity: 0; }
}

@keyframes fade-out {
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
}

@keyframes scale-down {
  to { transform: scale(0.9); opacity: 0; }
}

@keyframes scale-up {
  from { transform: scale(0.9); opacity: 0; }
}
```

## Продвинутые техники

### Динамические переходы

```css
/* Переходы с использованием CSS-переменных */
::view-transition-old(dynamic-element) {
  animation: 
    move-to-center var(--duration) var(--easing) both,
    fade-out var(--duration) var(--easing) both;
}

::view-transition-new(dynamic-element) {
  animation: 
    move-from-center var(--duration) var(--easing) 0.1s both,
    fade-in var(--duration) var(--easing) 0.1s both;
}
```

```javascript
// Динамическое управление переходами
function dynamicTransition(elementId, config = {}) {
  const {
    duration = 0.4,
    easing = 'ease',
    delay = 0
  } = config;
  
  // Установка CSS-переменных
  document.documentElement.style.setProperty('--duration', `${duration}s`);
  document.documentElement.style.setProperty('--easing', easing);
  document.documentElement.style.setProperty('--delay', `${delay}s`);
  
  return document.startViewTransition(() => {
    // Изменение DOM
    document.getElementById(elementId).classList.toggle('active');
  });
}
```

### Условные переходы

```javascript
// Переходы на основе условий
function conditionalTransition(condition, elementId) {
  if (condition) {
    // Используем view transition
    document.startViewTransition(() => {
      document.getElementById(elementId).classList.toggle('expanded');
    });
  } else {
    // Используем обычные CSS-переходы
    document.getElementById(elementId).classList.toggle('expanded');
  }
}
```

### Сложные анимации с transform

```css
.complex-element {
  view-transition-name: complex-transform;
}

::view-transition-group(complex-transform) {
  isolation: isolate;
}

::view-transition-old(complex-transform) {
  animation: complex-exit 0.5s cubic-bezier(0.22, 0.61, 0.36, 1) both;
}

::view-transition-new(complex-transform) {
  animation: complex-enter 0.5s cubic-bezier(0.22, 0.61, 0.36, 1) 0.1s both;
}

@keyframes complex-exit {
  0% {
    transform: scale(1) rotate(0) translateX(0);
    opacity: 1;
  }
  100% {
    transform: scale(0.8) rotate(5deg) translateX(50px);
    opacity: 0;
  }
}

@keyframes complex-enter {
  0% {
    transform: scale(0.8) rotate(-5deg) translateX(-50px);
    opacity: 0;
  }
  100% {
    transform: scale(1) rotate(0) translateX(0);
    opacity: 1;
  }
}
```

## Совместимость и резервные варианты

### Проверка поддержки

```javascript
// Проверка поддержки View Transitions API
const supportsViewTransitions = document.startViewTransition;

if (supportsViewTransitions) {
  // Используем View Transitions API
  function performTransition(updateCallback) {
    document.startViewTransition(updateCallback);
  }
} else {
  // Резервный вариант с традиционными анимациями
  function performTransition(updateCallback) {
    updateCallback();
    // Дополнительная логика для анимации, если необходимо
  }
}
```

### Постепенное улучшение

```css
/* Базовые стили без view transitions */
.content {
  transition: opacity 0.3s ease;
}

.content.hidden {
  opacity: 0;
  display: none;
}

/* Современные стили с view transitions */
@supports (view-transition-name: slide) {
  .slide-element {
    view-transition-name: slide;
  }
  
  ::view-transition-old(slide) {
    animation: slide-out 0.3s ease both;
  }
  
  ::view-transition-new(slide) {
    animation: slide-in 0.3s ease 0.1s both;
  }
}
```

## Производительность и оптимизация

### Оптимизация view transitions

```css
/* Оптимизированные стили для view transitions */
.optimized-element {
  view-transition-name: optimized;
  /* Используем свойства, которые не вызывают перерисовку */
  contain: layout style paint;
}

::view-transition-group(optimized) {
  isolation: isolate;
  /* Активация аппаратного ускорения */
  transform: translateZ(0);
}

::view-transition-old(optimized),
::view-transition-new(optimized) {
  /* Используем transform и opacity для анимации */
  animation-duration: 0.3s;
  /* Указываем, что эти свойства будут изменяться */
  will-change: transform, opacity;
}
```

### Управление сложностью

```javascript
// Оптимизация сложных переходов
function optimizedTransition(updateFn, options = {}) {
  const { 
    duration = 0.3, 
    skipTransition = false 
  } = options;
  
  // Пропускаем анимацию для быстрых последовательных действий
  if (skipTransition) {
    updateFn();
    return Promise.resolve();
  }
  
  return document.startViewTransition(updateFn);
}
```

## Примеры из промышленной разработки

### SPA-навигация с view transitions

```css
/* Стили для SPA с view transitions */
.page {
  view-transition-name: page;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  min-height: 100vh;
}

::view-transition-group(page) {
  isolation: isolate;
}

::view-transition-old(page) {
  animation: exit-to-left 0.3s ease-out both;
}

::view-transition-new(page) {
  animation: enter-from-right 0.3s ease-out both;
}

@keyframes exit-to-left {
  from { transform: translateX(0); }
  to { transform: translateX(-40px); opacity: 0; }
}

@keyframes enter-from-right {
  from { transform: translateX(40px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
```

```javascript
// Управление навигацией в SPA
class ViewTransitionRouter {
  constructor() {
    this.currentPage = null;
  }
  
  async navigateTo(viewName) {
    const updateFunction = () => {
      // Скрытие текущей страницы
      if (this.currentPage) {
        document.getElementById(this.currentPage).hidden = true;
      }
      
      // Показ новой страницы
      document.getElementById(viewName).hidden = false;
      this.currentPage = viewName;
    };
    
    // Запуск view transition
    const transition = document.startViewTransition(updateFunction);
    
    // Дополнительная логика после завершения перехода
    await transition.finished;
    console.log(`Transition to ${viewName} completed`);
  }
}

// Использование
const router = new ViewTransitionRouter();
document.getElementById('nav-home').addEventListener('click', () => {
  router.navigateTo('home');
});
```

### Детализация элементов с анимацией

```css
/* Анимация при детализации элемента */
.list-item {
  view-transition-name: list-item;
  cursor: pointer;
}

.detail-view {
  view-transition-name: detail-view;
}

/* Анимация преобразования элемента в детализированный вид */
::view-transition-group(list-item),
::view-transition-group(detail-view) {
  isolation: isolate;
}

::view-transition-old(list-item) {
  animation: transform-to-detail 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

::view-transition-new(detail-view) {
  animation: expand-from-list 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

@keyframes transform-to-detail {
  0% {
    transform: scale(1) translate(0, 0);
    z-index: 1;
  }
  100% {
    transform: scale(0.8) translate(50px, -50px);
    z-index: 1;
    opacity: 0;
  }
}

@keyframes expand-from-list {
  0% {
    transform: scale(0.8) translate(-50px, 50px);
    opacity: 0;
  }
  100% {
    transform: scale(1) translate(0, 0);
    opacity: 1;
  }
}
```

## Лучшие практики

### 1. Использование семантических имен

```css
/* Хорошо: понятные имена для view transitions */
.header {
  view-transition-name: page-header;
}

.main-content {
  view-transition-name: content-area;
}

.sidebar {
  view-transition-name: navigation-sidebar;
}

/* Избегайте: неописательные имена */
.generic {
  view-transition-name: t1;
}
```

### 2. Учет доступности

```css
/* Уважение к предпочтениям пользователя по анимации */
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
  }
}
```

### 3. Тестирование производительности

```javascript
// Функция для измерения производительности переходов
function measureTransitionPerformance(transitionFn) {
  const startTime = performance.now();
  
  const transition = document.startViewTransition(transitionFn);
  
  transition.finished.then(() => {
    const endTime = performance.now();
    const duration = endTime - startTime;
    console.log(`Transition took ${duration} milliseconds`);
  });
  
  return transition;
}
```

## Ограничения и подводные камни

### 1. Совместимость с браузерами

CSS View Transitions API — экспериментальная функция, которая пока поддерживается не во всех браузерах. Всегда проверяйте совместимость и предоставляйте резервные варианты.

### 2. Ограничения на анимируемые свойства

Не все CSS-свойства могут быть анимированы с помощью View Transitions API. Используйте свойства, которые не вызывают перерисовку (transform, opacity).

### 3. Сложность синхронизации

Сложные переходы могут потребовать синхронизации между несколькими элементами, что требует тщательного планирования.

## Заключение

CSS View Transitions API — это мощная функция для создания плавных и вовлекающих переходов между разными состояниями интерфейса. Она особенно полезна для SPA-приложений, где важно обеспечить плавную навигацию между представлениями.

При правильном использовании View Transitions API может значительно улучшить пользовательский опыт, обеспечивая более плавные и профессиональные переходы между разными частями приложения.

## См. также

- [[CSS Animations]]
- [[CSS Transitions]]
- [[Scroll-Driven Animations]]
- [[Animation Composition]]
- [[Modern CSS Animations]]
- [[CSS Animation Performance]]
- [[Advanced CSS Animations]]
- [[CSS Custom Properties]]
- [[CSS Containment]]
- [[CSS Transformations]]
- [[CSS Keyframes]]
- [[CSS Timing Functions]]
- [[Web Animations API]]
- [[CSS Architecture Patterns]]
- [[Frontend Performance]]

## Дополнительные ресурсы

- [MDN Web Docs: View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)
- [CSS Working Group: CSS View Transitions Module](https://www.w3.org/TR/css-view-transitions-1/)
- [Web.dev: View Transitions](https://web.dev/view-transitions/)
- [Chrome Developers: View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions/)