---
aliases: [angular-modules-guide, angular-module-architecture]
tags: [angular, typescript, modules, architecture, frontend]
---

# Модули Angular

Модули Angular - это контейнеры для частей приложения, которые определяют, какие компоненты, директивы, пайпы и сервисы принадлежат к определенной функциональности. Модули помогают организовать приложение и управлять зависимостями между различными частями кода.

## Основы Angular модулей

### Базовая структура модуля

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    // Компоненты, директивы и пайпы, принадлежащие этому модулю
    AppComponent
  ],
  imports: [
    // Модули, от которых зависит этот модуль
    BrowserModule
  ],
  providers: [
    // Сервисы, доступные для внедрения зависимостей
  ],
  bootstrap: [
    // Корневой компонент приложения
    AppComponent
  ]
})
export class AppModule { }
```

## Типы модулей

### Root Module

Корневой модуль - это точка входа в приложение Angular:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';
import { AppRoutingModule } from './app-routing.module';
import { CoreModule } from './core/core.module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    AppRoutingModule,
    CoreModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Feature Module

Модуль функций используется для организации связанной функциональности:

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';

import { UserComponent } from './user.component';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { UserService } from './user.service';

const routes: Routes = [
  { path: '', component: UserComponent },
  { path: ':id', component: UserProfileComponent }
];

@NgModule({
  declarations: [
    UserComponent,
    UserProfileComponent
  ],
  imports: [
    CommonModule,
    RouterModule.forChild(routes)
  ],
  providers: [
    UserService
  ]
})
export class UserModule { }
```

## SharedModule

Общий модуль для компонентов, директив и пайпов, которые используются в нескольких других модулях:

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

import { HighlightDirective } from './highlight.directive';
import { CapitalizePipe } from './capitalize.pipe';
import { LoadingSpinnerComponent } from './loading-spinner/loading-spinner.component';

@NgModule({
  declarations: [
    HighlightDirective,
    CapitalizePipe,
    LoadingSpinnerComponent
  ],
  imports: [
    CommonModule,
    FormsModule
  ],
  exports: [
    // Экспортируем элементы, чтобы они были доступны в других модулях
    HighlightDirective,
    CapitalizePipe,
    LoadingSpinnerComponent,
    CommonModule,
    FormsModule
  ]
})
export class SharedModule { }
```

## CoreModule

Модуль для сервисов и компонентов singleton, которые должны быть доступны только один раз в приложении:

```typescript
import { NgModule, Optional, SkipSelf } from '@angular/core';
import { CommonModule } from '@angular/common';

import { throwIfAlreadyLoaded } from './module-import-guard';
import { DataService } from './data.service';
import { AuthService } from './auth.service';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  providers: [
    DataService,
    AuthService
  ]
})
export class CoreModule {
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    throwIfAlreadyLoaded(parentModule, 'CoreModule');
  }
}

// Файл module-import-guard.ts
export function throwIfAlreadyLoaded(parentModule: any, moduleName: string) {
  if (parentModule) {
    throw new Error(`${moduleName} has already been loaded. Import Core modules in the AppModule only.`);
  }
}
```

## Lazy Loading Modules

Отложенная загрузка модулей для улучшения производительности:

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  },
  {
    path: 'user',
    loadChildren: () => import('./user/user.module').then(m => m.UserModule)
  },
  { path: '', redirectTo: '/home', pathMatch: 'full' }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

// admin.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';

import { AdminComponent } from './admin.component';
import { AdminGuard } from './admin.guard';

const routes: Routes = [
  {
    path: '',
    component: AdminComponent,
    canActivate: [AdminGuard]
  }
];

@NgModule({
  declarations: [
    AdminComponent
  ],
  imports: [
    CommonModule,
    RouterModule.forChild(routes)
  ],
  providers: [
    AdminGuard
  ]
})
export class AdminModule { }
```

##.forChild() и .forRoot() паттерны

### Модуль с паттерном forRoot/forChild

```typescript
import { NgModule, ModuleWithProviders } from '@angular/core';
import { CommonModule } from '@angular/common';

import { ConfigService } from './config.service';
import { FeatureComponent } from './feature.component';

@NgModule({
  declarations: [FeatureComponent],
  imports: [CommonModule],
  exports: [FeatureComponent]
})
export class FeatureModule {
  // Паттерн forRoot - используется для конфигурации модуля на уровне приложения
  static forRoot(config: FeatureConfig): ModuleWithProviders<FeatureModule> {
    return {
      ngModule: FeatureModule,
      providers: [
        ConfigService,
        { provide: 'FEATURE_CONFIG', useValue: config }
      ]
    };
  }

  // Паттерн forChild - используется для импорта модуля в дочерние модули
  static forChild(): ModuleWithProviders<FeatureModule> {
    return {
      ngModule: FeatureModule
    };
  }
}

export interface FeatureConfig {
  apiUrl: string;
  timeout: number;
}
```

### Использование паттернов

```typescript
// app.module.ts
import { FeatureModule } from './feature/feature.module';

@NgModule({
  imports: [
    FeatureModule.forRoot({
      apiUrl: 'https://api.example.com',
      timeout: 5000
    })
  ]
})
export class AppModule {}

// feature-routing.module.ts
import { FeatureModule } from './feature.module';

@NgModule({
  imports: [
    FeatureModule.forChild() // Используется в дочерних модулях
  ]
})
export class FeatureRoutingModule {}
```

## Модули с провайдерами

### Область видимости провайдеров

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { UserService } from './user.service';
import { ProfileService } from './profile.service';
import { UserComponent } from './user.component';

@NgModule({
  declarations: [UserComponent],
  imports: [CommonModule],
  providers: [
    // Эти сервисы будут созданы для каждого экземпляра модуля
    UserService,
    {
      provide: 'API_URL',
      useValue: 'https://api.example.com/v1'
    }
  ],
  exports: [UserComponent]
})
export class UserFeatureModule { }

// В другом модуле
import { UserFeatureModule } from './user-feature/user-feature.module';

@NgModule({
  imports: [UserFeatureModule], // UserService будет доступен только в этом модуле
  // ...
})
export class AnotherModule { }
```

## Утилиты для работы с модулями

### Генератор модулей

```typescript
// utils/module-generator.ts
import { NgModule, Type } from '@angular/core';
import { CommonModule } from '@angular/common';

export interface ModuleConfig {
  declarations: Type<any>[];
  imports?: any[];
  exports?: Type<any>[];
  providers?: any[];
}

export function createModule(config: ModuleConfig): NgModule {
  const moduleConfig: NgModule = {
    declarations: config.declarations,
    imports: [CommonModule, ...(config.imports || [])],
    exports: [...(config.exports || [])],
    providers: config.providers || []
  };
  
  return NgModule(moduleConfig);
}
```

## Лучшие практики

### 1. Организация по функциональности

```typescript
// Вместо большого модуля
// app.module.ts
@NgModule({
  declarations: [
    // 50+ компонентов
  ],
  // ...
})
export class AppModule { }

// Лучше использовать модули по функциональности
// user.module.ts
@NgModule({
  declarations: [
    UserComponent,
    UserProfileComponent,
    UserSettingsComponent
  ],
  // ...
})
export class UserModule { }

// product.module.ts
@NgModule({
  declarations: [
    ProductComponent,
    ProductDetailComponent,
    ProductListComponent
  ],
  // ...
})
export class ProductModule { }
```

### 2. Использование правильных областей видимости

```typescript
// CoreModule - только для singleton сервисов
@NgModule({
  providers: [AuthService, DataService] // Только сервисы!
})
export class CoreModule { }

// SharedModule - для компонентов, директив, пайпов
@NgModule({
  declarations: [CommonComponent, CommonDirective, CommonPipe],
  exports: [CommonComponent, CommonDirective, CommonPipe]
}) 
export class SharedModule { }
```

### 3. Защита от повторного импорта CoreModule

```typescript
@NgModule({
  // ...
})
export class CoreModule {
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error('CoreModule is already loaded. Import it in the AppModule only');
    }
  }
}
```

## Связанные темы

- [[Декораторы-в-Angular]]
- [[Типизация-сервисов]]
- [[Типизация-компонентов]]
- [[Инъекция-зависимостей]]

## Ключевые выводы

- Модули Angular помогают организовать приложение по функциональным областям
- Существуют различные типы модулей: Root, Feature, Shared, Core
- Правильное использование lazy loading улучшает производительность
- Паттерны forRoot/forChild позволяют гибко настраивать модули
- Следование лучшим практикам делает архитектуру приложения более предсказуемой и поддерживаемой