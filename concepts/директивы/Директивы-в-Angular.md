---
aliases: [Angular директивы, Angular Directives, Директивы Angular]
tags: [angular, frontend, directives, typescript]
---

# Директивы в Angular

## Обзор

Директивы в Angular - это классы, которые добавляют дополнительное поведение к элементам DOM или изменяют существующее поведение. Они являются одной из трех основных концепций Angular (вместе с компонентами и сервисами) и позволяют расширять возможности HTML с помощью пользовательской логики.

## Типы директив

Angular поддерживает три типа директив:

1. **Компоненты** - директивы с шаблоном
2. **Структурные директивы** - изменяют структуру DOM
3. **Атрибутные директивы** - изменяют внешний вид или поведение элементов

## Структурные директивы

### *ngIf

Условная отрисовка - добавляет или удаляет элемент из DOM на основе условия.

```typescript
// component.ts
export class MyComponent {
  showElement = true;
  
  toggleElement() {
    this.showElement = !this.showElement;
  }
}
```

```html
<!-- template.html -->
<div *ngIf="showElement">Этот элемент может быть скрыт</div>
<button (click)="toggleElement()">Переключить</button>

<!-- С использованием else -->
<div *ngIf="condition; else elseTemplate">
  Отображается, если условие истинно
</div>
<ng-template #elseTemplate>
  Отображается, если условие ложно
</ng-template>

<!-- С шаблонной переменной для доступа к результату условия -->
<div *ngIf="condition as value; else noValue">
  {{ value }}
</div>
<ng-template #noValue>
  Условие ложно
</ng-template>
```

### *ngFor

Отрисовка списков - позволяет повторять элементы на основе массива.

```html
<ul>
  <li *ngFor="let item of items; let i = index; trackBy: trackByFn">
    {{ i }} - {{ item.name }}
  </li>
</ul>
```

```typescript
export class MyComponent {
  items = [
    { id: 1, name: 'Яблоко' },
    { id: 2, name: 'Банан' },
    { id: 3, name: 'Апельсин' }
  ];
  
  trackByFn(index: number, item: any): any {
    return item.id;
  }
}
```

**Доступные переменные в ngFor:**
- `index` - индекс текущего элемента
- `count` - общее количество элементов
- `first` - true, если это первый элемент
- `last` - true, если это последний элемент
- `even` - true, если индекс четный
- `odd` - true, если индекс нечетный

```html
<li *ngFor="
  let item of items; 
  let i = index; 
  let isFirst = first; 
  let isLast = last; 
  let isEven = even; 
  let isOdd = odd
">
  <span [class.first]="isFirst" [class.last]="isLast" [class.even]="isEven" [class.odd]="isOdd">
    {{ i }} - {{ item.name }}
  </span>
</li>
```

### *ngSwitch

Условная отрисовка нескольких вариантов (аналог switch-case).

```html
<div [ngSwitch]="currentItem.feature">
  <app-stout-item *ngSwitchCase="'stout'" [item]="currentItem"></app-stout-item>
  <app-ipa-item *ngSwitchCase="'ipa'" [item]="currentItem"></app-ipa-item>
  <app-lager-item *ngSwitchCase="'lager'" [item]="currentItem"></app-lager-item>
  <app-default-item *ngSwitchDefault [item]="currentItem"></app-default-item>
</div>
```

## Атрибутные директивы

### NgClass

Динамическое применение CSS классов к элементу.

```html
<!-- Применение одного класса -->
<div [ngClass]="'special'">Текст</div>

<!-- Условное применение класса -->
<div [ngClass]="isSpecial ? 'special' : ''">Текст</div>

<!-- Применение нескольких классов через объект -->
<div [ngClass]="{ 
  'special': isSpecial, 
  'normal': !isSpecial,
  'disabled': isDisabled
}">Текст</div>

<!-- Применение классов через массив -->
<div [ngClass]="['special', 'highlight']">Текст</div>

<!-- Комбинация условий -->
<div [ngClass]="classObject">Текст</div>
```

```typescript
export class MyComponent {
  isSpecial = true;
  isDisabled = false;
  
  classObject = {
    'special': this.isSpecial,
    'disabled': this.isDisabled
  };
}
```

### NgStyle

Динамическое применение стилей к элементу.

```html
<!-- Применение стилей через объект -->
<div [ngStyle]="{
  'color': isSpecial ? 'red' : 'blue',
  'font-weight': isSpecial ? 'bold' : 'normal'
}">Текст</div>

<!-- Применение стилей через метод -->
<div [ngStyle]="setStyles()">Текст</div>
```

```typescript
export class MyComponent {
  isSpecial = true;
  
  setStyles() {
    return {
      'color': this.isSpecial ? 'red' : 'blue',
      'font-weight': this.isSpecial ? 'bold' : 'normal'
    };
  }
}
```

### NgModel

Двустороннее связывание данных (требует `FormsModule`).

```html
<input [(ngModel)]="userName" placeholder="Введите имя">
<p>Привет, {{ userName }}!</p>

<!-- С использованием имени переменной -->
<input [(ngModel)]="userName" #nameInput="ngModel" placeholder="Введите имя">
<p>Значение: {{ nameInput.value }}</p>

<!-- С валидацией -->
<input 
  [(ngModel)]="email" 
  #emailInput="ngModel" 
  required 
  email
  placeholder="Email">
<div [hidden]="emailInput.valid || emailInput.pristine" class="error">
  Email недействителен
</div>
```

## Создание пользовательских директив

### Атрибутная директива

Создадим директиву для подсветки текста при наведении:

```typescript
// highlight.directive.ts
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'  // CSS-селектор для применения директивы
})
export class HighlightDirective {
  @Input() appHighlight: string = 'yellow';  // Цвет подсветки
  @Input() defaultColor: string = 'white';   // Цвет по умолчанию
  
  constructor(private el: ElementRef) {
    // ElementRef предоставляет доступ к DOM-элементу
  }
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.appHighlight || this.defaultColor);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(this.defaultColor);
  }
  
  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

Использование директивы:

```html
<p appHighlight="yellow">Обычное выделение</p>
<p [appHighlight]="'orange'" [defaultColor]="'lightblue'">Выделение с настройками</p>
```

### Структурная директива

Создадим структурную директиву для отображения содержимого только для аутентифицированных пользователей:

```typescript
// auth-structural.directive.ts
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appAuth]'
})
export class AuthStructuralDirective {
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appAuth(condition: boolean) {
    if (condition) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
}
```

Использование:

```html
<div *appAuth="user.isAuthenticated">
  <p>Содержимое только для авторизованных пользователей</p>
</div>
```

### Более сложная атрибутная директива

Создадим директиву для изменения размера шрифта:

```typescript
// resize.directive.ts
import { 
  Directive, 
  ElementRef, 
  HostListener, 
  Input, 
  OnInit 
} from '@angular/core';

@Directive({
  selector: '[appResize]'
})
export class ResizeDirective implements OnInit {
  @Input() appResize: number = 16; // размер шрифта по умолчанию
  @Input() resizeStep: number = 2; // шаг изменения
  @Input() maxFontSize: number = 32;
  @Input() minFontSize: number = 12;
  
  currentSize: number = 0;
  
  constructor(private el: ElementRef) {}
  
  ngOnInit() {
    this.currentSize = this.appResize;
    this.updateFontSize();
  }
  
  @HostListener('wheel', ['$event']) onWheel(event: WheelEvent) {
    event.preventDefault();
    
    if (event.deltaY < 0) {
      // Прокрутка вверх - увеличение
      this.currentSize = Math.min(this.currentSize + this.resizeStep, this.maxFontSize);
    } else {
      // Прокрутка вниз - уменьшение
      this.currentSize = Math.max(this.currentSize - this.resizeStep, this.minFontSize);
    }
    
    this.updateFontSize();
  }
  
  private updateFontSize() {
    this.el.nativeElement.style.fontSize = `${this.currentSize}px`;
  }
}
```

Использование:

```html
<p appResize [resizeStep]="1" [maxFontSize]="24">Текст с изменяемым размером</p>
```

## Встроенные атрибутные директивы

### NgNonBindable

Предотвращает интерполяцию Angular в элементе и его потомках.

```html
<div>
  {{ не будет интерполировано }}
  <span ngNonBindable>{{ будет выведено как есть }}</span>
</div>
```

### AsyncPipe

Позволяет прямое использование Observables в шаблонах.

```typescript
// component.ts
import { Observable, interval } from 'rxjs';

export class MyComponent {
  seconds$: Observable<number> = interval(1000);
  
  promiseValue = new Promise((resolve) => {
    setTimeout(() => resolve('Promise resolved!'), 2000);
  });
}
```

```html
<!-- template.html -->
<div>{{ seconds$ | async }}</div>
<div>{{ promiseValue | async }}</div>
```

## Распространенные пользовательские директивы

### Директива для автоскрытия (click-outside)

```typescript
// click-outside.directive.ts
import { 
  Directive, 
  Output, 
  EventEmitter, 
  HostListener 
} from '@angular/core';

@Directive({
  selector: '[appClickOutside]'
})
export class ClickOutsideDirective {
  @Output() appClickOutside = new EventEmitter<void>();
  
  @HostListener('document:click', ['$event'])
  onClick(event: Event) {
    if (!event.composedPath().includes(this.el.nativeElement)) {
      this.appClickOutside.emit();
    }
  }
  
  constructor(private el: ElementRef) {}
}
```

Использование:

```html
<div appClickOutside (appClickOutside)="closeDropdown()">
  <p>Контент, который закрывается при клике вне</p>
</div>
```

### Директива для debounce

```typescript
// debounce.directive.ts
import { 
  Directive, 
  Input, 
  Output, 
  EventEmitter, 
  HostListener 
} from '@angular/core';
import { Subject } from 'rxjs';
import { debounceTime, distinctUntilChanged } from 'rxjs/operators';

@Directive({
  selector: '[appDebounce]'
})
export class DebounceDirective {
  @Input() appDebounceTime: number = 300;
  @Output() appDebounce = new EventEmitter<any>();
  
  private inputEvent = new Subject<any>();
  
  ngOnInit() {
    this.inputEvent
      .pipe(
        debounceTime(this.appDebounceTime),
        distinctUntilChanged()
      )
      .subscribe(value => {
        this.appDebounce.emit(value);
      });
  }
  
  @HostListener('input', ['$event'])
  onInput(event: any) {
    this.inputEvent.next(event.target.value);
  }
}
```

Использование:

```html
<input 
  appDebounce 
  [appDebounceTime]="500" 
  (appDebounce)="onDebouncedInput($event)"
  placeholder="Поиск с задержкой">
```

## Продвинутые возможности

### Директивы с ContentChildren

```typescript
// tab-container.directive.ts
import { 
  Directive, 
  QueryList, 
  ContentChildren, 
  AfterContentInit 
} from '@angular/core';
import { TabDirective } from './tab.directive';

@Directive({
  selector: 'app-tab-container'
})
export class TabContainerDirective implements AfterContentInit {
  @ContentChildren(TabDirective) tabs!: QueryList<TabDirective>;
  
  ngAfterContentInit() {
    // Настройка вкладок после инициализации содержимого
    const activeTabs = this.tabs.filter(tab => tab.active);
    
    if (activeTabs.length === 0) {
      this.selectTab(this.tabs.first);
    }
  }
  
  selectTab(tab: TabDirective) {
    this.tabs.toArray().forEach(t => t.active = false);
    tab.active = true;
  }
}
```

### Директивы с инъекцией зависимостей

```typescript
// loading.directive.ts
import { Directive, TemplateRef, ViewContainerRef } from '@angular/core';
import { LoadingService } from '../services/loading.service';

@Directive({
  selector: '[appLoading]'
})
export class LoadingDirective {
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private loadingService: LoadingService
  ) {
    this.loadingService.isLoading$.subscribe(isLoading => {
      if (isLoading) {
        this.viewContainer.createEmbeddedView(this.templateRef);
      } else {
        this.viewContainer.clear();
      }
    });
  }
}
```

## Лучшие практики

### 1. Правильное именование

Используйте префикс для пользовательских директив:

```typescript
// Хорошо
@Directive({ selector: '[appHighlight]' })

// Избегайте конфликта с встроенными директивами
@Directive({ selector: '[appCustomClass]' }) // вместо [ngClassCustom]
```

### 2. Очистка ресурсов

Если директива использует подписки, обязательно отписывайтесь:

```typescript
import { Directive, OnDestroy } from '@angular/core';
import { Subject } from 'rxjs';

@Directive({
  selector: '[appCleanup]'
})
export class CleanupDirective implements OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### 3. Использование HostListener и HostBinding

```typescript
import { Directive, HostListener, HostBinding } from '@angular/core';

@Directive({
  selector: '[appToggleClass]'
})
export class ToggleClassDirective {
  @HostBinding('class.active') isActive = false;
  
  @HostListener('click') toggle() {
    this.isActive = !this.isActive;
  }
}
```

### 4. Проверка условий

```typescript
@Directive({
  selector: '[appConditional]'
})
export class ConditionalDirective {
  @Input() set appConditional(condition: boolean) {
    if (condition) {
      this.applyStyle();
    } else {
      this.removeStyle();
    }
  }
  
  private applyStyle() {
    // Логика применения стиля
  }
  
  private removeStyle() {
    // Логика удаления стиля
  }
}
```

## Модуль директив

Для организации директив рекомендуется создавать отдельный модуль:

```typescript
// directives.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HighlightDirective } from './highlight.directive';
import { ResizeDirective } from './resize.directive';
import { ClickOutsideDirective } from './click-outside.directive';

@NgModule({
  declarations: [
    HighlightDirective,
    ResizeDirective,
    ClickOutsideDirective
  ],
  imports: [
    CommonModule
  ],
  exports: [
    HighlightDirective,
    ResizeDirective,
    ClickOutsideDirective
  ]
})
export class DirectivesModule { }
```

Затем импортируйте этот модуль в нужные компоненты или в основной модуль приложения.

> [!tip] 
> При создании пользовательских директив всегда учитывайте, что они должны быть сосредоточены на одной задаче, чтобы обеспечить переиспользуемость и тестируемость.

> [!warning] 
> Избегайте сложной логики в директивах. Если поведение становится слишком сложным, возможно, лучше создать компонент.

## Сравнение с другими фреймворками

- В Angular директивы более формализованы через декораторы и систему типов
- По сравнению с Vue, Angular требует больше шаблонного кода для создания директив
- В отличие от Svelte, Angular предоставляет более строгую архитектуру для директив

## См. также

- [[Директивы-в-Vue]]
- [[Директивы-в-Svelte]]
- [[Пользовательские-директивы]]
- [[Встроенные-директивы]]