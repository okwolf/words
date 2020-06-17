---
layout: post
title: Serving up simple
description: Fired up about the back end
image: /images/rocket-engine.jpg
---

Node.js is a concurrent connection marvel. I/O bound tasks can easily scale into the tens of thousands. This software is uniquely qualified for writing highly scalable microservices that can handle serious loads. The only piece missing is an API anyone wants to use. The full gory details are available in the [Node.js docs](https://nodejs.org/api/http.html), but here's a small incomplete taste of the fun you can expect when writing file servers in vanilla Node.js:

```js
const http = require("http");
const fs = require("fs");
const path = require("path");

const server = http.createServer((request, response) => {
  const filePath = `./public${
    request.url === "/" ? "/index.html" : request.url
  }`;
  const extname = String(
    path.extname(filePath)
  ).toLowerCase();
  mimeTypes = {
    ".html": "text/html",
    ".js": "text/javascript"
    // ...
  };
  const contentType =
    mimeTypes[extname] || "application/octet-stream";
  fs.readFile(filePath, (error, content) => {
    if (error) {
      if (error.code == "ENOENT") {
        fs.readFile("./404.html", (error, content) => {
          response.writeHead(404, {
            "Content-Type": "text/html"
          });
          response.end(content, "utf-8");
        });
      } else {
        response.writeHead(500);
        response.end(
          `Sorry, check with the site admin for error: ${error.code}`
        );
      }
    } else {
      response.writeHead(200, {
        "Content-Type": contentType
      });
      response.end(content, "utf-8");
    }
  });
  server.listen(8080);
});
```

Because of this - libraries exist to improve the experience of writing servers in Node.js - making the raw I/O power in its engine more accessible. The most popular of these solutions is Express, which happens to come with built-in file server support, giving us an equivalent working example with much less effort:

```js
const express = require("express");
const app = express();

app.use(express.static("public"));

app.listen(8080);
```

Serving files is nice and all, but to take things to the next level, we need to write API logic to dynamically handle requests at various endpoints. Express comes with a router and a middleware API built for handling all kinds of requests:

```js
app.get("/users/:userId/books/:bookId", (req, res) => {
  res.send(req.params);
});

// request URL: http://localhost:8080/users/34/books/8989
// req.params: { "userId": "34", "bookId": "8989" }
```

Those of you who have read my other posts will know I'm a bit of a functional programming fanatic and a [declarative programming](/the-declarative-imperative) devotee. This part of my brain enjoys seeing the `req` object with ordinary data representing the client's request. There are some methods and other junk attached to the request object, but I'll ignore those. The bigger issue is the `res` object, its API is entirely imperative with methods like `send()`, `redirect()`, `set()`, `append()`, and `end()`.

I wondered if there was a way to make the response work more like that request object. My answer turned into a library called [`fxapp`](https://github.com/fxapp/fxapp). I adapted the declarative front end programming model for the back end. This model was initially pioneered by [Elm](https://guide.elm-lang.org), made mainstream by [Redux](https://redux.js.org), then streamlined by [Hyperapp](https://hyperapp.dev).

In addition to supporting the traditional global state tree, `request` and `response` specific state is automatically merged with the global state that persists across requests for the user. To be extra helpful, I decided that properties directly under `request` and `response` would be merged for you as well.

### request state

The `request` state property in `fxapp` is heavily inspired by the `http.IncomingMessage` from Node.js and `req` from Express. Here's an example of some data you might find in a `request`:

```js
{
  // ...
  request: {
    method: "GET",
    url: "/path/other/123?param=value&multiple=1&multiple=2",
    path: "/path/other/123",
    query: { param: "value", multiple: ["1", "2"] },
    params: { id: "123" },
    headers: {
      Host: "localhost:8080"
    }
  }
  // ...
}
```

For requests that include content in their body, there will be a `body` property added. When the client sends a request body as proper JSON, there will also be a `jsonBody` property with the JSON-parsed value. Additional types of request data can be handled via [FX](#special-fx), explained below.

### response state

The `http.ServerResponse` from Node.js served as a source of inspiration for the `response` state property, except using an object with data to declaratively describe the response to send without actually calling any of the underlying imperative APIs. Here's an example of some data you might find in a `response`:

```js
{
  // ...
  response: {
    statusCode: HttpStatus.OK,
    headers: { Server: "fxapp" },
    text: "Hello World"
  }
  // ...
}
```

Other supported types of responses are `json`, `html`, piping the contents of a `filePath`, and `custom` for extending support to new types.

## updating state

While the `request` portion of the state is already populated by the library, to build a useful API, you'll need a way to fill in what the `response` should be for a given request. State updates are expressed as a pure function with parameters for the `state` and optionally, any added `props`; the function is then responsible for computing and returning the next `state`. These functions are similar to reducers in Redux.

Let's look at an example of such a function - responsible for adding data sent as JSON in the request body to an array of `heroes` stored in state:

```js
const AddHero = ({ request: { jsonBody }, heroes }) => {
  if (!jsonBody || !jsonBody.name) {
    return {
      response: { statusCode: HttpStatus.BAD_REQUEST }
    };
  }
  const hero = {
    id: heroes.length,
    ...jsonBody
  };
  return {
    response: {
      statusCode: HttpStatus.CREATED,
      json: hero
    },
    heroes: heroes.concat(hero)
  };
};
```

Notice how I don't need to import anything from `fxapp` to write my business logic. This example ties together all the concepts I've covered so far, but we haven't built anything useful yet. Our state management is pure and pleasant, but real-world servers need to interact with the impure and messy world, like the file server example earlier. I want to keep these parts of my codebase minimal and isolated from the rest of my server code.

> We're going to need just a hint of magic.

![](/images/sparkler-effect.jpg)

## special FX

The `fx` in `fxapp` is referring to the _effects as data_ paradigm coming from the Elm (and later Hyperapp) camp. I've already [written](/the-declarative-imperative#get-hyped) about [this](/ban-software-defects#enter-tests-asdata) multiple [times](/hyperapp-for-redux-refugees#effects-), but in summary: this approach encodes imperative code into a reusable data structure that the library feeds to a runtime for execution instead of running that code directly. This approach of working with data comes with many advantages, including improved live evaluation development, testing, and debugging experiences.

{% include youtube.html id="6EdXaWfoslc" %}

The data structure FX use is an object with a `run()` function where the magic happens. The library runtime will call `run()` with props that include any data passed in to configure this generic reusable magical black box along with a `dispatch` function. The `dispatch` function may be called with state update functions, FX, and arrays to batch them together or add optional props at the time of dispatch. For all the details of what's `dispatch`able, take a look [here](https://github.com/fxapp/fxapp#dispatchable-types).

Here's a pair of recyclable FX that handle reading/writing JSON-serialized data to/from files as an example:

```js
const ReadJsonFileEffect = {
  run: ({ path, dispatch, onSuccess, onError }) => {
    try {
      const data = fs.readFileSync(path);
      dispatch(onSuccess, JSON.parse(data));
    } catch (e) {
      dispatch(onError, e);
    }
  }
};

const WriteJsonFileEffect = {
  run({ path, data }) {
    fs.writeFileSync(path, JSON.stringify(data, null, 2));
  }
};
```

Notice how `ReadJsonFileEffect` dispatches different things depending on whether a parsable file exists at the given path. The runtime passes the parsed data or error object when executing the provided `onSuccess` or `onError` value. The `WriteJsonFileEffect` exists solely for the side effect of writing the data to a file and does not need to dispatch anything. _Not pictured_: FX may return a `Promise` if they are asynchronous, which will be considered still running until resolved or rejected.

### applied FX

Now that you have a basic understanding of FX, let's take a look at how we can wire them up to our `fxapp` application. This code builds on the previous examples by reading our heroes from a file on server start and writing the updates to the file when adding new heroes:

```js
const { app } = require("fxapp");

const ReadHeroesEffect = [
  ReadJsonFileEffect,
  {
    path: HEROES_PATH,
    onSuccess: (_, heroes) => ({ heroes })
  }
];

const WriteHeroesEffect = ({ heroes }) => [
  WriteJsonFileEffect,
  { path: HEROES_PATH, data: heroes }
];

app({
  initFx: [initialState, ReadHeroesEffect],
  routes: {
    POST: [AddHero, WriteHeroesEffect]
    // ...
  }
});
```

Notice how `ReadHeroesEffect` uses a 2 element array (representing a tuple) with `[fx, props]` to pass the `path` and `onSuccess` state update function to the effect as `props`. This state update receives an optional second parameter with the parsed data dispatched by `ReadJsonFileEffect`. `WriteHeroesEffect` composes concepts from the state update function with an FX tuple so that it can pass the `path` and `data` for writing to the `WriteJsonFileEffect`. I call these functions that transform the current state into FX actions.

Although I haven't introduced `app`, `initFx`, or `routes` yet, the discerning among you have probably already figured out what these do. The `app` function from `fxapp` is how we start an app with options. The `initFx` option for `app` specifies a value to dispatch (and finish running in the case of async FX) before starting the server and accepting requests. The `routes` option provides a built-in declarative routing solution, which in this case, will handle `POST` requests by updating the state to add the hero from the request and then write the updated heroes to a file. Next, let's take a look at how the declarative router works in more detail.

## it's about the route, not the destination

Routes are defined as a nested object structure with some properties having special meanings. The first matching route value is dispatched. Take a look at this sample set of routes:

```js
app({
  routes: {
    // GET /unknown/path
    _: fallbackAction,
    path: {
      some: {
        // GET /path/some
        GET: someReadAction,
        // POST /path/some
        POST: someAddAction
      },
      other: {
        // GET /path/other/123
        // { request: { params: { id: "123" } } }
        $id: otherAction
      }
    }
  }
});
```

For routes that match in the absence of a more specific route, use the special wildcard `_`. Routes with the name of an [HTTP request method](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) match any requests with that method. Routes beginning with `$` are reserved and define a path parameter for matching at that position. Additional info on `routes` is available [here](https://github.com/fxapp/fxapp#routes).

### advanced FX

The FX runtime included with `fxapp` incorporates innovations that help with the unique challenges of back end development. These include coordinating parallel vs. chained asynchronous operations, canceling processing the rest of a request for error/timeout cases, and deferring response handling until after all request logic has finished. For more information on these features of FX, check out the [docs](https://github.com/fxapp/fxapp#fx).

Here's a slightly more advanced example using one of these features for logging request/response information:

```js
const LoggingFx = [
  {
    run({ dispatch }) {
      const startedAt = Date.now();
      dispatch({
        request: {
          startedAt
        }
      });
    }
  },
  {
    after: true,
    run({ dispatch }) {
      dispatch(({ request, response }) => {
        const duration = Date.now() - request.startedAt;
        console.log(
          `${request.method} ${request.url} -> ${response.statusCode} in ${duration}ms`
        );
      });
    }
  }
];
```

This example uses a batch of two FX - the first one gets the current timestamp as its side effect and then dispatches a state update for storing that timestamp as the request start time. The second effect performs a `dispatch` to gain access to the current `request` and `response` state. It does some math on the duration of the request and then prints a summary of the request/response data to the console. This second effect uses the `after` option so that this effect will run after the request is fully processed and already sent if using one of the default response types.

Similarly to add support for `custom` responses, mark an effect as `after`, then when the `response` state corresponds to a `custom` type that this effect handles, make the appropriate calls to `serverResponse`. For decoding custom `request` types, the `serverRequest` is also available as one of the reserved props to FX.

## what's next?

If you're still reading this - chances are you agree with at least some of what I've built, and you might be wondering, "Is this project ready for primetime?" No, it's not yet. But if you like being an early adopter and living on the cutting edge of technology, I encourage you to try out the current alpha. I've hit 100% code coverage with my tests and feel comfortable that [`fxapp`](https://github.com/fxapp/fxapp) behaves as I designed it (even if that is different from what you expect).

Please report any bugs you come across along with features you'd like to see added. One of the top priorities on my roadmap is to start adding FX to the library for common use cases similar to my [`hyperapp-fx`](https://github.com/okwolf/hyperapp-fx) library. If you're interested in building something new on top of the same FX runtime, I'm making it available separately as the [`fxchain`](https://github.com/okwolf/fxchain) package.

In closing, remember:

### Keep your server simple and put fire in your back end 🔥
