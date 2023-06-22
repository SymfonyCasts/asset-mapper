# A World without Build Systems?

Whoa, hey! Welcome to the tutorial! We're going to do something that I *honestly* thought I would *never* do again. We're going to write a *modern* frontend with *zero* build system.

About seven years ago, we transitioned from just creating JavaScript or CSS files into wrapping them in a build system (like Webpack). We had things like Babel and, these days, there's newer tools like Vitae. These build systems were created because browsers didn't support modern features that we wanted to use. I'm talking about the `import` statement, `const`, the class syntax, and so on. The build system would actually transpile (that's a fancy word for "convert") that *new* looking JavaScript to *old* looking JavaScript so it could be run in our browser. The build system would also do things like combine JavaScript and CSS files together so we would have less files. It could also version the file names, process TypeScript and JavaScript, Sass, and *much* more.

Build systems are *incredibly* powerful, but you should also know that they can be complex and slow things down. For various reasons, which we'll talk more about in this series, *most* of what I just said isn't really needed anymore. We're going to use *all* of the stuff we love, but with *zero* build system.

Doing this won't be the best option for every single project, however. If you want to use TypeScript in your project, for example, *or* you're building a single page application with React, Vue, or Next.js, you'll probably still want a build system to handle that. But for many projects, this new way of doing things is just *awesome*.

There are pros and cons between the new system and a build system, and as we encounter those throughout this tutorial, I'll let you know. If you're already wondering about things like Sass, preprocessors, or Tailwind, you can *totally* use those with this system.

Okay, let's get to work! First, we need to set up our project. Coding without a build system is a blast, so you should *absolutely* download the course code from this page and code along with me. After you unzip that file, you'll have a start directory that's the same code that you see here.  This is the same project we've been using in our Symphony 6 series. Pop open this `README.md` file. As usual, this has all of the setup instructions that you need. I've done most of them already. The *last* step is to find your terminal, move into the project, and run

```terminal
symphony serve -d
```

using Symphony binary to start the built-in web server. I'll hold "command", click and... *hello* Mixed Vinyl. But *wow* is this thing weird and ugly looking.

As I mentioned, this is a Symphony 6.3 project. It has Doctrine in it, but there's nothing particularly special about it, and right now, it has literally *zero* CSS and JavaScript. There's no `/assets` directory and nothing hiding inside of the `/public` directory.

The first thing I want us to talk about is the fact that your browser can handle a lot more modern stuff than you realize (at least that was true for me). Let's see what all the hype is about by taking our browser for a modern JavaScript test drive. That's *next*.
