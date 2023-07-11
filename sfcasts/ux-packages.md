# UX JS Packages

We can now create custom Stimulus controllers with ease. The *other* half of StimulusBundle is the ability to get more Stimulus controllers from the UX packages, so let's install one and see how it works.

Pretend that we want to get Turbo. at your terminal, say:

```terminal
composer require symfony/ux-turbo
```

Here's the great thing about this: Just like when we added Stimulus, when we refresh the page, it just *works*. Turbo is going to eliminate the need for those full page refreshes. Head over to your Network tools and click on "Fetch/XHR". Let's actually clear this out so we can see everything. Perfect. Then, if we click up here... you can see that this is actually coming from an AJAX call. And if we click around the page, those full page refreshes are *gone*. So Turbo *just works*. There's no build system to get in the way, and that's *beautiful*.

In practice, is this is working because a new JavaScript file is being loaded called `turbo_controller`. I'll go to JavaScript and refresh, since I cleared it, and... there we go! You can see that this `turbo_controller` is being loaded, and *that* imports `@hotwired/turbo`, which is what *starts* Turbo. A second ago, when we installed the UX Turbo package, if we look at `importmap.php`... that *added* `@hotwired/turbo` here. This is a really common pattern when you install UX packages. If those UX packages depend on some third-party JavaScript, they're going to add that third-party JavaScript to your `importmap` *automatically* so that, when it's referenced, it just works. The question is: Who is loading this `turbo_controller`?

It's coming from deep inside the Symfony UX Turbo package itself, and this is actually the same trick we learned a moment ago. Search for `controllers.js` and open a new tab. This is the dynamic file that StimulusBundle builds. As it turns out, it looks for packages in our `/assets/controllers` directory, which is these two, and it also reads the `/assets/controllers.json` file. When we installed UX Turbo, it added this new section here, which is where we can activate different controllers. We can see that it activated one called `turbo-core` `"enabled": true`, and another option that's `"enabled": false`. So when this file is being built, it parses our `/assets/controllers` directory, finds the controllers that we've enabled, and this basically says that we want to enable this *specific* controller in that bundle. The end result is that it imports that here and returns it so the other file can register it inside of Stimulus. The point is, any controllers in the `/assets/controllers` directory *or* this file will be made available and registered *automatically*.

One little detail we didn't talk about with the UX packages is in `base.html.twig`. When we originally installed StimulusBundle, it added `ux_controller_link_tags()`. Right now, that's doing nothing. *However*, if you install certain UX packages, those packages will actually come with CSS files. You'll find them under a little key called `autoimport`. When you install the package, you'll get something that looks like this, but it will also have this `autoimport` here that points to some CSS file. This `ux_controller_link_tags()` finds all of those CSS files for all the controllers you have activated, and it outputs them. Most UX packages don't have this `autoimport`, but that's how it's going to be output.

Let's learn one more thing about Stimulus: How to make our controllers *lazy*. It's *super* easy. That's *next*.
