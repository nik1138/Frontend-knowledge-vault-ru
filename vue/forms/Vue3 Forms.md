# Forms

Работа с формами в Vue включает обработку пользовательского ввода, валидацию данных и отправку формы. Vue предоставляет мощные инструменты для создания интерактивных форм.

## Основные концепции

### [v-model](v-model.md)
Двусторонняя привязка данных между формой и компонентом

### [Input Bindings](input-bindings.md)
Привязка различных типов полей ввода (text, checkbox, radio, select)

### [Form Validation](validation.md)
Валидация формы перед отправкой

### [Form Submission](submission.md)
Обработка отправки формы и обработка данных

## Типы ввода

### Текстовые поля
```vue
<template>
  <input v-model="message" placeholder="Введите сообщение">
  <p>Сообщение: {{ message }}</p>
</template>
```

### Чекбоксы
```vue
<template>
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked ? 'Отмечено' : 'Не отмечено' }}</label>
</template>
```

### Радио-кнопки
```vue
<template>
  <input type="radio" id="one" value="Один" v-model="picked">
  <label for="one">Один</label>
  <input type="radio" id="two" value="Два" v-model="picked">
  <label for="two">Два</label>
  <p>Выбрано: {{ picked }}</p>
</template>
```

### Выпадающие списки
```vue
<template>
  <select v-model="selected">
    <option disabled value="">Выберите один</option>
    <option>А</option>
    <option>Б</option>
    <option>В</option>
  </select>
  <span>Выбрано: {{ selected }}</span>
</template>
```

## Модификаторы v-model

### .lazy
```vue
<input v-model.lazy="msg" />
```
Обновляет модель при событии change, а не input

### .number
```vue
<input v-model.number="age" type="number">
```
Преобразует значение к числу

### .trim
```vue
<input v-model.trim="msg">
```
Удаляет пробелы по краям

## Сложные формы

### Работа с массивами
```vue
<template>
  <input
    v-for="item in items"
    type="checkbox"
    :value="item.value"
    v-model="checkedNames"
  >
</template>
```

### Формы с композаблами
```javascript
import { ref } from 'vue'

export function useForm(initialData) {
  const formData = ref({ ...initialData })
  const errors = ref({})
  
  const validate = () => {
    // логика валидации
  }
  
  const submit = async () => {
    if (validate()) {
      // отправка формы
    }
  }
  
  return {
    formData,
    errors,
    validate,
    submit
  }
}
```

## Связь с другими концепциями
- [[directives/v-model]] - основная директива для форм
- [[components/Components]] - передача данных между компонентами форм
- [[state-management/State Management]] - управление состоянием форм
- [[testing/Testing]] - тестирование форм и валидации