# Deploying the Assets

How do we get our assets on this site? If you "View Page Source", it *looks* like things are working. We can see our `importmap` and, down here, all of these paths look the same with the version file names in them. The *problem* is that all of these are just 404ing. In the dev environment, there's some Symfony code that will serve all of these files to us. But in the prod environment, that system isn't even active. It's too slow to run on production, so everything just 404s. And that's okay! A long time ago, we learned about a command we can run to fix this:

```terminal
./bin/console asset-map:compile
```

Simply put, this takes all of the assets that AssetMapper knows about and moves them into the `/public/assets` directory. It's not a command you'll need to run locally, but it *is* something you'll need to run on production. Copy this `asset-map:compile`, head over to `platform.app.yaml`, and go down to the `build` step. This is pretty cool! We're going to let Symfony do its `build` thing, and afterwards, we can just add our own stuff. Right here, say `php bin/console asset-map:compile`. That should do it!

Head back over here and run:

```terminal
git add -p
```

That white space bothers me... there we go. And then we'll run

```terminal
git commit -m
```

with a commit message

```terminal
"asset-map:compile"
```

and finally,

```terminal
symfony deploy
```

It may take a minute or two for it to build that and deploy it. In here, you can actually see the command running: `Compiling assets to /app/public/assets/`, `Compiled 16 assets`. It also writes a few other files inside the `/public/assets` directory: `manifest.json` and `importmap.json`. These aren't important but, because they're there, Symfony is going to use them. These actually help dump the `importmap` and other files on the page *even faster*, and it doesn't require any calculations to do that. And... done! Spin over, refresh, and... it *still* looks bad. But check this out! If you head to the homepage, "Inspect" it, and look at the Console... our JavaScript *is* running. You can see our `console.log()` right there. So... our JavaScript is running, but our *CSS* is missing, and you can see that right here. We have a 404 for our `app.tailwind.css`.

Remember, when you have 404s and you don't see a versioned file name here, it means that AssetMapper's systems are having trouble finding that file. Do you remember what the problem is? This `app.tailwind.css` is a file that we're *building*, and it's not committed to our repository. We have this Tailwind script going here, so we'll stop that and rerun it. So we're *building* the `app.tailwind.css` file, it's being ignored from Git, and when you *deploy* with Platform.sh, it's deploying *from* Git, so that file is simply *missing*. That's not a big deal. This is just another thing we're going to add to our `build` step.

We need to do this *before* we run `asset-map:compile` so the file is available. We'll paste in the code for this... and, effectively, this is the same code that we ran earlier to set things up locally, except that, in this case, we're using the `linux-x64` build. We're downloading that during build, we'll move it into the `/bin` directory (it doesn't really matter where it goes), we're making it executable, and then we're running that same command on its input and output so that the file's *there* by the time `asset-map:compile` runs.

Okay, back over here, let's add that... and deploy again. And even while it's deploying, we can see this working. Last time, there were 16 files, now there are 17. And it looks like running Tailwind will work just fine. Once that finishes, spin over, refresh and... it's *alive*! All of the pages on our site have working CSS. I love it!

So we're on production, and it was really just as simple as making sure Tailwind is being built, and running the `asset-map:compile` command. But there are a few *other* things I want you to think about once you're on production. The *first* is checking that your web server can use HTTP/2. We talked about that earlier, so hopefully you already ensured that's the case.

*Second*, notice that nothing in AssetMapper ever combined our files. For the most part, they don't need to. We're going to talk more about that in a moment. You may have also noticed that nothing has *minified* our files. We *do* want our CSS and JavaScript files to be minified, or if not minified, at least *compressed*. But this is something that can be done by our web server. Basically, we want our web server to compress the assets so that they're smaller when they're being sent to our user. This is something that all web servers support, and it's done *automatically* by Platform.sh. Here's how we can see it: If we go to "Network" tools and select JavaScript... perfect. Now select one of our JavaScript files here, then "Headers". I'll make this a little bigger. Okay, see this "Content-Encoding"? *That's* compression. This "br" stands for something called Brotli, which is a very advanced compression engine. Another thing that would work here is Gzip. So all of our static files are already being compressed, and we get that for *free* with Platform.sh. We can check that item off our list.

Minifying is a lot like compression, except that minifying *also* removes comments. Web generally servers *don't* do that, but if you use Cloudflare, there *is* a way to utilize auto-minification and make your files a little smaller, should you choose to. That's not something to really worry about, but it's good to know you have options.

The third thing we want to do is make sure that all of our static files have *long-term* expiration. Since we have these nice versioned file names, when a user visits our site, we want them to download this file *one* time and *never* download it again. We want them to cache it forever, because if we change any code inside of this file, this *whole* file name will change. As a result, they're going to download the *new* file the next time they visit the site. Over here, under "Headers", we can see "Expires". Out of the box, Platform.sh gives static assets an "Expires" header of around an hour. We can do *much* better than that.

Over in `.platform.app.yaml`, under `locations`... there we go... we can see `expires: 1h`. That's *fine*, so we're going to leave that alone, but we'll put another rule under here to be a little more specific. Inside of this, we're going to add a regular expression - so a URL that starts with `^\/assets\/.*` - and below that, `expires: 365d`. Basically, in internet terms, that's *forever*. Let's commit that... and deploy it.

Once that finishes, *refresh*. We won't see an obvious difference, but let's check out one of our JavaScript files - `app.js`. What we're looking for, down here, is "Expires", which is *so far* in the future. If you *don't* see that, do a force refresh because you may have the old cached version of the file from earlier. And if you *do* see this, *congratulations*! That's want to see. *So*, our assets are being compressed and they have long-term "Expires" headers. We've checked *all* of our boxes for a performance site!

Next: We're going to *prove* it by using Lighthouse to actually *profile* the performance of our site. We'll learn about how files are downloaded, how the pages are built, and we'll make our frontend even *more* efficient.
