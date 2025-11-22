---
aliases: [CSS Transitions, Переходы CSS, CSS Animation Transitions]
tags: [css, animations, transitions, frontend]
---

# CSS-переходы: Полное руководство по теории и практике

## Введение в CSS-переходы

CSS-переходы (CSS Transitions) - это механизм, позволяющий плавно изменять значения CSS-свойств за определённый период времени. Они обеспечивают визуальную непрерывность между различными состояниями элементов, создавая более приятный пользовательский опыт.

Переходы особенно полезны для анимаций, вызванных взаимодействием пользователя с интерфейсом, например, при наведении курсора, клике или фокусировке на элементе.

## Основные свойства переходов

### transition-property

Свойство `transition-property` определяет, какие CSS-свойства будут анимироваться во время перехода. Можно указать одно или несколько свойств:

```css
.element {
  transition-property: background-color;
}

.multiple {
  transition-property: background-color, color, transform;
}

.all {
  transition-property: all; /* Анимирует все изменяемые свойства */
}
```

> [!tip] 
> Рекомендуется явно указывать свойства, которые нужно анимировать, вместо использования `all`, чтобы избежать непреднамеренных анимаций.

### transition-duration

Свойство `transition-duration` задает продолжительность перехода. Можно указать одно значение для всех свойств или разные значения для каждого свойства:

```css
.element {
  transition-duration: 0.3s;
}

.multiple {
  transition-duration: 0.3s, 0.5s; /* Для каждого свойства из transition-property */
}
```

### transition-timing-function

Свойство `transition-timing-function` определяет, как изменяются значения свойств во времени. Оно контролирует ускорение и замедление анимации:

```css
.linear {
  transition-timing-function: linear; /* Равномерная скорость */
}

.ease {
  transition-timing-function: ease; /* По умолчанию: медленно -> быстро -> медленно */
}

.ease-in {
  transition-timing-function: ease-in; /* Медленно в начале */
}

.ease-out {
  transition-timing-function: ease-out; /* Медленно в конце */
}

.ease-in-out {
  transition-timing-function: ease-in-out; /* Медленно в начале и в конце */
}

.cubic-bezier {
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); /* Пользовательская кривая */
}
```

### transition-delay

Свойство `transition-delay` задает задержку перед началом перехода:

```css
.delayed {
  transition-delay: 0.2s;
}
```

## Сокращенная запись transition

Все свойства перехода можно объединить в одном объявлении:

```css
.element {
  transition: property duration timing-function delay;
}

.example {
  transition: background-color 0.3s ease-in-out 0.1s;
}

.multiple-properties {
  transition: 
    background-color 0.3s ease,
    transform 0.5s cubic-bezier(0.4, 0, 0.2, 1),
    color 0.2s linear 0.1s;
}
```

## Примеры практических применений

### 1. Простой hover-эффект

```css
.button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.button:hover {
  background-color: #0056b3;
}
```

### 2. Анимация трансформации

```css
.card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.1);
}
```

### 3. Плавное изменение размеров

```css
.image-container {
  overflow: hidden;
  transition: height 0.4s ease;
}

.image-container.expanded {
  height: 300px;
}
```

## Типы анимируемых свойств

Не все CSS-свойства можно анимировать. Свойства, которые можно анимировать, включают:

- Цвета (color, background-color, border-color и т.д.)
- Числовые значения (width, height, margin, padding, font-size и т.д.)
- Трансформации (transform)
- Прозрачность (opacity)
- Тени (box-shadow, text-shadow)

> [!note] 
> Свойства, не поддерживающие анимацию, будут изменяться мгновенно без перехода.

## Функции времени (timing functions)

Функции времени определяют, как значения свойств изменяются в течение перехода. Они могут быть представлены как ключевые слова или кривые Безье:

### Ключевые слова

- `linear` - равномерная скорость изменения
- `ease` - медленное начало, быстрый серединный участок, медленное завершение
- `ease-in` - медленное начало
- `ease-out` - медленное завершение
- `ease-in-out` - медленное начало и завершение

### Кривые Безье

Для более точного контроля можно использовать `cubic-bezier()`:

```css
.custom-ease {
  transition-timing-function: cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
```

## Практические рекомендации

### 1. Оптимизация производительности

Для плавных анимаций и лучшей производительности рекомендуется анимировать только следующие свойства:

- `transform` (translate, scale, rotate)
- `opacity`

Эти свойства не вызывают перерисовки или перекомпоновки элементов, что улучшает производительность.

```css
/* Хорошо - анимирует свойства, не вызывающие перерисовку */
.optimized {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

/* Плохо - может вызвать перерисовку */
.not-optimized {
  transition: width 0.3s ease, height 0.3s ease, margin 0.3s ease;
}
```

### 2. Использование hardware acceleration

Для активации аппаратного ускорения можно использовать `transform` и `will-change`:

```css
.accelerated {
  transform: translateZ(0); /* Активирует аппаратное ускорение */
  will-change: transform; /* Предупреждает браузер о предстоящей анимации */
}
```

### 3. Плавность восприятия

Для создания естественных ощущений при анимации:

- Используйте `ease-out` для входящих анимаций
- Используйте `ease-in` для исходящих анимаций
- Продолжительность анимации должна быть в диапазоне 200-500 мс для интерактивных элементов

### 4. Доступность

Учитывайте пользователей с чувствительностью к движению:

```css
@media (prefers-reduced-motion: reduce) {
  * {
    transition-duration: 0.01ms !important;
    animation-duration: 0.01ms !important;
  }
}
```

## Продвинутые техники

### 1. Ступенчатые переходы

Для создания ступенчатых анимаций можно использовать `steps()`:

```css
.stepped {
  transition: left 1s steps(4, end); /* Разбивает анимацию на 4 ступени */
}
```

### 2. Каскадные анимации

Для создания каскадных анимаций элементов списка:

```css
.list-item {
  transition: transform 0.3s ease, opacity 0.3s ease;
  transition-delay: calc(var(--delay-index) * 0.1s);
}
```

### 3. Комбинирование с CSS-переменными

Использование CSS-переменных для гибкости:

```css
.interactive-element {
  --transition-duration: 0.3s;
  --transition-easing: ease;
  
  transition: 
    background-color var(--transition-duration) var(--transition-easing),
    transform var(--transition-duration) var(--transition-easing);
}
```

## События переходов

CSS предоставляет JavaScript-события для отслеживания переходов:

- `transitionstart` - начало перехода
- `transitionend` - завершение перехода
- `transitioncancel` - отмена перехода

```javascript
element.addEventListener('transitionend', function(event) {
  console.log('Transition finished for', event.propertyName);
});
```

## Распространенные ошибки

### 1. Анимация неанимируемых свойств

Анимация свойств, которые не поддерживают переходы, не даст ожидаемого результата.

### 2. Слишком длинные или короткие анимации

Слишком быстрые анимации могут быть незаметны, а слишком медленные - раздражать пользователя.

### 3. Использование all в transition-property

Использование `all` может привести к нежелательным анимациям:

```css
/* Плохо */
.bad {
  transition: all 0.3s ease;
}

/* Хорошо */
.good {
  transition: background-color 0.3s ease, transform 0.3s ease;
}
```

## Совместимость с браузерами

CSS-переходы поддерживаются во всех современных браузерах. Для поддержки старых браузеров можно использовать префиксы:

```css
.prefixed {
  -webkit-transition: all 0.3s ease;
  -moz-transition: all 0.3s ease;
  -o-transition: all 0.3s ease;
  transition: all 0.3s ease;
}
```

## Сравнение с CSS-анимациями

| Особенность | Transition | Animation |
|-------------|------------|-----------|
| Инициация | Изменение состояния | Автоматическая |
| Контроль | Простая | Сложная, с ключевыми кадрами |
| Повторение | Нет | Да |
| Направление | Вперед/назад | Вперед/назад, альтернирующее |

## Заключение

CSS-переходы - мощный инструмент для создания интерактивных и привлекательных веб-интерфейсов. Правильное использование переходов улучшает пользовательский опыт, делает интерфейс более живым и интуитивно понятным.

При разработке переходов важно учитывать производительность, доступность и эстетику. Следуя лучшим практикам, можно создавать плавные, быстрые и приятные анимации, которые улучшают восприятие интерфейса.

Для более сложных анимаций рекомендуется использовать [[CSS-анимации]], которые предоставляют больше возможностей по сравнению с переходами.

## См. также

- [[CSS-анимации]]
- [[CSS-transform]]
- [[CSS-performance]]
- [[Frontend-доступность]]
- [[CSS-layout]]
- [[CSS-selectors]]
- [[CSS-variables]]
- [[CSS-frameworks]]
- [[CSS-preprocessors]]
- [[CSS-methodology]]
- [[CSS-modern-features]]
- [[CSS-responsive]]
- [[CSS-styling]]
- [[CSS-basics]]
- [[CSS-components]]