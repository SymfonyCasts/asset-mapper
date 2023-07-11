# Adding Stimulus

We can write modern JavaScript in this file, we can import third-party packages:
we're free to write whatever code we want. *But*, if you're like me, you probably
like to use Stimulus. So let's get that installed.

Stimulus is just a JavaScript library, so we *could* just say

```terminal skip-ci
./bin/console importmap:require '@hotwired/stimulus'
```

Then follow their docs on how to get things set up.

## Installing StimulusBundle

But Symfony has special integration with Stimulus. So instead, run:

```terminal
composer require symfony/stimulus-bundle
```

This Stimulus bundle is a relatively new package that houses some Twig shortcuts
that we'll use, like `stimulus_controller()`. But, more importantly, it has a recipe
that will set our app up to load Stimulus controllers *effortlessly*.

Check it out: thanks to those recipe changes, we now have an `assets/controllers/`
directory and there's already a `hello_controller.js` inside of it. Without doing
*anything* else, open up `templates/vinyl/homepage.html.twig` and, right after the
`<h1>`, add a new `<div>`. Let's *attach* that new `hello` controller to that element.
Do that with: `stimulus_controller()` - that's one of the new functions that came
from StimulusBundle - passing `hello`.

That can't *possibly* already work... right? Refresh. It *does*. That's bananas!
And down in the console, we see logs about Stimulus initializing and our
`hello` controller connecting. With just one `composer require` line, Stimulus
is alive in our app.

## How Stimulus Loads

Let's look a bit closer at *how* this works and what the recipe actually did. In
`templates/base.html.twig`, this is probably the least important change it made.
It added `ux_controller_link_tag()`. We'll talk about that in the next chapter when
we explore UX packages. But, in short, if a UX package come with their own CSS,
this will output that. Right now, it's not doing anything.

More importantly, the recipe added a new `assets/bootstrap.js` file. And, in
`assets/app.js`, it patched in some code to *import* that file. So, `app.js`
loads, that imports `bootstrap.js`, and then *that* imports `@symfony/stimulus-bundle`.

Ooh, that's a bare import! It doesn't start with "../" or "./"! That means our browser
will look for this in the `importmap` to figure out which file to load.

Ok! Go open `importmap.php`. Surprise! The recipe added two new entries: One for
the `@hotwired/stimulus` library itself and *another* for `@symfony/stimulus-bundle`,
which points to this weird looking `path`.

Up here, when using a CDN, the entry will have a `url` key. When pointing to a *local*
file, the entry will have a `path` key, which will be the *logical* path to a file
in AssetMapper.

But, what does this weird path point to? Spin over to your terminal and run:

```terminal
php bin/console debug:asset
```

If you scroll to the top... ah! When we installed StimulusBundle, it added *another*
"asset path" to our system, which points to `vendor/symfony/stimulus-bundle/assets/dist`
and it has a "Namespace prefix". This means that, to point to a file in this directory,
the logical path will start with `@symfony/stimulus-bundle`.

So over here, when we say `@symfony/stimulus-bundle/loader.js`, we're referring to
this file right here: `vendor/symfony/stimulus-bundle/assets/dist/loader.js`. That's
the long way of saying, when we import `@symfony/stimulus-bundle`, it's actually
importing this `vendor/symfony/stimulus-bundle/assets/dist/loader.js` file. The bundle
exposes that file by adding the AssetMapper "asset path", which allows the recipe
to add an entry to `importmap.php` that points to it.

## How our Controllers are Registered

Okay, so we're loading this `loader.js` file, and we can see that over here. In your
browser, refresh... go to your Network tools, and search inside this box here for
"loader". There it is! Open this up in a new tab.

This code has functions to start the Stimulus application and register the controllers
from our app - like `hello_controller.js`. But... wait. This is just a hard-coded
file. How is it able to *dynamically* find and load the files that live inside of
our `assets/controllers/` directory?

The key is on top: `import`, `isApplicationDebug`, `eagerControllers`,
`lazycontrollers` from `./controllers.js`. This... is a bit of *magic*. Go back to
the Network tools and search for "controllers"... there it is - `controllers.js`.
Open *this* new tab. Woh! It has `import controller_0` from
`../../controllers/hello_controller.js`, which it then exports to a variable called
`eagerControllers`.

This file is built *dynamically* by the bundle. If we look down in the `vendor/`
directory, `loader.js` is a nice static file. But if you look at `controllers.js`,
it doesn't look like at *all* like what we have in the browser! When this file
is served, AssetMapper intercepts this, looks inside of our `assets/controllers/`
directory, finds all of the controllers there, and then returns a *dynamic* file
based on these.

Watch. Create another file called `goodbye-controller.js` (you can use dashes or
underscores). Change the text to `Goodbye controller!`.

You might expect that, when we refresh the file, we'll see the new controller pop
up here. And you're *almost* right. What really happens is... *nothing*! No change!
Or you might even get a 404 error. That's because the *content* of this file just
changed and so the *hash* will also change. We're looking at an out-of-date version
of the file!

Back on the site, if you refresh, we *should* see a new file with a new hash. We
*don't*, due to a caching bug which has already been fixed. To work around that,
I'll run:

```terminal
./bin/console cache:clear
```

Then go refresh. *Now* I see that this has a different file name, and the contents
*have* dynamically changed to include `goodbye-controller.js`!

So that's the deep dive into how Stimulus works with AssetMapper. The important
thing is that `bootstrap.js` loads a file that starts Stimulus... and that
automatically loads everything inside the `assets/controllers/` directory...
as well as any 3rd party UX packages in `assets/controllers.json`. Let's talk
about *that* part of Stimulus next.
