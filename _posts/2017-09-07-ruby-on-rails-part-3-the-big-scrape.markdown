---
layout: post
title: 'Ruby on Rails: Part 3 - The Big Scrape'
date: '2017-09-07 13:10:17'
tags:
- ruby-on-rails
---

We have the Model, the view (kinda), and the controller. Now we need the data, the information, the content to show users when the arrive to the site.

I've toyed with where to get this content, and while browsing reddit for an answer it occurred to me, REDDIT! For those of you who don't know reddit is '*the front page of the internet*'. Users if the site upload content, and the community handles it. Content is up-voted and down-voated, so reddit has become a sort of self-regulating community with all of the newest, most relevant content on the internet. Because of this it is Reddit that we will pull our information from.

Reddit also has an API, score!
so we're going to need a few new gems. we're gonna go ahead and add these gems to our gemfile:

```
gem 'httparty'
gem 'json'
```

Http party will allow us to make a request to reddit's, and json, you guessed it will allow us to parse the json we get back.

I'm kinda flying by the seat of my pants, but the general idea is to access the website, get the json and parse it, so lets go with that:

`        @response = JSON.parse(HTTParty.get("http://reddit.com/r/pics/top/.json?limit=25").body)
`

This might seems like a lot, but if you break it down its not too bad. working backwards, we make a http request to reddit.com to the subreddit *pics* for the json version of the top 25 posts for that day. The we parse the json and assign all of our fancy work to the variable @response.

now lets throw this into a loop, so we can loop through each subreddit and get the top 25 posts for each one, really putting that last part to work.

```
        if true
            attrs = @response['data']['children'].map do |child | {
            category: category,
            link: child['data']['url'],
            title: child['data']['title']
          }

        end
        Pic.create!(attrs)
      else
        puts "something went wrong"
      end
```

After the get request is made, its time to start digging into the json for the information we need. Here's a simple if else statement, if all is well, get the url, the category, and title of each of the 25 posts and with this information pass it into a `create method` making a new database entry for later use.

![database entry](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/09/Screen-Shot-2017-09-07-at-14.01.35.png)

now this is great, it works a treat but only for one subreddit, we want loads. Easy! lets create an array, and in it place the names of every subreddit we want to scrape, then from there place all of our code into an `for.each do |loop|`.
something like this:

```
array_of_subreddits = ["pics", "memes", "funny", "oldschoolcool", "memes", "birdswitharms"]
array_of_subreddits.each do |category |
  sleep 10
      @response = JSON.parse(HTTParty.get("http://reddit.com/r/#{category}/top/.json?limit=25"}).body)
      if true
          attrs = @response['data']['children'].map do |child | {
          category: category,
          link: child['data']['url'],
          title: child['data']['title']
        }
      end
      Pic.create!(attrs)
      puts generate_user_agent
    else
      puts "something went wrong"
    end
  end
```

now we're getting somewhere, this allows us to really get some information from the internet. There is one thing however, and it bothered me for ages. This would work perfectly sometimes and other times it would return an error stopping the whole scraping process. it was returning an error randomly and inconsistently:

```
NoMethodError (undefined method `[]' for nil:NilClass):
  app/controllers/home_controller.rb:13:in `block in index'
  app/controllers/home_controller.rb:7:in `each'
  app/controllers/home_controller.rb:7:in `index'
```

But as all things, after enough percaverence, stack overflow and profanity the answer became apparent. The problem was that reddit doesn't really want people scraping all of its information, and because i was making loads of requests as a bot it was stopping my request after the limit was reached. To combat this I needed to make the requests look like they're coming from a real person, and not just one but multiple ones. for this we're going to need another `gem`

```
gem 'random_user_agent'
```

This gem allows us to spoof where the requests are coming from in our http requests. Let's circle back to the original request code and make one alteration:

```
generate_user_agent = RandomUserAgent.randomize

@response = JSON.parse(HTTParty.get("http://reddit.com/r/#{category}/top/.json?limit=25", headers: {"User-Agent" => generate_user_agent}).body)

```

Because I'm too lazy to trigger this whole process of scraping information we're going to need a way to run this process at set times throughout the day. For this I'm going to use the `gem 'rufus-scheduler'` its super easy to setup. Add the gem to the Gemfile and run bundle install. Then in your `config/initializers/` folder add the file scheduler.rb, and add these lines:

```
scheduler = Rufus::Scheduler::singleton

scheduler.every '12h', :first_at => Time.now + 10 do

end
```

here is where we're going to put our code, and have it run twice a day. and just like that we have created a fairly slapdash web scraper to rival even the best. here's a look at the finished code for the scraper:

```
require 'rufus-scheduler'
require 'json'
require 'httparty'
ENV['TZ'] = 'Europe/Dublin'
require './random-user-agent'


scheduler = Rufus::Scheduler::singleton

scheduler.every '12m', :first_at => Time.now + 10 do

  generate_user_agent = RandomUserAgent.randomize
  array_of_subreddits = ["pics", "memes", "funny", "aww", "memes", "birdswitharms"]
  array_of_subreddits.each do |category |
    sleep 10
        @response = JSON.parse(HTTParty.get("http://reddit.com/r/#{category}/top/.json?limit=25", headers: {"User-Agent" => generate_user_agent}).body)

        if true
            attrs = @response['data']['children'].map do |child | {
            category: category,
            link: child['data']['url'],
            title: child['data']['title']
          }

        end
        Pic.create!(attrs)
        puts generate_user_agent
      else
        puts "something went wrong"
      end
    end
end
``` 

That's it, 32 lines of code and potentially thousands upon thousands of individual pieces of content.

I made these changes last night, and accidentally left the cron job to run every 12 minutes, and when I checked this morning was greeted with this:

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/09/Screen-Shot-2017-09-07-at-14.04.49.png)

... Well at least we know it works

[Part 4 - Back to Front](https://breenblog.herokuapp.com/ruby-on-rails-part-4-back-to-front/)






