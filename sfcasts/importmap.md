# JavaScript & importmap

Remove the `<img>` tag, so we can see our normal page. Don't worry about
our little penguin guy: we still have him up here in the logo.

When we refresh the page, notice that we *do* have a `console.log()` message...
which says it's coming from `assets/app.js`. If we head over to `assets/app.js`...
yup! There it is!

## How assets/app.js is Loaded

We know that we *can* write modern ES6 code in here, as well as import
other files. We're going to do *all* of that. But first: *How* and *why* is *this*
file even being executed? Our CSS is being loaded thanks to this nice, boring
`<link>` tag. We *don't* see a `<script>` tag for `app.js`... but we *do* see this
`importmap()` function. And *that's* the key.

Back over on the site, View the page source. Down here... *this* is what `importmap`
adds. We're going to talk about each part, but the most important thing right
now is at the bottom:

```js
<script type="module">import 'app';</script>
```

Earlier, when we created an `app.js` file inside the `public/` directory, this
is almost *exactly* the code we wrote to load it. We used `import` and then the *path*
to that file. But... this time, it just says `app`. Shouldn't it say something like
`/assets/app.12345.js`"? How does it know that `app` refers to the final version
of this file? *This* is where the `importmap` part, up here, shines.

## The Wonderful importmap

This section is generated from an `importmap.php` file inside our project. The
file isn't super-interesting *yet*: it'll be more useful soon when we talk about
third party JavaScript. But it *does* have this `app` key that points to
our `assets/app.js` file using its logical path.

Thanks to that, this `<script type="importmap">` dumps onto the page. When
you import something that doesn't start with a ".", "/", or "../", that's called
a *bare* import. We usually see this for third-party libraries. In the browser
environment, when it sees a "bare import", your browser looks for an `importmap` on
the page to find a matching entry. Our browser sees `import 'app'`, finds
this key here, and *that's* the path it downloads. It effectively copies
this path here and pastes it down there. *That's* why our `app.js` file is being
executed: it's team work between the `importmap` and the extra
`<script type="module">` that bootstraps our app!

The *greatest* thing about `importmap` is that it's *not* a Symfony thing: it's just
an *internet* thing. It's how your browser works. We *do* have this `importmap.php`
file, which *is* a Symfony thing. But once this is on the page, your *browser* is
the star.

## The importmap shim + Older Browsers

And `importmap` works in... *most* browsers. If you go to "caniuse.com" and search
for "importmap"... it currently works in about 81% of browsers. That *would* be
a huge problem, except that the `importmap()` function also dumps a *shim*. You can
see that here. Thanks to this, if a browser *doesn't* support `importmap`,
this *adds* that functionality. So, it's *just* going to work.

## Importing Relative JavaScript Files

Head into `app.js`: let's write some modern code. In `assets/`, first create
a new directory called `lib/`. And inside *that*, a new file called `vinyl.js`.
You can organize things however you want, and this is *one* example of isolating
some code into its own file.

I'll paste in the same class we had earlier. Back over in `app.js`,
import that: `import Vinyl` and I can hit "tab" to autocomplete the
`from './lib/vinyl'` part. Instantiate this using the same code as before... and
then `console.log(mix.describe())`.

## Using .js when Importing

I love it! We're coding like normal and using `./` to import. But when we go over
and refresh... it *doesn't* work. Check out the 404: `/assets/lib/vinyl`
coming from `app.js`.

So... what's going on here? We'll talk more later about debugging, but here's a hint: if
you ever notice that your browser is trying to download a path that *doesn't* include
the "version" part in the filename, *something is wrong with your path*... and you
should check for typos.

*Our* problem is that we need to add the `.js`. It turns out that leaving the `.js`
off is a Node thing... and it works if you're programming in Node. But in true
JavaScript environments, like in your browser, you *do* need to include it.

If we refresh now... that was it! It was really my editor's fault that the `.js`
was missing when it autocompleted it. Fortunately, we can fix that! Go into your
PhpStorm settings and search for "use file extension". Under "Code Style" and
"JavaScript", change "Use file extension" to "Always".

This time... if we say `import Vinyl` and hit "tab", nice! We get the `.js`.

## Automatic Importmap Entries

But the fun doesn't stop: there's something interesting happening behind the scenes.
Click into this `console.log()`... just as an easy way to see the source of the
final `app.js` file.

Yup, its contents look exactly like the original file, including the
`import from './lib/vinyl.js'`. There's just one problem: that's *not* the final
filename for `vinyl.js`!

Pop over to the Network tools, select "JS", and search for "vinyl". *All* files served
by AssetMapper have a versioned part of their file name, and we see that for `vinyl.js`.
But then... how the heck does our browser read `./lib/vinyl.js` and *know* that it
should download this long filename?

The *answer*, if you view the page source, is... dramatic drumroll... the `importmap`.
And I *love* this. The `importmap` is constructed from two sources. The *first* source
is obvious: `importmap.php`. And we'll add more entries to it soon. The *second*
source is more subtle. Whenever our JavaScript imports another JavaScript file using
a relative path, that imported file is automatically added.

This is powerful. It means that our final code can look like it originally does:
`./lib/vinyl.js`. But thanks to the `importmap`, our browser will smartly download
the real file with the long version part in the name. This is really an internal
detail, but it's cool to see how it works.

Okay, we've talked about `importmaps` a little... but we haven't seen its
*biggest* superpower: using third party packages. Let's explore that *next*.
