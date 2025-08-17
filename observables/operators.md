# RxJs Operators

## Introduction

Operators are functions. There are two kinds of operators.

- Pipeable Operators
- Creational Operators

## Pipepable Operators

**Pipeable Operators** are the kind that can be piped to `Observables`. It is a function that takes an Observable as its input and returns another Observable. It is a pure operation, the previous Observable stays unmodified.

**Pipeable Operator Factory** is a function that takes parameters to set the context and return a Pipeable Operator.

## Creation Operators

**Creational Operators** are functions that can be used to create an Observable with some common predefined behavior or by joining other obseravble.

### of

It converts the arguments to an observable sequence. It sends a complete signal at the end.

```typescript
ofOperator = of(1, 2, 3).subscribe({
  next: (value) => console.log(value),
});
```

```
Output
1
2
3
```

### from

It creates an Observable from an array , array-like object , promise , iterable object or an observable-like object.

```typescript
fromOperator = from(["A", "B", "C"]).subscribe({
  next: (value) => console.log(value),
});
promise = new Promise<string>((resolve, reject) => {
  resolve("Rxjs");
});

from(this.promise).subscribe((result) => {
  console.log(result);
});
```

```
Output
A
B
C
RxJs
```

### fromEvent

Creates an Observable that emits events of a specific type coming from the given event target.

```html
<div>
  <div id="items-list">Items</div>
  <button type="button" #userButton>Create item</button>
</div>
```

```typescript
  fromEventObservable$;
  addItems(id: number) {
    const div = document.createElement('div');
    div.innerHTML = `Item ${id}`;
    document.getElementById('items-list').appendChild(div);
  }

  ngAfterViewInit(): void {
    let count = 0;
    const buttonElement = this.btnElement()?.nativeElement;
    this.fromEventObservable$ = fromEvent(buttonElement, 'click').subscribe({
      next: (event) => {
        console.log(event);
        console.log('Button clicked!');
        this.addItems(++count);
      },
    });
  }
```

## Transformation Operators

### map

It is used to transfrom the data emitted by the source observable , and emits the resulting values as an Observable.

```typescript
from([3, 4, 5, 6, 7])
  .pipe(map((x) => x * 4))
  .subscribe((value) => console.log(value));
```

```
Output
12
16
20
24
28
```

## Filtering Operators

### filter

It is used to filter the items emitted by the source observable by only emitting those that satisfy a specified predicate.

```typescript
from([3, 4, 5, 6, 7])
  .pipe(
    map((x) => x * 4),
    filter((x) => x % 8 == 0)
  )
  .subscribe((value) => console.log(value));
```

```
Output
16
24
```
