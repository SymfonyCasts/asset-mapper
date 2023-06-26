# Adding Stimulus

Coming soon....

So we can write modern JavaScript in this file, we can import third-party packages, and you are free to write your JavaScript however you want. However, if you're like me, you probably want to use stimulus. So let's get it installed. Now stimulus is just a JavaScript library, so we could just import map require at hotwired slash stimulus and then follow their documentation on how to get things set up. But of course symphony has special integration with stimulus, so instead we're gonna run composer require symphony slash stimulus bundle. The stimulus bundle is a relatively new bundle that houses some twig shortcuts that we're gonna use to, that we're gonna use like stimulus underscore controller. But perhaps more importantly it has a recipe which is gonna get our application set up to load the stimulation controllers with no work from us. We're gonna talk about some of these recipe changes in a second, but before we do, check this out. Thanks to those recipe changes we have an assets controllers directory. There's already a hello controller inside of it. Without doing anything else, I'm gonna go into templates vinyl homepage dot HTML twig and right after the h1 we're going to add a new div that's going to use that controller. So I'll use that new, I'll use stimulus controller. That's one of the shortcuts that came from, new functions that came from stimulus bundle, and we'll say hello. And just like that it works. And you can see down here, you can see we have logs about stimulus initializing and our hello controller matching. So even without a bill system we compose a required stimulus and it just works. I love that. Alright so let's look a little bit closer at how this is working and what the recipe changes did. So in template slash base dot HTML twig, this is probably the least important change it made. It added this line right here called UX controller link tax. It's not important yet. We're gonna talk about that in the next chapter when we talk about UX packages, but sometimes UX packages will come with their own CSS and that's gonna output them. Right now that's not doing anything special for us at all.  Really the main thing that's important for us right now is that it added this new assets bootstrap dot JS file and also in assets app dot JS it added some code to import that. So when this file runs we're importing bootstrap dot JS and then bootstrap dot JS is importing from this at symphony slash stimulus bundle thing. Now let's think. We know this. This is a bare import. It doesn't start with dot dot slash or dot slash. So we know what that means. It's gonna look in the import map to figure out what that file is. If you look in your import map dot PHP file, surprise, the recipe added two new entries in there. Added an entry for stimulus itself but it also added this entry for at symphony slash stimulus bundle which actually points to this weird looking path right here. So you may notice from up here when we're using a CDN we're gonna have a URL key. When we're using when we're pointing to a local path it's gonna use this path key. So this is actually a local asset mapper path. What does it mean? Where is it pointing to? We'll spin over and run bin console debug asset and scroll to the top. Ah check this out. When we installed stimulus bundle it actually added another asset path to our system. It points to vendor symphony stimulus bundle assets dist and it has a namespace prefix. This basically means that any paths inside of this directory are gonna start with at symphony slash stimulus bundle. So over here when we say at symphony slash stimulus bundle slash loader dot JS we're actually referring to this file right here vendor symphony stimulus bundle assets dist slash loader dot JS. So that's a long way of saying that when we import at symphony stimulus bundle what that's actually doing is importing this vendor symphony stimulus bundle assets dist loader dot JS file. That file. So it's a really cool way the bundle basically exposed that to us by adding the asset mapper path and then the recipe added that to import map dot PHP so that it loads. Okay so apparently we're loading this loader dot JS file.  And we can actually see that over in a browser I'm gonna refresh I'm gonna go to my network tools and I'm gonna search inside this box here for loader and there it is. Let me open this up in a new tab. Check this out slash assets slash at symphony slash stimulus bundle slash loader dot JS and there is that code. So with this file does it contains just a bunch of code that just starts the stimulus application and it renders the controllers from our application. But this is just a hard-coded file so how is symphony how a stimulus bundle like able to dynamically load our files inside of the controllers directory. Look at my search inside of here. The answer to that is check out this import up here this import from dot slash controllers dot JS. This is a little bit of magic. If we go back to our network tools and search for controllers there it is controllers dot JS and open this new tab. Check this out it actually has imports controller underscore zero from dot dot slash dot slash slash controllers slash hello controller. And then it exports that in a variable called eager controllers. So this is actually dying this file is actually dynamically built by the bundle. This is not a if we look down in the vendor directory a second ago loader dot JS is just a nice static file. But if you look at controller dot JS it doesn't look like we have in the browser. The asset mapper system intercepts this looks inside of our controllers directory finds all the controllers in this and then dynamically returns this updated version which we then use to bootstrap all of our controllers. So watch check this out let's actually make another file here called goodbye dash controller. You can use dashes or underscores. And I'll just do the same thing but I'll say goodbye controller. Now as soon as we do this you're probably expecting that if I refresh this page we're gonna see it pop on here and you're almost right. The truth is refresh this nothing changes or depending on your app you might even get a 404. That's because the content of this file is going to change and so the hash is actually going to change.  So watch this right now you can see the file being downloaded is 9493 a very fresh right now. Go over and run bin console cache clone clear. And then to refresh that now you can see it has a different file name and the file inside has actually changed you can see dynamically actually add that spot. So hopefully we will fix that bug soon. But our goodbye controller is there. So that's kind of a deep dive into the internals of what's going on here but ultimately the important thing is that bootstrap.js loads a file that starts stimulus and is going to automatically load everything instead of a controllers directory as well as all the third-party libraries and controllers.json which we're going to talk about next by installing a UX package.