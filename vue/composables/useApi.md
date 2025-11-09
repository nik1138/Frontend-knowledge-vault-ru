# {{useApi}}

## Цель
Типобезопасные HTTP-запросы с отменой и обработкой ошибок.

## Типизация
```ts
interface ApiOptions {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  headers?: Record<string, string>;
  body?: BodyInit | null;
  signal?: AbortSignal;
}

export function useApi<T = unknown>(
  url: string,
  options: ApiOptions = {}
): {
   Ref<T | null>;
  error: Ref<Error | null>;
  loading: Ref<boolean>;
  abort: () => void;
} {
  const data = ref<T | null>(null);
  const error = ref<Error | null>(null);
  const loading = ref(false);
  const controller = new AbortController();

  const fetchData = async () => {
    try {
      loading.value = true;
      const response = await fetch(url, {
        ...options,
        signal: controller.signal,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      data.value = await response.json();
    } catch (e) {
      if (e instanceof Error && e.name !== 'AbortError') {
        error.value = e;
      }
    } finally {
      loading.value = false;
    }
  };

  fetchData();

  const abort = () => {
    controller.abort();
    loading.value = false;
  };

  onUnmounted(abort);

  return { data, error, loading, abort };
}
```

## Пример использования
```vue
<script setup lang="ts">
import { useApi } from '@/composables/useApi';
import type { User } from '@/types/User';

// Типизированный GET-запрос
const {  users, error, loading, abort } = useApi<User[]>('/api/users');

// Отмена при размонтировании (автоматически через onUnmounted)
// Но можно вызвать вручную:
function handleCancel() {
  abort();
}

// POST-запрос с параметрами
const { execute: createUser } = useApi('/api/users', {
  method: 'POST',
  body: JSON.stringify({ name: 'Alice' })
});
</script>

<template>
  <div>
    <button @click="handleCancel" :disabled="!loading">
      Отменить загрузку
    </button>
    
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error">Ошибка: {{ error.message }}</div>
    <ul v-else>
      <li v-for="user in users" :key="user.id">
        {{ user.name }}
      </li>
    </ul>
  </div>
</template>
```

### Связанные типы
- [[ApiResponse]]
- [[ApiError]]
- [[AbortController-wrapper]]

### Источники
- https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API?spm=a2ty_o01.29997173.0.0.3701c921HUvzSz
- https://vuejs.org/guide/reusability/composables.html?spm=a2ty_o01.29997173.0.0.3701c921HUvzSz
- https://www.typescriptlang.org/docs/handbook/2/generics.html?spm=a2ty_o01.29997173.0.0.3701c921HUvzSz
- https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal?spm=a2ty_o01.29997173.0.0.3701c921HUvzSz