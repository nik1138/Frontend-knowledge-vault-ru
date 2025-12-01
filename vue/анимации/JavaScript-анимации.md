---
aliases: [JavaScript Animations in Vue, Анимации JavaScript в Vue]
tags: [vue, javascript, animation, frontend]
---

# JavaScript-анимации в Vue.js

## Обзор

JavaScript-анимации в Vue.js предоставляют более гибкий и мощный способ создания анимационных эффектов по сравнению с чисто CSS-анимациями. Они позволяют реализовать сложные анимационные последовательности, динамически управлять параметрами анимации и взаимодействовать с анимациями на основе пользовательских действий или данных приложения.

## Основы JavaScript-анимаций

JavaScript-анимации в Vue реализуются через хуки переходов компонентов `<transition>` и `<transition-group>`. Эти хуки позволяют использовать JavaScript для запуска анимаций в определенные моменты жизненного цикла перехода.

### Хуки JavaScript-анимаций

- `before-enter` - вызывается перед вставкой элемента
- `enter` - вызывается при вставке элемента
- `after-enter` - вызывается после завершения вставки
- `enter-cancelled` - вызывается при отмене вставки
- `before-leave` - вызывается перед удалением элемента
- `leave` - вызывается при удалении элемента
- `after-leave` - вызывается после завершения удаления
- `leave-cancelled` - вызывается при отмене удаления

## Примеры JavaScript-анимаций

### Простая анимация с использованием requestAnimationFrame

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
      <p v-if="show">JavaScript-анимация в Vue!</p>
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
      el.style.opacity = 0
      el.style.transform = 'translateX(-30px)'
    },
    enter(el, done) {
      let opacity = 0
      let translateX = -30
      
      const tick = () => {
        opacity += 0.1
        translateX += 3
        
        el.style.opacity = opacity
        el.style.transform = `translateX(${translateX}px)`
        
        if (opacity < 1) {
          requestAnimationFrame(tick)
        } else {
          done()
        }
      }
      
      tick()
    },
    leave(el, done) {
      let opacity = 1
      let translateX = 0
      
      const tick = () => {
        opacity -= 0.1
        translateX += 3
        
        el.style.opacity = opacity
        el.style.transform = `translateX(${translateX}px)`
        
        if (opacity > 0) {
          requestAnimationFrame(tick)
        } else {
          done()
        }
      }
      
      tick()
    }
  }
}
</script>
```

### Использование библиотеки anime.js

```vue
<template>
  <div>
    <button @click="animateElement">Анимировать</button>
    <div ref="animatedElement" class="box"></div>
  </div>
</template>

<script>
import anime from 'animejs'

export default {
  methods: {
    animateElement() {
      anime({
        targets: this.$refs.animatedElement,
        translateX: 250,
        rotate: 180,
        borderRadius: ['0%', '50%'],
        backgroundColor: '#FFF',
        duration: 2000,
        direction: 'alternate',
        loop: true
      })
    }
  }
}
</script>

<style>
.box {
  width: 100px;
  height: 100px;
  background-color: #42b983;
  margin: 20px;
}
</style>
```

## Анимации с использованием Web Animations API

Web Animations API предоставляет современный способ создания анимаций, которые могут быть более производительными, чем традиционные CSS-анимации и JavaScript-анимации:

```vue
<template>
  <div>
    <button @click="animateWithWebAPI">Анимировать с Web API</button>
    <div ref="webAnimatedElement" class="box"></div>
  </div>
</template>

<script>
export default {
  methods: {
    animateWithWebAPI() {
      const element = this.$refs.webAnimatedElement
      const animation = element.animate([
        { transform: 'translateX(0px)', backgroundColor: '#42b983' },
        { transform: 'translateX(200px)', backgroundColor: '#FFF' },
        { transform: 'translateX(0px)', backgroundColor: '#42b983' }
      ], {
        duration: 2000,
        easing: 'ease-in-out',
        iterations: 1
      })
      
      animation.onfinish = () => {
        console.log('Анимация завершена!')
      }
    }
  }
}
</script>
```

## Анимации с использованием CSS-in-JS

Для более сложных анимаций можно использовать CSS-in-JS подход:

```vue
<template>
  <div>
    <button @click="startComplexAnimation">Сложная анимация</button>
    <div ref="complexElement" class="complex-box"></div>
  </div>
</template>

<script>
export default {
  methods: {
    startComplexAnimation() {
      const element = this.$refs.complexElement
      const duration = 2000
      
      // Определяем ключевые кадры анимации
      const keyframes = [
        { 
          transform: 'translateX(0) scale(1) rotate(0deg)', 
          backgroundColor: '#42b983',
          borderRadius: '0px'
        },
        { 
          transform: 'translateX(100px) scale(1.2) rotate(90deg)', 
          backgroundColor: '#FFF',
          borderRadius: '10px'
        },
        { 
          transform: 'translateX(200px) scale(0.8) rotate(180deg)', 
          backgroundColor: '#f8f8f8',
          borderRadius: '20px'
        },
        { 
          transform: 'translateX(0) scale(1) rotate(360deg)', 
          backgroundColor: '#42b983',
          borderRadius: '0px'
        }
      ]
      
      const options = {
        duration: duration,
        easing: 'cubic-bezier(0.25, 0.1, 0.25, 1.0)',
        fill: 'forwards'
      }
      
      element.animate(keyframes, options)
    }
  }
}
</script>

<style>
.complex-box {
  width: 100px;
  height: 100px;
  background-color: #42b983;
  margin: 20px;
}
</style>
```

## Совместимость с российскими реалиями 2025

В 2025 году в российской разработке особенно важна оптимизация производительности и поддержка различных браузеров, включая отечественные. JavaScript-анимации требуют особого внимания к производительности:

1. Используйте `requestAnimationFrame` вместо `setTimeout` или `setInterval`
2. Оптимизируйте сложные анимации с помощью `will-change` и `transform`
3. Учитывайте производительность на бюджетных устройствах, популярных в России
4. Обеспечьте резервные варианты для браузеров без поддержки Web Animations API

### Обработка пользовательских предпочтений

Важно учитывать системные настройки пользователя, такие как предпочтение уменьшения анимаций:

```javascript
// Проверка предпочтения пользователя на анимации
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)')
if (prefersReducedMotion.matches) {
  // Отключить или упростить анимации
  this.animationEnabled = false
} else {
  this.animationEnabled = true
}
```

## Лучшие практики

1. Используйте JavaScript-анимации для сложных сценариев, которые невозможно реализовать с помощью CSS
2. Обеспечьте доступность, учитывая предпочтения пользователя
3. Оптимизируйте производительность, используя аппаратное ускорение
4. Добавляйте обработчики ошибок для анимационных функций
5. Тестируйте на разных устройствах и браузерах

## Связанные темы

- [[CSS-анимации]]
- [[Встроенные-переходы]]
- [[Библиотеки-анимаций]]
- [[Пользовательские-анимации]]

## Заключение

JavaScript-анимации в Vue.js предоставляют мощный инструмент для создания сложных и интерактивных анимационных эффектов. Они особенно полезны для реализации анимаций, зависящих от состояния приложения или пользовательских действий. При правильном использовании JavaScript-анимации могут значительно улучшить пользовательский опыт.