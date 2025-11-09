# Form Submission

## Определение
Отправка форм - это процесс передачи данных, введенных пользователем в форме, на сервер для обработки. Vue предоставляет удобные методы для обработки отправки форм, валидации данных и управления состоянием.

## Базовая отправка форм

### Простая отправка формы
```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.name" placeholder="Имя">
    <input v-model="form.email" type="email" placeholder="Email">
    <button type="submit">Отправить</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({
  name: '',
  email: ''
})

const submitForm = async () => {
  try {
    const response = await fetch('/api/submit-form', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(form.value)
    })
    
    if (response.ok) {
      console.log('Форма успешно отправлена')
      resetForm()
    } else {
      console.error('Ошибка отправки формы')
    }
  } catch (error) {
    console.error('Ошибка сети:', error)
  }
}

const resetForm = () => {
  form.value.name = ''
  form.value.email = ''
}
</script>
```

## Обработка состояния загрузки

### Индикатор загрузки
```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.name" placeholder="Имя" :disabled="loading">
    <input v-model="form.email" type="email" placeholder="Email" :disabled="loading">
    
    <button type="submit" :disabled="loading">
      {{ loading ? 'Отправка...' : 'Отправить' }}
    </button>
    
    <div v-if="message" :class="message.type">
      {{ message.text }}
    </div>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({
  name: '',
  email: ''
})

const loading = ref(false)
const message = ref(null)

const submitForm = async () => {
  loading.value = true
  message.value = null
  
  try {
    const response = await fetch('/api/submit-form', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    
    if (response.ok) {
      message.value = { type: 'success', text: 'Форма успешно отправлена!' }
      resetForm()
    } else {
      message.value = { type: 'error', text: 'Ошибка при отправке формы' }
    }
  } catch (error) {
    message.value = { type: 'error', text: 'Сетевая ошибка' }
  } finally {
    loading.value = false
  }
}

const resetForm = () => {
  form.value = { name: '', email: '' }
}
</script>
```

## Валидация перед отправкой

### Проверка перед отправкой
```vue
<template>
  <form @submit.prevent="submitForm">
    <div class="field-group">
      <input 
        v-model="form.name" 
        placeholder="Имя" 
        :class="{ 'error': errors.name }"
      >
      <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
    </div>
    
    <button type="submit" :disabled="loading">Отправить</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({ name: '', email: '' })
const errors = ref({})
const loading = ref(false)

const validate = () => {
  errors.value = {}
  
  if (!form.value.name.trim()) {
    errors.value.name = 'Имя обязательно'
  }
  
  if (!form.value.email.trim()) {
    errors.value.email = 'Email обязателен'
  } else if (!/\S+@\S+\.\S+/.test(form.value.email)) {
    errors.value.email = 'Email недействителен'
  }
  
  return Object.keys(errors.value).length === 0
}

const submitForm = async () => {
  if (!validate()) return
  
  loading.value = true
  
  try {
    const response = await fetch('/api/submit-form', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    
    if (response.ok) {
      alert('Форма успешно отправлена!')
      resetForm()
    } else {
      throw new Error('Сервер вернул ошибку')
    }
  } catch (error) {
    alert('Ошибка отправки: ' + error.message)
  } finally {
    loading.value = false
  }
}

const resetForm = () => {
  form.value = { name: '', email: '' }
  errors.value = {}
}
</script>
```

## Отправка форм с файлами

### Загрузка файлов
```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.title" placeholder="Заголовок">
    
    <input 
      type="file" 
      ref="fileInput" 
      @change="handleFileChange"
      multiple
    >
    
    <button type="submit" :disabled="loading">Отправить</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({ title: '' })
const files = ref([])
const loading = ref(false)
const fileInput = ref(null)

const handleFileChange = (event) => {
  files.value = Array.from(event.target.files)
}

const submitForm = async () => {
  loading.value = true
  
  try {
    const formData = new FormData()
    formData.append('title', form.value.title)
    
    files.value.forEach((file, index) => {
      formData.append(`file_${index}`, file)
    })
    
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData // Не указываем Content-Type, браузер сам установит multipart/form-data
    })
    
    if (response.ok) {
      console.log('Файлы успешно загружены')
      resetForm()
    }
  } catch (error) {
    console.error('Ошибка загрузки:', error)
  } finally {
    loading.value = false
  }
}

const resetForm = () => {
  form.value = { title: '' }
  files.value = []
  if (fileInput.value) fileInput.value.value = ''
}
</script>
</template>
```

## Обработка разных методов HTTP

### PUT/PATCH/DELETE формы
```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.name" placeholder="Имя">
    <input v-model="form.email" type="email" placeholder="Email">
    
    <button v-if="!isEditing" type="submit">Создать</button>
    <button v-else type="submit">Обновить</button>
    <button v-if="isEditing" type="button" @click="deleteItem">Удалить</button>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps(['initialData'])
const form = ref({ name: '', email: '' })
const loading = ref(false)

// Определяем режим редактирования
const isEditing = computed(() => !!props.initialData)

// Заполняем форму начальными данными при редактировании
if (props.initialData) {
  form.value = { ...props.initialData }
}

const submitForm = async () => {
  loading.value = true
  
  try {
    const method = isEditing.value ? 'PATCH' : 'POST'
    const url = isEditing.value ? `/api/users/${props.initialData.id}` : '/api/users'
    
    const response = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    
    if (response.ok) {
      console.log('Данные успешно сохранены')
      // Обновление родительского компонента
    }
  } catch (error) {
    console.error('Ошибка:', error)
  } finally {
    loading.value = false
  }
}

const deleteItem = async () => {
  if (!confirm('Вы уверены, что хотите удалить?')) return
  
  loading.value = true
  
  try {
    const response = await fetch(`/api/users/${props.initialData.id}`, {
      method: 'DELETE'
    })
    
    if (response.ok) {
      console.log('Данные успешно удалены')
      // Уведомление родительского компонента о удалении
    }
  } catch (error) {
    console.error('Ошибка удаления:', error)
  } finally {
    loading.value = false
  }
}
</script>
</template>
```

## Композабл для форм

### useSubmitForm композабл
```javascript
// useSubmitForm.js
import { ref } from 'vue'

export function useSubmitForm(initialData = {}) {
  const form = ref({ ...initialData })
  const loading = ref(false)
  const errors = ref({})
  const message = ref(null)
  
  const submit = async (url, options = {}) => {
    loading.value = true
    message.value = null
    
    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        body: JSON.stringify(form.value)
      })
      
      const result = await response.json()
      
      if (response.ok) {
        message.value = { type: 'success', text: 'Форма успешно отправлена' }
        return { success: true, data: result }
      } else {
        errors.value = result.errors || {}
        message.value = { type: 'error', text: result.message || 'Ошибка отправки' }
        return { success: false, error: result }
      }
    } catch (error) {
      message.value = { type: 'error', text: 'Сетевая ошибка' }
      return { success: false, error }
    } finally {
      loading.value = false
    }
  }
  
  const reset = () => {
    form.value = { ...initialData }
    errors.value = {}
    message.value = null
  }
  
  return {
    form,
    loading,
    errors,
    message,
    submit,
    reset
  }
}

// Использование в компоненте
<script setup>
import { useSubmitForm } from '@/composables/useSubmitForm'

const { form, loading, errors, message, submit, reset } = useSubmitForm({
  name: '',
  email: ''
})

const handleSubmit = async () => {
  const result = await submit('/api/submit')
  if (result.success) {
    reset()
  }
}
</script>
```

## Обработка ошибок сервера

### Подробная обработка ошибок
```vue
<template>
  <form @submit.prevent="submitForm">
    <div v-for="field in Object.keys(form)" :key="field" class="field-group">
      <input 
        v-model="form[field]" 
        :type="field === 'email' ? 'email' : 'text'"
        :placeholder="field"
        :class="{ 'error': fieldErrors[field] }"
      >
      <span v-if="fieldErrors[field]" class="error-message">
        {{ fieldErrors[field] }}
      </span>
    </div>
    
    <div v-if="generalError" class="error-message general-error">
      {{ generalError }}
    </div>
    
    <button type="submit" :disabled="loading">
      {{ loading ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({
  name: '',
  email: '',
  phone: ''
})

const fieldErrors = ref({})
const generalError = ref(null)
const loading = ref(false)

const submitForm = async () => {
  loading.value = true
  fieldErrors.value = {}
  generalError.value = null
  
  try {
    const response = await fetch('/api/submit-form', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    
    if (response.ok) {
      console.log('Форма успешно отправлена')
      resetForm()
    } else {
      const errorData = await response.json()
      
      if (errorData.field_errors) {
        // Ошибки полей
        fieldErrors.value = errorData.field_errors
      } else if (errorData.message) {
        // Общая ошибка
        generalError.value = errorData.message
      }
    }
  } catch (error) {
    generalError.value = 'Сетевая ошибка при отправке формы'
  } finally {
    loading.value = false
  }
}

const resetForm = () => {
  form.value = { name: '', email: '', phone: '' }
  fieldErrors.value = {}
  generalError.value = null
}
</script>
```

## События жизненного цикла формы

### Сохранение данных при навигации
```vue
<script setup>
import { ref, onBeforeUnmount } from 'vue'

const form = ref({ name: '', email: '' })
const hasUnsavedChanges = ref(false)

const saveDraft = () => {
  localStorage.setItem('formDraft', JSON.stringify(form.value))
  hasUnsavedChanges.value = false
}

const loadDraft = () => {
  const saved = localStorage.getItem('formDraft')
  if (saved) {
    form.value = JSON.parse(saved)
  }
}

// Автосохранение
const autoSaveTimer = setInterval(() => {
  if (hasUnsavedChanges.value) {
    saveDraft()
  }
}, 30000) // каждые 30 секунд

// Сохранение при уходе со страницы
onBeforeUnmount(() => {
  if (hasUnsavedChanges.value) {
    saveDraft()
  }
  clearInterval(autoSaveTimer)
})

const updateForm = (field, value) => {
  form.value[field] = value
  hasUnsavedChanges.value = true
}
</script>
</template>
```

## Лучшие практики

### Безопасность
- Валидируйте данные как на клиенте, так и на сервере
- Используйте CSRF токены для защиты от подделки запросов
- Ограничивайте частоту отправки форм (rate limiting)

### UX
- Показывайте состояние загрузки
- Возвращайте пользователю на форму при ошибках
- Автоматически фокусируйте на поле с ошибкой

## Тестирование отправки форм
```javascript
import { mount } from '@vue/test-utils'
import FormComponent from '@/components/FormComponent.vue'

test('отправляет форму и показывает сообщение об успехе', async () => {
  global.fetch = jest.fn(() =>
    Promise.resolve({
      ok: true,
      json: () => Promise.resolve({ success: true })
    })
  )
  
  const wrapper = mount(FormComponent)
  
  await wrapper.find('input[name="name"]').setValue('John')
  await wrapper.find('input[name="email"]').setValue('john@example.com')
  await wrapper.find('form').trigger('submit.prevent')
  
  expect(fetch).toHaveBeenCalledWith(
    '/api/submit-form',
    expect.objectContaining({
      method: 'POST'
    })
  )
})
```

## Связь с другими концепциями
- [[forms/validation]] - валидация перед отправкой
- [[forms/v-model]] - двусторонняя привязка данных формы
- [[components/Events]] - отправка событий родителю при успехе/ошибке
- [[state-management/State Management]] - сохранение состояния формы