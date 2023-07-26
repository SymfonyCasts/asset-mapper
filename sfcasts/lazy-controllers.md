# Lazy Stimulus Controllers

It's getting messy in here: let me close a few files... then crack open
`assets/controllers/goodbye-controller.js`. Pretend, for a moment, that this
controller is huge. Or, more likely, it imports a big third-party package
like `d3` for charts. *But*, we're only using this controller on *some* pages.

Here's the deal. In order to register your controllers with Stimulus, all of these
files are downloaded immediately. So the page loads, Stimulus starts up, all of these
files are downloaded, and any files they import are *also* downloaded. That's often
ok, but if you're importing something *big*, that can be wasteful.

To fix that, above the class, you can add a very special syntax -
`/* stimulusFetch: 'lazy' */`.

[[[ code('3b03f096b8') ]]]

This works thanks to StimulusBundle. When it spots this, it tells Stimulus to
hold its horses and *not* download this JavaScript file or anything it imports
*until* an element that matches this is on the page.

Watch. Before making that change, if we searched for "goodbye", that controller *was*
being loaded, even though it's not used on this page. But *now*, refresh and search
for "goodbye". It's *not* there! Inspect the
`data-controller="hello"` element. Change that to `goodbye` and... boom! It works!
You can see that it activated (that's what our `Goodbye controller!` does), and if
we look at the Network tab, *now* it downloaded. I *love* this feature.

This can also be done for third-party packages. If you look in
`assets/controllers.json`... Turbo isn't a very good example of this, but if we
said `"fetch": "lazy"` on any of these, they would have the *same* behavior that
we just saw.

[[[ code('b292bac1cf') ]]]

That's it! Easiest chapter *ever*! Use this to keep your initial page lightweight
if you have some heavy Stimulus controllers that are only used on certain page.

Next: sometimes, deep sigh, the tech gods frown upon us and things don't work.
Let's learn a few tips to help debug when that happens.
