# HTML Доступность: Лучшие практики

Лучшие практики доступности HTML - это проверенные методы и подходы, которые помогают создавать веб-контент, который может быть использован всеми пользователями, включая тех, кто имеет ограничения по здоровью. Эти практики обеспечивают соответствие стандартам WCAG и создают инклюзивный пользовательский опыт.

## Основы инклюзивной разработки

### Принципы инклюзивного дизайна

1. **Равный доступ** - все пользователи имеют равный доступ к информации и функциям
2. **Универсальный дизайн** - создание решений, подходящих для всех пользователей
3. **Прогрессивное улучшение** - обеспечение базовой функциональности для всех
4. **Грейсфул деградация** - корректная работа при отсутствии возможностей
5. **Уважение к пользовательским предпочтениям** - учет настроек пользователя

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Инклюзивный дизайн</title>
    <style>
        /* Уважение к предпочтениям пользователя */
        @media (prefers-reduced-motion: reduce) {
            * {
                animation-duration: 0.01ms !important;
                animation-iteration-count: 1 !important;
                transition-duration: 0.01ms !important;
            }
        }
        
        @media (prefers-contrast: high) {
            body {
                background-color: black;
                color: white;
            }
            
            .button {
                border: 2px solid white;
                background-color: black;
                color: white;
            }
        }
        
        @media (prefers-color-scheme: dark) {
            body {
                background-color: #121212;
                color: #e0e0e0;
            }
            
            .card {
                background-color: #1e1e1e;
                border: 1px solid #444;
            }
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
        <h2>Инклюзивный контент</h2>
        
        <section aria-labelledby="inclusive-design-title">
            <h3 id="inclusive-design-title">Принципы инклюзивного дизайна</h3>
            
            <div class="design-principles">
                <div class="principle-card">
                    <h4>Равный доступ</h4>
                    <p>Все пользователи должны иметь равный доступ к информации и функциям.</p>
                </div>
                
                <div class="principle-card">
                    <h4>Универсальный дизайн</h4>
                    <p>Создание решений, подходящих для всех пользователей.</p>
                </div>
                
                <div class="principle-card">
                    <h4>Прогрессивное улучшение</h4>
                    <p>Обеспечение базовой функциональности для всех пользователей.</p>
                </div>
            </div>
        </section>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Инклюзивный веб-сайт. Все права защищены.</p>
    </footer>
</body>
</html>
```

## Семантическая разметка

### Использование семантических элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Семантическая разметка</title>
</head>
<body>
    <header role="banner">
        <h1>Семантическая разметка</h1>
        <p>Создание структурированного и доступного HTML</p>
    </header>

    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="#introduction">Введение</a></li>
            <li><a href="#content">Содержимое</a></li>
            <li><a href="#conclusion">Заключение</a></li>
        </ul>
    </nav>

    <main role="main">
        <article>
            <header>
                <h2>Статья о семантической разметке</h2>
                <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
            </header>

            <section id="introduction" aria-labelledby="intro-title">
                <h3 id="intro-title">Введение в семантическую разметку</h3>
                <p>Семантическая разметка помогает браузерам и вспомогательным технологиям понимать структуру документа.</p>
            </section>

            <section id="content" aria-labelledby="content-title">
                <h3 id="content-title">Основное содержимое</h3>
                
                <section aria-labelledby="html-elements-title">
                    <h4 id="html-elements-title">HTML элементы</h4>
                    <p>Различные HTML элементы для семантической разметки:</p>
                    
                    <ul>
                        <li><code>&lt;header&gt;</code> - шапка документа или раздела</li>
                        <li><code>&lt;nav&gt;</code> - навигационные ссылки</li>
                        <li><code>&lt;main&gt;</code> - основное содержимое документа</li>
                        <li><code>&lt;article&gt;</code> - независимый контент</li>
                        <li><code>&lt;section&gt;</code> - тематически связанный контент</li>
                        <li><code>&lt;aside&gt;</code> - сопутствующий контент</li>
                        <li><code>&lt;footer&gt;</code> - подвал документа или раздела</li>
                    </ul>
                </section>
                
                <section aria-labelledby="benefits-title">
                    <h4 id="benefits-title">Преимущества</h4>
                    <p>Семантическая разметка предоставляет множество преимуществ:</p>
                    
                    <ol>
                        <li>Улучшенная доступность для пользователей с ограниченными возможностями</li>
                        <li>Лучшая SEO оптимизация</li>
                        <li>Улучшенная поддерживаемость кода</li>
                        <li>Лучшее понимание структуры документа</li>
                    </ol>
                </section>
            </section>

            <section id="conclusion" aria-labelledby="conclusion-title">
                <h3 id="conclusion-title">Заключение</h3>
                <p>Семантическая разметка - основа доступного и качественного веб-контента.</p>
            </section>

            <footer>
                <p>Автор: <span>Имя автора</span></p>
                <p>Категория: <a href="/category/html">HTML</a>, <a href="/category/accessibility">Доступность</a></p>
            </footer>
        </article>
    </main>

    <aside role="complementary" aria-label="Сопутствующая информация">
        <section>
            <h3>Похожие статьи</h3>
            <ul>
                <li><a href="/article1">Статья 1</a></li>
                <li><a href="/article2">Статья 2</a></li>
                <li><a href="/article3">Статья 3</a></li>
            </ul>
        </section>
    </aside>

    <footer role="contentinfo">
        <p>&copy; 2023 Все права защищены</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
                <li><a href="/sitemap">Карта сайта</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

### Правильная иерархия заголовков

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Иерархия заголовков</title>
</head>
<body>
    <main>
        <h1>Основной заголовок страницы</h1>
        
        <section>
            <h2>Первый раздел</h2>
            
            <article>
                <h3>Статья в первом разделе</h3>
                
                <section>
                    <h4>Подраздел статьи</h4>
                    <p>Содержимое подраздела...</p>
                    
                    <section>
                        <h5>Подподраздел</h5>
                        <p>Содержимое подподраздела...</p>
                    </section>
                </section>
            </article>
            
            <section>
                <h3>Другой подраздел</h3>
                <p>Содержимое другого подраздела...</p>
            </section>
        </section>
        
        <section>
            <h2>Второй раздел</h2>
            
            <section>
                <h3>Подраздел второго раздела</h3>
                <p>Содержимое подраздела...</p>
                
                <section>
                    <h4>Еще один уровень</h4>
                    <p>Содержимое еще одного уровня...</p>
                </section>
            </section>
        </section>
    </main>
</body>
</html>
```

## ARIA (Accessible Rich Internet Applications)

### Основные ARIA атрибуты

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA атрибуты</title>
</head>
<body>
    <h1>ARIA атрибуты для доступности</h1>
    
    <!-- Роли элементов -->
    <div role="banner">Шапка сайта</div>
    <div role="main">Основное содержимое</div>
    <div role="contentinfo">Подвал сайта</div>
    <div role="search">Поиск</div>
    
    <!-- Состояния элементов -->
    <input type="checkbox" 
           id="terms-agreement" 
           name="terms"
           role="checkbox"
           aria-checked="false">
    <label for="terms-agreement">Согласен с условиями</label>
    
    <!-- Свойства элементов -->
    <div role="listbox" aria-labelledby="list-title">
        <h2 id="list-title">Список задач</h2>
        <div role="option" 
             aria-selected="false"
             aria-describedby="task-desc">
            Задача 1
        </div>
        <div role="option" 
             aria-selected="true"
             aria-describedby="task-desc">
            Задача 2
        </div>
    </div>
    
    <!-- Живые области -->
    <div aria-live="polite" aria-atomic="true" id="status-updates">
        <!-- Статусные обновления будут отображаться здесь -->
    </div>
    
    <!-- Описание элементов -->
    <div id="task-desc" class="sr-only">
        Задача, которую нужно выполнить
    </div>
    
    <!-- Связывание элементов -->
    <input type="text" 
           id="name-input" 
           name="name"
           aria-describedby="name-help"
           aria-labelledby="name-label">
    <div id="name-label">Имя пользователя</div>
    <div id="name-help">Введите ваше полное имя</div>
    
    <!-- Навигация по приложению -->
    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
            <li><a href="/contact">Контакты</a></li>
        </ul>
    </nav>
</body>
</html>
```

### ARIA для сложных компонентов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA для сложных компонентов</title>
</head>
<body>
    <h1>ARIA для сложных интерактивных компонентов</h1>
    
    <!-- Вкладки -->
    <div role="tablist" aria-label="Навигационные вкладки">
        <button role="tab" 
                aria-selected="true" 
                aria-controls="panel1" 
                id="tab1"
                tabindex="0">
            Вкладка 1
        </button>
        <button role="tab" 
                aria-selected="false" 
                aria-controls="panel2" 
                id="tab2"
                tabindex="-1">
            Вкладка 2
        </button>
        <button role="tab" 
                aria-selected="false" 
                aria-controls="panel3" 
                id="tab3"
                tabindex="-1">
            Вкладка 3
        </button>
    </div>
    
    <div id="panel1" 
         role="tabpanel" 
         aria-labelledby="tab1">
        <h2>Содержимое первой вкладки</h2>
        <p>Текст первой вкладки...</p>
    </div>
    <div id="panel2" 
         role="tabpanel" 
         aria-labelledby="tab2" 
         hidden>
        <h2>Содержимое второй вкладки</h2>
        <p>Текст второй вкладки...</p>
    </div>
    <div id="panel3" 
         role="tabpanel" 
         aria-labelledby="tab3" 
         hidden>
        <h2>Содержимое третьей вкладки</h2>
        <p>Текст третьей вкладки...</p>
    </div>
    
    <!-- Аккордеон -->
    <div role="main">
        <div class="accordion-item" role="region" aria-labelledby="accordion1-title">
            <h3 id="accordion1-title">
                <button aria-expanded="true" 
                        aria-controls="accordion1-content"
                        onclick="toggleAccordion('accordion1-content', this)">
                    Заголовок аккордеона 1
                </button>
            </h3>
            <div id="accordion1-content" role="region" aria-labelledby="accordion1-title">
                <p>Содержимое первого аккордеона...</p>
            </div>
        </div>
        
        <div class="accordion-item" role="region" aria-labelledby="accordion2-title">
            <h3 id="accordion2-title">
                <button aria-expanded="false" 
                        aria-controls="accordion2-content"
                        onclick="toggleAccordion('accordion2-content', this)">
                    Заголовок аккордеона 2
                </button>
            </h3>
            <div id="accordion2-content" role="region" aria-labelledby="accordion2-title" hidden>
                <p>Содержимое второго аккордеона...</p>
            </div>
        </div>
    </div>
    
    <!-- Раскрывающийся список -->
    <div role="combobox" 
         aria-haspopup="listbox" 
         aria-expanded="false" 
         aria-owns="dropdown-list">
        <input type="text" 
               role="combobox" 
               aria-autocomplete="list" 
               aria-controls="dropdown-list"
               placeholder="Выберите опцию...">
        <ul role="listbox" 
            id="dropdown-list" 
            style="display: none; border: 1px solid #ccc; background: white;">
            <li role="option">Опция 1</li>
            <li role="option">Опция 2</li>
            <li role="option">Опция 3</li>
        </ul>
    </div>

    <script>
        function toggleAccordion(contentId, button) {
            const content = document.getElementById(contentId);
            const isExpanded = button.getAttribute('aria-expanded') === 'true';
            
            button.setAttribute('aria-expanded', !isExpanded);
            content.hidden = isExpanded;
        }
        
        // Управление вкладками
        document.querySelectorAll('[role="tab"]').forEach(tab => {
            tab.addEventListener('click', () => {
                switchTab(tab);
            });
            
            tab.addEventListener('keydown', (e) => {
                switch(e.key) {
                    case 'ArrowLeft':
                    case 'ArrowRight':
                        e.preventDefault();
                        navigateTabs(e.key, tab);
                        break;
                    case 'Home':
                        e.preventDefault();
                        selectFirstTab();
                        break;
                    case 'End':
                        e.preventDefault();
                        selectLastTab();
                        break;
                    case 'Enter':
                    case ' ':
                        e.preventDefault();
                        switchTab(tab);
                        break;
                }
            });
        });
        
        function switchTab(clickedTab) {
            // Снятие выделения с текущей вкладки
            document.querySelectorAll('[role="tab"]').forEach(tab => {
                tab.setAttribute('aria-selected', 'false');
                tab.setAttribute('tabindex', '-1');
            });
            
            // Установка выделения на новую вкладку
            clickedTab.setAttribute('aria-selected', 'true');
            clickedTab.setAttribute('tabindex', '0');
            clickedTab.focus();
            
            // Скрытие всех панелей
            document.querySelectorAll('[role="tabpanel"]').forEach(panel => {
                panel.hidden = true;
            });
            
            // Показ соответствующей панели
            const panelId = clickedTab.getAttribute('aria-controls');
            document.getElementById(panelId).hidden = false;
        }
        
        function navigateTabs(direction, currentTab) {
            const tabs = document.querySelectorAll('[role="tab"]');
            const currentIndex = Array.from(tabs).indexOf(currentTab);
            
            let newIndex;
            if (direction === 'ArrowLeft') {
                newIndex = currentIndex === 0 ? tabs.length - 1 : currentIndex - 1;
            } else {
                newIndex = currentIndex === tabs.length - 1 ? 0 : currentIndex + 1;
            }
            
            switchTab(tabs[newIndex]);
        }
        
        function selectFirstTab() {
            const firstTab = document.querySelector('[role="tab"]');
            if (firstTab) switchTab(firstTab);
        }
        
        function selectLastTab() {
            const tabs = document.querySelectorAll('[role="tab"]');
            if (tabs.length > 0) switchTab(tabs[tabs.length - 1]);
        }
    </script>
    
    <style>
        [role="tab"] {
            background-color: #f8f9fa;
            border: 1px solid #ddd;
            border-bottom: none;
            padding: 10px 20px;
            cursor: pointer;
        }
        
        [role="tab"][aria-selected="true"] {
            background-color: white;
            border-bottom: 1px solid white;
            margin-bottom: -1px;
        }
        
        [role="tabpanel"] {
            border: 1px solid #ddd;
            padding: 20px;
            background-color: white;
        }
        
        .accordion-item button {
            width: 100%;
            padding: 15px;
            border: 1px solid #ddd;
            background: #f8f9fa;
            cursor: pointer;
            font-size: 16px;
        }
        
        .accordion-item button:focus {
            outline: 2px solid #007acc;
        }
        
        [role="option"] {
            padding: 10px;
            cursor: pointer;
        }
        
        [role="option"]:hover {
            background-color: #f8f9fa;
        }
    </style>
</body>
</html>
```

## Клавиатурная навигация

### Создание доступной клавиатурной навигации

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Клавиатурная навигация</title>
</head>
<body>
    <h1>Клавиатурная навигация</h1>
    
    <!-- Ссылка для пропуска навигации -->
    <a href="#main-content" class="skip-link">Перейти к основному содержимому</a>
    
    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
            <li><a href="/services">Услуги</a></li>
            <li><a href="/portfolio">Портфолио</a></li>
            <li><a href="/blog">Блог</a></li>
            <li><a href="/contact">Контакты</a></li>
        </ul>
    </nav>
    
    <main id="main-content" role="main">
        <h2>Основное содержимое</h2>
        
        <!-- Интерактивные элементы с доступной навигацией -->
        <div class="interactive-section">
            <button class="nav-item" tabindex="0">Кнопка 1</button>
            <button class="nav-item" tabindex="0">Кнопка 2</button>
            <button class="nav-item" tabindex="0">Кнопка 3</button>
            <div class="custom-element" 
                 tabindex="0" 
                 role="button" 
                 aria-label="Кастомный элемент"
                 onclick="handleCustomClick(this)">
                Кастомный элемент
            </div>
        </div>
        
        <!-- Карточки с клавиатурной навигацией -->
        <div class="cards-container">
            <div class="card" tabindex="0" role="button" aria-label="Карточка 1">
                <h3>Карточка 1</h3>
                <p>Содержимое первой карточки</p>
            </div>
            <div class="card" tabindex="0" role="button" aria-label="Карточка 2">
                <h3>Карточка 2</h3>
                <p>Содержимое второй карточки</p>
            </div>
            <div class="card" tabindex="0" role="button" aria-label="Карточка 3">
                <h3>Карточка 3</h3>
                <p>Содержимое третьей карточки</p>
            </div>
        </div>
    </main>

    <script>
        class KeyboardNavigationManager {
            constructor() {
                this.setupSkipLinks();
                this.setupCardNavigation();
                this.setupFocusIndicators();
            }
            
            setupSkipLinks() {
                // Обработка ссылок для пропуска навигации
                document.querySelectorAll('.skip-link').forEach(link => {
                    link.addEventListener('focus', () => {
                        link.style.position = 'static';
                        link.style.left = '0';
                    });
                    
                    link.addEventListener('blur', () => {
                        link.style.position = 'absolute';
                        link.style.left = '-9999px';
                    });
                });
            }
            
            setupCardNavigation() {
                // Обработка клавиатурной навигации по карточкам
                document.querySelectorAll('.card[tabindex="0"]').forEach(card => {
                    card.addEventListener('keydown', (e) => {
                        if (e.key === 'Enter' || e.key === ' ') {
                            e.preventDefault();
                            this.handleCardActivation(card);
                        }
                    });
                    
                    card.addEventListener('click', () => {
                        this.handleCardActivation(card);
                    });
                });
            }
            
            handleCardActivation(card) {
                const title = card.querySelector('h3').textContent;
                alert(`Активирована карточка: ${title}`);
            }
            
            setupFocusIndicators() {
                // Обработка показа индикаторов фокуса для клавиатурной навигации
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'Tab') {
                        document.body.classList.add('keyboard-navigation');
                    }
                });
                
                document.addEventListener('mousedown', () => {
                    document.body.classList.remove('keyboard-navigation');
                });
                
                // Улучшенные индикаторы фокуса
                document.querySelectorAll('a, button, input, select, textarea, [tabindex]').forEach(element => {
                    element.addEventListener('focus', () => {
                        element.classList.add('focused');
                    });
                    
                    element.addEventListener('blur', () => {
                        element.classList.remove('focused');
                    });
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new KeyboardNavigationManager();
        });
        
        function handleCustomClick(element) {
            alert('Кастомный элемент активирован');
        }
    </script>
    
    <style>
        .skip-link {
            position: absolute;
            top: -40px;
            left: 6px;
            background: #000;
            color: #fff;
            padding: 8px;
            text-decoration: none;
            border-radius: 4px;
            z-index: 1000;
        }
        
        .skip-link:focus {
            top: 6px;
        }
        
        .keyboard-navigation :focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
        
        .nav-item, .custom-element, .card {
            padding: 12px;
            margin: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.3s ease;
        }
        
        .nav-item:hover, .custom-element:hover, .card:hover {
            background-color: #f0f0f0;
        }
        
        .interactive-section {
            margin: 20px 0;
        }
        
        .cards-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
        }
        
        .card {
            background-color: white;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .card:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
        
        nav ul {
            list-style: none;
            padding: 0;
            display: flex;
            gap: 15px;
            margin: 20px 0;
        }
        
        nav a {
            padding: 10px 15px;
            text-decoration: none;
            border-radius: 4px;
        }
        
        nav a:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
    </style>
</body>
</html>
```

### Управление фокусом

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Управление фокусом</title>
</head>
<body>
    <h1>Управление фокусом</h1>
    
    <button id="open-modal">Открыть модальное окно</button>
    
    <div id="modal-overlay" class="modal-overlay" style="display: none;" role="dialog" aria-modal="true">
        <div class="modal-content" role="document" aria-labelledby="modal-title">
            <div class="modal-header">
                <h2 id="modal-title">Модальное окно</h2>
                <button id="close-modal" aria-label="Закрыть модальное окно">×</button>
            </div>
            <div class="modal-body">
                <p>Содержимое модального окна</p>
                <button id="modal-action">Действие в модальном окне</button>
            </div>
            <div class="modal-footer">
                <button id="modal-confirm">Подтвердить</button>
                <button id="modal-cancel">Отмена</button>
            </div>
        </div>
    </div>

    <script>
        class FocusManager {
            constructor() {
                this.modalOverlay = document.getElementById('modal-overlay');
                this.openModalBtn = document.getElementById('open-modal');
                this.closeModalBtn = document.getElementById('close-modal');
                this.modalActionBtn = document.getElementById('modal-action');
                this.modalConfirmBtn = document.getElementById('modal-confirm');
                this.modalCancelBtn = document.getElementById('modal-cancel');
                
                this.lastFocusedElement = null;
                this.focusableElements = null;
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.openModalBtn.addEventListener('click', () => {
                    this.openModal();
                });
                
                this.closeModalBtn.addEventListener('click', () => {
                    this.closeModal();
                });
                
                this.modalCancelBtn.addEventListener('click', () => {
                    this.closeModal();
                });
                
                // Закрытие по Escape
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'Escape' && this.modalOverlay.style.display === 'block') {
                        this.closeModal();
                    }
                });
            }
            
            openModal() {
                // Сохраняем последний сфокусированный элемент
                this.lastFocusedElement = document.activeElement;
                
                // Показываем модальное окно
                this.modalOverlay.style.display = 'flex';
                
                // Устанавливаем фокус на первое интерактивное поле в модальном окне
                this.modalActionBtn.focus();
                
                // Ограничиваем фокус внутри модального окна
                this.trapFocus();
            }
            
            closeModal() {
                // Скрываем модальное окно
                this.modalOverlay.style.display = 'none';
                
                // Возвращаем фокус на предыдущий элемент
                if (this.lastFocusedElement && this.lastFocusedElement.focus) {
                    this.lastFocusedElement.focus();
                }
            }
            
            trapFocus() {
                // Получаем все фокусируемые элементы в модальном окне
                this.focusableElements = this.modalOverlay.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                
                if (this.focusableElements.length === 0) return;
                
                const firstElement = this.focusableElements[0];
                const lastElement = this.focusableElements[this.focusableElements.length - 1];
                
                // Обработка Tab и Shift+Tab
                this.modalOverlay.addEventListener('keydown', (e) => {
                    if (e.key === 'Tab') {
                        if (e.shiftKey) {
                            if (document.activeElement === firstElement) {
                                lastElement.focus();
                                e.preventDefault();
                            }
                        } else {
                            if (document.activeElement === lastElement) {
                                firstElement.focus();
                                e.preventDefault();
                            }
                        }
                    }
                });
            }
            
            // Методы для других типов компонентов
            setupDropdownFocus() {
                // Управление фокусом для раскрывающихся списков
            }
            
            setupMenuFocus() {
                // Управление фокусом для меню
            }
            
            setupCarouselFocus() {
                // Управление фокусом для каруселей
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new FocusManager();
        });
    </script>
    
    <style>
        .modal-overlay {
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
            padding: 20px;
            border-radius: 8px;
            min-width: 400px;
            max-width: 600px;
        }
        
        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }
        
        .modal-header h2 {
            margin: 0;
        }
        
        .modal-header button {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
        }
        
        .modal-footer {
            margin-top: 20px;
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
        
        .modal-footer #modal-confirm {
            background-color: #28a745;
            color: white;
        }
        
        .modal-footer #modal-cancel {
            background-color: #6c757d;
            color: white;
        }
    </style>
</body>
</html>
```

## Альтернативный текст и изображения

### Создание доступных изображений

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные изображения</title>
</head>
<body>
    <h1>Доступные изображения</h1>
    
    <!-- Контентное изображение -->
    <img src="product-image.jpg" 
         alt="Смартфон iPhone 15 в золотом цвете с 6.1 дюймовым экраном, показывающий его характеристики" 
         width="300" 
         height="200">
    
    <!-- Декоративное изображение -->
    <img src="decoration.jpg" alt="" role="presentation">
    
    <!-- Изображение с длинным описанием -->
    <figure>
        <img src="complex-chart.jpg" 
             alt="Диаграмма роста продаж за последние 5 лет, показывающая увеличение на 15% в год"
             longdesc="chart-description.html">
        <figcaption>Рост продаж за последние 5 лет</figcaption>
    </figure>
    
    <!-- Изображение с лейблом -->
    <div>
        <img src="icon-warning.png" 
             alt="Предупреждение" 
             aria-labelledby="warning-title"
             width="24" 
             height="24">
        <h3 id="warning-title">Важное предупреждение</h3>
        <p>Содержимое предупреждения...</p>
    </div>
    
    <!-- Изображение с описанием в aria-describedby -->
    <div>
        <img src="process-diagram.png" 
             alt="Процесс регистрации пользователя"
             aria-describedby="process-desc">
        <div id="process-desc">
            <h4>Процесс регистрации:</h4>
            <ol>
                <li>Ввод имени и email</li>
                <li>Создание пароля</li>
                <li>Подтверждение email</li>
                <li>Активация аккаунта</li>
            </ol>
        </div>
    </div>
    
    <!-- Изображение с кастомным описанием -->
    <div class="image-with-description">
        <img src="infographic.jpg" 
             alt="Инфографика о преимуществах HTML5"
             aria-details="infographic-details">
        <div id="infographic-details" hidden>
            <h4>Детали инфографики:</h4>
            <ul>
                <li>Семантические элементы для лучшей структуры</li>
                <li>Встроенная поддержка мультимедиа</li>
                <li>Улучшенные возможности форм</li>
                <li>Лучшая производительность</li>
            </ul>
        </div>
    </div>
</body>
</html>
```

### Доступные медиа-элементы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные медиа-элементы</title>
</head>
<body>
    <h1>Доступные медиа-элементы</h1>
    
    <!-- Видео с субтитрами и описаниями -->
    <figure>
        <video controls 
               width="640" 
               height="360"
               aria-labelledby="video-title"
               aria-describedby="video-description">
            <source src="video.mp4" type="video/mp4">
            <source src="video.webm" type="video/webm">
            
            <!-- Субтитры -->
            <track kind="subtitles" 
                   src="subtitles-ru.vtt" 
                   srclang="ru" 
                   label="Русские">
            <track kind="subtitles" 
                   src="subtitles-en.vtt" 
                   srclang="en" 
                   label="English">
            
            <!-- Описания для слабовидящих -->
            <track kind="descriptions" 
                   src="descriptions.vtt" 
                   srclang="ru" 
                   label="Описания">
            
            <!-- Титры -->
            <track kind="captions" 
                   src="captions.vtt" 
                   srclang="ru" 
                   label="Титры (включая звуки)">
            
            Ваш браузер не поддерживает видео элемент.
        </video>
        
        <figcaption id="video-description">
            Обучающее видео о возможностях HTML5. Продолжительность: 15 минут.
        </figcaption>
    </figure>
    
    <!-- Аудио с транскрипцией -->
    <figure>
        <audio controls aria-labelledby="audio-title">
            <source src="podcast.mp3" type="audio/mpeg">
            <source src="podcast.ogg" type="audio/ogg">
            Ваш браузер не поддерживает аудио элемент.
        </audio>
        
        <figcaption>
            <h3 id="audio-title">Подкаст: Современные веб-технологии</h3>
            <p>Обсуждение последних тенденций в веб-разработке.</p>
        </figcaption>
        
        <!-- Транскрипция аудио -->
        <details aria-labelledby="transcription-title">
            <summary id="transcription-title">Транскрипция подкаста</summary>
            <div class="transcription">
                <p>Ведущий: Добро пожаловать на наш подкаст о современных веб-технологиях...</p>
                <p>Гость: Сегодня мы поговорим о новых возможностях HTML5...</p>
                <!-- Полная транскрипция -->
            </div>
        </details>
    </figure>
    
    <!-- Интерактивная карта с доступностью -->
    <div class="interactive-map" 
         role="application" 
         aria-label="Интерактивная карта офисов компании"
         tabindex="0">
        <div class="map-container">
            <img src="map.jpg" 
                 alt="Карта офисов компании в Москве, Санкт-Петербурге и Новосибирске"
                 usemap="#office-map">
            
            <map name="office-map">
                <area shape="rect" 
                      coords="100,100,200,200" 
                      href="/moscow-office" 
                      alt="Офис в Москве"
                      aria-describedby="moscow-info">
                <area shape="circle" 
                      coords="300,300,50" 
                      href="/spb-office" 
                      alt="Офис в Санкт-Петербурге"
                      aria-describedby="spb-info">
                <area shape="poly" 
                      coords="400,100,450,150,400,200,350,150" 
                      href="/nsk-office" 
                      alt="Офис в Новосибирске"
                      aria-describedby="nsk-info">
            </map>
        </div>
        
        <div id="moscow-info" hidden>Офис в Москве: ул. Тверская, д. 1, эт. 5</div>
        <div id="spb-info" hidden>Офис в Санкт-Петербурге: Невский проспект, д. 20, эт. 3</div>
        <div id="nsk-info" hidden>Офис в Новосибирске: ул. Красный проспект, д. 5, эт. 2</div>
    </div>
</body>
</html>
```

## Доступные формы

### Создание доступных форм

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные формы</title>
</head>
<body>
    <h1>Доступные формы</h1>
    
    <form id="accessible-form" novalidate>
        <!-- Использование fieldset и legend для группировки -->
        <fieldset>
            <legend>Личная информация</legend>
            
            <div class="form-group">
                <label for="acc-name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="text"
                       id="acc-name"
                       name="name"
                       required
                       minlength="2"
                       maxlength="50"
                       autocomplete="name"
                       aria-describedby="name-help name-error"
                       aria-invalid="false">
                <div id="name-help" class="help-text">Введите ваше имя</div>
                <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="acc-email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="email"
                       id="acc-email"
                       name="email"
                       required
                       autocomplete="email"
                       aria-describedby="email-help email-error"
                       aria-invalid="false">
                <div id="email-help" class="help-text">Введите действительный email адрес</div>
                <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>
        </fieldset>
        
        <fieldset>
            <legend>Настройки аккаунта</legend>
            
            <div class="form-group">
                <label for="acc-username">Имя пользователя: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="text"
                       id="acc-username"
                       name="username"
                       required
                       minlength="3"
                       maxlength="30"
                       pattern="[A-Za-z0-9_]+"
                       autocomplete="username"
                       aria-describedby="username-help username-error"
                       aria-invalid="false">
                <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-30 символов</div>
                <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="acc-password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="acc-password"
                       name="password"
                       required
                       minlength="8"
                       aria-describedby="password-requirements password-error"
                       aria-invalid="false">
                <ul id="password-requirements" class="requirements-list">
                    <li id="req-length">Не менее 8 символов</li>
                    <li id="req-upper">С заглавной буквой</li>
                    <li id="req-lower">Со строчной буквой</li>
                    <li id="req-number">С цифрой</li>
                    <li id="req-special">Со специальным символом</li>
                </ul>
                <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="acc-confirm">Подтверждение пароля: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="acc-confirm"
                       name="confirm_password"
                       required
                       aria-describedby="confirm-error"
                       aria-invalid="false">
                <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
        </fieldset>
        
        <fieldset>
            <legend>Дополнительная информация</legend>
            
            <div class="form-group">
                <label for="acc-bio">О себе:</label>
                <textarea id="acc-bio"
                          name="bio"
                          rows="4"
                          maxlength="500"
                          aria-describedby="bio-help bio-error"></textarea>
                <div id="bio-help" class="help-text">Расскажите немного о себе (максимум 500 символов)</div>
                <div id="bio-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>
            
            <div class="form-group">
                <label for="acc-country">Страна:</label>
                <select id="acc-country" name="country" aria-describedby="country-help">
                    <option value="">Выберите страну</option>
                    <option value="ru">Россия</option>
                    <option value="ua">Украина</option>
                    <option value="by">Беларусь</option>
                    <option value="kz">Казахстан</option>
                </select>
                <div id="country-help" class="help-text">Выберите вашу страну</div>
            </div>
            
            <div class="form-group">
                <fieldset aria-describedby="interests-help">
                    <legend>Интересы:</legend>
                    <div id="interests-help" class="help-text">Выберите ваши интересы (можно несколько)</div>
                    <div class="checkbox-group">
                        <label>
                            <input type="checkbox" name="interests" value="tech" aria-describedby="tech-desc">
                            Технологии
                        </label>
                        <div id="tech-desc" class="sr-only">Интерес к современным технологиям</div>
                        
                        <label>
                            <input type="checkbox" name="interests" value="sports" aria-describedby="sports-desc">
                            Спорт
                        </label>
                        <div id="sports-desc" class="sr-only">Интерес к спортивным мероприятиям</div>
                        
                        <label>
                            <input type="checkbox" name="interests" value="music" aria-describedby="music-desc">
                            Музыка
                        </label>
                        <div id="music-desc" class="sr-only">Интерес к музыкальным событиям</div>
                    </div>
                </fieldset>
            </div>
            
            <div class="form-group">
                <fieldset aria-describedby="gender-help">
                    <legend>Пол:</legend>
                    <div id="gender-help" class="help-text">Выберите ваш пол</div>
                    <label>
                        <input type="radio" name="gender" value="male">
                        Мужской
                    </label>
                    <label>
                        <input type="radio" name="gender" value="female">
                        Женский
                    </label>
                    <label>
                        <input type="radio" name="gender" value="other">
                        Другой
                    </label>
                </fieldset>
            </div>
        </fieldset>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="newsletter"
                       value="yes">
                Подписаться на рассылку
            </label>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="terms"
                       required
                       aria-describedby="terms-description">
                Я согласен с условиями использования <span class="required" aria-label="обязательное поле">*</span>
            </label>
            <div id="terms-description" class="help-text">Обязательное поле для регистрации</div>
        </div>
        
        <button type="submit">Зарегистрироваться</button>
    </form>

    <script>
        class AccessibleFormHandler {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupAccessibleValidation();
            }
            
            setupAccessibleValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                });
                
                // Валидация при потере фокуса
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input, textarea, select')) {
                        this.validateField(e.target);
                    }
                }, true);
            }
            
            validateAndSubmit() {
                // Сброс ошибок
                this.clearAllErrors();
                
                let isValid = true;
                
                // Проверка всех обязательных полей
                const fields = this.form.querySelectorAll('input[required], textarea[required]');
                for (const field of fields) {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                }
                
                // Проверка сложных требований
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }
                
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }
                
                if (isValid) {
                    this.submitSecurely();
                } else {
                    // Сообщение для скринридеров о наличии ошибок
                    const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
                    this.announceToScreenReader(`Форма содержит ${errorCount} ошибок. Пожалуйста, исправьте их перед отправкой.`);
                    
                    // Установка фокуса на первое поле с ошибкой
                    const firstErrorField = this.form.querySelector('.invalid');
                    if (firstErrorField) {
                        firstErrorField.focus();
                    }
                }
            }
            
            validateField(field) {
                const fieldName = field.name.replace(/-/g, '');
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showFieldError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                if (field.pattern && field.value && !new RegExp(field.pattern).test(field.value)) {
                    this.showFieldError(field, errorElement, field.title || 'Неверный формат данных');
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            validatePasswordRequirements() {
                const password = document.getElementById('acc-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                // Обновление индикаторов требований
                Object.entries(requirements).forEach(([key, met]) => {
                    const element = document.getElementById(`req-${key}`);
                    if (element) {
                        element.classList.toggle('met', met);
                    }
                });
                
                const allMet = Object.values(requirements).every(req => req);
                
                if (!allMet && password) {
                    const errorElement = document.getElementById('password-error');
                    this.showFieldError(document.getElementById('acc-password'), errorElement, 'Пароль не соответствует требованиям');
                    return false;
                }
                
                return allMet;
            }
            
            validatePasswordMatch() {
                const password = document.getElementById('acc-password').value;
                const confirmPassword = document.getElementById('acc-confirm').value;
                const errorElement = document.getElementById('confirm-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    this.showFieldError(document.getElementById('acc-confirm'), errorElement, 'Пароли не совпадают');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            showFieldError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Устанавливаем фокус на поле с ошибкой
                if (!field.matches(':focus')) {
                    field.focus();
                }
            }
            
            clearAllErrors() {
                this.form.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                this.form.querySelectorAll('input, textarea, select').forEach(field => {
                    field.classList.remove('invalid', 'valid');
                    field.setAttribute('aria-invalid', 'false');
                });
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            async submitSecurely() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор и санитизация данных
                    const formData = new FormData(this.form);
                    const sanitizedData = {};
                    
                    for (const [key, value] of formData.entries()) {
                        sanitizedData[key] = this.sanitizeInput(value);
                    }
                    
                    const response = await fetch('/api/accessible-register', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Регистрация прошла успешно!');
                        this.form.reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка регистрации: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
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
                }, 3000);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleFormHandler('accessible-form');
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        fieldset {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 1.5rem;
            margin-bottom: 1.5rem;
        }
        
        legend {
            font-weight: bold;
            padding: 0 0.5rem;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.valid {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }
        
        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }
        
        .requirements-list li.met {
            color: #28a745;
        }
        
        .checkbox-group {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .required {
            color: #d32f2f;
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

## Практические примеры и шаблоны

### Шаблон доступной формы с многоуровневой валидацией
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная форма с многоуровневой валидацией</title>
</head>
<body>
    <h1>Доступная форма с многоуровневой валидацией</h1>
    
    <form id="multi-level-accessible-form" novalidate>
        <div class="form-level-1">
            <h2>Базовая информация</h2>
            
            <div class="form-group">
                <label for="level1-name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="text"
                       id="level1-name"
                       name="name"
                       required
                       minlength="2"
                       maxlength="50"
                       autocomplete="name"
                       aria-describedby="name-help name-error"
                       aria-invalid="false">
                <div id="name-help" class="help-text">Введите ваше имя</div>
                <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="level1-email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="email"
                       id="level1-email"
                       name="email"
                       required
                       autocomplete="email"
                       aria-describedby="email-help email-error"
                       aria-invalid="false">
                <div id="email-help" class="help-text">Введите действительный email адрес</div>
                <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>
        </div>
        
        <div class="form-level-2" hidden>
            <h2>Дополнительная информация</h2>
            
            <div class="form-group">
                <label for="level2-phone">Телефон:</label>
                <input type="tel"
                       id="level2-phone"
                       name="phone"
                       autocomplete="tel"
                       aria-describedby="phone-help phone-error"
                       aria-invalid="false">
                <div id="phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
                <div id="phone-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>
            
            <div class="form-group">
                <label for="level2-bio">О себе:</label>
                <textarea id="level2-bio"
                          name="bio"
                          rows="4"
                          maxlength="500"
                          aria-describedby="bio-help bio-error"></textarea>
                <div id="bio-help" class="help-text">Расскажите немного о себе (максимум 500 символов)</div>
                <div id="bio-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>
        </div>
        
        <div class="form-level-3" hidden>
            <h2>Настройки безопасности</h2>
            
            <div class="form-group">
                <label for="level3-password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="level3-password"
                       name="password"
                       required
                       minlength="8"
                       aria-describedby="password-requirements password-error"
                       aria-invalid="false">
                <ul id="password-requirements" class="requirements-list">
                    <li id="req-length">Не менее 8 символов</li>
                    <li id="req-upper">С заглавной буквой</li>
                    <li id="req-lower">Со строчной буквой</li>
                    <li id="req-number">С цифрой</li>
                    <li id="req-special">Со специальным символом</li>
                </ul>
                <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="level3-confirm">Подтверждение пароля: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="level3-confirm"
                       name="confirm_password"
                       required
                       aria-describedby="confirm-error"
                       aria-invalid="false">
                <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
        </div>
        
        <div class="form-navigation">
            <button type="button" id="prev-btn" disabled>Назад</button>
            <button type="button" id="next-btn">Далее</button>
            <button type="submit" id="submit-btn" hidden>Отправить</button>
        </div>
    </form>

    <script>
        class MultiLevelAccessibleForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.currentLevel = 1;
                this.maxLevel = 3;
                
                this.level1 = this.form.querySelector('.form-level-1');
                this.level2 = this.form.querySelector('.form-level-2');
                this.level3 = this.form.querySelector('.form-level-3');
                
                this.prevBtn = document.getElementById('prev-btn');
                this.nextBtn = document.getElementById('next-btn');
                this.submitBtn = document.getElementById('submit-btn');
                
                this.setupNavigation();
                this.setupValidation();
            }
            
            setupNavigation() {
                document.getElementById('next-btn').addEventListener('click', () => {
                    this.goToNextLevel();
                });
                
                document.getElementById('prev-btn').addEventListener('click', () => {
                    this.goToPrevLevel();
                });
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация на каждом уровне
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches(`.form-level-${this.currentLevel} input[required], .form-level-${this.currentLevel} textarea[required]`)) {
                        this.validateField(e.target);
                    }
                });
            }
            
            goToNextLevel() {
                if (this.currentLevel === 1) {
                    // Проверка первого уровня
                    if (this.validateLevel1()) {
                        this.showLevel(2);
                    }
                } else if (this.currentLevel === 2) {
                    this.showLevel(3);
                }
            }
            
            goToPrevLevel() {
                if (this.currentLevel === 2) {
                    this.showLevel(1);
                } else if (this.currentLevel === 3) {
                    this.showLevel(2);
                }
            }
            
            showLevel(level) {
                // Скрытие всех уровней
                this.level1.hidden = true;
                this.level2.hidden = true;
                this.level3.hidden = true;
                
                // Показ выбранного уровня
                if (level === 1) {
                    this.level1.hidden = false;
                    this.prevBtn.disabled = true;
                } else if (level === 2) {
                    this.level2.hidden = false;
                    this.prevBtn.disabled = false;
                } else if (level === 3) {
                    this.level3.hidden = false;
                    this.nextBtn.hidden = true;
                    this.submitBtn.hidden = false;
                }
                
                this.currentLevel = level;
                
                // Фокус на первый элемент уровня
                const firstInput = this.form.querySelector(`.form-level-${level} input, .form-level-${level} textarea`);
                if (firstInput) firstInput.focus();
            }
            
            validateLevel1() {
                const nameField = document.getElementById('level1-name');
                const emailField = document.getElementById('level1-email');
                
                let isValid = true;
                
                if (!this.validateField(nameField)) isValid = false;
                if (!this.validateField(emailField)) isValid = false;
                
                return isValid;
            }
            
            validateField(field) {
                const fieldName = field.name.replace(/-/g, '');
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showFieldError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            validateAndSubmit() {
                // Валидация всех уровней
                let isValid = true;
                
                // Проверка всех полей на всех уровнях
                const allFields = this.form.querySelectorAll('input[required], textarea[required]');
                allFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });
                
                // Проверка сложных требований
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }
                
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }
                
                if (isValid) {
                    this.submitSecurely();
                }
            }
            
            validatePasswordRequirements() {
                const password = document.getElementById('level3-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                // Обновление индикаторов требований
                Object.entries(requirements).forEach(([key, met]) => {
                    const element = document.getElementById(`req-${key}`);
                    if (element) {
                        element.classList.toggle('met', met);
                    }
                });
                
                const allMet = Object.values(requirements).every(req => req);
                
                if (!allMet && password) {
                    const errorElement = document.getElementById('password-error');
                    this.showFieldError(document.getElementById('level3-password'), errorElement, 'Пароль не соответствует требованиям');
                    return false;
                }
                
                return allMet;
            }
            
            validatePasswordMatch() {
                const password = document.getElementById('level3-password').value;
                const confirmPassword = document.getElementById('level3-confirm').value;
                const errorElement = document.getElementById('confirm-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    this.showFieldError(document.getElementById('level3-confirm'), errorElement, 'Пароли не совпадают');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            showFieldError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Устанавливаем фокус на поле с ошибкой
                if (!field.matches(':focus')) {
                    field.focus();
                }
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            async submitSecurely() {
                const submitBtn = document.getElementById('submit-btn');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор данных формы
                    const formData = new FormData(this.form);
                    const data = Object.fromEntries(formData);
                    
                    const response = await fetch('/api/multi-level-submit', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(data)
                    });
                    
                    if (response.ok) {
                        alert('Форма успешно отправлена!');
                        this.form.reset();
                        this.showLevel(1); // Возврат к первому уровню
                    } else {
                        const error = await response.json();
                        alert('Ошибка: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MultiLevelAccessibleForm('multi-level-accessible-form');
        });
    </script>
    
    <style>
        .form-level-1, .form-level-2, .form-level-3 {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
        }
        
        .form-navigation {
            display: flex;
            justify-content: space-between;
            margin-top: 20px;
        }
        
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        input:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.valid {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }
        
        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }
        
        .requirements-list li.met {
            color: #28a745;
        }
        
        button {
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        #prev-btn {
            background-color: #6c757d;
            color: white;
        }
        
        #next-btn, #submit-btn {
            background-color: #007acc;
            color: white;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .required {
            color: #d32f2f;
        }
    </style>
</body>
</html>
```

### Доступные веб-компоненты с ARIA и семантикой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные веб-компоненты</title>
</head>
<body>
    <h1>Доступные веб-компоненты</h1>
    
    <accessible-modal title="Доступное модальное окно" id="accessible-modal">
        <modal-content>
            <h2>Заголовок модального окна</h2>
            <p>Содержимое модального окна с полной поддержкой доступности.</p>
            <button onclick="closeModal()">Закрыть</button>
        </modal-content>
    </accessible-modal>
    
    <form-validator form-id="user-form" id="accessible-validator"></form-validator>

    <script>
        // Доступное модальное окно
        class AccessibleModal extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                
                this.render();
                this.setupAccessibility();
            }
            
            static get observedAttributes() {
                return ['title', 'open'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'open') {
                        if (newValue !== null) {
                            this.openModal();
                        } else {
                            this.closeModal();
                        }
                    }
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: none;
                        }
                        
                        :host([open]) {
                            display: block;
                            position: fixed;
                            top: 0;
                            left: 0;
                            width: 100%;
                            height: 100%;
                            background-color: rgba(0,0,0,0.5);
                            z-index: 1000;
                        }
                        
                        .modal-content {
                            position: absolute;
                            top: 50%;
                            left: 50%;
                            transform: translate(-50%, -50%);
                            background-color: white;
                            padding: 20px;
                            border-radius: 8px;
                            min-width: 300px;
                            max-width: 600px;
                            max-height: 80vh;
                            overflow-y: auto;
                        }
                        
                        .modal-header {
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                            margin-bottom: 15px;
                            padding-bottom: 10px;
                            border-bottom: 1px solid #eee;
                        }
                        
                        .close-button {
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
                            margin-bottom: 20px;
                        }
                        
                        .modal-footer {
                            text-align: right;
                        }
                        
                        .modal-actions {
                            display: flex;
                            gap: 10px;
                            justify-content: flex-end;
                        }
                        
                        .modal-content:focus {
                            outline: none;
                        }
                    </style>
                    
                    <div class="modal-content" 
                         role="dialog" 
                         aria-modal="true" 
                         aria-labelledby="modal-title"
                         tabindex="-1">
                        <div class="modal-header">
                            <h2 id="modal-title">${this.getAttribute('title') || 'Модальное окно'}</h2>
                            <button class="close-button" 
                                    aria-label="Закрыть модальное окно" 
                                    onclick="this.closeModal()">
                                ×
                            </button>
                        </div>
                        <div class="modal-body">
                            <slot name="content"></slot>
                        </div>
                        <div class="modal-footer">
                            <slot name="actions"></slot>
                        </div>
                    </div>
                `;
            }
            
            setupAccessibility() {
                this.modalContent = this.shadowRoot.querySelector('.modal-content');
                this.closeButton = this.shadowRoot.querySelector('.close-button');
                
                // Управление фокусом
                this.focusableElements = this.shadowRoot.querySelectorAll(
                    'button, input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                
                // Обработка клавиатурных событий
                this.addEventListener('keydown', this.handleKeydown.bind(this));
            }
            
            openModal() {
                this.setAttribute('open', '');
                
                // Сохраняем текущий элемент с фокусом
                this.previousFocusedElement = document.activeElement;
                
                // Устанавливаем фокус на модальное окно
                this.modalContent.focus();
                
                // Управляем фокусом внутри модального окна
                this.trapFocus();
            }
            
            closeModal() {
                this.removeAttribute('open');
                
                // Возвращаем фокус на предыдущий элемент
                if (this.previousFocusedElement && this.previousFocusedElement.focus) {
                    this.previousFocusedElement.focus();
                }
            }
            
            trapFocus() {
                this.modalContent.addEventListener('keydown', (e) => {
                    if (e.key === 'Tab') {
                        this.handleTabKey(e);
                    }
                });
            }
            
            handleTabKey(event) {
                const focusable = this.shadowRoot.querySelectorAll(
                    'button, input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                
                const first = focusable[0];
                const last = focusable[focusable.length - 1];
                
                if (event.shiftKey && document.activeElement === first) {
                    last.focus();
                    event.preventDefault();
                } else if (!event.shiftKey && document.activeElement === last) {
                    first.focus();
                    event.preventDefault();
                }
            }
            
            handleKeydown(event) {
                if (event.key === 'Escape' && this.hasAttribute('open')) {
                    this.closeModal();
                }
            }
        }
        
        // Доступный валидатор форм
        class AccessibleFormValidator extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                this.form = null;
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['form-id'];
            }
            
            connectedCallback() {
                const formId = this.getAttribute('form-id');
                if (formId) {
                    this.form = document.getElementById(formId);
                    if (this.form) {
                        this.setupFormValidation();
                    }
                }
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'form-id') {
                        this.form = document.getElementById(newValue);
                        if (this.form) {
                            this.setupFormValidation();
                        }
                    }
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                        }
                        
                        .validation-results {
                            margin-top: 1rem;
                            padding: 0.75rem;
                            border-radius: 4px;
                            font-size: 0.9rem;
                        }
                        
                        .validation-success {
                            background-color: #d4edda;
                            color: #155724;
                        }
                        
                        .validation-error {
                            background-color: #f8d7da;
                            color: #721c24;
                        }
                        
                        .error-list {
                            margin: 0;
                            padding-left: 1.5rem;
                            margin-top: 0.5rem;
                        }
                        
                        .error-item {
                            margin-bottom: 0.25rem;
                        }
                    </style>
                    
                    <div class="validation-results" 
                         id="validation-results" 
                         role="status" 
                         aria-live="polite"></div>
                `;
            }
            
            setupFormValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateForm();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input[required], textarea[required], select[required]')) {
                        this.validateField(e.target);
                    }
                }, true);
            }
            
            validateForm() {
                let isValid = true;
                const errors = [];
                
                const requiredFields = this.form.querySelectorAll('input[required], textarea[required], select[required]');
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                        errors.push({
                            field: field.name || field.id,
                            message: this.getFieldErrorMessage(field)
                        });
                    }
                });
                
                if (isValid) {
                    this.showSuccess('Форма валидна. Готова к отправке.');
                } else {
                    this.showError(errors);
                }
                
                return isValid;
            }
            
            validateField(field) {
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                
                // Основные проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                return true;
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            getFieldErrorMessage(field) {
                if (field.hasAttribute('required') && !field.value.trim()) {
                    return 'Поле обязательно для заполнения';
                } else if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    return 'Введите действительный email адрес';
                } else if (field.minLength && field.value.length < field.minLength) {
                    return `Минимум ${field.minLength} символов`;
                } else if (field.maxLength && field.value.length > field.maxLength) {
                    return `Максимум ${field.maxLength} символов`;
                }
                return 'Неверный формат данных';
            }
            
            showError(errors) {
                const resultsElement = this.shadowRoot.getElementById('validation-results');
                resultsElement.className = 'validation-results validation-error';
                
                resultsElement.innerHTML = `
                    <strong>Найдены ошибки (${errors.length}):</strong>
                    <ul class="error-list">
                        ${errors.map(error => `
                            <li class="error-item">
                                ${error.field}: ${error.message}
                            </li>
                        `).join('')}
                    </ul>
                `;
            }
            
            showSuccess(message) {
                const resultsElement = this.shadowRoot.getElementById('validation-results');
                resultsElement.className = 'validation-results validation-success';
                resultsElement.innerHTML = `<strong>✓</strong> ${message}`;
            }
        }
        
        // Регистрация элементов
        customElements.define('accessible-modal', AccessibleModal);
        customElements.define('form-validator', AccessibleFormValidator);
        
        // Глобальные функции для демонстрации
        function openModal() {
            document.getElementById('accessible-modal').openModal();
        }
        
        function closeModal() {
            document.getElementById('accessible-modal').closeModal();
        }
    </script>
</body>
</html>
```

## Проверка и тестирование доступности

### Инструменты для проверки:
1. **axe-core** - автоматизированное тестирование доступности
2. **WAVE** - веб-инструмент для оценки доступности
3. **Lighthouse** - встроенная проверка доступности в Chrome DevTools
4. **Accessibility Insights** - расширение для браузеров
5. **Chrome DevTools Accessibility Inspector** - проверка дерева доступности
6. **NVDA** - скринридер для Windows
7. **VoiceOver** - скринридер для macOS
8. **JAWS** - скринридер для Windows

### Проверка доступности:
1. Проверка структуры документа
2. Проверка использования семантических элементов
3. Проверка ARIA атрибутов
4. Проверка клавиатурной навигации
5. Проверка контрастности цветов
6. Проверка альтернативного текста
7. Проверка управления фокусом
8. Проверка живых регионов (ARIA live regions)

### Проверка на реальных пользователях:
1. Тестирование с использованием скринридеров
2. Тестирование клавиатурной навигации
3. Тестирование с увеличением экрана
4. Тестирование с цветовыми ограничениями
5. Тестирование на различных устройствах
6. Тестирование с различными настройками браузера

## Лучшие практики доступности HTML

### 1. Семантическая разметка
```html
<!-- Правильно: использование семантических элементов -->
<main>
    <article>
        <header>
            <h1>Заголовок статьи</h1>
            <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
        </header>
        <section>
            <h2>Введение</h2>
            <p>Содержимое введения статьи...</p>
        </section>
        <section>
            <h2>Основная часть</h2>
            <p>Содержимое основной части статьи...</p>
        </section>
        <footer>
            <p>Автор: <span>Имя автора</span></p>
            <p>Теги: <a href="/tag/html">HTML</a>, <a href="/tag/accessibility">Доступность</a></p>
        </footer>
    </article>
</main>

<!-- Плохо: использование div для всего -->
<div>
    <div>Заголовок статьи</div>
    <div>Содержимое статьи...</div>
    <div>Имя автора</div>
</div>
```

### 2. Правильное использование заголовков
```html
<!-- Правильная иерархия заголовков -->
<h1>Основной заголовок страницы</h1>
<h2>Раздел 1</h2>
<h3>Подраздел 1.1</h3>
<h4>Подподраздел 1.1.1</h4>
<h3>Подраздел 1.2</h3>
<h2>Раздел 2</h2>
<h3>Подраздел 2.1</h3>
```

### 3. Альтернативный текст для изображений
```html
<!-- Хорошие примеры альтернативного текста -->
<img src="chart.png" alt="График роста продаж за последний квартал, показывающий увеличение на 25%">
<img src="logo.png" alt="Логотип компании Acme Inc">
<img src="decoration.png" alt=""> <!-- Декоративное изображение -->
<img src="photo.jpg" alt="Фотография сотрудников на корпоративном мероприятии">

<!-- Плохие примеры -->
<img src="chart.png" alt="Изображение"> <!-- Недостаточно информативно -->
<img src="photo.jpg" alt="Тут фото"> <!-- Не описывает содержимое -->
```

### 4. Формы с доступностью
```html
<!-- Доступная форма -->
<form>
    <div class="form-group">
        <label for="name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
        <input type="text"
               id="name"
               name="name"
               required
               aria-describedby="name-help name-error"
               aria-invalid="false">
        <div id="name-help" class="help-text">Введите ваше полное имя</div>
        <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <div class="form-group">
        <label for="email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
        <input type="email"
               id="email"
               name="email"
               required
               autocomplete="email"
               aria-describedby="email-help email-error"
               aria-invalid="false">
        <div id="email-help" class="help-text">Введите действительный email для связи</div>
        <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>
</form>
```

## Заключение

Доступность HTML-страниц - это комплексный подход, который включает в себя правильное использование семантических элементов, ARIA-атрибутов, управление фокусом, клавиатурную навигацию и другие техники. Создание доступных веб-сайтов и приложений позволяет использовать их всем пользователям, включая тех, кто использует вспомогательные технологии.

Ключевые аспекты доступности HTML:
1. Использование семантических элементов
2. Правильная структура документа
3. ARIA атрибуты для дополнительной информации
4. Управление фокусом и клавиатурной навигацией
5. Альтернативный текст для изображений
6. Доступные формы с понятными метками
7. Контрастность цветов
8. Совместимость с вспомогательными технологиями
9. Следование стандартам WCAG
10. Регулярное тестирование доступности
11. Использование современных возможностей безопасности
12. Поддержка когнитивной доступности
13. Моторная доступность
14. Сенсорная доступность
15. Уважение пользовательских предпочтений (prefers-reduced-motion, prefers-color-scheme)

Эти практики обеспечивают создание веб-сайтов, которые доступны для всех пользователей, независимо от их возможностей или используемых технологий.

Современные подходы к доступности включают:
- Использование Content Security Policy для предотвращения XSS
- Proper labeling and semantic markup
- Keyboard navigation and focus management
- ARIA roles, states, and properties
- Screen reader compatibility
- High contrast and color accessibility
- Reduced motion preferences
- Modern web component patterns with accessibility in mind
- Testing with automated tools and manual verification
- Following WCAG guidelines and standards
- Cognitive accessibility considerations
- Motor accessibility enhancements
- Sensory accessibility features
- Respecting user preferences and settings
- Using modern CSS for better accessibility
- Progressive enhancement principles

Эти методы позволяют создавать действительно доступные веб-приложения, которые обеспечивают равный доступ к информации и функциям для всех пользователей, включая людей с различными типами ограниченных возможностей.

Ключевые принципы доступности:
1. Perceivable - информация должна быть доступна для восприятия всеми пользователями
2. Operable - интерфейс должен быть удобен в использовании
3. Understandable - информация и интерфейс должны быть понятны
4. Robust - контент должен быть надежным и совместимым с вспомогательными технологиями
5. Inclusive design - проектирование с учетом всех пользователей
6. Universal design - создание решений, подходящих для всех
7. Respect for user preferences - уважение к настройкам пользователя
8. Progressive enhancement - постепенное улучшение функциональности
9. Graceful degradation - корректная работа при отсутствии возможностей
10. Consistent navigation - последовательная навигация по всему сайту

Современные технологии и подходы к доступности:
- Web Components с инкапсуляцией и безопасностью
- Shadow DOM для изоляции стилей и скриптов
- Custom Elements для создания семантически понятных элементов
- Intersection Observer для ленивой загрузки
- Modern CSS features for better accessibility
- Semantic HTML5 elements
- ARIA attributes and roles
- Proper focus management
- Keyboard navigation support
- Screen reader compatibility
- Reduced motion and contrast preferences
- Alternative text for images
- Accessible forms with proper labeling
- Responsive design for various devices

Эти технологии позволяют создавать действительно доступные веб-приложения, которые обеспечивают отличный пользовательский опыт для всех пользователей, независимо от их возможностей или используемых вспомогательных технологий.

Эти принципы помогают создавать веб-приложения, которые работают для всех пользователей и обеспечивают инклюзивный опыт.

## Современные лучшие практики доступности

### 1. Интеграция в процесс разработки

Современные команды разработки интегрируют доступность в каждый этап разработки:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интеграция доступности в разработку</title>
    <style>
        :root {
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --warning-color: #ffc107;
            --danger-color: #dc3545;
            --light-bg: #f8f9fa;
            --dark-text: #333;
            --border-color: #dee2e6;
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
        }

        body {
            font-family: system-ui, -apple-system, sans-serif;
            font-size: 18px;
            line-height: 1.5;
            color: var(--dark-text);
            margin: 0;
            padding: 20px;
            background-color: #fff;
        }

        .development-process {
            max-width: 1200px;
            margin: 0 auto;
        }

        .process-step {
            margin-bottom: 30px;
            padding: 20px;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            background-color: var(--light-bg);
        }

        .process-step h3 {
            margin-top: 0;
            color: var(--primary-color);
        }

        .checklist {
            list-style: none;
            padding: 0;
        }

        .checklist li {
            margin-bottom: 10px;
            padding-left: 25px;
            position: relative;
        }

        .checklist li::before {
            content: "✓";
            position: absolute;
            left: 0;
            color: var(--success-color);
            font-weight: bold;
        }

        .accessibility-tool {
            background-color: white;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 15px;
            margin: 10px 0;
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

        .focusable {
            outline: 2px solid transparent;
            outline-offset: 2px;
        }

        .focusable:focus {
            outline-color: var(--primary-color);
        }
    </style>
</head>
<body>
    <div class="development-process">
        <h1>Интеграция доступности в процесс разработки</h1>

        <section class="process-step" aria-labelledby="planning-title">
            <h3 id="planning-title">1. Планирование</h3>
            <ul class="checklist">
                <li>Включение требований доступности в спецификации</li>
                <li>Определение целевых уровней соответствия WCAG</li>
                <li>Вовлечение экспертов по доступности в команду</li>
                <li>Планирование тестирования с пользователями с ограниченными возможностями</li>
            </ul>
        </section>

        <section class="process-step" aria-labelledby="design-title">
            <h3 id="design-title">2. Дизайн</h3>
            <ul class="checklist">
                <li>Создание дизайнов с учетом принципов инклюзивного дизайна</li>
                <li>Обеспечение достаточной контрастности цветов</li>
                <li>Проектирование увеличенных сенсорных зон (минимум 44x44 пикселя)</li>
                <li>Планирование альтернативных способов навигации</li>
            </ul>
        </section>

        <section class="process-step" aria-labelledby="development-title">
            <h3 id="development-title">3. Разработка</h3>
            <ul class="checklist">
                <li>Использование семантически правильной разметки</li>
                <li>Правильное применение ARIA-атрибутов</li>
                <li>Обеспечение клавиатурной навигации</li>
                <li>Валидация и тестирование доступности в процессе разработки</li>
            </ul>

            <div class="accessibility-tool">
                <h4>Инструменты для разработчиков</h4>
                <ul>
                    <li><code>eslint-plugin-jsx-a11y</code> для проверки доступности JSX</li>
                    <li><code>axe-core</code> для автоматизированного тестирования</li>
                    <li><code>Pa11y</code> для CI/CD интеграции</li>
                    <li>Встроенные инструменты разработчика в браузерах</li>
                </ul>
            </div>
        </section>

        <section class="process-step" aria-labelledby="testing-title">
            <h3 id="testing-title">4. Тестирование</h3>
            <ul class="checklist">
                <li>Автоматизированное тестирование доступности</li>
                <li>Ручное тестирование с клавиатуры</li>
                <li>Тестирование со скринридерами (NVDA, JAWS, VoiceOver)</li>
                <li>Тестирование с реальными пользователями</li>
            </ul>
        </section>

        <section class="process-step" aria-labelledby="maintenance-title">
            <h3 id="maintenance-title">5. Поддержка</h3>
            <ul class="checklist">
                <li>Регулярный мониторинг доступности</li>
                <li>Обновление компонентов и библиотек</li>
                <li>Сбор и обработка обратной связи от пользователей</li>
                <li>Обучение команды новым практикам</li>
            </ul>
        </section>
    </div>

    <script>
        class DevelopmentAccessibilityChecker {
            constructor() {
                this.checklistItems = document.querySelectorAll('.checklist li');
                this.setupCheckboxes();
            }

            setupCheckboxes() {
                this.checklistItems.forEach((item, index) => {
                    const checkbox = document.createElement('input');
                    checkbox.type = 'checkbox';
                    checkbox.id = `check-${index}`;
                    checkbox.className = 'focusable';
                    checkbox.setAttribute('aria-label', `Отметить: ${item.textContent}`);

                    checkbox.addEventListener('change', (e) => {
                        if (e.target.checked) {
                            item.style.textDecoration = 'line-through';
                            item.style.opacity = '0.7';
                        } else {
                            item.style.textDecoration = 'none';
                            item.style.opacity = '1';
                        }
                    });

                    item.insertBefore(checkbox, item.firstChild);
                });
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new DevelopmentAccessibilityChecker();
        });
    </script>
</body>
</html>
```

### 2. Современные паттерны и компоненты

#### Доступная карточка с расширяемым содержимым

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная карточка с расширяемым содержимым</title>
    <style>
        .expandable-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            margin: 20px 0;
            overflow: hidden;
        }

        .card-header {
            padding: 15px;
            background-color: #f8f9fa;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .card-header:focus {
            outline: 2px solid #007acc;
            outline-offset: -2px;
        }

        .card-toggle-icon {
            transition: transform 0.3s ease;
        }

        .card-content {
            padding: 15px;
            display: none;
        }

        .card-content.expanded {
            display: block;
        }

        .card-toggle-icon.expanded {
            transform: rotate(180deg);
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
</head>
<body>
    <main>
        <h1>Доступные интерактивные компоненты</h1>

        <div class="expandable-card">
            <div class="card-header"
                 role="button"
                 tabindex="0"
                 aria-expanded="false"
                 aria-controls="card1-content"
                 onclick="toggleCard(this)"
                 onkeydown="handleCardKeydown(event, this)">
                <h3>Заголовок карточки</h3>
                <span class="card-toggle-icon" aria-hidden="true">▼</span>
            </div>
            <div id="card1-content" class="card-content" role="region" aria-labelledby="card-header-1">
                <p>Содержимое карточки, которое можно расширить или свернуть.</p>
                <p>Это дополнительная информация, которая по умолчанию скрыта.</p>
                <button>Действие в карточке</button>
            </div>
        </div>

        <div class="expandable-card">
            <div class="card-header"
                 role="button"
                 tabindex="0"
                 aria-expanded="false"
                 aria-controls="card2-content"
                 onclick="toggleCard(this)"
                 onkeydown="handleCardKeydown(event, this)">
                <h3>Еще одна карточка</h3>
                <span class="card-toggle-icon" aria-hidden="true">▼</span>
            </div>
            <div id="card2-content" class="card-content" role="region" aria-labelledby="card-header-2">
                <p>Содержимое второй карточки.</p>
                <ul>
                    <li>Элемент списка 1</li>
                    <li>Элемент списка 2</li>
                    <li>Элемент списка 3</li>
                </ul>
            </div>
        </div>
    </main>

    <script>
        function toggleCard(header) {
            const isExpanded = header.getAttribute('aria-expanded') === 'true';
            const content = document.getElementById(header.getAttribute('aria-controls'));
            const icon = header.querySelector('.card-toggle-icon');

            header.setAttribute('aria-expanded', !isExpanded);
            content.classList.toggle('expanded', !isExpanded);
            icon.classList.toggle('expanded', !isExpanded);
        }

        function handleCardKeydown(event, header) {
            if (event.key === 'Enter' || event.key === ' ') {
                event.preventDefault();
                toggleCard(header);
            }
        }
    </script>
</body>
</html>
```

### 3. Современные практики тестирования доступности

#### Интеграция автоматизированного тестирования

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интеграция автоматизированного тестирования</title>
    <style>
        .test-results {
            margin: 20px 0;
            padding: 15px;
            border-radius: 8px;
        }

        .test-results.success {
            background-color: #d4edda;
            border: 1px solid #c3e6cb;
            color: #155724;
        }

        .test-results.warning {
            background-color: #fff3cd;
            border: 1px solid #ffeaa7;
            color: #856404;
        }

        .test-results.error {
            background-color: #f8d7da;
            border: 1px solid #f5c6cb;
            color: #721c24;
        }

        .test-summary {
            margin: 10px 0;
        }

        .test-details {
            margin: 10px 0;
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 4px;
            display: none;
        }

        .test-details.show {
            display: block;
        }

        button {
            padding: 8px 16px;
            margin: 5px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }

        button:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }

        .test-item {
            margin: 10px 0;
            padding: 10px;
            border: 1px solid #eee;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <main>
        <h1>Автоматизированное тестирование доступности</h1>

        <div class="test-controls">
            <button onclick="runAccessibilityTests()">Запустить тесты</button>
            <button onclick="showTestDetails()">Показать детали</button>
            <button onclick="hideTestDetails()">Скрыть детали</button>
        </div>

        <div id="test-results" class="test-results" role="status" aria-live="polite">
            <p>Тесты не запущены. Нажмите "Запустить тесты" для начала проверки.</p>
        </div>

        <div id="test-details" class="test-details" aria-labelledby="details-title">
            <h3 id="details-title">Детали тестирования</h3>
            <div id="test-items"></div>
        </div>
    </main>

    <script>
        class AccessibilityTester {
            constructor() {
                this.resultsContainer = document.getElementById('test-results');
                this.detailsContainer = document.getElementById('test-details');
                this.itemsContainer = document.getElementById('test-items');
            }

            async runTests() {
                this.resultsContainer.innerHTML = '<p>Запуск тестов доступности...</p>';
                this.resultsContainer.className = 'test-results';

                // Имитация тестирования
                await new Promise(resolve => setTimeout(resolve, 2000));

                // Результаты тестирования
                const results = [
                    { type: 'success', message: 'Структура документа корректна', details: 'Все заголовки следуют правильной иерархии' },
                    { type: 'success', message: 'Контрастность текста достаточная', details: 'Все текстовые элементы соответствуют требованиям WCAG' },
                    { type: 'warning', message: 'Некоторые изображения без альтернативного текста', details: 'Найдено 3 изображения без атрибута alt' },
                    { type: 'error', message: 'Отсутствует пропуск навигации', details: 'Не найдена ссылка для пропуска основной навигации' },
                    { type: 'success', message: 'Формы правильно размечены', details: 'Все поля форм имеют соответствующие метки' }
                ];

                this.displayResults(results);
            }

            displayResults(results) {
                const passed = results.filter(r => r.type === 'success').length;
                const total = results.length;
                const hasErrors = results.some(r => r.type === 'error');
                const hasWarnings = results.some(r => r.type === 'warning');

                let resultClass = 'success';
                if (hasErrors) resultClass = 'error';
                else if (hasWarnings) resultClass = 'warning';

                this.resultsContainer.className = `test-results ${resultClass}`;
                this.resultsContainer.innerHTML = `
                    <h3>Результаты тестирования</h3>
                    <div class="test-summary">
                        <p>Пройдено: ${passed} из ${total} тестов</p>
                        <p>Статус: ${hasErrors ? 'Требуются исправления' : hasWarnings ? 'Есть замечания' : 'Все хорошо'}</p>
                    </div>
                `;

                this.displayTestItems(results);
            }

            displayTestItems(results) {
                this.itemsContainer.innerHTML = '';
                
                results.forEach((result, index) => {
                    const item = document.createElement('div');
                    item.className = `test-item test-item-${result.type}`;
                    item.innerHTML = `
                        <h4>${result.message}</h4>
                        <p>${result.details}</p>
                        <button onclick="toggleTestDetails(${index})">Подробнее</button>
                        <div id="detail-${index}" class="test-detail" style="display: none;">
                            <p>Дополнительная информация о проблеме и рекомендации по исправлению...</p>
                        </div>
                    `;
                    
                    this.itemsContainer.appendChild(item);
                });
            }
        }

        // Глобальный экземпляр
        const accessibilityTester = new AccessibilityTester();

        function runAccessibilityTests() {
            accessibilityTester.runTests();
        }

        function showTestDetails() {
            document.getElementById('test-details').classList.add('show');
        }

        function hideTestDetails() {
            document.getElementById('test-details').classList.remove('show');
        }

        function toggleTestDetails(index) {
            const detail = document.getElementById(`detail-${index}`);
            detail.style.display = detail.style.display === 'none' ? 'block' : 'none';
        }
    </script>
</body>
</html>
```

## Следующие темы
- [[HTML/Безопасность]]
- [[HTML/Производительность]]
- [[HTML/Совместимость]]
- [[HTML/Доступность/Стандарты WCAG]]
- [[HTML/Доступность/Тестирование]]
- [[HTML/Доступность/ARIA]]
- [[HTML/Доступность/Промышленные примеры]]

## Теги
#html #accessibility #a11y #web-development #frontend #wcag #aria #screen-readers #keyboard-navigation #focus-management #semantic-markup #inclusive-design #universal-design #best-practices #accessibility-testing #accessibility-insights #axe-core #wave #lighthouse #user-experience #cognitive-accessibility #motor-accessibility #visual-accessibility #auditory-accessibility #html5 #web-standards #web-components #javascript #dom-manipulation #input-sanitization #content-security-policy #subresource-integrity #modern-web #accessibility-api #web-api-integration #accessibility-patterns #expandable-cards #development-process #ci-cd-integration #automated-testing #accessibility-checker #eslint-plugin-jsx-a11y #pa11y #accessibility-audit #inclusive-ux #accessibility-monitoring #user-feedback #accessibility-training #development-workflow #accessibility-guidelines #accessibility-compliance #accessibility-metrics #accessibility-reporting #accessibility-automation #accessibility-devtools #accessibility-inspection #accessibility-debugging #accessibility-performance #accessibility-usability #accessibility-design #accessibility-research