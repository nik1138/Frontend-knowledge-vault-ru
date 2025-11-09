# Профилирование и измерение производительности

Профилирование и измерение производительности являются критически важными аспектами оптимизации функциональных приложений. В этой главе мы рассмотрим различные инструменты и техники для анализа производительности кода.

## Содержание

1. [Основы профилирования](#основы-профилирования)
2. [Измерение времени выполнения](#измерение-времени-выполнения)
3. [Профилирование памяти](#профилирование-памяти)
4. [Профилирование CPU](#профилирование-cpu)
5. [Анализ производительности функций](#анализ-производительности-функций)
6. [Профилирование асинхронных операций](#профилирование-асинхронных-операций)
7. [Практические примеры](#практические-примеры)

## Основы профилирования

Профилирование - это процесс измерения характеристик выполнения программы, таких как время выполнения, использование памяти и загрузка CPU. В функциональном программировании профилирование помогает выявить узкие места и оптимизировать код.

### Цели профилирования:

1. **Выявление узких мест** - Нахождение функций, которые занимают больше всего времени
2. **Оптимизация ресурсов** - Снижение использования CPU и памяти
3. **Сравнение реализаций** - Сравнение производительности различных подходов
4. **Мониторинг производительности** - Отслеживание деградации производительности

```typescript
// Простой пример профилирования
class SimpleProfiler {
  private measurements: Map<string, number[]> = new Map();
  
  start(label: string): void {
    if (!performance) return;
    
    const measurements = this.measurements.get(label) || [];
    measurements.push(performance.now());
    this.measurements.set(label, measurements);
  }
  
  end(label: string): number {
    if (!performance) return 0;
    
    const measurements = this.measurements.get(label);
    if (!measurements || measurements.length === 0) {
      throw new Error(`Нет начального измерения для ${label}`);
    }
    
    const startTime = measurements.pop()!;
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    console.log(`${label}: ${duration.toFixed(2)}ms`);
    return duration;
  }
  
  async timeAsync<T>(label: string, fn: () => Promise<T>): Promise<T> {
    this.start(label);
    try {
      const result = await fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  time<T>(label: string, fn: () => T): T {
    this.start(label);
    try {
      const result = fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  getStats(label: string): { min: number; max: number; avg: number; count: number } | null {
    const measurements = this.measurements.get(label);
    if (!measurements || measurements.length === 0) {
      return null;
    }
    
    const durations = measurements.filter((_, i) => i % 2 === 1); // Только завершенные измерения
    if (durations.length === 0) return null;
    
    const min = Math.min(...durations);
    const max = Math.max(...durations);
    const avg = durations.reduce((sum, d) => sum + d, 0) / durations.length;
    
    return { min, max, avg, count: durations.length };
  }
}

// Пример использования простого профайлера
const profiler = new SimpleProfiler();

function expensiveOperation(n: number): number {
  let result = 0;
  for (let i = 0; i < n * 1000000; i++) {
    result += Math.sin(i);
  }
  return result;
}

// Измерение синхронной операции
const result = profiler.time("expensiveOperation", () => expensiveOperation(10));
console.log(`Результат: ${result}`);

// Измерение асинхронной операции
async function asyncOperation(): Promise<string> {
  await new Promise(resolve => setTimeout(resolve, 100));
  return "Готово";
}

profiler.timeAsync("asyncOperation", asyncOperation).then(result => {
  console.log(`Асинхронный результат: ${result}`);
});

// Многократные измерения
for (let i = 0; i < 5; i++) {
  profiler.time("repeatedOperation", () => expensiveOperation(5));
}

// Получение статистики
const stats = profiler.getStats("repeatedOperation");
if (stats) {
  console.log(`Статистика: min=${stats.min.toFixed(2)}ms, max=${stats.max.toFixed(2)}ms, avg=${stats.avg.toFixed(2)}ms`);
}
```

### Расширенный профайлер

```typescript
// Расширенный профайлер с поддержкой категорий и тегов
interface ProfileMeasurement {
  startTime: number;
  endTime?: number;
  duration?: number;
  category?: string;
  tags?: string[];
  metadata?: Record<string, any>;
}

class AdvancedProfiler {
  private measurements: Map<string, ProfileMeasurement[]> = new Map();
  private activeMeasurements: Map<string, ProfileMeasurement> = new Map();
  
  start(
    label: string, 
    options: { category?: string; tags?: string[]; metadata?: Record<string, any> } = {}
  ): void {
    if (!performance) return;
    
    const measurement: ProfileMeasurement = {
      startTime: performance.now(),
      category: options.category,
      tags: options.tags,
      metadata: options.metadata
    };
    
    this.activeMeasurements.set(label, measurement);
  }
  
  end(label: string): number {
    if (!performance) return 0;
    
    const measurement = this.activeMeasurements.get(label);
    if (!measurement) {
      throw new Error(`Нет активного измерения для ${label}`);
    }
    
    measurement.endTime = performance.now();
    measurement.duration = measurement.endTime - measurement.startTime;
    
    this.activeMeasurements.delete(label);
    
    // Сохраняем измерение
    const measurements = this.measurements.get(label) || [];
    measurements.push(measurement);
    this.measurements.set(label, measurements);
    
    console.log(`${label}: ${measurement.duration.toFixed(2)}ms`);
    return measurement.duration;
  }
  
  async timeAsync<T>(
    label: string, 
    fn: () => Promise<T>,
    options: { category?: string; tags?: string[]; metadata?: Record<string, any> } = {}
  ): Promise<T> {
    this.start(label, options);
    try {
      const result = await fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  time<T>(
    label: string, 
    fn: () => T,
    options: { category?: string; tags?: string[]; metadata?: Record<string, any> } = {}
  ): T {
    this.start(label, options);
    try {
      const result = fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  getStats(label: string): { 
    min: number; 
    max: number; 
    avg: number; 
    count: number;
    total: number;
    category?: string;
  } | null {
    const measurements = this.measurements.get(label);
    if (!measurements || measurements.length === 0) {
      return null;
    }
    
    const durations = measurements
      .map(m => m.duration)
      .filter((d): d is number => d !== undefined);
    
    if (durations.length === 0) return null;
    
    const min = Math.min(...durations);
    const max = Math.max(...durations);
    const avg = durations.reduce((sum, d) => sum + d, 0) / durations.length;
    const total = durations.reduce((sum, d) => sum + d, 0);
    
    return { 
      min, 
      max, 
      avg, 
      count: durations.length,
      total,
      category: measurements[0].category
    };
  }
  
  getMeasurementsByCategory(category: string): Map<string, ReturnType<this["getStats"]>> {
    const result = new Map<string, any>();
    
    for (const [label, measurements] of this.measurements.entries()) {
      const firstMeasurement = measurements[0];
      if (firstMeasurement.category === category) {
        const stats = this.getStats(label);
        if (stats) {
          result.set(label, stats);
        }
      }
    }
    
    return result;
  }
  
  getMeasurementsByTag(tag: string): Map<string, ReturnType<this["getStats"]>> {
    const result = new Map<string, any>();
    
    for (const [label, measurements] of this.measurements.entries()) {
      const firstMeasurement = measurements[0];
      if (firstMeasurement.tags?.includes(tag)) {
        const stats = this.getStats(label);
        if (stats) {
          result.set(label, stats);
        }
      }
    }
    
    return result;
  }
  
  report(): void {
    console.log("\n=== Отчет профилирования ===");
    
    // Группируем по категориям
    const categories = new Set<string>();
    for (const measurements of this.measurements.values()) {
      const category = measurements[0].category;
      if (category) {
        categories.add(category);
      }
    }
    
    for (const category of categories) {
      console.log(`\nКатегория: ${category}`);
      const categoryMeasurements = this.getMeasurementsByCategory(category);
      
      for (const [label, stats] of categoryMeasurements) {
        console.log(`  ${label}: avg=${stats.avg.toFixed(2)}ms, min=${stats.min.toFixed(2)}ms, max=${stats.max.toFixed(2)}ms, count=${stats.count}`);
      }
    }
    
    // Измерения без категории
    console.log("\nБез категории:");
    for (const [label, measurements] of this.measurements.entries()) {
      const firstMeasurement = measurements[0];
      if (!firstMeasurement.category) {
        const stats = this.getStats(label);
        if (stats) {
          console.log(`  ${label}: avg=${stats.avg.toFixed(2)}ms, min=${stats.min.toFixed(2)}ms, max=${stats.max.toFixed(2)}ms, count=${stats.count}`);
        }
      }
    }
  }
  
  clear(): void {
    this.measurements.clear();
    this.activeMeasurements.clear();
  }
}

// Пример использования расширенного профайлера
const advancedProfiler = new AdvancedProfiler();

// Синхронные операции
advancedProfiler.time(
  "arrayProcessing", 
  () => {
    const arr = Array.from({ length: 100000 }, (_, i) => i);
    return arr.map(x => x * 2).filter(x => x % 3 === 0).reduce((sum, x) => sum + x, 0);
  },
  { category: "dataProcessing", tags: ["sync", "array"] }
);

// Асинхронные операции
async function fetchData(): Promise<any[]> {
  await new Promise(resolve => setTimeout(resolve, Math.random() * 100));
  return Array.from({ length: 100 }, (_, i) => ({ id: i, value: Math.random() }));
}

advancedProfiler.timeAsync(
  "dataFetching",
  fetchData,
  { category: "network", tags: ["async", "http"] }
);

// Генерируем несколько измерений для статистики
async function runBenchmark() {
  for (let i = 0; i < 10; i++) {
    advancedProfiler.time(
      "mathOperation",
      () => {
        let result = 0;
        for (let j = 0; j < 10000; j++) {
          result += Math.sqrt(j);
        }
        return result;
      },
      { category: "computations", tags: ["math"] }
    );
    
    await advancedProfiler.timeAsync(
      "asyncOperation",
      async () => {
        await new Promise(resolve => setTimeout(resolve, Math.random() * 50));
        return "done";
      },
      { category: "async", tags: ["delay"] }
    );
  }
  
  // Выводим отчет
  advancedProfiler.report();
}

// runBenchmark();
```

## Измерение времени выполнения

Точное измерение времени выполнения критически важно для выявления узких мест в производительности.

### Высокоточное измерение времени

```typescript
// Высокоточный таймер для измерения производительности
class HighResolutionTimer {
  static now(): number {
    if (typeof performance !== 'undefined' && performance.now) {
      return performance.now();
    }
    return Date.now();
  }
  
  static measure<T>(fn: () => T): { result: T; duration: number } {
    const start = this.now();
    const result = fn();
    const end = this.now();
    return { result, duration: end - start };
  }
  
  static async measureAsync<T>(fn: () => Promise<T>): Promise<{ result: T; duration: number }> {
    const start = this.now();
    const result = await fn();
    const end = this.now();
    return { result, duration: end - start };
  }
  
  static benchmark<T>(
    fn: () => T, 
    iterations: number = 1000
  ): { 
    min: number; 
    max: number; 
    avg: number; 
    total: number;
    iterations: number;
  } {
    const durations: number[] = [];
    
    // Первый прогон для прогрева
    try {
      fn();
    } catch (error) {
      // Игнорируем ошибки при прогреве
    }
    
    // Основные измерения
    for (let i = 0; i < iterations; i++) {
      const { duration } = this.measure(fn);
      durations.push(duration);
    }
    
    const min = Math.min(...durations);
    const max = Math.max(...durations);
    const avg = durations.reduce((sum, d) => sum + d, 0) / durations.length;
    const total = durations.reduce((sum, d) => sum + d, 0);
    
    return { min, max, avg, total, iterations };
  }
  
  static async benchmarkAsync<T>(
    fn: () => Promise<T>, 
    iterations: number = 100
  ): Promise<{ 
    min: number; 
    max: number; 
    avg: number; 
    total: number;
    iterations: number;
  }> {
    const durations: number[] = [];
    
    // Первый прогон для прогрева
    try {
      await fn();
    } catch (error) {
      // Игнорируем ошибки при прогреве
    }
    
    // Основные измерения
    for (let i = 0; i < iterations; i++) {
      const { duration } = await this.measureAsync(fn);
      durations.push(duration);
    }
    
    const min = Math.min(...durations);
    const max = Math.max(...durations);
    const avg = durations.reduce((sum, d) => sum + d, 0) / durations.length;
    const total = durations.reduce((sum, d) => sum + d, 0);
    
    return { min, max, avg, total, iterations };
  }
}

// Примеры использования высокоточного таймера
function testFunction(): number {
  let result = 0;
  for (let i = 0; i < 10000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

// Однократное измерение
const { result, duration } = HighResolutionTimer.measure(testFunction);
console.log(`Результат: ${result}, Время: ${duration.toFixed(4)}ms`);

// Бенчмарк
const benchmark = HighResolutionTimer.benchmark(testFunction, 1000);
console.log(`Бенчмарк: min=${benchmark.min.toFixed(4)}ms, max=${benchmark.max.toFixed(4)}ms, avg=${benchmark.avg.toFixed(4)}ms`);

// Асинхронная функция для тестирования
async function asyncTestFunction(): Promise<string> {
  await new Promise(resolve => setTimeout(resolve, Math.random() * 10));
  return "Готово";
}

// Асинхронный бенчмарк
async function runAsyncBenchmark() {
  const asyncBenchmark = await HighResolutionTimer.benchmarkAsync(asyncTestFunction, 100);
  console.log(`Асинхронный бенчмарк: min=${asyncBenchmark.min.toFixed(4)}ms, max=${asyncBenchmark.max.toFixed(4)}ms, avg=${asyncBenchmark.avg.toFixed(4)}ms`);
}

// runAsyncBenchmark();
```

### Сравнение производительности реализаций

```typescript
// Инструмент для сравнения производительности различных реализаций
class PerformanceComparator {
  static compare<T>(
    implementations: Record<string, () => T>,
    iterations: number = 1000
  ): Record<string, { 
    min: number; 
    max: number; 
    avg: number; 
    total: number;
    relativePerformance: number;
  }> {
    const results: Record<string, any> = {};
    
    // Выполняем бенчмарк для каждой реализации
    for (const [name, impl] of Object.entries(implementations)) {
      try {
        const benchmark = HighResolutionTimer.benchmark(impl, iterations);
        results[name] = benchmark;
      } catch (error) {
        console.error(`Ошибка при бенчмарке ${name}:`, error);
        results[name] = { error: (error as Error).message };
      }
    }
    
    // Вычисляем относительную производительность
    const avgTimes = Object.values(results)
      .filter((r): r is { avg: number } => typeof r === 'object' && 'avg' in r)
      .map(r => r.avg);
    
    if (avgTimes.length > 0) {
      const minAvgTime = Math.min(...avgTimes);
      
      for (const [name, result] of Object.entries(results)) {
        if (typeof result === 'object' && 'avg' in result) {
          result.relativePerformance = result.avg / minAvgTime;
        }
      }
    }
    
    return results;
  }
  
  static async compareAsync<T>(
    implementations: Record<string, () => Promise<T>>,
    iterations: number = 100
  ): Promise<Record<string, { 
    min: number; 
    max: number; 
    avg: number; 
    total: number;
    relativePerformance: number;
  }>> {
    const results: Record<string, any> = {};
    
    // Выполняем бенчмарк для каждой реализации
    for (const [name, impl] of Object.entries(implementations)) {
      try {
        const benchmark = await HighResolutionTimer.benchmarkAsync(impl, iterations);
        results[name] = benchmark;
      } catch (error) {
        console.error(`Ошибка при асинхронном бенчмарке ${name}:`, error);
        results[name] = { error: (error as Error).message };
      }
    }
    
    // Вычисляем относительную производительность
    const avgTimes = Object.values(results)
      .filter((r): r is { avg: number } => typeof r === 'object' && 'avg' in r)
      .map(r => r.avg);
    
    if (avgTimes.length > 0) {
      const minAvgTime = Math.min(...avgTimes);
      
      for (const [name, result] of Object.entries(results)) {
        if (typeof result === 'object' && 'avg' in result) {
          result.relativePerformance = result.avg / minAvgTime;
        }
      }
    }
    
    return results;
  }
  
  static reportComparison(results: Record<string, any>): void {
    console.log("\n=== Сравнение производительности ===");
    
    // Сортируем по среднему времени выполнения
    const sortedEntries = Object.entries(results)
      .filter(([, result]) => typeof result === 'object' && 'avg' in result)
      .sort(([, a], [, b]) => a.avg - b.avg);
    
    if (sortedEntries.length === 0) {
      console.log("Нет данных для сравнения");
      return;
    }
    
    const fastest = sortedEntries[0];
    console.log(`Быстрейшая реализация: ${fastest[0]} (${fastest[1].avg.toFixed(4)}ms)`);
    
    console.log("\nРейтинг производительности:");
    sortedEntries.forEach(([name, result], index) => {
      const relative = result.relativePerformance.toFixed(2);
      const avg = result.avg.toFixed(4);
      console.log(`${index + 1}. ${name}: ${avg}ms (x${relative})`);
    });
    
    // Показываем реализации с ошибками
    const errorEntries = Object.entries(results)
      .filter(([, result]) => typeof result === 'object' && 'error' in result);
    
    if (errorEntries.length > 0) {
      console.log("\nРеализации с ошибками:");
      errorEntries.forEach(([name, result]) => {
        console.log(`  ${name}: ${result.error}`);
      });
    }
  }
}

// Пример сравнения различных реализаций суммирования массива
const array = Array.from({ length: 10000 }, (_, i) => i);

const sumImplementations = {
  "for loop": () => {
    let sum = 0;
    for (let i = 0; i < array.length; i++) {
      sum += array[i];
    }
    return sum;
  },
  
  "reduce": () => {
    return array.reduce((sum, val) => sum + val, 0);
  },
  
  "forEach": () => {
    let sum = 0;
    array.forEach(val => sum += val);
    return sum;
  },
  
  "while loop": () => {
    let sum = 0;
    let i = 0;
    while (i < array.length) {
      sum += array[i];
      i++;
    }
    return sum;
  }
};

// Выполняем сравнение
const comparisonResults = PerformanceComparator.compare(sumImplementations, 1000);
PerformanceComparator.reportComparison(comparisonResults);

// Асинхронный пример сравнения
async function asyncComparisonExample() {
  const asyncImplementations = {
    "fetch with await": async () => {
      await new Promise(resolve => setTimeout(resolve, 1));
      return "result1";
    },
    
    "promise then": () => {
      return new Promise(resolve => setTimeout(resolve, 1)).then(() => "result2");
    },
    
    "direct resolve": async () => {
      return "result3";
    }
  };
  
  const asyncResults = await PerformanceComparator.compareAsync(asyncImplementations, 100);
  PerformanceComparator.reportComparison(asyncResults);
}

// asyncComparisonExample();
```

## Профилирование памяти

Профилирование памяти помогает выявить утечки памяти и оптимизировать использование ресурсов.

### Инструменты профилирования памяти

```typescript
// Инструмент для измерения использования памяти
class MemoryProfiler {
  static measureMemory(): { used: number; total: number; percentage: number } | null {
    if (typeof performance === 'undefined' || !performance.memory) {
      return null;
    }
    
    const memory = (performance as any).memory;
    const used = memory.usedJSHeapSize;
    const total = memory.totalJSHeapSize;
    const percentage = (used / total) * 100;
    
    return { used, total, percentage };
  }
  
  static logMemory(label: string): void {
    const memory = MemoryProfiler.measureMemory();
    if (memory) {
      console.log(`${label} - Память: ${Math.round(memory.used / 1024 / 1024)}MB / ${Math.round(memory.total / 1024 / 1024)}MB (${memory.percentage.toFixed(1)}%)`);
    }
  }
  
  static async measureMemoryGrowth<T>(fn: () => T): Promise<{ result: T; growth: number | null }> {
    const before = MemoryProfiler.measureMemory();
    const result = fn();
    const after = MemoryProfiler.measureMemory();
    
    const growth = before && after ? after.used - before.used : null;
    
    return { result, growth };
  }
  
  static async measureMemoryGrowthAsync<T>(fn: () => Promise<T>): Promise<{ result: T; growth: number | null }> {
    const before = MemoryProfiler.measureMemory();
    const result = await fn();
    const after = MemoryProfiler.measureMemory();
    
    const growth = before && after ? after.used - before.used : null;
    
    return { result, growth };
  }
  
  static trackMemoryUsage<T>(label: string, fn: () => T): T {
    this.logMemory(`${label} (начало)`);
    const result = fn();
    this.logMemory(`${label} (конец)`);
    return result;
  }
  
  static async trackMemoryUsageAsync<T>(label: string, fn: () => Promise<T>): Promise<T> {
    this.logMemory(`${label} (начало)`);
    const result = await fn();
    this.logMemory(`${label} (конец)`);
    return result;
  }
}

// Пример использования memory profiler
MemoryProfiler.logMemory("Начало");

const largeArray = Array.from({ length: 1000000 }, (_, i) => ({ id: i, value: Math.random() }));
MemoryProfiler.logMemory("После создания большого массива");

largeArray.splice(0, 500000); // Удаляем половину элементов
MemoryProfiler.logMemory("После удаления элементов");

// Force garbage collection if available (for testing purposes)
if ((global as any).gc) {
  (global as any).gc();
  MemoryProfiler.logMemory("После сборки мусора");
}
```

### Расширенный профайлер памяти

```typescript
// Расширенный профайлер памяти с отслеживанием утечек
class AdvancedMemoryProfiler {
  private memorySnapshots: Map<string, { used: number; timestamp: number }> = new Map();
  
  snapshot(label: string): void {
    const memory = this.getMemoryInfo();
    if (memory) {
      this.memorySnapshots.set(label, {
        used: memory.used,
        timestamp: Date.now()
      });
      console.log(`${label}: ${this.formatMemory(memory.used)}`);
    }
  }
  
  private getMemoryInfo(): { used: number; total: number; percentage: number } | null {
    if (typeof performance === 'undefined' || !performance.memory) {
      return null;
    }
    
    const memory = (performance as any).memory;
    const used = memory.usedJSHeapSize;
    const total = memory.totalJSHeapSize;
    const percentage = (used / total) * 100;
    
    return { used, total, percentage };
  }
  
  private formatMemory(bytes: number): string {
    const mb = bytes / 1024 / 1024;
    return `${mb.toFixed(2)} MB`;
  }
  
  analyzeGrowth(startLabel: string, endLabel: string): { growth: number; duration: number } | null {
    const start = this.memorySnapshots.get(startLabel);
    const end = this.memorySnapshots.get(endLabel);
    
    if (!start || !end) {
      return null;
    }
    
    const growth = end.used - start.used;
    const duration = end.timestamp - start.timestamp;
    
    return { growth, duration };
  }
  
  report(): void {
    console.log("\n=== Отчет по использованию памяти ===");
    
    const snapshots = Array.from(this.memorySnapshots.entries())
      .sort(([, a], [, b]) => a.timestamp - b.timestamp);
    
    if (snapshots.length === 0) {
      console.log("Нет данных о памяти");
      return;
    }
    
    let previous: { used: number; timestamp: number } | null = null;
    
    for (const [label, snapshot] of snapshots) {
      const formattedMemory = this.formatMemory(snapshot.used);
      
      if (previous) {
        const growth = snapshot.used - previous.used;
        const growthFormatted = this.formatMemory(Math.abs(growth));
        const sign = growth >= 0 ? '+' : '';
        
        console.log(`${label}: ${formattedMemory} (${sign}${growthFormatted})`);
      } else {
        console.log(`${label}: ${formattedMemory}`);
      }
      
      previous = snapshot;
    }
  }
  
  detectLeaks(threshold: number = 10 * 1024 * 1024): string[] { // 10MB по умолчанию
    const leaks: string[] = [];
    const snapshots = Array.from(this.memorySnapshots.entries())
      .sort(([, a], [, b]) => a.timestamp - b.timestamp);
    
    for (let i = 1; i < snapshots.length; i++) {
      const [, current] = snapshots[i];
      const [, previous] = snapshots[i - 1];
      
      const growth = current.used - previous.used;
      if (growth > threshold) {
        leaks.push(snapshots[i][0]);
      }
    }
    
    return leaks;
  }
  
  clear(): void {
    this.memorySnapshots.clear();
  }
}

// Пример использования расширенного профайлера памяти
const memoryProfiler = new AdvancedMemoryProfiler();

memoryProfiler.snapshot("Начало");

// Создаем данные
const dataSets: any[] = [];
for (let i = 0; i < 100; i++) {
  dataSets.push(Array.from({ length: 10000 }, (_, j) => ({
    id: j,
    value: Math.random(),
    data: new Array(100).fill(0).map(() => Math.random())
  })));
  
  if (i % 20 === 0) {
    memoryProfiler.snapshot(`После создания набора ${i}`);
  }
}

memoryProfiler.snapshot("После создания всех наборов");

// Освобождаем часть данных
dataSets.splice(0, 50);
memoryProfiler.snapshot("После освобождения части данных");

// Генерируем отчет
memoryProfiler.report();

// Проверяем утечки
const leaks = memoryProfiler.detectLeaks(5 * 1024 * 1024); // 5MB порог
if (leaks.length > 0) {
  console.log("Обнаружены потенциальные утечки памяти:", leaks);
} else {
  console.log("Утечки памяти не обнаружены");
}
```

## Профилирование CPU

Профилирование CPU помогает понять, какие функции потребляют больше всего процессорного времени.

### CPU профайлер

```typescript
// CPU профайлер для анализа загрузки процессора
class CPUProfiler {
  private cpuMeasurements: Map<string, { 
    startTime: number; 
    startCpuTime: number;
    endTime?: number;
    endCpuTime?: number;
  }> = new Map();
  
  start(label: string): void {
    if (typeof performance === 'undefined') return;
    
    const startTime = performance.now();
    const startCpuTime = this.getCpuTime();
    
    this.cpuMeasurements.set(label, {
      startTime,
      startCpuTime
    });
  }
  
  end(label: string): { duration: number; cpuTime: number; cpuUsage: number } | null {
    if (typeof performance === 'undefined') return null;
    
    const measurement = this.cpuMeasurements.get(label);
    if (!measurement) {
      throw new Error(`Нет начального измерения для ${label}`);
    }
    
    const endTime = performance.now();
    const endCpuTime = this.getCpuTime();
    
    measurement.endTime = endTime;
    measurement.endCpuTime = endCpuTime;
    
    const duration = endTime - measurement.startTime;
    const cpuTime = endCpuTime - measurement.startCpuTime;
    const cpuUsage = duration > 0 ? (cpuTime / duration) * 100 : 0;
    
    console.log(`${label}: Длительность=${duration.toFixed(2)}ms, CPU время=${cpuTime.toFixed(2)}ms, Загрузка CPU=${cpuUsage.toFixed(2)}%`);
    
    return { duration, cpuTime, cpuUsage };
  }
  
  private getCpuTime(): number {
    // В браузере это приблизительное значение
    // В Node.js можно использовать process.cpuUsage()
    if (typeof process !== 'undefined' && process.cpuUsage) {
      const usage = process.cpuUsage();
      return (usage.user + usage.system) / 1000; // Преобразуем в миллисекунды
    }
    
    // В браузере возвращаем приблизительное значение
    return performance.now();
  }
  
  async timeWithCPU<T>(label: string, fn: () => Promise<T>): Promise<T> {
    this.start(label);
    try {
      const result = await fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  timeWithCPU<T>(label: string, fn: () => T): T {
    this.start(label);
    try {
      const result = fn();
      this.end(label);
      return result;
    } catch (error) {
      this.end(label);
      throw error;
    }
  }
  
  getStats(label: string): { 
    avgDuration: number; 
    avgCpuTime: number; 
    avgCpuUsage: number;
    count: number;
  } | null {
    // Эта реализация упрощена, в реальном профайлере нужно собирать статистику
    // по множественным вызовам
    return null;
  }
}

// Пример использования CPU профайлера
const cpuProfiler = new CPUProfiler();

// CPU-интенсивная операция
function cpuIntensiveOperation(): number {
  let result = 0;
  for (let i = 0; i < 10000000; i++) {
    result += Math.sin(i) * Math.cos(i);
  }
  return result;
}

// Измеряем CPU использование
cpuProfiler.timeWithCPU("CPU интенсивная операция", cpuIntensiveOperation);

// Асинхронная операция с CPU нагрузкой
async function asyncCpuOperation(): Promise<number> {
  // Симуляция CPU нагрузки
  const result = cpuIntensiveOperation();
  await new Promise(resolve => setTimeout(resolve, 10));
  return result;
}

cpuProfiler.timeWithCPU("Асинхронная CPU операция", asyncCpuOperation);
```

## Анализ производительности функций

Анализ производительности отдельных функций помогает оптимизировать критические участки кода.

### Профайлер функций

```typescript
// Профайлер для анализа производительности отдельных функций
class FunctionProfiler {
  private functionStats: Map<string, {
    calls: number;
    totalDuration: number;
    totalCpuTime: number;
    minDuration: number;
    maxDuration: number;
    minCpuTime: number;
    maxCpuTime: number;
    lastCallTime: number;
  }> = new Map();
  
  profile<T extends (...args: any[]) => any>(
    fn: T,
    name: string
  ): T {
    const self = this;
    
    return function (...args: any[]) {
      const startTime = performance.now();
      const startCpuTime = self.getCpuTime();
      
      try {
        const result = fn.apply(this, args);
        
        const endTime = performance.now();
        const endCpuTime = self.getCpuTime();
        
        self.recordCall(name, endTime - startTime, endCpuTime - startCpuTime);
        
        return result;
      } catch (error) {
        const endTime = performance.now();
        const endCpuTime = self.getCpuTime();
        
        self.recordCall(name, endTime - startTime, endCpuTime - startCpuTime);
        
        throw error;
      }
    } as T;
  }
  
  private getCpuTime(): number {
    if (typeof process !== 'undefined' && process.cpuUsage) {
      const usage = process.cpuUsage();
      return (usage.user + usage.system) / 1000;
    }
    return performance.now();
  }
  
  private recordCall(name: string, duration: number, cpuTime: number): void {
    const stats = this.functionStats.get(name) || {
      calls: 0,
      totalDuration: 0,
      totalCpuTime: 0,
      minDuration: Infinity,
      maxDuration: -Infinity,
      minCpuTime: Infinity,
      maxCpuTime: -Infinity,
      lastCallTime: 0
    };
    
    stats.calls++;
    stats.totalDuration += duration;
    stats.totalCpuTime += cpuTime;
    stats.minDuration = Math.min(stats.minDuration, duration);
    stats.maxDuration = Math.max(stats.maxDuration, duration);
    stats.minCpuTime = Math.min(stats.minCpuTime, cpuTime);
    stats.maxCpuTime = Math.max(stats.maxCpuTime, cpuTime);
    stats.lastCallTime = Date.now();
    
    this.functionStats.set(name, stats);
  }
  
  getStats(name: string): {
    calls: number;
    avgDuration: number;
    avgCpuTime: number;
    minDuration: number;
    maxDuration: number;
    minCpuTime: number;
    maxCpuTime: number;
    lastCallTime: number;
  } | null {
    const stats = this.functionStats.get(name);
    if (!stats) return null;
    
    return {
      calls: stats.calls,
      avgDuration: stats.totalDuration / stats.calls,
      avgCpuTime: stats.totalCpuTime / stats.calls,
      minDuration: stats.minDuration,
      maxDuration: stats.maxDuration,
      minCpuTime: stats.minCpuTime,
      maxCpuTime: stats.maxCpuTime,
      lastCallTime: stats.lastCallTime
    };
  }
  
  getAllStats(): Record<string, ReturnType<this["getStats"]>> {
    const result: Record<string, any> = {};
    for (const name of this.functionStats.keys()) {
      const stats = this.getStats(name);
      if (stats) {
        result[name] = stats;
      }
    }
    return result;
  }
  
  report(): void {
    console.log("\n=== Отчет по производительности функций ===");
    
    const stats = this.getAllStats();
    const sortedEntries = Object.entries(stats)
      .sort(([, a], [, b]) => b.avgDuration - a.avgDuration);
    
    for (const [name, stat] of sortedEntries) {
      console.log(`${name}:`);
      console.log(`  Вызовов: ${stat.calls}`);
      console.log(`  Среднее время: ${stat.avgDuration.toFixed(4)}ms`);
      console.log(`  Мин/Макс время: ${stat.minDuration.toFixed(4)}ms / ${stat.maxDuration.toFixed(4)}ms`);
      console.log(`  Среднее CPU время: ${stat.avgCpuTime.toFixed(4)}ms`);
    }
  }
  
  clear(): void {
    this.functionStats.clear();
  }
}

// Пример использования профайлера функций
const functionProfiler = new FunctionProfiler();

// Профилируемые функции
const profiledFibonacci = functionProfiler.profile(
  function fibonacci(n: number): number {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  },
  "fibonacci"
);

const profiledArraySum = functionProfiler.profile(
  function arraySum(arr: number[]): number {
    return arr.reduce((sum, val) => sum + val, 0);
  },
  "arraySum"
);

const profiledObjectCreation = functionProfiler.profile(
  function createObjects(count: number): any[] {
    return Array.from({ length: count }, (_, i) => ({
      id: i,
      value: Math.random(),
      timestamp: Date.now()
    }));
  },
  "createObjects"
);

// Выполняем вызовы профилируемых функций
console.log("Выполняем профилируемые функции...");

profiledFibonacci(10);
profiledFibonacci(15);
profiledFibonacci(20);

const testArray = Array.from({ length: 10000 }, (_, i) => i);
profiledArraySum(testArray);
profiledArraySum(testArray.slice(0, 5000));
profiledArraySum(testArray.slice(0, 1000));

profiledObjectCreation(1000);
profiledObjectCreation(5000);
profiledObjectCreation(10000);

// Выводим отчет
functionProfiler.report();
```

## Профилирование асинхронных операций

Профилирование асинхронных операций требует специальных подходов для учета времени ожидания и выполнения.

### Асинхронный профайлер

```typescript
// Профайлер для асинхронных операций
class AsyncProfiler {
  private asyncOperations: Map<string, {
    startTime: number;
    endTime?: number;
    duration?: number;
    waitTime?: number;
    executionTime?: number;
    status: 'pending' | 'resolved' | 'rejected';
  }> = new Map();
  
  async profile<T>(
    operation: () => Promise<T>,
    label: string
  ): Promise<T> {
    const operationId = `${label}-${Date.now()}-${Math.random()}`;
    
    const startTime = performance.now();
    this.asyncOperations.set(operationId, {
      startTime,
      status: 'pending'
    });
    
    try {
      const result = await operation();
      
      const endTime = performance.now();
      const operationRecord = this.asyncOperations.get(operationId)!;
      operationRecord.endTime = endTime;
      operationRecord.duration = endTime - startTime;
      operationRecord.status = 'resolved';
      
      console.log(`${label}: Общее время=${operationRecord.duration.toFixed(2)}ms`);
      
      return result;
    } catch (error) {
      const endTime = performance.now();
      const operationRecord = this.asyncOperations.get(operationId)!;
      operationRecord.endTime = endTime;
      operationRecord.duration = endTime - startTime;
      operationRecord.status = 'rejected';
      
      console.log(`${label}: Ошибка после ${operationRecord.duration.toFixed(2)}ms`);
      
      throw error;
    }
  }
  
  profileWithDetails<T>(
    operation: () => Promise<T>,
    label: string,
    options: {
      onWaitStart?: () => void;
      onWaitEnd?: () => void;
      onExecutionStart?: () => void;
      onExecutionEnd?: () => void;
    } = {}
  ): Promise<T> {
    const operationId = `${label}-${Date.now()}-${Math.random()}`;
    
    const startTime = performance.now();
    this.asyncOperations.set(operationId, {
      startTime,
      status: 'pending'
    });
    
    let waitStartTime: number | null = null;
    let executionStartTime: number | null = null;
    
    // Оборачиваем операцию для отслеживания этапов
    const wrappedOperation = async (): Promise<T> => {
      // Фаза ожидания (например, сетевой запрос)
      waitStartTime = performance.now();
      options.onWaitStart?.();
      
      try {
        const result = await operation();
        
        const waitEndTime = performance.now();
        options.onWaitEnd?.();
        
        // Фаза выполнения (обработка результата)
        executionStartTime = performance.now();
        options.onExecutionStart?.();
        
        // Симуляция обработки результата
        const processedResult = await this.processResult(result);
        
        options.onExecutionEnd?.();
        const executionEndTime = performance.now();
        
        // Записываем детали
        const operationRecord = this.asyncOperations.get(operationId)!;
        operationRecord.waitTime = waitEndTime - waitStartTime;
        operationRecord.executionTime = executionEndTime - executionStartTime;
        
        return processedResult;
      } catch (error) {
        options.onWaitEnd?.();
        if (executionStartTime) {
          options.onExecutionEnd?.();
        }
        throw error;
      }
    };
    
    return this.profile(wrappedOperation, label);
  }
  
  private async processResult<T>(result: T): Promise<T> {
    // Симуляция обработки результата
    await new Promise(resolve => setTimeout(resolve, Math.random() * 10));
    return result;
  }
  
  getOperationStats(label: string): {
    totalOperations: number;
    avgDuration: number;
    minDuration: number;
    maxDuration: number;
    successRate: number;
    avgWaitTime?: number;
    avgExecutionTime?: number;
  } {
    const operations = Array.from(this.asyncOperations.entries())
      .filter(([key]) => key.startsWith(label));
    
    if (operations.length === 0) {
      return {
        totalOperations: 0,
        avgDuration: 0,
        minDuration: 0,
        maxDuration: 0,
        successRate: 0
      };
    }
    
    const durations = operations
      .map(([, op]) => op.duration)
      .filter((d): d is number => d !== undefined);
    
    const waitTimes = operations
      .map(([, op]) => op.waitTime)
      .filter((t): t is number => t !== undefined);
    
    const executionTimes = operations
      .map(([, op]) => op.executionTime)
      .filter((t): t is number => t !== undefined);
    
    const successful = operations
      .filter(([, op]) => op.status === 'resolved')
      .length;
    
    return {
      totalOperations: operations.length,
      avgDuration: durations.reduce((sum, d) => sum + d, 0) / durations.length,
      minDuration: Math.min(...durations),
      maxDuration: Math.max(...durations),
      successRate: successful / operations.length,
      avgWaitTime: waitTimes.length > 0 ? waitTimes.reduce((sum, t) => sum + t, 0) / waitTimes.length : undefined,
      avgExecutionTime: executionTimes.length > 0 ? executionTimes.reduce((sum, t) => sum + t, 0) / executionTimes.length : undefined
    };
  }
  
  report(): void {
    console.log("\n=== Отчет по асинхронным операциям ===");
    
    const labels = new Set<string>();
    for (const key of this.asyncOperations.keys()) {
      const label = key.split('-')[0];
      labels.add(label);
    }
    
    for (const label of labels) {
      const stats = this.getOperationStats(label);
      console.log(`${label}:`);
      console.log(`  Всего операций: ${stats.totalOperations}`);
      console.log(`  Среднее время: ${stats.avgDuration.toFixed(2)}ms`);
      console.log(`  Мин/Макс время: ${stats.minDuration.toFixed(2)}ms / ${stats.maxDuration.toFixed(2)}ms`);
      console.log(`  Успешность: ${(stats.successRate * 100).toFixed(1)}%`);
      
      if (stats.avgWaitTime !== undefined) {
        console.log(`  Среднее время ожидания: ${stats.avgWaitTime.toFixed(2)}ms`);
      }
      
      if (stats.avgExecutionTime !== undefined) {
        console.log(`  Среднее время выполнения: ${stats.avgExecutionTime.toFixed(2)}ms`);
      }
    }
  }
}

// Пример использования асинхронного профайлера
const asyncProfiler = new AsyncProfiler();

// Асинхронные операции для тестирования
async function fetchData(id: number): Promise<{ id: number; data: string }> {
  // Симуляция сетевого запроса
  await new Promise(resolve => setTimeout(resolve, 100 + Math.random() * 200));
  return { id, data: `Данные для ID ${id}` };
}

async function processData(data: { id: number; data: string }): Promise<string> {
  // Симуляция обработки данных
  await new Promise(resolve => setTimeout(resolve, 50 + Math.random() * 100));
  return `Обработанные данные: ${data.data}`;
}

// Профилирование асинхронных операций
async function testAsyncProfiling() {
  console.log("Начало тестирования асинхронного профилирования...");
  
  // Простое профилирование
  for (let i = 1; i <= 5; i++) {
    await asyncProfiler.profile(
      () => fetchData(i),
      "fetchData"
    );
  }
  
  // Профилирование с деталями
  for (let i = 1; i <= 3; i++) {
    await asyncProfiler.profileWithDetails(
      () => fetchData(i),
      "fetchDataDetailed",
      {
        onWaitStart: () => console.log(`Начало ожидания для ID ${i}`),
        onWaitEnd: () => console.log(`Конец ожидания для ID ${i}`),
        onExecutionStart: () => console.log(`Начало выполнения для ID ${i}`),
        onExecutionEnd: () => console.log(`Конец выполнения для ID ${i}`)
      }
    );
  }
  
  // Выводим отчет
  asyncProfiler.report();
}

// testAsyncProfiling();
```

## Практические примеры

### Комплексный пример профилирования веб-приложения

```typescript
// Комплексный пример профилирования веб-приложения
class WebAppProfiler {
  private profiler: AdvancedProfiler;
  private memoryProfiler: AdvancedMemoryProfiler;
  private functionProfiler: FunctionProfiler;
  private asyncProfiler: AsyncProfiler;
  
  constructor() {
    this.profiler = new AdvancedProfiler();
    this.memoryProfiler = new AdvancedMemoryProfiler();
    this.functionProfiler = new FunctionProfiler();
    this.asyncProfiler = new AsyncProfiler();
  }
  
  // Профилируемые функции приложения
  private profiledFunctions = {
    renderComponent: this.functionProfiler.profile(
      (component: string, props: any) => {
        // Симуляция рендеринга компонента
        const startTime = Date.now();
        while (Date.now() - startTime < 5 + Math.random() * 10) {
          // Имитация работы
        }
        return `<div class="${component}">Отрендеренный компонент с props: ${JSON.stringify(props)}</div>`;
      },
      "renderComponent"
    ),
    
    updateState: this.functionProfiler.profile(
      (state: any, updates: any) => {
        return { ...state, ...updates };
      },
      "updateState"
    ),
    
    processEvent: this.functionProfiler.profile(
      (event: any) => {
        // Симуляция обработки события
        const processed = {
          ...event,
          timestamp: Date.now(),
          processed: true
        };
        
        // Дополнительная обработка
        if (event.type === 'click') {
          processed.action = 'navigation';
        }
        
        return processed;
      },
      "processEvent"
    )
  };
  
  // Асинхронные операции
  private async fetchUserData(userId: number): Promise<any> {
    await new Promise(resolve => setTimeout(resolve, 100 + Math.random() * 200));
    return {
      id: userId,
      name: `Пользователь ${userId}`,
      email: `user${userId}@example.com`,
      preferences: {
        theme: Math.random() > 0.5 ? 'dark' : 'light',
        notifications: true
      }
    };
  }
  
  private async fetchAppData(): Promise<any> {
    await new Promise(resolve => setTimeout(resolve, 50 + Math.random() * 150));
    return {
      version: '1.0.0',
      features: ['feature1', 'feature2', 'feature3'],
      config: {
        apiUrl: 'https://api.example.com',
        timeout: 5000
      }
    };
  }
  
  // Основной цикл приложения
  async runApp(): Promise<void> {
    console.log("=== Запуск профилирования веб-приложения ===");
    
    // Начальные измерения
    this.profiler.start("appInitialization", { category: "lifecycle" });
    this.memoryProfiler.snapshot("appStart");
    
    // Загрузка данных
    this.profiler.start("dataLoading", { category: "data" });
    
    const [userData, appData] = await Promise.all([
      this.asyncProfiler.profile(() => this.fetchUserData(123), "fetchUserData"),
      this.asyncProfiler.profile(() => this.fetchAppData(), "fetchAppData")
    ]);
    
    this.profiler.end("dataLoading");
    
    // Инициализация состояния
    this.profiler.start("stateInitialization", { category: "state" });
    
    let appState = this.profiledFunctions.updateState({}, {
      user: userData,
      app: appData,
      ui: {
        loading: false,
        error: null
      }
    });
    
    this.profiler.end("stateInitialization");
    
    // Рендеринг UI
    this.profiler.start("uiRendering", { category: "rendering" });
    this.memoryProfiler.snapshot("beforeRendering");
    
    // Рендерим несколько компонентов
    const components = ['header', 'sidebar', 'main', 'footer'];
    const renderedComponents = components.map(component => 
      this.profiledFunctions.renderComponent(component, { 
        user: appState.user,
        theme: appState.user.preferences.theme
      })
    );
    
    this.profiler.end("uiRendering");
    this.memoryProfiler.snapshot("afterRendering");
    
    // Симуляция пользовательских действий
    this.profiler.start("userInteraction", { category: "interaction" });
    
    // Симуляция кликов
    for (let i = 0; i < 5; i++) {
      const clickEvent = {
        type: 'click',
        target: `button-${i}`,
        x: Math.random() * 1000,
        y: Math.random() * 800
      };
      
      const processedEvent = this.profiledFunctions.processEvent(clickEvent);
      
      // Обновляем состояние
      appState = this.profiledFunctions.updateState(appState, {
        lastEvent: processedEvent
      });
    }
    
    this.profiler.end("userInteraction");
    
    // Финальные измерения
    this.profiler.end("appInitialization");
    this.memoryProfiler.snapshot("appEnd");
    
    // Выводим отчеты
    this.generateReport();
  }
  
  private generateReport(): void {
    console.log("\n" + "=".repeat(50));
    console.log("ОТЧЕТ ПРОФИЛИРОВАНИЯ ВЕБ-ПРИЛОЖЕНИЯ");
    console.log("=".repeat(50));
    
    // Отчет по времени выполнения
    this.profiler.report();
    
    // Отчет по памяти
    console.log("\n--- Использование памяти ---");
    this.memoryProfiler.report();
    
    // Отчет по функциям
    console.log("\n--- Производительность функций ---");
    this.functionProfiler.report();
    
    // Отчет по асинхронным операциям
    console.log("\n--- Асинхронные операции ---");
    this.asyncProfiler.report();
  }
}

// Запуск комплексного примера
async function runWebAppProfiling() {
  const appProfiler = new WebAppProfiler();
  await appProfiler.runApp();
}

// runWebAppProfiling();
```

### Инструмент для непрерывного профилирования

```typescript
// Инструмент для непрерывного профилирования в production
class ContinuousProfiler {
  private isRunning = false;
  private intervalId: NodeJS.Timeout | null = null;
  private metrics: {
    timestamp: number;
    cpu: number;
    memory: number;
    eventLoopDelay: number;
    activeHandles: number;
    activeRequests: number;
  }[] = [];
  
  start(interval: number = 1000): void {
    if (this.isRunning) {
      console.log("Профайлер уже запущен");
      return;
    }
    
    this.isRunning = true;
    console.log("Непрерывный профайлер запущен");
    
    this.intervalId = setInterval(() => {
      this.collectMetrics();
    }, interval);
  }
  
  stop(): void {
    if (!this.isRunning) {
      console.log("Профайлер не запущен");
      return;
    }
    
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
    
    this.isRunning = false;
    console.log("Непрерывный профайлер остановлен");
  }
  
  private collectMetrics(): void {
    const timestamp = Date.now();
    
    // Сбор метрик CPU
    const cpu = this.getCpuUsage();
    
    // Сбор метрик памяти
    const memory = this.getMemoryUsage();
    
    // Сбор метрик event loop
    const eventLoopDelay = this.getEventLoopDelay();
    
    // Сбор метрик активных handles/requests (Node.js)
    const { activeHandles, activeRequests } = this.getActiveHandlesAndRequests();
    
    const metric = {
      timestamp,
      cpu,
      memory,
      eventLoopDelay,
      activeHandles,
      activeRequests
    };
    
    this.metrics.push(metric);
    
    // Ограничиваем размер буфера
    if (this.metrics.length > 1000) {
      this.metrics.shift();
    }
    
    // Логируем критические метрики
    if (cpu > 80) {
      console.warn(`Высокая загрузка CPU: ${cpu.toFixed(2)}%`);
    }
    
    if (memory > 80) {
      console.warn(`Высокое использование памяти: ${memory.toFixed(2)}%`);
    }
    
    if (eventLoopDelay > 100) {
      console.warn(`Высокая задержка event loop: ${eventLoopDelay.toFixed(2)}ms`);
    }
  }
  
  private getCpuUsage(): number {
    if (typeof process !== 'undefined' && process.cpuUsage) {
      const usage = process.cpuUsage();
      // Упрощенный расчет CPU usage
      return Math.min(100, (usage.user + usage.system) / 1000000 * 100);
    }
    return 0;
  }
  
  private getMemoryUsage(): number {
    if (typeof process !== 'undefined') {
      const used = process.memoryUsage().heapUsed;
      const total = process.memoryUsage().heapTotal;
      return (used / total) * 100;
    }
    return 0;
  }
  
  private getEventLoopDelay(): number {
    const start = process.hrtime.bigint();
    
    // Выполняем setImmediate для измерения задержки
    setImmediate(() => {
      const end = process.hrtime.bigint();
      const delay = Number(end - start) / 1000000; // Преобразуем в миллисекунды
      // В реальной реализации здесь нужно сохранить delay
    });
    
    // Возвращаем предыдущее значение или 0
    return 0;
  }
  
  private getActiveHandlesAndRequests(): { activeHandles: number; activeRequests: number } {
    if (typeof process !== 'undefined') {
      return {
        activeHandles: process._getActiveHandles ? process._getActiveHandles().length : 0,
        activeRequests: process._getActiveRequests ? process._getActiveRequests().length : 0
      };
    }
    return { activeHandles: 0, activeRequests: 0 };
  }
  
  getReport(period: number = 60000): {
    avgCpu: number;
    maxCpu: number;
    avgMemory: number;
    maxMemory: number;
    avgEventLoopDelay: number;
    maxEventLoopDelay: number;
    metricsCount: number;
  } {
    const now = Date.now();
    const periodMetrics = this.metrics.filter(m => now - m.timestamp <= period);
    
    if (periodMetrics.length === 0) {
      return {
        avgCpu: 0,
        maxCpu: 0,
        avgMemory: 0,
        maxMemory: 0,
        avgEventLoopDelay: 0,
        maxEventLoopDelay: 0,
        metricsCount: 0
      };
    }
    
    const cpuValues = periodMetrics.map(m => m.cpu);
    const memoryValues = periodMetrics.map(m => m.memory);
    const delayValues = periodMetrics.map(m => m.eventLoopDelay);
    
    return {
      avgCpu: cpuValues.reduce((sum, val) => sum + val, 0) / cpuValues.length,
      maxCpu: Math.max(...cpuValues),
      avgMemory: memoryValues.reduce((sum, val) => sum + val, 0) / memoryValues.length,
      maxMemory: Math.max(...memoryValues),
      avgEventLoopDelay: delayValues.reduce((sum, val) => sum + val, 0) / delayValues.length,
      maxEventLoopDelay: Math.max(...delayValues),
      metricsCount: periodMetrics.length
    };
  }
  
  exportMetrics(): any[] {
    return [...this.metrics];
  }
  
  clearMetrics(): void {
    this.metrics = [];
  }
}

// Пример использования непрерывного профайлера
function setupContinuousProfiling() {
  const profiler = new ContinuousProfiler();
  
  // Запускаем профайлер
  profiler.start(5000); // Сбор метрик каждые 5 секунд
  
  // Через 2 минуты выводим отчет
  setTimeout(() => {
    const report = profiler.getReport(120000); // За последние 2 минуты
    console.log("Отчет непрерывного профилирования:");
    console.log(`Средняя CPU загрузка: ${report.avgCpu.toFixed(2)}%`);
    console.log(`Максимальная CPU загрузка: ${report.maxCpu.toFixed(2)}%`);
    console.log(`Среднее использование памяти: ${report.avgMemory.toFixed(2)}%`);
    console.log(`Максимальное использование памяти: ${report.maxMemory.toFixed(2)}%`);
    console.log(`Средняя задержка event loop: ${report.avgEventLoopDelay.toFixed(2)}ms`);
    console.log(`Максимальная задержка event loop: ${report.maxEventLoopDelay.toFixed(2)}ms`);
    
    // Останавливаем профайлер
    profiler.stop();
  }, 120000);
}

// setupContinuousProfiling();
```

Профилирование и измерение производительности являются критически важными аспектами разработки высокопроизводительных функциональных приложений. Использование различных инструментов и техник профилирования позволяет разработчикам выявлять узкие места, оптимизировать код и обеспечивать стабильную работу приложений под нагрузкой. Правильное применение профилирования помогает создавать более эффективные и надежные программные решения.