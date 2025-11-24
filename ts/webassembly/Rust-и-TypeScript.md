---
aliases: [Rust и TypeScript, Rust + TypeScript]
tags: [typescript, webassembly, rust, wasm]
---

# Rust и TypeScript

Интеграция Rust и TypeScript через WebAssembly открывает возможности для создания высокопроизводительных веб-приложений с использованием системного языка Rust и удобства разработки TypeScript. Этот подход позволяет использовать сильные стороны обоих языков.

## Основы интеграции

Rust может компилироваться в WebAssembly, что позволяет использовать его вычислительную мощность в веб-браузерах. TypeScript, в свою очередь, обеспечивает удобную типизацию и разработку клиентской части приложения.

### Установка необходимых инструментов

Для начала работы с Rust и WebAssembly установите необходимые компоненты:

```bash
# Установка Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# Установка WebAssembly target
rustup target add wasm32-unknown-unknown

# Установка wasm-pack (для интеграции с npm)
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

## Создание Rust библиотеки

Создайте проект Rust с поддержкой WebAssembly:

```toml
# Cargo.toml
[package]
name = "rust-wasm-demo"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

Пример Rust кода для WebAssembly:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    log(&format!("Hello, {}!", name));
}

#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

// Работа с массивами
#[wasm_bindgen]
pub fn process_array(arr: &[f64]) -> f64 {
    arr.iter().sum()
}

// Возвращение массива
#[wasm_bindgen]
pub fn create_array(size: usize) -> Vec<u8> {
    (0..size).map(|i| i as u8).collect()
}
```

## Компиляция в WebAssembly

Компиляция Rust кода в WebAssembly с помощью wasm-pack:

```bash
wasm-pack build --target web
```

Это создаст папку `pkg` с файлами, готовыми для использования в веб-проекте.

## Интеграция с TypeScript

Подключение WebAssembly модуля в TypeScript проекте:

```typescript
// Импорт WASM модуля
import init, { greet, fibonacci, process_array, create_array } from './pkg/rust_wasm_demo.js';

async function runWasm() {
  // Инициализация WASM модуля
  await init();
  
  // Использование функций из Rust
  greet('TypeScript developer');
  
  const fibResult = fibonacci(10);
  console.log(`Fibonacci(10) = ${fibResult}`);
  
  const numbers = [1.5, 2.5, 3.0, 4.0];
  const sum = process_array(new Float64Array(numbers));
  console.log(`Sum of array: ${sum}`);
  
  const newArray = create_array(5);
  console.log(`Created array: ${Array.from(newArray)}`);
}

runWasm();
```

## Работа с типами данных

При передаче данных между TypeScript и Rust важно понимать, как типы преобразуются:

| TypeScript | Rust | wasm-bindgen |
|------------|------|--------------|
| number | f64, f32 | автоматически |
| number | i32, u32 | автоматически |
| boolean | bool | автоматически |
| string | &str, String | автоматически |
| Uint8Array | &[u8], Vec<u8> | автоматически |
| Array<number> | &[f64], Vec<f64> | через TypedArray |

Пример работы с более сложными типами:

```rust
use wasm_bindgen::prelude::*;
use js_sys::Array;

#[wasm_bindgen]
pub fn manipulate_array(input: &[f64]) -> Vec<f64> {
    input.iter()
        .map(|&x| x * 2.0 + 1.0)
        .collect()
}

#[wasm_bindgen]
pub fn get_array_slice() -> js_sys::Float64Array {
    let data = vec![1.0, 2.0, 3.0, 4.0, 5.0];
    js_sys::Float64Array::from(data.as_slice())
}
```

## Обработка ошибок

Для обработки ошибок возвращайте `Result<T, JsValue>`:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn divide(a: f64, b: f64) -> Result<f64, JsValue> {
    if b == 0.0 {
        Err(JsValue::from_str("Division by zero"))
    } else {
        Ok(a / b)
    }
}
```

В TypeScript это будет обрабатываться как исключение:

```typescript
try {
  const result = divide(10, 0);
  console.log(result);
} catch (error) {
  console.error('Error from Rust:', error);
}
```

## Работа с памятью

При работе с большими объемами данных важно понимать, как происходит управление памятью:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub struct DataProcessor {
    data: Vec<f64>,
}

#[wasm_bindgen]
impl DataProcessor {
    #[wasm_bindgen(constructor)]
    pub fn new(size: usize) -> DataProcessor {
        DataProcessor {
            data: vec![0.0; size],
        }
    }
    
    #[wasm_bindgen]
    pub fn set_value(&mut self, index: usize, value: f64) {
        if index < self.data.len() {
            self.data[index] = value;
        }
    }
    
    #[wasm_bindgen]
    pub fn get_value(&self, index: usize) -> f64 {
        self.data[index]
    }
    
    #[wasm_bindgen]
    pub fn process(&mut self) -> f64 {
        self.data.iter().sum()
    }
}
```

## Производительность

Rust в WebAssembly обеспечивает высокую производительность для вычислительно сложных задач:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn matrix_multiply(
    a: &[f64], 
    b: &[f64], 
    rows_a: usize, 
    cols_a: usize, 
    cols_b: usize
) -> Vec<f64> {
    let mut result = vec![0.0; rows_a * cols_b];
    
    for i in 0..rows_a {
        for j in 0..cols_b {
            for k in 0..cols_a {
                result[i * cols_b + j] += a[i * cols_a + k] * b[k * cols_b + j];
            }
        }
    }
    
    result
}
```

## Оптимизация сборки

Для оптимизации размера и производительности WASM модуля:

```toml
# Cargo.toml
[profile.release]
lto = true
opt-level = "s"  # или "z" для минимального размера
codegen-units = 1
panic = "abort"
strip = true
```

```bash
# Сборка с оптимизациями
wasm-pack build --release --target web
```

## Best Practices

1. **Минимизируйте вызовы между JS и WASM** - каждая граница вызова имеет накладные расходы
2. **Используйте подходящие типы данных** - передавайте TypedArray вместо обычных массивов
3. **Оптимизируйте алгоритмы в Rust** - именно там должна происходить тяжелая вычислительная работа
4. **Используйте профилирование** - определите узкие места в производительности
5. **Обрабатывайте ошибки должным образом** - используйте Result для передачи ошибок из Rust в JS

## Заключение

Комбинация Rust и TypeScript через WebAssembly позволяет создавать высокопроизводительные веб-приложения, используя сильные стороны обоих языков. Rust обеспечивает системную производительность, а TypeScript предоставляет удобную типизацию и разработку.

См. также:
- [[AssemblyScript]]
- [[Взаимодействие-с-WASM]]
- [[Оптимизация-производительности]]
- [[Отладка-WASM]]