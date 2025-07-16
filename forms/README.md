# Angular Forms

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
