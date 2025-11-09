# HTML Мультимедиа: Аудио и видео

HTML5 предоставляет встроенные элементы для включения аудио и видео контента без необходимости использования сторонних плагинов. Элементы `<audio>` и `<video>` делают мультимедиа частью веб-страницы и обеспечивают доступность.

## Элемент `<audio>` - Встраивание аудио

Элемент `<audio>` позволяет встраивать аудио контент в веб-страницу.

### Основное использование

```html
<!-- Простое встраивание аудио -->
<audio src="audio.mp3" controls>
    Ваш браузер не поддерживает аудио элемент.
</audio>

<!-- Альтернативные форматы -->
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    <source src="audio.wav" type="audio/wav">
    Ваш браузер не поддерживает аудио элемент.
</audio>
```

### Атрибуты элемента `<audio>`

- `src` - URL аудио файла
- `controls` - отображение элементов управления
- `autoplay` - автоматическое воспроизведение
- `loop` - зацикленное воспроизведение
- `muted` - выключенный звук по умолчанию
- `preload` - стратегия предзагрузки

```html
<!-- Пример с различными атрибутами -->
<audio 
    src="podcast.mp3" 
    controls 
    preload="metadata"
    loop>
    Ваш браузер не поддерживает аудио элемент.
</audio>
```

### Типы аудио файлов

| Формат | MIME тип | Поддержка |
|--------|----------|-----------|
| MP3 | `audio/mpeg` | Широкая поддержка |
| OGG | `audio/ogg` | Хорошая поддержка |
| WAV | `audio/wav` | Ограниченная поддержка |
| AAC | `audio/aac` | Поддержка в Safari |

```html
<!-- Рекомендуемая практика с несколькими форматами -->
<audio controls>
    <source src="music.mp3" type="audio/mpeg">
    <source src="music.ogg" type="audio/ogg">
    <p>Ваш браузер не поддерживает аудио элемент. 
       <a href="music.mp3">Скачать аудио</a></p>
</audio>
```

## Элемент `<video>` - Встраивание видео

Элемент `<video>` позволяет встраивать видео контент в веб-страницу.

### Основное использование

```html
<!-- Простое встраивание видео -->
<video src="video.mp4" controls width="640" height="360">
    Ваш браузер не поддерживает видео элемент.
</video>

<!-- Альтернативные форматы -->
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <source src="video.ogv" type="video/ogg">
    Ваш браузер не поддерживает видео элемент.
</video>
```

### Атрибуты элемента `<video>`

- `src` - URL видео файла
- `controls` - отображение элементов управления
- `autoplay` - автоматическое воспроизведение
- `loop` - зацикленное воспроизведение
- `muted` - выключенный звук по умолчанию
- `preload` - стратегия предзагрузки
- `poster` - изображение-обложка
- `width`, `height` - размеры видео

```html
<!-- Пример с различными атрибутами -->
<video 
    src="movie.mp4" 
    controls 
    width="800" 
    height="450" 
    poster="poster.jpg"
    preload="metadata">
    Ваш браузер не поддерживает видео элемент.
</video>
```

### Типы видео файлов

| Формат | MIME тип | Поддержка |
|--------|----------|-----------|
| MP4 | `video/mp4` | Широкая поддержка |
| WebM | `video/webm` | Хорошая поддержка |
| OGV | `video/ogg` | Ограниченная поддержка |

```html
<!-- Рекомендуемая практика с несколькими форматами -->
<video controls width="640" height="360" poster="thumbnail.jpg">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <p>Ваш браузер не поддерживает видео элемент. 
       <a href="video.mp4">Скачать видео</a></p>
</video>
```

## Субтитры и описания

Для улучшения доступности можно добавлять субтитры и описания с помощью элемента `<track>`.

```html
<video controls width="640" height="360">
    <source src="movie.mp4" type="video/mp4">
    
    <!-- Субтитры -->
    <track kind="subtitles" src="subtitles-en.vtt" srclang="en" label="English">
    <track kind="subtitles" src="subtitles-ru.vtt" srclang="ru" label="Русский">
    
    <!-- Описание для слабослышащих -->
    <track kind="descriptions" src="descriptions.vtt" srclang="en" label="Описания">
    
    <!-- Главы -->
    <track kind="chapters" src="chapters.vtt" srclang="en" label="Главы">
    
    Ваш браузер не поддерживает видео элемент.
</video>
```

### Формат VTT (WebVTT)

Файл субтитров (с расширением .vtt) имеет следующий формат:

```
WEBVTT

1
00:00:01.000 --> 00:00:04.000
Привет, это видео о HTML5.

2
00:00:05.000 --> 00:00:08.000
Здесь вы узнаете о мультимедиа элементах.
```

## Доступность мультимедиа

### Альтернативный контент

```html
<!-- Для аудио -->
<audio controls>
    <source src="podcast.mp3" type="audio/mpeg">
    <p>Аудио: "Введение в HTML5". 
       <a href="podcast.mp3">Скачать</a> | 
       <a href="transcript.txt">Транскрипция</a></p>
</audio>

<!-- Для видео -->
<video controls width="640" height="360">
    <source src="tutorial.mp4" type="video/mp4">
    <p>Видео: "Руководство по HTML5". 
       <a href="tutorial.mp4">Скачать</a> | 
       <a href="transcript.txt">Транскрипция</a></p>
</video>
```

### Транскрипции

```html
<video controls width="640" height="360">
    <source src="interview.mp4" type="video/mp4">
    Ваш браузер не поддерживает видео элемент.
</video>

<details>
    <summary>Транскрипция видео</summary>
    <p>Интервью с веб-разработчиком о современных технологиях...</p>
    <!-- Полная транскрипция -->
</details>
```

## JavaScript API для мультимедиа

### Основные методы и свойства

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Управление мультимедиа через JavaScript</title>
</head>
<body>
    <video id="myVideo" controls width="640" height="360">
        <source src="sample.mp4" type="video/mp4">
    </video>
    
    <div>
        <button id="playBtn">Воспроизвести</button>
        <button id="pauseBtn">Пауза</button>
        <button id="stopBtn">Стоп</button>
        <input type="range" id="volumeSlider" min="0" max="1" step="0.1" value="1">
    </div>
    
    <script>
        const video = document.getElementById('myVideo');
        const playBtn = document.getElementById('playBtn');
        const pauseBtn = document.getElementById('pauseBtn');
        const stopBtn = document.getElementById('stopBtn');
        const volumeSlider = document.getElementById('volumeSlider');
        
        // Воспроизведение
        playBtn.addEventListener('click', () => {
            video.play();
        });
        
        // Пауза
        pauseBtn.addEventListener('click', () => {
            video.pause();
        });
        
        // Остановка (переход к началу)
        stopBtn.addEventListener('click', () => {
            video.pause();
            video.currentTime = 0;
        });
        
        // Управление громкостью
        volumeSlider.addEventListener('input', () => {
            video.volume = volumeSlider.value;
        });
        
        // События видео
        video.addEventListener('play', () => {
            console.log('Видео воспроизводится');
        });
        
        video.addEventListener('pause', () => {
            console.log('Видео на паузе');
        });
        
        video.addEventListener('ended', () => {
            console.log('Видео завершено');
        });
        
        video.addEventListener('timeupdate', () => {
            console.log(`Текущее время: ${Math.floor(video.currentTime)} сек`);
        });
    </script>
</body>
</html>
```

### Прогресс бар

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Прогресс бар видео</title>
    <style>
        .video-container {
            max-width: 640px;
        }
        
        .progress-container {
            width: 100%;
            height: 5px;
            background-color: #ddd;
            cursor: pointer;
            margin: 10px 0;
        }
        
        .progress-bar {
            height: 100%;
            background-color: #007acc;
            width: 0%;
        }
    </style>
</head>
<body>
    <div class="video-container">
        <video id="myVideo" controls width="100%">
            <source src="sample.mp4" type="video/mp4">
        </video>
        
        <div class="progress-container" id="progressContainer">
            <div class="progress-bar" id="progressBar"></div>
        </div>
    </div>
    
    <script>
        const video = document.getElementById('myVideo');
        const progressContainer = document.getElementById('progressContainer');
        const progressBar = document.getElementById('progressBar');
        
        // Обновление прогресса
        video.addEventListener('timeupdate', updateProgress);
        
        // Изменение времени при клике на прогресс бар
        progressContainer.addEventListener('click', setProgress);
        
        function updateProgress() {
            const percent = (video.currentTime / video.duration) * 100;
            progressBar.style.width = `${percent}%`;
        }
        
        function setProgress(e) {
            const width = this.clientWidth;
            const clickX = e.offsetX;
            const duration = video.duration;
            
            video.currentTime = (clickX / width) * duration;
        }
    </script>
</body>
</html>
```

## Адаптивное видео

### CSS для адаптивного видео

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Адаптивное видео</title>
    <style>
        .video-wrapper {
            position: relative;
            width: 100%;
            height: 0;
            padding-bottom: 56.25%; /* 16:9 соотношение сторон */
        }
        
        .video-wrapper video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }
        
        /* Альтернативный подход с object-fit */
        .responsive-video {
            width: 100%;
            height: auto;
            max-width: 800px;
        }
        
        .responsive-video video {
            width: 100%;
            height: auto;
        }
    </style>
</head>
<body>
    <div class="video-wrapper">
        <video controls>
            <source src="responsive-video.mp4" type="video/mp4">
        </video>
    </div>
    
    <!-- Или с object-fit -->
    <div class="responsive-video">
        <video controls>
            <source src="responsive-video.mp4" type="video/mp4">
        </video>
    </div>
</body>
</html>
```

## Примеры использования

### Аудио плеер

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Аудио плеер</title>
    <style>
        .audio-player {
            max-width: 400px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: #f9f9f9;
        }
        
        .controls {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 10px;
        }
        
        .progress-container {
            flex: 1;
            height: 5px;
            background-color: #ddd;
            border-radius: 3px;
            cursor: pointer;
        }
        
        .progress-bar {
            height: 100%;
            background-color: #007acc;
            border-radius: 3px;
            width: 0%;
        }
        
        .time-display {
            font-size: 0.8em;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="audio-player">
        <h3>Музыкальный трек</h3>
        <p>Исполнитель: Пример</p>
        
        <audio id="audioPlayer">
            <source src="track.mp3" type="audio/mpeg">
        </audio>
        
        <div class="controls">
            <button id="playPauseBtn">▶</button>
            <div class="progress-container" id="progressContainer">
                <div class="progress-bar" id="progressBar"></div>
            </div>
            <span class="time-display" id="timeDisplay">0:00 / 0:00</span>
        </div>
    </div>
    
    <script>
        const audio = document.getElementById('audioPlayer');
        const playPauseBtn = document.getElementById('playPauseBtn');
        const progressContainer = document.getElementById('progressContainer');
        const progressBar = document.getElementById('progressBar');
        const timeDisplay = document.getElementById('timeDisplay');
        
        // Переключение между воспроизведением и паузой
        playPauseBtn.addEventListener('click', () => {
            if (audio.paused) {
                audio.play();
                playPauseBtn.textContent = '⏸';
            } else {
                audio.pause();
                playPauseBtn.textContent = '▶';
            }
        });
        
        // Обновление прогресса
        audio.addEventListener('timeupdate', () => {
            const percent = (audio.currentTime / audio.duration) * 100;
            progressBar.style.width = `${percent}%`;
            
            // Обновление времени
            const current = formatTime(audio.currentTime);
            const duration = formatTime(audio.duration);
            timeDisplay.textContent = `${current} / ${duration}`;
        });
        
        // Установка времени при клике на прогресс
        progressContainer.addEventListener('click', (e) => {
            const width = progressContainer.clientWidth;
            const clickX = e.offsetX;
            audio.currentTime = (clickX / width) * audio.duration;
        });
        
        // Форматирование времени
        function formatTime(seconds) {
            if (isNaN(seconds)) return '0:00';
            
            const min = Math.floor(seconds / 60);
            const sec = Math.floor(seconds % 60);
            return `${min}:${sec < 10 ? '0' : ''}${sec}`;
        }
        
        // Обработка окончания воспроизведения
        audio.addEventListener('ended', () => {
            playPauseBtn.textContent = '▶';
        });
    </script>
</body>
</html>
```

## Заключение

Элементы `<audio>` и `<video>` предоставляют мощные возможности для включения мультимедиа в веб-страницы. Они обеспечивают:
- Встроенную поддержку различных форматов
- Элементы управления по умолчанию
- Доступность через субтитры и транскрипции
- Возможность управления через JavaScript API
- Адаптивность и гибкость в дизайне

Правильное использование этих элементов с учетом доступности и пользовательского опыта помогает создать качественные мультимедийные веб-приложения.

## Следующие темы
- [[HTML/Мультимедиа/Изображения]]
- [[HTML/Мультимедиа/Canvas и SVG]]
- [[HTML/Доступность]]

## Теги
#html #multimedia #audio #video #web-development #accessibility #media #frontend