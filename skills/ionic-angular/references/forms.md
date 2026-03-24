# Ionic Angular Forms

Using Angular reactive forms and template-driven forms with Ionic input components.

## Reactive Forms

### Setup

**Standalone components** — import `ReactiveFormsModule` directly in the component:

```typescript
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import {
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonItem,
  IonInput,
  IonButton,
  IonNote,
} from '@ionic/angular/standalone';

@Component({
  selector: 'app-register',
  templateUrl: './register.page.html',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonList,
    IonItem,
    IonInput,
    IonButton,
    IonNote,
  ],
})
export class RegisterPage {
  // ...
}
```

**NgModule components** — import `ReactiveFormsModule` in the page module:

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule } from '@angular/forms';
import { IonicModule } from '@ionic/angular';
import { RegisterPage } from './register.page';
import { RegisterPageRoutingModule } from './register-routing.module';

@NgModule({
  imports: [CommonModule, ReactiveFormsModule, IonicModule, RegisterPageRoutingModule],
  declarations: [RegisterPage],
})
export class RegisterPageModule {}
```

### Form Definition

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({ /* ... */ })
export class RegisterPage {
  private fb = inject(FormBuilder);

  registerForm: FormGroup = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
  });

  onSubmit() {
    if (this.registerForm.valid) {
      const formData = this.registerForm.value;
      // Process form data
    }
  }
}
```

### Template with Ionic Components

```html
<ion-header>
  <ion-toolbar>
    <ion-title>Register</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
  <form [formGroup]="registerForm" (ngSubmit)="onSubmit()">
    <ion-list>
      <ion-item>
        <ion-input
          label="Name"
          labelPlacement="floating"
          formControlName="name"
          type="text"
          errorText="Name is required (min 2 characters)."
        ></ion-input>
      </ion-item>

      <ion-item>
        <ion-input
          label="Email"
          labelPlacement="floating"
          formControlName="email"
          type="email"
          errorText="Please enter a valid email."
        ></ion-input>
      </ion-item>

      <ion-item>
        <ion-input
          label="Password"
          labelPlacement="floating"
          formControlName="password"
          type="password"
          errorText="Password must be at least 8 characters."
        ></ion-input>
      </ion-item>
    </ion-list>

    <ion-button expand="block" type="submit" [disabled]="registerForm.invalid">
      Register
    </ion-button>
  </form>
</ion-content>
```

Ionic 7+ supports `errorText` on `<ion-input>` which displays automatically when the control is invalid and touched. For older versions, use `<ion-note>` below the input.

### Showing Validation Errors (Ionic 6 and earlier)

```html
<ion-item>
  <ion-input label="Email" labelPlacement="floating" formControlName="email" type="email"></ion-input>
</ion-item>
@if (registerForm.get('email')?.invalid && registerForm.get('email')?.touched) {
  <ion-note color="danger" class="ion-padding-start">
    Please enter a valid email.
  </ion-note>
}
```

## Template-Driven Forms

### Setup

**Standalone components** — import `FormsModule`:

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonInput, IonButton } from '@ionic/angular/standalone';

@Component({
  selector: 'app-login',
  templateUrl: './login.page.html',
  standalone: true,
  imports: [FormsModule, IonHeader, IonToolbar, IonTitle, IonContent, IonItem, IonInput, IonButton],
})
export class LoginPage {
  email = '';
  password = '';

  onLogin() {
    // Process login
  }
}
```

**NgModule components** — import `FormsModule` in the page module.

### Template

```html
<ion-header>
  <ion-toolbar>
    <ion-title>Login</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content class="ion-padding">
  <form (ngSubmit)="onLogin()">
    <ion-list>
      <ion-item>
        <ion-input
          label="Email"
          labelPlacement="floating"
          [(ngModel)]="email"
          name="email"
          type="email"
          required
        ></ion-input>
      </ion-item>

      <ion-item>
        <ion-input
          label="Password"
          labelPlacement="floating"
          [(ngModel)]="password"
          name="password"
          type="password"
          required
        ></ion-input>
      </ion-item>
    </ion-list>

    <ion-button expand="block" type="submit">Login</ion-button>
  </form>
</ion-content>
```

Each `[(ngModel)]` binding requires a `name` attribute on the input.

## Ionic Form Components

### ion-input

Text-based input. Supports `type` values: `text`, `password`, `email`, `number`, `search`, `tel`, `url`, `date`, `time`, `datetime-local`, `month`, `week`.

```html
<ion-input label="Username" labelPlacement="floating" formControlName="username" type="text" clearInput="true"></ion-input>
```

- `labelPlacement`: `"floating"`, `"stacked"`, `"fixed"`, `"start"`, `"end"`.
- `clearInput`: shows a clear button when the input has a value.

### ion-textarea

Multi-line text input:

```html
<ion-textarea label="Bio" labelPlacement="floating" formControlName="bio" rows="4" autoGrow="true"></ion-textarea>
```

### ion-select

Dropdown / picker:

```html
<ion-select label="Country" labelPlacement="floating" formControlName="country" interface="popover">
  <ion-select-option value="us">United States</ion-select-option>
  <ion-select-option value="uk">United Kingdom</ion-select-option>
  <ion-select-option value="de">Germany</ion-select-option>
</ion-select>
```

- `interface`: `"popover"`, `"action-sheet"`, `"alert"` (default).
- For multiple selection: add `multiple="true"` and bind to an array.

### ion-checkbox

```html
<ion-checkbox formControlName="agreeTerms" labelPlacement="end">
  I agree to the terms
</ion-checkbox>
```

### ion-toggle

```html
<ion-toggle formControlName="notifications" labelPlacement="start">
  Enable Notifications
</ion-toggle>
```

### ion-radio-group

```html
<ion-radio-group formControlName="plan">
  <ion-item>
    <ion-radio value="free" labelPlacement="end">Free</ion-radio>
  </ion-item>
  <ion-item>
    <ion-radio value="pro" labelPlacement="end">Pro</ion-radio>
  </ion-item>
</ion-radio-group>
```

### ion-range

```html
<ion-range formControlName="volume" min="0" max="100" pin="true">
  <ion-icon name="volume-low-outline" slot="start"></ion-icon>
  <ion-icon name="volume-high-outline" slot="end"></ion-icon>
</ion-range>
```

### ion-datetime

```html
<ion-datetime formControlName="birthdate" presentation="date" preferWheel="true"></ion-datetime>
```

- `presentation`: `"date"`, `"time"`, `"date-time"`, `"month-year"`.
- Typically displayed inside an `<ion-modal>` or `<ion-popover>`.

## Standalone Component Imports for Form Elements

When using standalone components, import each Ionic form component used in the template:

```typescript
import {
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption,
  IonCheckbox,
  IonToggle,
  IonRadio,
  IonRadioGroup,
  IonRange,
  IonDatetime,
} from '@ionic/angular/standalone';
```
