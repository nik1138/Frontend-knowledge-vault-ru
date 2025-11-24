---
aliases: [Tauri Framework, Tauri.js]
tags: [framework, desktop-applications, rust, typescript]
---

# Tauri

## Обзор

Tauri - это современный фреймворк с открытым исходным кодом для создания кроссплатформенных десктопных приложений с использованием веб-технологий в качестве основы для пользовательского интерфейса, но с нативным бэкендом, написанным на Rust. В отличие от Electron, Tauri использует системный веб-просмотр для отображения пользовательского интерфейса, что значительно уменьшает размер итогового приложения и потребление ресурсов.

## Архитектура Tauri

Tauri использует принципиально другую архитектуру по сравнению с Electron:

- **Frontend**: Веб-интерфейс (HTML, CSS, JavaScript/TypeScript)
- **Backend**: Нативный бэкенд на Rust
- **Tauri Core**: Обеспечивает связь между фронтендом и бэкендом

### Основные компоненты

- **Tauri API**: Позволяет фронтенду взаимодействовать с системой
- **Commands**: Функции Rust, вызываемые из фронтенда
- **Events**: Механизм обмена сообщениями между фронтендом и бэкендом

## Установка и настройка

### 1. Установка зависимостей

```bash
# Установка Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Установка Tauri CLI
cargo install tauri-cli

# Установка зависимостей в проекте
npm install @tauri-apps/api @tauri-apps/cli
```

### 2. Инициализация проекта

```bash
# В директории с фронтенд-проектом
npx tauri init
```

## Пример приложения

### Frontend (TypeScript)

```typescript
// src-tauri/src/main.ts
import { invoke } from '@tauri-apps/api/tauri';
import { listen } from '@tauri-apps/api/event';

// Пример вызова команды Rust
async function greet(name: string): Promise<string> {
  return await invoke('greet', { name });
}

// Пример обработки события
async function setupEventListeners() {
  await listen('rust-event', (event) => {
    console.log('Получено событие от Rust:', event.payload);
  });
}

// Использование команды
document.getElementById('greet-button')?.addEventListener('click', async () => {
  const name = document.getElementById('name-input') as HTMLInputElement;
  const greeting = await greet(name.value);
  document.getElementById('greeting-output').textContent = greeting;
});
```

### Backend (Rust)

```rust
// src-tauri/src/main.rs
#![cfg_attr(
  all(not(debug_assertions), target_os = "windows"),
  windows_subsystem = "windows"
)]

use tauri::{Manager, Window};

#[tauri::command]
fn greet(name: &str) -> String {
  format!("Привет, {}! Это Rust говорит!", name)
}

#[tauri::command]
fn save_file(window: Window, content: String) -> Result<(), String> {
  use std::fs::File;
  use std::io::Write;
  
  let file_path = std::path::Path::new("output.txt");
  let mut file = File::create(file_path).map_err(|e| e.to_string())?;
  file.write_all(content.as_bytes()).map_err(|e| e.to_string())?;
  
  // Отправка события обратно во frontend
  window.emit("file-saved", "Файл успешно сохранен").unwrap();
  
  Ok(())
}

fn main() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![greet, save_file])
    .run(tauri::generate_context!())
    .expect("error while running tauri application");
}
```

## Преимущества Tauri

- **Маленький размер приложения**: Благодаря использованию системного веб-просмотра
- **Низкое потребление памяти**: В отличие от Electron, Tauri не включает движок Chromium
- **Высокая производительность**: Нативный бэкенд на Rust
- **Лучшая безопасность**: Более строгий контроль доступа к системным ресурсам
- **Лицензионная чистота**: Не имеет проблем с лицензированием, как Electron

## Недостатки Tauri

- **Меньшее сообщество**: По сравнению с Electron, сообщество и экосистема меньше
- **Кривая обучения**: Необходимо изучать Rust для бэкенд-разработки
- **Меньше готовых решений**: Меньше плагинов и готовых компонентов

## Практические рекомендации

### 1. Работа с асинхронными командами

```typescript
// Frontend
import { invoke } from '@tauri-apps/api/tauri';

interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUsers(): Promise<User[]> {
  try {
    const users: User[] = await invoke('fetch_users');
    return users;
  } catch (error) {
    console.error('Ошибка при получении пользователей:', error);
    return [];
  }
}
```

```rust
// Backend
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
  id: u32,
  name: String,
  email: String,
}

#[tauri::command]
fn fetch_users() -> Result<Vec<User>, String> {
  // Симуляция получения данных
  Ok(vec![
    User {
      id: 1,
      name: "Иван Иванов".to_string(),
      email: "ivan@example.com".to_string(),
    },
    User {
      id: 2,
      name: "Петр Петров".to_string(),
      email: "petr@example.com".to_string(),
    },
  ])
}
```

### 2. Обработка ошибок

```rust
#[tauri::command]
fn safe_operation(input: String) -> Result<String, String> {
  if input.is_empty() {
    return Err("Входная строка не может быть пустой".to_string());
  }
  
  // Выполнение операции
  Ok(format!("Обработано: {}", input))
}
```

### 3. Работа с файловой системой

```rust
use std::fs;

#[tauri::command]
fn read_file(path: String) -> Result<String, String> {
  fs::read_to_string(&path)
    .map_err(|e| format!("Ошибка чтения файла {}: {}", path, e))
}

#[tauri::command]
fn write_file(path: String, content: String) -> Result<(), String> {
  fs::write(&path, content)
    .map_err(|e| format!("Ошибка записи в файл {}: {}", path, e))
}
```

## Связанные темы

- [[Electron]] - Альтернативный фреймворк для создания десктопных приложений
- [[Десктопные-паттерны]] - Паттерны проектирования для десктопных приложений
- [[Оптимизация-для-десктопа]] - Рекомендации по оптимизации десктопных приложений
- [[Rust]] - Язык программирования, используемый в Tauri
- [[Webpack]] - Сборка Tauri-приложений

## Заключение

Tauri представляет собой современную альтернативу Electron с фокусом на производительность, безопасность и малый размер приложений. Он идеально подходит для разработчиков, которые хотят использовать веб-технологии для интерфейса, но нуждаются в нативной производительности бэкенда.