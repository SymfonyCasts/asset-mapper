# Mapping Assets

AssetMapper isn't that big of a deal. Sure it dresses cool and has good dance
moves, but it's really quite simple. It has two main features.

Feature number one: we configure "paths" - like the `assets/` directory - and it
makes the files inside available publicly.

Let's see this in action. If you downloaded the course code, you should have a
`tutorial/` directory with an important `penguin.png` file inside. Copy that.
Inside `assets/`, we can organize things *however* we want. So let's create an
`images/` directory and transport our penguin there.

Now, remember, without the magic of AssetMapper, the only files that our browser
should be able to access are those inside the `public/` directory. So it should
be *impossible* to add an `img` tag that loads our penguin. But... it *is* possible.

## Using the "Logical Path"

Head into, how about, `templates/base.html.twig`. Anywhere - I'll go above the
`body` block - add an `img` with `src="{{ asset() }}"` passing this the path to
our file *relative* to the `assets/` directory. So `images/penguin.png`.

[[[ code('262e466110') ]]]

That's it. This is known as the "logical path" to the asset. Because we've pointed
AssetMapper at the `assets/` directory, we can refer to things inside of that via
their path *relative* to that root.

And there's a great way to see *all* assets that are in the AssetMapper paths
by going to the terminal and running:

```terminal
php bin/console debug:asset
```

Awesome! First, on top, it shows the AssetMapper paths, including the `assets/`
directory. This project also has Pagerfanta installed. And we're already seeing
how bundles can add their *own* AssetMapper paths to make their *own* files
available publicly. This won't be important for us, but we could point the browser
at *any* files inside that directory of the bundle.

Below, we see our image file, our CSS file, and our JavaScript file. These
are their filesystem paths and these are their logical paths.

## Versioned Filenames

The point is, by using the `asset()` function and the logical path to an asset,
when we refresh... it works! Woh! And if we Inspect Element, check out the URL!
It contains a *version* hash in the middle! I'm actually going to view the page
source... it's a little easier to see.

So not only is `penguin.png` available publicly, but the path is *not* just
`penguin.png`: it contains a version hash. If we *modified* the source `penguin.png`
file - like gave it a cool bowtie - the version hash would automatically change
forcing anyone using our site to download the fresh file. Booya!

This is also how `app.css` is loaded! Up near the top, the link tag uses
`asset('styles/app.css')`, which is the *logical* path in AssetMapper to that file.
And so it *also* output with a nice, versioned filename.

## How Are the Files Made Public?

Okay, but how does this *work*? If you're like me, you want to know *how* the
sausages are made. Well, in the `dev` environment, it works thanks to a core event
listener... basically a fancy, internal Symfony controller.

For example, when the browser loads this image, that request goes through Symfony.
It sees that we're trying to load `/assets/images/penguin-versionhash.png`, it
finds the *source* file and serves it.

We can prove it! On the main tab, click any icon on the web debug toolbar to go
into the profiler, then click "Last 10" to see the 10 most recent requests
through Symfony. And there it is: the request that served the penguin image. Adorable.

In production, loading our files through Symfony would *not* be fast enough. So,
instead, during deploy, you'll run a new console command:

```terminal
php bin/console asset-map:compile
```

We're going to talk more about deployment later. But this is really cool! It copies
*each* file of every AssetMapper path into the `public/assets/` directory
using their versioned filename. That's it.

Suddenly, this file is no longer being served by Symfony: we're seeing a real,
physical file! Over in `public/assets/`, yep! We can see the final files in all
their glory.

But... while we're developing, remove that directory so that everything continues
to load dynamically.

## Moving favicons into AssetMapper

And actually, while we're here, see those favicons inside the `public/` directory?
We're linking to them at the top of `base.html.twig`. That totally works: the
`asset()` function can *still* refer to things inside the `public/` directory.

[[[ code('d187eec649') ]]]

But... with almost no work, we can add free asset versioning to these! Step 1:
move them into the `assets/images/` directory. Step 2: prefix each path with
`images/` to get their logical path.

And... just like that, we still see the favicon up here... but more importantly,
if we view the page source, those are now versioned!

Let's go a bit deeper into CSS files next. Like, how can we refer to background
images from inside of CSS... if the final filename is versioned?
