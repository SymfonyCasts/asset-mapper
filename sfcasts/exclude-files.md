# Excluding Files

We now have CSS, which we're building with Tailwind, we have JavaScript, we're bringing in third-party JavaScript, *and* we're using *modern* JavaScript syntax. Our app has *everything* that a real app has! Sure, it's kind of small, but we're *almost* ready to deploy it. Before we do, let's do a quick audit on the assets that are inside of our AssetMapper. Find your terminal and run:

```terminal
./bin/console debug:asset
```

We know that this lists all of our asset paths, which includes our *main* asset path, plus a couple of bundles that have exposed a few of their own directories. Down here is a list of all of the files that are going to be exposed publicly. We're running this command to double-check and make sure there's nothing in here that we don't expect. In this case, we *do* actually have a minor thing that we don't really want in here, and I'll show you how you can hide it. It's this `/assets/styles/app.css` file. This is really a *source* file, so it's not meant for the user to download directly. We're using Tailwind to build that into the `app.tailwind.css` file, and *that's* what the user will download. It's not a *huge* deal that this is available publicly, but it's a good example of how we can hide source files that we don't want to expose. Let's start by running:

```terminal
./bin/console config:dump framework asset_mapper
```

We're basically asking the system to give us an example configuration for everything that can be configured under `framework.assetmapper`. If you remember, when we first installed AssetMapper, we got a `/config/packages/asset_mapper.yaml` file. Here, we have `framework`, `asset_mapper`, and a key called `paths`. When we run this command... sure enough, up here on top, we have `paths`. Then, below that, we have some other interesting things. The first is `excluded_patterns`. This will be a way for us to hide certain files from the AssetMapper, and we'll talk more about that in a second. You can also control the `public_prefix`, which is where your files are written into the `/assets` directory. This `extensions` isn't *super* important, since it's mostly just for the dev environment... and there are a few other things like your `importmap_path`, and even some attributes you can put on your `<script>` tags dumped by the `importmap`. There's some good stuff in here, but you won't need to need to worry about most of it, aside from `excluded_patterns`, which looks like it could be helpful for our example. Copy that... spin over to `asset_mapper.yaml`, and on the same level of `paths`, say `excluded_patterns`. We want to exclude `assets/styles/app.css`. That's not *quite* correct, and to prove it, after you add it, run

```terminal
./bin/console debug:asset
```

again. If you look up here... `assets/styles/app.css` is still there. That's because this `exclude_pattern` is a file system pattern *and* a glob pattern. That's a fancy way of saying that what you *want* here is to have `*/assets/styles/app.css`, and we'll actually surround this in quotes. This basically says any `"file system path"/assets/styles/app.css` will be ignored. If we try this again... *awesome*. This is what we want to see, because remember, all of the files we see in here will be dumped into our public `/assets` directory. Now our nice source `app.css` file, which doesn't need to be there, *won't* be there when we deploy.

Next: Let's learn how to deploy with "platform.sh".
