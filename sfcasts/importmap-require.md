# importmap:require - 3rd Party JS Libs

In our code, we can use `import` statements, relative imports, classes - everything we're normally used to. It's business as usual. But what about third party packages? As I mentioned earlier, you can actually import things via a full URL, like `import _ from`, and we'll paste in the same CDN URL that we used earlier. Then we can say something like `console.log(_.camelCase(mix.describe())`... and if we refresh and check the console... that *works*, but I *just* don't like it. I don't want to have to include this crazy URL everywhere. What happens if we upgrade and we have this in 10 different files? Yeah... not a great solution. Fortunately, there's a really nice way to solve this with `importmaps`.

If we were using a build system like Webpack, we would just say:

```terminal
yarn add lodash
```
or

```terminal
npm add lodash
```

And for JavaScript files, we can do the *same* thing. Over in the terminal, open a *new* terminal window, and run:

```terminal
./bin/console importmap:require
```

followed by the name of the NPM package we want, which is

```terminal
lodash
```

Done! It looks like it just added Lodash to `importmap.php` and tells us we can now use the package as usual. Now, all we have to do is say `import _ from 'lodash'` like we normally would, and everything works *fine*. How? When we ran the command, it made one *tiny* change. It added this here. So as cool as this is, it's not magic. It went to the JSDelivr CDN, found the latest version, and then added a new `lodash` key here set to that URL. If we head over and look at the page source, we have a new Lodash entry inside of our `importmap`! So, again, this is just a browser thing. When our browser sees `import _ from 'lodash'`, it looks inside of our `importmap` for `lodash`, and this is the URL it goes to. The point is, using third party packages is *really* simple with the importmap system.

One thing you'll notice is that I'm not getting autocompletion here. It says "Module not installed", and if I say "_ .*something*", it doesn't really work. It's autocompleting "camelCase", but only because I'm using that down here. This is something that I hope will be better supported in PagerStorm soon. There *is* a workaround for this, but it's manual. If you copy this package here, and go into `base.html.twig` and add a `<script>` tag for it (we won't actually *keep* it), when we hit "alt" + "enter", we have an option to "Download library". That's going to download it into our "External Libraries" down here - `/lodash`. We don't actually *need* this thing anymore, but removing it means, in `app.js`, it's still going to underline this as if it doesn't know what it is, when we *do*, in fact, have "_ .*whatever*" - `camelCase`, `JOIN`, `FROM`, `reduce` - etc. For example, `tail` is from Lodash, and if we select that, it will now autocomplete this *for* us. Hopefully that support improves soon, but in the meantime, there *is* a workaround for it. It could just be a little smoother.

Now, what about *updating* things inside of `importmap`, like new versions as they're released? There's a command for that! It's not *quite* as flexible as it could be yet, but by running

```terminal
./bin/console importmap:update
```

and that will go through all of your packages in here and make sure they're updated to the latest version. This is *already* the latest version. If I change this to... say `.19` and went back, it's going to move this back up to `.21`. Finally, if you don't want to rely on the CDN, you don't *have* to. In this `importmap:require lodash`, we can pass the `--download` option.

This *still* has the source URL for where it came from, but actually downloaded it into our `/assets/vendor` directory. This `downloaded_to` is pointing to the logical path to that. Thanks to *that*, if we go over and refresh the page and "View Page Source"... our `importmap` is now pointing to that local file instead, and we're no longer relying on the CDN. One obvious question you might have is: Do I commit this `vendor/lodash.js` file? The answer is... *yes*. That's the only way for us to version that file, so you *will* commit that to your repository.

And that's it for third party packages! Woohoo! Now, *sometimes*, instead of importing a whole package, we may only want to import a specific *file*. Let's talk about how we can do that *next*.
