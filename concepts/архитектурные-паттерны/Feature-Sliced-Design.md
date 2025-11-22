---
aliases: ["FSD", "Feature-Sliced Design"]
tags: ["#архитектурный-паттерн", "#frontend", "#scalability", "#modularity"]
---

# Feature-Sliced Design (FSD)

## Обзор

Feature-Sliced Design (FSD) - это архитектурный подход к структурированию frontend-приложений, разработанный командой [Фина́м] и популяризированный в русскоязычном сообществе. Он фокусируется на разделении приложения на функциональные "слои" и "срезы", обеспечивая масштабируемость, предсказуемость и удобство поддержки.

## Основные принципы

### 1. Разделение по слоям
- Каждый слой имеет свою ответственность
- Слои строго разделены и не пересекаются
- Зависимости идут сверху вниз

### 2. Разделение по срезам
- Функциональность разделяется на независимые "срезы"
- Каждый срез реализует конкретную бизнес-функцию
- Срезы могут использовать общие слои

### 3. Единые правила именования
- Стандартизированная структура проекта
- Легкость в ориентации и понимании кода

## Структура слоев

```
┌─────────────────────────────────────┐
│              App Layer              │
├─────────────────────────────────────┤
│             Process Layer           │
├─────────────────────────────────────┤
│             Page Layer              │
├─────────────────────────────────────┤
│            Feature Layer            │
├─────────────────────────────────────┤
│             Widget Layer            │
├─────────────────────────────────────┤
│             Shared Layer            │
└─────────────────────────────────────┘
```

### Shared Layer (Общий слой)
- **Ответственность**: Утилиты, общие компоненты, типы, конфигурации
- **Примеры**: UI-компоненты, сеттинг-файлы, библиотеки, хелперы
- **Правила**: Не зависит от других слоев

### Widget Layer (Виджетный слой)
- **Ответственность**: Составные блоки, состоящие из нескольких фич
- **Примеры**: Блоки навигации, профиль пользователя, панель управления
- **Правила**: Может использовать фичи и shared

### Feature Layer (Фичевый слой)
- **Ответственность**: Бизнес-функции, реализующие конкретную логику
- **Примеры**: Авторизация, добавление поста, лайк, комментирование
- **Правила**: Может использовать shared и другие фичи (редко)

### Page Layer (Страничный слой)
- **Ответственность**: Страницы приложения, компоновка фич
- **Примеры**: Главная страница, профиль, настройки
- **Правила**: Может использовать фичи, виджеты и shared

### Process Layer (Процессный слой)
- **Ответственность**: Сложные сценарии, требующие координации нескольких страниц
- **Примеры**: Онбординг, оформление заказа
- **Правила**: Может использовать страницы, фичи и shared

### App Layer (Прикладной слой)
- **Ответственность**: Скелет приложения, маршрутизация, настройка
- **Примеры**: Роутинг, провайдеры, стартовые настройки
- **Правила**: Может использовать все остальные слои

## Пример структуры проекта

```
src/
├── app/                    # App Layer
│   ├── providers/
│   ├── router/
│   └── config/
├── processes/              # Process Layer
│   └── onboarding/
├── pages/                  # Page Layer
│   ├── main/
│   ├── profile/
│   └── settings/
├── widgets/                # Widget Layer
│   ├── header/
│   ├── sidebar/
│   └── user-card/
├── features/               # Feature Layer
│   ├── auth/
│   ├── post/
│   ├── like/
│   └── comment/
└── shared/                 # Shared Layer
    ├── ui/
    ├── lib/
    ├── types/
    └── config/
```

## Пример реализации фичи

### Структура фичи "лайк" (/src/features/like/)

```
like/
├── model/
│   ├── selectors/
│   ├── slices/
│   └── types/
├── lib/
├── ui/
│   └── like-button/
└── index.ts
```

### Пример кода фичи

```javascript
// src/features/like/model/types.ts
export interface Like {
  id: string;
  targetId: string;
  targetType: 'post' | 'comment';
  userId: string;
  createdAt: string;
}

// src/features/like/model/slices/likeSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface LikeState {
  likes: Record<string, Like[]>;
  loading: boolean;
  error: string | null;
}

const initialState: LikeState = {
  likes: {},
  loading: false,
  error: null,
};

export const likeSlice = createSlice({
  name: 'like',
  initialState,
  reducers: {
    likeRequest: (state, action: PayloadAction<{ targetId: string, targetType: string }>) => {
      state.loading = true;
      state.error = null;
    },
    likeSuccess: (state, action: PayloadAction<Like>) => {
      const { targetId, targetType } = action.payload;
      const key = `${targetType}_${targetId}`;
      
      if (!state.likes[key]) {
        state.likes[key] = [];
      }
      
      state.likes[key].push(action.payload);
      state.loading = false;
    },
    likeFailure: (state, action: PayloadAction<string>) => {
      state.loading = false;
      state.error = action.payload;
    },
  },
});

export const { likeRequest, likeSuccess, likeFailure } = likeSlice.actions;
export default likeSlice.reducer;

// src/features/like/ui/like-button/LikeButton.tsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { likeRequest } from '../../model/slices/likeSlice';
import { selectLikesForTarget } from '../../model/selectors/selectors';

interface LikeButtonProps {
  targetId: string;
  targetType: 'post' | 'comment';
  userId: string;
}

export const LikeButton: React.FC<LikeButtonProps> = ({ targetId, targetType, userId }) => {
  const dispatch = useDispatch();
  const likes = useSelector(state => selectLikesForTarget(state, targetId, targetType));
  const userLiked = likes.some(like => like.userId === userId);
  
  const handleClick = () => {
    if (!userLiked) {
      dispatch(likeRequest({ targetId, targetType }));
    }
  };
  
  return (
    <button 
      onClick={handleClick}
      className={`like-button ${userLiked ? 'liked' : ''}`}
    >
      ❤️ {likes.length}
    </button>
  );
};

// src/features/like/index.ts
export { LikeButton } from './ui/like-button/LikeButton';
export { likeReducer } from './model/slices/likeSlice';
```

## Преимущества

- **Масштабируемость**: Легко добавлять новые функции
- **Поддерживаемость**: Ясная структура облегчает сопровождение
- **Предсказуемость**: Единые правила организации кода
- **Тестируемость**: Фичи изолированы и легко тестируются
- **Командная разработка**: Понятная структура для командной работы

## Недостатки

- **Сложность для маленьких проектов**: Может быть избыточной
- **Кривая обучения**: Требует понимания архитектурных принципов
- **Бойлерплейт**: Требует создания множества файлов и папок

## Когда использовать

- В проектах среднего и крупного размера
- В командах, где важна единая архитектура
- Когда планируется долгосрочная поддержка и развитие
- В проектах с высокой сложностью функционала

## Сравнение с другими подходами

| Критерий | FSD | MVC | Clean Architecture |
|----------|-----|-----|-------------------|
| Масштабируемость | Высокая | Средняя | Высокая |
| Сложность | Средняя | Низкая | Высокая |
| Подходящие проекты | Средние-крупные | Любые | Средние-крупные |
| Зависимости | Строгие | Слабые | Строгие |

## Связанные концепции

- [[Архитектурные-паттерны]] - общее понятие о паттернах архитектуры
- [[Компонентная-архитектура]] - подход к структурированию UI
- [[Модульность]] - принцип разделения кода на модули
- [[MVC-паттерн]] - классический архитектурный паттерн
- [[Clean-архитектура]] - более общая архитектурная концепция
- [[DDD]] - подход к проектированию, используемый в FSD

## Заключение

Feature-Sliced Design предоставляет современный подход к архитектуре frontend-приложений, особенно эффективный для сложных и долгосрочных проектов. Он сочетает в себе идеи модульности, разделения ответственностей и единообразия, что делает его мощным инструментом для создания масштабируемых приложений.