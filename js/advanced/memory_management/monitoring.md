# Мониторинг и профилирование памяти в JavaScript

## Введение

Мониторинг и профилирование памяти - важные инструменты для выявления проблем с производительностью, утечек памяти и оптимизации использования ресурсов в JavaScript приложениях. Правильное использование этих инструментов помогает создавать более эффективные и стабильные приложения.

## Использование Chrome DevTools

Chrome DevTools предоставляет мощные возможности для анализа использования памяти:

### Memory Tab

Вкладка Memory позволяет создавать heap snapshots и анализировать распределение памяти:

```javascript
// Пример кода для профилирования памяти
class MemoryProfiler {
  static takeHeapSnapshot() {
    if (performance.memory) {
      return {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit
      };
    }
    return null;
  }
  
  static measureFunctionMemory(fn, ...args) {
    const before = this.takeHeapSnapshot();
    const result = fn(...args);
    const after = this.takeHeapSnapshot();
    
    if (before && after) {
      console.log('Изменение памяти:', {
        used: after.used - before.used,
        total: after.total - before.total
      });
    }
    
    return result;
  }
  
  static createMemoryLeak() {
    const leak = [];
    return function() {
      // Эта функция создает утечку памяти
      leak.push(new Array(10000).fill(0));
    };
  }
}

// Пример использования профилировщика
function testFunction() {
  const data = new Array(100000).fill(0);
  return data.reduce((sum, val) => sum + val, 0);
}

// MemoryProfiler.measureFunctionMemory(testFunction);
```

### Performance Tab

Вкладка Performance позволяет отслеживать сборки мусора и другие события, связанные с памятью:

```javascript
// Создание контрольных точек для профилирования
class ProfilingTools {
  // Создание контрольных точек
  static checkpoint(label) {
    if (performance.mark) {
      performance.mark(label);
    }
  }
  
  // Измерение между контрольными точками
  static measure(startLabel, endLabel, measureName) {
    if (performance.measure) {
      performance.measure(measureName, startLabel, endLabel);
    }
  }
  
  // Получение всех измерений
  static getMeasurements() {
    if (performance.getEntriesByType) {
      return performance.getEntriesByType('measure');
    }
    return [];
  }
  
  // Пример использования
  static profileOperation() {
    this.checkpoint('start-operation');
    
    // Операция для профилирования
    const data = new Array(100000).fill(0).map((_, i) => i);
    const result = data.filter(x => x % 2 === 0).map(x => x * 2);
    
    this.checkpoint('end-operation');
    this.measure('start-operation', 'end-operation', 'data-processing');
    
    console.log(this.getMeasurements());
  }
}
```

## Создание отчетов об использовании памяти

Система отчетов помогает отслеживать использование памяти во времени:

```javascript
// Система отчетов об использовании памяти
class MemoryReport {
  constructor() {
    this.reports = [];
  }
  
  addReport(label, data) {
    const report = {
      label,
      timestamp: Date.now(),
      memory: performance.memory ? {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit
      } : null,
      data: data ? JSON.parse(JSON.stringify(data)) : null
    };
    
    this.reports.push(report);
  }
  
  generateReport() {
    return this.reports.map(report => ({
      label: report.label,
      timestamp: new Date(report.timestamp).toISOString(),
      memory: report.memory,
      dataSize: report.data ? JSON.stringify(report.data).length : 0
    }));
  }
  
  clear() {
    this.reports = [];
  }
}

// const memoryReport = new MemoryReport();
// memoryReport.addReport('Начало', { step: 1 });
// // ... выполнение кода
// memoryReport.addReport('Конец', { step: 2 });
// console.log(memoryReport.generateReport());
```

## Мониторинг в реальном времени

Система непрерывного мониторинга памяти:

```javascript
// Непрерывный мониторинг памяти
class ContinuousMemoryMonitor {
  constructor() {
    this.measurements = [];
    this.isMonitoring = false;
    this.monitoringInterval = null;
  }
  
  start(interval = 1000) {
    if (this.isMonitoring) return;
    
    this.isMonitoring = true;
    this.monitoringInterval = setInterval(() => {
      this.takeMeasurement();
    }, interval);
  }
  
  stop() {
    if (!this.isMonitoring) return;
    
    clearInterval(this.monitoringInterval);
    this.isMonitoring = false;
  }
  
  takeMeasurement() {
    if (performance.memory) {
      const measurement = {
        timestamp: Date.now(),
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit
      };
      
      this.measurements.push(measurement);
      
      // Ограничиваем размер массива для предотвращения утечки памяти
      if (this.measurements.length > 1000) {
        this.measurements.shift();
      }
      
      return measurement;
    }
    return null;
  }
  
  getStats() {
    if (this.measurements.length === 0) return null;
    
    const usedValues = this.measurements.map(m => m.used);
    const totalValues = this.measurements.map(m => m.total);
    
    return {
      avgUsed: usedValues.reduce((a, b) => a + b, 0) / usedValues.length,
      maxUsed: Math.max(...usedValues),
      minUsed: Math.min(...usedValues),
      avgTotal: totalValues.reduce((a, b) => a + b, 0) / totalValues.length,
      measurementsCount: this.measurements.length
    };
  }
  
  detectLeaks(threshold = 0.1) {
    if (this.measurements.length < 10) return false;
    
    // Анализируем последние 10 измерений
    const recent = this.measurements.slice(-10);
    const initial = recent[0].used;
    const current = recent[recent.length - 1].used;
    const growth = (current - initial) / initial;
    
    return growth > threshold;
  }
}
```

## Инструменты для автоматического тестирования памяти

Автоматизированные тесты для проверки утечек памяти:

```javascript
// Автоматическое тестирование памяти
class MemoryTest {
  static async testForLeaks(operation, iterations = 100) {
    const monitor = new ContinuousMemoryMonitor();
    monitor.start(100);
    
    // Начальное измерение
    const initialMemory = monitor.takeMeasurement();
    
    // Выполняем операцию множество раз
    for (let i = 0; i < iterations; i++) {
      await operation();
      
      // Даем возможность сборщику мусора поработать
      if (i % 10 === 0) {
        await new Promise(resolve => setTimeout(resolve, 10));
      }
    }
    
    // Финальное измерение
    await new Promise(resolve => setTimeout(resolve, 100));
    const finalMemory = monitor.takeMeasurement();
    
    monitor.stop();
    
    // Анализируем результаты
    if (initialMemory && finalMemory) {
      const memoryGrowth = (finalMemory.used - initialMemory.used) / initialMemory.used;
      
      console.log('Тест памяти завершен:');
      console.log(`Начальное использование: ${this.formatBytes(initialMemory.used)}`);
      console.log(`Конечное использование: ${this.formatBytes(finalMemory.used)}`);
      console.log(`Рост памяти: ${(memoryGrowth * 100).toFixed(2)}%`);
      
      if (memoryGrowth > 0.1) {
        console.warn('Возможна утечка памяти!');
        return false;
      } else {
        console.log('Утечек памяти не обнаружено');
        return true;
      }
    }
    
    return null;
  }
  
  static formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
  
  // Пример операции для тестирования
  static exampleOperation() {
    return new Promise(resolve => {
      // Создаем временные объекты
      const tempArray = new Array(1000).fill(0).map(() => ({
        id: Math.random(),
        data: new Array(100).fill(Math.random())
      }));
      
      // Имитируем обработку
      const result = tempArray.map(item => ({
        ...item,
        processed: true
      }));
      
      setTimeout(() => resolve(result), 1);
    });
  }
}

// Использование теста
// MemoryTest.testForLeaks(MemoryTest.exampleOperation, 50);
```

## Лучшие практики мониторинга

1. **Регулярный мониторинг** - настройте непрерывный мониторинг в production
2. **Профилирование перед релизом** - проводите профилирование памяти перед каждым релизом
3. **Сравнение метрик** - сравнивайте метрики до и после изменений
4. **Автоматизация тестов** - создавайте автоматизированные тесты для проверки утечек
5. **Использование нескольких инструментов** - комбинируйте DevTools, Performance API и кастомные решения

Мониторинг и профилирование памяти - неотъемлемая часть разработки качественных JavaScript приложений. Регулярное использование этих инструментов помогает выявлять и устранять проблемы с производительностью на ранних этапах.

#javascript #memory-monitoring #profiling #performance #devtools