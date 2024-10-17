---
layout: post
title: raspberry pi startup processes
date: '2017-05-23 23:56:45'
tags:
- misc
- misc-2
---

This is a handy little trick to get commands up and running from the get-go. To do this were going to be adding commands to the `rc.local` file on the raspberry pi.

So lets go ahead and open it and see whats what.

`sudo nano /etc/rc.local`

Cool, now we're in the rc.local file on the raspberry pi, it should look a little something like this.

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/05/raspberry_pi_rc-local1.png)

For this to work were going to want to put the code in before the line `exit 0` otherwise the file will exit before the code runs.

So for example if you read my instabot post and want to setup your bot to run on startup you could add a line like this:

`python /path/to/your/example.py &`

Theres actually a good bit going on here, so lets get into it. The `rc.local` file runs before the users are loaded so theres no need to use `sudo` because its already the root user.

As well as that the `&` at the end of the command tells the raspberry pi to run the code in this line and continue on with whats under it. Otherwise the pi will get stuck waiting for the process to finish (which it won't)

Aaand thats it, you now have processes running at startup!