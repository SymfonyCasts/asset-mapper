# Tailwind CSS

The HTML on our site is *already* styled with Tailwind: all the classes you see
here come from it. So if we can get Tailwind installed, we should have a *much*
less ugly site.

Tailwind is interesting because it's not just a CSS file you include: it
requires a build step. And that's *totally* fine! Even though we don't have a build
system for *everything* doesn't mean we can't choose to *add* one for some specific
things.

## Using TailwindBundle

Before we dive in... about a week after I recorded this, we created a bundle that
makes it *super* easy to add Tailwind called. It's called, creatively,
[TailwindBundle](https://github.com/symfonycasts/tailwind-bundle)!
Seeing how you can set up a small build system is still interesting - but if you
want to skip this chapter and head over to that bundle instead, it won't hurt my
feelings. The bundle basically automates what we're about to do.

## Downloading the Standalone Executable

To get all of this working, we need the Tailwind binary file. As we see here, we
*could* install it with Node... and that's a really flexible option. You would
have a `package.json` file... but instead of it containing WebpackEncore and a
ton of other stuff, it would just have Tailwind.

The other option, which avoids the need for Node entirely, is to use the standalone
executable. Click the "Standalone CLI build" to go to the Tailwind release page.
Find the version you need: for me, it's "tailwind-macos-arm64". You can download
that here, but I'll copy the link address... so I can download it *fancily* via
curl: `curl -slO` then paste!

It doesn't matter where you put this, but I'm going to move it into the `bin/`
directory and rename it to `tailwindcss`... instead of that long name. Finally,
because other machines - like the computers of our co-workers or the machine
that deploys our site - might be different, let's ignore this file.

So yes, this *does* mean that everyone will need to download their *own* Tailwind
binary.

The very last step is to make this executable. On a Linux-based system, that's:

```terminal skip-ci
chmod +x bin/tailwindcss
```

Also, on a Mac, I need to run:

```terminal skip-ci
open bin/tailwindcss
```

If this is the first time you've downloaded the file, it will ask you to verify
that you *do* want to open it from a security standpoint.

## Initializing Tailwind

Okay! We now have the `bin/tailwindcss` executable, which does *not* require Node.
From here, we can follow the normal docs. This is something I really like about
the new frontend philosophy. If you *do* need a build system, you can just use
Tailwind's build system directly and follow their instructions: no need for a
Symfony-specific solution.

Here, it says that we need to run:

```terminal skip-ci
tailwindcss init
```

So let's do that!

```terminal
./bin/tailwindcss init
```

This creates a shiny new `tailwind.config.js` file. Let's go check it out!

The most important thing is to configure the `content` key. This tells Tailwind
*where* it should look for HTML that may contain Tailwind classes. Search for their
Symfony-specific  documentation. Down here, they have exactly what we want! Copy
the `content`... then paste! I mean... paste it in the correct spot!

The last step is to copy the three base directive lines for Tailwind... and put 
those inside `app.css`. I'll remove the Bootstrap stuff... but keep a little bit
of our custom code down here. Nice!

## Building the CSS File

Finally, we're ready to build! At your command line, run `bin/tailwind`, use
`-i` to point to the input `assets/styles/app.css` file, then `-o` to tell it
where to *output* the final code. Use `assets/styles/app.tailwind.css` so it's in
the same directory, which is important so that any relative image paths will
still work. At the end, add `-w` so it will keep running and watching for changes:

```terminal-silent
./bin/tailwindcss -i assets/styles/app.css -o assets/styles/app.tailwind.css -w
```

And that's it! Built! Over here, we have an `app.tailwind.css` containing *all* the
goodies. Awesome!

In `base.html.twig`, instead of pointing at `app.css` - which is now kind of an
"internal" source file - point this at `app.tailwind.css`.

*Moment of truth*. Back to the browse! Refresh. Our site is styled! That means we
can get rid of the Bootstrap stuff: remove the Bootstrap CDN link... since we were
just demonstrating how that works... and also the button down here.

That looks good!

## Ignoring the Built File

But what about this `app.tailwind.css` built file? Do we *ignore* that from git?
Do we *commit* it?  It's up to you! We *can* commit - it - it would make deploying
easier, but we generally *don't* want to commit built stuff. I *will* ignore it...
then we'll see how that works into our deployment process a bit later.

Ok, done! Next: Let's turn to *JavaScript*.
