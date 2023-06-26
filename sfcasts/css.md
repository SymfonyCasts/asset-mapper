# CSS

Coming soon...

When we're talking about the front end of a site, we're mostly talking about two things, CSS and JavaScript. So let's start with CSS, which is just dead simple in AssetMapper. You create a CSS file, make sure it's somewhere inside of your assets directory, and then you include it with a good old-fashioned link tag using the logical path to that asset. That's it. You probably knew that. There's zero magic. There's no processing on this file. It just works. This is different than an Encore. With a build system like Encore, you may be familiar with doing things like this, import ./.style/.app.css. That kind of thing is not going to work anymore. In the browser environment, the import statement only works for JavaScript files, so your browser is going to try to import that and it's going to try to parse it like a JavaScript file. You can't do that, but that's okay because including the link tag works perfectly fine. Backing up, we know that we can refer to any file in our asset directory using the asset function, like we did for the CSS file and like we did down here for the image. What if we need to refer to a file like this image from within a CSS file? Check this out. Up here, there we go, I have a little record icon up in the upper left corner. I'm going to change that to be a span with class equals BG logo. We're going to turn that into... We're actually going to put the penguin image right up there. I'll copy that BG logo class and I'm going to app.css. Below we'll say.BG logo and then I'm just going to quickly add some basic styles. Now, if we're setting the background image, this is going to work just like normal. If app.css were just a file that our browser is reading directly, then what we can do inside of here is we can say.. slash images slash penguin.png because that's the relative paths work in a browser like that. This is also what we would do in Encore. I'm happy to say that's also what you do in Asset Mapper. You just point to the relative path and it's going to work. I'm going to also add a background size and a background repeat, no repeat.  Check this out. Refresh up here. That works. What I really want you to see if we inspect that is that there's not really a lot of magic. If we look at the... I clicked to look at the actual CSS file. I'm going to actually open this in a new tab. Perfect. You can see this is our asset styles app.css file and it's just being served verbatim. The relative path works because there actually is... The relative path works like normal except for one small thing. Remember, all these paths are versioned. If you look over here, we just have.. slash images slash penguin.png. Over here we have.. slash images slash penguin dash the version file name.png. There is a tiny bit of Asset Mapper magic happening here because the final file is versioned. Asset Mapper is going to automatically update that line for you. The point is you get to code just like you normally would, just with nice relative paths and everything just works. In this case, it works because Asset Mapper is giving you a little extra help for the version file names. That's it. The next part of CSS is going to be third party libraries like Bootstrap. Let's get that into our project next.