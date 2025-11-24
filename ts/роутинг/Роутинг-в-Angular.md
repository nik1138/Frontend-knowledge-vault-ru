---
aliases: [Angular Router, Роутинг в Angular, Angular Navigation]
tags: [angular, routing, typescript, frontend]
---

# Роутинг в Angular

## Обзор

Роутинг в Angular реализуется с помощью встроенной системы Angular Router, которая позволяет создавать одностраничные приложения с навигацией между различными представлениями (компонентами) без перезагрузки страницы. Angular Router предоставляет мощный и гибкий способ управления навигацией в приложении.

## Установка и настройка

Angular Router входит в состав Angular, поэтому дополнительной установки не требуется. Достаточно импортировать `RouterModule` и `Routes`.

### Базовая настройка

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { UserComponent } from './user/user.component';

const routes: Routes = [
  { path: '', component: HomeComponent, pathMatch: 'full' },
  { path: 'about', component: AboutComponent },
  { path: 'user/:id', component: UserComponent },
  { path: '**', redirectTo: '/' } // Wildcard route для 404
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Подключение к приложению

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    AboutComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Основные концепции

### Routes

```typescript
import { Routes } from '@angular/router';

export const appRoutes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    data: { title: 'Панель управления' },
    resolve: { user: UserResolver }
  },
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [AuthGuard],
    canDeactivate: [CanDeactivateGuard]
  }
];
```

### Route Configuration

```typescript
interface Route {
  path: string;
  component: Type<any>;
  redirectTo?: string;
  outlet?: string;
  canActivate?: any[];
  canActivateChild?: any[];
  canDeactivate?: any[];
  canLoad?: any[];
  data?: Data;
  resolve?: ResolveData;
  children?: Routes;
  loadChildren?: LoadChildren;
  // и другие свойства
}
```

## Навигация

### Декларативная навигация

```html
<!-- app.component.html -->
<nav>
  <a routerLink="/">Главная</a>
  <a routerLink="/about" routerLinkActive="active">О нас</a>
  <a [routerLink]="['/user', userId]" [queryParams]="{ tab: 'profile' }">Профиль</a>
</nav>

<!-- Router outlet для отображения компонентов -->
<router-outlet></router-outlet>

<!-- Named outlets -->
<router-outlet name="sidebar"></router-outlet>
```

### Программная навигация

```typescript
// user.service.ts
import { Injectable } from '@angular/core';
import { Router } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private router: Router) {}

  navigateToUser(userId: number) {
    // Простая навигация
    this.router.navigate(['/user', userId]);
    
    // С параметрами запроса
    this.router.navigate(['/user', userId], {
      queryParams: { tab: 'settings' },
      fragment: 'top'
    });
    
    // Замена записи в истории
    this.router.navigate(['/login'], { replaceUrl: true });
  }
}
```

## Параметры маршрута

### Route Parameters

```typescript
// user.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user',
  template: `<h1>Пользователь: {{ userId }}</h1>`
})
export class UserComponent implements OnInit {
  userId: string | null = null;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Получение параметров маршрута
    this.userId = this.route.snapshot.paramMap.get('id');
    
    // Для динамических изменений
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
      // Обработка изменения параметра
    });
  }
}
```

### Query Parameters

```typescript
// search.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-search',
  template: `
    <p>Поиск: {{ searchTerm }}</p>
    <p>Страница: {{ page }}</p>
  `
})
export class SearchComponent implements OnInit {
  searchTerm: string = '';
  page: number = 1;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Получение query параметров
    this.searchTerm = this.route.snapshot.queryParamMap.get('q') || '';
    this.page = +this.route.snapshot.queryParamMap.get('page') || 1;
    
    // Для динамических изменений
    this.route.queryParamMap.subscribe(params => {
      this.searchTerm = params.get('q') || '';
      this.page = +params.get('page') || 1;
    });
  }
}
```

## Вложенные маршруты

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    children: [
      {
        path: '',
        component: DashboardOverviewComponent,
        pathMatch: 'full'
      },
      {
        path: 'settings',
        component: DashboardSettingsComponent
      },
      {
        path: 'profile',
        component: DashboardProfileComponent
      }
    ]
  }
];
```

```html
<!-- dashboard.component.html -->
<div class="dashboard">
  <h1>Панель управления</h1>
  <nav>
    <a routerLink="./">Обзор</a>
    <a routerLink="./settings">Настройки</a>
    <a routerLink="./profile">Профиль</a>
  </nav>
  <!-- Вложенные маршруты отображаются здесь -->
  <router-outlet></router-outlet>
</div>
```

## Защита маршрутов

### CanActivate Guard

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    if (this.authService.isAuthenticated()) {
      return true;
    }

    // Перенаправление на страницу входа
    this.router.navigate(['/login'], { 
      queryParams: { returnUrl: state.url } 
    });
    return false;
  }
}
```

### CanDeactivate Guard

```typescript
// can-deactivate.guard.ts
import { Injectable } from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { Observable } from 'rxjs';

export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({
  providedIn: 'root'
})
export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(
    component: CanComponentDeactivate
  ): Observable<boolean> | Promise<boolean> | boolean {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}
```

### Component
```typescript
// form.component.ts
import { Component } from '@angular/core';
import { CanComponentDeactivate } from './can-deactivate.guard';

@Component({
  selector: 'app-form',
  template: `
    <form #form="ngForm" (ngSubmit)="onSubmit()">
      <input name="name" [(ngModel)]="model.name" #name="ngModel" required>
      <button type="submit">Сохранить</button>
    </form>
  `
})
export class FormComponent implements CanComponentDeactivate {
  model = { name: '' };
  hasUnsavedChanges = false;

  canDeactivate(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('У вас есть несохраненные изменения. Покинуть страницу?');
    }
    return true;
  }

  onSubmit() {
    // Логика сохранения
    this.hasUnsavedChanges = false;
  }

  onModelChange() {
    this.hasUnsavedChanges = true;
  }
}
```

## Lazy Loading

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canActivate: [AuthGuard]
  },
  {
    path: 'user',
    loadChildren: () => import('./user/user.module').then(m => m.UserModule)
  }
];
```

```typescript
// admin/admin.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';
import { AdminComponent } from './admin.component';

const routes: Routes = [
  { path: '', component: AdminComponent },
  { path: 'users', component: AdminUsersComponent },
  { path: 'settings', component: AdminSettingsComponent }
];

@NgModule({
  declarations: [
    AdminComponent,
    AdminUsersComponent,
    AdminSettingsComponent
  ],
  imports: [
    CommonModule,
    RouterModule.forChild(routes)
  ]
})
export class AdminModule { }
```

## Route Resolvers

```typescript
// user.resolver.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { UserService } from './user.service';

@Injectable({
  providedIn: 'root'
})
export class UserResolver implements Resolve<any> {
  constructor(private userService: UserService) {}

  resolve(route: ActivatedRouteSnapshot): Observable<any> {
    const userId = route.paramMap.get('id');
    
    return this.userService.getUser(userId).pipe(
      catchError(error => {
        console.error('Error loading user:', error);
        return of(null); // или перенаправление на 404
      })
    );
  }
}
```

```typescript
// user.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user',
  template: `<h1>{{ user?.name }}</h1>`
})
export class UserComponent implements OnInit {
  user: any;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    // Данные уже загружены через resolver
    this.user = this.route.snapshot.data['user'];
  }
}
```

## Обработка ошибок

### Global Error Handler

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes, Router } from '@angular/router';
import { ErrorHandler } from '@angular/core';

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    // Настройки роутинга
    enableTracing: false, // для отладки
    scrollPositionRestoration: 'top',
    anchorScrolling: 'enabled'
  })],
  exports: [RouterModule]
})
export class AppRoutingModule {
  constructor(private router: Router) {
    // Глобальный обработчик ошибок маршрутизации
    this.router.errorHandler = (error: any) => {
      console.error('Routing error:', error);
      // Можно перенаправить на страницу ошибки
      this.router.navigate(['/error']);
    };
  }
}
```

### Error Route

```typescript
// app-routing.module.ts
const routes: Routes = [
  // другие маршруты...
  { path: 'error', component: ErrorComponent },
  { path: '**', redirectTo: '/error' }
];
```

```typescript
// error.component.ts
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-error',
  template: `
    <div class="error-container">
      <h1>Произошла ошибка</h1>
      <p>Запрашиваемая страница не найдена или произошла другая ошибка.</p>
      <button (click)="goHome()">Вернуться на главную</button>
    </div>
  `
})
export class ErrorComponent {
  constructor(private router: Router) {}

  goHome() {
    this.router.navigate(['/']);
  }
}
```

## Лучшие практики

### Типизированные маршруты

```typescript
// types/route-types.ts
export interface AppRoute {
  path: string;
  component: any;
  data?: {
    title: string;
    requiresAuth?: boolean;
    role?: string;
  };
}

// router.config.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';

export const APP_ROUTES: Routes = [
  {
    path: '',
    component: HomeComponent,
    data: {
      title: 'Главная страница',
      requiresAuth: false
    }
  }
];
```

### Структура файлов

```
src/
├── app/
│   ├── core/
│   │   └── guards/
│   │       ├── auth.guard.ts
│   │       ├── can-deactivate.guard.ts
│   │       └── role.guard.ts
│   ├── shared/
│   │   └── resolvers/
│   │       ├── user.resolver.ts
│   │       └── post.resolver.ts
│   ├── features/
│   │   ├── user/
│   │   │   ├── user.module.ts
│   │   │   ├── user-routing.module.ts
│   │   │   └── components/
│   │   └── admin/
│   │       ├── admin.module.ts
│   │       ├── admin-routing.module.ts
│   │       └── components/
│   ├── app-routing.module.ts
│   └── app.component.ts
```

### Route Guards Module

```typescript
// core/guards/guards.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AuthGuard } from './auth.guard';
import { RoleGuard } from './role.guard';
import { CanDeactivateGuard } from './can-deactivate.guard';

@NgModule({
  declarations: [],
  imports: [CommonModule],
  providers: [
    AuthGuard,
    RoleGuard,
    CanDeactivateGuard
  ]
})
export class GuardsModule { }
```

### Управление состоянием роутинга

```typescript
// app.component.ts
import { Component, OnInit } from '@angular/core';
import { Router, NavigationEnd } from '@angular/router';
import { filter } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  template: `
    <nav>
      <a routerLink="/">Главная</a>
      <a routerLink="/about">О нас</a>
    </nav>
    <main>
      <router-outlet></router-outlet>
    </main>
  `
})
export class AppComponent implements OnInit {
  constructor(private router: Router) {}

  ngOnInit() {
    // Отслеживание навигации
    this.router.events
      .pipe(filter(event => event instanceof NavigationEnd))
      .subscribe((event: NavigationEnd) => {
        // Логика при навигации
        console.log('Navigated to:', event.url);
        
        // Обновление мета-данных
        this.updatePageTitle(event.url);
      });
  }

  private updatePageTitle(url: string) {
    // Логика обновления заголовка страницы
    document.title = this.getTitleFromUrl(url) || 'My App';
  }

  private getTitleFromUrl(url: string): string | null {
    // Реализация получения заголовка из URL
    return null;
  }
}
```

## Миграция с Angular 2/4

### Основные изменения

```typescript
// Angular 2/4 (устаревший)
import { Router } from '@angular/router';

// Прямой доступ к queryParams
this.router.routerState.queryParams.subscribe(params => {
  // обработка
});

// Angular 10+ (современный)
import { Router } from '@angular/router';

// Доступ через route
this.route.queryParamMap.subscribe(params => {
  // обработка
});
```

## Заключение

Роутинг в Angular предоставляет мощную и гибкую систему для управления навигацией в приложении. Правильное использование защит маршрутов, lazy loading и route resolvers позволяет создавать производительные и масштабируемые приложения.

См. также: [[Клиентский-роутинг]], [[Серверный-роутинг]], [[Роутинг-в-React]], [[Роутинг-в-Vue]]