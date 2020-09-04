---
layout: post
title: Respectable reduction
description: Boiling a function to its essence
image: /images/fire-pot.jpg
---

> Notes: this builds on my [previous video/article about map and filter](/map-mastery). I highly recommend you understand this prior material before proceeding. Consider this a written companion to my video about the reduce function from my [barely functional dev](https://www.youtube.com/c/barelyfunctionaldev) channel.

{% include youtube.html id="fEPfh3uQ6Qo" %}

Previously, [I introduced two](/map-mastery) out of the three essential functions commonly used in functional programming, but why did I leave one out? Well, I'm a [barely functional dev](https://www.youtube.com/c/barelyfunctionaldev), but I'm going to rectify that and complete the trifecta for you today.

To start, let's recap a little bit about the two functions we covered before. First of all, there's `map`. According to my prior analogy, `map` works similar to an assembly line in a factory where each item of the same kind comes in. We perform some transformation/upgrade/improvement/change to that item, and on the other side, the changed/improved/souped-up version gets spit out.

![](/images/factory-map-vehicles-output.png)

The second function is `filter`, and `filter` operates one at a time on the items like `map`, but the difference is that `filter` works like the QA in our factory. It gets to decide if what comes out the other end meets our factory's quality standards. In this case, it would do a check of all the vehicles coming in and see is this thing supercharged? Is it racy enough for what I want to ship to my consumers? The result is, if it is, it comes out the other end; if not, it gets blocked and doesn't come out.

![](/images/factory-filter-vehicles-output.png)

Now let's reset. We're going back to an empty factory here:

![](/images/factory-empty.png)

Instead of having just a one-way conveyor belt, I will introduce a circular conveyor belt here:

![](/images/factory-circular-conveyor.png)

Items enter in from the left side, they go around in a circle as many times as it takes, and then they get spit out on the right side. For this example, let's say this factory is building teddy bears.

![](/images/factory-reduce-bear-input-output.png)

We have all of our teddy bear parts on the left side come in, and we have our completed teddy bears on the right. This assembly process works here by each piece going around and around in this circular conveyor belt and entering the new `reduce` function that we're introducing. The `reduce` function is unique in that instead of transforming the elements one at a time, or including or excluding elements one at a time; it combines all of the elements to produce a single result on the other side.

![](/images/factory-reduce-bear-function.png)

You can imagine we go around and around with the different bear pieces we've got. The ears come in; at some point, the eyes come in, the face comes in, the paw prints come in. With each piece that comes in, we have tailoring that stitches it together and "reduces" it with the other items that have already come through. This process goes around as many times as it takes until we have a completed teddy bear that spits out on the other side.

Let's take a look at what pseudo code for this might look like:

```js
const bearParts = [ears, eyes, paws, head, ...];

function addBearPart(bear, part) {
  return {
    ...bear,
    [part.name]: part,
  };
}

// returns a bear with all the parts added
bearParts.reduce(addBearPart, emptyBear);
```

Once you have all the bear parts declared and inventoried (for the ears, eyes, paws, head, etc.), we move on to our `addBearPart` function. This function does the work inside the `reduce` box above where we take the partially assembled bear so far along with the bear part coming in next. Our function here takes those two pieces, combines them, and creates the next partially assembled bear. Once we have the bear parts and our function, we use the `reduce` function to take all the parts and `reduce` over them, passing them that function operating on the partially completed bear. Each of the bear parts is then stitched together one at a time onto it. The last thing that our `reduce` needs is we need to give it a starting point, so in this case, we start with an empty bear. This value becomes the initial state, or whatever you want to call this starting value passed for the first time; we want to combine the first part with nothing yet assembled. In this case, that would be nothingness (the empty bear).

This article gives a brief introduction to how `reduce` works as a concept. There's a lot more potential power available, and fundamentally there are many things you can do with `reduce`. You can even use `reduce` to represent all of the other functions we've already discussed. You can rewrite [`map` and `filter`](/map-mastery) using `reduce`. I might cover that in the future, but I want to provide a gentle introduction to how `reduce` works for now.

## that's one tasty reduction of knowledge
