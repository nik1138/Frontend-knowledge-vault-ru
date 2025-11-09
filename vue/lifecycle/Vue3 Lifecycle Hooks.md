# Lifecycle Hooks

Жизненный цикл компонента Vue - это набор хуков, которые вызываются на различных этапах существования компонента. Это позволяет выполнить код в определенные моменты жизненного цикла компонента.

## Фазы жизненного цикла

### Создание (Creation)
Эти хуки вызываются до того, как компонент будет добавлен в DOM.

### Монтирование (Mounting)
Эти хуки вызываются при добавлении компонента в DOM.

### Обновление (Updating)
Эти хуки вызываются при изменении реактивных данных, приводящих к повторному рендеру.

### Уничтожение (Unmounting)
Эти хуки вызываются при удалении компонента из DOM.

## Список хуков

### [setup](setup.md)
Функция setup в Composition API - выполняется до создания компонента

### [beforeCreate](beforeCreate.md)
Вызывается перед созданием компонента

### [created](created.md)
Вызывается после создания компонента

### [beforeMount](beforeMount.md)
Вызывается перед монтированием компонента

### [mounted](mounted.md)
Вызывается после монтирования компонента

### [beforeUpdate](beforeUpdate.md)
Вызывается перед обновлением компонента

### [updated](updated.md)
Вызывается после обновления компонента

### [beforeUnmount](beforeUnmount.md)
Вызывается перед размонтированием компонента

### [unmounted](unmounted.md)
Вызывается после размонтирования компонента

## Composition API vs Options API

В Composition API хуки используются с префиксом 'on':

```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

export default {
  setup() {
    onMounted(() => {
      console.log('Компонент смонтирован')
    })
    
    onUnmounted(() => {
      console.log('Компонент размонтирован')
    })
  }
}
```

## Связь с другими концепциями
- [[reactivity/Reactivity System]] - работа с реактивностью в хуках
- [[components/Components]] - жизненный цикл компонентов
- [[composables/Composables]] - организация логики в хуках