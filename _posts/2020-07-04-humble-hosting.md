---
layout: post
title: Humble hosting
description: Nothing but necessities
image: /images/server-racks.jpg
---

## creativity strikes

Anyone who lived through the early days of web development can remember the feeling. Inspiration strikes with a brilliant new idea for creating a website, and you hurry over to the keyboard. Quickly you create a new HTML file, type in your markup, open the file in your browser of choice, and forget about Internet Explorer for a few precious minutes. In this long-since gone world, the learning curve was primarily understanding the nuances of different browser APIs - the tools used to create websites faded into the background.

Then investments were made in frontend tooling to smooth over browser differences, experiment with new language features, and provide whiz-bang features such as hot reloading. Since then, the adoption of web standards has dramatically improved, and we are living in a new era of web dev. For the first time in 30 years, you can write code conforming to the latest standards, and that code will work as-is for the ~95% of users with modern browsers. With this change in mind, it's time to reevaluate the tools we invented during the era when this wasn't the case.

If your idea involves using popular JavaScript libraries - often seeing your new prototype requires installing a whole slew of dependencies like Webpack, Babel, Parcel, or even an entire CLI. These can easily take hundreds of storage megabytes _per project_ not to mention wasting precious minutes of your valuable time. All this wasted time and space made me so fed up with this that I had a brief lapse of sanity and came up with my solution.

## [`srvs`](https://github.com/okwolf/srvs) answers the call

I already learned the Node.js APIs for HTTP when implementing [my project for declarative JavaScript server apps](/serving-up-simple), so I figured I might as well use those skills to build the minimal modern frontend tool I desired. I created [`srvs`](https://github.com/okwolf/srvs), a zero dependency dev server with support for static content in addition to JavaScript modules hosted from local files and [`unpkg.com`](https://unpkg.com). Installing _this_ package will cost you roughly 20 kilobytes of space and time on the order of a few seconds. It gives you hot reloading, opening your site for you in your browser, and more. Included are sensible defaults that will work for many projects out-of-the-box and an intentionally small set of [configuration options](https://github.com/okwolf/srvs#command-line).

> Caveat: `srvs` is only for use as a development tool; please do not use it in production. This project isn't designed to be scalable or secure but instead to give a great experience starting a new web app with minimal tooling.

Give this project a spin instantly by creating an `index.html` and then running this command in a terminal opened to the folder containing it:

```console
npx srvs
```

## but why?

It's essential to understand my priorities to appreciate the philosophy behind `srvs`. Performance is of the highest importance. One of my success criteria is for a medium-sized app to launch and load in under a second. This guiding principle is why I choose to have zero dependencies and reinvent all the wheels. Additionally, I strove to avoid compile-time delays. Instead of full AST code parsing, I opted for the minimal set of code transformations necessary for writing modern web apps. I wanted support for `unpkg.com` because it allows for importing modern ES modules without an install step and using existing `node_modules` for performance. In addition to defaults for commonly used patterns from other tools, I included falling back to an `index.html` page for unknown URLs to support SPAs with routers using the `history.pushState` API.

Just as critical for understanding my thought process are the areas I specifically chose _not_ to pursue and why. It's important to draw clear boundaries - and knowing what something is not more definitively explains what it _is_. I've already mentioned that I'm not targeting production use cases, which means certain security, compression, proxy, etc. features will never ship with `srvs`. Other use cases I've left out are any non-HTTP protocols. I won't provide bundling in this tool. Many of the alternatives are combination dev servers and bundlers, but peer pressure like that doesn't work on me. Since I'm not parsing an AST of the code, transpilation is out of the question, including JSX. Because I'm not supporting older browsers, polyfills are not provided. Finally, this library is not designed to be user-extensible, so I don't plan to support any middleware-type features.

![](/images/machine-internals.jpg)

## how does it work?

At the heart of `srvs` is a reasonably straightforward file server that can make some educated guesses about which resource the client wants. If the path doesn't map to a local file in your static or script roots, then [Node.js module resolution](https://nodejs.org/api/modules.html#modules_require_resolve_request_options) is performed against these paths to try and find a partial match. When both of those resolution strategies fail, there is a fallback to `index.html` instead. After resolving the request to a file, sending the response is relatively straightforward: send the MIME type of the file as the `Content-Type` and then pipe its contents over the wire. Any unhandled errors in this flow result in a 500 response.

When streaming a JavaScript file, there's one extra step added in the response flow listed above. Using a [Node.js transform stream](https://nodejs.org/api/stream.html#stream_class_stream_transform), `import`s and `export`s are rewritten to meet my goal for supporting both `node_modules` and `unpkg.com`. Regular expressions find and replace the `import`s. Each matching `import` first bails out if the path is relative, then checks if the requested module is installed in the local `node_modules` folder, followed by falling back to `unpkg.com` with the version from `package.json` if specified.

The last feature I'm providing a technical explanation for is how hot module reload (HMR) works. First, a [Node.js API](https://nodejs.org/api/fs.html#fs_fs_watch_filename_options_listener) watches the directories being served by `srvs`. A special [server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) endpoint exists which broadcasts to all connected clients when a watched file changes. Finally, a script is injected into each page, adding an [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) that connects to this endpoint. The script also initializes the `window.module.hot` API to allow for partial [reuse of Webpack HMR](https://webpack.js.org/api/hot-module-replacement) logic. File change events fire any HMR callbacks registered for that file or a full page refresh when there are none.

## what's coming next

Remember that this open-source project is one of many things I work on in my spare time, and it's still a work in progress. The debugging output provided is not always particularly informative nor pretty. Likewise, error handling can be more robust and additional helpful information provided when there are errors. The next feature on my plate is to replace or inject environment variables such as `NODE_ENV`, frequently used to control which code runs in different environments. I'm open to new ideas and contributions that fit within the [philosophy](#but-why) of the project. Hopefully, this has inspired you to get started faster with your next great idea, and remember:

### the next time creativity strikes, consider [`srvs`](https://github.com/okwolf/srvs) your humble host
