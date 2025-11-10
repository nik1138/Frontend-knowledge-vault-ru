# Performance API

## Введение

Performance API предоставляет мощные инструменты для измерения и оптимизации производительности веб-приложений. В этой главе мы рассмотрим все аспекты производительности: измерение времени выполнения, профилирование, оптимизация рендеринга, управление памятью и другие техники повышения производительности.

## Содержание

- [[API измерения производительности]]
- [[Измерение времени выполнения]]
- [[Профилирование производительности]]
- [[Оптимизация рендеринга]]
- [[Управление памятью]]
- [[Оптимизация сетевых запросов]]
- [[Lazy loading и код-сплиттинг]]
- [[Анализ производительности в продакшене]]

## API измерения производительности

### Performance.now()

`performance.now()` предоставляет более точное время, чем `Date.now()`, с точностью до микросекунд.

```javascript
// Основы Performance API
class PerformanceMonitor {
    constructor() {
        this.entries = new Map();
        this.markers = new Map();
    }
    
    // Создание метки времени
    mark(name) {
        performance.mark(name);
        this.markers.set(name, performance.now());
    }
    
    // Измерение времени между метками
    measure(name, startMark, endMark) {
        performance.measure(name, startMark, endMark);
        const measure = performance.getEntriesByName(name)[0];
        this.entries.set(name, measure);
        return measure;
    }
    
    // Получение измерений
    getMeasure(name) {
        return this.entries.get(name) || null;
    }
    
    // Получение всех измерений
    getAllMeasures() {
        return Array.from(this.entries.values());
    }
    
    // Очистка
    clear() {
        this.entries.clear();
        this.markers.clear();
        performance.clearMarks();
        performance.clearMeasures();
    }
    
    // Измерение выполнения функции
    async measureFunction(name, fn) {
        this.mark(`${name}-start`);
        
        try {
            const result = await fn();
            this.mark(`${name}-end`);
            this.measure(name, `${name}-start`, `${name}-end`);
            return result;
        } catch (error) {
            this.mark(`${name}-end`);
            this.measure(name, `${name}-start`, `${name}-end`);
            throw error;
        }
    }
}

// Пример использования
const monitor = new PerformanceMonitor();

// Измерение загрузки данных
monitor.measureFunction('loadUserData', async () => {
    const response = await fetch('/api/user');
    return response.json();
});

// Измерение рендеринга
monitor.mark('render-start');
renderComponent();
monitor.mark('render-end');
monitor.measure('render-time', 'render-start', 'render-end');

// Получение результатов
const renderTime = monitor.getMeasure('render-time');
console.log(`Время рендеринга: ${renderTime.duration.toFixed(2)}ms`);
```

### PerformanceObserver

`PerformanceObserver` позволяет наблюдать за различными типами производительностных событий.

```javascript
// Наблюдение за производительностными событиями
class PerformanceObserverManager {
    constructor() {
        this.observers = new Map();
        this.performanceData = {
            navigation: [],
            resource: [],
            paint: [],
            measure: [],
            longtask: []
        };
    }
    
    // Наблюдение за навигацией
    observeNavigation() {
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                this.performanceData.navigation.push({
                    name: entry.name,
                    startTime: entry.startTime,
                    duration: entry.duration,
                    ...entry
                });
                
                console.log('Навигация:', {
                    url: entry.name,
                    loadTime: entry.loadEventEnd - entry.fetchStart
                });
            });
        });
        
        observer.observe({ entryTypes: ['navigation'] });
        this.observers.set('navigation', observer);
    }
    
    // Наблюдение за ресурсами
    observeResources() {
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                this.performanceData.resource.push({
                    name: entry.name,
                    duration: entry.duration,
                    startTime: entry.startTime,
                    ...entry
                });
                
                // Обнаружение медленных ресурсов
                if (entry.duration > 1000) { // > 1 секунды
                    console.warn('Медленный ресурс:', {
                        url: entry.name,
                        duration: entry.duration
                    });
                }
            });
        });
        
        observer.observe({ entryTypes: ['resource'] });
        this.observers.set('resource', observer);
    }
    
    // Наблюдение за отрисовкой
    observePaint() {
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                this.performanceData.paint.push({
                    name: entry.name,
                    startTime: entry.startTime,
                    ...entry
                });
                
                console.log(`${entry.name}: ${entry.startTime}ms`);
            });
        });
        
        observer.observe({ entryTypes: ['paint'] });
        this.observers.set('paint', observer);
    }
    
    // Наблюдение за длительными задачами
    observeLongTasks() {
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach((entry) => {
                this.performanceData.longtask.push({
                    startTime: entry.startTime,
                    duration: entry.duration,
                    ...entry
                });
                
                // Длительные задачи могут блокировать UI
                if (entry.duration > 50) { // > 50ms - считается долгим
                    console.warn('Длительная задача:', {
                        duration: entry.duration,
                        startTime: entry.startTime
                    });
                }
            });
        });
        
        observer.observe({ entryTypes: ['longtask'] });
        this.observers.set('longtask', observer);
    }
    
    // Начать наблюдение за всеми типами
    startAllObservations() {
        this.observeNavigation();
        this.observeResources();
        this.observePaint();
        this.observeLongTasks();
    }
    
    // Получить собранные данные
    getPerformanceData() {
        return { ...this.performanceData };
    }
    
    // Остановить наблюдение
    disconnect() {
        this.observers.forEach(observer => observer.disconnect());
        this.observers.clear();
    }
}

// Использование
const perfObserver = new PerformanceObserverManager();
perfObserver.startAllObservations();
```

## Измерение времени выполнения

### Точное измерение производительности

```javascript
// Продвинутый таймер производительности
class AdvancedPerformanceTimer {
    constructor() {
        this.timers = new Map();
        this.histograms = new Map();
    }
    
    start(name) {
        const startTime = performance.now();
        this.timers.set(name, {
            startTime,
            startMark: `start-${name}-${Date.now()}`,
            calls: 0,
            total: 0,
            min: Infinity,
            max: 0
        });
        
        performance.mark(this.timers.get(name).startMark);
    }
    
    end(name) {
        const timer = this.timers.get(name);
        if (!timer) {
            throw new Error(`Таймер ${name} не найден`);
        }
        
        const endTime = performance.now();
        const duration = endTime - timer.startTime;
        
        // Обновление статистики
        timer.calls++;
        timer.total += duration;
        timer.min = Math.min(timer.min, duration);
        timer.max = Math.max(timer.max, duration);
        
        const endMark = `end-${name}-${Date.now()}`;
        performance.mark(endMark);
        performance.measure(name, timer.startMark, endMark);
        
        // Обновление гистограммы
        this.updateHistogram(name, duration);
        
        return duration;
    }
    
    updateHistogram(name, value) {
        if (!this.histograms.has(name)) {
            this.histograms.set(name, []);
        }
        
        const histogram = this.histograms.get(name);
        histogram.push(value);
        
        // Ограничение размера для экономии памяти
        if (histogram.length > 1000) {
            histogram.shift(); // Удалить старые значения
        }
    }
    
    getStats(name) {
        const timer = this.timers.get(name);
        if (!timer) return null;
        
        const histogram = this.histograms.get(name) || [];
        const avg = timer.calls > 0 ? timer.total / timer.calls : 0;
        
        // Вычисление медианы
        const sorted = [...histogram].sort((a, b) => a - b);
        const median = sorted.length > 0 
            ? sorted[Math.floor(sorted.length / 2)] 
            : 0;
        
        return {
            name,
            calls: timer.calls,
            total: timer.total,
            avg,
            min: timer.min,
            max: timer.max,
            median,
            p95: this.getPercentile(sorted, 95),
            p99: this.getPercentile(sorted, 99)
        };
    }
    
    getPercentile(sortedArray, percentile) {
        if (sortedArray.length === 0) return 0;
        
        const index = Math.ceil(percentile / 100 * sortedArray.length) - 1;
        return sortedArray[Math.max(0, Math.min(index, sortedArray.length - 1))];
    }
    
    getAllStats() {
        const stats = [];
        for (const [name] of this.timers) {
            stats.push(this.getStats(name));
        }
        return stats;
    }
    
    // Измерение выполнения асинхронной функции
    async measureAsync(name, fn) {
        this.start(name);
        try {
            const result = await fn();
            this.end(name);
            return result;
        } catch (error) {
            this.end(name);
            throw error;
        }
    }
    
    // Измерение выполнения синхронной функции
    measureSync(name, fn) {
        this.start(name);
        try {
            const result = fn();
            this.end(name);
            return result;
        } catch (error) {
            this.end(name);
            throw error;
        }
    }
}

// Использование
const timer = new AdvancedPerformanceTimer();

// Измерение синхронной операции
timer.measureSync('array-sorting', () => {
    const arr = Array.from({length: 10000}, () => Math.random());
    return arr.sort((a, b) => a - b);
});

// Измерение асинхронной операции
await timer.measureAsync('api-call', async () => {
    const response = await fetch('/api/data');
    return response.json();
});

// Получение статистики
const stats = timer.getStats('array-sorting');
console.log(`Сортировка массива:`, stats);
```

### Профилирование алгоритмов

```javascript
// Класс для профилирования алгоритмов
class AlgorithmProfiler {
    constructor() {
        this.results = new Map();
    }
    
    // Профилирование с различными размерами данных
    async profileAlgorithm(algorithmName, algorithm, testSizes) {
        const results = [];
        
        for (const size of testSizes) {
            const testData = this.generateTestData(size);
            
            const startTime = performance.now();
            await algorithm(testData);
            const endTime = performance.now();
            
            results.push({
                size,
                time: endTime - startTime,
                memory: this.getMemoryUsage() // Условно, т.к. точное измерение памяти ограничено
            });
        }
        
        this.results.set(algorithmName, results);
        return results;
    }
    
    generateTestData(size) {
        // Генерация тестовых данных разных типов
        switch (size) {
            case 'small':
                return Array.from({length: 100}, () => Math.random());
            case 'medium':
                return Array.from({length: 1000}, () => Math.random());
            case 'large':
                return Array.from({length: 10000}, () => Math.random());
            default:
                return Array.from({length: size}, () => Math.random());
        }
    }
    
    getMemoryUsage() {
        // В реальных условиях можно использовать performance.memory в поддерживаемых браузерах
        if (performance.memory) {
            return {
                used: performance.memory.usedJSHeapSize,
                total: performance.memory.totalJSHeapSize,
                limit: performance.memory.jsHeapSizeLimit
            };
        }
        return null;
    }
    
    // Сравнение алгоритмов
    compareAlgorithms(algorithms) {
        const comparison = {};
        
        for (const [name, results] of this.results) {
            if (algorithms.includes(name)) {
                comparison[name] = this.analyzeResults(results);
            }
        }
        
        return comparison;
    }
    
    analyzeResults(results) {
        const times = results.map(r => r.time);
        const avgTime = times.reduce((sum, time) => sum + time, 0) / times.length;
        const minTime = Math.min(...times);
        const maxTime = Math.max(...times);
        
        // Оценка сложности (условно)
        let complexity = 'unknown';
        if (results.length >= 2) {
            const first = results[0];
            const last = results[results.length - 1];
            const sizeRatio = last.size / first.size;
            const timeRatio = last.time / first.time;
            
            if (timeRatio < sizeRatio * 0.5) complexity = 'O(1) or O(log n)';
            else if (timeRatio < sizeRatio * 2) complexity = 'O(n)';
            else if (timeRatio < sizeRatio * sizeRatio * 2) complexity = 'O(n log n)';
            else complexity = 'O(n²) or worse';
        }
        
        return {
            avgTime,
            minTime,
            maxTime,
            complexity,
            results
        };
    }
}

// Пример профилирования
const profiler = new AlgorithmProfiler();

// Алгоритмы для сравнения
const bubbleSort = async (arr) => {
    const n = arr.length;
    for (let i = 0; i < n - 1; i++) {
        for (let j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
            }
        }
    }
    return arr;
};

const quickSort = async (arr) => {
    if (arr.length <= 1) return arr;
    
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);
    
    return [
        ...(await quickSort(left)),
        ...middle,
        ...(await quickSort(right))
    ];
};

// Профилирование
await profiler.profileAlgorithm('bubble-sort', bubbleSort, [100, 500, 1000]);
await profiler.profileAlgorithm('quick-sort', quickSort, [100, 500, 1000]);

// Сравнение
const comparison = profiler.compareAlgorithms(['bubble-sort', 'quick-sort']);
console.log('Сравнение алгоритмов:', comparison);
```

## Оптимизация рендеринга

### Оптимизация DOM-операций

```javascript
// Менеджер оптимизации рендеринга
class RenderingOptimizer {
    constructor() {
        this.batchOperations = [];
        this.isBatching = false;
        this.observer = null;
        this.setupIntersectionObserver();
    }
    
    setupIntersectionObserver() {
        this.observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    // Элемент видим, можно выполнить отложенные операции
                    this.processDeferredOperations(entry.target);
                }
            });
        });
    }
    
    // Пакетная обработка DOM-операций
    batchDOMOperations(operations) {
        this.batchOperations.push(...operations);
        
        if (!this.isBatching) {
            this.isBatching = true;
            // Объединение операций в один цикл рендеринга
            requestAnimationFrame(() => {
                this.executeBatchOperations();
                this.isBatching = false;
            });
        }
    }
    
    executeBatchOperations() {
        // Группировка операций для минимизации reflow и repaint
        const readOperations = [];
        const writeOperations = [];
        
        this.batchOperations.forEach(op => {
            if (op.type === 'read') {
                readOperations.push(op);
            } else {
                writeOperations.push(op);
            }
        });
        
        // Сначала выполнить все операции чтения
        readOperations.forEach(op => op.execute());
        
        // Затем выполнить все операции записи
        writeOperations.forEach(op => op.execute());
        
        this.batchOperations = [];
    }
    
    // Оптимизация рендеринга списков
    renderList(container, items, renderItem) {
        // Использование DocumentFragment для минимизации reflow
        const fragment = document.createDocumentFragment();
        
        items.forEach(item => {
            const element = renderItem(item);
            fragment.appendChild(element);
        });
        
        // Очистка контейнера и добавление фрагмента за один раз
        container.innerHTML = '';
        container.appendChild(fragment);
    }
    
    // Виртуальный скроллинг
    createVirtualScroller(container, items, itemHeight, renderItem) {
        const viewportHeight = container.clientHeight;
        const itemsPerView = Math.ceil(viewportHeight / itemHeight) + 2; // +2 для буфера
        
        const scroller = document.createElement('div');
        scroller.style.height = `${items.length * itemHeight}px`;
        scroller.style.position = 'relative';
        
        const viewport = document.createElement('div');
        viewport.style.height = `${viewportHeight}px`;
        viewport.style.overflow = 'auto';
        viewport.style.position = 'relative';
        
        const itemsContainer = document.createElement('div');
        itemsContainer.style.position = 'absolute';
        itemsContainer.style.top = '0px';
        itemsContainer.style.width = '100%';
        
        viewport.appendChild(itemsContainer);
        scroller.appendChild(viewport);
        container.appendChild(scroller);
        
        let startIndex = 0;
        let renderCount = 0;
        
        const updateViewport = () => {
            const scrollTop = viewport.scrollTop;
            startIndex = Math.floor(scrollTop / itemHeight);
            const endIndex = Math.min(startIndex + itemsPerView, items.length);
            
            // Обновление позиции контейнера элементов
            itemsContainer.style.top = `${startIndex * itemHeight}px`;
            
            // Очистка и рендер видимых элементов
            itemsContainer.innerHTML = '';
            
            for (let i = startIndex; i < endIndex; i++) {
                if (i < items.length) {
                    const itemElement = renderItem(items[i]);
                    itemElement.style.position = 'absolute';
                    itemElement.style.top = `${(i - startIndex) * itemHeight}px`;
                    itemElement.style.width = '100%';
                    itemsContainer.appendChild(itemElement);
                }
            }
        };
        
        viewport.addEventListener('scroll', () => {
            requestAnimationFrame(updateViewport);
        });
        
        // Инициализация
        updateViewport();
        
        return {
            container: scroller,
            update: (newItems) => {
                items = newItems;
                scroller.style.height = `${items.length * itemHeight}px`;
                updateViewport();
            }
        };
    }
    
    // Ленивая загрузка изображений
    lazyLoadImages(container) {
        const images = container.querySelectorAll('img[data-src]');
        
        images.forEach(img => {
            this.observer.observe(img);
            
            // Загрузка при пересечении
            const handleIntersection = (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        this.loadImage(entry.target);
                        this.observer.unobserve(entry.target);
                    }
                });
            };
            
            this.observer.observe(img);
        });
    }
    
    loadImage(img) {
        const src = img.getAttribute('data-src');
        if (src) {
            img.src = src;
            img.removeAttribute('data-src');
            img.classList.add('loaded');
        }
    }
    
    // Оптимизация анимаций
    createOptimizedAnimation(element, keyframes, options) {
        // Использование transform и opacity для оптимизации производительности
        const animation = element.animate(keyframes, {
            ...options,
            // Использование hardware acceleration
            composite: 'accumulate'
        });
        
        // Оптимизация для анимаций, которые могут вызвать reflow
        element.style.willChange = 'transform, opacity';
        
        animation.addEventListener('finish', () => {
            element.style.willChange = 'auto';
        });
        
        return animation;
    }
}

// Использование оптимизатора рендеринга
const optimizer = new RenderingOptimizer();

// Оптимизированный рендер списка
function renderUserList(users) {
    const container = document.getElementById('user-list');
    
    optimizer.renderList(container, users, (user) => {
        const userElement = document.createElement('div');
        userElement.className = 'user-item';
        userElement.innerHTML = `
            <img src="${user.avatar}" alt="${user.name}">
            <span>${user.name}</span>
        `;
        return userElement;
    });
}
```

### Оптимизация React-компонентов

```javascript
// Оптимизация React-компонентов
import React, { memo, useMemo, useCallback, useState, useEffect } from 'react';

// Использование memo для предотвращения ненужных перерисовок
const OptimizedUserCard = memo(({ user, onEdit, onDelete }) => {
    // Мемоизированные значения
    const userInfo = useMemo(() => ({
        name: user.name,
        email: user.email,
        avatar: user.avatar
    }), [user.name, user.email, user.avatar]);
    
    // Мемоизированные колбэки
    const handleEdit = useCallback(() => {
        onEdit(user.id);
    }, [user.id, onEdit]);
    
    const handleDelete = useCallback(() => {
        onDelete(user.id);
    }, [user.id, onDelete]);
    
    return (
        <div className="user-card">
            <img src={userInfo.avatar} alt={userInfo.name} />
            <div>
                <h3>{userInfo.name}</h3>
                <p>{userInfo.email}</p>
            </div>
            <button onClick={handleEdit}>Редактировать</button>
            <button onClick={handleDelete}>Удалить</button>
        </div>
    );
}, (prevProps, nextProps) => {
    // Пользовательская функция сравнения
    return prevProps.user.id === nextProps.user.id &&
           prevProps.user.name === nextProps.user.name &&
           prevProps.user.email === nextProps.user.email &&
           prevProps.user.avatar === nextProps.user.avatar;
});

// Виртуальный скроллинг для React
const VirtualList = ({ items, itemHeight, renderItem, containerHeight = 400 }) => {
    const [scrollTop, setScrollTop] = useState(0);
    
    const startIndex = Math.floor(scrollTop / itemHeight);
    const visibleCount = Math.ceil(containerHeight / itemHeight) + 2; // +2 для буфера
    const endIndex = Math.min(startIndex + visibleCount, items.length);
    
    const visibleItems = items.slice(startIndex, endIndex);
    
    const handleScroll = useCallback((event) => {
        setScrollTop(event.target.scrollTop);
    }, []);
    
    return (
        <div 
            className="virtual-list"
            style={{ 
                height: containerHeight, 
                overflow: 'auto',
                position: 'relative'
            }}
            onScroll={handleScroll}
        >
            <div style={{ 
                height: items.length * itemHeight, 
                position: 'relative' 
            }}>
                <div style={{ 
                    position: 'absolute', 
                    top: startIndex * itemHeight,
                    width: '100%'
                }}>
                    {visibleItems.map((item, index) => (
                        <div 
                            key={item.id || index}
                            style={{ height: itemHeight }}
                        >
                            {renderItem(item, startIndex + index)}
                        </div>
                    ))}
                </div>
            </div>
        </div>
    );
};

// Ленивая загрузка компонентов
const LazyLoadedComponent = React.lazy(() => 
    import('./HeavyComponent').then(module => ({
        default: module.HeavyComponent
    }))
);

const LazyComponentWrapper = () => (
    <React.Suspense fallback={<div>Загрузка...</div>}>
        <LazyLoadedComponent />
    </React.Suspense>
);
```

## Управление памятью

### Отслеживание утечек памяти

```javascript
// Менеджер управления памятью
class MemoryManager {
    constructor() {
        this.objectRegistry = new Map();
        this.eventListeners = new Map();
        this.intervals = new Set();
        this.timeouts = new Set();
        this.observables = new Set();
        this.setupMemoryMonitoring();
    }
    
    setupMemoryMonitoring() {
        // Мониторинг памяти (где поддерживается)
        if ('memory' in performance) {
            setInterval(() => {
                this.checkMemoryUsage();
            }, 5000); // Проверка каждые 5 секунд
        }
    }
    
    checkMemoryUsage() {
        if (performance.memory) {
            const memory = performance.memory;
            const usagePercent = (memory.usedJSHeapSize / memory.jsHeapSizeLimit) * 100;
            
            if (usagePercent > 80) {
                console.warn('Высокое использование памяти:', {
                    used: this.formatBytes(memory.usedJSHeapSize),
                    total: this.formatBytes(memory.jsHeapSizeLimit),
                    percent: usagePercent.toFixed(2) + '%'
                });
                
                this.suggestGarbageCollection();
            }
        }
    }
    
    formatBytes(bytes) {
        if (bytes === 0) return '0 Bytes';
        const k = 1024;
        const sizes = ['Bytes', 'KB', 'MB', 'GB'];
        const i = Math.floor(Math.log(bytes) / Math.log(k));
        return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    }
    
    suggestGarbageCollection() {
        // В реальных условиях можно использовать performance.mark для отладки
        // или вызвать GC в отладочных целях (Chrome DevTools)
        console.log('Рекомендуется очистка памяти');
    }
    
    // Регистрация объекта для отслеживания
    registerObject(obj, name) {
        const id = Symbol(name);
        this.objectRegistry.set(id, obj);
        return id;
    }
    
    // Удаление объекта из реестра
    unregisterObject(id) {
        this.objectRegistry.delete(id);
    }
    
    // Отслеживание обработчиков событий
    addEventListener(element, event, handler) {
        element.addEventListener(event, handler);
        
        // Регистрация для последующего удаления
        const key = `${element.tagName}-${event}-${handler.name || 'anonymous'}`;
        if (!this.eventListeners.has(key)) {
            this.eventListeners.set(key, []);
        }
        this.eventListeners.get(key).push({ element, event, handler });
        
        return () => this.removeEventListener(element, event, handler);
    }
    
    removeEventListener(element, event, handler) {
        element.removeEventListener(event, handler);
        
        const key = `${element.tagName}-${event}-${handler.name || 'anonymous'}`;
        if (this.eventListeners.has(key)) {
            const listeners = this.eventListeners.get(key);
            const index = listeners.findIndex(l => l.handler === handler);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        }
    }
    
    // Управление таймерами
    setInterval(callback, delay) {
        const intervalId = setInterval(callback, delay);
        this.intervals.add(intervalId);
        return intervalId;
    }
    
    clearInterval(intervalId) {
        clearInterval(intervalId);
        this.intervals.delete(intervalId);
    }
    
    setTimeout(callback, delay) {
        const timeoutId = setTimeout(callback, delay);
        this.timeouts.add(timeoutId);
        return timeoutId;
    }
    
    clearTimeout(timeoutId) {
        clearTimeout(timeoutId);
        this.timeouts.delete(timeoutId);
    }
    
    // Очистка всех ресурсов
    cleanup() {
        // Очистка таймеров
        this.intervals.forEach(id => clearInterval(id));
        this.timeouts.forEach(id => clearTimeout(id));
        this.intervals.clear();
        this.timeouts.clear();
        
        // Очистка обработчиков событий
        this.eventListeners.forEach(listeners => {
            listeners.forEach(({ element, event, handler }) => {
                element.removeEventListener(event, handler);
            });
        });
        this.eventListeners.clear();
        
        // Очистка других ресурсов
        this.objectRegistry.clear();
        this.observables.clear();
    }
    
    // Анализ утечек памяти
    analyzeLeaks() {
        const report = {
            objectCount: this.objectRegistry.size,
            eventListenersCount: this.eventListeners.size,
            intervalsCount: this.intervals.size,
            timeoutsCount: this.timeouts.size,
            totalMemory: performance.memory ? performance.memory.usedJSHeapSize : null
        };
        
        console.log('Анализ утечек памяти:', report);
        return report;
    }
}

// Использование менеджера памяти
const memoryManager = new MemoryManager();

// Пример компонента с правильным управлением ресурсами
class OptimizedComponent {
    constructor(container) {
        this.container = container;
        this.id = memoryManager.registerObject(this, 'OptimizedComponent');
        this.setupComponent();
    }
    
    setupComponent() {
        // Использование управляемых таймеров
        this.timerId = memoryManager.setInterval(() => {
            this.update();
        }, 1000);
        
        // Использование управляемых обработчиков событий
        this.removeListener = memoryManager.addEventListener(
            this.container, 
            'click', 
            this.handleClick.bind(this)
        );
    }
    
    update() {
        // Обновление компонента
    }
    
    handleClick(event) {
        // Обработка клика
    }
    
    destroy() {
        // Очистка ресурсов
        memoryManager.clearInterval(this.timerId);
        this.removeListener();
        memoryManager.unregisterObject(this.id);
    }
}
```

## Примеры из промышленной разработки

### Комплексная система мониторинга производительности

```javascript
// Комплексная система мониторинга производительности
class PerformanceMonitoringSystem {
    constructor(options = {}) {
        this.options = {
            sampleRate: options.sampleRate || 1.0, // 100% сэмплирование
            uploadUrl: options.uploadUrl || '/api/performance',
            maxBufferSize: options.maxBufferSize || 100,
            uploadInterval: options.uploadInterval || 30000, // 30 секунд
            ...options
        };
        
        this.buffer = [];
        this.isSampling = Math.random() < this.options.sampleRate;
        this.setupMonitoring();
        this.startUploadInterval();
    }
    
    setupMonitoring() {
        if (!this.isSampling) return;
        
        // Мониторинг загрузки страницы
        this.monitorPageLoad();
        
        // Мониторинг производительности взаимодействия
        this.monitorUserInteraction();
        
        // Мониторинг ошибок
        this.monitorErrors();
        
        // Мониторинг долгих задач
        this.monitorLongTasks();
        
        // Мониторинг использования ресурсов
        this.monitorResourceUsage();
    }
    
    monitorPageLoad() {
        // Замер ключевых метрик загрузки
        window.addEventListener('load', () => {
            const navigation = performance.getEntriesByType('navigation')[0];
            if (navigation) {
                this.recordMetric('page_load_time', navigation.loadEventEnd - navigation.fetchStart);
                this.recordMetric('dom_content_loaded', navigation.domContentLoadedEventEnd - navigation.fetchStart);
                this.recordMetric('first_contentful_paint', navigation.loadEventEnd - navigation.fetchStart);
            }
        });
    }
    
    monitorUserInteraction() {
        // Мониторинг времени отклика на взаимодействие
        ['click', 'keypress', 'scroll', 'touchstart'].forEach(eventType => {
            document.addEventListener(eventType, (event) => {
                const startTime = performance.now();
                
                // Использование requestAnimationFrame для измерения времени до следующего кадра
                requestAnimationFrame(() => {
                    const endTime = performance.now();
                    const latency = endTime - startTime;
                    
                    if (latency > 100) { // > 100ms - высокое время отклика
                        this.recordMetric('interaction_latency', latency, {
                            eventType,
                            target: event.target.tagName,
                            timestamp: Date.now()
                        });
                    }
                });
            }, { passive: true });
        });
    }
    
    monitorErrors() {
        window.addEventListener('error', (event) => {
            this.recordError('javascript_error', {
                message: event.message,
                filename: event.filename,
                lineno: event.lineno,
                colno: event.colno,
                stack: event.error?.stack
            });
        });
        
        window.addEventListener('unhandledrejection', (event) => {
            this.recordError('unhandled_promise_rejection', {
                reason: event.reason?.message || String(event.reason),
                stack: event.reason?.stack
            });
        });
    }
    
    monitorLongTasks() {
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver((list) => {
                list.getEntries().forEach((entry) => {
                    if (entry.duration > 50) { // > 50ms - долгая задача
                        this.recordMetric('long_task', entry.duration, {
                            startTime: entry.startTime,
                            duration: entry.duration
                        });
                    }
                });
            });
            
            observer.observe({ entryTypes: ['longtask'] });
        }
    }
    
    monitorResourceUsage() {
        // Мониторинг производительности ресурсов
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver((list) => {
                list.getEntries().forEach((entry) => {
                    if (entry.duration > 1000) { // > 1 секунды
                        this.recordMetric('slow_resource', entry.duration, {
                            name: entry.name,
                            duration: entry.duration,
                            entryType: entry.entryType
                        });
                    }
                });
            });
            
            observer.observe({ entryTypes: ['resource', 'navigation'] });
        }
    }
    
    recordMetric(name, value, metadata = {}) {
        if (!this.isSampling) return;
        
        const metric = {
            name,
            value,
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent,
            ...metadata
        };
        
        this.buffer.push(metric);
        this.checkBufferSize();
    }
    
    recordError(type, error) {
        const errorReport = {
            type,
            ...error,
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent
        };
        
        this.buffer.push(errorReport);
        this.checkBufferSize();
    }
    
    checkBufferSize() {
        if (this.buffer.length >= this.options.maxBufferSize) {
            this.uploadData();
        }
    }
    
    startUploadInterval() {
        setInterval(() => {
            if (this.buffer.length > 0) {
                this.uploadData();
            }
        }, this.options.uploadInterval);
    }
    
    async uploadData() {
        if (this.buffer.length === 0) return;
        
        const dataToSend = [...this.buffer];
        this.buffer = [];
        
        try {
            await fetch(this.options.uploadUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    metrics: dataToSend,
                    session: this.getSessionId(),
                    page: window.location.href
                })
            });
        } catch (error) {
            console.error('Ошибка отправки данных производительности:', error);
            // Возвращаем данные в буфер для повторной попытки
            this.buffer.unshift(...dataToSend);
        }
    }
    
    getSessionId() {
        if (!sessionStorage.getItem('perf_session_id')) {
            const sessionId = Math.random().toString(36).substr(2, 9);
            sessionStorage.setItem('perf_session_id', sessionId);
        }
        return sessionStorage.getItem('perf_session_id');
    }
    
    // Получение текущих метрик
    getCurrentMetrics() {
        return {
            bufferLength: this.buffer.length,
            isSampling: this.isSampling,
            sampleRate: this.options.sampleRate,
            totalMetrics: this.buffer.length
        };
    }
}

// Инициализация системы мониторинга
const perfSystem = new PerformanceMonitoringSystem({
    sampleRate: 0.1, // 10% сэмплирование для продакшена
    uploadUrl: '/api/performance'
});

// Инструменты для разработки
class PerformanceDevTools {
    static measureFunctionPerformance(fn, iterations = 1000) {
        const times = [];
        
        for (let i = 0; i < iterations; i++) {
            const start = performance.now();
            fn();
            const end = performance.now();
            times.push(end - start);
        }
        
        const avg = times.reduce((sum, time) => sum + time, 0) / times.length;
        const min = Math.min(...times);
        const max = Math.max(...times);
        
        return {
            avg,
            min,
            max,
            median: [...times].sort((a, b) => a - b)[Math.floor(times.length / 2)],
            p95: this.getPercentile(times, 95),
            p99: this.getPercentile(times, 99)
        };
    }
    
    static getPercentile(sortedArray, percentile) {
        const index = Math.ceil(percentile / 100 * sortedArray.length) - 1;
        return sortedArray[Math.max(0, Math.min(index, sortedArray.length - 1))];
    }
    
    // Профилирование памяти (где поддерживается)
    static async profileMemoryUsage() {
        if (!performance.memory) {
            return { error: 'Performance.memory не поддерживается в этом браузере' };
        }
        
        const before = performance.memory;
        // Выполнить какую-то операцию
        const largeArray = new Array(1000000).fill(0).map((_, i) => i);
        const after = performance.memory;
        
        return {
            before,
            after,
            difference: after.usedJSHeapSize - before.usedJSHeapSize
        };
    }
}
```

### Оптимизация загрузки и кэширования

```javascript
// Система оптимизации загрузки ресурсов
class ResourceOptimizationSystem {
    constructor() {
        this.cache = new Map();
        this.loadingPromises = new Map();
        this.preloadQueue = [];
        this.setupIntersectionObserver();
    }
    
    setupIntersectionObserver() {
        this.observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    const url = entry.target.dataset.preloadUrl;
                    if (url) {
                        this.preloadResource(url);
                        this.observer.unobserve(entry.target);
                    }
                }
            });
        }, {
            rootMargin: '100px' // Начинать загрузку за 100px до появления
        });
    }
    
    // Умная загрузка изображений
    async loadImage(src, options = {}) {
        const cacheKey = `img-${src}`;
        
        // Проверка кэша
        if (this.cache.has(cacheKey)) {
            return this.cache.get(cacheKey);
        }
        
        // Проверка на параллельные загрузки
        if (this.loadingPromises.has(cacheKey)) {
            return this.loadingPromises.get(cacheKey);
        }
        
        const loadPromise = new Promise((resolve, reject) => {
            const img = new Image();
            
            if (options.crossOrigin) {
                img.crossOrigin = options.crossOrigin;
            }
            
            img.onload = () => {
                this.cache.set(cacheKey, img);
                this.loadingPromises.delete(cacheKey);
                resolve(img);
            };
            
            img.onerror = (error) => {
                this.loadingPromises.delete(cacheKey);
                reject(new Error(`Ошибка загрузки изображения: ${src}`));
            };
            
            img.src = src;
        });
        
        this.loadingPromises.set(cacheKey, loadPromise);
        return loadPromise;
    }
    
    // Предзагрузка ресурсов
    preloadResource(url, type = 'fetch') {
        if (this.cache.has(url)) return Promise.resolve(this.cache.get(url));
        
        if (this.loadingPromises.has(url)) {
            return this.loadingPromises.get(url);
        }
        
        let loadPromise;
        
        switch (type) {
            case 'image':
                loadPromise = this.loadImage(url);
                break;
            case 'fetch':
                loadPromise = fetch(url).then(response => response.blob());
                break;
            default:
                loadPromise = fetch(url);
        }
        
        this.loadingPromises.set(url, loadPromise);
        
        loadPromise.then(result => {
            this.cache.set(url, result);
            this.loadingPromises.delete(url);
        }).catch(() => {
            this.loadingPromises.delete(url);
        });
        
        return loadPromise;
    }
    
    // Ленивая загрузка с приоритетами
    lazyLoadWithPriority(elements) {
        const highPriority = [];
        const normalPriority = [];
        const lowPriority = [];
        
        elements.forEach(el => {
            const priority = el.dataset.loadPriority || 'normal';
            switch (priority) {
                case 'high':
                    highPriority.push(el);
                    break;
                case 'low':
                    lowPriority.push(el);
                    break;
                default:
                    normalPriority.push(el);
            }
        });
        
        // Загрузка по приоритетам
        this.loadElements(highPriority);
        setTimeout(() => this.loadElements(normalPriority), 100);
        setTimeout(() => this.loadElements(lowPriority), 500);
    }
    
    loadElements(elements) {
        elements.forEach(el => {
            if (el.dataset.src) {
                el.src = el.dataset.src;
                el.classList.add('loaded');
            } else if (el.dataset.href) {
                this.preloadResource(el.dataset.href);
            }
        });
    }
    
    // Кэширование API-ответов
    async fetchWithCaching(url, options = {}, cacheTime = 5 * 60 * 1000) { // 5 минут
        const cacheKey = `api-${url}-${JSON.stringify(options)}`;
        const cached = this.cache.get(cacheKey);
        
        if (cached && Date.now() - cached.timestamp < cacheTime) {
            return cached.data;
        }
        
        if (this.loadingPromises.has(cacheKey)) {
            return this.loadingPromises.get(cacheKey);
        }
        
        const fetchPromise = fetch(url, options)
            .then(response => response.json())
            .then(data => {
                this.cache.set(cacheKey, {
                    data,
                    timestamp: Date.now()
                });
                this.loadingPromises.delete(cacheKey);
                return data;
            })
            .catch(error => {
                this.loadingPromises.delete(cacheKey);
                throw error;
            });
        
        this.loadingPromises.set(cacheKey, fetchPromise);
        return fetchPromise;
    }
    
    // Очистка кэша
    clearCache() {
        this.cache.clear();
        this.loadingPromises.clear();
    }
    
    // Очистка устаревшего кэша
    cleanupExpiredCache() {
        const now = Date.now();
        for (const [key, value] of this.cache) {
            // Предположим, что у нас есть время жизни в значении
            if (value.timestamp && now - value.timestamp > 30 * 60 * 1000) { // 30 минут
                this.cache.delete(key);
            }
        }
    }
}

// Использование системы оптимизации
const resourceOptimizer = new ResourceOptimizationSystem();

// Оптимизированная загрузка галереи
async function loadGallery(images) {
    const gallery = document.getElementById('gallery');
    
    for (const image of images) {
        const imgElement = document.createElement('img');
        imgElement.dataset.src = image.url;
        imgElement.dataset.loadPriority = image.priority || 'normal';
        imgElement.alt = image.alt;
        
        gallery.appendChild(imgElement);
    }
    
    resourceOptimizer.lazyLoadWithPriority(gallery.querySelectorAll('img'));
}
```

## Лучшие практики

### 1. Измерение и мониторинг

```javascript
// Система метрик производительности
class PerformanceMetrics {
    static measureFunction(fn, name) {
        return function(...args) {
            const start = performance.now();
            const result = fn.apply(this, args);
            const end = performance.now();
            
            console.log(`${name} выполнена за ${(end - start).toFixed(2)}ms`);
            return result;
        };
    }
    
    static async measureAsyncFunction(asyncFn, name) {
        return async function(...args) {
            const start = performance.now();
            const result = await asyncFn.apply(this, args);
            const end = performance.now();
            
            console.log(`${name} выполнена за ${(end - start).toFixed(2)}ms`);
            return result;
        };
    }
}

// Использование
const optimizedFunction = PerformanceMetrics.measureFunction(myFunction, 'MyFunction');
const optimizedAsyncFunction = PerformanceMetrics.measureAsyncFunction(myAsyncFunction, 'MyAsyncFunction');
```

### 2. Оптимизация алгоритмов

```javascript
// Оптимизированные структуры данных
class OptimizedDataStructures {
    // Быстрая очередь с O(1) операциями
    static createQueue() {
        return {
            items: [],
            head: 0,
            
            enqueue(item) {
                this.items.push(item);
            },
            
            dequeue() {
                if (this.head === this.items.length) {
                    this.items = [];
                    this.head = 0;
                    return undefined;
                }
                return this.items[this.head++];
            },
            
            size() {
                return this.items.length - this.head;
            },
            
            isEmpty() {
                return this.size() === 0;
            }
        };
    }
    
    // Быстрый поиск в массиве
    static createSearchableArray(arr) {
        const index = new Map();
        
        arr.forEach((item, i) => {
            if (!index.has(item)) {
                index.set(item, []);
            }
            index.get(item).push(i);
        });
        
        return {
            find(value) {
                return index.get(value) || [];
            },
            
            add(item) {
                if (!index.has(item)) {
                    index.set(item, []);
                }
                index.get(item).push(arr.length);
                arr.push(item);
            }
        };
    }
}
```

## Безопасность

При мониторинге производительности важно учитывать безопасность:

```javascript
// Безопасный сбор данных производительности
class SecurePerformanceMonitoring {
    constructor() {
        this.allowedOrigins = ['https://api.example.com'];
        this.sensitiveDataPatterns = [
            /password/i,
            /token/i,
            /key/i,
            /secret/i
        ];
    }
    
    // Безопасная отправка данных производительности
    async sendPerformanceData(data) {
        // Очистка чувствительных данных
        const sanitizedData = this.sanitizeData(data);
        
        // Проверка URL
        const url = new URL('/api/performance', window.location.origin);
        if (!this.isAllowedOrigin(url.origin)) {
            throw new Error('Недопустимый URL для отправки данных');
        }
        
        await fetch(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-Requested-With': 'XMLHttpRequest'
            },
            body: JSON.stringify(sanitizedData)
        });
    }
    
    sanitizeData(data) {
        const sanitized = JSON.parse(JSON.stringify(data));
        
        // Рекурсивная очистка
        this.cleanObject(sanitized);
        return sanitized;
    }
    
    cleanObject(obj) {
        if (typeof obj !== 'object' || obj === null) return;
        
        for (const [key, value] of Object.entries(obj)) {
            if (typeof value === 'string') {
                if (this.containsSensitiveData(key)) {
                    obj[key] = '[REDACTED]';
                }
            } else if (typeof value === 'object') {
                this.cleanObject(value);
            }
        }
    }
    
    containsSensitiveData(str) {
        return this.sensitiveDataPatterns.some(pattern => pattern.test(str));
    }
    
    isAllowedOrigin(origin) {
        return this.allowedOrigins.includes(origin);
    }
}
```

## Теги

#javascript #performance #optimization #profiling #rendering #memory #timing #monitoring