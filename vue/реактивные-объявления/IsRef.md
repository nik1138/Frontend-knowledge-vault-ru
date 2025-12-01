---
aliases: [Проверка типа ref, isRef в Vue]
tags: [vue, реактивность, isRef, javascript]
---

# IsRef

`isRef` - это утилита в Vue 3, которая проверяет, является ли переданный аргумент ref'ом. Она возвращает `true`, если аргумент является ref'ом, и `false` в противном случае. Это полезно при создании универсальных функций, которые должны обрабатывать как обычные значения, так и ref'ы.

## Основное использование

```javascript
import { ref, isRef } from 'vue'

const count = ref(0)
const normalValue = 5

console.log(isRef(count)) // true
console.log(isRef(normalValue)) // false
console.log(isRef('string')) // false
console.log(isRef(null)) // false
```

## Практическое применение

`isRef` особенно полезен при создании универсальных функций:

```javascript
import { ref, isRef } from 'vue'

function getValue(value) {
  // Если value - ref, возвращаем .value, иначе само значение
  return isRef(value) ? value.value : value
}

const count = ref(10)
const normalNumber = 20

console.log(getValue(count)) // 10
console.log(getValue(normalNumber)) // 20
```

## Использование в пользовательских хуках

```javascript
import { ref, isRef, unref } from 'vue'

function useTimeout(callback, delay) {
  // Позволяет передавать как ref, так и обычное значение
  const delayValue = isRef(delay) ? delay.value : delay
  
  setTimeout(() => {
    callback()
  }, delayValue)
}

// Использование
const delayRef = ref(1000)
useTimeout(() => console.log('Привет'), delayRef)
useTimeout(() => console.log('Привет'), 2000)
```

## Совместимость с unref

`isRef` часто используется вместе с `unref` для обработки значений:

```javascript
import { ref, isRef, unref } from 'vue'

function processValue(value) {
  // unref автоматически разворачивает ref или возвращает значение
  return unref(value)
}

// Или с ручной проверкой
function processValueManual(value) {
  return isRef(value) ? value.value : value
}
```

## Пример с параметрами компонентов

```vue
<script setup>
import { ref, isRef, computed } from 'vue'

const props = defineProps({
  title: [String, Object],
  count: [Number, Object]
})

// Проверяем, являются ли пропсы ref'ами
const titleValue = computed(() => isRef(props.title) ? props.title.value : props.title)
const countValue = computed(() => isRef(props.count) ? props.count.value : props.count)
</script>
```

## Работа с массивами и коллекциями

```javascript
import { ref, isRef } from 'vue'

const items = [
  ref(1),
  2,
  ref(3),
  4
]

const resolvedItems = items.map(item => isRef(item) ? item.value : item)
console.log(resolvedItems) // [1, 2, 3, 4]
```

## Использование в условных выражениях

```javascript
import { ref, isRef } from 'vue'

function formatValue(value) {
  if (isRef(value)) {
    return `Ref: ${value.value}`
  } else {
    return `Value: ${value}`
  }
}

const count = ref(42)
console.log(formatValue(count)) // "Ref: 42"
console.log(formatValue(42)) // "Value: 42"
```

## Сравнение с другими проверками

```javascript
import { ref, isRef } from 'vue'

const myRef = ref('test')

// isRef - правильный способ
console.log(isRef(myRef)) // true

// Ручная проверка (не рекомендуется)
console.log(myRef && typeof myRef.value !== 'undefined' && Object.keys(myRef).includes('value')) // тоже true, но не надежно
```

## Практические рекомендации

- Используйте `isRef` для создания гибких API, которые принимают как ref'ы, так и обычные значения
- Полезно при создании библиотек и переиспользуемых компонентов
- Позволяет избежать ошибок при работе с универсальными функциями
- Улучшает типизацию и читаемость кода

## Связь с другими функциями

`isRef` часто используется в связке с другими реактивными утилитами:

- [[IsRef]] + [[Unref]] для безопасного получения значения
- [[IsRef]] + [[ToRefs]] для обработки объектов с ref'ами
- [[IsRef]] + [[Reactive]] для смешанных структур данных

## Типизированные проверки

В TypeScript можно создать типобезопасные функции с использованием `isRef`:

```typescript
import { ref, Ref, isRef } from 'vue'

function isRef<T>(r: Ref<T> | T): r is Ref<T> {
  return (r as Ref<T>).value !== undefined && typeof (r as Ref<T>).value !== 'undefined'
}

// На самом деле это уже встроено в Vue
```

## См. также

- [[Ref]]
- [[Reactive]]
- [[ToRefs]]
- [[Readonly]]
- [[Unref]]