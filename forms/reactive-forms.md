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

## Typed Forms

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

```ts
const login = new UntypedFormGroup({
  email: new UntypedFormControl(""),
  password: new UntypedFormControl(""),
});
```

## FormControl

```ts
const email = new FormControl("angularrox@gmail.com");

const age = new FormControl<number | null>(0); // We can also specify an explicit type.
```

This control will automatically inferred to `FormControl<string|null>`.

### Nullabilty

```ts
const email = new FormControl("iraianbu011@gmail.com");
email.reset();
console.log(email.value); // null
```

The `email` becomes `null` because the control can become `null` at any time. If we want to make this control non-nullable, we can use the `nonNullable` property.

```ts
const email = new FormControl("iraianbu011@gmail.com", { nonNullable: true });
email.reset();
console.log(email.value); // iraianbu011@gmail.com
```

## FormArray

A `FormArray` contains list of controls. The type parameter corresponds to the type of each inner control.

```ts
const names = new FormArray([new FormControl("Irai")]);
names.push(new FormControl("Anbu"));
```

This will be inferred to `FormArray<string|null>`. If we want to use multiple different element, we can use `UntypedFormArray`.

## FormGroup

The `FormGroup` type for forms with an enumerated set of keys.

```ts
const login = new FormGroup({
  email: new FormControl("", { nonNullable: true }),
  password: new FormControl("", { nonNullable: true }),
});
```

It is possible to **disable controls**. It will not appear in the group's value. The `login.value` is `Partial<{email:string,password:string}>. If we want to access the value including disabled controls, we can use `login.getRawValue()`.

### Optinal Controls and Dynamic Groups

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

```ts
const addresses = new FormRecord<FormControl<string | null>>({});
addresses.addControl("Andrew", new FormControl("2340 Folsom St"));
```

This will be inferred to `FormArray<string|null>`.

If we need a `FormGroup` that is both dynamic and heterogeneous, we should use `UntypedFormGroup`.

A `FormRecord` can be also built with `FormBuilder`.

```ts
const addresses = fb.record({ Andrew: "2340 Folsom St" });
```

## FormBuilder and NonNullableFormBuilder

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

### Validator Functions

### Built-in Validators Functions

### Custom Validators