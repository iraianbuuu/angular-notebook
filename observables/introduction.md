# Observables

## Introduction

An `Observable` is a wrapper around asynchronous data. We use an observable to handle asynchronous data. It is a function that converts the ordinary data stream into a `observable` one.

## Synchronous Programming

Javascript is a **single-threaded** programming language.The code is executed line by line one after the other. Synchronous code is blocking in nature.

## Asynchronous Programming

Javascript has a built-in function `Promise` to handle asynchronous code. Asynchronous code does not execute in single-threaded. It gets executed in the background.

## Pull vs Push

**Pull** and **Push** are two different protocols that describe how a data producer can communicate with data consumer.

|      | Single   | Multiple   |
| ---- | -------- | ---------- |
| Pull | Function | Iterator   |
| Push | Promise  | Observable |

|      | Producer                              | Consumer                               |
| ---- | ------------------------------------- | -------------------------------------- |
| Pull | Passive: produces data when requested | Active: decides when data is requested |
| Push | Active: produces data at its own pace | Passive: reacts to received data       |

### Pull

In Pull systems, the Consumer determines when it receives data from the data Producer. The producer itself is unware of when the data will be delivered to the consumer.

> Every Javascript function is a Pull System.

### Push

In Push systems, the Producer determines when to send data to the Consumer. The Consumer is unaware of when it will receive that data.`Promises` are the most common type of Push System in Javascript.

- `Function`: Lazily evaluated computation that synchronously return a single value.

- `Generator`: Lazily evaluated computation that synchronously return a zeros to infinite values.

- `Promise`: Computation that may (or may not) eventually return a single value.

- `Observable`: Lazily evaluated computation that synchronously or asynchronously return a zeros to infinite values.

## Promise vs Observables

| Promise                                                                      | Observables                                                    |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------- |
| It cannot handle stream of asynchronous data. | Handles stream of asychronous data. |
|It always return a single value | It returns multiple values |
| It will return a data if no code is using that data                          | It return a data if someone is going to use that data          |
| It is native to **Javascript**                                               | It is provided by **RxJs** library                             |

## Observer Pattern

1. Observable : An **Event Emitter** emits an event.

- Next
- Error
- Completion

2. Observer :
   A **Subscriber/Event Listener** listens to an event.

- Subscribe()

3. Handler :
   A **Event Handler** handles the event.

- Next()
- Error()
- Completion()

## Creating and Using an Observable

There are three types of values an Observable execution can deliver :

- `Next`
- `Error`
- `Complete`

## Next

The `Next` notification sends a value such as number, string or an object. They represent actual data being delivered to a subscriber.

`app.ts`

```typescript
  data: number[] = [];

  observable$ = new Observable<number>((observer) => {
    setTimeout(() => observer.next(1), 1000);
    setTimeout(() => observer.next(2), 2000);
    setTimeout(() => observer.next(3), 3000);
    setTimeout(() => observer.next(4), 4000);
    setTimeout(() => observer.next(5), 5000);
  });

  showData() {
    this.observable$.subscribe({
      next: (value) => {
        console.log(value);
        this.data.push(value);
      },
    });
  }

  ngOnInit() {
    this.showData();
  }
```

### Error

The `Error` notification sends a Javascript error or exception. If an `Error` notification is delivered, then nothing can be delivered afterwards.

```typescript
  data: number[] = [];

  observable$ = new Observable<number>((observer) => {
    setTimeout(() => observer.next(1), 1000);
    setTimeout(() => observer.next(2), 2000);
    setTimeout(() => observer.next(3), 3000);
    setTimeout(() => observer.error(new Error('Something went wrong')), 3000);
    setTimeout(() => observer.next(4), 4000);
    setTimeout(() => observer.next(5), 5000);
  });

  showData() {
    this.observable$.subscribe({
      next: (value) => {
        console.log(value);
        this.data.push(value);
      },
    });
  }

  ngOnInit() {
    this.showData();
  }
```

### Completion

The `Completion` notification does not send a value.If an `Completion` notification is delivered, then nothing can be delivered afterwards.

```typescript
   data: number[] = [];

    observable$ = new Observable<number>((observer) => {
    setTimeout(() => observer.next(1), 1000);
    setTimeout(() => observer.next(2), 2000);
    setTimeout(() => observer.next(3), 3000);
    //setTimeout(() => observer.error(new Error('Something went wrong')), 3000);
    setTimeout(() => observer.next(4), 4000);
    setTimeout(() => observer.next(5), 5000);
    setTimeout(() => observer.complete(), 6000);
  });

  showData() {
    this.observable$.subscribe({
      next: (value) => {
        console.log(value);
        this.data.push(value);
      },
      error: (error: Error) => {
        console.error(error.message);
      },
      complete: () => {
        console.info('Completed');
      },
    });
  }

  ngOnInit() {
    this.showData();
  }
```
