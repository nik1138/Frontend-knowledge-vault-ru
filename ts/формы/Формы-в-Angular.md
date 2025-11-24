---
aliases: ["Работа с формами в Angular с TypeScript", "Forms in Angular with TypeScript"]
tags: ["#typescript", "#angular", "#forms", "#reactive-forms", "#template-driven-forms"]
---

# Формы-в-Angular

Angular предоставляет мощную систему для работы с формами, включающую как шаблонно-ориентированные формы (template-driven forms), так и реактивные формы (reactive forms). Оба подхода поддерживают строгую типизацию TypeScript и обеспечивают эффективное управление состоянием форм, валидацию и обработку пользовательского ввода.

## Подходы к работе с формами в Angular

### Шаблонно-ориентированные формы (Template-driven Forms)

Шаблонно-ориентированные формы используют директивы в шаблоне для создания и управления формой. Они близки к нативному поведению HTML-форм.

```typescript
// contact-form.component.ts
import { Component } from '@angular/core';
import { NgForm } from '@angular/forms';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

@Component({
  selector: 'app-contact-form',
  template: `
    <form #contactForm="ngForm" (ngSubmit)="onSubmit(contactForm)">
      <div>
        <label for="name">Имя:</label>
        <input
          type="text"
          id="name"
          name="name"
          [(ngModel)]="formData.name"
          #name="ngModel"
          required
          minlength="2"
          class="form-control"
          [class.error]="name.invalid && name.touched"
        />
        <div *ngIf="name.invalid && name.touched" class="error-message">
          <span *ngIf="name.errors?.['required']">Имя обязательно</span>
          <span *ngIf="name.errors?.['minlength']">Имя должно содержать не менее 2 символов</span>
        </div>
      </div>
      
      <div>
        <label for="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          [(ngModel)]="formData.email"
          #email="ngModel"
          required
          email
          class="form-control"
          [class.error]="email.invalid && email.touched"
        />
        <div *ngIf="email.invalid && email.touched" class="error-message">
          <span *ngIf="email.errors?.['required']">Email обязателен</span>
          <span *ngIf="email.errors?.['email']">Email некорректен</span>
        </div>
      </div>
      
      <div>
        <label for="message">Сообщение:</label>
        <textarea
          id="message"
          name="message"
          [(ngModel)]="formData.message"
          #message="ngModel"
          required
          minlength="10"
          class="form-control"
          [class.error]="message.invalid && message.touched"
        ></textarea>
        <div *ngIf="message.invalid && message.touched" class="error-message">
          <span *ngIf="message.errors?.['required']">Сообщение обязательно</span>
          <span *ngIf="message.errors?.['minlength']">Сообщение должно содержать не менее 10 символов</span>
        </div>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="subscribe"
            [(ngModel)]="formData.subscribe"
          />
          Подписаться на рассылку
        </label>
      </div>
      
      <button
        type="submit"
        [disabled]="contactForm.invalid || isSubmitting"
        class="btn btn-primary"
      >
        {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
      </button>
    </form>
  `,
  styles: [`
    .form-control {
      width: 100%;
      padding: 0.5rem;
      margin-bottom: 1rem;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    
    .form-control.error {
      border-color: #dc3545;
    }
    
    .error-message {
      color: #dc3545;
      font-size: 0.875rem;
      margin-top: 0.25rem;
    }
    
    .btn {
      padding: 0.5rem 1rem;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    .btn:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
    
    .btn-primary {
      background-color: #007bff;
      color: white;
    }
  `]
})
export class ContactFormComponent {
  formData: ContactFormValues = {
    name: '',
    email: '',
    message: '',
    subscribe: false
  };
  
  isSubmitting = false;

  onSubmit(form: NgForm) {
    if (form.valid) {
      this.isSubmitting = true;
      
      // Имитация отправки данных
      setTimeout(() => {
        console.log('Данные формы:', this.formData);
        this.isSubmitting = false;
        form.resetForm();
      }, 1000);
    }
  }
}
```

Для использования шаблонно-ориентированных форм необходимо импортировать `FormsModule`:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { ContactFormComponent } from './contact-form/contact-form.component';

@NgModule({
  declarations: [
    AppComponent,
    ContactFormComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Реактивные формы (Reactive Forms)

Реактивные формы обеспечивают более предсказуемое управление состоянием формы через TypeScript-код, а не шаблон. Они обеспечивают лучшую тестируемость и контроль над формой.

```typescript
// reactive-contact-form.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators, AbstractControl } from '@angular/forms';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

@Component({
  selector: 'app-reactive-contact-form',
  template: `
    <form [formGroup]="contactForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="name">Имя:</label>
        <input
          type="text"
          id="name"
          formControlName="name"
          class="form-control"
          [class.error]="isFieldInvalid('name')"
        />
        <div *ngIf="isFieldInvalid('name')" class="error-message">
          <span *ngIf="getFieldError('name', 'required')">Имя обязательно</span>
          <span *ngIf="getFieldError('name', 'minlength')">Имя должно содержать не менее 2 символов</span>
        </div>
      </div>
      
      <div>
        <label for="email">Email:</label>
        <input
          type="email"
          id="email"
          formControlName="email"
          class="form-control"
          [class.error]="isFieldInvalid('email')"
        />
        <div *ngIf="isFieldInvalid('email')" class="error-message">
          <span *ngIf="getFieldError('email', 'required')">Email обязателен</span>
          <span *ngIf="getFieldError('email', 'email')">Email некорректен</span>
        </div>
      </div>
      
      <div>
        <label for="message">Сообщение:</label>
        <textarea
          id="message"
          formControlName="message"
          class="form-control"
          [class.error]="isFieldInvalid('message')"
        ></textarea>
        <div *ngIf="isFieldInvalid('message')" class="error-message">
          <span *ngIf="getFieldError('message', 'required')">Сообщение обязательно</span>
          <span *ngIf="getFieldError('message', 'minlength')">Сообщение должно содержать не менее 10 символов</span>
        </div>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            formControlName="subscribe"
          />
          Подписаться на рассылку
        </label>
      </div>
      
      <button
        type="submit"
        [disabled]="contactForm.invalid || isSubmitting"
        class="btn btn-primary"
      >
        {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
      </button>
    </form>
  `,
  styles: [`
    .form-control {
      width: 100%;
      padding: 0.5rem;
      margin-bottom: 1rem;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    
    .form-control.error {
      border-color: #dc3545;
    }
    
    .error-message {
      color: #dc3545;
      font-size: 0.875rem;
      margin-top: 0.25rem;
    }
    
    .btn {
      padding: 0.5rem 1rem;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    .btn:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
    
    .btn-primary {
      background-color: #007bff;
      color: white;
    }
  `]
})
export class ReactiveContactFormComponent implements OnInit {
  contactForm: FormGroup;
  isSubmitting = false;

  constructor(private fb: FormBuilder) {
    this.contactForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      message: ['', [Validators.required, Validators.minLength(10)]],
      subscribe: [false]
    });
  }

  ngOnInit() {
    // Подписка на изменения значения формы
    this.contactForm.valueChanges.subscribe(value => {
      console.log('Изменения формы:', value);
    });
    
    // Подписка на статус формы
    this.contactForm.statusChanges.subscribe(status => {
      console.log('Статус формы:', status);
    });
  }

  isFieldInvalid(fieldName: keyof ContactFormValues): boolean {
    const field = this.contactForm.get(fieldName);
    return !!(field && field.invalid && field.touched);
  }

  getFieldError(fieldName: keyof ContactFormValues, errorName: string): boolean {
    const field = this.contactForm.get(fieldName);
    return !!(field && field.errors && field.errors[errorName]);
  }

  onSubmit() {
    if (this.contactForm.valid) {
      this.isSubmitting = true;
      
      // Имитация отправки данных
      setTimeout(() => {
        console.log('Данные формы:', this.contactForm.value);
        this.isSubmitting = false;
        this.contactForm.reset();
      }, 1000);
    }
  }
}
```

Для использования реактивных форм необходимо импортировать `ReactiveFormsModule`:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ReactiveFormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { ReactiveContactFormComponent } from './reactive-contact-form/reactive-contact-form.component';

@NgModule({
  declarations: [
    AppComponent,
    ReactiveContactFormComponent
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Пользовательские валидаторы

Angular позволяет создавать пользовательские валидаторы для специфических требований:

```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Валидатор для проверки возраста
export function ageValidator(minAge: number, maxAge: number): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    if (!control.value) {
      return null; // Не валидировать пустое значение
    }
    
    const age = parseInt(control.value, 10);
    if (isNaN(age)) {
      return { invalidAge: true };
    }
    
    if (age < minAge || age > maxAge) {
      return { ageOutOfRange: { min: minAge, max: maxAge, actual: age } };
    }
    
    return null;
  };
}

// Валидатор для проверки совпадения паролей
export function passwordMatchValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const password = control.get('password');
    const confirmPassword = control.get('confirmPassword');
    
    if (!password || !confirmPassword) {
      return null;
    }
    
    if (password.value !== confirmPassword.value) {
      confirmPassword.setErrors({ passwordMismatch: true });
      return { passwordMismatch: true };
    } else {
      // Очистить ошибку, если пароли совпадают
      if (confirmPassword.errors && confirmPassword.errors['passwordMismatch']) {
        delete confirmPassword.errors['passwordMismatch'];
        if (!Object.keys(confirmPassword.errors).length) {
          confirmPassword.setErrors(null);
        }
      }
      return null;
    }
  };
}
```

Пример использования пользовательского валидатора:

```typescript
// registration-form.component.ts
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { ageValidator } from './validators/custom-validators';

interface RegistrationFormValues {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age: number;
}

@Component({
  selector: 'app-registration-form',
  template: `
    <form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="username">Имя пользователя:</label>
        <input
          type="text"
          id="username"
          formControlName="username"
          class="form-control"
        />
      </div>
      
      <div>
        <label for="email">Email:</label>
        <input
          type="email"
          id="email"
          formControlName="email"
          class="form-control"
        />
      </div>
      
      <div>
        <label for="password">Пароль:</label>
        <input
          type="password"
          id="password"
          formControlName="password"
          class="form-control"
        />
      </div>
      
      <div>
        <label for="confirmPassword">Подтверждение пароля:</label>
        <input
          type="password"
          id="confirmPassword"
          formControlName="confirmPassword"
          class="form-control"
          [class.error]="isFieldInvalid('confirmPassword')"
        />
        <div *ngIf="isFieldInvalid('confirmPassword')" class="error-message">
          <span *ngIf="getFieldError('confirmPassword', 'passwordMismatch')">
            Пароли не совпадают
          </span>
        </div>
      </div>
      
      <div>
        <label for="age">Возраст:</label>
        <input
          type="number"
          id="age"
          formControlName="age"
          class="form-control"
          [class.error]="isFieldInvalid('age')"
        />
        <div *ngIf="isFieldInvalid('age')" class="error-message">
          <span *ngIf="getFieldError('age', 'ageOutOfRange')">
            Возраст должен быть от {{ getErrorParam('age', 'min') }} до {{ getErrorParam('age', 'max') }}
          </span>
        </div>
      </div>
      
      <button type="submit" [disabled]="registrationForm.invalid || isSubmitting">
        {{ isSubmitting ? 'Регистрация...' : 'Зарегистрироваться' }}
      </button>
    </form>
  `
})
export class RegistrationFormComponent {
  registrationForm: FormGroup;
  isSubmitting = false;

  constructor(private fb: FormBuilder) {
    this.registrationForm = this.fb.group({
      username: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]],
      confirmPassword: ['', [Validators.required]],
      age: ['', [Validators.required, ageValidator(13, 120)]]
    }, {
      validators: [this.passwordMatchValidator()]
    });
  }

  passwordMatchValidator() {
    return (form: AbstractControl): ValidationErrors | null => {
      const password = form.get('password');
      const confirmPassword = form.get('confirmPassword');
      
      if (!password || !confirmPassword) {
        return null;
      }
      
      if (password.value !== confirmPassword.value) {
        confirmPassword.setErrors({ passwordMismatch: true });
        return { passwordMismatch: true };
      } else {
        if (confirmPassword.errors && confirmPassword.errors['passwordMismatch']) {
          delete confirmPassword.errors['passwordMismatch'];
          if (!Object.keys(confirmPassword.errors).length) {
            confirmPassword.setErrors(null);
          }
        }
        return null;
      }
    };
  }

  isFieldInvalid(fieldName: keyof RegistrationFormValues): boolean {
    const field = this.registrationForm.get(fieldName);
    return !!(field && field.invalid && field.touched);
  }

  getFieldError(fieldName: keyof RegistrationFormValues, errorName: string): boolean {
    const field = this.registrationForm.get(fieldName);
    return !!(field && field.errors && field.errors[errorName]);
  }

  getErrorParam(fieldName: keyof RegistrationFormValues, paramName: string): any {
    const field = this.registrationForm.get(fieldName);
    if (field && field.errors) {
      return field.errors[paramName];
    }
    return null;
  }

  onSubmit() {
    if (this.registrationForm.valid) {
      this.isSubmitting = true;
      
      // Имитация отправки данных
      setTimeout(() => {
        console.log('Данные регистрации:', this.registrationForm.value);
        this.isSubmitting = false;
        this.registrationForm.reset();
      }, 1000);
    }
  }
}
```

## Асинхронные валидаторы

Для проверки уникальности данных (например, имени пользователя или email) можно использовать асинхронные валидаторы:

```typescript
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of, timer } from 'rxjs';
import { map, switchMap, catchError } from 'rxjs/operators';

// Асинхронный валидатор для проверки уникальности email
export function uniqueEmailValidator(): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null); // Не валидировать пустое значение
    }

    // Имитация HTTP-запроса для проверки уникальности
    return timer(1000).pipe(
      switchMap(() => {
        // Здесь должен быть реальный HTTP-запрос
        // Например: return this.userService.checkEmail(control.value);
        
        // Имитация проверки - считаем, что email "admin@example.com" уже занят
        const isEmailTaken = control.value === 'admin@example.com';
        
        return isEmailTaken ? of({ emailTaken: true }) : of(null);
      }),
      catchError(() => of(null)) // В случае ошибки запроса не считаем это ошибкой валидации
    );
  };
}
```

## Лучшие практики

- Используйте реактивные формы для сложных сценариев и форм с динамическим поведением
- Используйте шаблонно-ориентированные формы для простых форм с минимальной логикой
- Применяйте строгую типизацию для значений форм
- Создавайте переиспользуемые пользовательские валидаторы
- Используйте асинхронные валидаторы для проверки уникальности данных
- Обрабатывайте различные состояния формы (загрузка, ошибка)
- Обеспечьте доступность форм (accessibility)
- Используйте `FormBuilder` для более краткой и читаемой инициализации форм

## Связанные темы

- [[Управление-формами]]
- [[Валидация-форм]]
- [[Формы-в-React]]
- [[Формы-в-Vue]]
- [[Angular-с-typescript]]