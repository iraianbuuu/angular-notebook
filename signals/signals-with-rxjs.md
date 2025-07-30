# Signals with RxJs

## Table of Contents

- [`toSignal`](#to-signal)
  - [Manually Destroying Observables](#manually-destroying-observables)
  - [Error and Completion](#error-and-completion)
  - [`toObservable`](#to-observable)
  - [`outputFromObservable`](#output-from-observable)
  - [`outputToObservable`](#output-to-observable)

### `toSignal`

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

### Error and Completion

If an Observable used in `toSignal` produces an error, that error is thrown when the signal is read.

### `toObservable`

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

### `outputFromObservable`

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
<app-rxjs-interop
  (outputFromObservable)="onCounterChange($event)"
></app-rxjs-interop>
```

### `outputToObservable`

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