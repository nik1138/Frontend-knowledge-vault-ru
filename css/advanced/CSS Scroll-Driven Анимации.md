---
aliases: [CSS Scroll-Driven Animation, Скролл-анимации, Scroll-Driven Animation]
tags: [css, animations, scroll-driven, frontend]
---

# CSS Scroll-Driven Анимации

## Обзор

CSS Scroll-Driven Анимации - это современная технология, позволяющая синхронизировать CSS анимации с прокруткой страницы. Эта функция позволяет создавать захватывающие визуальные эффекты, которые активируются или изменяются в зависимости от позиции прокрутки, без необходимости в JavaScript. Это открывает новые возможности для создания интерактивных веб-страниц и улучшения пользовательского опыта.

Scroll-driven анимации позволяют разработчикам использовать скролл как триггер для анимаций, что особенно полезно для создания эффектов всплывающих окон, параллакса, прогрессивного раскрытия контента и других интерактивных элементов.

## Основные понятия

### Scroll-Timeline

Scroll-timeline - это основной механизм, который позволяет связать анимацию с прокруткой. Он определяет, как анимация должна изменяться в зависимости от позиции прокрутки на странице. Scroll-timeline можно определить как на элементе, так и глобально для всего документа.

```css
.element {
  animation-timeline: scroll();
}
```

### View-Timeline

View-timeline - это более продвинутая форма scroll-timeline, которая позволяет привязывать анимации к видимости элемента на экране. Она активируется, когда элемент входит в область просмотра (viewport), что делает ее идеальной для анимаций при прокрутке.

```css
.element {
  animation-timeline: view();
}
```

## Синтаксис и свойства

### animation-timeline

Свойство `animation-timeline` заменяет традиционное свойство `animation-duration` и определяет, какая временная шкала будет использоваться для анимации. Значением может быть:

- `auto` - стандартное поведение анимации
- `scroll()` - привязка к прокрутке
- `view()` - привязка к видимости элемента
- `none` - отключение временной шкалы

```css
.hero-animation {
  animation-name: slide-in;
  animation-timeline: view();
  animation-fill-mode: both;
}
```

### scroll-timeline-name

Это свойство позволяет задать имя для scroll-timeline, которое затем можно использовать в других элементах:

```css
.container {
  scroll-timeline-name: my-scroll;
}

.animated-element {
  animation-timeline: my-scroll;
}
```

### view-timeline

Свойство `view-timeline` позволяет точно настроить, когда анимация должна начинаться и заканчиваться в зависимости от видимости элемента:

```css
.card {
  view-timeline: --custom-name block;
  animation-timeline: --custom-name;
}
```

## Практические примеры

### Простая анимация появления при прокрутке

```css
.fade-in-element {
  opacity: 0;
  transform: translateY(50px);
  animation-name: fadeInUp;
  animation-timeline: view();
  animation-fill-mode: both;
}

@keyframes fadeInUp {
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

### Анимация прогрессивного раскрытия

```css
.progressive-reveal {
  animation-name: reveal;
  animation-timeline: view();
  animation-fill-mode: both;
}

@keyframes reveal {
  0% {
    opacity: 0;
    width: 0;
  }
  100% {
    opacity: 1;
    width: 100%;
  }
}
```

### Сложная анимация с несколькими ключевыми кадрами

```css
.complex-scroll-animation {
  animation-name: complex-animation;
  animation-timeline: view();
  animation-fill-mode: both;
}

@keyframes complex-animation {
  0% {
    opacity: 0;
    transform: scale(0.5) rotate(0deg);
  }
  50% {
    opacity: 0.5;
    transform: scale(1.1) rotate(180deg);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(360deg);
  }
}
```

## Типы Scroll-Timeline

### Scroll-Timeline по осям

Scroll-timeline может быть привязан к различным осям прокрутки:

- `scroll()` - по умолчанию привязывается к вертикальной прокрутке
- `scroll(y)` - явно указывает вертикальную прокрутку
- `scroll(x)` - указывает горизонтальную прокрутку

```css
.horizontal-scroll {
  animation-timeline: scroll(x);
}

.vertical-scroll {
  animation-timeline: scroll(y);
}
```

### Custom Scroll-Timeline

Можно создавать пользовательские scroll-timeline с дополнительными параметрами:

```css
.custom-timeline {
  scroll-timeline: --custom-timeline y;
  animation-timeline: --custom-timeline;
}

.element {
  animation-name: my-animation;
  animation-timeline: --custom-timeline;
}
```

## Параметры View-Timeline

### View-Timeline Axis

Ось, вдоль которой отслеживается видимость элемента:

- `block` - по направлению потока (обычно вертикально)
- `inline` - по линии текста (обычно горизонтально)

```css
.timeline-block {
  view-timeline: --timeline block;
}

.timeline-inline {
  view-timeline: --timeline inline;
}
```

### View-Timeline Inset

Позволяет настроить смещение начала и конца временной шкалы:

```css
.timeline-inset {
  view-timeline: --timeline block;
  view-timeline-inset: 100px 200px; /* начало: 100px до входа, конец: 200px после выхода */
}
```

## Поддержка браузерами

CSS Scroll-Driven Анимации - это относительно новая технология, и поддержка браузерами варьируется:

- Chrome 115+ (с поддержкой view-timeline)
- Firefox - частичная поддержка
- Safari - ограниченная поддержка

Для обеспечения совместимости рекомендуется использовать проверку поддержки:

```css
@supports (animation-timeline: view()) {
  .supported-element {
    animation-timeline: view();
  }
}

@supports not (animation-timeline: view()) {
  .fallback-element {
    /* Резервная анимация с JavaScript */
    opacity: 1;
    transform: none;
  }
}
```

## Лучшие практики

### 1. Плавность анимаций

При создании scroll-driven анимаций важно обеспечить их плавность, чтобы не вызывать дискомфорта у пользователя:

```css
.smooth-scroll-animation {
  animation-name: smooth-transition;
  animation-timeline: view();
  animation-fill-mode: both;
  will-change: transform, opacity;
}

@keyframes smooth-transition {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### 2. Оптимизация производительности

Использование свойств `transform` и `opacity` предпочтительно, так как они могут быть оптимизированы браузером:

```css
.optimized-animation {
  animation-name: optimized-move;
  animation-timeline: view();
  animation-fill-mode: both;
}

@keyframes optimized-move {
  from {
    opacity: 0;
    transform: translate3d(0, 20px, 0);
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0, 0);
  }
}
```

### 3. Учет предпочтений пользователя

Важно учитывать системные настройки пользователя, такие как предпочтение уменьшения анимаций:

```css
@media (prefers-reduced-motion: reduce) {
  .scroll-animation {
    animation: none;
  }
}

@media (prefers-reduced-motion: no-preference) {
  .scroll-animation {
    animation-name: fadeIn;
    animation-timeline: view();
    animation-fill-mode: both;
  }
}
```

### 4. Тестирование на разных устройствах

Scroll-driven анимации могут по-разному работать на разных устройствах и с разными способами прокрутки (колесико мыши, трекпад, сенсорный экран), поэтому важно проводить тестирование:

```css
.responsive-scroll-animation {
  animation-name: responsive-appear;
  animation-timeline: view();
  animation-fill-mode: both;
  /* Дополнительные стили для обеспечения согласованности */
}
```

## Распространенные ошибки

### 1. Избыточные анимации

Не стоит использовать scroll-driven анимации для каждого элемента на странице, это может создать ощущение перегруженности:

```css
/* Плохо - слишком много анимаций */
.content p { animation-timeline: view(); }
.content img { animation-timeline: view(); }
.content div { animation-timeline: view(); }

/* Хорошо - стратегическое использование анимаций */
.hero-section { animation-timeline: view(); }
.key-features { animation-timeline: view(); }
```

### 2. Неправильные временные шкалы

Использование неподходящих временных шкал может привести к неожиданному поведению анимаций:

```css
/* Плохо - использование view-timeline для элемента, который всегда виден */
.always-visible { 
  animation-timeline: view(); /* Не имеет смысла */
}

/* Хорошо - использование подходящей временной шкалы */
.scroll-reveal { 
  animation-timeline: view(); /* Подходит для скрываемых элементов */
}
```

### 3. Отсутствие резервных решений

Всегда нужно предусматривать резервные решения для браузеров, не поддерживающих scroll-driven анимации:

```css
.element {
  /* Резервное решение */
  opacity: 1;
  transform: none;
}

@supports (animation-timeline: view()) {
  .element {
    opacity: 0;
    transform: translateY(20px);
    animation-name: reveal;
    animation-timeline: view();
    animation-fill-mode: both;
  }
}
```

## Продвинутые техники

### Анимация с несколькими элементами

Можно создавать синхронизированные анимации для нескольких элементов:

```css
.timeline-container {
  scroll-timeline-name: master-timeline;
}

.item-1 {
  animation-name: item-1-animation;
  animation-timeline: master-timeline;
}

.item-2 {
  animation-name: item-2-animation;
  animation-timeline: master-timeline;
  animation-delay: 0.2s; /* Смещение для последовательной анимации */
}

.item-3 {
  animation-name: item-3-animation;
  animation-timeline: master-timeline;
  animation-delay: 0.4s;
}
```

### Анимация с переменными CSS

Использование CSS-переменных позволяет создавать более гибкие анимации:

```css
.variable-animation {
  --start-opacity: 0;
  --end-opacity: 1;
  --start-transform: translateY(30px);
  --end-transform: translateY(0);
  animation-name: variable-keyframes;
  animation-timeline: view();
  animation-fill-mode: both;
}

@keyframes variable-keyframes {
  0% {
    opacity: var(--start-opacity);
    transform: var(--start-transform);
  }
  100% {
    opacity: var(--end-opacity);
    transform: var(--end-transform);
  }
}
```

## Отладка Scroll-Driven Анимаций

### Использование инструментов разработчика

Браузеры предоставляют инструменты для отладки scroll-driven анимаций:

1. Проверьте, правильно ли определены временные шкалы
2. Убедитесь, что элементы правильно реагируют на прокрутку
3. Проверьте, не возникает ли конфликтов между различными анимациями

### Обнаружение проблем производительности

Следите за производительностью при использовании scroll-driven анимаций:

```css
.performance-aware {
  animation-name: efficient-animation;
  animation-timeline: view();
  animation-fill-mode: both;
  will-change: transform;
  contain: layout style paint;
}
```

## Сравнение с JavaScript-анимациями

### Преимущества CSS Scroll-Driven Анимаций

1. **Производительность** - CSS-анимации обычно более эффективны, чем JavaScript-анимации
2. **Простота реализации** - меньше кода для достижения эффекта
3. **Синхронизация с прокруткой** - более точная привязка к прокрутке
4. **Автоматическая адаптация** - автоматически адаптируется к скорости прокрутки

### Когда использовать JavaScript

JavaScript-анимации все еще необходимы в следующих случаях:

1. Сложные логические условия
2. Интерактивные анимации (не только по прокрутке)
3. Анимации, требующие доступа к данным или состоянию приложения
4. Поддержка старых браузеров

## Будущее Scroll-Driven Анимаций

CSS Scroll-Driven Анимации продолжают развиваться, и в будущем ожидается:

1. Улучшенная поддержка браузерами
2. Дополнительные параметры настройки
3. Интеграция с другими современными CSS-функциями
4. Улучшенные инструменты разработчика

## Заключение

CSS Scroll-Driven Анимации открывают новые возможности для создания интерактивных и визуально привлекательных веб-страниц. Они позволяют синхронизировать анимации с прокруткой без необходимости в JavaScript, что улучшает производительность и упрощает реализацию.

При правильном использовании scroll-driven анимации могут значительно улучшить пользовательский опыт, делая интерфейс более живым и интерактивным. Однако важно использовать их умеренно и с учетом доступности, чтобы не создавать дискомфорт для пользователей.

Ключевые моменты для успешного использования:

1. Понимание основных концепций scroll-timeline и view-timeline
2. Соблюдение лучших практик производительности
3. Обеспечение доступности и совместимости
4. Тестирование на различных устройствах и браузерах

Scroll-driven анимации - это мощный инструмент в арсенале современного фронтенд-разработчика, который стоит изучить и внедрить в свои проекты.

## См. также

- [[CSS Анимации]]
- [[CSS Transform]]
- [[CSS Transitions]]
- [[CSS Performance]]
- [[Frontend Анимации]]
- [[CSS Viewport Units]]
- [[CSS Containment]]
- [[Web Animations API]]
- [[CSS Motion Path]]
- [[CSS Grid Layout]]
- [[CSS Flexbox]]
- [[CSS Variables]]
- [[CSS Media Queries]]
- [[CSS Pseudo-elements]]
- [[CSS Selectors]]

## Ключевые теги

#css #animations #scroll-driven #frontend #web-development #user-experience #performance #accessibility #modern-css #animation-timeline #view-timeline #scroll-timeline #css-animations #frontend-development #responsive-design