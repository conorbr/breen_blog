---
layout: post
title: URL shortener - Part 2
subtitle: 'The url shortener part 2 aka: url shortner reloaded aka: url shortener Tokyo Drift.'
date: '2021-01-14 11:57:08'
---

![](https://mybigplunge.com/wp-content/uploads/2018/04/url-shortener-theplungedaily.jpg)


Last time we left off with a bunch of failing specs, so lets begin. Our most pressing issue right now is that we're trying to test our requests against routes that don't exist. So that's a good place to start.


~~~ ruby
Rails.application.routes.draw do
  resources :links
  resources :redirect

  get '/s/:slug', to: 'redirects#index'
end
~~~

Now we can take a look at our new routes

~~~ shell
$ rails routes
~~~

Just beautiful. Now we have a list of all of the endpoints available to us. Not that they do anything mind you, but patience is a virtue.

...A virtue I don't have, here's the links controller

~~~ ruby
class LinksController < ApplicationController
  before_action :set_link, only: %i[show update destroy show]

  def index
    @links = Link.all
    json_response(@links)
  end

  def show
    json_response(@link, :ok)
  end

  def create
    @link = Link.new(link_params)
    @link.update_attribute(:slug, generate_slug(link_params[:slug]))

    return json_response(@link, :created) if @link.save

    json_response({ message: @link.errors.first }, :unprocessable_entity)
  end

  def update
    @link.update(link_params.except(:slug))
    @link.update_attribute(:slug, generate_slug(link_params[:slug])) if link_params[:slug]

    return json_response(@link, :ok) if @link.save

    json_response({ message: @link.errors.full_messages }, 422)
  end

  def destroy
    @link.delete
    json_response({ message: Message.deleted_link }, :ok)
  end

  private

  def set_link
    @link = Link.find(link_params[:id])

    json_response({ message: Message.not_found('link') }, :not_found) unless @link.present?
  end

  def link_params
    params.permit(:slug, :url, :id)
  end

  def generate_slug(slug_param = nil)
    slug = slug_param || SecureRandom.uuid[0..5]

    # if slug is taken add random characters to the end
    Link.find_by(slug: slug) ? slug.concat(SecureRandom.uuid[0..2]) : slug

    # if slug is still taken call self again
    Link.find_by(slug: slug) ? generate_slug(slug_param) : slug
  end
end
~~~

and redirects

~~~ ruby
  class RedirectsController < ApplicationController
    before_action :set_link

    def index
      @link.update_attribute(:clicked, @link.clicked + 1)
      redirect_to @link.url
    end

    private

    def set_link
      @link = Link.find_by(slug: link_params[:slug])

      json_response({ message: Message.not_found('link') }, :not_found) unless @link.present?
    end

    def link_params
      params.permit(:slug)
    end
  end
~~~

Notice the next helper `json_response`. This nifty little method will do just what it says on the tin. It will send a json response containing a message and a response code.

~~~ ruby
module Response
  def json_response(object, status = :ok)
    render json: object, status: status
  end
end
~~~

and finally reference this in our application controller so we can access it in our controllers

~~~ ruby
class ApplicationController < ActionController::API
  include Response
end
~~~

Lets see what this puppy can do, I'm using a handy little tool called [httpie](https://httpie.io/) to make requests. with zero effort. I will happily say goodbye to curl.

![](https://s2.gifyu.com/images/recordingbdc1b787cfeb1ab8.gif)

now that's just cool. The rest of the endpoints should work too.

~~~ shell
# GET /links
$ http :3000/links
# POST /links
$ http POST :3000/links url=google.ie slug=google
# PUT /links/:link_id
$ http PUT :3000/links/1 url=google.com
# DELETE /links/:link_id
$ http DELETE :3000/links/1
~~~

with our endpoints looking fly, we're almost done. One very important thing we need to consider is versioning. Our controller works great, but do did horses in the 1800s. We can make a pretty safe bet that in the future we'll want to try something new. And changing our api endpoints could break functionality for other people using our api. Lets talk versioning.

A great way to make sure we always have room to expand our functionality without removing what's currently there is versioning. Lets say our current api is `v1` and make it the default version. Then we can create a second version, and only use it when we ask for it. 

lets give it a go.

lets create a class called `ApiVersion` that will check requests coming in for the version, and act accordingly.

~~~ shell
$ touch app/lib/api_version.rb
~~~

~~~ ruby
class ApiVersion
  attr_reader :version, :default

  def initialize(version, default = false)
    @version = version
    @default = default
  end

  # check whether version is specified or is default
  def matches?(request)
    check_headers(request.headers) || default
  end

  private

  def check_headers(headers)
    # check version from Accept headers; expect custom media type `links`
    accept = headers[:accept]
    accept&.include?("application/vnd.links.#{version}+json")
  end
end

~~~

so, according to the Media Type Specification, you can define your own media types using the vendor tree i.e. application/vnd.example.resource+json.

We're going to follow this specification for our api. Here we define a custom vendor media type `application/vnd.links.{version_number}+json` giving clients the ability to choose which API version they require. No need to add `/v2/` to the url. Pretty cool right.

Now we can go ahead and update our routes to have multiple versions

~~~ ruby
Rails.application.routes.draw do

  # namespace the controllers without affecting the URI
  scope module: :v1, constraints: ApiVersion.new('v1', true) do
    resources :links
    resources :redirect

    get '/s/:slug', to: 'redirects#index'
  end
end
~~~

We've set the version constraint at the namespace level. Handily, this will be applied to all resources within it. We've also defined `v1` as the `default` version for cases where the version is not provided, the API will default to `v1`.

In the event we were to add new versions, they would have to be defined above the default version since Rails will just cycle through all routes from top to bottom searching for one that matches(till method `matches?` resolves to `true`).

lets give our controllers a new home

~~~ shell
$ mkdir app/controllers/v1
~~~

And move them in

~~~ shell
$ mv app/controllers/{links_controller.rb,redirects_controller.rb} app/controllers/v1
~~~

That's not all, lets define the controller in the v1 namespace

~~~ ruby
module V1
  class linksController < ApplicationController
  # [...]
  end
end
~~~

and the redirectsController

~~~ ruby
module V1
  class RedirectsController < ApplicationController
  # [...]
  end
end
~~~

Now lets take a beat and see what works and what doesn't

~~~ shell
# get links from API v1
$ http :3000/links Accept:'application/vnd.links.v1+json' 
# attempt to get from API v2
$ http :3000/links Accept:'application/vnd.links.v2+json'
~~~

In case we attempt to access a nonexistent version, the API will default to v1 since we set it as the default version. For testing purposes, let's define v2.

Generate a v2 links controller

~~~ shell
$ rails g v2/links
~~~

define a namespace 

~~~ ruby
#config/routes.rb
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html

  # module the controllers without affecting the URI
  scope module: :v2, constraints: ApiVersion.new('v2') do
    resources :links, only: :index
  end

  scope module: :v1, constraints: ApiVersion.new('v1', true) do
    # [...]
  end
  # [...]
end
~~~

Since this is test controller, we'll define an index controller method with a dummy response.

~~~ ruby
module V2
  class LinksController < ApplicationController
    def index
      json_response({ message: 'Its always important to version an API' })
    end
  end
end
~~~

Note: ruby will generate the shorthand for our class structure 
`class V2::LinksController < ApplicationController`
you can use either or, I prefer the long hand.

Great! now lets fire out some tests

~~~ shell
# get links from API v1
$ http :3000/links Accept:'application/vnd.links.v1+json' 
# attempt to get from API v2
$ http :3000/links Accept:'application/vnd.links.v2+json'
~~~

looking good.

<br>
<hr>
#### Optional Extra:


This is where I was going to call it a day, but I've had my coffee and am feeling generous. So lets do a BONUS ROUND!

I can see that we return the slug and the url but not them together. We could make a record in the table and save them combined, but then again we already have them. So what else can we do?

![](https://i.imgflip.com/4ts9wz.jpg)

Lets go ahead and add the rails serializer gem

~~~ ruby
gem 'active_model_serializers', '~> 0.10.0'
~~~

bundle it

~~~ shell
$ bunde install
~~~

Generate the serializer for the link model

~~~ shell
$ rails g serializer link
~~~

This creates a new directory `app/serializers` and adds a new file `link_serializer.rb`. Let's define the link serializer with the data that we want it to contain.

~~~ ruby
class LinkSerializer < ActiveModel::Serializer
  attributes :id, :url, :slug, :created_at, :updated_at, :short
end
~~~

What's that there? I don't remember our link model model having an attribute called short. Well look again!

~~~ ruby
# frozen_string_literal: true

class Link < ApplicationRecord
  before_validation :format_url

  validates_presence_of :url
  validates_presence_of :slug

  validates_uniqueness_of :slug

  validates :url, format: URI::DEFAULT_PARSER.make_regexp(%w[http https])
  validates_numericality_of :clicked

  validates_length_of :url, within: 3..255, on: :create
  validates_length_of :slug, within: 1..255, on: :create

  def format_url
    return false unless url.present?

    self.url = "http://#{url}" unless url[%r{\Ahttp://}] || url[%r{\Ahttps://}]
  end

  def short
    url = Rails.env.development? ? 'localhost:3000/s/' : 'example.com/'

    url + slug
  end
end
~~~

Now lets see if we can get our link

![](https://s3-eu-west-1.amazonaws.com/breenblogbucket/2021/01/Screen-Shot-2021-01-14-at-11.49.33.png)

well

![](https://media.giphy.com/media/R8afzAVZ1zHjy/giphy.gif)

<hr>

#### Conclusion

We now have a rails 6 api, complete with a full test suite, serializers and a beautifully versioned api. Not too shabby.

We've come a long way, and we've achieved so much. Stay tuned for the next part where I either:

* Implement Users & JWT authentication
* Dockerize the application
* Create a front end using Vue.js
* Leave this repo to collect dust in github

Email me your votes.



