# Signals

## Table of Contents

- [Introduction](#introduction)
- [Characteristics](#characteristics)
- [Types of Signals](#types-of-signals)
- [Writable Signals](#writable-signals)
- [Read-only Signals](#read-only-signals)
- [Computed Signals](#computed-signals)
- [Effects](#effects)
- [Equality Functions](#equality-functions)
- [Read without Tracking Dependencies](#read-without-tracking-dependencies)
- [Effective Cleanups](#effective-cleanups)

## Introduction

A signal is a essentially a wrapper around a value that notifies consumers (functions,components or services) when the value changes.

This allows for a reactive data flow, where components and services can subscribe to signals and react to changes in the value.

Signals in Angular are a way to manage state and reactive data flows, similar to Observables or BehaviorSubjects.

Signals are more lightweight and built directly into the framework, making them easier to use and understand.

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
    console.log(
      `User set to ${currentUser()} and the counter is ${untracked(counter)}`
    );
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