---
layout: post
title: 'Ruby on Rails: Part 1 - Rails new'
date: '2017-09-06 20:33:30'
tags:
- ruby-on-rails
---

Now that everything is set up and ready to go, its time to start. Open up your terminal.

Im going to call this application pic_aggregator, original I know

`rails new pic_aggregator`

Now to cd into the new directory

`cd pic_aggregator`

That's it, rails has generated all of the files necessary for you, just like that. crazy right?
Now lets see what our brand-spanking new site looks like. For that we'll have to spin up our server for the first time.

`rails s`


```
=> Booting WEBrick
=> Rails 4.2.6 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2017-09-06 18:03:23] INFO  WEBrick 1.3.1
[2017-09-06 18:03:23] INFO  ruby 2.3.1 (2016-04-26) [x86_64-darwin16]
[2017-09-06 18:03:23] INFO  WEBrick::HTTPServer#start: pid=56174 port=3000
```

Now that the server is up and running it time to see the website in action.

![welcome aboard!](https://namecheap.simplekb.com//SiteContents/2-7C22D5236A4543EB827F3BD8936E153E/media/rorapp6.png)

And just like that we're up and running, we're currently riding rails!

Ok, we've got momentum and all we want to do now is to go balls-to-the wall creating models, scaffolds and controllers but before that GitHub needs a moment. As of right now if anything major goes wrong we're screwed.

First thing's first, we need to initialize this folder as a git repository.

`git init`

Add the files in your new local repository. This stages them for the first commit.

`git add .`

Commit the files that you've staged in your local repository.

```
git commit -m "First commit"
# Commits the tracked changes and prepares them to be pushed to a remote repository. To remove this commit and modify the file, use 'git reset --soft HEAD~1' and commit and add the file again.
```

At the top of your GitHub repository's Quick Setup page, click to copy the remote repository URL. 

In Terminal, add the URL for the remote repository where your local repository will be pushed.

```
git remote add origin remote REPOSITORY_URL
# Sets the new remote
git remote -v
# Verifies the new REMOTE_URL
```

then you just need to push the changes in your local repository to GitHub.

```
git push -u origin master
# Pushes the changes in your local repository up to the remote repository you specified as the origin
```

And that's it, in the space of just a few short minutes we have a website, running locally on our server and the code safely stored on the cloud. kicking ass and taking names!

[Part 2; Putting the M in MVC](https://breenblog.herokuapp.com/ruby-on-rails-part-2-putting-the-m-in-mvc/)
