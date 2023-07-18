# Long-Term Caching, Compression & File Combining

It's time to celebrate! We're on production, and it was really just as simple as
making sure Tailwind is being built and running the `asset-map:compile` command.

Now that we're here, there are a few *other* things we need to check on... to make
sure our site is *fast*

## 1) HTTP/2: You Definitely Need It

you're on production. The *first* is checking that your web server is using HTTP/2.
We talked about that earlier, so hopefully you already ensured got that rocking.

## 2) Combining Files

The *second* thing is... well.. not really a thing at all. I just want to point
out that nothing in AssetMapper ever combined our files to reduce the number of HTTP
requests. We're going to talk more about this in a few minutes, but thanks to HTTP/2,
you almost definitely do *not* need to combine your files together. So if you were
thinking that this was missing, it's not! It's by design. One less thing to worry
about.

## 3) File Compression / Minification

But what about *minifying* our files? It's true: right now, our files are being
serviced *without* minification, and that *is* a problem. We *do* want our CSS and
JavaScript files to be minified... or at least *compressed*. And this is thing
number three to hink about.

But this is something that can be done by our web server. Basically, we need to
ask our web server to compress the assets so that they're smaller when they're being
sent across the network. This is something that all web servers support, and it's done
*automatically* by Platform.sh.

Here's how we can see it. Go to "Network" tools and select JavaScript. Select one
of the JavaScript files then go to "Headers". I'll make this a bit bigger. Okay,
see this "Content-Encoding" response header? *That's* compression. This "br" stands
for something called Brotli, which is more delicious than it sounds. Brotli is
an advanced compression engine. Another common value is `gzip`. So all of our static
files *are* already being compressed, and we get that for *free* with Platform.sh.
We can check that item off our list.

But is minifying different than compression? It is actually! Both minifying and
compressing *greatly* reduce the size of a file. We're using *compression*. Minification
*can* result in slightly smaller files - in part because it will remove code comments -
but it's not significant. Web servers themselves don't support minification, but
if you use Cloudflare, there *is* a way to enable auto-minification if you want.
It probably won't make a *huge* difference, but you can try it.

## 4) Long-Term File Caching

The fourth and final thing that we need to do is make sure that all of our static
files have *long-term* expiration. Because we have these nice versioned file names,
when a user visits our site, we want them to download this file *one* time and never,
*ever* download it again. We want them to cache it forever. Because, if we change
any code inside of this file, the *whole* filename will change... and the user's
browser will naturally download the new version the next time they visit the site.

Over here, under "Headers", we see an "Expires" response header. Out of the box,
Platform.sh *does* add an "Expires" header to static assets, set to an hour.
We can do *much* better than that.

Over in `.platform.app.yaml`, under `locations`... there we go... we see...
`expires: 1h`. That's *fine* as a default for *other* static assets that we may
on our site. So I'm going to leave that alone. But add another rule under here to
be more specific. For the regex, if the URL matches `/assets/` anything, then
set the `expires` header to `365d`. Yes, 1 year - that's *forever* in Internet
terms.

Cool! Lets commit that... and deploy, deploy!

```terminal-silent skip-ci
symfony deploy
```


Once that finishes, *refresh*. We won't see any obvious... but let's check out one
of our JavaScript files, like `app.js`. We're looking for "Expires". If you don't
see it - or it's still short - do a force refresh. This is weird case where the
file didn't change, but we might still have the cached version from a minute
ago. And if you *do* see this, *congratulations*! That's the goal.

*So*, our assets are being compressed and they have long-term "Expires" headers.
We've checked *all* of our boxes for a performant site!

Next: We're going to *prove* the site is fast by using Lighthouse to actually
*profile* the site's performance. We'll learn about how files are downloaded, how
pages are built, and we'll make our frontend even *more* efficient.
