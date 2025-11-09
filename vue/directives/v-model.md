# v-model

## Определение
`v-model` - это директива Vue, которая обеспечивает двустороннюю привязку данных между элементами формы и данными компонента. Это удобное сокращение для комбинации `v-bind` и `v-on`.

## Использование

### С текстовыми полями
```vue
<template>
  <input v-model="message" placeholder="Введите сообщение">
  <p>Сообщение: {{ message }}</p>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('')
</script>
```

### С чекбоксами
```vue
<template>
  <!-- Единичный чекбокс -->
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked ? 'Отмечено' : 'Не отмечено' }}</label>
  
  <!-- Множественный выбор в массив -->
  <input type="checkbox" id="jack" value="Джек" v-model="checkedNames">
  <label for="jack">Джек</label>
  <input type="checkbox" id="john" value="Джон" v-model="checkedNames">
  <label for="john">Джон</label>
</template>

<script setup>
import { ref } from 'vue'

const checked = ref(false)
const checkedNames = ref([])
</script>
```

### С радио-кнопками
```vue
<template>
  <input type="radio" id="one" value="Один" v-model="picked">
  <label for="one">Один</label>
  <input type="radio" id="two" value="Два" v-model="picked">
  <label for="two">Два</label>
  <p>Выбрано: {{ picked }}</p>
</template>

<script setup>
import { ref } from 'vue'

const picked = ref('')
</script>
```

### С выпадающими списками
```vue
<template>
  <!-- Одиночный выбор -->
  <select v-model="selected">
    <option disabled value="">Выберите один</option>
    <option>А</option>
    <option>Б</option>
    <option>В</option>
  </select>
  <span>Выбрано: {{ selected }}</span>
  
  <!-- Множественный выбор -->
  <select v-model="selectedMultiple" multiple>
    <option v-for="option in options" :value="option.value">
      {{ option.text }}
    </option>
  </select>
</template>

<script setup>
import { ref } from 'vue'

const selected = ref('')
const selectedMultiple = ref([])
const options = ref([
  { text: 'Один', value: 'one' },
  { text: 'Два', value: 'two' },
  { text: 'Три', value: 'three' }
])
</script>
</template>
```

## Модификаторы

### .lazy
```vue
<template>
  <!-- Обновляет модель при событии change, а не input -->
  <input v-model.lazy="msg">
</template>
```

### .number
```vue
<template>
  <!-- Автоматически приводит значение к числу -->
  <input v-model.number="age" type="number">
</template>
```

### .trim
```vue
<template>
  <!-- Удаляет пробелы по краям -->
  <input v-model.trim="msg">
</template>
</template>
```

## Работа с композаблами

```javascript
// useVModel.js
import { computed } from 'vue'

export function useVModel(props, emit, name) {
  return computed({
    get: () => props[name],
    set: (value) => emit(`update:${name}`, value)
  })
}

// В компоненте
<script setup>
import { useVModel } from '@/composables/useVModel'

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const model = useVModel(props, emit, 'modelValue')
</script>
```

## Кастомные компоненты с v-model

### Простая реализация
```vue
<!-- ChildComponent.vue -->
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
  >
</template>

<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent v-model="parentData" />
</template>

<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const parentData = ref('')
</script>
```

### Множественные v-model
```vue
<!-- ChildComponent.vue -->
<template>
  <input 
    :value="firstName" 
    @input="$emit('update:firstName', $event.target.value)"
  >
  <input 
    :value="lastName" 
    @input="$emit('update:lastName', $event.target.value)"
  >
</template>

<script setup>
defineProps(['firstName', 'lastName'])
defineEmits(['update:firstName', 'update:lastName'])
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent 
    v-model:firstName="first" 
    v-model:lastName="last" 
  />
</template>
```

## Связь с реактивностью
- `v-model` работает с реактивными значениями
- Изменение в input автоматически обновляет реактивное состояние
- Обновление реактивного состояния обновляет отображение в input

## Тестирование v-model
```javascript
// test/VModelComponent.test.js
import { mount } from '@vue/test-utils'
import VModelComponent from '@/components/VModelComponent.vue'

test('v-model обновляет состояние', async () => {
  const wrapper = mount(VModelComponent)
  
  await wrapper.find('input').setValue('новое значение')
  
  expect(wrapper.vm.modelValue).toBe('новое значение')
})
```

## Связь с другими концепциями
- [[directives/v-bind]] - основа для привязки значений
- [[directives/v-on]] - основа для обработки событий
- [[forms/Forms]] - использование в формах
- [[components/Props]] - передача данных в компоненты
- [[reactivity/Reactivity System]] - реактивность двусторонней связи