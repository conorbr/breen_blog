---
layout: post
title: Unplanned obsolescence
subtitle: 'Are managers doomed to forget it all and become suits. A TED talk'
date: '2022-07-13 21:29:41'
hidden: true
---

There comes a time in every man's life where he is confronted with the stark realisation of own inability, for every Mike Tyson there's an Evander Holyfield, for every Lance Armstrong there's a controlled substance committee. And for me there was this coding challenge 

A little back story, I've been working as a engineering team lead and scrum master for almost two years. In this time I've hung up my mechanical keyboard, company-branded hoodies and a sticker-clad mac for the monochromatic grey suit that I have become. Recently I was presented with a coding challenge, which on the surface looked like a cake walk. The problem was interesting, the language was familiar and the bulging ego from past victories meant I was unstoppable.

However, I brained it.

![](https://y.yarn.co/1789ea3a-19c7-4419-bf7f-87440abd7bf1_text.gif)

Let's get into it.

### The Challenge
The challenge is pretty straight forward, there's a bunch of forklifts that are moving around a warehouse. Each collecting data of where they are and relaying it to a "central server" (text file) that's read in and some funky stuff happens and it spits out a message in the terminal. Our job is to make those messages make sense.

you can find the code here: (pending permission from the owner)

The project structure looks like this:

~~~ txt
===================
ping.rb
position.rb
README
run_warehouse.sh
runner.rb
vehicle.rb
warehouse_pings.csv
warehouse_server.rb
===================
~~~

As you can see not crazy, there's the `warehouse_pings.csv` where the data is kept, a `README` with instructions and a `run_warehouse.sh` bash script to start it all. Each time the program loads we process the pings in the csv, creating instances of vehicle classes in the process (if needed or else just adding pings to one that's already there) It's also worth nothing that each time a ping class is initialized so is an associated position class.

Looking at the `README` there's three things we want to achieve, we'll start with the first:

~~~ text
1.  Add the function to vehicle.rb:

        self.get_total_distance(pings)

    This method takes a list of Pings and should return the total distance
    traversed by the given forklift. You can assume the input list is sorted
    with the earliest ping at the start. Once you are done, use
    vehicle.get_total_distance in your implementation of:

        get_average_speed

    This method should return the average speed (total distance / total time)
    observed for the given forklift. A forklift which has never moved would have
    an average speed of 0 meters per second. You can assume the input list of
    Pings is sorted with the earliest ping at the start.
~~~

Channeling [daddy's Silly Triangle](https://www.youtube.com/watch?v=dmdPwqLqFhk) (distance/speed/time) from third year maths, or you know actually reading the question, we know that with distance and time we can always figure out the speed. So let's start with those.

Looking at the `vehicle.rb` file a fair amount is done for us:

~~~ ruby
# ./vehicle.rb

# A named vehicle with a sequence of pings.
class Vehicle
    attr_reader :name, :pings

    def initialize(name)
        # The name of the vehicle.
        @name = name
        # The pings for the vehicle, in chronological order (earliest first).
        @pings = []
    end

    # Determines the total distance covered by the pings.
    def self.get_total_distance(pings)
      # TODO: implement
      0.0
    end

    # Determines the total distance traveled by the vehicle.
    def get_total_distance
        self.class.get_total_distance pings
    end

    # Determines the average speed of the vehicle.
    def get_average_speed
        # TODO: implement
        0.0
    end

    def to_s
        "#{name}: #{pings}"
    end
end

~~~

This doesn't look too bad now, but everything's a little clearer without the panic of a ticking clock, so let's take our time.
The first D in our oh-so reliable triangle is distance, and instead of trying to re-program everything like the first time let's have a look around and see if there's anything we can make use of.

Oh, Position has a method that does exactly what we want. Funny that.

~~~ ruby
# ./position.rb

    # Determines the distance between two Positions
    # Distance is calculated as the Euclidean distance in two dimensions
    # https://en.wikipedia.org/wiki/Euclidean_distance
    def self.get_distance(position_1, position_2)
        x_diff = (position_1.x - position_2.x).abs
        y_diff = (position_1.y - position_2.y).abs

        Math.sqrt(x_diff ** 2 + y_diff ** 2)
    end
~~~ 

This means that all we have to do is expose that classes method and iterate through our list of `@pings` and get the sum of the output of each call to this method for each ping's position.
We can use ruby's map function for this, maintaing an index so we can compare the current ping to the last one and then sum the array. Maybe I should have broken this out, but I've been doing a bunch of [katas](https://www.codewars.com/users/conorbr/completed) recently.

~~~ ruby
# ./vehicle.rb

    # Determines the total distance covered by the pings.
    def self.get_total_ping_distance(pings)
        @pings.map.with_index { |p, i| Position.get_distance(p.position, pings[i - 1].position) }.sum
    end
~~~

Now we can worry about time, each ping has a timestamp, I'm not really sure what format it's in but it doesn't really matter. Taking one from the other will still give us the difference in seconds. I wonder if there's code already here for this.

~~~ ruby
# ./position.rb

    # Determines the number of seconds between two given Pings. The result is
    # positive if ping1 is earlier than ping2.
    def self.seconds_between(ping1, ping2)
        ping2.timestamp - ping1.timestamp
    end
~~~

Again, I'm kicking myself.

Using this new-found knowledge we can pretty much copy and paste the last solution and sort time just like that. I'm also going to rename one of the two `get_total_distance` methods, I'm not sure why there's a class method and an instance method with the same name, but it's not doing anything useful as far as I can tell. I'll also define a new method for `get_total_time` in case we need it in the future.

~~~ ruby
# ./vehicle.rb

    # Determines the total distance covered by the pings.
    def self.get_total_ping_distance(pings)
        pings.map.with_index { |p, i| Position.get_distance(p.position, pings[i - 1].position) }.sum
    end

    # Determines the total distance traveled by the vehicle.
    # why is this necessary?
    def get_total_distance
        self.class.get_total_ping_distance(pings)
    end

    def get_total_time
        time_array = []

        @pings.each_with_index do |p, i|
            next if i == 0

            time_array << Ping.seconds_between(pings[i - 1], p)
        end

        time_array.sum
    end

    # Determines the average speed of the vehicle.
    # default average speed i m/s
    def get_average_speed
        get_total_distance / get_total_time
    end

    # https://www.hackmath.net/en/calculator/conversion-of-speed-units?unit1=m%2Fs&unit2=km%2Fh&dir=1
    def get_average_speed_kmh
        "#{( get_average_speed * 3.6).round(2) } km/h"
    end
~~~

Seeing as we're working with meters and seconds we can ask google how to calculate that and then interpolate that into a string to make our print statement a little more readable. Lets take a look at our print statement now:

![print_statement_1](/img/ruby_coding_challenge_1.png)

Looking good, but there's always one. And that one is vehicle M. I genuinely have no idea what's going on with this guy, so lets explore another one of my regrets and `require 'pry'` in our `runner.rb` file and take a look at why this vehicle isn't playing nice.

We can drop this guy into our `get_average_speed` method, but we only want to stop when we're on vehicle M so a little logic helps here too.

![print_statement_1](/img/ruby_coding_challenge_2.png)

Looks like this guy only has one ping, meaning his speed is 0 and so is his distance. Dividing by 0 won't do. So we're going to put in a return statement to catch stuff like this. It will also catch negative numbers, we don't want them either.

~~~ ruby
# ./vehicle.rb

    def get_average_speed
        return 0 unless (get_total_distance / get_total_time).positive?
        
        get_total_distance / get_total_time
    end
~~~

I think we can call it on the first question now, lets take a look at the next one.

~~~ txt
2.  In warehouse_server.rb, implement the method:

        get_most_traveled_since(max_results, timestamp)

    This method returns an array of the max_results forklifts that have traveled
    the most distance since timestamp (inclusive), sorted by those that have
    moved most to least. You can assume the lists of Pings for each vehicle are
    sorted with the earliest ping at the start.
~~~

Ok this next one is pretty handy, for each vehicle you want to get the pings that are greater than or equal to timestamp, throw them all into the `get_total_ping_distance` method we've already created, store, sort and grab only the first `max_results` elements to return. 

here's what we started with:

~~~ ruby
# ./warehouse_server.rb

    # Returns a sorted list of size max_results of vehicle names
    # corresponding to the vehicles that have traveled the most distance since
    # the given timestamp.
    def get_most_traveled_since(max_results, timestamp)
        # TODO: Implement
        []
    end
~~~

And here's what we're left with:

~~~ ruby
# ./warehouse_server.rb

    def get_most_traveled_since(max_results, timestamp)
        vehicle_hash = {}
        

        @vehicles.each do |v|
            pings = v.pings.select { |p| p.timestamp >= timestamp }
            vehicle_hash[v.name] = Vehicle.get_total_ping_distance(pings)
        end
        
        vehicle_hash.sort_by { |key, value| value }.reverse.first(max_results).to_h
    end
~~~

The requirements says to return a list (array) but it looks nicer as a hash, so I've gone with that. you can add `.keys` to the end of the last line if you so desire. We could also have done something fancy to round the values and concatonate a cheeky 'meters' on to the end of each value to make it look pretty. But sure look.

Now let's take a look at our console:

![print_statement_1](/img/ruby_coding_challenge_3.png)

pretty tight.
I'm going to leave question 3 for part 2, there's a couple of funky things that can be done with that one. There's also a fair reason to throw in a few tests and seeing as it's not rails we don't have all of the setup to worry about so stay tuned.

### Conclusion

There is no experience more sobering than discovering you can no longer do something that was once second nature, but what Joni Mitchell failed to mention in the seminal classic [Big Yellow Taxi](https://www.youtube.com/watch?v=94bdMSCdw20). It's not always that hard to get it back.

![print_statement_1](https://media4.giphy.com/media/OWIhXdUW4gRYLSL4Sb/giphy.gif?cid=ecf05e47sj1g53rn09ye0q9v7ghuj9u3iaomyme8i5fpiddn&rid=giphy.gif&ct=g)
