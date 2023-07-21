# Preloading

We just discovered a problem: our browser needs to download the page... *and*
a CSS file before it even *realizes* that it needs to download this font. This
may not be a huge deal, but there's a cool solution: preloading.

## Manually Adding a link rel="preload"

Go find the font URL - it's this "normal" one - and copy it. Next, open
`base.html.twig` and, up here, add a `<link>` tag. Unlike a normal
`<link rel="stylesheet"` that points to a CSS file, the purpose of *this* link
tag will be to yell to the browser:

> Hey, you don't know it yet, but you should download this font file.

To do that, say `rel="preload"` then `href=""` and paste that long URL. And when
preloading fonts, we need to add `as="font"`, `type="font/woff2"`, and
`crossorigin=""` at the end.

Ok, let's see what Lighthouse thinks of this! After... we *still* score 100 -
yay! -  and under "Avoid chaining critical requests", that font file is *gone*.

## Preloading via a Header

But... what about `app.tailwind.css`? The browser downloads the HTML and then
immediately sees the `link` tag for this. Is there a way to *hint* to the browser
that it needs to download `app.tailwind.css` even *before* it downloads the
HTML? The answer, surprisingly, is *yes*!

But we need a Symfony component called "WebLink". At your terminal, run:

```terminal
composer require symfony/web-link
```

Once that's done, back in `base.html.twig`, add *another* preload
down here that looks similar: `<link rel="preload" href="">`. This time,
use a Twig function called `preload()` passing the normal `asset()` function to point
to `styles/app.tailwind.css`. In this case, this `preload` function needs another
option called `as: 'style'`.

That's it! Go refresh the page and "View page source". No surprise: this outputs
a `preload` tag. And... so far, the `preload()` Twig function looks like nothing
more than a semi-worthless shortcut!

Even more, for the `app.tailwind.css` file, this `link` tag is pointless! This
basically tells the browser:

> Hey, you should start downloading this CSS file.

*But*... one line later, it would have found it anyway! So why did we do this?
It turns out that the `preload()` function does two things: it outputs the
`link` tag `href`... but it *also* tells Symfony that it should add a `preload`
*header* to the response.

Go to the Network tools, select "All", find our main page, go to "Headers", and look
under "Response Headers" Woh! We have a new "Link" header called "preload"
that points to our CSS file! So as the browser starts downloading the response,
at the *very* top it sees a *hint* that it should start downloading that CSS file!

If we go back over to Lighthouse and analyze the page load again... down
here... beautiful! That entire section is *gone*.

## preload JavaScript in importmap.php

There are a few other things here like "Keep request counts low and transfer sizes
small", but these aren't really warnings: just something to keep in mind.

But on the topic of preloading, even though we don't have any more warnings,
there *is* another spot where preloading can improve performance, and it has to do
with JavaScript.

Over in `importmap.php`, there's a key called `preload` that we haven't
talked about. It's set to `true` for `app`. Set that to `false`.

Now, move over and run another Lighthouse report. We still get a score of 100, but
if we go down here, ah: "Avoid chaining critical requests" is back! And check it
out! We have the HTML page, down to `app.js`, then `bootstrap.js`, the Stimulus
loader, Stimulus itself, controllers... *wow*. A *bunch* of stuff is chaining.

This is the same problem we saw with the CSS and font files. First, our
browser downloads the HTML. *Then* it sees that it needs to download `app.js`.
Once it downloads *that* file, it sees that it needs to download `bootstrap.js`.
Then, it realizes it needs to download the `stimulus-bundle/loader`, and so on:
one-by-one-by-one. Instead of downloading *all* of those things in parallel,
it has to *discover* them little-by-little.

`preload` fixes that. Change this back to `true`, refresh the page, and view
page's source.

We know that the `importmap()` Twig function dumps the `importmap` and the
`<script type="module">`. But it *also* dumps these `modulepreload` things. These
are cool. Because we said `'preload' => true` for `app`, it adds a
`<link rel="modulepreload">` for `app.js`. That hints to the browser that it should
start downloading `app.js` *immediately*. Though, that's not really important because
it would have figured that out in about 1 microsecond anyway.

The *real* power is that AssetMapper *then* sees that `assets/app.js` imports
`bootstrap.js`. And because `app.js` is preloaded, it *also* preloads
`bootstrap.js`. And since this imports `./lib/vinyl.js`, it *also* preloads
`./lib/vinyl.js`. So it will download all three of these files immediately.

At this point, if we ran Lighthouse again, it wouldn't complain about any of these
chained requests. But we *still* have room for improvement. On the network tools,
for JavaScript, check out the waterfall column. We see that a few files *start*
downloading, and then a few more... and a few more. So we still have this chaining
problem, though it's apparently not a big enough deal for Lighthouse to report on.

We know that `bootstrap.js` is being preloaded, but
`@symfony/stimulus-bundle` *isn't*... even though it's imported from
`bootstrap.js`. Why doesn't the preload "follow" that import like the others?

The key thing to understand is that, because we're preloading `app.js`, anything
that `app.js` imports with a *relative* import will automatically be preloaded as
well. But anything we import with a *bare* import, like `lodash/camelCase` or
`@symfony/stimulus-bundle`, *won't* be preloaded automatically. Perhaps they should,
but they have their own entries inside of `importmap.php`, so you control the
preloading for those independently.

At this point, we're *really* optimizing performance - maybe over-optimizing. But
if you want to avoid this chaining problem, you could add `preload` to the items
we *know* will be critical to the page rendering. For example, `@hotwired/stimulus`
is critical, `stimulus-bundle` is critical because that's what loads our controllers,
and `@hotwired/turbo` is also critical.

When we refresh... nothing changes: we just have more `modulepreload` items in the
HTML. If we run Lighthouse one more time, we're *still* scoring 100, and you can
see that there are no major problems down here. *Fantastic*.

## Preload Everything

By the way, if you're now thinking:

> Why don't we just preload everything?

That's a good thought! But, don't! Your browser is smart, and without any preloads,
it has a highly-intelligent algorithm to determine the best order to download files
to load things as fast as possible. If you preloaded everything, the loading
order *probably* wouldn't be as good. Just use preloads for *critical* assets.

Ok! We made it! I think AssetMapper is a breath of fresh air - and I hope you
feel the same! There are some things it *doesn't* do, like tree-shaking or handling
TypeScript. But for a large number of projects, it's a great fit! And the
cool thing is, you're still writing normal JavaScript. So if you ever *did* need to
move to a build system later, you could do that.

Let us know what you think, and hopefully we can make more improvements for
Symfony 6.4. Alright friends, see you next time!
