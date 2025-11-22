---
tags: [javascript, frontend, advanced, generators, iterators]
aliases: [генераторы в js для фронтенда, практические примеры генераторов, итераторы в javascript]
---

# Генераторы в JavaScript - Практические примеры для Frontend разработки

## Введение

Генераторы - это специальный тип функций в JavaScript, которые могут приостанавливать свое выполнение и возобновлять его позже. Они особенно полезны при работе с потоками данных, асинхронными операциями и итерациями в фронтенд разработке. В этой статье рассмотрим практические примеры использования генераторов с акцентом на задачи фронтенд разработки.

## Что такое генераторы?

Генераторы создаются с использованием ключевого слова `function*` и используют `yield` для возврата значений по одному за раз. Генераторы возвращают объект-итератор, который можно использовать для получения значений по мере необходимости.

## Практический пример: Пагинация данных

```javascript
// Генератор для постраничной загрузки данных
function* createDataPaginator(apiEndpoint, pageSize = 10) {
  let currentPage = 1;
  let hasMore = true;
  
  while (hasMore) {
    try {
      const response = yield fetch(`${apiEndpoint}?page=${currentPage}&size=${pageSize}`);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data = yield response.json();
      
      // Возвращаем текущую страницу данных
      yield {
        page: currentPage,
        data: data.items || data,
        hasMore: data.hasMore !== false && data.items && data.items.length === pageSize
      };
      
      hasMore = data.hasMore !== false && data.items && data.items.length === pageSize;
      currentPage++;
    } catch (error) {
      console.error('Ошибка загрузки страницы:', error);
      hasMore = false;
      yield { error, hasMore: false };
    }
  }
}

// Использование пагинатора данных
async function loadMoreData(paginator) {
  const result = paginator.next();
  
  if (!result.done) {
    const response = await result.value;
    const pageData = await paginator.next(response).value;
    
    if (pageData.error) {
      console.error('Ошибка загрузки:', pageData.error);
      return { hasMore: false };
    }
    
    // Обновление UI с новыми данными
    appendDataToUI(pageData.data);
    
    return {
      hasMore: pageData.hasMore,
      page: pageData.page
    };
  }
  
  return { hasMore: false };
}

// Использование в UI
const dataPaginator = createDataPaginator('/api/products', 20);

document.getElementById('load-more-btn').addEventListener('click', async () => {
  const result = await loadMoreData(dataPaginator);
  
  if (!result.hasMore) {
    document.getElementById('load-more-btn').style.display = 'none';
    showEndOfListMessage();
  }
});

function appendDataToUI(data) {
  const container = document.getElementById('data-container');
  
  data.forEach(item => {
    const element = document.createElement('div');
    element.className = 'data-item';
    element.innerHTML = `
      <h3>${item.title}</h3>
      <p>${item.description}</p>
    `;
    container.appendChild(element);
  });
}
```

## Практический пример: Анимации с кадрами

```javascript
// Генератор для создания плавных анимаций
function* createAnimationGenerator(duration, easing = 'linear') {
  const startTime = performance.now();
  const endTime = startTime + duration;
  
  while (true) {
    const currentTime = performance.now();
    let progress = Math.min(1, (currentTime - startTime) / duration);
    
    // Применение easing функции
    switch (easing) {
      case 'ease-in':
        progress = Math.pow(progress, 2);
        break;
      case 'ease-out':
        progress = 1 - Math.pow(1 - progress, 2);
        break;
      case 'ease-in-out':
        progress = progress < 0.5 
          ? 2 * progress * progress 
          : 1 - Math.pow(-2 * progress + 2, 2) / 2;
        break;
    }
    
    yield {
      progress,
      time: currentTime,
      isComplete: currentTime >= endTime
    };
    
    if (currentTime >= endTime) {
      return { progress: 1, time: endTime, isComplete: true };
    }
    
    // Ждем следующего кадра
    yield new Promise(resolve => requestAnimationFrame(resolve));
  }
}

// Использование генератора анимации
async function animateElement(element, duration = 1000) {
  const animation = createAnimationGenerator(duration, 'ease-out');
  
  for (const frame of take(Infinity, animation)) {
    if (frame.isComplete) break;
    
    // Ожидание следующего кадра
    await frame.value;
    
    // Обновление анимации
    element.style.transform = `translateX(${frame.progress * 200}px)`;
    element.style.opacity = frame.progress;
  }
}

// Вспомогательная функция для ограничения итераций
function* take(n, iterable) {
  let count = 0;
  for (const value of iterable) {
    if (count >= n) return;
    yield value;
    count++;
  }
}

// Использование анимации
const animatedElement = document.getElementById('animated-box');
animateElement(animatedElement, 2000);
```

## Практический пример: Управление очередью задач

```javascript
// Генератор для управления очередью асинхронных задач
function* createTaskQueue(concurrency = 1) {
  const queue = [];
  const running = new Set();
  
  while (true) {
    // Добавление новой задачи
    const { task, resolve, reject } = yield;
    
    queue.push({ task, resolve, reject });
    
    // Запуск задач в соответствии с лимитом
    while (queue.length > 0 && running.size < concurrency) {
      const { task, resolve, reject } = queue.shift();
      
      const taskPromise = task()
        .then(result => resolve(result))
        .catch(error => reject(error))
        .finally(() => {
          running.delete(taskPromise);
        });
      
      running.add(taskPromise);
    }
  }
}

// Класс для управления очередью задач
class TaskManager {
  constructor(concurrency = 3) {
    this.queue = createTaskQueue(concurrency);
    this.queue.next(); // Инициализация генератора
  }
  
  addTask(task) {
    return new Promise((resolve, reject) => {
      this.queue.next({ task, resolve, reject });
    });
  }
}

// Использование менеджера задач
const taskManager = new TaskManager(2); // Максимум 2 параллельные задачи

// Функция для загрузки изображения
async function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => resolve(img);
    img.onerror = () => reject(new Error(`Не удалось загрузить изображение: ${src}`));
    img.src = src;
  });
}

// Добавление задач загрузки изображений
const imageSources = [
  '/images/photo1.jpg',
  '/images/photo2.jpg',
  '/images/photo3.jpg',
  '/images/photo4.jpg',
  '/images/photo5.jpg'
];

imageSources.forEach(src => {
  taskManager.addTask(() => loadImage(src))
    .then(img => {
      document.getElementById('gallery').appendChild(img);
    })
    .catch(error => {
      console.error('Ошибка загрузки изображения:', error);
    });
});
```

## Практический пример: Обработка больших наборов данных

```javascript
// Генератор для обработки больших наборов данных
function* createDataProcessor(data, batchSize = 100) {
  for (let i = 0; i < data.length; i += batchSize) {
    const batch = data.slice(i, i + batchSize);
    
    // Выполняем обработку пакета
    const processedBatch = batch.map(item => ({
      ...item,
      processed: true,
      processedAt: new Date().toISOString()
    }));
    
    yield processedBatch;
    
    // Даем браузеру возможность обработать другие задачи
    yield new Promise(resolve => setTimeout(resolve, 0));
  }
}

// Использование процессора данных
async function processLargeDataset(data) {
  const processor = createDataProcessor(data, 50);
  const results = [];
  
  for (const batch of processor) {
    if (Array.isArray(batch)) {
      results.push(...batch);
      
      // Обновление прогресса
      const progress = Math.min(100, (results.length / data.length) * 100);
      updateProgressBar(progress);
    } else {
      // Это промис для уступки контроля
      await batch;
    }
  }
  
  return results;
}

function updateProgressBar(percentage) {
  const progressBar = document.getElementById('progress-bar');
  const progressText = document.getElementById('progress-text');
  
  if (progressBar) {
    progressBar.style.width = `${percentage}%`;
  }
  
  if (progressText) {
    progressText.textContent = `Обработано: ${Math.round(percentage)}%`;
  }
}

// Пример использования
const largeDataset = Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `Item ${i}`,
  value: Math.random() * 100
}));

processLargeDataset(largeDataset)
  .then(results => {
    console.log('Обработка завершена:', results.length, 'элементов');
    displayResults(results);
  })
  .catch(error => {
    console.error('Ошибка обработки:', error);
  });
```

## Практический пример: Создание потоков данных

```javascript
// Генератор для создания потоков данных с преобразованиями
function* createDataStream(source) {
  for (const item of source) {
    yield item;
  }
}

// Функции трансформации потока
function* map(stream, transformer) {
  for (const item of stream) {
    yield transformer(item);
  }
}

function* filter(stream, predicate) {
  for (const item of stream) {
    if (predicate(item)) {
      yield item;
    }
  }
}

function* take(stream, count) {
  let taken = 0;
  for (const item of stream) {
    if (taken >= count) return;
    yield item;
    taken++;
  }
}

// Использование потоков данных
function processUserStream(users) {
  const stream = createDataStream(users);
  
  // Применяем цепочку трансформаций
  const processedStream = take(
    map(
      filter(
        stream,
        user => user.active && user.age >= 18
      ),
      user => ({
        ...user,
        displayName: `${user.firstName} ${user.lastName}`,
        isAdult: true
      })
    ),
    100 // Берем только первые 100 подходящих пользователей
  );
  
  return Array.from(processedStream);
}

// Использование в реальном времени
async function* createLiveUserStream(apiEndpoint) {
  while (true) {
    try {
      const response = await fetch(apiEndpoint);
      const users = await response.json();
      
      for (const user of users) {
        yield user;
      }
      
      // Ждем перед следующим запросом
      await new Promise(resolve => setTimeout(resolve, 5000));
    } catch (error) {
      console.error('Ошибка получения пользователей:', error);
      await new Promise(resolve => setTimeout(resolve, 10000)); // Ждем дольше при ошибке
    }
  }
}

// Использование live stream
async function monitorUsers() {
  const userStream = createLiveUserStream('/api/users/live');
  
  for await (const user of userStream) {
    updateUserList(user);
  }
}

function updateUserList(user) {
  const userList = document.getElementById('user-list');
  const userElement = document.createElement('div');
  userElement.className = 'user-item';
  userElement.innerHTML = `
    <span class="user-name">${user.displayName}</span>
    <span class="user-status">${user.active ? 'Активен' : 'Неактивен'}</span>
  `;
  
  userList.prepend(userElement);
  
  // Ограничиваем количество элементов в списке
  if (userList.children.length > 50) {
    userList.removeChild(userList.lastChild);
  }
}
```

## Практический пример: Управление состоянием с генераторами

```javascript
// Генератор для управления состоянием приложения
function* createStateManager(initialState = {}) {
  let state = { ...initialState };
  const listeners = [];
  
  while (true) {
    const { type, payload } = yield;
    
    // Обработка действий
    switch (type) {
      case 'SET':
        const { key, value } = payload;
        state = { ...state, [key]: value };
        break;
        
      case 'UPDATE':
        state = { ...state, ...payload };
        break;
        
      case 'RESET':
        state = { ...initialState };
        break;
        
      default:
        console.warn('Неизвестное действие:', type);
    }
    
    // Уведомление слушателей
    listeners.forEach(listener => listener(state));
    
    // Возврат текущего состояния
    yield state;
  }
}

// Класс для управления состоянием
class StateManager {
  constructor(initialState = {}) {
    this.generator = createStateManager(initialState);
    this.generator.next(); // Инициализация
    this.listeners = [];
  }
  
  dispatch(action) {
    const result = this.generator.next(action);
    return result.value;
  }
  
  subscribe(listener) {
    this.listeners.push(listener);
    
    // Возвращаем функцию для отписки
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  getState() {
    const result = this.generator.next();
    return result.value;
  }
}

// Использование менеджера состояния
const stateManager = new StateManager({
  user: null,
  theme: 'light',
  notifications: true
});

// Подписка на изменения состояния
const unsubscribe = stateManager.subscribe((newState) => {
  console.log('Состояние изменилось:', newState);
  
  // Обновление UI при изменении состояния
  updateUI(newState);
});

// Диспетчеризация действий
stateManager.dispatch({
  type: 'SET',
  payload: { key: 'user', value: { name: 'John', email: 'john@example.com' } }
});

stateManager.dispatch({
  type: 'UPDATE',
  payload: { theme: 'dark', language: 'ru' }
});

function updateUI(state) {
  // Обновление элементов UI на основе состояния
  if (state.user) {
    document.getElementById('user-name').textContent = state.user.name;
  }
  
  document.body.className = state.theme;
}
```

## Современное использование генераторов с async/await

```javascript
// Генератор с асинхронными операциями
async function* createAsyncDataProcessor(dataSource) {
  for await (const data of dataSource) {
    // Асинхронная обработка данных
    const processed = await expensiveAsyncOperation(data);
    yield processed;
  }
}

// Асинхронный источник данных (например, из формы или API)
async function* createFormDataStream(formElement) {
  const formData = new FormData(formElement);
  
  for (const [key, value] of formData.entries()) {
    yield { key, value };
  }
}

// Использование асинхронного генератора
async function processFormData(formElement) {
  const stream = createAsyncDataProcessor(createFormDataStream(formElement));
  
  for await (const processedData of stream) {
    console.log(`Обработано поле: ${processedData.key} = ${processedData.value}`);
    updateFieldStatus(processedData.key, 'processed');
  }
}

async function expensiveAsyncOperation(data) {
  // Симуляция дорогой асинхронной операции
  await new Promise(resolve => setTimeout(resolve, 100));
  
  return {
    ...data,
    processed: true,
    timestamp: Date.now()
  };
}

function updateFieldStatus(fieldName, status) {
  const field = document.querySelector(`[name="${fieldName}"]`);
  if (field) {
    field.classList.add(`field-${status}`);
  }
}
```

## Лучшие практики использования генераторов в фронтенд разработке

### 1. Используйте генераторы для управления ресурсоемкими операциями

```javascript
// Хорошо - генератор позволяет уступать контроль браузеру
function* processLargeArray(array, chunkSize = 100) {
  for (let i = 0; i < array.length; i += chunkSize) {
    const chunk = array.slice(i, i + chunkSize);
    
    // Выполняем обработку чанка
    yield chunk.map(item => processItem(item));
    
    // Уступаем контроль браузеру
    yield new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

### 2. Обработка ошибок в генераторах

```javascript
// Обработка ошибок в генераторе
function* safeDataProcessor(data) {
  for (const item of data) {
    try {
      yield processItem(item);
    } catch (error) {
      console.error('Ошибка обработки элемента:', error);
      yield { error: true, original: item };
    }
  }
}
```

## Заключение

Генераторы в JavaScript фронтенд разработке позволяют создавать эффективные, гибкие и управляемые потоки данных. Они особенно полезны для обработки больших наборов данных, создания анимаций, управления очередями задач и реализации сложной логики без блокировки основного потока выполнения.

## См. также

- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
- [[События в JavaScript для фронтенд разработки]]