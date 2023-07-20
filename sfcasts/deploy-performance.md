# Long-Term Caching, Compression & File Combining

It's time to celebrate! We're on production, and it was really just as simple as
making sure Tailwind was being built and running the `asset-map:compile` command.

Now that we're here, there *are* a few things we need to check on, to make
sure our site is *fast*.

## 1) HTTP/2: You Definitely Need It

The *first* is to check that your web server is using HTTP/2. We talked about
that earlier, so hopefully you already got that rocking.

## 2) Combining Files

The *second* thing is... well.. not really a thing at all. I just want to point
out that nothing in AssetMapper ever *combined* our files to reduce the number of HTTP
requests. We're going to talk more about this in a few minutes, but thanks to HTTP/2,
you almost definitely do *not* need to combine your files together. So if you were
thinking that this was missing, it's not! It's by design. That's good! One less thing.

## 3) File Compression / Minification

But what about *minifying* our files? It's true: right now, our files are being
served *without* minification, and that *is* a problem. We *do* want our CSS and
JavaScript files to be minified... or at least *compressed*. And this is thing
number three to think about.

But... this is something that can be done by our web server. Yup, if you ask kindly,
you should be able to convince your web server to compress your files so they're
smaller when being sent across the network. This is something that all web servers
support, and it's done *automatically* by Platform.sh.

Here's how we can see it. Go to "Network" tools and select JavaScript. Select one
of the files then go to "Headers". I'll make this a bit bigger. Okay,
see this "Content-Encoding" response header? *That's* compression. This "br" stands
for something called Brotli, which is more delicious than it sounds. Brotli is
an advanced compression engine. The other common value is `gzip`. So all of our static
files *are* already being compressed! We get that for *free* with Platform.sh,
so we can check that item off our list. Check your deploy system or web server
docs for details on how to do this in *your* situation.

But wait, are minifying and compressing the same thing or different? Actually,
they're a bit different. Both minifying and compressing *greatly* reduce the size
of a file. We're using *compression*. Minification *can* result in slightly smaller
files - in part because it will remove code comments - but it's not significant.
Web servers themselves don't support minification, but if you use Cloudflare,
there *is* a way to enable auto-minification. It probably won't make a *huge*
difference, but you can try it.

## 4) Long-Term File Caching

The fourth and final thing that we need to do is make sure that all of our static
files are set up for *long-term* expiration. Because we have these nice versioned
file names, when a user visits our site, we want them to download this file *one*
time and never, *ever* download it again. We want them to cache it forever. Because,
if we change anything inside of this file, the whole filename will change! And the
user's browser will naturally download the new version the next time they visit
the site.

Over here, under "Headers", we *do* see an "Expires" response header. Out of the box,
Platform.sh adds an "Expires" header for static assets, set to one hour.
We can do *much* better than that.

Over in `.platform.app.yaml`, under `locations`... there we go... we see...
`expires: 1h`. That's *fine* as a default for *other* static assets that we may have
on our site. So I'm going to leave that alone. But add another rule to
be more specific. For the regex, if the URL matches `/assets/` anything, then
set the `expires` header to `365d`. Yes, 1 year - that's *forever* in Internet
time!

Cool! Lets commit that... and deploy, deploy!

```terminal-silent skip-ci
symfony deploy
```

When that finishes, *refresh*. We won't see anything obvious... but let's check
out one of our JavaScript files, like `app.js`. We're looking for "Expires". If
you don't  see it - or it's still short - do a force refresh. This is weird case
where the file didn't change, but we might still have the cached version from a
minute ago with the old header. And if you *do* see this, *congratulations*! That's
the goal.

*So*, our assets are being compressed, and they have long-term "Expires" headers.
We've checked *all* of our boxes for a performant site!

Next: We're going to *prove* the site is fast by using Lighthouse to
*profile* the site's performance. We'll learn about how files are downloaded, how
pages are built, and we'll make our frontend even *more* efficient.
