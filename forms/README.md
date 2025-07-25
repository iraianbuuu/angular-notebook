# Angular Forms

## Table of Contents

- [Introduction](#introduction)
- [Reactive Forms](#reactive-forms)
- [Template-Driven Forms](#template-driven-forms)
- [Key Differences](#key-differences)
- [Scalability](#scalability)
- [Common Form Foundation classes](#common-form-foundation-classes)
- [Data flows in Forms](#data-flows-in-forms)
  - [Reactive Forms](#reactive-forms)
  - [Template-Driven Forms](#template-driven-forms)
- [Mutability of the Data Model](#mutability-of-the-data-model)
- [Form group](#form-group)
- [Nested Form Groups](#nested-form-groups)
  - [Updating parts of the data model](#updating-parts-of-the-data-model)
- [Using FormBuilder Service](#using-formbuilder-service)
- [Validating Form Inputs](#validating-form-inputs)
- [Creating Dynamic Forms](#creating-dynamic-forms)

Angular provides two main types of forms:

- **Reactive Forms**
- **Template-Driven Forms**
## Reactive Forms

These forms are controlled primarily through the component
class. They are scalable, reusable and idea for complex scenarios with many form controls or logic-driven requirements. It provide direct, explicit access to the underlying form's object model.

## Template-Driven Forms

These forms are set up and managed directly in the template.
They work well for simpler forms where the validation and state management are minimal. They are useful for adding a simple form to an app, such as a login form.

## Key Differences

|                 | Reactive Forms                       | Template-Driven Forms           |
| --------------- | ------------------------------------ | ------------------------------- |
| Setup           | Explicit, created in component class | Implicit, created by directives |
| Data Model      | Structured and immutable             | Unstructured and mutable        |
| Data Flow       | Synchronous                          | Asynchronous                    |
| Form Validation | Functions                            | Directives                      |

## Scalability

**Reactive Forms** are more scalable and reusable, making them ideal for complex forms with many controls or logic-driven requirements. They provide direct access to th underlying form API, and use **Synchronous** data flow between the view and the data model. It requires less setup for testing.

**Template-Driven Forms** focus on simple scenarios and are not as resuasble. They use **Asynchronous** data flow between the view and the data model, and require more setup for testing.

## Common Form Foundation classes

| Base classes           | Details                                                                   |
| ---------------------- | ------------------------------------------------------------------------- |
| `FormControl`          | Tracks the value and validation status of an individual form control      |
| `FormGroup`            | Tracks the same values and validation status of a group of form controls  |
| `FormArray`            | Tracks the same values and validation status of an array of form controls |
| `ControlValueAccessor` | Creates a bridge between a form control and built-in DOM element          |

```ts
import { Component } from "@angular/core";
import { FormControl, FormsModule, ReactiveFormsModule } from "@angular/forms";

@Component({
  selector: "app-forms-demo",
  imports: [ReactiveFormsModule, FormsModule],
  templateUrl: "./forms-demo.component.html",
  styleUrl: "./forms-demo.component.scss",
})
export class FormsDemoComponent {
  favoriteColorControl = new FormControl("");
  favoriteColor = "";
}
```

```html
<div class="container">
  <h4>Reactive Form</h4>
  <div>
    <input type="text" [formControl]="favoriteColorControl" />
    <p>Favorite Color: <span>{{ favoriteColorControl.value }}</span></p>
  </div>

  <h4>Template-Driven Form</h4>
  <div>
    <input type="text" [(ngModel)]="favoriteColor" />
    <p>Favorite Color: <span>{{ favoriteColor }}</span></p>
  </div>
</div>
```

## Data flows in Forms

### Reactive Forms

Reactive forms in Angular allow for a direct connection between the form elements in the view and the data model through instances of FormControl. This ensures immediate and synchronous data binding between the view and the model.

In Angular reactive forms, the interaction flows in two directions :

**View to Model** : User action in the view update the data model.

1. The user types a value into the input element.
2. The form input element emits an "input" event with the latest value.
3. The `ControlValueAccessor` listening for events on the form input element immediately relays the new value to the `FormControl` instance.
4. The `ControlValueAccessor` calls the `setValue()` on the `FormControl`
5. The `FormControl` instance emits the new value through the `valueChanges` observable.
6. Any subscribers to the `valueChanges` observable receive the new value.

**Model to View** : Programmatic changes in the data model reflect in the view.

1. The user calls the `formControl.setValue()` method, which updates the `FormControl` value.
2. The `FormControl` instance emits the new value through the `valueChanges` observable.
3. Any subscribers to the `valueChanges` observable receive the new value.
4. The `ControlValueAccessor` on the form input element updates the element with the new value.

```ts
@Component({...})
export class App {
  favoriteColor = new FormControl('');

  constructor() {
    this.favoriteColor.valueChanges.subscribe((value) => {
      console.log(value);
    });
  }

  ngOnInit() {
    this.favoriteColor.setValue('Blue');
  }
}
```

### Template-Driven Forms

**View to Model** :

1. The user types a value into the input element.
2. The input element emits an "input" event.
   3.The control value accessor attached to the input triggers the `setValue()` method on the FormControl instance.
   4.The `FormControl` instance emits the new value through the `valueChanges` observable.
   5.Any subscribers to the `valueChanges` observable receive the new value.
   6.The control value accessor also calls the `NgModel.viewToModelUpdate()` method which emits an `ngModelChange` event.
   7.Because the component template uses two-way data binding, the property in the component is updated to the value emitted by the `ngModelChange event`.

**Model to View** :

1. The value is updated in the component and change detection begins.
2. During change detection, the `ngOnChanges` lifecycle hook is called on the `NgModel` directive instance because the value of one of its inputs has changed.
3. The `ngOnChanges()` method queues an async task to set the value for the internal `FormControl` instance and change detection completes.
4. On the next tick, the task to set the `FormControl` instance value is executed.
5. The `FormControl` instance emits the latest value through the `valueChanges` observable.
6. Any subscribers to the `valueChanges` observable receive the new value.
7. The control value accessor updates the form input element in the view with the latest value.

## Mutability of the Data Model

**Reactive Forms** : The data model is immutable, meaning that instead of modifying the existing data model, a new version of the data model is created each time.

Each change generates a new `FormControl` instance value, allowing us to track these unique changes. It's easier to manage change tracking and integrates with RxJS (Observables).

**Template-Driven Forms** : The data model is mutable, meaning that the existing data model is modified each time. Angular's two-way data binding updates the data model immediately as changes occur in the template.

## Form group

Forms typically contain several related controls. Reactive forms provide two types of grouping multiple related controls into a single input form.

**Form Group** : Defines a form with a fixed set of controls that you can manage together.

**Form Array** : Defines a dynamic form, where you can add and reove controls at run time. We can also nest form arrays to create more complex forms.

Just as a form control instance gives you control over a single input field, a form group instance tracks the form state of a group of form control instances (for example, a form). Each control in a form group instance is tracked by name when creating the form group.

### Example

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