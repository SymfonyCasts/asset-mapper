# Deploying to Platform.sh

I have a *wild* idea. Let's deploy this site *for real*. You can deploy your code however you want using any service or web server you want. It doesn't matter with AssetMapper. The only requirement is that your web server supports HTTP/2. We need that so our assets - our JavaScript and CSS files - can be downloaded in parallel super fast. That's why it's not terribly important that our files are being combined. All web servers are able to use HTTP/2, *or* you could just use Cloudflare in front of your site which gives you this for *free*, along with some other benefits. We're going to deploy with Platform.sh, and this first section is *all about* getting our project set up. In the next chapter, we'll talk about some specifics to deploying with AssetMapper.

*So*, let's get started! We're actually going to do most of this in the command line via our Symfony binary. Start by running:

```terminal
symfony project:init
```

This is going to Bootstrap a couple of files, which you can see right here. This one contains the instructions for how to deploy, this contains the services we need (like databases), the routes is a little less important, and *this* is a custom `php.ini` configuration. I've been committing along the way, so if I run

```terminal
git status
```

you can see that I just have a couple of new files there. I'll go ahead and commit those. *Perfect*.

Now that we have those local files created, in order to actually create a project on Platform.sh, we're going to say:

```terminal
symfony cloud:project:create
```

In my case, I already have some projects set up under Platform.sh. I already have an organization and it already has my credit card. In *your* case, if you're doing this for the first time, you may have a few extra steps along the way.

Let's give our project title, select your region (I'm using "eu-5"), and then this asks you which branch you'll be using as your production environment. I'm using the "main" branch, which is the default. Next, it asks us if we would like to set Mixed Vinyl as the *remote* for this repository. With Platform.sh, when you deploy, you're pushing to a git remote. This isn't really necessary, so we're going to say "no" to this. We'll deploy with the Symfony deploy command. Finally, it asks us to confirm the pricing. This $12 USD per month is the cheap developer rate you can pay for just playing around with stuff. It will be a little more expensive when you decide to deploy your site on production for real. This will take a minute or two to set everything up behind the scenes. When it finishes, it will print out some information like your Project ID. We're going to need this.

Side Note: You can also log on using this URL to Platform.sh. You can see details about your project there, so not *everything* needs to be done via the command line.

Go ahead and copy this Project ID. We have our local project file set up - that's the `.platform.app.yaml` type files - and we have our project set up in the cloud. Now we're going to link those two together with

```terminal
symfony project:set-remote
```

and then paste our Project ID. Finally, we can say:

```terminal
symfony deploy
```

We're on the "main" branch, so it's pushing to our production branch right now. And even though we used

```terminal
symfony deploy
```

you can see that this is the same stuff we get when we run

```terminal
git push
```

Behind the scenes, it's pushing to that Platform.sh remote, and then it starts doing our commits. Cool!

Right out of the box, we're getting a lot of the deploy details here. We *do* have a warning up here about using an old version of Composer, and we'll fix that in a second. But, down here, you can see it's running

```terminal
symfony composer install
```

and it's doing a few other optimizations - all the basic stuff you need for a Symfony application. At the bottom, it actually gives us a SSL certificate, and if we keep scrolling... oooh, we do have a message here about a database error. We'll talk about that in a minute as well. That isn't really surprising since we haven't done any customizations on our deploy script at all. When it finishes, it also gives us a URL. Since we're not actually on production yet, it gives us this cool *temporary* URL. Copy that, spin over and... our site is *alive*! It doesn't have any styling yet since we haven't talked about the AssetMapper, but we can at least *see* it. Super cool!

Now let's fix a few things. As I mentioned, on top, it uses an old version of Composer by default. I don't really know *why* it does that, but we can easily add a little configuration to have it use a *new* version of Composer. And it actually hints at how to do that! Copy this `dependencies` line here. Then, open `.platform.app.yaml`. There's a ton of documentation on this file, but this is your *build* file, and there are two main steps. There's the `build` step where it's *building* your code, and then there's the `deploy` step, which runs some final commands *after* it's been built and once it's on the final container. These `symfony-build` and `symfony-deploy` scripts are pre-made scripts that contain most of what you're going to need. You can then add some custom stuff on top of that. It also has some `mounts` for directories that you want to keep persistent between deploys, some PHP extensions, your PHP version, and so on. Anywhere inside of here, paste the `dependencies` line, make sure it's not indented, and just like that, we're using Composer version 2. Easy!

The second error that we had, down here near the bottom, was:

`could not find driver`

This is coming from when it runs our migrations, since it runs those automatically. We're using Postgres, and you can see that in our `docker-compose.yml` file. This is what I've been using locally while I've been developing. Do we have a database up on Platform.sh yet? The answer is actually *yes8. We have `.platform.app.yaml`, which is our main deploy script, and then we have this `/.platform` directory with a few other configuration files inside. The most important one is `services.yaml`. This is where we define services like database, Redis, and others. When we initialized the project, it noticed that we're using Postgres and added one database for us. The error we're getting is because we're missing the PDO-PGSQL driver and, thanks to `.platform.app.yaml`, is *super* easy to add. We have a little extension spot for it, so say `pdo_pgsql`. And that's it!

To redeploy, we actually need to commit these files. Say

```terminal
git comit -m
```

with a very descriptive message

```terminal
"tweaking deploy script"
```

and then we can say

```terminal
symfony deploy
```

and push that back to our main branch. This, like last time, is going to build our script. It takes a minute or so, but this time it should move a little faster because it doesn't need to reprovision the SSL certificate. At the very end, you can see that the migration *still* failed with a *different* error:

`Connection refused`

We'll talk about that in a moment. In fact, if you go back over to the site and refresh... the homepage still works, but that's because the homepage *doesn't* use the database. If you hit "Browse Mixes", we get a 500 error. That 500 error is probably due to the database problem that we saw when we deployed, but let's pretend that we have no idea where that came from. How could we figure that out? This is where the

```terminal
symfony logs
```

command comes in handy. That's going to connect to our active environment. There's a bunch of different logs here, and we'll say `2` to go to the "app" log. This one that actually comes from our application. And... *beautiful*! We can see several instances of

`connection to server at [..] failed: Connection
refused`

Let's think about this. We know that we have a database up there on Postgres, but we've never configured our app to talk to it. Remember, in our `.env` file, we have a `database_URL` that's supposed to point to wherever that database is. We haven't configured a `DATABASE_URL` environment variable on our production site yet, so it's just using this default value, which isn't working. So how do we configure `DATABASE_URL` to point to wherever this database server is? The answer to that is... we... uh... don't. It's kind of cool.

In Platform.sh has this idea of *relationships*. You can have a number of services in your code, and to link them to your project, you'll use relationships. If we search the file for "relationships"... it's not in here yet, so let's add a new section for that with `relationships`... and below that, we can make any key we want, so we'll just say `database`. I'll cover the significance of that in a moment. It's just an internal name. Then,  we're going to set that to the word `database`, because that's the key we have here, with `:` and then the *type* of service, which is `postgresql`. This syntax has always looked really weird to me, but the *important* thing is that this key could be anything, like `banana`, but this `database` here is referring to this `database` over here, and this `postgresql` *here* is referring to *this* `postgresql` here.

This first key is going to be important, because Symfony does a really nice thing when it deploys a Platform.sh. It will notice that this is a database relationship, so it will take this name and expose environment variables that point to that database. Since we called this "database", it will *automatically* create a new environment variable called `DATABASE_URL` that is *equal to* whatever the connection parameters are to that database. In other words, it's going to set this environment variable *for* us. I'll prove it!

One of the really cool things about Platform.sh is you can actually SSH onto your container. Let's say:

```terminal
symfony ssh
```

There we go. And if you want to see all of the different environment variables, you can say

```terminal
printenv
```

and it will print all of them out. Look at that! Inside of here, what you *won't* see is anything that starts with "database". But we *should* see it after we deploy this next change. Say

```terminal
git status
```

and then

```terminal
git add -p
```

That's what we want. Then say

```terminal
git commit -m "adding database relation"
```

and finally

```terminal
symfony deploy
```

You'll probably notice that deploying is *way* faster this time. Since we didn't change any application code, we didn't need to rebuild our application. Here, it says:

`Reusing existing build for this tree ID`

We *only* changed Platform.sh configuration. And hey! This time, we can see that it `Successfully migrated`, meaning that it ran our migration successfully with *no* problems. If we spin over and check the site... it *works*. It's still missing all of our styling, but we'll fix that next. The point is that the *database* is working. And, as I mentioned, you can see the difference that made if you run

```terminal
symfony ssh
```

and

```terminal
printenv
```

This time, you can see that there are several `DATABASE_` things inside of here, but the most important thing, for us, is `DATABASE_URL`. *This* is pointing at our database.

Next: Let's get our assets working, followed by some performance checks to make sure we have everything we need for our production deploy.
