# Optimizing & Profiling

Instead of me *telling* you the site is fast, let's prove it! In Chrome, there's
a tool called "Lighthouse", which you can also get for some other browsers.
Run this for just performance and select "Analyze page load".

This is the *best* way to see if you have any frontend performance problems. Our score
will likely be pretty high - simply because our site is small and quick - but
we can use the report to zero-in on a few possible problems. And... yep! We got
a 98 with *no build system*! That's *amazing*. But we can do even better.

## Eliminate Render-Blocking Resources

If we scroll down, we can see where our problem areas are. The first
is "Eliminate render-blocking resources", which points to our font file.
A lot of what we're going to talk about has nothing to do with AssetMapper: it's
just frontend performance in general. If you open `templates/base.html.twig`, we
have a `<link>` tag that points to this font file. 

[[[ code('0ed18657ba') ]]]

When your site sees a `<link rel="stylesheet"> tag, it downloads it before it 
renders the page. So it basically *freezes* the rendering of the page until 
the download finishes.

But this is interesting. Open that file... and let's get a non-minified version.
It has a bunch of potential font faces. Here's how this works: our browser
downloads *this* file immediately... but the font files themselves won't be downloaded
*until* and *unless* we *use* this font. Additionally, `font-display: swap` tells
the browser:

> Hey, it's ok to render some text that's supposed to use this font, even if the
> font isn't downloaded yet. You can use the default system font first, show the
> text, finish downloading this font file, and *then* use it.

Essentially, this CSS file is written in a way where all of these font files are
going to be downloaded *lazily*. The *problem*, which isn't really a *big* problem,
is that, at this point, our browser just sees a CSS file and thinks:

> I need to download that CSS file right now and I can't render the page until
> that finishes!

Once it *does* finally see the CSS contents, it discovers that there are a bunch
of font files that it can lazily download.

So, CSS files are render-blocking resources... which is *normally* great, because
we *don't* want the page to render unstyled for a half second before the CSS
downloads. But *this* particular file is funny because it is a "render-blocking"
resource... but doesn't contain anything critical.

If we care enough to eliminate this render-blocking resource, we can move it
into `app.css`. Start by copying this file... or really just the font faces
we need: a lot of these are for languages that we're not using. I'll copy the
two Latin fonts... though we likely don't even need this Latin extension one.
Then delete this CSS file entirely, go to `assets/styles/app.css`, and paste.
These aren't real URLs... so go copy the URL... paste, take off the `index.css`...
and that should be fine. Copy the URL again... and do the same thing down here.

[[[ code('55741db20b') ]]]

*Perfect*. This *is* adding some complexity to our code for only a small gain, so I'd
say this is a lower priority. We have basically the same amount of CSS as before,
but we've eliminated a small, unnecessary blocking resource.

The other failure we have is similar. It's for FontAwesome - specifically, this
JavaScript file. That's *also* in `base.html.twig`. Since this `<script>` tag doesn't
have `defer` or `async` on it, this will *also* block the rendering of the page.
If we want, we can add `defer` to this, which says:

> Start download this immediately, but don't block the page while it's finishing.

[[[ code('4bd1fba315') ]]]

Because this is for FontAwesome fonts, the worst-case scenario is that the page loads
and then our font icons show up just a moment later.

## Profiling Again!

Okay, now that we've changed a couple of things, let's *test* it. To save time
redeploying, I'll go back to my local site and run Lighthouse again.
"Analyze page load"... make this a bit bigger, and... awesome! We're getting
100 locally!

But if you look down here... we *do* still have some opportunities to improve.
We see "Serve images in next gen formats", which is a good thing to check on later,
but not related to Symfony or AssetMapper. This "Avoid serving legacy JavaScript
to modern browsers" - I believe that's referring to the `importmap` *shim*: the
code that makes the importmap work on all browsers. That's small & necessary,
so not a big deal.

## Avoiding Chaining Critical Requests

But below *that*, we see "Avoid chaining critical requests". This is probably *the*
most important item on this list.

Here's what's happening. As you can see, it downloads the HTML first. Once it
does, it realizes that it needs to download this CSS file. Once it downloads the
CSS file, it realizes that it needs to download this font file. See the problem?
Instead of knowing - from the start - that it needs these and downloading them
all at once in parallel, our browser is finding out about them little by little.
Ultimately, it means this font file will take *longer* to load because it can't
*start* downloading it until it downloads a few other files.

How can we fix this? Preloading. Let's talk about this important topic next.
