---
aliases: ["Отложенная загрузка медиа", "Ленивая загрузка", "Lazy loading"]
tags: [html, multimedia, performance, optimization, loading]
---

# Lazy-loading-медиа

## Введение

Lazy loading (ленивая загрузка) - это техника, при которой медиа-элементы загружаются только тогда, когда они становятся видимыми для пользователя. Это важная техника оптимизации производительности, особенно в условиях российского интернета, где скорость соединения может быть ограничена, а стоимость трафика - значительной.

## Принципы работы lazy loading

### Основная концепция
Lazy loading откладывает загрузку медиа-элементов до тех пор, пока они не понадобятся пользователю. Это уменьшает начальную нагрузку страницы и экономит трафик.

### Преимущества
- **Улучшение производительности**: Снижение времени загрузки страницы
- **Экономия трафика**: Пользователи загружают только те медиа, которые просматривают
- **Экономия ресурсов**: Меньше нагрузки на процессор и память
- **Улучшение пользовательского опыта**: Более быстрая загрузка видимого контента

## Реализация lazy loading в HTML

### Атрибут loading

#### Нативная реализация
HTML5 предоставляет встроенный атрибут `loading` для элементов `<img>` и `<iframe>`:

```html
<img src="image.jpg" alt="Описание" loading="lazy" width="800" height="600">
<iframe src="video.html" loading="lazy"></iframe>
```

#### Значения атрибута loading
- `auto`: Браузер решает, использовать ли lazy loading (по умолчанию)
- `lazy`: Включает lazy loading для элемента
- `eager`: Отключает lazy loading, элемент загружается сразу

### Совместимость
- Chrome 76+
- Firefox 75+
- Edge 79+
- Safari 15.4+

Для старых браузеров рекомендуется использовать JavaScript-полифиллы или альтернативные решения.

## Lazy loading изображений

### Простая реализация
```html
<img src="placeholder.jpg" data-src="actual-image.jpg" class="lazy" alt="Описание" width="800" height="600">
```

### Использование Intersection Observer API
```javascript
const images = document.querySelectorAll('img[data-src]');

const imageObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.remove('lazy');
      observer.unobserve(img);
    }
  });
});

images.forEach(img => imageObserver.observe(img));
```

### Пример с WebP и резервным вариантом
```html
<picture class="lazy-picture">
  <source data-src="image.webp" type="image/webp">
  <img data-src="image.jpg" alt="Описание" width="800" height="600">
</picture>
```

## Lazy loading видео

### HTML5 видео
```html
<video controls preload="none" width="800" height="600">
  <source data-src="video.mp4" type="video/mp4">
  Ваш браузер не поддерживает видео.
</video>
```

### Видео с превью
```html
<div class="video-container" data-video-src="video.mp4">
  <img src="preview.jpg" alt="Превью видео" class="video-preview" loading="lazy">
  <button class="play-button">▶</button>
</div>
```

### JavaScript для видео
```javascript
document.querySelectorAll('.video-container').forEach(container => {
  container.addEventListener('click', () => {
    const videoSrc = container.dataset.videoSrc;
    const video = document.createElement('video');
    video.controls = true;
    video.src = videoSrc;
    video.style.width = '100%';
    video.style.height = 'auto';
    
    container.innerHTML = '';
    container.appendChild(video);
    video.play();
  });
});
```

## Lazy loading аудио

### Реализация с превью
```html
<div class="audio-container" data-audio-src="audio.mp3">
  <div class="audio-preview">
    <h4>Название аудио</h4>
    <p>Описание аудио</p>
    <button class="play-audio">▶ Воспроизвести</button>
  </div>
</div>
```

### JavaScript для аудио
```javascript
document.querySelectorAll('.audio-container').forEach(container => {
  const playButton = container.querySelector('.play-audio');
  playButton.addEventListener('click', () => {
    const audioSrc = container.dataset.audioSrc;
    const audio = document.createElement('audio');
    audio.controls = true;
    audio.src = audioSrc;
    
    container.innerHTML = '';
    container.appendChild(audio);
    audio.play();
  });
});
```

## Практические рекомендации

### Определение области видимости
Использование Intersection Observer API позволяет точно определить, когда элемент становится видимым:

```javascript
const options = {
  root: null, // используем viewport
  rootMargin: '100px', // срабатывает за 100px до появления
  threshold: 0.1 // срабатывает когда 10% элемента видно
};

const observer = new IntersectionObserver(callback, options);
```

### Заглушки и превью
- Используйте маленькие заглушки (placeholder) вместо полноразмерных изображений
- Создавайте качественные превью для видео и аудио
- Обеспечьте доступность с помощью атрибутов `alt` и `title`

### Обработка ошибок
```javascript
const img = entry.target;
img.src = img.dataset.src;

img.addEventListener('load', () => {
  img.classList.remove('lazy');
  imageObserver.unobserve(img);
});

img.addEventListener('error', () => {
  img.src = img.dataset.fallback || 'default-image.jpg';
  img.classList.remove('lazy');
  imageObserver.unobserve(img);
});
```

## Практические рекомендации для российских реалий

### Скорость интернета
- В регионах России скорость интернета может быть ограничена
- Lazy loading особенно важен для мобильных пользователей
- Используйте минимальные размеры заглушек для ускорения начальной загрузки

### Тарификация трафика
- Многие пользователи в России имеют ограничения по трафику
- Lazy loading позволяет экономить трафик для пользователей
- Особенно важно для медиа-контента на информационных сайтах

### Популярные форматы
- Учитывайте поддержку форматов в российских браузерах
- Яндекс.Браузер, Mail.Ru Agent и другие могут иметь особенности
- Обеспечьте резервные варианты для старых браузеров

### SEO влияние
- Поисковые системы (включая Яндекс) учитывают lazy loading
- Используйте семантически правильные теги и атрибуты
- Обеспечьте доступность контента для поисковых роботов

## Продвинутые техники

### Предзагрузка при приближении к области видимости
```javascript
const preloadObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      // Создаем Image объект для предзагрузки
      const preloadImg = new Image();
      preloadImg.src = img.dataset.src;
      preloadImg.onload = () => {
        img.src = preloadImg.src;
        preloadObserver.unobserve(img);
      };
    }
  });
}, { rootMargin: '200px' }); // Предзагружаем за 200px до появления
```

### Lazy loading с прогрессивной загрузкой
```html
<div class="progressive-image">
  <img class="low-quality-preview" src="low-quality.jpg" alt="Описание">
  <img class="high-quality-image lazy" data-src="high-quality.jpg" alt="Описание">
</div>
```

### Использование CSS для визуальных эффектов
```css
.lazy {
  opacity: 0;
  transition: opacity 0.3s;
}

.lazy.loaded {
  opacity: 1;
}

.placeholder {
  background: #f0f0f0;
  background-image: linear-gradient(90deg, #f0f0f0 0px, #e0e0e0 40px, #f0f0f0 80px);
  background-size: 300px 100%;
  animation: loading 1.5s infinite;
}

@keyframes loading {
  0% { background-position: -100px 0; }
  100% { background-position: calc(200px + 100%) 0; }
}
```

## Инструменты и библиотеки

### Нативные решения
- Intersection Observer API (рекомендуемый подход)
- Атрибут `loading="lazy"` для изображений и iframe

### Популярные библиотеки
- **lozad.js**: Легкая библиотека для lazy loading
- **vanilla-lazyload**: Ещё одна легковесная библиотека
- **blazy**: Поддержка IE8+

### Пример с lozad.js
```html
<script src="https://cdn.jsdelivr.net/npm/lozad@1"></script>
<img class="lozad" data-src="image.jpg" alt="Описание">
```

```javascript
const observer = lozad(); // инициализация
observer.observe(); // запуск наблюдения
```

## Заключение

Lazy loading медиа-элементов - важная техника оптимизации производительности веб-сайтов. В условиях российского интернета, где скорость соединения и стоимость трафика являются важными факторами, внедрение lazy loading может значительно улучшить пользовательский опыт и SEO-показатели.

См. также:
- [[Оптимизация-изображений]]
- [[Альтернативные-форматы]]
- [[Производительность веб-страниц]]
- [[Адаптивные изображения]]
