# Adding Fonts

Coming soon...

Okay, there's one last thing I want to talk about. What if we need a custom *font*
for our site? My favorite resource for fonts is actually "fontsource.org". This is an
*awesome* site. They have a *ton* of free fonts that you can use, so you don't have
to rely on the Google CDN if you don't want to. For example, here's a font called
"Inter". If you look over here, you can download the file, and it gives you
installation instructions. Notice that, in the installation instructions, it's
showing you an NPM package. We know what to do with NPM packages.

Head over to jsDelivr and search for that. Notice that the package is called
"@fontsource-variable/inter". Let's actually try to search for "@fontsource/inter"
first. And just like with Bootstrap, *there's* our CSS file. For all the font nerds
out there, if you look inside of this file, you would see that this is actually the
400 weight. This is the file you would normally get if you installed it and used it
like we would with Encore. Copy that URL, paste it in your browser, and you can see
what it looks like. Notice that, over here, the original package is called
"@fontsource-variable". Variable fonts are a new thing Fontsource is doing, but I
won't go into that here. If we change this to "fontsource-variable", that's the one
we actually want. Copy this, head back over, go to `base.html.twig`, add `<link
rel="stylesheet">`, and paste that. Thanks to this, we can go over to `app.css`,
inside of our `body` tag, and say `font-family: 'Inter Variable'`, adding
`sans-serif` as a backup. Now, over here, watch this text closely. Boom! It shifted
*slightly* thanks to that new font. If you're wondering why we didn't just search for
this package directly in jsDelivr, that's *normally* what I do, because this is a
mirror of every single NPM package. *However*, due to a bug in NPM right now, these
relatively new variable packages aren't found. You can see that it finds this in its
search, but it doesn't find *this*. Because of this bug, the variable packages aren't
in their Algolia real time search at the moment. That will *hopefully* be fixed soon,
but the point is, jsDelivr *does* have it, you just can't find it via the search.
That's kind of annoying, and I wanted you to be aware of it.

Next: Let's make our CSS a little more complicated by introducing Tailwind. That's
going to be interesting because Tailwind requires a build step.
