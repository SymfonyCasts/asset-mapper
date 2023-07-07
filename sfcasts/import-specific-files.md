# Importing Specific Package Files

Sometimes, instead of importing *everything* from a package, you may only want to import *part* of a package, or even a single *file* from a package. Lodash is a pretty good example of this. Normally, we could say `import { camelCase } from 'lodash'` and, down here, we would use `camelCase`. *However*, when we go over here, we'll get a syntax error:

`The requested module 'lodash' does not provide an export named 'camelCase'`

This is *complicated*. Due to the way that this specific library, `lodash`, packages their module, you *can't* actually import it using this method. It *will* work in other cases, however, so it's a little confusing.

For example, if you say `import { Modal } from 'bootstrap'` (if you're using Bootstrap), that *will* work because Bootstrap actually has their stuff packaged together correctly. There's a problem with this syntax in both cases anyway. With AssetMapper, you need to do a *little* more thinking than you might need to do with the build system.

Here's the problem. Suppose that this *did* work. If we ran this code through Encore, Encore would do something called "tree shaking", where it would know that we're only importing `camelCase` from `lodash`. Ultimately, in the final JavaScript, it would only give us the code for `camelCase`, not the entire `lodash` package. In a browser environment, if you `import` from `lodash`, you're going to get *all* of `lodash`, even if you're only importing one part of it. Now, that *might* not be that big of a deal, since `lodash` isn't a huge library. But in *other* cases where the libraries are much larger, you may only want to import *specific* things.

A lot of times, there's a specific file that we can import, like `/camelCase` , and you'll find those details in the documentation. Lodash will show you that, but you can also go look for it. We can head back to JSDelivr... and down here, we'll search for "lodash". Then, down here under "Files", you can see all of the different files that are available. It's a *huge* list because this is a *huge* library. Here, you can see one called `camelCase.js`. It's very common for these packages to contain additional files that you'll want to import. *We're* going to say `lodash/camelCase`. If we try that... it looks like *that*, by itself, is not going to work. It's going to say `Failed to resolve module specifier 'lodash/camelCase'. Relative references must start with either "/", "./", or "../"`. This error that means that you're importing something and it wasn't found in the `importmap`. If we "View Page Source", we have an `importmap` for `lodash`, but not `lodash/camelCase`. The simplest way to handle `importmaps` is to have them match *exactly*. Instead of using `lodash`, we're using `lodash/camelCase`.

Head over to your terminal and run:

```terminal
./bin/console importmap:remove lodash
```

That's going to remove `lodash` from our `importmap`, and it will also delete the file from the `/vendor` directory as well. *Convenient*. In order to get `lodash/camelCase`, we're going to

```terminal
./bin/console importmap:require
```

with "the package name / the path that you want", so

```terminal
lodash/camelCase.js
```

This `camelCase.js` is actually the name of the file over in the CDN, but you'll notice that, a lot of times in the documentation, they're just going to say

```terminal
lodash/camelCase
```

You *are* allowed to do that.

We'll import this specific file and, just like last time, it's going to add a new entry to our `importmap`. I'll copy that so we can see it. There we go. This is showing you which file it's getting - `camelCase.js` - from that library. Now when we try it... it *works*! So this was a long way of saying that, when you want to import a specific file from a package, you can do that. Just use `importmap:require` to get the specific one you want. That's *it*.

Next, let's add Stimulus to our application.
