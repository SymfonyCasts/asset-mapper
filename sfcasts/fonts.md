# Adding Fonts

Another common CSS need is a custom *font*. My favorite source for fonts is
https://fontsource.org where you can search through a *huge* number of fonts that
have various open source licenses.

For example, one popular font is "Inter". Here, you can download the file, and it
gives some instal instructions, which are interesting: it uses the
font as an `npm` package.

We're not using `npm`, but we *can* use `npm` packages: and we know how.

Head over to jsDelivr to find it. Notice that the package is called
`@fontsource-variable/inter`. I'm going to search for `@fontsource/inter`. And...
just like with Bootstrap, *there's* the CSS file! For the font nerds out there, if
you looked inside of this file, you would see that this is the 400 weight, and it's
the file you would normally use if you installed this via `npm` and imported it.

Copy that URL and paste it in the browser to see what it looks like.

## Variable Fonts?

Notice that, back on FontSource, they recommend using a package starting
with `@fontsource-variable`. Variable fonts are cool: instead of needing a different
font file for each font *weight* like 400 vs 800, a single variable font can contain
*all* the weights, while still keeping the file size reasonably small. FontSource
starting offering variable fonts quite recently.

Change the URL to use `@fontsource-variable`. *This* is what we actually want. Copy
it, head back over to `base.html.twig`, add `<link rel="stylesheet">`, and paste.

Thanks to this, we can go over to `app.css`, inside of the `body` tag, and say
`font-family: 'Inter Variable'`, adding `sans-serif` as a backup.

Let's check it! Watch this text closely. Boom! It updated thanks to that new font.

If you're wondering why I didn't just search for the `@fontsource-variable`
package on jsDelivr originally, fair question: that's what I would *normally* do.
jsDelivr *is* a mirror of *every* NPM package. *However*, due to a bug in the
API of npmjs.com, right now, these new "variable" packages can't be found in the
search. The bug has apparently been fixed - so hopefully the issue will melt away
soon.

The point is, jsDelivr *does* have the package we need, we just can't find it via
the search. It's kind of annoying, but should be temporary.

Next: Let's make our CSS a bit fancier by introducing Tailwind. That's going to be
*especially* interesting because Tailwind requires a build step.
