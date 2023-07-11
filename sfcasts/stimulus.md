# Adding Stimulus

*So* we can write modern JavaScript in this file, we can import third-party packages, and we're free to write our JavaScript however we want. *But*, if you're like me, you probably want to use Stimulus. Let's get that installed.

Stimulus is just a JavaScript library, so we *could* just say

```terminal
./bin/console importmap:require '@hotwired/stimulus'
```

and follow their documentation on how to get things set up. But Symfony has special integration with stimulus, so instead, we're going to run:

```terminal
composer require symfony/stimulus-bundle
```

The Stimulus bundle is a relatively new bundle that houses some Twig shortcuts that we're going to use, like `stimulus_controller()`. But, more importantly, it has a recipe which will set our application up to load the stimulation controllers *effortlessly*. We'll talk about some of those recipe changes in a moment, but before we do, check this out. Thanks to those recipe changes, we now have an `/assets/controllers` directory, and there's already a `hello_controller.js` inside of it. Without doing anything else, we're going to go into `/templates/vinyl/homepage.html.twig` and, right after the `<h1>`, add a new `<div>`. This is going to use that controller, so say `stimulus_controller()` (that's one of the new functions that came from StimulusBundle) with `hello` inside. Just like that... it works! And down here, you can see that we have logs about Stimulus initializing and our `hello_controller.js` matching. So even without a build system, we "composer required" Stimulus and it just works. I love that!

All right, let's look a little closer at *how* this works and what the recipe changes actually did. In `/templates/base.html.twig`, this is probably the least important change it made. It added this line called `ux_controller_link_tag()`. We'll talk about that in the next chapter when we talk about UX packages, but, in short, sometimes UX packages come with their own CSS and that is going to output them. Right now, it's not really doing anything, but it *did* add this new `/assets/bootstrap.js` file and, in `/assets/app.js`, we get some code to import that. So when this file runs, we're importing `bootstrap.js`, and then *that* is importing from this `@symfony/stimulus-bundle`.

We know this! This is a *bare* import. It doesn't start with "../" or "./", and we know what that means. It's going to look in the `importmap` to figure out what that file is. If we look in our `importmap.php` file... *surprise*! The recipe added two new entries: One for Stimulus itself and *another* for `@symfony/stimulus-bundle`, which points to this weird looking `path` right here.

You may have noticed that, up here, when we're using a CDN, we're going to have a URL key. When we're pointing to a *local* path, it's going to use this `path` key. This is actually a local AssetMapper path. What does that mean? And *whereW is it pointing to? Spin over to your terminal and run:

```terminal
./bin/console debug:asset
```

If you scroll to the top... ah! Check this out! When we installed StimulusBundle, it added *another* asset path to our system, which points to `vendor/symfony/stimulus-bundle/assets/dist` and it has a "Namespace prefix". That means that any paths inside of this directory are going to start with `@symfony/stimulus-bundle`. So over here, when we say `@symfony/stimulus-bundle/loader.js`, we're actually referring to this file right here: `vendor/symphony/stimulus-bundle/assets/dist/loader.js`. That's the long way of saying, when we import `@symfony/stimulus-bundle`, it's actually importing this `vendor/symfony/stimulus-bundle/assets/dist/loader.js` file. The bundle basically exposed that to us by adding the AssetMapper path, and then the recipe added that to `importmap.php` so it loads.

Okay, we're loading this `loader.js` file, and we can see that over here. In your browser, refresh... go to your Network tools, and search inside this box here for "loader". There it is! If we open this up in a new tab... check this out - `/assets/@symfony/stimulus-bundle/loader.js`... and *there's* that code! So this file contains a bunch of code that starts the Stimulus application and then renders the controllers from our application. *But* this is just a hard-coded file. How is StimulusBundle able to *dynamically* load our files inside of the `/controllers` directory?

Check out this `import` up here - `import { isApplicationDebug, eagerControllers, lazycontrollers } from './controllers.js`. This is *magic*. If we go back to our Network tools and search for "controllers"... there it is - `controllers.js` - and open *this* new tab... look at this! It actually has `import controller_0 from '../../controllers/hello_controller.js`, and then it exports that in a variable called `eagerControllers`. This file is *dynamically* built by the bundle. If we look down in the `/vendor` directory, `loader.js` is just a nice static file. But if you look at `controller.js`, it doesn't look like what we have in the browser. The AssetMapper system intercepts this, looks inside of our `/controllers` directory, finds all of the controllers here, and then returns this updated version which we then use to *Bootstrap* all of our controllers. Watch this!

Let's make another file here called `goodbye-controller.js` (you can use dashes or underscores), and we'll do the same thing here, but say `Goodbye controller!`. As soon as we do this, you might expect that, when we refresh this page, we're going to see it pop up here, and you're *almost* right. What really happens is... *nothing*. Or, depending on your app, you might even get a 404 error. That's because the *content* of this file is going to change and so the *hash* is also going to change.

Right now, you can see that the file being downloaded is `9493a`. Head over and run

```terminal
./bin/console cache:clear
```

and then refresh. Now we can see that it has a different file name, and the file inside has dynamically changed at that spot. We'll hopefully fix that bug soon, but our `goodbye-controller.js` is there. So that's a deep dive into what's going on here. Ultimately, the important thing is that `bootstrap.js` loads a file that starts Stimulus and will automatically load everything instead of a `/controllers` directory and all the third-party libraries in `controllers.json`. We're going to talk about that by installing a UX package *next*.

