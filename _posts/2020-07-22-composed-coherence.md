---
layout: post
title: Composed coherence
description: Wiring widgets with wonder
image: /images/write-sheet-music.jpg
---

> Notes: this builds on my [previous video/article about functions](/whats-your-function). I highly recommend you understand this prior material before proceeding. Consider this a written companion to another video about functions from my [barely functional dev](https://www.youtube.com/c/barelyfunctionaldev) channel.

{% include youtube.html id="KU7K35UD028" %}

Let's talk about how function composition is like using cable adapters. It's important to note that there are different types of adapters you might encounter. For example, imagine a cable used for extending USB signals where we have both kinds of Standard-A connectors. You plug in your USB device to one end of it, and then the signal will travel through the length of the cable, and you'll plug in the other end of the cable into where you want your USB device plugged into, except now you have more range than plugging in your device directly.

![](/images/adapter-usb-extension.jpg)

This type of cable preserves the original signal, but some cables will convert from one signal to another. When these are short and allow for connecting to a computer, they are often referred to as dongles. Here's an example of one of those where on one end, we have a USB Type-A device and then, on the other end, a USB Type-C device. An adapter you'd use on a computer that only has USB Type-C ports, say on a Mac.

![](/images/adapter-usb-c-dongle.jpg)

There's also cables out there that add to signals. Imagine that we have this wacky cable that takes in a video signal, then it generates some dank memes, and it inserts the memes into the video on the other side. That's a handy cable. Other types of wires take away from the signal that comes through them. An example is a cable that's for charging only and removes the data in the signal. Then you could have cables that convert wildly different types of signals from one to another. Think of an adapter that takes in an audio signal, and then it spits out a video that does trippy visualizations of that audio.

Let's introduce one more type of cable here. This one is probably familiar to iPhone users, where you have a USB input signal and, on the other side, a lightning connector. These cords are used for charging your phone or connecting it to various devices.

![](/images/adapter-lightning.jpg)

It's important to take a break here and realize how these are related to functions. If you think about it, each of these adapters takes input and spits out an output. Here's what the USB-C dongle and lightning cable examples might look like as functions:

```js
function usbCDongle(usbCSignal) {
  return convertSignalToUsbA(usbCSignal);
}

function lightningAdapter(usbASignal) {
  return convertSignalToLightning(usbASignal);
}
```

Interestingly, we can take compatible adapters and chain them together to take the signal from one and attach the two compatible connectors.

![](/images/adapter-composed-usb-c-lightning.jpg)

I can plug in one side of the dongle to my computer with the USB Type-C connector and then that signal will flow through the dongle. The output of the dongle goes right into the input of the lightning adapter plugged into my phone. Imagine if we were to repackage this and cover up the fact that there are two cables connected, only exposing the two ends on the extremes. What we have is a new cable that directly takes the USB Type-C and then, on the other side, has the lightning cable.

Here is where we connect back to function composition. We've essentially created a new function that takes one type of input, some stuff happens (that we don't expose as part of the function) and we return something that is the result of the two functions we put together. That composed function looks like this:

```js
function lightningUsbCDongle(usbCSignal) {
  return lightningAdapter(usbCDongle(usbCSignal));
}
```

It's important to note that we're not limited to doing this with two functions. For example, starting with the cable where we take USB-C and take the signal from there into the USB extension cable. Then travel along the whole length of the extension, and on the other side is another compatible cable plugged into the lightning cable. Now we've made yet another type of cable.

![](/images/adapter-composed-usb-c-lightning-extension.jpg)

This new cable has the same input and the corresponding output, but it makes another internal connection that provides more reach for the wire, and none of that gets exposed as part of the implementation. All that matters to the users is providing the correct input and what comes out on the other side. In code, we would write this function similar to the last one, but with one more level of nested function calls.

```js
function lightningUsbCDongle(usbCSignal) {
  return lightningAdapter(
    usbExtensionCable(usbCDongle(usbCSignal))
  );
}
```

When you compose functions - data flows from the innermost function call outward - similar to how electrical signals flow between adapter cables, which are daisy-chained together. This pattern enables breaking down complex functions into many smaller functions that are easier to write. What's meant by _decomposing_ problems is once you have all the pieces, it's a matter of _composing_ them together into your solution. A positive byproduct of these is you will often find the decomposed functions are reusable for future problems, saving time, effort, and code duplication automatically.

### Get composing, and your functions will be music to your ears.
