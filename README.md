# Fantasy Observable

A specification for interoperability of push-based data sources in JavaScript.

- - -

This project provides a specification for the `Observable` type, along with its satellite types `Observer` and `Subscription`. It is compatible with definitions of Observables and Streams in libraries such as [RxJS](http://reactivex.io/rxjs), [most.js](https://github.com/cujojs/most), [Kefir](http://rpominov.github.io/kefir), and [xstream](https://github.com/staltz/xstream).

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Overview

Fantasy Observables represent push-based data sources. When transporting data from a *Producer* to a *Consumer*, the process may be *Pull* or *Push*. In Pull systems, the Consumer determines when it receives data from the Producer. In Push systems, the Producer determines when to send data to the Consumer. Fantasy Observable is a Push system.

In this system, the Producer is an *Observable* and the Consumer is an *Observer*. The observable collection of data may be *invoked* at the Producer with the method `observable.subscribe(observer)`, and from that point onwards data is *pushed* to the `observer`. No further invocation is required at the `observable`. The `subscribe` function returns a *Subscription*. This subscription has an `unsubscribe` method which can be invoked with no arguments to cancel the transportation of data from the Observable to the Observer.

## Observer

An *Observer* is a JavaScript object with three REQUIRED functions as properties: `next`, `error`, `complete`. Expressed as a TypeScript type:

```typescript
interface Observer {
  next: (x: any) => void;
  error: (e: any) => void;
  complete: (c?: any) => void;
}
```

The `next` and `error` functions MUST receive an argument, but the `complete` function MAY receive an argument.

Observers represent consumers of push-based data transportation. Data MUST be given to the `next` function by invoking this function with the data as argument. Errors occuring in the data producer MUST be given to the `error` function by invoking this function with the error as argument. The `complete` function MAY be invoked by the producer to inform the consumer that no more data nor errors will be delivered.

## Subscription

A *Subscription* is a JavaScript object with one REQUIRED function as property: `unsubscribe`.

```typescript
interface Subscription {
  unsubscribe: () => void;
}
```

A Subscription represents a single execution of a push-based data producer. Its only feature is to allow cancellation of the execution.

## Observable

An *Observable* is a JavaScript object with the REQUIRED `subscribe` function as property.

```typescript
interface Observable {
  subscribe: (observer: Observer) => Subscription;
}
```

Observables represent producers of data, delivered in Push style to Observers. Data MAY be delivered to the Observer functions in the same event loop as `subscribe` was invoked, or in subsequent event loops. The `subscribe` method MUST return a Subscription object.

## Observable-Observer contract

When an Observable and an Observer are connected through a Subscription initiated by a `subscribe`, the `next` method of the Observer MAY be invoked multiple times. The Observable implementation MUST conform to the so-called Observable-Observer contract, which specifies that:

- `next` MAY be invoked
- `error` MAY be invoked
- `complete` MAY be invoked
- Observer methods MUST NOT be invoked after the invocation of `error`
- Observer methods MUST NOT be invoked after the invocation of `complete`

The above contract may be expressed as a regular expression, specifying that `next` may be invoked zero or multiple times, but if either `error` or `complete` are invoked, no other Observer method invocation can occur:

```
(next)*(error|complete)?
```

# Optional

The following concepts are OPTIONAL for compliance with Fantasy Observable.

## `from(observable)`

It is RECOMMENDED that reactive programming libraries compatible with Fantasy Observable support the `from` method to convert an Observable to its corresponding type in the reactive programming library.

```typescript
interface ObservableEquivalent<T> {
  from(observable: Observable): T;
}
```

## Extra type: Subject

A *Subject* is both an Observable and Observer. It is both a Producer and a Consumer. You may use its Observer methods to send data to the Subject, while you may use the Observable `subscribe` method to consume data from the Subject.

```typescript
interface Subject extends Observable, Observer {}
```

# License

Creative Commons - CC BY 3.0 http://creativecommons.org/licenses/by/3.0/
