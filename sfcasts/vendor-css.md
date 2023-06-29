# 3rd Party CSS

We talked about adding CSS to our site, but what about *third-party* CSS like
Bootstrap? With a build system like Encore, we have a `package.json` file and we can
run something like

```terminal
yarn add bootstrap
```

or

```terminal
npm install bootstrap
```

In AssetMapper, because there's no Node, we don't have a *such* an easy system for
grabbing CSS packages. We *can* still get them, it's just one extra step.

## Finding Packages on jsDelivr

I like to use jsDelivr for this: a CDN for *all* NPM packages. Even if you don't
ultimately *use* it as a CDN, it's a nice way to find and download packages.

Search for "Bootstrap" and... there it is. A lot of times, you'll find the CSS file
you're looking for right up here, just like this. I'll hit "Copy HTML + SRI".
If you *don't* see the CSS file here... or you need a *different* file, you can
click the "Files" tab to browse the entire package. For example - `dist/css/`
and then whatever you need.

Okay, we know that CSS is with AssetMapper is delightfully *boring*... so go
back over to the `stylesheets` block and, above `styles/app.css`, paste the *new*
`link` tag.

If you want to *avoid* using the CDN, you *would* download this file directly into
our project. Because there's no package system like NPM, I would probably create
an `assets/vendor/` directory and put the file inside of that. Then I would
*commit* that `assets/vendor/` directory to Git to keep it in your project and
versioned. Committing vendor files into your project isn't *ideal*, but it's not
a huge deal and is your best option right now if you want to avoid a CDN. Anyway,
you'll see me do this later for a JavaScript. file.

Ok, let's see if this is working! Scroll down to the middle of the page and add
a `button` with `btn btn-primary` to use a few common Bootstrap styles.

When we head over to the site and refresh... it works! Lovely!

## Bootstrap Sass?

Ok, but what if I want to *modify* Bootstrap?

Bootstrap itself is built with Sass. And, if you want, you could build Sass files
that *override* Bootstrap variables - for example to change default colors.

Now, there are two important things about this. First, you absolutely *can* use
Sass with AssetMapper. There are details in the documentation about how to do that...
and hopefully we'll add a bundle soon to make it even easier.

Also, in a moment, we're going to add Tailwind CSS to our site, which doesn't require
Sass, but has a very similar workflow because Tailwind needs to be "built".

And, if you *do* want to use Sass with Bootstrap, one simple way get the Bootstrap
source code is via the official Composer package for Bootstrap - so:

```terminal skip-ci
composer require twbs/bootstrap
```

So if this is something you want, check out the AssetMapper docs.

## Bootstrap CSS Variables

The second important thing is that, depending on what you want to do, you *may* not
need to use Sass to customize Bootstrap. That's because Bootstrap also exposes
*CSS* variables, though they're not as powerful.

We can see this down in the "Customize" "CSS Variables" section. CSS variables are
a browser feature that allows you to *set* variables inside of CSS then reference
those variables. No fancy-pants Sass needed.

For example, over in `app.css`... it doesn't matter where, but we usually put this on
top... add a `:root` pseudo selector, which is a common place to initialize variables
that will be used later. Here, override a CSS variable that Bootstrap provides
and uses: `--bs-border-radius`. Set it to `1rem`.

That *should* make the borders noticeably larger. Back at the browser... it works!
The border radius is now larger across the site. That's one of the variables you
would find in the Bootstrap documentation.

However, it's not *always* this simple. For example, let's say we want to override
this primary color. You might think you could do that by searching for "primary"...
up here... and overriding something like `--bs-primary`. That's *sort of* correct.

If you inspect our button, this color *is* the actual background color. But watch.
Try to override that by changing it to a slightly lighter color. Then head back and
try that. It doesn't do *anything*.

Copy the CDN URL, pop that into your browser, and take off the `.min` so we can
see what's going on. On top, it's setting all of those nice CSS variables. Look
for `btn-primary`. I won't get too deep here, but inside of `.btn-primary`, it's
setting these CSS variables to these *hard-coded* colors, instead of *using*
the other CSS variables that we can control.

So what do we do? In this case, we're back to the basic strategy of overriding.
That's not *as* bad as it sounds because *we* can at least use CSS variables to
do this.

Spin back over to `app.css` and I'll paste in some styling for `.btn-primary`.
This overrides the variables that are set by Bootstrap to a different color.
Though, we're at least *using* the `bs-primary`: we set it up here, and can
reference it in as many spots as we want. So, pretty basic CSS overriding, but
at least using CSS variables.

And when we try it... it *does* change the color. Sweet! *So* CSS variables aer
*one* way to customize Bootstrap, but Sass is still the most powerful option.

Next: let's grab one more external styling thing: an open source font!
