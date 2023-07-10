# Importing Specific Package Files

Sometimes, instead of importing a package itself, you may want to import only *part*
of it: like a specific *file* from that package. Lodash is a good example of this.


## No Tree Shaking in the Browser

But before we get there, instead of importing *everything* from `lodash`, you should
be able to say `import { camelCase } from 'lodash'`. Then, down here, you could
then use `camelCase` directly.

*However*, when we move over and try this... error!

> The requested module `lodash` does not provide an export named `camelCase`.

This is *complicated*. Due to the way that *this* specific library, `lodash`, packages
their module, you *can't* import specific functions using this method. It *will*
work with most other packages, however.

For example, if you say `import { Modal } from 'bootstrap'` (if you're using
Bootstrap), that *will* work. Bootstrap packages their files correctly.

However, using this syntax *may* not always be ideal with AssetMapper.

Here's the problem. If we ran this code through Encore, Encore would do something
called "tree shaking". This is where it would see that we're only importing
`camelCase` from `lodash`. And so, in the final JavaScript, it would only give us
the code for `camelCase`, not the *entire* `lodash` package.

In a browser environment, if you `import` from `lodash`, you're going to get *all*
of `lodash`... even if you're only importing one part of it. Now, that *might* not
be that big of a deal. The *full* build of `lodash` is still only 24 kilobytes.
But what if we *are* using a big package... but only need to import one specific
thing?

## Importing a Specific File

A lot of times, there's a specific *file* that we can import, like `/camelCase`.
You'll usually find details about these files in the docs... though you can also
go look for them. Head back to JSDelivr... and down here, search for "lodash".
Below, click "Files" to see all of the files that are part of this package.

For `lodash`, it's a *huge* list... because this is a *huge* library. One of these
is `camelCase.js`.

Ok! So let's try importing `lodash/camelCase`.

I'm not including the `.js`... but it's not going to work anyway. Watch: when
we refresh... error!

> Failed to resolve module specifier `lodash/camelCase`. Relative references must
> start with either "/", "./", or "../"

This error means that we're importing something using a "bare" import and it was
*not* found in the `importmap`. If we "View Page Source", we *do* have an `importmap`
for `lodash`, but not `lodash/camelCase`. Yup, that matching is done *exactly*.
Ok, there *is* a way to do, sort of, "fuzzy" matching - `lodash/*` - but I don't
use that.

The point is: if you want to use `lodash/camelCase`, you should add *that* to
your importmap, not `lodash`.

Watch: find your terminal and run:

```terminal
php bin/console importmap:remove lodash
```

That will remove `lodash` from `importmap.php` and delete the file from the
`assets/vendor/`. Nice! Now run `./bin/console importmap:require` with the package
name / the path that you want: `lodash/camelCase.js`.

```terminal-silent
php bin/console importmap:require lodash/camelCase.js
```

`camelCase.js` is the name of the file over on the CDN. But you'll notice that, a
lot of times in the docs, they'll reference `lodash/camelCase` without the `.js`.
And in this case, you *can* leave the `.js` off: it's up to you. That works because
jsDelivr is friendly and makes both work.

The result of the command? The same thing as before! We get a new entry in
`importmap.php` matching what we want to import set to the a URL. Copy that URL
so we can see it. There we go! We get the code from *just* `camelCase.js`.

And when we try the page... it works!

Here's the takeaway: if you need to import a specific file from a package, you can
do that: just pass the package name + file path and done.

Next, let's add Stimulus to our app!
