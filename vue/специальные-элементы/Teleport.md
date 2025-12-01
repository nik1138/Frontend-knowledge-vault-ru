---
aliases: [Телепорт, Vue Teleport]
tags: [vue, teleport, frontend, ru-2025]
---

# Teleport

Teleport (ранее Portal) - это специальный элемент в Vue 3, который позволяет рендерить содержимое компонента в DOM-узел, который находится за пределами иерархии DOM этого компонента. Это особенно полезно для создания модальных окон, всплывающих подсказок, уведомлений и других элементов, которые должны отображаться поверх основного контента.

## Основы использования

Элемент `<teleport>` принимает атрибут `to`, который указывает CSS-селектор целевого DOM-узла:

```vue
<template>
  <div class="app">
    <h1>Основное приложение</h1>
    
    <!-- Модальное окно будет отрендерено в body -->
    <teleport to="body">
      <div v-if="showModal" class="modal">
        <p>Это модальное окно</p>
        <button @click="showModal = false">Закрыть</button>
      </div>
    </teleport>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const showModal = ref(false)
    
    return {
      showModal
    }
  }
}
</script>
```

## Атрибуты Teleport

### `to` (обязательный)
Указывает, в какой DOM-узел будет телепортировано содержимое. Принимает CSS-селектор или непосредственно DOM-элемент:

```vue
<!-- Телепорт в элемент с id 'modal-container' -->
<teleport to="#modal-container">
  <div>Содержимое модального окна</div>
</teleport>

<!-- Телепорт в body -->
<teleport to="body">
  <div>Содержимое в body</div>
</teleport>
```

### `disabled`
Позволяет отключить телепортацию:

```vue
<template>
  <teleport :to="target" :disabled="!isEnabled">
    <div>Содержимое</div>
  </teleport>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const isEnabled = ref(false)
    const target = ref('#modal-container')
    
    return {
      isEnabled,
      target
    }
  }
}
</script>
```

## Практическое применение в российских реалиях 2025

Teleport особенно востребован в следующих сценариях:

1. **Модальные окна и диалоги** - для корректного позиционирования и z-index
2. **Всплывающие меню и подсказки** - для избежания ограничений по переполнению контейнеров
3. **Уведомления и тосты** - для отображения поверх всего приложения
4. **Сторонние виджеты** - для интеграции с внешними компонентами
5. **Совместимость с legacy-кодом** - для интеграции Vue-компонентов в существующие приложения

## Пример реализации модального окна с учетом российских реалий

```vue
<template>
  <div class="modal-wrapper">
    <!-- Телепорт модального окна в body -->
    <teleport to="body">
      <div 
        v-if="isVisible" 
        class="modal-overlay"
        @click="handleOverlayClick"
        :class="{ 'modal-overlay--blur': isBackgroundBlurred }"
      >
        <div 
          class="modal-content"
          @click.stop
          :style="{ maxWidth: maxWidth + 'px' }"
        >
          <header class="modal-header">
            <h2>{{ title }}</h2>
            <button 
              class="modal-close-btn"
              @click="closeModal"
              aria-label="Закрыть модальное окно"
            >
              ×
            </button>
          </header>
          
          <main class="modal-body">
            <slot></slot>
          </main>
          
          <footer class="modal-footer" v-if="$slots.footer">
            <slot name="footer"></slot>
          </footer>
        </div>
      </div>
    </teleport>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  name: 'RussianModal',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: 'Модальное окно'
    },
    maxWidth: {
      type: Number,
      default: 600
    },
    closeOnOverlay: {
      type: Boolean,
      default: true
    },
    isBackgroundBlurred: {
      type: Boolean,
      default: true
    }
  },
  emits: ['update:visible', 'close'],
  setup(props, { emit }) {
    const isVisible = ref(props.visible)
    
    // Обработка нажатия Escape для закрытия модального окна
    const handleEscape = (e) => {
      if (e.key === 'Escape' && isVisible.value) {
        closeModal()
      }
    }
    
    const closeModal = () => {
      isVisible.value = false
      emit('update:visible', false)
      emit('close')
    }
    
    const handleOverlayClick = () => {
      if (props.closeOnOverlay) {
        closeModal()
      }
    }
    
    onMounted(() => {
      document.addEventListener('keydown', handleEscape)
    })
    
    onUnmounted(() => {
      document.removeEventListener('keydown', handleEscape)
    })
    
    // Синхронизация с пропсом
    watch(() => props.visible, (newVal) => {
      isVisible.value = newVal
    })
    
    return {
      isVisible,
      closeModal,
      handleOverlayClick
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-overlay--blur {
  backdrop-filter: blur(4px);
}

.modal-content {
  background: white;
  border-radius: 8px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  max-height: 90vh;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.modal-body {
  padding: 20px;
  flex: 1;
}

.modal-footer {
  padding: 20px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.modal-close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
}
</style>
```

## Совместимость с другими элементами

Teleport работает корректно с другими специальными элементами Vue:

- [[Keep-alive]] - компоненты внутри Teleport могут быть кэшированы
- [[Suspense]] - может использоваться для обработки асинхронного содержимого
- [[Slot]] - позволяет передавать содержимое в телепортированные элементы

## Особенности и ограничения

1. **CSS-стили**: Учтите, что телепортированный контент может не наследовать стили родительского компонента
2. **Отношения в дереве**: Хотя DOM-структура изменяется, отношения компонентов остаются прежними
3. **SSR**: Требует особого внимания при рендеринге на стороне сервера
4. **Безопасность**: Убедитесь, что телепорт не позволяет вставлять произвольный HTML в защищенные области

## Связанные темы

- [[Component]] - для понимания динамических компонентов
- [[Slot]] - для передачи содержимого в телепортированные компоненты
- [[Keep-alive]] - для кэширования телепортированных компонентов
- [[Vue компоненты]]
- [[Vue порталы]]

## Лучшие практики

1. Используйте телепорт только когда это действительно необходимо
2. Обеспечьте доступность (accessibility) для телепортированного содержимого
3. Используйте уникальные идентификаторы для целевых контейнеров
4. Учитывайте особенности стилизации телепортированного содержимого