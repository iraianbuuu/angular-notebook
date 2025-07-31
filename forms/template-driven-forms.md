# Template-Driven Forms

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

To add validation, we can use the native HTML form validation like `required` , `minlength` etc..,<input
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
