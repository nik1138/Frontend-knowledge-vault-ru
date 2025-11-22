---
aliases: ["Практические рекомендации по профилированию", "Профилирование JavaScript"]
tags: [js, performance, profiling, frontend]
---

# Практические рекомендации по профилированию

Практические советы и методы профилирования JavaScript приложений для выявления узких мест производительности и оптимизации кода.

## 1. Инструменты профилирования

### Встроенные инструменты браузера

```javascript
// Использование Performance API для измерения времени выполнения
class PerformanceTimer {
    constructor() {
        this.marks = new Map();
        this.measures = new Map();
    }
    
    // Установка метки времени
    mark(name) {
        performance.mark(name);
        this.marks.set(name, performance.now());
    }
    
    // Измерение времени между метками
    measure(name, startMark, endMark) {
        performance.measure(name, startMark, endMark);
        const measure = performance.getEntriesByName(name)[0];
        this.measures.set(name, measure.duration);
        return measure.duration;
    }
    
    // Измерение выполнения функции
    async measureFunction(name, fn) {
        const startMark = `${name}_start`;
        const endMark = `${name}_end`;
        
        performance.mark(startMark);
        const result = await Promise.resolve(fn());
        performance.mark(endMark);
        
        const duration = this.measure(name, startMark, endMark);
        return { result, duration };
    }
    
    // Получение всех измерений
    getMeasures() {
        return Object.fromEntries(this.measures);
    }
    
    // Очистка данных
    clear() {
        performance.clearMarks();
        performance.clearMeasures();
        this.marks.clear();
        this.measures.clear();
    }
}

// Пример использования Performance API
const timer = new PerformanceTimer();

async function examplePerformanceMeasurement() {
    // Измерение сложной операции
    timer.mark('operation_start');
    
    // Симуляция сложной операции
    const result = await timer.measureFunction('complex_operation', async () => {
        let sum = 0;
        for (let i = 0; i < 1000000; i++) {
            sum += Math.sqrt(i) * Math.sin(i);
        }
        return sum;
    });
    
    timer.mark('operation_end');
    const totalDuration = timer.measure('total_operation', 'operation_start', 'operation_end');
    
    console.log(`Результат операции: ${result.result}`);
    console.log(`Время выполнения функции: ${result.duration.toFixed(2)}ms`);
    console.log(`Общее время: ${totalDuration.toFixed(2)}ms`);
    
    return result.result;
}

// examplePerformanceMeasurement();
```

### Кастомные инструменты профилирования

```javascript
// Расширенный профайлер с различными метриками
class AdvancedProfiler {
    constructor() {
        this.metrics = new Map();
        this.runningTimers = new Map();
        this.histograms = new Map();
    }
    
    // Начало измерения
    start(name) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        this.runningTimers.set(name, {
            startTime,
            startMemory,
            startMark: performance.getEntriesByType('mark').length
        });
    }
    
    // Завершение измерения
    end(name) {
        if (!this.runningTimers.has(name)) {
            console.warn(`Таймер "${name}" не запущен`);
            return null;
        }
        
        const timer = this.runningTimers.get(name);
        const endTime = performance.now();
        const endMemory = this.getMemoryUsage();
        
        const duration = endTime - timer.startTime;
        const memoryDelta = endMemory - timer.startMemory;
        const marksDelta = performance.getEntriesByType('mark').length - timer.startMark;
        
        const metric = {
            duration,
            memoryDelta,
            marksDelta,
            timestamp: Date.now()
        };
        
        if (!this.metrics.has(name)) {
            this.metrics.set(name, []);
        }
        
        this.metrics.get(name).push(metric);
        
        // Обновляем гистограмму
        this.updateHistogram(name, duration);
        
        this.runningTimers.delete(name);
        
        return metric;
    }
    
    // Получение текущего использования памяти (если поддерживается)
    getMemoryUsage() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return 0; // Возврат 0, если метрика недоступна
    }
    
    // Обновление гистограммы для анализа распределения
    updateHistogram(name, value) {
        if (!this.histograms.has(name)) {
            this.histograms.set(name, {
                values: [],
                min: Infinity,
                max: -Infinity,
                sum: 0
            });
        }
        
        const hist = this.histograms.get(name);
        hist.values.push(value);
        hist.min = Math.min(hist.min, value);
        hist.max = Math.max(hist.max, value);
        hist.sum += value;
    }
    
    // Получение статистики для метрики
    getStats(name) {
        const metrics = this.metrics.get(name);
        if (!metrics || metrics.length === 0) {
            return null;
        }
        
        const durations = metrics.map(m => m.duration);
        const memoryDeltas = metrics.map(m => m.memoryDelta);
        
        return {
            count: metrics.length,
            avgDuration: durations.reduce((a, b) => a + b, 0) / durations.length,
            minDuration: Math.min(...durations),
            maxDuration: Math.max(...durations),
            avgMemoryDelta: memoryDeltas.reduce((a, b) => a + b, 0) / memoryDeltas.length,
            totalDuration: durations.reduce((a, b) => a + b, 0),
            lastRun: metrics[metrics.length - 1]
        };
    }
    
    // Получение всех статистик
    getAllStats() {
        const stats = {};
        for (const [name] of this.metrics) {
            stats[name] = this.getStats(name);
        }
        return stats;
    }
    
    // Получение гистограммы
    getHistogram(name) {
        const hist = this.histograms.get(name);
        if (!hist) return null;
        
        const sorted = [...hist.values].sort((a, b) => a - b);
        const length = sorted.length;
        
        return {
            min: hist.min,
            max: hist.max,
            avg: hist.sum / length,
            median: sorted[Math.floor(length / 2)],
            p95: sorted[Math.floor(length * 0.95)],
            p99: sorted[Math.floor(length * 0.99)],
            count: length
        };
    }
    
    // Сброс всех данных
    reset() {
        this.metrics.clear();
        this.runningTimers.clear();
        this.histograms.clear();
    }
    
    // Вывод отчета
    report() {
        console.group('Профилирование - Отчет');
        
        for (const [name, stats] of this.getAllStats()) {
            const histogram = this.getHistogram(name);
            console.log(`${name}:`);
            console.log(`  Вызовов: ${stats.count}`);
            console.log(`  Среднее время: ${stats.avgDuration.toFixed(2)}ms`);
            console.log(`  Мин/Макс время: ${stats.minDuration.toFixed(2)}ms / ${stats.maxDuration.toFixed(2)}ms`);
            console.log(`  Среднее изменение памяти: ${stats.avgMemoryDelta.toLocaleString()} bytes`);
            if (histogram) {
                console.log(`  Медиана: ${histogram.median.toFixed(2)}ms`);
                console.log(`  95-й перцентиль: ${histogram.p95.toFixed(2)}ms`);
                console.log(`  99-й перцентиль: ${histogram.p99.toFixed(2)}ms`);
            }
        }
        
        console.groupEnd();
    }
}

// Пример использования продвинутого профайлера
const profiler = new AdvancedProfiler();

function exampleAdvancedProfiling() {
    // Профилируем несколько операций
    for (let i = 0; i < 100; i++) {
        profiler.start(`operation_${i % 3}`);
        
        // Симуляция работы
        let sum = 0;
        for (let j = 0; j < 1000; j++) {
            sum += Math.random();
        }
        
        profiler.end(`operation_${i % 3}`);
    }
    
    // Вывод отчета
    profiler.report();
}

// exampleAdvancedProfiling();
```

## 2. Профилирование производительности

### Измерение производительности функций

```javascript
// Утилита для измерения производительности функций
class FunctionProfiler {
    constructor() {
        this.functionStats = new Map();
    }
    
    // Обертка для профилирования функции
    profileFunction(fn, name) {
        const self = this;
        
        return function(...args) {
            const startTime = performance.now();
            const startMemory = self.getMemoryUsage();
            
            try {
                const result = fn.apply(this, args);
                
                // Для асинхронных функций нужно обрабатывать по-другому
                if (result instanceof Promise) {
                    return result.then(resolvedResult => {
                        const endTime = performance.now();
                        const endMemory = self.getMemoryUsage();
                        
                        self.recordFunctionStats(name, {
                            duration: endTime - startTime,
                            memoryDelta: endMemory - startMemory,
                            error: false
                        });
                        
                        return resolvedResult;
                    }).catch(error => {
                        const endTime = performance.now();
                        const endMemory = self.getMemoryUsage();
                        
                        self.recordFunctionStats(name, {
                            duration: endTime - startTime,
                            memoryDelta: endMemory - startMemory,
                            error: true,
                            errorMessage: error.message
                        });
                        
                        throw error;
                    });
                }
                
                const endTime = performance.now();
                const endMemory = self.getMemoryUsage();
                
                self.recordFunctionStats(name, {
                    duration: endTime - startTime,
                    memoryDelta: endMemory - startMemory,
                    error: false
                });
                
                return result;
            } catch (error) {
                const endTime = performance.now();
                const endMemory = self.getMemoryUsage();
                
                self.recordFunctionStats(name, {
                    duration: endTime - startTime,
                    memoryDelta: endMemory - startMemory,
                    error: true,
                    errorMessage: error.message
                });
                
                throw error;
            }
        };
    }
    
    getMemoryUsage() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return 0;
    }
    
    recordFunctionStats(name, stats) {
        if (!this.functionStats.has(name)) {
            this.functionStats.set(name, {
                calls: [],
                errorCount: 0,
                totalDuration: 0,
                totalMemoryDelta: 0
            });
        }
        
        const funcStats = this.functionStats.get(name);
        funcStats.calls.push(stats);
        funcStats.totalDuration += stats.duration;
        funcStats.totalMemoryDelta += stats.memoryDelta;
        
        if (stats.error) {
            funcStats.errorCount++;
        }
    }
    
    getFunctionReport(name) {
        const stats = this.functionStats.get(name);
        if (!stats) return null;
        
        const callCount = stats.calls.length;
        const avgDuration = stats.totalDuration / callCount;
        const avgMemoryDelta = stats.totalMemoryDelta / callCount;
        const errorRate = (stats.errorCount / callCount) * 100;
        
        // Вычисляем перцентили
        const durations = stats.calls.map(call => call.duration).sort((a, b) => a - b);
        const p95Index = Math.floor(durations.length * 0.95);
        const p99Index = Math.floor(durations.length * 0.99);
        
        return {
            name,
            callCount,
            avgDuration: avgDuration.toFixed(2),
            totalDuration: stats.totalDuration.toFixed(2),
            avgMemoryDelta: avgMemoryDelta.toFixed(2),
            errorCount: stats.errorCount,
            errorRate: errorRate.toFixed(2) + '%',
            p95Duration: durations[p95Index]?.toFixed(2) || '0.00',
            p99Duration: durations[p99Index]?.toFixed(2) || '0.00'
        };
    }
    
    getAllReports() {
        const reports = [];
        for (const [name] of this.functionStats) {
            reports.push(this.getFunctionReport(name));
        }
        return reports;
    }
    
    generateReport() {
        console.group('Профилирование функций');
        
        const reports = this.getAllReports();
        reports.forEach(report => {
            console.log(`${report.name}:`);
            console.log(`  Вызовов: ${report.callCount}`);
            console.log(`  Среднее время: ${report.avgDuration}ms`);
            console.log(`  Общее время: ${report.totalDuration}ms`);
            console.log(`  Среднее изменение памяти: ${report.avgMemoryDelta} bytes`);
            console.log(`  Ошибки: ${report.errorCount} (${report.errorRate})`);
            console.log(`  95-й перцентиль: ${report.p95Duration}ms`);
            console.log(`  99-й перцентиль: ${report.p99Duration}ms`);
            console.log('---');
        });
        
        console.groupEnd();
    }
}

// Пример использования
const funcProfiler = new FunctionProfiler();

// Пример функции для профилирования
function exampleFunction(n) {
    let result = 0;
    for (let i = 0; i < n; i++) {
        result += Math.sqrt(i) * Math.sin(i);
    }
    return result;
}

// Оборачиваем функцию в профайлер
const profiledFunction = funcProfiler.profileFunction(exampleFunction, 'exampleFunction');

// Вызываем функцию несколько раз
for (let i = 0; i < 100; i++) {
    profiledFunction(10000);
}

// funcProfiler.generateReport();
```

### Профилирование анимаций

```javascript
// Профилирование анимаций
class AnimationProfiler {
    constructor() {
        this.frameStats = [];
        this.isProfiling = false;
        this.rafId = null;
        this.startTime = 0;
        this.lastTimestamp = 0;
        this.frameCount = 0;
    }
    
    start() {
        if (this.isProfiling) return;
        
        this.isProfiling = true;
        this.frameStats = [];
        this.frameCount = 0;
        this.startTime = performance.now();
        this.lastTimestamp = this.startTime;
        
        this.rafId = requestAnimationFrame(this.frameHandler.bind(this));
    }
    
    frameHandler(timestamp) {
        if (!this.isProfiling) return;
        
        const frameTime = timestamp - this.lastTimestamp;
        const totalRunTime = timestamp - this.startTime;
        
        this.frameStats.push({
            timestamp,
            frameTime,
            totalRunTime,
            fps: 1000 / frameTime // текущий FPS
        });
        
        this.lastTimestamp = timestamp;
        this.frameCount++;
        
        this.rafId = requestAnimationFrame(this.frameHandler.bind(this));
    }
    
    stop() {
        if (this.rafId) {
            cancelAnimationFrame(this.rafId);
        }
        this.isProfiling = false;
    }
    
    getStats() {
        if (this.frameStats.length === 0) {
            return null;
        }
        
        const frameTimes = this.frameStats.map(f => f.frameTime);
        const fpsValues = this.frameStats.map(f => f.fps).filter(fps => !isNaN(fps));
        
        return {
            totalFrames: this.frameStats.length,
            totalTime: this.frameStats[this.frameStats.length - 1].totalRunTime,
            avgFrameTime: frameTimes.reduce((a, b) => a + b, 0) / frameTimes.length,
            minFrameTime: Math.min(...frameTimes),
            maxFrameTime: Math.max(...frameTimes),
            avgFps: 1000 / (frameTimes.reduce((a, b) => a + b, 0) / frameTimes.length),
            minFps: fpsValues.length > 0 ? Math.min(...fpsValues) : 0,
            maxFps: fpsValues.length > 0 ? Math.max(...fpsValues) : 0,
            droppedFrames: frameTimes.filter(ft => ft > 16.67).length, // больше 16.67ms = меньше 60fps
            smoothness: 1 - (this.frameStats.filter(f => f.frameTime > 16.67).length / this.frameStats.length)
        };
    }
    
    generateReport() {
        const stats = this.getStats();
        if (!stats) {
            console.log('Нет данных профилирования анимации');
            return;
        }
        
        console.group('Профилирование анимации');
        console.log(`Всего кадров: ${stats.totalFrames}`);
        console.log(`Общее время: ${(stats.totalTime / 1000).toFixed(2)}s`);
        console.log(`Среднее время кадра: ${stats.avgFrameTime.toFixed(2)}ms`);
        console.log(`Средний FPS: ${stats.avgFps.toFixed(2)}`);
        console.log(`Мин/Макс FPS: ${stats.minFps.toFixed(2)} / ${stats.maxFps.toFixed(2)}`);
        console.log(`Пропущено кадров: ${stats.droppedFrames}`);
        console.log(`Плавность: ${(stats.smoothness * 100).toFixed(2)}%`);
        console.groupEnd();
    }
}

// Пример использования профайлера анимаций
function exampleAnimation() {
    const profiler = new AnimationProfiler();
    const element = document.querySelector('#animation-target') || document.body;
    
    let angle = 0;
    
    function animate() {
        if (!profiler.isProfiling) return;
        
        // Анимация вращения
        angle += 1;
        element.style.transform = `rotate(${angle}deg)`;
        
        requestAnimationFrame(animate);
    }
    
    profiler.start();
    animate();
    
    // Останавливаем через 5 секунд
    setTimeout(() => {
        profiler.stop();
        profiler.generateReport();
    }, 5000);
}

// exampleAnimation();
```

## 3. Выявление узких мест

### Поиск узких мест в коде

```javascript
// Утилита для поиска узких мест
class BottleneckDetector {
    constructor() {
        this.operationTimings = new Map();
        this.callStacks = [];
    }
    
    // Измерение времени выполнения блока кода
    async measureBlock(description, fn) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        try {
            const result = await Promise.resolve(fn());
            
            const endTime = performance.now();
            const endMemory = this.getMemoryUsage();
            const duration = endTime - startTime;
            const memoryDelta = endMemory - startMemory;
            
            this.recordTiming(description, duration, memoryDelta);
            
            return result;
        } catch (error) {
            const endTime = performance.now();
            const endMemory = this.getMemoryUsage();
            const duration = endTime - startTime;
            const memoryDelta = endMemory - startMemory;
            
            this.recordTiming(`${description}_error`, duration, memoryDelta);
            
            throw error;
        }
    }
    
    recordTiming(description, duration, memoryDelta) {
        if (!this.operationTimings.has(description)) {
            this.operationTimings.set(description, {
                durations: [],
                memoryDeltas: [],
                count: 0
            });
        }
        
        const timing = this.operationTimings.get(description);
        timing.durations.push(duration);
        timing.memoryDeltas.push(memoryDelta);
        timing.count++;
    }
    
    getMemoryUsage() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return 0;
    }
    
    // Поиск потенциальных узких мест
    findBottlenecks() {
        const bottlenecks = [];
        
        for (const [description, timing] of this.operationTimings) {
            const avgDuration = timing.durations.reduce((a, b) => a + b, 0) / timing.durations.length;
            const maxDuration = Math.max(...timing.durations);
            const avgMemoryDelta = timing.memoryDeltas.reduce((a, b) => a + b, 0) / timing.memoryDeltas.length;
            
            // Критерии для определения узких мест
            const isSlow = avgDuration > 10; // медленнее 10мс
            const hasHighVariation = maxDuration > avgDuration * 3; // высокая вариация
            const usesTooMuchMemory = avgMemoryDelta > 1024 * 1024; // больше 1MB
            
            if (isSlow || hasHighVariation || usesTooMuchMemory) {
                bottlenecks.push({
                    description,
                    avgDuration: avgDuration.toFixed(2),
                    maxDuration: maxDuration.toFixed(2),
                    avgMemoryDelta: avgMemoryDelta,
                    count: timing.count,
                    issues: [
                        isSlow ? 'slow' : null,
                        hasHighVariation ? 'high_variation' : null,
                        usesTooMuchMemory ? 'high_memory_usage' : null
                    ].filter(Boolean)
                });
            }
        }
        
        return bottlenecks.sort((a, b) => parseFloat(b.avgDuration) - parseFloat(a.avgDuration));
    }
    
    generateBottleneckReport() {
        const bottlenecks = this.findBottlenecks();
        
        console.group('Обнаруженные узкие места');
        
        if (bottlenecks.length === 0) {
            console.log('Узких мест не обнаружено');
        } else {
            bottlenecks.forEach(bottleneck => {
                console.log(`${bottleneck.description}:`);
                console.log(`  Среднее время: ${bottleneck.avgDuration}ms`);
                console.log(`  Максимальное время: ${bottleneck.maxDuration}ms`);
                console.log(`  Среднее изменение памяти: ${bottleneck.avgMemoryDelta} bytes`);
                console.log(`  Вызовов: ${bottleneck.count}`);
                console.log(`  Проблемы: ${bottleneck.issues.join(', ')}`);
                console.log('---');
            });
        }
        
        console.groupEnd();
    }
    
    // Сброс данных
    reset() {
        this.operationTimings.clear();
        this.callStacks = [];
    }
}

// Пример использования детектора узких мест
const bottleneckDetector = new BottleneckDetector();

async function exampleBottleneckDetection() {
    // Измеряем разные операции
    await bottleneckDetector.measureBlock('fast_operation', async () => {
        await new Promise(resolve => setTimeout(resolve, 1)); // 1ms
        return 'fast';
    });
    
    await bottleneckDetector.measureBlock('slow_operation', async () => {
        // Симуляция медленной операции
        let sum = 0;
        for (let i = 0; i < 1000000; i++) {
            sum += Math.sqrt(i);
        }
        return sum;
    });
    
    await bottleneckDetector.measureBlock('memory_intensive', async () => {
        // Создаем много объектов
        const arr = [];
        for (let i = 0; i < 100000; i++) {
            arr.push({ id: i, data: new Array(10).fill(i) });
        }
        return arr.length;
    });
    
    bottleneckDetector.generateBottleneckReport();
}

// exampleBottleneckDetection();
```

### Профилирование циклов и итераций

```javascript
// Профилирование циклов
class LoopProfiler {
    constructor() {
        this.loopStats = new Map();
    }
    
    // Профилирование for цикла
    profileForLoop(name, iterations, workFunction) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        for (let i = 0; i < iterations; i++) {
            workFunction(i);
        }
        
        const endTime = performance.now();
        const endMemory = this.getMemoryUsage();
        
        this.recordLoopStats(name, iterations, endTime - startTime, endMemory - startMemory);
    }
    
    // Профилирование for...of цикла
    profileForOfLoop(name, array, workFunction) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        for (const item of array) {
            workFunction(item);
        }
        
        const endTime = performance.now();
        const endMemory = this.getMemoryUsage();
        
        this.recordLoopStats(name, array.length, endTime - startTime, endMemory - startMemory);
    }
    
    // Профилирование forEach
    profileForEach(name, array, workFunction) {
        const startTime = performance.now();
        const startMemory = this.getMemoryUsage();
        
        array.forEach((item, index) => {
            workFunction(item, index);
        });
        
        const endTime = performance.now();
        const endMemory = this.getMemoryUsage();
        
        this.recordLoopStats(name, array.length, endTime - startTime, endMemory - startMemory);
    }
    
    recordLoopStats(name, iterations, duration, memoryDelta) {
        if (!this.loopStats.has(name)) {
            this.loopStats.set(name, {
                iterations: [],
                durations: [],
                memoryDeltas: [],
                totalIterations: 0,
                totalDuration: 0,
                totalMemoryDelta: 0
            });
        }
        
        const stats = this.loopStats.get(name);
        stats.iterations.push(iterations);
        stats.durations.push(duration);
        stats.memoryDeltas.push(memoryDelta);
        stats.totalIterations += iterations;
        stats.totalDuration += duration;
        stats.totalMemoryDelta += memoryDelta;
    }
    
    getMemoryUsage() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return 0;
    }
    
    compareLoops(array, workFunction) {
        console.group('Сравнение циклов');
        
        // Подготавливаем копии массива для честного сравнения
        const testArray = [...array];
        
        this.profileForLoop('for_loop', testArray.length, (i) => workFunction(testArray[i], i));
        this.profileForOfLoop('for_of_loop', testArray, workFunction);
        this.profileForEach('forEach', testArray, workFunction);
        
        for (const [name, stats] of this.loopStats) {
            const avgDuration = stats.totalDuration / stats.durations.length;
            const avgMemoryPerIteration = stats.totalMemoryDelta / stats.totalIterations;
            const avgTimePerIteration = stats.totalDuration / stats.totalIterations;
            
            console.log(`${name}:`);
            console.log(`  Общее время: ${stats.totalDuration.toFixed(2)}ms`);
            console.log(`  Среднее время: ${avgDuration.toFixed(2)}ms`);
            console.log(`  Общее количество итераций: ${stats.totalIterations}`);
            console.log(`  Среднее время на итерацию: ${avgTimePerIteration.toFixed(4)}ms`);
            console.log(`  Среднее изменение памяти на итерацию: ${avgMemoryPerIteration.toFixed(2)} bytes`);
            console.log('---');
        }
        
        console.groupEnd();
    }
}

// Пример использования
const loopProfiler = new LoopProfiler();

function exampleLoopProfiling() {
    const largeArray = Array.from({ length: 100000 }, (_, i) => i);
    
    // Функция работы для циклов
    function workFunction(item, index) {
        return item * 2 + index;
    }
    
    loopProfiler.compareLoops(largeArray, workFunction);
}

// exampleLoopProfiling();
```

## 4. Практические рекомендации

### Паттерны эффективного профилирования

```javascript
// Модуль для комплексного профилирования
class ComprehensiveProfiler {
    constructor(options = {}) {
        this.options = {
            autoReport: true,
            reportThreshold: 100, // ms
            memoryThreshold: 1024 * 1024, // bytes
            ...options
        };
        
        this.metrics = {
            functions: new Map(),
            dom: new Map(),
            network: new Map(),
            memory: [],
            performance: []
        };
        
        this.setupPerformanceObserver();
    }
    
    setupPerformanceObserver() {
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    this.metrics.performance.push({
                        name: entry.name,
                        entryType: entry.entryType,
                        duration: entry.duration || 0,
                        startTime: entry.startTime,
                        timestamp: Date.now()
                    });
                }
            });
            
            observer.observe({ 
                entryTypes: ['measure', 'mark', 'navigation', 'resource', 'paint', 'largest-contentful-paint'] 
            });
        }
    }
    
    // Профилирование функций с автоматическим анализом
    profileFunctionAuto(name, fn) {
        const self = this;
        
        return async function(...args) {
            const startMark = `start_${name}_${Date.now()}`;
            const endMark = `end_${name}_${Date.now()}`;
            
            performance.mark(startMark);
            const startMemory = self.getMemoryUsage();
            
            try {
                const result = await Promise.resolve(fn.apply(this, args));
                
                performance.mark(endMark);
                const measure = performance.measure(name, startMark, endMark);
                
                const memoryDelta = self.getMemoryUsage() - startMemory;
                
                self.recordFunctionMetrics(name, {
                    duration: measure.duration,
                    memoryDelta,
                    success: true,
                    argsLength: args.length
                });
                
                // Автоматическая проверка на превышение порогов
                if (self.options.autoReport) {
                    self.checkThresholds(name, measure.duration, memoryDelta);
                }
                
                return result;
            } catch (error) {
                performance.mark(endMark);
                const measure = performance.measure(name, startMark, endMark);
                
                const memoryDelta = self.getMemoryUsage() - startMemory;
                
                self.recordFunctionMetrics(name, {
                    duration: measure.duration,
                    memoryDelta,
                    success: false,
                    error: error.message,
                    argsLength: args.length
                });
                
                throw error;
            }
        };
    }
    
    recordFunctionMetrics(name, metrics) {
        if (!this.metrics.functions.has(name)) {
            this.metrics.functions.set(name, {
                calls: [],
                totalDuration: 0,
                totalMemoryDelta: 0,
                successCount: 0,
                errorCount: 0
            });
        }
        
        const funcMetrics = this.metrics.functions.get(name);
        funcMetrics.calls.push(metrics);
        funcMetrics.totalDuration += metrics.duration;
        funcMetrics.totalMemoryDelta += metrics.memoryDelta;
        
        if (metrics.success) {
            funcMetrics.successCount++;
        } else {
            funcMetrics.errorCount++;
        }
    }
    
    getMemoryUsage() {
        if (performance.memory) {
            return performance.memory.usedJSHeapSize;
        }
        return 0;
    }
    
    checkThresholds(name, duration, memoryDelta) {
        const issues = [];
        
        if (duration > this.options.reportThreshold) {
            issues.push(`Превышено время выполнения: ${duration.toFixed(2)}ms > ${this.options.reportThreshold}ms`);
        }
        
        if (memoryDelta > this.options.memoryThreshold) {
            issues.push(`Превышено использование памяти: ${memoryDelta.toLocaleString()} bytes > ${this.options.memoryThreshold.toLocaleString()} bytes`);
        }
        
        if (issues.length > 0) {
            console.warn(`Потенциальная проблема в "${name}":`, issues.join('; '));
        }
    }
    
    generateComprehensiveReport() {
        console.group('Комплексный отчет о производительности');
        
        // Отчет по функциям
        console.group('Функции');
        for (const [name, metrics] of this.metrics.functions) {
            const avgDuration = metrics.totalDuration / metrics.calls.length;
            const avgMemory = metrics.totalMemoryDelta / metrics.calls.length;
            const errorRate = (metrics.errorCount / metrics.calls.length) * 100;
            
            console.log(`${name}:`);
            console.log(`  Вызовов: ${metrics.calls.length}`);
            console.log(`  Среднее время: ${avgDuration.toFixed(2)}ms`);
            console.log(`  Среднее изменение памяти: ${avgMemory.toFixed(2)} bytes`);
            console.log(`  Успешных: ${metrics.successCount}, Ошибок: ${metrics.errorCount} (${errorRate.toFixed(2)}%)`);
        }
        console.groupEnd();
        
        // Отчет по производительности
        console.group('Производительность');
        if (this.metrics.performance.length > 0) {
            const paintEntries = this.metrics.performance.filter(e => 
                e.entryType === 'paint' || e.entryType === 'largest-contentful-paint'
            );
            
            if (paintEntries.length > 0) {
                console.log(`Количество paint событий: ${paintEntries.length}`);
                const paintTimes = paintEntries.map(e => e.duration);
                const avgPaintTime = paintTimes.reduce((a, b) => a + b, 0) / paintTimes.length;
                console.log(`Среднее время отрисовки: ${avgPaintTime.toFixed(2)}ms`);
            }
        }
        console.groupEnd();
        
        console.groupEnd();
    }
}

// Пример комплексного профилирования
const comprehensiveProfiler = new ComprehensiveProfiler({
    reportThreshold: 50, // 50ms
    memoryThreshold: 512 * 1024 // 512KB
});

// Пример использования
const profiledApiCall = comprehensiveProfiler.profileFunctionAuto('apiCall', async (url) => {
    const response = await fetch(url);
    return await response.json();
});

const profiledDataProcessing = comprehensiveProfiler.profileFunctionAuto('dataProcessing', async (data) => {
    // Тяжелая обработка данных
    return data.map(item => ({
        ...item,
        processed: true,
        timestamp: Date.now()
    }));
});

// profiledApiCall('https://api.example.com/data')
//     .then(data => profiledDataProcessing(data))
//     .then(result => console.log('Обработка завершена'));

// comprehensiveProfiler.generateComprehensiveReport();
```

### Рекомендации по интерпретации результатов

```javascript
console.group('Рекомендации по профилированию');

console.log(`
1. Измеряйте в реальных условиях:
   - Используйте продакшн-подобные данные
   - Тестируйте на целевых устройствах
   - Учитывайте сетевые задержки

2. Сравнивайте относительные изменения:
   - Не полагайтесь только на абсолютные значения
   - Сравнивайте до и после оптимизаций
   - Учитывайте статистическую значимость

3. Ищите узкие места:
   - Операции дольше 16.67ms вызывают прерывания анимации
   - Часто вызываемые функции с малой длительностью могут суммарно быть значительными
   - Операции с высокой вариацией времени выполнения требуют внимания

4. Оптимизируйте постепенно:
   - Начинайте с наибольших узких мест
   - Измеряйте эффект каждой оптимизации
   - Не оптимизируйте преждевременно

5. Используйте правильные инструменты:
   - Chrome DevTools для детального анализа
   - Performance API для измерений в коде
   - Сторонние библиотеки для специфичных задач
`);

console.log(`
Полезные метрики для отслеживания:
- Время выполнения функций
- Использование памяти
- Количество DOM-манипуляций
- Сетевые запросы и их время
- FPS при анимациях
- Время загрузки и отображения
`);

console.groupEnd();
```

Эти рекомендации помогут вам эффективно профилировать JavaScript приложения и выявлять узкие места производительности для последующей оптимизации.

См. также:
- [[js/performance/additional-techniques]] - Дополнительные техники оптимизации
- [[js/performance/comparison-techniques]] - Сравнение производительности разных подходов
- [[js/tricks/arrays-tricks]] - Хитрости работы с массивами
- [[js/tricks/objects-tricks]] - Работа с объектами