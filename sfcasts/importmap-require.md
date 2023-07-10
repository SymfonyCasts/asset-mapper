# importmap:require - 3rd Party JS Libs

In our code, we get to use `import` statements with relative paths, ES6 classes:
everything we're used to. It's business as usual. Except, how can we use third-party
packages?

As we saw earlier, we can actually import things via a full URL, like `import _ from`,
and I'll paste in the same CDN URL that we used earlier. With that done, the rest
is normal: add `_.camelCase()` to the log.

If we refresh and check the console... that *works*. But I *just* don't like it!
I don't want to have to include this crazy URL everywhere I use `lodash`. And what
happens if we upgrade lodash... and need to change the URL in 10 different files?
Lame!

## importmap:require to Fetch Node Packages

If we were using a build system like Webpack, we could just say:

```terminal skip-ci
yarn add lodash
```

or

```terminal skip-ci
npm install lodash
```

And even though we're not using `yarn` or `npm`, we can do *nearly* the same thing.
Over in the terminal, open a *new* terminal window, and run
`php bin/console importmap:require` followed by the name of the NPM package we want:
`lodash`:

```terminal-silent
php bin/console importmap:require lodash
```

Done! It added `lodash` to `importmap.php` and tells us we can use the package as
usual. This means we can say `import _ from 'lodash'` like we normally would, and,
everything works *fine*.

How? When we ran the command, it made one *tiny* change: adding this one small
section to `importmap.php`. And as cool as this is, it's not magic. Behind the
scenes, the command went to the JSDelivr CDN, found the latest version of `lodash`,
then added the `lodash` key set to that URL.

If you head over and look at the page source... no surprise! We have a new `lodash`
entry inside the `importmap`! When our browser sees `import _ from 'lodash'`, it
looks inside of the `importmap` for `lodash`, finds this URL, and downloads it
from *there*. Our browser is the hero!

## Telling your Editor about the Packages

One bummer is that we don't get autocompletion in the eidtor. It says "Module
not installed". And if I say `_.` something... it doesn't really work. It's
autocompleting `camelCase`... but only because I'm using that down here.

I hope this will be better supported in PhpStorm soon. There *is* a workaround, but
it's a bit manual. Copy the package, go into `base.html.twig` and add a temporary
`<script>` tag that points to this. Hit "alt" + "enter" and select "Download library".
This downloads that into the "External Libraries" section down here - `/lodash`.

Ok, remove that `script` tag. Back in `app.js`, it's *still* going to underline
the import as if it doesn't know what it is, but it *does* autocomplete when we
use `_.` something. For example, `tail` is from `lodash`.

## Updating Packages

What about *updating* the versions of packages inside `importmap`? Whelp, there's
a command for that:

```terminal
php bin/console importmap:update
```

That will loop through every package and update its URL to the latest version.
This is *already* the latest version... but if I change this to `.19`... then
run the `update` command... it moves back up to `.21`. The command *could* be
more flexible - like by allowing you to update just one package, or having some
version constraints - and those things may be added in the future.

## Downloading Packages Locally

Finally, if you don't want to rely on the CDN, you don't *have* to. Instead, when
you require the package - or any time later - pass the `--download` option:

```terminal-silent
php bin/console importmap:require lodash --download
```

In `importmap.php`, this *still* shows the source URL to the CDN, but it actually
downloaded that file into an `assets/vendor/` directory. This `downloaded_to` points
to the logical path of that file.

The result? When we go over and refresh.... and "View Page Source"... the `importmap`
now points to the *local* file. We're no longer relying on the CDN.

But... now what? Do we commit this `vendor/lodash.js` file? The answer is... *yes*.
At least at this moment, that's the only way to version that file and keep it in
your repository.

So even *without* npm or yarn, we *can* use *any* npm package we want. Woo!
But *sometimes*, instead of importing an entire package, we may only want to import
a specific *file*. Let's talk about how we can do that *next*.
