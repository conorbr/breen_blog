---
layout: post
title: Twitter Bot
date: '2018-06-13 14:24:37'
subtitle: 
tags:
- projects
- python
- misc
---

Taking a break from the usual ruby coding I decided to try something new. I've been seeing a lot of controversy over Donald Trump's _Russian spy bots_, so I've decided to make my own.

From what I can tell it's surprisingly easy to make one of these and there's a library already available to make use of the Twitter API, result.

Lets begin, to start we need to install the Tweepy library:

~~~ python
pip install tweepy
~~~

You can also clone the repository and install it manyally if you should so wish (I ended up using this method):

~~~ shell
git clone https://github.com/tweepy/tweepy.git
cd tweepy
python setup.py install
~~~

Lovely, now before we start we need a twitter account, get a profile picture and form an identity.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Screen-Shot-2018-06-13-at-13.13.05.png)


Now that we're up and running, lets open up our favorite [text editor](http://atom.io) and begin. To start we need to get the keys from twitter which requires you to go back and forth with registration forms and email verification etc. That's boring so lets skip that part and get into the code.

~~~ python
import tweepy
from tweepy.streaming import StreamListener

consumer_key = 'MY_CONSUMER_KEY_GOES_HERE'
consumer_secret = 'MY_CONSUMER_SECRET_KEY_GOES_HERE'
access_token = 'MY_ACCESS_TOKEN_GOES_HERE'
access_token_secret = 'MY_ACCESS_TOKEN_SECRET_GOES_HERE'

auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)

user = api.me()
~~~

Now we can begin, the first thing I want to do is to reply to Trump's tweets. As the political representative for Ireland Big Sean Murphy needs to be vigilant, striking as soon as a tweet is posted and replying post haste with the words of the Irish people. So lets do that.

This line will get the last tweet posted by the user realDonaldTrump, which will work nicely for our purposes

~~~ python
stuff = api.user_timeline(screen_name = 'realDonaldTrump', count = 1, include_rts = True)
~~~

and to reply all we need to do is to use the `update_status` method

~~~ python
api.update_status("@realdonaldtrump " + tweet, in_reply_to_status_id = status.id)
~~~

That's basically it. We need to make sure we don't reply to the same tweet twice, so it makes sense when replying to a tweet to save the tweet id to a text file `last_tweet_id.txt` and throw the whole thing into an `IF/ELSE` statement like so:

~~~ python
for status in stuff:
    with open('last_tweet_id.txt', 'r') as myfile:
        old_tweet_id = myfile.read().replace('\n', '')
        if old_tweet_id == str(status.id):
            print("no new tweets, taking a 5 minute break")
            time.sleep(300)
        else:
            tweet = "some kind words"
            api.update_status("@realdonaldtrump " + tweet, in_reply_to_status_id = status.id)
~~~

Looking good, the next thing I want to try is have Big Sean Murphy read a tweet sent to him. How cool would it be if you could tweet at him with the hashtag #TellHim and he would relay that message onto Doland Trump for you, in the voice of Ireland's political representative.

We've a good thing going with the loop n' sleep format from the last one so lets use that. However, while we can remain fairly certain trump isn't going to tweet 100 times in a minute, the popularity of Sean Murphy and his responsibility to the Irish People requires him to respond to every beck-and-call, for the good of the realm. Lets get the last 100 mentions just to be safe

~~~ python
for mentions in tweepy.Cursor(api.mentions_timeline).items():
~~~

(we can also use a text file to make sure we don't have any duplicates)

~~~ python
for mentions in tweepy.Cursor(api.mentions_timeline).items():
    if "#tellhim" in mentions.text.lower():
        if str(mentions.id) not in open('reply_tweet_ids.txt').read():
            text_file = open("reply_tweet_ids.txt", "a")
            mention_tweet = str(mentions.id) + "\n"
            text_file.write(mention_tweet)
            text_file.close()
            reply = ["{handle} I'll tell him"]
            handle = "@" + mentions.user.screen_name
            response = " " + random.choice(reply).format(handle=handle)
            api.update_status(response, in_reply_to_status_id = mentions.id)
            reply = "@realDonaldTrump" + mentions.text.replace("#tellhim", "").replace("#TellHim", "").replace("@Big_Sean_Murph", "")
            api.update_status(reply)
            print(reply)
        else:
            print("skipping old tweet")
    else:
        print("not a tell him tweet")
time.sleep(120)
~~~

Nice use of string interpolation there if I do say so myself, now to have these both run side by side we're going to need threading. multi-threaded programming, It sounds like I'm applying for a job.

First things first we need to `import threading` and while we're at it `import random` for later.

Then we can define each part of the code as its own thread like this

~~~ python
def thread_1():
   #send tweets to trump


def thread_2():
   #reads mentions and sends tweets

t1 = threading.Thread(target = thread_1)
t2 = threading.Thread(target = thread_2)

t1.start()
t2.start()
~~~

Now all we have to do is put everything together and run it. I have also flushed out the responses into three different arrays. This means that to form a tweet Sean Murphy can chose a string from each array and piece together a response on the fly.

~~~ python
import tweepy
import Tkinter
import random
import threading
import time
from tweepy.streaming import StreamListener

consumer_key = 'MY_CONSUMER_KEY_GOES_HERE'
consumer_secret = 'MY_CONSUMER_SECRET_KEY_GOES_HERE'
access_token = 'MY_ACCESS_TOKEN_GOES_HERE'
access_token_secret = 'MY_ACCESS_TOKEN_SECRET_GOES_HERE'

auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)

user = api.me()



def thread_1():
    while True:
        stuff = api.user_timeline(screen_name = 'realDonaldTrump', count = 1, include_rts = True)
        old_tweet_id = open("last_tweet_id.txt", "r")
        for status in stuff:
            with open('last_tweet_id.txt', 'r') as myfile:
                old_tweet_id = myfile.read().replace('\n', '')
                print(old_tweet_id)
                if old_tweet_id == str(status.id):
                    print("no new tweets, taking a 5 minute break")
                    time.sleep(300)
                else:
                    sentence_1 = ["you","you are a","you're a","christ almighty","mother of god","Jaysus you're a","",]
                    sentence_2 = ["feckin","complete","complete and utter","total",""]
                    sentence_3 = ["gobshite","numpty","moron","gowal","tik","thick","twat", "dope", "amadan"]
                    print("Found new tweet:'" + status.text + "' with status id: " + str(status.id))
                    print("sending a reply...")
                    tweet = random.choice(sentence_1) + " " + random.choice(sentence_2) + " " + random.choice(sentence_3)
                    print("sending tweet: " + tweet)
                    new_tweet_id = str(status.id)
                    text_file = open("last_tweet_id.txt", "w")
                    text_file.write(new_tweet_id)
                    text_file.close()
                #send tweet
                    api.update_status("@realdonaldtrump " + tweet, in_reply_to_status_id = status.id)

def thread_2():
    while True:
        for mentions in tweepy.Cursor(api.mentions_timeline).items():
            if "#tellhim" in mentions.text.lower():
                if str(mentions.id) not in open('reply_tweet_ids.txt').read():
                    text_file = open("reply_tweet_ids.txt", "a")
                    mention_tweet = str(mentions.id) + "\n"
                    text_file.write(mention_tweet)
                    text_file.close()
                    # read_reply_id = myfile.read().replace('\n', '')
                    reply = ["{handle} oh I'll feckin tell him", "{handle} I'll let him know", "{handle} oh i'll tell him", "don't worry {handle}, he'll hear the words of the irish people"]
                    print("new tweet request")
                    handle = "@" + mentions.user.screen_name
                    response = " " + random.choice(reply).format(handle=handle)
                    print("responding to tweet: " + response)
                    api.update_status(response, in_reply_to_status_id = mentions.id)
                    print("sending trump a tweet...")
                    reply = "@realDonaldTrump" + mentions.text.replace("#tellhim", "").replace("#TellHim", "").replace("@Big_Sean_Murph", "")
                    api.update_status(reply)
                    print(reply)
                else:
                    print("skipping old tweet")
            else:
                print("not a tell him tweet")
        time.sleep(120)




t1 = threading.Thread(target = thread_1)
t2 = threading.Thread(target = thread_2)

t1.start()
t2.start()
~~~

It appears to be working as it should.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Screen-Shot-2018-06-13-at-13.14.38-1.png)

don't forget to [follow him on twitter](https://twitter.com/Big_Sean_Murph/with_replies)
