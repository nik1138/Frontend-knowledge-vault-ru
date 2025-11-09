# Фреймворки и библиотеки JavaScript

## Введение

Экосистема JavaScript предлагает огромное количество фреймворков и библиотек, которые помогают разработчикам создавать современные веб-приложения. От полноценных фреймворков до специализированных библиотек, каждый инструмент решает определенные задачи и имеет свои преимущества и недостатки.

## Основные категории фреймворков и библиотек

### 1. Фронтенд фреймворки

#### React

React - библиотека для создания пользовательских интерфейсов, разработанная Facebook. Основана на компонентном подходе и виртуальном DOM.

```jsx
// Основные концепции React

// 1. Функциональные компоненты с хуками
import React, { useState, useEffect, useContext, useMemo, useCallback } from 'react';
import { UserContext } from './UserContext';

// Простой функциональный компонент
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}

// Компонент с использованием нескольких хуков
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const { currentUser } = useContext(UserContext);
  
  // useEffect для загрузки данных
  useEffect(() => {
    let isMounted = true;
    
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        const userData = await response.json();
        
        if (isMounted) {
          setUser(userData);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };
    
    fetchUser();
    
    // Cleanup функция
    return () => {
      isMounted = false;
    };
  }, [userId]);
  
  // useMemo для мемоизации вычислений
  const userDisplayName = useMemo(() => {
    if (!user) return '';
    return `${user.firstName} ${user.lastName}`.trim() || user.username;
  }, [user]);
  
  // useCallback для мемоизации функций
  const handleUpdateProfile = useCallback(async (updates) => {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updates),
      });
      
      if (!response.ok) {
        throw new Error('Failed to update profile');
      }
      
      const updatedUser = await response.json();
      setUser(updatedUser);
    } catch (err) {
      setError(err.message);
    }
  }, [userId]);
  
  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error}</div>;
  if (!user) return <div>Пользователь не найден</div>;
  
  return (
    <div>
      <h2>{userDisplayName}</h2>
      <p>Email: {user.email}</p>
      <p>Роль: {user.role}</p>
      {currentUser?.id === userId && (
        <button onClick={() => handleUpdateProfile({ lastSeen: new Date() })}>
          Обновить профиль
        </button>
      )}
    </div>
  );
}

// 2. Компоненты высшего порядка (HOC)
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { currentUser } = useContext(UserContext);
    
    if (!currentUser) {
      return <div>Пожалуйста, войдите в систему</div>;
    }
    
    return <WrappedComponent {...props} currentUser={currentUser} />;
  };
}

// 3. Render props
function DataFetcher({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return children({ data, loading, error });
}

// Использование render props
function UserList() {
  return (
    <DataFetcher url="/api/users">
      {({ data, loading, error }) => {
        if (loading) return <div>Загрузка пользователей...</div>;
        if (error) return <div>Ошибка: {error}</div>;
        if (!data) return <div>Нет данных</div>;
        
        return (
          <ul>
            {data.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        );
      }}
    </DataFetcher>
  );
}

// 4. Контекст для управления состоянием
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button 
      onClick={toggleTheme}
      style={{
        backgroundColor: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#000' : '#fff',
      }}
    >
      Переключить тему
    </button>
  );
}

// 5. Порталы для рендеринга вне иерархии DOM
import { createPortal } from 'react-dom';

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  );
}

// 6. Рефы для доступа к DOM элементам
function FocusInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Фокус на поле</button>
    </div>
  );
}

// 7. Оптимизация производительности
const ExpensiveComponent = React.memo(({ data }) => {
  // Дорогой рендеринг
  const expensiveValue = useMemo(() => {
    return data.reduce((acc, item) => acc + item.value, 0);
  }, [data]);
  
  return <div>Результат: {expensiveValue}</div>;
});

// Условный рендеринг
function ConditionalRender() {
  const [showDetails, setShowDetails] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowDetails(!showDetails)}>
        {showDetails ? 'Скрыть' : 'Показать'} детали
      </button>
      {showDetails && <ExpensiveComponent />}
    </div>
  );
}
```

#### Vue.js

Vue.js - прогрессивный фреймворк для создания пользовательских интерфейсов с интуитивно понятным API.

```vue
<!-- Основные концепции Vue.js -->

<!-- 1. Компонент с Composition API -->
<template>
  <div class="user-profile">
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error" class="error">Ошибка: {{ error }}</div>
    <div v-else-if="user">
      <h2>{{ userDisplayName }}</h2>
      <p>Email: {{ user.email }}</p>
      <p>Роль: {{ user.role }}</p>
      <button 
        v-if="currentUser?.id === userId" 
        @click="updateProfile"
        :disabled="updating"
      >
        {{ updating ? 'Обновление...' : 'Обновить профиль' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue';
import { useUserStore } from '@/stores/user';

// Реактивные данные
const props = defineProps({
  userId: {
    type: [String, Number],
    required: true
  }
});

const user = ref(null);
const loading = ref(true);
const error = ref(null);
const updating = ref(false);

// Хранилище Pinia
const userStore = useUserStore();
const currentUser = computed(() => userStore.currentUser);

// Вычисляемые свойства
const userDisplayName = computed(() => {
  if (!user.value) return '';
  return `${user.value.firstName} ${user.value.lastName}`.trim() || user.value.username;
});

// Методы
const fetchUser = async () => {
  try {
    loading.value = true;
    error.value = null;
    
    const response = await fetch(`/api/users/${props.userId}`);
    if (!response.ok) {
      throw new Error('Failed to fetch user');
    }
    
    user.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const updateProfile = async () => {
  try {
    updating.value = true;
    
    const response = await fetch(`/api/users/${props.userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ lastSeen: new Date() }),
    });
    
    if (!response.ok) {
      throw new Error('Failed to update profile');
    }
    
    user.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    updating.value = false;
  }
};

// Наблюдатели
watch(() => props.userId, (newId, oldId) => {
  if (newId !== oldId) {
    fetchUser();
  }
});

// Хуки жизненного цикла
onMounted(() => {
  fetchUser();
});

onUnmounted(() => {
  // Очистка ресурсов
});
</script>

<style scoped>
.user-profile {
  padding: 20px;
}

.error {
  color: red;
  font-weight: bold;
}
</style>

<!-- 2. Компонент с Options API -->
<template>
  <div class="counter">
    <p>Счетчик: {{ count }}</p>
    <p>Удвоенное значение: {{ doubledCount }}</p>
    <button @click="increment">Увеличить</button>
    <button @click="decrement">Уменьшить</button>
  </div>
</template>

<script>
export default {
  name: 'Counter',
  data() {
    return {
      count: 0
    };
  },
  computed: {
    doubledCount() {
      return this.count * 2;
    }
  },
  methods: {
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    }
  },
  watch: {
    count(newVal, oldVal) {
      console.log(`Счетчик изменился с ${oldVal} на ${newVal}`);
    }
  },
  mounted() {
    console.log('Компонент mounted');
  }
};
</script>

<!-- 3. Слоты для компонентов -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">
        <h3>Заголовок по умолчанию</h3>
      </slot>
    </div>
    <div class="card-body">
      <slot></slot>
    </div>
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Card'
};
</script>

<!-- Использование компонента с слотами -->
<Card>
  <template #header>
    <h2>Мой заголовок</h2>
  </template>
  
  <p>Основной контент карточки</p>
  
  <template #footer>
    <button>Действие</button>
  </template>
</Card>

<!-- 4. Директивы Vue -->
<template>
  <div>
    <!-- Условный рендеринг -->
    <div v-if="isVisible">Видимый элемент</div>
    <div v-show="isVisible">Элемент с display: none</div>
    
    <!-- Списки -->
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
    
    <!-- Атрибуты и классы -->
    <div 
      :class="{ active: isActive, disabled: isDisabled }"
      :style="{ color: textColor }"
    >
      Динамический элемент
    </div>
    
    <!-- События -->
    <button @click="handleClick" @mouseover="handleMouseOver">
      Кнопка
    </button>
    
    <!-- Двустороннее связывание -->
    <input v-model="message" placeholder="Введите текст">
    <p>Сообщение: {{ message }}</p>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';

const isVisible = ref(true);
const isActive = ref(false);
const isDisabled = ref(false);
const textColor = ref('blue');
const message = ref('');

const items = reactive([
  { id: 1, name: 'Элемент 1' },
  { id: 2, name: 'Элемент 2' },
  { id: 3, name: 'Элемент 3' }
]);

const handleClick = () => {
  console.log('Кнопка нажата');
};

const handleMouseOver = () => {
  console.log('Мышь над кнопкой');
};
</script>
```

#### Angular

Angular - полноценный фреймворк от Google с мощной системой внедрения зависимостей и строгой типизацией.

```typescript
// Основные концепции Angular

// 1. Компонент
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
import { UserService } from './user.service';
import { User } from './user.model';

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="loading">Загрузка...</div>
    <div *ngIf="error" class="error">Ошибка: {{ error }}</div>
    <div *ngIf="user && !loading && !error">
      <h2>{{ userDisplayName }}</h2>
      <p>Email: {{ user.email }}</p>
      <p>Роль: {{ user.role }}</p>
      <button 
        *ngIf="currentUser?.id === user.id" 
        (click)="updateProfile()"
        [disabled]="updating">
        {{ updating ? 'Обновление...' : 'Обновить профиль' }}
      </button>
    </div>
  `,
  styles: [`
    .error {
      color: red;
      font-weight: bold;
    }
  `]
})
export class UserProfileComponent implements OnInit {
  @Input() userId!: number;
  @Output() userUpdated = new EventEmitter<User>();
  
  user: User | null = null;
  loading = true;
  error: string | null = null;
  updating = false;
  currentUser: User | null = null;
  
  constructor(private userService: UserService) {}
  
  ngOnInit(): void {
    this.fetchUser();
    this.currentUser = this.userService.getCurrentUser();
  }
  
  get userDisplayName(): string {
    if (!this.user) return '';
    return `${this.user.firstName} ${this.user.lastName}`.trim() || this.user.username;
  }
  
  async fetchUser(): Promise<void> {
    try {
      this.loading = true;
      this.error = null;
      this.user = await this.userService.getUser(this.userId);
    } catch (err) {
      this.error = (err as Error).message;
    } finally {
      this.loading = false;
    }
  }
  
  async updateProfile(): Promise<void> {
    if (!this.user) return;
    
    try {
      this.updating = true;
      const updatedUser = await this.userService.updateUser(this.user.id, {
        lastSeen: new Date()
      });
      this.user = updatedUser;
      this.userUpdated.emit(updatedUser);
    } catch (err) {
      this.error = (err as Error).message;
    } finally {
      this.updating = false;
    }
  }
}

// 2. Сервис с внедрением зависимостей
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

export interface User {
  id: number;
  username: string;
  email: string;
  firstName?: string;
  lastName?: string;
  role: string;
  lastSeen?: Date;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private currentUser: User | null = null;
  
  constructor(private http: HttpClient) {}
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      catchError(this.handleError<User>('getUser'))
    );
  }
  
  updateUser(id: number, updates: Partial<User>): Observable<User> {
    return this.http.put<User>(`/api/users/${id}`, updates).pipe(
      catchError(this.handleError<User>('updateUser'))
    );
  }
  
  getCurrentUser(): User | null {
    return this.currentUser;
  }
  
  private handleError<T>(operation = 'operation', result?: T) {
    return (error: any): Observable<T> => {
      console.error(`${operation} failed: ${error.message}`);
      return of(result as T);
    };
  }
}

// 3. Директива
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input('appHighlight') highlightColor = 'yellow';
  
  constructor(private el: ElementRef) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight(this.highlightColor);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight('');
  }
  
  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}

// 4. Пайп
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate'
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 20): string {
    if (!value) return '';
    return value.length > limit ? value.substring(0, limit) + '...' : value;
  }
}

// 5. Модуль
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { HighlightDirective } from './directives/highlight.directive';
import { TruncatePipe } from './pipes/truncate.pipe';

@NgModule({
  declarations: [
    AppComponent,
    UserProfileComponent,
    HighlightDirective,
    TruncatePipe
  ],
  imports: [
    BrowserModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

// 6. Реактивные формы
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="username">Имя пользователя:</label>
        <input id="username" formControlName="username">
        <div *ngIf="userForm.get('username')?.invalid && userForm.get('username')?.touched">
          Имя пользователя обязательно
        </div>
      </div>
      
      <div>
        <label for="email">Email:</label>
        <input id="email" formControlName="email">
        <div *ngIf="userForm.get('email')?.invalid && userForm.get('email')?.touched">
          Введите корректный email
        </div>
      </div>
      
      <button type="submit" [disabled]="userForm.invalid">
        Сохранить
      </button>
    </form>
  `
})
export class UserFormComponent {
  userForm: FormGroup;
  
  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]]
    });
  }
  
  onSubmit(): void {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

### 2. Бэкенд фреймворки

#### Express.js

Express.js - минималистичный веб-фреймворк для Node.js.

```javascript
// Основные концепции Express.js

const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoose = require('mongoose');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const app = express();

// Middleware
app.use(helmet()); // Безопасность заголовков
app.use(cors()); // CORS
app.use(express.json()); // Парсинг JSON
app.use(express.urlencoded({ extended: true })); // Парсинг URL-encoded данных

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100 // ограничение на 100 запросов
});
app.use('/api/', limiter);

// Модели Mongoose
const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, default: 'user' },
  createdAt: { type: Date, default: Date.now }
});

userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

const User = mongoose.model('User', userSchema);

// Middleware для аутентификации
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.sendStatus(401);
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.sendStatus(403);
    }
    
    req.user = user;
    next();
  });
};

// Middleware для авторизации по роли
const requireRole = (role) => {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

// Маршруты

// Регистрация пользователя
app.post('/api/auth/register', async (req, res) => {
  try {
    const { username, email, password } = req.body;
    
    // Проверка существующего пользователя
    const existingUser = await User.findOne({
      $or: [{ username }, { email }]
    });
    
    if (existingUser) {
      return res.status(400).json({ error: 'User already exists' });
    }
    
    // Создание нового пользователя
    const user = new User({ username, email, password });
    await user.save();
    
    // Генерация JWT токена
    const token = jwt.sign(
      { userId: user._id, username: user.username },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.status(201).json({
      token,
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Вход пользователя
app.post('/api/auth/login', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    // Поиск пользователя
    const user = await User.findOne({ username });
    if (!user) {
      return res.status(400).json({ error: 'Invalid credentials' });
    }
    
    // Проверка пароля
    const isMatch = await user.comparePassword(password);
    if (!isMatch) {
      return res.status(400).json({ error: 'Invalid credentials' });
    }
    
    // Генерация JWT токена
    const token = jwt.sign(
      { userId: user._id, username: user.username, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({
      token,
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Получение профиля пользователя
app.get('/api/users/profile', authenticateToken, async (req, res) => {
  try {
    const user = await User.findById(req.user.userId).select('-password');
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Обновление профиля пользователя
app.put('/api/users/profile', authenticateToken, async (req, res) => {
  try {
    const updates = req.body;
    
    // Удаление полей, которые нельзя обновить
    delete updates.password;
    delete updates.role;
    delete updates._id;
    
    const user = await User.findByIdAndUpdate(
      req.user.userId,
      updates,
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Административные маршруты
app.get('/api/admin/users', authenticateToken, requireRole('admin'), async (req, res) => {
  try {
    const users = await User.find().select('-password');
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Обработка ошибок
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

// Обработка 404
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// Подключение к MongoDB
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => {
  console.log('Connected to MongoDB');
  app.listen(3000, () => {
    console.log('Server running on port 3000');
  });
}).catch(err => {
  console.error('MongoDB connection error:', err);
});
```

#### NestJS

NestJS - прогрессивный фреймворк для создания эффективных и масштабируемых серверных приложений на Node.js.

```typescript
// Основные концепции NestJS

// 1. Модуль
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UserModule } from './user/user.module';
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [
    MongooseModule.forRoot(process.env.MONGODB_URI),
    UserModule,
    AuthModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}

// 2. Контроллер
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  async create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @UseGuards(JwtAuthGuard)
  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.userService.findOne(id);
  }

  @UseGuards(JwtAuthGuard)
  @Get()
  async findAll() {
    return this.userService.findAll();
  }
}

// 3. Сервис
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User, UserDocument } from './schemas/user.schema';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UserService {
  constructor(
    @InjectModel(User.name) private userModel: Model<UserDocument>,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const createdUser = new this.userModel(createUserDto);
    return createdUser.save();
  }

  async findOne(id: string): Promise<User> {
    return this.userModel.findById(id).exec();
  }

  async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }

  async findByUsername(username: string): Promise<User> {
    return this.userModel.findOne({ username }).exec();
  }
}

// 4. Схема Mongoose
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

export type UserDocument = User & Document;

@Schema()
export class User {
  @Prop({ required: true, unique: true })
  username: string;

  @Prop({ required: true, unique: true })
  email: string;

  @Prop({ required: true })
  password: string;

  @Prop({ default: 'user' })
  role: string;

  @Prop({ default: Date.now })
  createdAt: Date;
}

export const UserSchema = SchemaFactory.createForClass(User);

// 5. DTO (Data Transfer Object)
import { IsString, IsEmail, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  username: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}

// 6. Guard
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      return false;
    }
    
    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env.JWT_SECRET,
      });
      request.user = payload;
    } catch {
      return false;
    }
    
    return true;
  }

  private extractTokenFromHeader(request: any): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// 7. Middleware
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Request: ${req.method} ${req.originalUrl}`);
    next();
  }
}

// 8. Pipe
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  async transform(value: any, metadata: ArgumentMetadata) {
    const { metatype } = metadata;
    
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    
    // Здесь будет логика валидации
    // Для примера просто возвращаем значение
    
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}

// 9. Interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}

// 10. Exception Filter
import { ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

### 3. Специализированные библиотеки

#### Lodash

Lodash - библиотека утилит для работы с массивами, объектами, функциями и другими типами данных.

```javascript
// Основные функции Lodash

const _ = require('lodash');

// Работа с массивами
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Фильтрация и сортировка
const evenNumbers = _.filter(numbers, n => n % 2 === 0);
const sortedDesc = _.orderBy(numbers, null, 'desc');
const chunks = _.chunk(numbers, 3); // [[1,2,3], [4,5,6], [7,8,9], [10]]

// Уникальные значения
const duplicates = [1, 2, 2, 3, 3, 4, 5, 5];
const unique = _.uniq(duplicates); // [1, 2, 3, 4, 5]

// Работа с объектами
const user = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    zip: '10001'
  },
  hobbies: ['reading', 'swimming']
};

// Глубокое клонирование
const userCopy = _.cloneDeep(user);

// Получение значений с дефолтными значениями
const email = _.get(user, 'contact.email', 'no-email@example.com');
const city = _.get(user, 'address.city', 'Unknown');

// Установка значений
_.set(user, 'address.country', 'USA');
_.set(user, 'preferences.theme', 'dark');

// Проверка наличия свойств
const hasAddress = _.has(user, 'address');
const hasPhone = _.has(user, 'contact.phone');

// Работа с функциями
// Мемоизация
const expensiveFunction = _.memoize((n) => {
  console.log('Вычисление для:', n);
  return n * n;
});

console.log(expensiveFunction(5)); // Вычисление для: 5, 25
console.log(expensiveFunction(5)); // 25 (без повторного вычисления)

// Debounce и throttle
const debouncedSearch = _.debounce((query) => {
  console.log('Поиск:', query);
}, 300);

const throttledScroll = _.throttle(() => {
  console.log('Прокрутка');
}, 100);

// Композиция функций
const add = (x) => x + 1;
const multiply = (x) => x * 2;
const subtract = (x) => x - 3;

const composedFunction = _.flow([add, multiply, subtract]);
console.log(composedFunction(5)); // ((5 + 1) * 2) - 3 = 9

// Работа с коллекциями
const users = [
  { name: 'John', age: 30, active: true },
  { name: 'Jane', age: 25, active: false },
  { name: 'Bob', age: 35, active: true }
];

// Группировка
const groupedByActive = _.groupBy(users, 'active');
// { true: [...], false: [...] }

// Индексация
const indexedByName = _.keyBy(users, 'name');
// { 'John': {...}, 'Jane': {...}, 'Bob': {...} }

// Плакирование
const names = _.map(users, 'name'); // ['John', 'Jane', 'Bob']
const activeUsers = _.filter(users, 'active');

// Работа со строками
const text = '  Hello World  ';
const trimmed = _.trim(text); // 'Hello World'
const capitalized = _.capitalize(trimmed); // 'Hello world'
const kebabCase = _.kebabCase('Hello World Example'); // 'hello-world-example'

// Работа с датами
const now = new Date();
const oneWeekAgo = _.subtract(now, 7, 'days');
const isAfter = _.isAfter(now, oneWeekAgo);

// Утилиты
// Генерация уникальных ID
const id1 = _.uniqueId('user_'); // 'user_1'
const id2 = _.uniqueId('user_'); // 'user_2'

// Проверки типов
_.isArray([1, 2, 3]); // true
_.isObject({}); // true
_.isFunction(() => {}); // true
_.isString('hello'); // true
_.isNumber(42); // true
_.isBoolean(true); // true
_.isNull(null); // true
_.isUndefined(undefined); // true

// Утилиты для разработки
// noop - пустая функция
const callback = _.noop; // () => {}

// identity - возвращает переданный аргумент
const value = _.identity('test'); // 'test'

// constant - возвращает функцию, которая всегда возвращает одно значение
const getDefaultValue = _.constant('default');
console.log(getDefaultValue()); // 'default'
```

#### RxJS

RxJS - библиотека для реактивного программирования с использованием Observable.

```javascript
// Основные концепции RxJS

import { Observable, of, from, interval, timer, Subject, BehaviorSubject } from 'rxjs';
import { map, filter, switchMap, catchError, debounceTime, distinctUntilChanged, takeUntil } from 'rxjs/operators';
import { ajax } from 'rxjs/ajax';

// Создание Observable
// Из значения
const singleValue$ = of(42);

// Из массива
const arrayValues$ = from([1, 2, 3, 4, 5]);

// Из Promise
const promiseValue$ = from(fetch('/api/data').then(response => response.json()));

// Из события
const click$ = fromEvent(document, 'click');

// Интервал
const interval$ = interval(1000); // каждую секунду
const timer$ = timer(3000, 1000); // через 3 секунды, затем каждую секунду

// Подписка на Observable
singleValue$.subscribe({
  next: value => console.log('Получено значение:', value),
  error: error => console.error('Ошибка:', error),
  complete: () => console.log('Завершено')
});

// Операторы трансформации
const numbers$ = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

// Map - преобразование значений
const doubled$ = numbers$.pipe(
  map(x => x * 2)
);

// Filter - фильтрация значений
const even$ = numbers$.pipe(
  filter(x => x % 2 === 0)
);

// Комбинация операторов
const processed$ = numbers$.pipe(
  filter(x => x > 5),
  map(x => x * x),
  filter(x => x < 50)
);

processed$.subscribe(value => console.log('Обработанное значение:', value));

// Обработка ошибок
const apiCall$ = ajax.getJSON('/api/users').pipe(
  catchError(error => {
    console.error('API ошибка:', error);
    return of([]); // возвращаем пустой массив в случае ошибки
  })
);

// Debounce и throttle для пользовательского ввода
const searchInput = document.getElementById('search');
const searchInput$ = fromEvent(searchInput, 'input').pipe(
  map(event => event.target.value),
  debounceTime(300), // ждем 300ms после последнего ввода
  distinctUntilChanged(), // игнорируем повторяющиеся значения
  filter(query => query.length > 2) // только если длина больше 2
);

const searchResults$ = searchInput$.pipe(
  switchMap(query => 
    ajax.getJSON(`/api/search?q=${encodeURIComponent(query)}`).pipe(
      catchError(error => {
        console.error('Ошибка поиска:', error);
        return of([]);
      })
    )
  )
);

searchResults$.subscribe(results => {
  console.log('Результаты поиска:', results);
  // Обновление UI
});

// Subject и BehaviorSubject
// Subject - позволяет вручную отправлять значения
const subject = new Subject();

subject.subscribe(value => console.log('Subject 1:', value));
subject.next(1);
subject.next(2);

const subject2 = subject.subscribe(value => console.log('Subject 2:', value));
subject.next(3); // оба подписчика получат значение

// BehaviorSubject - хранит последнее значение
const behaviorSubject = new BehaviorSubject(0);

behaviorSubject.subscribe(value => console.log('Behavior 1:', value)); // получит 0
behaviorSubject.next(1); // оба получат 1
behaviorSubject.next(2); // оба получат 2

const behavior2 = behaviorSubject.subscribe(value => console.log('Behavior 2:', value)); // получит 2 (последнее значение)

// Пример: управление состоянием приложения
class AppState {
  private stateSubject = new BehaviorSubject({
    user: null,
    loading: false,
    error: null
  });
  
  public state$ = this.stateSubject.asObservable();
  
  setLoading(loading: boolean) {
    const currentState = this.stateSubject.value;
    this.stateSubject.next({ ...currentState, loading });
  }
  
  setUser(user: any) {
    const currentState = this.stateSubject.value;
    this.stateSubject.next({ ...currentState, user, loading: false, error: null });
  }
  
  setError(error: string) {
    const currentState = this.stateSubject.value;
    this.stateSubject.next({ ...currentState, error, loading: false });
  }
}

const appState = new AppState();

// Подписка на изменения состояния
appState.state$.subscribe(state => {
  console.log('Новое состояние:', state);
  // Обновление UI
});

// Использование состояния
appState.setLoading(true);
// ... асинхронная операция
appState.setUser({ name: 'John', email: 'john@example.com' });

// Операторы комбинации
import { combineLatest, forkJoin, merge } from 'rxjs';

// combineLatest - комбинирует последние значения из нескольких Observable
const firstName$ = of('John');
const lastName$ = of('Doe');
const age$ = of(30);

const fullName$ = combineLatest([firstName$, lastName$, age$]).pipe(
  map(([first, last, age]) => `${first} ${last}, ${age} лет`)
);

fullName$.subscribe(name => console.log('Полное имя:', name));

// forkJoin - ждет завершения всех Observable
const user$ = ajax.getJSON('/api/user');
const posts$ = ajax.getJSON('/api/posts');
const comments$ = ajax.getJSON('/api/comments');

const allData$ = forkJoin([user$, posts$, comments$]).pipe(
  map(([user, posts, comments]) => ({
    user,
    posts: posts.slice(0, 5),
    comments: comments.slice(0, 10)
  }))
);

allData$.subscribe(data => {
  console.log('Все данные загружены:', data);
});

// merge - объединяет несколько Observable
const clicks$ = fromEvent(document, 'click');
const keyups$ = fromEvent(document, 'keyup');

const events$ = merge(clicks$, keyups$).pipe(
  map(event => event.type)
);

events$.subscribe(eventType => {
  console.log('Событие:', eventType);
});

// takeUntil для управления подписками
import { takeUntil } from 'rxjs/operators';

class Component {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // Подписка с автоматической отпиской при уничтожении компонента
    interval$.pipe(
      takeUntil(this.destroy$)
    ).subscribe(value => {
      console.log('Интервал:', value);
    });
    
    searchResults$.pipe(
      takeUntil(this.destroy$)
    ).subscribe(results => {
      console.log('Результаты поиска:', results);
    });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## Заключение

Фреймворки и библиотеки JavaScript предоставляют мощные инструменты для создания современных веб-приложений. Каждый инструмент имеет свои сильные и слабые стороны:

1. **Фронтенд фреймворки**:
   - React - гибкость и богатая экосистема
   - Vue.js - простота и постепенное внедрение
   - Angular - полноценный фреймворк с строгой архитектурой

2. **Бэкенд фреймворки**:
   - Express.js - минимализм и гибкость
   - NestJS - структурированный подход с TypeScript

3. **Специализированные библиотеки**:
   - Lodash - утилиты для повседневных задач
   - RxJS - реактивное программирование

Выбор инструментов зависит от требований проекта, команды разработчиков и долгосрочных целей. Важно понимать архитектурные принципы каждого инструмента и уметь применять их эффективно.

#javascript #frameworks #libraries #react #vue #angular #express #nestjs #lodash #rxjs #frontend #backend #web-development