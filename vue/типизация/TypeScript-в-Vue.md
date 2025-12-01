---
aliases: ["Использование TypeScript в Vue", "Vue с TypeScript", "Типизированный Vue"]
tags: ["vue", "typescript", "frontend", "programming"]
---

# TypeScript в Vue

## Обзор

TypeScript предоставляет мощную систему типизации для приложений Vue.js, позволяя улучшить надежность, поддерживаемость и автодополнение кода. В 2025 году использование TypeScript в экосистеме Vue стало стандартом де-факто для крупных и средних проектов.

## Основные преимущества

- Обнаружение ошибок на этапе компиляции
- Улучшенная автодополнение в IDE
- Лучшая документация кода через типы
- Повышенная поддерживаемость больших приложений

## Настройка TypeScript в проекте Vue

### Установка зависимостей

```bash
npm install -D typescript @types/node
```

### Конфигурация tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "allowJs": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue"
  ]
}
```

## Типизация компонентов

### Options API

```typescript
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'MyComponent',
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment(): void {
      this.count++
    }
  }
})
```

### Composition API

```typescript
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'MyComponent',
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  setup(props) {
    const count = ref<number>(0)
    
    const doubleCount = computed(() => count.value * 2)
    
    const increment = (): void => {
      count.value++
    }
    
    return {
      count,
      doubleCount,
      increment
    }
  }
})
```

## Типизация props

```typescript
import { defineComponent } from 'vue'

interface Props {
  msg: string
  count?: number
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  disabled: false
})

export default defineComponent({
  // ...
})
```

## Рекомендации по использованию

- Всегда используйте строгую типизацию
- Определите интерфейсы для всех props
- Используйте `defineProps` и `defineEmits` для лучшей типизации
- Применяйте типы для реактивных данных

## Связанные темы

- [[Типизация-компонентов]]
- [[Типизация-хранилищ]]
- [[Типизация-директив]]
- [[Утилиты-для-типизации]]
- [[Composition API]]
- [[TypeScript]]

## Заключение

TypeScript в сочетании с Vue предоставляет мощный инструментарий для создания надежных и масштабируемых приложений. В 2025 году это неотъемлемая часть современной разработки на Vue.js.