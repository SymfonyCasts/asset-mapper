# Debugging

Coming soon...

There are a few ways that things can go wrong sometimes with Asset Mapper, but also some telltale signs when they do. So I want to show you kind of a few of the most common ways. So the simplest way to mess things up is, for example, in templates="vinyl-homepage.html-twig". If you use the asset() function and you mess up a path. So remember we have inside of the assets directory, we have an images/.png. So images/.png would be its asset path. So I'll say images/. But then I'll say duck.png. So obviously not the right path. And not surprisingly on the homepage, we are going to get a 404. Now the key thing about that 404 is if we look at our console, look at the path there. What's suspicious about that path is that there's no version in the file name. That tells me that this path was not found inside of any of the asset paths. In other words, I have some sort of typo. This is not a valid logical path. And if you remember, you can see all the valid logical paths by saying debug asset. So up here, this is everything that you're allowed to put into the asset function that will go through the asset mapper. And there's images/.penguin.png. So of course we put penguin.png there. Now it works. And most importantly, the thing I'm looking for here is you can see the version hash in the file name. That's how you kind of know it's going through the asset mapper system. All right. Another common thing to do is you mess up an import somewhere. So maybe in styles.app.css, you mess up this URL path. Or in app.js, you're importing vinyl and you forget the.js in the end, or some other typo. So this is actually the same thing. When I refresh, I'm going to get a 404. You can see it here. But again, the key thing is no version hash. So whenever you have no version hash, you have to be thinking wherever I'm using that up here in app.js, I have some sort of invalid path there. Now when we're inside of a template and using the asset function, we're using the logical path. If we're inside of app.js or app. css, we're not using the app, we're not using the logical path here. We're using just the relative path. This is actually really cool. We get to code inside of here as if there's no asset mapper system. So you don't have to think about logical paths. You just think about what's the path relative to this file and it's lib slash vinyl.js. So the point is the missing version hash here is a really good hint that that path ultimately wasn't found inside of the asset mapper system. There's also another way to see any problems that you have. That's to run over and run bin console cache colon clear. Now clear symphony's cache and actually even the assets inside of asset mapper are cached internally. So by clearing that, now when I run debug asset, it's going to build all the cache for those assets internally. And the important thing about that is it's going to parse those files. And if it finds any missing imports like this, it's actually going to report them. Check this out. Warning, unable to find asset dot slash lib slash vinyl imported from app.js. In this case, you even have the extra thing. Try adding dot JS to the end of the import, since it sees that that is missing. This is a really good thing to see what's going on. So now we have that dot JS back and of course things work again. And probably the last common way to mess things up is to use a bear import. So something that doesn't start with dot slash or dot dot slash, but that doesn't exist in your import map. So clearly here, my intention is for me to use the bootstrap library, but I don't have it in my import map. So the error I'm going to get is this. The exact error is going to look a little bit different depending on your browser, but failed to resolve module specifier bootstrap relative references must start with slash dot slash or dot dot slash. So it's saying, hey, if you're trying to refer to a file with a relative path, it should start with dot slash or dot dot slash. And if you did mean to have a bear import, we can't figure out what that is because it's not inside the import map.  So solution usually here is to run import map require to install the library that you're using. All right. With those tools in hand, let's move on to deploying. I think I can't remember. We'll fix that later.