# Signal Inputs and Queries

## Table of Contents

- [Signal Inputs](#signal-inputs)
- [Signal Queries](#signal-queries)
  - [`viewChild`](#viewchild)
  - [`viewChildren`](#viewchildren)
  - [`contentChild`](#contentchild)
  - [`contentChildren`](#contentchildren)
  - [Options](#options)
  - [Required Queries](#required-queries)
  - [Query Availability Timing](#query-availability-timing)
  - [Query Declarations and Usage](#query-declarations-and-usage)

## Signal Inputs

Signal inputs that allow component values to be bound from parent components. These values are exposed using a signal, which can change throughout the lifecycle of your component.

```ts
@Component({...})
export class AppComponent {
  firstName = input.required<string>(); // Required input
  lastName = input<string>(); // Optional input
  age = input<number>(0); // Default value
  currentAge = input<number>(0, { alias: 'studentAge' }); // Default value with alias
}
```

## Signal Queries

The signal-based queries offer a way to retriev and interact with elements, components or directives in
a component's template (view) or projected content.

Angular provides two main types of queries:

1. View Queries : These retrives results from the component's own template. It let us retrieve elements or components that are part of the component's template.

2. Content Queries : These retrieve results from the component's projected content.

### `viewChild`

The `viewChild` function retrieves a single element or component either by string reference or by component/directive type.

```ts
@Component({...})
export class AppComponent {
  // Decorator
  @ViewChild('rxjsInteropComponent') rxjsInteropComponent!: RxjsInteropComponent;

  // viewChild by component/directive type
  component1 = viewChild(RxjsInteropComponent); // Signal

  component1 = viewChild(RxjsInteropComponent, {
    read: ElementRef,
  }); // ElementRef

  // viewChild by element type
  divElement = viewChild('rxjsInteropComponent');
}
```

### `viewChildren`

The `viewChildren` function retrieves multiple elements or components either by string reference or by component/directive type.

```ts
@Component({...})
export class AppComponent {
  elements = viewChildren(RxjsInteropComponent);
}
```

### `contentChild`

The `contentChild` function retrieves a single element or component from the component's projected content.

```ts
@Component({...})
export class AppComponent {
  headerElement = contentChild(HeaderComponent);
}
```

```html
<app-content-elements>
  <app-header />
</app-content-elements>
```

### `contentChildren`

The `contentChildren` function retrieves multiple elements or components from the component's projected content.

```ts
@Component({...})
export class AppComponent {
  elements = contentChildren<ElementRef>('h1Elements');
}
```

```html
<app-content-elements>
  <div #h1Elements>
    <h1>This is another heading element</h1>
    <h1>This is another heading element</h1>
  </div>
  <app-header />
</app-content-elements>
```

### Options

`descendants` : This option allows you to retrieve elements or components that are descendants of the component.

`read` : This option allows you to specify the type of the element or component.

### Required Queries

If a child query (`viewChild` or `contentChild`) does not find a result, its value is `undefined`.For such cases, you can mark child queries as required to enforce presence of at least one matching result. If no result is found, Angular will throw an runtime error.

```ts
@Component({...})
export class AppComponent {
  headerElement = contentChild.required(HeaderComponent);
}
```

### Query Availability Timing

When angular builds a component, there is a period where the signal query has been created but the template has not been fully processed. During this time, the queries will return :

- `undefined` (for single queries)
- An empty array (for multiple queries)

### Query Declarations and Usage

We can declare signal-based queries in class properties. These functions should not be called in other parts of the component (e.g., `constructor`).