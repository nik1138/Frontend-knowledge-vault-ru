---
aliases: [Ionic Framework, Ionic]
tags: [mobile-development, ionic, typescript, capacitor, cordova]
---

# Ionic

Ionic - это фреймворк для создания гибридных мобильных приложений с использованием веб-технологий: HTML, CSS и JavaScript/TypeScript. Он предоставляет богатую библиотеку компонентов, которые имитируют нативный внешний вид iOS и Android.

## Основы Ionic

Ionic может использоваться с различными фреймворками: Angular, React или Vue. Наиболее распространенный подход - использование с Angular или React.

### Установка и создание проекта

```bash
npm install -g @ionic/cli
ionic start myApp tabs --type=react
cd myApp
ionic capacitor add android
ionic capacitor add ios
```

## Архитектура Ionic

Ionic построен на веб-технологиях и использует Capacitor или Cordova для доступа к нативным возможностям устройства:

- **Capacitor** - современная платформа для запуска веб-приложений на мобильных устройствах
- **Cordova** - более старая платформа с широкой поддержкой плагинов

## Компоненты Ionic

Ionic предоставляет богатую библиотеку компонентов:

```typescript
import React, { useState } from 'react';
import { 
  IonApp, 
  IonContent, 
  IonHeader, 
  IonTitle, 
  IonToolbar,
  IonButton,
  IonLabel,
  IonItem,
  IonList,
  IonInput,
  IonAlert
} from '@ionic/react';

const App: React.FC = () => {
  const [name, setName] = useState<string>('');
  const [showAlert, setShowAlert] = useState<boolean>(false);

  return (
    <IonApp>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Приложение Ionic</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent className="ion-padding">
        <IonList>
          <IonItem>
            <IonLabel position="stacked">Имя</IonLabel>
            <IonInput 
              value={name} 
              placeholder="Введите ваше имя"
              onIonChange={e => setName(e.detail.value!)}
            />
          </IonItem>
          <IonItem>
            <IonButton 
              expand="block" 
              onClick={() => setShowAlert(true)}
            >
              Приветствовать
            </IonButton>
          </IonItem>
        </IonList>
        
        <IonAlert
          isOpen={showAlert}
          onDidDismiss={() => setShowAlert(false)}
          header={'Приветствие'}
          message={`Привет, ${name}!`}
          buttons={['OK']}
        />
      </IonContent>
    </IonApp>
  );
};

export default App;
```

## Типизация в Ionic

TypeScript особенно полезен в Ionic для обеспечения безопасности типов:

```typescript
import React, { useState, useEffect } from 'react';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent,
  IonList,
  IonItem,
  IonLabel,
  IonNote
} from '@ionic/react';
import { Plugins } from '@capacitor/core';

interface User {
  id: number;
  name: string;
  email: string;
}

const UsersPage: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        // Имитация API-запроса
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        const userData: User[] = await response.json();
        setUsers(userData);
      } catch (error) {
        console.error('Ошибка при загрузке пользователей:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (loading) {
    return (
      <IonPage>
        <IonHeader>
          <IonToolbar>
            <IonTitle>Пользователи</IonTitle>
          </IonToolbar>
        </IonHeader>
        <IonContent className="ion-padding">
          Загрузка...
        </IonContent>
      </IonPage>
    );
  }

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Пользователи</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent className="ion-padding">
        <IonList>
          {users.map(user => (
            <IonItem key={user.id}>
              <IonLabel>
                <h2>{user.name}</h2>
                <p>{user.email}</p>
              </IonLabel>
              <IonNote slot="end">{user.id}</IonNote>
            </IonItem>
          ))}
        </IonList>
      </IonContent>
    </IonPage>
  );
};

export default UsersPage;
```

## Работа с нативными возможностями

Capacitor позволяет легко получить доступ к нативным возможностям устройства:

```typescript
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';
import { Filesystem, Directory } from '@capacitor/filesystem';
import { Device } from '@capacitor/device';

interface Photo {
  filepath: string;
  webviewPath?: string;
}

const usePhotoGallery = () => {
  const [photos, setPhotos] = useState<Photo[]>([]);

  const takePhoto = async () => {
    const cameraPhoto = await Camera.getPhoto({
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
      quality: 90
    });

    const fileName = new Date().getTime() + '.jpeg';
    const savedFileImage = await savePhoto(cameraPhoto, fileName);

    const newPhotos = [savedFileImage, ...photos];
    setPhotos(newPhotos);
  };

  const savePhoto = async (photo: any, fileName: string): Promise<Photo> => {
    // Преобразуем изображение в формат blob
    const response = await fetch(photo.webPath!);
    const blob = await response.blob();

    // Сохраняем изображение в файловую систему
    const savedFile = await Filesystem.writeFile({
      path: fileName,
      data: blob,
      directory: Directory.Data
    });

    return {
      filepath: savedFile.uri,
      webviewPath: photo.webPath
    };
  };

  return {
    photos,
    takePhoto
  };
};
```

## Навигация в Ionic

Ionic предоставляет собственные компоненты навигации:

```typescript
import { Redirect, Route } from 'react-router-dom';
import { 
  IonApp, 
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs,
  IonLabel
} from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { 
  home, 
  person, 
  settings 
} from 'ionicons/icons';

import HomePage from './pages/HomePage';
import ProfilePage from './pages/ProfilePage';
import SettingsPage from './pages/SettingsPage';

const App: React.FC = () => {
  return (
    <IonApp>
      <IonReactRouter>
        <IonTabs>
          <IonRouterOutlet>
            <Route exact path="/home">
              <HomePage />
            </Route>
            <Route exact path="/profile">
              <ProfilePage />
            </Route>
            <Route path="/settings">
              <SettingsPage />
            </Route>
            <Route exact path="/">
              <Redirect to="/home" />
            </Route>
          </IonRouterOutlet>
          <IonTabBar slot="bottom">
            <IonTabButton tab="home" href="/home">
              <IonLabel>Главная</IonLabel>
            </IonTabButton>
            <IonTabButton tab="profile" href="/profile">
              <IonLabel>Профиль</IonLabel>
            </IonTabButton>
            <IonTabButton tab="settings" href="/settings">
              <IonLabel>Настройки</IonLabel>
            </IonTabButton>
          </IonTabBar>
        </IonTabs>
      </IonReactRouter>
    </IonApp>
  );
};

export default App;
```

## Плагины Ionic

Ionic предоставляет доступ к широкому спектру плагинов для нативных возможностей:

```typescript
// Установка плагинов
// npm install @capacitor/camera
// npm install @capacitor/geolocation
// npm install @capacitor/filesystem

import { Geolocation } from '@capacitor/geolocation';

interface Location {
  latitude: number;
  longitude: number;
}

const useLocation = () => {
  const [location, setLocation] = useState<Location | null>(null);
  const [error, setError] = useState<string | null>(null);

  const getCurrentPosition = async () => {
    try {
      const coordinates = await Geolocation.getCurrentPosition();
      setLocation({
        latitude: coordinates.coords.latitude,
        longitude: coordinates.coords.longitude
      });
    } catch (err) {
      setError('Не удалось получить местоположение');
      console.error('Ошибка геолокации:', err);
    }
  };

  return {
    location,
    error,
    getCurrentPosition
  };
};
```

## Стилизация и темизация

Ionic использует CSS-переменные для кастомизации:

```scss
// variables.scss
:root {
  --ion-color-primary: #3880ff;
  --ion-color-primary-rgb: 56, 128, 255;
  --ion-color-primary-contrast: #ffffff;
  --ion-color-primary-contrast-rgb: 255, 255, 255;
  --ion-color-primary-shade: #3171e0;
  --ion-color-primary-tint: #4c8dff;
}

.ios .header-md, .md .header-ios {
  --background: #f8f8f8;
  --color: #333;
}
```

## Практические советы

- Используйте `ion-content` для основного контента, чтобы обеспечить правильное позиционирование под навигационной панелью
- Всегда обрабатывайте ошибки при работе с нативными возможностями
- Используйте `virtual scroll` для списков с большим количеством элементов
- Оптимизируйте изображения и ресурсы для мобильных устройств
- Тестируйте приложение на разных устройствах и платформах

## Сборка и деплой

```bash
# Сборка для веба
ionic build

# Добавление платформ
ionic capacitor add android
ionic capacitor add ios

# Синхронизация изменений
ionic capacitor sync

# Открытие нативного проекта
ionic capacitor open android
ionic capacitor open ios

# Создание сборки
ionic capacitor run android --release
ionic capacitor run ios --release
```

## Связанные темы

- [[Capacitor]]
- [[Cordova]]
- [[Мобильные-паттерны]]
- [[Оптимизация-для-мобильных-устройств]]
- [[TypeScript-в-мобильной-разработке]]