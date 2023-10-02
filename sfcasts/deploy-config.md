# Configuring the Platform.sh Deploy

We just deployed a semi-working version of our site to platform.sh! All it took
was one command to bootstrap a few config files and another to create the
project inside of platform.sh.

But... we had some errors and warnings along the way. On top of the output,
we see a warning about using an old composer version. In a minute, we'll see
how and *why* composer is used during the deploy. But when it is, for *some*
reason, it's currently using an old version of Composer.

Fortunately, it warns us *and* tells us how to fix it.

## .platform.app.yaml and How Deploying Works

Copy this `dependencies` line. Then, open `.platform.app.yaml`. *This* is the
main deploy file: almost every deploy tweak you'll make will be made here.

There are two steps to the deploy process. The first is the `build` step where it's
*building* your code: you can think of this as the step that prepares all the
physical *files* that your project needs. Once the `build` step is done, it spins
up a container, puts the files inside and then runs the second and final part
of the process: the `deploy` step. This is where you can run some final commands.

See these `symfony-build` and `symfony-deploy` scripts? These are pre-made scripts
that contain *most* of what your app needs to deploy. If you downloaded them
and opened them up, you'd see things like running `composer install`, warming
up the cache and running database migrations. We can add custom stuff above or below.

The config has `mounts` - for directories that you want to keep persistent between
deploys - PHP extensions, your PHP version, and quite a bit more.

## Using a Newer Composer Version

Anywhere inside, paste the `dependencies` line... and make sure it's not
indented.

[[[ code('b1a9891dc6') ]]]

And just like that, we're using Composer version 2.

## Setting up the Database Serve

The second error that we had, down near the bottom, was,

> could not find driver

This come from when the `symfony-deploy` script tries to run our database migrations.
Locally, we're using Postgres. You can see that in `docker-compose.yml`.
Do we have a database up on Platform.sh yet? The answer is... actually *yes*.

In addition to the main deploy file - `.platform.app.yaml` - we have a `/.platform`
directory with a few other files. The most important is `services.yaml`.
This is where we define *services* like databases, Redis, Elasticsearch and others.
When we initialized the project, it noticed that we're using Postgres and added a
database for us!

[[[ code('e2d3109128') ]]]

The error we're getting isn't because it can't find the database: it's because
our PHP install is missing the `PDO_PGSQL` driver! And thanks to `.platform.app.yaml`,
adding that is easy.

Find `extensions`, and add `pdo_pgsql`.

[[[ code('a31d9791f2') ]]]

Ok, ready to re-deploy? Remember: deploying happens via a *push*, so we need to
commit these files. Run `git commit -m` with an inspirational message.

```terminal-silent
git commit -m "tweaking deploy script"
```

Now run:

```terminal skip-ci
symfony deploy
```

This runs the same steps as our first deploy, which we now understand include a
`build` step then a `deploy` step. It'll take a minute or two, but should be a
*bit* faster because it doesn't need to re-provision the SSL certificate.

## Viewing the Logs

At the very end... the migration command *still* failed, but with a *different*
error:

> Connection refused

Ah, ignore that for a minute. Instead, go back to the site and refresh. The
homepage still works. But... that's because our homepage *doesn't* use the database.
If you hit "Browse Mixes"... 500 error! That 500 error is *probably* due to a
database connection problem. But let's pretend that we have *no* idea what's
causing this. How could we figure that out? This is where the

```terminal skip-ci
symfony logs
```

command comes in handy. This connects to whatever platform.sh "environment" -
or "server" - that our current git branch is connected to and sends back
log info. There are a bunch of choices - but hit `2` to go to the "app" log. This
represents the Symfony logs coming from our app. And... oooh. I see several:

> connection to server [..] failed: Connection refused

## Adding the Database "Relationship"

Let's think about this. We apparently *do* have Postgres set up, thanks to the
`services.yaml` file. But we never configured our app to *talk* to it. Remember,
in `.env`, we have a `DATABASE_URL` env var that's supposed to point our database.
We never configured that on our production site, so it's just using this default
value. And no surprise, that's not working.

How *can* we configure `DATABASE_URL` to point to... wherever this database server
is? The answer is... we... uh... don't? And it's pretty cool.

Platform.sh has this idea of *relationships*. You have a number of services in
`services.yaml`. But your app can't *talk* to these until you link them together
using what's called a "relationship".

Search `.platform.app.yaml` for "relationships". It's not here yet, so let's add
that. Each "relationship" has an internal name. It could technically be anything,
but, in practice, you should use `database`. We'll see the *significance* of that
in a moment. Set this to the word `database`, because that's the key we have here,
then `:` followed by the *type* of the service, which is `postgresql`.

[[[ code('d2e99c869e') ]]]

This syntax has always looked weird to me. The *important* thing is that the key
could be anything, like `banana`, but this `database` refers to this `database`
over in `services.yaml`, and `postgresql` refers to *this* `postgresql`.

But though the first `database` key *could* be anything, I used `database` on
purpose. Symfony does a really nice thing when it deploys via Platform.sh. It
sees this relationship, notices it's for a database, and then automatically
exposes an environment variable containing the connection info *to* that
database!

What's this environment variable called? Since we used the key "database",
it will be called `DATABASE_URL`. In other words, it's going to set this environment
variable *for* us. I'll prove it!

## SSHing onto the Container

One of my favorite things about Platform.sh is that you can SSH onto your container.
Watch:

```terminal skip-ci
symfony ssh
```

There we go. Once here, if you want to see *every* environment variable, run:

```terminal
printenv
```

Look at that! You *won't* see anything that starts with "database", but
we *should* after we deploy this next change. Type `exit`, run

```terminal
git status
```

and then

```terminal
git add -p
```

That's what we want! Commit with

```terminal
git commit -m "adding database relation"
```

And

```terminal skip-ci
symfony deploy
```

This time, it deploys *way* faster. Because we didn't change any *application* code,
platform.sh was smart enough to use our old app code, instead of doing all that
building again. We can see that:

> Reusing existing build for this tree ID

And hey! This time, we see that it `Successfully migrated`! Yea! it ran our migration
with zero problems. When we spin over and check the site... it *works*. It's
still missing all of our styling... but we'll fix that next. The important thing
is that the database *is* working.

You can see the difference that made if you run

```terminal skip-ci
symfony ssh
```

and

```terminal
printenv
```

This time, there are several `DATABASE_` variables, including the most important
`DATABASE_URL`.

Ok, the final missing piece from our deployed site is... all of its assets! Let's
see what's needed to deploy an AssetMapper site next.
