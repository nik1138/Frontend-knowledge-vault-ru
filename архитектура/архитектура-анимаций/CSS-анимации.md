---
aliases: ["CSS Анимации", "Анимации CSS", "CSS Transition", "CSS Animation"]
tags: ["#css", "#анимации", "#frontend", "#производительность", "#архитектура"]
---

# CSS-анимации

CSS-анимации представляют собой мощный инструмент для создания плавных визуальных переходов и анимированных эффектов без использования JavaScript. Они обеспечивают высокую производительность за счет использования GPU и являются основой современной архитектуры анимаций в фронтенд-разработке.

## Основные понятия

CSS-анимации включают в себя несколько ключевых свойств и техник:

- `transition` - плавные переходы между состояниями
- `animation` - более сложные анимации с ключевыми кадрами
- `transform` - преобразования элементов без влияния на производительность
- `opacity` - изменения прозрачности для плавных эффектов

## Transition

Свойство `transition` позволяет создавать плавные переходы между значениями CSS-свойств:

```css
.button {
  background-color: #007bff;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}
```

### Параметры transition

- `transition-property` - указывает, какое свойство анимировать
- `transition-duration` - продолжительность анимации
- `transition-timing-function` - функция временной шкалы (easing)
- `transition-delay` - задержка перед началом анимации

## Animation

Для более сложных анимаций используется свойство `animation` с ключевыми кадрами:

```css
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

.slide-in-element {
  animation: slideIn 0.5s ease-out forwards;
}
```

### Параметры animation

- `animation-name` - имя анимации (определенное через @keyframes)
- `animation-duration` - продолжительность анимации
- `animation-timing-function` - функция временной шкалы
- `animation-delay` - задержка перед началом
- `animation-iteration-count` - количество повторений (infinite для бесконечных)
- `animation-direction` - направление анимации
- `animation-fill-mode` - состояние до и после анимации
- `animation-play-state` - управление воспроизведением (running/paused)

## Оптимизация производительности

Для достижения максимальной производительности при работе с CSS-анимациями важно:

1. **Использовать свойства, не вызывающие relayout и repaint**
   - `transform` и `opacity` наиболее эффективны
   - Избегать анимации свойств, влияющих на layout (width, height, margin, padding, top, left и др.)

2. **Использовать `will-change` для подготовки к анимации**
```css
.element {
  will-change: transform, opacity;
}
```

3. **Повышать элементы на отдельный слой при необходимости**
```css
.element {
  transform: translateZ(0); /* или translate3d(0,0,0) */
}
```

## Практические рекомендации

### 1. Структура CSS для анимаций

Для лучшей архитектуры рекомендуется выделять анимации в отдельные блоки:

```css
/* Анимации */
:root {
  --animation-duration-fast: 0.15s;
  --animation-duration-normal: 0.3s;
  --animation-duration-slow: 0.5s;
  --animation-easing-standard: cubic-bezier(0.4, 0.0, 0.2, 1);
  --animation-easing-accelerate: cubic-bezier(0.4, 0.0, 1, 1);
  --animation-easing-decelerate: cubic-bezier(0.0, 0.0, 0.2, 1);
}

/* Переходы */
.transition-fast {
  transition: all var(--animation-duration-fast) var(--animation-easing-standard);
}

.transition-normal {
  transition: all var(--animation-duration-normal) var(--animation-easing-standard);
}

/* Анимации */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-in {
  animation: fadeIn var(--animation-duration-normal) var(--animation-easing-standard);
}
```

### 2. Адаптивные анимации

С учетом российских реалий 2025 года, необходимо учитывать:

- Возможность отключения анимаций для пользователей с заболеваниями (например, эпилепсией)
- Различные устройства и производительность
- Снижение анимаций при слабом соединении

```css
/* Отключение анимаций для пользователей с ограничениями */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 3. Повышение доступности

Анимации должны быть доступны для всех пользователей:

- Обеспечить возможность отключения анимаций
- Использовать достаточный контраст для анимированных элементов
- Не использовать только цвет для передачи информации в анимациях

## Современные возможности CSS

### View Transitions API

Новый API, который позволяет создавать сложные переходы между представлениями:

```css
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}

::view-transition-old(slide-element) {
  animation: 90ms cubic-bezier(0.4, 0.0, 1, 1) both fade-out;
}

::view-transition-new(slide-element) {
  animation: 210ms cubic-bezier(0.0, 0.0, 0.2, 1) both fade-in;
}
```

### Scroll-driven animations

CSS-анимации, управляемые прокруткой:

```css
.scrolling-element {
  animation: slide 1s linear;
  animation-timeline: scroll();
}
```

## Лучшие практики в российском контексте 2025 года

1. **Соответствие требованиям доступности**
   - Обязательное соблюдение ГОСТ Р 52872-2019 и рекомендаций WCAG 2.1
   - Учет особенностей российского законодательства в области ИТ-доступности

2. **Производительность на слабых устройствах**
   - Оптимизация для бюджетных устройств, популярных в России
   - Учет различий в производительности между устройствами разных брендов

3. **Локализация и культурные особенности**
   - Использование анимаций, соответствующих культурным ожиданиям российской аудитории
   - Учет особенностей восприятия визуальной информации в российской культуре

## Связь с другими архитектурными компонентами

CSS-анимации тесно связаны с другими архитектурными элементами фронтенда:

- [[Архитектура интерфейсов]] - анимации как часть пользовательского опыта
- [[CSS-архитектура]] - структурирование анимаций в CSS-фреймворке
- [[Доступность]] - обеспечение доступности анимаций
- [[Производительность фронтенда]] - оптимизация анимаций для производительности
- [[JavaScript-анимации]] - взаимодействие с JS-анимациями
- [[Библиотеки анимаций]] - использование CSS в библиотеках
- [[Тестирование анимаций]] - проверка корректности CSS-анимаций

## Заключение

CSS-анимации являются важной частью современной архитектуры фронтенд-приложений. Правильное использование CSS-анимаций позволяет создавать плавные, доступные и производительные интерфейсы, что особенно актуально для российского рынка с его разнообразием устройств и пользовательских потребностей.