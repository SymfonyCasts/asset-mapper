# Debugging

Sometimes things won't work. But, with AssetMapper, there are some telltale signs
when things go wrong. I want to show you a few of the most common issues.

## Typo in Logical Path? Missing Versioned Path

One of the simplest ways to mess things up is in
`/templates/vinyl/homepage.html.twig`: if you use the `asset()` function and pass
an invalid path. Remember, in the `assets/images` directory, we have this
`penguin.png`. So, `images/penguin.png` is its "logical path" in AssetMapper.
Let's say `images/`, *but* then `duck.png`.

This is obviously not the right path or even the right animal. So no surprise that,
on the homepage, we get a 404. The key thing about this 404, if we look at our
console, is its suspicious-looking path. Look closely: there's no *version* in the
filename! This tells me that this path was *not* found in any of the AssetMapper
directories: its an invalid logical path. And so, AssetMapper ignored it and returned
the raw, unversioned filename.

To help visualize the *valid* logical paths, remember that you can run:

```terminal
php bin/console debug:asset
```

Up here, this is *everything* that you're allowed to pass to the `asset()` function.
And *there's* `images/penguin.png`. And if we put `images/penguin.png` here
instead... now it *works*.

The key thing to look for is the version hash in the filename. If it's not there,
AssetMapper couldn't find your path.

## Invalid Import Paths

Another common mistake is to mess up an *import* somewhere. Like... maybe in
`/styles/app.css`, we mistype a part of this image `url()`. Or, in `app.js`,
when importing `vinyl.js`, we forget the `.js` at the end.

Accidents like this give us the same result as the first mistake! When we refresh,
we're going to get a 404. You can see that here. But again, the *key* thing to notice
is the missing *version hash*. That's a sign that the path couldn't be found, so
it couldn't be handled by AssetMapper.

In this case, the invalid path lives in `app.js`. When we're inside of a *template*
and use the `asset()` function, we pass the *logical* path to a file. But if we're
inside of `app.js` or `app.css`, we *don't* using the logical path: just the
*relative* path. This is by design. We get to code inside of these files as if
the AssetMapper doesn't exist. We don't need to think about logical paths, we
just think:

> What path would I use if these files were all just being served directly
> to my browser?

Anyway, if the version has is missing, we have an invalid path, which could be
an invalid logical path in a template *or* an invalid *relative* path in some
other file.

## Seeing a List of Invalid Paths

By the way, there's an almost hidden way to see if any invalid imports appear
*anywhere* in your code. First, run:

```terminal
php bin/console cache:clear
```

That clears Symfony's cache, of course, but it also clears an internal cache in
AssetMapper. Now when we run

```terminal
php bin/console debug:asset
```

it rebuild the cache for all of those assets internally. To do that, it will parse
our files and report any missing imports. See?

> WARNING Unable to find asset `./lib/vinyl` imported from `assets/app.js`.

And in this case, you even receive an extra message:

> Try adding ".js" to the end of the import

Good idea. If we add that `.js` back... things work again.

## Importing Missing "Packages"

The last common way to mess things up is to use a bare import - an import that doesn't
start with `./` or `../` - for something that does *not* appear in your `importmap`.
Here, the intention is to use the `bootstrap` library... but we don't have it in
our `importmap`. The exact error will vary based on your browser, but for me it says:

> Failed to resolve module specifier "bootstrap". Relative references must start with
> either "/", "./", or "../".

This is basically saying:

> Hey! If you're trying to refer to a relative path, you forgot the `./` or `../`
> part! If you're trying to import a package, you forgot to add it to your importmap!

The usual solution is usually to run: `php bin/console importmap:require` to add
that package.

Okay, with those tools in hand, let's move on to *deploying* next!
