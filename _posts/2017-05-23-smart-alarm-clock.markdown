---
layout: post
title: Smart Alarm Clock
date: '2017-05-23 22:48:25'
tags:
- projects
- misc
- python
- misc-2
---

Tired of you boring alarm on your phone? check out my new alarm clock project that wakes you up, greets you and tells you the weather and the news. Lets begin, ssh into your raspberry pi.

obligatory updates:

<code>sudo apt-get update</code>
<code>apt-get upgrade</code>

now lets begin, the first thing your going to want to do is to clone my Github repository for this project


<code>git clone https://github.com/conorbr/talking-alarm-clock</code>

Great, now we have the project code. Go ahead and cd into the project folder and run the controller.py file. this is the file that pulls the strings.

<code>cd talking-alarm-clock</code>
<code>python controller.py</code>

you can set the time its triggered here in the project code, but what if we could make it easier. well you're in luck! i made an android app to allow you to set the time from your phone.

go ahead and download the app from this link:

http://www.droidbin.com/p1b9mn6b4rlr814tf15td7f01pvc3

and that's it. To make it even better, i know you think thats impossible, but bare with me. we can set the code to run as soon as the device is turned on.

check out my other blog post on raspberry pi startup processes check out my other blog post on startup processes <a href="https://breenblog.herokuapp.com/raspberry-pi-startup-processes/">here</a>

