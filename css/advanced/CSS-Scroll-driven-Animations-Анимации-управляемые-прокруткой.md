---
aliases: [Scroll-driven Animations, CSS Scroll-driven Animations, Анимации при прокрутке]
tags: [css, animations, advanced]
---

# CSS Scroll-driven Animations - Анимации, управляемые прокруткой

## Введение

CSS Scroll-driven Animations - это современная функция CSS и JavaScript, которая позволяет запускать и управлять анимациями на основе прокрутки пользователя. Эта технология позволяет создавать более плавные и синхронизированные с прокруткой эффекты без необходимости писать сложный JavaScript-код для отслеживания прокрутки.

## Понимание Scroll-driven Animations

### Традиционный подход vs Scroll-driven Animations

Ранее для создания анимаций, связанных с прокруткой, разработчики полагались на JavaScript-обработчики событий `scroll`, что приводило к проблемам с производительностью из-за частых вызовов функций при прокрутке. Scroll-driven Animations позволяет браузеру оптимизировать эти анимации на уровне CSS, что значительно улучшает производительность.

### Основные концепции

- **Scroll Timeline** - временная шкала, привязанная к прокрутке элемента
- **Animation Timeline** - временная шкала, управляющая прогрессом анимации
- **Scroll-driven animation** - анимация, чей прогресс зависит от положения прокрутки

## CSS-свойства Scroll-driven Animations

### view-timeline

Свойство `view-timeline` определяет временную шкалу, основанную на видимости элемента в определенном контейнере прокрутки.

```css
.fading-element {
  view-timeline: --fade-in;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fading-element {
  animation: fadeIn linear;
  animation-timeline: --fade-in;
}
```

### scroll-timeline

Свойство `scroll-timeline` определяет временную шкалу, основанную на прокрутке конкретного элемента.

```css
.scrolling-container {
  scroll-timeline: --slide-x;
  /* Или с указанием оси */
  scroll-timeline: --slide-x block;
}

@keyframes slideRight {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.slide-element {
  animation: slideRight linear;
  animation-timeline: --slide-x;
}
```

### animation-timeline

Свойство `animation-timeline` связывает анимацию с определенной временной шкалой (scroll или view timeline).

```css
.parallax-element {
  view-timeline: --parallax;
  animation: parallax linear;
  animation-timeline: --parallax;
}

@keyframes parallax {
  from { transform: translateY(0%) scale(1); }
  to { transform: translateY(-50%) scale(1.5); }
}
```

## Практические примеры

### Пример 1: Появление элементов при прокрутке

Создание анимации появления элементов по мере их прокручивания в область видимости:

```html
<div class="container">
  <div class="card reveal">Карточка 1</div>
  <div class="card reveal">Карточка 2</div>
  <div class="card reveal">Карточка 3</div>
</div>
```

```css
.container {
  height: 200vh; /* Длинный контейнер для прокрутки */
  padding: 20px;
}

.reveal {
  opacity: 0;
  transform: translateY(50px);
  animation: reveal linear;
  animation-fill-mode: forwards;
  view-timeline: --reveal-block block;
}

@keyframes reveal {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Элемент начинает анимацию, когда входит в область видимости */
.reveal {
  animation-timeline: --reveal-block;
  view-timeline: --reveal-block;
  view-timeline-axis: block;
}
```

### Пример 2: Параллакс-эффект

Создание параллакс-эффекта с использованием scroll-driven animations:

```html
<div class="parallax-container">
  <div class="parallax-background"></div>
  <div class="content">
    <h1>Содержание страницы</h1>
    <p>Длинный текст для прокрутки...</p>
  </div>
</div>
```

```css
.parallax-container {
  height: 200vh;
  position: relative;
  overflow-y: auto;
  scroll-timeline: --parallax-timeline block;
}

.parallax-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 150%;
  background: url('background.jpg') center/cover;
  animation: parallax linear;
  animation-timeline: --parallax-timeline;
}

@keyframes parallax {
  from { transform: translateY(0%); }
  to { transform: translateY(-25%); }  /* Фон движется медленнее, создавая параллакс */
}

.content {
  position: relative;
  z-index: 1;
  padding: 2rem;
}
```

### Пример 3: Прогресс-бар прокрутки

Создание визуального индикатора прогресса прокрутки страницы:

```html
<div class="progress-container">
  <div class="progress-bar"></div>
</div>
<div class="content">
  <h1>Заголовок</h1>
  <p>Длинный текст содержимого...</p>
</div>
```

```css
body {
  scroll-timeline: --body-scroll block;
}

.progress-bar {
  animation: fill linear;
  animation-timeline: --body-scroll;
  position: fixed;
  top: 0;
  left: 0;
  height: 4px;
  background: linear-gradient(90deg, #ff0000, #00ff00);
  width: 0%;
  z-index: 1000;
}

@keyframes fill {
  from { width: 0%; }
  to { width: 100%; }
}
```

## Параметры временных шкал

### view-timeline-name и view-timeline-axis

```css
.element {
  view-timeline-name: --my-timeline;
  view-timeline-axis: block;  /* или inline */
}
```

### view-timeline-inset

Позволяет настроить смещение начала и конца временной шкалы:

```css
.element {
  view-timeline: --fade-timeline;
  view-timeline-inset: 200px 200px;  /* Начало и конец смещены на 200px */
}
```

### scroll-timeline-name, scroll-timeline-axis и scroll-timeline-direction

```css
.container {
  scroll-timeline-name: --my-scroll-timeline;
  scroll-timeline-axis: block;  /* или inline */
  scroll-timeline-direction: normal;  /* или reverse, auto */
}
```

## Продвинутые техники

### Связывание нескольких анимаций

Можно синхронизировать несколько анимаций с одной временной шкалой:

```css
.scrolling-container {
  scroll-timeline: --master-timeline block;
}

.element-1 {
  animation: slide-in linear;
  animation-timeline: --master-timeline;
}

.element-2 {
  animation: fade-in linear;
  animation-timeline: --master-timeline;
}

.element-3 {
  animation: scale-up linear;
  animation-timeline: --master-timeline;
}
```

### Сложные ключевые кадры для Scroll-driven анимаций

```css
.staggered-animation {
  view-timeline: --stagger;
  animation: complex-behavior linear;
  animation-timeline: --stagger;
}

@keyframes complex-behavior {
  0% {
    transform: translateX(-100%) rotate(-10deg);
    opacity: 0;
  }
  25% {
    transform: translateX(0) rotate(0deg);
    opacity: 1;
  }
  75% {
    transform: scale(1.1) rotate(5deg);
  }
  100% {
    transform: scale(1) rotate(0deg);
  }
}
```

## Производительность и оптимизация

### Оптимизированные свойства

Для лучшей производительности используйте анимируемые свойства, которые не вызывают перерисовку (repaint):

- `transform` (все его вариации: translate, scale, rotate)
- `opacity`
- `filter` (некоторые значения)

### Оптимизация рендеринга

```css
.scroll-driven-element {
  will-change: transform, opacity;  /* Указывает браузеру, что свойства будут часто меняться */
  /* Или используйте contain для изоляции элемента */
  contain: layout style paint;
}
```

## Поддержка браузерами

На момент 2023 года Scroll-driven Animations:

- Поддерживается в Chrome 115+ (с флагом или без)
- Находится в разработке в Firefox и Safari
- Для полной поддержки может потребоваться polyfill

## Совместимость с JavaScript

Scroll-driven Animations можно комбинировать с JavaScript для дополнительного контроля:

```javascript
// Получение анимации, связанной с временной шкалой
const element = document.querySelector('.scroll-driven-element');
const animation = element.getAnimations().find(anim => anim.timeline === element.attributeStyleMap.get('animation-timeline'));

// Контроль воспроизведения
animation.pause();
animation.play();
```

## Лучшие практики

1. **Используйте оптимизированные свойства**: Для анимаций прокрутки используйте только свойства, которые не вызывают перерисовку (transform, opacity).

2. **Контролируйте количество**: Не создавайте слишком много scroll-driven анимаций на одной странице, чтобы не перегружать браузер.

3. **Обеспечьте резервные стили**: Для браузеров без поддержки используйте резервные стили с традиционными анимациями или JavaScript.

4. **Тестируйте производительность**: Тестируйте производительность на слабых устройствах, особенно при использовании сложных анимаций.

5. **Учитывайте доступность**: Предоставьте опцию отключения анимаций для пользователей с включенным `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  .scroll-driven-element {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
  }
}
```

## Ограничения и соображения

1. **Ограниченная поддержка браузерами**: На момент внедрения технологии поддержка ограничена и может потребовать флаги браузера.

2. **Сложность отладки**: Отладка scroll-driven анимаций может быть сложнее, чем традиционных анимаций.

3. **Поведение на мобильных устройствах**: Поведение может отличаться на различных мобильных платформах.

## Заключение

CSS Scroll-driven Animations открывают новые возможности для создания сложных анимационных эффектов, управляемых прокруткой, с высокой производительностью. Правильное их использование позволяет создавать более плавные и отзывчивые пользовательские интерфейсы, синхронизированные с действиями пользователя. По мере роста поддержки браузерами, эта технология станет стандартным способом создания анимаций, связанных с прокруткой.

## Смежные темы

- [[animations]]
- [[advanced-techniques]]
- [[performance]]
- [[accessibility]]

#css #animations #scroll-driven #frontend #performance