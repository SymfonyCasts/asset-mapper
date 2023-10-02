# Excluding Files

We now have CSS - which we're building with Tailwind - we have JavaScript, we're
bringing in third-party JavaScript, *and* we're using *modern* JavaScript syntax.
Our app has everything that a real app has! Sure, it's kind of small, but we're
*almost* ready to deploy it.

## Checking your Exposed Files

Before we do, let's do a quick audit on the assets that are inside AssetMapper.
Find your terminal and run:

```terminal
php bin/console debug:asset
```

This lists all of our asset paths, which includes our *main* asset
path - `assets/` - plus a few from bundles that have exposed their own directories.
Below is a list of *every* file that will be exposed publicly.

We're running this command to see if there's anything in this list that we do *not*
want to publicly expose. For example, this `assets/styles/app.css` file. This is really
a *source* file: it's not meant for the user to download directly. We're using
Tailwind to build that *into* `app.tailwind.css`, and *that's* what the user
will download. It's not a *huge* deal that this is available publicly, but it's a
good example of how we can hide "source" files that we *don't* want to expose.

## Asset Mapper Config

Start by running

```terminal
php bin/console config:dump framework asset_mapper
```

We're asking the system to give us example configuration for everything that can
be configured under `framework`, `asset_mapper`. When we first installed AssetMapper,
its recipe gave us a `config/packages/asset_mapper.yaml` file. Here, we have
`framework`, `asset_mapper`, and a key called `paths`. When we run this command...
sure enough, up here on top, it shows `paths`. Below that, we have some other
interesting things.

The first is `excluded_patterns`. *This* is how we're going to hide certain files
or paths - and we'll talk more about that in a minute. You can also control the
`public_prefix`, which is where your files are output to in the `public/` directory.

This `extensions` isn't *super* important... it's mostly just for the `dev`
environment... and there are a few other things like your `importmap_path`, and even
some attributes you can put on the `<script>` tag that's dumped by the `importmap()`
function.

## Excluding Files / Patterns

So there's some good stuff in here... but you won't need to worry about most
of it, aside from `excluded_patterns`.

Copy that key, spin over to `asset_mapper.yaml`, and on the same level as `paths`,
paste. We want to exclude `assets/styles/app.css`.

[[[ code('07a53d9490') ]]]

But this isn't *quite* correct. To prove it, run

```terminal
php bin/console debug:asset
```

again. If you look up... `assets/styles/app.css` is still there! That's because
`excluded_patterns` is meant to be a *glob*. In other words, change this to
`*/assets/styles/app.css`... and surround it by quotes.

[[[ code('42f5f44348') ]]]

This says that any "filesystem path" that ends with `/assets/styles/app.css` will
be ignored. And when we try the command again...

```terminal-silent
php bin/console debug:asset
```

*Awesome*. *This* is what we want to see. Every file here will be dumped into the
`/public/assets` directory. The fact that `assets/styles/app.css` is *not* here
means that it will *not* be dumped into the `public/` directory.

I think it's time to deploy our site! Let's get a deploy set up next on platform.sh.
