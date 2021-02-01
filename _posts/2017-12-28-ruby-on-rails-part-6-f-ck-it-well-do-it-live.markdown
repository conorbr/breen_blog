---
layout: post
title: 'Ruby on Rails: Part 6 - f*ck it we''ll do it live'
date: '2017-12-28 22:34:10'
tags:
- ruby-on-rails
---

So It's Christmas, and my liver looks looks like it's done 12 rounds with Tyson, so lets begin.

I was going to go ahead and delve deep into the realm of some sort of site wide self moderation for the images. But that would require more brainpower than my body is currently capable, so we're going to host the site instead. Another one to add to the long list of mistakes I've made.

This is the first time I've set up a web server, I mean really set one up. No hand holding, no Heroku, just balls-to-the-wall linux server and all the heartache and pride that goes with it.

Lets start with the service: 
The service I decided to go with is [Digital Ocean](https://www.digitalocean.com/) for the sole reason that I had a bunch of credit for free because I'm still a student. Its also pretty good, and has great scalability and is relatively cheap so long as you keep an eye on your traffic. So I went ahead and set up a Ubuntu 16.04 server with 1gb of ram and 30gb of storage and created a new super user called `deploy`

first things first, lets install ruby on our new server.

```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev nodejs yarn
```

Im also going to use [rvm](https://rvm.io/).
```
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 2.5.0
rvm use 2.5.0 --default
ruby -v
```
Last thing we need to do is install bundler
```
gem install bundler
```

I was reading on goRails that Phusion, the company that develops Passenger recently put out an ubuntu package that ships with Passenger pre-installed. So lets go with that

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates

# Add Passenger APT repository
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update

# Install Passenger & Nginx
sudo apt-get install -y nginx-extras passenger
```
And just like that you can start nginx
```
sudo service nginx start
```
Next we need to point passenger to the version of ruby we're using, so open up nano (or vim if your so inclined)
```
sudo nano /etc/nginx/nginx.conf
```
And uncomment these lines:
```
##
# Phusion Passenger
##
# Uncomment it if you installed ruby-passenger or ruby-passenger-enterprise
##

include /etc/nginx/passenger.conf;
```
save and exit, then open the file below with nano
```
sudo nano /etc/nginx/passenger.conf
```
And change the passenger _ruby to use the right version. Heads up because I screwed this up, if you type in `which ruby` into the command line it will give you the correct file path.
```
passenger_ruby /home/deploy/.rbenv/shims/ruby; # If you use rbenv
# passenger_ruby /home/deploy/.rvm/wrappers/ruby-2.1.2/ruby; # If use use rvm, be sure to change the version number
# passenger_ruby /usr/bin/ruby; # If you use ruby from source
```

Next we need to do our database, Im using PostgreSQL for this, so if your following this for yourself (which I hope your not) you have to too.

Lets get Postgres
```
sudo apt-get install postgresql postgresql-contrib libpq-dev
```
And lets create a Super User to create our database
```
sudo su - postgres
createuser --pwprompt deploy
createdb -O deploy my_app_name_production # change "my_app_name" to your app's name which we'll also use later on
exit
```
Ok, now you're beginning to see why I regret doing this, it takes ages. But I am in blood stepped in so far that should I wade no more, Returning were as tedious as go o'er.

Back to the slog, run this baby to get generate the required Capistrano files
```
cap install STAGES=production
```
Go ahead and add these lines into your swanky new `capfile`
```
require 'capistrano/rails'
require 'capistrano/passenger'

# If you are using rbenv add these lines:
# require 'capistrano/rbenv'
# set :rbenv_type, :user
# set :rbenv_ruby, '2.5.0'

# If you are using rvm add these lines:
# require 'capistrano/rvm'
# set :rvm_type, :user
# set :rvm_ruby_version, '2.5.0'
```
Now that we have capistrano installed we need to setup our `config/deploy.rb` file
```
set :application, "my_app_name"
set :repo_url, "git@example.com:me/my_repo.git"

set :deploy_to, '/home/deploy/my_app_name'

append :linked_files, "config/database.yml", "config/secrets.yml"
append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "vendor/bundle", "public/system", "public/uploads"
```
And in our  `config/deploy/production.rb` were going to add
```
# Replace 127.0.0.1 with your server's IP address!
server '127.0.0.1', user: 'deploy', roles: %w{app db web}
```

Oh my god this is taking ages, its ok though, final hurdle (almost)
In our `/etc/nginx/sites-enabled/default` file back on the server we're going to add these configuration settings
```
server {
        listen 80;
        listen [::]:80 ipv6only=on;

        server_name mydomain.com;
        passenger_enabled on;
        rails_env    production;
        root         /home/deploy/my_app_name/current/public;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```
We're so close now, Ive ordered Chinese so lets wrap this up. Its time to connect the database. Lets remove the database files from the repo and put them on the server manually. Look at me thinking of security.
```
echo "config/database.yml\nconfig/secrets.yml" >> .gitignore
git add .gitignore
git mv config/secrets.yml config/secrets.yml.example
git mv config/database.yml config/database.yml.example
git commit -m "Only store example secrets and database configs"
cp config/secrets.yml.example config/secrets.yml
cp config/database.yml.example config/database.yml
```
SSH back into the server and make your way to `/home/deploy/my_app_name/shared/config/database.yml`
and 
```
production:
  adapter: postgresql
  host: 127.0.0.1
  database: my_app_name_production
  username: deploy
  password: YOUR_POSTGRES_PASSWORD
  encoding: unicode
  pool: 5
```
Now for the `secrets.yml` file. Create the file `/home/deploy/my_app_name/shared/config/secrets.yml` and Back on your development machine run `rake secret` and pate that below where it says YOUR_SECRET_KEY. 
```
# /home/deploy/my_app_name/shared/config/secrets.yml
production:
  secret_key_base: YOUR_SECRET_KEY
```

Now on your machine run `cap production deploy` and watch with bitten lip every line run down that page. Restart your server if needs be and you should be good to go! you have a website running live on a production server.

Next time Part 7 - I give up programming

ppffttt who are we kidding, next time [Part 7 - The Mailer!](http://breen.ie)

