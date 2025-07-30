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
