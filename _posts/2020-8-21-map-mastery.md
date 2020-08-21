---
layout: post
title: Map mastery
description: Filter out the noise
image: /images/map-compass.jpg
---

> Notes: this builds on my [previous video/article about higher-order functions](/neatly-nested). I highly recommend you understand this prior material before proceeding. Consider this a written companion to my video about the map and filter functions from my [barely functional dev](https://www.youtube.com/c/barelyfunctionaldev) channel.

{% include youtube.html id="B1t0Ls_X5Os" %}

Previously [I wrote about "funception"](/neatly-nested) or higher-order functions in highfalutin language. One of the examples that I gave was that of a function that took another function as its input. That function was able to map the contents of a book to a summary of that book. The function would operate using that function to produce summaries of all the books in the library.

![](/images/map-books.png)

This concept is commonly called a `map` function in functional programming. I'm going to be explaining some more details of this, and also a bonus function. For another example, let's start with a factory:

![](/images/factory-empty.png)

And in our factory, we have this conveyor belt where items are processed along:

![](/images/factory-conveyor.png)

Let's say this happens to be a car factory that we're working with here, and so we start with some partially assembled incomplete vehicles:

![](/images/factory-map-vehicles-input.png)

And those go into our function here which we called `map`:

![](/images/factory-map-function.png)

`map` is going to do some maintenance on them, it's going to tune them up, it's going to spice them up, it's going to add some cool stuff to them, and then out the other end, we're going to get supercharged complete fully ready-to-race cars.

![](/images/factory-map-vehicles-output.png)

If we look at what that concept might look like in code:

```js
const vehicles = [ ... ];

function makeVehicleVroom(vehicle) {
  // add the turbo
  // install the flux capacitor
  // …
  return vehicleWithVroom;

}
// returns vehicles with vroom added
vehicles.map(makeVehicleVroom);
```

We start with a collection of vehicles. Then we have a function that can operate on each one of the vehicles, and that function will do all the tune-up, all the adding of special features, all the make it run fast things, and then return the new vehicle. So to use this, for example, in JavaScript, the collection of vehicles has a `map` function and that `map` function receives the function that I just described. This transformation runs on each one of the vehicles. And that's how `map` operates.

Let's move on to another commonly used functional programming concept called `filter`. To provide an example of this, let's start with the factory again. Instead of starting with all the same types of vehicles, we begin with a collection of differently-abled vehicles, some of which meet the cut, and some don't meet the cut.

![](/images/factory-filter-vehicles-input.png)

You can imagine `filter` is kind of like our QA; it inspects each item that comes through it and decides whether it wants to include that item or it wants to drop it on the floor and not include it.

![](/images/factory-filter-function.png)

In this case, what `filter` will do is look at each of the vehicles that come through here and decide: "Is this vehicle up to snuff? Am I good enough to be included, to be shipped, to be sold for millions of dollars?" In the case that it is, it allows the vehicle through, and then the smaller set of vehicles that do pass the `filter` is what the output becomes.

![](/images/factory-filter-vehicles-output.png)

Looking at this in code:

```js
const vehicles = [ ... ];

function isRacer(vehicle) {
  return vehicle.looksCool
            && vehicle.topSpeed > 9000;
}

// returns racers only
vehicles.filter(isRacer);
```

Here we start again with a collection of vehicles. Now we have a function that looks at each of the vehicles, and it returns a boolean result. It decides: "Does this vehicle look cool enough for me to use it? Is this vehicle fast enough for me to be including it in our sales?" Once we've built that boolean function, we can use the `filter` function on our vehicles collection and then return a new array where each time the `filter` is called it decides: "Hey, do you want to be in my new collection? Do you make the cut? Are you meeting my quality standards for this?" The result is every time `isRacer` returns `true`, it is included in the new set of vehicles.

Understanding how these two functions work are essential concepts when building apps with functional programming that process data, which are all of them. Having these tools in your toolbox allows you to keep your data transformations pure and immutable, giving your functions the maximum possible benefits.
Stay tuned for further updates on composing these basic building blocks and some of the more advanced ones.

## map a course to understand functional programming better
