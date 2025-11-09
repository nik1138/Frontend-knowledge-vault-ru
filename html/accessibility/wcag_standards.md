# Стандарты WCAG: Руководство по доступности веб-контента

WCAG (Web Content Accessibility Guidelines) - это международные рекомендации по созданию доступного веб-контента. Эти стандарты разработаны для обеспечения равного доступа к информации и функциям веб-сайтов для всех пользователей, включая людей с ограниченными возможностями.

## История и версии WCAG

### WCAG 1.0 (1999)
- Первая версия рекомендаций
- 14 руководящих принципов
- Уровни приоритета: 1, 2, 3

### WCAG 2.0 (2008)
- Основана на 4 основных принципах (POUR)
- 12 руководящих принципов
- Уровни соответствия: A, AA, AAA
- Наиболее широко используемая версия

### WCAG 2.1 (2018)
- Дополнение к WCAG 2.0
- Учет мобильных устройств и пользователей с когнитивными ограничениями
- 17 новых критериев успеха

### WCAG 2.2 (2023)
- Дополнение к WCAG 2.0 и 2.1
- Учет пользователей с моторными ограничениями
- 9 новых критериев успеха

### WCAG 3.0 (в разработке)
- Радикально обновленный подход
- Ориентация на результаты, а не на процессы
- Непрерывная шкала соответствия

## Принципы WCAG (POUR)

### 1. Perceivable (Воспринимаемость)
Информация должна быть представлена способами, которые пользователи могут воспринимать.

**Ключевые аспекты:**
- Альтернативный текст для изображений
- Субтитры для аудио/видео
- Достаточный контраст текста
- Возможность увеличения текста

**Примеры:**
```html
<!-- Альтернативный текст -->
<img src="chart.png" 
     alt="График роста продаж за последние 5 лет, показывающий увеличение на 15% в год">

<!-- Субтитры для видео -->
<video controls>
    <source src="video.mp4" type="video/mp4">
    <track kind="subtitles" src="subtitles.vtt" srclang="ru" label="Русские">
</video>

<!-- Достаточный контраст -->
<style>
.text {
    color: #000000; /* Черный */
    background-color: #ffffff; /* Белый */
    /* Соотношение контраста 21:1 */
}
</style>
```

### 2. Operable (Управляемость)
Интерфейс должен быть удобен в использовании.

**Ключевые аспекты:**
- Клавиатурная навигация
- Достаточное время для выполнения задач
- Предотвращение спонтанных вспышек
- Легкая навигация

**Примеры:**
```html
<!-- Клавиатурная навигация -->
<nav role="navigation">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>

<!-- Пропуск навигации -->
<a href="#main-content" class="skip-link">Перейти к основному содержимому</a>

<main id="main-content">
    <h1>Основное содержимое</h1>
</main>

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
</style>
```

### 3. Understandable (Понятность)
Информация и работа интерфейса должны быть понятны.

**Ключевые аспекты:**
- Читаемость текста
- Предсказуемое поведение
- Вспомогательная информация
- Понятные ошибки

**Примеры:**
```html
<!-- Понятные метки для форм -->
<form>
    <div class="form-group">
        <label for="email">Email для получения уведомлений:</label>
        <input type="email" id="email" name="email" required>
        <div class="help-text">Введите действительный email адрес</div>
    </div>
    
    <div class="form-group">
        <label for="password">Пароль (минимум 8 символов):</label>
        <input type="password" id="password" name="password" required minlength="8">
    </div>
</form>

<!-- Понятные заголовки -->
<h1>Регистрация нового пользователя</h1>
<h2>Личная информация</h2>
<h3>Контактные данные</h3>
```

### 4. Robust (Надежность)
Контент должен быть надежным, чтобы различные вспомогательные технологии могли его использовать.

**Ключевые аспекты:**
- Правильный HTML синтаксис
- Совместимость с вспомогательными технологиями
- Устойчивость к изменениям

**Примеры:**
```html
<!-- Правильный HTML синтаксис -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Пример страницы</title>
</head>
<body>
    <main>
        <h1>Заголовок страницы</h1>
        <p>Содержимое страницы</p>
    </main>
</body>
</html>

<!-- Совместимость с ARIA -->
<button role="button" aria-pressed="false" onclick="toggle()">
    Переключатель
</button>
```

## Уровни соответствия WCAG

### Уровень A (Минимальный)
- Базовый уровень доступности
- Игнорирование этих требований создает барьеры для доступности
- Примеры: альтернативный текст для изображений, заголовки структуры

### Уровень AA (Средний)
- Средний уровень доступности
- Наиболее часто используемый уровень соответствия
- Примеры: контрастность текста 4.5:1, клавиатурная навигация

### Уровень AAA (Высокий)
- Высокий уровень доступности
- Требуется для специфических ситуаций
- Примеры: контрастность текста 7:1, субтитры для живого аудио

## Критерии успеха WCAG 2.0

### 1.1 - Текстовая альтернатива
Предоставление текстовых альтернатив для всех непечатных элементов.

```html
<!-- Хорошо -->
<img src="logo.png" alt="Логотип компании Acme Inc">
<img src="decoration.png" alt="">

<!-- Плохо -->
<img src="logo.png">
<img src="logo.png" alt="изображение">
```

### 1.2 - Временные альтернативы для аудио и видео
Предоставление альтернатив для временных материалов.

```html
<!-- Видео с субтитрами -->
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <track kind="subtitles" src="subtitles.vtt" srclang="ru" label="Русские">
    <track kind="descriptions" src="descriptions.vtt" srclang="ru" label="Описания">
    Ваш браузер не поддерживает видео элемент.
</video>

<!-- Аудио с транскрипцией -->
<audio controls>
    <source src="podcast.mp3" type="audio/mpeg">
    <p>Транскрипция доступна ниже.</p>
</audio>
```

### 1.3 - Адаптируемость
Создание контента, который можно представить в разных формах.

```html
<!-- Семантическая разметка -->
<nav role="navigation" aria-label="Основная навигация">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</nav>

<!-- Правильная иерархия заголовков -->
<main>
    <h1>Основной заголовок</h1>
    <section>
        <h2>Раздел 1</h2>
        <h3>Подраздел 1.1</h3>
    </section>
</main>
```

### 1.4 - Отличимость
Пользователи должны иметь возможность отличать передний план от фона.

```css
/* Достаточный контраст текста */
.high-contrast-text {
    color: #000000;
    background-color: #ffffff;
    /* Соотношение: 21:1 */
}

.normal-contrast-text {
    color: #333333;
    background-color: #ffffff;
    /* Соотношение: 12.6:1 */
}

.low-contrast-text {
    color: #767676;
    background-color: #ffffff;
    /* Соотношение: 4.58:1 - минимальное для нормального текста */
}

/* Стили фокуса */
a:focus,
button:focus,
input:focus {
    outline: 2px solid #005a9e;
    outline-offset: 2px;
}
```

### 2.1 - Клавиатурная доступность
Все функции должны быть доступны через клавиатуру.

```html
<!-- Клавиатурная навигация -->
<div role="button" 
     tabindex="0" 
     onclick="handleClick()" 
     onkeydown="handleKeyDown(event)">
    Кнопка-имитация
</div>

<script>
function handleKeyDown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        handleClick();
    }
}
</script>
```

### 2.2 - Достаточное время
Предоставление пользователям достаточного времени для чтения и использования контента.

```html
<!-- Таймер с возможностью продления -->
<div id="timer" aria-live="polite">
    Осталось времени: <span id="time">30</span> секунд
</div>

<button onclick="extendTime()">Продлить время</button>

<script>
let timeLeft = 30;
const timerInterval = setInterval(() => {
    timeLeft--;
    document.getElementById('time').textContent = timeLeft;
    
    if (timeLeft <= 0) {
        clearInterval(timerInterval);
        submitForm();
    }
}, 1000);

function extendTime() {
    timeLeft = 30;
    document.getElementById('time').textContent = timeLeft;
}
</script>
```

### 2.3 - Спонтанные вспышки
Предотвращение содержимого, которое вспыхивает более 3 раз в секунду.

```html
<!-- Анимации с учетом предпочтений пользователя -->
<style>
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

.animate {
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
}
</style>
```

### 2.4 - Четкая навигация
Предоставление способов помощи пользователям в навигации, поиске содержимого и определении местоположения.

```html
<!-- Карта сайта -->
<nav role="navigation" aria-label="Карта сайта">
    <ul>
        <li><a href="/">Главная</a></li>
        <li>
            <a href="/products">Продукты</a>
            <ul>
                <li><a href="/products/electronics">Электроника</a></li>
                <li><a href="/products/clothing">Одежда</a></li>
            </ul>
        </li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>

<!-- Хлебные крошки -->
<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Главная</a></li>
        <li><a href="/products">Продукты</a></li>
        <li aria-current="page">Смартфоны</li>
    </ol>
</nav>
```

### 3.1 - Читаемость
Создание содержимого, которое можно читать и понимать.

```html
<!-- Язык страницы -->
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Пример страницы</title>
</head>
<body>
    <!-- Язык для отдельных фрагментов -->
    <p>В английском языке слово <span lang="en">"accessibility"</span> означает доступность.</p>
</body>
</html>
```

### 3.2 - Предсказуемость
Создание веб-страниц, которые появляются и ведут себя предсказуемо.

```html
<!-- Предсказуемое поведение форм -->
<form>
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <label for="password">Пароль:</label>
    <input type="password" id="password" name="password" required>
    
    <button type="submit">Войти</button>
</form>
```

### 3.3 - Вспомогательная информация
Помощь пользователям избежать и исправить ошибки.

```html
<!-- Форма с валидацией -->
<form id="registration-form">
    <div class="form-group">
        <label for="email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
        <input type="email" 
               id="email" 
               name="email" 
               required 
               aria-describedby="email-error"
               aria-invalid="false">
        <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <div class="form-group">
        <label for="password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
        <input type="password" 
               id="password" 
               name="password" 
               required 
               minlength="8"
               aria-describedby="password-requirements password-error"
               aria-invalid="false">
        <ul id="password-requirements">
            <li>Минимум 8 символов</li>
            <li>С заглавной буквой</li>
            <li>С цифрой</li>
        </ul>
        <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <button type="submit">Зарегистрироваться</button>
</form>

<script>
document.getElementById('registration-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const email = document.getElementById('email');
    const password = document.getElementById('password');
    
    let isValid = true;
    
    // Проверка email
    if (!email.value.trim()) {
        showError(email, 'Поле email обязательно для заполнения');
        isValid = false;
    } else if (!isValidEmail(email.value)) {
        showError(email, 'Введите действительный email адрес');
        isValid = false;
    } else {
        clearError(email);
    }
    
    // Проверка пароля
    if (!password.value.trim()) {
        showError(password, 'Поле пароля обязательно для заполнения');
        isValid = false;
    } else if (password.value.length < 8) {
        showError(password, 'Пароль должен содержать не менее 8 символов');
        isValid = false;
    } else {
        clearError(password);
    }
    
    if (isValid) {
        // Отправка формы
        alert('Форма успешно отправлена!');
    }
});

function showError(field, message) {
    field.setAttribute('aria-invalid', 'true');
    document.getElementById(field.id + '-error').textContent = message;
}

function clearError(field) {
    field.setAttribute('aria-invalid', 'false');
    document.getElementById(field.id + '-error').textContent = '';
}

function isValidEmail(email) {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
}
</script>
```

### 4.1 - Совместимость
Максимизация совместимости с текущими и будущими вспомогательными технологиями.

```html
<!-- Правильное использование ARIA -->
<button id="toggle-btn" 
        role="switch" 
        aria-checked="false" 
        onclick="toggleState()">
    <span id="toggle-label">Выключено</span>
</button>

<script>
function toggleState() {
    const button = document.getElementById('toggle-btn');
    const label = document.getElementById('toggle-label');
    const isCurrentlyChecked = button.getAttribute('aria-checked') === 'true';
    
    button.setAttribute('aria-checked', !isCurrentlyChecked);
    label.textContent = !isCurrentlyChecked ? 'Включено' : 'Выключено';
}
</script>
```

## WCAG 2.1 дополнительные критерии

### Учет мобильных устройств
- Управление сенсорными жестами
- Отключение жестов, которые могут вызвать спонтанное поведение
- Определение цели касания

### Учет пользователей с когнитивными ограничениями
- Единообразие в навигации
- Единообразие в идентификации
- Уведомления об изменениях статуса

## WCAG 2.2 дополнительные критерии

### Учет пользователей с моторными ограничениями
- Расширенные цели касания
- Помощь при вводе
- Неожиданные изменения при вводе

## Практические рекомендации по внедрению WCAG

### 1. Анализ текущего состояния
- Проведение аудита доступности
- Идентификация проблемных областей
- Оценка приоритетов исправлений

### 2. Планирование улучшений
- Установка целевого уровня соответствия
- Создание дорожной карты улучшений
- Распределение ответственности

### 3. Внедрение изменений
- Обучение команды
- Внедрение процессов проверки
- Регулярное тестирование

### 4. Мониторинг и поддержка
- Регулярные проверки
- Обратная связь от пользователей
- Обновление в соответствии с новыми версиями

## Инструменты для проверки соответствия WCAG

### Автоматизированные инструменты
1. **axe-core** - библиотека для автоматизированной проверки
2. **WAVE** - веб-инструмент для оценки доступности
3. **Lighthouse** - встроенная проверка в Chrome DevTools
4. **Pa11y** - инструмент командной строки
5. **Accessibility Insights** - расширение для браузеров

### Ручные проверки
1. Проверка клавиатурной навигации
2. Тестирование со скринридерами
3. Проверка контрастности цветов
4. Проверка альтернативного текста
5. Проверка структуры документа

## Заключение

WCAG представляет собой фундаментальный стандарт для создания доступного веб-контента. Эти рекомендации обеспечивают основу для инклюзивного веб-опыта, позволяя людям с различными ограничениями использовать веб-сайты и приложения эффективно.

Следование стандартам WCAG не только помогает людям с ограниченными возможностями, но и улучшает общий пользовательский опыт, SEO и совместимость с различными устройствами. Внедрение этих стандартов требует системного подхода, но результатом является более доступный и инклюзивный веб.

## Следующие темы
- [[HTML/Доступность/Тестирование]]
- [[HTML/Доступность/ARIA]]
- [[HTML/Доступность/Лучшие практики]]

## Теги
#html #accessibility #wcag #web-content-accessibility-guidelines #inclusive-design #universal-design #a11y #web-development #frontend #accessibility-testing #screen-readers #keyboard-navigation #aria #semantic-markup #web-standards #accessibility-guidelines #web-accessibility #user-experience #cognitive-accessibility #motor-accessibility #visual-accessibility #auditory-accessibility #wcag2.0 #wcag2.1 #wcag2.2 #wcag3.0 #accessibility-compliance #accessibility-audit #accessibility-tools