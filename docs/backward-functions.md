# Backward Functions

Cycle.js introduces the concept of a "Backward Function", which is very important for the
framework. It is a solution to the circular dependency problem that arises from the
cyclic structure between Model, View, Intent. Model depends on Intent, Intent depends on
View, View depends on Model.

```
  Model <───────┐
    │           │
    │           │
    V           │
  View ─────> Intent
```

Each of these components is supposed to work like a function, i.e., takes input, releases
output. However, it is impossible to express this using JavaScript functions. Here is a
naïve attempt:

```javascript
var x = foo(z);
var y = bar(x);
var z = baz(y);
```

`z` needs to be defined before `x` since we call `x = foo(z)`. But there isn't any way we
can reorder that to make it work. If we could somehow magically pass the arguments by name
and not by their values, then this problem could be solved as such:

```javascript
function addThree(a) {
  return a + 3;
}
var y = addThree(x); // x is still undefined here
var x = 2; // after this assignment, y should become 5
```

Certainly `addThree` is not a normal JavaScript function. Also, that snippet above assumes
that values are reactive, in the sense that `y` will change (according to `addThree`)
whenever `x` changes.

Hence, to solve the circular dependency problem in a reactive context, we can take
advantage of the fact that our values here are RxJS Observables as
inputs and outputs.

## What is a Backward Function?

A `BackwardFunction` is a JavaScript object that outputs RxJS Observables before it
receives Observables as inputs. In this sense, it works "backwards": the result is ready
before the input parameter is given. It is not a JavaScript `Function`, but as a
computation it has no side effects other than outputting RxJS Observables.

To provide a `BackwardFunction` an input, call the `inject(input)` function:
`backwardFn.inject(inputObject)`.

Because the output of a Backward Function exists before the input is given, this also
means that each Backward Function can only yield one output. A Backward Function is
therefore not reusable for multiple inputs.

## Anatomy of a Backward Function

You create a `BackwardFunction` by calling

```javascript
var backwardFn = Cycle.defineBackwardFunction(inputInterface, definitionFn);
```

The output is an instance of `BackwardFunction`, which contains the actual output RxJS
Observables that you expect, plus the `inject` function. `definitionFn` is a normal
JavaScript function with an input object as parameter, and must return an object whose
properties are RxJS Observables. `inputInterface` is an array of strings defining which
properties must exist in the input object given to `definitionFn`.

Internally, `BackwardFunction` works by creating a stub for the input, creating RxJS
Subjects with the names given by `inputInterface`. This stub is then given to the
`definitionFn` as the input parameter, which returns RxJS Observables depending on the
stub subjects. When `inject` is called with the real input object, all the events in the
Observables in the input (named according to `inputInterface`) are forwarded to the stub
observables. The stub, therefore, works as an early replacement to the real input, so we
can get an output without requiring the input. `inject` just commands the stub to mimic
the input.

As an example, if we call

```javascript
var backwardFn = Cycle.defineBackwardFunction(['foo$', 'bar$'], function(input) {
  return {
    quux$: Rx.Observable.merge(input.foo$, input.bar$)
  }
});
```

then `backwardFn` with be an object instance of `BackwardFunction` with the following
properties:

```javascript
{
  quux$ // Rx.Observable
  inject // Function
  clone // Function
}
```

## Common pitfalls

- **Backward Functions cannot be re-called like normal functions.** A `BackwardFunction`
  instance is the output, not a reusable computation. In this sense, it cannot be reused
  or called on other inputs like normal functions are. To reuse a Backward Function,
  call `clone()` on it to get another `BackwardFunction` (in essence, a different output).
- **Calling `inject` twice.** It should be called only once.
- **Using input properties not defined in `inputInterface`**. If you use `input.foo$`
  inside `definitionFn`, and `'foo$'` is not in the given `inputInterface` array, then
  `foo$` will be undefined in `definitionFn`. Make sure that you only use input properties
  that are defined in the `inputInterface`.
- **Not using RxJS Observables**. All properties in the input object should be RxJS
  Observables, and all properties in the output object of `definitionFn` should be RxJS
  Observables too. If you only need to output a single value, use `Rx.Observable.just()`.
  Use the convention of suffixing with `$` to denote that the property is an Observable.
