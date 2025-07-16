# Signals

Signals in Angular are a way to manage state and reactive data flows, similar to Observables or BehaviorSubjects.

Signals are more lightweight and built directly into the framework, making them easier to use and understand.

## Introduction

A signal is a essentially a wrapper around a value that notifies consumers (functions,components or services) when the value changes.

This allows for a reactive data flow, where components and services can subscribe to signals and react to changes in the value.

## Characteristics

1. Hold any type of value (Primitive, Object, Array, etc.)
2. Exposes a getter function to read the current value.
3. Notifies Angular when their value is used, allowing Angular to track dependencies automatically.

## Types of Signals

1. Writable Signals (Can be updated)
2. Read-only Signals (Cannot be updated)

## Writable Signals

```ts
const count = signal<number>(0);
count.set(1);
count.update((count) => count + 1);
```

- `set` - Sets the value of the signal directly
- `update` - Updates the value of the signal using a callback function

## Read-only Signals

```ts
const count: Signal<number> = signal<number>(0);
console.log(count());
```

## Computed Signals

Computed signals are read-only signals that are derived from other signals.

Computed signals are useful when you need to derive a value from another signal and you want to update the derived value when the original signal changes.

- Lazy Evaluation (Computed signals are only evaluated when the value is used)

- Memoization (Computed signals are memoized and only re-evaluate when the dependencies change)

### Dynamic Computed Signals

```ts
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return "Nothing to see here!";
  }
});
```

When `showCount` is false, `conditionalCount` returns "Nothing to see here!" without reading `count`. Updating `count` won't trigger recomputation.

When `showCount` is true, `conditionalCount` reads count and shows its value. Now updating count will trigger recomputation.

Dependencies can be added or removed dynamically. Setting `showCount` back to false removes `count` as a dependency.

### Reading Signals in `OnPush` Components

```ts
@Component({
  selector: 'app-my-component',
  template: `{{ count() }}`,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
```

In OnPush components, Angular automatically tracks signals used in templates and updates the component when signal values change.

### Usecases

1. Expensive Calculations
2. Data Transformation
3. Read-only data

## Key Benefits

1. Simple State Management (Provides an easy way to manage state rather than using Observables and Subscriptions)

2. Reactive Data Flow (Angular automatically tracks dependencies and updates components when the signal value changes)

3. Type Safety (Signals are typed and provide type safety)

## Effects

Effects allows us to perform side effects like API calls, logging, etc. when one or more signals change.

It is a function that run whenever one or more signals change. When we create an effect, Angular tracks the signals used within that effect and re-runs the effect when the signals change.

```ts
const count = signal(0);
effect(() => {
  console.log(count());
});
```

### Working

Initial execution : Effects always run once when the component is created.

Tracking dependencies : Effects track the signals used within them.

Asynchronous execution : Effects execute asynchronously during Angular's change detection cycle.

### Usecases

1. Logging data
2. Keeping in sync with `window.localStorage`
3. Adding Custom DOM behavior

### Avoid `effects` when

Avoid using `effect` for state propagation (i.e. when a signal is used to update another signal). This can result in infinte circular dependencies, `ExpressionChangedAfterItHasBeenChecked`.

```ts
const count = signal(0);
const doubleCount = signal(1);

effect(() => {
  doubleCount.set(count() * 2);
});
```

### Injection Context

We can create an `effect()` within an injection context like `constructor` or `inject`.

```ts
@Component({...})
export class SignalComponent {
  count = signal(0);
  constructor() {
    effect(() => {
      console.log(this.count());
    });
  }
}
```

We can assign the `effect` to a field.

```ts
@Component({...})
export class SignalComponent {
  count = signal(0);
  private loggingEffect = effect(() => {
    console.log(this.count());
  });
}
```

To create an effect outside the constructor, we can pass an `Injector` to the `effect` function.

```ts
@Component({...})
export class EffectiveCounterComponent {
  readonly count = signal(0);
  private injector = inject(Injector);
  initializeLogging(): void {
    effect(() => {
      console.log(`The count is: ${this.count()}`);
    }, {injector: this.injector});
  }
}
```

### Destroying Effects

When we create an `effect`, it's automatically destroyed when the component is destroyed. Effect return an `EffectRef` object which has a `destroy` method.
We can also manually destroy an effect by passing `{manualCleanup: true}` to the `effect` function.

```ts
@Component({...})
export class SignalComponent {
  count = signal(0);
  private loggingEffect = effect(() => {
    console.log(this.count());
  } , {manualCleanup: true});

  ngOnDestroy(): void {
    this.loggingEffect.destroy();
  }
}
```

## Equality Functions

Equality functions allow us to customize how changes to signal values are detected. By default, Angular uses referential equality `===` to compare values. This approach only checks if two variables reference the same object. For complex objects, this approach can be inefficient because it doesn't check for value equality.

Equality functions can be provided to both `Writable` and `Computed` signals. This is useful when dealing with deep equality checks rather than referential equality.

```ts
@Component({...})
export class SignalComponent {
  data = signal(['Angular'], {equal: isEqual}); // Lodash isEqual function
  constructor() {
    effect(() => {
      console.log(this.data());
    });
  }
  ngOnInit(): void {
    setTimeout(() => {
      this.data.set(['Angular']);
    }, 2000);
  }
}
```

## Read without Tracking Dependencies

If we want to execute function which may read signals without tracking dependencies, we can use `untracked`.

```ts
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
});
```

This will log the message if any one of the signal changes. However, if we want to run only if the `currentUser` changes, we can use `untracked`.

**⚠️ Important:** `untracked` is a function that allows us to read signals without tracking dependencies. We should pass the signal reference, not call it as a function.

```ts
effect(
  () => {
    console.log(`User set to ${currentUser()} and the counter is ${untracked(counter)}`);
  },
  { allowSignalWrites: true }
);
```

We can also use inside `effect` function.

```ts
effect(() => {
  untracked(() => {
    console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
  });
});
```

## Effective Cleanups

We can pass an `onCleanup` function to the `effect` function. This function will be called when the effect is destroyed.

```ts
effect((onCleanup) => {
  const user = currentUser();
  const timer = setInterval(() => {
    console.log(`1 second ago, the user became ${user}`);
  }, 1000);
  onCleanup(() => {
    clearInterval(timer);
  });
});
```

# Signals with RxJS

## `toSignal`

The `toSignal` function converts an RxJS Observable to a signal.

```ts
@Component({...})
export class SignalComponent {
 counterObservable = interval(1000);

 counter = toSignal(this.counterObservable, {
  initialValue : 0
 })
}
```

### Manually Destroying Observables

```ts
@Component({...})
export class SignalComponent {
counterObservable = interval(1000);
  private subscription!: Subscription;

  ngOnInit() {
    this.subscription = this.counterObservable.subscribe();
  }

  counterSignal = toSignal(this.counterObservable, {
    initialValue: 0,
    manualCleanup: true,

  });

  ngOnDestory() {
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
  }
```

Options :

- `initialValue` : The initial value of the signal.
- `manualCleanup` : This allows to clean up (unsubscribe) from the observable.
- `requireSync` : It enforces the Observables emits synchronously on subscription.

## Error and Completion

If an Observable used in `toSignal` produces an error, that error is thrown when the signal is read.

## `toObservable`

`toObservable` is a utility in Angular that converts a signal to an Observable.
This allows us to take the signal ans use it with RxJS operators like `map`, `filter`, `switchMap`, etc.

### Monitoring Signals

When a signal's value changes, the toObservable utility tracks the changes
using a `effect` and emits the update value through the Observable.

### Injection Context

The `toObservable` relies on an injection context. If there's no automatic injection
context, you can provide an `Injector` manually.

On Subscription, `toObservable` might emit the last known value of the signal immediately (if available) via `ReplaySubject`.

```ts
@Component({...})
export class SignalComponent {
  query = signal('');
  query$ = toObservable(this.query);

  ngOnInit() {
    this.query.set('A');
    this.query.set('B');
    this.query.set('C');
  }

  ngAfterViewInit() {
    this.query$.subscribe(console.log);
  }
}
```

## `outputFromObservable`

The `outputFromObservable` lets you create a component or directive output that emits based on an RxJS observable.

```ts
@Component({...})
export class RxjsInteropComponent {
  // Observable
  counter$ = new Observable<number>((subscriber) => {
    let count = 0;
    const interval = setInterval(() => {
      count++;
      this.counter.emit(count);
      subscriber.next(count);
    }, 1000);
    return () => clearInterval(interval);
  });
  // EventEmitter
  @Output() counter = new EventEmitter<number>();

  // outputFromObservable
  outputFromObservable = outputFromObservable(this.counter$);
}
```

```html
<!-- Using EventEmitter -->
<app-rxjs-interop (counter)="onCounterChange($event)"></app-rxjs-interop>

<!-- Using outputFromObservable -->
<app-rxjs-interop (outputFromObservable)="onCounterChange($event)"></app-rxjs-interop>
```

## `outputToObservable`

The `outputToObservable` function lets you create an RxJS observable from a component output.

```ts
@Component({...})
export class AppComponent {
  showCounterOperation = signal(false);
  counterO$!: Observable<number>;
  @ViewChild('rxjsInteropComponent') rxjsInteropComponent!: RxjsInteropComponent;


  ngAfterViewInit() {
    if (this.rxjsInteropComponent) {
      this.counterO$ = outputToObservable(this.rxjsInteropComponent.counter);
    }
  }

  onCounterChange(counter: number) {
    console.log('counter', counter);
  }

  onToggleComponent() {
    this.showCounterOperation.set(!this.showCounterOperation());
  }
```

```html
<app-rxjs-interop #rxjsInteropComponent></app-rxjs-interop>
```

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

## Linked Signals

`linkedSignal` is a utility that allows us to create a signal that is linked to another signal. It behaves like `computed` signals with a writable side.

`linkedSignal` is like `computed` but with 2 main differences :

1. The derived value of `linkedSignal` can be overridden at any time.

2. The `linkedSignal` provide access to the previous values

### Options

- `source` : The source signal that the linked signal is linked to.
- `computation` : It is a function that receives the new value of source and a previous object. The previous object contains the previous value of source and the previous value of the computation.
- `equal` : It is a function that compares the new value of source and the previous value of source.

```ts
import { Component, linkedSignal, signal } from "@angular/core";

@Component({
  selector: "app-linked-signals",
  imports: [],
  template: `
    <ul>
      @for (reaction of reactions(); track $index) {
      <li [class.selected]="isSelected(reaction)" (click)="selectedReaction.set(reaction)">
        {{ reaction }}
      </li>
      }
    </ul>
    <span>Selected Reaction: {{ selectedReaction() }}</span>
    <button class="btn btn-primary" (click)="addReaction()">More Reaction</button>
    <button class="btn btn-primary" (click)="removeReaction()">Less Reaction</button>
  `,
  styles: ``,
})
export class LinkedSignalsComponent {
  reactions = signal<string[]>(["🤩", "😒", "🤔"]);
  // selectedReaction = linkedSignal(() => this.reactions()[-1]);

  selectedReaction = linkedSignal<string[], string>({
    source: () => this.reactions(),
    computation: (source, previous) => {
      return (
        source.find((reaction) => reaction === previous?.value) ?? source[0]
      );
    },
  });

  addReaction = () => {
    this.reactions.set(["🤣", "😊", "😎", "😯", "🫠"]);
  };

  removeReaction = () => {
    this.reactions.set(["🤩", "😒", "🤔"]);
  };

  isSelected = (reaction: string) => {
    return this.selectedReaction() === reaction;
  };
}
```
