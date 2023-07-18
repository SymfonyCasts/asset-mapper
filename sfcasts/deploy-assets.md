# Deploying the Assets

How do we get our assets onto this site? If you "View Page Source", it *looks* like
things are working. We can see the `importmap` and... down here, all these paths
look correct: they even have the version part in their filenames.

## Compiling Assets for Production

Unfortunately, all of these files are returning a 404. Boo. In the `dev` environment,
when we're working locally, these files don't physically exist. But an internal
Symfony listener intercepts the request, finds the file, and returns them.

But in the `prod` environment, that system isn't even active. It's too slow to run
on production... so everything just 404s. And that's okay! A long time ago, we learned
about a command to fix this:

```terminal
php bin/console asset-map:compile
```

This command's job is simple: takes all of the assets that AssetMapper knows about
and move them into the `public/assets/` directory. It's not a command you'll need
to run locally, but it *is* something you need to run when you deploy.

Copy this, head over to `platform.app.yaml`, and go down to the `build` step. This
is pretty cool! We're going to let Symfony do its `build` thing, and afterwards,
we can add our own stuff. Right here, say `php bin/console asset-map:compile`.
That should do it!

Why are we running this during `build` and not `deploy`? As a rule of thumb, if
a command's job is to "prepare" some *files*, it should be in the `build` step.
Or, another way to think about it is: if a command does *not* require a connection
to the database, there's a good chance it's a "build" thing.

Head back over here and run:

```terminal
git add -p
```

That white space bothers me... so I'll fix it and preserve my sanity. Then run
`git commit -m` with a fancy message.

```terminal-silent
git commit -m "asset-map:compile"
```

You know what's next. Punch it!

```terminal skip-ci
symfony deploy
```

Let's fast forward to the good part. Here it is! We see it running the command!


> Compiling assets to `/app/public/assets/`
> Compiled 16 assets

It also writes a few other files inside the `public/assets/` directory:
`manifest.json` and `importmap.json`. These aren't important but, because they're
there, Symfony will use them. Their job is to help dump the `importmap` and other
things onto the page *even faster*.

And... done! Spin over, refresh, and... it *still* looks bad!? Ah, but things are
not as bad as you think! Head to the homepage and open your Console. Hey! Our
JavaScript *is* running! We see the `console.log()`!

## Building Tailwind on Deploy

So JavaScript, check! CSS... not so much. We still have a 404 on
`app.tailwind.css`.

Remember: when you see a 404 to a filename that does *not* include a version part,
it me here, it means that AssetMapper is having trouble finding that file. Can you
spot the problem? This `app.tailwind.css` is a file that we're *building*... and
it's *not* committed to the repository! I'll stop this command and rerun so we
can see. Yup, we're *building* the `app.tailwind.css` file, it's being ignored from
Git, and since Platform.sh deploys using our files *from* Git, that file is simply
*missing*.

No big deal. This is just another thing we need to add to our `build` step...
*before* we run `asset-map:compile` so that the file is available.

I'll paste in the code for this. This is basically the same code that we ran earlier
to set things up locally, except that, in this case, we're using the `linux-x64`
build. We're downloading that during build, moving it into the `/bin` directory
(it doesn't really matter where it goes), making it executable, and then running
that same command so that the output file *is* there by the time `asset-map:compile`
runs.

Oh, and don't forget about the TailwindBundle which makes using Tailwind - including
this deploy step - a bit easier.

Back over here, let's commit that new config change.... then deploy again:

```terminal-silent skip-ci
symfony deploy
```

Even while it's deploying, we can see this working. Last time, there were 16 files,
now there are 17. Once that finishes, spin over, refresh and... it's *alive*! All
of the pages have working CSS. I love it!

Next: now that we're on production, let's talk about the things *you* need to
check to make sure your assets are served blazingly fast.
