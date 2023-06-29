# 3rd Party CSS

We talked about adding CSS to our site, but what about *third-party* CSS like Bootstrap? With a build system like Encore, we have a `package.json` file and we can run something like

```terminal
yarn add bootstrap
```

or

```terminal
npm add bootstrap
```

In AssetMapper, because there's no Node, we don't have an easy system for grabbing CSS files. We can still get those files, but it's not quite as smooth as it is with a build system. When we *don't* have Node packages, I like to rely on jsDelivr - a CDN for all NPM packages. Even if you don't use it, it's a really nice way to find and download packages if and when you need to. Later, we'll see that there isn't really an *easy* way to automatically grab JavaScript files in AssetMapper, but I digress.

I'll search for "Bootstrap" here and... there it is. A lot of times, you'll find the CSS file you're looking for right up here, just like this. I'll hit "Copy HTML + SRI" to copy this file. If you don't find the file you're looking for up here, you may need to look a little deeper. You can click the "Files" tab to see all of them. You can go through `/dist/css` and find the CSS file or *files* you want.

Okay, we know that CSS is boring, so we're going to go back over to our `stylesheets` tag and, above ours, we'll paste the *new* stylesheet. If we wanted to *avoid* using the CDN, we *could* download this file directly into our project. And because there's no package system like NPM, I would probably create an `/assets/vendor` directory and put the file inside of that. Then I would actually commit that `/assets/vendor` directory to Git so that it's always there. That would act like version control for those `/vendor` assets. You'll actually see me do this later with a JavaScript file. The point is, if you ever need to manually find and add CSS files to your project, you know where to look.

Now let's see if this is working! Scroll down to the middle of the page and add:

`<button class="btn btn-primary">Primary Button</button>`

This is just a basic button using the Bootstrap style syntax, which is what we just installed. If we head over to our site and refresh... it *works*. Cool! But what if I want to *modify* Bootstrap?

Bootstrap itself is built with Sass. If you wanted to, you could actually point your build system at the Sass file *instead* of the CSS file. By doing that, you can override certain variables inside of Bootstrap. You could make the `primary` color a slightly different color, for example.

Now, there are two important things to note here. First, you can *absolutely* use Sass with AssetMapper. There are details in the documentation about how to do that. In a moment, we're also going to add Tailwind CSS to our site. That will have a very similar workflow, because Tailwind *also* requires a build step. If you wanted to use Sass with Bootstrap, there's actually a composer package that you can use, which contains CSS *and* Sass files. Second, depending on what you want to do, you may not need to use Sass to customize things at all. That's because Bootstrap also exposes CSS variables. We can see this down in the "Customize" "CSS Variables" section. This is just a browser feature that allows you to *set* variables inside of CSS, and then reference those variables.

For example, over in `app.css`... it doesn't matter where, but we usually put this on top... add a `:root` pseudo selector. It applies to the *root* of your page. Here, we're going to override a CSS variable called `--bs-border-radius`" that Bootstrap provides by default. Let's set this to `1rem`.  That should make the borders noticeably larger. Back over here, just by making that one change... boom! Our border radiuses are now larger across the site. That's one of the variables you would find in the Bootstrap documentation. That's *super* easy, but it isn't always that simple.

Let's say that we want to override this color. You might think you could do that by searching for "primary"... up here... and overriding something like `--bs-primary` (the primary color). That's *sort of* correct. If you inspect our button, this color here *is* the actual background color. But watch... if we try to override this here by changing this to a slightly lighter color, head back and try that... it doesn't do *anything*. As I mentioned, some things are really easy to override and some things are a little more difficult.

Copy the CDN URL here, pop that into your browser, and let's take off the `.min` so we can actually see what's going on. You can see that it's setting all of those nice CSS variables. If we look for "btn-primary", I won't get too specific here, but what's happening is, inside of `.btn-primary`, it's setting these CSS variables to these hard-coded colors. What it's *not* doing is referencing *our* CSS variable. That means we can't really override it. So... what do we do? In this case, we need to basically re-declare it inside our class. That's not as intimidating as it sounds because we can *reuse* CSS variables, but it's still a little bit of work. Check it out!

Spin back over to `app.css` and paste in the styling for `.btn-primary`. This is overriding the variables that are set over here to a different color. In fact, we're still using our `bs-primary`. You can see that up here, and we can reference that `bs-primary` in as many spots as we want. We're also customizing the hover color here. We're overrding this in the same way we override any CSS, but we can still utilize bootstrap variables. If we head back to our browser and check this... that *does* change the color. Sweet! *So*, while this *is* an option, if you need to customize *a lot* of stuff, Sass is probably still the better choice. I get this question sometimes and so want people to be aware of the possibilities inside Bootstrap without having to build with Sass.

Okay, there's one last thing I want to talk about. What if we need a custom *font* for our site? My favorite resource for fonts is actually "fontsource.org". This is an *awesome* site. They have a *ton* of free fonts that you can use, so you don't have to rely on the Google CDN if you don't want to. For example, here's a font called "Inter". If you look over here, you can download the file, and it gives you installation instructions. Notice that, in the installation instructions, it's showing you an NPM package. We know what to do with NPM packages.

Head over to jsDelivr and search for that. Notice that the package is called "@fontsource-variable/inter". Let's actually try to search for "@fontsource/inter" first. And just like with Bootstrap, *there's* our CSS file. For all the font nerds out there, if you look inside of this file, you would see that this is actually the 400 weight. This is the file you would normally get if you installed it and used it like we would with Encore. Copy that URL, paste it in your browser, and you can see what it looks like. Notice that, over here, the original package is called "@fontsource-variable". Variable fonts are a new thing Fontsource is doing, but I won't go into that here. If we change this to "fontsource-variable", that's the one we actually want. Copy this, head back over, go to `base.html.twig`, add `<link rel="stylesheet">`, and paste that. Thanks to this, we can go over to `app.css`, inside of our `body` tag, and say `font-family: 'Inter Variable'`, adding `sans-serif` as a backup. Now, over here, watch this text closely. Boom! It shifted *slightly* thanks to that new font. If you're wondering why we didn't just search for this package directly in jsDelivr, that's *normally* what I do, because this is a mirror of every single NPM package. *However*, due to a bug in NPM right now, these relatively new variable packages aren't found. You can see that it finds this in its search, but it doesn't find *this*. Because of this bug, the variable packages aren't in their Algolia real time search at the moment. That will *hopefully* be fixed soon, but the point is, jsDelivr *does* have it, you just can't find it via the search. That's kind of annoying, and I wanted you to be aware of it.

Next: Let's make our CSS a little more complicated by introducing Tailwind. That's going to be interesting because Tailwind requires a build step.
