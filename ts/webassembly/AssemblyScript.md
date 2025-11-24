---
aliases: [AssemblyScript, AS]
tags: [typescript, webassembly, assemblyscript, wasm]
---

# AssemblyScript

AssemblyScript - это язык программирования, который компилируется в WebAssembly. Он представляет собой строго типизированный подмножество TypeScript, позволяющее разработчикам использовать привычный синтаксис TypeScript для создания высокопроизводительного WebAssembly кода.

## Основы AssemblyScript

AssemblyScript предоставляет строгую типизацию и структурированный подход к разработке WebAssembly модулей. Он поддерживает большинство возможностей TypeScript, но с ограничениями, необходимыми для компиляции в WebAssembly.

```typescript
// Пример простой функции на AssemblyScript
export function add(a: i32, b: i32): i32 {
  return a + b;
}

// Пример работы с массивами
export function sumArray(arr: Int32Array): i32 {
  let sum: i32 = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
  }
  return sum;
}
```

## Установка и настройка

Для начала работы с AssemblyScript необходимо установить соответствующие инструменты:

```bash
npm install -g assemblyscript
asc --init
```

Это создаст необходимую структуру проекта с файлами `asconfig.json`, `assembly/` и `build/`.

## Типы данных

AssemblyScript предоставляет специфические типы данных, соответствующие WebAssembly:

- `i32` - 32-битное целое число со знаком
- `i64` - 64-битное целое число со знаком
- `f32` - 32-битное число с плавающей точкой
- `f64` - 64-битное число с плавающей точкой
- `bool` - логический тип

```typescript
// Пример использования специфических типов
export function calculateDistance(x1: f64, y1: f64, x2: f64, y2: f64): f64 {
  const dx = x2 - x1;
  const dy = y2 - y1;
  return Math.sqrt(dx * dx + dy * dy);
}
```

## Работа с памятью

В AssemblyScript доступна работа с памятью на низком уровне:

```typescript
// Пример работы с указателями
export function setMemoryValue(ptr: usize, value: u8): void {
  store<u8>(ptr, value);
}

export function getMemoryValue(ptr: usize): u8 {
  return load<u8>(ptr);
}
```

## Компиляция

Компиляция AssemblyScript в WebAssembly осуществляется с помощью команды:

```bash
asc assembly/index.ts -b build/optimized.wasm -t build/optimized.wat --optimize
```

## Интеграция с TypeScript

Для загрузки и использования WebAssembly модуля, скомпилированного из AssemblyScript, в TypeScript приложении:

```typescript
// Загрузка WASM модуля
async function loadWasm(): Promise<any> {
  const wasmModule = await WebAssembly.instantiateStreaming(
    fetch('/path/to/optimized.wasm')
  );
  return wasmModule.instance.exports;
}

// Использование функций из WASM модуля
async function example() {
  const wasm = await loadWasm();
  const result = wasm.add(5, 10);
  console.log(result); // 15
}
```

## Продвинутые возможности

AssemblyScript поддерживает шаблоны, но с ограничениями:

```typescript
// Пример шаблона
export class Vector<T> {
  private data: Array<T>;

  constructor() {
    this.data = new Array<T>();
  }

  push(item: T): void {
    this.data.push(item);
  }

  get(index: i32): T {
    return this.data[index];
  }

  length(): i32 {
    return this.data.length;
  }
}
```

## Best Practices

1. **Использование строгой типизации** - AssemblyScript требует явного указания типов, что помогает избежать ошибок
2. **Оптимизация алгоритмов** - WebAssembly особенно эффективен для вычислительно сложных задач
3. **Минимизация обращений к памяти** - избегайте частых операций чтения/записи в память
4. **Тестирование** - используйте встроенные возможности для тестирования AssemblyScript кода

## Сравнение с другими языками

AssemblyScript отличается от Rust или C++ тем, что:
- Использует синтаксис, близкий к TypeScript
- Имеет более ограниченные возможности по сравнению с Rust
- Легче для изучения разработчиками, знакомыми с JavaScript/TypeScript

## Заключение

AssemblyScript представляет собой отличный инструмент для разработчиков, уже знакомых с TypeScript, которые хотят создавать высокопроизводительные WebAssembly модули. Он позволяет использовать привычный синтаксис и подходы, при этом обеспечивая близкую к нативной производительность в браузере.

См. также:
- [[Rust-и-TypeScript]]
- [[Взаимодействие-с-WASM]]
- [[Оптимизация-производительности]]
- [[Отладка-WASM]]