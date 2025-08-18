# Subjects

## Introduction

**Subjects** is a special type of `Observable` that allows values to be multicasted to many observers. The plain `Observables` are unicast , `Subjects` are multicast. They are like `EventEmitters`.

> Each subscribed Observer owns an independent execution of the Observable.

> `Subjects` are both `Observable` and `Observer`.

## Observables vs Subjects

| Observables    | Subjects                     |
| -------------- | ---------------------------- |
| Unicast        | Multicast                    |
| Cold           | Hot                          |
| Data producers | Data producers and consumers |

```typescript
subject = new Subject();
observable = new Observable((observer) => {
  observer.next(Math.random());
});

this.subject.subscribe({
  next: (x) => console.log("Observer A (Subject) :", x),
});

this.subject.subscribe({
  next: (x) => console.log("Observer B (Subject) :", x),
});

this.observable.subscribe({
  next: (x) => console.log("Observer A: ", x),
});

this.observable.subscribe({
  next: (x) => console.log("Observer B: ", x),
});

this.subject.next(Math.random());
```

```
Output

Observer A (Subject) : 0.535
Observer B (Subject) : 0.535
Observer A : 0.1345
Observer B : 0.4563
```

## BehaviorSubject

**BehaviorSubject** is a subject which can hold an initial value which emits no new value is emitted. BehaviorSubject emits a initial value or last emitted value for new subscriber.

```typescript
behaviorSubject = new BehaviorSubject(0);
this.behaviorSubject.subscribe((value) => {
  console.log("Observer A: ", value);
});

this.behaviorSubject.next(1);
this.behaviorSubject.next(2);

this.behaviorSubject.subscribe((value) => {
  console.log("Observer B: ", value);
});

this.behaviorSubject.next(3);

this.behaviorSubject.subscribe((value) => {
  console.log("Observer C: ", value);
});
```

```
Observer A:  0
Observer A:  1
Observer A:  2
Observer B:  2
Observer A:  3
Observer B:  3
Observer C:  3
```
