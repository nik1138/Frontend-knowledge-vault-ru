---
aliases: [AngularJS, Angular 2+, Ангуляр]
tags: [javascript, фреймворк, angular, frontend]
---

# Angular

## Обзор

Angular - это платформа и фреймворк JavaScript для создания клиентских приложений, разработанный Google. В 2025 году Angular остается мощным решением для разработки корпоративных приложений и сложных систем.

## Основные особенности

- **Полнофункциональный фреймворк** - Включает все необходимые компоненты для разработки
- **Двустороннее связывание данных** - Упрощение синхронизации модели и представления
- **DI (Dependency Injection)** - Встроенная система внедрения зависимостей
- **RxJS** - Поддержка реактивного программирования
- **TypeScript** - Статическая типизация по умолчанию
- **Строгая архитектура** - Четкая структура приложения

## Установка и настройка

### Создание нового проекта

```bash
npm install -g @angular/cli
ng new my-angular-app
cd my-angular-app
ng serve
```

### Основные команды CLI

```bash
ng generate component my-component  # Создание компонента
ng generate service my-service     # Создание сервиса
ng generate directive my-directive # Создание директивы
ng build                           # Сборка проекта
ng test                            # Запуск тестов
ng update                          # Обновление Angular
```

## Основные концепции

### Компоненты

Компоненты - это основные строительные блоки Angular-приложения:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div class="counter">
      <p>Счетчик: {{ count }}</p>
      <button (click)="increment()">Увеличить</button>
    </div>
  `,
  styles: [`
    .counter {
      padding: 20px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
  `]
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++;
  }
}
```

### Сервисы

Сервисы используются для инкапсуляции бизнес-логики:

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  private todosSubject = new BehaviorSubject<any[]>([]);
  public todos$ = this.todosSubject.asObservable();

  addTodo(text: string) {
    const todos = this.todosSubject.value;
    const newTodo = { id: Date.now(), text, completed: false };
    this.todosSubject.next([...todos, newTodo]);
  }

  toggleTodo(id: number) {
    const todos = this.todosSubject.value.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    this.todosSubject.next(todos);
  }
}
```

### Модули

Модули объединяют компоненты, сервисы и другие части приложения:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { CounterComponent } from './counter/counter.component';

@NgModule({
  declarations: [
    AppComponent,
    CounterComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Практические примеры

### Простое приложение "Список задач"

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { TodoService } from './todo.service';

@Component({
  selector: 'app-root',
  template: `
    <div class="todo-app">
      <h1>Список задач</h1>
      <div class="input-section">
        <input 
          [(ngModel)]="newTodo" 
          (keyup.enter)="addTodo()" 
          placeholder="Добавить задачу"
        />
        <button (click)="addTodo()">Добавить</button>
      </div>
      <ul class="todo-list">
        <li 
          *ngFor="let todo of todos$ | async" 
          [class.completed]="todo.completed"
          (click)="toggleTodo(todo.id)"
        >
          {{ todo.text }}
        </li>
      </ul>
    </div>
  `,
  styles: [`
    .todo-app {
      max-width: 500px;
      margin: 0 auto;
      padding: 20px;
    }

    .input-section {
      display: flex;
      margin-bottom: 20px;
    }

    .input-section input {
      flex: 1;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px 0 0 4px;
    }

    .input-section button {
      padding: 8px 16px;
      border: 1px solid #ccc;
      border-left: none;
      border-radius: 0 4px 4px 0;
      background-color: #f5f5f5;
    }

    .todo-list {
      list-style: none;
      padding: 0;
    }

    .todo-list li {
      padding: 10px;
      border: 1px solid #eee;
      margin-bottom: 5px;
      cursor: pointer;
      border-radius: 4px;
    }

    .todo-list li.completed {
      text-decoration: line-through;
      color: #999;
    }
  `]
})
export class AppComponent {
  newTodo = '';
  todos$ = this.todoService.todos$;

  constructor(private todoService: TodoService) {}

  addTodo() {
    if (this.newTodo.trim() !== '') {
      this.todoService.addTodo(this.newTodo);
      this.newTodo = '';
    }
  }

  toggleTodo(id: number) {
    this.todoService.toggleTodo(id);
  }
}
```

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { TodoService } from './todo.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [TodoService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Экосистема Angular

- [[Angular Router]] - Маршрутизация
- [[Angular Material]] - Компонентная библиотека
- [[NgRx]] - Управление состоянием (альтернатива Redux)
- [[Angular Universal]] - Server-Side Rendering
- [[Angular CLI]] - Инструмент командной строки
- [[Protractor]] - Инструмент тестирования

## Российские реалии и особенности

### Применение в российских компаниях

Angular активно используется в крупных российских компаниях:

- [[Сбер]] - во многих цифровых продуктах
- [[Газпром нефть]] - в корпоративных системах
- [[РЖД]] - в веб-приложениях
- [[Росатом]] - в специализированных системах
- [[МТС]] - в клиентских интерфейсах

### Особенности разработки в России

- **Корпоративные решения** - Angular идеально подходит для крупных корпоративных систем
- **Типизация** - Статическая типизация TypeScript особенно важна для больших команд
- **Документация** - Хорошо документированный фреймворк с русскоязычными ресурсами
- **Обучение** - Много обучающих материалов и курсов на русском языке

## Лучшие практики

- Использование RxJS для управления асинхронными операциями
- Правильная архитектура модулей
- Тестирование с помощью Jasmine и Karma
- Использование Angular CLI для генерации компонентов
- Оптимизация производительности с помощью OnPush стратегии

## Современные тенденции (2025)

- **Angular Signals** - Новая система реактивности
- **Standalone Components** - Упрощение архитектуры
- **Angular DevTools** - Постоянное улучшение инструментов разработки
- **Ivy Renderer** - Улучшенная производительность и уменьшенный размер
- **Server-Side Rendering** - Улучшенная поддержка SEO и производительности

## Ресурсы для изучения

- [[Официальная документация Angular]]
- [[Angular University]]
- [[Angular Academy]]
- [[Angular Best Practices]]
- [[Angular Component Patterns]]

## Заключение

Angular остается мощным выбором для разработки корпоративных и сложных приложений в 2025 году. Его строгая архитектура и богатая экосистема делают его отличным выбором для команд, работающих над крупномасштабными проектами, особенно в российском контексте.

> [!tip] Совет
> Используйте Angular CLI для автоматизации рутинных задач и соблюдения стандартов кодирования.

> [!warning] Важно
> Angular имеет более крутую кривую обучения по сравнению с другими фреймворками, но это компенсируется мощной экосистемой и строгой архитектурой.