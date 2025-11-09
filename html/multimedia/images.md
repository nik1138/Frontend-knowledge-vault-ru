# HTML Мультимедиа: Изображения

Изображения являются важной частью современных веб-страниц. HTML предоставляет различные способы включения изображений, каждый из которых имеет свои особенности и области применения.

## Элемент `<img>` - Встраивание изображений

Элемент `<img>` используется для встраивания изображений в HTML документ. Это пустой элемент, который не требует закрывающего тега.

```html
<img src="image.jpg" alt="Описание изображения">
```

### Обязательные атрибуты

#### `src` - Источник изображения

Атрибут `src` указывает путь к изображению:

```html
<!-- Относительный путь -->
<img src="images/photo.jpg" alt="Фотография">

<!-- Абсолютный путь -->
<img src="https://example.com/images/photo.jpg" alt="Фотография">

<!-- Путь от корня сайта -->
<img src="/assets/images/photo.jpg" alt="Фотография">
```

#### `alt` - Альтернативный текст

Атрибут `alt` предоставляет текстовое описание изображения, которое отображается, если изображение не может быть загружено:

```html
<img src="mountain.jpg" alt="Заснеженная гора на фоне заката">
<img src="logo.png" alt="Логотип компании TechCorp">
<img src="decor.jpg" alt=""> <!-- Декоративное изображение -->
```

### Опциональные атрибуты

#### Атрибуты размеров

```html
<img src="photo.jpg" 
     alt="Фотография" 
     width="300" 
     height="200">
```

> **Примечание**: Задание размеров в HTML помогает браузеру заранее выделить место для изображения, предотвращая сдвиги содержимого при загрузке.

#### Атрибут `loading`

Управляет стратегией загрузки изображений:

```html
<!-- Ленивая загрузка для изображений вне области видимости -->
<img src="image.jpg" 
     alt="Описание" 
     loading="lazy">

<!-- Немедленная загрузка -->
<img src="hero-image.jpg" 
     alt="Главное изображение" 
     loading="eager">
```

#### Атрибут `decoding`

Указывает, как браузер должен декодировать изображение:

```html
<img src="image.jpg" 
     alt="Описание" 
     decoding="async">  <!-- Асинхронная декодировка -->
```

## Форматы изображений

### PNG
- Поддержка прозрачности
- Без потерь сжатие
- Хорош для графики и изображений с текстом

```html
<img src="logo.png" alt="Логотип" width="200" height="100">
```

### JPEG
- Компрессия с потерями
- Хорош для фотографий
- Меньший размер файла

```html
<img src="photo.jpg" alt="Фотография природы" width="800" height="600">
```

### GIF
- Поддержка анимации
- Ограниченная цветовая палитра
- Подходит для простой анимации

```html
<img src="loading.gif" alt="Анимация загрузки">
```

### WebP
- Современный формат с отличным сжатием
- Поддержка прозрачности и анимации
- Высокое качество при меньшем размере

```html
<!-- С поддержкой fallback -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.jpg" type="image/jpeg">
    <img src="image.jpg" alt="Описание изображения">
</picture>
```

### SVG
- Векторный формат
- Масштабируется без потери качества
- Можно встраивать непосредственно в HTML

```html
<!-- Как изображение -->
<img src="icon.svg" alt="Иконка" width="32" height="32">

<!-- Встроенный SVG -->
<svg width="32" height="32" viewBox="0 0 32 32">
    <circle cx="16" cy="16" r="10" fill="#007acc"/>
</svg>
```

## Адаптивные изображения

### Атрибут `srcset`

Позволяет указать несколько вариантов изображения для разных разрешений экрана:

```html
<img src="image-800.jpg" 
     srcset="image-400.jpg 400w, 
             image-800.jpg 800w, 
             image-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px, 
            (max-width: 1000px) 800px, 
            1200px"
     alt="Адаптивное изображение">
```

### Элемент `<picture>`

Используется для предоставления разных изображений для разных условий:

```html
<picture>
    <!-- Изображение для больших экранов -->
    <source media="(min-width: 800px)" srcset="large-image.jpg">
    <!-- Изображение для планшетов -->
    <source media="(min-width: 600px)" srcset="medium-image.jpg">
    <!-- Изображение по умолчанию -->
    <img src="small-image.jpg" alt="Адаптивное изображение">
</picture>
```

### Примеры использования `<picture>`

```html
<!-- Разные изображения для разных ориентаций -->
<picture>
    <source media="(orientation: portrait)" srcset="portrait.jpg">
    <source media="(orientation: landscape)" srcset="landscape.jpg">
    <img src="default.jpg" alt="Изображение для разных ориентаций">
</picture>

<!-- Разные форматы с fallback -->
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.jpg" type="image/jpeg">
    <img src="image.jpg" alt="Изображение с несколькими форматами">
</picture>
```

## Доступность изображений

### Альтернативный текст

```html
<!-- Хорошее описание -->
<img src="chart.png" alt="График роста продаж за последний квартал, показывающий увеличение на 25%">

<!-- Для декоративных изображений -->
<img src="decoration.png" alt="">

<!-- Для информативных изображений -->
<img src="warning-icon.png" alt="Предупреждение:">
```

### Длинное описание

Для сложных изображений можно использовать атрибут `longdesc` или элемент `<figure>` с `<figcaption>`:

```html
<figure>
    <img src="complex-chart.jpg" alt="Сложная диаграмма продаж">
    <figcaption>
        <p>Диаграмма показывает рост продаж по регионам с 2020 по 2023 год.</p>
        <p>Северный регион: +15%, Южный регион: +22%, Восточный регион: +18%</p>
    </figcaption>
</figure>
```

## Оптимизация изображений

### Размеры и сжатие

```html
<!-- Оптимизированные размеры -->
<img src="optimized-image.jpg" 
     alt="Оптимизированное изображение" 
     width="400" 
     height="300">
```

### Ленивая загрузка

```html
<!-- Ленивая загрузка для изображений вне области видимости -->
<img src="image.jpg" 
     alt="Описание" 
     loading="lazy" 
     width="300" 
     height="200">
```

## Примеры использования

### Аватар пользователя

```html
<img src="avatar.jpg" 
     alt="Аватар пользователя Ивана Иванова" 
     class="avatar" 
     width="50" 
     height="50">
```

### Галерея изображений

```html
<div class="gallery">
    <img src="photo1.jpg" alt="Фото 1" loading="lazy">
    <img src="photo2.jpg" alt="Фото 2" loading="lazy">
    <img src="photo3.jpg" alt="Фото 3" loading="lazy">
</div>
```

### Логотип

```html
<a href="/">
    <img src="logo.svg" 
         alt="Главная страница сайта" 
         width="150" 
         height="50">
</a>
```

## Стилизация изображений

```css
/* Базовая стилизация изображений */
img {
    max-width: 100%;
    height: auto;
    display: block;
}

/* Изображения-аватары */
.avatar {
    border-radius: 50%;
    border: 2px solid #ddd;
}

/* Изображения-карточки */
.card-image {
    width: 100%;
    height: 200px;
    object-fit: cover;
    border-radius: 8px 8px 0 0;
}

/* Адаптивные изображения */
.responsive-img {
    width: 100%;
    height: auto;
}
```

## Распространенные ошибки

### 1. Отсутствие атрибута `alt`

```html
<!-- Неправильно -->
<img src="photo.jpg">

<!-- Правильно -->
<img src="photo.jpg" alt="Описание изображения">
```

### 2. Использование `alt` для декоративных изображений

```html
<!-- Неправильно - декоративное изображение с описанием -->
<img src="decoration.png" alt="Декоративный элемент в правом углу">

<!-- Правильно - пустой alt для декоративных изображений -->
<img src="decoration.png" alt="">
```

### 3. Несоответствие размеров

```html
<!-- Неправильно - изображение 10x10 отображается как 300x200 -->
<img src="small-icon.png" alt="Иконка" width="300" height="200">

<!-- Правильно - правильные размеры -->
<img src="icon.png" alt="Иконка" width="32" height="32">
```

## Современные возможности изображений

### Изображения с WebP и AVIF форматами
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Современные форматы изображений</title>
</head>
<body>
    <main>
        <h1>Современные форматы изображений</h1>
        
        <!-- Поддержка WebP -->
        <picture>
            <source srcset="image.webp" type="image/webp">
            <img src="image.jpg" 
                 alt="Изображение с поддержкой WebP формата"
                 width="800" 
                 height="600">
        </picture>
        
        <!-- Поддержка AVIF -->
        <picture>
            <source srcset="photo.avif" type="image/avif">
            <source srcset="photo.webp" type="image/webp">
            <source srcset="photo.jpg" type="image/jpeg">
            <img src="photo.jpg" 
                 alt="Фотография с поддержкой AVIF, WebP и JPEG"
                 width="1200" 
                 height="800">
        </picture>
        
        <!-- Сложное изображение с несколькими форматами -->
        <picture>
            <!-- Для больших экранов -->
            <source media="(min-width: 1200px)" 
                    srcset="large.avif 1x, large@2x.avif 2x" 
                    type="image/avif">
            <source media="(min-width: 1200px)" 
                    srcset="large.webp 1x, large@2x.webp 2x" 
                    type="image/webp">
            
            <!-- Для средних экранов -->
            <source media="(min-width: 768px)" 
                    srcset="medium.avif 1x, medium@2x.avif 2x" 
                    type="image/avif">
            <source media="(min-width: 768px)" 
                    srcset="medium.webp 1x, medium@2x.webp 2x" 
                    type="image/webp">
            
            <!-- Для маленьких экранов -->
            <source srcset="small.avif 1x, small@2x.avif 2x" 
                    type="image/avif">
            <source srcset="small.webp 1x, small@2x.webp 2x" 
                    type="image/webp">
            
            <!-- Резервный вариант -->
            <img src="fallback.jpg" 
                 alt="Адаптивное изображение с поддержкой современных форматов"
                 width="600" 
                 height="400">
        </picture>
    </main>
</body>
</html>
```

### Изображения с Web Components
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Изображения с Web Components</title>
</head>
<body>
    <main>
        <h1>Изображения с Web Components</h1>
        
        <lazy-image 
            src="large-image.jpg"
            alt="Изображение, загружающееся лениво"
            width="800"
            height="600"
            loading-strategy="intersection">
        </lazy-image>
        
        <responsive-gallery>
            <gallery-item src="img1.jpg" alt="Изображение 1"></gallery-item>
            <gallery-item src="img2.jpg" alt="Изображение 2"></gallery-item>
            <gallery-item src="img3.jpg" alt="Изображение 3"></gallery-item>
        </responsive-gallery>
    </main>

    <script>
        // Кастомный элемент для ленивой загрузки изображений
        class LazyImage extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const src = this.getAttribute('src') || '';
                const alt = this.getAttribute('alt') || '';
                const width = this.getAttribute('width') || 'auto';
                const height = this.getAttribute('height') || 'auto';
                const loadingStrategy = this.getAttribute('loading-strategy') || 'lazy';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .image-container {
                            position: relative;
                            display: inline-block;
                        }
                        
                        .image {
                            width: ${width}px;
                            height: ${height}px;
                            object-fit: cover;
                            transition: opacity 0.3s;
                        }
                        
                        .placeholder {
                            background-color: #f0f0f0;
                            width: ${width}px;
                            height: ${height}px;
                            display: flex;
                            align-items: center;
                            justify-content: center;
                        }
                        
                        .loader {
                            border: 4px solid #f3f3f3;
                            border-top: 4px solid #007acc;
                            border-radius: 50%;
                            width: 40px;
                            height: 40px;
                            animation: spin 1s linear infinite;
                        }
                        
                        @keyframes spin {
                            0% { transform: rotate(0deg); }
                            100% { transform: rotate(360deg); }
                        }
                    </style>
                    
                    <div class="image-container">
                        <div class="placeholder">
                            <div class="loader" id="loader"></div>
                        </div>
                        <img class="image" id="image" src="" alt="${alt}" style="opacity: 0;">
                    </div>
                `;
                
                this.image = this.shadowRoot.getElementById('image');
                this.placeholder = this.shadowRoot.querySelector('.placeholder');
                this.loader = this.shadowRoot.getElementById('loader');
                
                this.loadImage(src, loadingStrategy);
            }
            
            loadImage(src, strategy) {
                if (strategy === 'intersection') {
                    // Используем Intersection Observer для ленивой загрузки
                    const observer = new IntersectionObserver((entries) => {
                        entries.forEach(entry => {
                            if (entry.isIntersecting) {
                                this.loadImageSrc(src);
                                observer.disconnect();
                            }
                        });
                    });
                    
                    observer.observe(this);
                } else {
                    // Простая ленивая загрузка
                    this.loadImageSrc(src);
                }
            }
            
            loadImageSrc(src) {
                const img = new Image();
                
                img.onload = () => {
                    this.image.src = src;
                    this.image.style.opacity = '1';
                    this.placeholder.style.display = 'none';
                };
                
                img.onerror = () => {
                    this.placeholder.innerHTML = '<div>Ошибка загрузки изображения</div>';
                };
                
                img.src = src;
            }
        }
        
        // Кастомный элемент для галереи
        class ResponsiveGallery extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .gallery {
                            display: grid;
                            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
                            gap: 15px;
                            padding: 20px;
                        }
                        
                        .gallery-item {
                            position: relative;
                            overflow: hidden;
                            border-radius: 8px;
                            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
                        }
                        
                        .gallery-image {
                            width: 100%;
                            height: 200px;
                            object-fit: cover;
                            display: block;
                            transition: transform 0.3s;
                        }
                        
                        .gallery-image:hover {
                            transform: scale(1.05);
                        }
                    </style>
                    
                    <div class="gallery" id="gallery-container">
                        <slot name="gallery-item"></slot>
                    </div>
                `;
                
                this.container = this.shadowRoot.getElementById('gallery-container');
            }
            
            connectedCallback() {
                // Обновляем галерею при добавлении элементов
                this.updateGallery();
                
                // Наблюдаем за изменениями в слоте
                const slot = this.shadowRoot.querySelector('slot');
                slot.addEventListener('slotchange', () => {
                    this.updateGallery();
                });
            }
            
            updateGallery() {
                const items = this.querySelectorAll('gallery-item');
                items.forEach(item => {
                    const img = document.createElement('img');
                    img.src = item.getAttribute('src');
                    img.alt = item.getAttribute('alt');
                    img.className = 'gallery-image';
                    
                    const galleryItem = document.createElement('div');
                    galleryItem.className = 'gallery-item';
                    galleryItem.appendChild(img);
                    
                    this.container.appendChild(galleryItem);
                });
            }
        }
        
        // Кастомный элемент для отдельного элемента галереи
        class GalleryItem extends HTMLElement {
            constructor() {
                super();
            }
        }
        
        // Регистрация кастомных элементов
        customElements.define('lazy-image', LazyImage);
        customElements.define('responsive-gallery', ResponsiveGallery);
        customElements.define('gallery-item', GalleryItem);
    </script>
</body>
</html>
```

### Изображения с WebSockets для динамического контента
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамические изображения через WebSocket</title>
</head>
<body>
    <main>
        <h1>Динамические изображения</h1>
        
        <div id="image-container">
            <img id="dynamic-image" 
                 src="placeholder.jpg" 
                 alt="Динамическое изображение" 
                 width="600" 
                 height="400">
        </div>
        
        <div id="image-info" aria-live="polite" aria-atomic="true"></div>
    </main>

    <script>
        // Подключение к WebSocket для получения обновлений изображений
        const ws = new WebSocket('ws://localhost:8080/images');
        const imageContainer = document.getElementById('dynamic-image');
        const infoContainer = document.getElementById('image-info');
        
        ws.onopen = function() {
            infoContainer.textContent = 'Соединение с сервером изображений установлено';
        };
        
        ws.onmessage = function(event) {
            const imageData = JSON.parse(event.data);
            
            if (imageData.type === 'image-update') {
                // Обновляем изображение
                imageContainer.src = imageData.url;
                imageContainer.alt = imageData.alt;
                
                // Объявляем обновление для скринридера
                announceImageUpdate(imageData.alt);
            } else if (imageData.type === 'image-info') {
                // Обновляем информацию об изображении
                infoContainer.textContent = imageData.info;
            }
        };
        
        ws.onerror = function(error) {
            infoContainer.textContent = 'Ошибка соединения с сервером изображений';
        };
        
        ws.onclose = function() {
            infoContainer.textContent = 'Соединение с сервером изображений закрыто';
        };
        
        function announceImageUpdate(alt) {
            const announcement = document.createElement('div');
            announcement.setAttribute('aria-live', 'polite');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.style.position = 'absolute';
            announcement.style.left = '-9999px';
            announcement.textContent = `Изображение обновлено: ${alt}`;
            
            document.body.appendChild(announcement);
            
            setTimeout(() => {
                if (document.body.contains(announcement)) {
                    document.body.removeChild(announcement);
                }
            }, 5000);
        }
        
        // Обработка ошибок загрузки изображения
        imageContainer.addEventListener('error', function() {
            this.src = 'error-placeholder.jpg';
            this.alt = 'Ошибка загрузки изображения';
            infoContainer.textContent = 'Ошибка загрузки изображения';
        });
    </script>
</body>
</html>
```

### Изображения с IndexedDB для оффлайн-доступа
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оффлайн-изображения с IndexedDB</title>
</head>
<body>
    <main>
        <h1>Оффлайн-изображения</h1>
        
        <div id="gallery-container">
            <h2>Галерея изображений</h2>
            <div id="gallery" role="grid" aria-label="Галерея изображений"></div>
        </div>
        
        <div id="status" aria-live="polite" aria-atomic="true"></div>
        
        <button id="sync-btn" onclick="syncImages()">Синхронизировать изображения</button>
    </main>

    <script>
        // Подключение к IndexedDB
        const request = indexedDB.open('OfflineImagesDB', 1);
        let db;
        
        request.onerror = function(event) {
            updateStatus('Ошибка при открытии базы данных', 'error');
        };
        
        request.onsuccess = function(event) {
            db = event.target.result;
            updateStatus('База данных изображений открыта', 'success');
            loadImages();
        };
        
        request.onupgradeneeded = function(event) {
            db = event.target.result;
            
            if (!db.objectStoreNames.contains('images')) {
                const store = db.createObjectStore('images', { keyPath: 'id' });
                store.createIndex('url', 'url', { unique: false });
                store.createIndex('date', 'date', { unique: false });
                store.createIndex('size', 'size', { unique: false });
            }
        };
        
        function syncImages() {
            updateStatus('Синхронизация изображений...', 'info');
            
            // Имитация загрузки изображений с сервера
            const sampleImages = [
                { id: '1', url: 'https://picsum.photos/300/200?random=1', alt: 'Случайное изображение 1', date: new Date().toISOString(), size: 'large' },
                { id: '2', url: 'https://picsum.photos/300/200?random=2', alt: 'Случайное изображение 2', date: new Date().toISOString(), size: 'medium' },
                { id: '3', url: 'https://picsum.photos/300/200?random=3', alt: 'Случайное изображение 3', date: new Date().toISOString(), size: 'small' }
            ];
            
            sampleImages.forEach(image => {
                saveImageToDB(image);
            });
        }
        
        function saveImageToDB(image) {
            const transaction = db.transaction(['images'], 'readwrite');
            const store = transaction.objectStore('images');
            const request = store.put(image);
            
            request.onsuccess = function() {
                updateStatus(`Изображение ${image.alt} сохранено`, 'success');
            };
            
            request.onerror = function() {
                updateStatus(`Ошибка при сохранении изображения ${image.alt}`, 'error');
            };
        }
        
        function loadImages() {
            const transaction = db.transaction(['images'], 'readonly');
            const store = transaction.objectStore('images');
            const request = store.getAll();
            
            request.onsuccess = function() {
                displayImages(request.result);
            };
            
            request.onerror = function() {
                updateStatus('Ошибка при загрузке изображений', 'error');
            };
        }
        
        function displayImages(images) {
            const gallery = document.getElementById('gallery');
            gallery.innerHTML = '';
            
            if (images.length === 0) {
                gallery.innerHTML = '<p>Нет сохраненных изображений.</p>';
                return;
            }
            
            images.forEach(image => {
                const imgElement = document.createElement('img');
                imgElement.src = image.url;
                imgElement.alt = image.alt;
                imgElement.width = 300;
                imgElement.height = 200;
                imgElement.className = 'gallery-item';
                imgElement.setAttribute('role', 'gridcell');
                imgElement.setAttribute('aria-label', image.alt);
                
                // Обработка ошибок загрузки
                imgElement.addEventListener('error', function() {
                    this.src = 'placeholder.jpg';
                    this.alt = 'Изображение недоступно';
                });
                
                gallery.appendChild(imgElement);
            });
        }
        
        function updateStatus(message, type = 'info') {
            const statusDiv = document.getElementById('status');
            statusDiv.textContent = message;
            statusDiv.className = `status status-${type}`;
            
            // Автоматическое скрытие через 5 секунд
            setTimeout(() => {
                if (statusDiv.textContent === message) {
                    statusDiv.textContent = '';
                    statusDiv.className = 'status';
                }
            }, 5000);
        }
    </script>
    
    <style>
        #gallery {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
        
        .gallery-item {
            width: 100%;
            height: 200px;
            object-fit: cover;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        
        .status {
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .status-success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .status-error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .status-info {
            background-color: #d1ecf1;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }
        
        #sync-btn {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        #sync-btn:hover {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Адаптивная карточка с изображением
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные карточки с изображениями</title>
    <style>
        .cards-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            padding: 20px;
        }
        
        .card {
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            transition: transform 0.3s, box-shadow 0.3s;
        }
        
        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 16px rgba(0,0,0,0.2);
        }
        
        .card-image-container {
            position: relative;
            width: 100%;
            height: 200px;
            overflow: hidden;
        }
        
        .card-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s;
        }
        
        .card:hover .card-image {
            transform: scale(1.05);
        }
        
        .card-content {
            padding: 15px;
        }
        
        .card-title {
            margin: 0 0 10px 0;
            font-size: 1.2em;
        }
        
        .card-description {
            color: #666;
            margin-bottom: 15px;
        }
        
        .card-button {
            background-color: #007acc;
            color: white;
            border: none;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
            text-decoration: none;
            display: inline-block;
        }
        
        .card-button:hover {
            background-color: #005a9e;
        }
        
        @media (max-width: 768px) {
            .cards-container {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <main>
        <h1>Адаптивные карточки с изображениями</h1>
        
        <div class="cards-container">
            <article class="card" role="article" aria-labelledby="card1-title">
                <div class="card-image-container">
                    <picture>
                        <source media="(min-width: 1200px)" srcset="large1.avif" type="image/avif">
                        <source media="(min-width: 768px)" srcset="medium1.avif" type="image/avif">
                        <source srcset="small1.avif" type="image/avif">
                        
                        <source media="(min-width: 1200px)" srcset="large1.webp" type="image/webp">
                        <source media="(min-width: 768px)" srcset="medium1.webp" type="image/webp">
                        <source srcset="small1.webp" type="image/webp">
                        
                        <img class="card-image" 
                             src="default1.jpg" 
                             alt="Описание первого изображения"
                             loading="lazy"
                             aria-describedby="card1-desc">
                    </picture>
                </div>
                
                <div class="card-content">
                    <h2 id="card1-title">Заголовок карточки 1</h2>
                    <p id="card1-desc" class="card-description">Краткое описание содержимого карточки 1...</p>
                    <a href="/card1" class="card-button">Подробнее</a>
                </div>
            </article>
            
            <article class="card" role="article" aria-labelledby="card2-title">
                <div class="card-image-container">
                    <picture>
                        <source media="(min-width: 1200px)" srcset="large2.avif" type="image/avif">
                        <source media="(min-width: 768px)" srcset="medium2.avif" type="image/avif">
                        <source srcset="small2.avif" type="image/avif">
                        
                        <source media="(min-width: 1200px)" srcset="large2.webp" type="image/webp">
                        <source media="(min-width: 768px)" srcset="medium2.webp" type="image/webp">
                        <source srcset="small2.webp" type="image/webp">
                        
                        <img class="card-image" 
                             src="default2.jpg" 
                             alt="Описание второго изображения"
                             loading="lazy"
                             aria-describedby="card2-desc">
                    </picture>
                </div>
                
                <div class="card-content">
                    <h2 id="card2-title">Заголовок карточки 2</h2>
                    <p id="card2-desc" class="card-description">Краткое описание содержимого карточки 2...</p>
                    <a href="/card2" class="card-button">Подробнее</a>
                </div>
            </article>
            
            <article class="card" role="article" aria-labelledby="card3-title">
                <div class="card-image-container">
                    <picture>
                        <source media="(min-width: 1200px)" srcset="large3.avif" type="image/avif">
                        <source media="(min-width: 768px)" srcset="medium3.avif" type="image/avif">
                        <source srcset="small3.avif" type="image/avif">
                        
                        <source media="(min-width: 1200px)" srcset="large3.webp" type="image/webp">
                        <source media="(min-width: 768px)" srcset="medium3.webp" type="image/webp">
                        <source srcset="small3.webp" type="image/webp">
                        
                        <img class="card-image" 
                             src="default3.jpg" 
                             alt="Описание третьего изображения"
                             loading="lazy"
                             aria-describedby="card3-desc">
                    </picture>
                </div>
                
                <div class="card-content">
                    <h2 id="card3-title">Заголовок карточки 3</h2>
                    <p id="card3-desc" class="card-description">Краткое описание содержимого карточки 3...</p>
                    <a href="/card3" class="card-button">Подробнее</a>
                </div>
            </article>
        </div>
    </main>
</body>
</html>
```

### Галерея с ленивой загрузкой и светофором
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Галерея с ленивой загрузкой</title>
    <style>
        .gallery-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .gallery {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 15px;
        }
        
        .gallery-item {
            position: relative;
            overflow: hidden;
            border-radius: 8px;
            aspect-ratio: 4/3;
            background-color: #f0f0f0;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .gallery-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .gallery-image.loaded {
            opacity: 1;
        }
        
        .gallery-loader {
            width: 40px;
            height: 40px;
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .lightbox {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.9);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s;
        }
        
        .lightbox.active {
            opacity: 1;
            pointer-events: all;
        }
        
        .lightbox-image {
            max-width: 90%;
            max-height: 90%;
            object-fit: contain;
        }
        
        .lightbox-close {
            position: absolute;
            top: 20px;
            right: 30px;
            color: white;
            font-size: 30px;
            cursor: pointer;
        }
        
        .gallery-controls {
            display: flex;
            justify-content: center;
            margin-top: 20px;
            gap: 10px;
        }
        
        .gallery-btn {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .gallery-btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <main>
        <div class="gallery-container">
            <h1>Галерея с ленивой загрузкой</h1>
            
            <div class="gallery" id="gallery" role="grid" aria-label="Фотогалерея"></div>
            
            <div class="gallery-controls">
                <button class="gallery-btn" onclick="loadMoreImages()">Загрузить еще</button>
                <button class="gallery-btn" onclick="clearGallery()">Очистить</button>
            </div>
        </div>
        
        <div class="lightbox" id="lightbox" aria-modal="true" role="dialog" aria-label="Модальное окно галереи">
            <span class="lightbox-close" onclick="closeLightbox()">&times;</span>
            <img class="lightbox-image" id="lightbox-img" alt="Увеличенное изображение">
        </div>
    </main>

    <script>
        class LazyImageGallery {
            constructor(containerId, lightboxId) {
                this.container = document.getElementById(containerId);
                this.lightbox = document.getElementById(lightboxId);
                this.lightboxImg = document.getElementById('lightbox-img');
                this.currentPage = 1;
                this.isLoading = false;
                
                this.init();
            }
            
            init() {
                this.loadImages();
                
                // Используем Intersection Observer для ленивой загрузки
                this.observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting && entry.target.dataset.src) {
                            this.loadImage(entry.target);
                        }
                    });
                }, {
                    rootMargin: '50px' // Начинать загрузку за 50px до появления
                });
            }
            
            async loadImages() {
                if (this.isLoading) return;
                
                this.isLoading = true;
                
                // Имитация загрузки изображений с API
                const images = await this.fetchImages(this.currentPage);
                
                images.forEach(image => {
                    this.createGalleryItem(image);
                });
                
                this.currentPage++;
                this.isLoading = false;
            }
            
            async fetchImages(page) {
                // Имитация API вызова
                return new Promise(resolve => {
                    setTimeout(() => {
                        const images = [];
                        const startIndex = (page - 1) * 12;
                        
                        for (let i = 0; i < 12; i++) {
                            const id = startIndex + i + 1;
                            images.push({
                                id: id,
                                url: `https://picsum.photos/600/400?random=${id}`,
                                thumbnail: `https://picsum.photos/300/200?random=${id}`,
                                alt: `Изображение ${id}`
                            });
                        }
                        
                        resolve(images);
                    }, 1000);
                });
            }
            
            createGalleryItem(image) {
                const item = document.createElement('div');
                item.className = 'gallery-item';
                item.setAttribute('role', 'gridcell');
                item.setAttribute('tabindex', '0');
                item.setAttribute('aria-label', image.alt);
                
                item.innerHTML = `
                    <div class="gallery-placeholder">
                        <div class="gallery-loader"></div>
                    </div>
                    <img class="gallery-image" 
                         data-src="${image.thumbnail}" 
                         alt="${image.alt}"
                         loading="lazy">
                `;
                
                const img = item.querySelector('.gallery-image');
                
                // Обработчики событий
                img.addEventListener('load', () => {
                    img.classList.add('loaded');
                    const placeholder = item.querySelector('.gallery-placeholder');
                    if (placeholder) placeholder.remove();
                });
                
                img.addEventListener('click', () => this.openLightbox(image.url, image.alt));
                item.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' || e.key === ' ') {
                        e.preventDefault();
                        this.openLightbox(image.url, image.alt);
                    }
                });
                
                this.container.appendChild(item);
                
                // Наблюдаем за элементом для ленивой загрузки
                this.observer.observe(img);
            }
            
            loadImage(imgElement) {
                if (!imgElement.dataset.src) return;
                
                imgElement.src = imgElement.dataset.src;
                imgElement.removeAttribute('data-src');
            }
            
            openLightbox(imageUrl, alt) {
                this.lightboxImg.src = imageUrl;
                this.lightboxImg.alt = alt;
                this.lightbox.classList.add('active');
                
                // Запрет прокрутки фона
                document.body.style.overflow = 'hidden';
            }
            
            closeLightbox() {
                this.lightbox.classList.remove('active');
                
                // Возобновление прокрутки
                document.body.style.overflow = '';
            }
            
            clearGallery() {
                this.container.innerHTML = '';
                this.currentPage = 1;
            }
        }
        
        // Инициализация галереи
        document.addEventListener('DOMContentLoaded', () => {
            const gallery = new LazyImageGallery('gallery', 'lightbox');
            
            // Закрытие лайтбокса по Esc
            document.addEventListener('keydown', (e) => {
                if (e.key === 'Escape') {
                    document.getElementById('lightbox').classList.remove('active');
                    document.body.style.overflow = '';
                }
            });
            
            // Закрытие лайтбокса по клику вне изображения
            document.getElementById('lightbox').addEventListener('click', (e) => {
                if (e.target === e.currentTarget) {
                    document.getElementById('lightbox').classList.remove('active');
                    document.body.style.overflow = '';
                }
            });
        });
        
        // Функции для кнопок
        function loadMoreImages() {
            // Эта функция будет вызвана из глобальной области видимости
            // и должна получить доступ к экземпляру галереи
            const galleryInstance = window.galleryInstance || 
                (window.galleryInstance = new LazyImageGallery('gallery', 'lightbox'));
            galleryInstance.loadImages();
        }
        
        function clearGallery() {
            document.getElementById('gallery').innerHTML = '';
            window.galleryInstance = new LazyImageGallery('gallery', 'lightbox');
        }
        
        function closeLightbox() {
            document.getElementById('lightbox').classList.remove('active');
            document.body.style.overflow = '';
        }
    </script>
</body>
</html>
```

## Проверка и оптимизация изображений

### Инструменты для проверки:
1. **Lighthouse** - встроенная проверка оптимизации изображений
2. **PageSpeed Insights** - рекомендации по оптимизации
3. **GTmetrix** - анализ производительности, включая изображения
4. **WebPageTest** - детальный анализ загрузки ресурсов
5. **ImageOptim, TinyPNG, Squoosh** - инструменты сжатия изображений

### Проверка доступности изображений:
1. Убедитесь, что все изображения имеют атрибут `alt`
2. Проверьте, что `alt` текст описывает содержимое изображения
3. Используйте пустой `alt` для декоративных изображений
4. Проверьте контрастность текста на изображениях
5. Убедитесь, что изображения масштабируются без потери качества

## Лучшие практики изображений

### 1. Оптимизация размеров и форматов
```html
<!-- Использование подходящих размеров -->
<img src="product-small.jpg" 
     srcset="product-tiny.jpg 400w,
             product-small.jpg 800w,
             product-medium.jpg 1200w,
             product-large.jpg 1600w"
     sizes="(max-width: 600px) 100vw,
            (max-width: 1000px) 50vw,
            33vw"
     alt="Описание продукта"
     loading="lazy">

<!-- Использование современных форматов -->
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.jpg" type="image/jpeg">
    <img src="image.jpg" alt="Описание изображения">
</picture>
```

### 2. Адаптивность и производительность
```html
<!-- Изображение с предварительной загрузкой критического изображения -->
<link rel="preload" as="image" href="hero-image.jpg" type="image/jpeg" />

<!-- Адаптивное изображение с правильным соотношением сторон -->
<div class="aspect-ratio-box" style="padding-bottom: 56.25%;"> <!-- 16:9 -->
    <img src="video-thumbnail.jpg" 
         alt="Превью видео"
         style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover;">
</div>

<style>
.aspect-ratio-box {
    position: relative;
    width: 100%;
    height: 0;
}

.aspect-ratio-box img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
}
</style>
```

### 3. Доступность
```html
<!-- Хорошее описание для информативных изображений -->
<img src="sales-chart.png" 
     alt="График роста продаж за 2023 год, показывающий увеличение на 25% по сравнению с 2022 годом">

<!-- Пустой alt для декоративных изображений -->
<img src="decorative-border.png" alt="" role="presentation">

<!-- Длинное описание для сложных изображений -->
<figure>
    <img src="complex-diagram.png" 
         alt="Сложная диаграмма системных взаимодействий"
         aria-describedby="diagram-description">
    <figcaption id="diagram-description">
        <p>Диаграмма показывает взаимодействие между компонентами A, B и C.</p>
        <p>Компонент A взаимодействует с B через API, B обрабатывает данные и передает C.</p>
    </figcaption>
</figure>
```

## Заключение

Современные возможности HTML для работы с изображениями включают не только базовые элементы `<img>` и `<picture>`, но и продвинутые техники, такие как использование современных форматов (WebP, AVIF), ленивая загрузка, адаптивные изображения, интеграция с веб-компонентами и оффлайн-доступ. Правильное использование этих возможностей с учетом доступности, производительности и пользовательского опыта создает качественные веб-страницы, которые быстро загружаются и доступны для всех пользователей.

Ключевые аспекты современных изображений:
1. Использование современных форматов (WebP, AVIF) с fallback
2. Адаптивные изображения с правильным использованием srcset и sizes
3. Ленивая загрузка для улучшения производительности
4. Доступность с подходящими alt-атрибутами
5. Оптимизация размеров и сжатие
6. Поддержка оффлайн-доступа через IndexedDB
7. Интеграция с современными веб-технологиями
8. Следование стандартам и лучшим практикам

Эти практики позволяют создавать изображения, которые не только выглядят хорошо, но и обеспечивают отличный пользовательский опыт на всех устройствах и подключениях.

## Следующие темы
- [[HTML/Мультимедиа/Аудио и видео]]
- [[HTML/Мультимедиа/Canvas и SVG]]
- [[HTML/Оптимизация производительности]]

## Теги
#html #multimedia #images #img #web-development #accessibility #performance #responsive-design #webp #avif #web-components #websockets #indexeddb #lazy-loading #modern-formats #image-optimization #best-practices