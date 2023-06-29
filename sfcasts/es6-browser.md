# Doing Modern JS Right in your Browser

Before we talk about *anything* related to Symfony, we're going to strip things
down to the bare minimum and prove that we *can* code modern JavaScript, right in
our browser.

## Directly Loading Some JavaScript

Go *directly* into the `public/` directory and create a new `app.js` file. To start,
just `console.log()` a message.

This won't be processed by Symfony or *anything*. In `templates/base.html.twig`,
up here in the `javascripts` block, though that doesn't make any difference, add
a boring `<script>` tag for this: `<script src="{{ asset('app.js') }}">`. I *am*
using the `asset()` function... but that's not doing anything either.

[[[ code('36691e46e5') ]]]

Ok, head to the browser, open up your Console and... refresh. There's the log!
It's snooze-worthy, but working.

## Writing Modern JavaScript

Time to make things interesting! Back in `app.js`, copy the mix name. Let's create a
*class*: `class MixedVinyl`, with a constructor and some properties. This uses
the class syntax introduced in ES6, or ECMAScript 6... basically version "6" of
JavaScript. You'll hear ES6 a lot because *most* modern features you're used to
came from this version - released *way* back in 2015.

[[[ code('0a5e0272ff') ]]]

In the `describe()` method, I'm leveraging string interpolation - another modern
feature from ES6 - to return the string. Below, use this: `const` - yet *another*
ES6 feature - `mix = new MixedVinyl()` and pass in the mix name and year. Finally,
`console.log(mix.describe())`.

[[[ code('1cf2eec3fa') ]]]

Cool! *This* is the kind of code I like to write every day. Unfortunately, this
is *also* the kind of code that browsers have historically choked on!

So, *normally*, we would have a build system like Encore that would read this modern
code and rewrite it to *old* JavaScript... so it would work in our browser. But...
tada! It *already* works in our browser! We don't need to do *anything*. And
that's not just because I'm using a new browser. This is going to work in *every*
browser.

If you're ever unsure, go to https://caniuse.com to check it out. Let's look up
"ES6 class". Yup, it's basically supported by *everything*... except for IE 11, which
is dead.

## Using "import" in the Browser

But what about the `import` statement? Copy the `class MixedVinyl` then create
another file directly inside `public/` called `vinyl.js`. Paste this in and
then `export` it: `export default class`.

[[[ code('92a5ef4225') ]]]

Back over in `app.js`, `import MixedVinyl from` and, just like we do in Encore,
use the relative path: `./vinyl.js`.

[[[ code('da3b7f12da') ]]]

Though, notice that I *am* including the `.js` file extension... which you *can*
do in Encore, but it's not required. More on that later - but this *was* on purpose.

## Importing as a Module

So... does my browser support the `import` statement? Let's find out! Refresh.
Booo:

> Cannot use import statement outside a module

Ok, not a "code red" kind of boo, more like a "code orange". Head back to
`base.html.twig`. When you hear the word "module", it's referring to files that
leverage `export` and `import`. And if you want your JavaScript to be able to *use*
these, you need to load the original *file* "as a module". It's a simple change.
Copy the `asset()` function and *now* say `<script type="module">`. Then, instead
of `src`, inside, we're going to write some JavaScript to `import` our `app.js` file.

[[[ code('25822ef79c') ]]]

This may look nutty at first, but... we're simply importing the path to our
`app.js` file. By doing this, `app.js` will execute *exactly* like it did before...
but as a "module"... which just means that `import` and `export` statements
"should" work.

Do they? They do! OMG, our browser supports the `import` statement! 

## Importing 3rd Party Package URLs

We can *even* import third-party packages. To find one, I'm going to use my favorite
CDN: "jsDelivr". We'll be using this quite a bit throughout the tutorial. But you
don't need to *use* jsDelivr's CDN in your final code. It's a mirror of *every* NPM
package... and so it's a convenient place to find what we need.

Search for the popular "lodash" package. When we select it, it shows us
a `<script>` tag we could use. Click on "ESM", which is short for ECMAScript
modules. When you're coding with imports and exports, you want the *ESM* version
of a package: it's a version that properly "exports" modules.

*Now* check out that `script` tag:

```js
<script type="module">
import lodash from '[...]'
</script>
```

That looks *very* similar to the code we have over here! We won't use this exactly,
but I *am* going to copy the URL. Now go back to `app.js`. To use `lodash` we
can say `import _ from` and paste that full URL.

[[[ code('f3c52fc010') ]]]

Yes, importing from a full URL is *totally* allowed. Or we could download this file
locally: I'll talk more about that later. Below, let's say `_.camelCase()` to call
one of its methods.

Let's try it! Spin over, refresh, and... look at that!. There's *no* build system
here: we're just playing with files inside the `public/` directory. And yet, we're
writing modern JavaScript, importing and exporting modules *and* using a third-party
NPM package. That's pretty amazing.

## What Features are Missing?

*However*, there *are* two remaining problems. First, importing packages using the
full URL is annoying. I want to be able to say `import from 'lodash'` The *second*
problem is asset versioning. To have a performant system, we need the final files
downloaded by the browser to have version hashes in their filenames, like
`app.1234abcd.js`. We need this so that we can instruct browsers to perform long-term
caching. And we *can't* get this by creating & serving files directly from `public/`.

These are *precisely* the two things that Symfony's new AssetMapper component will
help us solve. But I wanted to start with raw JavaScript so that we could see how...
most of what we're doing is *not* solved by Symfony or AssetMapper or AI:
it's solved by your browser and the modern web.

Ok, let's delete these two files so I don't get confused... and also remove the
`import` inside of `base.html.twig`. Don't worry! We'll see all of that code
in a different way soon.

Next: Let's install AssetMapper and get it rocking.
