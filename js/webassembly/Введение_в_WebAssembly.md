# Введение в WebAssembly

WebAssembly (сокращенно WASM) - это бинарный формат инструкций для стековой виртуальной машины. Он разработан как портативная цель компиляции для высокоуровневых языков, позволяющая разработчикам создавать приложения с высокой производительностью, которые могут работать в веб-браузерах и других средах.

## Основные понятия

### Что такое WebAssembly

WebAssembly - это:
- **Бинарный формат** - компактное представление кода
- **Виртуальная машина** - безопасная среда выполнения
- **Модульная система** - поддержка импорта/экспорта функций
- **Песочница** - ограниченный доступ к системным ресурсам
- **Многоплатформенность** - работает на разных архитектурах

```javascript
// Простой пример загрузки WebAssembly модуля
async function loadWebAssembly() {
    try {
        // Загрузка .wasm файла
        const response = await fetch('simple.wasm');
        const bytes = await response.arrayBuffer();
        
        // Компиляция и инстанцирование модуля
        const wasmModule = await WebAssembly.instantiate(bytes);
        
        // Использование экспортированных функций
        const result = wasmModule.instance.exports.add(5, 3);
        console.log('Результат сложения:', result); // 8
        
        return wasmModule;
    } catch (error) {
        console.error('Ошибка загрузки WebAssembly:', error);
    }
}
```

### Сравнение с JavaScript

| Характеристика | JavaScript | WebAssembly |
|----------------|------------|-------------|
| Скорость выполнения | Переменная, зависит от JIT | Высокая, предсказуемая |
| Типизация | Динамическая | Статическая |
| Доступ к DOM | Прямой | Ограниченный (через JS) |
| Размер бинарного кода | Зависит от оптимизации | Компактный |
| Время загрузки | Быстрое | Быстрое (бинарный формат) |
| Отладка | Хорошая поддержка | Ограниченная, но улучшается |

## Основные возможности

### Типы данных в WebAssembly

```javascript
// WebAssembly поддерживает ограниченный набор типов:
// - i32: 32-битное целое число со знаком
// - i64: 64-битное целое число со знаком
// - f32: 32-битное число с плавающей точкой
// - f64: 64-битное число с плавающей точкой

// Пример использования различных типов
async function demonstrateTypes() {
    const wasmCode = new Uint8Array([
        // WASM бинарный код (упрощенный пример)
        0x00, 0x61, 0x73, 0x6d, // WASM магическое число
        0x01, 0x00, 0x00, 0x00, // Версия
        // ... остальная часть WASM бинарного кода
    ]);
    
    const wasmModule = await WebAssembly.instantiate(wasmCode);
    const { exports } = wasmModule.instance;
    
    // Использование различных типов
    console.log('i32:', exports.getInt32Value());
    console.log('i64:', exports.getInt64Value());
    console.log('f32:', exports.getFloat32Value());
    console.log('f64:', exports.getFloat64Value());
}
```

### Память в WebAssembly

```javascript
// WebAssembly использует линейную память (ArrayBuffer)
class WasmMemoryManager {
    constructor(wasmInstance) {
        this.memory = wasmInstance.exports.memory;
        this.int32View = new Int32Array(this.memory.buffer);
        this.float64View = new Float64Array(this.memory.buffer);
    }
    
    // Чтение из памяти
    readInt32(offset) {
        return this.int32View[offset / 4];
    }
    
    // Запись в память
    writeInt32(offset, value) {
        this.int32View[offset / 4] = value;
    }
    
    // Чтение строки из памяти
    readString(offset) {
        const uint8View = new Uint8Array(this.memory.buffer);
        let str = '';
        for (let i = offset; uint8View[i] !== 0; i++) {
            str += String.fromCharCode(uint8View[i]);
        }
        return str;
    }
    
    // Запись строки в память
    writeString(offset, str) {
        const uint8View = new Uint8Array(this.memory.buffer);
        for (let i = 0; i < str.length; i++) {
            uint8View[offset + i] = str.charCodeAt(i);
        }
        uint8View[offset + str.length] = 0; // null-терминатор
    }
    
    // Выделение памяти (предполагается, что в WASM есть функция malloc)
    allocate(size) {
        return this.wasmInstance.exports.malloc(size);
    }
    
    // Освобождение памяти (предполагается, что в WASM есть функция free)
    deallocate(ptr) {
        this.wasmInstance.exports.free(ptr);
    }
}
```

## Загрузка и инстанцирование

### Методы загрузки WebAssembly

```javascript
// 1. Загрузка из бинарного файла
async function loadFromWasmFile() {
    const response = await fetch('module.wasm');
    const bytes = await response.arrayBuffer();
    return await WebAssembly.instantiate(bytes);
}

// 2. Загрузка с компиляцией отдельно
async function loadWithSeparateCompilation() {
    const response = await fetch('module.wasm');
    const bytes = await response.arrayBuffer();
    
    // Компиляция
    const wasmModule = await WebAssembly.compile(bytes);
    
    // Инстанцирование
    const wasmInstance = await WebAssembly.instantiate(wasmModule);
    
    return wasmInstance;
}

// 3. Загрузка с импортируемыми функциями
async function loadWithImports() {
    const response = await fetch('module.wasm');
    const bytes = await response.arrayBuffer();
    
    // Определение импортируемых функций
    const imports = {
        env: {
            // Функции, которые будут доступны WASM модулю
            js_print: (value) => console.log('WASM print:', value),
            js_random: () => Math.random()
        }
    };
    
    const wasmModule = await WebAssembly.instantiate(bytes, imports);
    return wasmModule;
}

// 4. Загрузка с общим ArrayBuffer
async function loadWithSharedMemory() {
    // Создание общей памяти
    const memory = new WebAssembly.Memory({ initial: 256, maximum: 512 });
    
    const imports = {
        env: { memory }
    };
    
    const response = await fetch('module.wasm');
    const bytes = await response.arrayBuffer();
    
    const wasmModule = await WebAssembly.instantiate(bytes, imports);
    return wasmModule;
}
```

### Обработка ошибок

```javascript
// Класс для безопасной загрузки WebAssembly
class SafeWasmLoader {
    static async loadWasm(url, imports = {}) {
        try {
            // Проверка поддержки WebAssembly
            if (!this.isSupported()) {
                throw new Error('WebAssembly не поддерживается в этом браузере');
            }
            
            // Загрузка WASM файла
            const response = await fetch(url);
            
            if (!response.ok) {
                throw new Error(`Ошибка загрузки WASM файла: ${response.status} ${response.statusText}`);
            }
            
            // Проверка типа контента
            const contentType = response.headers.get('Content-Type');
            if (contentType && !contentType.startsWith('application/wasm')) {
                console.warn('Непредпочтительный Content-Type для WASM файла:', contentType);
            }
            
            const bytes = await response.arrayBuffer();
            
            // Валидация WASM модуля
            if (!WebAssembly.validate(bytes)) {
                throw new Error('Невалидный WASM модуль');
            }
            
            // Компиляция и инстанцирование
            const wasmModule = await WebAssembly.instantiate(bytes, imports);
            
            return wasmModule;
        } catch (error) {
            console.error('Ошибка загрузки WebAssembly:', error);
            throw error;
        }
    }
    
    static isSupported() {
        return typeof WebAssembly !== 'undefined' && 
               typeof WebAssembly.instantiate === 'function';
    }
    
    static async loadWasmWithFallback(url, fallbackUrl, imports = {}) {
        try {
            return await this.loadWasm(url, imports);
        } catch (primaryError) {
            console.warn('Ошибка загрузки основного WASM файла, пробуем fallback:', primaryError);
            
            try {
                return await this.loadWasm(fallbackUrl, imports);
            } catch (fallbackError) {
                console.error('Оба WASM файла недоступны:', fallbackError);
                throw new Error(`Не удалось загрузить WASM модуль. Основной: ${primaryError.message}, Fallback: ${fallbackError.message}`);
            }
        }
    }
}

// Использование
async function initializeWasm() {
    try {
        const wasmModule = await SafeWasmLoader.loadWasm('./calculator.wasm');
        console.log('WASM модуль успешно загружен');
        return wasmModule;
    } catch (error) {
        console.error('Не удалось инициализировать WASM:', error);
        // Резервная реализация на JavaScript
        return null;
    }
}
```

## Примеры использования

### Простой калькулятор на WebAssembly

```javascript
// Представим, что у нас есть WASM модуль с простыми математическими функциями
class WasmCalculator {
    constructor(wasmModule) {
        this.wasm = wasmModule.instance.exports;
    }
    
    add(a, b) {
        return this.wasm.add(a, b);
    }
    
    subtract(a, b) {
        return this.wasm.subtract(a, b);
    }
    
    multiply(a, b) {
        return this.wasm.multiply(a, b);
    }
    
    divide(a, b) {
        if (b === 0) {
            throw new Error('Деление на ноль');
        }
        return this.wasm.divide(a, b);
    }
    
    // Работа с массивами
    sumArray(array) {
        // Выделение памяти для массива
        const ptr = this.wasm.malloc(array.length * 4); // 4 байта на i32
        const memory = new Int32Array(this.wasm.memory.buffer, ptr, array.length);
        
        // Копирование данных в WASM память
        memory.set(array);
        
        // Вызов функции
        const result = this.wasm.sum_array(ptr, array.length);
        
        // Освобождение памяти
        this.wasm.free(ptr);
        
        return result;
    }
    
    // Сортировка массива
    sortArray(array) {
        const ptr = this.wasm.malloc(array.length * 4);
        const memory = new Int32Array(this.wasm.memory.buffer, ptr, array.length);
        
        memory.set(array);
        this.wasm.sort_array(ptr, array.length);
        
        const result = Array.from(memory);
        
        this.wasm.free(ptr);
        return result;
    }
}

// Использование
async function useCalculator() {
    const wasmModule = await SafeWasmLoader.loadWasm('./calculator.wasm');
    const calc = new WasmCalculator(wasmModule);
    
    console.log('5 + 3 =', calc.add(5, 3));
    console.log('10 - 4 =', calc.subtract(10, 4));
    console.log('6 * 7 =', calc.multiply(6, 7));
    console.log('15 / 3 =', calc.divide(15, 3));
    
    const numbers = [5, 2, 8, 1, 9, 3];
    console.log('Сумма массива:', calc.sumArray(numbers));
    console.log('Отсортированный массив:', calc.sortArray(numbers));
}
```

### Работа с изображениями

```javascript
// Класс для обработки изображений с помощью WASM
class WasmImageProcessor {
    constructor(wasmModule) {
        this.wasm = wasmModule.instance.exports;
        this.memoryManager = new WasmMemoryManager(wasmModule.instance);
    }
    
    // Применение фильтра к изображению
    applyFilter(imageData, filterType) {
        const { width, height, data } = imageData;
        const pixelCount = width * height * 4; // 4 компонента на пиксель (RGBA)
        
        // Выделение памяти для изображения
        const imagePtr = this.wasm.malloc(pixelCount);
        const imageMemory = new Uint8ClampedArray(this.wasm.memory.buffer, imagePtr, pixelCount);
        
        // Копирование данных изображения
        imageMemory.set(data);
        
        // Применение фильтра
        switch (filterType) {
            case 'grayscale':
                this.wasm.apply_grayscale_filter(imagePtr, width, height);
                break;
            case 'blur':
                this.wasm.apply_blur_filter(imagePtr, width, height);
                break;
            case 'edge':
                this.wasm.apply_edge_filter(imagePtr, width, height);
                break;
            default:
                throw new Error(`Неизвестный тип фильтра: ${filterType}`);
        }
        
        // Создание нового imageData с обработанными данными
        const result = new ImageData(
            new Uint8ClampedArray(imageMemory.slice(0, pixelCount)),
            width,
            height
        );
        
        // Освобождение памяти
        this.wasm.free(imagePtr);
        
        return result;
    }
    
    // Изменение размера изображения
    resizeImage(imageData, newWidth, newHeight) {
        const { width, height, data } = imageData;
        const oldPixelCount = width * height * 4;
        const newPixelCount = newWidth * newHeight * 4;
        
        // Выделение памяти для старого и нового изображения
        const oldImagePtr = this.wasm.malloc(oldPixelCount);
        const newImagePtr = this.wasm.malloc(newPixelCount);
        
        const oldImageMemory = new Uint8ClampedArray(this.wasm.memory.buffer, oldImagePtr, oldPixelCount);
        const newImageMemory = new Uint8ClampedArray(this.wasm.memory.buffer, newImagePtr, newPixelCount);
        
        // Копирование данных
        oldImageMemory.set(data);
        
        // Изменение размера
        this.wasm.resize_image(oldImagePtr, width, height, newImagePtr, newWidth, newHeight);
        
        // Создание результата
        const result = new ImageData(
            new Uint8ClampedArray(newImageMemory.slice(0, newPixelCount)),
            newWidth,
            newHeight
        );
        
        // Освобождение памяти
        this.wasm.free(oldImagePtr);
        this.wasm.free(newImagePtr);
        
        return result;
    }
}

// Использование
async function processImage() {
    const wasmModule = await SafeWasmLoader.loadWasm('./image-processor.wasm');
    const processor = new WasmImageProcessor(wasmModule);
    
    // Получение ImageData (например, из canvas)
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    
    // Применение фильтра
    const filteredData = processor.applyFilter(imageData, 'grayscale');
    
    // Рисование результата
    ctx.putImageData(filteredData, 0, 0);
}
```

## Производительность и оптимизация

### Сравнение производительности

```javascript
// Класс для сравнения производительности JS и WASM
class PerformanceBenchmark {
    static async runBenchmark() {
        const wasmModule = await SafeWasmLoader.loadWasm('./benchmark.wasm');
        const wasmExports = wasmModule.instance.exports;
        
        // Тест вычислений
        const arraySize = 1000000;
        const testArray = Array.from({ length: arraySize }, () => Math.random() * 100);
        
        // JavaScript версия
        const jsStart = performance.now();
        const jsResult = this.jsSum(testArray);
        const jsEnd = performance.now();
        const jsTime = jsEnd - jsStart;
        
        // WASM версия
        const wasmStart = performance.now();
        const wasmResult = this.wasmSum(wasmExports, testArray);
        const wasmEnd = performance.now();
        const wasmTime = wasmEnd - wasmStart;
        
        console.log(`JS время: ${jsTime.toFixed(2)}ms, результат: ${jsResult}`);
        console.log(`WASM время: ${wasmTime.toFixed(2)}ms, результат: ${wasmResult}`);
        console.log(`Ускорение: ${(jsTime / wasmTime).toFixed(2)}x`);
    }
    
    static jsSum(array) {
        let sum = 0;
        for (let i = 0; i < array.length; i++) {
            sum += array[i] * array[i]; // квадраты чисел
        }
        return sum;
    }
    
    static wasmSum(wasmExports, array) {
        const ptr = wasmExports.malloc(array.length * 8); // 8 байт на f64
        const memory = new Float64Array(wasmExports.memory.buffer, ptr, array.length);
        
        memory.set(array);
        const result = wasmExports.sum_squares(ptr, array.length);
        
        wasmExports.free(ptr);
        return result;
    }
    
    // Тест сортировки
    static async runSortBenchmark() {
        const wasmModule = await SafeWasmLoader.loadWasm('./benchmark.wasm');
        const wasmExports = wasmModule.instance.exports;
        
        const arraySize = 100000;
        let testArray = Array.from({ length: arraySize }, () => Math.random() * 1000);
        
        // JavaScript сортировка
        const jsArray = [...testArray];
        const jsStart = performance.now();
        jsArray.sort((a, b) => a - b);
        const jsEnd = performance.now();
        const jsTime = jsEnd - jsStart;
        
        // WASM сортировка
        const wasmArray = [...testArray];
        const wasmStart = performance.now();
        this.wasmSort(wasmExports, wasmArray);
        const wasmEnd = performance.now();
        const wasmTime = wasmEnd - wasmStart;
        
        console.log(`JS сортировка: ${jsTime.toFixed(2)}ms`);
        console.log(`WASM сортировка: ${wasmTime.toFixed(2)}ms`);
        console.log(`Ускорение сортировки: ${(jsTime / wasmTime).toFixed(2)}x`);
    }
    
    static wasmSort(wasmExports, array) {
        const ptr = wasmExports.malloc(array.length * 8);
        const memory = new Float64Array(wasmExports.memory.buffer, ptr, array.length);
        
        memory.set(array);
        wasmExports.sort_array(ptr, array.length);
        
        // Копирование отсортированных данных обратно в массив
        for (let i = 0; i < array.length; i++) {
            array[i] = memory[i];
        }
        
        wasmExports.free(ptr);
    }
}

// Запуск бенчмарков
// PerformanceBenchmark.runBenchmark();
// PerformanceBenchmark.runSortBenchmark();
```

### Оптимизация передачи данных

```javascript
// Класс для эффективной передачи данных в WASM
class EfficientDataTransfer {
    constructor(wasmExports) {
        this.wasm = wasmExports;
        this.sharedBuffers = new Map(); // кэш выделенных буферов
    }
    
    // Выделение буфера с повторным использованием
    allocateTypedArray(size, type = 'f64') {
        const key = `${type}-${size}`;
        let buffer = this.sharedBuffers.get(key);
        
        if (!buffer) {
            // Создание нового буфера
            const byteSize = this.getTypeSize(type) * size;
            const ptr = this.wasm.malloc(byteSize);
            
            buffer = {
                ptr,
                size,
                type,
                array: this.createTypedArray(this.wasm.memory.buffer, ptr, size, type),
                used: false
            };
            
            this.sharedBuffers.set(key, buffer);
        }
        
        buffer.used = true;
        return buffer.array;
    }
    
    // Освобождение буфера
    releaseTypedArray(array) {
        for (const [key, buffer] of this.sharedBuffers) {
            if (buffer.array === array) {
                buffer.used = false;
                // Можно добавить логику очистки неиспользуемых буферов
                break;
            }
        }
    }
    
    // Очистка неиспользуемых буферов
    cleanupUnusedBuffers() {
        for (const [key, buffer] of this.sharedBuffers) {
            if (!buffer.used) {
                this.wasm.free(buffer.ptr);
                this.sharedBuffers.delete(key);
            }
        }
    }
    
    getTypeSize(type) {
        switch (type) {
            case 'i32': return 4;
            case 'f64': return 8;
            case 'f32': return 4;
            default: return 8; // по умолчанию f64
        }
    }
    
    createTypedArray(buffer, offset, size, type) {
        switch (type) {
            case 'i32':
                return new Int32Array(buffer, offset, size);
            case 'f64':
                return new Float64Array(buffer, offset, size);
            case 'f32':
                return new Float32Array(buffer, offset, size);
            default:
                return new Float64Array(buffer, offset, size);
        }
    }
    
    // Быстрая передача больших массивов
    async fastArrayOperation(array, operation) {
        const typedArray = this.allocateTypedArray(array.length, 'f64');
        typedArray.set(array);
        
        const result = await operation(typedArray, array.length);
        
        this.releaseTypedArray(typedArray);
        return result;
    }
}

// Использование
async function efficientProcessing() {
    const wasmModule = await SafeWasmLoader.loadWasm('./processor.wasm');
    const transfer = new EfficientDataTransfer(wasmModule.instance.exports);
    
    const largeArray = new Float64Array(1000000);
    for (let i = 0; i < largeArray.length; i++) {
        largeArray[i] = Math.random();
    }
    
    // Быстрая обработка большого массива
    const result = await transfer.fastArrayOperation(
        largeArray, 
        (typedArray, length) => {
            return wasmModule.instance.exports.process_array(typedArray.byteOffset, length);
        }
    );
    
    console.log('Результат обработки:', result);
}
```

## Современные возможности

### WebAssembly System Interface (WASI)

```javascript
// Использование WASI (WebAssembly System Interface)
async function useWASI() {
    // WASI позволяет WASM модулям взаимодействовать с системой
    // как с файловой системой, сетью и т.д.
    
    const wasi = new WebAssembly.WASI({
        args: [], // аргументы командной строки
        env: {}, // переменные окружения
        preopens: {} // предварительно открытые директории
    });
    
    const response = await fetch('module.wasi.wasm');
    const bytes = await response.arrayBuffer();
    
    const wasmModule = await WebAssembly.compile(bytes);
    const wasmInstance = await WebAssembly.instantiate(wasmModule, {
        wasi_snapshot_preview1: wasi.wasiImport
    });
    
    // Запуск WASI экземпляра
    wasi.start(wasmInstance);
    
    return wasmInstance;
}
```

### WebAssembly Threads

```javascript
// Использование WebAssembly с потоками (если поддерживается)
async function useWasmThreads() {
    // Проверка поддержки SharedArrayBuffer
    if (typeof SharedArrayBuffer === 'undefined') {
        console.warn('SharedArrayBuffer не поддерживается, многопоточность недоступна');
        return null;
    }
    
    const response = await fetch('threaded-module.wasm');
    const bytes = await response.arrayBuffer();
    
    // Создание общей памяти для многопоточности
    const memory = new WebAssembly.Memory({ 
        initial: 256, 
        maximum: 1024,
        shared: true // Важно для многопоточности
    });
    
    const wasmModule = await WebAssembly.instantiate(bytes, {
        env: { memory }
    });
    
    return wasmModule;
}
```

WebAssembly открывает новые возможности для создания высокопроизводительных веб-приложений. При правильном использовании он может значительно улучшить производительность вычислительно сложных задач в браузере.

#javascript #webassembly #wasm #performance #frontend #web-development #native-performance