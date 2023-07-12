# Debugging

There are a few ways that things can go wrong with Asset Mapper, but also some telltale signs when they do. I want to show you a few of the most common issues.

One of the simplest ways to mess things up is in `/templates/vinyl/homepage.html.twig`, if you happen to use the `asset()` function and use a wrong `path`. Remember, inside the `/assets` directory, we have this `penguin.png`. If you're using this graphic, `images/penguin.png` would be its asset path, so we'll say `images/`, *but* then we'll say `duck.png`. This is obviously not the right path and, not surprisingly, on the homepage, we get a 404. The key thing about this 404, if we look at our Console, is this suspicious looking path. Why? If you look closely, you'll see there's no *version* in the file name. This tells me that this path wasn't found inside of any of the asset paths. In other words, we've got some sort of typo. This is not a valid, logical path. If you'll recall, you can see *all* of the valid paths by saying:

```terminal
./bin/console debug:asset
```

Up here, this is everything that you're allowed to put into the `asset()` function that will go through the AssetMapper. And *there's* `images/penguin.png`. And if we put `images/penguin.png` there instead... now it *works*. The thing you want to look for here is the version hash in the file name. That's how you'll know if it's going through the AssetMapper system.

Okay, another common thing you might mess up is an import somewhere. Like... maybe in `/styles/app.css`, you mistype a part of this URL path. Or maybe, in `app.js`, you're importing `vinyl.js` and you forget the `.js` at the end. Accidents like this give us the same result. When we refresh, we're going to get a 404. You can see that here.

Again, the key thing to notice here is *no version hash*. When that's missing, that's a sign you have an invalid path where this is being used. In this case, we have an invalid path in `app.js`. When we're inside of a *template* and using the `asset()` function, we're using the *logical* path. If we're inside of `app.js` or `app.css`, we're not using the *logical* path, just the *relative* path. This is pretty cool. We get to code inside of here as if we don't have an AssetMapper system, so we don't have to think about logical paths. We're just thinking about which path is relative to this file and, in this case, it's `./lib/vinyl.js`. The *point* is that a missing version hash here can be a really good hint that that path ultimately wasn't found inside of the AssetMapper system.

Another way to see any problems you might have is to head over and run:

```terminal
./bin/console cache:clear
```

That clears Symfony's cache, as well as the assets inside of AssetMapper, which are cached internally. By clearing that, now when we run

```terminal
./bin/console debug:asset
```

it's going to build the cache for all of those assets internally. That's important because it's going to parse those files, and if it finds any missing imports like this, it will report them. Check it out:
```
WARNING [asset_mapper] Unable to find asset
"./lib/vinyl" imported from "[...]/asset-mapper/
assets/app.js.
```

And in this case, you even receive an extra message -

```
Try adding ".js" to the end of the import
```

since it sees that it's missing. This is really helpful to see what's going on. If we add that `.js` back... things work again.

The last common way to mess things up is to use a bare import - something that doesn't start with `./` or `../`, but that doesn't exist in your `importmap`. Here, the intention is to use the Bootstrap library, but we don't have it in our `importmap`. This will give us an error, and the exact error will look a little different depending on your browser. I see:

```
Failed to resolve module specifier "bootstrap". Relative references must start with either "/", "./", or "../".
```

This is basically saying:

`Hey, if you're trying to refer to a file with a
relative path, it should start with "./" or "../". And
if you did mean to have a bare import, we can't
figure out what that is because it's not inside the
importmap.`

The usual solution for this is to run

```terminal
importmap:require
```

to install the library that you're using.

Okay, with those tools in hand, let's move on to *deploying*. That's *next*.
