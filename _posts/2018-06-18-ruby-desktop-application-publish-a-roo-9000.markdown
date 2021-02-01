---
layout: post
title: Ruby Desktop Application - Publish-a-roo 9000
subtitle: A ruby desktop application, why not?
date: '2018-06-18 20:45:15'
tags:
- ruby
- misc
- misc-2
- projects
---

This project, like many was born from laziness and my general loathing for the mundane and the repetitive. It began its life as a humble command line program for publishing tasks. Known simply as the publish-a-roo, because of it's proficiency for lessening my workload it grew organically. And like some kind of grotesque coded caterpillar, it soon developed into a beautiful GUI Desktop application butterfly.

Anyone reading this outside the team I worked with in Microsoft will almost certainly have no interest in it, but it is the first Desktop application I have created using Ruby so I thought I'd do quick write up on it.

[Shoes](http://shoesrb.com/) is a handy, albeit turbulent way of creating a GUI desktop application using Ruby. It's actually pretty cool, like most ruby projects it's open source, but much of the credit belongs to the (once) elusive [_why](https://en.wikipedia.org/wiki/Why_the_lucky_stiff), who has been maintaining it since 2012.

The problem was that working in Microsoft I was tasked with using as CMS, which I soon learned to be just an abbreviation for the ways in which it made me feel; Cantankerous, Melancholy and Spiteful. Within this CMS, navigation was the most time consuming aspect of making changes, picture walking waist deep, through a pool of molasses. I needed a new way, and so the Publish-a-roo _9000_ was born

Lets begin using shoes:

I'm going to skip the [Download and Installation](http://shoesrb.com/manual/Installing.html) and get straight into it:

```
Shoes.app(title: "Publish-a-roo 9000", width: 280) do
	para("hello world")
end
```

Here's the bare bones GUI, with a title and a width value and string.

![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/pbr_blog.PNG)

Exciting stuff, lets go wild and add in some input fields and buttons, maybe even a cheeky dropdown menu and assign them values for later.

```
Shoes.app(title: "Publish-a-roo 9000", width: 280) do
	stack(margin: 30) do
		@locale = para " ", hidden: true
		@bids = para " ", hidden: true
		@destination = para " ", hidden: true

		para "Enter Locale:"
		@input_1 = edit_line do |a|
			@locale.text = @input_1.text
		end

		flow do
			@multi_locale = check
			para "multiple locales"
		end

		para "Enter BIDs"
			@input_2 = edit_box do |b|
			@bids.text = @input_2.text
		end

		 para "Choose a destination:"
		@input_3 = list_box items: ["a value","another value"."one here too"] do |c|
			where_to = @input_3.text
		end

		@button = button("open link", margin: 40) do
			# something cool
		end
end
```

Which should give us something like this:
![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/Capture2.PNG)

Now we need to just stick in the logic (and straighten that annoying off-center button) and we're all set. The idea behind this program is to take in multiple Big IDs or multiple locales and loop through each of them piecing together a URL. Then what we want to do is to simply open that URL as a new window in the users default browser. Easy enough to do with the variables we created earlier and a [case statement](https://www.skorks.com/2009/08/how-a-ruby-case-statement-works-and-what-you-can-do-with-it/).

So without further adieu, the final code:

```
Shoes.app(title: "Publish-a-roo 9000", width: 280) do
	stack(margin: 30) do
		@locale = para " ", hidden: true
		@bids = para " ", hidden: true
		@destination = para " ", hidden: true

		para "Enter Locale:"
		@input_1 = edit_line do |a|
			@locale.text = @input_1.text
		end

		flow do
			@multi_locale = check
			para "multiple locales"
		end

		para "Enter BIDs"
			@input_2 = edit_box do |b|
			@bids.text = @input_2.text
		end

		 para "Choose a destination:"
		@input_3 = list_box items: ["product storytelling", "pdp storytelling", "live site", "market data", "published products", "global settings",  "bundle data", "assets", "pdp history", "pdp history" , "broadcast tempalte", "preview", "publishing", "notes" ] do |c|
			where_to = @input_3.text
			case where_to
			when "pdp storytelling"
				@destination.text = "storytelling"
			when "product storytelling"
				@destination.text = "details"
			when "live site"
				@destination.text = "live"
			when "global settings"
				@destination.text = "global"
			when "market data"
				@destination.text = "market"
			when "bundle data"
				@destination.text = "bundle"
			when "assets"
				@destination.text = "assets"
			when "pdp history"
				@destination.text = "history"
			when "broadcast tempalte"
				@destination.text = "broadcasting"
			when "preview"
				@destination.text = "preview"
			when "publishing"
				@destination.text = "publish"
			when "notes"
				@destination.text = "notes"
			when "published products"
				@destination.text = "published_products"
			else
				@destination.text = "storytelling"
			end
		end

		@button = button("open link", margin: 40) do
			if @multi_locale.checked? && @destination.text.to_s != "live" && @destination.text.to_s != "published_products"
					locales = @locale.text.split(" ")
					locales.each do |locale|
					bid = @bids.text.to_s
					destination = @destination.text.to_s
					#system("start https://www.microsoft.com/#{locale}/#{bid}/b/#{destination}")
					system("start https://sftools.trafficmanager.net/store/#{locale}/products/#{bid}/#{destination}")
				end
			elsif @destination.text.to_s == "live"
					if @multi_locale.checked?
						locales = @locale.text.split(" ")
						locales.each do |locale|
						bid = @bids.text.to_s
						destination = @destination.text.to_s
						#system("start https://www.microsoft.com/#{locale}/#{bid}/b/#{destination}")
						system("start https://www.microsoft.com/#{locale}/store/d/pbr_9000/#{bid}")
					end
						else
							bids = @bids.text.split(" ")
							bids.each do |bid|
							locale = @locale.text.to_s
							system("start https://www.microsoft.com/#{locale}/store/d/pbr_9000/#{bid}")
							end
						end
					elsif @destination.text.to_s == "published_products"
						if @multi_locale.checked?
							locales = @locale.text.split(" ")
							locales.each do |locale|
							bid = @bids.text.to_s
							destination = @destination.text.to_s
							#system("start https://www.microsoft.com/#{locale}/#{bid}/b/#{destination}")
							system("start https://sftools.trafficmanager.net/store/#{locale}/published-products/#{bid}/storytelling")
						end
						else
						bids = @bids.text.split(" ")
						bids.each do |bid|
						locale = @locale.text.to_s
						system("start https://sftools.trafficmanager.net/store/#{locale}/published-products/#{bid}/storytelling")
						end
					end
			else
			bids = @bids.text.split(" ")
			bids.each do |bid|
				locale = @locale.text.to_s
				destination = @destination.text.to_s
			#system("start https://www.microsoft.com/#{locale}/#{bid}/b/#{destination}")
			system("start https://sftools.trafficmanager.net/store/#{locale}/products/#{bid}/#{destination}")
				end
			end
		end
	end
	para(
	link("click here for help").click do
		system("start https://github.com/conorbr/Publish-a-roo-9000#publish-a-roo-9000")
  end
)

end
```

And that's it, with this certain tasks that required and hour to do now could be completed in under 20 minutes. It's certainly not the most elegant piece of code you'll ever see but for the purposes of the work we were doing it worked just fine for myself and my team.

A gif of it working:
![alt](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2018/06/2018-06-18_21-02-00.gif)

