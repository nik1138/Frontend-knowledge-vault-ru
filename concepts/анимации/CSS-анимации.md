---
aliases: [CSS Animations, Анимации CSS, CSS Transition, CSS Transform]
tags: [css, animation, frontend, web-development]
---

# CSS-анимации

CSS-анимации позволяют создавать плавные переходы и анимационные эффекты без использования JavaScript. Они обеспечивают отличную производительность и простоту реализации для базовых анимационных эффектов.

## Основные свойства CSS-анимаций

### transition
Свойство `transition` позволяет плавно изменять значения CSS-свойств при изменении состояния элемента.

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

### transform
Свойство `transform` позволяет изменять форму, размер и положение элемента без влияния на окружающие элементы.

```css
.element {
  transform: translateX(100px) rotate(45deg) scale(1.2);
}
```

### animation
Свойство `animation` позволяет создавать сложные анимации с использованием ключевых кадров.

```css
@keyframes slideIn {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in-element {
  animation: slideIn 0.5s ease-out forwards;
}
```

## Типы CSS-анимаций

### 1. Плавные переходы (Transitions)
Используются для плавного изменения значений CSS-свойств при изменении состояния элемента (например, при наведении курсора).

```css
.card {
  background-color: white;
  border: 1px solid #ddd;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card:hover {
  background-color: #f8f9fa;
  border-color: #007bff;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  transform: translateY(-2px);
}
```

### 2. Ключевые кадры (Keyframes)
Позволяют создавать более сложные анимации с несколькими промежуточными состояниями.

```css
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

.bounce-element {
  animation: bounce 1s infinite;
}
```

### 3. Анимации загрузки (Spinners)
Часто используемые в веб-приложениях для индикации процесса загрузки.

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
```

## Продвинутые техники

### Стробоскопические анимации
Используются для создания эффектов мерцания или стробоскопа.

```css
@keyframes strobe {
  0%, 50%, 100% {
    opacity: 1;
  }
  25%, 75% {
    opacity: 0.2;
  }
}

.strobe-element {
  animation: strobe 1s infinite;
}
```

### Последовательные анимации
Анимации, которые срабатывают последовательно с задержкой.

```css
.item:nth-child(1) { animation-delay: 0.1s; }
.item:nth-child(2) { animation-delay: 0.2s; }
.item:nth-child(3) { animation-delay: 0.3s; }
.item:nth-child(4) { animation-delay: 0.4s; }
.item:nth-child(5) { animation-delay: 0.5s; }
```

## Практические рекомендации

### 1. Оптимизация производительности
- Используйте свойства `transform` и `opacity` для анимаций, так как они могут быть аппаратно ускорены
- Избегайте анимации свойств, которые вызывают перерисовку (например, `width`, `height`, `margin`)

```css
/* Хорошо - использует transform */
.optimized-animation {
  animation: slideIn 0.5s ease-out;
  transform: translateZ(0); /* Принудительно включает аппаратное ускорение */
}

/* Плохо - вызывает перерисовку */
.not-optimized {
  animation: widthChange 0.5s ease-out;
}
```

### 2. Пользовательские настройки
Учитывайте предпочтения пользователей, такие как отключение анимаций.

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 3. Семантические анимации
Анимации должны улучшать восприятие интерфейса, а не отвлекать от него.

> [!tip]
> Используйте анимации умеренно и только там, где они действительно улучшают пользовательский опыт.

## Сравнение с другими подходами

CSS-анимации отличаются от JavaScript-анимаций своей простотой реализации и производительностью для базовых эффектов. Однако для сложной логики анимации и синхронизации с событиями JavaScript может быть более подходящим решением.

Сравнение с [[JavaScript-анимации]] и [[Анимации-в-React]] показывает, что каждый подход имеет свои области применения.

## Заключение

CSS-анимации - это мощный инструмент для создания привлекательного и интерактивного пользовательского интерфейса. Они обеспечивают баланс между простотой реализации и производительностью, особенно для базовых анимационных эффектов.

Для более сложных сценариев анимации можно комбинировать CSS с JavaScript, используя [[JavaScript-анимации]] или [[Анимации-в-React]] в зависимости от используемого фреймворка.

## См. также
- [[JavaScript-анимации]]
- [[Анимации-в-React]]
- [[Анимации-в-Vue]]
- [[Анимации-в-Svelte]]