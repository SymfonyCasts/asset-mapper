# Symfony UX Stimulus Packages

We can now create custom Stimulus controllers with ease. The *other* half of
StimulusBundle is the ability to get more *free* Stimulus controllers by installing
a UX package. Let's add one and see how it works.

## Installing Turbo

Let's start by adding Turbo. At your terminal, say:

```terminal
composer require symfony/ux-turbo
```

Here's the great thing about this: just like when we added Stimulus, there's
absolutely *nothing* else you need to do to set this up. Refresh and... it just
works! Turbo eliminates the need for full page refreshes. Head over to your Network
tools and click on "Fetch/XHR". Let's actually clear this out so we can see
everything. Perfect. Then, if we click up here... you can see that this is coming
from an AJAX call! Yup, those full page refreshes are *gone*. So Turbo *just works*.
There's no build system to get in the way, and that's *beautiful*.

## UX Packages Often add Importmap Entries

In practice, this works because a new JavaScript file is being loaded called
`turbo_controller.js`. Filter the network calls to JavaScript and refresh, because
I cleared it. And... there we go! Our page loads `turbo_controller.js` and
*that* imports `@hotwired/turbo`, which *starts* Turbo.

Open up `importmap.php`. When we installed the UX Turbo package, its recipe
*added* this new `@hotwired/turbo` entry. This is a *really* common pattern with
UX packages: if a UX package depends on a third-party package, its recipe will
add that package to your `importmap` *automatically*. The result is that, when
that package is referenced - like `import '@hotwired/turbo'` - it just works.

## How UX Controllers are Loaded

The *real* question is: who's loading `turbo_controller.js`, which lives deep
inside of the `symfony/ux-turbo` PHP package?

The answer is: the same trick we learned a moment ago. Search for `controllers.js`
and open it in a new tab. This is the dynamic file that StimulusBundle builds. As
it turns out, it looks for packages in our `assets/controllers/` directory, which
is these two, *and* it reads the `assets/controllers.json` file. When we installed
UX Turbo, it added this new section here, which is where we activate different
controllers. It activated one called `turbo-core` with `"enabled": true` and added
another deactivated one with `"enabled": false`. So when this file is built, it parses
the `assets/controllers.json` file, finds the controllers that we've enabled, and
adds them here.

The end result is that it imports that controller file here and exports it so that
the `loader.js` file can register it in Stimulus. The point is, any controllers in
`assets/controllers/` *or* in this file will automatically be registered.

## autoimport & CSS Files

Head back into `base.html.twig`. When we installed StimulusBundle, it added
`ux_controller_link_tags()`. Right now, that does nothing. *However*, some UX
packages come with CSS files. You'll find them under a key called `autoimport`,
which the recipe will add under the controller. This `ux_controller_link_tags()`
finds all the CSS files for all the controllers you have activated, and it outputs
them. Nothing too fancy.

Next: let's learn one more thing about Stimulus, which just happens to be one of
my *favorite* things: how to make our controllers *lazy*.
