---
layout: post
title: 'Ruby on Rails: Part 4 - Back to Front'
date: '2017-10-08 13:39:05'
tags:
- ruby-on-rails
---

Apologies for the delay for part 4, I've been off it a while now. Well I guess that's changed, so lets begin.

so we have the skeleton of the backend more or less done, so now we need a way to navigate through the photos on the site. There's load of options for this, but i want this site to be both mobile and desktop friendly. And the ability to swipe as well is a must, just because.

Because I'm generally worse than most days we're going to use a gem. so with out further ado:

```
gem "fotoramajs"
```

[fotorama](http://fotorama.io/) is a nice little image scroller that supports keystrokes, swipe and onscreen buttons its perfect for the purposes of this project. Its also stupid-easy to setup:

In our `application.js` file:
```
//= require fotorama
```

and in our `application.scss` file ([more on that here](http://sass-lang.com/)):

```
 *= require fotorama
```

(I've also added bootstrap but there are a million and one guides on that so I won't dilute the pool any further)

lets put our swanky new image scroller to work. In our `index.html.erb` we're going to call our new javascript library and pass through some arguments. such as height, width etc. Then we're going to loop through and apply this to each one of our photos.

```
<body style="background-color:">
  <div class="slide-container">
    <div class="slide_show">
      <div class="fotorama"
           data-width="100%"
           data-maxwidth="100%"
           data-maxheight="95%"
           data-ratio="16/9"
           data-allowfullscreen="true"
           data-keyboard="true"
           data-nav="thumbs">
           <% @pictures.order("RANDOM()").limit(200).each do |data|%>
              <%= image_tag data.link, :data => { :caption => data.title} %>
            <%end%>
            </div>
          <div>
    </div>
  </div>
</div>
</body>
```

now we have that sorted. As you can see above we have a div and in it we call the `fotorama` class and inside this div we place each of our photos. And for that, you guessed it a for each loop:

```
<% @pictures.order("RANDOM()").limit(200).each do |data|%>
<%= image_tag data.link, :data => { :caption => data.title} %>
```

so for each photo we want to call the `@picture` record to get the `link` for the main image and the `title` for the caption.

I also added a background colour and some extra bits to the `application.scss` file:

```
 @import "bootstrap-sprockets";
 @import "bootstrap";

body{
  background-color: #272727;
}

.z-index-1{
  margin-bottom: 0px !important;
}

.slide-container{
  max-height: 95%;
}

```
Now, lets have a looks and see what this looks like.
 
![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/10/Screen-Shot-2017-10-08-at-14.12.21.png)

Honestly not bad, but it certainly need work. Work needs to be done on the navbar, maybe adding some functionality of choosing a category and maybe a way for users to moderate the site, being able to flag inappropriate photos.

Thats where I'm going to call it today, but stay tuned.

next: [Part 5 - Searching for love](www.breen.ie) 