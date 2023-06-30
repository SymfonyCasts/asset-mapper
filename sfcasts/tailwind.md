# Tailwind CSS

Our site is *actually* styled in Tailwind, and all of the classes you see here are Tailwind classes. So let's get that installed on our site! Tailwind is something that requires a build step, and that's *totally* fine. Even though we don't have a build step for all of our JavaScript and CSS assets, that doesn't mean we can't add a little build code for a few specific things.

As you can see here, you *could* install Tailwind with Node if you wanted to, and that's actually a really flexible way to do it. You would get a `package.json` file, but instead of having Webpack, Encore, and a ton of other stuff in there, you would just have Tailwind. The other option is to use a standalone executable, and *that* doesn't require *anything*. Let's grab that and open this standalone CLI build. This takes us to their Tailwind release page. Now we need to find the exact version we want. For my purposes, it's "tailwind-macos-arm64", but this can differ depending on your operating system. Copy this link address, head back over here, and now we're essentially just going to download that. We'll use these commands in our terminal,

```terminal
curl -slO
```

and *paste*. Perfect! Then (it doesn't matter where you put this, but I'm going to move it into the `/bin` directory), we'll rename this `tailwindcss` instead of that long name. Finally, since this might be different for certain machines, we're going to go ahead and ignore this new file. So yes, that *does* mean that, with this setup, everyone will need to download their *own* Tailwind CSS file. If you use the Node version, this step wouldn't be necessary.

The very last step is to make that executable:

```terminal
chmod +x bin/tailwindcss
```

And say

```terminal
open bin/tailwindcss
```

to open it. If this is the first time you've downloaded this file, it will ask you to verify that you *do* want to open it.

Okay, now we have this `bin/tailwindcss` executable. It doesn't require Node, and it's *completely* standalone. It's *awesome*. From here, we can just follow the normal docs.  This is the one of the things I really like about the new system. If you *do* need a build system, you can just use Tailwind's build system directly and follow their instructions.

Here, it shows us that we need to run

```terminal
tailwindcss init
```

so let's do that:

```terminal
./bin/tailwindcss init
```

This will create our new `tailwind.config.js` file. Let's go check that out.

The most important thing for us to configure is `content`. This is where our actual HTML files are going to be. And we can actually search for their Symfony-specific documentation for that. Down here, they have a really nice setup. This tells us we're going to be referencing Tailwind classes in our `/assets` and `/templates` directories. Whoops! Let me paste that in the correct spot... There we go. The last step is to copy three baselines for Tailwind and put those inside of our `app.css` file. We can get rid of the Bootstrap stuff, and we'll just keep a little bit of our custom code down here. Nice!

*Now* we're going to point the Tailwind executable at this `app.css` file, have it create a build file, and then we're just going to *reference* that build file. There's nothing very magical about this process. Over at your terminal, run:

```terminal
./bin/tailwindcss -i assets/styles/app.css -o
```

And we'll make sure we put it in the same directory with:

```terminal
assets/styles/app.tailwind.css
```

We're putting it in the same directory so that relative paths like this are still going to work. At the end I'll add

```terminal
-w
```

for "watch". This is a command that you're going to run while you develop. And that's it! If we head over and check this out... we have `app.tailwind.css` with *all* of our stuff inside of it. Awesome! In `base.html.twig`, we're just going to ignore the `app.css` file. That's kind of like our `/src` file now. We're going to point this at `app.tailwind.css` instead. *Moment of truth*. Back in our browser... refresh, and... beautiful! Our site is *actually* styled now. That means we can get rid of this Bootstrap stuff. Remove this Bootstrap CDN, since we were just demonstrating how that works... and we can also get rid of our Bootstrap button down here. That looks good! But what about this Tailwind file? Do we *ignore* that? Do we *commit* it? This is part of the build process, so we *can* commit it, but we're just going to ignore it. Later, we'll see how we build that *before* we deploy.

Back over in our `.gitignore` file, we're *also* going to ignore `/assets/styles/app.tailwind.css`. That's it! It's important to note that individual people on your team will need to download this `tailwindcss` file. You *can* commit one if everyone's using the same things, and then run this command to build those assets. But ultimately, we're just pointing at the build Tailwind file from inside `base.html.twig`. This is what's being served to the end user.

Next: Let's turn to *JavaScript*.
