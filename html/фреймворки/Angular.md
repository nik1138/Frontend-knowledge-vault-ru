---
aliases: [AngularJS, Angular 2+, Ангуляр]
tags: [programming, frontend, javascript, angular, framework, typescript]
---

# Angular

## Обзор

Angular - это платформа для разработки мобильных и веб-приложений, разработанная Google. Angular написан на TypeScript и позволяет создавать одностраничные приложения (SPA) с использованием компонентно-ориентированной архитектуры. Это полномасштабный фреймворк, который предоставляет все необходимые инструменты для разработки сложных приложений.

## Основные концепции

### Компоненты

Компоненты - это основные строительные блоки Angular-приложений. Каждый комппонент определяет представление (HTML-шаблон) и логику (TypeScript-класс), а также управляет частью пользовательского интерфейса.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: `<h1>Привет, {{ name }}!</h1>`,
  styles: ['h1 { color: blue; }']
})
export class HelloComponent {
  name = 'Angular';
}
```

### Директивы

Директивы позволяют изменять структуру DOM и поведение элементов. Существует три типа директив:

- Компоненты (компонентные директивы)
- Атрибутивные директивы (например, ngClass, ngStyle)
- Структурные директивы (например, ngIf, ngFor)

```html
<div *ngIf="isVisible">Отображается при условии</div>
<ul>
  <li *ngFor="let item of items">{{ item }}</li>
</ul>
```

### Сервисы и DI

Сервисы - это классы с узкоспециализированной задачей. Они могут быть внедрены в компоненты и другие сервисы с помощью системы внедрения зависимостей (Dependency Injection).

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  constructor(private http: HttpClient) {}

  getData() {
    return this.http.get('/api/data');
  }
}
```

### Модули

Модули (NgModule) - это контейнеры для компонентов, директив и сервисов. Они определяют, как части приложения объединяются вместе.

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

## Архитектура

Angular использует компонентно-ориентированную архитектуру:

- **Модули** - организуют приложение в логические части
- **Компоненты** - управляют представлением и логикой
- **Шаблоны** - определяют представление с помощью декларативного синтаксиса
- **Директивы** - изменяют поведение элементов DOM
- **Сервисы** - реализуют бизнес-логику
- **Роутинг** - управляет навигацией между представлениями

## RxJS и реактивное программирование

Angular активно использует RxJS (Reactive Extensions for JavaScript) для работы с асинхронными операциями и потоками данных. Observable - это ключевая концепция, позволяющая обрабатывать события, HTTP-запросы и другие асинхронные операции.

```typescript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();
});

observable.subscribe({
  next: value => console.log(value),
  complete: () => console.log('Завершено')
});
```

## Практические рекомендации

### Российские реалии 2025

В России в 2025 году Angular продолжает использоваться в корпоративной среде и крупных проектах. Особенности использования:

- Увеличение интереса к корпоративным решениям на Angular
- Адаптация под требования российского законодательства
- Интеграция с отечественными системами и API
- Использование локальных инфраструктур для разработки

### Лучшие практики

- Использование Angular CLI для инициализации и управления проектами
- Применение принципов чистого кода и SOLID
- Использование NgRx для управления сложным состоянием
- Тестирование с помощью Jasmine и Karma
- Использование Angular Material для UI-компонентов

## Экосистема

Angular имеет богатую экосистему инструментов и библиотек:

- **Angular CLI** - инструмент командной строки
- **Angular Material** - библиотека UI-компонентов
- **NgRx** - для управления состоянием
- **Angular Universal** - для серверного рендеринга
- **Protractor** - для end-to-end тестирования

## Преимущества и недостатки

### Преимущества

- Полнофункциональный фреймворк с богатым набором возможностей
- Статическая типизация с TypeScript
- Отличная документация и поддержка
- Мощная система внедрения зависимостей
- Отлично подходит для крупных приложений

### Недостатки

- Сложная кривая обучения
- Большой размер фреймворка
- Более жесткая структура по сравнению с другими фреймворками
- Более медленное развитие по сравнению с React/Vue

## Связанные темы

- [[TypeScript]]
- [[Frontend-разработка]]
- [[RxJS]]
- [[React]]
- [[Vue]]
- [[Svelte]]
- [[Сравнение-фреймворков]]
- [[Angular-CLI]]
- [[NgRx]]
- [[Angular-Material]]

## Источники и дополнительные ресурсы

- [Официальная документация Angular](https://angular.io/)
- [Angular на GitHub](https://github.com/angular/angular)
- [Angular CLI](https://cli.angular.io/)
- [Angular Material](https://material.angular.io/)