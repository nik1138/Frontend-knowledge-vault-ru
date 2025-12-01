---
aliases: [Suspense, Асинхронные компоненты]
tags: [vue, suspense, frontend, async, ru-2025]
---

# Suspense

Suspense - это специальный элемент в Vue 3, предназначенный для обработки асинхронных операций в компонентах. Он позволяет отображать резервный контент (fallback) во время загрузки асинхронных зависимостей, таких как асинхронные компоненты или асинхронные функции в Composition API.

## Основы использования

Suspense оборачивает асинхронные компоненты и предоставляет визуальную обратную связь во время их загрузки:

```vue
<template>
  <div>
    <h1>Мое приложение</h1>
    
    <Suspense>
      <template #default>
        <AsyncComponent />
      </template>
      
      <template #fallback>
        <div>Загрузка...</div>
      </template>
    </Suspense>
  </div>
</template>

<script>
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() => 
  import('./AsyncComponent.vue')
)

export default {
  components: {
    AsyncComponent
  }
}
</script>
```

## Состояния Suspense

Suspense имеет два основных состояния:

1. **pending** - состояние ожидания загрузки асинхронного контента
2. **resolved** - состояние, когда асинхронный контент успешно загружен

## Атрибуты и события

Suspense поддерживает следующие события:

- `@resolve` - вызывается при успешной загрузке асинхронного компонента
- `@pending` - вызывается при начале загрузки
- `@fallback` - вызывается при отображении резервного контента

```vue
<template>
  <Suspense 
    @resolve="onResolve"
    @pending="onPending"
    @fallback="onFallback"
  >
    <template #default>
      <AsyncComponent />
    </template>
    
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>

<script>
import { defineAsyncComponent } from 'vue'
import LoadingSpinner from './LoadingSpinner.vue'

const AsyncComponent = defineAsyncComponent(() => 
  import('./AsyncComponent.vue')
)

export default {
  components: {
    AsyncComponent,
    LoadingSpinner
  },
  methods: {
    onResolve() {
      console.log('Асинхронный компонент загружен')
    },
    onPending() {
      console.log('Началась загрузка асинхронного компонента')
    },
    onFallback() {
      console.log('Отображается резервный контент')
    }
  }
}
</script>
</template>
```

## Практическое применение в российских реалиях 2025

Suspense особенно востребован в следующих сценариях:

1. **Микросервисная архитектура** - при интеграции компонентов из разных сервисов
2. **Ленивая загрузка** - для оптимизации производительности приложений
3. **Многоуровневые приложения** - при работе с асинхронными данными из API
4. **Компоненты с внешними зависимостями** - при использовании сторонних библиотек
5. **Совместимость с CDN** - при загрузке компонентов с внешних источников

## Пример реализации с учетом российских реалий

```vue
<template>
  <div class="product-page">
    <h1>Страница продукта</h1>
    
    <!-- Заголовок и основная информация -->
    <ProductHeader :product="product" />
    
    <!-- Асинхронная секция с отзывами -->
    <section class="reviews-section">
      <h2>Отзывы покупателей</h2>
      
      <Suspense>
        <template #default>
          <ProductReviews 
            :product-id="productId"
            :locale="locale"
            @review-submitted="handleReviewSubmitted"
          />
        </template>
        
        <template #fallback>
          <div class="reviews-loading">
            <LoadingSpinner />
            <p>Загрузка отзывов...</p>
          </div>
        </template>
      </Suspense>
    </section>
    
    <!-- Асинхронная секция с рекомендациями -->
    <section class="recommendations-section">
      <h2>Вам может понравиться</h2>
      
      <Suspense>
        <template #default>
          <ProductRecommendations 
            :product-id="productId"
            :user-id="userId"
            :locale="locale"
          />
        </template>
        
        <template #fallback>
          <div class="recommendations-loading">
            <LoadingSpinner />
            <p>Подбираем рекомендации...</p>
          </div>
        </template>
      </Suspense>
    </section>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { defineAsyncComponent } from 'vue'
import LoadingSpinner from '@/components/LoadingSpinner.vue'
import ProductHeader from '@/components/ProductHeader.vue'

// Асинхронные компоненты
const ProductReviews = defineAsyncComponent(() => 
  import('@/components/ProductReviews.vue')
)

const ProductRecommendations = defineAsyncComponent(() => 
  import('@/components/ProductRecommendations.vue')
)

export default {
  name: 'RussianProductPage',
  components: {
    ProductHeader,
    ProductReviews,
    ProductRecommendations,
    LoadingSpinner
  },
  props: {
    productId: {
      type: String,
      required: true
    },
    locale: {
      type: String,
      default: 'ru-RU'
    },
    userId: {
      type: String,
      default: null
    }
  },
  setup(props) {
    const product = ref(null)
    
    // Загрузка основной информации о продукте
    onMounted(async () => {
      try {
        product.value = await fetchProductData(props.productId)
      } catch (error) {
        console.error('Ошибка загрузки данных продукта:', error)
      }
    })
    
    const handleReviewSubmitted = (review) => {
      console.log('Отзыв отправлен:', review)
      // Обновление статистики продукта
    }
    
    return {
      product,
      handleReviewSubmitted
    }
  }
}

async function fetchProductData(productId) {
  // Имитация асинхронного запроса
  const response = await fetch(`/api/products/${productId}`)
  return response.json()
}
</script>

<style scoped>
.product-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.reviews-section, .recommendations-section {
  margin: 40px 0;
  padding: 20px;
  border: 1px solid #eee;
  border-radius: 8px;
}

.reviews-loading, .recommendations-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
  text-align: center;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  margin-bottom: 15px;
}
</style>
```

## Совместимость с Composition API

Suspense работает отлично с Composition API и асинхронными функциями:

```vue
<template>
  <Suspense>
    <template #default>
      <UserProfile />
    </template>
    
    <template #fallback>
      <div>Загрузка профиля пользователя...</div>
    </template>
  </Suspense>
</template>

<script>
import { defineAsyncComponent } from 'vue'

const UserProfile = defineAsyncComponent(async () => {
  const response = await fetch('/api/user-profile')
  if (!response.ok) {
    throw new Error('Не удалось загрузить профиль пользователя')
  }
  const module = await response.json()
  return module
})

export default {
  components: {
    UserProfile
  }
}
</script>
```

## Обработка ошибок

Suspense не обрабатывает ошибки напрямую, но можно использовать вместе с глобальной обработкой ошибок:

```vue
<template>
  <div>
    <Suspense>
      <template #default>
        <AsyncComponentWithErrorHandling />
      </template>
      
      <template #fallback>
        <div>Загрузка...</div>
      </template>
    </Suspense>
  </div>
</template>

<script>
import { defineAsyncComponent } from 'vue'

const AsyncComponentWithErrorHandling = defineAsyncComponent({
  loader: () => import('./AsyncComponent.vue'),
  loadingComponent: LoadingComponent,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})

export default {
  components: {
    AsyncComponentWithErrorHandling
  }
}
</script>
```

## Особенности и ограничения

1. **Вложенность** - Suspense может быть вложенным, но это требует аккуратного управления
2. **SSR** - требует особого внимания при рендеринге на стороне сервера
3. **Совместимость** - работает только с асинхронными компонентами и Composition API
4. **Производительность** - может увеличить время загрузки при неправильном использовании

## Связанные темы

- [[Component]] - для понимания динамических компонентов
- [[Teleport]] - для вывода асинхронного содержимого в нужное место
- [[Keep-alive]] - для кэширования асинхронных компонентов
- [[Vue компоненты]]
- [[Vue асинхронность]]

## Лучшие практики

1. Используйте осмысленные резервные компоненты для улучшения UX
2. Устанавливайте таймауты для асинхронных операций
3. Обрабатывайте ошибки асинхронных компонентов
4. Используйте Suspense только для действительно асинхронных операций
5. Учитывайте особенности локализации при отображении сообщений загрузки