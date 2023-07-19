# Optimizing for Speed

Let's check the performance of our site! In Chrome, there's a tool called "Lighthouse". We're going to run just this for performance and select "Analyze page load". This is the best way to see if you have any problems on your site. Our score will likely be pretty high, simply because our site is so small and fast, but we'll use this to zero in on a couple of potential problems. And... yep! We got a 98 with *no build system*. That's *amazing*. But it looks like there's still room for improvement.

If we scroll down here, we can see where our problem areas are. The first thing we have is "Eliminate render-blocking resources", which is pointing to our font file. Keep in mind that a lot of what we're going to talk about right now has nothing to do with AssetMapper; It's just frontend performance in general. *This* file, if you open `/templates/base.html.twig`, we have a `<link>` tag that's pointing towards this font file. When your site sees `<link>` tags, it has to download them before it can render the page. So it's basically *freezing* the rendering of the page until it downloads this file. But this is interesting. If you open that file... let's get a non-minified version of it... this has a bunch of potential font faces. The font files themselves won't actually be downloaded *until* and *unless* we *use* this font. And this `font-display: swap` tells the browser:

`Hey, it's okay if you're rendering some text that's
supposed to be using this font template. You can use
the default system font first, show the text, finish
downloading this font file, and then use the font.`

Essentially, this CSS file is written in a way where all of these font files are going to be downloaded *lazily*. The *problem*, which isn't really a *big* problem, is that, at this point, our browser just sees a CSS file and it thinks:

`I need to download that CSS file right now and I
can't render the page until that CSS file is here.`

Once it does that, it finds out that there are a bunch of font files that it can lazily download. So if we *do* want to eliminate this render-blocking resource, we can start by copying this file. Or in our case, we just need to copy the font face we need. A lot of these are for other languages that we're not using, so we'll just copy the two Latin fonts here (we likely don't even need this Latin extension one, but we'll grab it anyway). Then we're going to delete this CSS file *entirely*, go to `/assets/styles/app.css`, and paste the two files here. You've probably noticed that these aren't real URLs, so we're going to go over and grab this... take off the `index.css`... and that should be fine. Then we can copy that and do the same thing down here. *Perfect*. This *is* adding some complexity to our code, so that's why this is a lower priority, optional fix.

Right now, our browser is going to attempt to download our `app.css` file, because it knows it's supposed to, and immediately, it it will see that it *doesn't* need to. It can just download these files when it's actually necessary. This eliminates that render-blocking resource, and we'll see that in action in a second.

The other failure we have is very similar. It's for FontAwesome - specifically, this JavaScript file. That's *also* in `base.html.twig`. Since this `<script>` tag doesn't have `defer` or `async` on it, this will *also* block the rendering of the page. If we want, we can add `defer` to this to basically say:

`Download this, but don't block the page while you're
downloading this.`

This is going to load our FontAwesome fonts, so the worst case scenario here is that our page loads and our fonts load from FontAwesome just a moment later because this wasn't blocking the page.

Okay, now that we've changed a couple of things, let's *test* it. We won't bother redeploying. We'll just go back to our local site over here and run Lighthouse again. Perfect! Then we'll "Analyze page load"... make this a little bigger, and... awesome! Now we're getting 100 locally! If you look down here... we can see some opportunities for improvement. We see "Serve images in next gen formats", which we should probably work on, and "Avoid serving legacy JavaScript to modern browsers". I'm pretty sure this is just talking about the `importmap` *shim* that we're including, so that's not a big deal. But below *that*, we see "Avoid chaining critical requests". This is probably the most important item on this list.

Here's what's happening. As you can see, it downloads the page first. Once it downloads the page, it realizes it needs to download this CSS file. Once it downloads the CSS file, it realizes that it needs to download this font file. Basically, instead of knowing it needs these files up front, it's finding out about them little by little. Ultimately, that means this font file is going to take *longer* to load, even though it's not blocking the page, because our browser has to run through a couple of steps before it even discovers that it needs to load that. If you care enough about this, you can go find that font file - that's this "normal" one here - take this URL and we can *preload* this. To do that, we'll go into our `base.html.twig` file and, up here, add a `<link>` tag that tells our browser:

`Hey, you don't know why just yet, but you need to
download this.`

In here, say `rel="preload" href=""` and paste that long URL. Then, for fonts, we'll need to add `as="font"`, `type="font/woff2"`, and `crossorigin=""` at the end. That's *long*, but that's what's going to hint that it needs to preload that file. If we go over and run Lighthouse again... we *still* score 100, and under "Avoid chaining critical requests", you can see that file is now *gone*. That's fixed!

But... what about `app.css`? You can't really load that any earlier, right? That's inside of our `base.html.twig` already. It downloads the page and as soon as it has the HTML, it *sees* this CSS tag. That's *fine*, but could we move this even *earlier*? Is there a way to *hint* to the browser that it needs to download this `app.tailwind.css` *before* it downloads the HTML? The answer, surprisingly, is *yes*.

To do that, we need a Symfony component called "WebLink". At your terminal, say:

```terminal
composer require symfony/web-link
```

Once that's done, back in `base.html.twig`, we're going to add *another* preload down here that looks pretty similar: `<link rel="preload" href="">`. This time, we're going to use a function called `preload`. I'll talk about that in a second. So say `{{ preload() }}`, and then we're going to use our normal `asset()` function to point to `styles/app.tailwind.css`. In this case, this `preload` function needs another option here called `as: 'style'`. That's it! We don't need any more attributes. If we go refresh the page and "View page source", this, by itself, is going to output a `preload` tag, which is kind of pointless. When the browser is reading HTML and it sees this, this basically says:

`Hey, you should start downloading this CSS file.`

*But* literally one line later, it would have found out anyway. So while this *is* pretty pointless, the fact that we call this `preload()` function *first* tells Symfony:

`Hey, when you return the response for this page, add a preload header.`

If we go over to the Network tools, select "All", find our main page, go to "Headers", and look under "Response Headers"... look! There's a new "Link" header here called "preload" that points to our CSS file. So as the response is being sent back to the user's browser, at the *very* top of the response in the header is that *hint* to start downloading that CSS file. So it actually *can* start downloading the CSS file before it even downloads the HTML file. If we go back over to Lighthouse and analyze the page load again... down here... beautiful! Look at that! That entire section is *gone*.

There are a couple of other things here like "Keep request counts low and transfer sizes small", but these aren't really warnings - just something we could work on improving if we wanted to. But on the topic of preloading, even though we don't have any more warnings here, there *is* another spot where preloading can improve performance, and it has to do with JavaScript.

Over in our `importmap.php` file, there's a key here called `preload` that we haven't talked about. You can see that it's set to `true` for `app`. We're going to set that to `false`. If we head over here and run another Lighthouse report... we still get a score of 100, but if we go down here, it says "Avoid chaining critical requests". And check this out! We have the HTML page, down to `app.js`, `bootstrap.js`, the Stimulus loader, Stimulus itself, controllers... *wow*. A *bunch* of stuff happened.

This is the same problem we saw with the CSS and font files earlier. First, our browser downloads the HTML page. *Then* it sees that it needs to download `app.js`. Once it downloads *that* file, it sees that it needs to download our `bootstrap.js` file. Then, when it sees that, it knows that it needs to download the `stimulus-bundle/loader`, and so on - one by one by one, instead of downloading *all* of those things as soon as the page loads. It has to *discover* them little by little, which *isn't* ideal. The `preload` is what's fixing that. I'll change this back to `true`, refresh the page, and "View page source".

One of the things here that we haven't talked about yet comes from the `importmap()`. We know that dumps the `importmap`, and we know it dumps our `<script type="module">` down here. But it *also* dumps these `modulepreload` things. This is pretty cool. Because we said `'preload' => true` for `app`, it adds a `<link rel="modulepreload">` here for `app.js`. That hints to the browser that it should start downloading `app.js` *immediately*. That's not super important because it would have figured that out down here anyway, but *then* AssetMapper sees that our `/assets/app.js` file imports *Bootstrap*. So since `app.js` is preloaded, it *also* preloads `bootstrap.js`. And since this imports `./lib/vinyl.js`, it *also* preloads `./lib/vinyl.js`. So it's going to start downloading all three of those files immediately.

At this point, if we ran Lighthouse again, it wouldn't complain about any of these chain requests. But we *still* have room for improvement. You can see that if you look over the JavaScript and check out this waterfall column. We see that a couple of files *start* downloading, and then a few more... and a few more. So we still have this chaining problem.

We know, for example, that the `bootstrap.js` is being preloaded, but this `@symfony/stimulus-bundle` file *isn't*. It downloads `bootstrap.js`, and *then* it discovers it needs this file, and *then* it discovers the things inside of it. This happens because we're preloading `app`, and so anything that `app` imports with a *relative* import will automatically be preloaded as well. But anything we're importing with a *bare* import, like `lodash/camelCase` or `@symfony/stimulus-bundle`, *won't* be preloaded automatically. That's because they have their own entries inside of here, so you control the preloading for those by themselves.

This is a bit of an optimization on performance, but if you wanted to, we could add `preload` to the items we *know* will be critical to our page rendering - `stimulus` is going to be critical, `stimulus-bundle` because that's what loads our controllers, and also `turbo`. When we refresh... nothing changes. We'll just see more of those `modulepreloads` down here. If we run Lighthouse one more time, we're *still* scoring 100, and you can see that there are no major problems down here. *Fantastic*.

All right, friends! Thanks so much for joining us! AssetMapper is *cool*. There are some things it *doesn't* do, like tree shaking or handling TypeScript, but for a large number of projects, it's going to work *great*. And the cool thing is, you're still running normal JavaScript, so if you ever did need to move to a build system later, you could do that *super* easily. Let us know what you think, and hopefully we can keep the improvements going for Symfony 6.4. Thanks again and I'll see you all *next time*!

