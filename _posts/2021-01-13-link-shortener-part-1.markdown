---
layout: post
title: 'Link Shortener: Part 1'
subtitle: Whether you're a person with a long link, or a down and out Nigerian prince. This is the blog post for you
image: https://mybigplunge.com/wp-content/uploads/2018/04/url-shortener-theplungedaily.jpg
date: '2021-01-13 23:21:47'
tags:
- ruby
- ruby-on-rails
- projects
---


![](https://mybigplunge.com/wp-content/uploads/2018/04/url-shortener-theplungedaily.jpg)

<br>

**You can find the github repo for this project [here](https://github.com/conorbr/url_shortener)

<br>

Well, It's been some time since my last post. And I'm well aware very few people, if any actually reads these. So lets just awkwardly pretend we didn't see each other and go on about our day.

so!

Lets talk url shorteners, url shorteners are a super handy way to grab a long and cumbersome http link and shorten it. This is great when we want to post a link somewhere there's a strict character limit, have to type something manually or are posing as a Nigerian prince looking for some temporary funds to gain access to their wider fortune.

Link shorteners really are the backbone of the modern age. And what's more they're relativey easy to implement. Just store the long url you want to go to, set an short easy to remember word (or slug) and when the time comes, take the slug and redirect to the corresponding url.

We're going to create an api allows us to CRUD the absolute beans out of a link record and when we ask politely it will redirect us to the target website. To do this we're going to start with Rails.

![](https://fiverr-res.cloudinary.com/images/q_auto,f_auto/gigs/107042204/original/b981cd0ed3a32f6e94a0ab767688e6a8de2dfa59/create-a-rails-api-for-your-application-or-website.png)

Rails has introduced a very handy API only version of itself, that allows you to ignore all of the clunky views and unnecessary boilerplate and create a super lean api that it just so happens lends itself quite well to this very purpose.

~~~ shell
$ rails new link_shortener --api -d postgresql -T && cd link_shortener
~~~

That unsuspecting little line just created our rails api, set the database and for good measure didn't set the default test folders, but more on that later. And for a little added flare it will cd you into the project. But that's just me being fancy.

First things first, lets talk tests. In our gemfile we're going to add the [rspec-rails gem](https://github.com/rspec/rspec).

~~~ ruby
group :development, :test do
  #[...]
  gem 'rspec-rails', '~> 3.5'
end
~~~

This is a little shorthand to require the gem in both your development and test environments.

Now add `factory_bot_rails`, `shoulda_matchers`, `faker` and `database_cleaner` to the :test group.

~~~ ruby
group :test do
  gem 'database_cleaner'
  gem 'factory_bot_rails'
  gem 'faker'
  gem 'shoulda-matchers', '~> 4.0'
end
~~~

Install these gems by running

~~~ shell
$ bundle install
~~~

This adds the following files which are used for configuration:

* .rspec
* spec/spec_helper.rb
* spec/rails_helper.rb

Create a factories directory (factory bot uses this as the default directory). This is where we’ll define the model factories.

~~~ shell
$ mkdir spec/factories
~~~

<hr>

#### Configuration
Configure your `/spec/rails_helper.rb`

~~~ ruby
# This file is copied to spec/ when you run 'rails generate rspec:install'
require 'spec_helper'
require 'rspec/rails'
require 'database_cleaner'

ENV['RAILS_ENV'] ||= 'test'

require File.expand_path('../config/environment', __dir__)

# Prevent database truncation if the environment is production
abort("The Rails environment is running in production mode!") if Rails.env.production?

# require support files
Dir[Rails.root.join('spec/support/**/*.rb')].sort.each { |f| require f }

# [...]
begin
  ActiveRecord::Migration.maintain_test_schema!
rescue ActiveRecord::PendingMigrationError => e
  puts e.to_s.strip
  exit 1
end

# [...]
RSpec.configure do |config|
  # Remove this line if you're not using ActiveRecord or ActiveRecord fixtures
  config.fixture_path = "#{::Rails.root}/spec/fixtures"

  config.use_transactional_fixtures = false

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
    Faker::Name.unique.clear # Clears used values for Faker::Name
    Faker::UniqueGenerator.clear # Clears used values for all generators
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, js: true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end

  config.before(:all) do
    DatabaseCleaner.start
  end

  config.after(:all) do
    DatabaseCleaner.clean
  end

  config.infer_spec_type_from_file_location!

  # Filter lines from Rails gems in backtraces.
  config.filter_rails_from_backtrace!

  config.include FactoryBot::Syntax::Methods

  Shoulda::Matchers.configure do |config|
    config.integrate do |with|
      with.test_framework :rspec
      with.library :rails
    end
  end
end

~~~

Oooft that was a lot. But it's done, its easy from here on out.

<hr>

#### Model

Ok, time to hit the model.

~~~ shell
$ rails g model Link url:string slug:string clicked:integer
~~~

This will result in the mirgration file being generated

~~~ ruby
# db/migrate/[timestamp]_create_links.rb
class CreateLinks < ActiveRecord::Migration[6.0]
  def change
    create_table :links do |t|
      t.string :url
      t.string :slug
      t.integer :clicked, default: 0

      t.timestamps
    end
  end
end
~~~

Looks good, lets get that change into the database

~~~ shell
$ rails db:migrate
~~~

We're test driven so lets go ahead and write the model specs

~~~ ruby
require 'rails_helper'

# Test suite for Link model
RSpec.describe Link, type: :model do
  # Validation tests
  it { should validate_presence_of(:slug) }
  it { should validate_presence_of(:url) }
  it { should validate_numericality_of(:clicked) }
end
~~~

Rspec has some very expressive language, its almost like reading a sentence. And those nifty shoulda matchers we referenced in our gemfile are coming in clutch.

Now these wont pass as is, we're testing model validations that don't exist. So rather than spoil the mood with some red dots lets go ahead and add some validations to the links model

~~~ ruby
class Link < ApplicationRecord
  before_validation :format_url

  validates_presence_of :url
  validates_presence_of :slug

  validates_uniqueness_of :slug

  validates :url, format: URI::DEFAULT_PARSER.make_regexp(%w[http https])
  validates_numericality_of :clicked

  validates_length_of :url, within: 3..255, on: :create
  validates_length_of :slug, within: 1..255, on: :create
end
~~~

now run

~~~ shell
$ bundle exec rspec
~~~

And that's it, all green baby!

<hr>

#### Controller

Now that our models are all setup, let’s generate the controllers.

~~~ shell
$ rails g controller Links
~~~

And while we're working with an API application we're going to be writing [request specs](https://relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec) instead.

Request specs work differently to normal controller tests, they're going to simulate http requests to the application rather than triggering the methods in our controller. And that's exactly what we're looking for.

Lets setup our factory

~~~ shell
$ touch spec/factories/links.rb
~~~

And paste this in

~~~ ruby
FactoryBot.define do
  factory :link do
    url { 'http://www.google.com' }
    slug { Faker::Name.unique.first_name }
  end
end
~~~

and with that, the controller specs. This is a big one, but hang tight we'll go through it.

~~~ ruby
# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'links API', type: :request do

  # Add links
  let!(:links) { create_list(:link, 10) }
  let!(:link) { links.last }
  let!(:link_id) { link.id }

  # Test suite for GET /links
  describe 'GET /links' do
    # make HTTP get request before each example
    before { get '/links', params: {} }
    # json method is a custom method defined in support/requests_spec_helper.rb
    it 'returns links' do
      expect(json).not_to be_empty
      expect(json.size).to eq(10)
    end

    it 'returns status code 200' do
      expect(response).to have_http_status(200)
    end
  end

  # Test suite for GET /links/:slug
  describe 'GET /s/:slug' do
    let(:link_slug) { link.slug }

    before { get "/s/#{link_slug}", headers: headers }

    context 'when the record exists' do
      it 'returns the link' do
        expect(response).to redirect_to %r{\Ahttp://www.google.com}
      end

      it 'returns status code 200' do
        expect(link.reload.clicked).to eq(1)
        # expect response to be a redirect
        expect(response).to have_http_status(302)
      end
    end

    context 'when the record does not exist' do
      let(:link_slug) { '12345' }

      it 'returns status code 404' do
        expect(response).to have_http_status(404)
      end

      it 'returns a not found message' do
        expect(response.body).to match(/Sorry, link not found./)
      end
    end
  end

  # Test suite for POST /links
  describe 'POST /links' do
    # valid payload
    let(:valid_attributes) { { slug: 'test', url: 'www.google.com' } }

    context 'when the request is valid' do
      before { post '/links', params: valid_attributes, headers: headers }

      it 'creates a link' do
        expect(json['slug']).to eq('test')
        expect(json['url']).to eq('http://www.google.com')
      end

      it 'returns status code 201' do
        expect(response).to have_http_status(201)
      end
    end

    context 'when the slug is taken' do
      let(:link) { create(:link, slug: 'test') }
      before { post '/links', params: valid_attributes, headers: headers }

      it 'creates a link with new slug' do
        expect(json['slug'][0..3]).to eq('test')
        expect(json['url']).to eq('http://www.google.com')
      end

      it 'returns status code 200' do
        expect(response).to have_http_status(201)
      end
    end

    context 'when the request is invalid' do
      let(:invalid_attributes) { { url: nil } }
      before { post '/links', params: invalid_attributes, headers: headers }

      it 'returns status code 422' do
        expect(response).to have_http_status(422)
      end

      it 'returns a validation failure message' do
        expect(json['message']).to eq(['url', "can't be blank"])
      end
    end
  end

  # Test suite for PUT /links/:id
  describe 'PUT /links/:id' do
    let!(:valid_attributes) { { url: 'www.yahoo.com' } }

    context 'when the record exists' do
      before { put "/links/#{link_id}", params: valid_attributes, headers: headers }

      it 'updates the record' do
        expect(json['url']).to eq('http://www.yahoo.com')
      end

      it 'returns status code 204' do
        expect(response).to have_http_status(200)
      end
    end
  end

  # Test suite for DELETE /links/:id
  describe 'DELETE /links/:id' do
    before { delete "/links/#{link_id}", params: {}, headers: headers }

    it 'returns status code 204' do
      expect(response).to have_http_status(200)
    end
  end
end
~~~

We start by populating the database with a list of 10 link records (thanks to factory bot).
We also have a custom helper method `json` which parses the JSON response to a Ruby Hash which is easier to work with in our tests.
Let’s define it in `spec/support/request_spec_helper`.

Add the directory and file:

~~~ shell
$ mkdir spec/support && touch spec/support/request_spec_helper.rb
~~~

~~~ ruby
# frozen_string_literal: true

module RequestSpecHelper
  # Parse JSON response to ruby hash
  def json
    JSON.parse(response.body)
  end
end

~~~

The support directory is not auto-loaded by default. To enable this, open the rails helper and comment out the support directory auto-loading and then
include it as shared module for all request specs in the RSpec configuration block.

Run the tests and you'll see a sea of red. But that's ok because we're going to start writing code that actually does stuff.

![](https://media.giphy.com/media/bg1MQ6IUVoVOM/source.gif)

<hr>

#### Conclusion

So a quick recap, we've created a ruby on rails api, created the models, and of course setup our test suite. In [part 2](https://breenblog.herokuapp.com/link-shortener-part-2/) we'll create the routes, version our api and start creating links!
