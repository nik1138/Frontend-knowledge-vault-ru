---
aliases: [Взаимодействие с WASM, WASM интерфейс]
tags: [typescript, webassembly, wasm, интерфейс, интеграция]
---

# Взаимодействие с WASM

Взаимодействие между TypeScript и WebAssembly (WASM) требует понимания специфики передачи данных между этими двумя средами. WASM поддерживает ограниченный набор базовых типов, поэтому важно знать, как правильно передавать сложные структуры данных.

## Основы взаимодействия

WebAssembly поддерживает только 4 базовых типа данных:
- `i32` - 32-битное целое число
- `i64` - 64-битное целое число
- `f32` - 32-битное число с плавающей точкой
- `f64` - 64-битное число с плавающей точкой

Для передачи других типов данных, таких как строки или объекты, используются специальные подходы.

## Простые типы данных

Прямая передача примитивных типов между TypeScript и WASM:

```typescript
// Пример TypeScript кода для вызова WASM функций
async function loadWasm() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/simple-calculations.wasm')
  );
  
  const { add, multiply, calculate } = wasmModule.instance.exports;
  
  // Вызов функций с простыми типами
  const sum = add(10, 20); // i32 -> i32
  const product = multiply(5.5, 4.0); // f64 -> f64
  const result = calculate(15, 3.14); // i32, f64 -> f64
  
  console.log({ sum, product, result });
}

// Пример WASM функции (на Rust)
/*
#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[no_mangle]
pub extern "C" fn multiply(a: f64, b: f64) -> f64 {
    a * b
}

#[no_mangle]
pub extern "C" fn calculate(x: i32, y: f64) -> f64 {
    (x as f64) * y
}
*/
```

## Работа с памятью

Для передачи массивов и строк используется общая память между JavaScript и WASM:

```typescript
class WasmMemoryManager {
  private wasmExports: WebAssembly.Exports;
  private memory: WebAssembly.Memory;

  constructor(wasmInstance: WebAssembly.Instance) {
    this.wasmExports = wasmInstance.exports as any;
    this.memory = this.wasmExports.memory as WebAssembly.Memory;
  }

  // Чтение строки из WASM памяти
  readString(ptr: number): string {
    const memory = new Uint8Array(this.memory.buffer);
    let end = ptr;
    while (memory[end] !== 0) end++;
    return new TextDecoder().decode(memory.subarray(ptr, end));
  }

  // Запись строки в WASM память
  writeString(str: string): number {
    const bytes = new TextEncoder().encode(str + '\0');
    const ptr = this.wasmExports.allocate_memory(bytes.length) as number;
    const memory = new Uint8Array(this.memory.buffer);
    memory.set(bytes, ptr);
    return ptr;
  }

  // Работа с массивами
  readFloatArray(ptr: number, length: number): Float64Array {
    return new Float64Array(this.memory.buffer, ptr, length);
  }

  writeFloatArray(data: number[]): number {
    const float64Array = new Float64Array(data);
    const ptr = this.wasmExports.allocate_memory(float64Array.byteLength) as number;
    const memory = new Float64Array(this.memory.buffer);
    memory.set(float64Array, ptr / 8); // 8 байт на f64
    return ptr;
  }
}
```

## Передача строк

Пример передачи строк между TypeScript и WASM:

```typescript
// TypeScript
async function stringExample() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/string-operations.wasm')
  );
  
  const { process_string, get_result } = wasmModule.instance.exports as any;
  const memoryManager = new WasmMemoryManager(wasmModule.instance);
  
  // Запись строки в WASM память
  const inputStr = "Hello, WebAssembly!";
  const inputPtr = memoryManager.writeString(inputStr);
  
  // Вызов WASM функции
  const resultPtr = process_string(inputPtr);
  
  // Чтение результата
  const result = memoryManager.readString(resultPtr);
  console.log(result);
}

// Пример WASM функции (на Rust)
/*
use std::ffi::{CStr, CString};

#[no_mangle]
pub extern "C" fn process_string(input_ptr: *const i8) -> *mut i8 {
    unsafe {
        let c_str = CStr::from_ptr(input_ptr);
        let input = c_str.to_str().unwrap();
        
        let processed = format!("Processed: {}", input.to_uppercase());
        let output = CString::new(processed).unwrap();
        output.into_raw()
    }
}

#[no_mangle]
pub extern "C" fn allocate_memory(size: usize) -> *mut u8 {
    let mut buf = Vec::with_capacity(size);
    let ptr = buf.as_mut_ptr();
    std::mem::forget(buf);
    ptr
}
*/
```

## Работа с массивами

Передача и обработка массивов данных:

```typescript
// TypeScript
async function arrayExample() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/array-processing.wasm')
  );
  
  const { process_array, get_array_length } = wasmModule.instance.exports as any;
  const memory = new WebAssembly.Memory({ initial: 256 });
  
  // Подготовка данных
  const inputArray = [1.0, 2.0, 3.0, 4.0, 5.0];
  const float64Array = new Float64Array(inputArray);
  
  // Выделение памяти в WASM
  const ptr = (wasmModule.instance.exports.allocate_memory as Function)(float64Array.byteLength);
  
  // Копирование данных в WASM память
  const wasmMemory = new Float64Array(memory.buffer);
  wasmMemory.set(float64Array, ptr / 8);
  
  // Вызов обработки
  const resultPtr = process_array(ptr, inputArray.length);
  const resultLength = get_array_length(resultPtr);
  
  // Чтение результата
  const resultArray = new Float64Array(memory.buffer, resultPtr, resultLength);
  console.log('Result:', Array.from(resultArray));
}

// Пример WASM функции (на Rust)
/*
#[no_mangle]
pub extern "C" fn process_array(ptr: *const f64, len: usize) -> *mut f64 {
    let slice = unsafe { std::slice::from_raw_parts(ptr, len) };
    let processed: Vec<f64> = slice.iter().map(|&x| x * 2.0 + 1.0).collect();
    
    let ptr = processed.as_mut_ptr();
    std::mem::forget(processed);
    ptr
}

#[no_mangle]
pub extern "C" fn get_array_length(ptr: *const f64) -> usize {
    // В реальном приложении длина массива должна передаваться отдельно
    // или храниться в структуре с метаданными
    0 // Заглушка
}
*/
```

## Использование wasm-bindgen

Для упрощения взаимодействия рекомендуется использовать wasm-bindgen:

```rust
// Rust код с использованием wasm-bindgen
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[wasm_bindgen]
pub fn calculate_fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2),
    }
}

#[wasm_bindgen]
pub fn process_numbers(numbers: &[f64]) -> Vec<f64> {
    numbers.iter()
        .map(|&x| x * 2.0 + 1.0)
        .collect()
}

#[wasm_bindgen]
pub struct Calculator {
    value: f64,
}

#[wasm_bindgen]
impl Calculator {
    #[wasm_bindgen(constructor)]
    pub fn new(initial_value: f64) -> Calculator {
        Calculator { value: initial_value }
    }
    
    #[wasm_bindgen]
    pub fn add(&mut self, value: f64) -> f64 {
        self.value += value;
        self.value
    }
    
    #[wasm_bindgen]
    pub fn get_value(&self) -> f64 {
        self.value
    }
}
```

```typescript
// TypeScript код для использования с wasm-bindgen
import init, { greet, calculate_fibonacci, process_numbers, Calculator } from './pkg/wasm_demo.js';

async function wasmBindgenExample() {
  const wasm = await init();
  
  // Простые функции
  console.log(greet('TypeScript'));
  console.log('Fibonacci(10):', calculate_fibonacci(10));
  
  // Работа с массивами
  const numbers = [1.0, 2.0, 3.0, 4.0, 5.0];
  const processed = process_numbers(new Float64Array(numbers));
  console.log('Processed numbers:', Array.from(processed));
  
  // Работа с классами
  const calc = new Calculator(10.0);
  console.log('Initial value:', calc.get_value());
  console.log('After adding 5:', calc.add(5.0));
  console.log('Final value:', calc.get_value());
}
```

## Обработка ошибок

Правильная обработка ошибок при взаимодействии с WASM:

```typescript
// TypeScript
async function errorHandling() {
  try {
    const wasmModule = await WebAssembly.instantiateStreaming(
      fetch('/error-handling.wasm')
    );
    
    const { divide } = wasmModule.instance.exports as any;
    
    // Проверка на ошибки
    const result = divide(10, 0);
    if (result === Infinity || isNaN(result)) {
      throw new Error('Division by zero detected');
    }
    
    console.log('Result:', result);
  } catch (error) {
    console.error('WASM Error:', error);
  }
}

// Пример обработки ошибок в Rust
/*
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn divide(a: f64, b: f64) -> Result<f64, JsValue> {
    if b == 0.0 {
        Err(JsValue::from_str("Division by zero"))
    } else {
        Ok(a / b)
    }
}
*/
```

## Асинхронная работа

Для асинхронной работы с WASM:

```typescript
// TypeScript
class AsyncWasmProcessor {
  private wasmModule: WebAssembly.Instance | null = null;
  
  async initialize() {
    this.wasmModule = (await WebAssembly.instantiateStreaming(
      fetch('/async-processor.wasm')
    )).instance;
  }
  
  async processLargeData(data: number[]): Promise<number[]> {
    if (!this.wasmModule) {
      throw new Error('WASM module not initialized');
    }
    
    // Асинхронная обработка данных
    return new Promise((resolve, reject) => {
      try {
        const { process_data } = this.wasmModule!.exports as any;
        const result = process_data(new Float64Array(data));
        resolve(Array.from(result));
      } catch (error) {
        reject(error);
      }
    });
  }
}

// Использование
async function asyncExample() {
  const processor = new AsyncWasmProcessor();
  await processor.initialize();
  
  const largeData = Array.from({ length: 1000000 }, (_, i) => i * 0.1);
  const result = await processor.processLargeData(largeData);
  console.log(`Processed ${result.length} items`);
}
```

## Best Practices

1. **Минимизируйте количество вызовов между JS и WASM** - каждый вызов имеет накладные расходы
2. **Используйте TypedArray для передачи массивов** - они обеспечивают эффективную передачу данных
3. **Рассмотрите использование wasm-bindgen** - он упрощает взаимодействие и обработку типов
4. **Обрабатывайте ошибки должным образом** - WASM не обеспечивает автоматическую обработку исключений
5. **Оптимизируйте передачу данных** - используйте буферы и избегайте ненужных копирований

## Заключение

Эффективное взаимодействие между TypeScript и WASM требует понимания ограничений WebAssembly и правильной организации передачи данных. Использование инструментов вроде wasm-bindgen может значительно упростить разработку и уменьшить количество ошибок.

См. также:
- [[AssemblyScript]]
- [[Rust-и-TypeScript]]
- [[Оптимизация-производительности]]
- [[Отладка-WASM]]