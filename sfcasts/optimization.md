# Optimizing for Speed

Coming soon...

Now, let's check the performance of our site. In Chrome, there is a tool called Lighthouse. I'm going to run just this for performance. Analyze page load. This is really the best way to see if you really have problems. I'm probably going to score pretty high just because my site is so small and so fast. We're going to zero in on a couple potential problems. And yep, 98 with no build system. Very little effort. That is amazing. So, let's look down here at where we did have some problems. So the first thing is you can see eliminate render blocking resources. And it's pointing to our font file. First thing it's doing is actually pointing to our font file. So a lot of what we're going to talk about right now has nothing to do with AssetMapper. It's just front end performance in general. So this file here, if you open templates slash base.html twig, this is one of our link tags up here. We have a link tag that's pointing towards this font that we need. When your site sees link tags, it has to download them first before it can render the page. So it's almost like it freezes rendering of the page right here and downloads this file. But here's the thing. If you open that file, and actually let's get a non-minified version of it, this has a bunch of potential font faces. And the cool thing about the way the fonts work is that these font files themselves won't actually be downloaded until and unless we use this font. And also because of this font display swap, that basically tells the browser, hey, it's okay if you are rendering some text that's supposed to be using this font template. It's actually okay if you use the default system font first, show the text, finish downloading this font file, and then use the font. So this CSS file is written in a way where it's kind of like all of these font files in here are going to be downloaded lazily.  The problem, and it's probably not actually a big problem, but if we want to fix this, the problem is that at this point our browser, all it does is it sees a CSS file and it thinks, I need to download that CSS file right now and I can't render the page until that CSS file is here. Once it does that, it finds out that there's a bunch of font files that it can lazily download. So if we do want to eliminate this render blocking resource, the way to do that is actually to copy this file, in this case I'm just going to copy the font face I need. A lot of these are for other different languages. I'll copy the two Latin. I probably don't even need the Latin extension one. Then we're going to delete this CSS file entirely and go to assets, styles, app.css, and we're going to paste the two files there. You're going to notice these are not real URLs, so I'm going to go up here and grab this, take off the index.css. That should be right. Let me copy that. So this is adding some complexity to our code, so that's why you can do it or not do it. Right now it's going to download our app.css file, which it knows it needs to, and it's going to immediately see that it doesn't need to. It can kind of download these files when it needs to. It eliminates that one render blocking resource. We're going to see that in a second. The other failure we have is actually very similar. It's for font awesome, so it's for a JavaScript file. That's also in base.html.twig. So because this script tag doesn't have defer or async on it, this is also going to block the rendering of the page. So if we want, we can add defer to this. So this says download this, but don't block the page when you're downloading this. So this is actually going to load our font awesome fonts, so the kind of worst case scenario here is that our page loads and our fonts load just a second later from font awesome because this wasn't blocking the page. All right, so let's test things. I'm not going to bother redeploying.  I'm actually going to go back to just our local site over here. We can also run Lighthouse over here. Perfect. I'll analyze the page load, make this a little bit bigger. And awesome. Now we're getting 100 locally. So let's look down there. Our opportunities, serve images in next gen formats. Probably I should do that. It says avoid serving legacy JavaScript to modern browsers. I'm pretty sure this is just talking about the import map shim that we're including, so not a big deal. But this avoid chaining critical requests, this is probably the most important thing to look at. So here's the thing. As you can see, it downloads the page first. Once it downloads the page, then it realizes it needs to download this CSS file. Then it downloads the CSS file and then it realizes that it needs to download this font file. So you can see it actually kind of doesn't... Instead of it knowing that it needs to download this file and this file immediately, it kind of has to find out about them little by little, which means ultimately this font file, even if it's not blocking the page, it's going to take longer to load that because we have a couple of steps before we even find out that we do need to load that. So if you care enough about this, you can go and find that font file. So that's actually this normal one here. You can take this URL and we can preload this. This is where we go into our base.html.twig. Up here, we put a little thing that's basically going to hint to the browser, hey, you don't see why yet, but you need to download this. So we'll say rel equals preload. href equals that long thing. Then for fonts, you need to have an as equals font. Then you need to have a type equals font slash wof f2. Then also this cross origin on the end. It's kind of a long thing here, but that's going to hint that it needs to preload that. If we go over now and run Lighthouse again, still score 100. You can see a Void Chaining Request. That file is gone. So we fixed that. What about app.css though? You can't really load that any earlier, right? That's inside of our base.html.twig already.  So we download the page and as soon as it downloads the HTML, it sees this CSS tag. Well, that's true, but could we move this even earlier? Could we somehow hint to the browser that it needs to download this app.taylor.css even before it downloads the HTML? The answer, crazily, is yes. To use this, we need a Symfony component called weblink. So say composer requires Symfony slash weblink. Once that's done, we're going to add another preload down here. It's going to look kind of similar. Link rel equals preload. href equals. This time we're going to use curly curly. We're going to use a function called preload. I'll talk about that in a second. And then we're going to use our normal asset function to point to styles slash app.taylor.css. In this case, this preload function needs another option here called as style. That's it. It doesn't need any more attributes. Now this by itself, if I go and refresh the page and view the page source, is going to output a preload tag. This by itself is kind of pointless. This is basically telling the browser, when the browser's reading HTML and it sees this, it's going to say, hey, you should start downloading this CSS file. But one line later, it would have found out anyways. So this is completely pointless. However, the fact that we call this preload function first, that actually tells Symfony, hey, when you return the response for this page, add a preload header. So the really important thing here is that if we go over to the network tools and let's go to all and actually find our main page here, if you go to headers and you go to response headers, look it, there's a new link header here called preload that points to our CSS file. So as the response is being sent back to the user's browser, at the very, very top of the response in the header is that hint to start downloading that CSS file. So it actually can start downloading the CSS file before it even downloads the HTML file. So if we go back over here to Lighthouse and look down here, beautiful. Look at that. The entire section is gone. Now there are a couple other things here like keep request counts low and transfer sizes low.  See, this isn't a warning. It's just something to look at, but it's not really a problem. But on that topic of preloading, we don't see any warnings here, but there is one more spot where preloading can improve performance and it has to do with JavaScript. So over in our import map.php file, there's a key inside here called preload that we haven't talked about. You can see it's set to true for app. I'm going to set that to false and then I'm going to go up here and run another Lighthouse report. So it's looking 100 score, but if we go down here, look it says avoid chaining critical requests and check this out. The HTML page down to app.js, bootstrap.js, the stimulus loader, stimulus itself, controllers. Holy moly, a bunch of stuff happened. So this is the same problem that we saw with the CSS and the font files earlier. First our browser downloads the HTML page. Then it sees it needs to download app.js. Then once it downloads that file, it sees it needs to download our bootstrap.js. Then when it sees that, it sees it needs to download the stimulus funder loader. And so one by one by one, instead of downloading all of those things as soon as the page loads, it has to discover them little by little by little, which is not ideal. So the preload is what's fixing that. I'm going to change that back to true, refresh the page, and then let's view the page source. One of the things that we haven't talked about yet here is that comes from the import map here, is we know that that dumps the import map, and we know it dumps our little script type equals module down here, but it also dumps these module preload things. So the idea is pretty cool. Because we said preload app true, it has a link rel equals module preload here for app.js. That hints to the browser that it should start downloading app.js immediately. Now that's not really that important because it would have figured that out down here anyways. But then asset mapper sees that our asset slash app.js imports bootstrap. And so since app.js is preloaded, it also preloads bootstrap.js. And since this also imports lib slash vinyl. js, it also preloads lib slash vinyl.js. So it's going to start downloading all three of those files immediately. Now notice... Now at this point, if we ran Lighthouse again, it wouldn't complain about any of these chain requests, but it's still not actually perfect at this moment. And you can see it if you look over the JavaScript thing, and you kind of focus on the waterfall over here. So you can see a couple... Oh, let me move that... A couple of files start downloading, and then some more start downloading, and then some more start downloading. So we still have a bit of this chaining problem. So we know, for example, that the bootstrap.js is being preloaded, but this file is not being preloaded. So it downloads bootstrap.js, and then it discovers it needs this file, and it discovers the things inside of it. So the reason this happens is that we're preloading app, and so anything that app imports with a relative import is automatically going to be preloaded as well. But anything that we're importing with a bare import, like lodash slash camelcase or at symphony slash stimulus bundle, is not automatically going to be preloaded, and that's because these have their own entries inside of here, so you kind of control the preloading for those by themselves. So this is a bit of an optimization on performance, but if you want to, we can actually add preload to the items here that we know are going to be critical to our page rendering. So stimulus is going to be critical, stimulus bundle, because that's what loads our controllers, and also turbo. So I refresh. Nothing changes. We're just going to see even more of those module preloads down there. If we run Lighthouse one more time, I mean, we were already scoring 100, and we're still scoring 100, and you can see there's no major problems down here. All right, friends, thank you for joining us, Asymapper's cool. There are some things it doesn't do, like tree shaking or handle TypeScript, but for a large number of projects, it's going to work great. And the cool thing is, you're still running normal JavaScript, so if you ever did need to move to a build system later, you could super easily.  All right, let me know what you think, and hopefully we can keep improving this for Symphony 6.4, and I'll see you all around. Bye.