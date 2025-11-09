# Утечки памяти в JavaScript

## Введение

Утечки памяти происходят, когда память, которая больше не нужна приложению, не освобождается сборщиком мусора. Это может привести к постепенному увеличению использования памяти, замедлению работы приложения и в крайних случаях к сбоям.

## Общие причины утечек памяти

### 1. Забытые таймеры и слушатели событий

Одна из самых распространенных причин утечек памяти - неправильное управление таймерами и обработчиками событий.

```javascript
// Плохо: утечка памяти из-за забытого таймера
class DataProcessor {
  constructor() {
    this.data = new Array(1000000).fill(0);
    
    // Таймер ссылается на this, предотвращая сборку мусора
    setInterval(() => {
      console.log(this.data.length);
    }, 1000);
  }
}

const processor = new DataProcessor();
processor = null; // Объект не будет собран, так как таймер активен

// Хорошо: очистка таймера
class BetterDataProcessor {
  constructor() {
    this.data = new Array(1000000).fill(0);
    this.timerId = setInterval(() => {
      console.log(this.data.length);
    }, 1000);
  }
  
  destroy() {
    clearInterval(this.timerId);
    this.data = null;
  }
}

const betterProcessor = new BetterDataProcessor();
// ... использование
betterProcessor.destroy(); // Очистка ресурсов
betterProcessor = null;    // Объект может быть собран
```

### 2. Глобальные переменные

Глобальные переменные всегда доступны и не могут быть собраны, пока приложение работает.

```javascript
// Плохо: глобальные переменные предотвращают сборку мусора
window.globalData = new Array(1000000).fill(0);

// Хорошо: использование локальных переменных и очистка
class DataManager {
  constructor() {
    this.data = new Array(1000000).fill(0);
  }
  
  clear() {
    this.data = null;
  }
}

const manager = new DataManager();
// ... использование
manager.clear(); // Очистка данных
manager = null;  // Объект может быть собран
```

### 3. Замыкания

Замыкания могут удерживать ссылки на большие объекты дольше, чем это необходимо.

```javascript
// Плохо: замыкание удерживает большие объекты
function createEventHandler() {
  const largeData = new Array(1000000).fill(0);
  
  return function() {
    // Эта функция удерживает ссылку на largeData
    console.log('Event handled');
  };
}

const handler = createEventHandler();
// largeData остается в памяти, пока handler существует

// Хорошо: освобождение данных после использования
function createBetterEventHandler() {
  let largeData = new Array(1000000).fill(0);
  
  return function() {
    console.log('Event handled');
    largeData = null; // Освобождение данных
  };
}
```

## Обнаружение утечек памяти

### Использование Performance API

Мониторинг использования памяти может помочь обнаружить утечки:

```javascript
// Мониторинг использования памяти
class MemoryMonitor {
  constructor() {
    this.measurements = [];
    this.startMonitoring();
  }
  
  startMonitoring() {
    setInterval(() => {
      if (performance.memory) {
        const memoryInfo = {
          used: performance.memory.usedJSHeapSize,
          total: performance.memory.totalJSHeapSize,
          limit: performance.memory.jsHeapSizeLimit,
          timestamp: Date.now()
        };
        
        this.measurements.push(memoryInfo);
        console.log('Память:', memoryInfo);
        
        // Проверка на утечку (рост более чем на 10% за 10 измерений)
        if (this.measurements.length > 10) {
          const recent = this.measurements.slice(-10);
          const initial = recent[0].used;
          const current = recent[recent.length - 1].used;
          const growth = (current - initial) / initial;
          
          if (growth > 0.1) {
            console.warn('Возможная утечка памяти обнаружена!');
          }
        }
      }
    }, 1000);
  }
}

// const monitor = new MemoryMonitor(); // Раскомментируйте для мониторинга
```

### Использование WeakMap для предотвращения утечек

WeakMap позволяет хранить метаданные без предотвращения сборки основных объектов:

```javascript
// Использование WeakMap для хранения приватных данных
const privateData = new WeakMap();

class User {
  constructor(name) {
    this.name = name;
    // Приватные данные не предотвращают сборку мусора пользователя
    privateData.set(this, {
      password: "secret",
      lastLogin: new Date()
    });
  }
  
  getPassword() {
    return privateData.get(this).password;
  }
  
  getLastLogin() {
    return privateData.get(this).lastLogin;
  }
}

const user = new User("Иван");
// Когда user становится недостижимым, приватные данные
// также становятся недостижимыми и будут собраны
```

## Инструменты для обнаружения утечек

### Chrome DevTools

Chrome DevTools предоставляет мощные инструменты для анализа памяти:

1. **Memory tab** - для создания heap snapshots и анализа распределения памяти
2. **Performance tab** - для мониторинга сборок мусора
3. **Detached elements** - для поиска отсоединенных DOM элементов

### Практические советы по предотвращению утечек

1. **Всегда удаляйте обработчики событий** при удалении элементов:

```javascript
class Component {
  constructor(element) {
    this.element = element;
    this.handleClick = this.handleClick.bind(this);
    this.element.addEventListener('click', this.handleClick);
  }
  
  handleClick() {
    console.log('Clicked');
  }
  
  destroy() {
    this.element.removeEventListener('click', this.handleClick);
    this.element = null;
  }
}
```

2. **Используйте WeakMap и WeakSet** для хранения метаданных:

```javascript
// Вместо Map, которая может предотвращать сборку объектов
const metadata = new WeakMap();

function attachMetadata(obj, data) {
  metadata.set(obj, data);
}

function getMetadata(obj) {
  return metadata.get(obj);
}
```

3. **Очищайте таймеры и интервалы**:

```javascript
class TimerManager {
  constructor() {
    this.timers = new Set();
  }
  
  setTimeout(callback, delay) {
    const timerId = setTimeout(() => {
      this.timers.delete(timerId);
      callback();
    }, delay);
    
    this.timers.add(timerId);
    return timerId;
  }
  
  clearTimeout(timerId) {
    clearTimeout(timerId);
    this.timers.delete(timerId);
  }
  
  clearAll() {
    for (const timerId of this.timers) {
      clearTimeout(timerId);
    }
    this.timers.clear();
  }
}
```

Предотвращение утечек памяти - важная часть разработки производительных JavaScript приложений. Регулярный мониторинг и следование лучшим практикам помогут избежать проблем с памятью.

#javascript #memory-leaks #performance #debugging