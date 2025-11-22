---
aliases: [React Animations, Анимации в React, React Animation]
tags: [react, animation, frontend, web-development, hooks]
---

# Анимации-в-React

Анимации в React требуют специального подхода из-за декларативной природы фреймворка. Состояния и эффекты управляют анимационными состояниями компонентов, что позволяет создавать плавные и интерактивные интерфейсы.

## Основные подходы к анимациям в React

### CSS-анимации в React
Классический подход с использованием CSS-анимаций и изменения классов через состояние.

```jsx
import React, { useState } from 'react';
import './AnimatedComponent.css';

const AnimatedComponent = () => {
  const [isVisible, setIsVisible] = useState(false);
  const [isAnimating, setIsAnimating] = useState(false);

  const toggleVisibility = () => {
    setIsAnimating(true);
    setIsVisible(!isVisible);
  };

  const handleAnimationEnd = () => {
    setIsAnimating(false);
  };

  return (
    <div>
      <button onClick={toggleVisibility}>
        {isVisible ? 'Скрыть' : 'Показать'}
      </button>
      
      <div 
        className={`animated-element ${isVisible ? 'visible' : 'hidden'}`}
        onAnimationEnd={handleAnimationEnd}
      >
        Анимированный контент
      </div>
    </div>
  );
};

export default AnimatedComponent;
```

Соответствующий CSS:
```css
.animated-element {
  opacity: 0;
  transform: translateY(-20px);
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.animated-element.visible {
  opacity: 1;
  transform: translateY(0);
}

.animated-element.hidden {
  opacity: 0;
  transform: translateY(-20px);
  pointer-events: none;
}
```

### Использование React Transition Group
Библиотека `react-transition-group` предоставляет компоненты для управления анимациями при монтировании/размонтировании.

```jsx
import React, { useState } from 'react';
import { CSSTransition } from 'react-transition-group';
import './FadeAnimation.css';

const FadeAnimation = () => {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        Переключить видимость
      </button>
      
      <CSSTransition
        in={isVisible}
        timeout={300}
        classNames="fade"
        unmountOnExit
      >
        <div className="content">
          Появляющийся и исчезающий контент
        </div>
      </CSSTransition>
    </div>
  );
};

export default FadeAnimation;
```

Соответствующий CSS:
```css
.fade-enter {
  opacity: 0;
}

.fade-enter-active {
  opacity: 1;
  transition: opacity 300ms ease-in;
}

.fade-exit {
  opacity: 1;
}

.fade-exit-active {
  opacity: 0;
  transition: opacity 300ms ease-in;
}
```

## Framer Motion

Framer Motion - одна из самых популярных библиотек для анимаций в React.

### Установка и базовое использование
```bash
npm install framer-motion
```

```jsx
import { motion } from 'framer-motion';

const MotionComponent = () => {
  const containerVariants = {
    hidden: { opacity: 0, x: -100 },
    visible: {
      opacity: 1,
      x: 0,
      transition: {
        duration: 0.5,
        staggerChildren: 0.2
      }
    }
  };

  const itemVariants = {
    hidden: { y: 20, opacity: 0 },
    visible: {
      y: 0,
      opacity: 1
    }
  };

  return (
    <motion.div
      className="container"
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      <motion.h2 variants={itemVariants}>Заголовок</motion.h2>
      <motion.p variants={itemVariants}>Параграф</motion.p>
      <motion.button variants={itemVariants}>
        Кнопка
      </motion.button>
    </motion.div>
  );
};
```

### Анимации при наведении и клике
```jsx
import { motion } from 'framer-motion';

const InteractiveCard = () => {
  return (
    <motion.div
      whileHover={{ 
        scale: 1.05,
        boxShadow: "0px 10px 30px -5px rgba(0,0,0,0.2)"
      }}
      whileTap={{ scale: 0.95 }}
      className="card"
    >
      Интерактивная карточка
    </motion.div>
  );
};
```

### Анимации маршрутов
```jsx
import { motion, AnimatePresence } from 'framer-motion';
import { Routes, Route, useLocation } from 'react-router-dom';

const AnimatedRoutes = () => {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <Routes location={location} key={location.pathname}>
        <Route 
          path="/" 
          element={
            <motion.div
              initial={{ opacity: 0, x: 100 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: -100 }}
              transition={{ duration: 0.3 }}
            >
              Главная страница
            </motion.div>
          } 
        />
        <Route 
          path="/about" 
          element={
            <motion.div
              initial={{ opacity: 0, x: 100 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: -100 }}
              transition={{ duration: 0.3 }}
            >
              О нас
            </motion.div>
          } 
        />
      </Routes>
    </AnimatePresence>
  );
};
```

## React Spring

Альтернативная библиотека для физических анимаций.

```jsx
import { useSpring, animated } from 'react-spring';

const SpringComponent = () => {
  const [isToggled, setIsToggled] = useState(false);
  
  const props = useSpring({
    x: isToggled ? 100 : 0,
    rotate: isToggled ? 180 : 0,
    backgroundColor: isToggled ? '#ff6d6d' : '#6d6dff',
    config: { tension: 300, friction: 20 }
  });

  return (
    <animated.div
      onClick={() => setIsToggled(!isToggled)}
      style={{
        ...props,
        width: '100px',
        height: '100px',
        cursor: 'pointer',
        borderRadius: '10px'
      }}
    />
  );
};
```

## Пользовательские хуки для анимаций

Создание собственных хуков для управления анимациями:

```jsx
import { useState, useEffect, useRef } from 'react';

// Хук для анимации чисел
const useNumberAnimation = (target, duration = 1000) => {
  const [count, setCount] = useState(0);
  const rafRef = useRef();

  useEffect(() => {
    let startTime = null;
    const startCount = count;

    const animate = (currentTime) => {
      if (!startTime) startTime = currentTime;
      const timeElapsed = currentTime - startTime;
      const progress = Math.min(timeElapsed / duration, 1);

      // Easing function
      const easeProgress = 1 - Math.pow(1 - progress, 3);
      const currentValue = startCount + (target - startCount) * easeProgress;

      setCount(Math.floor(currentValue));

      if (progress < 1) {
        rafRef.current = requestAnimationFrame(animate);
      }
    };

    rafRef.current = requestAnimationFrame(animate);

    return () => {
      if (rafRef.current) {
        cancelAnimationFrame(rafRef.current);
      }
    };
  }, [target, duration, count]);

  return count;
};

// Использование хука
const Counter = ({ target }) => {
  const count = useNumberAnimation(target);
  
  return (
    <div className="counter">
      {count}+
    </div>
  );
};
```

### Хук для анимации прокрутки
```jsx
const useScrollAnimation = (threshold = 0.1) => {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsVisible(entry.isIntersecting);
      },
      { threshold }
    );

    if (ref.current) {
      observer.observe(ref.current);
    }

    return () => {
      if (ref.current) {
        observer.unobserve(ref.current);
      }
    };
  }, [threshold]);

  return [ref, isVisible];
};

// Использование
const ScrollAnimatedComponent = () => {
  const [ref, isVisible] = useScrollAnimation(0.3);

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 50 }}
      animate={{ opacity: isVisible ? 1 : 0, y: isVisible ? 0 : 50 }}
      transition={{ duration: 0.6 }}
    >
      Контент, который анимируется при прокрутке
    </motion.div>
  );
};
```

## Практические примеры

### Анимированный список
```jsx
import { motion, AnimatePresence } from 'framer-motion';

const AnimatedList = () => {
  const [items, setItems] = useState(['Элемент 1', 'Элемент 2', 'Элемент 3']);
  
  const addItem = () => {
    setItems([...items, `Элемент ${items.length + 1}`]);
  };
  
  const removeItem = (index) => {
    setItems(items.filter((_, i) => i !== index));
  };

  return (
    <div>
      <button onClick={addItem}>Добавить элемент</button>
      
      <AnimatePresence>
        {items.map((item, index) => (
          <motion.div
            key={index}
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: 'auto' }}
            exit={{ opacity: 0, height: 0 }}
            transition={{ duration: 0.3 }}
            style={{ 
              padding: '10px', 
              margin: '5px 0', 
              backgroundColor: '#f0f0f0',
              borderRadius: '4px'
            }}
          >
            {item}
            <button 
              onClick={() => removeItem(index)}
              style={{ marginLeft: '10px' }}
            >
              Удалить
            </button>
          </motion.div>
        ))}
      </AnimatePresence>
    </div>
  );
};
```

### Анимированный табы
```jsx
import { motion, AnimatePresence } from 'framer-motion';

const AnimatedTabs = () => {
  const [activeTab, setActiveTab] = useState(0);
  
  const tabs = [
    { title: 'Вкладка 1', content: 'Содержимое первой вкладки' },
    { title: 'Вкладка 2', content: 'Содержимое второй вкладки' },
    { title: 'Вкладка 3', content: 'Содержимое третьей вкладки' }
  ];

  return (
    <div>
      <div style={{ display: 'flex', marginBottom: '20px' }}>
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            style={{
              padding: '10px 20px',
              backgroundColor: activeTab === index ? '#007bff' : '#f0f0f0',
              color: activeTab === index ? 'white' : 'black',
              border: 'none',
              cursor: 'pointer',
              borderRadius: '4px 4px 0 0'
            }}
          >
            {tab.title}
          </button>
        ))}
      </div>
      
      <AnimatePresence mode="wait">
        <motion.div
          key={activeTab}
          initial={{ opacity: 0, x: 100 }}
          animate={{ opacity: 1, x: 0 }}
          exit={{ opacity: 0, x: -100 }}
          transition={{ duration: 0.3 }}
          style={{
            padding: '20px',
            border: '1px solid #ddd',
            borderRadius: '0 4px 4px 4px'
          }}
        >
          {tabs[activeTab].content}
        </motion.div>
      </AnimatePresence>
    </div>
  );
};
```

## Лучшие практики

### 1. Оптимизация производительности
- Используйте `transform` и `opacity` для анимаций
- Избегайте анимации свойств, вызывающих перерисовку
- Используйте `useMemo` и `useCallback` для предотвращения ненужных перерендеров

```jsx
const OptimizedComponent = ({ data }) => {
  const [isVisible, setIsVisible] = useState(false);
  
  // Оптимизируем вычисления
  const processedData = useMemo(() => 
    data.map(item => ({ ...item, id: item.id })),
    [data]
  );
  
  // Оптимизируем обработчики
  const handleClick = useCallback(() => {
    setIsVisible(!isVisible);
  }, [isVisible]);

  return (
    <motion.div
      animate={{ opacity: isVisible ? 1 : 0 }}
      transition={{ duration: 0.3 }}
    >
      <button onClick={handleClick}>Переключить</button>
      {/* Рендерим только при необходимости */}
      {isVisible && <DataList data={processedData} />}
    </motion.div>
  );
};
```

### 2. Учет предпочтений пользователя
```jsx
const useReducedMotion = () => {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setMatches(mediaQuery.matches);
    
    const handler = (e) => setMatches(e.matches);
    mediaQuery.addEventListener('change', handler);
    
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);
  
  return matches;
};

// Использование в компоненте
const MyComponent = () => {
  const prefersReducedMotion = useReducedMotion();
  
  return (
    <motion.div
      animate={{ x: 100 }}
      transition={{ 
        duration: prefersReducedMotion ? 0 : 0.5 
      }}
    >
      Контент
    </motion.div>
  );
};
```

> [!tip]
> Всегда учитывайте предпочтения пользователей, отключающих анимации, для обеспечения доступности интерфейса.

### 3. Адаптивные анимации
```jsx
const useResponsiveAnimation = () => {
  const [isMobile, setIsMobile] = useState(false);
  
  useEffect(() => {
    const checkIfMobile = () => {
      setIsMobile(window.innerWidth < 768);
    };
    
    checkIfMobile();
    window.addEventListener('resize', checkIfMobile);
    
    return () => window.removeEventListener('resize', checkIfMobile);
  }, []);
  
  return {
    duration: isMobile ? 0.4 : 0.6,
    stiffness: isMobile ? 100 : 200,
    damping: isMobile ? 10 : 15
  };
};
```

## Заключение

Анимации в React требуют особого подхода из-за декларативной природы фреймворка. Существует несколько подходов:

- Использование CSS-анимаций с изменением состояний
- Библиотека `react-transition-group` для управления анимациями при монтировании
- Framer Motion для сложных и интерактивных анимаций
- React Spring для физических анимаций

Выбор подхода зависит от сложности анимации и требований проекта. Для простых эффектов достаточно CSS, для сложных интерактивных анимаций лучше использовать специализированные библиотеки.

## См. также
- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[Анимации-в-Vue]]
- [[Анимации-в-Svelte]]