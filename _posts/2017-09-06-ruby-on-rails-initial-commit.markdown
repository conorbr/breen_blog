---
layout: post
title: 'Ruby on Rails: Pre-flight checks'
date: '2017-09-06 12:22:00'
tags:
- ruby-on-rails
---

#Pre-flight checks
Ruby on Rails is easy to set up, and there are a million and one tutorials for this. Maybe one day I will make a *how to* for setting up a rails environment, but for now external links will have to suffice.

This blog post will be purely about what I'm going to be using in my rails environment, you can skip to the next one if you've already done this, if this is your first time don't sweat it, its not as bad as it seems.
<br>
### The Framework
First things first, we need to install ruby and Ruby on Rails. This is the Framework that will be used and kinda the reason for this series of blog posts, so yeah we'll need that. We're also going to need a way to manage our code. To add, remove and update our code safely and without breaking anything. And most importantly a way to go back after an inevitable, disastrous change to the project, for this GitHub. This will take about 30 mins and requires you to have a GitHub account, so if you don't have one check out the version control section. Once you're sorted just follow this link, specify what OS your using and follow the guide to the letter. (I use rbenv)

[Ruby on Rails and GitHub setup guide](https://gorails.com/setup/ubuntu/16.04)

<br>
### The Version Control
Version control is going to be handled with git, honestly I don't know how people lived without it, its amazing. Along with git, we're going to use GitHub, this allows us to keep our work online and safe from coffee spills and other natural disasters. Its super easy to setup

[Join GitHub](https://github.com/join)

<br>
### The Database
The database Im going to be using is PostgresQL. It's not essential but for your own sanity I would recommend getting this program. It's invaluable for when something goes wrong, and forgive me if I sound pessimistic, but when it comes to programming, everything that can go wrong will, and in spectacular fashion.

[PostgreSQL desktop application](https://postgresapp.com/)

<br>
### The Editor
The editor I'm going to use is [Atom](https://atom.io/). Atom prides itself on being the *hackable text editor* and to be fair it is. You can add and remove packages to have it just how you want it. Its lightweight so no clunky IDE to fight with, but not so lightweight that you'll pull your hair out trying to look cool in front of the other nerds, looking at you VIM users.
[Atom Download](https://atom.io/beta)
<br>
Im also Going to leave a list of the packages I have installed on atom that I find handy, They're not going to be essential to everyone, and the idea is that you install the ones that make your life easier, don't just copy the ones off some punter from the internet. Find below some punter's from the internet's packages

[How to install Atom packages](http://flight-manual.atom.io/using-atom/sections/atom-packages/)

My list of Packages:

* emmet
* atom-beautify
* copy-as-rtf
* language-lua

<br>
aaaaaand that's it, your all set.
I'll update this post if i think of anything essential to this, but for now this is all you should need.

so without further ado:

[Part 1: rails new](http://www.breen.ie)