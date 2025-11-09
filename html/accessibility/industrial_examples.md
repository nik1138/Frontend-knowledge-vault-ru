# Промышленные примеры и лучшие практики доступности HTML

В этой статье рассматриваются реальные примеры из промышленной разработки, а также лучшие практики создания доступных веб-сайтов и приложений. Мы изучим, как крупные компании и проекты применяют принципы доступности в своих продуктах.

## Промышленные примеры

### 1. Google - доступный поиск

Google внедрил множество практик доступности в свой поисковый интерфейс:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступный поисковый интерфейс</title>
</head>
<body>
    <header role="banner">
        <h1>Доступный поиск</h1>
    </header>

    <main role="main">
        <form role="search" aria-label="Поиск по сайту">
            <div class="search-container">
                <label for="search-input" class="sr-only">Поиск</label>
                <input type="search"
                       id="search-input"
                       name="q"
                       aria-label="Введите поисковый запрос"
                       autocomplete="off"
                       aria-autocomplete="list"
                       aria-controls="search-suggestions">
                
                <button type="submit" aria-label="Выполнить поиск">
                    <svg width="20" height="20" viewBox="0 0 24 24" aria-hidden="true">
                        <path d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
                    </svg>
                </button>
            </div>

            <div id="search-suggestions"
                 role="listbox"
                 aria-label="Поисковые предложения"
                 hidden>
                <!-- Сюда будут добавляться предложения -->
            </div>
        </form>
    </main>

    <script>
        class AccessibleSearch {
            constructor() {
                this.searchInput = document.getElementById('search-input');
                this.suggestionsContainer = document.getElementById('search-suggestions');
                this.currentFocus = -1;

                this.setupEventListeners();
            }

            setupEventListeners() {
                this.searchInput.addEventListener('input', (e) => {
                    this.handleInput(e.target.value);
                });

                this.searchInput.addEventListener('keydown', (e) => {
                    this.handleKeydown(e);
                });

                // Предотвращение отправки формы при нажатии Enter в предложениях
                this.suggestionsContainer.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter') {
                        e.preventDefault();
                        this.selectSuggestion(this.currentFocus);
                    }
                });
            }

            async handleInput(query) {
                if (query.length > 2) {
                    // Симуляция поиска
                    const suggestions = await this.getSearchSuggestions(query);
                    this.displaySuggestions(suggestions);
                } else {
                    this.hideSuggestions();
                }
            }

            async getSearchSuggestions(query) {
                // В реальном приложении здесь будет вызов API
                return [
                    `${query} технологии`,
                    `${query} новости`,
                    `${query} обучение`,
                    `${query} примеры`
                ];
            }

            displaySuggestions(suggestions) {
                if (suggestions.length === 0) {
                    this.hideSuggestions();
                    return;
                }

                this.suggestionsContainer.innerHTML = suggestions.map((suggestion, index) => `
                    <div role="option"
                         id="suggestion-${index}"
                         class="suggestion-item"
                         tabindex="-1"
                         onclick="accessibleSearch.selectSuggestion(${index})"
                         onkeydown="accessibleSearch.handleSuggestionKeydown(event, ${index})">
                        ${suggestion}
                    </div>
                `).join('');

                this.suggestionsContainer.hidden = false;
                this.currentFocus = -1;
            }

            hideSuggestions() {
                this.suggestionsContainer.hidden = true;
                this.currentFocus = -1;
            }

            handleKeydown(e) {
                switch(e.key) {
                    case 'ArrowDown':
                        e.preventDefault();
                        this.currentFocus++;
                        this.focusSuggestion();
                        break;
                    case 'ArrowUp':
                        e.preventDefault();
                        this.currentFocus--;
                        this.focusSuggestion();
                        break;
                    case 'Enter':
                        e.preventDefault();
                        if (this.currentFocus > -1) {
                            this.selectSuggestion(this.currentFocus);
                        } else {
                            this.submitSearch();
                        }
                        break;
                    case 'Escape':
                        this.hideSuggestions();
                        break;
                }
            }

            handleSuggestionKeydown(e, index) {
                switch(e.key) {
                    case 'ArrowDown':
                        e.preventDefault();
                        this.currentFocus = index + 1;
                        this.focusSuggestion();
                        break;
                    case 'ArrowUp':
                        e.preventDefault();
                        this.currentFocus = index - 1;
                        this.focusSuggestion();
                        break;
                    case 'Enter':
                        e.preventDefault();
                        this.selectSuggestion(index);
                        break;
                }
            }

            focusSuggestion() {
                const suggestions = this.suggestionsContainer.querySelectorAll('[role="option"]');
                
                if (this.currentFocus >= suggestions.length) {
                    this.currentFocus = 0;
                }
                
                if (this.currentFocus < 0) {
                    this.currentFocus = suggestions.length - 1;
                }

                suggestions.forEach((suggestion, index) => {
                    if (index === this.currentFocus) {
                        suggestion.setAttribute('aria-selected', 'true');
                        suggestion.classList.add('focused');
                        suggestion.scrollIntoView({ block: 'nearest' });
                    } else {
                        suggestion.setAttribute('aria-selected', 'false');
                        suggestion.classList.remove('focused');
                    }
                });
            }

            selectSuggestion(index) {
                const suggestions = this.suggestionsContainer.querySelectorAll('[role="option"]');
                if (suggestions[index]) {
                    this.searchInput.value = suggestions[index].textContent;
                    this.hideSuggestions();
                    this.searchInput.focus();
                    this.submitSearch();
                }
            }

            submitSearch() {
                // В реальном приложении здесь будет отправка формы
                console.log('Поиск:', this.searchInput.value);
            }
        }

        // Инициализация
        const accessibleSearch = new AccessibleSearch();
    </script>

    <style>
        .search-container {
            position: relative;
            max-width: 600px;
            margin: 20px auto;
        }

        #search-input {
            width: 100%;
            padding: 12px 40px 12px 16px;
            font-size: 16px;
            border: 1px solid #dfe1e5;
            border-radius: 24px;
            outline: none;
        }

        #search-input:focus {
            border-color: #4d90fe;
            box-shadow: 0 1px 1px rgba(0,0,0,.1), 0 0 8px rgba(77,144,254,.5);
        }

        .search-container button {
            position: absolute;
            right: 12px;
            top: 50%;
            transform: translateY(-50%);
            background: none;
            border: none;
            cursor: pointer;
            padding: 4px;
        }

        .search-container button:focus {
            outline: 2px solid #4d90fe;
            border-radius: 50%;
        }

        #search-suggestions {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 1px solid #dfe1e5;
            border-top: none;
            border-radius: 0 0 24px 24px;
            z-index: 1000;
            max-height: 300px;
            overflow-y: auto;
        }

        .suggestion-item {
            padding: 12px 16px;
            cursor: pointer;
            border-bottom: 1px solid #f3f3f3;
        }

        .suggestion-item:last-child {
            border-bottom: none;
        }

        .suggestion-item:hover,
        .suggestion-item.focused {
            background-color: #f5f5f5;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

### 2. Microsoft - доступная навигация

Microsoft внедрила продвинутые паттерны навигации в своих продуктах:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная навигация Microsoft</title>
</head>
<body>
    <header role="banner">
        <div class="header-container">
            <h1>Доступная навигация</h1>
            
            <nav role="navigation" aria-label="Основная навигация">
                <ul class="main-nav" role="menubar">
                    <li role="none">
                        <a href="/" role="menuitem" aria-haspopup="true" aria-expanded="false">
                            Файл
                        </a>
                        
                        <ul class="submenu" role="menu" aria-label="Подменю Файл" hidden>
                            <li role="none">
                                <a href="/new" role="menuitem">Создать</a>
                            </li>
                            <li role="none">
                                <a href="/open" role="menuitem">Открыть</a>
                            </li>
                            <li role="separator"></li>
                            <li role="none">
                                <a href="/save" role="menuitem">Сохранить</a>
                            </li>
                            <li role="none">
                                <a href="/save-as" role="menuitem">Сохранить как...</a>
                            </li>
                        </ul>
                    </li>
                    
                    <li role="none">
                        <a href="/edit" role="menuitem" aria-haspopup="true" aria-expanded="false">
                            Правка
                        </a>
                        
                        <ul class="submenu" role="menu" aria-label="Подменю Правка" hidden>
                            <li role="none">
                                <a href="/cut" role="menuitem">Вырезать</a>
                            </li>
                            <li role="none">
                                <a href="/copy" role="menuitem">Копировать</a>
                            </li>
                            <li role="none">
                                <a href="/paste" role="menuitem">Вставить</a>
                            </li>
                        </ul>
                    </li>
                    
                    <li role="none">
                        <a href="/view" role="menuitem" aria-haspopup="true" aria-expanded="false">
                            Вид
                        </a>
                        
                        <ul class="submenu" role="menu" aria-label="Подменю Вид" hidden>
                            <li role="none">
                                <a href="/zoom-in" role="menuitem">Увеличить</a>
                            </li>
                            <li role="none">
                                <a href="/zoom-out" role="menuitem">Уменьшить</a>
                            </li>
                            <li role="separator"></li>
                            <li role="none">
                                <a href="/full-screen" role="menuitem">Полноэкранный режим</a>
                            </li>
                        </ul>
                    </li>
                </ul>
            </nav>
        </div>
    </header>

    <main role="main">
        <h2>Содержимое страницы</h2>
        <p>Это основное содержимое страницы с доступной навигацией.</p>
    </main>

    <script>
        class AccessibleMenu {
            constructor() {
                this.menuItems = document.querySelectorAll('.main-nav [role="menuitem"]');
                this.submenus = document.querySelectorAll('.submenu');
                
                this.currentMenu = null;
                this.currentItem = null;
                
                this.setupEventListeners();
            }

            setupEventListeners() {
                this.menuItems.forEach(item => {
                    item.addEventListener('click', (e) => {
                        this.handleItemClick(e, item);
                    });
                    
                    item.addEventListener('keydown', (e) => {
                        this.handleItemKeydown(e, item);
                    });
                    
                    item.addEventListener('mouseenter', (e) => {
                        this.handleItemHover(e, item);
                    });
                });
                
                // Закрытие меню при клике вне его
                document.addEventListener('click', (e) => {
                    if (!e.target.closest('.main-nav')) {
                        this.closeAllMenus();
                    }
                });
            }

            handleItemClick(e, item) {
                e.preventDefault();
                
                const submenu = item.nextElementSibling;
                
                if (submenu && submenu.classList.contains('submenu')) {
                    if (submenu.hidden) {
                        this.openMenu(item, submenu);
                    } else {
                        this.closeMenu(submenu);
                    }
                } else {
                    // Обычная ссылка
                    window.location.href = item.href;
                }
            }

            handleItemKeydown(e, item) {
                switch(e.key) {
                    case 'Enter':
                    case ' ':
                        e.preventDefault();
                        this.handleItemClick(e, item);
                        break;
                    case 'ArrowDown':
                        e.preventDefault();
                        this.focusNextItem(item);
                        break;
                    case 'ArrowUp':
                        e.preventDefault();
                        this.focusPreviousItem(item);
                        break;
                    case 'ArrowRight':
                        if (item.nextElementSibling && item.nextElementSibling.classList.contains('submenu')) {
                            e.preventDefault();
                            this.openMenu(item, item.nextElementSibling);
                        } else {
                            e.preventDefault();
                            this.focusNextMenuItem(item);
                        }
                        break;
                    case 'ArrowLeft':
                        e.preventDefault();
                        this.closeCurrentMenu();
                        break;
                    case 'Escape':
                        e.preventDefault();
                        this.closeAllMenus();
                        item.focus();
                        break;
                    case 'Home':
                        e.preventDefault();
                        this.focusFirstMenuItem();
                        break;
                    case 'End':
                        e.preventDefault();
                        this.focusLastMenuItem();
                        break;
                }
            }

            handleItemHover(e, item) {
                const submenu = item.nextElementSibling;
                
                if (submenu && submenu.classList.contains('submenu')) {
                    this.openMenu(item, submenu);
                }
            }

            openMenu(item, submenu) {
                this.closeAllMenus();
                
                submenu.hidden = false;
                item.setAttribute('aria-expanded', 'true');
                
                this.currentMenu = submenu;
                this.currentItem = item;
                
                // Фокус на первый элемент подменю
                const firstItem = submenu.querySelector('[role="menuitem"]');
                if (firstItem) {
                    firstItem.focus();
                }
            }

            closeMenu(submenu) {
                submenu.hidden = true;
                const menuItem = submenu.previousElementSibling;
                if (menuItem) {
                    menuItem.setAttribute('aria-expanded', 'false');
                }
                
                if (this.currentMenu === submenu) {
                    this.currentMenu = null;
                }
            }

            closeAllMenus() {
                this.submenus.forEach(submenu => {
                    submenu.hidden = true;
                    const menuItem = submenu.previousElementSibling;
                    if (menuItem) {
                        menuItem.setAttribute('aria-expanded', 'false');
                    }
                });
                
                this.currentMenu = null;
            }

            closeCurrentMenu() {
                if (this.currentMenu) {
                    this.closeMenu(this.currentMenu);
                    if (this.currentItem) {
                        this.currentItem.focus();
                    }
                }
            }

            focusNextItem(item) {
                const submenu = this.findParentSubmenu(item);
                if (submenu) {
                    const items = Array.from(submenu.querySelectorAll('[role="menuitem"]'));
                    const currentIndex = items.indexOf(item);
                    const nextIndex = (currentIndex + 1) % items.length;
                    
                    if (items[nextIndex]) {
                        items[nextIndex].focus();
                    }
                }
            }

            focusPreviousItem(item) {
                const submenu = this.findParentSubmenu(item);
                if (submenu) {
                    const items = Array.from(submenu.querySelectorAll('[role="menuitem"]'));
                    const currentIndex = items.indexOf(item);
                    const prevIndex = (currentIndex - 1 + items.length) % items.length;
                    
                    if (items[prevIndex]) {
                        items[prevIndex].focus();
                    }
                }
            }

            focusNextMenuItem(item) {
                const menuItems = Array.from(this.menuItems);
                const currentIndex = menuItems.indexOf(item);
                const nextIndex = (currentIndex + 1) % menuItems.length;
                
                if (menuItems[nextIndex]) {
                    menuItems[nextIndex].focus();
                }
            }

            focusFirstMenuItem() {
                if (this.menuItems[0]) {
                    this.menuItems[0].focus();
                }
            }

            focusLastMenuItem() {
                const lastItem = this.menuItems[this.menuItems.length - 1];
                if (lastItem) {
                    lastItem.focus();
                }
            }

            findParentSubmenu(item) {
                let parent = item.parentElement;
                while (parent && !parent.classList.contains('submenu')) {
                    parent = parent.parentElement;
                }
                return parent;
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleMenu();
        });
    </script>

    <style>
        .header-container {
            background-color: #f3f2f1;
            padding: 0 20px;
        }

        .main-nav {
            list-style: none;
            padding: 0;
            margin: 0;
            display: flex;
        }

        .main-nav > li {
            position: relative;
        }

        .main-nav [role="menuitem"] {
            display: block;
            padding: 12px 16px;
            text-decoration: none;
            color: #323130;
            cursor: pointer;
        }

        .main-nav [role="menuitem"]:hover,
        .main-nav [role="menuitem"]:focus {
            background-color: #edebe9;
            outline: none;
        }

        .main-nav [role="menuitem"][aria-expanded="true"] {
            background-color: #edebe9;
        }

        .submenu {
            position: absolute;
            top: 100%;
            left: 0;
            background: white;
            border: 1px solid #c8c6c4;
            min-width: 200px;
            list-style: none;
            padding: 0;
            margin: 0;
            box-shadow: 0 3px 6px rgba(0,0,0,0.2);
            z-index: 1000;
        }

        .submenu [role="menuitem"] {
            display: block;
            padding: 8px 12px;
            text-decoration: none;
            color: #323130;
            cursor: pointer;
            white-space: nowrap;
        }

        .submenu [role="menuitem"]:hover,
        .submenu [role="menuitem"]:focus {
            background-color: #f3f2f1;
            outline: none;
        }

        .submenu [role="separator"] {
            height: 1px;
            background-color: #c8c6c4;
            margin: 4px 0;
        }
    </style>
</body>
</html>
```

### 3. BBC - доступный медиа-контент

BBC демонстрирует передовой опыт в области доступности медиа-контента:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступный медиа-контент BBC</title>
</head>
<body>
    <main>
        <h1>Доступный медиа-контент</h1>

        <article>
            <h2>Новости дня</h2>
            
            <figure role="group" aria-labelledby="video-title">
                <video controls
                       width="640"
                       height="360"
                       preload="metadata"
                       aria-labelledby="video-title"
                       aria-describedby="video-description">
                    <source src="news-video.mp4" type="video/mp4">
                    <source src="news-video.webm" type="video/webm">

                    <!-- Множественные дорожки -->
                    <track kind="subtitles"
                           src="news-ru.vtt"
                           srclang="ru"
                           label="Русские"
                           default>
                    <track kind="subtitles"
                           src="news-en.vtt"
                           srclang="en"
                           label="English">
                    <track kind="descriptions"
                           src="news-descriptions.vtt"
                           srclang="ru"
                           label="Описания">
                    <track kind="captions"
                           src="news-captions.vtt"
                           srclang="ru"
                           label="Титры">

                    Ваш браузер не поддерживает видео элемент.
                </video>

                <figcaption id="video-description">
                    <h3 id="video-title">Главные новости дня</h3>
                    <p>Ежедневный обзор важнейших событий. Продолжительность: 5 минут.</p>
                </figcaption>
            </figure>

            <!-- Аудио-версия -->
            <figure role="group" aria-labelledby="audio-title">
                <audio controls aria-labelledby="audio-title">
                    <source src="news-audio.mp3" type="audio/mpeg">
                    <source src="news-audio.ogg" type="audio/ogg">
                    Ваш браузер не поддерживает аудио элемент.
                </audio>

                <figcaption>
                    <h3 id="audio-title">Аудио-версия новостей</h3>
                    <p>Для пользователей с нарушениями зрения</p>
                </figcaption>
            </figure>

            <!-- Транскрипция -->
            <section aria-labelledby="transcription-title">
                <h3 id="transcription-title">Транскрипция новостей</h3>
                <div class="transcription-content">
                    <h4>Политика</h4>
                    <p>Главные политические события дня...</p>
                    
                    <h4>Экономика</h4>
                    <p>Аналитика экономической ситуации...</p>
                    
                    <h4>Культура</h4>
                    <p>Новости культуры и искусства...</p>
                </div>
            </section>
        </article>
    </main>

    <script>
        class AccessibleMediaPlayer {
            constructor(videoSelector) {
                this.video = document.querySelector(videoSelector);
                this.playerContainer = this.video.parentElement;
                
                this.setupPlayer();
                this.setupAccessibilityFeatures();
            }

            setupPlayer() {
                // Добавляем элементы управления доступностью
                this.addAccessibilityControls();
            }

            addAccessibilityControls() {
                // Контроль скорости воспроизведения
                const speedControl = document.createElement('select');
                speedControl.id = 'playback-speed';
                speedControl.setAttribute('aria-label', 'Скорость воспроизведения');
                speedControl.innerHTML = `
                    <option value="0.5">0.5x</option>
                    <option value="0.75">0.75x</option>
                    <option value="1" selected>1x</option>
                    <option value="1.25">1.25x</option>
                    <option value="1.5">1.5x</option>
                    <option value="2">2x</option>
                `;
                
                speedControl.addEventListener('change', (e) => {
                    this.video.playbackRate = parseFloat(e.target.value);
                });
                
                // Добавляем элемент управления в контейнер
                this.playerContainer.insertBefore(speedControl, this.video.nextSibling);
            }

            setupAccessibilityFeatures() {
                // Объявление состояния воспроизведения
                this.video.addEventListener('play', () => {
                    this.announceToScreenReader('Видео началось');
                });

                this.video.addEventListener('pause', () => {
                    this.announceToScreenReader('Видео приостановлено');
                });

                this.video.addEventListener('ended', () => {
                    this.announceToScreenReader('Видео завершено');
                });
            }

            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 2000);
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleMediaPlayer('video');
        });
    </script>

    <style>
        figure {
            margin: 20px 0;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        video, audio {
            width: 100%;
            max-width: 640px;
            display: block;
        }

        figcaption h3 {
            margin-top: 0;
        }

        #playback-speed {
            margin-top: 10px;
            padding: 5px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .transcription-content {
            background-color: #f9f9f9;
            padding: 20px;
            border-radius: 8px;
            margin-top: 20px;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

## Современные паттерны доступности

### 1. Паттерн "Ловушка фокуса" (Focus Trap)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Паттерн ловушки фокуса</title>
</head>
<body>
    <main>
        <h1>Паттерн ловушки фокуса</h1>
        
        <button id="open-modal" onclick="openModal()">Открыть модальное окно</button>
        
        <div id="main-content">
            <h2>Основное содержимое</h2>
            <p>Это основное содержимое страницы. Когда открыто модальное окно, фокус должен быть ограничен только этим окном.</p>
        </div>
    </main>

    <div id="modal" class="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title" hidden>
        <div class="modal-content">
            <div class="modal-header">
                <h2 id="modal-title">Модальное окно</h2>
                <button id="close-modal" aria-label="Закрыть">&times;</button>
            </div>
            <div class="modal-body">
                <p>Это модальное окно с ловушкой фокуса. Фокус будет ограничен только этим окном.</p>
                <button id="btn1">Кнопка 1</button>
                <button id="btn2">Кнопка 2</button>
                <input type="text" id="input1" placeholder="Поле ввода">
                <a href="#" id="link1">Ссылка</a>
            </div>
            <div class="modal-footer">
                <button id="cancel-btn">Отмена</button>
                <button id="ok-btn">ОК</button>
            </div>
        </div>
    </div>

    <script>
        class FocusTrap {
            constructor(element) {
                this.element = element;
                this.focusableElements = null;
                this.firstFocusableElement = null;
                this.lastFocusableElement = null;
            }

            activate() {
                this.focusableElements = this.element.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );

                if (this.focusableElements.length > 0) {
                    this.firstFocusableElement = this.focusableElements[0];
                    this.lastFocusableElement = this.focusableElements[this.focusableElements.length - 1];

                    // Устанавливаем фокус на первый элемент
                    this.firstFocusableElement.focus();

                    // Добавляем обработчик события
                    this.element.addEventListener('keydown', this.trapFocus.bind(this));
                }
            }

            deactivate() {
                this.element.removeEventListener('keydown', this.trapFocus.bind(this));
            }

            trapFocus(e) {
                if (e.key === 'Tab') {
                    if (e.shiftKey) {
                        // Shift + Tab
                        if (document.activeElement === this.firstFocusableElement) {
                            this.lastFocusableElement.focus();
                            e.preventDefault();
                        }
                    } else {
                        // Tab
                        if (document.activeElement === this.lastFocusableElement) {
                            this.firstFocusableElement.focus();
                            e.preventDefault();
                        }
                    }
                }
            }
        }

        let focusTrap = null;
        let previousActiveElement = null;

        function openModal() {
            const modal = document.getElementById('modal');
            
            // Сохраняем активный элемент
            previousActiveElement = document.activeElement;
            
            // Показываем модальное окно
            modal.hidden = false;
            
            // Создаем и активируем ловушку фокуса
            focusTrap = new FocusTrap(modal);
            focusTrap.activate();
            
            // Скрываем основное содержимое для скринридеров
            document.getElementById('main-content').setAttribute('aria-hidden', 'true');
        }

        function closeModal() {
            const modal = document.getElementById('modal');
            
            // Деактивируем ловушку фокуса
            if (focusTrap) {
                focusTrap.deactivate();
                focusTrap = null;
            }
            
            // Скрываем модальное окно
            modal.hidden = true;
            
            // Восстанавливаем фокус на предыдущий элемент
            if (previousActiveElement && previousActiveElement.focus) {
                previousActiveElement.focus();
            }
            
            // Восстанавливаем основное содержимое для скринридеров
            document.getElementById('main-content').removeAttribute('aria-hidden');
        }

        // Настройка обработчиков событий
        document.getElementById('close-modal').addEventListener('click', closeModal);
        document.getElementById('cancel-btn').addEventListener('click', closeModal);
        document.getElementById('ok-btn').addEventListener('click', closeModal);

        // Закрытие по Escape
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape' && !document.getElementById('modal').hidden) {
                closeModal();
            }
        });

        // Закрытие по клику вне модального окна
        document.getElementById('modal').addEventListener('click', (e) => {
            if (e.target === e.currentTarget) {
                closeModal();
            }
        });
    </script>

    <style>
        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .modal-content {
            background-color: white;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
            overflow: hidden;
        }

        .modal-header {
            padding: 20px;
            border-bottom: 1px solid #eee;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .modal-header h2 {
            margin: 0;
        }

        .modal-header button {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            padding: 0;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .modal-body {
            padding: 20px;
        }

        .modal-body button,
        .modal-body input,
        .modal-body a {
            margin: 5px;
            display: block;
            margin-bottom: 10px;
        }

        .modal-footer {
            padding: 20px;
            border-top: 1px solid #eee;
            display: flex;
            justify-content: flex-end;
            gap: 10px;
        }

        .modal-footer button {
            padding: 8px 16px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }

        .modal-footer button:last-child {
            background-color: #007acc;
            color: white;
            border-color: #007acc;
        }
    </style>
</body>
</html>
```

### 2. Паттерн "Живые области" (Live Regions)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Паттерн живых областей</title>
</head>
<body>
    <main>
        <h1>Паттерн живых областей</h1>
        
        <div class="chat-interface">
            <div id="chat-messages" 
                 role="log" 
                 aria-live="polite" 
                 aria-atomic="false" 
                 aria-label="Сообщения чата">
                <!-- Сообщения будут добавляться сюда -->
            </div>
            
            <div id="status-updates" 
                 role="status" 
                 aria-live="polite" 
                 aria-atomic="true" 
                 class="sr-only">
                <!-- Статусные обновления для скринридеров -->
            </div>
            
            <div id="alerts" 
                 role="alert" 
                 aria-live="assertive" 
                 aria-atomic="true">
                <!-- Критические уведомления -->
            </div>
            
            <form id="message-form">
                <input type="text" 
                       id="message-input" 
                       placeholder="Введите сообщение..." 
                       aria-label="Поле ввода сообщения">
                <button type="submit">Отправить</button>
            </form>
        </div>
    </main>

    <script>
        class LiveRegionManager {
            constructor() {
                this.messagesContainer = document.getElementById('chat-messages');
                this.statusContainer = document.getElementById('status-updates');
                this.alertsContainer = document.getElementById('alerts');
                this.messageForm = document.getElementById('message-form');
                this.messageInput = document.getElementById('message-input');
                
                this.setupEventListeners();
            }

            setupEventListeners() {
                this.messageForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.sendMessage();
                });
            }

            sendMessage() {
                const message = this.messageInput.value.trim();
                
                if (message) {
                    // Добавляем сообщение в лог
                    this.addMessage(message);
                    
                    // Очищаем поле ввода
                    this.messageInput.value = '';
                    
                    // Объявляем отправку для скринридеров
                    this.announceStatus(`Сообщение отправлено: ${message}`);
                }
            }

            addMessage(text) {
                const messageElement = document.createElement('div');
                messageElement.className = 'message';
                messageElement.setAttribute('role', 'logitem');
                messageElement.setAttribute('aria-label', `Сообщение: ${text}`);
                
                const timestamp = new Date().toLocaleTimeString();
                messageElement.innerHTML = `
                    <strong>Вы:</strong>
                    <span class="timestamp">(${timestamp})</span>
                    <p>${this.escapeHtml(text)}</p>
                `;
                
                this.messagesContainer.appendChild(messageElement);
                
                // Прокручиваем к новому сообщению
                messageElement.scrollIntoView({ behavior: 'smooth' });
            }

            announceStatus(message) {
                this.statusContainer.textContent = message;
                
                // Очищаем статус через 5 секунд
                setTimeout(() => {
                    if (this.statusContainer.textContent === message) {
                        this.statusContainer.textContent = '';
                    }
                }, 5000);
            }

            showAlert(message) {
                this.alertsContainer.textContent = message;
                
                // Очищаем алерт через 10 секунд
                setTimeout(() => {
                    if (this.alertsContainer.textContent === message) {
                        this.alertsContainer.textContent = '';
                    }
                }, 10000);
            }

            escapeHtml(text) {
                const div = document.createElement('div');
                div.textContent = text;
                return div.innerHTML;
            }

            // Имитация получения сообщений от других пользователей
            simulateIncomingMessage() {
                const messages = [
                    'Привет! Как дела?',
                    'Все отлично, спасибо!',
                    'Рад слышать!',
                    'Когда встречаемся?',
                    'На этой неделе, как насчет четверга?'
                ];
                
                const randomMessage = messages[Math.floor(Math.random() * messages.length)];
                this.addMessageFromOtherUser(randomMessage);
            }

            addMessageFromOtherUser(text) {
                const messageElement = document.createElement('div');
                messageElement.className = 'message other-user';
                messageElement.setAttribute('role', 'logitem');
                messageElement.setAttribute('aria-label', `Сообщение от другого пользователя: ${text}`);
                
                const timestamp = new Date().toLocaleTimeString();
                messageElement.innerHTML = `
                    <strong>Другой пользователь:</strong>
                    <span class="timestamp">(${timestamp})</span>
                    <p>${this.escapeHtml(text)}</p>
                `;
                
                this.messagesContainer.appendChild(messageElement);
                
                // Прокручиваем к новому сообщению
                messageElement.scrollIntoView({ behavior: 'smooth' });
                
                // Объявляем для скринридеров
                this.announceStatus(`Новое сообщение от другого пользователя: ${text}`);
            }
        }

        // Инициализация
        const liveRegionManager = new LiveRegionManager();
        
        // Имитация входящих сообщений
        setInterval(() => {
            if (Math.random() > 0.7) { // 30% шанс имитации сообщения
                liveRegionManager.simulateIncomingMessage();
            }
        }, 10000);
    </script>

    <style>
        .chat-interface {
            max-width: 600px;
            margin: 0 auto;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
        }

        #chat-messages {
            height: 400px;
            overflow-y: auto;
            margin-bottom: 20px;
            padding: 10px;
            border: 1px solid #eee;
            border-radius: 4px;
        }

        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            background-color: #f0f0f0;
        }

        .message.other-user {
            background-color: #e3f2fd;
            border-left: 4px solid #2196f3;
        }

        .timestamp {
            color: #666;
            font-size: 0.8em;
        }

        #message-form {
            display: flex;
            gap: 10px;
        }

        #message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        #message-input:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }

        button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        #alerts {
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
            background-color: #ffebee;
            color: #c62828;
            border-left: 4px solid #f44336;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

## Лучшие практики промышленной разработки

### 1. Инклюзивный дизайн с самого начала

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Инклюзивный дизайн</title>
    <style>
        /* Универсальные стили доступности */
        :root {
            --focus-color: #005a9e;
            --text-color: #333;
            --bg-color: #fff;
            --link-color: #0066cc;
            --border-color: #ccc;
            --success-color: #28a745;
            --warning-color: #ffc107;
            --error-color: #dc3545;
        }

        /* Уважение к пользовательским предпочтениям */
        @media (prefers-reduced-motion: reduce) {
            * {
                animation-duration: 0.01ms !important;
                animation-iteration-count: 1 !important;
                transition-duration: 0.01ms !important;
            }
        }

        @media (prefers-contrast: high) {
            :root {
                --border-color: #000;
            }
            
            body {
                background-color: #fff;
                color: #000;
            }
            
            .card {
                border: 2px solid #000;
            }
        }

        @media (prefers-color-scheme: dark) {
            :root {
                --text-color: #e0e0e0;
                --bg-color: #121212;
                --link-color: #64b5f6;
            }
        }

        /* Базовые стили */
        body {
            font-family: system-ui, -apple-system, sans-serif;
            font-size: 18px; /* Минимум 16px */
            line-height: 1.5;
            color: var(--text-color);
            background-color: var(--bg-color);
            margin: 0;
            padding: 20px;
        }

        /* Видимые индикаторы фокуса */
        *:focus {
            outline: 2px solid var(--focus-color);
            outline-offset: 2px;
        }

        /* Контрастность текста */
        .high-contrast-text {
            color: #000;
            background-color: #fff;
            /* Соотношение контраста: 21:1 */
        }

        .normal-contrast-text {
            color: #333;
            background-color: #fff;
            /* Соотношение контраста: 12.6:1 */
        }

        /* Карточки с доступностью */
        .card {
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
            background-color: var(--bg-color);
        }

        /* Связывание элементов */
        .form-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            font-size: 16px;
        }

        input:focus, select:focus, textarea:focus {
            outline: 2px solid var(--focus-color);
            outline-offset: 2px;
        }

        /* Кнопки с доступностью */
        .btn {
            display: inline-block;
            padding: 12px 24px;
            background-color: var(--link-color);
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        .btn:focus {
            outline: 2px solid var(--focus-color);
            outline-offset: 2px;
        }

        /* Скрытие элементов для скринридеров */
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }

        /* Визуальные скрытые, но доступные элементы управления */
        .visually-hidden:not(:focus):not(:active) {
            position: absolute;
            width: 1px;
            height: 1px;
            margin: -1px;
            border: 0;
            padding: 0;
            white-space: nowrap;
            clip-path: inset(100%);
            clip: rect(0 0 0 0);
            overflow: hidden;
        }
    </style>
</head>
<body>
    <header role="banner">
        <h1>Инклюзивный веб-сайт</h1>
        
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main role="main">
        <h2>Инклюзивный подход в разработке</h2>
        
        <section aria-labelledby="inclusive-design-title">
            <h3 id="inclusive-design-title">Принципы инклюзивного дизайна</h3>
            
            <div class="card">
                <h4>Равный доступ</h4>
                <p>Все пользователи должны иметь равный доступ к информации и функциям.</p>
            </div>
            
            <div class="card">
                <h4>Универсальный дизайн</h4>
                <p>Создание решений, подходящих для всех пользователей.</p>
            </div>
            
            <div class="card">
                <h4>Прогрессивное улучшение</h4>
                <p>Обеспечение базовой функциональности для всех пользователей.</p>
            </div>
        </section>

        <section aria-labelledby="preferences-title">
            <h3 id="preferences-title">Пользовательские предпочтения</h3>
            
            <form id="preferences-form">
                <div class="form-group">
                    <label for="font-size">Размер шрифта:</label>
                    <select id="font-size" name="font-size">
                        <option value="small">Маленький</option>
                        <option value="medium" selected>Средний</option>
                        <option value="large">Большой</option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" name="high-contrast" value="on">
                        Высокая контрастность
                    </label>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" name="reduce-motion" value="on">
                        Уменьшить анимацию
                    </label>
                </div>
                
                <button type="submit" class="btn">Сохранить настройки</button>
            </form>
        </section>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Инклюзивный веб-сайт. Все права защищены.</p>
    </footer>

    <script>
        class InclusiveDesignManager {
            constructor() {
                this.preferencesForm = document.getElementById('preferences-form');
                
                this.loadUserPreferences();
                this.setupEventListeners();
            }

            setupEventListeners() {
                this.preferencesForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.saveUserPreferences();
                });
            }

            loadUserPreferences() {
                const preferences = JSON.parse(localStorage.getItem('accessibilityPreferences')) || {};
                
                // Установка сохраненных предпочтений
                if (preferences.fontSize) {
                    document.getElementById('font-size').value = preferences.fontSize;
                    this.applyFontSize(preferences.fontSize);
                }
                
                if (preferences.highContrast) {
                    this.applyHighContrast(preferences.highContrast);
                }
                
                if (preferences.reduceMotion) {
                    this.applyReduceMotion(preferences.reduceMotion);
                }
            }

            saveUserPreferences() {
                const formData = new FormData(this.preferencesForm);
                const preferences = {
                    fontSize: formData.get('font-size'),
                    highContrast: formData.get('high-contrast') === 'on',
                    reduceMotion: formData.get('reduce-motion') === 'on'
                };
                
                localStorage.setItem('accessibilityPreferences', JSON.stringify(preferences));
                
                // Применение предпочтений
                this.applyFontSize(preferences.fontSize);
                this.applyHighContrast(preferences.highContrast);
                this.applyReduceMotion(preferences.reduceMotion);
                
                // Уведомление для скринридеров
                this.announceToScreenReader('Настройки доступности сохранены');
            }

            applyFontSize(size) {
                const fontSizeMap = {
                    'small': '16px',
                    'medium': '18px',
                    'large': '22px'
                };
                
                document.body.style.fontSize = fontSizeMap[size] || '18px';
            }

            applyHighContrast(enabled) {
                if (enabled) {
                    document.body.classList.add('high-contrast');
                } else {
                    document.body.classList.remove('high-contrast');
                }
            }

            applyReduceMotion(enabled) {
                if (enabled) {
                    document.body.classList.add('reduce-motion');
                    // Переопределение всех анимаций
                    const style = document.createElement('style');
                    style.textContent = `
                        *, *::before, *::after {
                            animation-duration: 0.01ms !important;
                            animation-iteration-count: 1 !important;
                            transition-duration: 0.01ms !important;
                        }
                    `;
                    document.head.appendChild(style);
                } else {
                    document.body.classList.remove('reduce-motion');
                }
            }

            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 2000);
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new InclusiveDesignManager();
        });
    </script>
</body>
</html>
```

### 2. Паттерны для сложных интерактивных компонентов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Сложные интерактивные компоненты</title>
</head>
<body>
    <main>
        <h1>Сложные интерактивные компоненты</h1>
        
        <!-- Дерево навигации -->
        <section aria-labelledby="tree-nav-title">
            <h2 id="tree-nav-title">Дерево навигации</h2>
            <div id="tree-container" role="tree" aria-label="Структура сайта">
                <div class="tree-item" role="treeitem" aria-expanded="true" aria-level="1">
                    <div class="tree-item-label" role="none" tabindex="0" onclick="toggleTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                        <span class="tree-toggle">▼</span>
                        <span>Документация</span>
                    </div>
                    <div class="tree-children" role="group">
                        <div class="tree-item" role="treeitem" aria-expanded="false" aria-level="2">
                            <div class="tree-item-label" role="none" tabindex="0" onclick="toggleTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                <span class="tree-toggle">▶</span>
                                <span>HTML</span>
                            </div>
                            <div class="tree-children" role="group" hidden>
                                <div class="tree-item" role="treeitem" aria-level="3">
                                    <div class="tree-item-label" role="none" tabindex="0" onclick="selectTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                        <span class="tree-toggle"></span>
                                        <span>Основы HTML</span>
                                    </div>
                                </div>
                                <div class="tree-item" role="treeitem" aria-level="3">
                                    <div class="tree-item-label" role="none" tabindex="0" onclick="selectTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                        <span class="tree-toggle"></span>
                                        <span>Формы</span>
                                    </div>
                                </div>
                                <div class="tree-item" role="treeitem" aria-level="3">
                                    <div class="tree-item-label" role="none" tabindex="0" onclick="selectTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                        <span class="tree-toggle"></span>
                                        <span>Доступность</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="tree-item" role="treeitem" aria-expanded="false" aria-level="2">
                            <div class="tree-item-label" role="none" tabindex="0" onclick="toggleTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                <span class="tree-toggle">▶</span>
                                <span>CSS</span>
                            </div>
                            <div class="tree-children" role="group" hidden>
                                <div class="tree-item" role="treeitem" aria-level="3">
                                    <div class="tree-item-label" role="none" tabindex="0" onclick="selectTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                        <span class="tree-toggle"></span>
                                        <span>Селекторы</span>
                                    </div>
                                </div>
                                <div class="tree-item" role="treeitem" aria-level="3">
                                    <div class="tree-item-label" role="none" tabindex="0" onclick="selectTreeItem(this)" onkeydown="handleTreeItemKeydown(event, this)">
                                        <span class="tree-toggle"></span>
                                        <span>Адаптивный дизайн</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
        
        <!-- Прогресс бар с доступностью -->
        <section aria-labelledby="progress-title">
            <h2 id="progress-title">Прогресс загрузки</h2>
            <div class="progress-container">
                <div id="progress-bar"
                     role="progressbar"
                     aria-valuenow="0"
                     aria-valuemin="0"
                     aria-valuemax="100"
                     aria-valuetext="Загрузка: 0%">
                    <div class="progress-fill" style="width: 0%"></div>
                </div>
                <button onclick="startProgress()">Начать загрузку</button>
            </div>
        </section>
    </main>

    <script>
        class AccessibleTree {
            constructor() {
                this.setupTreeNavigation();
            }

            setupTreeNavigation() {
                // Добавляем обработчики для клавиатурной навигации
                const treeItems = document.querySelectorAll('[role="treeitem"]');
                
                treeItems.forEach((item, index) => {
                    item.addEventListener('keydown', (e) => {
                        this.handleTreeNavigation(e, item, index, treeItems);
                    });
                });
            }

            handleTreeNavigation(e, currentItem, currentIndex, allItems) {
                switch(e.key) {
                    case 'ArrowRight':
                        e.preventDefault();
                        this.expandItem(currentItem);
                        break;
                    case 'ArrowLeft':
                        e.preventDefault();
                        this.collapseItem(currentItem);
                        break;
                    case 'ArrowDown':
                        e.preventDefault();
                        this.focusNextItem(currentIndex, allItems);
                        break;
                    case 'ArrowUp':
                        e.preventDefault();
                        this.focusPreviousItem(currentIndex, allItems);
                        break;
                    case 'Home':
                        e.preventDefault();
                        allItems[0].focus();
                        break;
                    case 'End':
                        e.preventDefault();
                        allItems[allItems.length - 1].focus();
                        break;
                    case 'Enter':
                    case ' ':
                        e.preventDefault();
                        this.toggleItem(currentItem);
                        break;
                }
            }

            expandItem(item) {
                const isExpanded = item.getAttribute('aria-expanded') === 'true';
                
                if (!isExpanded) {
                    item.setAttribute('aria-expanded', 'true');
                    const children = item.querySelector('.tree-children');
                    if (children) {
                        children.hidden = false;
                    }
                    item.querySelector('.tree-toggle').textContent = '▼';
                }
            }

            collapseItem(item) {
                const isExpanded = item.getAttribute('aria-expanded') === 'true';
                
                if (isExpanded) {
                    item.setAttribute('aria-expanded', 'false');
                    const children = item.querySelector('.tree-children');
                    if (children) {
                        children.hidden = true;
                    }
                    item.querySelector('.tree-toggle').textContent = '▶';
                }
            }

            toggleItem(item) {
                const isExpanded = item.getAttribute('aria-expanded') === 'true';
                
                if (isExpanded) {
                    this.collapseItem(item);
                } else {
                    this.expandItem(item);
                }
            }

            focusNextItem(currentIndex, allItems) {
                const nextIndex = (currentIndex + 1) % allItems.length;
                allItems[nextIndex].focus();
            }

            focusPreviousItem(currentIndex, allItems) {
                const prevIndex = (currentIndex - 1 + allItems.length) % allItems.length;
                allItems[prevIndex].focus();
            }
        }

        // Инициализация дерева
        const accessibleTree = new AccessibleTree();

        // Функции для дерева
        function toggleTreeItem(element) {
            const treeItem = element.parentElement;
            const isExpanded = treeItem.getAttribute('aria-expanded') === 'true';
            
            treeItem.setAttribute('aria-expanded', !isExpanded);
            
            const children = element.parentElement.querySelector('.tree-children');
            if (children) {
                children.hidden = isExpanded;
            }
            
            element.querySelector('.tree-toggle').textContent = isExpanded ? '▶' : '▼';
        }

        function selectTreeItem(element) {
            // Снятие выделения с других элементов
            document.querySelectorAll('.tree-item-label').forEach(item => {
                item.classList.remove('selected');
            });
            
            // Выделение текущего элемента
            element.classList.add('selected');
            
            // Объявление выбора для скринридеров
            announceToScreenReader(`Выбран элемент: ${element.textContent.trim()}`);
        }

        function handleTreeItemKeydown(e, element) {
            switch(e.key) {
                case 'Enter':
                case ' ':
                    e.preventDefault();
                    if (element.parentElement.getAttribute('aria-expanded') !== null) {
                        toggleTreeItem(element);
                    } else {
                        selectTreeItem(element);
                    }
                    break;
            }
        }

        // Прогресс бар
        function startProgress() {
            const progressBar = document.getElementById('progress-bar');
            const progressFill = progressBar.querySelector('.progress-fill');
            let progress = 0;
            
            const interval = setInterval(() => {
                progress += 5;
                progressFill.style.width = `${progress}%`;
                
                progressBar.setAttribute('aria-valuenow', progress);
                progressBar.setAttribute('aria-valuetext', `Загрузка: ${progress}%`);
                
                if (progress >= 100) {
                    clearInterval(interval);
                    announceToScreenReader('Загрузка завершена');
                }
            }, 200);
        }

        function announceToScreenReader(message) {
            const announcement = document.createElement('div');
            announcement.setAttribute('aria-live', 'polite');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.style.position = 'absolute';
            announcement.style.left = '-9999px';
            announcement.textContent = message;

            document.body.appendChild(announcement);

            setTimeout(() => {
                if (document.body.contains(announcement)) {
                    document.body.removeChild(announcement);
                }
            }, 3000);
        }
    </script>

    <style>
        #tree-container {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            max-width: 300px;
        }

        .tree-item {
            margin: 5px 0;
        }

        .tree-item-label {
            display: flex;
            align-items: center;
            padding: 8px;
            cursor: pointer;
            border-radius: 4px;
        }

        .tree-item-label:hover,
        .tree-item-label:focus,
        .tree-item-label.selected {
            background-color: #e3f2fd;
            outline: 2px solid #007acc;
        }

        .tree-toggle {
            margin-right: 8px;
            width: 16px;
            display: inline-block;
            text-align: center;
        }

        .tree-children {
            margin-left: 20px;
        }

        .progress-container {
            max-width: 400px;
            margin: 20px 0;
        }

        #progress-bar {
            width: 100%;
            height: 24px;
            border: 1px solid #ccc;
            border-radius: 12px;
            overflow: hidden;
            margin-bottom: 10px;
        }

        .progress-fill {
            height: 100%;
            background-color: #4caf50;
            transition: width 0.3s ease;
        }

        button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

## Сравнение различных подходов

### Традиционный подход vs Современный доступный подход

**Традиционный подход (не доступный):**
```html
<!-- Плохо: недоступная кнопка-имитация -->
<div onclick="doSomething()" style="cursor: pointer; background: #007acc; color: white; padding: 10px;">
    Нажми меня
</div>

<!-- Плохо: отсутствие альтернативного текста -->
<img src="chart.png">

<!-- Плохо: неправильная структура заголовков -->
<div>Заголовок страницы</div>
<div>Подзаголовок</div>
<div>Еще один подзаголовок</div>
```

**Современный доступный подход:**
```html
<!-- Хорошо: семантически правильная кнопка -->
<button onclick="doSomething()">Нажми меня</button>

<!-- Или если нужна кнопка-имитация, то с правильной разметкой -->
<div role="button" 
     tabindex="0" 
     onclick="doSomething()" 
     onkeydown="handleKeyDown(event)"
     style="cursor: pointer; background: #007acc; color: white; padding: 10px;">
    Нажми меня
</div>

<!-- Хорошо: альтернативный текст -->
<img src="chart.png" alt="График роста продаж за последние 5 лет, показывающий увеличение на 25% ежегодно">

<!-- Хорошо: правильная структура заголовков -->
<h1>Заголовок страницы</h1>
<h2>Подзаголовок</h2>
<h3>Еще один подзаголовок</h3>
```

## Рекомендации по применению в реальных проектах

### 1. Интеграция в процесс разработки

- Внедрение проверок доступности в CI/CD пайплайны
- Обучение команды принципам доступности
- Использование linter'ов и проверок в IDE
- Регулярные аудиты доступности

### 2. Тестирование с реальными пользователями

- Вовлечение пользователей с ограниченными возможностями в тестирование
- Использование скринридеров (NVDA, JAWS, VoiceOver) для тестирования
- Тестирование клавиатурной навигации
- Проверка контрастности цветов

### 3. Мониторинг и поддержка

- Регулярные проверки автоматизированными инструментами
- Сбор обратной связи от пользователей
- Обновление компонентов и паттернов
- Поддержание соответствия стандартам WCAG

## Заключение

Промышленные примеры доступности показывают, как крупные компании внедряют принципы инклюзивного дизания в свои продукты. Эти примеры служат образцами для подражания и демонстрируют, что доступность не только возможна, но и улучшает общий пользовательский опыт.

Ключевые моменты:
1. Доступность должна быть приоритетом с самого начала проекта
2. Использование семантически правильной разметки
3. Правильное применение ARIA-атрибутов
4. Обеспечение клавиатурной навигации
5. Учет пользовательских предпочтений
6. Регулярное тестирование и улучшение
7. Вовлечение пользователей с ограниченными возможностями

Эти практики помогают создавать веб-сайты и приложения, которые действительно работают для всех пользователей, независимо от их возможностей или используемых технологий.

## Следующие темы
- [[HTML/Доступность/Стандарты WCAG]]
- [[HTML/Доступность/Тестирование]]
- [[HTML/Доступность/ARIA]]

## Теги
#html #accessibility #inclusive-design #universal-design #web-development #frontend #a11y #industrial-examples #best-practices #accessibility-patterns #user-experience #screen-readers #keyboard-navigation #aria #wcag #accessibility-audit #accessibility-testing #focus-trap #live-regions #tree-navigation #progressive-enhancement #graceful-degradation #user-preferences #reduced-motion #high-contrast #dark-mode #accessibility-api #web-components #javascript #dom-manipulation #accessibility-ux #cognitive-accessibility #motor-accessibility #visual-accessibility #auditory-accessibility #accessibility-compliance #accessibility-tools