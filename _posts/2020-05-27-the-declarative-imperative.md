---
layout: post
title: The declarative imperative
description: A declaration of independence
image: /images/declaration-of-independence.jpg
---

> "But how on earth will we test it?"

This was my question to Sophia. She told me to "just use the marbles."

> "I can't, I think I've lost my marbles writing this code. I'm already programming with pure functions. Why can't I just assert the actual output equals what's expected?"

Sophia was offended: "Because that's not how programming with asynchronous streams of events works!"

> "You mean that's not how programming with Observables works."

"You're just not embracing the right mental model," she retorted and stormed off. I sat in quiet contemplation wondering what had caused this escalation in our discussion. It annoyed me on a deeply personal level that I was able to develop these tidy reusable functions that individual operators used, but once I flowed events through them I lost all the tools I knew and loved.

My inner TDD zealot was having a very hard time embracing RxJS. I longed for the freedom of building my application logic like assembling little lego blocks from the comfort of my console, REPL, or other similar live evaluation environments. Tracing bugs from individual steps in the flow felt much harder than it should be. And when it comes to marble testing, it's probably best to just show an example of what they look like from the RxJS docs:

```js
import { TestScheduler } from "rxjs/testing";

const testScheduler = new TestScheduler(
  (actual, expected) => {
    // asserting the two objects are equal
    // e.g. using chai.
    expect(actual).deep.equal(expected);
  }
);

// This test will actually run *synchronously*
it("generate the stream correctly", () => {
  testScheduler.run(helpers => {
    const {
      cold,
      hot,
      expectObservable,
      expectSubscriptions,
      flush
    } = helpers;
    const e1 = cold("-a--b--c---|");
    const subs = "^----------!";
    const expected = "-a-----c---|";

    expectObservable(
      e1.pipe(throttleTime(3, testScheduler))
    ).toBe(expected);
    expectSubscriptions(e1.subscriptions).toBe(subs);
  });
});
```

Yuck. `expectObservable` and `expectSubscriptions` seem like somewhat reasonable test helpers, although I'd prefer not to need them. `cold`, `hot`, and `flush` are beyond unintuitive, and after reading the documentation are downright confusing. We even need to understand how schedulers work just to verify that our application logic is correct!

## hook, line, and sinker

Those of you thinking other popular libraries like React do a better job of staying pure and declarative, think again. In the Paleolithic Age, React only supported state and lifecycle hooks in `class` components, with all the baggage that comes with them.

Then [Redux](https://redux.js.org) came onto the scene, and for a time the reducers were testable, and there was much rejoicing. Function components could use state and remain pure. But there was still no good solution for the lifecycles and async code. Also, the natives were getting restless about the boilerplate.

Dan Abramov introduced [hooks](https://reactjs.org/docs/hooks-intro.html) at React Conf 2018 to great fanfare. Even though hooks are "completely opt-in" and not intended to replace `class` components, the community quickly embraced them and by now most React libraries and apps have adopted them.

{% include youtube.html id="dpw9EHDh2bM?start=1071" %}

The major promise of hooks is that your function components gained superpowers - giving them access to the remaining React APIs that previously required writing a `class` component. Let's take a look at an example from the React docs of what using these hooks looks like:

```jsx
import React, { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Did something die in here? That once-pure component as a function of props now takes no parameters at all and is full of side-effectful hooks. Hooks also come with [archaic rules](https://reactjs.org/docs/hooks-rules.html) that expose the implementation details giving meaning to the order in which they are called. Hooks have a rather poor experience [in devtools](https://github.com/facebook/react/issues/16474), even years later.

To top it all off - forget about unit testing the logic in components that use hooks. The bits that call React hook APIs won't work outside React, thanks to the rules of hooks™️. You're going to need to mount a full component in an environment that has React, the DOM, a gorilla holding a banana, and the entire jungle.

![](/images/gorilla-banana.jpg)

## get hyped

Fear not, dear reader, for there are solutions out there embracing a functional and declarative approach to building user interfaces. The [Elm](https://elm-lang.org) language and architecture are the most established players in this space since 2012. Instead of mixing side effects into the view, they are separated from the app code and represented with a data structure that the Elm runtime uses to execute your intent. This approach is called effects as data. Lispy languages adopt a similar philosophy of code as data.

{% include youtube.html id="6EdXaWfoslc" %}

Those who prefer to stay in the realm of JavaScript - instead of treating it as a compile target - while still retaining the benefits of Elm have a better option. [Hyperapp](https://hyperapp.dev) is the tiny framework for building purely functional, declarative web apps in JavaScript. The latest version comes with support for representing effects as data, along with subscriptions for those pesky asynchronous event streams mentioned earlier. Like Elm, the view is always a pure function of the state. Let's take a look at one possible Hyperapp version of the previous React hooks example component:

```js
import { app } from "hyperapp";
import { div, p, button } from "html-helper-library";
import { DocumentTitle } from "some-fx-library";

const Inc = count => [
  // next state value
  count + 1,
  // effects as data returned by DocumentTitle
  DocumentTitle(`You clicked ${count + 1} times`)
];

app({
  init: 0,
  view: count =>
    div(
      p(`You clicked ${count} times`),
      button({ onclick: Inc }, "Click me")
    ),
  node: document.getElementById("app")
});
```

Much better. The `view` is now pure as the driven snow. That's because the suck is all contained within the libraries for Hyperapp and effects represented as data. Those libraries are responsible for handling all the DOM manipulation and other side-effectful code in our app, we merely need to pass data around to use them. Calling the `DocumentTitle` function doesn't perform any side effects on the `document.title` but instead returns an object telling Hyperapp what side effects to perform on your behalf. Tuples of `[state, ...effects]` are used to declaratively represent the results of actions as their return value.

One of those libraries for effects is my own [`hyperapp-fx`](https://github.com/okwolf/hyperapp-fx), which makes working with Hyperapp more pleasant. FX are included for working with everything from HTTP to `localStorage` and WebSockets. These FX can be treated as black boxes that I'm responsible for writing and testing. All you need to do is return the right FX with the right props in response to the right action and Hyperapp will do the rest, with help from my FX library.

## But how on earth will we test it?

Back on the topic that we stared with - how exactly do these declarative effects as data help with the development, testing, and debugging experience? All the app logic is contained in functions that return real, honest values that can be printed, inspected, or asserted to equal their expected results. This is true for the VDOM returned by the view, the tuples returned by actions for the next state and effects as data, and even the effects themselves.

The only thing needed to test code this simple is the ability to assert the actual value returned from calling a function with a given input deeply equals the expected value. My [`tead`](https://github.com/teadjs/tead) library is built for exactly this purpose. There's almost no API to learn. Unlike other testing libraries, the test descriptions are written declaratively. Objects are used for grouping and describing tests and arrays are used to represent test assertions as `[actual, expected]` values.

Here's an actual sample of what some basic tests might look like for the `Inc` action from the earlier Hyperapp example:

```js
import { DocumentTitle } from "some-fx-library";
import { Inc } from "./actions";

export default {
  Inc: {
    "is a function": [typeof Inc, "function"],
    "increments and updates the title": {
      "for zero": [
        Inc(0),
        [1, DocumentTitle(`You clicked 1 times`)]
      ],
      "for nonzero": [
        Inc(99),
        [100, DocumentTitle(`You clicked 100 times`)]
      ]
    }
  }
};
```

Not a line of imperative code in sight. The app code and tests for that code both use a fully declarative syntax. A double whammy. The benefits of the simple approach are now becoming clear. This comes with a side benefit of stripping the tests clean of boilerplate, boiling them down to their very essence. This simpler approach also runs noticeably faster, and in watch mode, hundreds of tests can run in the blink of an eye. For way more information on `tead` read [my previous post](/ban-software-defects) on the subject.

Sophia was unconvinced by my new approach: "That's an awful lot to learn just to make testing easier." I responded, "No more than learning to think of your code as marbles," in my last attempt to explain. "It's not just the code," she said, "it's the new tools, the weird syntax for markup, mashing all my state together in one object, and all the other complexity!"

"Things are simpler with the declarative approach since there are fewer concepts and APIs to learn, but you must fully embrace all of it to reap the benefits I've described." I offered a friendly competition to settle the debate: "Tell you what, let's each build the next feature in our preferred styles and compare results tomorrow. We'll get Jimmy, Andrea, and Louis to be the judges. The winner gets to continue using their paradigm without argument from the other." I'll let you guess who won.

### And that's how I declared my declarative independence.
