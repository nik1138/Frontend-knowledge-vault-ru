---
aliases: [AngularJS, Angular Framework, Ангуляр]
tags: [javascript, frontend, framework, angular, typescript, components, dependency-injection]
---

# Angular

## Общее описание

Angular - это платформа для разработки мобильных и веб-приложений, созданная командой Google. Это полнофункциональный фреймворк, который предоставляет все необходимые инструменты для создания сложных приложений "из коробки".

## Архитектура и основные концепции

### Модули (Modules)
Angular-приложения состоят из модулей, каждый из которых может содержать компоненты, сервисы и другие элементы. Главный модуль - AppModule.

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Компоненты
Компоненты - это строительные блоки пользовательского интерфейса. Каждый компонент имеет шаблон, класс и метаданные.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>
      <p>Счетчик: {{ count }}</p>
      <button (click)="increment()">Увеличить</button>
    </div>
  `
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++;
  }
}
```

### Сервисы и внедрение зависимостей
Сервисы позволяют разделять логику между компонентами. Angular предоставляет мощную систему внедрения зависимостей.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  constructor(private http: HttpClient) { }

  getData(): Observable<any> {
    return this.http.get('/api/data');
  }
}
```

## Основные возможности

### Шаблоны и привязка данных
Angular использует HTML с дополнительными синтаксическими конструкциями для привязки данных:

```html
<!-- Односторонняя привязка -->
<h1>{{ title }}</h1>
<img [src]="imageUrl" />

<!-- Двусторонняя привязка -->
<input [(ngModel)]="user.name" />

<!-- Событийная привязка -->
<button (click)="onSubmit()">Отправить</button>
```

### Директивы
Директивы позволяют изменять структуру DOM или внешний вид элементов:

```html
<!-- Условное отображение -->
<div *ngIf="isVisible">Отображается при условии</div>

<!-- Повторение элементов -->
<ul>
  <li *ngFor="let item of items">{{ item }}</li>
</ul>

<!-- Условие с альтернативой -->
<div *ngIf="user; else noUser">
  Привет, {{ user.name }}!
</div>
<ng-template #noUser>
  Пользователь не авторизован
</ng-template>
```

### Маршрутизация
Angular Router позволяет реализовать навигацию между представлениями:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: '**', redirectTo: '' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

## Экосистема Angular

### Angular CLI
Официальный инструмент командной строки для создания, разработки и тестирования Angular-приложений.

### RxJS
Библиотека для работы с асинхронными данными и событиями с использованием наблюдаемых (observables).

### Angular Material
Набор компонентов UI, реализующих Material Design от Google.

### NgRx
Библиотека для управления состоянием приложения, вдохновленная Redux.

## Практические рекомендации

### Структура проекта
```
src/
├── app/
│   ├── components/
│   ├── services/
│   ├── models/
│   ├── modules/
│   └── shared/
├── assets/
├── environments/
└── styles/
```

### Лучшие практики
- Используйте TypeScript для статической типизации
- Разделяйте приложение на модули
- Используйте сервисы для логики вне компонентов
- Оптимизируйте производительность с помощью OnPush стратегии
- Используйте lazy loading для уменьшения размера начальной загрузки

## Российские реалии 2025

### Популярность в России
Angular остается одним из ведущих фреймворков в России, особенно в корпоративной среде и крупных компаниях. Его "батарейки включены" подход делает его привлекательным для больших проектов.

### Ресурсы на русском языке
- [Angular - русская документация](https://angular.io/guide/typescript-configuration)
- YouTube-каналы с обучающими материалами
- Сообщество Angular-разработчиков в Telegram и Discord
- Курсы на Udemy, Stepik и других платформах

### Особенности рынка труда
- Высокий спрос на Angular-разработчиков в крупных компаниях
- Средняя зарплата в Москве: 180 000 - 350 000 рублей в месяц
- Часто требуют знание TypeScript, RxJS и архитектурных паттернов

## Сравнение с другими фреймворками

| Функция | Angular | React | Vue |
|---------|---------|-------|-----|
| Сложность изучения | Высокая | Средняя | Низкая |
| Производительность | Средняя | Высокая | Высокая |
| Экосистема | Полная (все включено) | Богатая (нужно собирать) | Средняя (опционально) |

## Ссылки и ресурсы

- [[React]] - Альтернативный фреймворк от Facebook
- [[Vue]] - Прогрессивный фреймворк для интерфейсов
- [[TypeScript]] - Язык программирования, на котором работает Angular
- [[Dependency Injection]] - Паттерн внедрения зависимостей в Angular
- [[RxJS]] - Библиотека реактивных расширений
- [[Angular CLI]] - Инструмент командной строки для Angular
- [[Angular Material]] - Компоненты Material Design
- [[NgRx]] - Управление состоянием в Angular

## Заключение

Angular остается мощной платформой для разработки корпоративных приложений. Его полная экосистема и строгая архитектура делают его отличным выбором для крупных проектов, особенно в российских реалиях 2025 года, где важны стабильность и масштабируемость.

> [!tip] Совет
> Начинайте изучение Angular с понимания основ TypeScript и архитектурных паттернов. Используйте Angular CLI для создания проектов и компонентов.

> [!warning] Важно
> Angular имеет крутую кривую обучения, но обеспечивает надежную архитектуру для сложных приложений. Следите за обновлениями и новыми возможностями фреймворка.