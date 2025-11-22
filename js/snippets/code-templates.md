---
aliases: ["Шаблоны кода", "JS Templates", "Code Templates"]
tags: [js, templates, boilerplate, patterns]
---

# Шаблоны для копирования и адаптации

## Введение

Этот раздел содержит готовые шаблоны кода, которые можно копировать и адаптировать под конкретные задачи. Эти шаблоны охватывают типичные сценарии разработки и могут служить основой для более сложных решений.

## Шаблоны компонентов

### 1. Базовый класс компонента

```javascript
/**
 * Базовый класс компонента с жизненным циклом
 */
class BaseComponent {
  /**
   * @param {HTMLElement} container - Контейнер для монтирования компонента
   * @param {Object} props - Свойства компонента
   */
  constructor(container, props = {}) {
    this.container = container;
    this.props = { ...this.getDefaultProps(), ...props };
    this.state = { ...this.getInitialState() };
    this.refs = {};
    this.element = null;
    this.isMounted = false;
  }

  /**
   * Возвращает свойства по умолчанию
   * @returns {Object}
   */
  getDefaultProps() {
    return {};
  }

  /**
   * Возвращает начальное состояние
   * @returns {Object}
   */
  getInitialState() {
    return {};
  }

  /**
   * Метод вызывается перед монтированием
   */
  componentWillMount() {}

  /**
   * Метод вызывается после монтирования
   */
  componentDidMount() {}

  /**
   * Метод вызывается перед обновлением
   */
  componentWillUpdate() {}

  /**
   * Метод вызывается после обновления
   */
  componentDidUpdate() {}

  /**
   * Метод вызывается перед размонтированием
   */
  componentWillUnmount() {}

  /**
   * Проверяет, нужно ли обновлять компонент
   * @param {Object} nextProps - Новые свойства
   * @param {Object} nextState - Новое состояние
   * @returns {boolean}
   */
  shouldComponentUpdate(nextProps, nextState) {
    return true;
  }

  /**
   * Обновляет состояние компонента
   * @param {Object} newState - Новое состояние
   * @param {Function} callback - Коллбэк после обновления
   */
  setState(newState, callback) {
    const prevState = { ...this.state };
    this.state = { ...this.state, ...newState };

    if (this.shouldComponentUpdate(this.props, this.state)) {
      this.componentWillUpdate();
      this.render();
      this.componentDidUpdate();
      
      if (callback) {
        callback(prevState, this.state);
      }
    }
  }

  /**
   * Возвращает HTML разметку компонента
   * @returns {string}
   */
  template() {
    return '';
  }

  /**
   * Создает ссылки на элементы
   */
  createRefs() {
    const refElements = this.element.querySelectorAll('[data-ref]');
    refElements.forEach(el => {
      const refName = el.getAttribute('data-ref');
      this.refs[refName] = el;
    });
  }

  /**
   * Привязывает обработчики событий
   */
  bindEvents() {}

  /**
   * Отрисовывает компонент
   */
  render() {
    if (!this.element) {
      this.element = document.createElement('div');
      this.container.appendChild(this.element);
    }

    this.element.innerHTML = this.template();
    this.createRefs();
    this.bindEvents();
  }

  /**
   * Монтирует компонент в DOM
   */
  mount() {
    if (this.isMounted) return;

    this.componentWillMount();
    this.render();
    this.isMounted = true;
    this.componentDidMount();
  }

  /**
   * Размонтирует компонент из DOM
   */
  unmount() {
    if (!this.isMounted) return;

    this.componentWillUnmount();
    if (this.element && this.element.parentNode) {
      this.element.parentNode.removeChild(this.element);
    }
    this.isMounted = false;
  }
}

// Пример использования базового компонента
class ButtonComponent extends BaseComponent {
  getDefaultProps() {
    return {
      text: 'Кнопка',
      onClick: () => {},
      disabled: false,
      className: 'btn'
    };
  }

  template() {
    return `
      <button 
        class="${this.props.className} ${this.props.disabled ? 'btn-disabled' : ''}"
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

### 2. Шаблон для модального окна

```javascript
/**
 * Компонент модального окна
 */
class ModalComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      title: '',
      content: '',
      closable: true,
      showCloseButton: true,
      onClose: () => {},
      onOpen: () => {},
      className: '',
      ...props
    });
    
    this.state = {
      visible: false,
      ...this.state
    };
  }

  template() {
    if (!this.state.visible) {
      return '';
    }

    return `
      <div class="modal-overlay ${this.props.className}" data-ref="overlay">
        <div class="modal-content">
          <div class="modal-header">
            <h3 data-ref="title">${this.props.title}</h3>
            ${this.props.showCloseButton && this.props.closable ? 
              '<button class="modal-close" data-ref="closeButton">&times;</button>' : ''}
          </div>
          <div class="modal-body" data-ref="body">
            ${this.props.content}
          </div>
          <div class="modal-footer" data-ref="footer">
            ${this.props.footer || ''}
          </div>
        </div>
      </div>
    `;
  }

  bindEvents() {
    if (this.refs.overlay && this.props.closable) {
      this.refs.overlay.addEventListener('click', (e) => {
        if (e.target === this.refs.overlay) {
          this.close();
        }
      });
    }

    if (this.refs.closeButton) {
      this.refs.closeButton.addEventListener('click', () => this.close());
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
    document.body.style.overflow = 'hidden';
    this.props.onOpen();
  }

  close() {
    this.setState({ visible: false });
    document.body.style.overflow = '';
    this.props.onClose();
  }

  componentWillUnmount() {
    document.removeEventListener('keydown', this.handleEscape);
    document.body.style.overflow = '';
  }
}

// Пример использования
// const modal = new ModalComponent(document.body, {
//   title: 'Заголовок модального окна',
//   content: '<p>Содержимое модального окна</p>',
//   footer: '<button class="btn" onclick="modal.close()">Закрыть</button>',
//   onClose: () => console.log('Модальное окно закрыто')
// });
// modal.mount();
// modal.open();
```

### 3. Шаблон для формы

```javascript
/**
 * Компонент формы с валидацией
 */
class FormComponent extends BaseComponent {
  constructor(container, props = {}) {
    super(container, {
      onSubmit: () => {},
      onValidate: () => true,
      validateOnSubmit: true,
      showErrors: true,
      ...props
    });
    
    this.state = {
      errors: {},
      submitting: false,
      ...this.state
    };
    
    this.validators = new Map();
  }

  /**
   * Добавляет валидатор для поля
   * @param {string} fieldName - Имя поля
   * @param {Function} validator - Функция валидации
   * @param {string} errorMessage - Сообщение об ошибке
   */
  addValidator(fieldName, validator, errorMessage) {
    if (!this.validators.has(fieldName)) {
      this.validators.set(fieldName, []);
    }
    this.validators.get(fieldName).push({ validator, errorMessage });
  }

  /**
   * Валидирует конкретное поле
   * @param {string} fieldName - Имя поля
   * @param {*} value - Значение поля
   * @returns {Array} Массив ошибок
   */
  validateField(fieldName, value) {
    const fieldValidators = this.validators.get(fieldName) || [];
    const errors = [];

    for (const { validator, errorMessage } of fieldValidators) {
      if (!validator(value)) {
        errors.push(errorMessage);
      }
    }

    return errors;
  }

  /**
   * Валидирует всю форму
   * @param {Object} formData - Данные формы
   * @returns {Object} Объект с ошибками
   */
  validateForm(formData) {
    const errors = {};

    for (const [fieldName, fieldValidators] of this.validators) {
      const value = formData[fieldName];
      const fieldErrors = [];

      for (const { validator, errorMessage } of fieldValidators) {
        if (!validator(value)) {
          fieldErrors.push(errorMessage);
        }
      }

      if (fieldErrors.length > 0) {
        errors[fieldName] = fieldErrors;
      }
    }

    return errors;
  }

  template() {
    return `
      <form class="form-component" data-ref="form">
        <div class="form-body" data-ref="body">
          ${this.renderFields()}
        </div>
        <div class="form-footer">
          <button type="submit" class="btn btn-primary" data-ref="submitBtn">
            ${this.state.submitting ? 'Отправка...' : 'Отправить'}
          </button>
        </div>
        ${this.state.errors._form ? 
          `<div class="form-error">${this.state.errors._form}</div>` : ''}
      </form>
    `;
  }

  renderFields() {
    // Переопределяется в наследниках
    return '';
  }

  bindEvents() {
    if (this.refs.form) {
      this.refs.form.addEventListener('submit', this.handleSubmit.bind(this));
    }
  }

  async handleSubmit(event) {
    event.preventDefault();
    
    if (this.state.submitting) return;
    
    this.setState({ submitting: true });
    
    try {
      const formData = this.getFormData();
      
      if (this.props.validateOnSubmit) {
        const errors = this.validateForm(formData);
        if (Object.keys(errors).length > 0) {
          this.setState({ errors });
          return;
        }
      }
      
      await this.props.onSubmit(formData);
      this.setState({ errors: {}, submitting: false });
    } catch (error) {
      this.setState({ 
        errors: { _form: error.message || 'Произошла ошибка' },
        submitting: false 
      });
    }
  }

  getFormData() {
    const form = this.refs.form;
    const formData = new FormData(form);
    const data = {};
    
    for (let [key, value] of formData.entries()) {
      data[key] = value;
    }
    
    return data;
  }

  setFieldError(fieldName, error) {
    this.setState(prevState => ({
      errors: {
        ...prevState.errors,
        [fieldName]: [error]
      }
    }));
  }

  clearFieldError(fieldName) {
    this.setState(prevState => {
      const errors = { ...prevState.errors };
      delete errors[fieldName];
      return { errors };
    });
  }
}
```

## Шаблоны для API

### 4. Универсальный API клиент

```javascript
/**
 * Универсальный API клиент с обработкой ошибок и повторными попытками
 */
class ApiClient {
  constructor(baseURL, options = {}) {
    this.baseURL = baseURL;
    this.defaultOptions = {
      timeout: 10000,
      retries: 3,
      retryDelay: 1000,
      headers: {
        'Content-Type': 'application/json'
      },
      ...options
    };
  }

  /**
   * Общий метод для выполнения запросов
   * @param {string} endpoint - Конечная точка
   * @param {Object} options - Опции запроса
   * @returns {Promise}
   */
  async request(endpoint, options = {}) {
    const config = {
      ...this.defaultOptions,
      ...options,
      headers: {
        ...this.defaultOptions.headers,
        ...options.headers
      }
    };

    let lastError;
    
    for (let attempt = 1; attempt <= config.retries; attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), config.timeout);
        
        const response = await fetch(`${this.baseURL}${endpoint}`, {
          ...config,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new ApiError(
            `HTTP ошибка: ${response.status}`,
            response.status,
            await response.json().catch(() => ({}))
          );
        }
        
        return await response.json();
      } catch (error) {
        lastError = error;
        
        if (attempt === config.retries) {
          break;
        }
        
        // Ждем перед следующей попыткой
        await new Promise(resolve => 
          setTimeout(resolve, config.retryDelay * Math.pow(2, attempt - 1))
        );
      }
    }
    
    throw lastError;
  }

  /**
   * GET запрос
   * @param {string} endpoint - Конечная точка
   * @param {Object} params - Параметры запроса
   * @returns {Promise}
   */
  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url, { method: 'GET' });
  }

  /**
   * POST запрос
   * @param {string} endpoint - Конечная точка
   * @param {Object} data - Данные для отправки
   * @returns {Promise}
   */
  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  /**
   * PUT запрос
   * @param {string} endpoint - Конечная точка
   * @param {Object} data - Данные для отправки
   * @returns {Promise}
   */
  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  /**
   * DELETE запрос
   * @param {string} endpoint - Конечная точка
   * @returns {Promise}
   */
  delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

/**
 * Класс ошибки API
 */
class ApiError extends Error {
  constructor(message, status, data = {}) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.data = data;
  }
}

// Пример использования
// const api = new ApiClient('https://api.example.com');
// 
// try {
//   const users = await api.get('/users');
//   console.log(users);
// } catch (error) {
//   if (error instanceof ApiError) {
//     console.error(`API ошибка: ${error.status}`, error.data);
//   } else {
//     console.error('Сетевая ошибка:', error.message);
//   }
// }
```

### 5. Шаблон для сервиса данных

```javascript
/**
 * Базовый класс сервиса данных
 */
class DataService {
  constructor(apiClient) {
    this.api = apiClient;
    this.cache = new Map();
    this.cacheTimeout = 5 * 60 * 1000; // 5 минут
  }

  /**
   * Получает данные с кэшированием
   * @param {string} key - Ключ кэша
   * @param {Function} fetcher - Функция получения данных
   * @returns {Promise}
   */
  async getCached(key, fetcher) {
    const cached = this.cache.get(key);
    
    if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
      return cached.data;
    }
    
    const data = await fetcher();
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }

  /**
   * Очищает кэш
   * @param {string} key - Ключ для очистки (если не указан, очищается весь кэш)
   */
  clearCache(key) {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }

  /**
   * Получает список с пагинацией
   * @param {string} endpoint - Конечная точка
   * @param {Object} params - Параметры запроса
   * @returns {Promise}
   */
  async getList(endpoint, params = {}) {
    const defaultParams = { page: 1, limit: 10 };
    const queryParams = { ...defaultParams, ...params };
    
    return await this.api.get(endpoint, queryParams);
  }

  /**
   * Получает один элемент
   * @param {string} endpoint - Конечная точка
   * @param {string|number} id - ID элемента
   * @returns {Promise}
   */
  async getOne(endpoint, id) {
    return await this.api.get(`${endpoint}/${id}`);
  }

  /**
   * Создает новый элемент
   * @param {string} endpoint - Конечная точка
   * @param {Object} data - Данные для создания
   * @returns {Promise}
   */
  async create(endpoint, data) {
    const result = await this.api.post(endpoint, data);
    this.clearCache(); // Очищаем кэш после создания
    return result;
  }

  /**
   * Обновляет элемент
   * @param {string} endpoint - Конечная точка
   * @param {string|number} id - ID элемента
   * @param {Object} data - Данные для обновления
   * @returns {Promise}
   */
  async update(endpoint, id, data) {
    const result = await this.api.put(`${endpoint}/${id}`, data);
    this.clearCache(); // Очищаем кэш после обновления
    return result;
  }

  /**
   * Удаляет элемент
   * @param {string} endpoint - Конечная точка
   * @param {string|number} id - ID элемента
   * @returns {Promise}
   */
  async delete(endpoint, id) {
    const result = await this.api.delete(`${endpoint}/${id}`);
    this.clearCache(); // Очищаем кэш после удаления
    return result;
  }
}

/**
 * Конкретный сервис для работы с пользователями
 */
class UserService extends DataService {
  async getCurrentUser() {
    return await this.getCached('currentUser', () => 
      this.api.get('/users/me')
    );
  }

  async getUserProfile(userId) {
    return await this.getCached(`user_${userId}`, () =>
      this.api.get(`/users/${userId}/profile`)
    );
  }

  async updateUserProfile(userId, profileData) {
    const result = await this.api.put(`/users/${userId}/profile`, profileData);
    this.clearCache(`user_${userId}`); // Очищаем кэш конкретного пользователя
    return result;
  }
}

// Пример использования
// const api = new ApiClient('https://api.example.com');
// const userService = new UserService(api);
// 
// const currentUser = await userService.getCurrentUser();
// const userProfile = await userService.getUserProfile(123);
```

## Шаблоны для управления состоянием

### 6. Простой менеджер состояния

```javascript
/**
 * Простой менеджер состояния
 */
class StateManager {
  constructor(initialState = {}) {
    this.state = { ...initialState };
    this.subscribers = [];
    this.middleware = [];
  }

  /**
   * Возвращает текущее состояние
   * @returns {Object}
   */
  getState() {
    return { ...this.state };
  }

  /**
   * Обновляет состояние
   * @param {Object} newState - Новое состояние
   */
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.notifySubscribers();
  }

  /**
   * Подписывается на изменения состояния
   * @param {Function} callback - Функция обратного вызова
   * @returns {Function} Функция для отписки
   */
  subscribe(callback) {
    this.subscribers.push(callback);
    
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }

  /**
   * Уведомляет подписчиков об изменении состояния
   */
  notifySubscribers() {
    const state = this.getState();
    this.subscribers.forEach(callback => callback(state));
  }

  /**
   * Добавляет middleware
   * @param {Function} middleware - Middleware функция
   */
  use(middleware) {
    this.middleware.push(middleware);
  }

  /**
   * Диспетчеризирует действие
   * @param {Object} action - Действие
   */
  dispatch(action) {
    // Применяем middleware
    let dispatch = this._dispatch.bind(this);
    
    this.middleware.slice().reverse().forEach(mw => {
      dispatch = mw(this)(dispatch);
    });
    
    return dispatch(action);
  }

  _dispatch(action) {
    if (typeof action === 'function') {
      return action(this.dispatch.bind(this), this.getState.bind(this));
    }
    
    this.setState(action.payload || {});
    return action;
  }
}

/**
 * Пример использования StateManager
 */
class TodoStore extends StateManager {
  constructor() {
    super({
      todos: [],
      filter: 'all',
      loading: false
    });
  }

  addTodo(text) {
    const newTodo = {
      id: Date.now(),
      text,
      completed: false,
      createdAt: new Date()
    };
    
    this.setState({
      todos: [...this.state.todos, newTodo]
    });
  }

  toggleTodo(id) {
    const todos = this.state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    
    this.setState({ todos });
  }

  deleteTodo(id) {
    const todos = this.state.todos.filter(todo => todo.id !== id);
    this.setState({ todos });
  }

  setFilter(filter) {
    this.setState({ filter });
  }

  getFilteredTodos() {
    const { todos, filter } = this.state;
    
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  }
}

// Пример использования
// const store = new TodoStore();
// 
// store.subscribe((state) => {
//   console.log('Состояние изменилось:', state);
// });
// 
// store.addTodo('Изучить JavaScript');
// store.toggleTodo(1);
```

## Шаблоны для обработки событий

### 7. Универсальный шин событий

```javascript
/**
 * Шина событий для коммуникации между компонентами
 */
class EventBus {
  constructor() {
    this.events = {};
  }

  /**
   * Подписывается на событие
   * @param {string} event - Имя события
   * @param {Function} callback - Функция обратного вызова
   * @returns {Function} Функция для отписки
   */
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    
    this.events[event].push(callback);
    
    return () => {
      this.off(event, callback);
    };
  }

  /**
   * Отписывается от события
   * @param {string} event - Имя события
   * @param {Function} callback - Функция обратного вызова
   */
  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }

  /**
   * Вызывает событие
   * @param {string} event - Имя события
   * @param {*} data - Данные события
   */
  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }

  /**
   * Подписывается на событие и отписывается после первого вызова
   * @param {string} event - Имя события
   * @param {Function} callback - Функция обратного вызова
   */
  once(event, callback) {
    const unsubscribe = this.on(event, (data) => {
      callback(data);
      unsubscribe();
    });
  }

  /**
   * Очищает все подписки на событие
   * @param {string} event - Имя события
   */
  clear(event) {
    if (event) {
      delete this.events[event];
    } else {
      this.events = {};
    }
  }
}

/**
 * Пример использования EventBus
 */
// const eventBus = new EventBus();
// 
// // Подписка на событие
// const unsubscribe = eventBus.on('user:login', (userData) => {
//   console.log('Пользователь вошел:', userData);
// });
// 
// // Вызов события
// eventBus.emit('user:login', { id: 1, name: 'Иван' });
// 
// // Отписка
// unsubscribe();
```

## Шаблоны для работы с данными

### 8. Универсальный репозиторий данных

```javascript
/**
 * Универсальный репозиторий данных
 */
class Repository {
  constructor(adapter) {
    this.adapter = adapter;
  }

  /**
   * Находит записи по критериям
   * @param {Object} criteria - Критерии поиска
   * @returns {Promise}
   */
  async find(criteria = {}) {
    return await this.adapter.find(criteria);
  }

  /**
   * Находит первую запись по критериям
   * @param {Object} criteria - Критерии поиска
   * @returns {Promise}
   */
  async findOne(criteria = {}) {
    const results = await this.find(criteria);
    return results[0] || null;
  }

  /**
   * Находит запись по ID
   * @param {string|number} id - ID записи
   * @returns {Promise}
   */
  async findById(id) {
    return await this.findOne({ id });
  }

  /**
   * Создает новую запись
   * @param {Object} data - Данные для создания
   * @returns {Promise}
   */
  async create(data) {
    return await this.adapter.create(data);
  }

  /**
   * Обновляет запись
   * @param {string|number} id - ID записи
   * @param {Object} data - Данные для обновления
   * @returns {Promise}
   */
  async update(id, data) {
    return await this.adapter.update(id, data);
  }

  /**
   * Удаляет запись
   * @param {string|number} id - ID записи
   * @returns {Promise}
   */
  async delete(id) {
    return await this.adapter.delete(id);
  }

  /**
   * Подсчитывает количество записей по критериям
   * @param {Object} criteria - Критерии
   * @returns {Promise}
   */
  async count(criteria = {}) {
    return await this.adapter.count(criteria);
  }

  /**
   * Находит записи с пагинацией
   * @param {Object} criteria - Критерии поиска
   * @param {number} page - Номер страницы
   * @param {number} limit - Количество на странице
   * @returns {Promise}
   */
  async findWithPagination(criteria = {}, page = 1, limit = 10) {
    const offset = (page - 1) * limit;
    return await this.adapter.findWithPagination(criteria, offset, limit);
  }
}

/**
 * Адаптер для работы с localStorage
 */
class LocalStorageAdapter {
  constructor(key) {
    this.key = key;
    this.data = this.load();
  }

  load() {
    const stored = localStorage.getItem(this.key);
    return stored ? JSON.parse(stored) : [];
  }

  save() {
    localStorage.setItem(this.key, JSON.stringify(this.data));
  }

  async find(criteria = {}) {
    return this.data.filter(item => 
      Object.entries(criteria).every(([key, value]) => 
        item[key] === value
      )
    );
  }

  async create(data) {
    const newItem = {
      id: Date.now().toString(),
      ...data,
      createdAt: new Date().toISOString()
    };
    
    this.data.push(newItem);
    this.save();
    
    return newItem;
  }

  async update(id, data) {
    const index = this.data.findIndex(item => item.id === id);
    if (index !== -1) {
      this.data[index] = {
        ...this.data[index],
        ...data,
        updatedAt: new Date().toISOString()
      };
      this.save();
      return this.data[index];
    }
    return null;
  }

  async delete(id) {
    const index = this.data.findIndex(item => item.id === id);
    if (index !== -1) {
      const [deleted] = this.data.splice(index, 1);
      this.save();
      return deleted;
    }
    return null;
  }

  async count(criteria = {}) {
    return (await this.find(criteria)).length;
  }

  async findWithPagination(criteria = {}, offset = 0, limit = 10) {
    const all = await this.find(criteria);
    return all.slice(offset, offset + limit);
  }
}

// Пример использования
// const userAdapter = new LocalStorageAdapter('users');
// const userRepository = new Repository(userAdapter);
// 
// await userRepository.create({ name: 'Иван', email: 'ivan@example.com' });
// const users = await userRepository.find({});
```

## Ссылки по теме

- [[js/components]] - Компоненты JavaScript
- [[js/patterns]] - Паттерны JavaScript
- [[js/api]] - Работа с API
- [[js/state]] - Управление состоянием
- [[js/utils]] - Утилиты JavaScript