# HTML Элементы: Интерактивные элементы

Интерактивные элементы в HTML позволяют пользователям взаимодействовать с веб-страницей. Они включают в себя кнопки, ссылки, элементы форм и другие элементы, которые реагируют на действия пользователя.

## Основные интерактивные элементы

### `<button>` - Кнопка

Элемент `<button>` создает интерактивную кнопку, которую пользователь может нажать для выполнения действия.

```html
<!-- Кнопка отправки формы -->
<button type="submit">Отправить</button>

<!-- Кнопка сброса формы -->
<button type="reset">Сбросить</button>

<!-- Простая кнопка с JavaScript обработчиком -->
<button type="button" onclick="alert('Кнопка нажата!')">Нажми меня</button>

<!-- Неактивная кнопка -->
<button disabled>Неактивная кнопка</button>

<!-- Кнопка с иконкой -->
<button type="button">
    <img src="icon.png" alt="" width="16" height="16">
    Кнопка с иконкой
</button>
```

#### Атрибуты кнопки

- `type` - определяет тип кнопки:
  - `submit` - отправка формы
  - `reset` - сброс формы
  - `button` - обычная кнопка
- `disabled` - делает кнопку неактивной
- `form` - связывает кнопку с конкретной формой
- `formaction`, `formmethod`, `formenctype`, `formtarget`, `formnovalidate` - переопределяют атрибуты формы

### `<a>` - Гиперссылка

Элемент `<a>` создает гиперссылки на другие страницы или ресурсы.

```html
<!-- Внешняя ссылка -->
<a href="https://example.com">Ссылка на внешний сайт</a>

<!-- Внутренняя ссылка (якорь) -->
<a href="#section1">Перейти к разделу</a>

<!-- Email ссылка -->
<a href="mailto:someone@example.com">Отправить email</a>

<!-- Телефонная ссылка -->
<a href="tel:+71234567890">Позвонить</a>

<!-- Ссылка на скачивание файла -->
<a href="document.pdf" download>Скачать документ</a>

<!-- Ссылка с атрибутами безопасности -->
<a href="https://externalsite.com" target="_blank" rel="noopener noreferrer">Внешняя ссылка</a>
```

#### Атрибуты ссылки

- `href` - URL ресурса
- `target` - указывает, где открыть ссылку
- `rel` - отношения между текущей страницей и ссылкой
- `download` - указывает, что ссылка предназначена для скачивания
- `hreflang` - язык связанного ресурса
- `type` - MIME тип связанного ресурса

### `<input>` - Поле ввода

Элемент `<input>` создает поле ввода различных типов для сбора данных от пользователя.

```html
<!-- Текстовое поле -->
<input type="text" name="username" placeholder="Введите имя пользователя">

<!-- Поле email -->
<input type="email" name="email" placeholder="email@example.com" required>

<!-- Поле пароля -->
<input type="password" name="password" placeholder="Введите пароль">

<!-- Чекбокс -->
<input type="checkbox" id="agree" name="agreement" required>
<label for="agree">Согласен с условиями</label>

<!-- Радиокнопка -->
<input type="radio" id="male" name="gender" value="male">
<label for="male">Мужчина</label>
<input type="radio" id="female" name="gender" value="female">
<label for="female">Женщина</label>

<!-- Поле числа -->
<input type="number" name="age" min="1" max="120" value="25">

<!-- Поле даты -->
<input type="date" name="birthdate">

<!-- Поле поиска -->
<input type="search" name="query" placeholder="Поиск...">

<!-- Скрытое поле -->
<input type="hidden" name="token" value="abc123">
```

#### Атрибуты полей ввода

- `type` - тип поля ввода
- `name` - имя поля для отправки формы
- `value` - начальное значение
- `placeholder` - подсказка в поле
- `required` - поле обязательно для заполнения
- `disabled` - поле неактивно
- `readonly` - поле только для чтения
- `min`, `max`, `step` - для числовых полей
- `pattern` - регулярное выражение для проверки

### `<select>` и `<option>` - Выпадающий список

Эти элементы создают выпадающий список для выбора одного или нескольких вариантов.

```html
<!-- Простой выпадающий список -->
<label for="country">Страна:</label>
<select id="country" name="country">
    <option value="">Выберите страну</option>
    <option value="ru">Россия</option>
    <option value="us">США</option>
    <option value="de">Германия</option>
    <option value="fr">Франция</option>
</select>

<!-- Множественный выбор -->
<select name="interests" multiple>
    <option value="tech">Технологии</option>
    <option value="sports">Спорт</option>
    <option value="music">Музыка</option>
    <option value="travel">Путешествия</option>
</select>

<!-- Группировка опций -->
<select name="cars">
    <optgroup label="Европейские">
        <option value="bmw">BMW</option>
        <option value="mercedes">Mercedes</option>
    </optgroup>
    <optgroup label="Японские">
        <option value="toyota">Toyota</option>
        <option value="honda">Honda</option>
    </optgroup>
</select>
```

### `<textarea>` - Многострочное текстовое поле

Элемент `<textarea>` создает многострочное текстовое поле для ввода большого объема текста.

```html
<textarea name="message" rows="5" cols="50" placeholder="Введите ваше сообщение..."></textarea>

<!-- Неизменяемый размер -->
<textarea name="comment" style="resize: none;" placeholder="Комментарий"></textarea>

<!-- Автоматическое изменение размера (с помощью JavaScript) -->
<textarea name="content" id="auto-resize" placeholder="Текст с автоподгонкой высоты"></textarea>
```

### `<label>` - Метка для элемента формы

Элемент `<label>` создает метку для элемента формы, улучшая доступность.

```html
<!-- Метка с привязкой через for -->
<label for="username">Имя пользователя:</label>
<input type="text" id="username" name="username">

<!-- Метка с вложенным элементом -->
<label>
    <input type="checkbox" name="subscribe"> Подписаться на рассылку
</label>

<!-- Метка для радиокнопок -->
<fieldset>
    <legend>Выберите цвет:</legend>
    <label><input type="radio" name="color" value="red"> Красный</label>
    <label><input type="radio" name="color" value="blue"> Синий</label>
    <label><input type="radio" name="color" value="green"> Зеленый</label>
</fieldset>
```

## Дополнительные интерактивные элементы

### `<details>` и `<summary>` - Раскрывающийся раздел

Эти элементы создают интерактивный раздел, который можно раскрыть или свернуть.

```html
<details>
    <summary>Часто задаваемые вопросы</summary>
    <p>Здесь содержится подробная информация, которая отображается при раскрытии.</p>
</details>

<details>
    <summary>Как использовать этот продукт?</summary>
    <ol>
        <li>Шаг 1: Подготовьте материалы</li>
        <li>Шаг 2: Следуйте инструкции</li>
        <li>Шаг 3: Наслаждайтесь результатом</li>
    </ol>
</details>
```

### `<dialog>` - Диалоговое окно

Элемент `<dialog>` создает диалоговое окно или другие интерактивные компоненты.

```html
<dialog id="modal">
    <h3>Подтверждение действия</h3>
    <p>Вы уверены, что хотите удалить этот элемент?</p>
    <button id="confirm">Да</button>
    <button id="cancel">Нет</button>
</dialog>

<button onclick="document.getElementById('modal').showModal()">Открыть диалог</button>

<script>
document.getElementById('confirm').addEventListener('click', () => {
    document.getElementById('modal').close();
    // Выполнить действие подтверждения
});

document.getElementById('cancel').addEventListener('click', () => {
    document.getElementById('modal').close();
});
</script>
```

### `<meter>` - Измерительное значение

Элемент `<meter>` представляет собой скалярное измерение в известном диапазоне.

```html
<!-- Уровень заполнения -->
<label for="disk">Занятое место на диске:</label>
<meter id="disk" value="0.7" min="0" max="1">70%</meter>

<!-- Рейтинг -->
<label for="rating">Рейтинг продукта:</label>
<meter id="rating" value="4.5" min="0" max="5" high="4" low="2">4.5 из 5</meter>
```

### `<progress>` - Индикатор выполнения

Элемент `<progress>` представляет собой индикатор выполнения задачи.

```html
<!-- Индикатор с известным значением -->
<label for="file-upload">Загрузка файла:</label>
<progress id="file-upload" value="70" max="100">70%</progress>

<!-- Индикатор неизвестного прогресса -->
<label for="task">Выполнение задачи:</label>
<progress id="task">Выполняется...</progress>
```

## Атрибуты доступности для интерактивных элементов

### ARIA-атрибуты

```html
<!-- Кнопка с описанием -->
<button aria-label="Закрыть окно">✕</button>

<!-- Кнопка с описанием в другом элементе -->
<button aria-describedby="help-text">Помощь</button>
<div id="help-text">Нажмите для получения справки</div>

<!-- Состояния для чекбоксов -->
<input type="checkbox" id="terms" aria-checked="false">
<label for="terms">Согласен с условиями</label>

<!-- Роль элемента -->
<div role="button" tabindex="0" onclick="doSomething()">Кнопка-имитация</div>
```

### Атрибут `tabindex`

```html
<!-- Элемент в порядке табуляции -->
<input type="text" tabindex="0">

<!-- Элемент в специфическом порядке табуляции -->
<input type="text" tabindex="1">
<button tabindex="2">Кнопка</button>

<!-- Элемент исключенный из табуляции -->
<div tabindex="-1">Не получает фокус при табуляции</div>
```

## Стилизация интерактивных элементов

Интерактивные элементы можно стилизовать с помощью CSS:

```html
<style>
/* Стили для кнопок */
.button {
    background-color: #007acc;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
}

.button:hover {
    background-color: #005a9e;
}

.button:active {
    transform: translateY(1px);
}

.button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}

/* Стили для ссылок */
.link {
    color: #007acc;
    text-decoration: none;
}

.link:hover {
    text-decoration: underline;
}

.link:focus {
    outline: 2px solid #007acc;
    outline-offset: 2px;
}

/* Стили для полей ввода */
.input-field {
    padding: 8px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
}

.input-field:focus {
    outline: 2px solid #007acc;
    border-color: #007acc;
}
</style>

<button class="button">Нажми меня</button>
<a href="#" class="link">Ссылка</a>
<input type="text" class="input-field" placeholder="Введите текст">
```

## Заключение

Интерактивные элементы являются ключевыми компонентами веб-интерфейсов, позволяя пользователям взаимодействовать с содержимым страницы. Правильное использование этих элементов с учетом семантики, доступности и пользовательского опыта создает более удобные и функциональные веб-приложения. Важно также учитывать доступность, добавляя соответствующие ARIA-атрибуты и обеспечивая навигацию с клавиатуры.

## Современные интерактивные элементы HTML5

### Новые элементы интерфейса
```html
<!-- Диалоговое окно -->
<dialog id="modal">
    <h2>Заголовок модального окна</h2>
    <p>Содержимое модального окна</p>
    <button onclick="document.getElementById('modal').close()">Закрыть</button>
</dialog>

<!-- Измерительные элементы -->
<meter value="0.7" min="0" max="1" low="0.3" high="0.8" optimum="0.8">
    70% выполнено
</meter>

<progress value="70" max="100">70%</progress>

<!-- Раскрывающийся раздел -->
<details>
    <summary>Дополнительная информация</summary>
    <p>Подробная информация, которая изначально скрыта</p>
</details>
```

### Интерактивные элементы с улучшенной семантикой
```html
<!-- Взаимодействие с пользователем -->
<menu type="toolbar">
    <button type="button">Вырезать</button>
    <button type="button">Копировать</button>
    <button type="button">Вставить</button>
</menu>

<!-- Выбор даты/времени -->
<input type="date" name="birthdate">
<input type="datetime-local" name="event-time">
<input type="time" name="start-time">
<input type="month" name="month">
<input type="week" name="week">

<!-- Цвет и диапазон -->
<input type="color" name="favorite-color" value="#ff0000">
<input type="range" name="volume" min="0" max="100" value="50" step="1">
```

## Расширенные примеры интерактивных элементов

### Комплексная форма с валидацией
```html
<form id="registration-form" novalidate>
    <div class="form-group">
        <label for="username">Имя пользователя:</label>
        <input type="text" 
               id="username" 
               name="username" 
               required 
               minlength="3" 
               maxlength="20" 
               pattern="[A-Za-z0-9_]+"
               aria-describedby="username-help username-error">
        <small id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-20 символов</small>
        <div id="username-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" 
               id="email" 
               name="email" 
               required 
               aria-describedby="email-error">
        <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="password">Пароль:</label>
        <div class="password-container">
            <input type="password" 
                   id="password" 
                   name="password" 
                   required 
                   minlength="8"
                   aria-describedby="password-requirements">
            <button type="button" 
                    onclick="togglePasswordVisibility('password')" 
                    aria-label="Показать/скрыть пароль">👁️</button>
        </div>
        <div id="password-requirements">
            <small>Пароль должен содержать не менее 8 символов, включая заглавную букву и цифру</small>
        </div>
    </div>

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
                   value="accepted" 
                   required 
                   aria-describedby="terms-description">
            Я согласен с условиями использования
        </label>
        <div id="terms-description">
            <small>Обязательное поле для регистрации</small>
        </div>
    </div>

    <button type="submit">Зарегистрироваться</button>
</form>

<script>
function togglePasswordVisibility(fieldId) {
    const field = document.getElementById(fieldId);
    const isPassword = field.type === 'password';
    field.type = isPassword ? 'text' : 'password';
}

document.getElementById('registration-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // Сброс ошибок
    document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
    
    // Валидация формы
    let isValid = true;
    
    // Проверка имени пользователя
    const username = document.getElementById('username');
    if (!/^[A-Za-z0-9_]{3,20}$/.test(username.value)) {
        document.getElementById('username-error').textContent = 'Имя пользователя должно содержать 3-20 символов, только буквы, цифры и подчеркивание';
        isValid = false;
    }
    
    // Проверка email
    const email = document.getElementById('email');
    if (!email.validity.valid) {
        document.getElementById('email-error').textContent = 'Введите действительный email';
        isValid = false;
    }
    
    if (isValid) {
        alert('Форма успешно отправлена!');
        // Отправка формы на сервер
    }
});
</script>
```

### Интерактивные элементы с JavaScript
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивные элементы</title>
    <style>
        .tab-container {
            display: flex;
            flex-direction: column;
        }
        
        .tab-buttons {
            display: flex;
            list-style: none;
            padding: 0;
            margin: 0;
            border-bottom: 1px solid #ddd;
        }
        
        .tab-button {
            padding: 10px 20px;
            cursor: pointer;
            border: 1px solid transparent;
            border-bottom: none;
            background: #f0f0f0;
            margin-right: 2px;
        }
        
        .tab-button.active {
            background: white;
            border-color: #ddd;
            border-bottom: 1px solid white;
        }
        
        .tab-panel {
            display: none;
            padding: 20px;
            border: 1px solid #ddd;
            border-top: none;
        }
        
        .tab-panel.active {
            display: block;
        }
        
        .accordion-item {
            border: 1px solid #ddd;
            margin-bottom: 5px;
        }
        
        .accordion-header {
            background: #f0f0f0;
            padding: 10px;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .accordion-content {
            padding: 10px;
            display: none;
        }
        
        .accordion-content.open {
            display: block;
        }
    </style>
</head>
<body>
    <h1>Интерактивные компоненты</h1>
    
    <!-- Вкладки -->
    <div class="tab-container">
        <ul class="tab-buttons" role="tablist">
            <li><button class="tab-button" role="tab" aria-selected="true" aria-controls="panel1">Вкладка 1</button></li>
            <li><button class="tab-button" role="tab" aria-selected="false" aria-controls="panel2" tabindex="-1">Вкладка 2</button></li>
            <li><button class="tab-button" role="tab" aria-selected="false" aria-controls="panel3" tabindex="-1">Вкладка 3</button></li>
        </ul>
        
        <div id="panel1" class="tab-panel active" role="tabpanel" aria-labelledby="tab1">
            <h3>Содержимое первой вкладки</h3>
            <p>Содержимое первой вкладки...</p>
        </div>
        
        <div id="panel2" class="tab-panel" role="tabpanel" aria-labelledby="tab2" hidden>
            <h3>Содержимое второй вкладки</h3>
            <p>Содержимое второй вкладки...</p>
        </div>
        
        <div id="panel3" class="tab-panel" role="tabpanel" aria-labelledby="tab3" hidden>
            <h3>Содержимое третьей вкладки</h3>
            <p>Содержимое третьей вкладки...</p>
        </div>
    </div>
    
    <!-- Аккордеон -->
    <div class="accordion">
        <div class="accordion-item">
            <div class="accordion-header" onclick="toggleAccordion(this)">
                <span>Раздел 1</span>
                <span class="toggle-icon">▼</span>
            </div>
            <div class="accordion-content">
                <p>Содержимое первого раздела аккордеона...</p>
            </div>
        </div>
        
        <div class="accordion-item">
            <div class="accordion-header" onclick="toggleAccordion(this)">
                <span>Раздел 2</span>
                <span class="toggle-icon">▼</span>
            </div>
            <div class="accordion-content">
                <p>Содержимое второго раздела аккордеона...</p>
            </div>
        </div>
    </div>
    
    <script>
        // Управление вкладками
        document.querySelectorAll('.tab-button').forEach(button => {
            button.addEventListener('click', () => {
                // Снять выделение с текущей вкладки
                document.querySelectorAll('.tab-button').forEach(btn => {
                    btn.setAttribute('aria-selected', 'false');
                    btn.setAttribute('tabindex', '-1');
                });
                
                document.querySelectorAll('.tab-panel').forEach(panel => {
                    panel.classList.remove('active');
                    panel.setAttribute('hidden', '');
                });
                
                // Выделить выбранную вкладку
                button.setAttribute('aria-selected', 'true');
                button.setAttribute('tabindex', '0');
                button.focus();
                
                const panelId = button.getAttribute('aria-controls');
                const panel = document.getElementById(panelId);
                panel.classList.add('active');
                panel.removeAttribute('hidden');
            });
        });
        
        // Управление аккордеоном
        function toggleAccordion(header) {
            const content = header.nextElementSibling;
            const icon = header.querySelector('.toggle-icon');
            const isOpen = content.classList.contains('open');
            
            // Закрыть все аккордеоны
            document.querySelectorAll('.accordion-content').forEach(item => {
                item.classList.remove('open');
            });
            
            document.querySelectorAll('.toggle-icon').forEach(icon => {
                icon.textContent = '▼';
            });
            
            // Открыть выбранный аккордеон, если он был закрыт
            if (!isOpen) {
                content.classList.add('open');
                icon.textContent = '▲';
            }
        }
    </script>
</body>
</html>
```

## Доступность интерактивных элементов

### Клавиатурная навигация
```html
<!-- Правильная последовательность табуляции -->
<form>
    <input type="text" name="field1" tabindex="1">
    <input type="text" name="field2" tabindex="2">
    <button type="button" tabindex="3">Кнопка</button>
    <input type="text" name="field3" tabindex="4" disabled> <!-- Исключено из табуляции -->
    <button type="submit" tabindex="5">Отправить</button>
</form>

<!-- Пропуск навигации -->
<a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
<a class="skip-link" href="#search">Перейти к поиску</a>

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

### ARIA-атрибуты для интерактивных элементов
```html
<!-- Роли и состояния -->
<button role="button" 
        aria-pressed="false" 
        onclick="toggleState(this)"
        tabindex="0">Переключатель</button>

<!-- Описание элементов -->
<button aria-label="Закрыть окно">×</button>
<button aria-describedby="help-text">Помощь</button>
<div id="help-text">Нажмите для получения справки</div>

<!-- Живые области -->
<div id="status" aria-live="polite" aria-atomic="true">
    <!-- Статусные сообщения будут появляться здесь -->
</div>

<!-- Слайдер с ARIA -->
<div role="slider" 
     tabindex="0"
     aria-valuemin="0" 
     aria-valuemax="100" 
     aria-valuenow="50"
     aria-valuetext="50%"
     onkeydown="handleSliderKeydown(event)"
     onclick="handleSliderClick(event)">
    <div style="width: 50%; height: 20px; background: #007acc;"></div>
</div>
```

### Интерактивные элементы в формах
```html
<form>
    <!-- Интерактивные поля ввода -->
    <div class="form-group">
        <label for="search">Поиск:</label>
        <div class="search-container">
            <input type="search" 
                   id="search" 
                   name="search" 
                   role="combobox"
                   aria-haspopup="listbox"
                   aria-owns="search-results"
                   aria-expanded="false"
                   autocomplete="off">
            <button type="button" aria-label="Очистить поле поиска" onclick="clearSearch()">✕</button>
        </div>
        <ul id="search-results" 
            role="listbox" 
            aria-label="Результаты поиска"
            style="display: none; border: 1px solid #ccc; background: white; margin: 0; padding: 0;">
            <!-- Результаты будут добавляться сюда -->
        </ul>
    </div>

    <!-- Выбор с автодополнением -->
    <div class="form-group">
        <label for="city">Город:</label>
        <input type="text" 
               id="city" 
               name="city" 
               list="cities"
               aria-autocomplete="list"
               aria-controls="city-suggestions">
        <datalist id="cities">
            <option value="Москва">
            <option value="Санкт-Петербург">
            <option value="Новосибирск">
        </datalist>
        <div id="city-suggestions" role="status" aria-live="polite"></div>
    </div>

    <!-- Группировка чекбоксов -->
    <fieldset>
        <legend>Интересы:</legend>
        <div class="checkbox-group">
            <label>
                <input type="checkbox" name="interests" value="tech" checked>
                Технологии
            </label>
            <label>
                <input type="checkbox" name="interests" value="sports">
                Спорт
            </label>
            <label>
                <input type="checkbox" name="interests" value="music">
                Музыка
            </label>
        </div>
    </fieldset>

    <!-- Группировка радиокнопок -->
    <fieldset>
        <legend>Способ доставки:</legend>
        <div class="radio-group">
            <label>
                <input type="radio" name="delivery" value="courier" checked>
                Курьером
            </label>
            <label>
                <input type="radio" name="delivery" value="pickup">
                Самовывоз
            </label>
        </div>
    </fieldset>
</form>
```

## Современные паттерны интерактивных элементов

### Кастомные элементы управления
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Кастомные интерактивные элементы</title>
    <style>
        .switch {
            position: relative;
            display: inline-block;
            width: 60px;
            height: 34px;
        }

        .switch input {
            opacity: 0;
            width: 0;
            height: 0;
        }

        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: #ccc;
            transition: .4s;
            border-radius: 34px;
        }

        .slider:before {
            position: absolute;
            content: "";
            height: 26px;
            width: 26px;
            left: 4px;
            bottom: 4px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }

        input:checked + .slider {
            background-color: #2196F3;
        }

        input:checked + .slider:before {
            transform: translateX(26px);
        }

        .rating {
            display: flex;
            direction: rtl;
        }

        .rating input[type="radio"] {
            display: none;
        }

        .rating label {
            font-size: 2em;
            color: #ddd;
            cursor: pointer;
        }

        .rating label:hover,
        .rating label:hover ~ label,
        .rating input[type="radio"]:checked ~ label {
            color: #ffc107;
        }
    </style>
</head>
<body>
    <h1>Кастомные интерактивные элементы</h1>
    
    <!-- Переключатель -->
    <div class="switch-container">
        <label for="toggle">Темная тема</label>
        <label class="switch">
            <input type="checkbox" id="toggle" onchange="toggleTheme(this.checked)">
            <span class="slider"></span>
        </label>
    </div>
    
    <!-- Система рейтинга -->
    <div class="rating-container">
        <label>Оцените статью:</label>
        <div class="rating" role="radiogroup" aria-label="Оценка статьи">
            <input type="radio" id="star5" name="rating" value="5">
            <label for="star5" aria-label="5 звезд">★</label>
            
            <input type="radio" id="star4" name="rating" value="4">
            <label for="star4" aria-label="4 звезды">★</label>
            
            <input type="radio" id="star3" name="rating" value="3">
            <label for="star3" aria-label="3 звезды">★</label>
            
            <input type="radio" id="star2" name="rating" value="2">
            <label for="star2" aria-label="2 звезды">★</label>
            
            <input type="radio" id="star1" name="rating" value="1">
            <label for="star1" aria-label="1 звезда">★</label>
        </div>
    </div>
    
    <script>
        function toggleTheme(isDark) {
            document.body.style.backgroundColor = isDark ? '#333' : '#fff';
            document.body.style.color = isDark ? '#fff' : '#000';
        }
    </script>
</body>
</html>
```

### Интерактивные медиа элементы
```html
<div class="video-player">
    <video id="myVideo" width="640" height="360" controls>
        <source src="movie.mp4" type="video/mp4">
        <source src="movie.ogg" type="video/ogg">
        Ваш браузер не поддерживает видео элемент.
    </video>
    
    <div class="video-controls">
        <button onclick="playPause()" aria-label="Воспроизвести/Пауза">⏯️</button>
        <button onclick="toggleFullscreen()" aria-label="Полноэкранный режим">⛶</button>
        <input type="range" id="volumeSlider" min="0" max="100" value="50" 
               aria-label="Громкость" onchange="setVolume(this.value)">
        <span id="currentTime">00:00</span> / <span id="duration">00:00</span>
    </div>
</div>

<script>
const video = document.getElementById('myVideo');
const playPauseBtn = document.querySelector('[onclick="playPause()"]');
const currentTimeSpan = document.getElementById('currentTime');
const durationSpan = document.getElementById('duration');

video.addEventListener('timeupdate', function() {
    const current = formatTime(video.currentTime);
    const duration = formatTime(video.duration);
    currentTimeSpan.textContent = current;
    durationSpan.textContent = duration;
});

function playPause() {
    if (video.paused) {
        video.play();
        playPauseBtn.textContent = '⏸️';
    } else {
        video.pause();
        playPauseBtn.textContent = '⏯️';
    }
}

function toggleFullscreen() {
    if (video.requestFullscreen) {
        video.requestFullscreen();
    } else if (video.mozRequestFullScreen) {
        video.mozRequestFullScreen();
    } else if (video.webkitRequestFullscreen) {
        video.webkitRequestFullscreen();
    }
}

function setVolume(volume) {
    video.volume = volume / 100;
}

function formatTime(seconds) {
    const min = Math.floor(seconds / 60);
    const sec = Math.floor(seconds % 60);
    return `${min.toString().padStart(2, '0')}:${sec.toString().padStart(2, '0')}`;
}
</script>
```

## Лучшие практики интерактивных элементов

### 1. Доступность
```html
<!-- Хорошо: доступный интерактивный элемент -->
<button onclick="doSomething()" 
        aria-label="Удалить элемент"
        aria-describedby="delete-help">
    🗑️
</button>
<div id="delete-help" class="sr-only">Нажмите для удаления текущего элемента</div>

<!-- Плохо: недоступный элемент -->
<span onclick="doSomething()" style="cursor: pointer;">🗑️</span>
```

### 2. Клавиатурная навигация
```html
<!-- Обеспечение фокуса для кастомных элементов -->
<div class="custom-button" 
     tabindex="0" 
     onclick="handleClick(event)" 
     onkeydown="handleKeydown(event)"
     role="button" 
     aria-pressed="false">
    Кастомная кнопка
</div>

<script>
function handleClick(event) {
    // Обработка клика
    event.target.setAttribute('aria-pressed', 'true');
}

function handleKeydown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        handleClick(event);
    }
}
</script>
```

### 3. Семантическая правильность
```html
<!-- Правильно: использование семантически правильных элементов -->
<nav aria-label="Основная навигация">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</nav>

<button type="submit">Отправить</button>
<details>
    <summary>Дополнительная информация</summary>
    <p>Содержимое</p>
</details>

<!-- Неправильно: использование неподходящих элементов -->
<div role="navigation">
    <div><span>Главная</span></div>
    <div><span>О нас</span></div>
</div>

<div onclick="submitForm()" role="button">Отправить</div>
<div onclick="toggleInfo()" role="button">Дополнительная информация</div>
```

## Пример комплексной интерактивной страницы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интерактивные элементы в действии</title>
    <meta name="description" content="Пример использования различных интерактивных элементов HTML">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Интерактивные элементы в действии">
    <meta property="og:description" content="Пример использования различных интерактивных элементов HTML">
    <meta property="og:type" content="website">
</head>
<body>
    <!-- Ссылки для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <header role="banner">
        <h1>Интерактивные элементы HTML</h1>
        
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/" aria-current="page">Главная</a></li>
                <li><a href="/examples">Примеры</a></li>
                <li><a href="/documentation">Документация</a></li>
            </ul>
        </nav>
        
        <div role="search">
            <form action="/search" method="get">
                <label for="search-input">Поиск:</label>
                <input type="search" id="search-input" name="q" placeholder="Поиск...">
                <button type="submit">Найти</button>
            </form>
        </div>
    </header>

    <main id="main-content" role="main">
        <section>
            <h2>Интерактивные формы</h2>
            
            <form id="contact-form">
                <div class="form-group">
                    <label for="name">Имя:</label>
                    <input type="text" id="name" name="name" required>
                </div>
                
                <div class="form-group">
                    <label for="email">Email:</label>
                    <input type="email" id="email" name="email" required>
                </div>
                
                <div class="form-group">
                    <label for="subject">Тема:</label>
                    <select id="subject" name="subject" required>
                        <option value="">Выберите тему</option>
                        <option value="general">Общий вопрос</option>
                        <option value="support">Техническая поддержка</option>
                        <option value="feedback">Обратная связь</option>
                    </select>
                </div>
                
                <div class="form-group">
                    <label for="message">Сообщение:</label>
                    <textarea id="message" name="message" rows="5" required></textarea>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" name="newsletter" value="yes">
                        Подписаться на рассылку
                    </label>
                </div>
                
                <button type="submit">Отправить сообщение</button>
            </form>
        </section>

        <section>
            <h2>Интерактивные компоненты</h2>
            
            <!-- Аккордеон -->
            <details>
                <summary>Часто задаваемые вопросы</summary>
                <div class="faq-content">
                    <details>
                        <summary>Как использовать этот сайт?</summary>
                        <p>Для использования сайта просто наведите курсор на интересующий вас раздел...</p>
                    </details>
                    <details>
                        <summary>Как связаться с поддержкой?</summary>
                        <p>Вы можете связаться с нашей поддержкой через форму обратной связи...</p>
                    </details>
                </div>
            </details>
            
            <!-- Прогресс бар -->
            <div class="progress-container">
                <label for="progress">Прогресс выполнения:</label>
                <progress id="progress" value="75" max="100">75%</progress>
                <span aria-live="polite" aria-atomic="true" id="progress-text">75% выполнено</span>
            </div>
            
            <!-- Измеритель -->
            <div class="meter-container">
                <label for="meter">Уровень удовлетворенности:</label>
                <meter id="meter" value="0.8" min="0" max="1" low="0.3" high="0.7" optimum="0.9">
                    80%
                </meter>
            </div>
        </section>

        <section>
            <h2>Интерактивные списки</h2>
            
            <div class="filterable-list">
                <div class="filter-controls">
                    <input type="text" id="filter-input" placeholder="Фильтр...">
                    <select id="category-filter">
                        <option value="">Все категории</option>
                        <option value="html">HTML</option>
                        <option value="css">CSS</option>
                        <option value="javascript">JavaScript</option>
                    </select>
                </div>
                
                <ul id="items-list" role="listbox" aria-label="Список элементов">
                    <li role="option" data-category="html">HTML формы</li>
                    <li role="option" data-category="css">CSS стили</li>
                    <li role="option" data-category="javascript">JavaScript события</li>
                    <li role="option" data-category="html">HTML элементы</li>
                </ul>
            </div>
        </section>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Название компании. Все права защищены.</p>
        
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
                <li><a href="/accessibility">Доступность</a></li>
            </ul>
        </nav>
    </footer>

    <script>
        // Обработка формы
        document.getElementById('contact-form').addEventListener('submit', function(e) {
            e.preventDefault();
            alert('Форма отправлена!');
        });

        // Фильтрация списка
        document.getElementById('filter-input').addEventListener('input', function() {
            const filter = this.value.toLowerCase();
            const items = document.querySelectorAll('#items-list li');
            
            items.forEach(item => {
                const text = item.textContent.toLowerCase();
                item.style.display = text.includes(filter) ? 'list-item' : 'none';
            });
        });

        // Обновление прогресса
        const progress = document.getElementById('progress');
        const progressText = document.getElementById('progress-text');
        
        progress.addEventListener('change', function() {
            const percent = this.value;
            progressText.textContent = `${percent}% выполнено`;
        });
    </script>
</body>
</html>
```

## Современные возможности интерактивных элементов

### Веб-компоненты и интерактивные элементы
```html
<script>
// Кастомный элемент кнопки
class CustomButton extends HTMLElement {
    constructor() {
        super();
        
        // Создание теневого DOM
        const shadow = this.attachShadow({ mode: 'open' });
        
        // Стили
        const style = document.createElement('style');
        style.textContent = `
            .custom-btn {
                background-color: #007acc;
                color: white;
                border: none;
                padding: 10px 20px;
                border-radius: 4px;
                cursor: pointer;
                font-size: 16px;
                transition: background-color 0.3s;
            }
            
            .custom-btn:hover {
                background-color: #005a9e;
            }
            
            .custom-btn:disabled {
                background-color: #cccccc;
                cursor: not-allowed;
            }
        `;
        
        // HTML
        const button = document.createElement('button');
        button.className = 'custom-btn';
        button.textContent = this.getAttribute('text') || 'Кнопка';
        button.disabled = this.hasAttribute('disabled');
        
        // Добавление обработчика событий
        button.addEventListener('click', () => {
            this.dispatchEvent(new CustomEvent('custom-click', {
                bubbles: true,
                composed: true
            }));
        });
        
        shadow.appendChild(style);
        shadow.appendChild(button);
    }
}

// Регистрация кастомного элемента
customElements.define('custom-button', CustomButton);
</script>

<!-- Использование кастомного элемента -->
<custom-button text="Нажми меня" id="my-custom-btn"></custom-button>

<script>
document.getElementById('my-custom-btn').addEventListener('custom-click', () => {
    alert('Кастомная кнопка нажата!');
});
</script>
```

### Интерактивные элементы с асинхронными операциями
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивные элементы с асинхронными операциями</title>
    <style>
        .async-button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            position: relative;
        }
        
        .async-button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        .spinner {
            display: inline-block;
            width: 16px;
            height: 16px;
            border: 2px solid #ffffff;
            border-radius: 50%;
            border-top-color: transparent;
            animation: spin 1s linear infinite;
            margin-right: 8px;
            vertical-align: middle;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <button id="async-button" class="async-button" onclick="performAsyncAction()">
        Выполнить действие
    </button>
    
    <div id="result"></div>

    <script>
        async function performAsyncAction() {
            const button = document.getElementById('async-button');
            const result = document.getElementById('result');
            
            // Показать индикатор загрузки
            button.disabled = true;
            button.innerHTML = '<span class="spinner"></span> Выполняется...';
            
            try {
                // Симуляция асинхронной операции
                const response = await fetch('/api/data');
                const data = await response.json();
                
                result.innerHTML = `<p>Данные получены: ${JSON.stringify(data)}</p>`;
            } catch (error) {
                result.innerHTML = `<p style="color: red;">Ошибка: ${error.message}</p>`;
            } finally {
                // Восстановить кнопку
                button.disabled = false;
                button.textContent = 'Выполнить действие';
            }
        }
        
        // Функция-заглушка для демонстрации
        window.fetch = function(url) {
            return new Promise((resolve) => {
                setTimeout(() => {
                    resolve({
                        json: () => Promise.resolve({ message: "Данные успешно получены", timestamp: new Date().toISOString() })
                    });
                }, 2000);
            });
        };
    </script>
</body>
</html>
```

### Интерактивные элементы с Drag and Drop
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Drag and Drop интерактивные элементы</title>
    <style>
        .draggable {
            padding: 10px;
            margin: 5px;
            background-color: #f0f0f0;
            border: 1px solid #ccc;
            cursor: move;
        }
        
        .draggable:hover {
            background-color: #e0e0e0;
        }
        
        .drop-zone {
            width: 200px;
            height: 200px;
            border: 2px dashed #ccc;
            margin: 10px;
            padding: 10px;
            background-color: #fafafa;
        }
        
        .drop-zone.drag-over {
            border-color: #007acc;
            background-color: #e6f0ff;
        }
    </style>
</head>
<body>
    <h2>Перетаскиваемые элементы</h2>
    
    <div class="drop-zone" id="source" ondrop="drop(event)" ondragover="allowDrop(event)">
        <h3>Источник</h3>
        <div class="draggable" draggable="true" ondragstart="drag(event)" id="drag1">Элемент 1</div>
        <div class="draggable" draggable="true" ondragstart="drag(event)" id="drag2">Элемент 2</div>
        <div class="draggable" draggable="true" ondragstart="drag(event)" id="drag3">Элемент 3</div>
    </div>
    
    <div class="drop-zone" id="target" ondrop="drop(event)" ondragover="allowDrop(event)">
        <h3>Цель</h3>
        <p>Перетащите сюда элементы</p>
    </div>

    <script>
        function allowDrop(ev) {
            ev.preventDefault();
            ev.currentTarget.classList.add('drag-over');
        }

        function drag(ev) {
            ev.dataTransfer.setData("text", ev.target.id);
        }

        function drop(ev) {
            ev.preventDefault();
            ev.currentTarget.classList.remove('drag-over');
            
            const data = ev.dataTransfer.getData("text");
            ev.currentTarget.appendChild(document.getElementById(data));
        }
        
        // Удалить класс при завершении перетаскивания
        document.addEventListener('dragend', function() {
            document.querySelectorAll('.drop-zone').forEach(zone => {
                zone.classList.remove('drag-over');
            });
        });
    </script>
</body>
</html>
```

### Интерактивные элементы с WebSockets
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>WebSocket интерактивные элементы</title>
    <style>
        .status {
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
        }
        
        .status.connected {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .status.disconnected {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .message {
            padding: 5px;
            margin: 5px 0;
            border-left: 3px solid #007acc;
        }
        
        .message.received {
            background-color: #e6f7ff;
        }
        
        .message.sent {
            background-color: #f0f0f0;
        }
    </style>
</head>
<body>
    <h2>WebSocket чат</h2>
    
    <div id="status" class="status disconnected">Соединение закрыто</div>
    
    <div id="messages"></div>
    
    <div>
        <input type="text" id="message-input" placeholder="Введите сообщение..." disabled>
        <button onclick="sendMessage()" id="send-btn" disabled>Отправить</button>
        <button onclick="connectWebSocket()">Подключиться</button>
    </div>

    <script>
        let ws;
        
        function connectWebSocket() {
            const status = document.getElementById('status');
            const messageInput = document.getElementById('message-input');
            const sendBtn = document.getElementById('send-btn');
            
            try {
                // В реальном приложении здесь будет реальный WebSocket URL
                ws = new WebSocket('ws://localhost:8080');
                
                ws.onopen = function() {
                    status.textContent = 'Соединение установлено';
                    status.className = 'status connected';
                    messageInput.disabled = false;
                    sendBtn.disabled = false;
                };
                
                ws.onmessage = function(event) {
                    addMessage(event.data, 'received');
                };
                
                ws.onclose = function() {
                    status.textContent = 'Соединение закрыто';
                    status.className = 'status disconnected';
                    messageInput.disabled = true;
                    sendBtn.disabled = true;
                };
                
                ws.onerror = function(error) {
                    console.error('WebSocket ошибка:', error);
                    status.textContent = 'Ошибка подключения';
                    status.className = 'status disconnected';
                };
            } catch (e) {
                status.textContent = 'Ошибка подключения: ' + e.message;
                status.className = 'status disconnected';
            }
        }
        
        function sendMessage() {
            const input = document.getElementById('message-input');
            if (input.value.trim() !== '' && ws && ws.readyState === WebSocket.OPEN) {
                ws.send(input.value);
                addMessage(input.value, 'sent');
                input.value = '';
            }
        }
        
        function addMessage(text, type) {
            const messagesDiv = document.getElementById('messages');
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${type}`;
            messageDiv.textContent = text;
            messagesDiv.appendChild(messageDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }
        
        // Отправка сообщения по нажатию Enter
        document.getElementById('message-input').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });
    </script>
</body>
</html>
```

## Практические примеры и шаблоны

### Интерактивная таблица с сортировкой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивная таблица</title>
    <style>
        table {
            border-collapse: collapse;
            width: 100%;
        }
        
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        
        th {
            background-color: #f2f2f2;
            cursor: pointer;
            user-select: none;
        }
        
        th:hover {
            background-color: #e6e6e6;
        }
        
        th.sort-asc::after {
            content: " ▲";
        }
        
        th.sort-desc::after {
            content: " ▼";
        }
        
        tr:nth-child(even) {
            background-color: #f9f9f9;
        }
    </style>
</head>
<body>
    <h2>Интерактивная таблица с сортировкой</h2>
    
    <table id="sortable-table">
        <thead>
            <tr>
                <th onclick="sortTable(0)" data-column="0">Имя</th>
                <th onclick="sortTable(1)" data-column="1">Возраст</th>
                <th onclick="sortTable(2)" data-column="2">Город</th>
                <th onclick="sortTable(3)" data-column="3">Зарплата</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Иван Иванов</td>
                <td>30</td>
                <td>Москва</td>
                <td>75000</td>
            </tr>
            <tr>
                <td>Мария Петрова</td>
                <td>25</td>
                <td>Санкт-Петербург</td>
                <td>65000</td>
            </tr>
            <tr>
                <td>Алексей Сидоров</td>
                <td>35</td>
                <td>Новосибирск</td>
                <td>80000</td>
            </tr>
            <tr>
                <td>Елена Козлова</td>
                <td>28</td>
                <td>Екатеринбург</td>
                <td>70000</td>
            </tr>
        </tbody>
    </table>

    <script>
        let sortDirection = {};
        
        function sortTable(columnIndex) {
            const table = document.getElementById('sortable-table');
            const tbody = table.querySelector('tbody');
            const rows = Array.from(tbody.querySelectorAll('tr'));
            const header = table.querySelectorAll('th')[columnIndex];
            
            // Определить направление сортировки
            const isAscending = sortDirection[columnIndex] !== 'desc';
            sortDirection[columnIndex] = isAscending ? 'desc' : 'asc';
            
            // Сбросить все индикаторы сортировки
            table.querySelectorAll('th').forEach(th => {
                th.classList.remove('sort-asc', 'sort-desc');
            });
            
            // Установить индикатор для текущего столбца
            header.classList.add(isAscending ? 'sort-asc' : 'sort-desc');
            
            // Сортировать строки
            rows.sort((a, b) => {
                const aValue = a.cells[columnIndex].textContent.trim();
                const bValue = b.cells[columnIndex].textContent.trim();
                
                // Попробовать как числа
                const aNum = Number(aValue);
                const bNum = Number(bValue);
                
                if (!isNaN(aNum) && !isNaN(bNum)) {
                    return isAscending ? aNum - bNum : bNum - aNum;
                }
                
                // Иначе как строки
                if (isAscending) {
                    return aValue.localeCompare(bValue, 'ru-RU');
                } else {
                    return bValue.localeCompare(aValue, 'ru-RU');
                }
            });
            
            // Перестроить таблицу
            rows.forEach(row => tbody.appendChild(row));
        }
    </script>
</body>
</html>
```

### Интерактивная карточка с деталями
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивная карточка</title>
    <style>
        .card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 16px;
            margin: 16px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            max-width: 400px;
        }
        
        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }
        
        .card-title {
            margin: 0;
            font-size: 1.2em;
        }
        
        .card-actions {
            display: flex;
            gap: 8px;
        }
        
        .btn {
            background-color: #007acc;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .btn:hover {
            background-color: #005a9e;
        }
        
        .btn-danger {
            background-color: #dc3545;
        }
        
        .btn-danger:hover {
            background-color: #c82333;
        }
        
        .card-details {
            margin-top: 12px;
            padding: 12px;
            background-color: #f9f9f9;
            border-radius: 4px;
            display: none;
        }
        
        .card-details.visible {
            display: block;
        }
        
        .card-footer {
            margin-top: 12px;
            padding-top: 12px;
            border-top: 1px solid #eee;
            display: flex;
            justify-content: space-between;
        }
    </style>
</head>
<body>
    <h2>Интерактивные карточки</h2>
    
    <div class="card">
        <div class="card-header">
            <h3 class="card-title">Продукт 1</h3>
            <div class="card-actions">
                <button class="btn" onclick="toggleDetails('details1')">Детали</button>
                <button class="btn btn-danger" onclick="deleteCard('card1')">Удалить</button>
            </div>
        </div>
        <p>Краткое описание продукта 1</p>
        <div id="details1" class="card-details">
            <p>Детальное описание продукта 1. Здесь может быть много информации о продукте, его характеристиках, преимуществах и т.д.</p>
            <p>Цена: $99.99</p>
            <p>В наличии: 15 шт.</p>
        </div>
        <div class="card-footer">
            <span>Рейтинг: ⭐⭐⭐⭐⭐</span>
            <span>Категория: Электроника</span>
        </div>
    </div>
    
    <div class="card" id="card1">
        <div class="card-header">
            <h3 class="card-title">Продукт 2</h3>
            <div class="card-actions">
                <button class="btn" onclick="toggleDetails('details2')">Детали</button>
                <button class="btn btn-danger" onclick="deleteCard('card2')">Удалить</button>
            </div>
        </div>
        <p>Краткое описание продукта 2</p>
        <div id="details2" class="card-details">
            <p>Детальное описание продукта 2. Здесь может быть много информации о продукте, его характеристиках, преимуществах и т.д.</p>
            <p>Цена: $149.99</p>
            <p>В наличии: 8 шт.</p>
        </div>
        <div class="card-footer">
            <span>Рейтинг: ⭐⭐⭐⭐</span>
            <span>Категория: Одежда</span>
        </div>
    </div>

    <script>
        function toggleDetails(detailsId) {
            const details = document.getElementById(detailsId);
            details.classList.toggle('visible');
        }
        
        function deleteCard(cardId) {
            if (confirm('Вы уверены, что хотите удалить эту карточку?')) {
                const card = document.getElementById(cardId);
                if (card) {
                    card.style.opacity = '0';
                    card.style.transform = 'translateX(-100%)';
                    card.style.transition = 'opacity 0.3s, transform 0.3s';
                    
                    setTimeout(() => {
                        card.remove();
                    }, 300);
                }
            }
        }
    </script>
</body>
</html>
```

## Проверка и тестирование интерактивных элементов

### Инструменты для проверки интерактивности:
1. **Accessibility Insights** - проверка доступности интерактивных элементов
2. **axe-core** - автоматизированная проверка доступности
3. **Lighthouse** - комплексная проверка, включая интерактивность
4. **WAVE** - веб-инструмент для оценки доступности
5. **Инструменты разработчика браузера** - проверка событий и фокуса

### Проверка клавиатурной навигации:
1. Убедитесь, что все интерактивные элементы доступны через Tab
2. Проверьте, что у элементов есть видимые индикаторы фокуса
3. Убедитесь, что последовательность табуляции логична
4. Проверьте, что все функции доступны без мыши

### Проверка доступности:
1. Используйте скринридеры (NVDA, JAWS, VoiceOver) для тестирования
2. Проверьте наличие ARIA-атрибутов при необходимости
3. Убедитесь, что интерактивные элементы имеют понятные метки
4. Проверьте, что состояния элементов правильно отражаются

## Лучшие практики интерактивных элементов

### 1. Предсказуемость взаимодействия
```html
<!-- Хорошо: предсказуемое поведение -->
<button type="button" onclick="openModal()" aria-expanded="false" aria-controls="dialog">
    Открыть диалог
</button>

<!-- Плохо: неочевидное поведение -->
<div onclick="openModal()" role="button" tabindex="0">
    Кликни меня
</div>
```

### 2. Обратная связь пользователю
```html
<!-- Элемент с обратной связью -->
<button id="submit-btn" onclick="handleSubmit()">
    Отправить
</button>
<div id="feedback" aria-live="polite" style="display: none;"></div>

<script>
async function handleSubmit() {
    const btn = document.getElementById('submit-btn');
    const feedback = document.getElementById('feedback');
    
    // Показать состояние загрузки
    btn.textContent = 'Отправка...';
    btn.disabled = true;
    feedback.textContent = 'Отправка данных...';
    feedback.style.display = 'block';
    feedback.className = 'loading';
    
    try {
        // Симуляция отправки данных
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        feedback.textContent = 'Данные успешно отправлены!';
        feedback.className = 'success';
    } catch (error) {
        feedback.textContent = 'Ошибка отправки данных. Попробуйте еще раз.';
        feedback.className = 'error';
    } finally {
        btn.textContent = 'Отправить';
        btn.disabled = false;
        
        // Скрыть сообщение через 5 секунд
        setTimeout(() => {
            feedback.style.display = 'none';
        }, 5000);
    }
}
</script>
```

### 3. Безопасность и защита от атак
```html
<!-- Защита от XSS в интерактивных элементах -->
<form id="user-input-form">
    <label for="comment">Комментарий:</label>
    <textarea id="comment" name="comment"></textarea>
    <button type="submit">Отправить</button>
</form>

<div id="output"></div>

<script>
document.getElementById('user-input-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const comment = document.getElementById('comment').value;
    
    // Экранирование HTML для предотвращения XSS
    const safeComment = escapeHtml(comment);
    
    document.getElementById('output').innerHTML = `
        <div class="comment">
            <p>Ваш комментарий: ${safeComment}</p>
        </div>
    `;
});

function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}
</script>
```

## Заключение

Современные интерактивные элементы HTML предоставляют мощные возможности для создания динамичных и доступных веб-интерфейсов. Правильное использование этих элементов с учетом семантики, доступности, безопасности и пользовательского опыта создает качественные веб-приложения, которые удобны для всех пользователей. Важно сочетать HTML элементы с CSS и JavaScript для достижения наилучших результатов, а также учитывать современные веб-стандарты и лучшие практики разработки.

Ключевые моменты для запоминания:
1. Используйте семантически правильные элементы для конкретных задач
2. Обеспечьте доступность для всех пользователей, включая пользователей вспомогательных технологий
3. Предоставляйте обратную связь о состоянии интерактивных элементов
4. Обеспечьте клавиатурную навигацию и взаимодействие
5. Используйте ARIA-атрибуты при необходимости
6. Тестируйте интерактивность с помощью различных инструментов
7. Защищайте интерактивные элементы от потенциальных угроз безопасности

Эти практики помогут создать интерактивные веб-интерфейсы, которые будут не только функциональными, но и безопасными, доступными и удобными для пользователей.

## Следующие темы
- [[HTML/Формы]]
- [[HTML/Доступность]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Веб-компоненты]]

## Теги
#html #elements #interactive-elements #forms #web-development #frontend #accessibility #user-interface #javascript #html5 #web-components #drag-and-drop #websockets #security #best-practices