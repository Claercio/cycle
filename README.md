# Cycle

A web application framework using the Reactive Model-View-Intent architecture and Virtual
DOM Rendering.

* **Unidirectional Dataflow**: based on the Model-View-Intent architecture, data moves
  from Model to View, events move from View to Intent, and Intent sends "user intentions"
  to the Model. They are tied together as a cycle.
* **Purely Reactive**: all components are event-driven and [RxJS](https://github.com/Reactive-Extensions/RxJS)
  is a hard dependency, which simplifies all code related to events, asynchronicity, and
  errors.
* **Virtual DOM Rendering**: Views re-render completely whenever Models emit any event.
  The use of [virtual-dom](https://github.com/Matt-Esch/virtual-dom) keeps performance
  optimal by patching the actual DOM with only the minimum necessary changes.
* **MVI is functional**: each component in the MVI cycle behaves like a function,
  receiving events as input, and outputting events. No side effects. This makes it
  convenient for composing with other components, or for automating tests.

## Example

```javascript
var HelloModel = Cycle.defineModel(['changeName$'], function (intent) {
  return {
    name$: intent.changeName$.startWith('')
  };
});

var HelloView = Cycle.defineView(['name$'], function (model) {
  return {
    vtree$: model.name$
      .map(function (name) {
        return Cycle.h('div', {}, [
          Cycle.h('label', 'Name:'),
          Cycle.h('input', {
            'attributes': {'type': 'text'},
            'ev-input': 'inputText$'
          }),
          Cycle.h('hr'),
          Cycle.h('h1', 'Hello ' + name)
        ]);
      }),
    events: ['inputText$']
  };
});

var HelloIntent = Cycle.defineIntent(['inputText$'], function (view) {
  return {
    changeName$: view.inputText$
      .map(function (ev) { return ev.target.value; })
  };
});

Cycle.renderEvery(HelloView.vtree$, '.js-container');
Cycle.link(HelloModel, HelloView, HelloIntent);
```

Notice that each of the 3 components has a neighbour component as input, and each outputs
an object mostly containing Rx Observables. At the bottom, `Cycle.renderEvery` subscribes
to changes of `HelloView.vtree$` and renders those virtual elements into `.js-container`
in the DOM. `Cycle.link` just ties all three Model, View, and Intent together by telling
them that they depend on each other circularly.

## Installing through npm

`npm install cyclejs`

## Installing through Bower

`bower install cycle`

## Learn more

Model-View-Intent is similar to Flux, and Virtual DOM Rendering is inspired by React,
however there are several differences worth paying attention. Read the [seminal blog post
that lead to the creation of Cycle.js](http://futurice.com/blog/reactive-mvc-and-the-virtual-dom).

It has [virtual-dom](https://github.com/Matt-Esch/virtual-dom) and [RxJS](https://github.com/Reactive-Extensions/RxJS)
as hard dependencies. Cycle is a "glue" framework providing helpers for building the
Model-View-Intent architecture properly with those technologies. Cycle's code itself is
still under 300 lines of code only.

## Why?

Why would you use Cycle.js instead of other web frameworks such as Angular and React? Here
are a couple of strong reasons:

- **Separation of concerns.** The reactive pattern for Models, Views, and Intents makes it
  possible for no component to have functions such as `updateSomething()` which inherently
  create coupling. You can write code with single responsibilities throughout. For
  instance, the View just takes model data and renders virtual elements, it doesn't even
  have callbacks to handle events.
- **Superb testability.** Everything is a JavaScript function or a [BackwardFunction](https://github.com/staltz/cycle/blob/master/docs/backward-functions.md),
  so testing is mostly a matter of feeding input and inspecting the output. You can even
  test styles if you use functions to output your styles instead of using CSS files.
- **Rendering separated from View.** Contrary to what you expect, a View in Cycle.js does
  not directly render anything to the browser. Instead, it just outputs virtual DOM
  elements. This allows for better testability, and also makes it easier to implement
  UI skins since it is just a matter of introducing Skin components taking Views as input
  and doing some post-processing before the actual rendering.
- **Welcomes immutable and stateless programming.** Cycle.js is built for, in
  combination with RxJS, a programming style that favors immutability and statelessness.
  This allows code to be clearer and less prone to bugs.

## Disclaimer

Cycle is in alpha mode, many changes to the API will likely occur before v1.0 is released.
Use this framework only for experiments before that. PS: we don't want to stay as alpha
forever, either. ;)

## LICENSE

The MIT License (MIT)

Copyright (c) 2014 Andre Staltz

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
