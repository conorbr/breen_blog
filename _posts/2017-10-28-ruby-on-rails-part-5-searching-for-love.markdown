---
layout: post
title: 'Ruby on Rails: Part 5 - searching for love'
date: '2017-10-28 21:54:27'
tags:
- ruby-on-rails
---

I wasn't sure what to do this time around, but I'm currently on the way to Belfast nuts deep in a spice bag, so lets begin.

Having a site that has loads of pictures is no good unless there's some form of search functionality. so that's today's job. I kinda did this backwards, but its my site, and nobody reads this so sue me.

I started this section on a whim looking through nice input fields and came across [semantic UI](https://semantic-ui.com/) and their lovely [multiple select](https://semantic-ui.com/modules/dropdown.html#multiple-search-selection) dropdown menus, so, against all logic this is where this post begins.

Lets go head and integrate Semantic UI into out project:

```
gem 'semantic-ui-sass'
```
And then in both our application css and js files respectively:

```
@import "semantic-ui";
```
and 
```
//= require semantic-ui

```
And then we just go ahead and create a `<select>` tag with the styling and elements required by semantic UI and then add it it in, piece of cake.

```
<select id="MultipleSelect" multiple="" name="skills" class="ui fluid search dropdown form-control">
  <option value='aww'>aww</option>
  <option value='funny'>funny</option>
  <option value='memes'>memes</option>
  <option value='pics'>pics</option>
  <option value='birdswitharms'>birdswitharms</option>
</select>

<script>
$('.ui.dropdown')
  .dropdown();
</script>
```

Life's too short to go through each `<option>` tag and writing in the content and value so I wrote [this](https://gist.github.com/conorbr/a09ba9662a7cd19154ba0da728a38b96). As the saying goes, why stand when you can sit.


And just like that we have a nifty little dropdown menu that had both search functionality and the capacity for multiple selections, nice.

Having a nice dropdown menu is all well and good but I've kinda shot myself in the foot, with a cannon, while standing on a bag of salt, in mass, during communion. Reading this you can begin to imagine the hacky way I went about this, and to be honest its probably worse than that. So lets see how I shoe-horned a search feature around a dropdown menu.

well first we need to go ahead and consult the controller, because that is whats going to parse the url that comes in when a search is made.

```
def get_url_params
    url_params = request.fullpath.split("?")
    tidy_params = url_params.map { |word| word.gsub('/','') }
    if url_params == ['/']
      tidy_params = ["aww"]
    else
    return tidy_params
  end
end

  def index
    @pictures = Pic.all
    @params = get_url_params
  end
```

So I created a method that will look at the incoming url, find the path and split it into an array every time the `?` deliminator occurs. And if there is nothing just default to the 'aww' category. Now we're getting places because with that we can now pass the `get_url_params` variable into our `index.html.erb`

I went ahead and updated the ruby code in the index page to accept this new array and grab each of the categories requested:

```
<% @pictures.where(category: @params).limit(200).each do |data|%>
  <%= image_tag data.link, :data => { :caption => data.title} %>
<%end%>
```

now, we just need to be able to search our categories. So coming back full circle now we need to actually get the values the user selects when choosing a category, and for that, what else but JavaScript.

```
  <script>
  $("#MultipleSelect").change(function(getUserSearch){
      var category_array = []
      value = $('#MultipleSelect').val().join("?");
      root = window.location.host
      search_result_url = "/?"+ value
      document.getElementById("burl").href = search_result_url;
  });
  </script>
```


so what's happening above is im grabbing the users input, pusing them into an array, the splitting them up with a `?`. Then i go ahead and assign this value to the button i created with the id of `burl`

```
<a href="?aww" id="burl" onclick="window.location.reload(true);" class="btn btn-default">Search</a>

```
Just a side note, I added a `window.location.reload()` function to this button to refresh the page just to avoid anything weird going on when reloading the DOM. It works without it but having this means that nothing weird will happen when doing multiple searches in a row.

now lets take a gander and see what this abomination looks like

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2017/10/Screen-Shot-2017-10-27-at-09.23.23.png)

Aaaaaaand thats it really, it works a treat to my surprise, but the points of failure on this are pretty high. but hey, it's not stupid if it works!

next: [Ruby on Rails - part 6: F*ck it we'll do it live](http://https://breenblog.herokuapp.com/ruby-on-rails-part-6-f-ck-it-well-do-it-live/)   