# Reactive Forms

## Table of Contents

- [Form group](#form-group)
- [Nested Form Groups](#nested-form-groups)
  - [Updating parts of the data model](#updating-parts-of-the-data-model)
- [Using FormBuilder Service](#using-formbuilder-service)
- [Validating Form Inputs](#validating-form-inputs)
- [Creating Dynamic Forms](#creating-dynamic-forms)
- [Typed Forms](#typed-forms)
- [Untyped Forms](#untyped-forms)
- [FormControl](#formcontrol)
  - [Nullabilty](#nullabilty)
- [FormArray](#formarray)
- [FormGroup](#form-group)
  - [Optional Controls and Dynamic Groups](#optinal-controls-and-dynamic-groups)
- [FormRecord](#formrecord)
- [FormBuilder and NonNullableFormBuilder](#formbuilder-and-nonnullableformbuilder)
- [Validating Input](#validating-input)
  - [Validators Functions](#validator-functions)
  - [Built-in Validators functions](#built-in-validators-functions)
  - [Custom Validators](#custom-validators)
- [Control status CSS classes](#control-status-css-classes)
  - [Validation state classes](#validation-state-classes)
  - [Interaction state classes](#interaction-state-classes)
  - [Form state class](#form-state-class)
- [Cross Validation](#cross-validation)
- [Asynchronous Validation](#asynchronous-validation)
  - [Optimize Performance](#optimize-performance)


## Form group

Forms typically contain several related controls. Reactive forms provide two types of grouping multiple related controls into a single input form.

**Form Group** : Defines a form with a fixed set of controls that you can manage together.

**Form Array** : Defines a dynamic form, where you can add and reove controls at run time. We can also nest form arrays to create more complex forms.

Just as a form control instance gives you control over a single input field, a form group instance tracks the form state of a group of form control instances (for example, a form). Each control in a form group instance is tracked by name when creating the form group.

### Example

`reactive-form.html`

```html
<h3>Formgroup</h3>

<form [formGroup]="profileEditor" (ngSubmit)="onSubmit()">
  <label for="firstName">First Name: </label>
  <input type="text" name="" id="firstName" formControlName="firstName" />
  <br />
  <br />
  <label for="lastName">Last Name: </label>
  <input type="text" name="" id="lastName" formControlName="lastName" />
  <br />
  <br />
  <button type="submit">Submit</button>
</form>
```

`reactive-form.component.ts`

```ts
@Component({...})
export class AppComponent {
  profileForm = new FormGroup({
    firstName: new FormControl(""),
    lastName: new FormControl(""),
  });
}
```

## Nested Form Groups

`reactive-form.html`

```html
<form [formGroup]="profileEditor" (ngSubmit)="onSubmit()">
  <label for="firstName">First Name: </label>
  <input type="text" name="" id="firstName" formControlName="firstName" />
  <br />
  <br />
  <label for="lastName">Last Name: </label>
  <input type="text" name="" id="lastName" formControlName="lastName" />
  <br />
  <br />
  <div formGroupName="address">
    <label for="city">City</label>
    <input type="text" name="" id="city" formControlName="city" />
    <br />
    <br />
    <label for="state">State</label>
    <input type="text" name="" id="state" formControlName="state" />
    <br />
    <br />
  </div>
  <button type="submit">Submit</button>
</form>
```

`reactive-form.component.ts`
```ts
@Component({...})
export class AppComponent {
profileEditor = new FormGroup({
  firstName: new FormControl(""),
  lastName: new FormControl(""),
  address: new FormGroup({
    city: new FormControl(""),
    state: new FormControl(""),
  }),
});
}
```

### Updating parts of the data model

There are two ways to update the model value :

- `setValue()` : It sets a new value for an individual control. This method strictly follows to the structure of the form group and replaces the entire value for the control.

- `patchValue()` : Replace any properties defined in the object.

`reactive-form.component.ts`
```ts
@Component({...})
export class AppComponent {
   updateProfile() {
    this.profileEditor.patchValue({
      firstName: 'Iraianbuu',
      address: {
        city: 'Bengaluru',
      },
    });
    console.log(this.profileEditor.value);
  }

  saveProfile() {
    this.profileEditor.setValue({
      firstName: 'Iraianbu',
      lastName: 'A',
      address: {
        city: 'Bengaluru',
        state: 'Karnataka',
      },
    });
    console.log(this.profileEditor.value);
  }
}
```

## Using FormBuilder Service

`reactive-form.component.ts`
```ts
@Component({...})
export class AppComponent {
  private fb = inject(FormBuilder);

  profileEditor = this.fb.group({
    firstName: [''],
    lastName: [''],
    address: this.fb.group({
      city: [''],
      state: [''],
    }),
  });
}
```

## Validating Form Inputs

`reactive-form.component.ts`
```ts
@Component({...})
export class AppComponent {
  private fb = inject(FormBuilder);

  profileEditor = this.fb.group({
    firstName: ['', Validators.required],
    lastName: [''],
    address: this.fb.group({
      city: [''],
      state: [''],
    }),
  });
}
```

`reactive-form.html`
```html
<form [formGroup]="profileEditor" (ngSubmit)="onSubmit()">
  <label for="firstName">First Name: </label>
  <input type="text" name="" id="firstName" formControlName="firstName" />
  @if (profileEditor.get('firstName')?.invalid &&
  profileEditor.get('firstName')?.touched) {
  <p>First name is required</p>
  }
  <label for="lastName">Last Name: </label>
  <input type="text" name="" id="lastName" formControlName="lastName" />
  <br />
  <br />
  <div formGroupName="address">
    <label for="city">City</label>
    <input type="text" name="" id="city" formControlName="city" />
    <br />
    <br />
    <label for="state">State</label>
    <input type="text" name="" id="state" formControlName="state" />
    <br />
    <br />
  </div>
  <button type="submit" [disabled]="!profileEditor.valid">Submit</button>
</form>
<p>Profile form {{profileEditor.status}}</p>
```

## Creating Dynamic Forms

`reactive-form.html`
```html
<div formArrayName="aliases">
  <h3>Aliases</h3>
  <button type="button" (click)="addAliases()">Add</button>
  @for (alias of aliases.controls; track $index ; let i = $index) {
  <div>
    <label for="alias-{{ i }}">Alias - {{ i }}</label>
    <input type="text" name="" id="alias-{{ i }}" [formControlName]="i" />
  </div>
  }
</div>
<pre>{{ profileEditor.value | json }}</pre>
```

`reactive-form.component.ts`
```ts
@Component({...})
export class AppComponent {
  private fb = inject(FormBuilder);

  profileEditor = this.fb.group({
    firstName: ['', Validators.required],
    lastName: [''],
    address: this.fb.group({
      city: [''],
      state: [''],
    }),
    aliases: this.fb.array([this.fb.control('')]),
  });
}

  get aliases() {
    return this.profileEditor.get('aliases') as FormArray;
  }

  addAliases() {
    this.aliases.push(this.fb.control(''));
  }
```

## Typed Forms

`reactive-form.component.ts`
```ts
const login = new FormGroup({
  email: new FormControl(""),
  password: new FormControl(""),
});
```

The Angular provides many APIs for `FormGroup`. For example, you may use `login.value` , `login.controls` , `login.patchValue`.

```ts
const emailDomain = login.value.email.domain;
```

The above code does not compile, because there is no `domain` property on `email`.

## Untyped Forms

`reactive-form.component.ts`
```ts
const login = new UntypedFormGroup({
  email: new UntypedFormControl(""),
  password: new UntypedFormControl(""),
});
```

## FormControl

`reactive-form.component.ts`
```ts
const email = new FormControl("angularrox@gmail.com");

const age = new FormControl<number | null>(0); // We can also specify an explicit type.
```

This control will automatically inferred to `FormControl<string|null>`.

### Nullabilty

`reactive-form.component.ts`
```ts 
const email = new FormControl("iraianbu011@gmail.com");
email.reset();
console.log(email.value); // null
```

The `email` becomes `null` because the control can become `null` at any time. If we want to make this control non-nullable, we can use the `nonNullable` property.

`reactive-form.component.ts`
```ts
const email = new FormControl("iraianbu011@gmail.com", { nonNullable: true });
email.reset();
console.log(email.value); // iraianbu011@gmail.com
```

## FormArray

A `FormArray` contains list of controls. The type parameter corresponds to the type of each inner control.

`reactive-form.component.ts`
```ts
const names = new FormArray([new FormControl("Irai")]);
names.push(new FormControl("Anbu"));
```

This will be inferred to `FormArray<string|null>`. If we want to use multiple different element, we can use `UntypedFormArray`.

## FormGroup

The `FormGroup` type for forms with an enumerated set of keys.

`reactive-form.component.ts`
```ts
const login = new FormGroup({
  email: new FormControl("", { nonNullable: true }),
  password: new FormControl("", { nonNullable: true }),
});
```

It is possible to **disable controls**. It will not appear in the group's value. The `login.value` is `Partial<{email:string,password:string}>. If we want to access the value including disabled controls, we can use `login.getRawValue()`.

### Optinal Controls and Dynamic Groups

`reactive-form.component.ts`
```ts
interface LoginForm {
  email: FormControl<string>;
  password?: FormControl<string>;
}
const login = new FormGroup<LoginForm>({
  email: new FormControl("", { nonNullable: true }),
  password: new FormControl("", { nonNullable: true }),
});
login.removeControl("password");
```

The Typescript will only allow optional controls to be added or removed.

## FormRecord

`FormRecord` can be used when the keys are not known ahead of time.

`reactive-form.component.ts`
```ts
const addresses = new FormRecord<FormControl<string | null>>({});
addresses.addControl("Andrew", new FormControl("2340 Folsom St"));
```

This will be inferred to `FormArray<string|null>`.

If we need a `FormGroup` that is both dynamic and heterogeneous, we should use `UntypedFormGroup`.

A `FormRecord` can be also built with `FormBuilder`.

`reactive-form.component.ts`
```ts
const addresses = fb.record({ Andrew: "2340 Folsom St" });
```

## FormBuilder and NonNullableFormBuilder

`reactive-form.component.ts`
```ts
const fb = inject(FormBuilder);
const login = fb.nonNullable.group({
  email: "",
  password: "",
});
```

```ts
const fb = inject(NonNullableFormBuilder);
const login = fb.group({
  email: "",
  password: "",
});
```

Both example set inner controls will be `nonNullable`.

## Validating Input

Instead of adding validators through attributes in the template, we can add directly to the form control model in the component class.

### Validator Functions

The validators functions can be either synchronous or asynchronous.

| Type             | Definition                                                                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Sync validators  | This will take a control instance and return either set of validation errors or null. It should be passed as a second argument.                              |
| Async validators | This will take a control instance and return a Promise or Observable that emits a set of validation errors or null. It should be passed as a third argument. |

> Angular runs async validators if all sync validators pass.

### Built-in Validators Functions

The same built-in validators that are available in [Template Driven Forms](template-driven-forms.md) as such as `required`, `min` , `max` , `minLength` , `maxLength` etc..,

`reactive-form.component.ts`
```ts
 private fb = inject(FormBuilder);

  actorForm = this.fb.group({
    name: this.fb.control('', [Validators.required, Validators.minLength(4)]),
    role: this.fb.control(''),
    skill: this.fb.control('', [Validators.required]),
  });

  get name() {
    return this.actorForm.get('name');
  }

  get skill() {
    return this.actorForm.controls.skill;
  }
```

`reactive-form.html`
```html
<form [formGroup]="actorForm">
  <div class="form-group">
    <label for="name">Name</label>
    <input
      type="text"
      class="form-control"
      name="name"
      id="name"
      formControlName="name"
    />
    @if(name?.invalid && (name?.touched || name?.dirty)){ @if
    (name?.hasError('required')) {
    <div class="alert alert-danger">Name is required</div>
    } @if (name?.hasError('minlength')) {
    <div class="alert alert-danger">Name must be 4 characters long</div>
    } }
    <label for="role">Role</label>
    <input
      type="text"
      name="role"
      class="form-control"
      id="role"
      formControlName="role"
    />
    <label for="skill">Skill</label>
    <input
      type="text"
      name="skill"
      class="form-control"
      id="skill"
      formControlName="skill"
    />
    @if(skill.invalid && (skill.touched || skill.dirty)){ @if
    (skill.hasError('required')) {
    <div class="alert alert-danger">Name is required</div>
    } }
  </div>
  <button class="btn btn-success" [disabled]="actorForm.invalid">Submit</button>
</form>
```

### Custom Validators

In reactive forms, we can add a custom validator by passing a function directly to the `FormControl`.

`reactive-form.component.ts`
```ts
actorForm = this.fb.group({
  name: this.fb.control("", [
    Validators.required,
    Validators.minLength(4),
    forbiddenNameValidator(/irai/i),
  ]),
  role: this.fb.control(""),
  skill: this.fb.control("", [Validators.required]),
});
```

## Control status CSS classes

Angular provides many control properties onto the form control element as CSS classes.

### Validation state classes

- `.ng-valid`
- `.ng-invalid`
- `.ng-pending` (Used in Asynchronous validation)

### Interaction state classes

- `.ng-pristine`
- `.ng-dirty`
- `.ng-touched`
- `.ng-untouched`

### Form state class

- `.ng-submitted` (After the form gets submitted)

## Cross Validation

A cross-field validator is a [custom validator](#custom-validators) that will compares the values of different fields in aform and accepts or rejects them in combination.

`reactive-form.component.ts`
```ts
actorForm = this.fb.group(
  {
    name: this.fb.control(""),
    role: this.fb.control(""),
    skill: this.fb.control(""),
  },
  { validators: unAmbiguousValidator }
);
```

`reactive-form.html`
```html
@if(actorForm.hasError('unambiguous') && (actorForm.touched ||
actorForm.dirty)){
<div class="alert alert-danger">Name and Role cannot be same</div>
}
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

## Asynchronous Validation

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

`reactive-form.html`
```html
<label for="role">Role</label>
<input
  type="text"
  name="role"
  class="form-control"
  id="role"
  formControlName="role"
/>

@if(role.pending && (role.dirty || role.touched)) {
<div class="alert alert-warning">Checking role...</div>
} @if(role.hasError('uniqueRole')) {
<div class="alert alert-danger">Role must be unique</div>
}
```

`reactive-form.component.ts`
```ts
  private roleService = inject(UniqueRoleService);

  actorForm = this.fb.group(
    {
      name: this.fb.control(''),
      role: this.fb.control('', {
        asyncValidators: uniqueRoleValidation(this.roleService),
        updateOn : 'blur'
      }),
      skill: this.fb.control(''),
    },
  );
```

### Optimize Performance

We can delay the form validity by changing the `updateOn` property from `change` to `submit` or `blur`.

`reactive-form.component.ts`
```ts
new FormControl('', {updateOn: 'blur'});
```