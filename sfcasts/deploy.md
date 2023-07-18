# Deploying to Platform.sh

I have a *wild* idea. Let's deploy this site *for real*.

## AssetMapper Deploy Requirements

You can deploy your code however you wan...t using any service or web server you
want. It doesn't matter with AssetMapper. The only requirement is that your web server
supports HTTP/2 so that our assets - the JavaScript and CSS files - can be downloaded
in *parallel* super fast. HTTP/2 is why it's not terribly important that our files
*aren't* being combined to minimize requests.

All web servers - nginx, Caddy, whatever - *do* support HTTP/2. *Or* you could add
Cloudflare or a similar service in front of your site which gives you this for
*free*... along with some other benefits.

## platform.sh Config File Setup

Anyway, we're going to deploy with Platform.sh, which is a Platform as a Service...
which basically means you can deploy *simply* by creating a few config files. And
this first section is *all about* getting that set up. Once we're done, we'll talk
about some specifics of deploying with AssetMapper.

*So*, let's get started! We're going to do most of this in the command line via our
Symfony binary. Start by running:

```terminal
symfony project:init
```

This will Bootstrap a few platform.sh files, which you can see right here.
`.platform.app.yaml` contains instructions for how to deploy - like which commands
to run, what version of PHP to use, web server configuration and more. `services.yaml`
is where you setup which services you need - like databases, queues, etc - and
`routes.yaml` sets up your domains, and it's a bit less important. Oh, and you
can also add any custom `php.ini` config with this file.

I've been committing my progress along the way. So when I run

```terminal
git status
```

it just shows these new files. Let's commit these and... great!

## Registering the platform.sh Project

Now that we have those local files, we need to tell the platform.sh service *itself*
that we want to create a new project. Do that with:

```terminal
symfony cloud:project:create
```

In my case, I already have some projects under Platform.sh.... which means I already
have an organization... and it already has my credit card. If you're doing this for
the first time, you'll have a few extra steps. 

Give your project title, select a region  -I'm using "eu-5" - and then this asks
you which branch will be your production environment. I'm using the default `main`
branch.

Next, it asks us if we want to set "Mixed Vinyl" as the *remote* for this repository.
This is kinda cool because it exposes *how* platform.sh works. To deploy with
platform.sh, you actually push your git repository to a remote repository on
their services. They see this, take the code, and deploy!

Anyway, I'm going to say "no" - but you can say "yes". Because I'm saying no, you'll
see me do this manually in a minute - and I'll explain more about it.

*Finally*, it asks us to confirm the pricing. This $12 USD per month is the developer
rate that you can pay to play around with stuff. It *will* be a more expensive when
you decide to deploy your site to production for real. I love platform.sh because
of how easy it makes my life, but they *are* cheaper options.

This will take a minute or two to set everything up behind the scenes. When it
finishes... ding! We get some info, including our new Project ID

Side Note: There *is* also a web interface on Platform.sh, so not *everything* needs
to be done via the command line.

## Linking the Local Code to the Remote Project

Copy the Project ID. At this point, we have some local config files - like
`.platform.app.yaml` - *and* we created a new "project" on platform.sh. But the
two aren't linked together yet: our local code doesn't know that it "belongs"
to this project up on platform.sh.

To link them, run

```terminal
symfony project:set-remote
```

and paste the Project ID.

## Our First Deploy

Done! Ready to deploy? Do it with:

```terminal
symfony deploy
```

We're currently on the "main" branch, so it's pushing to, basically our "production"
machine, which is  called an *environment*. One of the coolest parts of platform.sh
is that you will deploy your `main` branch to the "production" environment, but
you can also deploy other git branches to other platform.sh" environments, which
you can think of as other platform.sh servers.

Anyway, as I mentioned, behind the scenes, this command is just a shortcut to
`git push` or branch up to a git remote on the platform.sh servers. And doing
that kicks off the deploy!

And we see a lot of the deploy details, including a warning about using an old version
of Composer. We'll fix that in a minute. Down here, you can see that it's runninh
`symfony composer install`, and doing some other steps: all the basic stuff you need
to deploy any Symfony app.

At the bottom, it actually gives us an SSL certificate, and if we keep scrolling...
oooh, we have a message about a database error! Ignore that for now because...
when it finishes, it gives us a URL!

Because we haven't configured any domains for the site, it gives us a nice,
*temporary* URL. Copy that, spin over and... our site is *alive*! It doesn't have
any styling... since we haven't talked about the AssetMapper, but it at least
kinda works!

But how? How did it know to run `composer install` and those other things? What
about that Composer warning and the database error? Let's dive into all of that
stuff next.
