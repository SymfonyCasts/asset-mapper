# Page-Specific CSS & JS

Head over to `/admin`. Surprise! We *do* have an admin section on our site. Well...
*sort of*. It's only a big rectangle, but it represents a make-believe admin.
Why? Well, suppose we have some CSS and JS that are *only* needed here. If we write
that in the normal way and in the normal files, that code is going to be downloaded
everywhere, including when someone goes to the frontend of our site. That, at the
very least, is wasteful. A better way is to *only* download the admin CSS and JS
when you visit the admin area!

My favorite way to do this is with lazy Stimulus controllers, and we've already
talked about those. But another option is to create an extra set of CSS and JavaScript
that are explicitly loaded *only* on these pages. Let's see how to do that with
AssetMapper.

If we were using Webpack Encore, we'd open the `webpack.config.js` file and add a
second *entry*. That would result in a new CSS and JavaScript file. In AssetMapper,
we can do something really similar.

## Creating a new CSS File

Let's start with CSS, which is pretty darn simple. In the `assets/styles/` directory,
create an `admin.css` file and, to see if things are working, add `.admin-wrapper`
with some X-Y padding. 

[[[ code('bbe80d9cf4') ]]]

That'll add a little space right here. *Then*, go into the template
for this page - `templates/admin/dashboard.html.twig` - and,
right here, add that class: `class="admin-wrapper"`.

[[[ code('b7696d4790') ]]]

At this point, the new `admin.css` file *is* technically available publicly... because
it's in the `assets/` directory. But, we're not *using* it yet. To do that, we need
a link tag.

There's nothing special about this. Say `{% block stylesheets %}` and `{% endblock
%}` to override the block from the parent template. Then call `{{ parent() }}` to
include the normal stuff and, down here, add `<link rel="stylesheet"` pointing to
`asset('styles/admin.css')`. And... let me fix my typo up here. *That's* what
we want.

[[[ code('79f2e96dbd') ]]]

Back on the site... yup! The CSS *is* being applied: we've got extra padding.
Refreshingly simple.

## Creating a Page-Specific JavaScript File

But... what about JavaScript? Once again, we'll start a lot like Encore. Create
a new file... maybe next to `app.js` called `admin.js`. Add
`console.log('admin.js file')` so we can see if it's loading.

[[[ code('4fb4e62029') ]]]

Like with the CSS file, this file *is* now publicly available... but nothing is
*loading* it. Remember: the `app.js` file is loaded thanks to this
`<script type="module">` line down here that imports `app`. We *automatically* get
this, over in `base.html.twig`, via the `importmap()` Twig function.

So... is there a way to tell this function to *also* import our `admin.js` file?
Actually, no! Why? Mostly because... it's just so easy to add ourselves!

Watch: back in `dashboard.html.twig`, say `{% block javascripts %}`, `{% endblock%}`,
then `{{ parent() }}`. Below that, add a `<script>` tag with `type="module"`. Now
we're going to code as if we're in a JavaScript file. Say `import` and then the *path*
to the JavaScript file. Effectively, we want something like - `/assets/admin.js`.
But, of course, to get the real path we use the `asset()` function and pass the
logical path: `admin.js`.

[[[ code('369c9b0192') ]]]

That's it! Let's try this thing! Refresh and check the console. Got it! Our `admin.js`
file *is* being loaded! If you check out the page source... down here... *yep*. You
can see `<script type="module">` from the `importmap()` function where it says
`import 'app'`. And, after, we import `admin.js` via its path.

The original is just `import 'app'`... because we rely on the
`importmap` to *map* that to its URL. That's *nice*... but it's not actually
necessary. Putting the path right here works fine too. That's what we're doing
for simplicity.

One of the things we saw in this chapter is that *everything*
in the `assets/` directory is exposed publicly... which is the whole point of
AssetMapper! But sometimes you may have a few files that you want to put in that
directory, but keep private. Let's check into AssetMapper's exclude feature and
other config options next.
