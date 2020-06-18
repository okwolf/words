---
layout: post
title: Ban software defects
description: Testing considered harmful
image: /images/fukushima-protesters.jpg
---

## The frontend JavaScript ecosystem is full of magical tools. 🔮

There are a range of UI frameworks available from ones with built-in solutions for everything to lightweight libraries for building only what you need in a modular fashion. You have options for writing your view as a template language for interpolating HTML or syntactic sugar for plain JavaScript objects known as the virtual DOM. There are different philosophies for state management from using implicit/imperative `Proxy` objects to the explicit/declarative unidirectional data flow approach. Application bundlers come in different varieties from zero configuration with lots of automagic to highly customizable to the point of being overwhelming.

The available options for unit test libraries are fundamentally less diverse. Your choices are as mundane as implicit globals or explicit library exports. There's a handful of popular styles of writing test assertions for different types of data and waiting on asynchronous results. These are just variations on the same theme. Write imperative code that organizes and names your tests and then use more imperative code inside a callback that throws exceptions to signal test failures. Writing this style of test code often requires using mocks, spies, and secret agents. Not a single pure function in sight.

It's the 21st century and we're living in the middle of a declarative renaissance, but our frontend testing tools are still stuck in the imperative JavaScript mindset of yesteryear. I would offer that this is one of the biggest factors discouraging test-first practices and in some cases outright hostility to the point of even avoiding test-later approaches. On top of all that, take a look at the price we are paying for installing just one of the most popular of these testing tools:

```console
$ npm i -D jest

> node install

+ jest@23.4.2
added 583 packages in 29.795s

$ du -hd 0 node_modules
 59M node_modules
```

When I saw this, I thought for sure I was just missing something. When I asked around the general consensus was "that's just how things are". Folks have acquiesced to the bloat in their test software like a wrongful prison sentence for a crime they didn't commit. I was writing lots of boilerplate and using less than 1% of the functionality I'm digitally paying for.

I was so shocked and disappointed by this state of affairs that I choose to write a new type of test library, against my own better judgement. That library is known as [Tead](https://github.com/teadjs/tead). I built this tool from scratch and only included the few pieces I needed for testing pure declarative code. The resulting package is a bit slimmer:

```console
$ npm i -D tead

+ tead@0.4.4
added 2 packages in 0.772s

$ du -hd 0 node_modules
284K node_modules
```

## Enter tests as data

It turned out all I needed to describe my tests was a way to group them together with descriptions and provide two values to compare for the test assertion itself. The only communities I saw getting this right are outside of JavaScript. [Elm](https://elm-lang.org) — for example, already pushes a purely functional approach and provides [a test library](https://package.elm-lang.org/packages/elm-explorations/test/latest) that falls more closely in line with my philosophy here, where each test must return an `Expectation` that is evaluated to determine if the test passed.

Elm also advocates for _effects as data_ for their approach to removing side effects from application code and returning data that describes what effect you want performed on your behalf and what to do with the results, if any. A native JavaScript approach for this will be shipping in the next major version of the [Hyperapp](https://github.com/hyperapp/hyperapp/pull/726) web app micro-framework and is available today in my [`hyperapp-fx`](https://github.com/hyperapp/fx) library. I decided to take this all the way to its logical conclusion and provide _tests as data_.

{% include youtube.html id="6EdXaWfoslc" %}

### _Let's take a look 👀_

Tead tests are written as a data structure representing test assertions and named groupings thereof. Here is an example of just such a value:

```js
// sum is a function that takes two arguments and adds them
{
  "sum can add": {
    zeros: [sum(0, 0), 0],
    "positive numbers": [sum(1, 2), 3],
    "negative numbers": [sum(-1, -2), -3],
    "mixed sign numbers": [sum(1, -2), -1]
  }
}
// If the sum functions works correctly the above will be equal to:
{
  "sum can add": {
    zeros: [0, 0],
    "positive numbers": [3, 3],
    "negative numbers": [-3, -3],
    "mixed sign numbers": [-1, -1]
  }
}
```

Notice how the only function called is the one under test without any test boilerplate. Object keys are used to group and describe tests. Array values represent 2-tuples for test expectations. The order of the elements in the expectation is `[actual, expected]` and differences will be reported as test failures. You can think of this expression as representing a more compact version of `expect(action).toEqual(expected)` purely using data. Anyone who tells you that Tead is actually TEAD and stands for **T**est **E**verything **A**s **D**ata… is thinking more than I was when I named it.

### testable code

I used a functional style to write most of Tead itself, with the exception of the file and console parts. But I kept much of the other code pure to take in your _tests as data_ exported by the test files, evaluate the assertions described in the data, and transform the results into values that will print nicely to the console. As a bonus, it runs way faster than any other solution I've seen. In watch mode it flashes almost too fast to believe the tests ran again, even with hundreds in the project. It almost creates the illusion that the tests ran before I saved my changes.

If by now you're wondering how to test async code or other side effects with Tead, you don't. This tool is very intentionally optimized for testing pure functions that always output the same result for the same input without affecting anything outside the scope of that function. Tead works best when matched with a runtime for _effects as data_ (as described above), so that the values returned by your business logic are comparable for equality, which keeps the code pure and testable. If you can accept that limitation as a feature and not a bug then your reward will be the most testable application code you'll ever find.

Adhering to this mantra is perhaps easier said than done. This won't work for library authors writing solutions for pushing the dirty impure logic to the outer edges of your software (such as myself), but it will keep your application code that uses them pure. Authors still need to write the runtime and effects that empower such style of programming. Tead is not meant for this purpose, since you will need mocks/stubs/spies to test the dirty work performed by these building blocks. This is a use case the existing test tools have a huge head start on, and should continue to be used. Luckily you only need do this once, and from then on treat that implementation as a black box you're not concerned with. Your application logic remains simple and focused on your business problem.

![](/images/fork-in-road.jpg)

## Alternatives

Perhaps now I've sold you on the value of _tests as data_, but not the implementation I choose for representing that data. Let's consider a few alternate types of data values that could be used to encode tests and why they weren't chosen.

### tagged template literals 🏷

These are all the rage right now thanks to libraries like [`styled-components`](https://www.styled-components.com), [`emotion`](https://emotion.sh), and [`hyperx`](https://github.com/choojs/hyperx#virtual-dom-node-example). They define a function for parsing and interpolating multiline string values. One possible API for writing tests this way might look like this:

```js
test`
sum can add
  zeros: ${sum(0, 0)} equals ${0}
  positive numbers: ${sum(1, 2)} equals ${3}
  negative numbers: ${sum(-1, -2)} equals ${-3}
  mixed sign numbers: ${sum(1, -2)} equals ${-1}
`;
```

This is essentially implementing an interpreter for an entire Domain-Specific Language. There's new syntax to learn and the concepts of test groupings/labels have been tangled with the definitions of the test assertions in a single compound value that should be broken down. On top of all that, I'm too lazy to implement and document the complexity of this beast.

### arrays all the way down 🐢

If you're convinced that the test data should be decomposed then maybe you don't agree with how I choose to do so. Perhaps the holy grail of test definitions lies in taking the Lisp analogy to its logical conclusion and using lists/arrays for grouping tests in addition to writing assertions. Here's what that might look like:

```js
[
  "sum can add",
  ["zeros", sum(0, 0), 0],
  ["positive numbers", sum(1, 2), 3],
  ["negative numbers", sum(-1, -2), -3],
  ["mixed sign numbers", sum(1, -2), -1]
];
```

I would argue that this approach is actually too simple. This also introduces ambiguity to the API since it's difficult to tell the difference between a test comparing two arrays and a test grouping with a label and two child tests. After enough levels the nested arrays will become more difficult to read than nested objects. Another scheme considered, but rejected.

### object-oriented tests 👩‍🔬

A different option would be to use objects for both test groupings and assertions along the lines of:

```js
{
  "sum can add": {
    zeros: {
      expect: sum(0, 0),
      toEqual: 0
    },
    "positive numbers": {
      expect: sum(1, 2),
      toEqual: 3
    },
    "negative numbers": {
      expect: sum(-1, -2),
      toEqual: -3
    },
    "mixed sign numbers": {
      expect: sum(1, -2),
      toEqual: -1
    }
  }
}
```

Regardless of what properties one chooses for the test assertion object, those become reserved names that must be documented and understood by users. Using objects here are a bit heavyweight for what fundamentally is a 2-tuple of values used to verify actual results match the expected ones. I'll admit that not going down this path was mostly a matter of style and preference vs ideology.

If you enjoyed this article please read part one that covers some background material related to simplicity as it applies to testing.

[Test code, not sanity](/test-code-not-sanity)

Also read my prior article on Hyperapp and how it supports functional programming that can be tested by tools like Tead.

[Hyperapp for Redux refugees](/hyperapp-for-redux-refugees)
