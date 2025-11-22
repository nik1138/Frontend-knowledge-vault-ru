---
aliases: [Vue Animations, Анимации в Vue, Vue Animation]
tags: [vue, animation, frontend, web-development, transitions]
---

# Анимации-в-Vue

Vue.js предоставляет мощные и интуитивные инструменты для создания анимаций. Встроенные возможности Vue позволяют легко реализовывать переходы и анимации при изменении состояния, монтировании компонентов и работе с динамическими списками.

## Встроенные переходы и анимации в Vue

### Компонент `<transition>`
Компонент `<transition>` позволяет применять анимации к одиночным элементам при их монтировании/размонтировании.

```vue
<template>
  <div>
    <button @click="show = !show">Переключить</button>
    
    <transition name="fade">
      <p v-if="show">Этот параграф анимируется</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  }
}
</script>

<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

### Переходы для списков с `<transition-group>`
Компонент `<transition-group>` позволяет анимировать элементы списка при добавлении, удалении или перемещении.

```vue
<template>
  <div>
    <button @click="addItem">Добавить элемент</button>
    <button @click="shuffle">Перемешать</button>
    
    <transition-group 
      name="list" 
      tag="ul" 
      class="list-container"
    >
      <li 
        v-for="item in items" 
        :key="item.id"
        @click="removeItem(item.id)"
      >
        {{ item.text }}
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Элемент 1' },
        { id: 2, text: 'Элемент 2' },
        { id: 3, text: 'Элемент 3' }
      ],
      nextId: 4
    }
  },
  methods: {
    addItem() {
      this.items.push({
        id: this.nextId++,
        text: `Элемент ${this.nextId - 1}`
      });
    },
    removeItem(id) {
      this.items = this.items.filter(item => item.id !== id);
    },
    shuffle() {
      this.items = [...this.items].sort(() => Math.random() - 0.5);
    }
  }
}
</script>

<style>
.list-container {
  position: relative;
}

.list-enter-active, .list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from, .list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* Анимация перемещения */
.list-move {
  transition: transform 0.5s ease;
}
</style>
```

## Типы переходов в Vue

### CSS-переходы
Vue автоматически добавляет CSS-классы в определенные моменты анимации:

- `*-enter-from`: Начальное состояние при входе
- `*-enter-active`: Активное состояние при входе
- `*-enter-to`: Конечное состояние при входе
- `*-leave-from`: Начальное состояние при выходе
- `*-leave-active`: Активное состояние при выходе
- `*-leave-to`: Конечное состояние при выходе

```vue
<template>
  <div>
    <button @click="show = !show">Переключить</button>
    
    <transition name="slide-fade">
      <p v-if="show">Слайд и фейд одновременно</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  }
}
</script>

<style>
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from {
  transform: translateX(20px) rotateZ(-5deg);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateX(-20px) rotateZ(5deg);
  opacity: 0;
}
</style>
```

### JavaScript-переходы
Для более сложной логики анимаций можно использовать JavaScript-обратные вызовы.

```vue
<template>
  <div>
    <button @click="show = !show">Переключить</button>
    
    <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @leave="leave"
      :css="false"
    >
      <p v-if="show" class="js-animation">JS анимация</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  },
  methods: {
    beforeEnter(el) {
      el.style.opacity = 0;
      el.style.transform = 'translateX(-100px)';
    },
    enter(el, done) {
      // Анимация появления
      let opacity = 0;
      let x = -100;
      
      const tick = () => {
        opacity += 0.1;
        x += 10;
        
        el.style.opacity = opacity;
        el.style.transform = `translateX(${x}px)`;
        
        if (opacity < 1) {
          requestAnimationFrame(tick);
        } else {
          done();
        }
      };
      
      tick();
    },
    leave(el, done) {
      // Анимация исчезновения
      let opacity = 1;
      let x = 0;
      
      const tick = () => {
        opacity -= 0.1;
        x -= 10;
        
        el.style.opacity = opacity;
        el.style.transform = `translateX(${x}px)`;
        
        if (opacity > 0) {
          requestAnimationFrame(tick);
        } else {
          done();
        }
      };
      
      tick();
    }
  }
}
</script>

<style>
.js-animation {
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 4px;
}
</style>
```

## Анимации с использованием библиотек

### Vue с GSAP
Интеграция Vue с GSAP для сложных анимаций.

```vue
<template>
  <div>
    <button @click="animateWithGSAP">Анимировать с GSAP</button>
    <div ref="animatedElement" class="gsap-element">GSAP элемент</div>
  </div>
</template>

<script>
import { gsap } from 'gsap';

export default {
  methods: {
    animateWithGSAP() {
      gsap.to(this.$refs.animatedElement, {
        x: 300,
        rotation: 360,
        duration: 2,
        ease: 'power2.inOut',
        yoyo: true,
        repeat: 1
      });
    }
  }
}
</script>

<style>
.gsap-element {
  width: 100px;
  height: 100px;
  background-color: #42b983;
  border-radius: 8px;
  margin: 20px 0;
}
</style>
```

### Vue с Framer Motion (через Vue Motion)
Для использования похожих возможностей, как в React.

```vue
<template>
  <div>
    <Motion
      :initial="{ opacity: 0, x: -100 }"
      :animate="{ opacity: 1, x: 0 }"
      :transition="{ duration: 0.5 }"
      class="motion-element"
    >
      Анимированный элемент
    </Motion>
  </div>
</template>

<script>
import { Motion } from 'vue-motion';

export default {
  components: {
    Motion
  }
}
</script>
```

## Практические примеры

### Анимированный аккордеон
```vue
<template>
  <div class="accordion">
    <div 
      v-for="(item, index) in items" 
      :key="index"
      class="accordion-item"
    >
      <div 
        class="accordion-header" 
        @click="toggleItem(index)"
      >
        {{ item.title }}
        <span :class="{ 'rotate': activeIndex === index }">▼</span>
      </div>
      
      <transition 
        name="slide"
        @enter="onEnter"
        @leave="onLeave"
      >
        <div 
          v-if="activeIndex === index" 
          class="accordion-content"
          ref="content"
        >
          {{ item.content }}
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeIndex: null,
      items: [
        { title: 'Заголовок 1', content: 'Содержимое 1' },
        { title: 'Заголовок 2', content: 'Содержимое 2' },
        { title: 'Заголовок 3', content: 'Содержимое 3' }
      ]
    }
  },
  methods: {
    toggleItem(index) {
      this.activeIndex = this.activeIndex === index ? null : index;
    },
    onEnter(el) {
      el.style.height = 'auto';
      const height = getComputedStyle(el).height;
      el.style.height = '0';
      
      // Force repaint
      getComputedStyle(el).height;
      
      setTimeout(() => {
        el.style.height = height;
      });
    },
    onLeave(el) {
      const height = getComputedStyle(el).height;
      el.style.height = height;
      
      // Force repaint
      getComputedStyle(el).height;
      
      setTimeout(() => {
        el.style.height = '0';
      });
    }
  }
}
</script>

<style>
.accordion-item {
  border: 1px solid #ddd;
  margin-bottom: 5px;
  border-radius: 4px;
  overflow: hidden;
}

.accordion-header {
  padding: 15px;
  background-color: #f5f5f5;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.accordion-header:hover {
  background-color: #e9ecef;
}

.accordion-content {
  padding: 15px;
  overflow: hidden;
  transition: height 0.3s ease;
}

.slide-enter-active, .slide-leave-active {
  transition: height 0.3s ease;
  overflow: hidden;
}

.slide-enter-from, .slide-leave-to {
  height: 0 !important;
}

.rotate {
  transform: rotate(180deg);
  transition: transform 0.3s ease;
}
</style>
```

### Анимированный прогресс-бар
```vue
<template>
  <div class="progress-container">
    <div class="progress-bar">
      <div 
        class="progress-fill" 
        :style="{ width: animatedProgress + '%' }"
      >
        {{ Math.round(percentage) }}%
      </div>
    </div>
    <div class="controls">
      <button @click="increaseProgress">Увеличить</button>
      <button @click="resetProgress">Сбросить</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      percentage: 0,
      targetPercentage: 0,
      animatedProgress: 0
    }
  },
  mounted() {
    this.animateProgress();
  },
  methods: {
    increaseProgress() {
      this.targetPercentage = Math.min(100, this.targetPercentage + 25);
      this.animateProgress();
    },
    resetProgress() {
      this.percentage = 0;
      this.targetPercentage = 0;
      this.animatedProgress = 0;
    },
    animateProgress() {
      let startTime = null;
      const duration = 1000;
      
      const animation = (currentTime) => {
        if (!startTime) startTime = currentTime;
        const elapsed = currentTime - startTime;
        const progress = Math.min(elapsed / duration, 1);
        
        // Easing function
        const easeProgress = 1 - Math.pow(1 - progress, 3);
        this.animatedProgress = this.percentage + (this.targetPercentage - this.percentage) * easeProgress;
        
        if (progress < 1) {
          requestAnimationFrame(animation);
        } else {
          this.percentage = this.targetPercentage;
        }
      };
      
      requestAnimationFrame(animation);
    }
  },
  watch: {
    targetPercentage() {
      this.animateProgress();
    }
  }
}
</script>

<style>
.progress-container {
  max-width: 400px;
  margin: 20px auto;
}

.progress-bar {
  height: 30px;
  background-color: #e9ecef;
  border-radius: 15px;
  overflow: hidden;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #42b983, #35495e);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  transition: width 0.3s ease;
}

.controls {
  margin-top: 20px;
  text-align: center;
}

.controls button {
  margin: 0 5px;
  padding: 8px 16px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.controls button:hover {
  background-color: #35495e;
}
</style>
```

## Анимации с использованием Composition API

### Хук для анимации чисел
```vue
<template>
  <div>
    <h2>{{ animatedCount }}</h2>
    <button @click="updateTarget">Обновить значение</button>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';

// Пользовательский хук для анимации чисел
function useAnimatedNumber(target, duration = 1000) {
  const animatedValue = ref(0);
  let animationFrame = null;
  
  const animate = (startValue, endValue) => {
    const startTime = performance.now();
    
    const runAnimation = (currentTime) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Easing function
      const easeProgress = 1 - Math.pow(1 - progress, 3);
      const currentValue = startValue + (endValue - startValue) * easeProgress;
      
      animatedValue.value = Math.floor(currentValue);
      
      if (progress < 1) {
        animationFrame = requestAnimationFrame(runAnimation);
      }
    };
    
    requestAnimationFrame(runAnimation);
  };
  
  const updateValue = (newValue) => {
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }
    animate(animatedValue.value, newValue);
  };
  
  return {
    value: animatedValue,
    updateValue
  };
}

export default {
  setup() {
    const target = ref(100);
    const { value: animatedCount, updateValue } = useAnimatedNumber(target.value);
    
    const updateTarget = () => {
      target.value = Math.floor(Math.random() * 1000);
      updateValue(target.value);
    };
    
    onMounted(() => {
      updateValue(target.value);
    });
    
    return {
      animatedCount,
      updateTarget
    };
  }
}
</script>
```

## Лучшие практики

### 1. Оптимизация производительности
- Используйте `transform` и `opacity` вместо изменений макета
- Избегайте анимации свойств, вызывающих перерисовку
- Используйте `will-change` для элементов, которые будут анимироваться

```vue
<style>
.optimized-animation {
  will-change: transform, opacity;
  transform: translateZ(0); /* Включает аппаратное ускорение */
}

/* Плохо - вызывает перерисовку */
.not-optimized {
  animation: widthChange 0.5s ease;
}
</style>
```

### 2. Учет предпочтений пользователя
```vue
<template>
  <transition
    :name="transitionName"
    :duration="transitionDuration"
  >
    <div v-if="show">Контент</div>
  </transition>
</template>

<script>
export default {
  data() {
    return {
      show: true,
      prefersReducedMotion: false
    }
  },
  computed: {
    transitionName() {
      return this.prefersReducedMotion ? 'no-animation' : 'fade';
    },
    transitionDuration() {
      return this.prefersReducedMotion ? 0 : 300;
    }
  },
  mounted() {
    if (window.matchMedia) {
      const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
      this.prefersReducedMotion = mediaQuery.matches;
      
      const handler = (e) => {
        this.prefersReducedMotion = e.matches;
      };
      
      mediaQuery.addEventListener('change', handler);
      this.$once('hook:beforeDestroy', () => {
        mediaQuery.removeEventListener('change', handler);
      });
    }
  }
}
</script>

<style>
.no-animation-enter-active, .no-animation-leave-active {
  transition: none;
}

.no-animation-enter-from, .no-animation-leave-to {
  opacity: inherit;
}
</style>
```

> [!tip]
> Всегда учитывайте предпочтения пользователей, отключающих анимации, для обеспечения доступности интерфейса.

### 3. Семантические анимации
Анимации должны улучшать восприятие интерфейса, а не отвлекать от него.

## Заключение

Vue предоставляет мощные инструменты для создания анимаций:

- Встроенный компонент `<transition>` для одиночных элементов
- Компонент `<transition-group>` для списков
- Поддержка CSS и JavaScript-анимаций
- Возможность интеграции с внешними библиотеками анимаций

Vue позволяет гибко подходить к анимациям, от простых переходов до сложных интерактивных анимаций, сохраняя при этом читаемость и поддерживаемость кода.

## См. также
- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[Анимации-в-React]]
- [[Анимации-в-Svelte]]