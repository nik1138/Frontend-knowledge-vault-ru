---
aliases: ["Медиа запросы", "CSS медиа запросы", "Адаптивный дизайн"]
tags: [html, css, multimedia, responsive, media-queries]
---

# Медиа-запросы-в-HTML

## Введение

Медиа-запросы - это мощный инструмент CSS, который позволяет применять разные стили в зависимости от характеристик устройства пользователя, таких как ширина экрана, ориентация, разрешение и другие параметры. В контексте HTML мультимедиа, медиа-запросы играют ключевую роль в создании адаптивного и доступного пользовательского опыта.

## Основы медиа-запросов

### Синтаксис
Медиа-запросы определяются с помощью правила `@media` в CSS:

```css
@media media-type and (media-feature) {
  /* CSS правила */
}
```

### Типы медиа
- `all` - все устройства (по умолчанию)
- `print` - принтеры
- `screen` - экраны компьютеров, планшетов, телефонов
- `speech` - речевые синтезаторы

## Применение к мультимедиа элементам

### Адаптация размеров видео и аудио
```css
/* Основные стили для видео */
video, audio {
  width: 100%;
  height: auto;
  max-width: 800px;
}

/* На мобильных устройствах уменьшаем размер */
@media screen and (max-width: 768px) {
  video, audio {
    max-width: 100%;
    height: auto;
  }
}

/* В портретной ориентации */
@media screen and (orientation: portrait) {
  video {
    width: 100%;
    height: 60vh;
  }
}

/* В ландшафтной ориентации */
@media screen and (orientation: landscape) {
  video {
    width: 80vw;
    height: 45vw;
  }
}
```

### Управление воспроизведением на основе устройства
```css
/* Отключение автовоспроизведения на мобильных устройствах */
@media screen and (max-width: 768px) {
  video[autoplay] {
    display: none;
  }
  
  .mobile-video-controls {
    display: block;
  }
}

/* Показ специальных элементов управления */
@media screen and (min-width: 1024px) {
  .desktop-controls {
    display: flex;
  }
}
```

## Практические примеры использования

### Адаптивное видео встроенное в HTML
```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Адаптивное видео</title>
  <style>
    .video-container {
      position: relative;
      width: 100%;
      height: 0;
      padding-bottom: 56.25%; /* 16:9 */
      overflow: hidden;
    }
    
    .video-container video {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
    
    /* На мобильных устройствах */
    @media screen and (max-width: 768px) {
      .video-container {
        padding-bottom: 75%; /* 4:3 для мобильных */
      }
      
      .video-title {
        font-size: 1.2em;
        margin: 10px 0;
      }
    }
    
    /* На планшетах */
    @media screen and (min-width: 769px) and (max-width: 1024px) {
      .video-container {
        padding-bottom: 56.25%; /* 16:9 */
      }
    }
    
    /* На больших экранах */
    @media screen and (min-width: 1025px) {
      .video-container {
        max-width: 1200px;
        margin: 0 auto;
      }
    }
  </style>
</head>
<body>
  <div class="video-container">
    <video controls>
      <source src="video.mp4" type="video/mp4">
      <source src="video.webm" type="video/webm">
      Ваш браузер не поддерживает видео элемент.
    </video>
  </div>
  <h2 class="video-title">Название видео</h2>
</body>
</html>
```

### Адаптивные аудио плееры
```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Адаптивный аудио плеер</title>
  <style>
    .audio-player {
      width: 100%;
      max-width: 600px;
      margin: 20px auto;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 8px;
      background-color: #f9f9f9;
    }
    
    .audio-player audio {
      width: 100%;
    }
    
    /* На мобильных устройствах */
    @media screen and (max-width: 480px) {
      .audio-player {
        padding: 10px;
        margin: 10px;
      }
      
      .audio-info {
        font-size: 0.9em;
      }
    }
    
    /* В темном режиме */
    @media (prefers-color-scheme: dark) {
      .audio-player {
        background-color: #2c2c2c;
        color: white;
        border-color: #555;
      }
    }
    
    /* Если пользователь предпочитает уменьшенную анимацию */
    @media (prefers-reduced-motion: reduce) {
      .audio-player {
        transition: none;
      }
    }
  </style>
</head>
<body>
  <div class="audio-player">
    <audio controls>
      <source src="audio.mp3" type="audio/mpeg">
      <source src="audio.ogg" type="audio/ogg">
      Ваш браузер не поддерживает аудио элемент.
    </audio>
    <div class="audio-info">
      <h3>Название аудио</h3>
      <p>Исполнитель: Автор</p>
    </div>
  </div>
</body>
</html>
```

## Продвинутые медиа-запросы

### Медиа-запросы для производительности
```css
/* Ограничение использования тяжелых медиа на слабых устройствах */
@media (prefers-reduced-data: reduce) {
  video[preload="auto"] {
    preload: metadata;
  }
  
  .background-video {
    display: none;
  }
}

/* Для устройств с высоким разрешением экрана */
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .high-res-video {
    src: url('video@2x.mp4');
  }
}
```

### Медиа-запросы для доступности
```css
/* Для пользователей, предпочитающих уменьшенную анимацию */
@media (prefers-reduced-motion: reduce) {
  video[autoplay] {
    display: none;
  }
  
  .animated-media-controls {
    animation: none;
  }
}

/* Для пользователей, использующих темный режим */
@media (prefers-color-scheme: dark) {
  .media-container {
    background-color: #1a1a1a;
    color: white;
  }
  
  video::-webkit-media-controls {
    background-color: #333;
  }
}

/* Для пользователей, предпочитающий высокую контрастность */
@media (prefers-contrast: high) {
  .media-controls {
    border: 2px solid #000;
    background-color: #fff;
    color: #000;
  }
}
```

## Практические рекомендации для российских реалий

### Учет особенностей российского рынка
- **Скорость интернета**: В различных регионах России скорость интернета может значительно отличаться
- **Устройства пользователей**: Учет широкого спектра устройств, включая старые модели
- **Языковые особенности**: Использование русскоязычных интерфейсов и меток

### Адаптация под российские условия
```css
/* Для регионов с низкой скоростью интернета */
@media screen and (max-width: 768px) and (max-device-width: 768px) {
  /* Уменьшение качества видео для мобильных */
  .video-player {
    background-image: url('low-res-preview.jpg');
  }
  
  /* Скрытие тяжелых элементов */
  .heavy-media-element {
    display: none;
  }
}

/* Учет времени суток (для темного режима) */
@media (prefers-color-scheme: dark) {
  .media-container {
    background-color: #1e1e1e;
    color: #ffffff;
  }
}
```

### Совместимость с российскими браузерами
- Учитывайте особенности Яндекс.Браузера, Mail.Ru Agent и других локальных решений
- Тестируйте медиа-запросы в различных браузерах
- Обеспечьте резервные стили для старых браузеров

### SEO и семантика
- Используйте семантически правильные теги для мультимедиа
- Обеспечьте альтернативный контент для поисковых роботов
- Добавляйте субтитры и транскрипции

## Современные подходы

### Container Queries (экспериментально)
```css
/* При поддержке container queries */
@container (min-width: 300px) {
  .media-element {
    flex-direction: row;
  }
}

@container (max-width: 299px) {
  .media-element {
    flex-direction: column;
  }
}
```

### Условные стили в HTML
```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Условные стили</title>
  <style>
    /* Стандартные стили */
    .responsive-media {
      width: 100%;
      height: auto;
    }
    
    /* Условия для разных устройств */
    @media screen and (max-width: 576px) {
      .mobile-styles {
        font-size: 14px;
      }
    }
    
    @media screen and (min-width: 577px) and (max-width: 768px) {
      .tablet-styles {
        font-size: 16px;
      }
    }
    
    @media screen and (min-width: 769px) {
      .desktop-styles {
        font-size: 18px;
      }
    }
  </style>
</head>
<body>
  <div class="responsive-media">
    <video controls class="mobile-styles tablet-styles desktop-styles">
      <source src="video.mp4" type="video/mp4">
      Ваш браузер не поддерживает видео элемент.
    </video>
  </div>
</body>
</html>
```

## Инструменты и отладка

### Использование DevTools
- Chrome DevTools: вкладка Device Toolbar для тестирования разных устройств
- Firefox Responsive Design Mode
- Safari Responsive Design Mode

### Тестирование на разных устройствах
```css
/* Тестирование специфических устройств */
/* iPhone */
@media screen and (device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2) {
  .iphone-specific {
    /* Стили для iPhone 6/7/8 */
  }
}

/* iPad */
@media screen and (device-width: 768px) and (device-height: 1024px) and (-webkit-device-pixel-ratio: 2) {
  .ipad-specific {
    /* Стили для iPad */
  }
}
```

## Заключение

Медиа-запросы в HTML играют важную роль в создании адаптивного и доступного мультимедийного контента. При правильном использовании они позволяют создать оптимальный пользовательский опыт независимо от устройства или условий просмотра. В условиях российского рынка особенно важно учитывать разнообразие устройств, скоростей интернета и предпочтений пользователей.

См. также:
- [[Адаптивный дизайн]]
- [[Lazy-loading-медиа]]
- [[Оптимизация-изображений]]
- [[Производительность веб-страниц]]
- [[Доступность веб-контента]]
