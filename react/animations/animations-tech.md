---
tags: [react, animations, frontend, performance, accessibility]
aliases: [react-animations, react-animation-techniques]
---

# React Анимации: Техники и Подходы

## Введение

React анимации позволяют создавать интерактивные и привлекательные пользовательские интерфейсы. Существует множество подходов для реализации анимаций в React-приложениях, от простых CSS-анимаций до сложных библиотек с физикой пружин.

## CSS Анимации в React

CSS анимации остаются одним из самых простых способов добавления анимации в React-компоненты:

```css
.fade-in {
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
}

.fade-in.active {
  opacity: 1;
}
```

```jsx
import './styles.css';

function FadeInComponent({ isVisible }) {
  return (
    <div className={`fade-in ${isVisible ? 'active' : ''}`}>
      Содержимое с анимацией появления
    </div>
  );
}
```

Преимущества:
- Высокая производительность (анимации на GPU)
- Простота реализации
- Совместимость с существующими CSS-фреймворками

## JavaScript Анимации

JavaScript позволяет создавать более сложные и динамические анимации:

```jsx
import { useState, useEffect, useRef } from 'react';

function JSAnimationComponent() {
  const [position, setPosition] = useState(0);
  const elementRef = useRef(null);

  useEffect(() => {
    let animationFrame;
    
    function animate() {
      setPosition(prev => (prev + 1) % 300);
      animationFrame = requestAnimationFrame(animate);
    }
    
    animationFrame = requestAnimationFrame(animate);
    
    return () => cancelAnimationFrame(animationFrame);
  }, []);

  return (
    <div 
      ref={elementRef}
      style={{ 
        transform: `translateX(${position}px)`,
        transition: 'transform 0.1s linear'
      }}
    >
      Движущийся элемент
    </div>
  );
}
```

## React Transition Group

[[react-transition-group]] предоставляет компоненты для управления анимациями при монтировании/размонтировании:

```jsx
import { CSSTransition } from 'react-transition-group';

function TransitionComponent({ isVisible }) {
  return (
    <CSSTransition
      in={isVisible}
      timeout={300}
      classNames="fade"
      unmountOnExit
    >
      <div>Содержимое с анимацией</div>
    </CSSTransition>
  );
}
```

Соответствующий CSS:
```css
.fade-enter {
  opacity: 0;
}
.fade-enter-active {
  opacity: 1;
  transition: opacity 300ms;
}
.fade-exit {
  opacity: 1;
}
.fade-exit-active {
  opacity: 0;
  transition: opacity 300ms;
}
```

## Framer Motion

[[framer-motion]] - мощная библиотека для анимаций с физикой пружин:

```jsx
import { motion, AnimatePresence } from 'framer-motion';

function FramerMotionComponent({ isVisible }) {
  return (
    <AnimatePresence>
      {isVisible && (
        <motion.div
          initial={{ opacity: 0, scale: 0.8 }}
          animate={{ opacity: 1, scale: 1 }}
          exit={{ opacity: 0, scale: 0.8 }}
          transition={{ type: "spring", stiffness: 260, damping: 20 }}
        >
          Анимированный элемент
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## Анимированные Компоненты

Создание повторно используемых анимированных компонентов:

```jsx
import { motion } from 'framer-motion';

const AnimatedCard = ({ children, delay = 0 }) => (
  <motion.div
    initial={{ opacity: 0, y: 20 }}
    animate={{ opacity: 1, y: 0 }}
    transition={{ delay, duration: 0.5 }}
  >
    {children}
  </motion.div>
);
```

## Рассмотрение Производительности

При создании анимаций важно учитывать производительность:

- Используйте свойства `transform` и `opacity` для анимаций на GPU
- Избегайте анимации свойств, вызывающих рефлоу
- Используйте `will-change` для подготовки браузера к анимации
- Оптимизируйте рендеринг с помощью `React.memo`
- Ограничивайте частоту обновлений анимаций

## Техника FLIP

Техника FLIP (First, Last, Invert, Play) позволяет эффективно анимировать изменения в DOM:

```jsx
// First: фиксируем начальное состояние
const first = element.getBoundingClientRect();

// Last: вычисляем конечное состояние
element.style.transform = 'translateX(100px)';
const last = element.getBoundingClientRect();

// Invert: инвертируем разницу
const invert = first.left - last.left;
element.style.transform = `translateX(${invert}px)`;

// Play: запускаем анимацию
element.style.transition = 'transform 0.3s';
element.style.transform = 'translateX(100px)';
```

## Ступенчатые Анимации

Создание последовательных анимаций для списков:

```jsx
import { motion } from 'framer-motion';

const StaggeredList = ({ items }) => (
  <motion.div
    initial="hidden"
    animate="visible"
    variants={{
      visible: { transition: { staggerChildren: 0.1 } }
    }}
  >
    {items.map(item => (
      <motion.div
        key={item.id}
        variants={{
          hidden: { opacity: 0, y: 20 },
          visible: { opacity: 1, y: 0 }
        }}
      >
        {item.text}
      </motion.div>
    ))}
  </motion.div>
);
```

## Физика Пружин в Анимациях

Физика пружин создает естественные и реалистичные анимации:

```jsx
const springConfig = {
  type: "spring",
  stiffness: 300,
  damping: 20,
  mass: 1
};

<motion.div
  animate={{ x: 100 }}
  transition={springConfig}
/>
```

## Анимации на Основе Жестов

Реагирование на пользовательские жесты:

```jsx
import { motion } from 'framer-motion';

<motion.div
  drag
  dragConstraints={{ left: 0, right: 0, top: 0, bottom: 0 }}
  whileDrag={{ scale: 1.1 }}
  onDragEnd={(event, info) => {
    console.log('Drag ended at:', info.point);
  }}
/>
```

## Анимации на Основе Скролла

Интеграция с событиями прокрутки:

```jsx
import { useScroll, motion } from 'framer-motion';

function ScrollAnimation() {
  const { scrollYProgress } = useScroll();
  
  return (
    <motion.div
      style={{ scaleX: scrollYProgress }}
      className="progress-bar"
    />
  );
}
```

## Последовательности Анимаций

Создание сложных последовательностей:

```jsx
import { motion } from 'framer-motion';

<motion.div
  animate={{
    x: [0, 100, 0],
    scale: [1, 1.2, 1],
    rotate: [0, 90, 180]
  }}
  transition={{
    duration: 2,
    times: [0, 0.5, 1]
  }}
/>
```

## Сравнение Библиотек Анимаций

| Библиотека | Производительность | Сложность | Особенности |
|------------|-------------------|-----------|-------------|
| CSS | Высокая | Низкая | Простота, ограниченность |
| Framer Motion | Высокая | Средняя | Физика пружин, жесты |
| React Spring | Высокая | Средняя | Физика, интерполяция |
| GSAP | Высокая | Высокая | Мощные возможности |

## Пользовательские Хуки Анимации

Создание пользовательских хуков для повторного использования:

```jsx
import { useState, useEffect } from 'react';

function useAnimationState(duration = 300) {
  const [isAnimating, setIsAnimating] = useState(false);
  
  const startAnimation = () => {
    setIsAnimating(true);
    setTimeout(() => setIsAnimating(false), duration);
  };
  
  return { isAnimating, startAnimation };
}
```

## Рассмотрение Доступности

При создании анимаций учитывайте пользователей с различными потребностями:

- Уважайте настройку `prefers-reduced-motion`
- Обеспечьте возможность остановки анимаций
- Не используйте слишком яркие или быстро мигающие анимации
- Обеспечьте альтернативный способ взаимодействия

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Отладка Анимаций

Для отладки анимаций используйте:

- DevTools браузера (вкладка Animation)
- React DevTools Profiler
- Логирование состояний анимаций
- Инструменты производительности браузера

## Связи с другими файлами

- [[react-performance]] - для оптимизации анимаций
- [[react-hooks]] - для создания пользовательских хуков анимации
- [[css-animations]] - для понимания основ CSS-анимаций
- [[react-components]] - для интеграции анимаций в компоненты
- [[accessibility]] - для обеспечения доступности анимаций