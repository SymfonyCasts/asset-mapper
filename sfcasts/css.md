# CSS & Background Images

When we're talking about the frontend of a site, we're mostly talking about two
things, CSS and JavaScript. Let's start with the CSS side of things... which is
dead simple with AssetMapper. You create a CSS file inside the `assets/` directory
then include it with a good old-fashioned `link` tag that uses the file's logical
path. That's it. Zero magic.

This *is* a bit different than Encore. With a build system like Encore, you
may be familiar with doing things like this: `import './styles/app.css`. That kind
of thing will *not* work in a browser environment. Import statement are for importing
JavaScript files, period. Ok, you *can* technically lazily-load CSS like this,
but that's an edge-case we don't need to worry about right now.

The point is, you can't import CSS files from JavaScript files and that's ok: adding
a `link` tag works great.

## Referencing Images from inside CSS Files

Ok: so we know that we can refer to any file in the `assets/` directory using the
`asset()` function... which we've now done twice.

But what if we need to refer to a file - like this image - from *inside* a CSS file?

[[[ code('de3f8d3200') ]]]

Check it out. Up here, we have a little record icon in the upper-left corner. Change
that to be a `span` with `class="bg-logo"` so we can include our penguin image
instead. Copy that `bg-logo` class head to `app.css`, add `.bg-logo` and...
I'll add some basic styles.

[[[ code('17c2fac770') ]]]

The *big* question is: how can we set the background image... since the final
`penguin.png` will have a versioned filename? The answer is: exactly how you would
*normally* do it: `url()` and then the relative path to the file: `../images/penguin.png`.

[[[ code('62ff749566') ]]]

This is *exactly* how you do it in Encore and exactly how you would do it if these
files were being served directly to our browser. We simply need to write "correct"
code and it'll work.

Let's add 2 more styles for the background... then testing time! Refresh and...
yes, it *does* work! Inspect that image... then look at the final CSS file. Let's
open this in a new tab.

Perfect! For the most part, the final files *exactly* match the source files. No
magic. However, in this case, AssetMapper *did* make one small change. In the
original file, we referred to `../images/penguin.png`. But over here we have
`../images/penguin-versionhash.png`. Yup, AssetMapper made that tiny change to
keep things working, despite the versioned filenames.

The point is: you get to code like normal... and everything just works.

Next: let's invite some third-party CSS like Bootstrap and fonts... to the party!
