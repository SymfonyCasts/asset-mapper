# Installing AssetMapper

We now know that we can run modern JavaScript directly in our browser. But to help
smooth the process, we're going to install a new Symfony component called
AssetMapper.

Find your terminal and run:

```terminal
composer require symfony/asset-mapper symfony/asset
```

I'm including the second package because it gives us that nice `asset()` function
in Twig. It's *already* installed in this project - just make sure you have it
in yours.

Before we start: AssetMapper *is* experimental in Symfony 6.3, so there *will* likely
be some backwards compatibility breaks before 6.4. But as we will see, the concepts
are solid, and you can deploy a super-performant site with AssetMapper today.

## Changes from the Flex Recipe

Ok, run:

```terminal
git status
```

Oooh: the Flex recipe for AssetMapper added several things. Time for a quick tour!
First, it gave us an `assets/` directory... which looks pretty much *identical*
to what you would get if you installed WebpackEncore. We have an `app.js` file -
this will be the main, *one* file that's executed - and also `app.css`: the main
CSS file.

[[[ code('68df41237d') ]]]

[[[ code('9031ca801a') ]]]

In `templates/base.html.twig`, the recipe also added a `link` tag to point to
`app.css`. We're going to talk more about stylesheets later, but you can already
see that the CSS setup is perfectly straightforward.

[[[ code('d44f1d1210') ]]]

The recipe added one more important line to this file: `{{ importmap() }}`. That
partners with a new `importmap.php` file. Those *are* important, and we'll go into
detail about them soon.

The takeaway is that the recipe created a few files in the `assets/` and added a
`link` tag to `base.html.twig`. But otherwise, there's not a lot going on yet.

## AssetMapper "Paths"

Looking back at the terminal, the recipe also created a new `asset_mapper.yaml` file.
Let's open that up: `config/packages/asset_mapper.yaml`.

[[[ code('abd34ac304') ]]]

AssetMapper has one, *main* concept: you point it at a directory or set of directories,
like `assets/`, and it makes all the files inside available publicly, as
*if* they lived in the `public/` directory. We'll see *how* that's accomplished in
a minute.

But before we do *anything* else, refresh the page and... the background turned blue!
That's coming from the `app.css` file. *And*, in the console log, we see a message
that's coming from `assets/app.js`. So, somehow, magically, just by running a
`composer require` command, these two files are already exposed publicly *and* are
being loaded onto the page. Next, let's learn *how* this is all working.
