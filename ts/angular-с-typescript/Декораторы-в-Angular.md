---
aliases: [angular-decorators, angular-decorator-guide]
tags: [angular, typescript, decorators, frontend]
---

# Декораторы в Angular

Декораторы в Angular - это специальные функции, которые добавляют метаданные к классам, методам, свойствам и параметрам. Они являются ключевой частью архитектуры Angular и позволяют фреймворку понимать, как обрабатывать компоненты, сервисы и другие элементы приложения.

## Основные декораторы Angular

### @Component

Декоратор `@Component` используется для определения компонентов Angular. Он предоставляет метаданные, которые Angular использует для создания и отображения компонента.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.css']
})
export class ExampleComponent {
  title = 'Angular Decorators';
}
```

### @Directive

Декоратор `@Directive` используется для создания директив, которые могут изменять поведение DOM-элементов.

```typescript
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  constructor(private el: ElementRef) {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

### @Pipe

Декоратор `@Pipe` используется для создания кастомных пайпов, которые трансформируют отображаемые значения в шаблонах.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'reverse'
})
export class ReversePipe implements PipeTransform {
  transform(value: string): string {
    return value.split('').reverse().join('');
  }
}
```

## Декораторы свойств

### @Input

Декоратор `@Input` позволяет передавать данные из родительского компонента в дочерний.

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `<h2>{{ message }}</h2>`
})
export class ChildComponent {
  @Input() message: string = '';
}
```

### @Output

Декоратор `@Output` позволяет дочернему компоненту отправлять данные родительскому компоненту.

```typescript
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `<button (click)="sendData()">Send Data</button>`
})
export class ChildComponent {
  @Output() dataEvent = new EventEmitter<string>();

  sendData() {
    this.dataEvent.emit('Hello from child');
  }
}
```

## Декораторы методов

### @HostListener

Декоратор `@HostListener` позволяет подписаться на события хост-элемента.

```typescript
import { Directive, HostListener } from '@angular/core';

@Directive({
  selector: '[appClick]'
})
export class ClickDirective {
  @HostListener('click', ['$event'])
  onClick(event: Event) {
    console.log('Element clicked', event);
  }
}
```

## Декораторы внедрения зависимостей

### @Injectable

Декоратор `@Injectable` помечает класс как доступный для внедрения зависимостей.

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class DataService {
  getData(): string {
    return 'Data from service';
  }
}
```

## Декораторы параметров

### @Inject

Декоратор `@Inject` позволяет указать, какой токен внедрения использовать для параметра.

```typescript
import { Component, Inject } from '@angular/core';
import { DOCUMENT } from '@angular/common';

@Component({
  selector: 'app-example',
  template: `<div>Document title: {{ title }}</div>`
})
export class ExampleComponent {
  title: string;

  constructor(@Inject(DOCUMENT) private document: Document) {
    this.title = this.document.title;
  }
}
```

## Практические рекомендации

1. **Всегда указывайте типы** при использовании декораторов для обеспечения безопасности типов:
   ```typescript
   @Input() count: number = 0;
   @Output() countChange = new EventEmitter<number>();
   ```

2. **Используйте строгую типизацию** для событий:
   ```typescript
   @Output() userEvent = new EventEmitter<{ id: number, name: string }>();
   ```

3. **Следите за производительностью** при использовании декораторов, особенно в директивах, чтобы избежать лишних вычислений при каждом цикле изменений.

## Связанные темы

- [[Типизация-компонентов]]
- [[Типизация-сервисов]]
- [[Инъекция-зависимостей]]
- [[Модули-Angular]]

## Ключевые выводы

- Декораторы являются фундаментальной частью архитектуры Angular
- Они обеспечивают метаданные, необходимые для компиляции и выполнения приложения
- Правильное использование декораторов улучшает читаемость и поддерживаемость кода
- Всегда используйте строгую типизацию при работе с декораторами