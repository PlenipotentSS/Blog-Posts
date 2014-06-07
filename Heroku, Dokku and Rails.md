Heroku, Dokku
------------------------
Setup can be a real B%@#%@! Expletives aside, I was learning how to implement rails, and it is actually really fun framework to work in. After doing a fair amount of iOS coding, I totally see how easy it can be to transition between rails and iOS. It does help that both have an amazing network of support and examples. Cocoapods, as I mentioned before is an amazing source if not for features of your app, than just getting pure inspiration of what others have done. Rails has Bootstrap, which has been an amazing tool as I have just made a few designs. Not to mention sass (.scss files), .haml, and flat out ruby files making my life so much more fun. 

For those who are new to programming, it gets easier after each language you learn. Syntax seems to be an issue only after the first few glances at a language, and as soon as you start writing code in that language; things just flow. Now documentation is another nightmare, and that's honestly what friends are for if you ever get stuck. The MVC ideals still follow in rails (for those who don't know), and after objective-c's straight forward approach, rails was fairly simple to comprehend. 


####Heroku
Heroku has a great free system, that you can easily setup and run your rails service in less than 15 minutes. For Heroku, go yo herokuapp.com and setup an account, and you'll get an IP address to access through ssh (terminal) to configure and push your deployment rails.

The best practice, as I have learned, is to setup a git (the service, not necesarily github) for your project:

```
git init
git touch .gitignore //look for rails or other git ignores online
git add -A .
git remote add REMOTE_NAME LINK_TO_DEPLOYMENT_SITE
```

That's the basic get process to get it started, and when you do have the ssh key for heroku, you'll just use ```git push REMOTE_NAME master``` to setup the rails site! So much easier than sftp and whatever to reset your system and db. Just know that if you do any database configurations on your own computer, you have to do them again on the heroku server. Try this out, and see where you get! Heroku is a great free service, especially if you're just testing things out!

When you are in heroku, just note that to run rails and everything you cannot just type ```rails db:migrate```, you actually have to type ```heroku run rails db:migrate```. It's a silly addition, but it ensures the virtual machine unit behavior with this process.

####Dokku (Digital Ocean)
So if you want to deploy your server on a more dedicated service box, say digitalocean.com - it's also super doable with the Dokku frame running on Linux. You go to digitalocean.com and create a droplet, and make sure you add the application dokku to your droplet!

The procedure is about the same with git, but a little different when you actually get to building your rails (database) and deploying it with git. Now for the database, it's best to use postgresql (you'll likely have to download it via brew or other providers). For Rails, in your databasel.yml, you'll want to change the adapter to postgresql and not use sqlite because Dokku doesn't support that database.

Running on your own machine, you have to start your database as well as rails server:

```
//startup postgres
postgres -D /usr/local/var/postgres

//startup rails
rails s

```

This is important to note, because you'll have to also link your postgres database when you're on your dokku (you didn't necessarily need to do this with your heroku, but it held me up for a while when installing on dokku).

Now that you have your droplet and a rails site, you need to git push to the dokku remote (the ssh ip similar to how you did with heroku). This is also assuming your dokku is setup with rails, but if this is not true, then install everything on your VM that you did for your local machine. Also note that the remote needs to contain the app deployment name. This is the same as the app name and is put at the end of your ssh url as 123.45.67.89:SITE_NAME. 

Next ssh into your VM, and you'll need to first off run the postgres database linkage to your app:

```
dokku postgresql:link APP_NAME DB_NAME
```

This then creates the database link and you can do the db:migrate, create from here:

```
dokku run APP_NAME rake db:seed
```

And that's it! You now know how to setup rails on heroku and dokku! With all these fun names, I just want to make people start saying Gitbo for Github Repos... But I digress...

Cheers!
