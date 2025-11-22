---
aliases: ["Псевдоклассы форм", "Псевдоклассы состояний", "Формы и состояния CSS"]
tags: [css, pseudo-classes, forms, states]
---

# Псевдоклассы форм и состояний (focus, hover, active, focus-within и др.)

## Введение

Псевдоклассы форм и состояний позволяют применять стили к элементам в зависимости от их текущего состояния. Эти псевдоклассы играют важную роль в создании интерактивных интерфейсов и улучшении пользовательского опыта.

## Псевдоклассы взаимодействия с элементами

### :hover

Применяется, когда пользователь наводит курсор на элемент:

```css
.button:hover {
  background-color: #0056b3;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.link:hover {
  color: #0056b3;
  text-decoration: underline;
}

.card:hover {
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  transform: scale(1.02);
}
```

### :active

Применяется, когда элемент активен (обычно во время нажатия):

```css
.button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  background-color: #004085;
}

.link:active {
  color: #004085;
}
```

### :focus

Применяется, когда элемент находится в фокусе (например, при навигации с клавиатуры):

```css
.input:focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
  border-color: #80bdff;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
}

.button:focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
}
```

### :focus-within

Применяется к элементу, который содержит дочерний элемент в фокусе:

```css
.form-group {
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group:focus-within {
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
}
```

### :focus-visible

Применяется к элементу в фокусе, когда фокус должен быть визуально показан (например, при навигации с клавиатуры, но не при клике мышью):

```css
.button {
  outline: none;
}

.button:focus {
  /* Стили для фокуса при любом взаимодействии */
}

.button:focus-visible {
  /* Стили для фокуса только при клавиатурной навигации */
  outline: 2px solid #007bff;
  outline-offset: 2px;
}
```

## Псевдоклассы форм

### :enabled и :disabled

Определяют, включен или выключен элемент формы:

```css
.input:enabled {
  background-color: white;
  cursor: text;
}

.input:disabled {
  background-color: #e9ecef;
  color: #6c757d;
  cursor: not-allowed;
}

.button:enabled {
  opacity: 1;
  cursor: pointer;
}

.button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
```

### :checked

Применяется к выбранным элементам (input[type="radio"], input[type="checkbox"], option):

```css
.checkbox {
  position: relative;
  display: inline-block;
  width: 20px;
  height: 20px;
}

.checkbox input[type="checkbox"] {
  opacity: 0;
  position: absolute;
}

.checkbox input[type="checkbox"] + label {
  display: inline-block;
  width: 20px;
  height: 20px;
  background: #ddd;
  border: 1px solid #999;
  cursor: pointer;
}

.checkbox input[type="checkbox"]:checked + label {
  background: #007bff;
  border-color: #007bff;
}

.radio {
  position: relative;
  display: inline-block;
  width: 20px;
  height: 20px;
}

.radio input[type="radio"] {
  opacity: 0;
  position: absolute;
}

.radio input[type="radio"] + label {
  display: inline-block;
  width: 20px;
  height: 20px;
  background: #ddd;
  border-radius: 50%;
  border: 1px solid #999;
  cursor: pointer;
}

.radio input[type="radio"]:checked + label {
  background: #007bff;
  border-color: #007bff;
}
```

### :indeterminate

Применяется к элементам input типа checkbox, которые находятся в неопределенном состоянии:

```css
.progress-checkbox input[type="checkbox"]:indeterminate {
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 16 16'%3e%3cpath stroke='white' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M4 8h8'/%3e%3c/svg%3e");
  background-color: #0d6efd;
  background-size: 100% 100%;
  background-position: center;
  background-repeat: no-repeat;
}
```

### :default

Применяется к элементам формы, которые являются по умолчанию:

```css
input:default {
  border: 2px solid #28a745;
}

option:default {
  background-color: #d4edda;
}
```

### :valid и :invalid

Применяются к элементам формы в зависимости от валидности:

```css
.input:valid {
  border-color: #28a745;
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'%3e%3cpath fill='%2328a745' d='M2.3 6.73L.6 4.53c-.4-1.04.46-1.4 1.1-.8l1.1 1.4 3.4-3.8c.6-.63 1.6-.27 1.2.7l-4 4.6c-.43.5-.8.4-1.1.1z'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}

.input:invalid {
  border-color: #dc3545;
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='%23dc3545'%3e%3ccircle cx='6' cy='6' r='4.5'/%3e%3cpath stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/%3e%3ccircle cx='6' cy='8.2' r='.6' fill='%23dc3545' stroke='none'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}

.input:valid:focus {
  border-color: #28a745;
  box-shadow: 0 0 0 0.2rem rgba(40,167,69,.25);
}

.input:invalid:focus {
  border-color: #dc3545;
  box-shadow: 0 0 0 0.2rem rgba(220,53,69,.25);
}
```

### :required и :optional

Определяют, является ли элемент формы обязательным или необязательным:

```css
.input:required {
  border-left: 4px solid #dc3545;
}

.input:optional {
  border-left: 4px solid #28a745;
}

.label:has(+ .input:required)::after {
  content: " *";
  color: #dc3545;
}
```

### :read-only и :read-write

Определяют, доступен ли элемент для редактирования:

```css
.input:read-only {
  background-color: #e9ecef;
  cursor: default;
}

.input:read-write {
  background-color: white;
  cursor: text;
}
```

## Практические примеры

### Интерактивная форма с валидацией

```html
<form class="validation-form">
  <div class="form-group">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <span class="validation-message">Пожалуйста, введите действительный email</span>
  </div>
  
  <div class="form-group">
    <label for="password">Пароль:</label>
    <input type="password" id="password" name="password" required minlength="8">
    <span class="validation-message">Пароль должен содержать не менее 8 символов</span>
  </div>
  
  <div class="form-group">
    <input type="checkbox" id="terms" name="terms" required>
    <label for="terms">Я согласен с условиями</label>
  </div>
  
  <button type="submit">Отправить</button>
</form>
```

```css
.validation-form {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
  position: relative;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.form-group input:required {
  border-left: 4px solid #007bff;
}

.form-group input:valid {
  border-color: #28a745;
}

.form-group input:invalid:not(:placeholder-shown) {
  border-color: #dc3545;
}

.form-group input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
}

.validation-message {
  display: block;
  color: #dc3545;
  font-size: 0.875em;
  margin-top: 5px;
  opacity: 0;
  transition: opacity 0.3s;
}

.form-group input:invalid:not(:placeholder-shown) + .validation-message {
  opacity: 1;
}

.form-group input[type="checkbox"] {
  width: auto;
  margin-right: 10px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  width: 100%;
}

button:hover {
  background-color: #0056b3;
}

button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}
```

### Интерактивные переключатели (toggle switches)

```html
<div class="toggle-container">
  <label class="toggle-switch">
    <input type="checkbox">
    <span class="slider"></span>
    <span class="toggle-label">Темная тема</span>
  </label>
</div>
```

```css
.toggle-switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.toggle-switch input {
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

input:focus + .slider {
  box-shadow: 0 0 1px #2196F3;
}

.toggle-label {
  margin-left: 70px;
  user-select: none;
}
```

## Совместимость и доступность

При использовании псевдоклассов состояний важно учитывать доступность:

```css
/* Обеспечение видимости фокуса для пользователей клавиатуры */
.focus-indicator:focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* Использование prefers-reduced-motion для пользователей, чувствительных к анимации */
@media (prefers-reduced-motion: reduce) {
  .button:hover,
  .card:hover {
    transform: none; /* Убираем анимацию трансформации */
  }
  
  .button,
  .card {
    transition: none; /* Убираем все переходы */
  }
}
```

## Заключение

Псевдоклассы форм и состояний являются важным инструментом для создания интерактивных и доступных веб-интерфейсов. Правильное использование этих псевдоклассов позволяет улучшить пользовательский опыт и обеспечить интуитивное взаимодействие с элементами страницы.

Для изучения других категорий псевдоклассов рекомендуется ознакомиться с [[Псевдоклассы структурного соответствия (nth-child, first-child, last-child, only-child и др.)]] и [[Псевдоклассы пользовательских действий (target, lang, dir и др.)]].

## Ссылки

- [MDN: Псевдоклассы](https://developer.mozilla.org/ru/docs/Web/CSS/Pseudo-classes)
- [MDN: :focus-within](https://developer.mozilla.org/ru/docs/Web/CSS/:focus-within)
- [MDN: Псевдоклассы действий](https://developer.mozilla.org/ru/docs/Web/CSS/Pseudo-classes#%D0%BF%D1%81%D0%B5%D0%B2%D0%B4%D0%BE%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D1%8B_%D0%B4%D0%B5%D0%B9%D1%81%D1%82%D0%B2%D0%B8%D1%8F)
- [CSS-Tricks: The :focus-within Pseudo-Class](https://css-tricks.com/almanac/selectors/f/focus-within/)