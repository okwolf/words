---
layout: post
title: Neatly nested
description: Funception adeption
image: /images/nesting-dolls.jpg
---

> Consider this a written companion to the following video about higher-order functions from my [barely functional dev](https://www.youtube.com/c/barelyfunctionaldev) channel.

{% include youtube.html id="zQUVnxLBJKI" %}

> I heard you like functions,
>
> So we put a function in your function,
>
> So that you can function,
>
> While you function.

This time around - we're talking about a topic I call "funception," that is, functions nested within other functions (also known as higher-order functions). It's important to distinguish this topic from [function composition](/composed-coherence), which I previously covered. With function composition, what happens is one function's output feeds into another function as input, and then the output of that becomes the output of the resulting composed function.

![](/images/composed-function.png)

When you combine those functions, and you put a box around that, what you've done is you've composed the two functions together. It's like you've created a chain of functions that behaves as one function.

![](/images/function-chain.png)

Funception is a little bit different because we have a function as the input or the output (or both) to another function. This approach may seem odd, so I'm going to take a step back and go back to my analogy of a [function as a library](/whats-your-function). Imagine that we have a library of all the possible books in the world. In my previous library analogy, the input tells us which shelf to go to, what book to pull off the shelf, and what page to open. Instead, maybe you want information about all the books in the library. The library doesn't know what information you want; it doesn't know how to summarize each of the books that you want to get information on.

What you could do is you could teach the function how to get that summary by providing... another function. You could give a lookup saying, "given this book as my input - here's what I want the summary to be for the output," and then you could pass that function as the input into this other function. Then it would operate on all the books that existed, and out the other end, you would get a summary of every single book in existence.

![](/images/pass-function-to-function.png)

Here's some conceptual code showing the two functions involved and how they are related by passing one as input into the other:

```js
function summarizeBook(book) {
  const bookSummary = `${book.substring(100)}...`;
  return bookSummary;
}

function getInformationFromAllBooks(
  getInformationForSingleBook
) {
  // use getInformationForSingleBook for each book
  const informationFromAllBooks = [...];
  return informationFromAllBooks;
}

// returns summaries of all books
getInformationFromAllBooks(summarizeBook);
```

Now let's say we don't want to have to write that summarize function ourselves. Well, what we could do instead is we could have a library where instead of it being a library of books, it's a library of libraries. Sounds crazy at first, but ultimately what we're doing here is generalizing the whole operation one level up.

Say we have a category of books, for example, a topic like cats, and we have a library that knows how to summarize cats (along with every other topic in the world). If we tell that library "cats," we'll get as the output a function that can take any book in existence, and then it will output a summary of that cat book. This summary will be contextual; it will understand all the terms of cats, the nuances of cats, the purrs of cats, the meows of cats, and everything to do with cats.

![](/images/return-function-from-function.png)

We've just built a layer of reusability for our functions. Now we can pass in any topic, and the output will be this function. Then we can even go one step further; we could pass that function into another library, and get a summary of all the possible cat books that exist. Check out this pseudocode describing how such a function might look and how to use it:

```js
function makeSummarizeForTopic(topic) {
  return function summarizeBook(book) {
    // use topic-specific knowledge to summary book
    return "...";
  };
}

// make a summarize for cat books
const summarizeCatBook = makeSummarizeForTopic("cats");

// returns summaries of all cat books
getInformationFromAllCatBooks(summarizeCatBook);
```

Nesting functions give us more building blocks to start simple and build our way towards advanced programs. These patterns also enable high degrees of reuse and flexibility for future change. Understanding this unlocks a whole world of functional programming potential.

### Nest functions neatly, and they won't warp your mind.
