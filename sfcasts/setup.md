# A World without Build Systems?

Whoa, hey! Welcome to my frontend laboratory where we're going to do something that
I *honestly* thought I would *never* do again. Something bold! Something... maybe
just a bit crazy. We're going to write a *modern* frontend with *zero* build
system.

## How we got here

Back-story time! 7 years ago I was talking about how modern JavaScript
*requires* a build system. I was shouting to the world that we needed to transition
from creating JavaScript and CSS files in a "public" directory towards *building*
them with a system, like Webpack or Vite.

These build systems were created because browsers didn't support modern features
that  we wanted to use. I'm talking about the `import` statement, `const`, the class
syntax, and so on. If you tried to run this kind of JavaScript in a browser, you
would have been greeted with sad error messages.

So, the build system would *transpile* (that's a fancy word for "convert") that *new*
looking JavaScript to *old* looking JavaScript, so it could run in the browser. It
would also combine JavaScript and CSS files, so we would have fewer requests,
it could create versioned filenames, process TypeScript and JSX, Sass, and
*much* more.

These systems are *incredibly* powerful. But they also add complexity and can slow
down coding. So I'm here, 7 years later to say that... we might *not* need those
build systems anymore! In this tutorial, we're going to write *all* the modern
JavaScript that we know and love... but with *zero* build system, and no Node. Just
you and the browser: the way the Gods of the Internet intended it.

## Is this for Every Project?

Now, I admit, doing this won't be the best option for *every* project. If you want to
use TypeScript, or you're using React, Vue or Next.js, you'll probably *still* want a
build system... and you should probably use *their* build system. Skipping a build
system *also* means no automatic tree-shaking - if you know and care about that -
though we'll learn how that *can* still work.

For the most part, coding with and without a build system is identical, but I'll
point out the small differences along the way. And if you're wondering about things
like Sass preprocessors, or Tailwind, you can *totally* use those and we'll see
how. The final site is also going to be *as* performant and fast as one built
with a build system.

## Project Setup

Okay, let's get to work! Coding without a build system is a *joy*: no node *or*
batteries required. So you should *absolutely* download the course code from this
page and code along with me. After you unzip that file, you'll have a `start/`
directory with the same code that you see here. Pop open this `README.md` file. As
usual, it holds all the setup details you'll need. I've done most of them already.
The *last* step is to find your terminal, move into the project, and run:

```terminal
symfony serve -d
```

to use the `symfony` binary to start the built-in web server. I'll hold "command",
click and... *hello* Mixed Vinyl! But *wow* is this thing weird and ugly-looking.

This is a Symfony 6.3 project - the same project we've built in the Symfony series.
It has Doctrine installed... but there's nothing particularly special about it, and
right now, it has literally *zero* CSS and JavaScript. There's no `assets/` directory
and nothing hiding inside the `public/` directory.

The first thing I want to explore is the reality that our browser can handle
*more* modern stuff than we might realize... certainly much more than *I* realized
a few months ago. Let's see what all the hype is about by taking our browser for
a modern JavaScript test drive next.
