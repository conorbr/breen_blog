---
layout: post
title: Opensesamessage
subtitle: House keys? like a peasant?
date: '2018-09-17 23:31:37'
tags:
- python
- ruby-on-rails
- misc
- projects
---

###Update: Project published in PiMag, Check it out [here](https://www.raspberrypi.org/magpi/raspberry-pi-smart-door-project/) 

<br>
<br>

This is my College Final Year Project. And because of its scale I'm going to take a higher-than-usual level approach to this blog post. If you want to read the full [Technical Document](https://www.scribd.com/document/382048062/Technical-Report-Opensesamessage) for whatever reason by all means, enjoy it.

###The problem 
..I found myself facing was that I regularly leave the house without my keys. I'm perpetually writing into the family group asking if anyone was nearby to let me back in. Then it hit me, every time I forgot my keys I invariably had my phone on hand to ask for rescue. And so, the idea was born. OpenSesamessage, conceptually at least, was thrust firmly into existence. A _Smart Home Entry System_ to rival even the greats

<br>

###What's needed
The way this is project is going to work is essentially in three parts. Part one being the device itself, a raspberry pi capable of receiving incoming messages, parsing them and performing functions based on the information received. Second we need a web application allowing users to register their device and add and remove users with access to the OpenSesamessage raspberry pi. And lastly we need a way in which the communication between the Raspberry Pi, the web application and the users smartphone. Something to hold everything together, the gluey goodness that will pull all of these rapidly spinning plates together.

<br>

###The Web App
To start things off I needed a web application, This would act as the base hub for the entire project. From here you could create an account, and admin account where you would register your device. With this account you could add and remove users. The rails application was the first step in a three part system. I'm taking a relatively high stance in describing this, but for a taste of how it worked used the gem [devise](https://github.com/plataformatec/devise) for the user authentication, [bootstrap](https://getbootstrap.com/) for the front end, [psql](http://postgresguide.com/utilities/psql.html) for the database and [telegram_api](http://) for, well more on that later

![rails](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/youre_riding_rails.jpg)

Once the web application was setup I went ahead and bought the domain name and secured hosting for it, just to bump up the marks I was to receive for the project. Below are some examples of the website, the homepage and the `add_user` view:

Homepage:
![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Screen-Shot-2018-06-19-at-15.03.55.png)

New number form:
![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Screen-Shot-2018-06-19-at-15.05.27.png)

<br>
###The Device
Now that I had the web application setup I needed the deice itself, this was an easy decision as I was given a Rasberry Pi as part of my Introduction to IOT module, result! With this I needed a way to establish communication between my Raspberry Pi, my web application and the user's smartphone. I landed on Telegram.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/m6BFtJQW_400x400.png)

Telegram is great, telegram is a privacy focused messenger, with an API and to top it off its cross platform. Telegram was the obvious choice. As mentioned above the gem could be used to send and receive communication on the web application, but for the Raspberry Pi I needed something else. Queue [This GitHub Repository](https://github.com/vysheng/tg), it allowed me to establish a way in which communication could be established between both.

Once I had telegram setup, I needed a way in which I could read incoming messages from users and perform commands based on the incoming information. There were no special libraries for me to call upon so I needed to get my hands dirty. I settled on [Lua](https://www.lua.org/) for this. Its strange, very strange. Lua is this bizzare programming language that was created by Brazilians in the 90's, a language where arrays start at 1. But for the purposes of this project it was perfect. Using Lua I was able to decrypt the messages coming from the web appliaction and execute the specific commands based on the information recieved, such as running the python script to operate the servo to open the door. Below is an example of the accepting a number from the rails website and adding it to the database.

Here's a look at the telegram screen with some of the use case messages. You can see how the device responds to users who have permissions and the message for those who don't.

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/09/70-7d0b417189.jpg)

Add contact method being performed:
![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Screen-Shot-2018-06-19-at-15.36.53.png)

I won't go into the details of the python scripts to operate the servo or any of the other commands as they are in the technical document linked at the start of the post. They now worked for users. They could be added to the system via the web application, informed of their privileges via SMS from [Twilio](http://twilio.com), and given instructions on how to install Telegram and access the system.

###The Prototype
The last was to build the prototype, I'm not a handyman by any stretch of the imagination, but after this I felt like Ron Swanson.

Creating the frame and door was easy enough. Measure twice, cut once. Once the pieces we're cut all that was left was to line up the components before it got a nick of paint.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/09/Snapchat-897179756.jpg)

Don't say it, painting is not my strong suit:

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/09/Snapchat-2070127235.jpg)

And that just about does it. I added a webcam to give a camera function to the admin user for the device. allowing you to toggle automatic photographs of whoever is using the door, or even just request one in real time.

I guess that wraps it up, the door is finished and assembled, the website is allowing user creation and telegram is allowing entry to those who have been previously permitted. And with time to spare to create my technical document.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/09/ezgif.com-video-to-gif.gif)

