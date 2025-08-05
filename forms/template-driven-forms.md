# Template-Driven Forms

## Table of contents

- [Directives](#directives)
- [Bind input controls](#bind-input-controls)
- [Access Overall Form](#access-overall-form)
- [Naming control](#naming-control)
- [Track form states](#track-form-states)
- [Track control states](#track-control-states)
- [Show/Hide validation error messages](#showhide-validation-error-messages)
- [Validating Input](#validating-input)
- [Custom Validators](#custom-validators)
- [Cross-Validation](#cross-validation)
- [Asynchronous Validation](#asynchronous-validation)
  - [Optimize Performance](#optimize-performance)

## Directives

| Directive      | Description                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `NgModel`      | Reconciles value changes in the attached form element with changes in the data model                                |
| `NgForm`       | Creates a top-level `FormGroup` instance and binds it to a `form` element to track form value and validation status |
| `NgModelGroup` | Creates and binds a `FormGroup` instance to DOM element.                                                            |

## Bind input controls

Find the `<input>` tag and add `ngModel` directive. Use the syntax for two-way binding.

## Access Overall Form

Create a template reference variable and update with the `form` tag.

```html
<form #actorForm="ngForm"></form>
```

## Naming control

When we use `ngModel` on a element, we must define a `name` attribute.

`template-driven-form.html`

```html
<div class="form-group">
  <label for="name">Name</label>
  <input
    class="form-control"
    type="text"
    name="name"
    id="name"
    [(ngModel)]="model.name"
    name="name"
  />
</div>
<div class="form-group">
  <label for="skill">Skill</label>
  <select name="skill" class="form-control" [(ngModel)]="model.skill">
    @for (skill of skills; track $index) {
    <option [value]="skill">{{ skill }}</option>
    }
  </select>
</div>

<p>Form Status : {{ model | json }}</p>
```

## Track form states

The angular automatically applies the `ng-submitted` class to the form once the form has been submitted. This is applied to the `form` element not to the controls inside the `form` element.

## Track control states

Adding the `ngModel` directive adds class names to the control to describe its state. These classes can be used to change tht a control's style based on its state.

| State   | Class (True) | Class (False)  |
| ------- | ------------ | -------------- |
| Visited | `ng-touched` | `ng-untouched` |
| Changed | `ng-dirty`   | `ng-pristine`  |
| Valid   | `ng-valid`   | `ng-invalid`   |

```html
<input class="form-control ng-untouched ng-pristine ng-valid" />;
```

## Show/Hide validation error messages

We can use the template reference variable to access the input field to add custom error messages.

```css
.ng-valid.required {
  border-left: 5px solid red;
}
// This will check only the controls not the entire form element
.ng-invalid:not(form) {
  border-left: 5px solid blue;
}
```

`template-driven-form.html`
```html
<div class="form-group">
  <label for="name">Name</label>
  <input
    class="form-control"
    type="text"
    name="name"
    id="name"
    #name="ngModel"
    [(ngModel)]="model.name"
    required
    name="name"
  />
</div>
<div [hidden]="name.valid || name.pristine" class="alert alert-danger">
  Name is required
</div>
```

### Example

`template-driven-form.html`

```html
<form #actorForm="ngForm" (ngSubmit)="onFormSubmit(actorForm)">
  <div class="form-group">
    <label for="name">Name</label>
    <input
      class="form-control"
      type="text"
      name="name"
      id="name"
      #userName="ngModel"
      [(ngModel)]="model.name"
      required
      name="name"
    />
  </div>
  <div
    [hidden]="userName.valid || userName.pristine"
    class="alert alert-danger"
  >
    Name is required
  </div>
  <div class="form-group">
    <label for="skill">Skill</label>
    <select name="skill" class="form-control" [(ngModel)]="model.skill">
      @for (skill of skills; track $index) {
      <option [value]="skill">{{ skill }}</option>
      }
    </select>
  </div>
  <button
    type="submit"
    class="btn btn-success"
    [disabled]="!actorForm.form.valid"
    (click)="onSubmit()"
  >
    Submit
  </button>
  <br />
  <br />
  <button type="button" class="btn btn-secondary" (click)="addActor()">
    Add Actor
  </button>
  <br />
  <br />
</form>

<pre>Form Status : {{ model | json }}</pre>
```
`template-driven-form.component.ts`

```ts
  skills = ['Acting', 'Singing', 'Dancing', 'Fighting'];
  model = new Actor(1, 'Brad Pitt', this.skills[0]);

  formSubmitted = false;

  onSubmit() {
    this.formSubmitted = !this.formSubmitted;
  }

  onFormSubmit(form: NgForm) {
    form.reset();
    console.log(form)
  }

  addActor() {
    this.model = new Actor(1, '', '');
  }
```

## Validating Input

To add validation, we can use the native HTML form validation like `required` , `minlength` etc..,

`template-driven-form.html`
```html
<input
  type="text"
  name="user-name"
  id="user-name"
  class="form-control"
  required
  minlength="4"
  [(ngModel)]="user"
  #testUser="ngModel"
/>
<br />

@if (testUser.invalid && (testUser.dirty || testUser.pristine)) {
<div>
  @if (testUser.hasError('required')) {
  <p class="alert alert-danger">Name is required</p>
  } @if (testUser.hasError('minlength')) {
  <p class="alert alert-danger">Name must be 4 characters long</p>
  }
</div>
}
```

## Custom Validators

The built-in validators don't always match the usecase of the application. This example demonstrates to check the forbidden name.

`forbidden-name.validator.ts`

```ts
import { AbstractControl, ValidatorFn } from "@angular/forms";
export const forbiddenNameValidator =
  (name: RegExp): ValidatorFn =>
  (control: AbstractControl) => {
    const value = control.value;
    const forbidden = name.test(value);
    return forbidden ? { forbiddenName: { value } } : null;
  };
```

`forbidden-name.directive.ts`
```ts
@Directive({
  selector: "[appForbiddenName]",
  providers: [
    {
      provide: NG_VALIDATORS,
      useExisting: ForbiddenNameDirective,
      multi: true,
    },
  ],
})
export class ForbiddenNameDirective implements Validator {
  forbiddenName = input<string>("", {
    alias: "appForbiddenName",
  });
  validate(control: AbstractControl): ValidationErrors | null {
    return this.forbiddenName
      ? forbiddenNameValidator(new RegExp(this.forbiddenName(), "i"))(control)
      : null;
  }
}
```
`template-driven-form.html`

```html
@if(testUser.hasError('forbiddenName')){
<p class="alert alert-danger">Invalid Name</p>
}
```

>The custom validation directive is instantiated with `useExisting` rather than `useClass`.

### Cross-Validation

A cross-field validator is a [custom validator](#custom-validators) that will compares the values of different fields in aform and accepts or rejects them in combination.

`template-driven-form.html`

```html
<form #actorForm="ngForm" appUnambiguous (ngSubmit)="onFormSubmit(actorForm)">
  @if(actorForm.hasError('unambiguous') && (actorForm. touched ||
  actorForm.dirty)){
  <div class="alert alert-danger">Name and Role cannot be same</div>
  }
</form>
```

`unambiguous.validator.ts`

```ts
import { AbstractControl, ValidationErrors, ValidatorFn } from "@angular/forms";

export const unAmbiguousValidator: ValidatorFn = (
  control: AbstractControl
): ValidationErrors | null => {
  const name = control.get("name");
  const role = control.get("role");

  return name && role && name.value === role.value
    ? {
        unambiguous: true,
      }
    : null;
};
```
### Asynchronous Validation

The Asynchronous validators implement the `AsyncValidatorFn` and `AsyncValidator` interfaces. The following are the difference between the synchronous validation,

- The `validate()` function return a Promise or Observable.
- The `Observable` must be finite.

The validation happens after the synchronous validation, and is performed only if the synchronous validation is successful. After validation begins, the form control enters a `pending` state.

`unique-role.validator.ts`
```ts
import { AbstractControl, AsyncValidatorFn } from "@angular/forms";
import { map } from "rxjs";
import { UniqueRoleService } from "../services/unique-role";

export const uniqueRoleValidation =
  (roleService: UniqueRoleService): AsyncValidatorFn =>
  (control: AbstractControl) => {
    const role = control.value;
    return roleService
      .isUnique(role)
      .pipe(map((isUnique) => (isUnique ? { uniqueRole: true } : null)));
  };
```

`unique-role.service.ts`
```ts
import { Injectable } from "@angular/core";
import { delay, Observable, of } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class UniqueRoleService {
  roles = ["actor", "director", "producer"];

  isUnique(role: string): Observable<boolean> {
    return of(this.roles.includes(role)).pipe(delay(1000));
  }
}
```

`unique-role.directive.ts`
```ts
import { Directive, inject } from '@angular/core';
import {
  AbstractControl,
  AsyncValidator,
  NG_ASYNC_VALIDATORS,
  ValidationErrors,
} from '@angular/forms';
import { Observable } from 'rxjs';
import { UniqueRoleService } from '../services/unique-role';
import { uniqueRoleValidation } from './unique-role.validator';

@Directive({
  selector: '[appUniqueRole]',
  providers: [
    {
      provide: NG_ASYNC_VALIDATORS,
      useExisting: UniqueRoleDirective,
      multi: true,
    },
  ],
})
export class UniqueRoleDirective implements AsyncValidator {
  private uniqueRoleService = inject(UniqueRoleService);
  validate(
    control: AbstractControl
  ): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
    return uniqueRoleValidation(this.uniqueRoleService)(control);
  }
}

```

`template-driven-form.html`

```html
 <div class="form-group">
    <label for="role">Role</label>
    <input
      class="form-control"
      type="text"
      name="role"
      id="role"
      [(ngModel)]="model.role"
      #userRole="ngModel"
      required
      name="role"
      [ngModelOptions]="{ updateOn: 'blur' }"
      appUniqueRole
    />
  </div>

  @if(userRole.pending){
  <div class="alert alert-warning">Validating Role....</div>
  } 
  
  @if(userRole.hasError('uniqueRole')){
  <div class="alert alert-danger">Role should be unique</div>
  }
```

### Optimize Performance

We can delay the form validity by changing the `updateOn` property from `change` to `submit` or `blur`.

`template-driven-form.html` 

```html
<input [(ngModel)]="name" [ngModelOptions]="{updateOn: 'blur'}">
```