---
layout: post
title: 'Ruby on Rails: Part 2 - Putting the MC in MVC'
date: '2017-09-06 22:03:35'
tags:
- ruby-on-rails
---

We're flying along now. So far we have our rails environment set up, we have the site created and our code stored on GitHub. Now it's time to get down to brass tax.

We want to make a website that will scrape images from the internet and display them to the user in a format that is easy for the user's consumption.
A quick mental check list for this project would be:

* controller
* model for database
* code for scraping
* front end framework

The model is the way in which the data is kept, so this is going to take some thought as it will be the format in which every image will be stored. Every image will have a link, a title, date added, a category as well as a unique id. That seems quite comprehensive so lets go with that.
In our trusty terminal we use the rails command:

`rails generate controller home index`

this will generate the home controller, which will I will then set as the root controller, meaning when someone searches the website they will be brought to this page. To do this we need to change some things in the routes.rb file.

```
Rails.application.routes.draw do
  get 'home/index'

  # The priority is based upon order of creation: first created -> highest priority.
  # See how all your routes lay out with "rake routes".

  # You can have the root of your site routed with "root"
  root 'home#index'

  # Example of regular route:
  #   get 'products/:id' => 'catalog#view'
```

Now when we search `localhost:3000/home` We're brought to our lovely new *index.htlm.erb* page that was generated with the controller. It'll look a little something like this:

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/09/Screen-Shot-2017-09-06-at-22.36.00.png)

Ok, so we've got the controller done, landing page sorted and we haven't broken a sweat. Lets strike while the iron is hot and sort the model. 

The model is the structured data, the view is the pretty way you can interact with that data and the controller is the glue that connects the two. currently we have the view and the controller, all we need now for a bare bones MVC is the model.

Head over to the terminal and we'll use rails to create the model:

`rails generate model Pic title:string link:string category:string date_added:datetime extension:string`
```
$ bin/rails generate model Pic
      invoke  active_record
      create    db/migrate/20120528062523_create_pic.rb
      create    app/models/pic.rb
      invoke    test_unit
      create      test/models/pic_test.rb
      create      test/fixtures/pic.yml
```
This handy-af command created the model with the arguments i want in one fell swoop. If we go over to the schema.rb we can see the model that we have just generated.

```
ActiveRecord::Schema.define(version: 20170902155544) do

  create_table "pics", force: :cascade do |t|
    t.string   "title"
    t.string   "link"
    t.string   "category"
    t.string   "extension"
    t.datetime "date_added"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end
```

That's it really, the next thing to do is to start figuring out a way to populate our database.

[Part 3: The Big Scrape](https://breenblog.herokuapp.com/ruby-on-rails-part-3-the-big-scrape/)

