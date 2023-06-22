# A World without Build Systems?

Whoa, hey! Welcome to my frontend laboratory where we're going to do something that I *honestly* thought
I would *never* do again. Something bold! Something... maybe just a little crazy. We're going to write a
*modern* frontend with *zero* build system.

## How we got here

Back-story time! 7 years ago, at a conference, I spoke about how modern JavaScript *requires* a build system.
I was shouting to the world that we needed to transition from creating JavaScript and CSS files in a "public"
directory towards *building* them with a system, like Webpack or Vite.

These build systems were created because browsers didn't support modern features that we wanted to use.
I'm talking about the `import` statement, `const`, the class syntax, and so on. If you tried to run
this kind of JavaScript in a browser, it would bomb out.

So, the build system would *transpile& (that's a fancy word for "convert") that *new* looking JavaScript to *old*
looking JavaScript so it could run in the browser. It would also combine JavaScript and CSS files so we would
have less files and requests, it could create versioned filenames, process TypeScript and JavaScript, Sass,
and *much* more.

These systems are *incredibly* powerful. But they also add complexity and can slow down coding. So I'm here,
7 years later to say that... we might *not* need those build systems anymore! In this tutorial, we're going
to write *all* the modern JavaScript that we know and love... but with *zero* build system. Just you and
the browser, like the Gods of the Internet intended it.

## Is this for Every Project

Now, I admit, doing this won't be the best option for *every* project. If you want to use TypeScript, or
you're using React, Vue or Next.js, you'll probably *still* want a build system... and you should probably
use *their* build system. Skipping the build system *also* means no automatic tree-shaking - if you know and
care about that - though I'll show you how that *can* still work.

For the most part, coding with and without a build system is identical, but I'll point out the small differences
along the way. And if you're wondering about things like Sass preprocessors, or Tailwind, you can *totally* use
those and I'll show you how.

## Project Setup

Okay, let's get to work! First, we need to set up our project. Coding without a build system is a blast, so you should *absolutely* download the course code from this page and code along with me. After you unzip that file, you'll have a start directory that's the same code that you see here.  This is the same project we've been using in our Symphony 6 series. Pop open this `README.md` file. As usual, this has all of the setup instructions that you need. I've done most of them already. The *last* step is to find your terminal, move into the project, and run

```terminal
symphony serve -d
```

using Symphony binary to start the built-in web server. I'll hold "command", click and... *hello* Mixed Vinyl. But *wow* is this thing weird and ugly looking.

As I mentioned, this is a Symphony 6.3 project. It has Doctrine in it, but there's nothing particularly special about it, and right now, it has literally *zero* CSS and JavaScript. There's no `/assets` directory and nothing hiding inside of the `/public` directory.

The first thing I want us to talk about is the fact that your browser can handle a lot more modern stuff than you realize (at least that was true for me). Let's see what all the hype is about by taking our browser for a modern JavaScript test drive. That's *next*.
