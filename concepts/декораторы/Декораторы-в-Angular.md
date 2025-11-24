---
aliases: [Angular Decorators, Декораторы в Angular, Angular, Фронтенд, TypeScript]
tags: [angular, decorators, frontend, typescript, programming]
---

# Декораторы в Angular

## Обзор

Декораторы в Angular - это специальные функции TypeScript, которые добавляют метаданные к классам, методам, свойствам и параметрам. Они являются фундаментальной частью архитектуры Angular и позволяют фреймворку понимать, как обрабатывать компоненты, сервисы, модули и другие элементы приложения.

Декораторы обозначаются символом `@` и применяются перед объявлением класса, метода или свойства. Angular предоставляет множество встроенных декораторов, каждый из которых имеет свою специфическую цель и функциональность.

## Основные декораторы Angular

### @Component

Декоратор `@Component` используется для определения компонента Angular - основного строительного блока пользовательского интерфейса.

#### Свойства декоратора

- `selector` - CSS-селектор, который указывает, где будет отображаться компонент
- `template` или `templateUrl` - HTML-шаблон компонента
- `styles` или `styleUrls` - стили компонента
- `providers` - список сервисов, которые будут инстанцированы и доступны в области видимости компонента

#### Пример

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.css']
})
export class ExampleComponent {
  title = 'Пример компонента';
}
```

[[Компоненты в Angular]]

### @NgModule

Декоратор `@NgModule` используется для определения Angular модуля, который организует приложение в логические блоки.

#### Свойства декоратора

- `declarations` - список компонентов, директив и пайпов, принадлежащих модулю
- `imports` - список модулей, от которых зависит текущий модуль
- `exports` - список компонентов, директив и пайпов, доступных другим модулям
- `providers` - список сервисов, доступных в приложении
- `bootstrap` - главный компонент, с которого начинается загрузка приложения

#### Пример

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

[[Модули в Angular]]

### @Injectable

Декоратор `@Injectable` используется для обозначения сервиса, который может быть внедрен в другие классы. Это необходимо для правильной работы механизма внедрения зависимостей Angular.

#### Пример

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // Сервис будет предоставлен в корне приложения
})
export class DataService {
  getData() {
    return 'Данные из сервиса';
  }
}
```

[[Сервисы в Angular]]

### @Input

Декоратор `@Input` используется для передачи данных из родительского компонента в дочерний компонент.

#### Пример

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-child',
  template: '<p>{{ message }}</p>'
})
export class ChildComponent {
  @Input() message: string = '';
}
```

[[Передача данных между компонентами]]

### @Output

Декоратор `@Output` используется для отправки данных из дочернего компонента в родительский компонент через события.

#### Пример

```typescript
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: '<button (click)="sendMessage()">Отправить сообщение</button>'
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>();

  sendMessage() {
    this.messageEvent.emit('Сообщение от дочернего компонента');
  }
}
```

### @ViewChild и @ViewChildren

Декораторы `@ViewChild` и `@ViewChildren` используются для получения ссылок на дочерние элементы или компоненты в шаблоне.

#### Пример

```typescript
import { Component, ViewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-example',
  template: '<div #myDiv>Пример элемента</div>'
})
export class ExampleComponent {
  @ViewChild('myDiv', { static: true }) myDiv!: ElementRef;

  ngAfterViewInit() {
    // Доступ к элементу после инициализации представления
    console.log(this.myDiv.nativeElement);
  }
}
```

## Декораторы для работы с HTTP

### @Http

Хотя `@Http` больше не используется в современных версиях Angular (заменен `HttpClient`), важно понимать исторический контекст.

### @Inject и @Optional

Декораторы `@Inject` и `@Optional` используются в конструкторах для внедрения зависимостей с дополнительными опциями.

#### Пример

```typescript
import { Component, Inject, Optional } from '@angular/core';
import { DOCUMENT } from '@angular/common';

@Component({
  selector: 'app-example',
  template: '<p>Пример использования Inject</p>'
})
export class ExampleComponent {
  constructor(@Inject(DOCUMENT) @Optional() private document: any) {}
}
```

## Декораторы для роутинга

### @RouteConfig

> Примечание: `@RouteConfig` устарел. Современный Angular использует декларативную конфигурацию маршрутов.

### @CanActivate, @CanDeactivate

Эти декораторы использовались в старых версиях Angular для защиты маршрутов. В современном Angular используются специальные интерфейсы и функции-кантины.

#### Пример с современным подходом

```typescript
import { Injectable } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  isLoggedIn(): boolean {
    return !!localStorage.getItem('token');
  }
}

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;
  }

  router.navigate(['/login']);
  return false;
};
```

[[Маршрутизация в Angular]]

## Пользовательские декораторы

Можно создавать собственные декораторы для повторного использования логики.

#### Пример пользовательского декоратора свойства

```typescript
function LogProperty(target: any, propertyKey: string) {
  console.log(`Свойство ${propertyKey} было определено в классе ${target.constructor.name}`);
}

class ExampleClass {
  @LogProperty
  myProperty: string = 'значение';
}
```

## Практические рекомендации

### 1. Правильное использование providedIn

При использовании `@Injectable` рекомендуется указывать `providedIn: 'root'` для автоматического предоставления сервиса в корне приложения:

```typescript
@Injectable({
  providedIn: 'root'
})
export class MyService { }
```

### 2. Использование OnPush стратегии изменений

При работе с декораторами компонентов можно оптимизировать производительность:

```typescript
@Component({
  selector: 'app-optimized',
  template: '<p>{{ data }}</p>',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  @Input() data: string = '';
}
```

### 3. Комбинирование декораторов

Можно комбинировать несколько декораторов на одном элементе:

```typescript
export class MyComponent {
  @Input() @HostBinding('class.active') isActive: boolean = false;
}
```

## Заключение

Декораторы являются ключевым элементом архитектуры Angular и обеспечивают мощный механизм для добавления метаданных и функциональности к классам и свойствам. Понимание и правильное использование декораторов позволяет создавать более чистый, поддерживаемый и эффективный код.

## См. также

- [[Компоненты в Angular]]
- [[Сервисы в Angular]]
- [[Маршрутизация в Angular]]
- [[Внедрение зависимостей в Angular]]
- [[TypeScript для Angular]]
- [[Шаблоны в Angular]]
- [[Директивы в Angular]]
- [[Пайпы в Angular]]
- [[Жизненный цикл компонентов в Angular]]
- [[Тестирование в Angular]]

## Ключевые теги

#angular #decorators #frontend #typescript #programming #web-development #components #services #routing #dependency-injection