# Observables

## Introduction

An `Observable` is a wrapper around asynchronous data. We use an observable to handle asynchronous data. It is a function that converts the ordinary data stream into a `observable` one.

## Synchronous Programming

Javascript is a **single-threaded** programming language.The code is executed line by line one after the other. Synchronous code is blocking in nature.

## Asynchronous Programming

Javascript has a built-in function `Promise` to handle asynchronous code. Asynchronous code does not execute in single-threaded. It gets executed in the background.

## Promise vs Observables

| Promise                                                                      | Observables                                                    |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------- |
| It cannot handle stream of asynchronous data.It always return a single value | Handles stream of asychronous data. It returns multiple values |
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
