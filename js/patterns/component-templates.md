---
aliases: ["Шаблоны компонентов", "Component Templates", "JS Components"]
tags: [js, patterns, components, frontend]
---

# Шаблоны для создания компонентов

## Введение в компонентный подход

Компонентный подход позволяет создавать переиспользуемые, изолированные части интерфейса с четким API и жизненным циклом.

## Базовые шаблоны компонентов

### Простой компонент с жизненным циклом

```javascript
class BaseComponent {
  constructor(container, props = {}) {
    this.container = container;
    this.props = props;
    this.state = {};
    this.refs = {};
    this.element = null;
  }
  
  // Методы жизненного цикла
  componentWillMount() {
    // Вызывается перед монтированием
  }
  
  componentDidMount() {
    // Вызывается после монтирования
  }
  
  componentWillReceiveProps(nextProps) {
    // Вызывается при получении новых props
  }
  
  shouldComponentUpdate(nextProps, nextState) {
    // Определяет, нужно ли обновлять компонент
    return true;
  }
  
  componentWillUpdate() {
    // Вызывается перед обновлением
  }
  
  componentDidUpdate() {
    // Вызывается после обновления
  }
  
  componentWillUnmount() {
    // Вызывается перед размонтированием
    this.cleanup();
  }
  
  // Управление состоянием
  setState(newState, callback) {
    const prevState = { ...this.state };
    this.state = { ...this.state, ...newState };
    
    if (this.shouldComponentUpdate(this.props, this.state)) {
      this.componentWillUpdate();
      this.render();
      this.componentDidUpdate();
      
      if (callback) callback(prevState, this.state);
    }
  }
  
  // Отрисовка компонента
  render() {
    if (!this.element) {
      this.element = document.createElement('div');
      this.container.appendChild(this.element);
    }
    
    this.element.innerHTML = this.template();
    this.refs = this.createRefs();
    this.bindEvents();
  }
  
  // Шаблон компонента (переопределяется в наследниках)
  template() {
    return '';
  }
  
  // Создание ссылок на элементы
  createRefs() {
    const refs = {};
    const refElements = this.element.querySelectorAll('[data-ref]');
    
    refElements.forEach(el => {
      const refName = el.getAttribute('data-ref');
      refs[refName] = el;
    });
    
    return refs;
  }
  
  // Привязка событий
  bindEvents() {
    // Переопределяется в наследниках
  }
  
  // Очистка ресурсов
  cleanup() {
    // Очистка событий, таймеров и т.д.
  }
  
  // Монтирование компонента
  mount() {
    this.componentWillMount();
    this.render();
    this.componentDidMount();
  }
  
  // Размонтирование компонента
  unmount() {
    this.componentWillUnmount();
    if (this.element && this.element.parentNode) {
      this.element.parentNode.removeChild(this.element);
    }
  }
}

// Пример использования базового компонента
class ButtonComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      text: 'Кнопка',
      onClick: () => {},
      disabled: false,
      ...props
    });
  }
  
  template() {
    return `
      <button 
        class="btn ${this.props.disabled ? 'btn-disabled' : ''}"
        ${this.props.disabled ? 'disabled' : ''}
        data-ref="button"
      >
        ${this.props.text}
      </button>
    `;
  }
  
  bindEvents() {
    if (this.refs.button) {
      this.refs.button.addEventListener('click', this.props.onClick);
    }
  }
  
  setDisabled(disabled) {
    this.props.disabled = disabled;
    this.setState({}); // Перерисовка
  }
}
```

### Компонент с управляемыми полями

```javascript
class FormFieldComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      label: '',
      type: 'text',
      value: '',
      placeholder: '',
      required: false,
      onChange: () => {},
      ...props
    });
    
    this.state = { value: this.props.value };
  }
  
  template() {
    return `
      <div class="form-field">
        ${this.props.label ? `<label>${this.props.label}</label>` : ''}
        <input
          type="${this.props.type}"
          value="${this.state.value}"
          placeholder="${this.props.placeholder}"
          ${this.props.required ? 'required' : ''}
          data-ref="input"
        >
        <div class="error-message" data-ref="error" style="display: none;"></div>
      </div>
    `;
  }
  
  bindEvents() {
    if (this.refs.input) {
      this.refs.input.addEventListener('input', (e) => {
        this.setState({ value: e.target.value });
        this.props.onChange(e.target.value, e);
      });
      
      this.refs.input.addEventListener('blur', () => {
        this.validate();
      });
    }
  }
  
  validate() {
    let error = '';
    
    if (this.props.required && !this.state.value.trim()) {
      error = 'Поле обязательно для заполнения';
    } else if (this.props.type === 'email' && this.state.value && !this.isValidEmail(this.state.value)) {
      error = 'Введите корректный email';
    }
    
    if (error) {
      this.showError(error);
      return false;
    } else {
      this.hideError();
      return true;
    }
  }
  
  showError(message) {
    if (this.refs.error) {
      this.refs.error.textContent = message;
      this.refs.error.style.display = 'block';
    }
  }
  
  hideError() {
    if (this.refs.error) {
      this.refs.error.style.display = 'none';
    }
  }
  
  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  getValue() {
    return this.state.value;
  }
  
  setValue(value) {
    this.setState({ value });
  }
}
```

## Продвинутые шаблоны

### Компонент с вложенными компонентами

```javascript
class CompositeComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, props);
    this.children = new Map();
  }
  
  // Добавление дочернего компонента
  addChild(key, component) {
    this.children.set(key, component);
    return this;
  }
  
  // Удаление дочернего компонента
  removeChild(key) {
    const component = this.children.get(key);
    if (component) {
      component.unmount();
      this.children.delete(key);
    }
    return this;
  }
  
  // Получение дочернего компонента
  getChild(key) {
    return this.children.get(key);
  }
  
  // Отрисовка всех дочерних компонентов
  renderChildren() {
    this.children.forEach((component, key) => {
      const container = this.element.querySelector(`[data-child="${key}"]`);
      if (container) {
        component.container = container;
        component.mount();
      }
    });
  }
  
  render() {
    super.render();
    this.renderChildren();
  }
  
  cleanup() {
    this.children.forEach(component => component.unmount());
    this.children.clear();
  }
}

// Пример составного компонента
class UserProfileComponent extends CompositeComponent {
  constructor(container, props) {
    super(container, props);
    
    // Создание дочерних компонентов
    this.addChild('avatar', new AvatarComponent(null, { 
      src: props.avatarUrl,
      alt: 'Аватар пользователя'
    }));
    
    this.addChild('info', new UserInfoComponent(null, {
      name: props.name,
      email: props.email
    }));
    
    this.addChild('actions', new ButtonComponent(null, {
      text: 'Редактировать',
      onClick: props.onEdit
    }));
  }
  
  template() {
    return `
      <div class="user-profile">
        <div class="avatar-container" data-child="avatar"></div>
        <div class="info-container" data-child="info"></div>
        <div class="actions-container" data-child="actions"></div>
      </div>
    `;
  }
}
```

### Компонент с асинхронной загрузкой данных

```javascript
class AsyncDataComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      apiUrl: '',
      loadingTemplate: '<div>Загрузка...</div>',
      errorTemplate: '<div>Ошибка загрузки</div>',
      ...props
    });
    
    this.state = {
      data: null,
      loading: false,
      error: null
    };
  }
  
  template() {
    if (this.state.error) {
      return this.props.errorTemplate;
    }
    
    if (this.state.loading) {
      return this.props.loadingTemplate;
    }
    
    if (this.state.data) {
      return this.renderData(this.state.data);
    }
    
    return '<div>Нет данных</div>';
  }
  
  // Метод для рендеринга данных (переопределяется в наследниках)
  renderData(data) {
    return JSON.stringify(data);
  }
  
  async loadData() {
    this.setState({ loading: true, error: null });
    
    try {
      const response = await fetch(this.props.apiUrl);
      if (!response.ok) {
        throw new Error(`HTTP ошибка! статус: ${response.status}`);
      }
      
      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      this.setState({ error: error.message, loading: false });
    }
  }
  
  componentDidMount() {
    if (this.props.apiUrl) {
      this.loadData();
    }
  }
  
  // Метод для ручной перезагрузки данных
  refresh() {
    this.loadData();
  }
}

// Пример конкретного компонента с данными
class UserListComponent extends AsyncDataComponent {
  renderData(users) {
    return `
      <div class="user-list">
        ${users.map(user => `
          <div class="user-item" data-user-id="${user.id}">
            <img src="${user.avatar}" alt="${user.name}" class="user-avatar">
            <div class="user-info">
              <h4>${user.name}</h4>
              <p>${user.email}</p>
            </div>
          </div>
        `).join('')}
      </div>
    `;
  }
}
```

### Компонент с кэшированием

```javascript
class CachedComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, props);
    this.cache = new Map();
    this.cacheTimeout = props.cacheTimeout || 5 * 60 * 1000; // 5 минут
  }
  
  async getCachedData(key, dataLoader) {
    const cached = this.cache.get(key);
    
    if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
      return cached.data;
    }
    
    const data = await dataLoader();
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  cleanup() {
    // Очистка кэша при размонтировании
    this.clearCache();
  }
}
```

## Шаблоны для конкретных задач

### Компонент списка с фильтрацией

```javascript
class FilterableListComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      items: [],
      filterFields: [],
      searchPlaceholder: 'Поиск...',
      ...props
    });
    
    this.state = {
      items: this.props.items,
      filteredItems: this.props.items,
      searchTerm: '',
      sortField: null,
      sortDirection: 'asc'
    };
  }
  
  template() {
    return `
      <div class="filterable-list">
        <div class="search-container">
          <input
            type="text"
            placeholder="${this.props.searchPlaceholder}"
            value="${this.state.searchTerm}"
            data-ref="searchInput"
          >
        </div>
        <div class="items-container">
          ${this.state.filteredItems.map(item => this.renderItem(item)).join('')}
        </div>
      </div>
    `;
  }
  
  bindEvents() {
    if (this.refs.searchInput) {
      this.refs.searchInput.addEventListener('input', (e) => {
        this.setState({ searchTerm: e.target.value });
        this.filterItems();
      });
    }
  }
  
  renderItem(item) {
    // Переопределяется в наследниках
    return `<div>${JSON.stringify(item)}</div>`;
  }
  
  filterItems() {
    let filtered = this.state.items;
    
    if (this.state.searchTerm) {
      const term = this.state.searchTerm.toLowerCase();
      filtered = filtered.filter(item => {
        return this.props.filterFields.some(field => {
          const value = this.getNestedProperty(item, field);
          return String(value).toLowerCase().includes(term);
        });
      });
    }
    
    this.setState({ filteredItems: filtered });
  }
  
  // Вспомогательный метод для получения вложенных свойств
  getNestedProperty(obj, path) {
    return path.split('.').reduce((current, key) => current && current[key], obj);
  }
  
  // Обновление списка
  updateItems(newItems) {
    this.setState({
      items: newItems,
      filteredItems: newItems
    });
    this.filterItems();
  }
}

// Пример конкретного компонента списка
class TodoListComponent extends FilterableListComponent {
  constructor(container, props) {
    super(container, {
      filterFields: ['title', 'description'],
      ...props
    });
  }
  
  renderItem(todo) {
    return `
      <div class="todo-item ${todo.completed ? 'completed' : ''}">
        <input
          type="checkbox"
          ${todo.completed ? 'checked' : ''}
          onchange="todoList.toggleTodo(${todo.id})"
        >
        <span class="todo-title">${todo.title}</span>
        <button onclick="todoList.deleteTodo(${todo.id})">Удалить</button>
      </div>
    `;
  }
  
  toggleTodo(id) {
    const items = this.state.items.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    this.updateItems(items);
  }
  
  deleteTodo(id) {
    const items = this.state.items.filter(todo => todo.id !== id);
    this.updateItems(items);
  }
}
```

### Компонент модального окна

```javascript
class ModalComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      title: '',
      closable: true,
      onClose: () => {},
      ...props
    });
    
    this.state = { visible: false };
  }
  
  template() {
    if (!this.state.visible) {
      return '';
    }
    
    return `
      <div class="modal-overlay" data-ref="overlay">
        <div class="modal">
          <div class="modal-header">
            <h3>${this.props.title}</h3>
            ${this.props.closable ? '<button class="modal-close" data-ref="closeBtn">×</button>' : ''}
          </div>
          <div class="modal-body" data-ref="body">
            ${this.renderBody()}
          </div>
          <div class="modal-footer" data-ref="footer">
            ${this.renderFooter()}
          </div>
        </div>
      </div>
    `;
  }
  
  renderBody() {
    // Переопределяется в наследниках
    return '';
  }
  
  renderFooter() {
    // Переопределяется в наследниках
    return '';
  }
  
  bindEvents() {
    if (this.refs.overlay) {
      this.refs.overlay.addEventListener('click', (e) => {
        if (e.target === this.refs.overlay && this.props.closable) {
          this.close();
        }
      });
    }
    
    if (this.refs.closeBtn) {
      this.refs.closeBtn.addEventListener('click', () => this.close());
    }
    
    // Закрытие по клавише Escape
    this.handleEscape = (e) => {
      if (e.key === 'Escape' && this.state.visible) {
        this.close();
      }
    };
    
    document.addEventListener('keydown', this.handleEscape);
  }
  
  open() {
    this.setState({ visible: true });
    document.body.style.overflow = 'hidden'; // Предотвращение скролла
  }
  
  close() {
    this.setState({ visible: false });
    document.body.style.overflow = ''; // Восстановление скролла
    this.props.onClose();
  }
  
  cleanup() {
    document.removeEventListener('keydown', this.handleEscape);
    document.body.style.overflow = '';
  }
}

// Пример конкретного модального окна
class ConfirmModal extends ModalComponent {
  constructor(container, props) {
    super(container, {
      title: 'Подтверждение',
      ...props
    });
    
    this.onConfirm = props.onConfirm || (() => {});
  }
  
  renderBody() {
    return `<p>${this.props.message || 'Вы уверены?'}</p>`;
  }
  
  renderFooter() {
    return `
      <button class="btn btn-primary" onclick="confirmModal.confirm()">ОК</button>
      <button class="btn btn-secondary" onclick="confirmModal.close()">Отмена</button>
    `;
  }
  
  confirm() {
    this.onConfirm();
    this.close();
  }
}
```

### Компонент табов

```javascript
class TabsComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      tabs: [],
      activeTab: 0,
      ...props
    });
    
    this.state = { activeTab: this.props.activeTab };
  }
  
  template() {
    return `
      <div class="tabs">
        <div class="tab-headers">
          ${this.props.tabs.map((tab, index) => `
            <button
              class="tab-header ${index === this.state.activeTab ? 'active' : ''}"
              onclick="tabs.setActiveTab(${index})"
            >
              ${tab.title}
            </button>
          `).join('')}
        </div>
        <div class="tab-content">
          ${this.renderTabContent(this.state.activeTab)}
        </div>
      </div>
    `;
  }
  
  renderTabContent(index) {
    const tab = this.props.tabs[index];
    if (tab.component) {
      // Если указан компонент, рендерим его
      const container = document.createElement('div');
      const component = new tab.component(container, tab.props || {});
      component.mount();
      return container.innerHTML;
    }
    return tab.content || '';
  }
  
  setActiveTab(index) {
    this.setState({ activeTab: index });
  }
}
```

## Практические шаблоны

### Компонент с валидацией формы

```javascript
class ValidatedFormComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, props);
    this.validators = new Map();
    this.errors = new Map();
  }
  
  // Добавление валидатора для поля
  addValidator(fieldName, validatorFn, errorMessage) {
    this.validators.set(fieldName, {
      validate: validatorFn,
      message: errorMessage
    });
    return this;
  }
  
  // Валидация конкретного поля
  validateField(fieldName, value) {
    const validator = this.validators.get(fieldName);
    if (!validator) return true;
    
    const isValid = validator.validate(value);
    if (!isValid) {
      this.errors.set(fieldName, validator.message);
    } else {
      this.errors.delete(fieldName);
    }
    return isValid;
  }
  
  // Валидация всей формы
  validateForm(formData) {
    let isValid = true;
    this.errors.clear();
    
    for (const [fieldName, validator] of this.validators) {
      const value = formData[fieldName];
      if (!validator.validate(value)) {
        this.errors.set(fieldName, validator.message);
        isValid = false;
      }
    }
    
    return isValid;
  }
  
  // Получение ошибки для поля
  getFieldError(fieldName) {
    return this.errors.get(fieldName) || '';
  }
  
  // Проверка, есть ли ошибки
  hasErrors() {
    return this.errors.size > 0;
  }
  
  // Очистка ошибок
  clearErrors() {
    this.errors.clear();
  }
}
```

### Компонент с drag and drop

```javascript
class DraggableListComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      items: [],
      onReorder: () => {},
      ...props
    });
    
    this.state = { items: this.props.items };
    this.draggedItem = null;
    this.draggedIndex = null;
  }
  
  template() {
    return `
      <div class="draggable-list">
        ${this.state.items.map((item, index) => `
          <div
            class="draggable-item"
            draggable="true"
            data-index="${index}"
            ondragstart="draggableList.handleDragStart(event, ${index})"
            ondragover="draggableList.handleDragOver(event)"
            ondrop="draggableList.handleDrop(event, ${index})"
            ondragend="draggableList.handleDragEnd(event)"
          >
            ${this.renderItem(item, index)}
          </div>
        `).join('')}
      </div>
    `;
  }
  
  renderItem(item, index) {
    return `${index + 1}. ${item.title || item.name || item}`;
  }
  
  handleDragStart(event, index) {
    this.draggedItem = this.state.items[index];
    this.draggedIndex = index;
    event.dataTransfer.setData('text/plain', index);
    event.target.classList.add('dragging');
  }
  
  handleDragOver(event) {
    event.preventDefault(); // Необходимо для drop
  }
  
  handleDrop(event, targetIndex) {
    event.preventDefault();
    
    if (this.draggedIndex !== targetIndex) {
      const newItems = [...this.state.items];
      newItems.splice(this.draggedIndex, 1);
      newItems.splice(targetIndex, 0, this.draggedItem);
      
      this.setState({ items: newItems });
      this.props.onReorder(newItems);
    }
    
    this.handleDragEnd(event);
  }
  
  handleDragEnd(event) {
    event.target.classList.remove('dragging');
    this.draggedItem = null;
    this.draggedIndex = null;
  }
}
```

## Лучшие практики

### Структура компонента

```javascript
// Шаблон для создания компонентов по лучшим практикам
class BestPracticeComponent extends BaseComponent {
  constructor(container, props = {}) {
    // 1. Валидация props
    this.validateProps(props);
    
    super(container, {
      // 2. Установка значений по умолчанию
      className: '',
      style: {},
      ...props
    });
    
    // 3. Инициализация состояния
    this.state = this.getInitialState();
    
    // 4. Привязка методов к контексту
    this.handleClick = this.handleClick.bind(this);
    this.handleInputChange = this.handleInputChange.bind(this);
  }
  
  // 5. Валидация входных параметров
  validateProps(props) {
    if (props.requiredProp === undefined) {
      throw new Error('requiredProp is required');
    }
  }
  
  // 6. Инициализация начального состояния
  getInitialState() {
    return {
      data: this.props.initialData || [],
      loading: false,
      error: null
    };
  }
  
  // 7. Чистое определение шаблона
  template() {
    return `
      <div class="best-practice-component ${this.props.className}">
        ${this.renderContent()}
      </div>
    `;
  }
  
  renderContent() {
    // Переопределяется в наследниках
    return '';
  }
  
  // 8. Четкое разделение обработчиков событий
  handleClick(event) {
    // Логика обработки клика
  }
  
  handleInputChange(event) {
    // Логика обработки изменения input
  }
  
  // 9. Методы для внешнего API
  getData() {
    return this.state.data;
  }
  
  setData(data) {
    this.setState({ data });
  }
  
  // 10. Очистка ресурсов
  cleanup() {
    // Очистка таймеров, событий и т.д.
  }
}
```

## Ссылки по теме

- [[js/components/lifecycle]] - Жизненный цикл компонентов
- [[js/components/state-management]] - Управление состоянием компонентов
- [[js/patterns/component-pattern]] - Паттерн Компонент
- [[js/components/event-handling]] - Обработка событий в компонентах
- [[js/components/performance]] - Производительность компонентов