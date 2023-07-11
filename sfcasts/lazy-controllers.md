# Lazy Stimulus Controllers

Okay, let me close a couple files.... and let's open up `/assets/controllers/goodbye-controller.js`. Pretend, for a moment, that this controller is really big and complex. Or, even more likely, let's say it imports some big third party package like `d3` for charts. But we're only using this on *some* pages.

Here's the deal. In order to register your controllers with Stimulus, all of these files are downloaded at runtime. So the page loads, Stimulus starts up, all of these files are downloaded, and anything they import is *also* downloaded. It's kind of wasteful if you're only going to be using this on a few pages. To fix that, above this, you can add a very special syntax - `/* stimulusFetch: 'lazy' */`. This is a very special thing we do in StimulusBundle, which instructs Stimulus to *not* download this JavaScript file or anything it imports until an element that matches this is on the page.

A moment ago, if I searched for "goodbye", this was being loaded on the page load. But *now*, if I refresh and search for "goodbye, it's *not* there. But check this out! Inspect this element on our `data-controller="hello"`. We're going to change that to `goodbye` and... boom! It works! You can see that it activated (that's what our `Goodbye controller!` does), and if we look at the Network tab, *now* it downloaded. I *love* this feature.

This can also be done for third party packages. If you look in `/assets/controllers.json`... Turbo isn't a very good example of this, but if we had said `"fetch": "lazy"` on any of these, they would have the *same* behavior as our own controllers. And that's it! Easiest chapter *ever*. You can use that to keep your initial page lightweight in the event you have some Stimulus controllers that are only used in certain places.

Next: Let's go through some tips for debugging when things *don't* work.
