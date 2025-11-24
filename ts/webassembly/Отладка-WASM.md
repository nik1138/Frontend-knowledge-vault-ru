---
aliases: [Отладка WASM, WASM дебаггинг]
tags: [typescript, webassembly, wasm, отладка, debugging]
---

# Отладка WASM

Отладка WebAssembly (WASM) может быть сложной задачей из-за низкоуровневой природы WASM кода. Однако существуют инструменты и техники, которые помогают эффективно находить и устранять проблемы в WASM модулях.

## Основы отладки WASM

Отладка WASM отличается от традиционной отладки JavaScript. WASM код компилируется из других языков (Rust, C++, AssemblyScript), и отладка требует специфических подходов и инструментов.

### Включение отладочной информации

При компиляции WASM модулей важно включать отладочную информацию:

```bash
# Для Rust
cargo build --target wasm32-unknown-unknown

# Для AssemblyScript
asc assembly/index.ts -b build/debug.wasm -t build/debug.wat --debug

# Для C/C++ с Emscripten
emcc -g4 -O0 -s WASM=1 input.cpp -o output.js
```

## Инструменты отладки

### Использование Chrome DevTools

Chrome DevTools предоставляет поддержку отладки WASM:

1. Откройте DevTools (F12)
2. Перейдите на вкладку "Sources"
3. Найдите WASM файл в списке источников
4. Установите точки останова в WASM коде
5. Используйте стандартные инструменты отладки

```typescript
// Пример кода для отладки
async function debugWasmExample() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/debug-example.wasm')
  );
  
  const { add_with_debug, process_array } = wasmModule.instance.exports as any;
  
  // Вызов функций для отладки
  const result1 = add_with_debug(10, 20);
  console.log('Add result:', result1);
  
  const result2 = process_array([1, 2, 3, 4, 5]);
  console.log('Array result:', result2);
}
```

### Использование Firefox Developer Tools

Firefox также предоставляет инструменты для отладки WASM:

1. Откройте Developer Tools (F12)
2. Перейдите на вкладку "Debugger"
3. Найдите WASM файл в списке исходников
4. Установите точки останова

### WASM-стек и память

Для отладки важно понимать, как устроен WASM стек и память:

```rust
// Пример Rust кода с отладочными возможностями
use std::ffi::c_void;

#[no_mangle]
pub extern "C" fn debug_function(x: i32, y: i32) -> i32 {
    let result = x * y;
    
    // Отладочный вывод (работает только в отладочной сборке)
    #[cfg(debug_assertions)]
    {
        let msg = format!("Debug: {} * {} = {}", x, y, result);
        unsafe {
            log_debug_message(msg.as_ptr() as *const i8, msg.len());
        }
    }
    
    result
}

// Функция для вывода отладочных сообщений
#[no_mangle]
extern "C" fn log_debug_message(msg_ptr: *const i8, len: usize) {
    #[cfg(debug_assertions)]
    unsafe {
        let message = std::slice::from_raw_parts(msg_ptr as *const u8, len);
        let str_msg = std::str::from_utf8(message).unwrap();
        web_sys::console::log_1(&str_msg.into());
    }
}
```

## Отладка через JavaScript

Один из подходов к отладке WASM - это интеграция с JavaScript:

```typescript
// Обертка для отладки WASM вызовов
class WasmDebugger {
  private wasmExports: WebAssembly.Exports;
  private debugEnabled: boolean;

  constructor(wasmInstance: WebAssembly.Instance, debug: boolean = false) {
    this.wasmExports = wasmInstance.exports;
    this.debugEnabled = debug;
  }

  // Обертка для вызова WASM функций с логированием
  callFunction<T extends WebAssembly.ExportValue>(
    fnName: string,
    ...args: any[]
  ): T {
    if (this.debugEnabled) {
      console.group(`WASM Call: ${fnName}`);
      console.log('Arguments:', args);
    }

    const result = (this.wasmExports[fnName] as Function)(...args);

    if (this.debugEnabled) {
      console.log('Result:', result);
      console.groupEnd();
    }

    return result as T;
  }

  // Отладка памяти
  inspectMemory(offset: number, length: number): number[] {
    const memory = new Uint8Array(
      (this.wasmExports.memory as WebAssembly.Memory).buffer,
      offset,
      length
    );
    return Array.from(memory);
  }
}

// Использование отладчика
async function debugExample() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/debug-module.wasm')
  );
  
  const debuggerInstance = new WasmDebugger(wasmModule.instance, true);
  
  // Вызов с отладкой
  const result = debuggerInstance.callFunction('add', 10, 20);
  console.log('Result:', result);
  
  // Проверка памяти
  const memoryContent = debuggerInstance.inspectMemory(0, 100);
  console.log('Memory content:', memoryContent);
}
```

## Использование wasm-objdump и wasm2wat

Для анализа WASM файлов можно использовать инструменты командной строки:

```bash
# Дизассемблирование WASM в текстовый формат
wasm-objdump -x debug-module.wasm

# Конвертация в текстовый формат WASM
wabt/wasm2wat debug-module.wasm -o debug-module.wat

# Просмотр секций WASM
wasm-objdump -h debug-module.wasm
```

## Отладка AssemblyScript

AssemblyScript предоставляет специальные возможности для отладки:

```typescript
// AssemblyScript код с отладочными утверждениями
export function debugAdd(a: i32, b: i32): i32 {
  // Отладочные утверждения
  assert(a >= 0, "Parameter 'a' must be non-negative");
  assert(b >= 0, "Parameter 'b' must be non-negative");
  
  const result = a + b;
  
  // Отладочный вывод
  trace("Debug: Adding " + a.toString() + " and " + b.toString() + " = " + result.toString());
  
  return result;
}

export function debugProcessArray(arr: Int32Array): i32 {
  assert(arr.length > 0, "Array cannot be empty");
  
  let sum: i32 = 0;
  for (let i = 0; i < arr.length; i++) {
    assert(arr[i] >= 0, "Array element at index " + i.toString() + " must be non-negative");
    sum += arr[i];
    trace("Debug: Sum after element " + i.toString() + " = " + sum.toString());
  }
  
  return sum;
}
```

## Отладка Rust с помощью wasm-bindgen

Использование отладочных возможностей при работе с wasm-bindgen:

```rust
// Rust код с отладочными возможностями
use wasm_bindgen::prelude::*;
use web_sys;

#[wasm_bindgen]
pub fn debug_calculate(x: f64, y: f64) -> f64 {
    // Отладочный вывод
    web_sys::console::log_1(&format!("Calculating: {} + {}", x, y).into());
    
    let result = x + y;
    
    web_sys::console::log_1(&format!("Result: {}", result).into());
    
    result
}

#[wasm_bindgen]
pub fn debug_process_data(data: &[f64]) -> f64 {
    web_sys::console::log_1(&format!("Processing array with {} elements", data.len()).into());
    
    let sum = data.iter().sum::<f64>();
    
    web_sys::console::log_1(&format!("Sum: {}", sum).into());
    
    sum
}

// Отладка с использованием логов
#[wasm_bindgen]
pub fn debug_with_error_handling(x: i32) -> Result<f64, JsValue> {
    if x < 0 {
        let error_msg = format!("Invalid input: {}", x);
        web_sys::console::error_1(&error_msg.into());
        return Err(JsValue::from_str(&error_msg));
    }
    
    web_sys::console::log_1(&format!("Processing valid input: {}", x).into());
    
    Ok((x as f64) * 2.0)
}
```

## Тестирование и отладка

Создание тестов для WASM кода:

```typescript
// Тестирование WASM функций
class WasmTester {
  private wasmExports: WebAssembly.Exports;

  constructor(wasmInstance: WebAssembly.Instance) {
    this.wasmExports = wasmInstance.exports;
  }

  async testFunction(fnName: string, inputs: any[], expected: any) {
    try {
      const result = (this.wasmExports[fnName] as Function)(...inputs);
      
      if (result === expected) {
        console.log(`✓ Test passed: ${fnName}(${inputs.join(', ')}) = ${result}`);
        return true;
      } else {
        console.error(`✗ Test failed: ${fnName}(${inputs.join(', ')}) = ${result}, expected: ${expected}`);
        return false;
      }
    } catch (error) {
      console.error(`✗ Test error: ${fnName}(${inputs.join(', ')}) - ${error}`);
      return false;
    }
  }

  async runTests() {
    const tests = [
      { fn: 'add', inputs: [2, 3], expected: 5 },
      { fn: 'multiply', inputs: [4, 5], expected: 20 },
      { fn: 'process_array', inputs: [1, 2, 3, 4, 5], expected: 15 }
    ];

    let passed = 0;
    let total = tests.length;

    for (const test of tests) {
      const result = await this.testFunction(test.fn, test.inputs, test.expected);
      if (result) passed++;
    }

    console.log(`Tests passed: ${passed}/${total}`);
    return { passed, total };
  }
}

// Использование тестера
async function runDebugTests() {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/test-module.wasm')
  );
  
  const tester = new WasmTester(wasmModule.instance);
  await tester.runTests();
}
```

## Обработка ошибок и исключений

Правильная обработка ошибок в WASM коде:

```typescript
// Обертка для безопасного вызова WASM функций
class SafeWasmInvoker {
  private wasmExports: WebAssembly.Exports;

  constructor(wasmInstance: WebAssembly.Instance) {
    this.wasmExports = wasmInstance.exports;
  }

  safeCall<T extends WebAssembly.ExportValue>(fnName: string, ...args: any[]): T | null {
    try {
      return (this.wasmExports[fnName] as Function)(...args) as T;
    } catch (error) {
      console.error(`WASM function ${fnName} failed:`, error);
      return null;
    }
  }

  // Вызов с проверкой границ памяти
  safeMemoryAccess(offset: number, length: number): Uint8Array | null {
    try {
      const memory = this.wasmExports.memory as WebAssembly.Memory;
      if (offset + length > memory.buffer.byteLength) {
        throw new Error(`Memory access out of bounds: ${offset} + ${length} > ${memory.buffer.byteLength}`);
      }
      
      return new Uint8Array(memory.buffer, offset, length);
    } catch (error) {
      console.error('Memory access error:', error);
      return null;
    }
  }
}
```

## Использование отладочных флагов

При разработке используйте отладочные флаги и опции:

```rust
// Использование условной компиляции для отладки
#[cfg(debug_assertions)]
macro_rules! debug_print {
    ($($arg:tt)*) => {
        web_sys::console::log_1(&format!($($arg)*).into());
    }
}

#[cfg(not(debug_assertions))]
macro_rules! debug_print {
    ($($arg:tt)*) => {};
}

#[wasm_bindgen]
pub fn calculate_with_debug(x: i32, y: i32) -> i32 {
    debug_print!("Calculating: {} + {}", x, y);
    
    let result = x + y;
    
    debug_print!("Result: {}", result);
    
    result
}
```

## Best Practices

1. **Используйте отладочные сборки при разработке** - они содержат отладочную информацию
2. **Включайте отладочные утверждения** - для раннего выявления ошибок
3. **Используйте логирование в консоль** - для отслеживания выполнения
4. **Создавайте модульные тесты** - для проверки функций WASM
5. **Используйте инструменты вроде Chrome DevTools** - для визуальной отладки
6. **Проверяйте границы памяти** - для предотвращения ошибок доступа
7. **Изолируйте WASM код в тестах** - для более точной диагностики проблем

## Заключение

Отладка WebAssembly требует понимания специфики взаимодействия между JavaScript и WASM, а также использования специализированных инструментов. Эффективная отладка позволяет находить и устранять проблемы на ранних стадиях разработки, что критически важно для надежности приложений.

См. также:
- [[AssemblyScript]]
- [[Rust-и-TypeScript]]
- [[Взаимодействие-с-WASM]]
- [[Оптимизация-производительности]]