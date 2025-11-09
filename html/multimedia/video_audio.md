# HTML Мультимедиа: Видео и аудио

HTML предоставляет встроенные элементы для включения аудио и видео контента на веб-страницы без необходимости использования внешних плагинов. Элементы `<video>` и `<audio>` обеспечивают богатые возможности для воспроизведения мультимедиа с широкими возможностями кастомизации и управления.

## Основы элемента `<video>`

### Простое включение видео

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>HTML Видео</title>
</head>
<body>
    <h1>Воспроизведение видео</h1>
    
    <video controls width="800" height="450">
        <source src="movie.mp4" type="video/mp4">
        <source src="movie.webm" type="video/webm">
        <source src="movie.ogv" type="video/ogg">
        Ваш браузер не поддерживает элемент video.
    </video>

    <script>
        // Получение ссылки на видео элемент
        const video = document.querySelector('video');
        
        // Обработка событий видео
        video.addEventListener('loadstart', () => {
            console.log('Началась загрузка видео');
        });
        
        video.addEventListener('canplay', () => {
            console.log('Видео может быть воспроизведено');
        });
        
        video.addEventListener('play', () => {
            console.log('Видео началось');
        });
        
        video.addEventListener('pause', () => {
            console.log('Видео приостановлено');
        });
        
        video.addEventListener('ended', () => {
            console.log('Видео завершено');
        });
        
        video.addEventListener('timeupdate', () => {
            console.log(`Текущее время: ${video.currentTime.toFixed(2)} сек`);
        });
    </script>
</body>
</html>
```

### Атрибуты элемента `<video>`

```html
<video 
    src="movie.mp4"
    controls          <!-- Показывать элементы управления -->
    autoplay          <!-- Автоматическое воспроизведение -->
    muted             <!-- Заглушить звук -->
    loop              <!-- Повторять видео -->
    preload="metadata" <!-- Загрузка метаданных -->
    poster="poster.jpg" <!-- Изображение-обложка -->
    width="800"       <!-- Ширина -->
    height="450"      <!-- Высота -->
    playsinline       <!-- Воспроизведение без перехода в полноэкранный режим -->
>
    Ваш браузер не поддерживает видео элемент.
</video>
```

### Поддержка нескольких форматов

```html
<video controls width="640" height="360" preload="metadata">
    <!-- MP4 формат (широкая поддержка) -->
    <source src="movie.mp4" type="video/mp4">
    
    <!-- WebM формат (открытый стандарт) -->
    <source src="movie.webm" type="video/webm">
    
    <!-- Ogg формат (Theora/Vorbis) -->
    <source src="movie.ogv" type="video/ogg">
    
    <!-- Резервный вариант -->
    <p>Ваш браузер не поддерживает видео элемент. 
       <a href="movie.mp4">Скачать видео</a></p>
</video>
```

## Основы элемента `<audio>`

### Простое включение аудио

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>HTML Аудио</title>
</head>
<body>
    <h1>Воспроизведение аудио</h1>
    
    <audio controls>
        <source src="song.mp3" type="audio/mpeg">
        <source src="song.ogg" type="audio/ogg">
        <source src="song.wav" type="audio/wav">
        Ваш браузер не поддерживает аудио элемент.
    </audio>

    <script>
        const audio = document.querySelector('audio');
        
        audio.addEventListener('loadstart', () => {
            console.log('Началась загрузка аудио');
        });
        
        audio.addEventListener('canplay', () => {
            console.log('Аудио может быть воспроизведено');
        });
        
        audio.addEventListener('play', () => {
            console.log('Аудио началось');
        });
        
        audio.addEventListener('pause', () => {
            console.log('Аудио приостановлено');
        });
        
        audio.addEventListener('ended', () => {
            console.log('Аудио завершено');
        });
        
        audio.addEventListener('timeupdate', () => {
            console.log(`Текущее время: ${audio.currentTime.toFixed(2)} сек`);
        });
    </script>
</body>
</html>
```

### Атрибуты элемента `<audio>`

```html
<audio 
    src="song.mp3"
    controls     <!-- Показывать элементы управления -->
    autoplay     <!-- Автоматическое воспроизведение -->
    muted        <!-- Заглушить звук -->
    loop         <!-- Повторять аудио -->
    preload="auto" <!-- Автоматическая предзагрузка -->
    volume="0.5" <!-- Громкость (0.0 - 1.0) -->
>
    Ваш браузер не поддерживает аудио элемент.
</audio>
```

## Продвинутые возможности воспроизведения

### Управление воспроизведением с JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Управление мультимедиа</title>
    <style>
        .player-container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
        
        .video-container {
            position: relative;
            margin-bottom: 20px;
        }
        
        .video-player {
            width: 100%;
            display: block;
        }
        
        .controls {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-bottom: 15px;
        }
        
        .control-btn {
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            background-color: #007acc;
            color: white;
        }
        
        .control-btn:hover {
            background-color: #005a9e;
        }
        
        .progress-container {
            width: 100%;
            height: 5px;
            background-color: #ddd;
            border-radius: 3px;
            cursor: pointer;
            margin: 10px 0;
        }
        
        .progress-bar {
            height: 100%;
            background-color: #007acc;
            border-radius: 3px;
            width: 0%;
        }
        
        .time-display {
            font-size: 0.9em;
            color: #666;
        }
        
        .volume-control {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        
        .volume-slider {
            width: 100px;
        }
    </style>
</head>
<body>
    <div class="player-container">
        <h1>Продвинутый медиа-проигрыватель</h1>
        
        <div class="video-container">
            <video id="video-player" class="video-player" controls preload="metadata">
                <source src="sample-video.mp4" type="video/mp4">
                <source src="sample-video.webm" type="video/webm">
                Ваш браузер не поддерживает видео элемент.
            </video>
        </div>
        
        <div class="controls">
            <button id="play-btn" class="control-btn">▶ Воспроизвести</button>
            <button id="pause-btn" class="control-btn">⏸ Пауза</button>
            <button id="stop-btn" class="control-btn">⏹ Стоп</button>
            <button id="mute-btn" class="control-btn">🔇 Звук</button>
        </div>
        
        <div class="progress-container" id="progress-container">
            <div class="progress-bar" id="progress-bar"></div>
        </div>
        
        <div class="time-display">
            <span id="current-time">00:00</span> / <span id="duration">00:00</span>
        </div>
        
        <div class="volume-control">
            <span>🔊</span>
            <input type="range" 
                   id="volume-slider" 
                   class="volume-slider" 
                   min="0" 
                   max="1" 
                   step="0.01" 
                   value="1">
        </div>
    </div>

    <script>
        class MediaPlayer {
            constructor(videoId) {
                this.video = document.getElementById(videoId);
                this.playBtn = document.getElementById('play-btn');
                this.pauseBtn = document.getElementById('pause-btn');
                this.stopBtn = document.getElementById('stop-btn');
                this.muteBtn = document.getElementById('mute-btn');
                this.progressContainer = document.getElementById('progress-container');
                this.progressBar = document.getElementById('progress-bar');
                this.currentTimeDisplay = document.getElementById('current-time');
                this.durationDisplay = document.getElementById('duration');
                this.volumeSlider = document.getElementById('volume-slider');
                
                this.setupEventListeners();
                this.updateTimeDisplays();
            }
            
            setupEventListeners() {
                this.playBtn.addEventListener('click', () => this.play());
                this.pauseBtn.addEventListener('click', () => this.pause());
                this.stopBtn.addEventListener('click', () => this.stop());
                this.muteBtn.addEventListener('click', () => this.toggleMute());
                
                this.progressContainer.addEventListener('click', (e) => {
                    this.seekTo(e);
                });
                
                this.volumeSlider.addEventListener('input', (e) => {
                    this.setVolume(e.target.value);
                });
                
                this.video.addEventListener('timeupdate', () => {
                    this.updateProgress();
                    this.updateTimeDisplays();
                });
                
                this.video.addEventListener('loadedmetadata', () => {
                    this.updateDuration();
                });
                
                this.video.addEventListener('ended', () => {
                    this.onEnded();
                });
            }
            
            play() {
                this.video.play().catch(error => {
                    console.error('Ошибка воспроизведения:', error);
                });
                this.playBtn.textContent = '⏸ Пауза';
            }
            
            pause() {
                this.video.pause();
                this.playBtn.textContent = '▶ Воспроизвести';
            }
            
            stop() {
                this.video.pause();
                this.video.currentTime = 0;
                this.playBtn.textContent = '▶ Воспроизвести';
            }
            
            toggleMute() {
                this.video.muted = !this.video.muted;
                this.muteBtn.textContent = this.video.muted ? '🔈 Звук' : '🔇 Звук';
            }
            
            seekTo(event) {
                const rect = this.progressContainer.getBoundingClientRect();
                const pos = (event.pageX - rect.left) / rect.width;
                this.video.currentTime = pos * this.video.duration;
            }
            
            setVolume(volume) {
                this.video.volume = parseFloat(volume);
            }
            
            updateProgress() {
                const percentage = (this.video.currentTime / this.video.duration) * 100;
                this.progressBar.style.width = `${percentage}%`;
            }
            
            updateTimeDisplays() {
                this.currentTimeDisplay.textContent = this.formatTime(this.video.currentTime);
            }
            
            updateDuration() {
                this.durationDisplay.textContent = this.formatTime(this.video.duration);
            }
            
            formatTime(seconds) {
                if (isNaN(seconds)) return '00:00';
                
                const minutes = Math.floor(seconds / 60);
                const remainingSeconds = Math.floor(seconds % 60);
                
                return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
            }
            
            onEnded() {
                console.log('Воспроизведение завершено');
                this.playBtn.textContent = '▶ Воспроизвести';
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MediaPlayer('video-player');
        });
    </script>
</body>
</html>
```

### Субтитры и текстовые дорожки

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Субтитры и текстовые дорожки</title>
</head>
<body>
    <h1>Видео с субтитрами</h1>
    
    <video controls width="800" height="450">
        <source src="movie.mp4" type="video/mp4">
        
        <!-- Субтитры -->
        <track kind="subtitles" 
               src="subtitles-ru.vtt" 
               srclang="ru" 
               label="Русские"
               default>
        
        <track kind="subtitles" 
               src="subtitles-en.vtt" 
               srclang="en" 
               label="English">
        
        <!-- Титры -->
        <track kind="captions" 
               src="captions.vtt" 
               srclang="ru" 
               label="Титры для слабослышащих">
        
        <!-- Описания для слабовидящих -->
        <track kind="descriptions" 
               src="descriptions.vtt" 
               srclang="ru" 
               label="Описания">
        
        <!-- Главы -->
        <track kind="chapters" 
               src="chapters.vtt" 
               srclang="ru" 
               label="Главы">
        
        Ваш браузер не поддерживает видео элемент.
    </video>
    
    <script>
        // Управление текстовыми дорожками
        const video = document.querySelector('video');
        
        function setupTrackControls() {
            const tracks = video.textTracks;
            
            for (let i = 0; i < tracks.length; i++) {
                const track = tracks[i];
                
                console.log(`Дорожка ${i}: ${track.label} (${track.language}) - ${track.kind}`);
                
                // Слушаем изменения состояния дорожки
                track.addEventListener('cuechange', function() {
                    if (this.activeCues.length > 0) {
                        console.log('Активный текст:', this.activeCues[0].text);
                    }
                });
            }
        }
        
        // Управление отображением субтитров
        function toggleSubtitles(language) {
            const tracks = video.textTracks;
            
            for (let i = 0; i < tracks.length; i++) {
                const track = tracks[i];
                track.mode = (track.language === language) ? 'showing' : 'hidden';
            }
        }
        
        // Инициализация
        video.addEventListener('loadeddata', setupTrackControls);
    </script>
</body>
</html>
```

## Адаптивные видео и аудио

### Адаптивное видео

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Адаптивные мультимедиа</title>
    <style>
        .responsive-video-container {
            position: relative;
            width: 100%;
            height: 0;
            padding-bottom: 56.25%; /* 16:9 соотношение */
        }
        
        .responsive-video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }
        
        .video-wrapper {
            max-width: 100%;
            margin: 0 auto;
        }
        
        /* Адаптация под разные размеры экрана */
        @media (max-width: 768px) {
            .video-wrapper {
                max-width: 100%;
            }
        }
        
        @media (min-width: 769px) and (max-width: 1024px) {
            .video-wrapper {
                max-width: 800px;
            }
        }
        
        @media (min-width: 1025px) {
            .video-wrapper {
                max-width: 1200px;
            }
        }
    </style>
</head>
<body>
    <h1>Адаптивное видео</h1>
    
    <div class="video-wrapper">
        <div class="responsive-video-container">
            <video class="responsive-video" controls>
                <source src="responsive-video.mp4" type="video/mp4">
                <source src="responsive-video.webm" type="video/webm">
                Ваш браузер не поддерживает видео элемент.
            </video>
        </div>
    </div>
    
    <h2>Адаптивное аудио</h2>
    
    <div class="audio-wrapper">
        <audio controls style="width: 100%; max-width: 600px;">
            <source src="responsive-audio.mp3" type="audio/mpeg">
            <source src="responsive-audio.ogg" type="audio/ogg">
            Ваш браузер не поддерживает аудио элемент.
        </audio>
    </div>
</body>
</html>
```

### Видео с различными качествами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Видео с выбором качества</title>
</head>
<body>
    <h1>Видео с выбором качества</h1>
    
    <div class="video-quality-container">
        <video id="quality-video" controls>
            <!-- Качество по умолчанию -->
            <source src="video-720p.mp4" type="video/mp4" data-quality="720p">
            
            <!-- Альтернативные качества -->
            <source src="video-1080p.mp4" type="video/mp4" data-quality="1080p">
            <source src="video-480p.mp4" type="video/mp4" data-quality="480p">
        </video>
        
        <div class="quality-controls">
            <label for="quality-select">Качество:</label>
            <select id="quality-select">
                <option value="480p">480p</option>
                <option value="720p" selected>720p</option>
                <option value="1080p">1080p</option>
            </select>
        </div>
    </div>

    <script>
        class QualityVideoPlayer {
            constructor(videoId, qualitySelectorId) {
                this.video = document.getElementById(videoId);
                this.qualitySelector = document.getElementById(qualitySelectorId);
                this.sources = Array.from(this.video.querySelectorAll('source'));
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.qualitySelector.addEventListener('change', (e) => {
                    this.changeQuality(e.target.value);
                });
            }
            
            changeQuality(quality) {
                // Находим источник с нужным качеством
                const source = this.sources.find(s => s.dataset.quality === quality);
                
                if (source) {
                    // Запоминаем текущее время воспроизведения
                    const currentTime = this.video.currentTime;
                    const isPlaying = !this.video.paused;
                    
                    // Меняем источник
                    this.video.src = source.src;
                    
                    // Восстанавливаем состояние воспроизведения
                    this.video.currentTime = currentTime;
                    
                    if (isPlaying) {
                        this.video.play();
                    }
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new QualityVideoPlayer('quality-video', 'quality-select');
        });
    </script>
</body>
</html>
```

## Современные форматы и кодеки

### Современные видео форматы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Современные форматы видео</title>
</head>
<body>
    <h1>Современные форматы видео</h1>
    
    <!-- VP9/WebM -->
    <video controls width="800" height="450">
        <source src="video-vp9.webm" type="video/webm; codecs=vp9">
        <source src="video-vp9.mp4" type="video/mp4; codecs=avc1.42E01E">
        Ваш браузер не поддерживает видео элемент.
    </video>
    
    <!-- AV1 -->
    <video controls width="800" height="450">
        <source src="video-av1.mp4" type="video/mp4; codecs=av01.0.05M.08">
        <source src="video-av1.webm" type="video/webm; codecs=av1">
        Ваш браузер не поддерживает видео элемент.
    </video>
    
    <!-- HEVC/H.265 -->
    <video controls width="800" height="450">
        <source src="video-hevc.mp4" type="video/mp4; codecs=hvc1.1.6.L120.90">
        <source src="video-h264.mp4" type="video/mp4; codecs=avc1.42E01E">
        Ваш браузер не поддерживает видео элемент.
    </video>

    <script>
        // Детекция поддержки кодеков
        class CodecDetector {
            static detectVideoCodecs() {
                const video = document.createElement('video');
                const codecs = {
                    h264: video.canPlayType('video/mp4; codecs="avc1.42E01E"'),
                    vp9: video.canPlayType('video/webm; codecs="vp9"'),
                    av1: video.canPlayType('video/mp4; codecs="av01.0.05M.08"'),
                    hevc: video.canPlayType('video/mp4; codecs="hvc1.1.6.L120.90"'),
                    ogg: video.canPlayType('video/ogg; codecs="theora"')
                };
                
                return codecs;
            }
            
            static detectAudioCodecs() {
                const audio = document.createElement('audio');
                const codecs = {
                    mp3: audio.canPlayType('audio/mpeg'),
                    aac: audio.canPlayType('audio/mp4; codecs="mp4a.40.2"'),
                    vorbis: audio.canPlayType('audio/ogg; codecs="vorbis"'),
                    opus: audio.canPlayType('audio/ogg; codecs="opus"'),
                    flac: audio.canPlayType('audio/flac')
                };
                
                return codecs;
            }
            
            static getBestSupportedFormat(formats) {
                const codecSupport = this.detectVideoCodecs();
                
                for (const format of formats) {
                    if (codecSupport[format.codec] !== '') {
                        return format;
                    }
                }
                
                return formats[0]; // Возврат первого формата как резервного
            }
        }
        
        // Пример использования
        const supportedCodecs = CodecDetector.detectVideoCodecs();
        console.log('Поддерживаемые кодеки:', supportedCodecs);
        
        const videoFormats = [
            { src: 'video-av1.mp4', type: 'video/mp4', codec: 'av1' },
            { src: 'video-vp9.webm', type: 'video/webm', codec: 'vp9' },
            { src: 'video-h264.mp4', type: 'video/mp4', codec: 'h264' }
        ];
        
        const bestFormat = CodecDetector.getBestSupportedFormat(videoFormats);
        console.log('Лучший поддерживаемый формат:', bestFormat);
    </script>
</body>
</html>
```

### Аудио с современными форматами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Современные аудио форматы</title>
</head>
<body>
    <h1>Современные аудио форматы</h1>
    
    <!-- AAC -->
    <audio controls>
        <source src="audio-aac.m4a" type="audio/mp4; codecs='mp4a.40.2'">
        <source src="audio-mp3.mp3" type="audio/mpeg">
        Ваш браузер не поддерживает аудио элемент.
    </audio>
    
    <!-- Opus -->
    <audio controls>
        <source src="audio-opus.oga" type="audio/ogg; codecs='opus'">
        <source src="audio-vorbis.oga" type="audio/ogg; codecs='vorbis'">
        <source src="audio-mp3.mp3" type="audio/mpeg">
        Ваш браузер не поддерживает аудио элемент.
    </audio>
    
    <!-- FLAC (без потерь) -->
    <audio controls>
        <source src="audio-flac.flac" type="audio/flac">
        <source src="audio-wav.wav" type="audio/wav">
        <source src="audio-mp3.mp3" type="audio/mpeg">
        Ваш браузер не поддерживает аудио элемент.
    </audio>
</body>
</html>
```

## Аудио- и видеоплееры

### Простой аудиоплеер

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Простой аудиоплеер</title>
    <style>
        .audio-player {
            max-width: 500px;
            margin: 20px auto;
            padding: 20px;
            background: #f5f5f5;
            border-radius: 8px;
            font-family: Arial, sans-serif;
        }
        
        .now-playing {
            text-align: center;
            margin-bottom: 20px;
        }
        
        .track-info {
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .player-controls {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-bottom: 15px;
        }
        
        .control-btn {
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            background-color: #007acc;
            color: white;
        }
        
        .control-btn:hover {
            background-color: #005a9e;
        }
        
        .control-btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .progress-container {
            width: 100%;
            height: 6px;
            background-color: #ddd;
            border-radius: 3px;
            cursor: pointer;
            margin: 10px 0;
        }
        
        .progress-bar {
            height: 100%;
            background-color: #007acc;
            border-radius: 3px;
            width: 0%;
        }
        
        .time-display {
            text-align: center;
            font-size: 0.9em;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="audio-player-container">
        <h1>Музыкальный плеер</h1>
        
        <div class="audio-player">
            <div class="now-playing">
                <div class="track-info" id="track-info">Нет активного трека</div>
                <div class="track-album" id="track-album"></div>
            </div>
            
            <audio id="audio-player" preload="metadata">
                <source src="track1.mp3" type="audio/mpeg">
            </audio>
            
            <div class="player-controls">
                <button id="prev-btn" class="control-btn" disabled>⏮</button>
                <button id="play-btn" class="control-btn">▶</button>
                <button id="pause-btn" class="control-btn" disabled>⏸</button>
                <button id="next-btn" class="control-btn" disabled>⏭</button>
            </div>
            
            <div class="progress-container" id="progress-container">
                <div class="progress-bar" id="progress-bar"></div>
            </div>
            
            <div class="time-display">
                <span id="current-time">0:00</span> / <span id="total-time">0:00</span>
            </div>
            
            <div class="playlist-controls">
                <label for="playlist-select">Выберите трек:</label>
                <select id="playlist-select">
                    <option value="track1.mp3" data-title="Трек 1" data-artist="Исполнитель 1">Трек 1 - Исполнитель 1</option>
                    <option value="track2.mp3" data-title="Трек 2" data-artist="Исполнитель 2">Трек 2 - Исполнитель 2</option>
                    <option value="track3.mp3" data-title="Трек 3" data-artist="Исполнитель 3">Трек 3 - Исполнитель 3</option>
                </select>
            </div>
        </div>
    </div>

    <script>
        class AudioPlayer {
            constructor(playerId, playlistId) {
                this.audio = document.getElementById(playerId);
                this.playlist = document.getElementById(playlistId);
                this.playBtn = document.getElementById('play-btn');
                this.pauseBtn = document.getElementById('pause-btn');
                this.prevBtn = document.getElementById('prev-btn');
                this.nextBtn = document.getElementById('next-btn');
                this.progressContainer = document.getElementById('progress-container');
                this.progressBar = document.getElementById('progress-bar');
                this.currentTimeDisplay = document.getElementById('current-time');
                this.totalTimeDisplay = document.getElementById('total-time');
                this.trackInfo = document.getElementById('track-info');
                this.trackAlbum = document.getElementById('track-album');
                
                this.tracks = Array.from(this.playlist.options).map(option => ({
                    src: option.value,
                    title: option.dataset.title,
                    artist: option.dataset.artist
                }));
                
                this.currentTrackIndex = 0;
                
                this.setupEventListeners();
                this.updateTrackInfo();
            }
            
            setupEventListeners() {
                this.playBtn.addEventListener('click', () => this.play());
                this.pauseBtn.addEventListener('click', () => this.pause());
                
                this.progressContainer.addEventListener('click', (e) => {
                    this.seekTo(e);
                });
                
                this.audio.addEventListener('timeupdate', () => {
                    this.updateProgress();
                    this.updateTimeDisplay();
                });
                
                this.audio.addEventListener('loadedmetadata', () => {
                    this.updateTotalTime();
                });
                
                this.audio.addEventListener('ended', () => {
                    this.nextTrack();
                });
                
                this.playlist.addEventListener('change', (e) => {
                    this.changeTrack(parseInt(e.target.selectedIndex));
                });
            }
            
            play() {
                this.audio.play();
                this.playBtn.disabled = true;
                this.pauseBtn.disabled = false;
            }
            
            pause() {
                this.audio.pause();
                this.playBtn.disabled = false;
                this.pauseBtn.disabled = true;
            }
            
            nextTrack() {
                this.currentTrackIndex = (this.currentTrackIndex + 1) % this.tracks.length;
                this.loadCurrentTrack();
                this.play();
            }
            
            prevTrack() {
                this.currentTrackIndex = (this.currentTrackIndex - 1 + this.tracks.length) % this.tracks.length;
                this.loadCurrentTrack();
                this.play();
            }
            
            loadCurrentTrack() {
                this.audio.src = this.tracks[this.currentTrackIndex].src;
                this.updateTrackInfo();
                this.audio.load();
            }
            
            changeTrack(index) {
                this.currentTrackIndex = index;
                this.loadCurrentTrack();
                this.play();
            }
            
            updateProgress() {
                const percentage = (this.audio.currentTime / this.audio.duration) * 100;
                this.progressBar.style.width = `${percentage}%`;
            }
            
            updateTimeDisplay() {
                this.currentTimeDisplay.textContent = this.formatTime(this.audio.currentTime);
            }
            
            updateTotalTime() {
                this.totalTimeDisplay.textContent = this.formatTime(this.audio.duration);
            }
            
            updateTrackInfo() {
                const track = this.tracks[this.currentTrackIndex];
                this.trackInfo.textContent = track.title;
                this.trackAlbum.textContent = track.artist;
                
                // Обновляем выбор в плейлисте
                this.playlist.selectedIndex = this.currentTrackIndex;
            }
            
            formatTime(seconds) {
                if (isNaN(seconds)) return '0:00';
                
                const mins = Math.floor(seconds / 60);
                const secs = Math.floor(seconds % 60);
                
                return `${mins}:${secs.toString().padStart(2, '0')}`;
            }
            
            seekTo(event) {
                const rect = this.progressContainer.getBoundingClientRect();
                const pos = (event.offsetX / rect.width);
                this.audio.currentTime = pos * this.audio.duration;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AudioPlayer('audio-player', 'playlist-select');
        });
    </script>
</body>
</html>
```

### Продвинутый видеоплеер

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Продвинутый видеоплеер</title>
    <style>
        .video-player-container {
            max-width: 800px;
            margin: 20px auto;
            background: #000;
            border-radius: 8px;
            overflow: hidden;
            position: relative;
        }
        
        .video-player {
            width: 100%;
            display: block;
        }
        
        .player-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            background: rgba(0,0,0,0.5);
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .player-overlay:hover {
            opacity: 1;
        }
        
        .play-pause-btn {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            background: rgba(255,255,255,0.2);
            border: 2px solid white;
            color: white;
            font-size: 24px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .player-controls {
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 10px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .control-btn {
            background: none;
            border: none;
            color: white;
            font-size: 16px;
            cursor: pointer;
            padding: 5px;
        }
        
        .control-btn:hover {
            color: #007acc;
        }
        
        .progress-container {
            flex: 1;
            height: 5px;
            background: rgba(255,255,255,0.3);
            border-radius: 3px;
            cursor: pointer;
            position: relative;
        }
        
        .progress-bar {
            height: 100%;
            background: #007acc;
            border-radius: 3px;
            width: 0%;
        }
        
        .progress-buffer {
            position: absolute;
            top: 0;
            left: 0;
            height: 100%;
            background: rgba(255,255,255,0.3);
            border-radius: 3px;
        }
        
        .time-display {
            font-size: 12px;
            min-width: 80px;
            text-align: center;
        }
        
        .settings-menu {
            position: absolute;
            bottom: 50px;
            right: 10px;
            background: rgba(0,0,0,0.9);
            border-radius: 4px;
            padding: 10px;
            display: none;
        }
        
        .settings-menu.active {
            display: block;
        }
        
        .setting-option {
            color: white;
            text-decoration: none;
            display: block;
            padding: 5px;
        }
        
        .setting-option:hover {
            background: rgba(255,255,255,0.1);
        }
    </style>
</head>
<body>
    <div class="video-player-container">
        <video id="advanced-video-player" class="video-player" preload="metadata">
            <source src="sample-video.mp4" type="video/mp4">
            <source src="sample-video.webm" type="video/webm">
            Ваш браузер не поддерживает видео элемент.
        </video>
        
        <div class="player-overlay" id="player-overlay">
            <button id="play-pause-overlay" class="play-pause-btn">▶</button>
        </div>
        
        <div class="player-controls">
            <button id="play-pause-btn" class="control-btn">▶</button>
            <button id="fullscreen-btn" class="control-btn">⛶</button>
            <button id="settings-btn" class="control-btn">⚙️</button>
            
            <div class="progress-container" id="progress-container">
                <div class="progress-bar" id="progress-bar"></div>
                <div class="progress-buffer" id="progress-buffer"></div>
            </div>
            
            <div class="time-display">
                <span id="current-time">00:00</span> / <span id="duration">00:00</span>
            </div>
        </div>
        
        <div class="settings-menu" id="settings-menu">
            <a href="#" class="setting-option" data-setting="quality">Качество</a>
            <a href="#" class="setting-option" data-setting="speed">Скорость</a>
            <a href="#" class="setting-option" data-setting="captions">Субтитры</a>
        </div>
    </div>

    <script>
        class AdvancedVideoPlayer {
            constructor(videoId) {
                this.video = document.getElementById(videoId);
                this.overlay = document.getElementById('player-overlay');
                this.playPauseBtn = document.getElementById('play-pause-btn');
                this.playPauseOverlay = document.getElementById('play-pause-overlay');
                this.fullscreenBtn = document.getElementById('fullscreen-btn');
                this.settingsBtn = document.getElementById('settings-btn');
                this.settingsMenu = document.getElementById('settings-menu');
                this.progressContainer = document.getElementById('progress-container');
                this.progressBar = document.getElementById('progress-bar');
                this.progressBuffer = document.getElementById('progress-buffer');
                this.currentTimeDisplay = document.getElementById('current-time');
                this.durationDisplay = document.getElementById('duration');
                
                this.isPlaying = false;
                this.isFullscreen = false;
                
                this.setupEventListeners();
                this.updateTimeDisplays();
            }
            
            setupEventListeners() {
                this.playPauseBtn.addEventListener('click', () => this.togglePlayPause());
                this.playPauseOverlay.addEventListener('click', () => this.togglePlayPause());
                
                this.fullscreenBtn.addEventListener('click', () => this.toggleFullscreen());
                
                this.settingsBtn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    this.toggleSettingsMenu();
                });
                
                document.addEventListener('click', (e) => {
                    if (!this.settingsMenu.contains(e.target) && !this.settingsBtn.contains(e.target)) {
                        this.hideSettingsMenu();
                    }
                });
                
                this.progressContainer.addEventListener('click', (e) => {
                    this.seekTo(e);
                });
                
                this.video.addEventListener('timeupdate', () => {
                    this.updateProgress();
                    this.updateTimeDisplays();
                });
                
                this.video.addEventListener('progress', () => {
                    this.updateBufferProgress();
                });
                
                this.video.addEventListener('loadedmetadata', () => {
                    this.updateDuration();
                });
                
                this.video.addEventListener('play', () => {
                    this.isPlaying = true;
                    this.updatePlayPauseButtons();
                });
                
                this.video.addEventListener('pause', () => {
                    this.isPlaying = false;
                    this.updatePlayPauseButtons();
                });
                
                this.video.addEventListener('ended', () => {
                    this.onVideoEnded();
                });
                
                // Обработка событий полноэкранного режима
                document.addEventListener('fullscreenchange', () => {
                    this.isFullscreen = !!document.fullscreenElement;
                    this.updateFullscreenButton();
                });
            }
            
            togglePlayPause() {
                if (this.video.paused) {
                    this.video.play();
                } else {
                    this.video.pause();
                }
            }
            
            updatePlayPauseButtons() {
                const icon = this.isPlaying ? '⏸' : '▶';
                this.playPauseBtn.textContent = icon;
                this.playPauseOverlay.textContent = icon;
            }
            
            toggleFullscreen() {
                if (!document.fullscreenElement) {
                    this.enterFullscreen();
                } else {
                    this.exitFullscreen();
                }
            }
            
            enterFullscreen() {
                if (this.video.requestFullscreen) {
                    this.video.requestFullscreen();
                } else if (this.video.mozRequestFullScreen) { /* Firefox */
                    this.video.mozRequestFullScreen();
                } else if (this.video.webkitRequestFullscreen) { /* Chrome, Safari & Opera */
                    this.video.webkitRequestFullscreen();
                } else if (this.video.msRequestFullscreen) { /* IE/Edge */
                    this.video.msRequestFullscreen();
                }
            }
            
            exitFullscreen() {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                } else if (document.mozCancelFullScreen) { /* Firefox */
                    document.mozCancelFullScreen();
                } else if (document.webkitExitFullscreen) { /* Chrome, Safari & Opera */
                    document.webkitExitFullscreen();
                } else if (document.msExitFullscreen) { /* IE/Edge */
                    document.msExitFullscreen();
                }
            }
            
            updateFullscreenButton() {
                this.fullscreenBtn.textContent = this.isFullscreen ? '⛶' : '⛶';
            }
            
            toggleSettingsMenu() {
                this.settingsMenu.classList.toggle('active');
            }
            
            hideSettingsMenu() {
                this.settingsMenu.classList.remove('active');
            }
            
            seekTo(event) {
                const rect = this.progressContainer.getBoundingClientRect();
                const pos = (event.offsetX / rect.width);
                this.video.currentTime = pos * this.video.duration;
            }
            
            updateProgress() {
                const percentage = (this.video.currentTime / this.video.duration) * 100;
                this.progressBar.style.width = `${percentage}%`;
            }
            
            updateBufferProgress() {
                if (this.video.buffered.length > 0) {
                    const bufferedEnd = this.video.buffered.end(this.video.buffered.length - 1);
                    const bufferPercentage = (bufferedEnd / this.video.duration) * 100;
                    this.progressBuffer.style.width = `${bufferPercentage}%`;
                }
            }
            
            updateTimeDisplays() {
                this.currentTimeDisplay.textContent = this.formatTime(this.video.currentTime);
            }
            
            updateDuration() {
                this.durationDisplay.textContent = this.formatTime(this.video.duration);
            }
            
            formatTime(seconds) {
                if (isNaN(seconds)) return '00:00';
                
                const hours = Math.floor(seconds / 3600);
                const minutes = Math.floor((seconds % 3600) / 60);
                const remainingSeconds = Math.floor(seconds % 60);
                
                if (hours > 0) {
                    return `${hours}:${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
                } else {
                    return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`;
                }
            }
            
            onVideoEnded() {
                console.log('Видео завершено');
                // Здесь можно добавить логику для следующего видео или повтора
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AdvancedVideoPlayer('advanced-video-player');
        });
    </script>
</body>
</html>
```

## Доступность мультимедиа

### Доступные медиа-элементы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные мультимедиа</title>
</head>
<body>
    <h1>Доступные видео и аудио элементы</h1>
    
    <div role="region" aria-label="Видео проигрыватель">
        <video controls 
               width="800" 
               height="450"
               aria-describedby="video-description"
               aria-label="Обучающее видео по HTML">
            <source src="tutorial.mp4" type="video/mp4">
            <source src="tutorial.webm" type="video/webm">
            
            <!-- Субтитры -->
            <track kind="subtitles" 
                   src="subtitles-ru.vtt" 
                   srclang="ru" 
                   label="Русские"
                   default>
            
            <!-- Описания для слабовидящих -->
            <track kind="descriptions" 
                   src="descriptions.vtt" 
                   srclang="ru" 
                   label="Описания">
            
            Ваш браузер не поддерживает видео элемент.
        </video>
        
        <div id="video-description">
            Обучающее видео о возможностях HTML5 для включения мультимедиа.
        </div>
    </div>
    
    <div role="region" aria-label="Аудио проигрыватель">
        <audio controls aria-describedby="audio-description">
            <source src="podcast.mp3" type="audio/mpeg">
            <source src="podcast.ogg" type="audio/ogg">
            Ваш браузер не поддерживает аудио элемент.
        </audio>
        
        <div id="audio-description">
            Подкаст о веб-технологиях и современных подходах к разработке.
        </div>
    </div>
    
    <!-- Транскрипция аудио -->
    <details>
        <summary>Транскрипция подкаста</summary>
        <div class="transcription">
            <p><strong>Ведущий:</strong> Добро пожаловать на наш подкаст о веб-технологиях...</p>
            <p><strong>Гость:</strong> Сегодня мы поговорим о последних тенденциях в веб-разработке...</p>
            <!-- Полная транскрипция -->
        </div>
    </details>

    <script>
        class AccessibleMediaPlayer {
            constructor(mediaElement) {
                this.media = mediaElement;
                this.setupAccessibilityFeatures();
            }
            
            setupAccessibilityFeatures() {
                // Добавляем ARIA-атрибуты
                this.media.setAttribute('role', 'application');
                this.media.setAttribute('aria-label', 'Медиа проигрыватель');
                
                // Создаем доступные элементы управления
                this.createAccessibleControls();
                
                // Обработка клавиатурных событий
                this.setupKeyboardNavigation();
            }
            
            createAccessibleControls() {
                // Создаем элементы управления с правильной семантикой
                const controlsContainer = document.createElement('div');
                controlsContainer.className = 'accessible-controls';
                controlsContainer.setAttribute('role', 'toolbar');
                controlsContainer.setAttribute('aria-label', 'Элементы управления медиа');
                
                const playButton = document.createElement('button');
                playButton.textContent = 'Воспроизвести';
                playButton.setAttribute('aria-label', 'Воспроизвести медиа');
                playButton.addEventListener('click', () => this.media.play());
                
                const pauseButton = document.createElement('button');
                pauseButton.textContent = 'Пауза';
                pauseButton.setAttribute('aria-label', 'Приостановить воспроизведение');
                pauseButton.addEventListener('click', () => this.media.pause());
                
                controlsContainer.appendChild(playButton);
                controlsContainer.appendChild(pauseButton);
                
                // Вставляем после медиа элемента
                this.media.parentNode.insertBefore(controlsContainer, this.media.nextSibling);
            }
            
            setupKeyboardNavigation() {
                this.media.addEventListener('keydown', (e) => {
                    switch(e.key) {
                        case ' ':
                        case 'Spacebar':
                            e.preventDefault();
                            if (this.media.paused) {
                                this.media.play();
                            } else {
                                this.media.pause();
                            }
                            break;
                        case 'ArrowLeft':
                            e.preventDefault();
                            this.media.currentTime = Math.max(0, this.media.currentTime - 10);
                            break;
                        case 'ArrowRight':
                            e.preventDefault();
                            this.media.currentTime = Math.min(this.media.duration, this.media.currentTime + 10);
                            break;
                        case 'Home':
                            e.preventDefault();
                            this.media.currentTime = 0;
                            break;
                        case 'End':
                            e.preventDefault();
                            this.media.currentTime = this.media.duration;
                            break;
                    }
                });
                
                // Устанавливаем tabIndex для доступности с клавиатуры
                this.media.setAttribute('tabindex', '0');
            }
        }
        
        // Инициализация для всех медиа элементов
        document.addEventListener('DOMContentLoaded', () => {
            const videos = document.querySelectorAll('video');
            const audios = document.querySelectorAll('audio');
            
            videos.forEach(video => new AccessibleMediaPlayer(video));
            audios.forEach(audio => new AccessibleMediaPlayer(audio));
        });
    </script>
    
    <style>
        .accessible-controls {
            margin: 10px 0;
        }
        
        .accessible-controls button {
            margin-right: 10px;
            padding: 8px 16px;
            border: 2px solid #007acc;
            background: white;
            color: #007acc;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .accessible-controls button:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
        
        .transcription {
            margin-top: 20px;
            padding: 15px;
            background-color: #f9f9f9;
            border-left: 4px solid #007acc;
        }
    </style>
</body>
</html>
```

## Практические примеры

### Видео-галерея

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Видео-галерея</title>
    <style>
        .video-gallery {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            padding: 20px;
        }
        
        .video-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        
        .video-thumbnail {
            width: 100%;
            height: 200px;
            object-fit: cover;
            cursor: pointer;
        }
        
        .video-info {
            padding: 15px;
        }
        
        .video-title {
            margin: 0 0 10px 0;
            font-size: 1.1em;
        }
        
        .video-description {
            color: #666;
            margin-bottom: 10px;
            font-size: 0.9em;
        }
        
        .video-meta {
            font-size: 0.8em;
            color: #888;
        }
        
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.8);
            z-index: 1000;
            align-items: center;
            justify-content: center;
        }
        
        .modal-content {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            max-width: 90%;
            max-height: 90%;
        }
        
        .modal-video {
            width: 100%;
            max-width: 800px;
        }
        
        .close-modal {
            position: absolute;
            top: 10px;
            right: 15px;
            font-size: 2em;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <h1>Видео-галерея</h1>
    
    <div class="video-gallery" id="video-gallery">
        <!-- Видео карточки будут добавлены сюда -->
    </div>
    
    <div class="modal" id="video-modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal-content">
            <span class="close-modal" aria-label="Закрыть">&times;</span>
            <h2 id="modal-title">Видеопроигрыватель</h2>
            <video id="modal-video" class="modal-video" controls></video>
        </div>
    </div>

    <script>
        class VideoGallery {
            constructor(containerId, modalId) {
                this.container = document.getElementById(containerId);
                this.modal = document.getElementById(modalId);
                this.modalVideo = document.getElementById('modal-video');
                this.closeBtn = this.modal.querySelector('.close-modal');
                
                this.videos = [
                    {
                        id: 1,
                        title: 'Введение в HTML5',
                        description: 'Основы HTML5 и его новых возможностей',
                        thumbnail: 'thumb1.jpg',
                        source: 'video1.mp4',
                        duration: '12:34',
                        views: '1,234'
                    },
                    {
                        id: 2,
                        title: 'CSS Grid и Flexbox',
                        description: 'Современные подходы к верстке',
                        thumbnail: 'thumb2.jpg',
                        source: 'video2.mp4',
                        duration: '18:22',
                        views: '2,567'
                    },
                    {
                        id: 3,
                        title: 'JavaScript ES6+',
                        description: 'Современные возможности JavaScript',
                        thumbnail: 'thumb3.jpg',
                        source: 'video3.mp4',
                        duration: '25:41',
                        views: '3,890'
                    }
                ];
                
                this.setupEventListeners();
                this.renderGallery();
            }
            
            setupEventListeners() {
                this.closeBtn.addEventListener('click', () => this.closeModal());
                
                // Закрытие модального окна при клике вне видео
                this.modal.addEventListener('click', (e) => {
                    if (e.target === this.modal) {
                        this.closeModal();
                    }
                });
                
                // Закрытие при нажатии Escape
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'Escape' && this.modal.style.display === 'flex') {
                        this.closeModal();
                    }
                });
            }
            
            renderGallery() {
                this.videos.forEach(video => {
                    const card = document.createElement('div');
                    card.className = 'video-card';
                    card.innerHTML = `
                        <img src="${video.thumbnail}" 
                             alt="${video.title}" 
                             class="video-thumbnail"
                             onclick="gallery.openVideo(${video.id})">
                        <div class="video-info">
                            <h3 class="video-title">${video.title}</h3>
                            <p class="video-description">${video.description}</p>
                            <div class="video-meta">
                                <span>${video.duration}</span> • <span>${video.views} просмотров</span>
                            </div>
                        </div>
                    `;
                    
                    this.container.appendChild(card);
                });
            }
            
            openVideo(videoId) {
                const video = this.videos.find(v => v.id === videoId);
                
                if (video) {
                    this.modalVideo.src = video.source;
                    document.getElementById('modal-title').textContent = video.title;
                    
                    // Загружаем видео и показываем модальное окно
                    this.modalVideo.load();
                    this.modal.style.display = 'flex';
                }
            }
            
            closeModal() {
                this.modal.style.display = 'none';
                this.modalVideo.pause();
                this.modalVideo.src = '';
            }
        }
        
        // Инициализация
        let gallery;
        document.addEventListener('DOMContentLoaded', () => {
            gallery = new VideoGallery('video-gallery', 'video-modal');
        });
    </script>
</body>
</html>
```

## Заключение

Элементы `<video>` и `<audio>` в HTML предоставляют мощные возможности для включения мультимедийного контента на веб-страницы. Современные браузеры поддерживают различные форматы, кодеки и возможности для кастомизации. Правильное использование этих элементов с учетом доступности, адаптивности и производительности позволяет создавать богатые мультимедийные веб-приложения.

Ключевые аспекты работы с мультимедиа в HTML:
1. Поддержка различных форматов для кросс-браузерной совместимости
2. Использование субтитров и текстовых дорожек для доступности
3. Адаптивная разметка для различных устройств
4. Эффективное управление воспроизведением
5. Современные форматы и кодеки для оптимального качества
6. Безопасность и защита от XSS атак
7. Интеграция с JavaScript для расширенного функционала
8. Доступность для пользователей с ограниченными возможностями

Эти элементы являются важной частью современного веб-дизайна и позволяют создавать интерактивные и привлекательные веб-сайты с богатым мультимедийным контентом.

## Следующие темы
- [[HTML/Мультимедиа/Canvas и SVG]]
- [[HTML/Производительность]]
- [[HTML/Доступность]]

## Теги
#html #video #audio #multimedia #web-development #frontend #accessibility #subtitles #captions #webm #mp4 #ogg #av1 #vp9 #hevc #h264 #media-player #streaming #format-compatibility #cross-browser #user-experience