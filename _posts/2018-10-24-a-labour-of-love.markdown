---
layout: post
title: A labour of love
subtitle: Raspberry pi with a side of hot coffee
date: '2018-10-24 13:41:36'
tags:
- python
- misc-2
- projects
---

From the dawn of time men have famously had their own primary drive, something that they will go without food/sleep to see complete. In the way that Neil Armstrong felt the need for aviation and later space exploration, the way George Bool created a seemingly useless form of mathematics to which every device we use now owes its existence. I, it would seem, found my time best spent shamelessly trying to impress a girl.

Some backstory, I'm terrible for looking at my phone. It's perpetually on silent, because I feel that I should be the one who dictates when it gets my attention. Because of this people are *sometimes* left unread.

Cue, what I have come to call the Megan Beacon

##A quick overview

I have a raspberry pi on my desk, that's perpetually running small programs that I have written, from sensor readings to a python script to enter me in this weekly competition for a free brunch.

A [Speaker](https://media.4rgos.it/i/Argos/8266619_R_Z002A?$Web$&$DefaultPDP1240$), the very basic [Bush Cube Wireless Speaker](https://www.argos.co.uk/product/8266619?rec=PDP[4284183]:bottomSlider:P1:OHAT:alternative:8266619:53nlbKjTsxsfQBlMl41d)

She has an iphone, [because of course she does](https://developer.apple.com/swift/).




So, this project will be a mobile app written in swift, that will send a message using [dweet](https://dweet.io/) via the library [dweepy](https://github.com/paddycarey/dweepy), to be heard by my raspberry pi using Python.

##The script
The script is super easy. Its just a small Python script that will compare a stored string in a text file against a string sent through dweet from the app. If there's a discrepancy then the audio file plays and the text file is updated. Simple

~~~ python
import dweepy
import json
import time
import os

from gtts import gTTS
tts = gTTS('pay attention to me')
tts.save('sound.mp3')

while True:
    try:
        with open('need_token.txt', 'r') as myfile:
            file_data = myfile.read()
        dweet = dweepy.get_latest_dweet_for('MY-SECRET-DWEET-PATH')
        value = str(dweet[0]["content"]["token"])
        if file_data == value:
            print("yep")
            time.sleep(10)
        else:
            print("nope")
            # play audio file
            # write new number to token file
            print(value)
            os.system("afplay sound.mp3")
            with open('need_token.txt', 'w') as myfile:
                myfile.write(str(value))
            print("updated old value " + file_data + " file with " + value)
            time.sleep(10)
    except:
        print("there was nothing there")
        time.sleep(10)

~~~

##The App
I've never created an IOS app before so this was a huge learning curve that involved getting to grips with [swift](http://), but in the end it was actually fine, and proved to be a nice environment. The app itself is quite simple, It's just a single button that when pressed generates a random eight character string and send it to Dweet.io via a post request. when run it also makes use of the [Google Text to Speach](https://github.com/pndurette/gTTS) library to create an audio file to play. This could easily be replaced by, in my case a WhatApp voice message.

~~~ python
import UIKit
import Alamofire

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }
    
    @IBAction func UIButton(_ sender: UIButton) {
        let parameters = [
            "app": "megan",
            // calling upon random string function, passing in the arguement of 8 for 8 characters
            "token": randomString(length: 8)
        ]
        
        let alertController = UIAlertController(title: "PRIVATE ENDEERING MESSAGE", message:
            randomString(length: 8), preferredStyle: UIAlertController.Style.alert)
        alertController.addAction(UIAlertAction(title: "Dismiss", style: UIAlertAction.Style.default,handler: nil))
        
        self.present(alertController, animated: true, completion: nil)
        Alamofire.request("HTTPS PATH TO SECRET DWEET URL", method: .post, parameters: parameters, encoding: JSONEncoding.default, headers: nil)

    }
    
    //function to generate a random sting
    func randomString(length: Int) -> String {
        
        let letters : NSString = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
        let len = UInt32(letters.length)
        
        var randomString = ""
        
        for _ in 0 ..< length {
            let rand = arc4random_uniform(len)
            var nextChar = letters.character(at: Int(rand))
            randomString += NSString(characters: &nextChar, length: 1) as String
        }
        
        return randomString
    }

}
~~~

The I used [alamofire](http://https://github.com/Alamofire/Alamofire) for the post request, and installed it using [cocoapods](https://cocoapods.org/). I'll link the repositories at a later date for any of the fellow romantics out there.
