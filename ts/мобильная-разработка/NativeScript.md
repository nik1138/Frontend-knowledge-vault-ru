---
aliases: [NativeScript, NS]
tags: [mobile-development, nativescript, typescript, angular, vue]
---

# NativeScript

NativeScript - это фреймворк для создания кроссплатформенных нативных мобильных приложений с использованием JavaScript, TypeScript и Angular, Vue.js или React. В отличие от гибридных решений, NativeScript создает по-настоящему нативные приложения, используя нативные компоненты платформы.

## Основы NativeScript

NativeScript позволяет разработчикам использовать веб-технологии для создания приложений, которые компилируются в нативный код. Это обеспечивает нативную производительность и доступ к полному API платформы.

### Установка и создание проекта

```bash
npm install -g nativescript
tns create MyApp --template tns-template-blank-ts
cd MyApp
```

## Архитектура NativeScript

NativeScript состоит из нескольких ключевых компонентов:

- **Runtime** - обеспечивает выполнение JavaScript кода на нативной платформе
- **Modules** - набор нативных API, доступных через JavaScript
- **UI Framework** - система для создания пользовательского интерфейса
- **Plugins** - расширения для доступа к специфичным возможностям платформы

## Основные концепции

### Платформо-независимый код

NativeScript позволяет писать логику один раз и использовать её на iOS и Android:

```typescript
// app/home/home-page.ts
import { EventData } from 'data/observable';
import { Page } from 'ui/page';
import { HelloWorldModel } from './home-view-model';

let viewModel = new HelloWorldModel();

export function navigatingTo(args: EventData) {
    const page = <Page>args.object;
    page.bindingContext = viewModel;
}
```

### Шаблоны и привязка данных

NativeScript использует XML для описания интерфейса и поддерживает привязку данных:

```xml
<!-- app/home/home-page.xml -->
<Page xmlns="http://schemas.nativescript.org/tns.xsd" navigatingTo="navigatingTo">
  <StackLayout>
    <Label text="{{ message }}" textWrap="true" class="title"/>
    <Button text="Tap me!" tap="{{ onTap }}" class="btn btn-primary"/>
  </StackLayout>
</Page>
```

```typescript
// app/home/home-view-model.ts
import { Observable } from 'data/observable';

export class HelloWorldModel extends Observable {
  private _counter: number;
  private _message: string;

  constructor() {
    super();
    this._counter = 42;
    this.updateMessage();
  }

  get message(): string {
    return this._message;
  }

  set message(value: string) {
    if (this._message !== value) {
      this._message = value;
      this.notifyPropertyChange('message', value);
    }
  }

  public onTap() {
    this._counter--;
    this.updateMessage();
  }

  private updateMessage() {
    if (this._counter <= 0) {
      this.message = 'Hoorraaay! You unlocked the NativeScript clicker achievement!';
    } else {
      this.message = `${this._counter} taps left`;
    }
  }
}
```

## Типизация в NativeScript

TypeScript особенно полезен в NativeScript для обеспечения безопасности типов:

```typescript
// app/models/user-model.ts
export interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

// app/services/user-service.ts
import { User } from '~/models/user-model';
import { ObservableArray } from 'data/observable-array';

export class UserService {
  private users: ObservableArray<User>;
  
  constructor() {
    this.users = new ObservableArray<User>([]);
  }

  public async fetchUsers(): Promise<User[]> {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      const userData: User[] = await response.json();
      
      this.users.splice(0, this.users.length, ...userData);
      return userData;
    } catch (error) {
      console.error('Ошибка при загрузке пользователей:', error);
      throw error;
    }
  }

  public getUsers(): ObservableArray<User> {
    return this.users;
  }

  public getUserById(id: number): User | undefined {
    return this.users.filter(user => user.id === id)[0];
  }
}
```

## Работа с UI компонентами

NativeScript предоставляет широкий набор нативных компонентов:

```xml
<!-- app/components/user-list.xml -->
<Page xmlns="http://schemas.nativescript.org/tns.xsd" xmlns:lv="nativescript-ui-listview">
  <ActionBar title="Пользователи">
    <ActionItem icon="res://add" tap="onAddUser" ios.position="right" />
  </ActionBar>
  <StackLayout>
    <ActivityIndicator busy="{{ isLoading }}" />
    <lv:RadListView items="{{ users }}" itemTap="onUserTap">
      <lv:RadListView.itemTemplate>
        <StackLayout padding="16" class="item">
          <Label text="{{ name }}" class="name" textWrap="true" />
          <Label text="{{ email }}" class="email" textWrap="true" />
        </StackLayout>
      </lv:RadListView.itemTemplate>
    </lv:RadListView>
  </StackLayout>
</Page>
```

```typescript
// app/components/user-list.ts
import { EventData, Observable } from 'data/observable';
import { Page } from 'ui/page';
import { UserService } from '~/services/user-service';
import { User } from '~/models/user-model';

let userService = new UserService();
let viewModel = new Observable();

export function navigatingTo(args: EventData) {
  const page = <Page>args.object;
  
  viewModel.set('isLoading', true);
  userService.fetchUsers()
    .then(users => {
      viewModel.set('users', userService.getUsers());
      viewModel.set('isLoading', false);
    })
    .catch(error => {
      console.error('Ошибка:', error);
      viewModel.set('isLoading', false);
    });
  
  page.bindingContext = viewModel;
}

export function onUserTap(args: GestureEventData) {
  const item = args.view;
  const user = item.bindingContext as User;
  console.log(`Выбран пользователь: ${user.name}`);
}
```

## Использование Angular с NativeScript

NativeScript может использоваться с Angular для создания приложений:

```typescript
// app/app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'ns-app',
  templateUrl: './app.component.html'
})
export class AppComponent {
  title = 'NativeScript Angular App';
}
```

```html
<!-- app/app.component.html -->
<FlexboxLayout flexDirection="column" class="p-20">
  <Label [text]="title" class="h1 text-center"></Label>
  <Button text="Нажми меня" (tap)="onTap()" class="btn btn-primary"></Button>
</FlexboxLayout>
```

```typescript
// app/app.module.ts
import { NgModule, NO_ERRORS_SCHEMA } from '@angular/core';
import { NativeScriptModule } from 'nativescript-angular/nativescript.module';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    NativeScriptModule,
    AppRoutingModule
  ],
  bootstrap: [AppComponent],
  schemas: [NO_ERRORS_SCHEMA]
})
export class AppModule { }
```

## Работа с нативными возможностями

NativeScript предоставляет прямой доступ к нативным API:

```typescript
// app/services/device-service.ts
import * as platform from 'platform';
import { Device } from 'platform';

export class DeviceService {
  public getDeviceInfo(): string {
    const device: Device = platform.device;
    
    return `Платформа: ${device.os}
Версия: ${device.osVersion}
Модель: ${device.model}
Тип: ${device.deviceType}
UUID: ${device.uuid}`;
  }

  public getScreenInfo(): string {
    const screen = platform.screen;
    
    return `Ширина: ${screen.mainScreen.widthDIPs}
Высота: ${screen.mainScreen.heightDIPs}
Плотность: ${screen.mainScreen.scale}`;
  }
}
```

## Плагины NativeScript

NativeScript поддерживает широкий спектр плагинов:

```bash
# Установка плагинов
npm install nativescript-ui-listview
npm install nativescript-camera
npm install nativescript-geolocation
```

```typescript
// app/services/camera-service.ts
import * as camera from 'nativescript-camera';
import * as imagepicker from 'nativescript-imagepicker';
import { ImageAsset } from 'image-asset';

export class CameraService {
  public async takePicture(): Promise<ImageAsset> {
    if (camera.isAvailable()) {
      const options = {
        width: 300,
        height: 300,
        keepAspectRatio: true,
        saveToGallery: true
      };
      
      try {
        const imageAsset = await camera.takePicture(options);
        return imageAsset;
      } catch (error) {
        console.error('Ошибка при съемке:', error);
        throw error;
      }
    } else {
      throw new Error('Камера недоступна');
    }
  }

  public async pickImage(): Promise<ImageAsset[]> {
    const context = imagepicker.create({
      mode: 'single' // или 'multiple'
    });
    
    try {
      const selection = await context;
      return selection.entries;
    } catch (error) {
      console.error('Ошибка при выборе изображения:', error);
      throw error;
    }
  }
}
```

## Стилизация и темизация

NativeScript использует CSS для стилизации:

```css
/* app/app.css */
.title {
  font-size: 30;
  horizontal-align: center;
  margin: 20;
}

.btn-primary {
  font-size: 18;
  padding: 15;
  background-color: #3498db;
  color: white;
  border-radius: 5;
}

.item {
  border-bottom-width: 1;
  border-bottom-color: #eee;
}

.name {
  font-size: 18;
  font-weight: bold;
}

.email {
  font-size: 14;
  color: #666;
}
```

Также можно использовать NativeScript Theme:

```bash
tns install nativescript-theme-core
```

```css
@import '~nativescript-theme-core/css/core.css';

.btn {
  @include btn-primary;
}
```

## Практические советы

- Используйте `tns preview` для быстрой проверки изменений на устройстве
- Оптимизируйте изображения и ресурсы для мобильных устройств
- Используйте `ObservableArray` для списков с динамическими данными
- Обрабатывайте жизненный цикл приложения (приложение в фоне/на переднем плане)
- Используйте `setTimeout` и `setInterval` осторожно, чтобы избежать утечек памяти
- Проверяйте доступность устройств перед их использованием (камера, геолокация и т.д.)

## Сборка и деплой

```bash
# Проверка доступных устройств
tns device

# Запуск на эмуляторе iOS
tns run ios --emulator

# Запуск на эмуляторе Android
tns run android --emulator

# Запуск на подключенном устройстве
tns run ios
tns run android

# Сборка для релиза
tns build ios --release --for-device
tns build android --release
```

## Связанные темы

- [[Mobile-UI-Frameworks]]
- [[Мобильные-паттерны]]
- [[Оптимизация-для-мобильных-устройств]]
- [[TypeScript-в-мобильной-разработке]]
- [[Angular-NativeScript]]